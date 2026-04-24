# Upstream fix for `Brighty28/Local-Gov-Umbraco`

This folder holds the artefacts needed to open a PR against the upstream
package repo that fixes the "empty back-office on first boot" issue
observed when running this demo site.

## Files

- **`schema-seeding.patch`** — unified diff against
  `Brighty28/Local-Gov-Umbraco@main`. Verified to apply cleanly with
  `git apply --check` (commit `6b2d37c` and later).
- **`PR_DESCRIPTION.md`** — drop-in PR title, summary, rationale,
  verification steps, and checklist.
- **`MANUAL_EDITS.md`** — if `git apply` gives you trouble, this is the
  same change expressed as three simple find-and-replace edits you can
  make by hand.

## How to apply the patch (Git Bash on Windows)

`git apply` must be run **from inside the Local-Gov-Umbraco repo**, not
from inside this one. The earlier "corrupt patch" error was because the
patch file was malformed — it's now been regenerated and verified.

```bash
# 1. Clone the upstream repo somewhere separate from this one
cd /c/dev
git clone https://github.com/Brighty28/Local-Gov-Umbraco.git
cd Local-Gov-Umbraco

# 2. Create a branch
git checkout -b fix/schema-seeding

# 3. Apply the patch — note the path points *out* of this repo, into
#    the Greenfield checkout next to it. Adjust to match where you
#    cloned Greenfield-District-Council.
git apply /c/dev/Greenfield-District-Council/docs/upstream-fix/schema-seeding.patch

# 4. Sanity check: three files should be modified
git status
# Expected:
#   modified:   Directory.Build.props
#   modified:   Directory.Packages.props
#   modified:   src/LocalGov.Umbraco.Core/LocalGov.Umbraco.Core.csproj

# 5. Build to confirm nothing breaks
dotnet build --configuration Release

# 6. Commit
git add Directory.Build.props Directory.Packages.props \
        src/LocalGov.Umbraco.Core/LocalGov.Umbraco.Core.csproj
git commit -m "Fix empty back-office on first boot: embed uSync schemas and reference uSync metapackage"
```

## Common errors

### `error: corrupt patch at line N`

Older copies of the patch file in this repo had malformed blank-line
prefixes and wrong hunk counts. Re-pull the latest from
`claude/local-gov-umbraco-cms-gnTl4` — the current `schema-seeding.patch`
has been regenerated with `diff -u` from the real upstream files and
verified with `git apply --check`.

### `error: patch does not apply`

Either:
1. You ran `git apply` from the wrong directory. It must be from the
   root of the `Local-Gov-Umbraco` clone.
2. The upstream files have moved on since the patch was generated (date:
   2026-04-24). In that case, use **`MANUAL_EDITS.md`** — the edits are
   small and well-defined.

### `git apply` complains about whitespace

Try the more forgiving fallbacks in this order:

```bash
# Let git's 3-way merge handle minor context drift
git apply --3way /c/dev/Greenfield-District-Council/docs/upstream-fix/schema-seeding.patch

# Or use the system patch tool with fuzz
patch -p1 --fuzz=3 < /c/dev/Greenfield-District-Council/docs/upstream-fix/schema-seeding.patch
```

If both fail, fall back to `MANUAL_EDITS.md`.

## Opening the PR

```bash
# Fork and push (requires gh CLI authenticated)
gh repo fork Brighty28/Local-Gov-Umbraco --remote --remote-name fork
git push -u fork fix/schema-seeding

# Open the PR with the description from this repo
gh pr create --repo Brighty28/Local-Gov-Umbraco \
             --base main \
             --head "$(gh api user --jq .login):fix/schema-seeding" \
             --title "Fix empty back-office on first boot: embed uSync schemas and reference uSync metapackage" \
             --body-file /c/dev/Greenfield-District-Council/docs/upstream-fix/PR_DESCRIPTION.md
```

## After the upstream PR is merged

Once the packages are re-released (e.g. `1.0.1`), update this repo:

- Bump `LocalGov.Umbraco` in `src/GreenfieldDC.WebUI/GreenfieldDC.WebUI.csproj`.
- Add the required uSync settings to `src/GreenfieldDC.WebUI/appsettings.json`:
  ```json
  "uSync": { "Settings": { "ImportAtStartup": "All" } }
  ```
- Delete this `docs/upstream-fix/` folder — the fix has shipped.
