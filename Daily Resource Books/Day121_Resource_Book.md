# Day 121 Resource Book — Rate Limiter, Formalized

**Series:** SDE-2 Interview Prep Resource Books · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 120](./Day120_Resource_Book.md) · **Next ▶:** [Day 122](./Day122_Resource_Book.md)
**Companion to:** Day 121 of `Week_18_Revised.md`

---

## Recap: what today is

Back in Week 10 (Day 68), you built a Token Bucket rate limiter for the platform's Gateway — a single, working algorithm, on a single instance. Today does two things Day 68 deliberately left for later: it places Token Bucket alongside the three other algorithms that actually get compared in a real interview, and it exposes the specific way Token Bucket's original implementation quietly breaks the moment the Gateway (Week 10, Day 66) runs as more than one instance — which, once Spring Cloud Gateway and a Load Balancer (Week 10, Day 72) are both already in the picture, is exactly the realistic situation this platform is now in.

A small, deliberately scoped exception to the usual "teach the tool in full before using it" rule, worth stating plainly: today reaches for a handful of specific Redis commands (`INCR`, `EXPIRE`, `HMGET`/`HMSET`) and Lua scripting, narrowly, for exactly the one job of making a counter check-and-increment atomic across instances — the same way Day 120 reached for `SELECT ... FOR UPDATE SKIP LOCKED` narrowly, without a full SQL locking course first. Redis's *full* picture — its data structure catalog, how it compares to Memcached, persistence, pub/sub, TTL as a general-purpose feature rather than just "the thing keeping a rate-limit key from growing forever" — is Day 123's job, two days from now. Today borrows only what's needed for this one problem.

## Learning Objectives

By the end of today, without notes:

1. Compare Token Bucket, Leaking Bucket, Fixed Window, and Sliding Window Log — mechanism, trade-off, and when each is the right (or wrong) choice — from memory.
2. Explain, precisely, the specific flaw Fixed Window has that the other three don't, with a worked numeric example.
3. Explain why a per-instance in-memory counter silently breaks once a service is horizontally scaled, and fix it using Redis with an atomic Lua script.
4. Justify a rate-limiting design's key dimension (per-user, per-IP, or per-API-key) and its rejection behavior (hard reject vs. queue) as explicit requirements decisions, not defaults.

## Concept Dependency Map

```
Prerequisites, confirmed already covered:
  Token Bucket (built) ................. Week 10, Day 68  (recapped Week 17, Day 114)
  Spring Cloud Gateway .................. Week 10, Day 66
  Load Balancers (L4 vs. L7) ............ Week 10, Day 72
  The check-then-act race shape ......... Week 16, Day 111 (Parking Lot) · Week 17, Day 113 (ATM) · Day 117 (BookMyShow)
  SQL / relational basics ............... Week 6, Day 40
        │
        ▼
NEW today: the other 3 algorithms in the landscape
  Leaking Bucket ─── smooths to a constant output rate
  Fixed Window ───── simple, but a real 2x-at-the-boundary flaw
  Sliding Window Log  accurate, memory-heavy
        │
        ▼
Distributed rate limiting (NEW): why a per-instance counter breaks
  once the Gateway is horizontally scaled
        │
        ▼
Redis, borrowed narrowly for this one job (full depth: Day 123)
  INCR / EXPIRE / HMGET / HMSET ── new syntax, scoped
        │
        ▼
Lua scripting in Redis (NEW) — atomicity via single-threaded execution
        │
        ▼
Applied: Token Bucket, made correctly distributed
```

---

## Part 1 — Requirements, First (Framework Discipline Carries Over)

Yesterday's framework doesn't stop applying just because today's scope is narrower than a full system. Two requirements questions are worth stating explicitly before comparing algorithms, because the "right" algorithm and the "right" rejection behavior both depend on the answer:

