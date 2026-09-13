# Day 119 Resource Book — LLD #10: Hotel Booking System, Mock Interview #5, and LLD Complete

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 118 Resource Book](Day118_Resource_Book.md)
**Next ▶:** Day 120 (Week 18) — *HLD #1: URL Shortener, Applying the 5-Step Framework Explicitly* — generated when Week 18 begins.
**Companion to:** Day 119 of `Week_17_Revised.md`

---

## Recap

Day 118 reused Strategy a second time and drew the precise line between its constructor-config heuristic and its actual definition, then previewed — deliberately shallow — the System Design framework Week 18 spends real time on. Today closes the LLD phase: the tenth and final system, a fifth and final mock interview, and a full consolidation of everything Weeks 16–17 built together. No new pattern, no DSA revision block — today opens with a self-check instead.

## Learning Objectives

By the end of today, without notes:

1. List all ten LLD systems built across Weeks 16–17, each with its primary pattern (or the deliberate absence of one), cold.
2. Design Hotel Booking's search → reserve → confirm lifecycle, correctly modeling room availability as a **date-range** property, not a boolean — and recognize the overlap check as the exact same interval-intersection logic already taught (Week 4).
3. Write a concrete, well-reasoned comparison of Hotel Booking's eventual-consistency, multi-channel model against BookMyShow's strict real-time locking — citing *why* the two systems land on genuinely different points on the same trade-off, not just that they differ.
4. Walk out of Mock Interview #5 having chosen a subject deliberately, for a legible reason, not by default.

## Concept Dependency Map

```
Week 16 Day 106-112: 5-step framework; 10 GoF patterns; 4 LLD systems
Week 17 Day 113-118: 6 more LLD systems; Chain of Responsibility (11th pattern);
                     Strategy applied twice for real; a new heap role;
                     pessimistic + optimistic locking; System Design previewed
Week 4:              Interval overlap / merge-intervals logic
Week 9  Day 61:      Replication models — synchronous vs. asynchronous,
                     the durability/consistency-vs-latency/availability trade-off
Week 17 Day 116-117: BookMyShow's strict real-time concurrency model, in full
        │
        ▼
Today: Hotel Booking System (NO new pattern — pure application, one more time)
  ├─ Room availability scoped by DATE RANGE (same lesson as Day 116's
  │  per-show seat scoping, applied to a different axis — time instead of show)
  ├─ Overlap check — the SAME interval-intersection test as Week 4,
  │  cited directly, not re-derived
  └─ Concurrency model comparison vs. BookMyShow — grounded in Week 9's
     sync-vs-async replication trade-off, not invented fresh
        │
        ▼
Mock Interview #5 — candidate's choice, 45 min
        │
        ▼
Week 17 Consolidation — full week, closing the LLD phase
```

---

# Self-Check (15 min) — All Ten LLD Systems, Cold

**Do this before reading the table below.** Write out, from memory, all ten systems in order and each one's primary pattern. Check afterward, not before.

| # | Week | Day | System | Primary Pattern |
|---|------|-----|--------|------------------|
| 1 | 16 | 108 | Tic-Tac-Toe | None — first live run of the 5-step framework itself |
| 2 | 16 | 109 | Vending Machine | State |
| 3 | 16 | 110–111 | Parking Lot | None (Singleton considered, declined) — concurrency via per-spot locking |
| 4 | 16 | 112 | Library Management System | None — data-driven transition table, contrasted against genuine State |
| 5 | 17 | 113 | ATM Machine | State (reapplied) + Chain of Responsibility (new) |
| 6 | 17 | 114 | Elevator System | State (reapplied) + SCAN/LOOK dispatch (new, not a GoF pattern) |
| 7 | 17 | 115 | Splitwise | Strategy (first full application) |
| 8 | 17 | 116–117 | BookMyShow | None new — pessimistic + optimistic locking |
| 9 | 17 | 118 | Food Delivery | Strategy (second full application) |
| 10 | 17 | 119 | Hotel Booking | None — pure framework application |

**🔑 Key Takeaway, worth noticing directly:** only **four** of the ten systems chose a GoF pattern as their headline decision (Vending Machine, ATM, Elevator, Splitwise/Food Delivery for Strategy) — the other six deliberately didn't, either because no pattern actually fit (Tic-Tac-Toe, Parking Lot, Library Management, Hotel Booking) or because the interesting problem was somewhere else entirely (BookMyShow's concurrency). **Recognizing when *not* to reach for a pattern has been at least as much the point of this phase as recognizing when to.**

