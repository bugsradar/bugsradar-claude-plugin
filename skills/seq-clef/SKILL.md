---
name: seq-clef
description: Send errors to BugsRadar from a project that already uses Seq or Serilog.Sinks.Seq, with no new package, by adding a second Seq sink that points at BugsRadar's CLEF address. Use when the user already sends logs to Seq, uses Serilog.Sinks.Seq or another Seq or CLEF client, cannot add a package, or asks how to post CLEF events to BugsRadar.
---

# CLEF and Seq clients

BugsRadar takes CLEF, the compact JSON format of Serilog and Seq, and sends the errors in it to the user's channels. No BugsRadar package is needed. The user's own Seq sink stays as it is.

## Rules

- Never read, ask for or print the project's key; the user puts it in an environment variable or configuration. The key belongs only in code that runs on servers.
- If the project can add a package, mention `BugsRadar.Serilog`: it does the same and more (repeats leave as one request with a count; an error seen by both ILogger and Serilog is sent once). Let the user choose.

## Serilog.Sinks.Seq

Add a second `WriteTo.Seq` with the BugsRadar address and the project's key as the API key:

```csharp
using Serilog;
using Serilog.Events;

Log.Logger = new LoggerConfiguration()
    .WriteTo.Seq("https://api.bugsradar.com/api/v3/clef",
        apiKey: Environment.GetEnvironmentVariable("BUGSRADAR_KEY"),
        restrictedToMinimumLevel: LogEventLevel.Error)
    .CreateLogger();
```

The sink adds the path `/api/events/raw` itself. `restrictedToMinimumLevel: LogEventLevel.Error` keeps everything below errors on the user's side: BugsRadar sends only errors to channels, so shipping the rest is pointless.

## Other Seq clients

Seq clients for other loggers and languages take the same address, `https://api.bugsradar.com/api/v3/clef`, and add the path themselves. Anything else can post CLEF directly: one JSON object per line to `https://api.bugsradar.com/api/v3/clef/ingest/clef`, with the key in the `X-Seq-ApiKey` header:

```bash
curl -X POST "https://api.bugsradar.com/api/v3/clef/ingest/clef" \
  -H "X-Seq-ApiKey: $BUGSRADAR_KEY" \
  -H "Content-Type: application/vnd.serilog.clef" \
  --data-binary '{"@t":"2026-09-26T10:00:00Z","@l":"Error","@mt":"Payment {PaymentId} declined","PaymentId":42}'
```

## What BugsRadar takes

- One JSON object per line, up to 1 MB per request.
- Up to 50 requests per minute per project on the Free and Pro plans; the Business and On-Premises plans have no such limit. A Seq client sends one request per batch, so a client that sends only errors stays far below it.
- Error and Fatal events go to the project's channels. Verbose, Debug, Information and Warning are accepted and dropped. An event without `@l` is Information.
- `@mt`, the message template, groups repeats: `Payment {PaymentId} declined` for every payment is one error with a count. Without a template, the text in `@m` is grouped with numbers, ids and quoted values masked.
- `@x`, the exception as .NET prints it, gives the exception type, the inner exceptions and the stack; repeats are grouped by the type and the top frames of the code.
- `SourceContext` becomes the category, `MachineName` the host and `EnvironmentName` the environment. Other properties arrive as properties.

## Answers

| Code | Meaning |
|---|---|
| 201 | The batch is accepted, even if none of its events was an error. |
| 400 | Not one line of the body is a CLEF event. |
| 401 | The API key is wrong. |
| 413 | The body is over 1 MB. |
| 429 | Over 50 requests in a minute on Free or Pro, or too many at once. `Retry-After` says how long; Seq clients wait and resend. |

Source: https://bugsradar.com/docs/clef/
