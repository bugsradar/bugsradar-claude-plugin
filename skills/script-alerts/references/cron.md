# Cron

Source: https://bugsradar.com/guides/cron/

Cron runs jobs quietly. Add one call after the command; `||` runs curl only when the command exits with a code other than 0. The key goes into a variable at the top of the crontab (the user puts the value there).

```
BUGSRADAR_KEY=<set by the user>
# Every night at 03:00
0 3 * * * /opt/scripts/import.sh || curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY" --data-binary "Nightly import failed on $(hostname) with exit code $?" "https://api.bugsradar.com/api/v3/notify?category=cron" > /dev/null
```

Change the text so the jobs can be told apart.

## The % sign

Cron treats `%` in a command as a line break. Write `\%` if the command has one, for example `date +\%F`.

## A wrapper for all jobs

With many jobs, a wrapper keeps the crontab readable. It runs the command, and when the command fails it sends the job's name, the exit code and the last line of the output. The output still goes to cron's mail or log.

`/usr/local/bin/cron-alert`:

```bash
#!/usr/bin/env bash
# Usage: cron-alert NAME COMMAND [ARGS...]
name="$1"; shift
output=$("$@" 2>&1)
code=$?
if [ "$code" -ne 0 ]; then
  last=$(printf '%s\n' "$output" | tail -n 1)
  curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY" \
    --data-binary "$name failed on $(hostname) with exit code $code: $last" \
    "https://api.bugsradar.com/api/v3/notify?category=cron" > /dev/null
fi
printf '%s\n' "$output"
exit "$code"
```

Make it executable (`chmod +x /usr/local/bin/cron-alert`) and put it in front of each job:

```
BUGSRADAR_KEY=<set by the user>
0 3 * * *    /usr/local/bin/cron-alert nightly-import /opt/scripts/import.sh
*/15 * * * * /usr/local/bin/cron-alert sync-prices /opt/scripts/sync-prices.sh --all
```

A job that runs every 15 minutes and keeps failing does not flood the chat: the first failure arrives in full, the next ones are counted in that message.

## Test

`false` always fails, so this sends a test alert (the user runs it with their own key set):

```bash
/usr/local/bin/cron-alert test false
```
