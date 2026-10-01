---
name: environments
description: Set the environment name for BugsRadar messages and decide how production, staging and development should reach the chat. Use when the user asks how to tell staging from production in alerts, how to keep staging or development errors from waking anyone, or whether to use one BugsRadar project or several.
---

# Production, staging, dev

The same code runs in production, on staging and on laptops. Errors from all of them should not land in the same place with the same weight, and a staging error should never read like a production one at 2 a.m.

## Set the environment where the app starts

| Platform | Where |
|---|---|
| .NET | `configuration.Environment = builder.Environment.EnvironmentName;` in `AddBugsRadar` (or `environment` on the NLog target and log4net appender) |
| Node.js | `environment: process.env.NODE_ENV` in the client options |
| Python | `bugsradar.init(api_key=..., environment="Production")` |
| Scripts | `?environment=Production` in the query string of the notify request |

From then on every message carries it on the context line: `Production · web-1 · v1.4.2`.

## What it does

- **Counted apart.** The same error in two environments is counted separately, so Staging and Production never share a message. A bug reproduced on staging does not inflate the production count.
- **Filters.** On the Pro plan a channel can take only the errors that fit its filter (minimum level, modules, environments). An error with no environment fits no filter that names environments, so set the environment everywhere if filters are used.

## Options to offer

1. **One chat (Free).** Everything arrives in one chat and the environment is the second line of every message. Often enough for a solo developer or a small team.
2. **Several chats (Pro).** Staging gets a channel that nobody has notifications on; production gets the team's group and a Pushover for Critical; a channel without a filter catches everything else. An error that fits no channel goes nowhere, and the project's page warns when every channel has a filter.
3. **Development machines.** Errors from laptops are rarely worth a message. The simplest way to keep them out is not to register BugsRadar in the development configuration at all.

## One project or one per environment

One project per application with the environments inside it keeps the number of projects small and the rules simple. A project per environment also works and needs no rules, at the cost of two projects per application. Free allows 2 projects; Pro has unlimited projects and channels.

## Keys

Each project has its own key. Never read, ask for or print a key; the user puts the right key in each environment's configuration or secrets. Never register BugsRadar in a browser, mobile or desktop app that is shipped to others.

Read more: https://bugsradar.com/docs/rules/
