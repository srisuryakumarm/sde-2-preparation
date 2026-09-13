# Day 111 — LLD #3: Parking Lot, Part 2 — Concurrency

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 110 Resource Book](Day110_Resource_Book.md)
**Next ▶:** [Day 112 Resource Book](Day112_Resource_Book.md)
**Companion to:** Day 111 of `Week_16_Revised.md`

---

## Recap

Yesterday's `ParkingLot` is correct — for one thread at a time. Today's entire job is finding, proving, and fixing exactly where it stops being correct once multiple threads compete for the same spots, using material this series has already fully paid for: `synchronized` and `ReentrantLock` (Week 6, Day 37), `wait()`/`notify()` (Week 6, Day 38), `ConcurrentHashMap`'s CAS-based bucket locking (Week 6, Day 39), and `volatile`/the Java Memory Model (Week 15, Day 100). One genuinely new class appears today — `AtomicInteger` — introduced explicitly as *new API, already-understood mechanism*: it uses the same Compare-And-Swap primitive Day 39 already opened up inside `ConcurrentHashMap`, not a new concept from zero.

---

## Learning Objectives

By the end of today, without notes:

1. Identify the exact check-then-act race condition in yesterday's `parkVehicle`/`findAvailableSpot` pair, and explain precisely how two threads could both be assigned the same spot.
2. Fix it with per-spot locking, and justify why that granularity beats locking the whole `ParkingLot` or each `Level`.
3. Write a concurrency test that reliably proves no double-booking across repeated runs, using only already-taught primitives (`Thread`, `synchronized`, `wait`/`notifyAll`, `Thread.join`).
4. Explain why `parkedVehicle` doesn't need `volatile` once every read and write to it goes through a `synchronized` method.
5. Solve Redundant Connection (LC 684) cold, without hints.

---

## Concept Dependency Map

```
Week 6 Day 37: synchronized, ReentrantLock
Week 6 Day 38: wait() / notify() / notifyAll()
Week 6 Day 39: ConcurrentHashMap — CAS, bucket-level locking
Week 15 Day 100: volatile, the Java Memory Model, happens-before
Week 11 Day 74: Union-Find — path compression, union by rank (today's revision problem)
Day 110: single-threaded ParkingLot — correct for one thread, not yet proven for many
        │
        ▼
Today: the race, found and proven
  check-then-act: findAvailableSpot() (read) then assignVehicle() (write),
  as two SEPARATE, non-atomic steps — the exact gap two threads can both land in
        │
        ▼
The fix: per-ParkingSpot locking (synchronized tryAssign/removeVehicle)
  AtomicInteger for ticket IDs (NEW class — built on Day 39's already-taught CAS)
  ConcurrentHashMap for the active-tickets map (Day 39, reused directly)
        │
        ▼
Where to Put the Lock — whole-lot vs. per-level vs. per-spot, contention vs. correctness
        │
        ▼
🔗 Forward: Day 112's Library Management System is this week's last new LLD system,
   and Mock Interview #2 explicitly pushes "now make it thread-safe" mid-interview —
   today's reasoning is the material that mock draws on directly.
```

---

# Part 1 — DSA Revision Block (1 hr)

## Revision — Redundant Connection (LeetCode 684, Medium)

**🔗 Originally taught in full depth:** Week 11, Day 74 — Union-Find, path compression, union by rank.

**Deliberately chosen over a Dijkstra problem this week** — the plan explicitly asks for whichever of Union-Find or Dijkstra is newer and less-rehearsed; both were taught in the same stretch of Weeks 11–12, but Union-Find hasn't resurfaced anywhere in this series since, where several later graph problems gave Dijkstra's core mechanism incidental extra reinforcement.

**Statement:** a graph started as a tree with `n` nodes and `n-1` edges, then had exactly one extra edge added, creating exactly one cycle. Given the edges in the order they were added, return the edge that can be removed to restore a tree — if multiple edges could work, return the one that occurs **last** in the input.

**Attempt cold before reading on.**

### Recap: The Approach

