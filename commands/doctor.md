---
description: Check the key for Claude's notifications and send a test message to your BugsRadar chat
---

Check that BugsRadar notifications from Claude work, as I asked with this command.

1. Check that the `BUGSRADAR_KEY_FOR_CLAUDE` environment variable exists without printing it (bash: `[ -n "$BUGSRADAR_KEY_FOR_CLAUDE" ] && echo set || echo missing`; PowerShell: `if ($env:BUGSRADAR_KEY_FOR_CLAUDE) { 'set' } else { 'missing' }`). If it is missing, or you cannot run commands on my computer, use the `claude-key-setup` skill and stop.
2. Use the `notify` skill to send the test message "BugsRadar test from Claude Code" with `level=information` and `category=Claude`.
3. Read the answer: 202 means accepted; for 401, 400, 413, 429 or a network error use the `troubleshoot` skill. Never print, log or ask for the key.
4. Tell me the result, and ask me to confirm that the message arrived in my chat.
