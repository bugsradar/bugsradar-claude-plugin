---
name: read-alert
description: Explain a BugsRadar message the user pasted (an error alert from a chat or a push notification) and find its cause in the project's code. Use when the user pastes or describes an alert with a project name, a level line, an exception, a place such as Namespace.Class.Method, a count like "×158 total", or a bundle of errors, and asks what it means, where it comes from or how to fix it.
---

# Read a BugsRadar alert

A BugsRadar message is a fixed layout. Read it field by field before looking at code; `references/message-anatomy.md` has the layout with a sample.

## Steps

1. **Identify the kind of message.**
   - First occurrence: the full message with the stack trace.
   - Repeat count: the same message with a line such as `×158 total since ... last ...` (a channel that can edit a message, Telegram and Discord today, edits the first one; a channel that cannot, Pushover today, gets a summary with `×157 more since the last message`).
   - Bundle: "3 errors from 2 projects", one line per error with type, place and count, no stack traces. The channel could not keep up, so the first messages were bundled.
2. **Read the fields.** Project name (first line), level (Critical, Error, Warning, Information), the exception and its message, the place (`Namespace.Class.Method`), the context line (environment, host, version, time in UTC), properties (`OrderId = 1042`), the count line and the stack.
3. **Judge the weight.** A count that climbs quickly means a loop or a hot path. A single occurrence after a deploy points at the new code (compare the version on the context line). An error seen in Staging only is a different error from the same one in Production: they are counted separately.
4. **Find the code.** Search the repository for the class and method on the place line and the top frames of the stack. Open only the files you need. Read the code around the failing line, the inputs named in the properties, and what changed recently if version control is available.
5. **Explain** what happened, the most likely cause and how sure you are. Name the file and line.
6. **Propose a fix** and wait for the user's approval before changing code, unless they already asked you to fix it.

## Notes

- Never ask the user for the project's API key; nothing in this task needs it.
- A message that came from a script has no stack: the text is the whole message. The `category` is under the text (for example the job name) and the host is on the context line.
- Numbers, ids, URLs and quoted text are ignored when messages are compared, so "Order 48213 failed" and "Order 48214 failed" are one error with a count.
- When an error stops and a repeat window passes without it, it is closed. If it comes back, it arrives as a new first message, so a regression after a deploy is not buried in an old thread.
- If the alert is noisy or poorly worded, see `write-good-alerts`. If a message did not arrive at all, see `troubleshoot`.
