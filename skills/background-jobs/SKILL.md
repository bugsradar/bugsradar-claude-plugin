---
name: background-jobs
description: Make background jobs, workers, queue consumers and job runners report their failures to BugsRadar, with retries counted instead of flooding the chat. Use when the user has workers, consumers, schedulers, Hangfire, Quartz, Celery, BullMQ or similar that fail quietly, retry forever, or restart in a crash loop.
---

# Background jobs that fail quietly

A worker throws, the queue retries, the retry throws, the message lands in a dead-letter queue, and the metric that would show it is on a page nobody opens. The goal is that the first failure arrives in the chat with the stack, and the retries are counted in that message.

## Rules

- BugsRadar goes only into code that runs on servers. Never read, ask for or print the project's key; the user puts it in configuration or secrets (the `setup` skill has the steps).
- Do not replace the project's logging; connect BugsRadar to it.

## What to do by platform

- **.NET.** A worker service registers BugsRadar once and adds the ILogger provider: everything the worker logs at Error and above arrives, with the message template and the properties, and an exception that the host logs when the worker crashes arrives too. In a consumer loop, `SendException(error, "Consumer : Handle", module: "Orders")` in the catch.
- **Node.js.** One client per process reports uncaught exceptions and unhandled rejections by itself. Add the winston or pino transport for what is logged, and `sendException` in the catch of the consumer loop.
- **Python.** `init` once; `logger.exception` in the consumer's `except` brings the traceback, and an exception that ends the worker is reported through the exit hook.

## Job runners and schedulers

If the job runner logs a failed job at ERROR through the logger that BugsRadar is connected to (ILogger, winston, pino, Python `logging`), the failure arrives with no code of yours. If it swallows exceptions or logs them elsewhere, report in the job itself: one `SendException`, `sendException` or `send_exception` in the catch, with the job's name as the module. The message then says which job, on which host, with the stack.

## Retries become a count

A job that fails, is retried and fails again is the same error repeating. The first failure arrives in full; the retries are counted in that message, so "×24 since 03:00" says the queue has been retrying for an hour without a flood. Group by what matters: log with a message template (`Job {JobName} failed`), so every failed order is one error with a count, not a message per order.

## Crash loops

A worker that dies and is restarted by its supervisor every few seconds is the worst case for any alerting. The package folds repeats before they leave the process, BugsRadar counts them in one message, and if the chat still cannot keep up the waiting errors arrive bundled. One message with a rising count, not thousands.

## Short-lived jobs

A job that exits right after its work can exit before the report has left. Call `FlushAsync` (.NET), `flush()` (Node.js) or `bugsradar.flush()` (Python; the exit hook does it for you) before the process ends. A job that is not an application, a script, reports with one HTTP request: see `script-alerts`.

## Limits to mention

- BugsRadar reports a failure the job reports; it cannot tell that a job never ran at all (no heartbeat). A success message at the end of a run (`level=information`) makes a missing message visible.
- Nothing is stored and there is no dashboard.
