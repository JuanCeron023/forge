# Clean Code Principles

Concrete, actionable rules — not philosophy. Distilled from Kepler, Simplify, and Cleaner Agent.

## Functions
- Cyclomatic complexity ≤ 6 per function
- One function does one thing
- ≤ 30 lines per function (excluding declarations/config)
- ≤ 3-4 parameters per function (use a parameter object or options interface if more are required)
- Names reveal intention: `calculateTotalPrice()` not `calc()`, `isUserAuthenticated()` not `check()`
- Early returns over nested conditionals

## Naming
- Variables: nouns that describe what they hold
- Functions: verbs that describe what they do
- Booleans: `is`, `has`, `can`, `should` prefix
- Constants: UPPER_SNAKE_CASE with context
- No abbreviations except universally understood ones (id, url, api)
- Naming should make comments unnecessary for simple logic

## Structure
- Prefer composition over inheritance
- Interfaces only when they clarify contracts or enable testability
- Don't abstract prematurely — extract when pattern repeats 3+ times
- One level of abstraction per function
- Group related code together, separate unrelated code
- Smallest useful scope for every variable and function

## Error handling
- Handle errors at the appropriate level, not everywhere
- Fail fast with clear error messages
- Don't swallow errors silently
- Clean up resources in all paths (success and error)
- Use typed errors when the caller needs to distinguish error types

## External APIs
- Source before memory: always read official docs before writing integration code
- Copy the official example, then adapt
- Never invent parameter names, endpoint paths, or webhook events from memory
- Pin versions for dependencies

## Tests
- Test behavior, not implementation
- One assertion concept per test
- Descriptive test names: "should return 404 when user not found"
- Tests are documentation: a reader should understand the contract from the tests
- Don't mock what you don't own — use integration tests for real boundaries
- Avoid testing private methods directly

## Anti-patterns to avoid
- God objects / God functions (> 200 lines)
- Boolean parameters that change behavior (use separate functions)
- Magic numbers without named constants
- Deep nesting (> 3 levels)
- Commented-out code (use version control instead)
- Premature optimization without measurement
- Copy-paste with minor variations
