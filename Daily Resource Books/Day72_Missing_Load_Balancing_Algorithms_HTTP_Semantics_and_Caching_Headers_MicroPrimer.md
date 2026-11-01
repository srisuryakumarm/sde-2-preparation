# Day 72 — Missing Primer: Load-Balancing Algorithms, Sticky Sessions, and the HTTP Semantics / Caching Headers Every API Answer Relies On

**Do this on Day 72 (≈ 30 min, right after Day 72's HTTP and load-balancer theory block and the gRPC micro-primer), then continue with Day 72's regular plan.**
**Needed by:** Day 73 (its exercise asks you to contrast **ALB vs NLB** — an L7 and an L4 balancer), Days 120–133 (every HLD diagram has a load-balancer box, and "how does it choose?" is the follow-up), Day 123 (cache and CDN design), Day 127 (WebSocket servers need *least connections*, not round robin), Day 129 (CDN — it is an HTTP cache), Day 142 (high availability: "is your load balancer a single point of failure?"), and the **"define the API"** step of every HLD and LLD answer.
**Why it is missing:** Day 72's theory teaches L4 vs L7 and health checks — but never *how the balancer picks a server*. "Round robin", "least connections" and "sticky sessions" appear in none of the 148 books. HTTP method semantics (safe / idempotent), status-code families and **caching headers** (`Cache-Control`, `ETag`) are never taught, although retries (Day 64), CDNs (Day 129), idempotency (Day 130) and pagination (Day 78) all rest on them.
**Already in place:** HTTP basics and REST endpoints (Days 32–35), TCP / UDP / DNS (Day 71), L4 vs L7, health checks, forward vs reverse proxies (Day 72's theory and the gRPC micro-primer), consistent hashing (Day 60), a shared store such as Redis (Day 44), JWT (Day 67 — it is why stateless servers work).

---

## 1. How a load balancer picks a server
| Algorithm | Rule | Good for | Watch out |
|---|---|---|---|
| **Round robin** | next server in a circle | identical, stateless servers, similar request cost | ignores how busy each server is |
| **Weighted round robin** | bigger weight → proportionally more requests | mixed hardware, canary (weight 5 %) | static weights |
| **Least connections** | fewest open connections | **long-lived or uneven requests** (WebSockets, uploads, slow queries) | needs live counts; new servers get flooded without *slow start* |
| **Least response time** | lowest latency (+ few connections) | heterogeneous backends | noisy measurements |
| **IP hash** | `hash(client IP) % N` | crude stickiness | many users behind one NAT IP; adding a server reshuffles everyone |
| **Consistent hashing** | key on the ring (Day 60) | **caches** — the same key keeps hitting the same node; minimal remapping when nodes change | uneven without virtual nodes |
| **Power of two choices** | pick **2 random servers**, send to the less loaded | large fleets, no global state | — |
**Why "power of two choices" matters:** pure "send to the least-loaded server" with *slightly stale* load numbers makes every balancer pick the *same* server at the same moment (**herding**). Picking just two at random and taking the better one avoids the stampede and is almost as balanced as a perfect choice — a favourite deep-dive answer.
Pick by traffic shape: stateless + uniform → round robin; long-lived connections → least connections; caches → consistent hashing.

## 2. Sticky sessions vs stateless servers
**Sticky sessions** (a cookie inserted by the balancer, or an IP hash) send one user to the same server so in-memory session data is found. Costs: **uneven load**, the session is **lost when that server dies or scales in**, deployments get awkward. **The interview answer: make servers stateless** — keep session state in a shared store (Redis, Day 44) or in a signed token (JWT, Day 67) — then *any* server can serve *any* request and plain round robin or least connections work.

## 3. What a production balancer also does
- **Health checks:** *active* (probe `/health` every few seconds; N failures → out of rotation) and *passive* (notice errors/timeouts on real traffic). The check must test the instance, not its downstream dependencies (Day 64's rule).
- **Connection draining / deregistration delay:** stop sending *new* requests to a server that is being removed, let in-flight ones finish — the difference between a clean deploy and a burst of 502s.
- **Slow start**, **TLS termination** (the balancer decrypts; backends speak plain HTTP inside the private network), **timeouts and bounded retries**, edge **rate limiting** (Day 68/121).
- **The balancer is itself a single point of failure.** Fixes: an **active–passive pair sharing a floating (virtual) IP** (VRRP / keepalived), **active–active** behind several DNS records or **anycast**, or a managed cloud balancer that spans availability zones. **Global** traffic: **GeoDNS / latency-based DNS** or anycast picks a *region*, then a regional balancer picks a server. DNS-based failover is only as fast as the record's **TTL** and client caching.

## 4. HTTP semantics in one table
| Method | Safe (no side effects) | Idempotent (repeat = same result) | Typical use |
|---|---|---|---|
| `GET`, `HEAD`, `OPTIONS` | ✅ | ✅ | read |
| `PUT` | ❌ | ✅ | **replace** a resource |
| `DELETE` | ❌ | ✅ | delete (a second call returns 404 — the *state* is the same) |
| `PATCH` | ❌ | not guaranteed | partial update |
| `POST` | ❌ | ❌ — needs an **idempotency key** (Day 50's primer, Day 130) | create / action |
Status-code families and the ones you must know: **2xx** `200` OK, `201` Created (+ `Location`), `202` Accepted (work continues asynchronously), `204` No Content · **3xx** `301` permanent, `307/308` temporary (method preserved), `304` Not Modified · **4xx** `400` malformed, `401` *not authenticated*, `403` *authenticated but not allowed*, `404`, `409` Conflict (version clash, duplicate), `422` semantically invalid, `429` Too Many Requests (+ `Retry-After`) · **5xx** `500`, `502` bad gateway (upstream gave garbage), `503` unavailable (+ `Retry-After`), `504` gateway timeout.
**Versioning:** put a version in the path (`/v1/orders`) or a header; prefer **additive, backward-compatible** changes (new optional fields) and cut a new version only for breaking ones.

## 5. HTTP caching in one screen
`Cache-Control` tells browsers and shared caches (reverse proxies, CDNs) what they may keep:
- `max-age=N` — fresh for N seconds; `s-maxage=N` — same, for **shared caches only**; `public` / `private` (only the user's browser);
- `no-store` — **never** keep it (sensitive data); `no-cache` — you *may* keep it but **must revalidate** before reuse (a common confusion);
- `immutable` — it will never change; `stale-while-revalidate` — serve stale while refreshing.
**Validation:** the server sends `ETag: "abc"`; later the client sends `If-None-Match: "abc"`; if unchanged the server answers **`304 Not Modified`** with no body — a cheap check instead of a full download (`Last-Modified` / `If-Modified-Since` is the older form). `Vary: Accept-Encoding` tells caches the response differs by that header. The cache **key** is the URL plus the `Vary` headers.
**Pattern worth stating in a CDN answer:** version file names (`app.3f9a1c.js`), give them `max-age=31536000, immutable`, and keep the HTML short-lived — a deploy "invalidates" assets simply by changing their URLs.

## 6. Exercise (≈ 10 min)
1. Run `curl -I https://example.com` and read the headers: find any `Cache-Control`, `ETag`, `Content-Type`. Run it twice; then `curl -I -H 'If-None-Match: <the etag you saw>' https://example.com` and see whether you get `304` (depends on the site).
2. **Paper:** three servers have 10, 3 and 7 open connections — which does *least connections* pick? With weights 5:1:1, how many of the next 7 requests does server A get? A new server joins with 0 connections — what problem can *least connections* cause, and what feature fixes it (slow start)?
3. **Say aloud (45 s):** "WebSocket chat servers sit behind a balancer — which algorithm, and what do you do about sticky sessions?" (least connections; make the servers stateless, keep presence/session in Redis; a reconnect can land anywhere).
4. Write the status code you would return for: duplicate order id; token expired; token valid but wrong tenant; downstream timed out; accepted for async processing.

## Checklist
- [ ] Can name six balancing algorithms and pick one for a stated traffic shape; can explain why power-of-two-choices avoids herding.
- [ ] Can argue "stateless servers + shared store" over sticky sessions.
- [ ] Can say how a load balancer stays highly available and how global traffic picks a region.
- [ ] Can fill in the safe/idempotent table and use the right status codes (401 vs 403, 409, 429, 502 vs 504).
- [ ] Can explain `Cache-Control` (`no-cache` vs `no-store`), `ETag` + `304`, and versioned asset URLs. Continue with Day 72's regular plan.
