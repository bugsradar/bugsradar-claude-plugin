# BugsRadar for Python

Source: https://bugsradar.com/docs/python/ (updated 1 October 2026)

One package from PyPI for Python 3.9 and later, with no dependencies outside the standard library: the standard `logging` module, loguru, uncaught exceptions and direct calls. It is for code on servers only: never in a desktop app that is shipped to others (PyInstaller, PySide, PyQt builds).

## Install

```bash
pip install bugsradar
```

## Set it up (once, at the start of the program)

```python
import os
import bugsradar

bugsradar.init(
    api_key=os.environ["BUGSRADAR_KEY"],
    environment="Production",  # optional
)
```

`init` connects BugsRadar to logging and to uncaught exceptions. It adds no handlers, so the logging setup works as before. The key is required: an empty one raises `ValueError`. The user sets the environment variable; do not write the key into the source.

## logging

Records at ERROR and above from every logger go to the channels, whatever handlers the logging setup has. `logger.exception("Order %s failed", order_id)` brings the exception with its traceback, and the format string becomes the message template, so failures of different orders are one error with a count. Another level: `logging_level=logging.WARNING`. To leave logging alone, pass `logging_level=None` and add `bugsradar.LoggingHandler()` to the loggers you choose.

## loguru

loguru writes past the `logging` module, so `init` does not see it. Add the sink after your own loguru setup (`logger.remove()` without an id removes every sink, this one too):

```python
from loguru import logger

bugsradar.init(api_key=os.environ["BUGSRADAR_KEY"])
logger.add(bugsradar.LoguruSink())  # ERROR and above; LoguruSink("WARNING") for more
```

Values bound with `logger.bind` arrive as properties. If logging is routed into loguru with an InterceptHandler, those records are sent once, by `init`.

## Django, Flask, FastAPI

One `init` call is enough:

- Django: in `settings.py`. The `LOGGING` setting stays as it is.
- Flask: next to `app = Flask(__name__)`.
- FastAPI: Uvicorn logs unhandled exceptions to `uvicorn.error`, so `init` is enough. To see the request path as well, report from an exception handler:

```python
@app.exception_handler(Exception)
async def report_error(request: Request, error: Exception):
    bugsradar.send_exception(error, properties={"path": request.url.path})
    return JSONResponse(status_code=500, content={"detail": "Internal Server Error"})
```

## Uncaught exceptions

`init` sets `sys.excepthook` and `threading.excepthook`: the report is sent, `init` waits up to `shutdown_timeout`, then the previous hook runs and the traceback is still printed. `capture_uncaught=False` turns this off.

## Direct calls

```python
try:
    create_order(order_id)
except Exception as error:
    bugsradar.send_exception(error, module="Orders")
```

`send` and `send_exception` queue the event and return at once. Nothing raises; delivery problems are logged as warnings by the `bugsradar` logger, which BugsRadar never sends anywhere.

## Before the program exits

`init` registers an `atexit` hook that waits up to `shutdown_timeout`. In a short script you can wait yourself: `bugsradar.flush()`.

## Arguments of init

| Argument | Default | Meaning |
|---|---|---|
| `api_key` | none | Project API key. Required. |
| `environment` | None | Environment name (Production, Staging). |
| `host` | `socket.gethostname()` | Host for events. |
| `app_version` | None | Application version, shown in messages. Not used for grouping. |
| `logging_level` | `logging.ERROR` | Lowest level of the log records that go to BugsRadar; None leaves logging alone. |
| `capture_uncaught` | True | Report exceptions that end the program or a thread. |
| `repeat_interval` | 5 s | Repeats within this interval travel as one request. |
| `queue_capacity` | 1000 | Events beyond this are dropped with a warning. |
| `shutdown_timeout` | 5 s | How long `flush()`, the exit hook and an uncaught exception wait. |
| `request_timeout` | 15 s | One request to the server. |
| `api_url` | api.bugsradar.com | Change only for a self-hosted BugsRadar. |

## One-off test error

The key is read from the environment, so the value never passes through the assistant:

```bash
python -c "import os, bugsradar; bugsradar.init(api_key=os.environ['BUGSRADAR_KEY']); bugsradar.send_exception(Exception('BugsRadar test error')); bugsradar.flush()"
```
