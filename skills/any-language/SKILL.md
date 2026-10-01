---
name: any-language
description: Send BugsRadar alerts from a language that has no BugsRadar package (Go, PHP, Ruby, Java, Rust, Kotlin, shell and others) with one HTTP request. Use when the project is not .NET, Node.js or Python and the user wants error alerts or notifications in their chat, or asks how to call the BugsRadar API from code.
---

# Alerts without a package

There is no BugsRadar package for Go, PHP, Ruby or Java, and there does not need to be one. Everything that can send an HTTP request can send an alert: the project's key in a header, the text of the message in the body, and optional query parameters for the level, a category, the environment and the host.

## Rules

- `POST https://api.bugsradar.com/api/v3/notify` with `X-Api-Key` and a plain-text body. The server answers 202 as soon as it has the message; delivery goes on in the background. Parameters, answers and limits: `references/notify-api.md`.
- Never read, ask for, print or copy the project's key. The code reads it from an environment variable or configuration (the site's guides use `BUGSRADAR_KEY`); the user puts the value there.
- Only in code that runs on servers. Never in browser, mobile or desktop apps that are shipped to others.
- Give the request a timeout of a few seconds and call it from the catch that matters. A failed alert must never break the code that sends it: catch everything around the call.
- Over HTTP you send what you put in the text; there is no automatic stack trace, no uncaught-error capture and no folding of repeats. Put the identifying words in the text and what varies in a number or in quotes (see `write-good-alerts`), and include the error message.
- Encode query parameter values (spaces as `+` or `%20`).

## Steps

1. Identify the language and how the project logs and handles errors.
2. Add one small function that sends the alert, in the project's style, using the matching reference: `references/go.md`, `references/php.md`, `references/ruby.md` or `references/java.md`. For other languages follow the same shape: one POST, one header, one string.
3. Call it from the places that matter: the top-level error handler, job runners, the catch of a payment or webhook handler. Do not scatter calls.
4. Tell the user where to set the key, then send a test message (the user runs it with their own key set) and ask them to confirm it arrived.

If the user would like a package for their platform, they can write to support@bistriy.com.
