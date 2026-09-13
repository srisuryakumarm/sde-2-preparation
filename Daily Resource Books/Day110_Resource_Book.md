# Day 110 — LLD #3: Parking Lot, Part 1 — Design and Core Logic

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 109 Resource Book](Day109_Resource_Book.md)
**Next ▶:** [Day 111 Resource Book](Day111_Resource_Book.md)
**Companion to:** Day 110 of `Week_16_Revised.md`

---

## Recap

Tic-Tac-Toe (Day 108) needed no pattern. Vending Machine (Day 109) needed State, genuinely. Today's Parking Lot needs neither — a third data point in the same running theme, and the last new system before Parking Lot itself becomes the two-day centerpiece of this week (single-threaded core today, concurrency tomorrow). Today also opens the theory this system exists partly to motivate: **Composition over Inheritance**, made concrete through an actual refactor rather than taught in the abstract.

---

## Learning Objectives

By the end of today, without notes:

1. Design and implement a single-threaded Parking Lot with correct, size-aware spot assignment across multiple levels.
2. Explain, with a worked example, why size-fit must be directional (a smaller vehicle may use a larger spot; the reverse is never true) and why freeing a small spot never helps a larger vehicle that's waiting.
3. Given a spot-hierarchy design built with inheritance, identify concretely what breaks when a new size tier is added, and refactor it to composition.
4. Solve Validate Binary Search Tree (LC 98) cold, without hints, and explain why comparing only to a node's immediate parent is insufficient.

---

## Concept Dependency Map

```
Day 106-109: 5-step framework, applied live twice, both times correctly
             recognizing when a pattern from Days 106-107 doesn't apply
Week 15 Day 101: Open/Closed Principle (Factory Method vs. Simple Factory)
Week 8 Day 50: Tree DFS with propagated bounds (today's revision problem)
        │
        ▼
Today: framework applied a third time — Parking Lot, Part 1
  Vehicle, ParkingSpot both sized via a shared, ordered enum (composition)
  — NOT via a subclass per size (the flawed alternative, built and
  critiqued directly in Part 3)
        │
        ▼
Composition Over Inheritance (NEW, formalized — the instinct has been
building since Week 1 Day 5's "favor composition" aside, now given its
own full argument, grounded in Open/Closed)
        │
        ▼
🔗 Forward: Day 111 takes this exact single-threaded ParkingLot and makes
   assignSpot thread-safe — today's correctness is what tomorrow's
   concurrency work has to preserve under simultaneous access.
```

---

# Part 1 — DSA Revision Block (1 hr)

## Revision — Validate Binary Search Tree (LeetCode 98, Medium)

**🔗 Originally taught in full depth:** Week 8, Day 50 — Tree DFS with bounds propagated through the recursion, not just compared against an immediate parent.

**Statement:** given a binary tree, determine whether it is a valid binary search tree — every node's value strictly greater than *every* value in its left subtree and strictly less than *every* value in its right subtree, not just its immediate children.

**Attempt cold before reading on.**

### Recap: The Approach

DFS, carrying a `(min, max)` bound down through the recursion — each node must fall strictly between the bounds it inherits, and each recursive call **tightens** the relevant bound for its subtree: going left tightens the upper bound to the current node's value; going right tightens the lower bound. `Long`, not `int`, holds the bounds — node values can legally sit at `Integer.MIN_VALUE`/`MAX_VALUE`, so a nullable, wider type is needed to represent "no bound yet" without colliding with a real, valid value.

```java
public boolean isValidBST(TreeNode root) {
    return validate(root, null, null);
}

private boolean validate(TreeNode node, Long min, Long max) {
    if (node == null) return true;   // an empty subtree is trivially valid
    if ((min != null && node.val <= min) || (max != null && node.val >= max)) {
        return false;
    }
    return validate(node.left, min, (long) node.val) && validate(node.right, (long) node.val, max);
}
```

### Fresh Trace — Demonstrating *Why* Bounds Must Propagate, Not Just Compare to the Parent

Tree: root `10`, left child `5` (leaf), right child `15`, and `15` has left child `6`, right child `20`.

