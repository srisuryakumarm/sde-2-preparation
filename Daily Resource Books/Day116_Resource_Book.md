# Day 116 Resource Book — LLD #8: BookMyShow, Part 1 — Design, and Why This Is a Concurrency Problem in Disguise

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 115 Resource Book](Day115_Resource_Book.md)
**Next ▶:** [Day 117 Resource Book](Day117_Resource_Book.md)
**Companion to:** Day 116 of `Week_17_Revised.md`

---

## Recap

Day 115 built Splitwise on Strategy (applied for real, for the first time) and proved a genuine algorithm — dual-heap greedy settlement — was hiding underneath what looked like a plain expense-tracking system. Today opens BookMyShow, a two-day system split deliberately in half: **today is design only**, a single-threaded booking flow that will pass every single-threaded test written against it. **No new pattern gets introduced today.** The entire point of today is building something that looks complete and correct — and then, in the theory block, showing precisely why it isn't, without fixing it yet. The fix is tomorrow.

## Learning Objectives

By the end of today, without notes:

1. Apply the 5-step framework to BookMyShow's core objects without new pattern scaffolding, and explain why seat availability must be scoped **per-show**, not stored globally on `Seat`.
2. Implement the single-threaded booking flow (Available → Held → Booked) correctly for one user at a time.
3. Trace, precisely, the exact thread interleaving that breaks this design under two simultaneous bookings — not just assert that concurrency is a risk, but produce the actual sequence of reads and writes that causes two users to believe they both hold the same seat.
4. Name why this specific race is structurally identical to two races already proven this week (Parking Lot's `findAvailableSpot`, the ATM's `canDispense`/`commitDispense`).
5. Solve Clone Graph (LC 133) cold, narrating the visited-map mechanism from memory.

## Concept Dependency Map

```
Week 16 Day 106: 5-step LLD framework
Week 16 Day 111: Parking Lot's check-then-act race — findAvailableSpot() reads,
                 assignVehicle() writes, and nothing stops another thread
                 reading in between
Week 17 Day 113: The ATM's canDispense/commitDispense split — the identical
                 check-then-act shape, named explicitly as assuming
                 single-threaded execution
        │
        ▼
Today: BookMyShow, Part 1 — Design (NO new pattern)
  ├─ Movie / Theater / Screen / Show / Seat / Booking — plain object design
  ├─ Seat status scoped PER-SHOW (map on Show), not global on Seat —
  │  a real, common mistake avoided deliberately
  ├─ Single-threaded flow: hold → confirm, correct for one caller at a time
  └─ Theory: the EXACT same check-then-act race as Parking Lot and the ATM,
     traced concretely — set up, not fixed, today
        │
        ▼
DSA Revision: Clone Graph (LC 133)
  (needs: Graph DFS — Wk10-11; HashMap as a visited-set — Wk3)
        │
        ▼
Tomorrow: BookMyShow, Part 2 — the fix (pessimistic + optimistic locking)
```

---

# Part 1 — Core Design (5-Step Framework, No New Pattern)

## Requirements clarified first

Search shows by movie/theater/time (out of scope for the code today — assume a `Show` is already selected); select seats; hold them; confirm (pay); a held-but-unpaid seat should eventually expire, but timed expiry is out of scope today — the plan's two-day split reserves the *correctness-under-concurrency* question for tomorrow, and adding a timeout mechanism today would blur that boundary rather than sharpen it.

## Core objects and the one deliberate design decision that matters most

```java
public enum SeatStatus { AVAILABLE, HELD, BOOKED }

public class Seat {
    private final String seatId;
    private final String seatNumber;

    public Seat(String seatId, String seatNumber) {
        this.seatId = seatId;
        this.seatNumber = seatNumber;
    }
    public String getSeatId() { return seatId; }
}
```

**⚠️ Common Mistake, avoided deliberately: `Seat` has no `status` field.** The obvious first instinct is to put a `SeatStatus status` field directly on `Seat` — and it's wrong. The same physical seat belongs to one `Screen`, but that screen hosts many `Show`s across a day (a 3 PM showing and a 6 PM showing use the identical physical seats). If status lived on `Seat`, booking a seat for the 3 PM show would incorrectly mark it unavailable for the 6 PM show too. **Seat availability must be scoped per-`Show`**, not per-`Seat`.

