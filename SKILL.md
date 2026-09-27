---
name: forge
description: "Orchestrate an end-to-end senior software engineering workflow for any ticket, feature, refactor, or investigation. Runs 7 sequential phases: Understand → Contract → Design → Implement → Clean → Verify → Review. Features Adaptive Execution (Multi-Agent Fleet Mode via recon-agent/architect-agent/coder-agent/reviewer-agent or Single-Agent Solo Mode), backend security audits, distributed resilience testing, and mutation-verified tests."
---

# Forge: The Master Software Engineering Workflow

One unified skill to take any ticket, feature, refactor, or investigation from requirements to thoroughly reviewed, production-grade implementation. Every phase defines strict entry, exit, and evidence criteria, distilling recon, implementation, adversarial review, backend security, resilience, and falsification into a standalone master skill.

---

## 🚀 Adaptive Execution: Multi-Agent Fleet vs Solo Mode

This skill automatically adapts to the capabilities of the host environment:

| Capability | Multi-Agent Fleet Mode (Supported Runtimes) | Single-Agent Solo Mode (Standard Runtimes) |
|---|---|---|
| **Phase 1: Recon** | Delegates to **`recon-agent`** (`subagents/recon-agent.md`) to explore read-only and return an Evidence Packet without bloating parent context. | Adopts the Recon persona in-session with strict read-only discipline. |
| **Phase 3: Design** | Delegates to **`architect-agent`** (`subagents/architect-agent.md`) for dual analysis, trade-offs, and Mermaid diagram. | Generates architecture options and visual Mermaid diagram. |
| **Phases 4-5: Code & Clean** | Delegates to **`coder-agent`** (`subagents/coder-agent.md`) for scoped implementation with cyclomatic complexity $\le 6$. | Executes Coder & Cleaner phases adhering strictly to clean code standards. |
| **Phase 7: Peer Review** | Delegates to **`reviewer-agent`** (`subagents/reviewer-agent.md`) on a **higher model tier** (e.g., Opus/Pro) with clean context to break confirmation bias. | Shifts into the Adversarial Reviewer persona, aggressively challenging the diff. |

*Rule:* If subagent tools (`invoke_subagent`, Claude Code subagents, Cursor background workers) are present, use Fleet Mode for maximum speed and unbiased verification. If unavailable, proceed smoothly in Solo Mode—zero broken dependencies.

---

## 🎯 Three Entry Paths

Choose the entry path matching the nature of the request:

### Path A: Jira / Product Ticket Flow (Business features, user stories, complex bugs)
Use when the request is a business requirement or user ticket where behavior and architecture must be agreed upon first:
$$\text{Recon (01)} \longrightarrow \text{Specifier (02: Conceptual Scenarios)} \longrightarrow \text{Architecture (03: Mermaid)} \longrightarrow \text{Coder (04)} \longrightarrow \text{Cleaner (05)} \longrightarrow \text{Hardener (06)} \longrightarrow \text{QA (07)}$$

### Path B: Direct Code Flow (Technical refactors, localized bug fixes, direct developer tasks)
Use when the problem and code location are already pinpointed and do not require product ceremony:
$$\text{Coder (04: Implement)} \longrightarrow \text{Cleaner (05: Simplify)} \longrightarrow \text{Hardener (06: Falsify)} \longrightarrow \text{QA (07: Review)}$$
*(Note: If a direct code task alters system boundaries or component interfaces, invoke Architecture (03) first to draw the Mermaid diagram).*

### Path C: Investigation & Spike Flow (Root cause analysis, feasibility spikes, architectural audits)
Use when the request is exploratory, diagnostic, or research-only without implementation:
$$\text{Recon (01: Frame & Map)} \longrightarrow \text{Evidence Packet (Anchored Facts)} \longrightarrow \text{Actionable Verdict / Path A or B Recommendation}$$
- **Strictly read-only:** Zero production code modifications. Do not invoke Coder, Cleaner, or QA.
- **Anchored evidence:** Every claim is tied to `file:line` locations and classified (`observed`, `inferred`, `reported`, `unknown`).

---

## 📋 Phase Overview & Engineering Roles

