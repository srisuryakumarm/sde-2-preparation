# Day 64 — Missing Primer: Resilience Patterns — Timeouts, Retries with Backoff and Jitter, Bulkheads, Load Shedding

**Do this first on Day 64 (≈ 40 min, before the Resilience4j theory block), then continue with Day 64's regular plan.**
**Needed by:** Day 64 itself (Resilience4j: circuit breaker, **time limiter**, retry, bulkhead — the book teaches the circuit breaker well and little else), Day 65 (Feign timeouts), Days 70 and 79 (saga steps are retried), Day 93 (Kafka retry/DLQ), Day 122 (notification retries), Day 138 (chaos experiments only make sense if you know the patterns they test), Day 142 (high availability), and the standard SDE-2 question: *"service B is slow — what happens to service A, and what do you do about it?"*
**Why it is missing:** the word **bulkhead** is never taught — it appears only twice in the whole repo, both in Day 42's primer (a "needed by" entry and a one-line pointer to Day 64); **jitter** and **exponential backoff** appear only inside the Kafka/notification discussions (Days 93 and 122), never as a general rule for service calls; timeouts and retries are never contrasted, and nothing explains how the patterns **compose** or why retries can make an outage worse.
**Already in place:** thread pools (Day 42's primer), the semaphore (Day 54's file — a bulkhead is built from it), HTTP status codes (Day 32's file), idempotency and at-least-once (Day 50's primer), microservices orientation (Day 61's primer). The circuit breaker itself is Day 64's book — read this file first.

---

## 1. The failure that all of this prevents
Service B becomes slow (not down). A's threads call B and wait. A's thread pool fills with waiting threads, A stops answering *its* callers, and the slowness **cascades** upward — one slow dependency takes down everything that calls it. Down is easy (you get a fast error). **Slow is the dangerous failure.**

## 2. Timeouts — the rule with no exceptions
**Every remote call needs a timeout**, and many clients default to *none* or to minutes.
- Two kinds: **connect timeout** (could not open the connection, ~1 s) and **read/response timeout** (waited for the answer, sized to the callee's p99 plus a margin). Plus an overall **deadline** for the whole user request.
- **Budgets shrink as you go deeper:** gateway 5 s → order-service 3 s → payment-service 1.5 s. The caller's timeout must be **larger** than the callee's *total* time including its retries, or the caller gives up while the callee is still working (wasted, duplicated work).
- Resilience4j's `TimeLimiter` wraps a `Future`/`CompletionStage`; for a plain blocking call the real timeout belongs on the **HTTP client** (Feign `connectTimeout` / `readTimeout`, Day 65).

## 3. Retries — powerful, and the most common way to cause an outage
**What to retry:** timeouts, connection resets, `503`, `429` (honour `Retry-After`). **Never retry** `4xx` validation errors (they will fail identically).
**When it is safe:** the operation is **idempotent** — `GET`, `PUT`, `DELETE` — or the request carries an **idempotency key** (`POST /payments` with `Idempotency-Key`, Day 50's primer; Day 130 builds on it). A retried non-idempotent `POST` without a key can charge a card twice.
**Exponential backoff:** wait longer after each failure — `delay = min(cap, base × 2^attempt)` — so a struggling dependency gets room to recover.
**Jitter:** if 1,000 clients all fail at the same moment and all wait exactly 200 ms, they retry in a synchronized wave and knock the service over again. Randomise the wait. The recommended form is **full jitter**: `sleep = random(0, min(cap, base × 2^attempt))` (see the AWS Architecture Blog post *Exponential Backoff and Jitter*).
```java
static <T> T retry(Supplier<T> call, int maxAttempts, long baseMillis, long capMillis) {
    RuntimeException last = null;
    for (int attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
            return call.get();
        } catch (RuntimeException e) {                         // real code: retry ONLY retryable errors
            last = e;
            if (attempt == maxAttempts) break;
            long ceiling = Math.min(capMillis, baseMillis * (1L << (attempt - 1)));   // base * 2^(attempt-1), capped
            long sleep = ThreadLocalRandom.current().nextLong(0, ceiling + 1);        // FULL jitter
            try { Thread.sleep(sleep); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); throw e; }
        }
    }
    throw last;
}
```
(Compiled and tested: succeeds on the 3rd attempt of a flaky supplier; rethrows the last error when attempts run out.)
**Retry amplification — the maths worth memorising:** if the gateway, order-service and payment-client each make 3 attempts, one slow payment call becomes **3 × 3 × 3 = 27** requests hitting the payment service at the worst moment. **Retry at one layer only** (usually the layer closest to the failing call, or the edge), keep attempts at 2–3, and use a **retry budget** (e.g. retries may be at most 10 % of traffic).

## 4. Bulkheads — isolate so one dependency cannot drown the rest
Named after the watertight compartments in a ship's hull. Give **each dependency its own limited resource** so a slow payment service can consume at most *its* share, not all of A's threads.
| Flavour | How | Trade-off |
|---|---|---|
| **Semaphore bulkhead** | cap *concurrent calls* (Day 54's `Semaphore`); runs on the caller's thread; rejects instantly when full (`BulkheadFullException`) or waits up to `maxWaitDuration` | cheap; does not isolate a *hung* thread |
| **Thread-pool bulkhead** | a dedicated pool + queue per dependency; returns a `CompletableFuture` | real isolation and timeouts; costs context switches and memory |

## 5. Circuit breaker — one table, then Day 64's book
`CLOSED` (normal; counting failures) → failure rate over threshold → `OPEN` (fail **immediately** for `waitDuration`, protecting both sides) → `HALF_OPEN` (let a few probe calls through) → success ⇒ `CLOSED`, failure ⇒ `OPEN`. It **stops** the pile-up; retries and timeouts only **bound** it.

## 6. How the patterns compose
Resilience4j's documented default decoration order (check the docs for your version) is
`Retry ( CircuitBreaker ( RateLimiter ( TimeLimiter ( Bulkhead ( Function ) ) ) ) )` — i.e. the bulkhead and time limiter are innermost, each *retry attempt* passes through the breaker (so failures are counted), and the retry sits outermost. Mental model, outside → in: *"try again, unless the breaker says stop; within a rate; within a deadline; within a concurrency cap."*
```yaml
resilience4j:                                   # property names as of Resilience4j 2.x; Day 63's file covers the Boot 4 starter
  retry.instances.payment:
    maxAttempts: 3
    waitDuration: 200ms
    enableExponentialBackoff: true
    exponentialBackoffMultiplier: 2
    enableRandomizedWait: true
    randomizedWaitFactor: 0.5
  bulkhead.instances.payment:
    maxConcurrentCalls: 10
    maxWaitDuration: 0
  circuitbreaker.instances.payment:
    slidingWindowSize: 10
    failureRateThreshold: 50
    waitDurationInOpenState: 10s
```

## 7. Load shedding, backpressure and fallbacks
- **Shed load early:** when saturated, answer `503` (or `429` per client) *immediately* rather than queue without limit — an unbounded queue turns overload into huge latency for everyone. Use **bounded** queues; reject the least important work first.
- **Backpressure:** let slow consumers slow producers (bounded queues, pull-based consumers such as Kafka).
- **Fallbacks / graceful degradation:** cached or default data, partial pages ("recommendations unavailable"), or marking an order `PAYMENT_PENDING` for later. A fallback must not call another unprotected dependency.
- **Health checks must not cascade:** a service whose database is down should report *not ready*, but a dependency being down must not make *liveness* fail and restart healthy pods (you meet probes in Kubernetes, Days 95–99).

## 8. Exercise (≈ 20 min)
1. **Paper:** draw `gateway → order-service → payment-service`. Give each hop a timeout (they must shrink), decide **which one hop retries** (and only on connection errors / `503`, only with an idempotency key), put a bulkhead of 10 around the payment call, and write the fallback for "payment too slow". State the worst-case number of requests the payment service sees per user click.
2. **Code (≈ 12 min, `java-fundamentals`):** type the `retry` helper above. Test it with a supplier that fails twice then succeeds, and one that always fails. Print each sleep and confirm it is random inside a growing ceiling.
3. Say aloud in 30 seconds: *"Why can retries make an outage worse, and what three things stop that?"* (retry at one layer + backoff with jitter + retry budget/circuit breaker).

## Checklist
- [ ] Can explain cascading failure and why "slow" is worse than "down".
- [ ] Can say connect vs read timeout and why budgets shrink down the call chain.
- [ ] Can state when a retry is safe, what to retry, and write backoff **with full jitter** from memory.
- [ ] Can compute retry amplification (3 layers × 3 attempts = 27) and name the fix.
- [ ] Can contrast semaphore and thread-pool bulkheads, and recite the circuit breaker's three states. Continue with Day 64's regular plan.
