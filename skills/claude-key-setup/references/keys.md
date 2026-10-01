# Keys in BugsRadar

Source: https://bugsradar.com/docs/api-key/

A BugsRadar project has two API keys, primary and secondary, and either one sends messages to the project's channels. A key identifies the project, so the key decides which channels receive a message.

## Where a key may go

- Back ends, APIs, worker services, scheduled jobs, CI/CD pipelines and scripts on your own servers.
- In configuration, an environment variable or the secrets of a CI system, not in the source.
- Never in a browser app (React, Angular, Vue or any JavaScript that reaches the browser), in a mobile or desktop app that is shipped to others, or in a public repository. Anyone can take the key out of such code.

## What a key can do

Whoever has the key can fill the channels with messages, but cannot read anything or change settings.

## If a key leaks

Move your applications and scripts to the project's other key, then press Regenerate next to the leaked key on the project's page in the web app. The leaked key stops working at once, and the apps keep reporting through the other one.

## Which project does a key belong to?

The Check API Key page of the web app shows which of your projects a key belongs to.

## The two keys in this plugin

| Key | Variable | Used for | Who sets it |
|---|---|---|---|
| Key for Claude | `BUGSRADAR_KEY_FOR_CLAUDE` | Notifications that Claude sends when asked | The user, once per project, session or machine (see the `claude-key-setup` skill) |
| Key of the user's project | chosen by that project; the site's guides use `BUGSRADAR_KEY` | The project's own errors and alerts, sent by its code or scripts | The user, in the project's configuration or secrets; the plugin never reads or asks for it |

## Claude Code settings

Claude Code applies an `env` block from `.claude/settings.local.json` (personal, for one project) or from `~/.claude/settings.json` (all projects) to its sessions. `.claude/settings.local.json` is the file for personal values: Claude Code adds it to the global git excludes when it creates the file, and a file created by hand needs a `.gitignore` entry. Docs: https://code.claude.com/docs/en/settings