A naive approach comparing each node only to its **immediate** parent would check: `5 < 10` ✓, `15 > 10` ✓, `6 < 15` ✓, `20 > 15` ✓ — and wrongly conclude this tree is valid. It isn't: `6` sits in root `10`'s **right** subtree, so it must be greater than `10` — it's `6`, not greater at all.

```
validate(10, min=null, max=null)                     10: no bound violated
├─ validate(5, min=null, max=10)                       5 < 10 ✓, leaf → true
└─ validate(15, min=10, max=null)                      15 > 10 ✓
   ├─ validate(6, min=10, max=15)                      6 <= min(10) → VIOLATION → false
   └─ (short-circuits — right side never evaluated)

Result: false — correctly invalid.
```

The bound `min=10` reaching node `6` is the entire mechanism: it's not a comparison to `6`'s immediate parent (`15`, and `6 < 15` alone looks fine) — it's the accumulated constraint from **every ancestor `6` descended from**, correctly carrying root `10`'s own right-subtree requirement three levels down.

**Complexity:** `O(n)` time — every node visited exactly once. `O(h)` space for the recursion stack, where `h` is tree height — `O(log n)` for a balanced tree, `O(n)` worst case for a fully skewed one.

### Common Mistakes Checklist

- [ ] Comparing each node only to its immediate parent or immediate children, rather than propagating bounds from every ancestor — the exact failure mode traced above, and the single most common way this problem goes wrong.
- [ ] Using `int` with sentinel values (`Integer.MIN_VALUE`/`MAX_VALUE`) to represent "no bound yet" — breaks silently on the input where a real node value legitimately equals that sentinel, since it becomes indistinguishable from "no bound." `Long` with `null` sidesteps this correctly.
- [ ] Using `<=`/`>=` where strict `<`/`>` is required — the problem defines a valid BST as having **no duplicate values** anywhere in the comparison, so bound violations must be checked with strict inequality on both sides.

**If any of this needed re-deriving rather than confirming:** Week 8, Day 50 has the full original treatment.

---

# Part 2 — LLD #3: Parking Lot, Part 1 — Single-Threaded Core

### Steps 1–4, Briskly

**Requirements, clarified:** a multi-level garage. Vehicles come in a small number of size tiers (motorcycle, compact/car, large/bus); spots come in the same tiers. A vehicle may park in a spot sized for it **or larger** — a motorcycle can use a compact or large spot if nothing motorcycle-sized is free, but a bus can never use a compact spot. Parking issues a ticket recording the vehicle, its assigned spot, and entry time; leaving uses that ticket to free the spot. Payment, reservations, and multi-entrance routing are explicitly out of scope today.

**Core objects:** `ParkingLot` (owns every `Level`, tracks active tickets), `Level` (owns its own `ParkingSpot`s, finds an available one), `ParkingSpot` (sized, tracks its own occupancy), `Vehicle` (sized), `Ticket` (the park/leave correlation record).

**Pattern, considered and explicitly declined:** Singleton was considered for `ParkingLot` itself — realistically, only one instance should exist for a given physical garage. **Declined for this exercise:** nothing in the requirements demands the *class itself* enforce single-instantiation; a single instance created once by whatever wires the application together is simpler, with none of Week 15 Day 100's DCL/`volatile`/Enum-Singleton machinery earning its cost here. Singleton is worth its complexity when uncontrolled multiple instantiation would be an actual bug the class must prevent — not merely because a design happens, today, to only ever create one instance.

### Step 5 — Code the Core

```java
public enum VehicleSize {
    MOTORCYCLE, COMPACT, LARGE   // deliberately ordered smallest to largest — see Part 3
}

public class Vehicle {
    private final String licensePlate;
    private final VehicleSize size;

    public Vehicle(String licensePlate, VehicleSize size) {
        this.licensePlate = licensePlate;
        this.size = size;
    }

    public String getLicensePlate() { return licensePlate; }
    public VehicleSize getSize() { return size; }
}
```

