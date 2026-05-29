# SESSION_STATE — FlowState Engineering Website

## Identity

- **Solution:** FlowState Engineering marketing site (fstate.dev)
- **Company:** Joticle, Inc.
- **Creator:** Scott Michael Wilson
- **Project Origin:** October 2025
- **Stack:** .NET 10 Razor Pages, SmarterASP.NET hosting
- **CSS Token Prefix:** `--fse-*`

## Build State

- **Current:** 0 errors / 0 warnings
- **Target Framework:** `net10.0`
- **Solution File:** `FlowState.sln`
- **Project File:** `FlowState.csproj`

## Lessons Learned

1. **`DefaultAzureCredential` crashes non-Azure hosts at startup.**
   `builder.Configuration.AddAzureKeyVault(..., new DefaultAzureCredential())`
   throws during app startup on SmarterASP.NET (no managed identity, no env
   credentials). The ANCM wrapper reports this as a generic 502.5
   startup failure, not as the real exception — wildly misleading.
   **How to apply:** never wire Key Vault unconditionally on non-Azure hosts.
   Either remove it, or guard with an env-var / config flag that only enables
   it in Azure.

2. **SmarterASP.NET shared pools reject `hostingModel="inprocess"`.**
   Other apps in the shared app pool already run out-of-process, and IIS
   returns `HTTP 500.34 — mixing hosting models is not supported`.
   **How to apply:** for SmarterASP.NET, set `hostingModel="outofprocess"`
   in `web.config`. This is now the default in the committed `web.config`.

3. **Temp URLs on SmarterASP.NET don't serve HTTPS.**
   `ltempurl.com` slots don't have TLS certs — `UseHttpsRedirection()` sends
   clients to an HTTPS endpoint that resets the connection.
   **How to apply:** don't call `UseHttpsRedirection()` until a real domain
   (`fstate.dev`) is bound with a cert. Re-add it in `Program.cs` after
   DNS + cert are in place.

4. **Diagnostic unlock: ANCM 502.5 can hide a dozen root causes.**
   Setting `stdoutLogEnabled="true"` + `ASPNETCORE_DETAILEDERRORS=true`
   in `web.config` surfaced a 500.34 that was otherwise invisible.
   **How to apply:** when a deploy returns 502.5 with no log detail,
   flip stdout logging on first and redeploy before changing anything else.

## Known Technical Debt

1. **Deploy password in VS user profile (plaintext).**
   Pre-launch site; nobody knows it exists yet. Stack-later item. Before the
   site goes public, rotate the SmarterASP.NET Web Deploy password and move
   it out of `.pubxml.user` into an env var consumed by `dotnet publish`.
2. **DB password committed in `appsettings.json` (plaintext) — RESOLVED 2026-05-28 (forward), HISTORY OUTSTANDING.**
   The key originally named `FState-SQLConn` carried the real SmarterASP.NET
   SQL password in the working tree and was committed in initial commit
   `8a46d0b` (2026-04-29) to the public GitHub repo `Joticle/fse-website`.
   The earlier note that "the repo is not yet a git repository" was incorrect
   — the repo had already been pushed publicly when that note was written.
   **Remediation (2026-05-28):** password rotated externally on the
   SmarterASP DB; `appsettings.json` removed from version control
   (`git rm --cached` + added to `.gitignore`); config key renamed from
   `FState-SQLConn` to `ConnectionStrings:FState` so the rotated password is
   delivered to the host via the env-var override `ConnectionStrings__FState`
   (set in the SmarterASP control panel, NOT in `web.config`); local
   working-tree `appsettings.json` scrubbed — the key remains with an empty
   string value as a structure placeholder.
   **Outstanding:** the old (rotated-out) password is still present in the
   git history at commit `8a46d0b` on the public GitHub remote. Removing it
   requires rewriting history (`git filter-repo` or BFG) and force-pushing
   `main`. Tracked as a separate follow-up — "part 2".
3. **Key Vault wiring removed entirely.**
   Azure.Identity and Azure.Extensions.AspNetCore.Configuration.Secrets are
   gone from the csproj. `Program.cs` no longer calls `AddAzureKeyVault`.
   `appsettings.json` no longer carries a `KeyVault` section. If the site
   ever needs secret retrieval from a vault, re-add it conditionally on an
   env-var flag so non-Azure hosts skip the init.

4. **`UseHttpsRedirection()` removed until cert is in place.**
   Currently not called in `Program.cs`. Re-add it once `fstate.dev` is bound
   with a TLS cert on the SmarterASP.NET slot. Leaving it out on the temp
   URL avoids the HTTPS connection-reset loop.
4. **No page currently reads the database.**
   The connection string is wired but unused at runtime. If the site remains
   fully static, consider removing the DB wiring entirely.

## Next Session Priorities

1. Verify deployment to SmarterASP.NET succeeds on .NET 10 via the existing
   `joticle-001-site10 - Web Deploy` profile.
2. Confirm Google Fonts loads correctly in production (hosting may have CSP
   or proxy constraints).
3. Decide on DB strategy: use it (fill password, build features) or remove
   it (simplify to static-only).
4. Before public launch: rotate deploy password and move secrets out of
   committed files.

## Session History

### Session 1 — FSE onboarding + full site rebuild
- Onboarded the FlowState Engineering website to the FSE operating model.
- Created `CLAUDE.md`, `SESSION_STATE.md`, `DISCOVERY_REPORT.md`.
- Upgraded `FlowState.csproj` from `net9.0` to `net10.0`.
- Updated `Azure.Extensions.AspNetCore.Configuration.Secrets` to 1.5.0.
- Updated `Azure.Identity` to 1.21.0.
- Fixed duplicate `@page` / `@model` directives in `TenetThree.cshtml` (then deleted).
- Built a new design system under `wwwroot/css/` (variables, base, layout, components)
  using the `--fse-*` token prefix exclusively.
- Rebuilt `_Layout.cshtml` with sticky top nav and mobile hamburger.
- Rewrote `Index.cshtml` as the new homepage.
- Created `HowItWorks.cshtml`, `Tenets.cshtml`, `Results.cshtml`, `Legacy.cshtml`.
- Rewrote `About.cshtml`.
- Deleted legacy pages: `Pages/Tenants/` (folder), `Manifesto.cshtml`, `Principles.cshtml`,
  `Resources.cshtml`, `Learn.cshtml`.
- Final build: 0 errors / 0 warnings.