- **What's the rate-limiting key?** Per user (fair per person, but a single user can still open many connections), per IP (catches abuse from one source, but punishes every user behind a shared corporate NAT or campus network equally), or per API key (the standard choice for a public API product, since it ties directly to a billing/tier relationship). No universally correct answer — state which one a given system needs and why.
- **What happens on rejection?** A hard reject — HTTP **429 Too Many Requests**, the standard status code for exactly this situation — tells the client immediately and cheaply. Queuing the request instead smooths bursts at the cost of added latency and a queue that itself needs bounding (an unbounded queue just moves the overload problem from "rejected requests" to "an ever-growing queue," which is arguably worse). Most public APIs hard-reject with a `Retry-After` header; internal, latency-tolerant systems sometimes queue instead.

---

## Part 2 — The Four Algorithms

### Recap: Token Bucket (already built — Week 10, Day 68)

**Mechanism, briefly:** a bucket holds up to `capacity` tokens, refilled at a fixed `rate` (tokens/sec). Each request consumes one token; if the bucket is empty, reject. Tokens accumulate while idle, up to the cap — which is *exactly* why Token Bucket allows **bursts**: a client that hasn't sent a request in a while has a full bucket and can legitimately fire `capacity` requests instantly.

### NEW: Leaking Bucket

**Mechanism:** picture a bucket with a hole in the bottom, leaking at a fixed rate. Incoming requests are added to the bucket (a queue, really); if the bucket is full, new requests are dropped. The bucket drains — processes queued requests — at a strictly constant rate, regardless of how bursty the input was.

**The behavioral difference from Token Bucket, stated precisely, since it's the single most commonly asked contrast:** Token Bucket controls the rate at which requests are *admitted*, and permits bursts up to the bucket's capacity. Leaking Bucket controls the rate at which requests are *processed*, and enforces a genuinely constant output rate — no bursts survive it, ever, even if the input briefly spikes. **When this distinction actually matters:** a downstream service that degrades badly under bursty traffic (say, a legacy system with no headroom for spikes) wants Leaking Bucket's smoothing specifically; an API serving human users, where the occasional legitimate burst (a user rapid-clicking, a page loading several resources at once) shouldn't be needlessly punished, wants Token Bucket's burst tolerance.

**Trade-off against Token Bucket:** Leaking Bucket's smoothing is also its cost — a legitimate burst that a real user would consider normal gets throttled exactly the same as an actual abusive spike, since the algorithm has no concept of "this traffic pattern is fine, just uneven."

### NEW: Fixed Window Counter

**Mechanism:** the simplest of the four. Divide time into fixed windows (e.g., every 60-second block, aligned to the clock: `[0:00, 1:00)`, `[1:00, 2:00)`, …). Maintain one counter per window; increment on each request; reset to zero when a new window starts. Reject once the counter hits the limit for the current window.

**The real, commonly-probed flaw — worked precisely:** suppose the limit is 100 requests/minute, windows aligned to the clock.

- A client sends 100 requests at `11:59:59` — all allowed (window `[11:59:00, 12:00:00)` reaches exactly 100, its limit).
- The clock ticks to `12:00:00` — a **new window starts, counter resets to 0.**
- The same client immediately sends 100 more requests at `12:00:01` — all allowed too (window `[12:00:00, 12:01:00)` independently reaches its own limit of 100).

**Result: 200 requests were permitted within roughly a 2-second real span**, against a stated limit of 100/minute — a burst of up to **2x** the intended rate, purely because the requests straddled a window boundary. Nothing in Fixed Window's own bookkeeping ever sees this as a violation, because it only ever looks at *one* window at a time, never the actual rolling interval a real client experiences.

**Why it's still genuinely used despite this:** it's by far the simplest and cheapest of the four — one counter, one reset — and for traffic that isn't adversarially timed against the exact window boundary, the flaw rarely matters in practice. Simplicity is a real trade-off, not just a compromise.

### NEW: Sliding Window Log

**Mechanism:** maintain a log (conceptually, a timestamp per request) covering the trailing window. On each new request: evict every timestamp older than `now - window_size`; count what's left; if count `< limit`, allow and record the new timestamp; otherwise reject.