```java
public class ParkingSpot {
    private final String spotId;
    private final VehicleSize spotSize;
    private Vehicle parkedVehicle;

    public ParkingSpot(String spotId, VehicleSize spotSize) {
        this.spotId = spotId;
        this.spotSize = spotSize;
    }

    // A vehicle fits if the spot is unoccupied AND at least as large as the vehicle needs —
    // ordinal comparison works ONLY because VehicleSize's declaration order IS the size order.
    public boolean canFit(Vehicle vehicle) {
        return !isOccupied() && vehicle.getSize().ordinal() <= spotSize.ordinal();
    }

    public void assignVehicle(Vehicle vehicle) {
        if (!canFit(vehicle)) {
            throw new IllegalStateException("Vehicle " + vehicle.getLicensePlate() + " cannot fit in spot " + spotId);
        }
        this.parkedVehicle = vehicle;
    }

    public void removeVehicle() {
        this.parkedVehicle = null;
    }

    public boolean isOccupied() {
        return parkedVehicle != null;
    }

    public String getSpotId() { return spotId; }
    public VehicleSize getSpotSize() { return spotSize; }
}
```

```java
public class Level {
    private final int levelNumber;
    private final List<ParkingSpot> spots;

    public Level(int levelNumber, List<ParkingSpot> spots) {
        this.levelNumber = levelNumber;
        this.spots = spots;
    }

    // Picks the SMALLEST fitting spot, not just the first one found — a deliberate
    // allocation choice: parking a motorcycle in a free motorcycle spot instead of
    // a free large spot avoids stranding large-vehicle capacity on small vehicles.
    public ParkingSpot findAvailableSpot(Vehicle vehicle) {
        ParkingSpot bestFit = null;
        for (ParkingSpot spot : spots) {
            if (spot.canFit(vehicle)) {
                if (bestFit == null || spot.getSpotSize().ordinal() < bestFit.getSpotSize().ordinal()) {
                    bestFit = spot;
                }
            }
        }
        return bestFit;
    }

    public int getLevelNumber() { return levelNumber; }
}
```

```java
public class Ticket {
    private final String ticketId;
    private final Vehicle vehicle;
    private final ParkingSpot spot;
    private final LocalDateTime entryTime;

    public Ticket(String ticketId, Vehicle vehicle, ParkingSpot spot, LocalDateTime entryTime) {
        this.ticketId = ticketId;
        this.vehicle = vehicle;
        this.spot = spot;
        this.entryTime = entryTime;
    }

    public String getTicketId() { return ticketId; }
    public Vehicle getVehicle() { return vehicle; }
    public ParkingSpot getSpot() { return spot; }
    public LocalDateTime getEntryTime() { return entryTime; }
}
```

```java
public class ParkingLot {
    private final List<Level> levels;
    private final Map<String, Ticket> activeTickets = new HashMap<>();
    private int nextTicketId = 1;

    public ParkingLot(List<Level> levels) {
        this.levels = levels;
    }

    public Ticket parkVehicle(Vehicle vehicle) {
        for (Level level : levels) {
            ParkingSpot spot = level.findAvailableSpot(vehicle);
            if (spot != null) {
                spot.assignVehicle(vehicle);
                String ticketId = "T-" + (nextTicketId++);
                Ticket ticket = new Ticket(ticketId, vehicle, spot, LocalDateTime.now());
                activeTickets.put(ticketId, ticket);
                return ticket;
            }
        }
        return null;   // no fitting spot anywhere in the lot for this vehicle's size
    }

    public boolean removeVehicle(String ticketId) {
        Ticket ticket = activeTickets.get(ticketId);
        if (ticket == null) {
            return false;   // invalid or already-used ticket
        }
        ticket.getSpot().removeVehicle();
        activeTickets.remove(ticketId);
        return true;
    }
}
```

### Worked Trace — Allocation Direction, Proven Both Ways

Level 1: `M1`(MOTORCYCLE), `C1`(COMPACT), `C2`(COMPACT), `L1`(LARGE) — one level, one `ParkingLot`.

