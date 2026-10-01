---
name: silent-failures
description: Find places in the user's code where errors are swallowed or only logged, so nobody hears about them, and propose where BugsRadar should report them. Use when the user asks about silent failures, swallowed exceptions, empty catch blocks, errors nobody reads in the logs, jobs that fail quietly, or asks to audit a project for missing error alerts.
---

# Find silent failures

A failure goes silent when the code survives it. The error is caught, logged or ignored, the program carries on, and no person ever sees the line. The goal of this skill is a short list of such places in the user's code and what to do about each.

## Rules

- This is a read-only audit by default. Report findings; change code only if the user asks.
- Do not open `.env` files, secret stores or the secret parts of configuration. You do not need keys for this task and must never ask for them.
- Keep the report short: the most important places first, each with a file and line, why it is silent and the smallest change that makes it loud.

## Steps

1. Find out what the project is (language, framework, logger) and whether BugsRadar is already connected: search for the BugsRadar package or calls such as `AddBugsRadar`, `bugsradar`, `BugsRadarTransport`. If it is not connected, say so; offer the `setup` skill at the end, do not start it.
2. Search for the patterns in `references/patterns.md` for the project's language. Prioritize the paths that matter: payments and webhooks, imports and exports, background jobs and consumers, scheduled tasks, startup code, retry loops.
3. For each finding decide: is the failure already reported by the logger that BugsRadar listens to (Error and above)? A catch that logs at Warning or Information does not reach BugsRadar, which takes Error and above by default.
4. Group the findings: (a) swallowed with no log, (b) logged below Error, (c) caught and converted to a success response or a default value, (d) fire-and-forget work whose failure nobody observes, (e) scripts without failure handling.
5. For each, propose the smallest fix: log at Error with the exception, call `SendException` / `sendException` / `send_exception`, rethrow, or add the alert to the script (see `script-alerts`). Warn when a fix could create noise (a loop that fails every second is one message with a count, but the cause should still be fixed).
6. End with a ranked list and ask which items to change.

## What BugsRadar adds

The first occurrence of an error arrives at once in the chat with the stack trace, and repeats are counted in that message. It does not store anything and has no dashboard: it makes sure the first signal is a message rather than a user complaint. It also cannot tell that a job never ran at all (no heartbeat).
