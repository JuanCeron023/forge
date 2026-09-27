# Phase 1: Understand

This phase is distilled from the Recon Agent and Work Intake. The goal is to fully understand the request, trace data flow, and map the Change Surface before writing any code.

---

## Adaptive Execution Mode

Choose the execution mode based on your runtime environment:
- **Multi-Agent Runtime (Fleet Mode):**
  - If your environment supports subagent delegation (e.g. Antigravity's `invoke_subagent`, Claude Code subagents, Cursor sub-workers), invoke the specialized **`recon-agent`** subagent (`subagents/recon-agent.md`).
  - Instruct `recon-agent` with the concrete mission prompt: *"Trace `[operation/symptom]` from entry point to outcome and return an anchored Evidence Packet."*
  - Receive the Evidence Packet without polluting your main context with extensive file reads.
- **Single-Agent Runtime (Solo Mode):**
  - Adopt the **Recon Persona** directly in your session.
  - Follow the steps below sequentially while maintaining a strictly read-only discipline.

---

## 1. Read the Request Completely
- Read the full ticket/Jira/idea description carefully.
- Read all comments, linked issues, and pull requests.
- Identify: what is being asked? what is the expected outcome?
- If ambiguous, ask for clarification before proceeding.

## 2. Identify Primary Objective & Scope
Understand the real nature of the work without forcing it into artificial, mutually exclusive boxes:
- **Core Intent:** What is the actual goal? (e.g., fixing unexpected behavior, introducing a capability, refactoring for maintainability, or researching). Real tickets often blend these.
- **Root Trigger:** Why is this being requested now? (e.g., user report, performance bottleneck, technical debt, or workflow blocker).
- **Blast Radius Estimate:** Is this a localized, self-contained fix, or does it cross multiple subsystems, APIs, or database boundaries?

## 3. Explore the Relevant Codebase
- Search for the nouns of the ticket: function names, components, routes, models.
- Locate entry points, data flow, and dependencies.
- Read existing tests that encode current behavior.
- Run cheap commands: `--help`, test list, route table.
- Prefer anchors (`file:line`) over reading whole files.

## 4. Map the Change Surface
- Which files need to change?
- What depends on those files? What do they depend on?
- Are there sibling surfaces (docs, configs, tests) that need updating?
- Prefer shared layers over duplicating changes.

## 5. Classify Every Claim
Every material statement in your analysis must carry an explicit class:
- **`observed`**: You ran it or read it at a specific `file:line` anchor.
- **`inferred`**: Reasoned from observed evidence (state the logical step).
- **`reported`**: Comments, docs, or ticket descriptions say so, but unverified in code.
- **`unknown`**: Couldn't confirm. Never smooth over unknowns—name them explicitly.

Format findings in a concise claims table:
| Statement / Finding | Class | Anchor (`file:line`) | Evidence / Notes |
|---|---|---|---|
| Handshake uses HMAC-SHA256 | observed | auth.py:54 | Read directly from source |
| Worker retries on 503 | inferred | worker.py:120 | Backoff configured, but no retry test found |
| Upstream rate limit is 100 req/s | reported | README.md:12 | Unverified against external gateway |
| Lock timeout duration under load | unknown | None | Needs load benchmark to determine |

## 6. Deliver the Investigation Packet (For Path C)
When operating under **Path C (Investigation & Spike Flow)**, do NOT proceed to Coder, Cleaner, or QA:
- Deliver an **Evidence Packet**:
  1. **Core Question Answered:** Direct, unambiguous response to the question or diagnostic hypothesis.
  2. **Code Anchors:** Traced hops through the codebase with exact `file:line` locations.
  3. **Observed vs Inferred Facts:** Clear claims table separating what was directly proven from what is reasoned or unknown.
  4. **Actionable Recommendation:** Explicit verdict: *Resolve as answered*, *Reject as non-viable*, or *Promote to implementation* (defining the proposed outcome for Path A or Path B).

## Complete when:
- **For Path A (Product/Feature):** You can explain the ticket scope, relevant code, change surface, and unknowns before defining the contract (Phase 2).
- **For Path C (Investigation):** The Evidence Packet is delivered with anchored facts and an actionable recommendation, concluding the task without editing code.