```
park(BIKE-1, MOTORCYCLE)  → fits M1, C1, C2, L1 (ordinals 0<=0, 0<=1, 0<=1, 0<=2)
                              smallest-ordinal fit wins → M1 assigned. Ticket T-1.
park(CAR-1, COMPACT)      → M1 excluded (occupied); fits C1, C2, L1 (1<=1, 1<=1, 1<=2)
                              smallest fit among these → C1 assigned. Ticket T-2.
park(BUS-1, LARGE)        → M1, C1 excluded (occupied); C2 excluded — LARGE(2) doesn't
                              fit COMPACT(2<=1 is false); only L1 fits → L1 assigned. Ticket T-3.
park(BIKE-2, MOTORCYCLE)  → M1, C1, L1 excluded (occupied); C2 available, fits (0<=1)
                              → C2 assigned. Ticket T-4.  Level now fully occupied.

removeVehicle(T-1)         → M1 freed.
park(CAR-2, COMPACT)      → M1 is free but too small: COMPACT(1) <= MOTORCYCLE(0) is FALSE
                              — CAR-2 does NOT fit M1, and every other spot is still occupied
                              → findAvailableSpot returns null → parkVehicle returns null (lot full for this size)
```

The last two lines are the important proof: freeing a **smaller** spot never helps a **larger** vehicle waiting to park — `canFit`'s ordinal comparison enforces that directionality exactly, not just in the "happy path" allocations above it.

**Complexity:** `findAvailableSpot` is `O(s)` per level, where `s` is that level's spot count — it must inspect every spot to find the smallest fit, not just the first fit. `parkVehicle` is `O(L × s)` across `L` levels in the worst case (no fitting spot anywhere). `removeVehicle` is `O(1)` — a direct map lookup by ticket ID.

---

# Part 3 — Theory: Composition Over Inheritance, Applied (1 hr)

**The instinct has been building since Week 1, Day 5** (`implements Runnable` preferred over `extends Thread`, for the same single-inheritance reason that favors composition/interfaces generally) — today gives it a full, concrete argument, grounded directly in a design decision this very system just made.

### The Flawed Alternative — Built, Not Just Described

Here's what `ParkingSpot` would have looked like using inheritance — one subclass per size tier, instead of today's single class with a `VehicleSize` field:

```java
// THE FLAWED VERSION — shown to critique directly, not used anywhere in this system.
public abstract class ParkingSpot {
    private Vehicle parkedVehicle;
    public abstract boolean canFit(Vehicle vehicle);
    // ... shared assign/remove/isOccupied logic, identical to the composition version
}

public class MotorcycleSpot extends ParkingSpot {
    @Override
    public boolean canFit(Vehicle vehicle) {
        return vehicle.getSize() == VehicleSize.MOTORCYCLE;   // exact match only
    }
}

public class CompactSpot extends ParkingSpot {
    @Override
    public boolean canFit(Vehicle vehicle) {
        return vehicle.getSize() == VehicleSize.MOTORCYCLE || vehicle.getSize() == VehicleSize.COMPACT;
    }
}

public class LargeSpot extends ParkingSpot {
    @Override
    public boolean canFit(Vehicle vehicle) {
        return true;   // accepts everything — reasoned out independently, by hand, for this one class
    }
}
```

### What Concretely Breaks — Traced, Not Asserted

Suppose a new tier is added: `OVERSIZED`, for RVs, which should fit **only** in a new `OversizedSpot` — not in `LargeSpot`.

- `MotorcycleSpot` and `CompactSpot` need no changes — their exact-match logic happens not to be affected.
- **`LargeSpot` must change.** Its `canFit` currently returns `true` unconditionally — meaning it currently, silently, accepts `OVERSIZED` vehicles too. Once `OVERSIZED` exists as a distinct concept, `LargeSpot.canFit` is now **wrong** and must be edited to explicitly exclude it: `return vehicle.getSize() != VehicleSize.OVERSIZED;` (or an equivalent rewrite).

That's a direct **Open/Closed violation** (Week 1, Day 6; the exact same mechanism Week 15, Day 101 used to formally separate Simple Factory from true Factory Method) — adding a new tier forced a change to an *existing, already-tested* class, `LargeSpot`, not just the addition of a new one. And the deeper issue isn't just that one edit: the rule "which vehicle sizes fit which spot sizes" is scattered across `N` independently hand-written `canFit` bodies, each one a separate place that same rule could silently drift out of sync with the others — nothing in this design expresses "sizes have an order, and bigger accepts smaller" as *one* fact anywhere; it's `N` boolean expressions that merely happen, today, to agree with each other.

