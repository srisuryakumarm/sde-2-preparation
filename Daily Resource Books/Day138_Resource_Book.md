# Day 138 — Chaos Engineering, Deepened; a Light Service Mesh; a Thread-Safe Queue Built by Hand

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 137 Resource Book](Day137_Resource_Book.md)
**Next ▶:** [Day 139 Resource Book](Day139_Resource_Book.md)
**Companion to:** Day 138 of `Week_20_Revised.md`

---

> ⚠️ **Flag, not a silent fix.** Today's plan describes itself as "deepening" a prior "single manual container kill" chaos test. `00_Curriculum_Map.md`'s detailed history doesn't corroborate a distinctly-logged chaos experiment anywhere before this week — the most plausible candidate is an informal check during Resilience4j's original introduction (Week 10, Day 64: confirming the circuit breaker actually trips when a dependency is stopped locally), small enough that it never became its own line item. Either way, it doesn't change today's actual work: the mechanism under real test today is the same Resilience4j Circuit Breaker taught in full on Day 64, regardless of how many times it was informally poked at before. Noted here and in the curriculum map, not silently assumed or silently corrected.

---

## Recap

Today reuses more prior material at once than almost any other day this week. From Week 10: the **Resilience4j Circuit Breaker** (Day 64) — CLOSED/OPEN/HALF_OPEN, proxy-based, tripped by a real failure rate over a sliding window — and **Spring Cloud Gateway** (Day 66). From Week 6: **`ReentrantLock`, `Condition`, and the Producer-Consumer exercise** (Days 37–38) — `notFull`/`notEmpty` conditions guarding a shared buffer, the direct ancestor of this afternoon's concurrency work. From Week 17: the **`CountDownLatch`-gated concurrent test** (Days 116–117) that proved BookMyShow's booking race was actually fixed — reused today to prove a data structure's invariants instead of a business-logic race. From Day 134: the reconcile loop, and the exact consequence of a Service having zero matching Pods — deliberately induced today instead of being an accident.

---

## Learning Objectives

By the end of today, without notes:

1. Explain what a chaos experiment is actually verifying, and why "the config says it should work" isn't the same claim as "it was observed to work under a real, induced failure."
2. Correctly induce a *sustained* dependency failure in Kubernetes (and explain why a naive attempt fails to sustain one at all), and read the resulting circuit breaker behavior against its actual configured thresholds.
3. Explain what a connection pool is, why it exists, and what "graceful degradation" versus "cascading failure" concretely looks like when one is saturated.
4. Explain what a service mesh sidecar moves out of application code and into infrastructure, and why that's a different kind of thing than a library like Resilience4j.
5. Build a generic, thread-safe bounded blocking queue from `ReentrantLock`/`Condition` alone, prove a specific naive version is broken via a real race, and prove the correct version's invariants hold under genuine concurrent contention.

---

## Concept Dependency Map

```
Week 6, Days 37-38: ReentrantLock, Condition, Producer-Consumer
                     (notFull / notEmpty, correctness argued not just run)
Week 10, Day 64: Resilience4j Circuit Breaker — CLOSED/OPEN/HALF_OPEN,
                 proxy-based, self-invocation caveat
Week 10, Day 66: Spring Cloud Gateway
Week 17, Days 116-117: CountDownLatch-gated 10-thread concurrent test
Day 134: reconcile loop; a Service with zero matching Pods
        │
        ▼
Day 138
        │
        ├── Chaos Engineering, deepened
        │     ├─ (a) Kill Payment sustainedly → observe Order's circuit breaker
        │     ├─ (b) Toxiproxy/Pumba latency → observe Gateway timeout behavior
        │     └─ (c) NEW: connection pooling, from zero → saturate it,
        │           observe graceful degradation vs. cascade
        │
        ├── Light Service Mesh (Istio/Linkerd)
        │     ├─ Sidecar injection — resilience moved OUT of application code
        │     └─ Weighted traffic split — a capability plain K8s Services lack
        │
        └── Databricks add-on: BoundedBlockingQueue<T>
              ├─ Broken (if, not while) version — a real race, traced
              ├─ Correct version — extends Day 38's exact mechanism
              ├─ Worked trace proving correctness on one interleaving
              └─ CountDownLatch-gated test proving it under real contention
        │
        ▼
Day 139: CI/CD pipeline finalized around this now chaos-tested, mesh-aware platform
```

