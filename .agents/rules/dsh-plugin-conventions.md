---
trigger: model_decision
description: "Apply when editing lib/index.js, cordis.patch.yml, dsh.plugin.json, package.json dsh/compatibility metadata, or anything that affects how DSH loads this plugin."
---

# DSH plugin conventions

Facts verified against the installed DSH distribution. Treat as constraints, not suggestions.

## Provider contract

`lib/index.js` registers one provider into the host `ctx.skills` registry. The contract
comes from `@deepseek-ai/dsh-skill`:

| Field | Value here | Why |
|---|---|---|
| provider `name` | `dsh-ecc` | Registry key; duplicates throw at registration |
| `inject` | `['skills']` | Hard dependency on the skills service |
| candidate `rank` | `550` | Lower wins. DSH bundled root is `600`, so project/user skills override these |
| candidate `source` | `custom` | Prompt-visible bucket label, not precedence |
| `invocation` | mapped from `disable-model-invocation` | Upstream `true` → `{modelInvocable:false, userInvocable:true}` |

`SkillCandidate` requires `rank` (number), `locator` (opaque handle passed back to `get()`),
and optional `metadata`. `SkillDefinition` returned by `get()` requires `content` plus the
summary fields. `resourceBase` is `{kind:'directory', path}` — relative skill references
resolve against it.

## Version compatibility metadata

`package.json` → `dsh.compatibility.dshReleases` is a declared map of DSH release → status.
**It is a claim, not a check.** Currently it lists the `0.1.x` line through
`0.1.5-rc.2` and **omits `0.2.0-rc.2`** (and `0.2.0-rc.1`).

When adding a release: only mark `compatible` after verifying the API surface still matches
(the four items above). The provider contract has been stable across `0.1.x` → `0.2.0-rc.2`,
but that was checked by reading the installed types, not by running the plugin inside DSH.
**Do not mark a release compatible on the basis of a version-number guess.**

`engines.dsh` (`>=0.1.0-rc.6`) is a semver range, separate from the compatibility map.

## Install and removal

```powershell
dsh plugin --profile web add dsh-ecc-skills   # published package
dsh plugin --profile web add github:gongyijie85/dsh-ecc
```

Plugin packaging is declared by `dsh.bundle.patch` → `cordis.patch.yml`, which contains a single
`insert` row. `dsh.plugin.json` holds the manifest.

## Runtime behaviour to preserve

- Read-only. No writes, no network, no child processes.
- Bodies load on demand (`get()` re-reads the file), so editing a skill needs no cache bust.
- Missing or unreadable `SKILL.md` returns `undefined` — never throws.
- `list()` returns `[]` if the skills directory is missing.

Any change that adds filesystem writes, network access, or command execution to `lib/index.js`
contradicts the security posture documented in `AGENTS.md` and should be rejected outright.