### The Composition Version — Already Built, Now Justified Retroactively

Today's actual `ParkingSpot` (Part 2 above) uses a `VehicleSize spotSize` **field**, and exactly one fit rule: `vehicle.getSize().ordinal() <= spotSize.ordinal()`. Adding `OVERSIZED`:

- **One line changes** — one new enum constant, appended to `VehicleSize`.
- **Zero existing classes change.** `ParkingSpot`'s `canFit` method is untouched — the ordinal comparison automatically, correctly handles the new tier, because it encodes the general *rule* ("bigger accepts smaller") rather than enumerating specific cases.
- **The rule exists in exactly one place** — one method, testable once, instead of `N` hand-written bodies to keep independently consistent.

### 🔑 Key Takeaway — When This Generalizes, and When It Doesn't

**The tell, worth being precise about rather than treating "composition good, inheritance bad" as a blanket rule:** this specific case is a textbook fit for composition because the varying dimension (size) has a **natural total order**, and every variant's behavior is a **pure function of that one ordered value** — nothing about a `CompactSpot` genuinely behaves differently from a `LargeSpot` beyond "which sizes fit." Inheritance still earns its place elsewhere in this exact series — Week 15's Creational patterns (`SavingsAccount extends Account`, Day 100–101's Factory Method hierarchy) use it correctly, because there, each subtype has **genuinely distinct behavior** (a different interest calculation, a different construction step), not just a different value along one shared, orderable axis. The lens isn't "never subclass" — it's "is the difference between variants a *value* (favor a field) or a *behavior* (favor a subclass, or one of Days 106–107's patterns)?"

> ⚠️ **Common Mistake:** reaching for inheritance the moment two or more "kinds" of something appear, without first checking whether the actual difference between them is expressible as data. `MotorcycleSpot`/`CompactSpot`/`LargeSpot` *look* like a natural class hierarchy — three related things, each a specialization of a common concept — right up until the moment a fourth tier needs adding and the cost above becomes concrete instead of theoretical.

---

# Project Block Guide (3.5 hrs)

**Repository:** `lld-java`. **Module:** new — `parking-lot/`.

**Task:** the full single-threaded implementation from Part 2 — `VehicleSize`, `Vehicle`, `ParkingSpot`, `Level`, `Ticket`, `ParkingLot`.

**Definition of done:**
- `parkVehicle` correctly assigns the smallest fitting spot, respecting size directionality (verified with unit tests covering at least: an exact-size match, a smaller vehicle using a larger spot, a larger vehicle correctly rejected from a smaller spot, and a full lot correctly returning `null`).
- `removeVehicle` correctly frees the spot and invalidates the ticket (verified by confirming a second `removeVehicle` call with the same ticket ID returns `false`).
- Class diagram committed.
- Pushed to `lld-java/parking-lot/`.

**The write-up deliverable — composition over inheritance:** Part 3 above *is* the model for this — read it once fully, then write your own version, in your own words, covering the same three beats: what the inheritance-based `LargeSpot` change concretely breaks when a new tier is added, why the composition version avoids it, and the "value vs. behavior" tell for when each approach is actually the right call. The exercise is the writing itself, not just reading Part 3 and agreeing with it.

---

# Career Block Guide (1 hr)

**LinkedIn engagement:** comment on 5 posts today — a bit more than the usual 3–5, still engagement, not a new post (Day 111 is a new-post day).