Process edges **in the given order**, maintaining a Union-Find structure. For each edge `(u, v)`: if `u` and `v` are already in the same set (`find(u) == find(v)`), this edge connects two nodes that were already reachable from each other through prior edges — it's the one closing the graph's one guaranteed cycle. Return it immediately. Otherwise, union their sets and continue.

```java
public int[] findRedundantConnection(int[][] edges) {
    int n = edges.length;
    int[] parent = new int[n + 1];
    int[] rank = new int[n + 1];
    for (int i = 1; i <= n; i++) parent[i] = i;

    for (int[] edge : edges) {
        int rootU = find(parent, edge[0]);
        int rootV = find(parent, edge[1]);
        if (rootU == rootV) {
            return edge;   // u and v already connected -> this edge closes the cycle
        }
        union(parent, rank, rootU, rootV);
    }
    return new int[0];   // unreachable given the problem's own guarantee
}

private int find(int[] parent, int x) {
    if (parent[x] != x) {
        parent[x] = find(parent, parent[x]);   // path compression
    }
    return parent[x];
}

private void union(int[] parent, int[] rank, int rootU, int rootV) {
    if (rank[rootU] < rank[rootV]) {
        parent[rootU] = rootV;
    } else if (rank[rootU] > rank[rootV]) {
        parent[rootV] = rootU;
    } else {
        parent[rootV] = rootU;
        rank[rootU]++;
    }
}
```

### Fresh Trace — A Different Example Than Day 74 Used

`edges = [[1,2], [1,3], [2,3]]` — three nodes, three edges (one more than the two a 3-node tree needs).

```
parent=[_,1,2,3]  rank=[_,0,0,0]

edge [1,2]: find(1)=1, find(2)=2 — different roots.
            union: ranks tied (0,0) → parent[2]=1, rank[1]→1
            parent=[_,1,1,3]  rank=[_,1,0,0]

edge [1,3]: find(1)=1, find(3)=3 — different roots.
            union: rank[1]=1 > rank[3]=0 → parent[3]=1
            parent=[_,1,1,1]  rank=[_,1,0,0]

edge [2,3]: find(2)=1 (via parent[2]=1), find(3)=1 (via parent[3]=1) — SAME ROOT
            → return [2,3]
```

`1`, `2`, and `3` were already fully connected by the first two edges (a spanning tree over all three nodes); the third edge, `[2,3]`, connects two nodes already in the same component — exactly the cycle-closing edge, and correctly the last one encountered, since edges are processed strictly in input order.

**Complexity:** with both path compression and union by rank applied together, each `find`/`union` call runs in amortized `O(α(n))` — the inverse Ackermann function, which grows so slowly it's effectively constant for any `n` that could realistically appear. Across all `n` edges, total time is `O(n · α(n))`, in practice treated as `O(n)`. Space is `O(n)` for the `parent` and `rank` arrays.

### Common Mistakes Checklist

- [ ] Omitting path compression — still correct, but each `find` degrades toward `O(log n)` or worse on adversarial input order, since chains aren't flattened as they're traversed.
- [ ] Omitting union by rank — also still correct alone, but without it, naive unioning can build long chains, losing the near-constant-time guarantee path compression alone would otherwise provide close to.
- [ ] Reaching for a DFS-based cycle detection instead, then being unable to cleanly justify *why* the specific edge found is the one that appears **last** — Union-Find's in-order processing makes "last" fall out directly (every edge before the one returned successfully united two distinct components; the first one that doesn't is, by the problem's own one-cycle guarantee, the unique answer), where a general cycle-detection DFS doesn't naturally single out "last in input order" without extra bookkeeping.

**If any of this needed re-deriving rather than confirming:** Week 11, Day 74 has the full original treatment, including how Union-Find's amortized bound is actually derived rather than just asserted.

---

# Part 2 — LLD #3: Parking Lot, Part 2 — Making `assignSpot` Thread-Safe

## Finding the Race, Precisely

Yesterday's flow, called from any number of threads: `Level.findAvailableSpot(vehicle)` **reads** every spot's occupancy to pick the best fit, returns that spot; the caller then separately calls `spot.assignVehicle(vehicle)` to **write** the claim. Between that read and that write, **nothing stops a second thread from doing the exact same read and getting the exact same answer.**

