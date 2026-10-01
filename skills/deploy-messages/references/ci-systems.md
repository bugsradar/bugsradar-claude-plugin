# CI/CD pipelines

Source: https://bugsradar.com/guides/ci-cd/

Add one step that runs only when the pipeline fails. Keep the key in the secrets of the CI system, never in the repository. The link to the run in the message changes every time, but it does not make repeats look different (URLs are ignored when messages are compared). The user creates the secret named `BUGSRADAR_KEY` themselves.

## GitHub Actions

Last step of the job; `if: failure()` runs it only when an earlier step failed. Only for the main branch: `if: failure() && github.ref == 'refs/heads/main'`.

```yaml
      - name: Alert BugsRadar
        if: failure()
        env:
          BUGSRADAR_KEY: ${{ secrets.BUGSRADAR_KEY }}
        run: |
          curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY" \
            --data-binary "Deploy of $GITHUB_REPOSITORY failed: $GITHUB_SERVER_URL/$GITHUB_REPOSITORY/actions/runs/$GITHUB_RUN_ID" \
            "https://api.bugsradar.com/api/v3/notify?category=GitHub+Actions&environment=Production"
```

## GitLab CI/CD

A job in the built-in `.post` stage that runs when a job of an earlier stage fails. The key is a masked CI/CD variable named `BUGSRADAR_KEY`.

```yaml
alert-bugsradar:
  stage: .post
  when: on_failure
  image:
    name: curlimages/curl:latest
    entrypoint: [""]
  script:
    - >
      curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY"
      --data-binary "Pipeline of $CI_PROJECT_PATH failed on $CI_COMMIT_REF_NAME: $CI_PIPELINE_URL"
      "https://api.bugsradar.com/api/v3/notify?category=GitLab+CI"
```

Only for the default branch, replace `when: on_failure` with a rule:

```yaml
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: on_failure
```

## Azure Pipelines

Last step of the job. The key is a secret variable named `BUGSRADAR_KEY`; the `$(...)` syntax puts secret variables into the script.

```yaml
- script: >
    curl -fsS --max-time 15 -H "X-Api-Key: $(BUGSRADAR_KEY)"
    --data-binary "Pipeline $(Build.DefinitionName) failed: $(System.CollectionUri)$(System.TeamProject)/_build/results?buildId=$(Build.BuildId)"
    "https://api.bugsradar.com/api/v3/notify?category=Azure+Pipelines"
  displayName: Alert BugsRadar
  condition: failed()
```

Only for the main branch: `condition: and(failed(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))`.

## Jenkins

The `post` section of a declarative pipeline. The key is a Secret text credential with the ID `bugsradar-key`. The script is in single quotes, so the key reaches the shell as a variable and never appears in the Groovy code.

```groovy
post {
    failure {
        withCredentials([string(credentialsId: 'bugsradar-key', variable: 'BUGSRADAR_KEY')]) {
            sh '''
                curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY" \
                  --data-binary "Build $JOB_NAME #$BUILD_NUMBER failed: $BUILD_URL" \
                  "https://api.bugsradar.com/api/v3/notify?category=Jenkins"
            '''
        }
    }
}
```

## Report finished deploys

To see each release in the chat, send an information message when the deploy succeeds. GitHub Actions:

```yaml
      - name: Tell BugsRadar
        if: success()
        env:
          BUGSRADAR_KEY: ${{ secrets.BUGSRADAR_KEY }}
        run: |
          curl -fsS --max-time 15 -H "X-Api-Key: $BUGSRADAR_KEY" \
            --data-binary "Deployed $GITHUB_REPOSITORY at $GITHUB_SHA" \
            "https://api.bugsradar.com/api/v3/notify?category=GitHub+Actions&level=information"
```

It arrives with the level Information. The commit hash differs from deploy to deploy, so every deploy arrives as a message of its own.