---

# Part 1 — Chaos Engineering, Beyond One Manual Test

**What a chaos experiment actually verifies, stated precisely:** not "does the system have resilience configuration" — that's answered just by reading the code. A chaos experiment verifies that the *configured* resilience mechanism actually behaves as configured, **under a real, induced failure**, rather than only under the artificial conditions of a unit test (a mocked dependency, a manually-thrown exception). The gap between "this passes a unit test" and "this survives an actual dependency going down in a real cluster" is exactly what today closes.

Three distinct, semi-automated experiments, run **while a k6 load test is active against the Kubernetes-deployed platform** (Day 134's script, reused), so every experiment happens under realistic concurrent traffic rather than against an idle system.

---

# Part 2 — Experiment (a): Kill Payment, Observe Order's Circuit Breaker

**⚠️ Common Mistake, worth catching before it wastes the whole experiment:** the obvious first instinct is `kubectl delete pod <payment-pod-name>`. This does **not** produce a sustained failure — it's Day 134's own reconcile loop working exactly as designed: the ReplicaSet controller notices the missing Pod within seconds and creates a replacement. A single deleted Pod is a very brief blip, likely too short for Resilience4j's sliding window to even register enough consecutive failures to trip.

**The correct technique:** remove the *desired state* itself, so the reconcile loop has nothing to recreate:

```bash
kubectl scale deployment payment-service --replicas=0 -n ecommerce
```

Now Payment's Service has **zero** matching Pods — precisely Day 134's "empty Endpoints" scenario, deliberately induced this time. Every call Order makes to Payment fails with a connection error.

**🔗 Backward Reference (Week 10, Day 64):** what to actually observe, against real configured numbers, not vibes — pull Order's real `resilience4j.circuitbreaker` configuration — its `sliding-window-size`, `failure-rate-threshold`, and `wait-duration-in-open-state`. Watch the circuit breaker's state (exposed via `/actuator/health` or the Micrometer metrics already wired since Week 11) as load continues to hit it:

1. **CLOSED** — calls flow through normally, each one failing against the now-nonexistent Payment Service, each failure counted in the sliding window.
2. **OPEN** — once the failure rate crosses the configured threshold within the window, the breaker trips. Subsequent calls short-circuit immediately to the fallback method, **without attempting to reach Payment at all** — verify this concretely by confirming request latency for these calls drops sharply (a short-circuited call returns almost instantly; a call that actually attempts and fails against a truly-gone Service can take meaningfully longer, depending on how quickly the connection is refused).
3. **HALF_OPEN** — after the configured wait duration, scale Payment back up (`kubectl scale deployment payment-service --replicas=2 -n ecommerce`) and confirm the breaker allows a limited number of trial calls through, then transitions back to **CLOSED** once they succeed.

"Matched what Resilience4j was actually configured to do" means exactly this: the observed trip point and the observed recovery timing should line up with the *actual configured numbers* from Day 64 — not a vague "yes, it seemed to work."

---

# Part 3 — Experiment (b): Inject Latency Into Product, Observe Gateway Timeout Behavior

**Two tools, genuinely different mechanisms — worth being precise about which is which:**

- **Toxiproxy** sits *between* a service and its dependency as a TCP proxy; traffic has to be deliberately routed through it. In exchange, it offers fine-grained, on-demand "toxics" — an exact added latency, jitter, bandwidth caps, connection resets — controllable live via its own API, without touching either endpoint.
- **Pumba** operates directly against a running container's network namespace (using Linux `tc`/`netem` underneath), or can pause/kill containers outright. It doesn't require re-routing traffic through anything — simpler to point at an already-running deployment, at the cost of coarser control than Toxiproxy's per-connection toxics.

**The experiment:** inject several seconds of artificial latency into Product's responses (either tool), then watch the **Gateway's** behavior specifically — not Product's.

**What "correct" looks like:** if the Gateway has an explicit timeout configured (`spring.cloud.gateway.httpclient.response-timeout`, or a Resilience4j `TimeLimiter` wrapping the downstream call), it gives up waiting after that configured duration and returns a fast error or fallback to the *client* — it does not hang indefinitely just because Product happens to be slow right now.

**What "wrong" looks like, and why it's dangerous specifically under load:** with no timeout configured, the Gateway's request-handling threads (or reactive subscriptions) each hang for the full artificial delay, *per request*. Under Day 134's concurrent k6 load, this means many concurrent requests simultaneously tie up Gateway-side resources waiting on a dependency that's merely slow, not down — the exact same *shape* of danger as Experiment (c) below (resource exhaustion from a slow dependency), just at the Gateway tier instead of the database-connection tier.

---

# Part 4 — Connection Pooling, From Zero, and Experiment (c): Saturate It

This concept hasn't come up anywhere earlier in the series — worth building properly before chaos-testing it.

## Why Not Just Open a New Database Connection Per Request?

A brand-new database connection means a fresh TCP handshake plus the database's own authentication handshake — real, measurable overhead (commonly tens of milliseconds) paid **every single time**, on top of whatever the actual query itself costs. At any meaningful request volume, paying that cost per-request is unacceptable — the fix is to keep a set of already-established, already-authenticated connections open and **reuse** them.

## The Mechanism: HikariCP

Spring Boot's default connection pool (since Spring Boot 2.0) maintains a fixed-size set of live database connections, sized by `spring.datasource.hikari.maximum-pool-size`. A thread needing to run a query **borrows** a connection from the pool, uses it, and **returns** it when finished — it does not own that connection permanently. If every connection is currently borrowed when a new request needs one, the requesting thread **waits**, up to `connectionTimeout` (30 seconds by default), for one to be returned. If none frees up in time, it throws a `SQLTransientConnectionException` — the pool is **exhausted**.

**🔑 Key Takeaway:** a connection pool turns "how many concurrent database operations can this module actually sustain" into a real, fixed, configured number — smaller than the number of concurrent HTTP requests the module might otherwise try to handle at once. That gap is exactly what makes saturation possible, and exactly what today's experiment deliberately exercises.

## The Experiment

While k6 load continues, drive enough concurrent, DB-bound requests at Order to hold **every** connection in its pool simultaneously (either through raw concurrency alone, or by adding an artificial delay to a query to hold connections longer, borrowing directly from Experiment (b)'s latency-injection idea).

**What "fails gracefully" looks like:** new requests needing a connection queue briefly, then either succeed once a connection frees up, or time out cleanly with a clear error — and, critically, **Order's ability to serve requests that don't need the database at all is unaffected.**

**What "cascades" looks like — the more dangerous, more subtle failure:** threads blocked waiting on a database connection are still threads — if enough of them pile up waiting, they can exhaust the module's own HTTP-request-handling thread pool (e.g., Tomcat's worker threads) too. Once *that's* exhausted, the module can no longer accept **any** new request, including ones that never needed the database at all — a single saturated resource quietly takes the entire module down with it, and nothing here ever returns an explicit "the database is down" signal the way Experiment (a)'s dependency failure did. This is a genuinely different failure *shape* from a dependency being down: nothing looks unhealthy from a status-code point of view while requests are simply queuing, until the whole module stops responding at once.

**⚠️ Common Mistake:** treating "no errors yet" as "no problem." A saturating-but-not-yet-exhausted pool produces rising latency with zero errors — exactly the kind of thing Day 137's tracing (and Day 140's bottleneck analysis) exists to catch before it becomes a full cascade.

