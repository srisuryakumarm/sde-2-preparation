# Day 117 Resource Book — LLD #8: BookMyShow, Part 2 — Concurrency, and Mock Interview #4

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 116 Resource Book](Day116_Resource_Book.md)
**Next ▶:** [Day 118 Resource Book](Day118_Resource_Book.md)
**Companion to:** Day 117 of `Week_17_Revised.md`

---

## Recap

Day 116 built a booking flow that passed every single-threaded test and then proved, via an exact traced interleaving, that two users could both successfully hold the same seat — the identical check-then-act shape as Parking Lot's race (Day 111) and the ATM's check-then-commit assumption (Day 113). Today closes that gap two different ways: **pessimistic locking** (a direct extension of Day 111's per-object locking, plus a genuinely new database-level variant) and **optimistic locking** (a genuinely new named pattern, though its actual mechanism — compare-and-swap — is not new at all, just applied somewhere new). Today also introduces one new concurrency primitive, `CountDownLatch`, needed to make the 10-thread test *actually* prove what it claims to prove.

## Learning Objectives

By the end of today, without notes:

1. Fix yesterday's race with an in-memory per-seat lock, citing Day 111's per-`ParkingSpot` technique directly rather than re-deriving why per-object locking works.
2. Explain precisely why a database-level lock (`SELECT FOR UPDATE`) is needed in addition to an in-memory lock — not as a redundant belt-and-suspenders habit, but because of a specific failure mode in-memory locking cannot cover.
3. Implement optimistic locking via a version-guarded compare-and-swap, and explain why it's the exact same CAS mechanism already used for `AtomicInteger` (Day 111, Week 6 Day 39), just applied to a database row's version column instead of an in-memory variable.
4. Explain why a naive loop of `.start()` calls doesn't reliably prove a concurrency fix works, and how `CountDownLatch` fixes that specific gap.
5. Justify, unprompted and with real trade-offs (not a coin flip), when pessimistic locking beats optimistic locking and vice versa.
6. Solve Longest Increasing Subsequence (LC 300) cold, in both the O(n²) and O(n log n) forms, explaining why the second one is correct, not just faster.

## Concept Dependency Map

```
Week 6  Day 37: synchronized keyword — mutual exclusion
Week 6  Day 39: ConcurrentHashMap's CAS; AtomicInteger
Week 15 Day 100: volatile / Java Memory Model — cross-thread visibility
Week 16 Day 111: Per-ParkingSpot locking (lock granularity); AtomicInteger for
                 ticket IDs; raw Thread + join() for the 10-thread test
Week 6  Day 36: @Entity/@Id/@Column — JPA annotations, read via reflection
Week 17 Day 116: The exact race this fixes, traced precisely
        │
        ▼
Today: BookMyShow, Part 2 — Concurrency
  ├─ Pessimistic locking
  │     ├─ In-memory per-seat lock — DIRECT extension of Day 111, no new
  │     │  mechanism, new domain
  │     └─ SELECT FOR UPDATE (NEW — DB-level row lock; needed because
  │        in-memory locks don't protect across multiple app server processes)
  │
  ├─ Optimistic locking (NEW named pattern — mechanism is NOT new: CAS,
  │  Day 111/Wk6 D39, applied to a version column instead of a variable)
  │     └─ @Version — a natural extension of @Entity/@Id (Wk6 D36)
  │
  ├─ CountDownLatch (NEW — a start-gate; Day 111's raw .start() loop
  │  doesn't guarantee true simultaneity, this does)
  │
  └─ Pessimistic vs. Optimistic — real trade-offs, not a coin flip
        │
        ▼
Mock Interview #4 — 60-min LLD mock, BookMyShow + concurrency, pushed on
                    pessimistic-vs-optimistic justification
        │
        ▼
DSA Revision: Longest Increasing Subsequence (LC 300)
  (needs: 1D DP — Wk12; binary search — Wk2)
```

---

# Part 1 — Pessimistic Locking

**Prerequisites confirmed:** per-object locking and lock granularity (Week 16, Day 111); `synchronized` (Week 6, Day 37).

## In-memory per-seat lock — a direct extension, not new material

Day 111 gave every `ParkingSpot` its own lock object so that locking one spot never blocked an unrelated spot. The identical idea, applied to seats:

```java
public class Show {
    ...
    private final Map<String, Object> seatLocks = new HashMap<>();

    public Show(String showId, Movie movie, Screen screen, LocalDateTime startTime) {
        ...
        for (Seat seat : screen.getSeats()) {
            seatStatus.put(seat.getSeatId(), SeatStatus.AVAILABLE);
            seatLocks.put(seat.getSeatId(), new Object());
        }
    }

    Object getSeatLock(String seatId) { return seatLocks.get(seatId); }
}
```

```java
public class PessimisticBookingService {
    public boolean holdSeat(Show show, String seatId, String userId) {
        Object lock = show.getSeatLock(seatId);
        synchronized (lock) {
            if (show.getSeatStatus(seatId) != SeatStatus.AVAILABLE) {
                return false;
            }
            show.setSeatStatus(seatId, SeatStatus.HELD);
            return true;
        }
    }
}
```

**Why this closes yesterday's exact race:** the check and the write are now inside the *same* `synchronized` block, guarded by *that specific seat's* lock object. A second thread attempting to lock the same seat blocks at `synchronized (lock)` until the first thread's block finishes entirely — check and write both — so there is no longer any window where two threads can both observe `AVAILABLE`. Locking a *different* seat is entirely unaffected, since it synchronizes on a different lock object — this is exactly Day 111's granularity argument, reapplied without modification.

## SELECT FOR UPDATE — genuinely new, and here's precisely why it's needed

An in-memory `synchronized` lock only protects against races **within a single JVM process.** A real booking service, at any meaningful scale, runs as **multiple separate application server instances** behind a load balancer — a user's request might land on server A, another user's request on server B. Server A's `synchronized` lock and server B's `synchronized` lock are two completely independent locks in two completely independent JVMs; **neither knows the other exists.** The exact race from Day 116 reopens at the process level, invisible to any single-server test.

```sql
BEGIN TRANSACTION;

SELECT status FROM seats
WHERE seat_id = 'seat5' AND show_id = 'show123'
FOR UPDATE;
-- This row is now exclusively locked by THIS transaction, at the database level —
-- visible to and enforced against every other connection, from any app server.
-- Any other transaction's SELECT ... FOR UPDATE on this same row BLOCKS here
-- until this transaction commits or rolls back.

-- application checks: is status == 'AVAILABLE'?

UPDATE seats SET status = 'HELD'
WHERE seat_id = 'seat5' AND show_id = 'show123';

COMMIT;
-- lock released here
```

**🔑 Key Takeaway:** the database is the one piece of shared state every application server instance actually agrees on — which is exactly why the lock has to live there, not in any one process's memory, the moment there's more than one process. `SELECT FOR UPDATE` extends Day 111's per-object-locking *idea* (lock only the specific resource in contention, not everything) into a context an in-memory `Object` lock structurally cannot reach.

**⚠️ Common Mistake:** treating the in-memory lock and the DB-level lock as redundant, or picking only one because "we already have the other." They protect against two different failure modes — same-process races (in-memory lock) and cross-process races (DB lock) — and a single-server toy deployment can get away with only the first, but a real multi-instance deployment needs both, or the first becomes theater.

---

# Part 2 — Optimistic Locking (New Named Pattern, Old Mechanism)

**Prerequisites confirmed:** compare-and-swap via `AtomicInteger` (Day 111; Week 6, Day 39's `ConcurrentHashMap`); `volatile` (Week 15, Day 100); `@Entity`/`@Id` JPA annotations (Week 6, Day 36).

## The idea: don't lock — verify at commit time, retry on conflict

Instead of blocking other threads out while checking and writing, optimistic locking lets every thread proceed **without any lock at all**, and instead attaches a **version number** to the row. A write only succeeds if the version hasn't changed since it was read — if it has, someone else got there first, and the caller retries against the fresh state.

**🔑 Key Takeaway — this is not a new mechanism.** It's the exact same compare-and-swap idea `AtomicInteger.compareAndSet` already uses (Day 111's ticket-ID generation; Week 6 Day 39's `ConcurrentHashMap` internals) — "attempt an update, but only if the value is still what I last saw it as" — applied to a database row's version column instead of an in-memory integer. What's new is the *name* and the *domain*, not the underlying idea.

```sql
SELECT status, version FROM seats WHERE seat_id = 'seat5' AND show_id = 'show123';
-- application reads: status = AVAILABLE, version = 7

UPDATE seats SET status = 'HELD', version = 8
WHERE seat_id = 'seat5' AND show_id = 'show123' AND version = 7;
-- If another transaction already updated this row since our SELECT (its version
-- is no longer 7), this UPDATE matches ZERO rows. The application checks the
-- affected-row count; 0 means "someone else moved first" — retry from the SELECT.
```

**In Java, without a real database** — the same idea, built on `AtomicInteger`:

```java
public class OptimisticSeatRecord {
    private volatile SeatStatus status = SeatStatus.AVAILABLE;
    private final AtomicInteger version = new AtomicInteger(0);

    public SeatStatus getStatus() { return status; }
    public int getVersion() { return version.get(); }

    /** Attempts AVAILABLE -> HELD, guarded by the version read at call time. */
    public boolean tryHold(int expectedVersion) {
        if (status != SeatStatus.AVAILABLE) {
            return false;                                  // genuinely taken — don't bother racing
        }
        if (!version.compareAndSet(expectedVersion, expectedVersion + 1)) {
            return false;                                   // version moved — stale read, caller retries
        }
        status = SeatStatus.HELD;
        return true;
    }
}

public class OptimisticBookingService {
    public boolean holdSeatWithRetry(OptimisticSeatRecord record, int maxRetries) {
        for (int attempt = 0; attempt < maxRetries; attempt++) {
            if (record.getStatus() != SeatStatus.AVAILABLE) {
                return false;   // actually taken — retrying changes nothing
            }
            if (record.tryHold(record.getVersion())) {
                return true;
            }
            // lost this round — someone else's compareAndSet won first; try again
        }
        return false;   // exhausted retries under heavy contention
    }
}
```

**Why this is safe despite the unsynchronized read of `status`:** the actual gate is `version.compareAndSet`, and `AtomicInteger`'s CAS is a single atomic hardware-level operation — exactly one competing thread can ever succeed when several race on the same `expectedVersion`, no matter how the earlier, unsynchronized reads happened to interleave. A stale read of `status` can only cause a *wasted* attempt (caught by the CAS failing), never an *incorrect* success.

**💡 Interview Insight — the ABA problem, worth naming if asked about CAS pitfalls:** a classic hazard with compare-and-swap is a value changing away and back to its original value between a read and a CAS attempt, fooling the CAS into succeeding when it shouldn't. That can't happen here specifically because `version` only ever increases by exactly 1 per successful update and is never reused or decremented — it can never cycle back to a previously-seen value, which is precisely what makes a monotonically increasing version counter a safe CAS guard.

**In production JPA:** `@Version` is a direct extension of the `@Entity`/`@Id` annotations already known (Week 6, Day 36) — annotate a field, and the ORM automatically appends `AND version = ?` to every `UPDATE`'s `WHERE` clause and increments it on success, throwing an `OptimisticLockException` on a zero-row update for the application to catch and retry.

```java
@Entity
public class SeatEntity {
    @Id
    private String id;
    private String status;
    @Version
    private int version;   // JPA handles the WHERE version = ? and the increment automatically
}
```

---

# Part 3 — Proving It: The 10-Thread Test, and Why `CountDownLatch` Is Needed

**Prerequisites confirmed:** `Thread`, `Runnable` (anonymous inner class), `.join()` — all Day 111.

## Why a plain `.start()` loop doesn't reliably prove anything

```java
// NOT what's being built — a naive loop like this can still often catch the bug,
// but isn't GUARANTEED to. Threads might be scheduled staggered enough that
// they never actually overlap, especially for an operation this fast — giving
// false confidence that the fix works when the test just got lucky.
for (Thread t : threads) { t.start(); }
```

`CountDownLatch` fixes exactly this gap. It's a counter, initialized to some value; `await()` blocks the calling thread until the counter reaches zero; `countDown()` decrements it by one. Used as a **starting gate**: every worker thread is started, immediately calls `startGate.await()`, and blocks there — genuinely all ten threads sit parked at the same line, having already done their setup — until a single `startGate.countDown()` releases every one of them at (as close to) the same instant as the JVM scheduler allows. That's a meaningfully stronger guarantee of real contention than hoping ten `.start()` calls happen to overlap.

```java
public class ConcurrencyTest {
    public static void main(String[] args) throws InterruptedException {
        Show show = buildTestShowWithOneSeat("seat5");
        PessimisticBookingService pessimisticService = new PessimisticBookingService();
        int threadCount = 10;
        CountDownLatch startGate = new CountDownLatch(1);
        AtomicInteger successCount = new AtomicInteger(0);

        Thread[] threads = new Thread[threadCount];
        for (int i = 0; i < threadCount; i++) {
            final int userIndex = i;
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        startGate.await();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                    boolean success = pessimisticService.holdSeat(show, "seat5", "user" + userIndex);
                    if (success) {
                        successCount.incrementAndGet();
                    }
                }
            });
            threads[i].start();
        }

        startGate.countDown();          // release all 10 at (as close to) once
        for (Thread t : threads) {
            t.join();                   // Day 111's completion-wait — unchanged, still correct
        }

        System.out.println("Successful holds: " + successCount.get());   // must be exactly 1, every run
    }
}
```

**🔑 Key Takeaway:** only *one* new primitive was needed — `CountDownLatch`, and only for the start gate. The completion wait still uses `.join()` exactly as Day 111 did; there was no reason to replace something that already works. Run this against the **broken** Day 116 version first — `successCount` will unpredictably read greater than 1 across repeated runs. Run it against `PessimisticBookingService` — `successCount` reads exactly 1, every single time, deterministically. That deterministic repeatability, not a single lucky passing run, is the actual proof. The identical test, pointed at `OptimisticBookingService.holdSeatWithRetry` instead, must also print exactly 1, every time — the CAS-based guard makes the same deterministic guarantee through a completely different mechanism.

---

# Part 4 — Pessimistic vs. Optimistic: The Real Trade-off

**⚠️ Common Mistake:** presenting this as a coin flip, or worse, as "optimistic is always better because it doesn't block." Neither is universally correct — the right choice depends on **how often two requests are actually expected to conflict.**

**Pessimistic locking** pays a lock-acquisition cost on *every* attempt, whether or not a conflict would have happened — under low contention, that's pure overhead for a race that was never going to occur. Under **high** contention, though, it's efficient: exactly the threads that would conflict simply queue and wait their turn, and no work is ever wasted on doomed attempts.

**Optimistic locking** pays nothing up front and lets every thread proceed freely — under low contention, most attempts succeed on the first try with zero locking overhead at all. Under **high** contention, many attempts fail their CAS and must retry, potentially several times, burning real CPU cycles on work that gets thrown away — in a pathological worst case (very high contention, low retry limits), a thread could exhaust its retries and fail outright even though the seat was, at some instant, genuinely available.

**The concrete answer for BookMyShow:** most seats, for most shows, most of the time, see essentially no contention — two users independently choosing the exact same seat within the same few-hundred-millisecond window is rare. **Optimistic locking is the right default.** The exception is a specific, identifiable scenario: a blockbuster's opening-day, first-few-seconds rush, where hundreds of users may genuinely race for the same handful of good seats simultaneously — that's exactly the high-contention regime where optimistic locking's retry storms become a real cost, and where pessimistic locking (or, in a real system, an upstream virtual waiting-room queue that never even lets that many concurrent requests reach the seat-selection step at all) is the better-justified choice. **Naming this — not just "pessimistic vs optimistic," but which one for which specific traffic pattern, and why — is what Mock Interview #4 is explicitly going to push on.**

---

# Mock Interview #4 — Logistics

**Format:** 60-minute LLD mock, BookMyShow as the subject, concurrency included by design. **What to expect:** the interviewer will very likely ask "what happens if two users book the same seat at the same time?" — Part 1–3 above is the material needed to answer that completely, not just gesture at "we'd need some kind of lock." **What's specifically being pushed on:** justifying pessimistic *or* optimistic with a real trade-off tied to expected traffic pattern (Part 4), not reciting both definitions and stopping there. Say the concrete BookMyShow answer (optimistic by default, pessimistic or a queue for a predictable high-contention hot spot) unprompted if the conversation gets there naturally.

---

# DSA Revision Block (1 hr)

## Longest Increasing Subsequence (LeetCode 300, Medium) — Pattern: 1D DP, with an O(n log n) Refinement

**Originally taught:** Week 12 (1D DP).

**Statement:** given an integer array, return the length of the longest strictly increasing subsequence.

**Approach 1 — O(n²) DP:** `dp[i]` = length of the longest increasing subsequence *ending exactly at index i*.

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1);
    int maxLen = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        maxLen = Math.max(maxLen, dp[i]);
    }
    return maxLen;
}
```

Every element starts as a subsequence of length 1 (itself). For each `i`, check every earlier `j`: if `nums[j] < nums[i]`, index `i` could extend whatever subsequence ends at `j`. **Complexity: O(n²)** time (nested loop over all pairs), **O(n)** space.

**Approach 2 — O(n log n), patience-sorting-style, with a real correctness argument:**

```java
public int lengthOfLIS(int[] nums) {
    List<Integer> tails = new ArrayList<>();
    for (int num : nums) {
        int pos = Collections.binarySearch(tails, num);
        if (pos < 0) {
            pos = -(pos + 1);
        }
        if (pos == tails.size()) {
            tails.add(num);
        } else {
            tails.set(pos, num);
        }
    }
    return tails.size();
}
```

**Why this is correct, not just faster — the invariant, proven by induction:** `tails` stays sorted throughout, and `tails[i]` always holds the *smallest possible* tail value achievable among every increasing subsequence of length `i+1` found so far. When a new number extends past every current tail, appending it is correct — a subsequence of a new maximum length now genuinely exists. When it doesn't extend the longest subsequence, replacing the first tail `≥` it with the smaller value can only *help* future extensibility (a smaller tail is at least as easy to extend further as a larger one) and never *hurts* — it doesn't fabricate a longer subsequence than actually exists, since `tails.size()` is left unchanged by a replacement. Since the invariant holds after every step, `tails.size()` equals the true LIS length once every element has been processed.

**Worked trace:** `nums = [10, 9, 2, 5, 3, 7, 101, 18]`.
```
10  → tails=[10]
9   → replaces 10 (9<10, same length-1 slot, smaller tail is better) → tails=[9]
2   → replaces 9 → tails=[2]
5   → extends (5>2) → tails=[2,5]
3   → replaces 5 (3 fits between 2 and 5) → tails=[2,3]
7   → extends (7>3) → tails=[2,3,7]
101 → extends → tails=[2,3,7,101]
18  → replaces 101 (18 fits between 7 and 101) → tails=[2,3,7,18]
Final: tails.size() = 4
```

**⚠️ Common Mistake — `tails` is not itself a valid LIS.** `[2,3,7,18]` is *a* valid increasing subsequence of length 4 here, but that's coincidental to this example — in general, `tails`'s *contents* don't necessarily form an actual subsequence of the original array in order; only its **length** is guaranteed correct. Reconstructing the actual subsequence (not just its length) needs extra bookkeeping (parent pointers alongside `dp[i]` in the O(n²) version) that this faster approach doesn't track.

**Complexity: O(n log n)** — n elements, each an O(log n) binary search. **Space O(n)** for `tails`.

**Edge cases:** a strictly decreasing array (`tails` never grows past length 1 — every new element replaces index 0); a single element (`tails.size() = 1` immediately); duplicate values (since the problem asks for *strictly* increasing, equal values never extend a subsequence — verify `Collections.binarySearch` correctly treats an exact match as "replace here," not "extend," which it does).

---

# Project Block Guide

**Repository:** `lld-java/bookmyshow/`. Add `PessimisticBookingService`, `OptimisticSeatRecord`/`OptimisticBookingService`, and the `CountDownLatch`-based concurrency test. **Definition of done:** the 10-thread test against the *broken* Day 116 service demonstrably (and non-deterministically) allows more than one success across repeated runs; the identical test against **both** the pessimistic and the optimistic implementation prints exactly 1 successful hold, **every single run** — deterministic repeatability is the actual bar, not a single passing execution. Trade-off write-up (Part 4 above, or your own version of it) committed alongside the code. Pushed.

# Career Block Guide

**LinkedIn (20 min):** engagement only — no new post required today.
**Networking:** continue outreach ahead of Day 118's application push.

---

# Day 117 — Interview Questions

**Q1. Why does an in-memory `synchronized` lock alone not fully solve this problem in a real deployment?** It only protects against races within a single JVM process. A real deployment runs multiple application server instances behind a load balancer, and two different processes' independent in-memory locks don't know about each other — the race reopens at the process level, invisible to any single-server test.

**Q2. What does `SELECT ... FOR UPDATE` actually lock, and for how long?** The specific matched row, at the database level, held for the duration of the current transaction — released on commit or rollback. Any other transaction's own `SELECT ... FOR UPDATE` against that same row blocks until this one finishes, regardless of which application server issued it.

**Q3. Is optimistic locking a new concurrency mechanism, or a new name for something already known?** A new name and a new domain — the mechanism is the exact same compare-and-swap idea already used by `AtomicInteger` (Day 111, Week 6 Day 39), just guarding a database row's version column instead of an in-memory integer.

**Q4. Why is a monotonically increasing version counter immune to the ABA problem?** ABA requires a value to change away and then back to something previously seen, fooling a CAS check. A counter that only ever increases by exactly one, never decremented or reused, can never return to a prior value — there's no "back" for it to return to.

**Q5. Why does a naive loop of `.start()` calls not reliably prove a fix works?** Threads might be scheduled with enough of a gap that they never actually overlap, especially for a fast operation — a passing run could just mean the race didn't happen to trigger, not that it can't. `CountDownLatch` used as a start gate forces every thread to block at the same point until released together, producing genuine, repeatable contention.

**Q6. In the `CountDownLatch` test, why does the completion wait still use `.join()` instead of a second `CountDownLatch`?** `.join()` already correctly solves that problem — it was established in Day 111 and nothing about today's scenario changes it. Only the start-side synchronization was a genuinely new need; introducing a second new primitive where an already-known one works would be unmotivated.

**Q7. When does pessimistic locking outperform optimistic locking, and why?** Under high contention — many genuine conflicts mean optimistic locking wastes real work on doomed attempts that fail their CAS and must retry, while pessimistic locking's queued waiting does no wasted work at all, just delay.

**Q8. When does optimistic locking outperform pessimistic locking, and why?** Under low contention — most attempts succeed on the first try with zero locking overhead, while pessimistic locking pays a lock-acquisition cost on every attempt regardless of whether a conflict would ever have actually occurred.

**Q9. What's BookMyShow's actual concrete answer, not just the abstract trade-off?** Optimistic locking by default, since most seats for most shows see negligible real contention — with pessimistic locking, or better, an upstream virtual waiting-room queue, reserved specifically for identifiable high-contention hot spots like a blockbuster's opening-seconds rush.

**Q10. In Longest Increasing Subsequence's O(n log n) approach, does the final `tails` array represent an actual valid subsequence of the input?** Not necessarily — only its *length* is guaranteed correct; `tails`'s specific contents don't reliably correspond to an actual increasing subsequence found in order within the original array. Reconstructing the real subsequence needs separate parent-pointer bookkeeping.

**Q11. Why is replacing an existing tail with a smaller value always safe, never harmful, in the O(n log n) approach?** It can only improve future extensibility (a smaller tail is at least as easy to build on as a larger one) and never fabricates a longer subsequence than actually exists, since a replacement leaves `tails.size()` — the tracked answer — completely unchanged.

---

# Daily Deliverable Check

- [ ] Pessimistic (per-seat in-memory lock + `SELECT FOR UPDATE` explained) and optimistic (version-guarded CAS) implementations both complete, pushed to `lld-java/bookmyshow/`.
- [ ] `CountDownLatch`-gated 10-thread test built; broken version shown non-deterministic, both fixed versions shown deterministically correct (exactly 1 success, every run).
- [ ] Trade-off write-up complete — the concrete BookMyShow answer (optimistic default, pessimistic/queue for hot spots), not just the abstract definitions.
- [ ] Mock Interview #4 completed — pessimistic-vs-optimistic justification pushed on and answered with real trade-offs.
- [ ] Longest Increasing Subsequence (LC 300) solved cold in both forms; the O(n log n) invariant argument explainable from memory, not just the code.

---

## What Tomorrow Assumes You Already Know Cold

Day 118 reuses Strategy a second time (Food Delivery's partner-matching), the pattern now fully established as reflexive after two full system-level uses. Nothing from today's concurrency material carries forward directly — Food Delivery doesn't need locking — but the discipline this week has built (naming a design's real trade-offs unprompted, the way Part 4 named pessimistic-vs-optimistic concretely rather than abstractly) is exactly what Day 118's Strategy-implementation choice (nearest-partner vs. highest-rated) will expect.

**Next:** [Day 118 Resource Book](Day118_Resource_Book.md) — Food Delivery System, and the System Design Framework Preview.
