# Day 54 — Missing Primer: `java.util.concurrent` Beyond Locks — Semaphore, ReadWriteLock, Barriers, and the Concurrent Collections

**Do this on Day 54 (≈ 40 min, any time before the Day 54 project block), then continue with Day 54's regular plan.**
**Needed by:** Day 64 (a Resilience4j **bulkhead** is a semaphore or a thread pool — that file builds on this one), Day 100 (Singleton thread-safety), Days 111 and 117 (LLD concurrency; Day 117 explains `CountDownLatch` as a start-gate and nothing else), Day 121 (rate limiter), Day 138 (the hand-built blocking queue), and the Java interview question "which concurrency utility would you use for X?".
**Why it is missing:** `Semaphore` appears in **none** of the 148 books. `ReadWriteLock` is only *named* in the Day 37 primer ("an extension"). `CyclicBarrier`, `CopyOnWriteArrayList`, `ConcurrentLinkedQueue`, `LongAdder`, `AtomicReference` and `ScheduledExecutorService` are never mentioned; `CountDownLatch` is taught on Day 117 for one purpose only. Day 42's primer says a bulkhead is "needed by Day 64" but no file ever defines one.
**Already in place:** threads (Day 29), races, `synchronized`, atomics and CAS (Day 37's primer), `ReentrantLock` (Day 37), `wait`/`notify` (Day 38), `ConcurrentHashMap` (Day 39), thread pools and `CompletableFuture` (Day 42's primer), fail-fast vs weakly-consistent iterators (Day 18's file).

---

## 1. Pick the tool by the question you are asking
| You need… | Use | One-line reason |
|---|---|---|
| exclusive access to a critical section | `synchronized` / `ReentrantLock` | one thread at a time (Day 37) |
| **many readers, rare writers** | `ReentrantReadWriteLock` | readers share, a writer is exclusive |
| **at most N threads at once** | `Semaphore(N)` | a counter of permits |
| wait until N events have happened (once) | `CountDownLatch(N)` | one-shot gate |
| N threads must all arrive before any continues, **repeatedly** | `CyclicBarrier(N)` | reusable meeting point |
| hand work from producers to consumers | `BlockingQueue` | `put`/`take` block (Day 38 by hand) |
| a counter many threads bump | `AtomicLong`, or `LongAdder` under heavy contention | no lock |
| a shared map | `ConcurrentHashMap` (Day 39) | per-bucket locking / CAS |
| a read-mostly list (listeners) | `CopyOnWriteArrayList` | snapshot iteration, O(n) writes |
| run something later / every N ms | `ScheduledExecutorService` | thread pool with a clock |

## 2. `Semaphore` — a bounded number of permits
`acquire()` takes a permit (blocks at zero), `release()` returns one, `tryAcquire()` returns `false` instead of blocking (also with a timeout). Uses: cap concurrent calls to a dependency, bound a pool of connections, throttle.
```java
class Bulkhead {
    private final Semaphore permits;
    Bulkhead(int maxConcurrent) { this.permits = new Semaphore(maxConcurrent); }

    <T> T call(Supplier<T> action) {
        if (!permits.tryAcquire()) {                       // acquire OUTSIDE the try …
            throw new IllegalStateException("bulkhead full");
        }
        try {
            return action.get();
        } finally {
            permits.release();                             // … so you only release what you acquired
        }
    }
}
```
Verified: 10 threads released at the same instant against `new Bulkhead(3)` with 200 ms of work → 3 succeed, 7 are rejected. This is exactly what Resilience4j's *semaphore bulkhead* does (`maxConcurrentCalls`; it rejects, or waits up to `maxWaitDuration`) — Day 64 uses it.
**Semaphore ≠ mutex:** a semaphore has no *owner* (any thread may `release()`), and is not reentrant; `Semaphore(1)` is a "binary semaphore", not a lock. Interview line: *"a mutex is for exclusion, a semaphore is for counting."*

## 3. `ReadWriteLock` — when reads dominate
```java
class ConfigCache {
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final Map<String, String> map = new HashMap<>();

    String get(String k)            { lock.readLock().lock();  try { return map.get(k); } finally { lock.readLock().unlock(); } }
    void   put(String k, String v)  { lock.writeLock().lock(); try { map.put(k, v); }     finally { lock.writeLock().unlock(); } }
}
```
Many threads can hold the **read** lock together; the **write** lock excludes everyone. A thread holding the write lock may take the read lock (**downgrade**); a reader cannot upgrade to a writer (it deadlocks against other readers). Only worth it when reads far outnumber writes *and* the guarded work is not trivial — otherwise plain `synchronized` is faster. `StampedLock` adds optimistic reads; know the name, skip the detail.

## 4. `CountDownLatch` vs `CyclicBarrier`
- **`CountDownLatch(n)`** — threads call `await()`; other threads call `countDown()`; when the count hits 0 everyone is released, **once, forever** (Day 117: a count of 1 is a start-gate that releases all test threads at the same instant).
- **`CyclicBarrier(n)`** — the *n* threads call `await()` and all block until the *n*th arrives; then all go, and the barrier **resets** for the next round (verified: three workers meet twice). Optional barrier action runs once per trip. Typical use: phased simulations.

## 5. The concurrent collections you should be able to name
- `ConcurrentHashMap` — Day 39. `ConcurrentSkipListMap/Set` — a *sorted* concurrent map (you meet skip lists on Day 132).
- `CopyOnWriteArrayList/Set` — every write copies the array; iterators see a snapshot (see Day 18's file). For listener lists, not for hot lists.
- `ConcurrentLinkedQueue/Deque` — lock-free, **non-blocking** queues (CAS); no waiting `take()`.
- `BlockingQueue`: `ArrayBlockingQueue` (bounded), `LinkedBlockingQueue` (optionally bounded), `PriorityBlockingQueue`, `SynchronousQueue` (a hand-off with zero capacity — `Executors.newCachedThreadPool` uses one).
- `Collections.synchronizedList/Map` — wrap every method in a lock, but **iteration and compound actions (`if (!m.containsKey(k)) m.put(k, v)`) still need your own lock** — the reason `ConcurrentHashMap.putIfAbsent`/`computeIfAbsent` exist.
- **Atomics:** `AtomicInteger/Long/Boolean/Reference` (CAS loops). **`LongAdder`** keeps several cells that are summed on read — much better than `AtomicLong` when many threads only *increment*; `sum()` is not an atomic snapshot (verified: 8 threads × 100,000 increments → exactly 800,000 after they finish).

## 6. `ScheduledExecutorService`
`schedule(task, delay)`, `scheduleAtFixedRate(task, initial, period)` (starts are evenly spaced; a slow run delays the next), `scheduleWithFixedDelay` (waits *after* each run finishes). **If a periodic task throws, all later runs are silently cancelled** (verified: a task that throws on run 2 never runs a 3rd time) — wrap the body in `try/catch`. Spring's `@Scheduled` (you use it on Days 130–131) is built on this.

## 7. Exercise (≈ 20 min, repo `java-fundamentals`)
1. Copy `Bulkhead`; run 10 threads released by a `CountDownLatch(1)` start-gate against `new Bulkhead(3)` with a 200 ms task; assert **3 ok / 7 rejected**. Then make the task throw and prove the permit is still released.
2. Copy `ConfigCache`; write one test with a writer thread and four reader threads; explain why `HashMap` inside is safe here.
3. Write, from memory, the table in §1 as five questions and answers ("N at once?" → …). Say one sentence on semaphore vs mutex and latch vs barrier.
4. Explain in two sentences why `LongAdder` can beat `AtomicLong`, and why `synchronizedList` is not enough for check-then-act.

## Checklist
- [ ] Can pick the right utility for "N at once", "wait for N", "N meet repeatedly", "mostly reads".
- [ ] Wrote a semaphore bulkhead with acquire outside / release in `finally`.
- [ ] Can say semaphore vs mutex, `CountDownLatch` vs `CyclicBarrier`, `CopyOnWriteArrayList` vs `synchronizedList`.
- [ ] Know that an exception cancels a periodic scheduled task. Continue with Day 54's regular plan.
