# Using this repo

This repo has **two distinct uses**, and conflating them is the main way to misuse it. Read this
before installing anything.

## What it is — and is not

| It is | It is not |
|---|---|
| A **port of ECC's skill knowledge** to DSH | A port of ECC's *behaviour* |
| A **workbench** for making those skills DSH-correct | A finished, ready-to-use package |
| Skills as **Markdown injected into context** | Skills that execute, enforce, or hook anything |

Upstream ECC ships 71 agents, 95 commands, a hook runtime, and AgentShield scanning. **None of
that is here.** Everything in this package is prompt text. See [DSH-ADAPTATION.md](DSH-ADAPTATION.md).

**Roughly 30 of the 273 skills still carry Claude-specific assumptions** and are documented as
broken or pending retirement. Loading them wholesale advertises capabilities that do not exist.

## The two ways to consume it

### A. Selective copy — recommended for personal use

DSH discovers skills from disk natively; no plugin required.

```powershell
New-Item -ItemType Directory -Force -Path .dsh\skills | Out-Null
Copy-Item <path-to-this-repo>\skills\tdd-workflow .dsh\skills\ -Recurse
```

Pick skills you will actually use. They load in that project only, the repo becomes a *source*
rather than a dependency, and removing one is `Remove-Item`.

### B. Install the plugin — for distribution

```powershell
dsh plugin --profile web add github:<your-fork>/dsh-ecc
```

**Know the cost before doing this.** DSH injects a *durable catalog of every skill name plus a
capped description into every session*, before the first request
(`dsh-tool-skill`: `catalogDescriptionMaxLength`, default 500). Across 273 skills that is
**roughly 16–18k tokens added to every session, permanently**.

Install it when the goal is to *distribute* the package. For your own day-to-day use, method A
gives you the same skills without the tax.

`catalogDescriptionMaxLength` is configurable, but lowering it shrinks each entry rather than
reducing the count — it is not a lever for the total.

## When to use which

| Situation | Do this |
|---|---|
| You want a few skills in your own projects | **A** — copy them |
| You are adapting skills | Work in this repo; see [DSH-ADAPTATION.md](DSH-ADAPTATION.md) |
| You are publishing skills to others | **B** — but finish Tier A first |
| You want ECC's hooks, agents, or commands | Not available here — that is a separate project |

## Before installing: finish the adaptation

Installing now loads ~30 skills you have already documented as wrong. Those defects stop being
inert files and start being claims your agent reads every session. Either finish Tier A or copy
only skills that have been checked.
