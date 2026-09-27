# Handoff

The handoff exists so the next session resumes without rereading the transcript. Its value is not the summary — it is **the distance between what the conversation believes and what the disk says**.

## Before writing: verify against git

Run these commands BEFORE writing a single line. This is pure reading, it touches nothing.

```bash
git status --short
git log --oneline -15
git status -sb | head -1          # ahead/behind
```

Contrast every claim from the conversation against that output. Disagreements are the most valuable part of the handoff: "the conversation said X, the disk says Y."

Classify every material claim:
- **Recoverable**: someone can rerun the command and see the same thing (SHA, PR number, output)
- **Reported**: stated in conversation but nobody observed it run
- **Inferred**: deduced from something else. Code looking right ≠ code working
- **Unknown**: couldn't confirm

Reported and inferred don't become facts by repetition. Unknown is not omitted — it's named.

## Before saving: check for these 4 failures

1. **Numbers without provenance.** Every number carries how it was obtained. "119 silences detected, measured before writing the parser" works; "many silences" doesn't.
2. **"Done" without observed evidence.** Committed ≠ verified. Code looking right ≠ verification. If nobody watched it run, say so.
3. **Chronology instead of state.** "First we did X, then Y" is what git log already tells. The handoff reports state, not narrative.
4. **Pending documented instead of closed.** If something can be fixed in 2 minutes now, fix it instead of writing it down.

## Template

# Handoff: [project/ticket name]

Date: [YYYY-MM-DD]
Phase / Step completed: [Current Phase / Role]
Entry Path: [Path A: Jira/Product | Path B: Direct Code]

## Current State
What is actually done RIGHT NOW (verified, not assumed):
- Delivery: [local / committed / pushed / merged / released]
- Verification: [unverified / partially verified / fully verified]
- Human review: [pending / approved]

### Git Verification (Command Outputs)
- `git status --short`: [output or "clean"]
- `git log --oneline -1`: [SHA + commit message]

### Claim Classification
- **Recoverable (Observed):** [fact with command/evidence that proves it]
- **Reported/Inferred (Unverified):** [claim stated but not independently verified]
- **Unknown/Unconfirmed:** [couldn't confirm]

## Where to Resume
In priority order (if #1 fails, it invalidates #2+):
1. [Next concrete action with the exact command or file to open]
2. ...

## Decisions Made
### Settled (don't re-litigate)
- [decision]: [reason]

### Pending human judgment
- [decision needed]: [options and trade-offs]

## Out of Scope Findings
- [issue found but not addressed, with evidence]
