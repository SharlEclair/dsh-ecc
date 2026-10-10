# Baseline — verified 2026

Recorded before any adaptation, so regressions are detectable. Everything here was executed
on this machine, not copied from documentation.

## Environment

| Item | Value |
|---|---|
| Workspace | `C:\Users\91704\Desktop\dsh-ecc` |
| Remote | `https://github.com/gongyijie85/dsh-ecc.git` (upstream — see fork step below) |
| Branch | `main` |
| Commit | `e73abf7` — `chore(release): v0.6.0 - rename design-system to frontend-design-system` |
| Repo version | `0.6.0` |
| `origin/main` | `e73abf7`, also version `0.6.0` — the clone is current with the repository |
| Latest git tag | `v0.6.0` — **there is no `v0.6.1` tag** |
| Published npm version | `0.6.1` (npm registry) |
| Node | `v24.15.0` |
| Files in checkout | 291 |

## Baseline test result

```powershell
Set-Location C:\Users\91704\Desktop\dsh-ecc
node scripts/verify-provider.mjs
```

Result at that commit:

```
OK: all 274 ecc skills discovered and get() passes
```

All 274 candidates were listed, `get()` returned content and a `resourceBase` for each, and no
description started with `>`.

> **Historical snapshot.** The figures immediately above record the untouched clone at commit
> `e73abf7`, *before* any adaptation. They are the reference point, not the current state.
>
> *2026-10-11 — `configure-ecc` was retired (an upstream setup wizard for Claude Code / Codex /
> Kimi that has no DSH meaning), so the expected count and the passing message are now **273**.
> The 274 above stays as the recorded baseline.*

The current equivalent of the check above must still pass after every change.

## Installed DSH (the target harness)

| Item | Value |
|---|---|
| Version | `0.2.0-rc.2` |
| Install | `C:\Users\91704\AppData\Roaming\npm\node_modules\@deepseek-ai\dsh\` |
| Profile | `web` |
| `@deepseek-ai/cordis` | `4.0.4` (satisfies the plugin's `^4.0.1` peer range) |

## Known discrepancies to fix

1. **`dsh.compatibility.dshReleases` does not list `0.2.0-rc.2`.**
   `package.json` L53–L76 covers the `0.1.x` line only. The provider contract was inspected
   against installed 0.2.0-rc.2 types (`ctx.skills.registerProvider`, `SkillCandidate`,
   `SkillInvocationPolicy`, `SkillSource`) and matches — but the plugin has **not** been run
   inside a live DSH session, so this stays a documented gap until it is.
2. **Stale star count.** `package.json` description says "227k-star"; upstream
   [affaan-m/ECC](https://github.com/affaan-m/ECC) reports **276,389** stars.
3. **`files` whitelist excludes `scripts/`** — so `verify-provider.mjs` is not in the published
   tarball even though `prepublishOnly` runs it. Intentional or not, do not assume a published
   artifact can self-verify.

## Not yet done

- **No fork created.** `origin` still points at the upstream repo. See
  [FORK-AND-PR.md](FORK-AND-PR.md).
- **`gh` not authenticated** (`gh` 2.102.0 installed). Run `gh auth login` once.
- Plugin is **not** installed into the live DSH profile, so runtime behaviour inside DSH's
  Web UI is unverified.

## Version note — corrected

An earlier claim that this clone was "one behind" npm was imprecise. What is actually true:

- The **GitHub repository** is at `0.6.0` on `main`, tagged `v0.6.0`. This clone matches it exactly.
- **npm** carries `0.6.1`, a version with **no corresponding git tag**.

So npm is ahead of the repository, not the clone behind the repository. That is mildly irregular
for a project whose release flow is tag-driven, and it means `0.6.1`'s contents are not reviewable
in git. If you ever need exactly what npm ships, diff the published tarball against this checkout
rather than assuming the tags describe it.
