# Contract

This phase is distilled from Issue Contract and Solution Gate's contract freezing. Define WHAT to build before thinking about HOW.

## 1. Define the Outcome
- One sentence: what is true after this change that isn't true now?
- Frame it as a violated property and desired outcome, not symptoms.

## 2. Define Acceptance QA Scenarios (Specifier Role)
Define the concrete scenarios that will prove the work is complete. This is a **conceptual reasoning tool**, completely independent of whether a specific testing library (like Cucumber or Gherkin runners) is installed:
- Format each scenario conceptually as:
  - **Context:** Given [initial system state or precondition]
  - **Action:** When [event, action, or input occurs]
  - **Expected Result:** Then [observable outcome, return value, or state change]
- Scenarios must describe **observable behavior**, not internal code mechanics (no private variable names or database table internals).
- Cover:
  - Happy path (primary expected use case).
  - Key edge cases and invalid inputs.
  - Failure/error modes (e.g., service unavailable, malformed payload).
- **Execution flexibility:** Record these scenarios directly in the contract, a scratch test file, or the session notes. They can be executed later via existing test suites (Jest/Pytest/Go), CLI commands, API calls, or ad-hoc verification scripts.

## 3. Define Non-Goals
- What this change explicitly does NOT do.
- What's out of scope even if related.
- This prevents scope creep during implementation.

## 4. Define Invariants
- Existing behavior that MUST NOT break.
- Working workflows that cross the same code.
- Performance characteristics that must be preserved.

## 5. Identify Risks and Edge Cases
- What could go wrong?
- What inputs are unusual but valid?
- What happens under load, with bad data, with timeouts?
- What assumptions are we making? Mark each as `verified` or `assumed`.

## Complete when:
A clear contract exists with: single-sentence outcome, conceptual QA scenarios, non-goals, preserved invariants, and identified edge cases. Any developer or agent reading this contract understands exactly what must be built and how it will be verified without relying on prior conversation context.
