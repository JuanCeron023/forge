# Phase 6: Verify

**Goal:** Ensure the changes work as intended, fail explicitly and safely when expected (without masking real errors with artificial fallbacks), and meet performance and security requirements.

**Execution Scope:**
- **Path B (Direct Code Flow):** Steps 1, 2, and 4 (checks, falsification, and failure safety).
- **Path A (Jira / Product Flow):** Steps 1 through 6 (checks, falsification, edge cases, failure safety, security, and performance).
- **Step 7 (Mutation / Property testing):** Prioritize when modifying critical business rules, state machines, or security-sensitive algorithms.

---

## 1. Run All Checks
- Execute the full test suite (unit, integration, e2e) for the affected components.
- Run linters, formatters, and static analysis tools.
- Address any warnings or errors introduced by the changes.
- Ensure test coverage metrics meet or exceed the project's threshold for the modified code.

## 2. Falsify the Protection (Hardener Agent & Test Teeth)
A test suite that never fails protects nothing. See `references/test-teeth.md` for the complete guide.
- **Reversible break:** Invert a condition, remove the fix, or force an error in the modified logic. The corresponding test MUST turn red.
- **Fail for the right reason:** Ensure failure is caused by the broken business rule, NOT by a syntax error, compile failure, or broken setup.
- **Falsify at the call site:** A unit test testing an internal helper directly does not prove the application wiring works. Always ensure verification covers the actual call site the system executes.
- **For complex parsers, protocols, or state machines:** Define behavioral dimensions (inputs, modes, prior states, and expected post-states) against an independent specification or reference oracle, rather than copying expectations from the implementation's own branch structure.

## 3. Exercise Edge Cases
- Test boundary conditions (empty inputs, maximum sizes, null values, malformed data).
- **Real boundary testing:** Use integration tests for package, database, process, protocol, filesystem, or browser boundaries. Browser-generated events, real serialized bytes, and process signals outrank hand-built mock objects.
- Validate state transitions that are rare but possible.
- Verify that incorrect types or formats are rejected with explicit, expected errors.

## 4. Verify Failure Paths & Resilience (Backend Distributed Systems)
For operations crossing process, database, API, queue, or filesystem boundaries, see `references/resilience-backend.md`:
- **Map the failure boundary:** Identify owner, timeout, cancellation path, retry policy, idempotency, and durable state cleanup.
- **Split at the commit point (Crucial Rule):** For operations with durable side effects (database writes, charges, messages):
  - Force failure *before* the commit point $\rightarrow$ verify state was not mutated and resources are freed.
  - Force failure *immediately after* the commit point (ambiguous success) $\rightarrow$ verify that subsequent retries do NOT corrupt state, duplicate records, or double-charge. *Rule: Writer-local cleanup is not evidence for caller-level rollback.*
- **Force one fault at a time:** Test timeouts during active work, network disconnections, or partial payloads using controlled mocks or test doubles.
- **Fail explicitly when intended:** Never invent artificial fallback paths, synthetic backups, or silent defaults when the correct architectural behavior is to reject the operation and surface the failure clearly.
- **Classify the outcome:** Confirm whether the result is:
  - `preserved_invariant` / `expected_rejection`: Ideal behavior (operation either succeeds cleanly or fails with a clear, handled error without corrupting state).
  - `corrupted_state` / `leak` / `retry_storm` / `silent_fallback`: Regression that must be fixed.

## 5. Security & Trust Boundaries
Trigger when the diff touches authentication, authorization, user inputs, secrets, file operations, queries, or external communication. See `references/security-audit.md`:
- **Map the trust model:** Identify assets, actors, principals, trust boundaries, entry points, and security-sensitive sinks.
  - *Authority principle:* For every principal, record the authority it already has. A secret reaching a process is NOT automatically an exposure if that process already executes with equivalent authority or read permissions.
- **Build the exploitability chain:** Before claiming a vulnerability, trace the complete link:
  $$\text{attacker capability} \longrightarrow \text{reachability} \longrightarrow \text{attacker control} \longrightarrow \text{boundary crossed} \longrightarrow \text{security impact}$$
  Ensure each link is anchored in actual code or direct observation. Missing links remain explicit gaps—never invent deployment assumptions to finish a chain.
- **Classify findings & separate scope:**
  - `confirmed_vulnerability` / `likely_vulnerability`: Proven exploitability chain crossing a trust boundary (Blocker).
  - `hardening`: General defensive improvement (e.g., stricter headers, proactive validation), but NOT an exploitable blocker.
  - `in_scope_regression`: A vulnerability introduced by the diff itself (MUST block merge).
  - `out_of_scope_finding`: Pre-existing or adjacent security improvement (document clearly, but do not block the current ticket).

## 6. Performance Check (When touching high-volume or critical paths)
Avoid premature optimization or theoretical data structure comparisons:
- **Measure before optimizing:** Profile with real workloads or benchmarks. Do not guess bottlenecks.
- **Inspect realistic pain points:** Check for N+1 queries, unindexed lookups, blocking I/O inside loops, redundant serialization, or unbounded memory/cache growth.
- **Compare distributions honestly:** Compare baseline vs new implementation across multiple runs. If performance gains are within measurement noise or add excessive code complexity, prefer the simpler, clearer code.

## 7. Mutation and Property Testing
- **Mutation testing:** For changed or critical logic, run mutation campaigns (or mental mutations). Treat surviving mutants in critical logic as leads. Focus on evaluating test quality over chasing a universal mutation percentage.
- **Property-based testing:** Implement property-based tests for invariants and equivalence classes to discover edge cases that example-based testing might miss.

---

## Complete when:
- All checks and tests for the appropriate profile level pass.
- Falsification proves tests fail when the guarded logic is broken.
- Failure paths, security boundaries, and resilience invariants have been verified or documented.