```java
public class SlidingWindowLogLimiter {
    private final Deque<Long> timestamps = new ArrayDeque<>();
    private final int limit;
    private final long windowMillis;

    public SlidingWindowLogLimiter(int limit, long windowMillis) {
        this.limit = limit;
        this.windowMillis = windowMillis;
    }

    public synchronized boolean allowRequest(long now) {
        while (!timestamps.isEmpty() && timestamps.peekFirst() <= now - windowMillis) {
            timestamps.pollFirst();   // evict everything outside the trailing window
        }
        if (timestamps.size() < limit) {
            timestamps.addLast(now);
            return true;
        }
        return false;
    }
}
```

**Why this genuinely fixes Fixed Window's flaw:** the window here is a true rolling interval anchored to *now*, not a fixed clock-aligned block — there is no boundary for a client to straddle, because there's no fixed boundary at all. Re-running the exact 11:59:59-and-12:00:01 scenario above: at `12:00:01`, the log still contains the 100 timestamps from `11:59:59` (only 2 seconds old, well inside a 60-second trailing window), so the 101st request is correctly rejected.

**The cost, precisely:** memory is `O(limit)` — up to `limit` timestamps stored per key, which is meaningfully more than Fixed Window's single integer or Token Bucket's two numbers (tokens, last-refill-time). At high request volumes with a generous limit, this is a genuine, not theoretical, memory cost.

> ⚠️ **Extension material, skippable under time pressure:** a fifth algorithm, **Sliding Window Counter**, exists specifically to approximate Sliding Window Log's accuracy at close to Fixed Window's memory cost — it keeps two fixed-window counters (current and previous) and computes a weighted estimate based on how far into the current window `now` falls, trading a small amount of accuracy for `O(1)` memory instead of `O(limit)`. Worth knowing it exists and roughly what problem it solves; not required for today's deliverable, and going deeper here would cut into today's actual required content.

### The Comparison Table (Today's Deliverable)

| | Token Bucket | Leaking Bucket | Fixed Window | Sliding Window Log |
|---|---|---|---|---|
| Allows bursts? | Yes, up to bucket capacity | No — strictly constant output | Yes, up to 2x at a boundary (a flaw, not a feature) | No — a true rolling limit |
| Memory per key | O(1) — tokens + timestamp | O(queue size) | O(1) — one counter | O(limit) — one timestamp per request in-window |
| Accuracy | Exact, by design | Exact, by design | Inexact at boundaries | Exact |
| Best fit | Bursty, human-facing traffic | Smoothing output to a fragile downstream | Cheap, approximate limiting where the flaw is tolerable | Correctness-critical limiting, moderate volume |

---

## Part 3 — Distributed Rate Limiting: Why the Old Implementation Breaks

**The concrete failure, stated precisely:** Day 68's Token Bucket kept its bucket state — tokens remaining, last refill time — as an in-memory field inside the Gateway process. That was correct for *one* Gateway instance. Now that Spring Cloud Gateway (Day 66) sits behind a Load Balancer (Day 72) with multiple instances for availability, each instance holds its **own, independent** bucket. A client hitting instance A has used, say, 80 of its 100 allowed tokens on A's bucket — but instance B, having never seen that client before, still shows a full bucket. The load balancer's routing decides, essentially arbitrarily from the client's perspective, which bucket a given request checks against — so the *effective* combined limit across N instances is closer to `N × intended_limit`, not the intended limit at all. This is silent — nothing crashes, nothing logs an error — the system just quietly enforces a much weaker limit than configured.

**The fix, at a high level:** the bucket state has to live somewhere every instance can see and update — a **shared, external store.** Redis is the standard answer specifically because it's in-memory (sub-millisecond round trips, appropriate for a check that runs on every single request) and supports the one operation this problem actually needs: an atomic read-modify-write.

**Why the atomicity requirement is exactly the same shape you've already proven you can reason about:** two Gateway instances both checking "is there capacity?" and then both separately "using" that capacity is a **check-then-act race** — the identical shape Parking Lot's double-booking (Week 16, Day 111), the ATM's naive dispenser (Week 17, Day 113), and BookMyShow's seat race (Week 17, Day 116–117) each already demonstrated. The fix pattern is the same one you already know in spirit: make the check and the act a single, indivisible operation. The *mechanism* differs — Redis, not a JVM lock or a database row lock — but the underlying reasoning is not new.

