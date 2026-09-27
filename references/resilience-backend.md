# Backend Resilience & Fault Tolerance Reference

A pragmatic engineering guide for designing, verifying, and reviewing resilient distributed systems and backend services.

---

## 1. Core Principle

> **"A happy-path test says nothing about failure behavior. You must force each material failure and observe that system invariants hold."**

When software crosses process, network, queue, database, or disk boundaries, failure is inevitable. Resilient software guarantees deterministic degradation and zero data corruption.

---

## 2. The Commit-Point Split (Critical Rule)

For every operation with durable side effects (database writes, credit card charges, outbound webhooks, queue messages), split the execution path at the **commit point**:

$$\text{Preparation} \longrightarrow \mathbf{[\text{Commit Point}]} \longrightarrow \text{Post-Commit Notifications}$$

### Phase A: Failure Before Commit
- Force a simulated network failure, database timeout, or unhandled exception *before* transaction commit.
- **Invariant:** The transaction rolls back cleanly. No partial records remain. Acquired locks or temporary files are immediately freed.

### Phase B: Failure At or Immediately After Commit (Ambiguous Success)
- Force failure *after* the database commit succeeds, but *before* the HTTP response or acknowledgment reaches the client (or during subsequent cache updates / webhook dispatches).
- The client receives an error/timeout and will naturally retry.
- **Invariant:** Retrying the identical request with the same idempotency key **must NOT**:
  - Insert duplicate records or double-charge a user.
  - Corrupt state or panic due to unique constraint violations.
- **The Golden Rule:** *Writer-local cleanup is not evidence for caller-level rollback.* Verify caller retry behavior directly.

---

## 3. Timeouts, Retries, and Backoff

### A. Non-Negotiable Timeout Policy
- **Every I/O call must have an explicit timeout:** Database queries, Redis commands, HTTP client requests, gRPC calls, RPC queues, filesystem locks.
- **Never use infinite timeouts.** Defaults in many standard libraries (e.g. Go `http.DefaultClient`, Python `requests` default) have no timeout and can hang forever, starving thread pools.
- Differentiate connect timeout (short: 1-3s) from read/total timeout (workload-appropriate: 3-10s).

### B. Retry Strategy & Storm Prevention
- **Never retry non-idempotent operations without an idempotency key.**
- **Never retry 4xx errors** (client errors like 400 Bad Request, 401 Unauthorized, 404 Not Found, 422 Unprocessable). Only retry transient 5xx errors (502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout) and connection drops.
- **Exponential Backoff with Full Jitter:**
  $$\text{sleep} = \text{random}(0, \min(M, B \times 2^{\text{attempt}}))$$
  *Never use fixed delay retries* across distributed clients—they produce synchronized retry storms that crush recovering servers.
- **Retry Budgets & Circuit Breakers:** Cap maximum retries (typically 2-3). Trip circuit breakers when error rates exceed thresholds to allow upstream dependencies to recover.

---

## 4. Connection Pools & Resource Leaks

- **Bounded Connection Pools:** Database, Redis, and HTTP connection pools must be strictly bounded with clear acquisition timeouts. An unbounded pool can exhaust database connections and crash the database cluster.
- **Resource Defer / Finally:** Ensure database connections, HTTP response bodies, file descriptors, and stream buffers are closed in `finally` / `defer` blocks, even if exceptions occur during parsing.
- **Goroutine / Thread / Memory Leaks:** Background tasks and workers must respect context cancellation (`ctx.Done()`, signal traps) to terminate promptly on shutdown.

---

## 5. Idempotency Key Implementation Standard

When implementing idempotency for critical write operations:
1. Client generates a unique UUID `Idempotency-Key` header.
2. Server acquires a short-lived atomic lock (Redis `SET key value NX EX 30` or DB row lock).
3. If key exists and is already completed: Return the cached prior response payload and status code immediately.
4. If key exists and is currently in-progress: Return `409 Conflict` or wait with short polling.
5. If key is new: Execute business logic within a database transaction, store the final response payload along with the idempotency key atomically, and release the lock.

---

## 6. Safe Degradation vs Silent Corruption

Never invent synthetic fallbacks that mask real architectural failures:

| Scenario | Anti-Pattern (Silent Corruption) | Resilient Pattern (Explicit & Safe) |
|---|---|---|
| Payment gateway times out | Return a dummy success and mark order paid | Return `504 Gateway Timeout` or `202 Accepted` (Pending async reconciliation) |
| Cache read fails | Log error and return empty list (client assumes no data exists) | Fallback to primary database read or return 503 if database would be overwhelmed |
| Configuration missing | Substitute unverified hardcoded default | Fail fast at process startup with a descriptive error |
| Partial batch insertion fails | Silently skip broken items without informing caller | Transaction rollback or return detailed per-item error manifest |

---

## 7. Resilience Pressure Matrix

Run this matrix mentally or via integration tests during Phase 6 (Verify):

| Fault to Inject | Injection Mechanism | Expected Invariant |
|---|---|---|
| **Downstream Timeout** | Mock sleep exceeding timeout limit | Client aborts at timeout deadline; resources released; metrics incremented. |
| **Connection Drop** | Kill connection abruptly mid-payload | Server rejects partial stream; rollback occurs; no corrupt records saved. |
| **Duplicate Delivery** | Send identical webhook/event payload 3x | Idempotency handles second/third calls cleanly; exactly-once business outcome. |
| **High Concurrency** | 50 concurrent requests for same balance/seat | Atomic lock / DB transaction serializes access; no race conditions or negative balance. |
| **Graceful Shutdown** | Send `SIGTERM` while request is in flight | Server stops accepting new connections; finishes in-flight request before exiting. |