```java
public class Screen {
    private final String screenId;
    private final List<Seat> seats;
    public Screen(String screenId, List<Seat> seats) { this.screenId = screenId; this.seats = seats; }
    public List<Seat> getSeats() { return seats; }
}

public class Movie {
    private final String movieId;
    private final String title;
    public Movie(String movieId, String title) { this.movieId = movieId; this.title = title; }
}

public class Theater {
    private final String theaterId;
    private final String name;
    private final List<Screen> screens;
    public Theater(String theaterId, String name, List<Screen> screens) {
        this.theaterId = theaterId; this.name = name; this.screens = screens;
    }
}

public class Show {
    private final String showId;
    private final Movie movie;
    private final Screen screen;
    private final LocalDateTime startTime;
    private final Map<String, SeatStatus> seatStatus;   // seatId -> status, scoped to THIS show only

    public Show(String showId, Movie movie, Screen screen, LocalDateTime startTime) {
        this.showId = showId;
        this.movie = movie;
        this.screen = screen;
        this.startTime = startTime;
        this.seatStatus = new HashMap<>();
        for (Seat seat : screen.getSeats()) {
            seatStatus.put(seat.getSeatId(), SeatStatus.AVAILABLE);
        }
    }

    public SeatStatus getSeatStatus(String seatId)              { return seatStatus.get(seatId); }
    public void setSeatStatus(String seatId, SeatStatus status) { seatStatus.put(seatId, status); }
}
```

```java
public enum BookingStatus { HELD, CONFIRMED, CANCELLED }

public class Booking {
    private final String bookingId;
    private final Show show;
    private final List<String> seatIds;
    private final String userId;
    private BookingStatus status;

    public Booking(String bookingId, Show show, List<String> seatIds, String userId, BookingStatus status) {
        this.bookingId = bookingId; this.show = show; this.seatIds = seatIds;
        this.userId = userId; this.status = status;
    }
    public Show getShow()            { return show; }
    public List<String> getSeatIds() { return seatIds; }
    public void setStatus(BookingStatus status) { this.status = status; }
}
```

## The single-threaded booking flow

```java
public class BookingService {
    private int bookingCounter = 0;

    public Booking holdSeats(Show show, List<String> seatIds, String userId) {
        for (String seatId : seatIds) {
            if (show.getSeatStatus(seatId) != SeatStatus.AVAILABLE) {
                throw new IllegalStateException("Seat " + seatId + " is not available.");
            }
        }
        for (String seatId : seatIds) {
            show.setSeatStatus(seatId, SeatStatus.HELD);
        }
        return new Booking("BKG-" + (++bookingCounter), show, seatIds, userId, BookingStatus.HELD);
    }

    public void confirmBooking(Booking booking) {
        for (String seatId : booking.getSeatIds()) {
            booking.getShow().setSeatStatus(seatId, SeatStatus.BOOKED);
        }
        booking.setStatus(BookingStatus.CONFIRMED);
    }

    public void cancelHold(Booking booking) {
        for (String seatId : booking.getSeatIds()) {
            booking.getShow().setSeatStatus(seatId, SeatStatus.AVAILABLE);
        }
        booking.setStatus(BookingStatus.CANCELLED);
    }
}
```

**🔑 Key Takeaway:** run any single-threaded test against this — hold two seats, confirm them, verify they're `BOOKED`; try to hold an already-`HELD` seat, verify it throws; cancel a hold, verify seats return to `AVAILABLE` — and every one passes. This code is genuinely, fully correct **for one caller at a time.** That qualifier is the entire subject of the theory section below.

---

# Part 2 — Theory: Why Ticket Booking Is a Concurrency Problem in Disguise

Nothing above is broken, in the sense that every line does exactly what it says. The bug isn't in any single method — it's in the **gap between two method calls that look atomic from the outside but aren't.**

## The exact race, traced

```
Initial state: seat5 → AVAILABLE

Thread A calls holdSeats(show, ["seat5"], "userA"):
  Line: show.getSeatStatus("seat5") != AVAILABLE?  → reads AVAILABLE → check passes
                                              ┃
Thread B calls holdSeats(show, ["seat5"], "userB"):
  Line: show.getSeatStatus("seat5") != AVAILABLE?  → ALSO reads AVAILABLE (A hasn't written yet!)
                                              ┃      → check ALSO passes
                                              ┃
Thread A: show.setSeatStatus("seat5", HELD)  ← writes HELD
Thread B: show.setSeatStatus("seat5", HELD)  ← writes HELD again (no error — just overwrites)

Result: BOTH threads return a successful Booking object for seat5.
Both users' apps show "seat held, proceed to payment."
Only one of them can actually sit in that seat.
```