---

# Part 1 — Hotel Booking System

**Prerequisites confirmed:** interval overlap logic (Week 4); the "scope availability correctly, don't make it a global boolean" lesson (Day 116, BookMyShow's seats).

## Core design

```java
public enum RoomType { STANDARD, DELUXE, SUITE }

public class Room {
    private final String roomId;
    private final RoomType type;
    private final double pricePerNight;
    public Room(String roomId, RoomType type, double pricePerNight) {
        this.roomId = roomId; this.type = type; this.pricePerNight = pricePerNight;
    }
    public RoomType getType() { return type; }
    public String getRoomId() { return roomId; }
}

public class Hotel {
    private final String hotelId;
    private final List<Room> rooms;
    public Hotel(String hotelId, List<Room> rooms) { this.hotelId = hotelId; this.rooms = rooms; }
    public List<Room> getRooms() { return rooms; }
}

public enum BookingStatus { RESERVED, CONFIRMED, CANCELLED }

public class RoomBooking {
    private final String bookingId;
    private final Room room;
    private final LocalDate checkIn;
    private final LocalDate checkOut;
    private BookingStatus status;

    public RoomBooking(String bookingId, Room room, LocalDate checkIn, LocalDate checkOut, BookingStatus status) {
        this.bookingId = bookingId; this.room = room; this.checkIn = checkIn;
        this.checkOut = checkOut; this.status = status;
    }
    public LocalDate getCheckIn()  { return checkIn; }
    public LocalDate getCheckOut() { return checkOut; }
    public BookingStatus getStatus() { return status; }
    public void setStatus(BookingStatus status) { this.status = status; }
}
```

**⚠️ Common Mistake, avoided deliberately — same lesson as Day 116, different axis.** A `boolean isAvailable` field on `Room` is exactly the same category of bug as putting `SeatStatus` directly on `Seat` (Day 116): a room is available for *some date ranges* and not others, simultaneously — a room booked next week is still available today. Availability has to be checked **per date range**, not stored as a single global flag.

```java
public class HotelInventory {
    private final Map<String, List<RoomBooking>> bookingsByRoom = new HashMap<>();

    public boolean isAvailable(Room room, LocalDate checkIn, LocalDate checkOut) {
        List<RoomBooking> existing = bookingsByRoom.getOrDefault(room.getRoomId(), new ArrayList<RoomBooking>());
        for (RoomBooking booking : existing) {
            if (booking.getStatus() == BookingStatus.CANCELLED) {
                continue;
            }
            if (overlaps(checkIn, checkOut, booking.getCheckIn(), booking.getCheckOut())) {
                return false;
            }
        }
        return true;
    }

    public void addBooking(RoomBooking booking) {
        String roomId = booking.getRoom().getRoomId();
        if (!bookingsByRoom.containsKey(roomId)) {
            bookingsByRoom.put(roomId, new ArrayList<RoomBooking>());
        }
        bookingsByRoom.get(roomId).add(booking);
    }

    private boolean overlaps(LocalDate startA, LocalDate endA, LocalDate startB, LocalDate endB) {
        return startA.isBefore(endB) && startB.isBefore(endA);
    }
}
```

**🔑 Key Takeaway — `overlaps()` is not new logic.** `startA < endB && startB < endA` is the exact interval-intersection test already taught for merge-intervals-style problems (Week 4) — cited here directly rather than re-derived, applied to a new domain (date ranges instead of a generic `[start, end)` array).

**Getting the boundary right — worth being precise about, since it's easy to get backwards:** `checkOut` is treated as **exclusive**. A guest checking out on the 10th is gone by check-in time that day, so a new guest checking in on the 10th does *not* conflict with them — `[checkIn, checkOut)` as a half-open interval matches this real-world convention exactly. Using an inclusive `checkOut` here would incorrectly block same-day turnover, which is standard hotel practice, not an edge case to special-case around.

```java
public class RoomSearchService {
    private final Hotel hotel;
    private final HotelInventory inventory;

    public RoomSearchService(Hotel hotel, HotelInventory inventory) {
        this.hotel = hotel; this.inventory = inventory;
    }

    public List<Room> search(RoomType type, LocalDate checkIn, LocalDate checkOut) {
        List<Room> results = new ArrayList<>();
        for (Room room : hotel.getRooms()) {
            if (room.getType() == type && inventory.isAvailable(room, checkIn, checkOut)) {
                results.add(room);
            }
        }
        return results;
    }
}

public class HotelBookingService {
    private int counter = 0;
    private final HotelInventory inventory;

    public HotelBookingService(HotelInventory inventory) { this.inventory = inventory; }

    public RoomBooking reserve(Room room, LocalDate checkIn, LocalDate checkOut) {
        if (!inventory.isAvailable(room, checkIn, checkOut)) {
            throw new IllegalStateException("Room not available for these dates.");
        }
        RoomBooking booking = new RoomBooking("BKG-" + (++counter), room, checkIn, checkOut, BookingStatus.RESERVED);
        inventory.addBooking(booking);
        return booking;
    }

    public void confirm(RoomBooking booking) { booking.setStatus(BookingStatus.CONFIRMED); }
    public void cancel(RoomBooking booking)  { booking.setStatus(BookingStatus.CANCELLED); }
}
```

---

# Part 2 — Concurrency Model, Compared Against BookMyShow

**Prerequisites confirmed:** BookMyShow's strict real-time locking (Days 116–117); synchronous vs. asynchronous replication trade-offs (Week 9, Day 61).

**⚠️ Common Mistake:** treating "compare the concurrency models" as an invitation to bolt the exact same pessimistic/optimistic machinery from Day 117 onto Hotel Booking. That's not what the comparison is actually testing — the interesting answer is that **these two systems land on genuinely different points on the same trade-off, for concrete, defensible reasons**, not that one system "does concurrency" and the other "forgot to."

**BookMyShow's case for strict, real-time locking:** demand for a specific show is concentrated — hundreds of users can be hitting the exact same small seat inventory within the same few-second window (an opening-night release), and every one of them expects an **immediate** yes-or-no answer the instant they pick a seat. That combination — high real-time contention, low latency tolerance, a single system that fully owns the inventory — is exactly the regime where Day 117's synchronous locking (pessimistic or optimistic-with-immediate-retry) earns its cost.

**Hotel Booking's case for eventual consistency across channels:** a hotel's room inventory is typically sold through **many independent channels at once** — the hotel's own site, several third-party travel platforms, walk-ins — none of which the hotel operates or can hold a shared real-time lock across. Each channel commonly works from a locally cached view of inventory that syncs with the hotel's central system **periodically**, not instantly. This is a direct application of Week 9's asynchronous-replication trade-off: accepting a small window of staleness (and therefore a small, non-zero risk of overselling a room type) in exchange for not needing every independent, third-party-operated channel to coordinate through one real-time locking service at all — which, for channels the hotel doesn't control, may not even be technically achievable, not merely inconvenient.

**Why the risk is tolerable here specifically, and wouldn't be for BookMyShow:** an oversold room type has a real operational escape valve the hospitality industry actually uses — upgrade the affected guest to a better room, or in rare cases relocate them to a partner property with compensation. An **oversold seat** has no equivalent graceful fallback in the same few-second window a movie is about to start. The tolerance for staleness isn't a technology limitation being excused — it's a direct consequence of the business having a cheap way to absorb the rare failure case, which BookMyShow's business does not have.

**🔑 Key Takeaway, the actual point of the comparison:** "which system needs strict locking" isn't a question with one right answer in general — it depends on **contention concentration, latency expectations, how many independently-operated parties need to coordinate, and how expensive the failure mode actually is to absorb operationally.** BookMyShow and Hotel Booking score differently on every one of those axes, which is why they land on genuinely different, both-correct points on the same underlying trade-off.

---

# Mock Interview #5 — Logistics

**Format:** 45-minute LLD mock, candidate's choice of subject from the last two weeks. **Choosing deliberately, not by default:** the strongest choices are systems that let a full story get told in 45 minutes — the 5-step framework, a real pattern decision (including *why not* a plausible-looking alternative), and ideally a trade-off discussion. **BookMyShow** (pattern absence + a genuine concurrency trade-off, Days 116–117) and **Splitwise** (Strategy applied for real + a provable algorithm with an honestly-stated limit, Day 115) are both strong choices for exactly that reason — either lets the full five-step-framework-to-trade-off arc run start to finish. Whichever is chosen, be ready to say **why** that one, not just which one — the same "state the real reason, not just the answer" discipline this whole week has built.

---

# Project Block Guide

**Repository:** `lld-java/hotel-booking/`. Build the classes above. **Definition of done:** a room booked for one date range remains correctly searchable and bookable for a non-overlapping range on the same room (verify same-day checkout/check-in turnover works — book a room through the 10th, then successfully book the same room starting the 10th); the written concurrency-model comparison (Part 2, or your own version) committed alongside the code. Pushed.

# Career Block Guide

**Weekly Industry Awareness Ritual (30 min):** review what moved in the industry this week — any relevant product launches, layoffs, funding news, or engineering blog posts from the seven companies applied to Day 118 — arriving prepared to any conversation those applications generate is worth more than the 30 minutes it costs.

**Weekly Scorecard:** the LLD phase closes today — five mocks deep (#1 Tic-Tac-Toe, #2 Parking Lot, #3 Machine Coding/rate limiter, #4 BookMyShow concurrency, #5 today's choice), ten systems built, eleven design patterns earned (ten GoF plus the framework itself), zero new DSA problems added (all six of this week's DSA blocks were spaced-repetition revision — a deliberate choice, not an oversight; see the Week 17 Consolidation below for why). Take stock honestly against the diagnostic list before treating this phase as closed.

---

# Day 119 — Interview Questions

**Q1. Of the ten LLD systems built across Weeks 16–17, how many chose a GoF pattern as their headline decision, and what does that split signal?** Four (Vending Machine, ATM, Elevator, and Strategy across Splitwise/Food Delivery) — the other six deliberately didn't, either because no pattern genuinely fit or because the real problem was elsewhere (BookMyShow's concurrency). Recognizing when *not* to reach for a pattern was as much the point of this phase as recognizing when to.

**Q2. Why can't room availability be a boolean field on `Room`?** A room is available for some date ranges and unavailable for others simultaneously — a room booked next week is still available today. Availability must be checked against existing bookings for a specific date range, not stored as one global flag — the same underlying mistake as putting seat status directly on `Seat` (Day 116), on a different axis.

**Q3. Why is `checkOut` treated as exclusive in the overlap check?** A guest checking out on a given day is gone by check-in time, so a new guest checking in that same day doesn't conflict with them — standard same-day hotel turnover. An inclusive `checkOut` would incorrectly block that, treating standard practice as a conflict.

**Q4. Is the `overlaps()` check genuinely new logic?** No — it's the identical interval-intersection test from Week 4's merge-intervals-style problems, applied to date ranges instead of a generic array of intervals.

**Q5. Why does BookMyShow need strict, real-time locking while Hotel Booking can tolerate eventual consistency?** BookMyShow has high contention concentrated on one show's small seat inventory, in a tight time window, with users expecting immediate confirmation, and no cheap way to fix an oversold seat before showtime. Hotel Booking sells the same rooms across many independently-operated channels that can't easily share a real-time lock, on a much longer booking horizon, with a real operational fallback (room upgrade, guest relocation) for the rare oversold case.

**Q6. Is "which system needs strict locking" a question with one universally correct answer?** No — it depends on how concentrated contention is, how much latency users will tolerate, how many independent parties need to coordinate, and how expensive the failure mode is to absorb operationally. BookMyShow and Hotel Booking land on different, both-defensible points on that same trade-off.

**Q7. Why did this week's six DSA blocks add zero new problems to the curriculum?** They were spaced-repetition revision of patterns already fully mastered with extensive original and extra practice weeks earlier — a single cold retrieval check serves that purpose; padding already-reflexive patterns with more reps would spend the plan's 1-hour daily time-box on repetition that wasn't needed, not genuine new learning.

---

# Daily Deliverable Check

- [ ] All ten LLD systems and their primary patterns listed correctly from memory, before checking against the table above.
- [ ] Hotel Booking LLD complete — date-range availability, the shared interval-overlap logic, search → reserve → confirm lifecycle, pushed to `lld-java/hotel-booking/`.
- [ ] Same-day checkout/check-in turnover verified working (not incorrectly blocked).
- [ ] Concurrency-model comparison against BookMyShow written, grounded in concrete factors (contention concentration, latency tolerance, number of independent parties, cost of the failure mode) — not a vague "one is sync, one is async" restatement.
- [ ] Mock Interview #5 completed, subject chosen deliberately with a stated reason.
- [ ] Weekly Industry Awareness Ritual and Weekly Scorecard done.

---

# Week 17 Consolidation

## What actually got built

Six LLD systems (ATM, Elevator, Splitwise, BookMyShow across two days, Food Delivery, Hotel Booking), bringing the phase total to all ten planned systems, closed. One genuinely new GoF pattern (Chain of Responsibility — the eleventh this series has taught). Strategy moved from "taught conceptually" (Week 16) to "applied twice at full system scale" (Splitwise, Food Delivery), with the second application deliberately chosen to show the pattern generalizing beyond its first, structurally different-looking use. One new non-GoF algorithm (SCAN/LOOK dispatch, with the SCAN-vs-LOOK distinction drawn precisely rather than used loosely). One new heap role (dual-heap greedy settlement — the seventh distinct heap role this series has built), proven to guarantee ≤(n−1) transactions with an honest, explicit caveat about where that stops being the true theoretical minimum (LC 465, NP-hard, out of scope). A full concurrency arc — a race precisely traced (Day 116) before being fixed two ways (Day 117): pessimistic (a direct in-memory extension of Day 111, plus a genuinely new database-level `SELECT FOR UPDATE` variant, motivated specifically by multi-process deployment) and optimistic (a new name for an old CAS mechanism, applied to a version column) — both proven deterministic via a `CountDownLatch`-gated 10-thread test, and compared with a concrete, traffic-pattern-grounded trade-off rather than an abstract one. The System Design framework previewed, deliberately shallow, ahead of Week 18's real treatment. Three mock interviews (#3 Machine Coding format via a from-scratch Token Bucket rebuild; #4 BookMyShow with concurrency pushed on; #5 candidate's choice), bringing the LLD phase's mock total to five. One LinkedIn post (#22, Splitwise's settlement algorithm). Seven job applications submitted.

