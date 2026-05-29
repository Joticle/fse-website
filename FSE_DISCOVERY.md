# DISCOVERY_REPORT — FlowState Engineering Website

## Solution Overview

Single-project ASP.NET Core Razor Pages solution serving as the public marketing
site for FlowState Engineering (fstate.dev). No database at runtime, no user
auth, no server-side state — every page is a Razor view rendered from static
content plus shared layout.

## Solution Structure

```
FlowState.sln
└── FlowState.csproj                 (Microsoft.NET.Sdk.Web, net10.0)
    ├── Program.cs                   (Razor Pages host + Key Vault wiring)
    ├── appsettings.json             (Key Vault URL only)
    ├── appsettings.Development.json (LocalDB conn string — historical, unused)
    ├── Pages/
    │   ├── _ViewImports.cshtml
    │   ├── _ViewStart.cshtml
    │   ├── Error.cshtml
    │   ├── Index.cshtml             (Home)
    │   ├── HowItWorks.cshtml
    │   ├── Tenets.cshtml
    │   ├── Results.cshtml
    │   ├── Legacy.cshtml
    │   ├── About.cshtml
    │   ├── Privacy.cshtml
    │   └── Shared/
    │       ├── _Layout.cshtml
    │       └── _ValidationScriptsPartial.cshtml
    ├── Properties/
    │   └── PublishProfiles/
    └── wwwroot/
        ├── css/
        │   ├── fse-variables.css
        │   ├── fse-base.css
        │   ├── fse-layout.css
        │   └── fse-components.css
        ├── js/
        ├── images/
        └── lib/
```

## Target Framework

`net10.0` — matches the FSE standard at the time of this rebuild.

## Dependencies

| Package | Version | Purpose |
|---|---|---|
| Azure.Extensions.AspNetCore.Configuration.Secrets | 1.5.0 | Key Vault configuration provider |
| Azure.Identity | 1.21.0 | `DefaultAzureCredential` for Key Vault auth |

No EF Core, no database provider, no auth framework — this is intentional.

## Pages Inventory

Public pages:

| Route | File | Purpose |
|---|---|---|
| `/` | `Index.cshtml` | Home / entry point |
| `/HowItWorks` | `HowItWorks.cshtml` | FSE methodology explained |
| `/Tenets` | `Tenets.cshtml` | The 10 FSE tenets, philosophy + practice |
| `/Results` | `Results.cshtml` | Track record and outcomes |
| `/Legacy` | `Legacy.cshtml` | Original October 2025 manifesto, preserved |
| `/About` | `About.cshtml` | Origin story, creator, company |
| `/Privacy` | `Privacy.cshtml` | Standard privacy notice |
| `/Error` | `Error.cshtml` | Framework error page |

Shared:

- `Pages/Shared/_Layout.cshtml` — sticky top nav, footer, CSS load order.
- `Pages/Shared/_ValidationScriptsPartial.cshtml` — scaffolded, unused at runtime.

## Data / Persistence

A SQL Server database exists on SmarterASP.NET
(`SQL8006.site4now.net` / `db_a64293_fstate`) and its connection string is
wired into `appsettings.json` under the `ConnectionStrings:FState` key. The password in
the committed file is the literal placeholder `YOUR_DB_PASSWORD` — the real
password is supplied at deploy time (host env var or Key Vault).

The database schema is currently empty and no page in the site reads from it.
`Program.cs` resolves the connection string (LocalDB in Development,
`ConnectionStrings:FState` otherwise) but the result is not consumed by anything
downstream. See `SESSION_STATE.md` &rarr; Known Technical Debt for the decision
point: use the DB or remove it.

## Configuration

- `appsettings.json` — Azure Key Vault URL (`https://fstate-keyvault.vault.azure.net/`).
- `appsettings.Development.json` — local LocalDB string (unused).
- `Program.cs` — wires Key Vault in non-Development environments using
  `DefaultAzureCredential`.

## Hosting

SmarterASP.NET. Publish target is IIS on Windows with .NET 10 runtime.
Publish profile lives under `Properties/PublishProfiles/`.

## CSS Architecture

All styling is token-driven via the `--fse-*` custom property namespace,
split across four files loaded in order:

1. `fse-variables.css` — color tokens, spacing scale, radius scale, shadow
   scale, transitions, font stacks.
2. `fse-base.css` — reset, base typography, link defaults, utility classes.
3. `fse-layout.css` — nav, container, section rhythm, footer, media queries.
4. `fse-components.css` — hero, cards, tenet grid, stats, callouts, buttons,
   badges, timeline.

No file uses hardcoded color values. No page uses inline `style` attributes.
All media queries in `.cshtml` files escape the `@` as `@@media`.

## Build Baseline

`dotnet build FlowState.sln` — 0 errors, 0 warnings.
