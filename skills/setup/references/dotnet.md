# BugsRadar for .NET

Source: https://bugsradar.com/docs/dotnet/ (updated 1 October 2026)

NuGet packages for ASP.NET Core, worker services, desktop and console apps. They target .NET Standard 2.0, so modern .NET and .NET Framework both work. The package major version matches the API version (3.x uses API v3). Versions below 3.0.0.1 no longer deliver errors.

## Which package

| Logger in the project | Command |
|---|---|
| Microsoft.Extensions.Logging (ILogger) or no logger | `dotnet add package BugsRadar` |
| Serilog | `dotnet add package BugsRadar.Serilog` |
| NLog | `dotnet add package BugsRadar.NLog` |
| log4net | `dotnet add package BugsRadar.Log4Net` |

The Serilog, NLog and log4net packages bring BugsRadar with them, so the ILogger provider and direct calls are there too.

## Register the client (dependency injection)

```csharp
using BugsRadar.Extensions.DependencyInjection;
builder.Services.AddBugsRadar(configuration =>
{
    configuration.ApiKey = builder.Configuration["BugsRadar:ApiKey"];
    configuration.Environment = builder.Environment.EnvironmentName; // optional
});
```

The key is required: an empty one throws `ArgumentException` when the client is created. The user puts the value in configuration or user secrets; do not write it into the source.

## ILogger provider

```csharp
using BugsRadar.Extensions.Logging;
builder.Logging.AddBugsRadar();
```

Everything logged at Error and above arrives, with the message template, properties, scopes and category. `logger.LogError(ex, "Order {OrderId} failed", 42)` arrives with the template `Order {OrderId} failed` and the property `OrderId = 42`. The minimum level is `MinimumLevel` (Error by default).

## Serilog

```csharp
using BugsRadar.Extensions.Serilog;
Log.Logger = new LoggerConfiguration()
    .WriteTo.BugsRadar(Environment.GetEnvironmentVariable("BUGSRADAR_KEY"))
    .CreateLogger();
```

With dependency injection, sharing the client registered with `AddBugsRadar`:

```csharp
builder.Host.UseSerilog((context, services, configuration) => configuration
    .WriteTo.BugsRadar(services));
```

The sink takes Error and above by default.

## NLog (NLog.config)

```xml
<extensions>
  <add assembly="BugsRadar.NLog" />
</extensions>
<targets>
  <target xsi:type="BugsRadar" name="bugsradar"
          apiKey="${environment:BUGSRADAR_KEY}" environment="Production" />
</targets>
<rules>
  <logger name="*" minlevel="Error" writeTo="bugsradar" />
</rules>
```

`${environment:...}` reads an environment variable; `${configsetting:...}` reads appsettings.json through NLog.Extensions.Logging. With NLog as the host's logger and the client registered with `AddBugsRadar`, leave `apiKey` out. `LogManager.Shutdown()` drains the queue.

## log4net (log4net.config)

```xml
<appender name="BugsRadar" type="BugsRadar.Extensions.Log4Net.BugsRadarAppender, BugsRadar.Log4Net">
  <apiKey value="${BUGSRADAR_KEY}" />
  <environment value="Production" />
</appender>
<root>
  <level value="INFO" />
  <appender-ref ref="BugsRadar" />
</root>
```

The appender takes ERROR and above. log4net has no message templates, so an event without an exception is grouped by its text with numbers, ids and quoted values masked.

## Direct calls

```csharp
public class OrderService
{
    private readonly IBugsRadar _bugsRadar;
    public OrderService(IBugsRadar bugsRadar) { _bugsRadar = bugsRadar; }

    public void CreateOrder(int orderId)
    {
        try { /* ... */ }
        catch (Exception error)
        {
            _bugsRadar.SendException(error, "OrderService : CreateOrder", module: "Orders");
        }
    }
}
```

`Send` and `SendException` queue the event and return at once; nothing throws. Delivery problems are written to ILogger under the category `BugsRadar.Client`. `SendAsync` and `SendExceptionAsync` were removed in 3.0.0.2.

## Console apps and scripts

```csharp
using var bugsRadar = new BugsRadarClient(new BugsRadarConfiguration { ApiKey = apiKey });
bugsRadar.SendException(error);
await bugsRadar.FlushAsync(); // before exit; Dispose also drains the queue
```

Call `FlushAsync` before a short-lived process exits.

## Configuration

| Property | Default | Meaning |
|---|---|---|
| `ApiKey` | none | Project API key. Required. |
| `Environment` | null | Default environment name (Production, Staging). |
| `Host` | machine name | Default host. |
| `AppVersion` | entry assembly version | Application version, shown in messages. Not used for grouping. |
| `MinimumLevel` | Error | Minimum level for the ILogger provider. |
| `RepeatInterval` | 5 s | Repeats within this interval travel as one request. |
| `QueueCapacity` | 1000 | Events beyond this are dropped with a warning. |
| `ShutdownTimeout` | 10 s | How long Dispose waits for the queue. |
| `RequestTimeout` | 15 s | One request to the server. |
| `ApiUrl` | api.bugsradar.com | Change only for a self-hosted BugsRadar. |

## One-off test error

A console snippet that reads the key from the environment, so the value never passes through the assistant:

```csharp
using BugsRadar;
using var client = new BugsRadarClient(new BugsRadarConfiguration
{
    ApiKey = Environment.GetEnvironmentVariable("BUGSRADAR_KEY")
});
client.SendException(new Exception("BugsRadar test error"));
await client.FlushAsync();
```

## Grouping

Repeats of an error are grouped by a fingerprint computed from the exception type and the top frames of the stack, or from the message template. To group by your own key, set `Fingerprint` on the event.
