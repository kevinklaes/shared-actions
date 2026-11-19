# shared-actions

Reusable GitHub Actions workflows shared across kevinklaes repositories.

- `action-security-check`: runs [zizmor](https://github.com/zizmorcore/zizmor) over a caller's workflows.
- `actions-lock-check`: verifies `.github/workflows/actions.lock` with `gh actions-lock --verify`,
  following https://hugovk.dev/blog/2026/lock-github-actions/.

Callers use them as lint jobs via `jobs.<id>.uses: kevinklaes/shared-actions/.github/workflows/<name>.yml@<tag>`.

## Usage

`.github/workflows/lint-actions.yml` in the calling repo:

```yaml
name: Lint actions

on:
  pull_request:
  push:
    branches: [main]

permissions: {}

jobs:
  zizmor:
    uses: kevinklaes/shared-actions/.github/workflows/action-security-check.yml@<tag>
    permissions:
      contents: read

  actions-lock:
    uses: kevinklaes/shared-actions/.github/workflows/actions-lock-check.yml@<tag>
    permissions:
      contents: read
```

Neither workflow needs secrets. Both only need `contents: read`; no SARIF upload, so no
`security-events: write`.

**After editing any workflow, run `gh actions-lock` and commit the workflows and
`.github/workflows/actions.lock` together.** `actions-lock-check` fails when they drift apart.

## Developing this repo

`.github/workflows/ci.yml` runs both workflows against this repo through `$/` self-repository calls, so a
PR is checked by its own version of them. Dependabot (`.github/dependabot.yml`, weekly, 7-day cooldown)
proposes action bumps together with `actions.lock` changes.

The reusable workflows use tag refs (`actions/checkout@v7.0.1`). The pins live in this repo's
`actions.lock`, and the runner enforces them for callers too: a reusable workflow's dependencies are covered
by the lockfile of the repository that defines it
([docs](https://docs.github.com/en/actions/concepts/security/workflow-dependency-lockfiles)). Callers
pin this repo itself with the tag or SHA in their `jobs.<id>.uses`.

## Releases

- `v1.0.1`: `action-security-check` no longer fails on an `actions.lock` without dependencies (see below),
  and takes a `path` input. Callers can stay on `v1.0.0` unless they hit this; moving up means changing
  the `@v1.0.0` refs and running `gh actions-lock`.
- `v1.0.0`: first release.

## Inputs

### `action-security-check.yml`

| Input | Default | Description |
|-------|---------|-------------|
| `runs-on` | `'"ubuntu-slim"'` | JSON for `runs-on`, read with `fromJSON`. A self-hosted runner: `'["self-hosted", "training-mini"]'`. |
| `zizmor-version` | `1.32.0` | Exact zizmor version. Keep it at 1.32.0 or later: earlier versions don't understand `actions.lock` and demand hash-pinned `uses:`. |
| `min-severity` | `low` | Lowest severity that fails the job: `unknown`, `informational`, `low`, `medium` or `high`. The default fails on every real finding and ignores informational notes. |
| `path` | `.` | Directory to audit, relative to the checkout root. A subdirectory is audited as a repo of its own: its own `.github/workflows/actions.lock` and `zizmor.yml`. |

zizmor audits the whole checkout (workflows, composite actions, Dependabot config) and reports findings as
annotations. It picks up a `zizmor.yml` (repo root or `.github/`) on its own, so use that to ignore or tune
individual audits.

When the caller's workflows use no actions directly (only reusable workflows like these),
`gh actions-lock` writes an `actions.lock` with no `dependencies:` key, and zizmor 1.32.0 can't parse it
("missing field `dependencies`"). Such a lockfile pins nothing, so the workflow sets it aside for the zizmor
run and leaves a notice; `actions-lock-check` still verifies it. `.github/workflows/ci.yml` covers this case
with the fixture repo in `tests/fixtures/no-dependencies`.

### `actions-lock-check.yml`

| Input | Default | Description |
|-------|---------|-------------|
| `runs-on` | `'"ubuntu-slim"'` | JSON for `runs-on`, read with `fromJSON`. |
| `gh-actions-lock-version` | `v0.4.1` | Tag of [`github/gh-actions-lock`](https://github.com/github/gh-actions-lock) to install. |
