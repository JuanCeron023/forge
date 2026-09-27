# Essential Review Lenses

Focused review passes triggered by what the diff actually changes. Use these during Phase 7 (QA & Review) to catch regressions that automated tests often miss. Run each applicable lens as a focused pass—do not dilute them into a generic, blurry review.

---

## 1. Stale Value Sweep
- **Trigger:** The diff renames or deprecates a value (function, variable, flag, event name, route, or constant).
- **Check:** Run a repository-wide grep for the old name. It must return zero hits in active code and configs.
- **Why:** Renaming only the instances you already remember leaves subtle runtime failures where old references linger.

## 2. Caller & Contract Sweep
- **Trigger:** The diff changes a function's signature, return type, thrown errors, or default parameters.
- **Check:** Locate every caller outside the diff (`git grep` or IDE references) and verify each caller handles the new contract correctly.
- **Why:** Modifying a function's contract without updating all call sites is the #1 cause of unhandled runtime exceptions.

## 3. Surface & Documentation Sweep (Line-Level)
- **Trigger:** The diff modifies user-visible behavior, API contracts, public routes, or environment configuration.
- **Check:** Verify that every related documentation surface (README, API docs, help text, `.env.example`, type definitions) is updated. 
- **Critical rule:** Check *untouched lines* in modified files. A file appearing in the diff is NOT automatically covered if other sections in that same file still describe the old behavior.
- **Why:** Code that changes behavior while documentation describes the old behavior creates immediate developer confusion and support debt.

## 4. Force-Red (Test Teeth)
- **Trigger:** The diff introduces a test meant to prevent a bug or verify a new requirement.
- **Check:** Temporarily revert the implementation fix. Does the test turn red? Restore the fix: does it turn green?
- **Why:** A test that passes without the fix present does not test the defect; it only gives false confidence.

## 5. New Domain & Relaxed Validator Matrix
- **Trigger:** The diff widens a trigger condition or relaxes a validator/guard to accept more inputs.
- **Check:** Re-generate verification cases from the *new* input domain, not just the original bug report. Check the downstream consumers and physical constraints (filesystem name limits, external protocol boundaries, special characters).
- **Why:** Reused downstream helpers often harbor latent assumptions that were only harmless because those inputs were previously unreachable.

## 6. Inverse Regression Surface
- **Trigger:** The diff replaces a parser, matcher, lookup, or data source with a new one.
- **Check:** Ask the inverse question: "Beyond what the new source catches, what valid inputs did the *old* source accept that the new source quietly rejects or erases?"
- **Why:** Testing only what the new parser was designed to fix blinds you to items omitted by the new source.

## 7. Artifact Cleanup Registration
- **Trigger:** The diff starts writing a new persistent file, temporary cache, lock, or state file.
- **Check:** Verify that the file path is explicitly registered in cleanup commands (`clean`, `teardown`, uninstallers, restart handlers).
- **Why:** The writer works and tests pass, but disk state drifts and leaks data across invocations because the cleanup routine was simply unaware of the new file.

## 8. Resolution-Rule Consistency Across Consumers
- **Trigger:** The diff changes how a key, route, framework, or command is resolved or discovered from raw input.
- **Check:** Find all code paths that reconstruct or parse that same input (e.g., error messages, suggestion builders, loggers, environment binders). Verify they use the identical deepened rule.
- **Why:** Updating the main resolver while an error builder or logger re-parses raw input naively produces silent discrepancies and misleading error messages.

## 9. Shell Metacharacter & Subshell Parsing (When touching CLI/Scripts)
- **Trigger:** The diff appends, injects, or scans command strings executed via a shell (`sh -c`, package manager runners, Makefile, npm/bun scripts).
- **Check:** Test constructs that change parsing without being simple separators: `#` (trailing comment that silently discards everything appended), `()` / `$()` (subshells and substitutions), and redirections (`>`).
- **Why:** Appending flags to a command ending in `# comment` causes the flags to be silently swallowed by the comment without producing an error.


