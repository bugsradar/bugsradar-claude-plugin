# The notify endpoint

Source: https://bugsradar.com/docs/http/

```
POST https://api.bugsradar.com/api/v3/notify?level=warning&category=Nightly+backup
X-Api-Key: <project API key>
Content-Type: text/plain; charset=utf-8

Backup of shop-db failed: disk is full
```

- The key goes in the `X-Api-Key` header. It is the API key of one BugsRadar project; the project's channels receive the message.
- The body is the text of the message in UTF-8. Any `Content-Type` is accepted, so `curl -d` and `--data-binary` work as they are. The body is plain text; JSON is not parsed and arrives as text.
- Query parameters, all optional, spaces written as `+`:

| Parameter | Default | Meaning |
|---|---|---|
| `level` | `error` | `critical`, `error`, `warning` or `information`; `fatal`, `warn` and `info` work too. Shown under the project name. |
| `category` | none | A label under the message, such as the name of a job or a pipeline. |
| `environment` | none | `Production`, `Staging` and so on. |
| `host` | none | The machine the message comes from. |

## Answers and limits

| Code | Meaning |
|---|---|
| 202 | Accepted. Delivery to the channels goes on in the background. |
| 400 | The body is empty. |
| 401 | The key is missing or wrong. |
| 413 | The body is larger than 64 KB. |
| 429 | Slow down: wait for the number of seconds in `Retry-After`. |

Text beyond 8,000 characters is cut. A channel shows a message on one line: line breaks become spaces, and the first 300 characters are shown. With `curl -f`, curl exits with an error on any code except 202.

## Repeats and grouping

The first message arrives at once. The same message again is counted in the first one (in Pushover the repeats arrive as summaries): after 10 minutes, then 30 minutes, an hour, and every 6 hours while it continues. Numbers, GUIDs, URLs, email addresses and text in quotes do not count when messages are compared, so "Backup failed at 03:00" and "Backup failed at 04:00" are the same message.

Nothing is stored: a message stays in memory only until it is delivered.

## The key

The key is secret. It belongs in an environment variable or in the secrets of a CI system, never in code that is committed and never in browser, mobile or desktop apps. Whoever has the key can fill the channels with messages but cannot read anything or change settings. If a key leaks, switch to the project's other key and press Regenerate next to the leaked one in the web app: https://bugsradar.com/docs/api-key/