### Redis, Borrowed Narrowly: the Commands This Job Needs

- **`INCR key`** — atomically increments an integer stored at `key` by 1 and returns the new value; creates the key at 0 first if it doesn't exist yet.
- **`EXPIRE key seconds`** — sets a key to auto-delete after the given number of seconds. Used here purely so a rate-limit key for an inactive client doesn't sit in Redis forever.
- **`HMGET` / `HMSET`** — get/set multiple fields on a Redis *hash* (a single key holding several named fields) in one round trip — needed below because Token Bucket's state is two numbers (tokens, last-refill-time), not one.

**Why Redis rather than "just use a shared database":** a relational database round trip (even a fast one) is milliseconds, and involves connection/transaction overhead that a plain in-memory key-value store doesn't. A rate-limit check runs on the hot path of *every single request* — the same "the redirect path is what users feel directly" reasoning from Day 120 applies here to the whole request path, not just a redirect.

### Lua Scripting: Where the Atomicity Actually Comes From

**The mechanism, precisely:** Redis executes commands on a single thread. A Lua script submitted to Redis runs as **one indivisible unit** — no other client's command, from any other Gateway instance, can execute in the middle of it. This is the actual source of atomicity here: not a lock anyone has to acquire or release, but the guarantee that nothing else gets a turn until the whole script finishes.

**A simple illustration first — Fixed Window via `INCR` + `EXPIRE`, atomically:**

```lua
-- KEYS[1] = rate limit key, e.g. "ratelimit:fixed:user:123"
-- ARGV[1] = limit (max requests per window)
-- ARGV[2] = window size in seconds
local current = redis.call('INCR', KEYS[1])
if current == 1 then
    redis.call('EXPIRE', KEYS[1], ARGV[2])   -- only set TTL on the window's first request
end
if current > tonumber(ARGV[1]) then
    return 0   -- reject
end
return 1       -- allow
```

Without the Lua wrapper, `INCR` then a separate check-and-`EXPIRE` would be at least two round trips — and two Gateway instances could both `INCR` past the limit before either checks the result, the exact race this whole section exists to prevent.

**The actual task — a distributed Token Bucket:**

```lua
-- KEYS[1] = bucket key, e.g. "ratelimit:bucket:user:123"
-- ARGV[1] = capacity (max tokens)
-- ARGV[2] = refill_rate (tokens per second)
-- ARGV[3] = now (current Unix timestamp, seconds — passed in, see note below)

local capacity = tonumber(ARGV[1])
local refill_rate = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

local bucket = redis.call('HMGET', KEYS[1], 'tokens', 'last_refill')
local tokens = tonumber(bucket[1])
local last_refill = tonumber(bucket[2])

if tokens == nil then
    tokens = capacity          -- first request for this key: bucket starts full
    last_refill = now
end

local elapsed = math.max(0, now - last_refill)
tokens = math.min(capacity, tokens + elapsed * refill_rate)   -- refill for time elapsed, capped

local allowed = 0
if tokens >= 1 then
    tokens = tokens - 1
    allowed = 1
end

redis.call('HMSET', KEYS[1], 'tokens', tokens, 'last_refill', now)
redis.call('EXPIRE', KEYS[1], 3600)
return allowed
```

**Trace, for `capacity=10`, `refill_rate=1` token/sec, a bucket that hasn't been touched in 15 seconds, `now = 1000`:** `tokens` read as some value from before, `last_refill` read as `985`. `elapsed = 1000 - 985 = 15`. `tokens + 15×1` far exceeds `capacity`, so `math.min` caps it at `10` — the bucket refills to full, correctly, exactly as an idle bucket should.

