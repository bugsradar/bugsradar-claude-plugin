---
name: troubleshoot
description: Find out why a BugsRadar message did not arrive or why alerts look wrong. Use when a test error or alert does not show up in the chat, when the notify endpoint or a package answers 400, 401, 413 or 429, when a channel is turned off, when the count of repeats looks wrong, or when messages arrive in the wrong chat.
---

# Why did the message not arrive?

Work from the sender to the chat. Ask the user only for what you need, and never ask for, print or copy a key: when a key might be wrong, ask the user to check it themselves.

## 1. Is the request accepted?

If the sender is a script or direct HTTP call, look at the status code the sender got (anything but 202 is a failure). `references/status-codes.md` lists the codes for the notify and CLEF endpoints.

- **401**: the key is missing or wrong. Ask the user to check the variable or secret the code reads (name, no extra spaces or quotes, the right project). The Check API Key page in the web app shows which project a key belongs to.
- **400**: the body is empty (notify) or has no CLEF line (CLEF).
- **413**: the body is too large (64 KB for notify, 1 MB for CLEF).
- **429**: wait for `Retry-After`. On CLEF the Free and Pro plans allow 50 requests per minute per project.
- **202 or 201** but nothing in the chat: continue below.

If the sender is a package, its own delivery problems are logged, never thrown:

- .NET: ILogger category `BugsRadar.Client` (Serilog sink: Serilog's SelfLog; NLog: the internal log; log4net: its internal log).
- Node.js: warnings on the console.
- Python: warnings of the `bugsradar` logger.

## 2. Is the error actually sent?

- The packages send Error and above by default (`MinimumLevel`, `logging_level`, the sink level). A `LogWarning` does not arrive.
- Is BugsRadar registered at startup, and is the key not empty (an empty key throws when the client is created)?
- A short-lived process may exit before the report left: call `FlushAsync` / `flush()` before exit.
- CLEF: only Error and Fatal events go to channels; Information and Warning are accepted and dropped.

## 3. Does the chat get it?

- **Channel turned off.** If the service keeps refusing (token revoked, bot removed from the chat, webhook deleted), BugsRadar turns the channel off, marks it "Turned off by BugsRadar" with the last error and drops the waiting messages. After fixing the cause, press Check again on the channel in the web app. Send test on the channel sends a test message to it.
- **No channel on the project**, or the channel is Paused (the On / Paused switch is the user's).
- **Filters (Pro plan).** A channel takes an error only when it fits every condition: minimum level, modules, environments. An error with no environment fits no filter that names environments. An error that fits no channel goes nowhere; the project's page warns when every working channel has a filter. When the Pro plan ends, filters count as extras of Free and error alerts are paused until the filters are removed or the plan is renewed.
- **Wrong project.** The key decides the project, and so the chats. Check which project the key belongs to in the web app.

## 4. Is it counted instead of sent?

Identical messages are counted in the first one rather than sent again; numbers, ids, URLs, email addresses and quoted text are ignored when messages are compared. The count is written into the first message (Telegram, Discord, silently) 10 minutes later, then after 30 minutes, an hour, and every 6 hours; Pushover gets summaries. If Live count is off for the channel, each count arrives as a new message. When the chat cannot keep up, waiting errors arrive bundled (a line per error, no stack traces). See `read-alert` and `write-good-alerts`.

## Still stuck

Write to support@bistriy.com and include what the sender got: the status codes, or the lines the package logged. Never include the key.
