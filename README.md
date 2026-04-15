# AIShell-Gate — AI Execution Gateway for Unix

> v1.02 · policy 1.10 · exec 0.43.0 · mcp 1.0

**A policy layer that stands between AI-generated shell commands and your Unix systems.**

AI tools are increasingly capable of generating and running shell commands directly on Unix infrastructure. That capability is useful. It is also a new category of operational risk. AIShell-Gate exists to make that risk explicit, manageable, and auditable.

---

## The Problem

When an AI agent — or an AI-assisted operator — proposes a shell command, several things are missing that experienced teams rely on in every other part of their operation:

- No declared policy governing what is and is not permitted
- No risk assessment before the command runs
- No mandatory confirmation step proportional to the danger involved
- No tamper-evident audit record of what ran, what was denied, and why

A misplaced flag, a wrong path, or a misunderstood scope in an AI-generated command can cause irreversible damage. Unlike human operators who carry years of pattern recognition, AI agents have no instinctive hesitation. They will execute a destructive command with the same confidence as a safe one.

AIShell-Gate closes that gap.

---

## What AIShell-Gate Does

AIShell-Gate intercepts every proposed command — whether from an AI agent, a human operator, or a scripted pipeline — and evaluates it against a declared policy stack before a single byte reaches the kernel. It never invokes a shell. It does not interpret shell syntax. Every command is treated as a structured, tokenised action with a deterministic outcome.

**The evaluation produces:**

- **Allow or deny** — with the matched rule and the policy layer it came from
- **Risk score** (0–100) — based on the command, its flags, its target paths, and its blast radius
- **Confirmation level** — none, plan-level, action-level, or typed challenge — calibrated to the risk
- **Flag assessment** — 2,088 individual flag assessments across 188+ commands identify dangerous modifiers before execution
- **Network warning** — commands targeting local inference endpoints, cloud metadata services, or inherently dangerous ports are flagged with appropriate escalation
- **Audit log entry** — every decision, whether allow or deny, is recorded in a tamper-evident log

---

## The Components

AIShell-Gate ships as five components that work together:

**aishell-gate-policy** — The policy engine. Receives a proposed command, evaluates it, and returns a structured JSON decision. It never executes anything. This is the component that enforces your rules.

**aishell-gate-exec** — The execution gateway. Reads a JSON action plan from an AI agent or operator, submits each command to the policy engine, collects the required human confirmation, and — only if the policy allows and the operator confirms — calls `execve()` with the validated argument array. No shell is ever invoked. This component is commercially licensed.

**aishell-confirm** — The operator confirmation relay. For deployments where the operator is not at the same terminal as the AI agent — SSH sessions, remote pipelines, headless servers — aishell-confirm provides a separate confirmation channel. Confirmation requests arrive at the operator's terminal; their response is transmitted back. This component is commercially licensed.

**aishell-gate-mcp** — The MCP server. Exposes two tools to AI coding environments (Claude Code, Cursor, and any MCP-compatible client):
- **`evaluate_plan`** — Submit a goal and command list for policy-gated inspection without executing anything. Returns per-action decision, confirm level, resolved binary path, risk score, and reason.
- **`execute_plan`** — Submit a goal and command list for live execution. Runs a pre-flight evaluate_plan internally. Plans containing `action` or `typed` confirm actions are blocked and reported. Transport: stdio. Configuration: `aishell-mcp.json` in the project directory.

**aishell-gate** — The single named entry point for all AIShell-Gate sessions. Performs pre-flight checks on both binaries (root, existence, setuid/setgid), automatically injects `--policy-binary`, and hands off to the executor via `exec()`. Use this in preference to calling `aishell-gate-exec` directly. It is the intended `authorized_keys` forced command for SSH deployments and is designed to remain the stable user-facing entry point as the system evolves.

---

## MCP Integration (Claude Code / Cursor)

`aishell-gate-mcp` connects AIShell-Gate to Claude Code, Cursor, and any MCP-compatible environment in three steps.

**1. Create `aishell-mcp.json` in your project root:**

```json
{
  "exec_binary":   "aishell-gate-exec",
  "policy_binary": "aishell-gate-policy",
  "preset":        "ops_safe",
  "source":        "ai"
}
```

**2. Add to `.mcp.json`:**

```json
{
  "mcpServers": {
    "aishell-gate": {
      "command": "python3",
      "args": ["/path/to/aishell-gate-mcp.py"]
    }
  }
}
```

**3. Restart your AI coding environment.** The tools are discovered automatically.

Every other MCP tool gives an AI access to one capability. AIShell-Gate gives it access to the operating system — the thing that contains everything else. The policy engine is what makes that access safe.

---


---

## Editions

AIShell-Gate is available in two editions. Run `--version` to identify which edition you have.

**Standard** — Policy engine, execution gateway, all presets, confirmation gates, interactive mode, audit logging (JSON Lines), jail-root enforcement, session policy, custom policy files.

**Enterprise** — All standard features plus: `--dump-policy` (full resolved policy stack as JSON), HMAC-SHA256 tamper-evident audit chain, `--audit-verify` (chain integrity verification), `--audit-key` (keyed HMAC), and cryptographic session ID correlation.

Invoking an enterprise-only flag on a standard binary produces a clear `not available in standard edition` message — not a silent failure.

