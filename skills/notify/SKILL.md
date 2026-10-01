---
name: notify
description: Send a short notification from Claude to the user's BugsRadar chat (the channels the user connected to BugsRadar). Use only when the user asks for it, for example "when you finish, send me a BugsRadar message", "notify me that the build is done", or the /bugsradar:notify command. Needs Claude Code with the BUGSRADAR_KEY_FOR_CLAUDE environment variable.
---

# Send a notification as Claude

This skill sends one short text message to the chat that the user connected to BugsRadar. It uses Claude's own key, `BUGSRADAR_KEY_FOR_CLAUDE`, which is not the key of any project the user is working on.

## When to send

Send only when the user asked for it. That means a direct request, the `/bugsradar:notify` command, or a standing instruction in the prompt such as "do X and when you are done, send a BugsRadar message saying Y". Never send on your own initiative, and never send a second message the user did not ask for.

## What may go into the message

- Short plain text, one line. The channel shows the first 300 characters on one line; line breaks become spaces.
- Only what the user asked to report: what finished, how it ended, a count or a version.
- Never code, file contents, command output, personal data, keys, tokens or passwords.
- If the user gave the exact text, send exactly that text.
- If the user asked for a summary, write a one-line summary yourself and tell the user afterwards what you sent.

## Before the first send

1. Check that the variable exists without printing it:
   - bash: `[ -n "$BUGSRADAR_KEY_FOR_CLAUDE" ] && echo set || echo missing`
   - PowerShell: `if ($env:BUGSRADAR_KEY_FOR_CLAUDE) { 'set' } else { 'missing' }`
2. If it is missing, or you cannot run commands on the user's computer, do not send. Follow the `claude-key-setup` skill: explain what is needed, where to get the key and how to set it. Never ask the user to paste the key into the chat, and never read, print or log it.

## Send

Parameters (all in the query string, spaces written as `+`):

| Parameter | Default for this skill | Meaning |
|---|---|---|
| `level` | `information` | `information`, `warning`, `error` or `critical` |
| `category` | `Claude` | A label under the message, for example the task name |

The text goes in the request body. Pass it through standard input so that quotes and `$` in the text cannot break the command.

bash:

```bash
curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY_FOR_CLAUDE" \
  --data-binary @- \
  "https://api.bugsradar.com/api/v3/notify?level=information&category=Claude" <<'BUGSRADAR_TEXT'
Build finished: 120 tests passed
BUGSRADAR_TEXT
```

PowerShell:

```powershell
$text = @'
Build finished: 120 tests passed
'@
Invoke-RestMethod -Method Post -Uri 'https://api.bugsradar.com/api/v3/notify?level=information&category=Claude' `
  -Headers @{ 'X-Api-Key' = $env:BUGSRADAR_KEY_FOR_CLAUDE } `
  -ContentType 'text/plain; charset=utf-8' -Body $text | Out-Null
```

Windows PowerShell 5.1 may need TLS 1.2: add `[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12` before the request.

## Read the answer

- `202`: accepted. Tell the user the message was sent and quote the text you sent.
- `401`: the key is missing or wrong. Ask the user to check the value themselves; do not print it.
- `400`: the text was empty.
- `429`: wait for the seconds in `Retry-After`, then retry once.
- Network error: say that api.bugsradar.com could not be reached.

Details of the endpoint are in `references/notify-api.md`.

## Repeats

BugsRadar counts identical messages instead of sending them again: numbers, ids, URLs and quoted text are ignored when messages are compared. Two "Build finished: 120 tests passed" messages become one message with a count. If the user wants each one to arrive separately, put something that identifies it in the text, such as the task name or the commit hash.

## Where this works

It needs a shell on the user's computer with access to the internet and the `BUGSRADAR_KEY_FOR_CLAUDE` variable, so it is meant for Claude Code. In a chat that cannot run commands on the user's computer, say so and do not pretend to send.
