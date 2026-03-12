# Privacy Notice

This notice applies to `trajectly/trajectly-action`.

## Processing Model

The action runs inside the user's GitHub Actions runner.

By default, this action does not upload run artifacts, reports, or traces to Trajectly-managed servers.

## Data Used by the Action

Depending on configuration, the action may access:

- repository files required to run the workflow
- GitHub workflow and pull request context
- generated report files under `.trajectly/**`

If `comment_pr: "true"` is enabled, report summary content is posted as a GitHub pull request comment.

If `upload_artifacts: "true"` is enabled, `.trajectly/**` is uploaded to GitHub Actions artifacts for that run.

## Data Sharing

This action itself does not send data to a Trajectly-hosted backend service.

Data may still be processed by:

- GitHub, as part of GitHub Actions and artifact/comment features
- external providers called by the user's own code (for example model APIs)

## User Controls

Users control behavior through:

- workflow permissions
- repository secrets
- action inputs (`comment_pr`, `upload_artifacts`)
- artifact retention settings in GitHub
