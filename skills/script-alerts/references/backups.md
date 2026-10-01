# Backups

Source: https://bugsradar.com/guides/backups/

A backup that stops working unnoticed is found out on the day it is needed. The backup script reports its own failure with one request. The scripts read the key from `BUGSRADAR_KEY`.

## PostgreSQL: pg_dump

`set -e` stops the script at the first failed command, `pipefail` catches a failure inside a pipe, and the trap sends the alert with the line and the exit code:

```bash
#!/usr/bin/env bash
set -euo pipefail
notify() {
  curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY" --data-binary "$1" \
    "https://api.bugsradar.com/api/v3/notify?category=Backup&host=$(hostname)" > /dev/null || true
}
trap 'notify "Backup of shop-db failed at line $LINENO (exit code $?)"' ERR

file="/backups/shop-$(date +%F).dump"
pg_dump --format=custom --file="$file" shop
rsync -a /backups/ backup@nas:/volume1/backups/
```

The copy to another machine is part of the same script, so a broken `rsync` is reported too. Run the script from cron or a systemd timer.

## MySQL and MariaDB: mysqldump

Without `pipefail` a failed `mysqldump` would go unnoticed, because `gzip` at the end of the pipe succeeds:

```bash
#!/usr/bin/env bash
set -euo pipefail
notify() {
  curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY" --data-binary "$1" \
    "https://api.bugsradar.com/api/v3/notify?category=Backup&host=$(hostname)" > /dev/null || true
}
trap 'notify "MySQL backup of shop failed at line $LINENO (exit code $?)"' ERR
mysqldump --single-transaction --routines shop | gzip > "/backups/shop-$(date +%F).sql.gz"
```

## Windows: robocopy

Its exit codes from 8 up mean that files were not copied:

```powershell
robocopy D:\Data \\nas\backup\data /MIR /R:2 /W:5 /NP /LOG:C:\Logs\backup.log
if ($LASTEXITCODE -ge 8) {
    Invoke-RestMethod -Method Post -Uri 'https://api.bugsradar.com/api/v3/notify?category=Backup' `
        -Headers @{ 'X-Api-Key' = $env:BUGSRADAR_KEY } -ContentType 'text/plain; charset=utf-8' `
        -Body "File backup to NAS failed on $env:COMPUTERNAME (robocopy exit code $LASTEXITCODE)"
}
```

Run it from Task Scheduler (see `task-scheduler.md`).

## SQL Server

SQL Server backups usually run as SQL Server Agent jobs, maintenance plans included. Add an alert step to the job (see `sql-server-agent.md`). The message then names the step that failed and the error of `BACKUP DATABASE`.

## Know that the backup ran

An alert comes when a backup fails, but not when it never started (the server was off, the job was disabled). To see every night that it ran, send an information message at the end of a successful run:

```bash
curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY" \
  --data-binary "Backup of shop-db finished: $(du -h "$file" | cut -f1)" \
  "https://api.bugsradar.com/api/v3/notify?category=Backup&level=information" > /dev/null || true
```

`|| true` keeps a network hiccup here from setting off the failure alert. A morning without the message means the backup did not run. Keep such messages in a channel of their own if they should not sit next to errors: one project per purpose, each with its own channel.
