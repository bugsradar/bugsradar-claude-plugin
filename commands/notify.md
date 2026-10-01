---
description: Send a short notification from Claude to your BugsRadar chat
argument-hint: "[level=information|warning|error|critical] [category=name] text"
---

Use the `notify` skill to send one short notification as Claude, because I asked for it with this command.

The arguments are: $ARGUMENTS

Optional leading `level=` and `category=` words set the level (default `information`) and the category (default `Claude`); everything after them is the text. Send only that text, as one short plain line. Never include code, file contents, command output or secrets. If `BUGSRADAR_KEY_FOR_CLAUDE` is not set or you cannot run commands on my computer, use the `claude-key-setup` skill instead of sending. Afterwards tell me what you sent and the result.
