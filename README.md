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
[safe-settings](#org-wide-repository-settings-via-githubsafe-settings), configured in this repo.

Changes to the preset are validated by `.github/workflows/validate-renovate.yml`
before merge.

## Org-wide repository settings via `github/safe-settings`

Repository settings, labels and branch rulesets are enforced across the org using
[github/safe-settings](https://github.com/github/safe-settings). The workflow runs on
every push to `main` (when `safe-settings/` files change) and as a daily cron job for
drift correction. Daily rather than weekly because a repo created after a sync is
completely unmanaged until the next one, and nothing reports it.

Configuration files:
- `safe-settings/settings.yml` — org-wide repository defaults and the label set
- `safe-settings/suborgs/all.yml` — the default branch ruleset, for repos with no
  suborg file of their own (per-repo, see note below)
- `safe-settings/suborgs/00-*.yml` — one per repo that needs a ruleset different
  from that default
- `safe-settings/deployment-settings.yml` — which repos are in scope

### Repos out of scope

| Repo | Why |
|------|-----|
| `.github-private` | Archived — read-only, and safe-settings fails the sync when it can't write |

Archived or private repos have to be listed there too — see "Archiving or making
a repo private" below. None are excluded today: `webshooter` and the six
`rotation_target_*` component repos were deleted on 2026-09-05.

`webshooter-cli` was **partially** managed until it was made public: private repos
on a free plan return 403 from the rulesets API, and safe-settings exits non-zero on
any API error, so one 403 would have failed the whole run. It therefore had a suborg
with no `rulesets:` key — settings and labels managed, branch protection not. Making
it public removed the constraint, and `suborgs/00-webshooter-cli.yml` now carries a
full `protect-main` with its three required checks.

### Archiving or making a repo private

**Both break the sync, and safe-settings will not tell you in advance.** There is
no skip for either state — `updateRepos` runs every plugin regardless — so the
exclusion is manual and has to happen at the same time as the archive or the
visibility change.

| Change | What breaks | Why |
|--------|-------------|-----|
| Make private | rulesets, immediately | On the free plan the rulesets API 403s for private repos. The plugin 403s on its `find()` GET before it writes anything, so the repo fails even when nothing needs changing. |
| Archive | any plugin that needs to write | Archived repos are read-only. A converged repo can pass for a while — nothing needs writing — and then fail the first time the config changes. |

One 403 fails the whole run, for every repo, not just the one.

**When you archive a repo or make it private, add it to
`safe-settings/deployment-settings.yml` in the same change.** A repo that stays
managed after either will fail the next sync, or the one after the next config
edit, with nothing linking the failure back to the change that caused it.

The alternative for a private repo that should stay managed is its own suborg
file with no `rulesets:` key — settings and labels managed, branch protection
not. That is what `suborgs/00-webshooter-cli.yml` was until the repo was made
public. It does not help an archived repo, which cannot be written to at all.

### Why rulesets live in `suborgs/`, not `settings.yml`

A top-level `rulesets:` block in `settings.yml` is created as an **organization**
ruleset, which requires GitHub Team. This org is on the free plan, so the rulesets
are defined at suborg scope instead, which makes safe-settings create them as
per-repo rulesets — free on any plan.

`suborgrepos` lists both `"*"` and `".*"`. safe-settings matches with minimatch,
where `*` does not match a leading dot, so `"*"` alone silently skips `.github` —
it would still get org-level settings and labels, but no rulesets, and the sync
would stay green while doing it.

`suborgs/all.yml` also carries an empty `repository: {}`. That is a workaround for a
safe-settings bug: without a suborg entry every repo shares one mutated config
object, and repos can end up with each other's names.

### safe-settings deletes rulesets it doesn't know about

The rulesets plugin removes any repository ruleset that is not listed in
`suborgs/all.yml`. Adding a ruleset through the GitHub UI means it survives until
the next sync and is then deleted. Add it to `all.yml` instead — a ruleset can be
scoped to individual repos with an `include:` list if it genuinely needs to be.

`protect-main` deliberately covers `release/*` and `hotfix/*` alongside the default
branch on every repo, rather than existing as a per-repo exception. No repo uses
those branches yet; the point is that the day one does, it is already protected.

### Renovate automerge rests on the required-check list

The org preset sets `platformAutomerge: true`, so a Renovate pull request merges
itself as soon as the branch is mergeable. The ruleset is what makes that safe:
every check that runs on every pull request has to be a required check, or
auto-merge has nothing to wait for. `.github` required none, and merged #29
twenty-three seconds before its own validator finished.

Two ways a required check backfires, both of which block a pull request forever:
a **path-filtered** workflow that does not run never reports, and a **PR-policy**
check that a Renovate pull request cannot satisfy — a branch-name or title rule
that does not allow `renovate/…` and `chore(<datasource>): …` — fails on every one
of them. Confirm a context against a real Renovate pull request before requiring it.

Every required context carries `integration_id: 15368` (GitHub Actions). A context
is only a string, so an unpinned entry is satisfied by whichever app reports that
name first; the id ties it to the workflow that is meant to answer for it.

Renovate has no ruleset bypass. The `renovate-approve` App (id 7394, installed
org-wide) supplies the approval instead, so Renovate is held to the same rules as a
human author. `dismiss_stale_reviews_on_push` drops that approval when Renovate
pushes into an open pull request, and the App re-approves.

`required_approving_review_count` is 1 everywhere. With a single human author, who
cannot approve their own pull request, that means a human pull request — every layer
of a stack included — needs the merge-async endpoint as an OrganizationAdmin to
land. That cost is accepted in exchange for the setup being identical across orgs.

### GitHub App setup (manual, one-time)

safe-settings requires a GitHub App installed on the org:

1. **Create the app** in https://github.com/organizations/Malmo-Skyttegille-Pistolsektionen/settings/apps
   - Required permissions:
     - `Repository administration: Write` — repo settings, rulesets, vulnerability-alert toggles
     - `Repository checks: Write` — posts the run's own pass/fail check on the commit
     - `Repository contents: Read` — reads `safe-settings/*.yml` from this repo
     - `Repository environments: Write` — environments and their required reviewers
     - `Repository issues: Write` — **labels are part of the Issues permission**
     - `Repository metadata: Read` — mandatory, granted automatically

   That is the minimal set for the plugins in use here (`repository`, `labels`,
   `rulesets`, `environments`). The upstream docs list many more because they
   cover every plugin.

   Adding a permission later takes two steps, not one: change it on the app, then
   approve it on the installation
   (https://github.com/organizations/Malmo-Skyttegille-Pistolsektionen/settings/installations/155051310).
   Additions stay inert until approved; removals apply immediately.

   Without `checks`, a failed sync shows up only as a red Actions run and not as a
   check on the commit, which is how one went unnoticed for two hours.
2. **Install the app** on *All repositories* in the org
3. **Generate a Private Key** from the app settings page (download the `.pem`)
4. **Add to this repo** (https://github.com/Malmo-Skyttegille-Pistolsektionen/.github/settings):
   - Variable `SAFE_SETTINGS_APP_ID` → the App ID integer
   - Secret `SAFE_SETTINGS_PRIVATE_KEY` → full contents of the `.pem` file

Until those two exist the workflow runs and fails.

### Preview mode is broken upstream

safe-settings has no `DRY_RUN` variable — the knob is `FULL_SYNC_NOP`, exposed as the
`nop` input on manual dispatch. It does not currently work: in 2.1.18 the NOP
reporting path crashes, because `lib/settings.js:274` dereferences
`y.action.additions` before the `undefined` guard three lines below it. Disabling
`CREATE_PR_COMMENT` only moves the crash elsewhere.

Until that is fixed upstream, there is no way to preview a run. Diff the intended
config against the live API by hand instead.

> **Note:** Secret scanning is not configurable via safe-settings YAML and must be
> enabled per repo or via the org security settings.

---

## Org-level security settings (manual, one-time)

These live at
**github.com/organizations/Malmo-Skyttegille-Pistolsektionen/settings/security_analysis**
and cannot be managed by safe-settings, which is repo-scoped only.

| Setting | State | Reason |
|---------|-------|--------|
| Dependabot alerts | **On** | Free; alerts only, no PRs created |
| Dependabot grouped security updates | **Off** | Renovate owns dependency updates; enabling this creates duplicate PRs |
| Secret scanning (alerts) | **On** | Free for public repos |
| Secret scanning push protection | **On** | Free for public repos; blocks accidental secret commits |

### Paid features — out of scope

These require GitHub Advanced Security / Code Security and are not available on the
free plan: extended CodeQL query suites, Copilot Autofix, bulk org-level CodeQL, and
secret scanning push protection for **private** repos.

### CodeQL per-repo setup (free for public repos)

Bulk enablement via the org UI requires GHAS. For public repos it can be enabled per
repo for free via the API — one-time bootstrap, run locally:

```bash
gh repo list Malmo-Skyttegille-Pistolsektionen --visibility public --no-archived --json name --jq '.[].name' \
| while read repo; do
    echo "Enabling CodeQL on $repo..."
    gh api --method PATCH \
      /repos/Malmo-Skyttegille-Pistolsektionen/"$repo"/code-scanning/default-setup \
      -f state=configured \
      -f query_suite=extended \
      && echo "  OK" || echo "  FAILED (may need GHAS or repo has no supported language)"
  done
```

Repos with no supported language fail gracefully and can be ignored.
