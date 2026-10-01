# Windows Task Scheduler

Source: https://bugsradar.com/guides/task-scheduler/

Task Scheduler records a failed run in the task's history but tells no one, and there is no action for a failed task. The script reports the failure itself.

## A function for all scripts

`C:\Scripts\Send-BugsRadar.ps1`:

```powershell
function Send-BugsRadar {
    param(
        [Parameter(Mandatory)] [string] $Message,
        [string] $Level = 'error',
        [string] $Category = 'Task Scheduler'
    )
    # Windows PowerShell 5.1 does not always use TLS 1.2 by itself
    [Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
    $uri = 'https://api.bugsradar.com/api/v3/notify?level={0}&category={1}&host={2}' -f `
        $Level, [Uri]::EscapeDataString($Category), [Uri]::EscapeDataString($env:COMPUTERNAME)
    Invoke-RestMethod -Method Post -Uri $uri -Headers @{ 'X-Api-Key' = $env:BUGSRADAR_KEY } `
        -ContentType 'text/plain; charset=utf-8' -Body $Message | Out-Null
}
```

## Report a failure

`$ErrorActionPreference = 'Stop'` turns every PowerShell error into one that `catch` sees. A program that exits with an error code does not throw, so `$LASTEXITCODE` is checked by hand:

```powershell
. C:\Scripts\Send-BugsRadar.ps1
$ErrorActionPreference = 'Stop'
try {
    & C:\Tools\report.exe --daily
    if ($LASTEXITCODE -ne 0) { throw "report.exe exited with code $LASTEXITCODE" }
}
catch {
    Send-BugsRadar -Message ('Nightly report failed: ' + $_.Exception.Message)
    exit 1
}
```

`exit 1` keeps the failure in the task's Last Run Result.

## Set up the task (the user does this)

1. In Task Scheduler create a task and choose "Run whether user is logged on or not".
2. On the Actions tab add "Start a program": program `powershell.exe`, arguments `-NoProfile -ExecutionPolicy Bypass -File C:\Scripts\nightly-report.ps1`.
3. Give the account the task runs as an environment variable `BUGSRADAR_KEY` with the project's key, or let only that account and administrators read a file that holds it.

## Programs and batch files

`||` runs curl only if the program exits with an error. `curl.exe` comes with Windows 10 version 1803, Windows Server 2019 and later:

```bat
@echo off
C:\Tools\export.exe --all || curl.exe -fsS --max-time 15 -H "X-Api-Key: %BUGSRADAR_KEY%" --data-binary "Export failed on %COMPUTERNAME%" "https://api.bugsradar.com/api/v3/notify?category=Task+Scheduler"
```

## Test

```powershell
. C:\Scripts\Send-BugsRadar.ps1
Send-BugsRadar -Message 'Test alert from Task Scheduler' -Level information
```

Then run the task itself with Run in Task Scheduler and check Last Run Result.
