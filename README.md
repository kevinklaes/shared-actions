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

## Inputs

### `action-security-check.yml`

| Input | Default | Description |
|-------|---------|-------------|
| `runs-on` | `'"ubuntu-slim"'` | JSON for `runs-on`, read with `fromJSON`. A self-hosted runner: `'["self-hosted", "training-mini"]'`. |
| `zizmor-version` | `1.32.0` | Exact zizmor version. Keep it at 1.32.0 or later: earlier versions don't understand `actions.lock` and demand hash-pinned `uses:`. |
| `min-severity` | `low` | Lowest severity that fails the job: `unknown`, `informational`, `low`, `medium` or `high`. The default fails on every real finding and ignores informational notes. |

zizmor audits the whole checkout (workflows, composite actions, Dependabot config) and reports findings as
annotations. It picks up a `zizmor.yml` (repo root or `.github/`) on its own, so use that to ignore or tune
individual audits.

### `actions-lock-check.yml`

| Input | Default | Description |
|-------|---------|-------------|
| `runs-on` | `'"ubuntu-slim"'` | JSON for `runs-on`, read with `fromJSON`. |
| `gh-actions-lock-version` | `v0.4.1` | Tag of [`github/gh-actions-lock`](https://github.com/github/gh-actions-lock) to install. |
