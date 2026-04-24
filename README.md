# Greenfield District Council — LocalGov Umbraco Demo Site

A fictional UK district council demonstrating all Phase 1 [LocalGov Umbraco](https://github.com/Brighty28/Local-Gov-Umbraco) packages running together on Umbraco 13 with GOV.UK Frontend v5.

## What's included

| Module | Package |
|---|---|
| GOV.UK Frontend theme + per-council branding | `LocalGov.Umbraco.Theme` |
| Core helpers, breadcrumb, side nav, pagination | `LocalGov.Umbraco.Core` |
| Service landing pages + navigation cards | `LocalGov.Umbraco.Services` |
| Newsroom, news articles, category filtering | `LocalGov.Umbraco.News` |
| Alert banners (announcement/minor/major/notable death) | `LocalGov.Umbraco.AlertBanner` |
| Multi-page guides with sidebar nav | `LocalGov.Umbraco.Guides` |
| GOV.UK step-by-step journeys | `LocalGov.Umbraco.StepByStep` |
| Content review with email reminders (LGR-ready) | `LocalGov.Umbraco.ContentReview` |

---

## Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8)
- The [LocalGov.Umbraco](https://github.com/Brighty28/Local-Gov-Umbraco) packages repo cloned **as a sibling** of this repo:
  ```
  C:\Dev\
  ├── Greenfield-District-Council\   ← this repo
  └── LocalGov.Umbraco\              ← packages repo
  ```

---

## Quick start

### 1 — Pack the LocalGov.Umbraco packages

```bash
cd ../LocalGov.Umbraco
dotnet build --configuration Release
dotnet pack src/LocalGov.Umbraco.AlertBanner --configuration Release --output ./nupkgs --no-build -p:Version=1.0.0
dotnet pack src/LocalGov.Umbraco.ContentReview --configuration Release --output ./nupkgs --no-build -p:Version=1.0.0
dotnet pack src/LocalGov.Umbraco.Core --configuration Release --output ./nupkgs --no-build -p:Version=1.0.0
dotnet pack src/LocalGov.Umbraco.Guides --configuration Release --output ./nupkgs --no-build -p:Version=1.0.0
dotnet pack src/LocalGov.Umbraco.News --configuration Release --output ./nupkgs --no-build -p:Version=1.0.0
dotnet pack src/LocalGov.Umbraco.Services --configuration Release --output ./nupkgs --no-build -p:Version=1.0.0
dotnet pack src/LocalGov.Umbraco.StepByStep --configuration Release --output ./nupkgs --no-build -p:Version=1.0.0
dotnet pack src/LocalGov.Umbraco.Theme --configuration Release --output ./nupkgs --no-build -p:Version=1.0.0
dotnet pack meta/LocalGov.Umbraco --configuration Release --output ./nupkgs -p:Version=1.0.0
```

### 2 — Run the demo site

```bash
cd ../Greenfield-District-Council
dotnet run --project src/GreenfieldDC.WebUI
```

Browse to **https://localhost:44332** (or the port shown in the terminal).

### 3 — Complete the Umbraco installer

1. Create your admin account
2. **Skip** the starter kit — the LocalGov content types install automatically

### 4 — Create the content tree

In the Umbraco back-office, create the following root nodes:

| Name | Document type | Notes |
|---|---|---|
| Greenfield District Council | `lgHome` | Root — your home page |
| Site Settings | `lgSettings` | Set site name, primary colour (`#1d70b8`), logo |
| Alert Banners | `lgAlertList` | Container for alert banner items |
| News | `lgNewsList` | Under lgHome or as root |

Under **lgHome**, add:
- One or more `lgServiceLanding` pages (e.g. "Bins and recycling")
  - Add `lgServicePage` children for individual topics
- An `lgGuide` page (e.g. "How to pay your council tax")
  - Add `lgGuidePage` children for each section

---

## Content types reference

All content types are prefixed `lg` and installed automatically on first startup via uSync.

| Alias | Description |
|---|---|
| `lgHome` | Root home page |
| `lgSettings` | Site-wide config (name, logo, brand colours) |
| `lgAlertList` | Container for alert banners |
| `lgAlertItem` | Single alert (severity, schedule, page scope) |
| `lgNewsList` | Newsroom container (list view) |
| `lgNewsItem` | Individual news article |
| `lgServiceLanding` | Top-level service area |
| `lgServiceSubLanding` | Service sub-area |
| `lgServicePage` | Leaf service content page |
| `lgGuide` | Multi-page guide hub |
| `lgGuidePage` | Single section of a guide |
| `lgStepByStep` | GOV.UK step-by-step journey |
| `lgStep` | Single step within a step-by-step |

---

## Production use

Once `LocalGov.Umbraco` packages are published to NuGet, replace the local `nuget.config` entry with the standard NuGet feed and update your `PackageReference` versions accordingly.

For production, swap SQLite for SQL Server by replacing `Umbraco.Cms.Persistence.Sqlite` with `Umbraco.Cms.Persistence.SqlServer` and updating the connection string.

---

## Licence

MIT — same as [LocalGov Umbraco](https://github.com/Brighty28/Local-Gov-Umbraco/blob/main/LICENSE)
