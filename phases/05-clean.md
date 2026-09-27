# Phase 5: Clean (Cleaner Role)

**Goal:** Review the implementation without changing its intended behavior. Make the existing solution clearer, simpler, and easier to maintain.

> [!IMPORTANT]
> The Cleaner **must not expand the scope or redesign the feature**. Its job is strictly to refine what was already built, not to invent new abstractions or add capabilities.

---

## 1. The Cleaner Checklist
Inspect the modified and newly created code for:
- [ ] **Unclear naming:** Do variable, function, and class names immediately reveal their intention?
- [ ] **Poor function boundaries:** Is each function focused on doing one single thing well?
- [ ] **Unnecessary complexity:** Can convoluted logic, deeply nested conditionals, or loops be simplified?
- [ ] **Duplication:** Is there obvious, copy-pasted logic that should be extracted?
- [ ] **Incorrect or inappropriate patterns:** Are language idioms and project conventions respected?
- [ ] **Excessive abstractions:** Remove speculative wrappers, premature generic types, or indirection that adds no concrete value.
- [ ] **Difficult-to-understand control flow:** Use early returns to eliminate nested `if/else` ladders.
- [ ] **Useful interfaces:** Add interfaces only where they genuinely clarify contracts, responsibilities, or make dependencies easier to test.
- [ ] **Cyclomatic complexity > 6:** Any function with complexity $> 6$ MUST be broken down into well-named, small helper functions.

---

## 2. Verify Zero Behavior Change
- Run the full test suite after any cleanup edits.
- The diff must reflect structural improvements only—zero functional changes.
- If any test breaks or behavior shifts, revert that specific edit immediately.

---

## Complete when:
- The code passes the 9-point checklist.
- All functions satisfy **cyclomatic complexity $\le 6$**.
- All tests continue to pass with zero regressions.