```
Thread A: findAvailableSpot(vehicle) → sees spot C1 as free → returns C1
Thread B: findAvailableSpot(vehicle) → sees spot C1 as STILL free (A hasn't written yet) → returns C1
Thread A: C1.assignVehicle(carA)     → C1.parkedVehicle = carA
Thread B: C1.assignVehicle(carB)     → C1.parkedVehicle = carB   (silently overwrites A's claim!)
```

Both threads receive a `Ticket` claiming spot `C1`. Only `carB` is actually recorded as parked there — `carA`'s ticket now points at a spot that, physically, isn't holding `carA` at all. This is a textbook **check-then-act** race: `findAvailableSpot` (check) and `assignVehicle` (act) are two separate, non-atomic steps, and the gap between them is exactly wide enough for a second thread to walk through.

## The Fix — Make Check-and-Claim One Atomic Operation, Per Spot

```java
public class ParkingSpot {
    private final String spotId;
    private final VehicleSize spotSize;
    private Vehicle parkedVehicle;

    public ParkingSpot(String spotId, VehicleSize spotSize) {
        this.spotId = spotId;
        this.spotSize = spotSize;
    }

    // ATOMIC: the check (canFit) and the claim (setting parkedVehicle) now happen
    // under the SAME lock (this ParkingSpot instance's own monitor) — no other
    // thread can observe or act on this spot's state in between the two.
    public synchronized boolean tryAssign(Vehicle vehicle) {
        if (isOccupied() || vehicle.getSize().ordinal() > spotSize.ordinal()) {
            return false;
        }
        this.parkedVehicle = vehicle;
        return true;
    }

    // ALSO synchronized on the same lock — every mutation of parkedVehicle must
    // go through this object's monitor, or the visibility guarantee below breaks.
    public synchronized void removeVehicle() {
        this.parkedVehicle = null;
    }

    // Only ever called from WITHIN tryAssign, which already holds the lock —
    // Java's synchronized is reentrant, so this doesn't need its own synchronized
    // keyword, but it must never be called from outside an already-locked context.
    private boolean isOccupied() {
        return parkedVehicle != null;
    }

    public String getSpotId() { return spotId; }
    public VehicleSize getSpotSize() { return spotSize; }
}
```

```java
public class Level {
    private final int levelNumber;
    private final List<ParkingSpot> spots;   // fixed at construction — never structurally
                                              // modified afterward, so safe to iterate
                                              // concurrently without its own lock

    public Level(int levelNumber, List<ParkingSpot> spots) {
        this.levelNumber = levelNumber;
        this.spots = spots;
    }

    // Scans tier by tier (smallest fit first), TRYING each candidate spot — the scan
    // itself needs no synchronization, because correctness rests entirely on tryAssign
    // being atomic, not on the scan seeing a perfectly consistent snapshot.
    public ParkingSpot tryParkVehicle(Vehicle vehicle) {
        VehicleSize[] sizesInOrder = VehicleSize.values();
        for (int tier = vehicle.getSize().ordinal(); tier < sizesInOrder.length; tier++) {
            for (ParkingSpot spot : spots) {
                if (spot.getSpotSize() == sizesInOrder[tier] && spot.tryAssign(vehicle)) {
                    return spot;   // won the race for this spot
                }
                // tryAssign returning false just means "already taken (by an earlier
                // occupant, or by another thread that just won it)" — try the next spot.
            }
        }
        return null;
    }

    public int getLevelNumber() { return levelNumber; }
}
```

