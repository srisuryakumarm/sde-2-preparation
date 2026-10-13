# Day 67 — Missing Primer: The JJWT 0.12+ API (Day 67's code will not compile as written)

**Do this first on Day 67 (≈ 15 min), then continue with Day 67's regular plan.**
**Needed by:** Day 67's Project Block (JWT validation filter at the Gateway) and Day 68 (rate limiting keyed by the token's subject, if you do it).
**Why it is missing:** Day 67's book uses the JJWT 0.11 API — `Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token)`. Since JJWT 0.12.0, `parserBuilder()` is gone (renamed `parser()`), `setSigningKey` became `verifyWith`, `parseClaimsJws` became `parseSignedClaims`, and `getBody()` became `getPayload()`.
**Already in place:** the JWT/Auth primer (Day 67), the Reactor primer (Day 65), JWT validation concepts.

## 1. Dependencies (explicit version — Boot does not manage JJWT)
`io.jsonwebtoken:jjwt-api` (compile), `jjwt-impl` and `jjwt-jackson` (runtime) — all three at the **same 0.12.x-or-newer version**.

## 2. Old → new
| JJWT 0.11 (the book) | JJWT 0.12+ |
|---|---|
| `Jwts.parserBuilder()` | `Jwts.parser()` |
| `.setSigningKey(key)` | `.verifyWith(key)` (a `SecretKey` for HMAC) |
| `.parseClaimsJws(token)` | `.parseSignedClaims(token)` |
| `.getBody()` | `.getPayload()` |
| `Jwts.builder().setSubject(..).setExpiration(..)` | `.subject(..)` `.expiration(..)` |
| `signWith(key, SignatureAlgorithm.HS256)` | `signWith(key)` — the algorithm is inferred from the key |

## 3. The shape to use
```java
SecretKey key = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));   // HS256 needs a secret of at least 32 bytes,
                                                                                // otherwise WeakKeyException

String token = Jwts.builder()
        .subject("user-42")
        .issuedAt(new Date())
        .expiration(new Date(System.currentTimeMillis() + 15 * 60_000))
        .signWith(key)
        .compact();

Claims claims = Jwts.parser().verifyWith(key).build()
        .parseSignedClaims(token)          // throws on a bad signature, malformed token, or expiry
        .getPayload();
```
In the Gateway filter catch `io.jsonwebtoken.JwtException` (the base class of `ExpiredJwtException`, `MalformedJwtException`, signature failures) and answer `401` with `exchange.getResponse().setComplete()`. Validation is pure CPU work, so doing it synchronously inside the filter does not block the event loop.

## 4. Two practical notes
- **Secret:** read it from an environment variable (`JWT_SECRET`), never commit it — Day 96's Kubernetes Secrets are where it ends up.
- **Getting a token to test with:** the plan validates tokens at the Gateway but never issues them. Write a 10-line `TokenGenerator` `main` (or a test) that prints a token for `user-42`, then call `curl -H "Authorization: Bearer <token>"`. A login endpoint is outside the plan.

## Checklist
- [ ] Three JJWT artifacts at one 0.12.x+ version; `verifyWith` / `parseSignedClaims` / `getPayload` compile.
- [ ] A valid token passes; a tampered, expired, or missing token gets `401`.
- [ ] Continue with Day 67's regular plan.