**Document all three experiments in `docs/chaos-test-results.md`:** what failed, what didn't, and whether the observed fallback behavior matched what Resilience4j was actually configured to do — evidence, not claims.

---

# Part 5 — A Light Service Mesh

**The core idea:** a service mesh (Istio or Linkerd) injects a lightweight proxy — a **sidecar** — into every Pod, alongside the application container. Every byte of network traffic in and out of that Pod passes through the sidecar transparently; the application code makes calls exactly as it always did, with no awareness the sidecar even exists.

**What this genuinely moves, architecturally — worth stating precisely:** Resilience4j's Circuit Breaker (Day 64) is a **library**, living inside the application's own JVM, in application-level configuration. A service mesh can implement conceptually similar cross-cutting behavior — retries, timeouts, traffic-based routing decisions — **entirely outside application code, at the network layer**, uniformly, regardless of what language or framework any given service happens to be written in. That's a genuinely different kind of thing than a library dependency, not merely a fancier version of the same idea — worth being able to name this distinction directly if asked "how is this different from what Resilience4j already does."

**Sidecar injection, mechanically:** a namespace gets labeled for automatic injection (Istio: `istio-injection=enabled`); from then on, a mutating admission webhook automatically adds the sidecar container to every Pod created in that namespace — the application's own Deployment manifest doesn't change at all. **⚠️ Common Mistake:** this only applies at Pod-*creation* time. An already-running Pod does not retroactively gain a sidecar just because the namespace label was added afterward — it needs a rollout restart (`kubectl rollout restart deployment/...`) to be recreated under the new injection rule.

