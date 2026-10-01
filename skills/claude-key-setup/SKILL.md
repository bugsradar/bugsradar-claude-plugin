---
name: claude-key-setup
description: Help the user create and set the BugsRadar key that Claude uses to send notifications (BUGSRADAR_KEY_FOR_CLAUDE), and explain that Claude Code can use a different key per project, per session or one for everything. Use when the key is missing, when the user asks how to set it up, how to get notifications from Claude, or when /bugsradar:notify or /bugsradar:doctor finds no key.
---

# Set up the key for Claude's notifications

Claude sends notifications with its own BugsRadar key, kept in the environment variable `BUGSRADAR_KEY_FOR_CLAUDE`. This skill explains the key and where to set it. It never asks for the value, never reads it and never prints it.

## Two different keys

Say this to the user in your own words, briefly:

- The **key for Claude** is the key of a BugsRadar project that you use for Claude's own messages, such as "the task is finished". It is the only key this plugin needs. The messages go to the channels of that project.
- The **key of a project** you are developing is a different key, used by that project's code to send its errors and alerts. This plugin does not read or ask for it. It is needed only if the user wants Claude to connect BugsRadar to that project (the `setup` skill), and then the user puts it in place themselves.

## Get the key (the user does this)

Claude cannot do these steps, because it has no account:

1. Sign in at https://app.bugsradar.com/ with an email address (the link in the email signs you in and creates the account the first time).
2. Press New project and name it for what it is for, for example "Claude". Copy the project's API key.
3. On the project's page press Add channel and choose where the messages go. What each channel needs: https://bugsradar.com/channels/
4. Press Send test on the channel to check that it reaches the chat.

Background: https://bugsradar.com/docs/ and https://bugsradar.com/docs/api-key/

## Set the variable (the user does this)

Explain the three ways, and recommend the first for Claude Code because developers often want a different chat for each piece of work:

1. **One key per project.** In the project's folder, in `.claude/settings.local.json`, add an `env` block. Different projects can have different keys, and so different chats.

   ```json
   {
     "env": {
       "BUGSRADAR_KEY_FOR_CLAUDE": "paste the key here"
     }
   }
   ```

   This file is personal. Claude Code keeps it out of git when it creates the file; if the user creates it by hand, they must add `.claude/settings.local.json` to `.gitignore` themselves. Do not create this file with the real value yourself.
2. **One key per session.** Set the variable in the terminal before starting Claude Code. It applies to that session only.
   - bash: `export BUGSRADAR_KEY_FOR_CLAUDE='paste the key here'`
   - PowerShell: `$env:BUGSRADAR_KEY_FOR_CLAUDE = 'paste the key here'`
   - cmd: `set BUGSRADAR_KEY_FOR_CLAUDE=paste the key here`
3. **One key for everything.** A permanent user variable. On Windows: System Properties, Environment Variables, or `setx BUGSRADAR_KEY_FOR_CLAUDE "paste the key here"` (it applies to terminals opened afterwards). On macOS and Linux: an `export` line in the shell profile. Restart Claude Code afterwards.

Use one way at a time, so it is clear which key is in effect. Which project a message lands in is decided only by the key, so a different key means a different project and a different chat.

## After it is set

1. Ask the user to restart Claude Code if they used the third way, or to start it from the terminal where they set the variable (second way).
2. Check without printing: bash `[ -n "$BUGSRADAR_KEY_FOR_CLAUDE" ] && echo set || echo missing`; PowerShell `if ($env:BUGSRADAR_KEY_FOR_CLAUDE) { 'set' } else { 'missing' }`.
3. Run `/bugsradar:doctor` to send a test message.

## Rules

- Never ask the user to paste the key into the chat. If they paste it anyway, do not repeat it, do not write it into any file, and suggest regenerating that key in the web app.
- Never print, log or echo the variable.
- Claude can send notifications only from Claude Code, where it can run commands on the user's computer and see the variable. In a chat that cannot do that, say so.

More detail, including what to do if a key leaks, is in `references/keys.md`.
