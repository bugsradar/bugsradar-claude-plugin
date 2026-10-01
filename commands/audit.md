---
description: Audit the current project for swallowed errors that nobody hears about
argument-hint: "[folder or file]"
---

Use the `silent-failures` skill to audit this project for places where errors are swallowed or only logged, as I asked with this command. Limit the audit to this path if I gave one: $ARGUMENTS

This is read-only: report the findings, ranked, with file and line and the smallest fix for each, and ask which ones I want changed. Do not open `.env` files or secret stores and never ask for any key.