**The traffic-split demo:** a plain Kubernetes Service (Day 134) load-balances roughly evenly across every Pod matching its selector, with no concept of "90% here, 10% there." A service mesh's own routing resources add exactly that capability — deploy `product-service-v1` and `product-service-v2` (a trivially distinguishable change, e.g., a different response header, is enough to verify the split visually), and configure a 90/10 weighted split between them. This is the real payoff: progressive rollouts become possible — route a small percentage of real traffic to a new version, watch its behavior via the mesh's own automatic observability, and ramp up only once it's confirmed healthy — a capability genuinely absent from plain Kubernetes Services.

**⚠️ Common Mistake, an honest trade-off rather than a free upgrade:** every request now passes through an additional proxy hop. This is a real, if typically small, added latency and resource cost on **every single call** — worth stating plainly rather than presenting the mesh as a strict improvement with no cost.

**Definition of done:** sidecar injection and a 90/10 traffic split demonstrated and screenshotted.

**💡 Interview Insight:** "What's the difference between a circuit breaker in your application code and what a service mesh gives you?" is a real, fair follow-up once both appear on a resume. The strong answer is today's library-versus-infrastructure distinction, stated precisely — not "the mesh is more advanced."

---

# Part 6 — Databricks Add-On: A Hand-Rolled Thread-Safe Bounded Blocking Queue

**Why this specific exercise, stated honestly:** real candidate reports describe Databricks' concurrency round as its hardest — a real implementation (a thread-safe logger, a bounded queue with backpressure), not a LeetCode pattern to recognize. Today's chaos experiments already exercised concurrent-failure thinking under load; this closes the loop with an actual from-scratch build. This is directly, explicitly an extension of **Week 6, Day 38's Producer-Consumer** exercise — `ReentrantLock` plus two `Condition`s (`notFull`, `notEmpty`) guarding a shared buffer, correctness argued rather than just run. Today formalizes that exact mechanism into a real, generic, tested class — **not** new concurrency primitives.

**🔗 Backward Reference (Week 6, Day 38):** every mechanism this section uses — `ReentrantLock`, `Condition`, the `notFull`/`notEmpty` naming itself — is Day 38's Producer-Consumer exercise, unchanged. Today formalizes it; it doesn't introduce it.

## The Task

Implement a generic `BoundedBlockingQueue<T>`:

- `void put(T item)` — blocks if the queue is full until space is available, then adds the item.
- `T take()` — blocks if the queue is empty until an item is available, then removes and returns it.
- Fixed capacity, set at construction.
- **No `java.util.concurrent` queue classes** — `ArrayBlockingQueue`/`LinkedBlockingQueue` are exactly what this exercise is proving you can build from first principles; using them would defeat the entire point.

## The Broken First Attempt — and the Real Race It Contains

A very common instinct: guard the wait with `if` instead of `while`.

```java
public class BrokenBoundedQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;

    public BrokenBoundedQueue(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void put(T item) throws InterruptedException {
        if (queue.size() == capacity) {
            wait();                       // BUG
        }
        queue.add(item);
        notifyAll();
    }

    public synchronized T take() throws InterruptedException {
        if (queue.isEmpty()) {
            wait();                       // BUG
        }
        T item = queue.poll();
        notifyAll();
        return item;
    }
}
```

**The race, traced concretely — capacity = 1, queue currently full with one item:**

