# BugsRadar for Node.js

Source: https://bugsradar.com/docs/nodejs/ (updated 1 October 2026)

One npm package for servers, workers and command-line tools on Node.js 18 and later: uncaught errors, direct calls, Express, winston and pino. ESM and CommonJS, with TypeScript types. It is for Node.js on servers only: never in browser code (React, Angular, Vue, Next.js client components) or in an Electron app that is shipped to others.

## Install

```bash
npm install bugsradar
```

## Create the client (one per process)

```js
import { BugsRadar } from 'bugsradar';

export const bugsRadar = new BugsRadar({
  apiKey: process.env.BUGSRADAR_KEY,
  environment: process.env.NODE_ENV, // optional
});
```

CommonJS: `const { BugsRadar } = require('bugsradar');`. The key is required: an empty one throws when the client is created. The user sets the environment variable; do not write the key into the source.

## Uncaught errors

The client reports uncaught exceptions and unhandled promise rejections by itself, waits up to `shutdownTimeout` for the report to leave, and the process exits with code 1 as it would without BugsRadar. If the application has its own `uncaughtException` handler, the exit is left to it. Turn this off with `captureUncaught: false`.

## Direct calls

```js
try {
  await createOrder(orderId);
} catch (error) {
  bugsRadar.sendException(error, { module: 'Orders' });
}
```

`send` and `sendException` queue the event and return at once. Nothing throws; delivery problems are written to the console as warnings.

## Express

```js
import { expressErrorHandler } from 'bugsradar/express';
// after your routes:
app.use(expressErrorHandler(bugsRadar));
```

It reports every unhandled error of a request with its method and path, then passes it on with `next(error)`. Errors with a status below 500 are passed on without a report.

## winston

```js
import { BugsRadarTransport } from 'bugsradar/winston';

export const logger = winston.createLogger({
  transports: [
    new winston.transports.Console(),
    new BugsRadarTransport({ client: bugsRadar, level: 'error' }),
  ],
});
```

An `Error` in the `error` or `err` field travels as the exception, with its stack; the other fields become properties.

## pino

pino runs transports in a worker thread, so this one gets the key in its options:

```js
export const logger = pino({
  transport: {
    targets: [
      { target: 'pino/file', options: { destination: 1 } },
      { target: 'bugsradar/pino', level: 'error', options: { apiKey: process.env.BUGSRADAR_KEY } },
    ],
  },
});
```

It sends `error` and `fatal`; add `level: 'warn'` to its options to send warnings too.

## Before the process exits

Command-line tools, scheduled jobs and serverless functions often exit right after their work:

```js
await bugsRadar.flush(); // waits up to shutdownTimeout
```

## Configuration

| Option | Default | Meaning |
|---|---|---|
| `apiKey` | none | Project API key. Required. |
| `environment` | none | Environment name (Production, Staging). |
| `host` | `os.hostname()` | Host for events. |
| `appVersion` | none | Application version, shown in messages. Not used for grouping. |
| `captureUncaught` | `true` | Report uncaught exceptions and unhandled rejections. |
| `repeatInterval` | 5000 ms | Repeats within this interval travel as one request. |
| `queueCapacity` | 1000 | Events beyond this are dropped with a warning. |
| `shutdownTimeout` | 5000 ms | How long `flush()` and an uncaught exception wait for the queue. |
| `requestTimeout` | 15000 ms | One request to the server. |
| `apiUrl` | api.bugsradar.com | Change only for a self-hosted BugsRadar. |

## One-off test error

The key is read from the environment, so the value never passes through the assistant:

```bash
node --input-type=module -e "import { BugsRadar } from 'bugsradar'; const c = new BugsRadar({ apiKey: process.env.BUGSRADAR_KEY }); c.sendException(new Error('BugsRadar test error')); await c.flush();"
```

## Grouping

Repeats are grouped by the exception type and the top frames of the stack, or by the message template. To group by your own key, set `fingerprint` on the event. The same `Error` object seen twice, for example by Express and by winston, is sent once.