```java
public class ParkingLot {
    private final List<Level> levels;   // also fixed at construction — same reasoning as Level.spots
    private final Map<String, Ticket> activeTickets = new ConcurrentHashMap<>();   // Week 6 Day 39
    private final AtomicInteger nextTicketId = new AtomicInteger(1);               // NEW — see below

    public ParkingLot(List<Level> levels) {
        this.levels = levels;
    }

    public Ticket parkVehicle(Vehicle vehicle) {
        for (Level level : levels) {
            ParkingSpot spot = level.tryParkVehicle(vehicle);
            if (spot != null) {
                String ticketId = "T-" + nextTicketId.getAndIncrement();
                Ticket ticket = new Ticket(ticketId, vehicle, spot, LocalDateTime.now());
                activeTickets.put(ticketId, ticket);
                return ticket;
            }
        }
        return null;
    }

    public boolean removeVehicle(String ticketId) {
        Ticket ticket = activeTickets.get(ticketId);
        if (ticket == null) {
            return false;
        }
        ticket.getSpot().removeVehicle();
        activeTickets.remove(ticketId);
        return true;
    }
}
```

**Three things changed from Day 110, each with its own specific reason, not applied uniformly "just to be safe":**

1. **`activeTickets` becomes `ConcurrentHashMap`.** A plain `HashMap` isn't safe for concurrent `put`/`get`/`remove` from multiple threads — its internal structure (bucket arrays, resize behavior) can corrupt under concurrent structural modification. `ConcurrentHashMap`, already fully taught (Week 6, Day 39), is the direct, idiomatic fix.

2. **`nextTicketId` becomes `AtomicInteger`.** A plain `int nextTicketId; nextTicketId++` is a read-modify-write sequence with **no** atomicity guarantee — two threads can both read the same value before either writes back the incremented result, producing **duplicate ticket IDs**, a correctness bug independent of the spot-assignment race entirely. `AtomicInteger.getAndIncrement()` performs that same read-modify-write as one indivisible hardware-level operation. **Worth being precise about what's actually new here:** the *class* is new today, but the *mechanism* underneath it — Compare-And-Swap — is not; it's the exact same primitive Day 39 already opened up inside `ConcurrentHashMap`'s own internals. This is new API surface on an already-understood foundation, not a new concept from zero.

3. **`removeVehicle()` becomes `synchronized`, matching `tryAssign`.** Both methods mutate the same `parkedVehicle` field, from potentially different threads; both must go through the *same* lock for Java's happens-before guarantee to hold for that field consistently. Synchronizing only one of the two would leave the other free to race against it.

**Why `parkedVehicle` does *not* also need `volatile`, precisely — this is a real question, not a rhetorical one:** Week 15, Day 100 established that `volatile` guarantees visibility (and ordering) for a single field's reads and writes across threads. Here, **every** read of `parkedVehicle` (inside `isOccupied()`, always called from within `tryAssign`) and **every** write to it (inside `tryAssign` and `removeVehicle`) happens while holding the *same* monitor — `synchronized` already establishes a happens-before edge for **everything touched inside the block**, not just one field, which is a strictly stronger guarantee than `volatile` alone provides for a single variable. Adding `volatile` on top would be redundant, not incorrect — worth stating precisely rather than reaching for both "to be extra safe," since knowing *which* guarantee is actually doing the work is the entire point of having learned both mechanisms separately.

> ⚠️ **Common Mistake:** synchronizing `tryAssign` but leaving `removeVehicle` unsynchronized, on the reasoning that "removing is simpler, it's just a null assignment." Simplicity of the write doesn't matter — what matters is whether *any* other thread's synchronized access to the same field can race against it, and here it can: a `tryAssign` call on one thread and a `removeVehicle` call on another, targeting the same spot, are exactly as capable of racing as two `tryAssign` calls are.

## Proving It — 10 Threads, 3 Spots, Zero Double-Bookings

