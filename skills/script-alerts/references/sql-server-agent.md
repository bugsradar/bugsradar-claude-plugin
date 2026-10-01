# SQL Server Agent

Source: https://bugsradar.com/guides/sql-server-agent/

SQL Server Agent notifies operators by email, which needs Database Mail. With BugsRadar a failed job arrives in the chat instead: which job, which step and the error. The steps work in SQL Server Management Studio on SQL Server for Windows.

## How it works

Add one last step to the job that sends the alert, and every other step jumps to it when it fails. When all steps succeed the job ends before it reaches the alert step.

1. On the Steps page add a last step named `Alert BugsRadar` of type PowerShell with a script below. The user puts the project's key in it.
2. On the Advanced page of every other step set "On failure action" to "Go to step: Alert BugsRadar". For the step right before the alert, also set "On success action" to "Quit the job reporting success".
3. For the alert step itself set "On success action" to "Quit the job reporting failure", so the job history keeps showing the failure.

The key is written in the job step, so everyone who can open the job sees it. If it leaks, put the project's other key in the step, then press Regenerate next to the leaked key in the web app.

## Simple version (PowerShell step)

```powershell
# Windows PowerShell 5.1 does not always use TLS 1.2 by itself
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
$server = '$(ESCAPE_SQUOTE(SRVR))'  # SQL Server Agent puts the server name here before the step runs
$uri = 'https://api.bugsradar.com/api/v3/notify?category=SQL+Server+Agent&host=' + [Uri]::EscapeDataString($server)
Invoke-RestMethod -Method Post -Uri $uri `
  -Headers @{ 'X-Api-Key' = '<project key, set by the user>' } `
  -ContentType 'text/plain; charset=utf-8' -Body "Job Nightly backup failed on $server"
```

SQL Server Agent has no token for the job name, so the name is written in the text.

## Version that reports the failed step and its error

Reads the failed step and its message from msdb, so one script fits every job:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12
$server = '$(ESCAPE_SQUOTE(SRVR))'
# The failed step of the current run: history rows after the previous run's outcome
$failed = Invoke-Sqlcmd -ServerInstance $server -Database msdb -Query @"
SELECT TOP (1) j.name AS job_name, h.step_name, h.message
FROM dbo.sysjobhistory AS h
JOIN dbo.sysjobs AS j ON j.job_id = h.job_id
WHERE h.job_id = $(ESCAPE_NONE(JOBID)) AND h.step_id > 0 AND h.run_status = 0
  AND h.instance_id > ISNULL((SELECT MAX(instance_id) FROM dbo.sysjobhistory
                             WHERE job_id = h.job_id AND step_id = 0), 0)
ORDER BY h.instance_id DESC;
"@
if ($failed) {
    $text = 'Job {0} failed on {1} at step {2}: {3}' -f $failed.job_name, $server, $failed.step_name, $failed.message
} else {
    $text = 'Test alert from SQL Server Agent on {0}' -f $server
}
$uri = 'https://api.bugsradar.com/api/v3/notify?category=SQL+Server+Agent&host=' + [Uri]::EscapeDataString($server)
Invoke-RestMethod -Method Post -Uri $uri `
  -Headers @{ 'X-Api-Key' = '<project key, set by the user>' } `
  -ContentType 'text/plain; charset=utf-8' -Body $text
```

SQL Server Agent treats every `$(...)` in a step as a token, which is why the text is built with `-f` rather than with PowerShell's `$(...)` inside strings. The step runs as the SQL Server Agent service account, which can read msdb by default.

## CmdExec with curl

On Windows Server 2019 and later the alert step can be of type Operating system (CmdExec):

```bat
curl.exe -fsS --max-time 15 -H "X-Api-Key: <project key, set by the user>" --data-binary "Job Nightly backup failed on $(ESCAPE_DQUOTE(SRVR))" "https://api.bugsradar.com/api/v3/notify?category=SQL+Server+Agent"
```

## Test

In SQL Server Management Studio right-click the job, choose "Start Job at Step..." and pick Alert BugsRadar. The history version sends a test alert when the run has no failed step.
