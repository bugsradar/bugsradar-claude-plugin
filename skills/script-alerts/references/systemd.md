# systemd

Source: https://bugsradar.com/guides/systemd/

`OnFailure=` in a service names the units to start when the service enters the failed state. One template unit serves every service: the name of the failed service comes in as the instance name.

`/etc/systemd/system/bugsradar-alert@.service`:

```ini
[Unit]
Description=BugsRadar alert for %i
[Service]
Type=oneshot
EnvironmentFile=/etc/bugsradar.env
ExecStart=/usr/bin/curl -fsS --max-time 15 -H "X-Api-Key: ${BUGSRADAR_KEY}" --data-binary "%i failed on %H" "https://api.bugsradar.com/api/v3/notify?category=systemd"
```

`%i` is the name of the failed service and `%H` the host name. The key lives in a file that only root can read. The user creates it with their own key (the value is not typed in the assistant's session):

```bash
sudoedit /etc/bugsradar.env      # one line: BUGSRADAR_KEY=<the project key>
sudo chmod 600 /etc/bugsradar.env
sudo systemctl daemon-reload
```

## Watch a service

Add `OnFailure=` with a drop-in, so the unit file stays as the package installed it:

```bash
sudo systemctl edit backup.service
```

```ini
[Unit]
OnFailure=bugsradar-alert@%n.service
```

`%n` is the full name of the service, such as `backup.service`. `systemctl edit` reloads systemd when you save.

## Timers and restarts

- A timer starts a service, and it is the service that fails. Put `OnFailure=` in the service, not in the `.timer` unit.
- A service with `Restart=` enters the failed state only when systemd gives up restarting it. Short crashes that a restart fixes send no alert.

## Test

```bash
sudo systemctl start bugsradar-alert@test.service
```

The message "test failed on your host" arrives within seconds. If it does not, `journalctl -u bugsradar-alert@test.service` shows what curl answered.
