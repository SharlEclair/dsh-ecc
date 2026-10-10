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
| Published npm version | `0.6.1` ← **one ahead of this clone** |
| Node | `v24.15.0` |
| Files in checkout | 291 |

## Baseline test result

```powershell
Set-Location C:\Users\91704\Desktop\dsh-ecc
node scripts/verify-provider.mjs
```

Result:

```
OK: all 274 ecc skills discovered and get() passes
```

All 274 candidates were listed, `get()` returned content and a `resourceBase` for each, and no
description started with `>`. **This is the known-good state. It must still pass after every change.**

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
