# What a BugsRadar message contains

Source: https://bugsradar.com/docs/messages/

The channel behavior below describes the channels available today (Telegram, Discord and Pushover); the current list is at https://bugsradar.com/channels/.

One rule: a full message while the channel keeps up, a bundle when it cannot. While an error repeats, its message keeps the count. A message about one error starts with the project name and, under it, the level: Critical, Error, Warning or Information. Telegram and Discord get the same content; Pushover, a short push notification, comes without the stack trace.

## The first occurrence

A new error arrives at once: the project, the level, the exception and its message, where it happened, the environment, host, version and time, the event's properties and the stack trace.

```
Shop API
Error
System.TimeoutException: The operation has timed out.
Shop.Payments.PaymentService.ChargeAsync
Production · web-1 · v1.4.2 · 2026-09-22 12:00:05 UTC
OrderId = 1042
at Shop.Payments.PaymentService.ChargeAsync(Order order)
at Shop.Orders.OrderService.CreateOrder(Int32 orderId)
```

## Repeats

While the error keeps happening, its repeats are counted. In Telegram and Discord the count is written into the message of the first occurrence, without a new notification: the message is updated 10 minutes after it was sent, then after 30 minutes, then after an hour, then every 6 hours. When a window passes without repeats, the error is closed, and its next occurrence arrives as a new first message.

```
×158 total since 2026-09-22 12:00:05 UTC · last 2026-09-22 12:09:58 UTC
```

Pushover cannot edit a message, so there repeats arrive as summaries at the same moments (`×157 more since the last message · 158 total since ...`). The same happens in Telegram and Discord when Live count is turned off for a channel: each summary is then a new message with its own notification.

## Bundles

When a channel cannot keep up (errors waiting in its queue, the service asked for a pause, or a project sent many messages within the hour), the waiting errors are bundled into one message: a line per error with its type, place and count, grouped by project, without stack traces.

```
3 errors from 2 projects
Shop API
- TimeoutException · PaymentService.ChargeAsync · ×120
- SqlException · OrderRepository.Save · ×4
Admin panel
- NullReferenceException · ReportsController.Export · ×1
```

## Messages from scripts

A message sent to the notify endpoint is handled like an error from an application: the first one arrives at once, repeats are counted. The text is the whole message; `category` appears under it (for example the job name), `host` and `environment` on the context line. A channel shows a script message on one line: line breaks become spaces and the first 300 characters are shown.

## Which channel gets what

Without rules, every channel of a project gets every error of that project. With the Pro plan a rule is a filter on one channel of one project: minimum level, modules (the module, the logger name or a part of it), environments. An error that fits no channel goes nowhere. The same error in two environments is counted separately.