```java
import java.util.concurrent.atomic.AtomicInteger;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class ParkingLotConcurrencyTest {

    @Test
    void tenThreadsCompetingForThreeSpotsNeverDoubleBook() throws InterruptedException {
        List<ParkingSpot> spots = new ArrayList<>();
        spots.add(new ParkingSpot("C1", VehicleSize.COMPACT));
        spots.add(new ParkingSpot("C2", VehicleSize.COMPACT));
        spots.add(new ParkingSpot("C3", VehicleSize.COMPACT));
        ParkingLot lot = new ParkingLot(List.of(new Level(1, spots)));

        int threadCount = 10;
        Ticket[] results = new Ticket[threadCount];
        Object startingGate = new Object();
        AtomicInteger threadsWaiting = new AtomicInteger(0);
        List<Thread> threads = new ArrayList<>();

        for (int i = 0; i < threadCount; i++) {
            final int id = i;
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    synchronized (startingGate) {
                        threadsWaiting.incrementAndGet();
                        try {
                            startingGate.wait();   // every thread parks here until released together
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                            return;
                        }
                    }
                    results[id] = lot.parkVehicle(new Vehicle("CAR-" + id, VehicleSize.COMPACT));
                }
            });
            threads.add(thread);
            thread.start();
        }

        // Wait until all 10 threads have genuinely reached the gate before releasing —
        // more precise than a fixed sleep, and safe: the main thread's own synchronized
        // block below can't acquire startingGate's monitor until the last waiting
        // thread has actually released it by calling wait(), so this can't fire early.
        while (threadsWaiting.get() < threadCount) {
            Thread.sleep(5);
        }
        synchronized (startingGate) {
            startingGate.notifyAll();   // release all 10 at once — maximizes real contention
        }

        for (Thread thread : threads) {
            thread.join();
        }

        Set<String> claimedSpotIds = new HashSet<>();
        int successCount = 0;
        for (Ticket ticket : results) {
            if (ticket != null) {
                successCount++;
                assertTrue(claimedSpotIds.add(ticket.getSpot().getSpotId()),
                        "Spot " + ticket.getSpot().getSpotId() + " was claimed by more than one ticket!");
            }
        }

        assertEquals(3, successCount);           // exactly 3 of 10 vehicles succeed — 3 spots exist
        assertEquals(3, claimedSpotIds.size());  // and all 3 are distinct — no double-booking
    }
}
```

**Why this test would actually have caught Day 110's race, not just pass by coincidence:** `startingGate.wait()`/`notifyAll()` deliberately releases all 10 threads at the *same instant* rather than letting them start (and likely finish) one at a time — without this, 10 threads created and started in a tight loop could easily finish sequentially fast enough that the race window never gets exercised, and the test would pass even against Day 110's genuinely broken code, proving nothing. Run this test against yesterday's un-synchronized `findAvailableSpot`/`assignVehicle` pair and it fails intermittently — not every run, since races are timing-dependent, which is exactly why the plan's definition of done asks for **5 consecutive passing runs**, not one.

**Complexity, for the fix itself, not the test:** `tryAssign` remains `O(1)` — a synchronized method still does the same constant amount of work, synchronization adds a (small, in the uncontended case) locking overhead, not a growth-rate change. `tryParkVehicle`'s scan is unchanged from Day 110's `O(s)` per level.

---

# Part 3 — Theory: Where to Put the Lock (1 hr)

Three granularities, genuinely compared — not "here's the right answer," but "here's the trade-off, and here's why one side of it was chosen today."

| Granularity | What contends | Correctness | Throughput |
|---|---|---|---|
| **Whole `ParkingLot`** (`synchronized` on `parkVehicle` itself) | *Every* concurrent parking attempt, anywhere in the entire garage | Trivially correct — full serialization | Worst — two vehicles parking on unrelated levels still wait on each other |
| **Per-`Level`** | Every attempt on the *same level* | Correct | Better — different levels proceed in parallel, but same-level attempts still contend even when headed for different spots |
| **Per-`ParkingSpot`** (today's actual choice) | Only threads racing for the *exact same spot* | Correct (proven above) | Best of the three — spots on the same level, even adjacent ones, proceed fully in parallel |

**Why per-spot was the right call here, not just the finest-grained option available for its own sake:** there's no shared mutable state *between* two different `ParkingSpot` instances that a broader lock would need to protect — each spot's occupancy is entirely its own concern, and nothing about assigning `C1` needs to read or write anything belonging to `C2`. A lock's job is to protect genuinely *shared* mutable state; where none exists between two entities, sharing a lock between them only adds contention with no matching correctness benefit. **This also means there's no deadlock risk in this design** — a thread never needs to hold more than one spot's lock at the same time (compare to a hypothetical "transfer a vehicle from spot A to spot B" operation, which *would* need to hold both locks simultaneously, and would need a consistent lock-ordering rule to avoid two threads deadlocking by acquiring A-then-B and B-then-A respectively — worth naming as the next real problem this exact granularity choice would eventually have to solve, even though it's out of scope for park/leave alone).

