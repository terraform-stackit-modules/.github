# Rulesets — terraform-stackit-modules

Branch- and tag-protection rulesets for the organization's module repositories.

> **Why repo-level?** Organization-level rulesets are **not enforced** on the GitHub **Free**
> plan (they require GitHub **Team** or above). The `*.json` files here are org-level definitions,
> kept for the day the org upgrades.

## Files

| File | Scope | Use |
|------|-------|-----|
| `protect-main.json` | Repository   | Apply per repo on Free plan |
| `protect-tags.json` | Repository   | Apply per repo on Free plan |

`protect-main` protects `refs/heads/main` (PR + 1 review, thread resolution, dismiss stale, strict
status checks `Max TF pre-commit` + `Validate PR title`, required signatures, no deletion /
force-push). `protect-tags` makes release tags `refs/tags/v*` immutable (no deletion / update /
non-fast-forward), so a published Registry version can never move.

## Prerequisites

```bash
# GitHub CLI authenticated with an org admin account
gh auth status
# run all commands from this directory
cd .github/rulesets
```

## Apply per repository (Free plan)

Set the repository once, then POST both rulesets:

```bash
REPO=terraform-stackit-modules/terraform-stackit-dns   # change per repo

gh api -X POST "/repos/$REPO/rulesets" --input protect-main.json
gh api -X POST "/repos/$REPO/rulesets" --input protect-tags.json
```

### Apply to every module repo in one pass

```bash
cd .github/rulesets

for REPO in $(gh repo list terraform-stackit-modules --no-archived --limit 200 \
                --json name --jq '.[].name' | grep '^terraform-stackit-'); do
  echo "=== $REPO ==="
  gh api -X POST "/repos/terraform-stackit-modules/$REPO/rulesets" --input protect-main.json \
    && echo "  protect-main OK"
  gh api -X POST "/repos/terraform-stackit-modules/$REPO/rulesets" --input protect-tags.json \
    && echo "  protect-tags OK"
done
```

> Re-running POST creates a **duplicate** ruleset. To update an existing one, list and PUT instead:
> ```bash
> gh api "/repos/$REPO/rulesets"                       # find the ruleset id
> gh api -X PUT "/repos/$REPO/rulesets/<id>" --input protect-main.json
> ```

## Verify

```bash
REPO=terraform-stackit-modules/terraform-stackit-dns
gh api "/repos/$REPO/rulesets" --jq '.[] | {id, name, enforcement, target}'
```

## Apply org-wide (after upgrading to GitHub Team)

```bash
cd .github/rulesets
gh api -X POST /orgs/terraform-stackit-modules/rulesets --input protect-main.json
gh api -X POST /orgs/terraform-stackit-modules/rulesets --input protect-tags.json
```

The org-level files target every `terraform-stackit-*` repo automatically, so new repos are covered
without re-running anything.

## Notes

- `integration_id: 15368` is the GitHub App that publishes the `Max TF pre-commit` and
  `Validate PR title` status checks. Confirm it matches your org before applying.
- `required_signatures` blocks unsigned commits/tags. On a public repo open to external
  contributions this raises the bar — drop the rule if it deters casual PRs.