**Networking — reference cultivation.** Reach out to 2–3 former colleagues or managers specifically to reconnect ahead of eventually asking for a reference — a short, warm, no-ask message now (catching up, sharing what you've been building) reads very differently from a cold ask that arrives the same week a reference is actually needed. Planting this early, the same way Day 5's accountability-partner search was planted ahead of when mocks actually started, is deliberate.

---

# Day 110 — Interview Questions

**Q1. Why must vehicle-to-spot size fit be directional rather than an exact match?** A smaller vehicle can safely occupy a larger spot (a motorcycle fits fine in a compact or large spot), but a larger vehicle cannot occupy a smaller one (a bus cannot fit in a compact spot) — the relationship is an ordering, not an equivalence, and the design needs to reflect that asymmetry directly.

**Q2. Why does `findAvailableSpot` pick the smallest fitting spot instead of the first one found?** Assigning a motorcycle to a free large spot when a free motorcycle spot also exists would strand that large-vehicle capacity unnecessarily — picking the smallest sufficient spot avoids wasting spots that only bigger vehicles can actually use.

**Q3. Trace what happens when a COMPACT vehicle tries to park after only a MOTORCYCLE-sized spot has been freed.** `canFit` checks `vehicle.getSize().ordinal() <= spotSize.ordinal()` — COMPACT's ordinal (1) is not `<=` MOTORCYCLE's ordinal (0), so the check fails; the vehicle does not fit, regardless of the spot being unoccupied. Freeing a smaller spot never helps a larger vehicle.

**Q4. Why was Singleton considered for `ParkingLot` and then explicitly declined?** Realistically only one instance should exist for a given garage, but nothing in the requirements demands the *class itself* enforce that — a single instance created once by the application's own wiring is simpler, and Singleton's real cost (thread-safety machinery, as built in Week 15) is only worth paying when uncontrolled multiple instantiation would be an actual bug the class must actively prevent.

**Q5. Concretely, what breaks in the inheritance-based `ParkingSpot` hierarchy when a new size tier is added?** `LargeSpot.canFit`, which currently returns `true` unconditionally (accepting everything), becomes wrong the moment a new, larger tier exists that should *not* fit in a `LargeSpot` — it must be edited to explicitly exclude the new tier, meaning an existing, already-tested class had to change to accommodate new behavior, a direct Open/Closed violation.

**Q6. State the general tell for when a varying dimension should be a field (composition) versus a subclass (inheritance).** If the difference between variants is a *value* along some naturally ordered or comparable axis, and every variant's behavior is a pure function of that value, favor a field. If variants have *genuinely distinct behavior* that isn't reducible to comparing a shared value, a subclass (or one of Days 106–107's patterns) is the better fit.

**Q7. In Validate BST, construct a tree where comparing each node only to its immediate parent gives the wrong answer, and explain why.** Root `10`, left `5`, right `15` (with `15`'s own left child `6`, right child `20`). Immediate-parent comparisons all individually pass (`5<10`, `15>10`, `6<15`, `20>15`), but `6` sits in root `10`'s right subtree, so it must exceed `10` — it doesn't, making the tree invalid; only bounds propagated from every ancestor, not just the immediate parent, catch this.

**Q8. Why does Validate BST's bound-tracking use `Long` with nullable bounds rather than `int` with `Integer.MIN_VALUE`/`MAX_VALUE` sentinels?** A real node value can legitimately equal `Integer.MIN_VALUE` or `MAX_VALUE`, which would be indistinguishable from the "no bound yet" sentinel under that scheme; a wider type (`Long`) with an explicit `null` for "unbounded" avoids the collision entirely.

---

## Daily Deliverable Check

- [ ] Validate Binary Search Tree (LC 98) solved cold, without hints.
- [ ] Parking Lot single-threaded core complete and unit-tested; class diagram committed.
- [ ] Composition-over-inheritance write-up completed, in your own words.
- [ ] 2–3 former colleagues/managers reconnected with.

---

## What Tomorrow Assumes You Already Know Cold

Day 111 takes today's exact `ParkingLot`/`Level`/`ParkingSpot` design and makes `assignSpot` thread-safe under concurrent access — it assumes today's single-threaded correctness (the size-directionality proof, the smallest-fit allocation) is fully settled, since tomorrow's entire job is preserving that correctness while multiple threads compete for the same spots, not re-deriving what "correct" means from scratch. It also assumes `synchronized`/`ReentrantLock` (Week 6, Day 37) and `ConcurrentHashMap`'s CAS-based locking (Week 6, Day 39) are reflexive — tomorrow's Theory Block on lock granularity is a direct extension of both, not a re-introduction.
