# Upstream fix for `Brighty28/Local-Gov-Umbraco`

This folder holds the artefacts needed to open a PR against the upstream package repo that fixes the "empty back-office on first boot" issue observed when running this demo site.

## Files

- **`schema-seeding.patch`** — unified diff against `Brighty28/Local-Gov-Umbraco@main`. Modifies three files:
  - `Directory.Build.props`
  - `Directory.Packages.props`
  - `src/LocalGov.Umbraco.Core/LocalGov.Umbraco.Core.csproj`
- **`PR_DESCRIPTION.md`** — ready-to-paste PR title, summary, rationale, verification steps, and checklist.

## How to use

```bash
# 1. Clone the upstream repo (not a fork yet — we'll fork on push)
git clone https://github.com/Brighty28/Local-Gov-Umbraco.git
cd Local-Gov-Umbraco

# 2. Branch and apply the patch
git checkout -b fix/schema-seeding
git apply /path/to/Greenfield-District-Council/docs/upstream-fix/schema-seeding.patch

# 3. Verify it builds
dotnet build --configuration Release

# 4. Commit
git add Directory.Build.props Directory.Packages.props \
        src/LocalGov.Umbraco.Core/LocalGov.Umbraco.Core.csproj
git commit -m "Fix empty back-office on first boot: embed uSync schemas and reference uSync metapackage"

# 5. Push to your fork and open the PR using the body from PR_DESCRIPTION.md
gh repo fork Brighty28/Local-Gov-Umbraco --remote --remote-name fork
git push -u fork fix/schema-seeding
gh pr create --repo Brighty28/Local-Gov-Umbraco \
             --base main \
             --head "$(gh api user --jq .login):fix/schema-seeding" \
             --title "Fix empty back-office on first boot: embed uSync schemas and reference uSync metapackage" \
             --body-file /path/to/Greenfield-District-Council/docs/upstream-fix/PR_DESCRIPTION.md
```

## If `git apply` complains about whitespace or context

The patch uses standard unified-diff format with 3-line context. Fallbacks in order of preference:

```bash
# Fuzzy match with the system patch tool
patch -p1 --fuzz=3 < /path/to/schema-seeding.patch

# Or let git's 3-way merge resolve minor context drift
git apply --3way /path/to/schema-seeding.patch

# Or hand-apply: the three edits are small enough that PR_DESCRIPTION.md
# documents the final state of each file section. Worst case, copy the
# xml snippets from there.
```

## After the upstream PR is merged

Once the upstream packages are re-released (e.g. `1.0.1`), update this repo:

- Bump `LocalGov.Umbraco` in `src/GreenfieldDC.WebUI/GreenfieldDC.WebUI.csproj` to the new version.
- Add the required uSync settings to `src/GreenfieldDC.WebUI/appsettings.json`:
  ```json
  "uSync": { "Settings": { "ImportAtStartup": "All" } }
  ```
- Delete this `docs/upstream-fix/` folder — the fix has shipped.
