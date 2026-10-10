---
name: security-scan
description: How to run the external AgentShield scanner (npx ecc-agentshield) against AI-agent configuration for vulnerabilities, misconfigurations, and injection risks. AgentShield targets Claude Code configs (.claude/, CLAUDE.md, settings.json, mcp.json, hooks, agents); DSH coverage is not established. Use when the user explicitly wants AgentShield or is auditing Claude Code configuration.
metadata:
  origin: ECC
---

# Security Scan Skill

Audit your Claude Code configuration for security issues using [AgentShield](https://github.com/affaan-m/agentshield).

> **Harness scope — read before using.** This skill is a usage manual for the external
> `ecc-agentshield` package (v1.6.0, not installed on this machine). It is **not** a DSH security
> feature and it does **not** scan DSH configuration.
>
> - The scanner's stated target is Claude Code configuration. Whether it understands DSH's
>   equivalents (`AGENTS.md`, `~/.dsh/.credentials.yaml`, `cordis.patch.yml`, the plugin tree)
>   is **unverified** and must not be assumed.
> - Running `--opus` requires `ANTHROPIC_API_KEY` and bills Anthropic; it is not a DeepSeek call.
> - **DSH ships no configuration scanner.** Its security surface is the sandbox and approval
>   plugin family — see "DSH security surface" below before reaching for an external tool.


## When to Activate

- Setting up a new Claude Code project
- After modifying `.claude/settings.json`, `CLAUDE.md`, or MCP configs
- Before committing configuration changes
- When onboarding to a new repository with existing Claude Code configs
- Periodic security hygiene checks

## What It Scans

| File | Checks |
|------|--------|
| `CLAUDE.md` | Hardcoded secrets, auto-run instructions, prompt injection patterns |
| `settings.json` | Overly permissive allow lists, missing deny lists, dangerous bypass flags |
| `mcp.json` | Risky MCP servers, hardcoded env secrets, npx supply chain risks |
| `hooks/` | Command injection via interpolation, data exfiltration, silent error suppression |
| `agents/*.md` | Unrestricted tool access, prompt injection surface, missing model specs |

## Prerequisites

AgentShield must be installed. Check and install if needed:

```bash
# Check if installed
npx ecc-agentshield --version

# Install globally (recommended)
npm install -g ecc-agentshield

# Or run directly via npx (no install needed)
npx ecc-agentshield scan .
```

## Usage

### Basic Scan

Run against the current project's `.claude/` directory:

```bash
# Scan current project
npx ecc-agentshield scan

# Scan a specific path
npx ecc-agentshield scan --path /path/to/.claude

# Scan with minimum severity filter
npx ecc-agentshield scan --min-severity medium
```

### Output Formats

```bash
# Terminal output (default) — colored report with grade
npx ecc-agentshield scan

# JSON — for CI/CD integration
npx ecc-agentshield scan --format json

# Markdown — for documentation
npx ecc-agentshield scan --format markdown

# HTML — self-contained dark-theme report
npx ecc-agentshield scan --format html > security-report.html
```

### Auto-Fix

Apply safe fixes automatically (only fixes marked as auto-fixable):

```bash
npx ecc-agentshield scan --fix
```

This will:
- Replace hardcoded secrets with environment variable references
- Tighten wildcard permissions to scoped alternatives
- Never modify manual-only suggestions

### Opus 4.6 Deep Analysis

Run the adversarial three-agent pipeline for deeper analysis:

```bash
# Requires ANTHROPIC_API_KEY
export ANTHROPIC_API_KEY=your-key
npx ecc-agentshield scan --opus --stream
```

This runs:
1. **Attacker (Red Team)** — finds attack vectors
2. **Defender (Blue Team)** — recommends hardening
3. **Auditor (Final Verdict)** — synthesizes both perspectives

### Initialize Secure Config

Scaffold a new secure `.claude/` configuration from scratch:

```bash
npx ecc-agentshield init
```

Creates:
- `settings.json` with scoped permissions and deny list
- `CLAUDE.md` with security best practices
- `mcp.json` placeholder

### GitHub Action

Add to your CI pipeline:

```yaml
- uses: affaan-m/agentshield@v1
  with:
    path: '.'
    min-severity: 'medium'
    fail-on-findings: true
```

## Severity Levels

| Grade | Score | Meaning |
|-------|-------|---------|
| A | 90-100 | Secure configuration |
| B | 75-89 | Minor issues |
| C | 60-74 | Needs attention |
| D | 40-59 | Significant risks |
| F | 0-39 | Critical vulnerabilities |

## DSH security surface

Verified against the installed DSH (`0.2.0-rc.2`). **DSH has no scanner**; its controls are
runtime enforcement plugins, which is a different kind of thing:

| Plugin | Role |
|---|---|
| `dsh-sandbox`, `dsh-sandbox-policy`, `dsh-sandbox-local` | sandbox mode ladder: `read-only` → `workspace-write` → `danger-full-access` |
| `dsh-bash-sandbox`, `dsh-pwsh-sandbox` | confine shell command file effects |
| `dsh-fs-sandbox` | confine model file writes/edits; denies with `FS_SANDBOX_DENIED` |
| `dsh-permission-presets` | the user-facing Permissions select bundling sandbox mode + approval policy |
| `dsh-user-approval` | approval policy `ask` / `never`; `ask` fails closed |
| `dsh-authorization` | authorization service |
| `dsh-sandbox-windows-acl` | Windows ACL diagnosis/repair |

Two limitations worth remembering, because they are easy to over-trust:

- **Sandbox modes govern file effects only.** Network access is unrestricted, and process
  visibility is backend-specific. The sandbox is not a network boundary.
- Escalation is strictly *wider*: `read-only` → `workspace-write` → `danger-full-access`, per call,
  with approval. There is no path to a narrower mode.

To review the live permission posture:

```powershell
dsh --dump-config    # inspect sandbox / permission / approval rows
```

## Interpreting Results

### Critical Findings (fix immediately)
- Hardcoded API keys or tokens in config files
- `Bash(*)` in the allow list (unrestricted shell access)
- Command injection in hooks via `${file}` interpolation
- Shell-running MCP servers

### High Findings (fix before production)
- Auto-run instructions in CLAUDE.md (prompt injection vector)
- Missing deny lists in permissions
- Agents with unnecessary Bash access

### Medium Findings (recommended)
- Silent error suppression in hooks (`2>/dev/null`, `|| true`)
- Missing PreToolUse security hooks
- `npx -y` auto-install in MCP server configs

### Info Findings (awareness)
- Missing descriptions on MCP servers
- Prohibitive instructions correctly flagged as good practice

## Links

- **GitHub**: [github.com/affaan-m/agentshield](https://github.com/affaan-m/agentshield)
- **npm**: [npmjs.com/package/ecc-agentshield](https://www.npmjs.com/package/ecc-agentshield)