This is not a hypothetical edge case — it's the **default outcome** the instant two requests for the same seat land close enough together, which for a popular show's release-time rush is not rare at all.

**💡 Interview Insight — say this unprompted, don't wait to be asked "what about concurrency?":** this is the identical **check-then-act** shape as two races already proven this week. Parking Lot's `findAvailableSpot()` reads spot availability, then a separate `assignVehicle()` writes to it — Day 111 fixed this with per-`ParkingSpot` locking. The ATM's `canDispense`/`commitDispense` split explicitly assumed nothing else mutates state between the read and the write — Day 113 flagged, but didn't need to fix, the exact same assumption. **Today's `holdSeats` has the identical gap: a loop of reads (the availability check), followed by a separate loop of writes (the status update), with no lock holding the gap between them shut.** Naming this connection precisely — not vaguely gesturing at "concurrency is hard" — is what a tier-1 interviewer is listening for.

**🔗 Forward reference:** Day 117 fixes exactly this, two different ways — a pessimistic per-seat lock (extending Day 111's technique directly) and an optimistic version-based retry (extending the CAS/`AtomicInteger` idea from Day 111/Week 6 to a database row). Both are deferred deliberately; today's job was proving the bug exists precisely, not patching it.

**⚠️ Common Mistake to avoid saying out loud in an interview:** "I'll just add `synchronized` to `holdSeats`." That's not wrong as a starting instinct, but stated with no further thought it signals the check-then-act shape wasn't actually understood — a single global lock around the whole method would work but serializes *every* booking across the *entire show*, not just the seats actually in conflict, which is a real throughput problem at scale. The precise answer names *what* needs to be locked (the specific seats involved) and *why a global lock is safe but wasteful* — exactly the granularity conversation Day 111 already had once, about parking spots instead of seats.

---

# DSA Revision Block (1 hr)

## Clone Graph (LeetCode 133, Medium) — Pattern: Graph DFS + Visited Map for Cycle Safety

**Originally taught:** Week 10–11 (Graphs BFS/DFS pattern).

**Statement:** given a reference to a node in a connected undirected graph, return a deep copy (clone) of the graph.

```java
public Node cloneGraph(Node node) {
    if (node == null) return null;
    Map<Node, Node> visited = new HashMap<>();
    return cloneHelper(node, visited);
}

private Node cloneHelper(Node node, Map<Node, Node> visited) {
    if (visited.containsKey(node)) {
        return visited.get(node);
    }
    Node clone = new Node(node.val);
    visited.put(node, clone);
    for (Node neighbor : node.neighbors) {
        clone.neighbors.add(cloneHelper(neighbor, visited));
    }
    return clone;
}
```

**The mechanism, recapped:** `visited` maps an *original* node to its *already-created clone* — checked **before** recursing into any neighbor. This is what makes the graph's cycles safe: a cyclic graph (A → B → A) would recurse infinitely without it, since B's neighbor list points straight back to A. The first time A is seen, its clone is created and registered in `visited` **before** iterating its neighbors — so when B's neighbor loop reaches back to A, `visited.containsKey(A)` is already true, and the existing clone is returned instead of recursing again.

**⚠️ Common Mistake:** registering the clone in `visited` *after* the neighbor loop instead of before it — this looks like a minor reordering but reintroduces the infinite recursion the map exists to prevent, since a cycle would revisit the original node before its clone was ever recorded.

**Complexity: Time O(V + E)** — every node visited exactly once (guarded by `visited`), every edge traversed exactly once. **Space O(V)** for the `visited` map plus the recursion stack, which in the worst case (a graph shaped like a long path) is also O(V).

**Edge cases:** a single node with no neighbors (returns immediately, one clone, empty neighbor list); the input node itself being `null` (handled explicitly, first line); a graph containing a self-loop (a node listing itself as a neighbor) — handled correctly by the same `visited` check, no special-casing needed.

**💡 Interview Insight:** this is the same "have I seen this before" cycle-guard shape as Course Schedule's visited/in-progress tracking (Week 16's revision pick) — different problem, same underlying discipline: never let a traversal revisit a node without a map (or set) remembering it already has.