## Planned vs. actual

Every plan-required deliverable across all seven days was completed: 6/6 LLD systems, 3/3 mocks, 6/6 DSA revision blocks (Heap: LC 973; Trie: LC 211; Backtracking: LC 22; Graph: LC 133; DP: LC 300; String DP: LC 72 — each logged below and confirmed against the curriculum map's constraint that this week's Backtracking, Graph, and DP picks differ from Week 16's, which they do), 1/1 LinkedIn post, 7/7 applications. **Zero new DSA problems were added this week, required or extra** — a deliberate choice, not a shortfall: every DSA block this week was explicitly revision of patterns closed with extensive original and extra practice weeks earlier (Weeks 8–13), and this week introduced no new DSA pattern that would need repeated exposure to become reflexive. Padding an already-mastered pattern with unrequested extra problems would have spent the daily time-box on repetition the plan didn't ask for, at the cost of the LLD depth it did.

## Diagnostic — verify each of these cold before treating the phase as closed

- All ten LLD systems and their patterns (or deliberate absence of one), listed without notes.
- Chain of Responsibility's definition, and the ATM's exchange-argument proof for why largest-first dispensing is safe — plus the exact counterexample showing where that same greedy idea fails (tying back to Coin Change).
- The precise SCAN-vs-LOOK distinction, not used interchangeably.
- Splitwise's settlement algorithm: both halves — the proven (n−1) guarantee *and* its honest limit against the true NP-hard-optimal case.
- The exact traced race from Day 116, and both of Day 117's fixes, including *why* a database-level lock is needed beyond an in-memory one specifically.
- A concrete, non-abstract justification for pessimistic vs. optimistic locking, and separately for BookMyShow vs. Hotel Booking's concurrency models — four different trade-off arguments, not one memorized template reused four times.
- Why Food Delivery's stateless Strategy implementations are still genuinely Strategy — the who-chooses test, not the constructor-config heuristic.

## What Week 18 assumes

Week 18 opens System Design in earnest, applying the five-step framework previewed today for real, starting with a worked URL Shortener example (Day 120). It assumes today's preview is genuinely in hand as a shape to fill in, not something being seen for the first time. It also assumes Token Bucket (Week 10, recapped Day 114) solidly enough to serve as the concrete anchor for Day 121's fuller rate-limiting landscape (Leaking Bucket, Fixed Window, Sliding Window Log) — material this week deliberately did not teach ahead of, on purpose. And it assumes this week's concurrency and trade-off reasoning generally: Week 18's "Detailed Design" steps will very likely reuse exactly this kind of check-then-act vigilance and concrete, factor-grounded trade-off argument, just applied at a larger, distributed-systems scale rather than a single class's.

---

**Next:** Day 120 (Week 18) — *HLD #1: URL Shortener, Applying the 5-Step Framework Explicitly.*
