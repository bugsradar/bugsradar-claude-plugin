---
name: deploy-messages
description: Add BugsRadar messages to a deploy pipeline, so a failed build or deploy and a finished deploy arrive in the user's chat with the version and a link. Use for GitHub Actions, GitLab CI/CD, Azure Pipelines or Jenkins, or when the user asks to be told when a deploy finishes or fails, or how to tell whether an error comes from the new release.
---

# Deploy messages

The first hour after a deploy goes better when the chat says three things: the deploy finished, which version an error comes from, and whether an error is new or came back.

## Rules

- The pipeline uses the project's key from the CI system's secret store, under the name `BUGSRADAR_KEY` (GitHub Actions and GitLab CI/CD variable, Azure Pipelines secret variable, Jenkins credential `bugsradar-key`). Never read, ask for or print the key and never put it in the repository. The user creates the secret.
- Add steps; do not change the existing build and deploy steps.

## Steps

1. Identify the CI system from the repository (`.github/workflows`, `.gitlab-ci.yml`, `azure-pipelines.yml`, `Jenkinsfile`).
2. Open `references/ci-systems.md` and add:
   - a failure step: runs only when an earlier step failed, sends the repository, the branch and a link to the run, with `category` set to the CI system and `environment` set to the target environment;
   - optionally a success step: `level=information`, with the commit hash and the environment.
3. Decide with the user whether the failure alert is for every branch or for the main branch only, and show the condition for that.
4. Tell the user to create the `BUGSRADAR_KEY` secret and how to test: break a step on purpose in a throwaway branch, or run the alert step by hand.

## Why it works

- Each deploy is a message of its own, because the commit hash differs. A failed pipeline sends its own message with the link to the run; the link changes each time but does not make repeats look different (URLs are ignored when messages are compared).
- Every message from the application carries the version next to the environment and host (`Production · web-1 · v1.4.3`). The .NET package takes the version from the entry assembly; in Node.js pass `appVersion`, in Python pass `app_version`. An error from the old build and the same error from the new one are told apart at a glance.
- A regression looks new. When an error stops and a window passes without repeats, it is closed; if the deploy brings it back, it arrives as a new first message with the stack trace.
- Deploy to staging first and watch its channel; set the `environment` so Staging and Production are counted apart.

## Suggest

- Setting `appVersion` / `app_version` / `AppVersion` in the application so errors show the release (the `setup` skill).
- Using a channel of its own for deploy messages if they should not sit next to errors: one project per purpose.
