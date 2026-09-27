# Phase 7: Review Gate & QA

This phase is distilled from the Reviewer Agent and Review Gate. It acts as the final quality and correctness gate before a change is shipped.

The core principle: **Deterministic checks first, judgment second**. Automated checks have full recall on their class—exhaust them before spending human judgment.

---

## Adaptive Execution Mode

Eliminating confirmation bias is essential for senior-grade reviews:
- **Multi-Agent Runtime (Fleet Mode - Recommended):**
  - If subagents are supported (`invoke_subagent`, Claude Code subagents, Cursor sub-workers), invoke the specialized **`reviewer-agent`** subagent (`subagents/reviewer-agent.md`).
  - **Fresh Eyes Principle:** Configure `reviewer-agent` with a **higher model tier** (e.g., Opus or Pro) and a clean context containing only the diff, conventions, and contract. The authoring agent must NOT review its own code.
  - Receive Reviewer Agent's structured review report. If blockers exist, route back to Coder (Phase 4).
- **Single-Agent Runtime (Solo Mode):**
  - Explicitly shift your persona to the **Adversarial Reviewer Agent**.
  - Detach emotionally from the code you just wrote. Assume defects exist and actively attempt to find edge cases that break the diff.

---

## 1. Deterministic Checks
Run everything the repo defines before doing any manual reading:
- Lint and format tools
- Static type checking (`tsc`, `mypy`, `go vet`)
- Full automated test suite
- Build / compilation checks
- Pre-commit hooks or local CI validation scripts

*Every automated check finding must be fixed or acknowledged with evidence. Never skip silently.*

## 2. Sibling Surface Sweep
For every file the diff touches, search for sibling surfaces:
- Does the feature appear in docs, README, help text, config files?
- If you renamed something, grep repo-wide for the old name—**zero hits required**.
- If you changed behavior, check every documentation surface that mentions it.
- Check tests: are there tests that still encode the OLD behavior?

*A file already in the diff is NOT automatically covered—check if UNTOUCHED lines in that file reference the changed behavior.*

For the complete catalog of essential review lenses, see `references/review-lenses.md`.

## 3. Caller and Dependency Sweep
If the change modified a function's contract (new parameter, new error, changed return type or semantics):
- Find ALL call sites outside the diff across the entire repository.
- Verify each handles the new contract correctly.
- If unhandled call sites remain, report them as blockers or document the explicit migration path.

## 4. Review the Diff Holistically Through Focused Lenses
Run separate passes over the diff using our backend engineering guides:
- **Security Lens (`references/security-audit.md`):** Verify parameterized queries, tenant isolation (`tenant_id`), authentication/authorization checks, SSRF, and secret sanitization.
- **Resilience Lens (`references/resilience-backend.md`):** Verify timeouts on all network/DB I/O, commit-point split invariants, exponential backoff, and idempotent retry safety.
- **Test Teeth Lens (`references/test-teeth.md`):** Confirm tests test real behavior at the call site and kill deliberate mutants.
- **Clean Code Lens (`references/clean-code.md`):** Cyclomatic complexity $\le 6$, clear naming, no dead code or leftover debugging logs.

## 5. Validate QA Scenarios & Architecture Integrity (QA Role)
- **Execute every QA scenario** established in the Phase 2 Contract (`templates/contract.md`):
  - Run the automated test suite if tests were authored.
  - If no framework exists, test pragmatically: execute CLI commands, send API payloads, or run temporary scratch scripts to observe the actual output.
  - Verify happy paths, edge cases, error handling, and realistic system conditions.
- **Verify against the Architecture Diagram (Phase 3):** Does the code touch only the components and data flows declared in the diagram? Flag any undocumented side effects or unintended dependencies.
- **Verify the EFFECT, not the ARTIFACT:** computed style over "class is in CSS", real HTTP response over "env var is set", live output over "code looks right".
- **Refute findings at the layer of the claim:** A unit test of a helper does not refute an ordering defect at the caller.

## 6. Strict Failure Loop (If QA or Checks Fail)
Never patch code casually during review:
1. Report the exact failing scenario or check, accompanied by verbatim command output/evidence.
2. Route the defect back through the formal pipeline:
   $$\text{QA / Review} \longrightarrow \text{Coder (Phase 4)} \longrightarrow \text{Cleaner (Phase 5)} \longrightarrow \text{Hardener (Phase 6)} \longrightarrow \text{QA (Phase 7)}$$
3. Repeat this loop until all required scenarios pass and the resulting code satisfies clean code standards.

## 7. Report Findings
Structure findings by severity:
- **Blockers:** Must fix before promotion or merge.
- **Improvements:** Should fix, but do not block delivery.
- **Notes:** Informational observations for future tickets.

Always include:
- **Exemptions claimed:** Each with its one-sentence verifiable evidence.
- **Out-of-scope findings:** Real issues discovered outside the diff's scope (do not let them die in conversation—document them clearly).

---

## Complete when:
All deterministic checks pass cleanly. All QA scenarios pass with observed evidence. The code faithfully matches the Architecture Diagram. No open blockers remain. The change is ready for human promotion.
