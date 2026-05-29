<!-- ============================================================ -->
<!-- FSE:START — FlowState Engineering Universal Operating Context -->
<!-- ============================================================ -->

# FlowState Engineering — Operating Context

This file is the operating context for any Claude agent working in this solution.
It is read at the start of every session. Everything inside the FSE START/END
markers is universal across all FSE solutions. Everything after FSE:END is
project-specific.

---

## Session Protocol

Every session follows four phases, in order. No phase is skipped.

### 1. VERIFY
- Read `CLAUDE.md`, `SESSION_STATE.md`, and `DISCOVERY_REPORT.md` in full.
- Confirm the working directory, solution file, and target framework.
- Run `dotnet build` on the full solution. Record error count and warning count.
- Report the baseline to the user before proposing any change.

### 2. PLAN
- Restate the user's request in one sentence.
- Identify every file that will be created, modified, or deleted.
- Surface conflicts with existing state, standing orders, or known technical debt.
- If the plan would touch more than five files or introduce a new dependency,
  pause for user confirmation before execution.

### 3. EXECUTE
- Write complete files only. No stubs, no placeholders, no TODOs.
- One file at a time. Each file is production-ready when written.
- After each meaningful change, run the Self-Healing Build Loop.

### 4. VALIDATE
- Final `dotnet build` must report 0 errors and 0 warnings.
- Manually trace every new route, view, and binding path.
- Update `SESSION_STATE.md` with completed work, lessons learned, and next priorities.
- Execute the Session End Protocol.

---

## Self-Healing Build Loop

After every change that could affect compilation:

1. Run `dotnet build` on the full solution.
2. If errors or warnings exist, read the first error, form a hypothesis, fix it.
3. Rebuild.
4. Maximum three retries on the same error class. On the third failure, STOP
   and invoke the Counter-Point Protocol.

Zero errors and zero warnings is the only acceptable final state.

---

## Universal Standing Orders

These rules apply to every FSE solution, unconditionally.

1. **Complete files only.** Never emit a stub, a placeholder, or a TODO comment.
2. **Zero warnings is the floor.** Warnings are errors that haven't happened yet.
3. **CSS tokens only.** All styling uses solution-scoped `--*-*` custom properties.
   No hardcoded colors, no inline `style` attributes.
4. **Razor escape.** Use `@@media` (not `@media`) in every `.cshtml` file.
5. **Read before edit.** Never modify a file without reading its current state.
6. **No speculative abstraction.** Build for what is asked; do not add hooks,
   flags, or generic layers for hypothetical future needs.
7. **No backwards-compat cruft.** Delete unused code cleanly; do not leave
   tombstone comments or renamed `_unused` vars.
8. **State file is truth.** `SESSION_STATE.md` is the authoritative backlog.
   There is no separate ticketing system to reconcile.
9. **Secrets never commit.** Connection strings, keys, and tokens live in Key
   Vault or environment-scoped config only.
10. **Git is append-only by default.** No force-push, no `reset --hard`, no
    `--no-verify` unless the user explicitly authorizes the specific command.
11. **Destructive actions require explicit consent.** Deleting files, dropping
    tables, killing processes — confirm scope before acting.
12. **Report at milestones, not at every step.** One consolidated report at the
    end of a task beats twelve interruptions.

---

## Counter-Point Protocol

When the Self-Healing Build Loop hits its third retry on the same error, or when
a user instruction conflicts with a standing order, the agent must:

1. STOP executing.
2. State the conflict plainly: what was attempted, why it failed, what the
   conflicting rule says.
3. Offer two or three concrete alternative approaches.
4. Wait for user selection. Do not guess.

Counter-Point exists to prevent the agent from drilling deeper into a bad path.
Friction here is a feature.

---

## Session End Protocol

Before the agent reports "done":

1. `dotnet build` → 0 / 0.
2. `SESSION_STATE.md` updated:
   - Session History entry added with date and scope.
   - Lessons Learned appended if any new patterns or pitfalls emerged.
   - Known Technical Debt updated if new debt was taken on or old debt retired.
   - Next Session Priorities rewritten to reflect the new state.
3. File inventory reported to the user: every NEW / UPDATED / DELETED path.
4. If a git operation is in scope, commit with a descriptive message and
   report the resulting commit hash.

---

## Document Ecosystem

FSE uses a three-tier documentation model. Documents grow organically and
split when they exceed 15 KB.

### Tier 1 — Operating Context (always loaded)
- `CLAUDE.md` — this file. Universal protocol + project identity.
- `SESSION_STATE.md` — living state of the solution.

### Tier 2 — Reference (loaded on demand)
- `DISCOVERY_REPORT.md` — solution structure, dependencies, architecture snapshot.
- `CLAUDE_POLICE.md` — created the first time a standing order is violated.
  Records the incident, the correction, and the guard rule.

### Tier 3 — Historical (loaded rarely)
- `SESSION_HISTORY_*.md` — split from `SESSION_STATE.md` when the history section
  exceeds 15 KB.
- `LESSONS_*.md` — split from `SESSION_STATE.md` when lessons learned exceeds 15 KB.

### Growth Rule
A document starts unified. When it crosses 15 KB, split the oldest or most
self-contained section into a Tier 3 file and leave a pointer in the origin.

<!-- ========================================================== -->
<!-- FSE:END — Project-Specific Context Begins Below -->
<!-- ========================================================== -->

## Product Identity

- **Name:** FlowState Engineering
- **Domain:** fstate.dev
- **Company:** Joticle, Inc.
- **Stack:** .NET 10 Razor Pages, static marketing site
- **Hosting:** SmarterASP.NET
- **CSS Token Prefix:** `--fse-*`

## Build State

- **Baseline:** 0 errors / 0 warnings on `dotnet build FlowState.sln`
- **Target Framework:** `net10.0`
- **Packages:**
  - Azure.Extensions.AspNetCore.Configuration.Secrets 1.5.0
  - Azure.Identity 1.21.0

## Project Notes

- No primary database. Connection string present in `appsettings.Development.json`
  for historical reasons; the site itself is static and reads no data at runtime.
- Azure Key Vault is wired in `Program.cs` for non-development environments
  (URL: `https://fstate-keyvault.vault.azure.net/`). The `ConnectionStrings:FState` secret
  is read but currently unused by any page.
- All pages are Razor Pages under `Pages/`. Shared layout in `Pages/Shared/_Layout.cshtml`.
- All CSS lives under `wwwroot/css/` and uses the `--fse-*` token prefix exclusively.