**`ConcurrentHashMap` and lock striping, as a generalization worth naming explicitly:** today's `activeTickets` reuses `ConcurrentHashMap` directly (Day 39) rather than needing its own custom locking scheme. Internally, `ConcurrentHashMap` achieves something structurally similar to today's per-spot locking, but for a **much larger, dynamically-changing** key set — it can't reasonably give every possible key its own dedicated lock object, so it stripes a bounded number of locks across buckets, accepting that two unrelated keys occasionally, harmlessly share a stripe. Today's per-`ParkingSpot` locking is really lock striping's limiting case, at the scale where "one stripe per key" is entirely affordable — a small, fixed number of spots, each cheaply able to own its own monitor. The *principle* underneath both is identical: narrow the lock's scope to only the state that's genuinely shared, and no further.

> 🔑 **Key Takeaway:** "where to put the lock" is really "how much genuinely-shared mutable state does this operation actually touch" — whole-lot locking pretends every spot is shared with every other spot (untrue); per-spot locking matches the actual boundary of what's shared (each spot's occupancy is shared only among threads targeting *that* spot). Correctness is achievable at every granularity in this table; only the *contention cost* changes, which is precisely why this is a genuine trade-off table and not a "pick the smallest lock always" rule.

---

# Project Block Guide (3.5 hrs)

**Repository:** `lld-java`, `parking-lot/` module (continuing directly from Day 110, not a new module).

**Task:** the thread-safety fixes above — `ParkingSpot.tryAssign`/`removeVehicle`, `Level.tryParkVehicle`, `ParkingLot` with `ConcurrentHashMap` and `AtomicInteger`.

**Definition of done:**
- The concurrency test above (or your own equivalent covering the same guarantee) passes **reliably across at least 5 consecutive runs** — not once. Run it in a loop; a single pass proves little given how timing-dependent races are.
- No double-booking, verified the way the test verifies it: every successful ticket's spot ID is genuinely unique across the batch.
- Pushed to `lld-java/parking-lot/`.

---

# Career Block Guide (1 hr)

**LinkedIn — Post 21.** A draft to adapt, not copy verbatim:

> Spent today making a Parking Lot's spot assignment thread-safe — and specifically proving it, not just believing it.
>
> Wrote a test that fires 10 threads at 3 remaining spots, all released at the exact same instant. Against my first version, it failed intermittently — a classic check-then-act race, two threads both reading "spot is free" before either one had written its claim. Fixed it by making the check-and-claim one atomic operation, locked at the level of a single spot rather than the whole garage, so unrelated spots never wait on each other.
>
> The real lesson wasn't the fix itself — it was that "looks correct" and "is correct under concurrency" are different claims, and only one of them is provable by actually reading the code.

**Networking reflection:** 15 minutes reviewing the past week's outreach — response rate so far, which personalized notes landed, which didn't. Adjust the next batch based on what's actually working, not what felt right in the moment.

**Format note, worth filing away:** Rippling's onsite LLD round and Uber's own follow-up rounds have been independently reported as structurally similar — both push candidates from a working single-threaded design into "now make it concurrent" mid-interview, the exact escalation today's work (and Day 112's Mock Interview #2) is built to rehearse specifically, not incidentally.

---

# Day 111 — Interview Questions

**Q1. Describe the exact race condition in the original `findAvailableSpot` + `assignVehicle` design.** They're two separate, non-atomic steps — a read (checking which spot is free) followed later by a write (claiming it). Two threads can both perform the read before either performs the write, both see the same spot as free, and both proceed to claim it — the second write silently overwrites the first, double-booking the spot.

