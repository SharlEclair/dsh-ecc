# DSH Adaptation Plan

Turning the 274 ported ECC skills from Claude-Code-shaped into DSH-native.

**Baseline verified** — `node scripts/verify-provider.mjs` → `OK: all 273 ecc skills discovered and get() passes` (Node v24.15.0).

> **Where the evidence lives.** Every Tier-C change is backed by a specific verification, recorded
> in [Evidence and decision trail](#evidence-and-decision-trail) below. Decisions and their
> provenance are in [SESSION-NOTES.md](SESSION-NOTES.md). Do not re-open a settled decision
> without new evidence — cite the ledger entry instead.

## Why this work exists

The upstream port copied skill bodies almost verbatim from [affaan-m/ECC](https://github.com/affaan-m/ECC).
Its own README states the only adaptation was two command references
(`/bug-check` → `bug-check`, `/prompt-optimize` → `prompt-optimizer`).

Measured on this clone:

| Signal | Files / 273 |
|---|---|
| mention `claude` | 80 (29%) |
| reference `.claude/` paths | 27 (10%) |
| reference `mcp.json` / `mcpServers` | 13 (5%) |
| reference Claude hook names | 11 (4%) |
| `Task(` / `subagent_type` | 5 (2%) |
| reference the `ecc` CLI | 3 (1%) |
| **mention DSH at all** | **5 (2%)** |

So ~98% of the content has never been checked against the harness it now runs on. That is the
entire scope of this project.

## What is *not* in scope

The port deliberately ships **no** agents, hooks, commands, or learning runtime — upstream ECC's
71 agents, 95 commands, hook runtime, and AgentShield scanning are absent. That is a design
boundary, not a bug to fix here. Building a DSH bridge for hooks/agents would be a separate,
much larger project; do not start it inside a skill-cleanup branch.

## Triage

### Tier C — entirely broken (do these first)

References the `ecc` / `ecc-universal` / `ecc-agentshield` CLI that is not installed here.
Exact offending lines, verified on this clone:

| Skill | Line | Problem |
|---|---|---|
| `configure-ecc` | L16 | `npx ecc-universal setup` — a Claude Code setup wizard, wrong harness |
| ~~`security-scan`~~ | ~~L3~~ | ~~**frontmatter `description` advertises scanning `.claude/`**~~ — **FIXED**, see below |
| `security-scan` | L15, L26, L49 | `.claude/settings.json`, `CLAUDE.md`, `mcp.json`, "current project's `.claude/` directory" |
| `security-scan` | L36, L42 | `npx ecc-agentshield --version` / `scan .` — not installed |
| `unified-memory` | L28 | `node scripts/ecc.js memory ...` — script does not exist in this repo |
| `unified-memory` | L146 | references ECC's default `.mcp.json` |

`security-scan` is the highest-value first fix: a broken description is visible to every session
that lists the skill catalog, whereas a broken body line only misleads once loaded.

### ✔ `security-scan` — description fixed

The frontmatter `description` no longer claims to scan `.claude/` for DSH. It now describes the
skill accurately: a usage manual for the **external** `ecc-agentshield` package, whose target is
Claude Code configuration, with **DSH coverage explicitly recorded as not established**.

Verified facts behind that wording:

| Claim | Evidence |
|---|---|
| `ecc-agentshield` is real and maintained | npm 1.6.0, MIT, published 2026 |
| It targets Claude Code, not DSH | its own description: "Scans Claude Code setups" |
| It is not installed on this machine | absent from `%APPDATA%\npm\node_modules` |
| `--opus` bills Anthropic | depends on `@anthropic-ai/sdk`; needs `ANTHROPIC_API_KEY` |
| **DSH ships no config scanner** | installed `@deepseek-ai` tree has sandbox/approval/permission plugins only |

A "DSH security surface" section was added to the skill body mapping the real DSH controls
(sandbox ladder, approval policy, permission presets) and recording the two easy-to-over-trust
limitations: sandbox modes govern **file effects only** (network is unrestricted), and escalation
is strictly wider — there is no path back to a narrower mode.

**Still open in this file:** the body's Claude-specific remediation guidance (L15, L26, L36, L42,
L49 region) has not been rewritten. Whether AgentShield can scan DSH config at all is unverified —
do not invent a mapping.

### Tier A — broken assumptions (30 files)

Reference `.claude/` paths, `hooks.json`, `PreToolUse`/`PostToolUse`, or `subagent_type`.

```
agent-sort                 autonomous-loops           ck
agentic-os                 blueprint                  config-gc
ai-regression-testing      canary-watch               cost-tracking
autonomous-agent-harness   council                    delivery-gate
dmux-workflows             eval-harness               hookify-rules
inherit-legacy-style       knowledge-ops              opensource-pipeline
plan-canvas                plankton-code-quality      rules-distill
safety-guard               search-first               security-scan
skill-scout                skill-stocktake            tdd-workflow
team-builder               verification-loop          workspace-surface-audit
```

`security-scan` appears in both Tier A and Tier C — do it as part of Tier C.

### Tier B — MCP config that DSH does not read (13 files)

```
agent-payment-x402   agentic-os            autonomous-agent-harness
codehealth-mcp       config-gc             context-budget
ito-compute          jira-integration      laravel-plugin-discovery
nutrient-document-processing                security-scan
unified-memory       workspace-surface-audit
```

Tier A and B overlap (`agentic-os`, `config-gc`, `security-scan`, `unified-memory`,
`autonomous-agent-harness`, `workspace-surface-audit`). Handle overlaps once, in the earlier tier.

**Estimated distinct files to touch: ~40 of 273.** The other ~234 are generic engineering
guidance that happens to be harness-neutral; leave them alone.

## Recommended first batch

Tier C — three files, fully broken, small enough to finish and verify in one sitting.
Do them in this order:

1. **`security-scan/SKILL.md`** — fix the frontmatter `description` first (it advertises
   `.claude/` to the whole catalog), then the body's broken paths and CLI calls.
2. `unified-memory/SKILL.md` — remove the `scripts/ecc.js` and `.mcp.json` dependencies, or
   mark the skill as requiring ECC infrastructure it does not have here.
3. `configure-ecc/SKILL.md` — this one configures *ECC for Claude Code*; it may not have a
   DSH meaning at all. Decide between: rewrite to describe this plugin's own install, or
   **retire it** (see the retire rule). Say which and why in the commit.

## Guardrails

- `npm run verify` before every commit; it must stay at `OK: all 273`.
- One skill per commit: `fix(<skill-name>): <what changed>`.
- Read [.agents/rules/skill-adaptation.md](../.agents/rules/skill-adaptation.md) before editing —
  especially the "retire instead of fake" rule.
- Do **not** bulk find-and-replace across `skills/`. A blind replace of `.claude/` → something
  will silently corrupt at least some of the 27 files that reference it.

## Open questions to resolve before large-scale work

1. **Is the plugin even the right delivery mechanism?** DSH already discovers skills natively
   from `.dsh/skills`, `.agents/skills`, and `~/.agents/skills`. Mounting this plugin puts all
   273 into every session's catalog. For your own use, copying selected skills may be better
   than shipping 273. Decide this before polishing all 40 files.
2. **Which skills do you actually want?** Adapting 40 files nobody loads is waste. Pick the
   subset first, then adapt only that subset.
3. **Upstream sync.** Upstream GitHub `main` is `0.6.0` (this clone matches it); npm carries
   `0.6.1` with no matching git tag. Decide whether you track upstream or diverge, because
   divergence makes future rebases harder.

## Status

| Tier | Files | Done |
|---|---|---|
| C | 3 | **3 — complete** |
| A | 30 | 0 |
| B | 13 | 0 |
| **distinct** | **~40** | **3** |

Tier C landed as three commits, each independently green:

| Commit | Skill | Outcome |
|---|---|---|
| `8ee1bc4` | `security-scan` | description stopped advertising `.claude/`; real DSH security surface documented |
| `e4d06d1` | `configure-ecc` | **retired** — upstream setup wizard for Claude Code / Codex / Kimi, no DSH meaning |
| `ea65171` | `unified-memory` | broken `scripts/ecc.js` clause removed; `.mcp.json` replaced with a `dsh-mcp-client` row; `ECC_MEMORY_HARNESS=dsh` verified valid |

`configure-ecc` retirement moved the active skill count 274 → 273. The verifier computes
its count from the expected Set, so it self-updated; the port-coverage metric in the READMEs
stays at 274/286 (retiring a skill locally does not un-port it).

Update this table as work lands. Do not mark a tier done without a green `npm run verify`.

## Before starting Tier A — read this

Tier A is not the same kind of work as Tier C. Tier C was three files with unambiguous
defects. Tier A's 30 files split into two very different groups:

**Adaptation candidates** — the skill is useful to a DSH user and just needs harness-correct
wording: `search-first`, `safety-guard`, `eval-harness`, `cost-tracking`.

**Retirement candidates** — the skill is ECC-internal bookkeeping that only functions alongside
ECC's hook/agent runtime, which this port deliberately does not ship: `agent-sort`, `config-gc`,
`hookify-rules`, `rules-distill`, `skill-stocktake`, `skill-scout`, `plankton-code-quality`,
`inherit-legacy-style`.

Retiring 8–15 skills is a far larger ask of a maintainer than fixing three. Decide the PR
strategy first:

- **If the goal is a merged PR** — stop after Tier C and open it. Three focused, well-reasoned
  commits are reviewable; a fifteen-skill deletion is a redesign of the package.
- **If the goal is your own fork** — Tier A retirements are fine, and arguably should happen.
  But then the upstream-sync question matters much more.

## Evidence and decision trail

Every Tier-C change, with the verification behind it. "Verified" below means the check was
actually run against the named artifact, not read from documentation.

| # | Claim | Verification | Verdict | Commit |
|---|---|---|---|---|
| E1 | `security-scan`'s frontmatter `description` advertised scanning `.claude/` for DSH | Read the frontmatter; descriptions are what a session catalogs | **Confirmed** — the catalog entry itself was false | `8ee1bc4` |
| E2 | `ecc-agentshield` is real but Claude-Code-targeted | npm registry: v1.6.0, MIT, deps include `@anthropic-ai/sdk`; own description says "Scans Claude Code setups" | **Confirmed** — and **not installed** on this machine | `8ee1bc4` |
| E3 | DSH ships no configuration scanner | Enumerated the installed `@deepseek-ai` tree: sandbox / approval / permission plugins only | **Confirmed** — DSH's controls are runtime enforcement, not scanning | `8ee1bc4` |
| E4 | DSH does not read `.mcp.json` | Grepped installed `@deepseek-ai/dsh-mcp-client` and `dsh-mcp-resources` | **Confirmed** — zero hits | `ea65171` |
| E5 | DSH MCP config is a plugin row | Read `dsh-mcp-client` `Config` schema (L780–L793) | **Confirmed** — `transport` / `serverName` / `command` (+ optional `args`, `env`) | `ea65171` |
| E6 | `dsh` is a valid `ECC_MEMORY_HARNESS` value | Unpacked `ecc-universal@2.2.3` (not installed globally) and read the source | **Confirmed** — see below | `ea65171` |
| E7 | Retiring a skill does not require editing the verifier's count | Read `verify-provider.mjs` L141: prints `${expected.size}`, computed from the Set | **Confirmed** — the pass message self-updates | `e4d06d1` |
| E8 | `configure-ecc` has no DSH meaning | Read the whole file: every instruction targets `claude plugin` / `codex plugin` / `./.kimi-code` / `$CLAUDE_PLUGIN_ROOT` / Claude hook profiles | **Confirmed** — **retired**, not adapted | `e4d06d1` |

### E6 in detail — why `dsh` is a legal harness id

The skill hardcoded `ECC_MEMORY_HARNESS=codex`, which every DSH reader would have copied. The
value is a free-form slug, not an enum:

```javascript
// ecc-universal@2.2.3 scripts/lib/memory-vault-format.js:28
const SLUG_PATTERN = /^[a-z0-9][a-z0-9._-]{0,63}$/;

// scripts/memory-mcp.mjs (resolveServiceSecurity) — throws at startup if it does not match
```

Confirmed against three independent artifacts in the same package: the server's startup check,
the slug constant, and `schemas/memory.schema.json`'s `#/definitions/slug`. No file checks a
harness whitelist.

**Cross-harness sharing is unaffected.** Reads filter on `target_harnesses`, which defaults to
`['all']` (`memory-vault.js:310`, `memory-mcp.mjs:285`), so a memory authored under `dsh` stays
readable by every other harness unless the author narrows it deliberately.

**Consequence now documented in the skill:** an uppercase value such as `DeepSeek-Harness` makes
the MCP server *throw at startup*, not merely fail one call.

