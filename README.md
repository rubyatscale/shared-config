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
| **zizmor** (`zizmor.yml`) | Runs the [zizmor](https://github.com/zizmorcore/zizmor) security linter against the calling repo's workflows, actions, and Dependabot config. |

### Repository workflows

| Workflow | Description |
|----------|-------------|
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

### zizmor

By default the results go to the repository's Security tab, and the job passes whatever zizmor finds. To make zizmor a required check, set `advanced-security: false`: findings are then reported as annotations and fail the job. The job takes its permissions from the caller.

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

With the default `advanced-security: true`, grant `actions: read` and `security-events: write` as well.

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