# Day 63 — Missing Primer: Web and API Security Fundamentals

**Do this first on Day 63 (Sunday; ≈ 60 min), then continue with Day 63's regular plan.**
**Needed by:** Day 67 (JWT at the Gateway), Day 96 (Kubernetes Secrets), and every SDE-2 loop — "how do you secure this API?" is asked about every design.
**Why it is missing:** SQL injection, XSS, CSRF, CORS, password hashing and TLS are mentioned nowhere in the books. Day 67 teaches JWT only (which assumes you already know what threats a token defends against).
**Already in place:** JWT/Auth primer (Day 67 — authN vs authZ, JWT structure), validation (Day 33), SQL basics (Day 36/40).

---

## 1. Injection (SQL injection)
Building SQL by string concatenation lets input become code:
```java
"SELECT * FROM users WHERE name = '" + name + "'"      // name = ' OR '1'='1   → returns every user
```
**Fix:** parameterized queries — `PreparedStatement` with `?`, JPA `:param` / Spring Data derived queries. A native query that concatenates strings is **not** protected. Validate input (Day 33) as defense in depth, not as the primary defense.

## 2. XSS, CSRF, CORS — three different things
| Threat | What it is | Defense |
|---|---|---|
| **XSS** | attacker's script runs in a victim's browser (stored, reflected, DOM) | output-encode, don't `innerHTML` untrusted text, Content-Security-Policy |
| **CSRF** | another site makes the victim's browser send *cookie-authenticated* requests | `SameSite` cookies + CSRF tokens; **not an issue for APIs authenticated by a header token** |
| **CORS** | a *browser* rule: a page may only read responses from other origins if the server opts in (`Access-Control-Allow-Origin`; non-simple requests trigger an `OPTIONS` preflight) | configure allowed origins explicitly; never `*` together with credentials. **CORS protects browsers, not your server** |

## 3. Sessions vs tokens, and where a token lives
Cookie session: server stores state; cookie flags `HttpOnly` (script can't read), `Secure` (HTTPS only), `SameSite`. Token (JWT): stateless, but **cannot be revoked early** (use short expiry + refresh tokens). A token in `localStorage` is readable by any XSS; in an `HttpOnly` cookie it is safe from script but needs CSRF protection. Pick consciously.

## 4. Passwords
Never store plaintext, never encrypt reversibly. Store a **salted, slow hash** — bcrypt, scrypt or Argon2id — with a tunable work factor; MD5/SHA-1/plain SHA-256 are too fast. Compare in constant time (the library does). Add rate limiting (Day 68) and lock-out/MFA for logins.

## 5. TLS in a paragraph
The server presents a certificate signed by a trusted CA; the client verifies the chain and hostname, then both sides agree on **symmetric session keys** (TLS 1.3 does this in one round trip). Traffic is encrypted and integrity-protected. Usually **terminated at the load balancer/Gateway**; **mTLS** (both sides present certificates) secures service-to-service calls (service mesh, Day 138). `HSTS` tells browsers to always use HTTPS.

## 6. Authorization bugs beat crypto bugs
**IDOR / broken object-level authorization:** `GET /orders/42` returns the order to *any* logged-in user. The service must check that order 42 belongs to the caller (use the JWT's `sub`). Default to **deny**; use least privilege; RBAC (roles) vs ABAC (attributes).
**SSRF:** the server fetches a URL an attacker supplied (e.g. the cloud metadata address `169.254.169.254`) — allow-list destinations.

## 7. Secrets and supply chain
Never commit secrets. Progression: environment variable → Kubernetes Secret (**base64 is not encryption**) → a secrets manager (Vault / cloud). Rotate. Scan dependencies (Dependabot, OWASP dependency-check) and pin versions. Never log passwords, tokens or personal data.

## 8. Exercise (30 min)
1. Plain JDBC (H2 or your Postgres): reproduce the injection above with string concatenation; then fix it with a `PreparedStatement`.
2. `org.springframework.security:spring-security-crypto` (version managed by Boot): hash a password with `new BCryptPasswordEncoder().encode(...)`, verify with `matches`, and show that two hashes of the same password differ (salt).
3. Threat-model the platform on one page: list five attacks (e.g. IDOR on `/orders/{id}`, forged JWT, flooding, leaked secret, injection via search) and the mitigation each needs.

## Checklist
- [ ] Can explain SQL injection and the fix; can tell XSS, CSRF and CORS apart.
- [ ] Can say why CSRF does not apply to header-token APIs, and what CORS does *not* protect.
- [ ] Can say how passwords are stored and why a fast hash is wrong.
- [ ] Can describe TLS termination and mTLS; can explain IDOR.
- [ ] Exercises done. Continue with Day 63's regular plan.
