# Trajectly GitHub Action

Official GitHub Action for running Trajectly deterministic regression checks in CI.

This action is a thin wrapper around the CLI. TRT logic stays in the `trajectly` package.

## What it does

1. Installs Trajectly (`pypi` by default, pinned to `trajectly_version`)
2. Runs `python -m trajectly run ...`
3. Generates optional PR comment markdown with `python -m trajectly report --pr-comment`
4. Optionally posts/updates PR comment
5. Optionally uploads `.trajectly/**` artifacts
6. Exits with the run verdict code (`0/1/2`)

## Minimal usage (read-only permissions)

```yaml
name: Trajectly
on: [push, pull_request]

jobs:
  trajectly:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: trajectly/trajectly-action@v1.0.1
        with:
          spec_glob: "specs/*.agent.yaml"
          project_root: "."
```

## Version selection

- `@v1.0.1` is the recommended default for reproducible runs.
- `@v1` tracks the latest stable `v1.x.y` patch release automatically.
- `@<full_commit_sha>` provides the strongest supply-chain pinning.

## End-to-end demo (pass + regression)

For a full agent demo, use the Procurement Approval demo:
<https://github.com/trajectly/procurement-approval-demo>

Full walkthroughs with detailed observed outputs:
- <https://github.com/trajectly/procurement-approval-demo/blob/main/README.md>
- <https://github.com/trajectly/procurement-approval-demo/blob/main/TUTORIAL.md>

You can run this workflow to see both expected outcomes in one run:

```yaml
name: Trajectly E2E Demo
on: [workflow_dispatch]

jobs:
  pass:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: trajectly/trajectly-action@v1.0.1
        with:
          spec_glob: "specs/trt-procurement-agent-baseline.agent.yaml"
          project_root: "."

  regression_expected_failure:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
      - id: regression
        uses: trajectly/trajectly-action@v1.0.1
        continue-on-error: true
        with:
          spec_glob: "specs/trt-procurement-agent-regression.agent.yaml"
          project_root: "."
      - name: Assert regression is intentional
        shell: bash
        run: |
          set -euo pipefail
          if [ "${{ steps.regression.outcome }}" != "failure" ]; then
            echo "Expected regression run to fail."
            exit 1
          fi
          test -f .trajectly/reports/latest.md
          grep -F 'trt: `FAIL`' .trajectly/reports/latest.md
```

Expected results:

- `pass` job: action succeeds and report includes `trt: \`PASS\``.
- `regression_expected_failure` job: action step fails with regression (`exit code 1`), and the assertion step confirms `trt: \`FAIL\``.
- Artifacts (when enabled) include `.trajectly/reports/latest.md`, `.trajectly/reports/latest.json`, and repro files under `.trajectly/repros/`.

## PR comment usage

Enable PR comments explicitly and grant `pull-requests: write`.

```yaml
jobs:
  trajectly:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: trajectly/trajectly-action@v1.0.1
        with:
          comment_pr: "true"
```

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `spec_glob` | `specs/*.agent.yaml` | Spec files/glob passed to `trajectly run` |
| `project_root` | `.` | Working directory for run/report steps |
| `python_version` | `3.11` | Python version installed via `actions/setup-python` |
| `trajectly_version` | `0.4.2` | Trajectly package version used when `install: pypi` |
| `install` | `pypi` | `pypi` => install `trajectly==<trajectly_version>`, `editable` => `pip install -e <project_root>` |
| `comment_pr` | `false` | Post/update PR comment with report markdown |
| `upload_artifacts` | `true` | Upload `${project_root}/.trajectly/**` artifact |

## Exit codes

| Code | Meaning |
|---|---|
| `0` | All specs passed |
| `1` | Regression(s) detected |
| `2` | Config or tooling error |

## Security and pinning

For stronger supply-chain control, pin to immutable refs or a full commit SHA:

```yaml
- uses: trajectly/trajectly-action@v1.0.1
# or track stable v1 patch updates
- uses: trajectly/trajectly-action@v1
# or
- uses: trajectly/trajectly-action@<full_commit_sha>
```

## Release policy

- `vX.Y.Z` tags are immutable releases.
- Major tags move to latest stable patch release (`v1` -> newest `v1.x.y`).
