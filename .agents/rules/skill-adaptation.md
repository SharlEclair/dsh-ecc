---
trigger: model_decision
description: "Apply when adapting a skill file under skills/, triaging which skills need DSH adaptation, deciding whether a skill should be retired, or updating the expected-skill set in scripts/verify-provider.mjs."
---

# Skill adaptation

How to change a skill file so it is true for DSH instead of Claude Code.

## Hard rule: retire instead of fake

A skill that instructs the agent to use a tool, path, or CLI that DSH does not have is **worse
than no skill** — it teaches a confident wrong action. If a skill cannot be adapted without
inventing capability, do not paper over it. Either:

1. Rewrite the specific instruction against a DSH-native equivalent that you have **verified
   exists**, or
2. Add a short note at the top of the body stating the dependency is unavailable and what to do
   instead, or
3. Remove the skill and its entry from the verifier set, and record the removal.

Never leave a reference to a nonexistent tool pretending to work.

## Verified DSH surface you may rely on

| Claude Code assumption | DSH reality |
|---|---|
| `Skill` tool | `skill` tool (registered by `@deepseek-ai/dsh-tool-skill`) |
| `Task(subagent_type=...)` | `subagent` / `subagent_fork` tools, or Agent Teams |
| `.claude/settings.json` hooks | no direct equivalent — hooks do not port |
| `.claude/agents/*.md` | subagent providers / Agent Teams |
| `CLAUDE.md` | **`AGENTS.md`** (DSH discovers `AGENTS.md` and `CLAUDE.md`, plus `.local.md` overlays and `~/.dsh/AGENTS.md`) |
| `/ecc:...` slash commands | not available |
| `ecc` CLI / `scripts/ecc.js` | not installed here |
| `mcp.json` / `mcpServers` | DSH MCP config is not that file |

**Do not invent DSH config keys.** If you need one, read the installed source under
`C:\Users\91704\AppData\Roaming\npm\node_modules\@deepseek-ai\dsh\` first, or say the detail
is unverified.

## Frontmatter rules

- Keep `name`, `description`, and any `whenToUse` intact unless they are themselves wrong.
- `name` must equal the directory name.
- `description` must be a single non-empty line (folded/`>` scalars break the verifier).
- Preserve `metadata.origin: ECC` — this is MIT attribution, not decoration.
- `disable-model-invocation: true` means user-invoked only; the plugin maps it to
  `{modelInvocable:false, userInvocable:true}`. Do not strip it.

## Editing checklist

1. Confirm the claim is actually wrong for DSH — read the surrounding context, not just the match.
2. Make the minimum edit. Do not reorganise or "improve" unrelated prose.
3. `npm run verify` → must pass.
4. Commit `fix(<skill-name>): <what changed>`.

## Keep the verifier honest

`scripts/verify-provider.mjs` L21–L109 is an **exact set**. On any add/rename/remove you must
update it, otherwise it reports both an unexpected and a missing skill. That strictness is
intentional — it catches accidental additions — so update the set rather than loosening the check.
