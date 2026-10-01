# Status codes

Sources: https://bugsradar.com/docs/http/ and https://bugsradar.com/docs/clef/

## Notify endpoint: `POST https://api.bugsradar.com/api/v3/notify`

Key in the `X-Api-Key` header.

| Code | Meaning |
|---|---|
| 202 | Accepted. Delivery to the channels goes on in the background. |
| 400 | The body is empty. |
| 401 | The key is missing or wrong. |
| 413 | The body is larger than 64 KB. |
| 429 | The server asks to slow down: wait for the number of seconds in `Retry-After`. |

Text beyond 8,000 characters is cut. A channel shows the message on one line, the first 300 characters.

## CLEF endpoint: `https://api.bugsradar.com/api/v3/clef` (Seq clients) and `.../clef/ingest/clef`

Key in the `X-Seq-ApiKey` header (a Seq client sends its API key there).

| Code | Meaning |
|---|---|
| 201 | The batch is accepted, even if none of its events was an error. |
| 400 | Not one line of the body is a CLEF event. |
| 401 | The API key is wrong. |
| 413 | The body is over 1 MB. |
| 429 | Over 50 requests in a minute on the Free or Pro plan, or too many requests at once. `Retry-After` says how long; Seq clients wait and send the batch again. |

## Packages

The packages queue events and send them in the background. On 429 the client waits as long as the server asks; on 5xx and network failures it retries three times. Events beyond the queue capacity (1000 by default) are dropped with a warning. API v2 is shut down: .NET package versions below 3.0.0.1 no longer deliver errors.

## Where problems are logged

| Platform | Where |
|---|---|
| .NET | ILogger category `BugsRadar.Client`; Serilog sink without DI: Serilog's SelfLog; NLog: NLog's internal log; log4net: its internal log |
| Node.js | Warnings on the console |
| Python | Warnings of the `bugsradar` logger (never sent anywhere by BugsRadar itself) |

Support: support@bistriy.com (include these lines, never the key).
