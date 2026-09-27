# Common Failure Patterns

Distilled from Solution Gate's failure-shapes.md and 100+ real engineering cases. Watch for these during Design (phase 3) and Review (phase 7).

## Design-Phase Failures

### S1: Symptom fix, not root cause
The fix addresses what the user reported, not the underlying mechanism. The symptom reappears under different conditions.
**Guard:** trace the failure to its mechanism. If the fix doesn't address the mechanism, it's a band-aid.

### S2: Primitive-contract mismatch  
The solution uses a primitive (string, number, boolean) where the problem domain requires a richer contract (enum, typed object, state machine).
**Guard:** does the chosen type prevent invalid states, or just represent them?

### S3: Missing state transition model
Persistent state (sessions, caches, queues) is added without defining: who owns it, what clears it, what happens on restart, what happens on conflict.
**Guard:** for each new persistent artifact, answer: lifetime, cleanup, restart behavior, conflict resolution.

### S4: Assumed environment
The fix works on the developer's machine because of an implicit dependency (installed tool, running service, OS feature) not documented or checked.
**Guard:** what does the code assume is available? Is that assumption checked or documented?

### S5: Silent fallback hides failure
An error is caught and silently defaults, returning fake data or masking the failure, making the problem invisible to the user and caller.
**Guard:** If an operation fails, let it fail explicitly with a clear, handled error. Never introduce artificial backup paths or silent defaults when it is correct for the operation to fail.

## Implementation-Phase Failures

### S6: Merged with stale base
The branch was cut weeks ago. Invariants that landed after the branch was cut are not reflected.
**Guard:** check what landed on main since the branch point. Do those changes affect the same subsystem?

### S7: Test encodes the defect
The test passes because it asserts the wrong expected value — the expected value IS the bug.
**Guard:** derive expected values from an independent oracle (spec, reference implementation), not from the current code.

### S8: Mock doesn't match reality
A mock returns success, but the real service has different error codes, pagination, rate limits, or field shapes.
**Guard:** validate mock behavior against the real service at least once. Use recorded responses when possible.

## Review-Phase Failures

### S9: File-level coverage, not line-level
A file appears in the diff, so it's considered "covered" — but untouched lines in that file still reference the old behavior.
**Guard:** for each file in the diff, grep for the changed behavior on lines NOT in the diff.

### S10: Verified on artifact, not behavior
The CSS class is in the compiled output, but nobody checked if the element actually looks right. The env var is set, but nobody checked if the service uses it.
**Guard:** verify the EFFECT, not the ARTIFACT. Computed style over "class is present." Real response over "env var is set."

### S11: "Listo" without evidence
"Done" was declared based on: code looks right, tests should pass, it should work. None of these is evidence.
**Guard:** "done" requires observed command output. Committed ≠ verified. Green tests ≠ correct behavior.
