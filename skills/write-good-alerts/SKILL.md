---
name: write-good-alerts
description: Write the text, level, category, environment and host of a BugsRadar alert so it reads well in the chat and repeats are grouped correctly. Use when the user writes or reviews alert messages for scripts, direct calls or log statements, asks why messages are grouped or not grouped, or wants alerts to be clearer, shorter or less noisy.
---

# Write alerts that group and read well

## How BugsRadar compares messages

Messages are compared by a fingerprint. When the text is compared, these parts are replaced by placeholders and so do not count:

- numbers
- GUIDs
- URLs
- email addresses
- text in quotes

So "Backup failed at 03:00" and "Backup failed at 04:00" are the same message, and "Order 48213 failed" and "Order 48214 failed" are one error with a count. The first one arrives in full; repeats are counted in it (after 10 minutes, then 30 minutes, an hour, then every 6 hours while they continue).

A commit hash in the text makes every different hash a message of its own: the CI guide relies on this so that each deploy ("Deployed a1b2c3") arrives separately. Keep that in mind when an error message happens to contain a hash.

Errors in different environments are counted separately.

## Rules for the text

1. **Put what identifies the problem in the words, and what varies in a number or in quotes.** "Payment webhook failed for order 48213" and "Payment webhook failed for order 48214" are one error with a count, because the numbers are masked. "Payment webhook failed for order \"A-17\"" groups the same way, because quoted text is masked.
2. **One line, short.** A channel shows the message on one line, line breaks become spaces, and only the first 300 characters are visible (text beyond 8,000 characters is cut). Put the important part first.
3. **Say what failed and where.** The job or service name, the step, the cause if known. Add the host through `host`, not in the text, when it is a script.
4. **Never include secrets**, tokens, passwords, personal data or large dumps. Messages go to a third-party chat.
5. **Decide whether repeats should be one message or many.** Normally one message with a count is what you want: a job that fails every minute sends one message and counts the repeats. Add something that differs each time (such as a commit hash) only when each occurrence must arrive on its own.

## Level

| Level | Use for |
|---|---|
| `critical` | Wake someone up (Pushover sends it as a high-priority notification) |
| `error` | Something failed. The default for the notify endpoint. |
| `warning` | Something is wrong but not failing yet |
| `information` | A success or a status: a finished backup, a finished deploy |

`fatal`, `warn` and `info` also work as level names on the notify endpoint. A channel with a filter (Pro plan) takes errors from a minimum level up, so a level is also how a message is routed.

## Category, environment, host

- `category`: a short label under the message, such as the job or pipeline name (`Nightly backup`, `GitHub Actions`).
- `environment`: `Production`, `Staging`. It is part of grouping and of filters: an error with no environment fits no filter that names environments.
- `host`: the machine. On the notify endpoint it is optional; the packages fill it in with the machine name.

On the HTTP endpoint these go in the query string, with spaces written as `+`. See the `script-alerts` skill for the full request.

## Messages worth sending

- Failures with the cause: "Backup of shop-db failed: disk is full".
- Deploys: "Deployed a1b2c3 to production" at `level=information`.
- Success messages for jobs whose silence would mean "it did not run": "Nightly backup finished: 4.2 GB".

Keep success messages in a channel of their own if they should not sit next to errors: one project per purpose, each with its own channel.