> ⚠️ **Common Mistake:** reaching for Lua's own `os.time()` inside the script instead of passing `now` in as `ARGV[3]`. Redis deliberately disables non-deterministic Lua functions like `os.time()` and unseeded `math.random()` inside scripts — a script's behavior has to be reproducible for replication correctness, so time has to come from either `redis.call('TIME')` (Redis's own server clock, called as a normal command) or, as above, be supplied by the caller.

**Complexity:** every operation here — `HMGET`, `HMSET`, `EXPIRE`, and the Lua arithmetic — is `O(1)`, so the whole check-and-update is `O(1)` per request, same as the original in-memory version, just now correct across any number of instances.

**Edge cases:** the very first request for a brand-new key (handled — `tokens == nil` branch initializes a full bucket); a client that's been silent long enough that `elapsed × refill_rate` would overflow past `capacity` many times over (handled — `math.min` caps it, so an idle bucket never over-fills); clock skew between Gateway instances if each computed `now` locally instead of from a synchronized source (worth flagging as a real operational concern — passing a consistently-sourced timestamp, or using Redis's own `TIME` command as the single source of truth, avoids two instances disagreeing about how much time has "elapsed").

---

## Project Block

**Repository:** `scalable-ecommerce-platform`. **Task:** replace the in-memory Token Bucket state from Week 10 with the Redis-backed Lua script above, invoked from the Gateway on every request. **Definition of done:** running two Gateway instances behind a load balancer, the shared limit is enforced correctly across both — send enough requests, split across both instances, to exceed the configured limit, and confirm the combined total stays capped at the intended limit rather than doubling.

## Career Block

**LinkedIn engagement:** 20 minutes commenting on 3–5 posts.

