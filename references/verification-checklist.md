# Verification Checklist

A practical checklist for Phase 6 (Verify) and Phase 7 (QA). Use only what is relevant to the change surface.

---

## 1. Test Quality & Falsification (Always)
- [ ] **Falsification at call-site:** Invert or remove the fix logic where the product actually calls it. Does the test turn red for the expected reason?
- [ ] **Restoration:** When restored, does the test reliably pass without flakiness?
- [ ] **Observable behavior:** Do tests verify the real effect (API response, database state, return value), rather than internal mock details?

## 2. Input Robustness & Edge Cases (As applicable)
- [ ] **Empty / Missing:** Null, undefined, empty string `""`, empty list `[]`.
- [ ] **Explicit bounds:** Exact minimum, maximum, and invalid out-of-bound values for numbers, dates, or string lengths defined in business logic.
- [ ] **Special characters:** Quotes, unicode, trailing spaces, or unusual formatting in user text.
- [ ] **Idempotency:** What happens if the same request or event is sent twice?

## 3. Failure Paths & External Boundaries (When touching network, DB, or APIs)
- [ ] **Timeouts & errors:** If the downstream service or database fails/times out, does the system fail safely or crash unhandled?
- [ ] **Resource cleanup:** Are connections, transactions, file handles, or locks closed/rolled back in both success and error paths?

## 4. Security & Performance (When touching sensitive or high-volume paths)
- [ ] **Authorization & Input Safety:** Is authorization validated on the server? Are inputs sanitized against SQL, command, or script injection?
- [ ] **Secrets:** Are credentials, API keys, or tokens kept out of version control and logs?
- [ ] **No N+1 queries:** Are database calls or network fetches batched rather than looped?
- [ ] **No unbounded memory:** Are collections, buffers, or caches constrained to prevent memory leaks?

