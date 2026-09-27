# Contract: [ticket/feature name]

## Outcome
One sentence: what is true after this change that isn't true now?

## Acceptance QA Scenarios (Gherkin)
Describe observable behavior (no implementation details):
- [ ] Scenario 1: Happy path
  - Given [precondition / initial state]
  - When [action / event happens]
  - Then [observable outcome / state change]
- [ ] Scenario 2: Edge case / boundary
  - Given [boundary state]
  - When [action with invalid or extreme input]
  - Then [explicit rejection or expected error response]
- [ ] Scenario 3: Error handling / failure mode
  - Given [failure condition, e.g. service down]
  - When [action triggered]
  - Then [expected error response and cleanup]

## Non-Goals
- NG1: [what this change does NOT do]
- NG2: ...

## Invariants (must not break)
- I1: [existing behavior that must be preserved]
- I2: ...

## Edge Cases & Risks
- EC1: [unusual but valid scenario] → expected behavior: [what should happen]
- R1: [risk] → mitigation: [how to handle]

## Change Surface
- Files to modify: [...]
- Files to create: [...]
- Tests to add/update: [...]
- Docs to update: [...]

## Verification
How to verify each scenario:
- Scenario 1: `[test command, curl, CLI run, or script]`
- Scenario 2: ...