---

# Project Block Guide

**Repository:** `lld-java/bookmyshow/`. Build the classes above. **Definition of done:** every single-threaded test passes (hold → confirm → BOOKED; hold an unavailable seat → throws; cancel → back to AVAILABLE); a demonstration test (or a written trace, if a live multi-threaded test isn't yet built) reproduces the race above concretely — two calls to `holdSeats` for the same seat, both succeeding, is the required proof this design needs tomorrow's fix. Pushed.

# Career Block Guide

**LinkedIn (20 min):** engagement only — no new post required today.
**Networking:** continue outreach ahead of Day 118's application push.

---

# Day 116 — Interview Questions

**Q1. Why can't `SeatStatus` live directly on `Seat`?** The same physical seat is reused across every show a screen hosts in a day — booking it for one showtime must not affect its availability for a different showtime on the same screen. Status has to be scoped per-`Show`, tracked in a map keyed by seat ID, not stored as a field on the seat itself.

**Q2. Does the single-threaded `BookingService` above have any logic bugs?** No — every method is correct for one caller at a time, and passes every single-threaded test that could be written against it. The problem isn't a logic bug; it's a gap between reads and writes that isn't visible from any single-threaded test at all.

**Q3. Trace the exact interleaving that lets two users successfully hold the same seat.** Both threads' availability checks (`getSeatStatus != AVAILABLE`) can read `AVAILABLE` before either thread's write (`setSeatStatus(HELD)`) has happened — so both checks pass, and both writes then succeed, each overwriting the other with no error raised.

**Q4. Name the two other races this week that share this exact shape.** Parking Lot's `findAvailableSpot()`/`assignVehicle()` split (Day 111) and the ATM's `canDispense`/`commitDispense` split (Day 113) — both read state, then separately write to it, with nothing holding the gap between the two operations shut.

**Q5. Why is "just add `synchronized` to the whole method" an incomplete answer, even though it would technically work?** It would serialize every booking attempt across the *entire show*, not just the specific seats actually in contention — a real throughput cost at scale. The stronger answer names that only the seats actually involved need locking, echoing the lock-granularity conversation Day 111 already had about parking spots.

**Q6. In Clone Graph, why must the clone be registered in `visited` before recursing into its neighbors, not after?** A cyclic graph can lead back to the original node before the neighbor loop finishes. If the clone weren't registered yet, that cycle would trigger infinite recursion instead of finding the already-created clone and stopping.

**Q7. What's the time and space complexity of Clone Graph, and why?** O(V + E) time — every node and every edge is visited exactly once, guarded by the visited map. O(V) space for the map, plus O(V) worst-case recursion depth on a graph shaped like a long path.

---

# Daily Deliverable Check

- [ ] BookMyShow single-threaded design complete — `Seat`/`Show`/`Screen`/`Theater`/`Movie`/`Booking`/`BookingService`, pushed to `lld-java/bookmyshow/`.
- [ ] Seat status confirmed scoped per-show (a map on `Show`), not global on `Seat`.
- [ ] Every single-threaded flow test passes: hold → confirm → BOOKED; double-hold on an unavailable seat throws; cancel returns seats to AVAILABLE.
- [ ] The race condition reproduced concretely — two successful holds on the same seat — and the exact interleaving that causes it explainable from memory, unprompted.
- [ ] Clone Graph (LC 133) solved cold; the before-vs-after `visited` registration bug explainable without notes.

---

## What Tomorrow Assumes You Already Know Cold

Day 117 fixes today's race two ways — pessimistic locking (a direct extension of Day 111's per-object locking, plus a genuinely new database-level variant, `SELECT FOR UPDATE`) and optimistic locking (a genuinely new named pattern, though it's CAS — Day 111's `AtomicInteger`, Week 6 Day 39's `ConcurrentHashMap` — applied to a database row's version column instead of an in-memory variable). Today's precise trace of the race is the exact thing tomorrow's fix needs to demonstrably close — if that trace isn't solid, tomorrow's "prove the fix works" step won't have anything to prove against.

**Next:** [Day 117 Resource Book](Day117_Resource_Book.md) — BookMyShow, Part 2: Concurrency, and Mock Interview #4.
