# PR: Fix empty back-office on first boot — embed uSync schemas and reference uSync metapackage

**Target repo:** `Brighty28/Local-Gov-Umbraco`
**Base branch:** `main`
**Suggested branch name:** `fix/schema-seeding`
**Suggested title:** `Fix empty back-office on first boot: embed uSync schemas and reference uSync metapackage`

---

## Summary

- Ship `uSync/**/*` as `<EmbeddedResource>` instead of `<Content>` so `USyncDeploymentComponent` can actually find and materialise them.
- Replace `uSync.Core` with the full `uSync` metapackage in `LocalGov.Umbraco.Core` so `uSync.BackOffice` handlers flow transitively to consumers.
- Drop the now-unused `uSync.Core` version pin.

## Why

Spinning up the Greenfield District Council demo site (`Brighty28/Greenfield-District-Council`) against the published packages boots into an empty Umbraco back-office — none of the `lg*` document types are installed. Three linked issues combine to produce that:

### 1. `contentFiles` does not work with SDK-style `PackageReference`

`Directory.Build.props` packs the schemas with:

```xml
<Content Update="uSync\**\*"
         CopyToOutputDirectory="PreserveNewest"
         Pack="true"
         PackagePath="contentFiles\any\any\uSync\" />
```

`contentFiles` is a `packages.config`-era mechanism. Modern SDK projects (which is every Umbraco 13 site) ignore it, so the `.config` files never land in the consumer's `/uSync/v13/ContentTypes/` folder.

### 2. The runtime fallback reads embedded resources, but nothing is embedded

`src/LocalGov.Umbraco.Core/Components/USyncDeploymentComponent.cs` does:

```csharp
var resourceNames = assembly.GetManifestResourceNames()
    .Where(n => n.Contains(".uSync.v13."));
```

…which requires the schema files to be `<EmbeddedResource>`. The csproj items are `<Content>`, so `GetManifestResourceNames()` returns nothing matching and the component silently copies zero files.

### 3. No uSync.BackOffice handlers in the consumer graph

`LocalGov.Umbraco.Core.csproj` references `uSync.Core` only — that's the serializer library. None of the packages reference `uSync` / `uSync.BackOffice`, which is the part that reads `/uSync/v13/` on boot and imports schema into Umbraco. Consumers don't reference it either.

Any one of these alone would prevent schema seeding. All three together guarantee an empty back-office.

## Changes

Three files, small diffs — see `schema-seeding.patch`:

- **`Directory.Build.props`** — remove the `contentFiles` pack metadata for `uSync\**\*`, explicitly `<Content Remove>` the implicit RCL-SDK content inclusion, and add `<EmbeddedResource Include="uSync\**\*" />`. Resources end up with manifest names like `LocalGov.Umbraco.Core.uSync.v13.ContentTypes.lgHome.config`, which is exactly what `USyncDeploymentComponent.ResourceNameToFilePath` expects.
- **`src/LocalGov.Umbraco.Core/LocalGov.Umbraco.Core.csproj`** — switch `<PackageReference Include="uSync.Core" />` to `<PackageReference Include="uSync" />`. `uSync` depends on `uSync.BackOffice` which depends on `uSync.Core`, so nothing is lost and handlers become available.
- **`Directory.Packages.props`** — drop the orphaned `uSync.Core` `PackageVersion`, update the comment.

## What consumers still need to do

uSync 13 does **not** import on startup by default. Consumers must add this to their `appsettings.json` for freshly-deployed schemas to be imported:

```json
"uSync": {
  "Settings": {
    "ImportAtStartup": "All"
  }
}
```

This should be called out in the top-level `README.md` Quick Start (step 3). Happy to include that edit in this PR if preferred — left out here to keep the change strictly code.

## Follow-up (not in this PR)

A nicer UX would be to have `USyncDeploymentComponent` trigger `ISyncService.Import(folder)` itself whenever it deploys new files. That removes the consumer-side `appsettings.json` requirement entirely and gives true "install and go". Worth a separate PR once this one's merged.

## Verification

Locally against `Brighty28/Greenfield-District-Council`:

1. Apply the patch, bump package versions (e.g. `1.0.1-preview`), and `dotnet pack` all projects + the meta package into `./nupkgs`.
2. In the Greenfield repo, bump `LocalGov.Umbraco` to the new version in `src/GreenfieldDC.WebUI/GreenfieldDC.WebUI.csproj`.
3. Add the `uSync.Settings.ImportAtStartup = All` block to `appsettings.json`.
4. Delete any prior `umbraco.sqlite.db` and `/uSync` folder in the site, then `dotnet run`.
5. Complete the Umbraco installer (skip starter kit).
6. Expected: `/src/GreenfieldDC.WebUI/uSync/v13/ContentTypes/lgHome.config` (and siblings) are on disk, and the Settings → Document Types tree in the back-office contains `lgHome`, `lgSettings`, `lgNewsList`, `lgAlertList`, `lgServiceLanding`, `lgGuide`, `lgStepByStep`, etc.

## Checklist

- [ ] Patch applies cleanly to `main` (`git apply docs/upstream-fix/schema-seeding.patch`)
- [ ] `dotnet build` clean across all package projects
- [ ] `dotnet pack` produces nupkgs that contain the uSync `.config` files as embedded resources (verify with `unzip -l` on the nupkg's dll, or via ILSpy — they should appear under the resource table, not under `contentFiles/`)
- [ ] Smoke test against the Greenfield demo as described above — back-office shows all `lg*` document types after first boot
- [ ] README update (either in this PR or follow-up) documenting the `ImportAtStartup` requirement
