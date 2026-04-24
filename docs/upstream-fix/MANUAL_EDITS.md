# Manual edits (if `git apply` still gives you trouble)

If you'd rather just make the edits by hand, here's every change as a
simple find-and-replace in the `Brighty28/Local-Gov-Umbraco` repo. Three
files, three edits.

---

## 1. `Directory.Build.props`

Find this block (lines 26-30):

```xml
  <!-- uSync config files: the RCL SDK already includes them implicitly as Content.
       Use Update (not Include) to set packaging metadata on the already-included items. -->
  <ItemGroup>
    <Content Update="uSync\**\*" CopyToOutputDirectory="PreserveNewest" Pack="true" PackagePath="contentFiles\any\any\uSync\" />
  </ItemGroup>
```

Replace with:

```xml
  <!-- uSync config files ship as EmbeddedResource so the runtime deployment component
       (LocalGov.Umbraco.Core.Components.USyncDeploymentComponent) can materialise them
       to the host site's /uSync/v13 folder on first boot. contentFiles does not work
       with SDK-style PackageReference projects. -->
  <ItemGroup>
    <Content Remove="uSync\**\*" />
    <EmbeddedResource Include="uSync\**\*" />
  </ItemGroup>
```

---

## 2. `Directory.Packages.props`

Find this block (lines 16-18):

```xml
    <!-- uSync -->
    <PackageVersion Include="uSync" Version="13.3.2" />
    <PackageVersion Include="uSync.Core" Version="13.3.2" />
```

Replace with:

```xml
    <!-- uSync: the full metapackage pulls in uSync.BackOffice, required for startup import of embedded schemas -->
    <PackageVersion Include="uSync" Version="13.3.2" />
```

(The `uSync.Core` line is deleted; the comment is updated.)

---

## 3. `src/LocalGov.Umbraco.Core/LocalGov.Umbraco.Core.csproj`

Find this line (line 14):

```xml
    <PackageReference Include="uSync.Core" />
```

Replace with:

```xml
    <PackageReference Include="uSync" />
```

---

## After editing

```bash
dotnet build --configuration Release
git add Directory.Build.props Directory.Packages.props \
        src/LocalGov.Umbraco.Core/LocalGov.Umbraco.Core.csproj
git commit -m "Fix empty back-office on first boot: embed uSync schemas and reference uSync metapackage"
```

Then push to your fork and open the PR using the body from
`PR_DESCRIPTION.md`.
