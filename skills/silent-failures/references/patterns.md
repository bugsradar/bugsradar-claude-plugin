# Patterns that hide failures

## .NET (C#)

- `catch { }`, `catch (Exception) { }`, or a catch that only assigns a default value.
- `catch (Exception ex) { _logger.LogWarning(...) }` or `LogInformation` / `LogDebug`: below Error, so BugsRadar's ILogger provider (Error and above) does not see it.
- `catch (Exception ex) { return Ok(); }` or a handler that turns an exception into a success response.
- `async void` methods and `Task.Run(...)` or un-awaited tasks whose exceptions nobody observes.
- `BackgroundService.ExecuteAsync` loops that catch everything and continue.
- `Parallel.ForEach` / `Task.WhenAll` where only the first exception is read.
- Middleware that swallows exceptions and writes a generic page.
- Report with `ILogger.LogError(ex, "Order {OrderId} failed", id)` or `_bugsRadar.SendException(ex, "Where : What")`.

## Node.js / TypeScript

- `catch (e) {}` or `catch { }`; `.catch(() => {})` or `.catch(console.log)`.
- Promises that are neither awaited nor returned (fire-and-forget); `forEach(async ...)`.
- Event handlers and `setInterval` / `setTimeout` callbacks with their own try/catch that only logs.
- Express handlers that catch and `res.status(200).json(...)`, or error handlers that log below `error`.
- Consumer loops (`for await`, queue clients) that catch everything and continue.
- Report with `bugsRadar.sendException(error, { module: 'Orders' })`, the Express error handler, or `logger.error('...', { error })` with the winston or pino transport.

## Python

- `except: pass`, `except Exception: pass`, `except Exception as e: print(e)`.
- `logger.warning(...)` / `logger.info(...)` in an except block, or `logger.error(str(e))` without `exc_info` or `logger.exception`.
- `contextlib.suppress(Exception)`.
- Threads and `asyncio.create_task(...)` whose exceptions nobody awaits.
- Celery / RQ tasks and consumers that catch everything and acknowledge the message.
- Report with `logger.exception("Order %s failed", order_id)` or `bugsradar.send_exception(error, module="Orders")`.

## Shell and PowerShell scripts

- No `set -e` / `set -o pipefail` (a failed `mysqldump | gzip` looks like success); commands whose exit code is never checked.
- `command || true`, `2>/dev/null` or `-ErrorAction SilentlyContinue` hiding the only signal.
- PowerShell: no `$ErrorActionPreference = 'Stop'`; `$LASTEXITCODE` never checked after a native program.
- Cron jobs, systemd services and scheduled tasks with no failure handling (see the `script-alerts` skill).

## Everywhere

- Retry loops that retry forever and never report the final failure.
- Health endpoints that always return 200.
- Dead-letter queues nobody reads.
- Imports and exports that skip bad rows and finish "successfully".
- Webhook handlers that return 500 after catching the error, so the provider retries and nobody learns why.
