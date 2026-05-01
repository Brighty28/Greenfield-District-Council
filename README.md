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
- Access to the **3CDigital Azure Artifacts feed** at
  `https://pkgs.dev.azure.com/3CDigital/Digital Project/_packaging/3CDigital/nuget/v3/index.json`
  (configured in this repo's `nuget.config`)

### Authenticating with Azure Artifacts

Before your first restore, authenticate against the feed using one of:

**Visual Studio** — sign in to the same Microsoft account that has access to the
3CDigital DevOps organisation. NuGet picks up the credential automatically.

**dotnet CLI** — create a Personal Access Token with `Packaging > Read` scope at
<https://dev.azure.com/3CDigital/_usersSettings/tokens>, then run:

```bash
dotnet nuget add source "https://pkgs.dev.azure.com/3CDigital/Digital Project/_packaging/3CDigital/nuget/v3/index.json" \
    --name 3cdigital \
    --username <your-email> \
    --password <PAT> \
    --store-password-in-clear-text
```

**Azure Artifacts Credential Provider** (recommended for headless dev) —
follow <https://github.com/microsoft/artifacts-credprovider#installation> to
install the cross-platform credential provider; it handles auth flows for you.

---

## Quick start

```bash
cd Greenfield-District-Council
dotnet restore
dotnet run --project src/GreenfieldDC.WebUI
```

The browser auto-opens at **https://localhost:44332** (configurable in
`src/GreenfieldDC.WebUI/Properties/launchSettings.json`).

### 1 — Complete the Umbraco installer

1. Create your admin account
2. **Skip** the starter kit — the LocalGov content types and templates install
   automatically via uSync

### 2 — Verify the structure imported

In the back-office:
- **Settings → Document Types** — should show `lgHome`, `lgSettings`, `lgServiceLanding` etc.
- **Settings → Templates** — should show `LG Home`, `LG Service Landing`, `LG News List` etc.
- **Settings → uSync** — if anything is missing, click **Import** to retry.

### 3 — Create the content tree

Create the following root nodes:

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

For production, swap SQLite for SQL Server by replacing
`Umbraco.Cms.Persistence.Sqlite` with `Umbraco.Cms.Persistence.SqlServer` and
updating the connection string in `appsettings.json`.

---

## Licence

MIT — same as [LocalGov Umbraco](https://github.com/Brighty28/Local-Gov-Umbraco/blob/main/LICENSE)
