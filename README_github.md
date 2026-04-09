# AIShell-Gate — AI Execution Gateway for Unix

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
- **Flag assessment** — 1,371 individual flag assessments across 188 commands identify dangerous modifiers before execution
- **Network warning** — commands targeting local inference endpoints, cloud metadata services, or inherently dangerous ports are flagged with appropriate escalation
- **Audit log entry** — every decision, whether allow or deny, is recorded in a tamper-evident chain-linked log

---

## The Components

AIShell-Gate ships as four components that work together:

**aishell-gate-policy** — The policy engine. Receives a proposed command, evaluates it, and returns a structured JSON decision. It never executes anything. This is the component that enforces your rules. It is available under a free individual license for personal, non-commercial use.

**aishell-gate-exec** — The execution gateway. Reads a JSON action plan from an AI agent or operator, submits each command to the policy engine, collects the required human confirmation, and — only if the policy allows and the operator confirms — calls `execve()` with the validated argument array. No shell is ever invoked. This component is commercially licensed.

**aishell-confirm** — The operator confirmation relay. For deployments where the operator is not at the same terminal as the AI agent — SSH sessions, remote pipelines, headless servers — aishell-confirm provides a separate confirmation channel. Confirmation requests arrive at the operator's terminal; their response is transmitted back. This component is commercially licensed.

**aishell-gate** — A wrapper script that assembles the policy engine and executor into a single invocation for standard local use.

---

## Policy and Confirmation Levels

Policy is a stack of three layers evaluated in order: **base** (organisational floor), **project** (workflow-specific rules), and **user** (personal preferences). A deny at any layer is final. Built-in presets — `ops_safe`, `dev_sandbox`, `read_only`, `danger_zone` — provide working starting postures without manually assembling policy files.

Confirmation levels are assigned by the policy engine based on risk:

| Level | Meaning |
|---|---|
| `none` | Safe, routine command — proceeds without interruption |
| `plan` | The operator is shown the full plan before any action runs |
| `action` | Each action requires individual acknowledgement |
| `typed` | A random challenge code must be typed to confirm — designed for high-risk, irreversible commands |

An AI agent running with `source: ai` receives additional scrutiny. Known AI inference API ports on loopback are flagged as potential self-access risks.

---

## The Audit Log

Every action evaluated by AIShell-Gate — whether allowed, denied, confirmed, or refused — produces a structured JSON log entry. Entries are chain-linked: each record includes a hash of the previous record, making any post-hoc modification detectable. The chain can be verified with a single command.

The log records the command, the decision, the confirmation level, the matched policy rule and layer, the risk score, the blast radius, and the timestamp. It is designed to be human-readable without a JSON parser, and machine-parseable for integration with existing logging infrastructure.

---

## Who It Is For

AIShell-Gate is designed for **Unix system administrators, DevOps engineers, and security-conscious operators** who are deploying or evaluating AI-assisted shell automation in production or near-production environments. It is equally applicable to teams managing their own AI agents and to individuals who want a formal boundary between AI suggestions and system execution.

It is not a replacement for standard access controls, system permissions, or network security. It is a focused, auditable execution boundary that formalises the AI-to-shell interface.

---

## Beta Access

AIShell-Gate 1.0 is currently in a controlled beta program. The beta testing package — compiled binaries for Debian 11 x86_64, full documentation, and the beta README — is available by request to qualified evaluators.

**To request access:** [www.aishell.org/aishellgate](https://www.aishell.org/aishellgate) or email [info@aishell.org](mailto:info@aishell.org)

Testing should be performed in isolated, non-production environments.

---

## License

**aishell-gate-policy** is available at no charge to individual users for personal, non-commercial, single-operator use. See the LICENSE file for full conditions.

**aishell-gate-exec, aishell-confirm, and aishell-gate** require a commercial license. Fixed-host and cloud/ephemeral deployment licensing is available. Beta evaluation copies expire 30 days from the date of delivery. Source code is available to commercial licensees who have executed a Non-Disclosure Agreement for security review purposes.

License inquiries: [info@aishell.org](mailto:info@aishell.org) · [www.aishell.org/aishellgate](https://www.aishell.org/aishellgate)

---

## Documentation

| Document | Description |
|---|---|
| Getting Started Guide | Local installation, build instructions, first session walkthrough |
| Remote Deployment Guide | SSH and multi-terminal confirmation relay setup |
| White Paper | Architecture, design rationale, security positioning |
| Policy Man Page | Complete policy file reference, rule syntax, preset definitions |
| Executor Man Page | Full flag reference, JSON plan format, audit log specification |

All documentation ships with the beta testing package and is available at [www.aishell.org/aishellgate](https://www.aishell.org/aishellgate).

---

*AIShell-Gate is proprietary software. Copyright © 2026 AIShell Labs LLC, Winston-Salem NC USA. All Rights Reserved. Use requires a valid license. This repository does not contain source code.*
