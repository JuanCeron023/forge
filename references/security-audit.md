# Backend Security Audit Reference

A practical, senior-level checklist for evaluating backend changes across security and trust boundaries. 

---

## 1. Threat & Trust Model

A security flaw is **an attacker gaining unauthorized capability across a trust boundary**. Secret-shaped strings or unhandled errors alone do not establish a vulnerability.

Always establish:
- **Principal:** Who is executing the request? (Anonymous guest, authenticated user, tenant admin, background worker, system process).
- **Authority:** What permissions does this principal legitimately hold?
  - *Authority Principle:* If a process or user already has read access to the database or host filesystem, reading a config file is not an escalation of privilege.
- **Trust Boundary:** Where untrusted input crosses into trusted storage, internal networks, or elevated execution contexts.

---

## 2. The Exploitability Chain

Before classifying an observation as a security vulnerability or blocker, prove the complete 5-link chain:

$$\text{Attacker Capability} \longrightarrow \text{Reachability} \longrightarrow \text{Attacker Control} \longrightarrow \text{Boundary Crossed} \longrightarrow \text{Security Impact}$$

1. **Attacker Capability:** What prerequisite access does the attacker need? (Public internet, valid low-privilege JWT, internal subnet access).
2. **Reachability:** Can the affected code path actually be invoked from an external or untrusted interface?
3. **Attacker Control:** Does the attacker control or influence the payload, parameters, headers, or state that reaches the sink?
4. **Boundary Crossed:** Does execution escalate privilege, read unauthorized tenant data, or cross isolation boundaries?
5. **Security Impact:** Confidentiality loss, integrity violation, unauthorized execution, or denial of service.

*Rule: If any link is missing, document it as an explicit gap. Never invent speculative deployment setups to fabricate a vulnerability.*

---

## 3. Core Backend Attack Vectors Checklist

### A. SQL Injection & ORM Escapes
- [ ] Are all queries parameterized? Raw string concatenation (`f"SELECT ... WHERE id = {user_input}"`) is an immediate blocker.
- [ ] For ORMs (SQLAlchemy, Prisma, Django ORM, GORM): Are raw SQL fragments (`.raw()`, `.whereRaw()`, `text()`, `ORDER BY`) protected against user-controlled column names or direction?
- [ ] Are JSON / JSONB field extractors sanitized against injection?

### B. Authorization & Multi-Tenant Isolation (BOLA / IDOR)
- [ ] **Tenant Scoping:** Does every query filter by `tenant_id` or `workspace_id` derived directly from the verified session/token, never from client-supplied request bodies or query params?
  ```sql
  -- BAD: trusts client input
  SELECT * FROM documents WHERE id = :doc_id;

  -- GOOD: enforces tenant isolation
  SELECT * FROM documents WHERE id = :doc_id AND tenant_id = :authenticated_tenant_id;
  ```
- [ ] **Horizontal Escalation:** Can User A access or mutate User B's resources by guessing or swapping an ID?
- [ ] **Vertical Escalation:** Are administrative endpoints protected by role/permission checks executed on the server, not solely relying on UI hiding?

### C. Authentication & Session Hygiene
- [ ] Are password hashes using salted, slow cryptographic algorithms (Argon2id, bcrypt, PBKDF2)?
- [ ] Are JWTs validated with explicit algorithm whitelisting (preventing `alg: none` or RSA/HMAC confusion)?
- [ ] Are sensitive tokens, refresh tokens, and session cookies configured with `HttpOnly`, `Secure`, and `SameSite=Lax/Strict`?
- [ ] Does logout or password change immediately revoke active sessions/tokens or invalidate token version counters?

### D. Server-Side Request Forgery (SSRF)
- [ ] If the backend fetches user-supplied URLs (webhooks, avatar imports, PDF generators):
  - [ ] Are private IP ranges blocked? (`127.0.0.1`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.169.254` AWS metadata).
  - [ ] Is DNS rebinding prevented (resolve DNS before connecting and validate the resulting IP)?
  - [ ] Are redirects validated so a public URL cannot redirect to an internal host?

### E. Command Injection & Unsafe Deserialization
- [ ] Are subprocesses executed with array arguments rather than shell strings?
  - `subprocess.run(["ls", user_arg])` ✅ vs `subprocess.run(f"ls {user_arg}", shell=True)` ❌
- [ ] Are untrusted payloads deserialized using safe parsers (`json.loads`, `yaml.safe_load`) instead of unsafe picklers (`pickle.loads`, Java native deserialization)?

### F. Secret Leakage & Logging Hygiene
- [ ] Are API keys, passwords, bearer tokens, or PII excluded from application logs, error messages, and URL query strings?
- [ ] Are secrets read strictly from environment variables or dedicated secret managers (Vault, AWS Secrets Manager), never committed to git or static files?
- [ ] Are stack traces hidden from external clients in production error responses?

### G. Rate Limiting & Denial of Service
- [ ] Are expensive endpoints (authentication, password reset, search, file upload, report generation) protected by rate limits?
- [ ] Are request body and file upload sizes strictly bounded before buffering in memory?

---

## 4. Classification & Merge Policy

Categorize every finding with unambiguous criteria:

| Classification | Meaning | Action |
|---|---|---|
| **Confirmed Vulnerability** | Complete exploitability chain proven with reproducible evidence. | **Blocker.** Must fix before shipping. |
| **Likely Vulnerability** | Strong code evidence of flaw; minimal environmental preconditions. | **Blocker.** Requires fix or definitive refutation. |
| **Hardening** | Defense-in-depth improvement (e.g. adding stricter security headers or input constraints), but no active exploit chain exists. | **Non-blocking improvement.** File as follow-up if not trivial. |
| **In-Scope Regression** | Flaw introduced by the current diff. | **Blocker.** Must fix in this change. |
| **Out-of-Scope Vulnerability** | Pre-existing flaw discovered in adjacent code outside the diff. | **Non-blocking for this PR.** Document as high-priority issue. |
| **Verification Gap** | Cannot verify due to missing staging credentials or sandbox. | **Explicitly documented gap.** Must not be reported as passed. |
