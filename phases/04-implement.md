# Phase 4: Implement

This phase is distilled from the Coder Agent (`subagents/coder-agent.md`). Write the code cleanly with an unbroken evidence chain. In Multi-Agent mode, you may delegate implementation tasks to `coder-agent`.

## 1. Reuse Before Reinventing
- Search for existing utilities, helpers, patterns in the codebase.
- Check for existing components, layouts, shared services.
- A shared layer beats N copies: fix in the token, not each component; analytics in the layout, not each page.

## 2. Source Before Memory
- Any code touching an external API: read the official docs first.
- Never code from memory — that's where 404 URLs, invented parameters, and wrong webhook events come from.
- Copy the official example, then adapt.

## 3. Write Clean Code
- Read `references/clean-code.md` for the full principles.
- Cyclomatic complexity ≤ 6 per function.
- One function does one thing.
- Names reveal intention.
- No premature abstractions.
- Interfaces only when they clarify contracts or enable testing.

## 4. Add Appropriate Tests
- Unit tests for business logic and edge cases.
- Integration tests for boundaries (DB, API, filesystem).
- Tests describe observable behavior, not implementation details.
- Tests should fail if the behavior breaks (this is verified in phase 6).

## 5. Maintain the Evidence Chain
- Verify the **effect**, not the artifact: computed style over "the class is in the CSS"; real API response over "the env var is set"; actual rendered output over "the component exists"
- After **2 failed attempts** on the same symptom, stop iterating blind: add instrumentation, read the error trace carefully, or read the working reference implementation. Blind iteration is how 30-minute fixes become 3-hour rabbit holes.
- Every material claim about the code has a class: observed or inferred.
- "Tests pass" is a claim — the command output is the evidence.
- If the environment blocks verification, report the gap explicitly — never let it become a silent pass.

## 6. Authority Boundary
- NEVER push to main, publish, release, change repo visibility, or message third parties
- Stage the work and report
- Never widen write scope beyond what was granted
- Report format: what changed (files), evidence observed (command output), exemptions/assumptions (each with evidence), out-of-scope findings (title + evidence + why out of scope)
- Implement only what the contract defines.
- Don't expand scope, don't refactor unrelated code.

## Complete when
The implementation exists, tests pass (with observed command output), the code follows clean code principles, and every decision has a reason. "Should work" is not a valid state.