Two producer threads, P1 and P2, both call `put()`.

1. P1 acquires the monitor, sees `size() == capacity` (true), calls `wait()` — releases the monitor and suspends.
2. P2 acquires the (now-free) monitor, **also** sees `size() == capacity` (still true — nothing has been consumed yet), **also** calls `wait()`. Both P1 and P2 are now waiting.
3. A consumer thread removes the one item (`queue` is now empty), calls `notifyAll()` — waking **both** P1 and P2.
4. **The bug:** a thread woken from `wait()` does not automatically re-check anything — it simply resumes at the line immediately after `wait()`. Since the guard was an `if`, **neither** P1 nor P2 re-verifies `size() == capacity` before proceeding. Both add their item directly. With `capacity = 1`, the queue now holds **2 items** — the bounded invariant is violated, silently, with no exception thrown anywhere.

**🔑 Key Takeaway:** waking from `wait()`/`notifyAll()` means "the condition *might* have changed, go check again" — never "the condition is now false, proceed." A woken thread can be racing another woken thread for the same now-possibly-already-claimed resource, and only re-checking the actual condition (a `while` loop) closes that gap. This is the single most important correctness rule for `wait()`/`notify()`-based code, demonstrated here as a genuine violation rather than asserted as a rule to memorize.

## The Correct Version

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class BoundedBlockingQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public BoundedBlockingQueue(int capacity) {
        if (capacity <= 0) {
            throw new IllegalArgumentException("capacity must be positive");
        }
        this.capacity = capacity;
    }

    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await();
            }
            queue.add(item);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            T item = queue.poll();
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }

    public int size() {
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }
}
```

**Every deliberate design choice, explained — not just restated from Day 38, but justified for this specific class:**

- **`while`, not `if`** — fixes the exact race traced above; every waking thread re-verifies reality before touching shared state.
- **Two separate `Condition`s, not one** — this is Day 38's own reasoning, reused unmodified: a blocked `put()` only cares "is there space," a blocked `take()` only cares "is there an item." One shared condition would wake *every* waiter — producers and consumers alike — on *every* change, regardless of relevance, adding needless wake-and-recheck overhead under contention. Two conditions let each `signal()` target exactly the waiters who might actually be unblocked by this specific state change.
- **`signal()`, not `signalAll()`** — safe here specifically because of an exact one-to-one correspondence: each successful `put()` creates exactly one new "an item is available" fact, so waking exactly one `notEmpty` waiter is both sufficient and correct (not an under-wake); the symmetric argument holds for `take()` and `notFull`. `signalAll()` would also be safe (the `while` guard means an unnecessarily-woken thread simply re-blocks) but wastes CPU waking threads that have nothing to do — worth being able to state both are *correct* and explain precisely why `signal()` is the more efficient, still-justified choice, rather than just asserting it's "faster."
- **`lock.unlock()` inside `finally`** — unlike `synchronized`, a `ReentrantLock` is **not** released automatically if an exception is thrown inside the critical section. Skipping the `finally` here means one exception permanently deadlocks every future caller of this queue — a `finally`-guaranteed unlock isn't a style preference, it's the only thing preventing a single failure from being unrecoverable.
- **`throws InterruptedException`, propagated rather than swallowed** — an empty `catch (InterruptedException e) {}` silently discards the thread's interrupt status, which can cause a caller relying on interruption for cancellation to hang indefinitely instead. Letting it propagate (or, where it must be caught, calling `Thread.currentThread().interrupt()` to restore the flag) is the standard, correct discipline.

## Worked Trace: Proving Correctness on One Real Interleaving

**Setup:** `capacity = 2`, queue starts empty.

| Step | Thread | Action | Queue state after |
|---|---|---|---|
| 1 | P1 | `put(A)` — `size()`=0≠2, skips loop, adds A, signals `notEmpty` | `[A]` |
| 2 | P2 | `put(B)` — `size()`=1≠2, skips loop, adds B, signals `notEmpty` | `[A, B]` |
| 3 | P3 | `put(C)` — `size()`=2==2, **enters loop**, calls `notFull.await()`, releases lock, suspends | `[A, B]` (P3 waiting) |
| 4 | C | `take()` — lock free (P3 released it on await), `size()`=2≠0, skips loop, polls A, calls `notFull.signal()` (wakes P3), returns A, unlocks | `[B]` |
| 5 | P3 | Wakes, **re-acquires** lock, **re-checks** loop condition: `size()`=1≠2 → **exits loop this time** (correctly — there's genuinely room now), adds C, signals `notEmpty`, unlocks | `[B, C]` |

**Final state:** `[B, C]`, size 2, capacity 2 — invariant intact throughout, no item lost, no capacity violation, no deadlock. Step 5 is the direct, provable contrast with the broken version's Step 4 above: P3 re-checks reality before proceeding, and because exactly one slot genuinely freed up (Step 4's `take()`), exactly one waiting producer correctly proceeds — not zero, not both.

## Proving It Under Genuine Concurrent Contention, Not Just One Hand-Traced Interleaving

The worked trace above proves one specific interleaving is handled correctly. A real proof needs many threads, genuinely racing, not a single hand-picked sequence — the same standard already set for this platform's own concurrency work: **the identical `CountDownLatch`-gated pattern first used to prove BookMyShow's booking race was actually fixed (Week 17, Days 116–117)**, reused here to prove a data structure's invariants instead of a business-logic race.

```java
@Test
void provesCorrectnessUnderConcurrentAccess() throws InterruptedException {
    int capacity = 5;
    int producers = 4;
    int itemsPerProducer = 250;
    int totalItems = producers * itemsPerProducer;

    BoundedBlockingQueue<Integer> queue = new BoundedBlockingQueue<>(capacity);
    CountDownLatch startLatch = new CountDownLatch(1);      // releases every thread at once
    CountDownLatch doneLatch = new CountDownLatch(producers + 1);
    List<Integer> consumed = Collections.synchronizedList(new ArrayList<>());

    for (int p = 0; p < producers; p++) {
        int producerId = p;
        new Thread(() -> {
            try {
                startLatch.await();
                for (int i = 0; i < itemsPerProducer; i++) {
                    queue.put(producerId * itemsPerProducer + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                doneLatch.countDown();
            }
        }).start();
    }

    new Thread(() -> {
        try {
            startLatch.await();
            for (int i = 0; i < totalItems; i++) {
                consumed.add(queue.take());
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            doneLatch.countDown();
        }
    }).start();

    startLatch.countDown();     // maximize real contention — everyone starts together
    doneLatch.await();

    assertEquals(totalItems, consumed.size());                       // nothing lost
    assertEquals(totalItems, new HashSet<>(consumed).size());        // nothing duplicated
    assertEquals(0, queue.size());                                   // nothing stuck
}
```

**Why `startLatch` matters, precisely:** without it, threads start sequentially, one JVM thread-creation call after another — real overlapping contention on the lock is left to chance. Gating every thread behind one shared latch and releasing them together forces genuine simultaneous contention for the same lock, which is exactly the condition under which the broken `if`-based version's race would actually manifest — a test that doesn't force this could pass against broken code purely by luck.

**What the three assertions together actually prove:** every item put in was eventually taken out exactly once (no loss, from the count matching; no duplication, from the set-size matching), and nothing was left stranded (the queue is empty at the end) — a genuine, evidence-based correctness claim, not "it ran without throwing."

## Complexity, and a Distinction Worth Naming Precisely

**Time, on the non-blocking fast path:** O(1) — `LinkedList` add/poll from the appropriate end are O(1), and acquiring an uncontended lock is fast, with no OS-level blocking involved.

**Time, when actually blocked:** this doesn't have a classical Big-O answer, and saying so explicitly is itself the correct, sophisticated answer. Concurrent correctness has two genuinely different dimensions: **safety** (nothing bad ever happens — capacity is never exceeded, no item is ever lost or duplicated) and **liveness** (something good eventually happens — a blocked thread doesn't wait forever while the system as a whole keeps making progress). Big-O as used for single-threaded algorithms describes neither of these directly; naming this distinction explicitly, rather than forcing a time-complexity answer where one doesn't cleanly apply, is a stronger signal than a confident-but-wrong number.

**Space:** O(capacity) — bounded by construction, unlike an unbounded queue.

## Common Mistakes and Edge Cases — Full Checklist

- `if` instead of `while` for the wait guard (proven above).
- `lock.unlock()` outside a `finally` block — one exception mid-critical-section permanently deadlocks every future caller.
- Swallowing `InterruptedException` silently instead of propagating it or restoring the interrupt flag.
- Not validating `capacity <= 0` at construction (handled above).
- Assuming `signalAll()` would be *wrong* here — it wouldn't be incorrect, only less efficient; conflating "less efficient" with "wrong" overstates the actual issue.

**💡 Interview Insight:** the strongest opening move, before writing a line of code, is stating the invariant being protected (capacity never exceeded, nothing lost) and naming the two-condition design and the `while`-not-`if` rule **before** it's needed, rather than discovering the race live under interviewer pressure. Two very likely follow-ups, both worth having a ready answer for: **"what actually prevents two `put()` calls from racing each other?"** — the single shared `ReentrantLock` fully serializes every critical-section entry across both `put` and `take`; there is no genuine simultaneity inside the critical section, only apparent concurrency from outside it. And **"why `ReentrantLock` here instead of plain `synchronized`, if `synchronized` also supports `wait`/`notify`?"** — because `synchronized` gives exactly one implicit condition (one wait-set) per monitor; this design fundamentally needs *two* logically separate wait conditions, which only `Lock.newCondition()` provides — not a vague "`ReentrantLock` is more powerful," but the specific capability this exact design requires.

---

# Project Block Guide (3.5 hrs)

**Repository:** `scalable-ecommerce-platform` (chaos experiments and service mesh); `java-fundamentals` (the queue).

**Task 1:** while Day 134's k6 script runs against the Kubernetes-deployed platform, execute all three chaos experiments above, in order. Document findings in `docs/chaos-test-results.md` — what failed, what didn't, and whether the observed behavior matched Resilience4j's actual configured thresholds.

**Task 2:** install Istio or Linkerd on Minikube; inject sidecars into every Pod; demonstrate and screenshot a 90/10 traffic split between two versions of one module.

**Task 3:** implement `BoundedBlockingQueue<T>` in `java-fundamentals`, exactly as built above, with the `CountDownLatch`-gated concurrent test proving correctness.

**Definition of done:** all three chaos experiments documented with real evidence; service mesh sidecar injection and traffic split demonstrated and screenshotted; the queue implementation pushed with a passing concurrent test.

---

# Career Block Guide (1 hr)

**LinkedIn — Post 27:** "I ran three chaos experiments against my own Kubernetes deployment — here's what broke and what didn't." A genuinely differentiated post specifically because it's backed by real evidence from today, not a claim.

**Networking:** continue the application push across all 7 target companies.

---

# Day 138 — Interview Questions

**Q1. What does a chaos experiment actually verify that a unit test of the same resilience mechanism doesn't?**
*Answer:* A unit test verifies the mechanism behaves correctly under artificial, controlled conditions (a mocked failure). A chaos experiment verifies it behaves the same way under a real, induced failure in the actual running system — closing the gap between "configured to work" and "observed to work."

**Q2. Why doesn't `kubectl delete pod` alone produce a sustained failure for a chaos test?**
*Answer:* The ReplicaSet controller's reconcile loop notices the missing Pod within seconds and creates a replacement automatically. A sustained failure requires removing the desired state itself — scaling the Deployment to zero replicas — not deleting a single Pod instance.

**Q3. What does a connection pool's `maximum-pool-size` actually bound, and what happens when a request needs a connection and none is free?**
*Answer:* It bounds how many concurrent database operations the module can sustain at once, independent of how many concurrent HTTP requests it might otherwise accept. A request needing a connection when the pool is fully borrowed waits up to `connectionTimeout`; if none frees up in time, it throws a pool-exhausted exception.

**Q4. Why can connection pool saturation cascade into total module unavailability, even for requests that never touch the database?**
*Answer:* Threads blocked waiting for a database connection are still occupying the module's own request-handling thread pool. If enough pile up, that thread pool itself gets exhausted, leaving no threads available to handle any request — including ones with no database dependency at all.

**Q5. What's the architectural difference between Resilience4j's circuit breaker and what a service mesh sidecar provides?**
*Answer:* Resilience4j is a library living inside the application's own process, configured in application code. A service mesh sidecar implements similar cross-cutting behavior (retries, timeouts, routing) entirely outside application code, at the network layer, uniformly across services regardless of language or framework.

**Q6. Why can't a plain Kubernetes Service do a 90/10 weighted traffic split on its own?**
*Answer:* A plain Service load-balances roughly evenly across every Pod matching its selector, with no concept of differential weighting between subsets. Weighted splitting requires a service mesh's own routing resources on top of the base Service/Pod model.

**Q7. Trace the exact race in a `wait()`/`notify()` bounded queue that uses `if` instead of `while` to guard the wait.**
*Answer:* With capacity 1 and the queue full, two producers can both see the full condition and both call `wait()`. A single `notifyAll()` after one item is consumed wakes both; since the guard was `if`, neither re-checks the condition before adding — both proceed, and the queue ends up with 2 items in a capacity-1 queue.

**Q8. Why does this bounded queue implementation use two separate `Condition` objects instead of one?**
*Answer:* A blocked `put()` only needs to know "is there space now," and a blocked `take()` only needs to know "is there an item now." A single shared condition would wake every waiter — producers and consumers alike — on every change regardless of relevance; two conditions let each `signal()` target only the waiters that specific change could actually unblock.

**Q9. Is `signal()` safe to use here instead of `signalAll()`, and why?**
*Answer:* Yes — each successful `put()` creates exactly one new "an item is available" fact, so waking exactly one `notEmpty` waiter is sufficient and correct (the symmetric argument holds for `take()`/`notFull`). `signalAll()` would also be correct, just less efficient, since the `while` guard makes an unnecessarily-woken thread simply re-block.

**Q10. Why must `lock.unlock()` be called inside a `finally` block for a `ReentrantLock`, when `synchronized` doesn't need the equivalent?**
*Answer:* `synchronized` releases its monitor automatically on any exit path, including an exception. `ReentrantLock` does not — if an exception is thrown inside the critical section and `unlock()` isn't in a `finally`, the lock is held forever, deadlocking every future caller.

**Q11. In the concurrent test proving the queue's correctness, why gate every thread behind a shared `CountDownLatch` instead of just starting them normally?**
*Answer:* Threads started normally begin sequentially, with real contention left to chance. A shared latch releases every thread simultaneously, forcing genuine overlapping contention for the same lock — exactly the condition under which a race like the broken `if`-based version would actually manifest; without it, a flawed implementation could pass purely by luck.

**Q12. What does "Big-O of `put()`" mean when the call might block, and why is that a slightly different kind of question than usual?**
*Answer:* The non-blocking fast path is O(1). While blocked, the relevant properties are safety (the invariant is never violated) and liveness (the thread eventually proceeds once the system makes progress) — properties classical Big-O, built for single-threaded algorithms, doesn't directly capture. Naming that distinction is the correct answer, not forcing a number onto a question that isn't really asking for one.

---

## Daily Deliverable Check

- [ ] All three chaos experiments executed and documented in `docs/chaos-test-results.md` with real evidence.
- [ ] Can explain why a single `kubectl delete pod` fails to sustain a chaos test, and the correct fix.
- [ ] Can explain connection pool saturation's cascade failure mode, distinct in kind from a dependency simply being down.
- [ ] Service mesh sidecar injection and a 90/10 traffic split demonstrated and screenshotted. LinkedIn Post 27 published.
- [ ] `BoundedBlockingQueue<T>` implemented in `java-fundamentals`, with the broken `if`-based version's race understood concretely, not just avoided by luck.
- [ ] `CountDownLatch`-gated concurrent test passing, proving no loss, no duplication, nothing stuck.

---

## What Tomorrow Assumes You Already Know Cold

Day 139 wires today's chaos-tested, mesh-aware platform into a real CI/CD pipeline and adds readiness/liveness probes — it assumes today's Resilience4j-under-real-failure results and the connection-pool-saturation distinction are solid, since tomorrow's discussion of *why* a coverage floor and startup-sequencing matter draws directly on "a config that looks right on paper can still fail in ways only real testing reveals" — today's actual lesson, not a new one tomorrow re-derives.
