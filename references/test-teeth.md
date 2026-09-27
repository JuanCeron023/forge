# Test Teeth & Mutation Mindset Reference

A guide for writing and auditing tests that provide genuine behavioral protection, not just empty code coverage metrics. 

---

## 1. Core Principle

> **"Execution is not protection. A test protects nothing until you have observed it turn red against the specific defect it claims to prevent."**

Green test suites frequently offer a false sense of security. Common anti-patterns include:
- Asserting trivial facts (e.g. `expect(response).not.toBeNull()`).
- Testing mocked implementations against mocks of themselves (mock tautology).
- Testing internal helper functions while leaving the application wiring at the call site untested.
- Asserting only what the developer's code *currently returns*, rather than what the specification *requires*.

---

## 2. Falsification Protocol (The Hardener Method)

Whenever you add or modify a test, you MUST prove it has "teeth" by intentionally breaking the code:

1. **Introduce a minimal semantic fault:**
   - Invert a boolean check: `if (isValid)` $\rightarrow$ `if (!isValid)`.
   - Off-by-one error: `price * count` $\rightarrow$ `price * (count - 1)` or `>` $\rightarrow$ `>=`.
   - Remove an authorization check or filter clause.
   - Force a hardcoded return or empty list.
2. **Execute the test suite:**
   - The test **MUST turn RED**.
3. **Verify the reason for failure:**
   - The failure message must point directly to the broken business rule or invariant.
   - A failure caused by syntax errors, compilation failure, or broken test setup **does not count** as a successful falsification.
4. **Restore the code:**
   - Re-run the suite and ensure it returns to **GREEN**.

---

## 3. Test at the Call Site, Not Only the Helper

A test that calls an internal utility directly cannot prove the system works.

$$\text{User Request} \longrightarrow \mathbf{[\text{Call Site / Controller / Pipeline}]} \longrightarrow \text{Helper Logic} \longrightarrow \text{Database}$$

- **The Risk:** If you test `calculateTax()` in isolation, but the route handler never passes the tax to the database save call, the tax calculation is broken in production even though unit tests are 100% green.
- **Rule:** Always have at least one test exercising the complete path from the caller/entry point through the call site.
- **When definition and call site differ:** If a bug was reported at the call site, falsify the fault at the call site to prove the wiring is guarded.

---

## 4. Mutation Testing Mindset

Treat every statement in the diff as a candidate for mutation:

| Original Code | Mutation to Mentally Test | Does an Existing Test Fail? |
|---|---|---|
| `if (user.role == ADMIN)` | `if (user.role != ADMIN)` | Must fail with 403 Forbidden |
| `order.total = subtotal + tax` | `order.total = subtotal` | Must fail with calculation error |
| `retryCount <= MAX_RETRIES` | `retryCount < MAX_RETRIES` | Must fail on edge boundary |
| `WHERE status = 'ACTIVE'` | *clause omitted* | Must fail by returning inactive rows |
| `timeout = 5.0` | `timeout = 0` / infinite | Must fail with timeout assertion |

If a mutant survives (i.e., you can delete or change business logic and the test suite remains green), that logic is **unprotected**. Add an assertion or test case to kill the mutant.

---

## 5. Behavioral Modeling: Dimensions Before Branches

When testing parsers, serializers, state machines, or complex business logic:
- **Do NOT derive test cases by reading your implementation's `if/else` statements.** If your implementation missed a branch, your tests will miss it too.
- **Derive dimensions from the external contract or domain rules first:**

```markdown
### Dimension Table for Order Checkout
| Dimension | Values |
|---|---|
| User State | Guest, Authenticated, Suspended |
| Inventory | In Stock, Low Stock (Race), Out of Stock |
| Coupon | Valid, Expired, Already Used, Minimum Spend Unmet |
| Payment | Instant Success, 3DS Required, Insufficient Funds, Gateway Timeout |
```

Combine dimensions to test realistic combinatorial states, rather than testing a single happy-path example.

---

## 6. Avoiding Mock Tautology & Brittle Tests

- **Mock Boundaries, Not Internal Logic:**
  - Mock third-party network APIs (Stripe, Twilio, S3) and slow hardware.
  - Do NOT mock domain entities, value objects, utility functions, or database repositories when an in-memory SQLite/Postgres container is viable.
- **Verify the Effect, Not the Artifact:**
  - Bad: `expect(emailService.send).toHaveBeenCalledWith(...)` (verifies an implementation detail).
  - Good: Call the endpoint, inspect the database or outbox table, and verify the message was queued with correct payload.
- **Don't Assert Non-Null When You Expect a Specific Value:**
  - Weak: `assert result is not None`
  - Strong: `assert result.status == OrderStatus.PAID and result.total_cents == 1500`
