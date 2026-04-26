# AIShell-Gate

**"Because AI shouldn't have root."**

A deterministic policy layer that sits between an AI agent and your Unix system. Every command — from Claude Code, Cursor, a local model, or a pipeline script — is evaluated against declared policy before a single byte reaches the kernel. Safe commands are allowed. Unsafe commands are denied, with a reason. Everything is audited.

→ **[aishellgate.com](https://www.aishellgate.com)**

---

## The Problem

AI coding assistants are good at generating shell commands. They are also probabilistic. Unix execution is deterministic and irreversible. A single misplaced flag or path can permanently alter system state. Between the AI and your infrastructure, there is currently nothing.

AIShell-Gate closes that gap.

---

## Claude Code & Cursor — MCP Integration

AIShell-Gate exposes two MCP tools to Claude Code and Cursor:

- **`evaluate_plan`** — submit a list of commands to the policy engine without executing anything. Returns a per-action decision, risk score, confirmation level, and reason for each command.
- **`execute_plan`** — live execution with pre-flight policy check. Plans containing high-risk or blocked actions are stopped before any command runs.

Two config files. Ten minutes. Full walkthrough: [Getting Started Guide §20 →](https://www.aishellgate.com/AIShell_Gate_Getting_Started.html#mcp-quickstart)

---

## How It Works

```
AI Agent
 │ JSON action plan
 ▼
aishell-gate-exec  (no policy logic — pure execution harness)
 │ forks policy engine for each action
 ▼
aishell-gate-policy  (deterministic policy engine — separate process)
 │ ALLOW / DENY + confirmation level + validated argv
 ▼
aishell-gate-exec  (collects human confirmation if required)
 │
 ▼
execve(absolute_path, validated_argv, safe_environment)
(no shell — no PATH inheritance — no environment injection)
```

The executor has no policy logic. The policy engine has no ability to execute. Neither can reach across that boundary — including if the executor is compromised.

---

## What's in the Box

| | |
|---|---|
| **346** Unix commands catalogued | **3,489** individual flag assessments |
| **7** built-in policy presets | **4** confirmation levels (`none`, `plan`, `action`, `typed`) |
| Tamper-evident audit log (SHA-256 chain) | HMAC-SHA256 audit mode for compliance |
| No shell invocation — direct `execve()` | No external dependencies beyond libc |
| Agent-agnostic (Claude Code, Cursor, pipelines, SSH) | NIST AI RMF / AI 600-1 aligned |

---

## Policy Presets

`ops_safe` · `dev_sandbox` · `read_only` · `danger_zone` and three more. A working starting posture without manually assembling policy files. Layered: base → project → user. A deny at any layer is final.

---

## Compliance

For organizations subject to HIPAA, PCI-DSS, CMMC, or FedRAMP requirements, the audit chain provides a documented, cryptographically linked record of every AI-proposed command — whether allowed, denied, or confirmed — before execution. The two-binary reference monitor architecture reflects NIST SP 800-162.

---

## Documentation

All documentation is at **[aishellgate.com](https://www.aishellgate.com)**:

- [The Problem Statement](https://www.aishellgate.com/aishellgate-problem-v2.html)
- [White Paper](https://www.aishellgate.com/AIShell_Gate_White_Paper.html)
- [Getting Started Guide](https://www.aishellgate.com/AIShell_Gate_Getting_Started.html)
- [Claude Code & Cursor MCP Integration](https://www.aishellgate.com/AIShell_Gate_Getting_Started.html#mcp-quickstart)
- [Remote Deployment Guide](https://www.aishellgate.com/AIShell_Gate_Remote_Howto.html)
- [Federal / Regulated Environments](https://www.aishellgate.com/aishell-gate-federal-context.html)
- [Policy Engine Man Page](https://www.aishellgate.com/aishell_gate_policy_manpage.txt)
- [Executor Man Page](https://www.aishellgate.com/aishell_gate_exec_manpage.txt)

---

## Beta Program

AIShell-Gate 1.0 beta is available by request for Unix engineers, DevOps teams, and security engineers. The policy engine, execution gateway, and MCP server are functionally complete.

**[Request beta access →](https://www.aishellgate.com/#beta)**

---

## Channel Partners

Available through a partner program for MSPs, MSSPs, VARs, and system integrators.  
Contact: [partners@aishell.org](mailto:partners@aishell.org)

---

*AIShell Labs LLC · Winston-Salem, NC · [info@aishell.org](mailto:info@aishell.org) · [aishellgate.com](https://www.aishellgate.com)*
