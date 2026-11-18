# shared-actions

Reusable GitHub Actions workflows shared across kevinklaes repositories.

- `action-security-check`: runs [zizmor](https://github.com/zizmorcore/zizmor) over a caller's workflows.
- `actions-lock-check`: verifies `.github/workflows/actions.lock` with `gh actions-lock --verify`,
  following https://hugovk.dev/blog/2026/lock-github-actions/.

Callers use them as lint jobs via `jobs.<id>.uses: kevinklaes/shared-actions/.github/workflows/<name>.yml@<tag>`.
