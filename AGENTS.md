# AGENTS.md — dsh-ecc

Adaptation workspace for porting [affaan-m/ECC](https://github.com/affaan-m/ECC) skills to
**DeepSeek Harness (DSH)**. Upstream port by [gongyijie85/dsh-ecc](https://github.com/gongyijie85/dsh-ecc) (MIT).

This is a **fork of a port**. The goal is not to add skills — it is to make the existing
273 skill files correct for DSH instead of for Claude Code.

## What this repo is

A Cordis plugin that registers the 273 `skills/<name>/SKILL.md` files into DSH's `ctx.skills`
registry. Almost all runtime logic lives in [`lib/index.js`](lib/index.js) (~195 lines).
Skill bodies are plain Markdown injected into agent context.

**Nothing here executes commands, writes files, or makes network calls at runtime.** The plugin
only reads Markdown. Keep it that way.

**Consuming this repo has two modes** — selective copy (recommended) or plugin install, which
adds ~16–18k tokens to every session. See [docs/USAGE.md](docs/USAGE.md) before advising either.

## Invariants — do not break these

1. **`npm run verify` must pass before every commit.** It is the only real test.
2. `scripts/verify-provider.mjs` holds a **hardcoded exact set of 273 skill names** (L21–L109).
   Adding, renaming, or removing a skill directory *requires* updating that set, or the script
   fails twice: once for an unexpected skill, once for a missing one.
3. **Skill directory name must equal the `name:` in that skill's frontmatter.** The verifier
   compares the discovered set to expected names.
4. Every `SKILL.md` must have YAML frontmatter with a non-empty `description` that does **not**
   start with `>` (a folded scalar leaked into the description fails the verifier).
5. Do not change `lib/index.js` provider semantics (`rank: 550`, `source: 'custom'`) without
   reading pinned DSH source first. The rank deliberately sits below DSH's bundled root (600)
   so user/project skills win name collisions.
6. **Keep the `files` whitelist in `package.json` accurate.** `scripts/` and `.github/` are
   currently excluded from the published tarball — do not assume a published artifact contains them.

## The actual work: adaptation, not addition

98% of these skills were copied verbatim from upstream and still assume Claude Code. Only
2 command references were ever fixed. The job is to make them DSH-native.

**Do not rewrite a skill wholesale.** Make the minimum edit that removes a broken assumption.
A skill that references tools or paths DSH does not have is worse than no skill: it teaches the
agent to do something impossible.

Priority order (counts measured on this clone — see [docs/DSH-ADAPTATION.md](docs/DSH-ADAPTATION.md)):

| Tier | What | Count |
|---|---|---|
| **A** | References `.claude/` paths, `hooks.json`, `PreToolUse`/`PostToolUse`, `subagent_type` | 30 |
| **B** | References `mcp.json` / `mcpServers` config that DSH does not read | 13 |
| **C** | References the `ecc` CLI / `scripts/ecc` that is not installed here | 3 |

Tier C first (3 files, entirely broken), then Tier A, then Tier B.

## Conventions when editing a skill

- Preserve the YAML frontmatter exactly except for the field being changed.
- Preserve upstream attribution. Frontmatter carries `metadata.origin: ECC` — never remove it.
- One logical change per commit; message format `fix(<skill-name>): <what changed>`.
- If a skill cannot be meaningfully adapted (it depends on Claude-only infrastructure), **say so
  and mark it** rather than shipping a plausible-looking lie. See the "Retire instead of fake"
  rule in [.agents/rules/skill-adaptation.md](.agents/rules/skill-adaptation.md).

## Settled decisions — do not re-litigate

Tier C is complete and every change is backed by a recorded verification. Before proposing to
revisit one, read the ledger and cite what changed:

- **[docs/DSH-ADAPTATION.md](docs/DSH-ADAPTATION.md#evidence-and-decision-trail)** — claim →
  verification → verdict → commit, for each of the three Tier-C items.
- **[docs/SESSION-NOTES.md](docs/SESSION-NOTES.md)** — where the decisions came from, and the five
  defects the review gate caught.

Two specific things that look wrong but are not, both already investigated:

- `configure-ecc` was **retired, not adapted** — it is an upstream setup wizard for Claude Code /
  Codex / Kimi with no DSH equivalent. See ledger E8.
- The READMEs say **274/286** while the active count is **273**. That is deliberate: 274/286 is
  *upstream port coverage* (274 + 12 un-ported = 286), not the count of skills that ship.

## Source of truth for DSH behaviour

**Do not guess DSH APIs, config keys, or package names.** Verify against the installed
distribution before writing them into a skill:

```powershell
# Installed DSH version and plugin tree
& "$env:LOCALAPPDATA\..\Roaming\npm\node_modules\.bin\dsh.cmd" --version
dsh --dump-config
```

Installed reference source (read it, it is ground truth):
`C:\Users\91704\AppData\Roaming\npm\node_modules\@deepseek-ai\dsh\`

Key facts already verified — do not re-litigate:

- Provider API is `ctx.skills.registerProvider`, contract from `@deepseek-ai/dsh-skill`.
- `SkillCandidate` requires `rank`, `locator`, `metadata`.
- `SkillInvocationPolicy` is `{ modelInvocable, userInvocable }`.
- `source: 'custom'` is a valid `SkillSource`.
- DSH also discovers skills natively from disk (`.dsh/skills`, `.agents/skills`, `~/.agents/skills`),
  which is why this plugin must add value beyond "copies files".

## Verification before claiming done

```powershell
npm run verify        # must end: OK: all 273 ecc skills discovered and get() passes
git diff --stat       # confirm you changed what you meant to
```

Report what you actually ran. If you did not run the verifier, say so.