| Phase | Engineering Role | Subagent Spec | What it does | Complete when |
|---|---|---|---|---|
| **01-Understand** | **Recon** | `subagents/recon-agent.md` | Investigates code, traces data flow, isolates unknowns. | Change surface anchored with `file:line`. |
| **02-Contract** | **Specifier** | — | Defines observable behavior in **conceptual QA scenarios**. | Acceptance scenarios, non-goals, and invariants frozen. |
| **03-Design** | **Architecture** | `subagents/architect-agent.md` | Evaluates options; produces **Visual Architecture Diagram**. | Ground-truth Mermaid diagram approved; no overengineering. |
| **04-Implement** | **Coder** | `subagents/coder-agent.md` | Implements logic; cyclomatic complexity $\le 6$; source-before-memory. | Tests pass; verified on real effect, not mere artifact. |
| **05-Clean** | **Cleaner** | `subagents/coder-agent.md` | Refactors for clarity and simplicity without expanding scope. | Clean code; cyclomatic complexity $\le 6$; zero behavior change. |
| **06-Verify** | **Hardener** | — | Falsification at call-site; failure matrix; security; resilience. | Mutants killed; failure paths and security verified. |
| **07-Review** | **QA & Review Gate** | `subagents/reviewer-agent.md` | Validates QA scenarios; checks architecture; diff lenses. | All scenarios pass; deterministic checks clean; ready to ship. |

---

## 🛡️ Core Principles

1. **Evidence over assumptions:** Run it. Read it. Verify it. Classify every claim as `observed`, `inferred`, `reported`, or `unknown`.
2. **Understand before modifying:** Map the change surface and dependencies before editing code.
3. **Design before coding:** Challenge architecture with a visual Mermaid diagram. Build for actual scale, not hypothetical millions.
4. **Reuse before reinventing:** Search for existing utilities and patterns. A shared layer beats $N$ copies.
5. **Source before memory:** For external APIs and libraries, read official documentation first. Never code from stale memory.
6. **Small functions, clear names:** Cyclomatic complexity $\le 6$ per function. One function does one thing.
7. **Commit-point resilience:** Split fallible operations at the commit point. Verify caller retry safety and idempotency.
8. **Test teeth & falsification:** A test that never fails protects nothing. Invert conditions and confirm the test turns red.
9. **Security is boundary defense:** An attacker gaining unauthorized capability across a trust boundary is a vulnerability. Prove the exploitability chain.
10. **Fresh-eyes review:** Deterministic checks first, judgment second. Review through focused lenses on a clean context.

---

## 📚 References & Guides

The skill contains actionable engineering reference manuals in `references/`:
- **`references/clean-code.md`**: Complexity $\le 6$, naming rules, function sizing, and error handling.
- **`references/security-audit.md`**: Threat modeling, SQL/ORM injection, tenant isolation (BOLA), SSRF, secret hygiene, and exploitability chains.
- **`references/resilience-backend.md`**: Distributed resilience, commit-point split, exponential backoff with jitter, retry storms, circuit breakers, and bounded resource pools.
- **`references/test-teeth.md`**: Falsification testing, call-site testing, mutation mindset, and behavioral dimension modeling.
- **`references/review-lenses.md`**: Sibling surfaces, stale values, and caller sweep catalogs.
- **`references/failure-patterns.md`**: Common distributed and state pitfalls.
- **`references/verification-checklist.md`**: Quick verification checklist for PRs.

---

## 🔄 Strict Failure & Recovery Loop

If QA (Phase 7) fails or review checks catch defects, never patch casually in place:
$$\text{QA / Review Defect} \longrightarrow \text{Coder (Phase 4)} \longrightarrow \text{Cleaner (Phase 5)} \longrightarrow \text{Hardener (Phase 6)} \longrightarrow \text{QA (Phase 7)}$$

If implementation thrashes ($2+$ reversals on the same behavior), the underlying design is flawed $\longrightarrow$ Return to **Design (Phase 3)**.

---

## 📝 Handoff Protocol

When a session ends before completing all phases, generate a handoff artifact using `templates/handoff.md`:
- Current phase and completed items.
- Explicitly verified facts vs pending assumptions.
- Architectural decisions made and reasons why.
- Next immediate action to resume work.
