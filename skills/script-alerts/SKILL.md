---
name: script-alerts
description: Make a script, scheduled job or pipeline report its own failure to the user's BugsRadar chat with one HTTP request. Use for cron jobs, systemd services and timers, Windows Task Scheduler tasks, SQL Server Agent jobs, backup scripts (pg_dump, mysqldump, rsync, robocopy), and CI/CD pipelines (GitHub Actions, GitLab CI, Azure Pipelines, Jenkins), or when the user says a job fails silently or asks for an alert when something fails.
---

# Alerts from scripts and jobs

A script that fails is reported with one request: `POST https://api.bugsradar.com/api/v3/notify`, the project's key in the `X-Api-Key` header, the text of the message in the body, and optional `level`, `category`, `environment` and `host` in the query string. See `references/notify-api.md`.

## Rules that always apply

- The script reads the project's key from an environment variable named `BUGSRADAR_KEY` (the name the site's guides use) or from the secrets of the CI system. Use the project's own name if it already has a convention.
- Never read, ask for, print or copy the key. Do not open `.env` files or secret stores. Tell the user where to put the key and let them do it. In a SQL Server Agent step the key has to be written into the step; say that everyone who can open the job sees it, and how to rotate it (https://bugsradar.com/docs/api-key/).
- Never put the key into a file that is committed. In CI use the secret store.
- Use `curl -fsS --max-time 15` so the alert never hangs the script and a failed alert is visible. In a backup or cron script add `|| true` after the alert call when the alert itself must not change the script's exit status.
- Do not change the script's behavior beyond adding the alert: keep its exit code, its output and its error handling.

## Steps

1. Ask what should be reported and where the script runs, unless it is obvious from the file.
2. Pick the reference for the case: `references/cron.md`, `references/systemd.md`, `references/task-scheduler.md`, `references/sql-server-agent.md`, `references/backups.md` or `references/ci-cd.md`.
3. Add the alert in the smallest way that fits: a `||` after the command, a trap, an `OnFailure=` drop-in, a final failure step. Put the job's name and the host in the text.
4. Write the text so repeats group correctly: see the `write-good-alerts` skill. A job that fails every minute sends one message and then counts the repeats in it.
5. Tell the user where to set the key and how to test the alert. The guides test with a command that always fails (`false`) or by starting the alert unit or step by hand.
6. Mention what BugsRadar does not do: it reports a failure the script reports, and it cannot tell that a script never ran at all (no heartbeat). To know a nightly job ran, the job can send an information message at the end of a successful run; keep those in a channel of their own.

## Messages for success

A success message uses `level=information`, for example at the end of a backup or after a deploy. It arrives like an error but with the level Information. A morning without it means the job did not run.