---

## Policy and Confirmation Levels

Policy is a stack of layers evaluated in order: **base** (organisational floor), **project** (workflow-specific rules), and **user** (personal preferences). A deny at any layer is final. Seven built-in presets provide working starting postures without manually assembling policy files:

| Preset | Posture |
|---|---|
| `read_only` | Inspection only — no writes, no deletions |
| `ops_safe` | Conservative read and repository commands (default) |
| `dev_sandbox` | Developer workflow: git, make, compilers, package managers |
| `ci_build` | Unattended build/test pipeline, all allowed commands at confirm:none |
| `ci_deploy` | Unattended deploy pipeline, adds container/k8s/Terraform tooling |
| `ci_admin` | Supervised admin with mixed confirmation levels |
| `danger_zone` | Minimal restrictions, typed confirmation for most commands |

Confirmation levels are assigned by the policy engine based on risk:

| Level | Meaning |
|---|---|
| `none` | Safe, routine command — proceeds without interruption |
| `plan` | The operator is shown the full plan before any action runs |
| `action` | Each action requires individual acknowledgement |
| `typed` | A challenge code must be typed to confirm — for high-risk, irreversible commands |

An AI agent running with `source: ai` receives additional scrutiny. Known AI inference API ports on loopback are flagged as potential self-referential access risks and require explicit operator confirmation.

---

## The Audit Log

Every action evaluated by AIShell-Gate — whether allowed, denied, confirmed, or refused — produces a structured JSON log entry. In the Enterprise edition, entries are chain-linked: each record includes an HMAC-SHA256 of the previous record, making any post-hoc modification detectable. The chain can be verified with a single command (`--audit-verify`). The standard edition writes the same JSON Lines entries without the HMAC chain.

The log records the command, the decision, the confirmation level, the matched policy rule and layer, the risk score, the blast radius, and the timestamp. It is designed to be human-readable without a JSON parser, and machine-parseable for integration with existing logging infrastructure.

---

## What's New in v1.02

- **MCP integration** — `aishell-gate-mcp.py` exposes `evaluate_plan` and `execute_plan` to Claude Code and Cursor
- **`--dry-run-json`** — machine-readable plan inspection without execution (AI-friendly)
- **`--test-plan`** — policy test runner for CI gates (PASS/FAIL per case, exit 0/1/2)
- **Net default-deny** — `ops_safe`, `read_only`, `dev_sandbox` now require explicit allow rules for network targets
- **Flag catalog 2,088** — up from 1,371: ansible, az, extended kubectl/helm/terraform, insmod, debugfs, socat, nft, LVM
- **Protocol versioning** — all four JSON interfaces versioned at 1.0

---

## Who It Is For

AIShell-Gate is designed for **Unix system administrators, DevOps engineers, and security-conscious operators** who are deploying or evaluating AI-assisted shell automation in production or near-production environments. It is equally applicable to teams managing their own AI agents and to individuals who want a formal boundary between AI suggestions and system execution.

It is not a replacement for standard access controls, system permissions, or network security. It is a focused, auditable execution boundary that formalises the AI-to-shell interface.

---

## Security

To report a vulnerability privately, use the contact form at [www.aishellgate.com](https://www.aishellgate.com). Please do not use public GitHub issues for security reports. We will acknowledge reports within 48 hours.

---

## Beta Access

AIShell-Gate 1.0 is currently in a controlled beta program. The beta testing package — compiled binaries (Linux x86_64 and arm64, tar.gz), full documentation, and release notes — is available by request to qualified evaluators. Testing should be performed in isolated, non-production environments.

**To request access:** [www.aishellgate.com](https://www.aishellgate.com)

---

## License

AIShell-Gate is available under three license tiers:

**Evaluation** — A time-limited evaluation copy is available at no charge. Evaluation copies operate with a reduced feature set and may not be used in production environments. See the LICENSE file for full conditions.

**Standard Edition** — Perpetual, single fixed host. Activated with a license key issued at time of purchase. Until activated, the Software displays an "Unlicensed" notice at each invocation.

**Enterprise Edition** — Perpetual, single fixed host. Issued as a cryptographically signed license file bound to the hardware identity of the licensed host. Includes all features, direct support, and cloud/ephemeral deployment licensing.

License inquiries: [www.aishellgate.com](https://www.aishellgate.com)

---

## Documentation

| Document | Description |
|---|---|
| Getting Started Guide | Installation, first session walkthrough, policy introduction |
| Remote Deployment Guide | SSH forced-command setup, multi-terminal confirmation relay |
| White Paper | Architecture, design rationale, security positioning |
| Educational Overview | Policy engine as a learning environment for new Unix administrators |
| Policy Man Page | Complete policy file reference, rule syntax, preset definitions, --test-plan |
| Executor Man Page | Full flag reference, --dry-run-json, JSON plan format, audit log specification |

All documentation ships with the beta testing package and is available at [www.aishellgate.com](https://www.aishellgate.com). Documents listed above are not included in this repository.

---

*AIShell-Gate is proprietary software. Copyright © 2026 AIShell Labs LLC, Winston-Salem NC USA. All Rights Reserved. Use requires a valid license. Source code is available to organisations conducting a security review — contact us at [www.aishellgate.com](https://www.aishellgate.com). This repository contains documentation only.*
