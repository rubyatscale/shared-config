# shared-config

Shared reusable GitHub Actions workflows for [rubyatscale](https://github.com/rubyatscale) gems.

## Workflows

### Reusable workflows (`workflow_call`)

These workflows are called from individual gem repos via `uses: rubyatscale/shared-config/.github/workflows/<name>.yml@main`.

| Workflow | Description |
|----------|-------------|
| **CI** (`ci.yml`) | Runs tests across Ruby 3.3–4.0, Sorbet type checking, and linting (RuboCop). Test and linter commands are configurable via inputs. |
| **CD** (`cd.yml`) | Publishes the gem to RubyGems and creates a GitHub Release on successful main builds. |
| **Stale** (`stale.yml`) | Marks issues and PRs as stale after 180 days of inactivity, then closes them after 7 more days. |
| **Triage** (`triage.yml`) | Labels new issues with `triage`. |
| **CodeQL** (`codeql.yml`) | Runs [CodeQL](https://codeql.github.com) analysis for the given languages and uploads the results to the Security tab. |
| **zizmor** (`zizmor.yml`) | Runs the [zizmor](https://github.com/zizmorcore/zizmor) security linter against the calling repo's workflows, actions, and Dependabot config. |

### Repository workflows

| Workflow | Description |
|----------|-------------|
| **CodeQL self-scan** (`codeql-self-scan.yml`) | Runs `codeql.yml` against this repo's workflows on pushes and PRs to main, and weekly. |
| **zizmor self-scan** (`zizmor-self-scan.yml`) | Runs `zizmor.yml` against this repo on every push and PR. |

## Usage

In a gem repo, create a workflow that calls the shared workflow:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    uses: rubyatscale/shared-config/.github/workflows/ci.yml@main
```

### CI inputs

| Input | Default | Description |
|-------|---------|-------------|
| `test-command` | `bundle exec rspec` | Command to run tests |
| `linter-command` | `bundle exec rubocop` | Command to run the linter |

### CodeQL

The job requests `actions: read`, `contents: read` and `security-events: write`, so the calling job must grant all three.

```yaml
# .github/workflows/codeql.yml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '30 1 * * 0'

permissions: {}

jobs:
  analyze:
    permissions:
      actions: read
      contents: read
      security-events: write
    uses: rubyatscale/shared-config/.github/workflows/codeql.yml@main # zizmor: ignore[unpinned-uses] internal reusable workflow tracked at @main by convention
    with:
      languages: '["actions","ruby"]'
```

| Input | Default | Description |
|-------|---------|-------------|
| `languages` | (required) | JSON array of [CodeQL languages](https://codeql.github.com/docs/codeql-overview/supported-languages-and-frameworks/) to analyze, e.g. `'["actions","ruby"]'` |

### zizmor

By default the results go to the repository's Security tab, and the job passes whatever zizmor finds. To make zizmor a required check, set `advanced-security: false`: findings are then reported as annotations and fail the job. The check is named `<calling job id> / zizmor`, so require `zizmor / zizmor` with the example below. The job takes its permissions from the caller.

```yaml
# .github/workflows/zizmor.yml
name: zizmor

on:
  push:
    branches: [main]
  pull_request:

permissions: {}

jobs:
  zizmor:
    permissions:
      contents: read
    uses: rubyatscale/shared-config/.github/workflows/zizmor.yml@main # zizmor: ignore[unpinned-uses] internal reusable workflow tracked at @main by convention
    with:
      advanced-security: false
```

With the default `advanced-security: true`, grant `security-events: write` as well, plus `actions: read` in a private repo.

Callers track `@main`, so when this repo upgrades zizmor-action, new audits can start failing the check in every caller at once.

| Input | Default | Description |
|-------|---------|-------------|
| `advanced-security` | `true` | Upload results to the Security tab (`true`), or annotate and fail on findings (`false`) |

### Required secrets

The **CD** workflow requires the following secrets in the calling repo:

- `GUSTO_GIT_EMAIL` / `GUSTO_GIT_NAME` — Git identity for tagging
- `RUBYGEMS_API_KEY` — API key for publishing to RubyGems
- `SLACK_WEBHOOK_URL` — Incoming webhook URL for failure notifications (used by both CI and CD)

## Security

- All action references are pinned to SHA hashes
- [zizmor](https://github.com/zizmorcore/zizmor) runs on every PR to lint workflows for security issues
- [Dependabot](https://docs.github.com/en/code-security/dependabot) is configured for monthly GitHub Actions updates