**Q2. Why does making `tryAssign` synchronized, alone, fully fix the race — what makes "check and claim" atomic now?** The check (`isOccupied`/size comparison) and the claim (setting `parkedVehicle`) both happen inside the same `synchronized` method, holding the same monitor — no other thread can execute any of `ParkingSpot`'s synchronized methods on that same instance until the current call fully returns, so no thread can observe the "free" state and act on it while another thread is mid-claim.

**Q3. Why must `removeVehicle` also be synchronized, given it's "just a null assignment"?** Simplicity of the write is irrelevant — what matters is whether another thread's synchronized access to the same field can race against it, and it can: an unsynchronized `removeVehicle` racing against a synchronized `tryAssign` on the same spot has exactly the same kind of race as two unsynchronized methods would.

**Q4. Why doesn't `parkedVehicle` also need to be declared `volatile`?** Every read and write of it already happens inside a `synchronized` block on the same monitor, which establishes a happens-before edge for everything touched inside that block — a strictly stronger visibility guarantee than `volatile` provides for a single field alone. Adding `volatile` on top would be redundant, not incorrect.

**Q5. Why is per-`ParkingSpot` locking preferred over locking the entire `ParkingLot`, precisely — not just "it's faster"?** There's no shared mutable state *between* different spots that a broader lock would need to protect; each spot's occupancy is entirely its own concern. A lock should scope to genuinely shared state — locking the whole lot serializes operations that don't actually conflict with each other, adding contention with no matching correctness benefit.

**Q6. Why is there no deadlock risk in today's design?** A thread never needs to hold more than one spot's lock at the same time — deadlock requires at least two locks acquired in inconsistent orders by different threads, and today's operations (`tryAssign`, `removeVehicle`) each only ever touch one spot's monitor per call.

**Q7. Why does the concurrency test release all 10 threads via a shared `wait()`/`notifyAll()` gate instead of just starting them in a loop?** Threads started in a plain loop can easily run mostly sequentially — finishing one before the next really gets going — never actually exercising the race window at all; releasing all of them from a blocked `wait()` at the same instant forces genuine simultaneous contention, which is the only way the test could actually have caught Day 110's bug.

**Q8. What is `AtomicInteger` actually built on, and why does that matter for how "new" it really is today?** The same Compare-And-Swap primitive already taught underneath `ConcurrentHashMap` (Week 6, Day 39) — it's new API surface exposing an already-understood mechanism directly, not a new concept requiring its own from-scratch treatment.

**Q9. In Redundant Connection, why does returning the first edge Union-Find finds already connected correctly give the *last* qualifying edge overall?** The problem guarantees exactly one cycle exists. Every edge processed before the one that fails to unite two distinct components must have successfully merged two previously-separate groups; the first edge that instead connects two nodes already in the same group is, by the one-cycle guarantee, the unique edge closing that cycle — and since edges are processed strictly in input order, that's necessarily the last one that could have been removed to restore a tree.

**Q10. What's the amortized time complexity of Union-Find with both path compression and union by rank, and why is neither optimization alone sufficient to claim it?** `O(α(n))` per operation, where α is the inverse Ackermann function — effectively constant for realistic `n`. Path compression alone still allows long chains from suboptimal unioning; union by rank alone still leaves compression opportunities unexploited on each `find`. The near-constant bound specifically requires both together.

---

## Daily Deliverable Check

- [ ] Redundant Connection (LC 684) solved cold, without hints.
- [ ] `assignSpot`/`tryAssign` made thread-safe; concurrency test passes reliably across 5+ consecutive runs with no double-booking.
- [ ] Locking-granularity trade-off (whole-lot vs. per-level vs. per-spot) written down.
- [ ] LinkedIn Post 21 published.

---

## What Tomorrow Assumes You Already Know Cold

Day 112 assumes today's full concurrency reasoning — the race, the fix, the granularity trade-off — is solid enough to defend live, since Mock Interview #2 explicitly pushes "now make it thread-safe" mid-interview on a *different* system (Library Management), expecting the same reasoning applied fresh under time pressure, not recited from memory against the one system it was originally built for.