**Networking — a research task, not new theory:** look up how System Design architectures actually work at your target companies specifically — how Stripe handles idempotency (Week 19, Day 130, covers this formally; today is purely reconnaissance, reading what's public), how Uber handles surge pricing. This is deliberately research, not something today's material teaches in depth — the goal is having specific, real examples ready for a "tell me about a system you find interesting" conversation, not mastering either topic today.

## Daily Deliverable Check

- [ ] Can compare all four rate limiting algorithms and their trade-offs from memory, including Fixed Window's 2x boundary flaw with the worked numeric example.
- [ ] Can explain, precisely, why a per-instance in-memory counter breaks once a service is horizontally scaled.
- [ ] Redis-backed distributed Token Bucket live, verified correct across multiple Gateway instances behind a load balancer.
- [ ] Can explain why a Lua script guarantees atomicity in Redis, and connect it explicitly to the check-then-act race pattern from Parking Lot, the ATM, and BookMyShow.

---

## Day 121 — Interview Questions

---

**1. Name all four rate limiting algorithms and, for each, the one-sentence core trade-off.**

*Answer:* Token Bucket — allows bursts up to capacity, O(1) memory. Leaking Bucket — smooths to a strictly constant output rate, no bursts survive it. Fixed Window — cheapest, but allows up to 2x the limit at a window boundary. Sliding Window Log — exact rolling accuracy, at O(limit) memory cost.

---

**2. [Worked example] Demonstrate Fixed Window's boundary flaw with concrete numbers.**

*Answer:* Limit 100/minute. 100 requests at 11:59:59 fill that window's counter to exactly 100. At 12:00:00 the counter resets. 100 more requests at 12:00:01 fill the new window to 100 as well. Total: 200 requests allowed within about 2 real seconds, against a stated limit of 100/minute.

---

**3. Why does Sliding Window Log not suffer from that same flaw?**

*Answer:* It anchors the window to "now," not to a fixed clock boundary, so there's no boundary to straddle — at any instant, it counts exactly the requests within the trailing window, giving a true rolling limit.

---

**4. What's the key behavioral difference between Token Bucket and Leaking Bucket?**

*Answer:* Token Bucket controls admission and allows bursts up to the bucket's capacity when idle time has accumulated tokens. Leaking Bucket controls the processing rate and enforces a strictly constant output rate regardless of how the input arrived — no burst ever gets through it.

---

**5. Why does a single in-memory rate-limit counter break once a service runs as multiple instances behind a load balancer?**

*Answer:* Each instance holds its own independent counter/bucket. A request's outcome depends on which instance the load balancer happened to route it to, so the effective combined limit across N instances approaches N times the intended limit, silently — nothing errors, the system just under-enforces.

---

**6. Why is Redis the standard fix, rather than a shared relational database?**

*Answer:* The rate-limit check runs on every single request, so it needs sub-millisecond round trips — an in-memory store like Redis fits that hot-path latency requirement in a way a relational database's connection/transaction overhead doesn't.

---

**7. What race condition does an un-atomic "check count, then increment" have, and what established pattern is it the same shape as?**

*Answer:* Two instances can both read a count below the limit, both decide to allow, and both increment — letting the true count exceed the limit. It's the identical check-then-act race already seen in Parking Lot's double-booking (Week 16, Day 111), the ATM's naive dispenser (Week 17, Day 113), and BookMyShow's seat race (Week 17, Day 116–117).

---

**8. Why does wrapping the check-and-increment in a Lua script make it atomic in Redis specifically?**

*Answer:* Redis executes commands on a single thread, and a submitted Lua script runs as one indivisible unit — no other client's command can execute partway through it, so there's no window for a race to occur.

---

**9. [Trace] In the Token Bucket Lua script, a bucket with `capacity=10`, `refill_rate=1`/sec was last touched 15 seconds ago. What happens?**

*Answer:* `elapsed = 15`, so `tokens + 15` far exceeds `capacity`; `math.min(capacity, ...)` caps the refill at 10 — the bucket is treated as fully refilled, correctly, since it sat idle longer than it takes to fill from empty.

---

**10. Why shouldn't a Redis Lua script call `os.time()` directly to get the current time?**

*Answer:* Redis deliberately disables non-deterministic Lua functions like `os.time()` inside scripts, since script behavior must be reproducible for replication. Time should come from an argument passed in by the caller, or from `redis.call('TIME')`.

---

**11. When would you rate-limit per-IP instead of per-user, and what's the downside?**

*Answer:* Per-IP catches abuse from a source regardless of whether it's authenticated, useful before a user is even identified. The downside: many real users can share one IP (a corporate NAT, a campus network), so per-IP limiting can unfairly throttle all of them for one bad actor's traffic.

---

**12. What's the trade-off between hard-rejecting with a 429 and queuing an over-limit request instead?**

*Answer:* A hard reject is cheap and immediate but drops the request entirely. Queuing smooths bursts for the client at the cost of added latency, and the queue itself needs a bound — an unbounded queue just relocates the overload problem rather than solving it.

---

**13. What does HTTP 429 specifically mean, and what's a well-behaved companion to send with it?**

*Answer:* "Too Many Requests" — the client has exceeded the configured rate limit. A `Retry-After` header telling the client when it's safe to retry is standard practice alongside it.

---

**14. [Scenario] Two Gateway instances sit behind a load balancer, each still running Week 10's original in-memory Token Bucket. A client sends 150 requests against a limit of 100/minute, split evenly by the load balancer. What happens, and why?**

*Answer:* Roughly 75 requests hit each instance. Each instance's independent bucket (capacity 100) has plenty of room for 75, so all 150 requests are likely allowed — 1.5x the intended limit — because neither instance has any visibility into the other's count.

---

**15. Why would a distributed Sliding Window Log naturally reach for a Redis Sorted Set rather than a plain list?**

*Answer:* A Sorted Set keyed by timestamp supports efficiently removing everything below a score (evicting expired entries) and counting what remains, both without a full scan — exactly the two operations Sliding Window Log needs on every check. Full depth on Sorted Sets: Week 19, Day 132.

---

## What Tomorrow Assumes You Already Know Cold

Day 122 assumes today's core distributed-systems move — a shared, external, atomically-updated store standing in for what used to be safe in-memory state on a single instance — is now a reflex, since tomorrow's Notification System leans on Redis again, for a different purpose (deduplication) that uses the same underlying instinct: state that needs to be visible across instances doesn't live in process memory anymore. It also assumes the DLQ pattern (Week 14, Day 93) is solid, since tomorrow's retry-queue design cites it directly rather than re-deriving it.

**Next:** [Day 122 Resource Book](./Day122_Resource_Book.md) — Notification System, and HLD Mock #1.
