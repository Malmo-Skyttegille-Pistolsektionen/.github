# .github

Org-wide shared GitHub configuration for the
[Malmo-Skyttegille-Pistolsektionen](https://github.com/Malmo-Skyttegille-Pistolsektionen)
organization.

## Shared Renovate preset

`.github/renovate.json5` is the single source of truth for Renovate behaviour
across the org. Each repository opts in with a one-line `renovate.json5`:

```json5
{
  extends: ["github>Malmo-Skyttegille-Pistolsektionen/.github//.github/renovate.json5"],
}
```

Repo-specific overrides go in that file, below the `extends`.

### What the preset does

| Area | Behaviour |
|------|-----------|
| Versions | `rangeStrategy: "pin"` — every dependency pinned exactly, so upgrades arrive as reviewable PRs that CI must pass |
| Commits | Conventional Commits, `chore({{datasource}}): …` |
| Automerge | Minor and patch automerge after a 3-day cooldown; major waits 7 days and needs a human |
| Labels | `bot-renovate` on every PR, plus `renovate-type-*` (ecosystem) and `renovate-version-*` (major/minor/patch/digest) |
| Pausing | Add `bot-renovate-stop` to a PR to stop Renovate updating that branch |
| Actions | Third-party actions pinned to commit digests; `actions/*`, `github/*` and this org's own actions stay on major tags |
| Security | Vulnerability alerts and OSV alerts enabled, labelled `security` |

The labels the preset applies are created and kept in sync by
[safe-settings](https://github.com/Malmo-Skyttegille-Pistolsektionen/.github-private).

Changes to the preset are validated by `.github/workflows/validate-renovate.yml`
before merge.
