# Session Notes — provenance and method

Where the Tier-C decisions came from, and how the work was actually done. Recorded so a future
reader can tell *decided* from *assumed*, and can find the original reasoning instead of trusting
a summary.

## Provenance

| Item | Value |
|---|---|
| Agent | Google Antigravity IDE |
| Session | `d93a616a-c3c3-4a7d-8d59-3e70b6c698ab` |
| Transcript | `~/.gemini/antigravity/brain/d93a616a-c3c3-4a7d-8d59-3e70b6c698ab/.system_generated/logs/transcript.jsonl` |
| Span | 2026-10-10 19:36:02Z → 20:11:47Z (~36 min) |
| Volume | 254 transcript records · 9 user turns · 63 steps |
| Models | Gemini 3.8 Flash (High) → **3.1 Pro (High)** at the review turn → back to 3.8 Flash |

The transcript is **not** in this repository and is not required to use it. Everything
load-bearing is reproduced below or in [DSH-ADAPTATION.md](DSH-ADAPTATION.md#evidence-and-decision-trail).

## Method, and what it got wrong

The work ran as **diagnose → review → apply**, with a correction turn between diagnosis and
editing. That gate was not ceremony; it caught five defects that a single-pass agent would have
shipped:

| Defect | Caught by | Kind |
|---|---|---|
| `unified-memory` L17 flagged as a Claude assumption | reading the whole sentence | **false positive** — the line lists *install types*; "Claude plugin installs" is one item among four |
| Verifier told to update a hardcoded `274` | reading `verify-provider.mjs` | **invented edit** — the count is `expected.size`, computed |
| `git diff --stat` reported 12 files, deletion invisible | `git status --short` | **incomplete verification** — `--stat` omits staged changes |
| README metric changed `274/286` → `273/286` | arithmetic: `273 + 12 = 285 ≠ 286` | **broken invariant** — port *coverage* is not active count |
| `ECC_MEMORY_HARNESS=codex` missed in the first pass | targeted re-read of the MCP section | **incomplete defect list** |

**Two of those five were the reviewer's errors, not the agent's** — a vague "annotate this"
instruction, and a disposition table that implied changing the wrong numerator. Recorded because
the pattern matters more than the blame: the review gate catches the reviewer too.

### Model correlation — weak evidence, stated as such

The one diagnosed round that used 3.1 Pro (step 40 → 50) produced an accurate defect list and
correctly refused to guess `ECC_MEMORY_HARNESS`, attributing it as unverified instead. The Flash
rounds before and after both needed correction.

**n = 1 round.** That is a hint about task fit, not a finding. Do not generalise it into a routing
policy on this evidence.

## What the port deliberately does not do

Upstream ECC ships 71 agents, 95 commands, a hook runtime, and AgentShield scanning. **This
package ships none of them** — it is a skills-only port. That is a design boundary, not a gap to
close here. Building a DSH bridge for hooks/agents would be a separate project; see the Tier A
note in [DSH-ADAPTATION.md](DSH-ADAPTATION.md).

## Rule of thumb this session produced

**Trust the investigation; re-verify the bookkeeping.**

Every failure above was in counting, coverage, or reporting — not in reading code. The agent's
*investigations* were consistently sound (it unpacked a tarball rather than installing it,
read a schema rather than assuming it, and ran the server). Its *accounting* was where errors
appeared, and none of them were visible in its own self-report. Each needed an independently
re-run check.
