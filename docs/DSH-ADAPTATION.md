# DSH Adaptation Plan

Turning the 274 ported ECC skills from Claude-Code-shaped into DSH-native.

**Baseline verified** — `node scripts/verify-provider.mjs` → `OK: all 274 ecc skills discovered and get() passes` (Node v24.15.0).

## Why this work exists

The upstream port copied skill bodies almost verbatim from [affaan-m/ECC](https://github.com/affaan-m/ECC).
Its own README states the only adaptation was two command references
(`/bug-check` → `bug-check`, `/prompt-optimize` → `prompt-optimizer`).

Measured on this clone:

| Signal | Files / 274 |
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

**Estimated distinct files to touch: ~40 of 274.** The other ~234 are generic engineering
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

- `npm run verify` before every commit; it must stay at `OK: all 274`.
- One skill per commit: `fix(<skill-name>): <what changed>`.
- Read [.agents/rules/skill-adaptation.md](../.agents/rules/skill-adaptation.md) before editing —
  especially the "retire instead of fake" rule.
- Do **not** bulk find-and-replace across `skills/`. A blind replace of `.claude/` → something
  will silently corrupt at least some of the 27 files that reference it.

## Open questions to resolve before large-scale work

1. **Is the plugin even the right delivery mechanism?** DSH already discovers skills natively
   from `.dsh/skills`, `.agents/skills`, and `~/.agents/skills`. Mounting this plugin puts all
   274 into every session's catalog. For your own use, copying selected skills may be better
   than shipping 274. Decide this before polishing all 40 files.
2. **Which skills do you actually want?** Adapting 40 files nobody loads is waste. Pick the
   subset first, then adapt only that subset.
3. **Upstream sync.** This clone is `v0.6.0`; npm has `0.6.1`. Decide whether you track upstream
   or diverge, because divergence makes future rebases harder.

## Status

| Tier | Files | Done |
|---|---|---|
| C | 3 | **1** (`security-scan` description) |
| A | 30 | 0 |
| B | 13 | 0 |
| **distinct** | **~40** | **1** |

Update this table as work lands. Do not mark a tier done without a green `npm run verify`.
