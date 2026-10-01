---
name: setup
description: Add BugsRadar error alerts to the user's project. Use when the user asks to connect BugsRadar, send errors or exceptions to a chat, add error alerts or notifications to an app, or wants to set up BugsRadar for .NET, Node.js or Python. Always ask first whether the user wants BugsRadar in this project; never read or ask for the project's API key.
---

# Connect BugsRadar to a project

BugsRadar sends the errors of an application to the user's chat. The first occurrence of an error arrives at once, with where it happened and the stack trace; repeats are counted in that message. Packages exist for .NET (NuGet), Node.js (npm) and Python (PyPI). Any other language or script sends one HTTP request (see the `any-language` and `script-alerts` skills).

## Rules that always apply

- Ask first: "Do you want BugsRadar error alerts in this project?" If the answer is no, stop.
- Never read, ask for, print or copy the project's API key. The user puts the key in place themselves. Do not open `.env` files, user secrets, secret stores or the secret parts of configuration files. If you must read a configuration file to learn its structure, do not repeat any value from it.
- BugsRadar belongs only in code that runs on servers: back ends, workers, scheduled jobs, CI pipelines, scripts. If the project is a browser app (React, Angular, Vue), a mobile app or a desktop app that is shipped to other people (WPF, WinForms, MAUI, Blazor WebAssembly, Electron, PyInstaller builds), stop and explain that a key inside such an app can be taken out by anyone. Offer to report from the project's own back end instead.
- Read the key from configuration or an environment variable. Never write it into code or into files that get committed.
- Do not replace the logging the project already has. The package adds a provider, a sink, a target or a transport next to it.

## Steps

1. **Ask** whether BugsRadar should be added. Then ask whether the user already has a BugsRadar project with a channel. If not, send them to https://bugsradar.com/docs/ (Quick start): they sign in at https://app.bugsradar.com/, create a project, add a channel and copy the key. You cannot do this step, and the key should not be pasted into the chat.
2. **Find the platform and the logger** from the project's files (`*.csproj`, `package.json`, `requirements.txt` or `pyproject.toml`). Note which logger it uses: ILogger, Serilog, NLog, log4net; winston, pino, Express; logging, loguru, Django, Flask, FastAPI.
3. **Read the page** for the platform: `references/dotnet.md`, `references/nodejs.md` or `references/python.md`.
4. **Install** the package and connect it to the existing logging and to uncaught errors. Keep the change small: usually one or two lines where the application starts.
5. **Read the key** from the place the project already uses for secrets or configuration. The site's guides use the environment variable `BUGSRADAR_KEY`; use that name unless the project has its own convention.
6. **Tell the user where to put the key**, in the right place for this project (user secrets, environment variable, hosting settings, CI secrets), and link https://bugsradar.com/docs/api-key/. Wait until they say it is set.
7. **Send a test error.** Run a one-off check in the project's own environment, where the process reads the key itself, so the value never passes through you. Examples are in the reference pages. The error arrives in the user's channel within seconds; ask the user to confirm it did.
8. **Summarize** what you changed: files, package versions and the line that reads the key.

If the test error does not arrive, use the `troubleshoot` skill.

## Where to go next

- Messages in the chat: `read-alert`.
- Staging versus production: `environments`.
- Failed jobs and workers: `background-jobs` and `script-alerts`.
- Deploy messages: `deploy-messages`.
