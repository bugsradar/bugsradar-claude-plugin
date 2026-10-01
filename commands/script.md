---
description: Make a script or job report its own failure to BugsRadar
argument-hint: "<script file or job description>"
---

Use the `script-alerts` skill to make this script or job report its own failure to BugsRadar: $ARGUMENTS

Keep the script's behavior and exit code as they are and add only the alert. The script reads the project's key from the `BUGSRADAR_KEY` environment variable or the CI secret store; never read, ask for or print the key, and tell me where to put it.
