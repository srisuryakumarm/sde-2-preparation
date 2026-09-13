# Week 16 — Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Week:** [Day 106](Day106_Resource_Book.md) · [Day 107](Day107_Resource_Book.md) · [Day 108](Day108_Resource_Book.md) · [Day 109](Day109_Resource_Book.md) · [Day 110](Day110_Resource_Book.md) · [Day 111](Day111_Resource_Book.md) · [Day 112](Day112_Resource_Book.md)
**Companion to:** `Week_16_Revised.md`

---

Every interview question from this week's seven Resource Books, pulled into one document for review — 64 questions total, covering the 5-step LLD framework, all ten Structural and Behavioral patterns, TDD's Red-Green-Refactor cycle, Coupling/Cohesion/Law of Demeter, Composition over Inheritance, concurrency and lock-granularity trade-offs, and six cold-solved DSA revision problems (Course Schedule, Coin Change, Combination Sum, Longest Substring Without Repeating Characters, Validate BST, Redundant Connection). Organized by day, in the order originally taught — each question stands on its own, but the day headers make it easy to jump back to a specific Resource Book for the full derivation if an answer doesn't come immediately.

---

## Day 106 — The LLD Interview Framework, and Structural Patterns

**Q1. Walk through the 5-step LLD framework from memory.** Clarify requirements → identify core objects (nouns become classes, verbs become methods) → define relationships and sketch a class diagram (composition vs. inheritance, associations, multiplicities) → apply patterns deliberately, only where a specific requirement calls for one → code the core (skeletons, key relationships, one or two representative method bodies).

**Q2. Why is skipping requirements clarification specifically dangerous, rather than just a missed nicety?** Every later step builds directly on the assumed scope from Step 1 — wrong requirements produce wrong core objects, which produce a wrong class diagram, which gets wrong patterns applied to it. There's no later step that corrects the error; it only compounds.

**Q3. What are the three GoF pattern categories, and what distinguishes them?** Creational (how objects get created — Singleton, Factory Method, Builder), Structural (how objects are composed into larger structures — today's five), Behavioral (how objects communicate and vary their behavior at runtime — Observer, Strategy, State, Command, Template Method, Day 107).

**Q4. Adapter and Facade both wrap existing code behind a simpler interface. How do they differ?** Adapter bridges exactly two incompatible interfaces so one can stand in for the other — a 1-to-1 translation. Facade simplifies a *multi-class subsystem* into one smaller interface, coordinating several calls in the right order — a many-to-one simplification, not a translation between two shapes.

**Q5. Why can't `CondimentDecorator.wrappedBeverage` be typed as `Espresso` instead of `Beverage`?** Decorators need to wrap *anything* implementing `Beverage`, including other decorators — typing the field as the concrete `Espresso` class would make it impossible to wrap a `Milk`-decorated beverage in a `Caramel` decorator, which is exactly what makes stacking multiple decorators possible.

**Q6. Trace what `getCost()` returns for `new Caramel(new Espresso())`.** `Caramel.getCost()` calls `wrappedBeverage.getCost()` — the `Espresso` — which returns `1.99` directly (no further delegation, the base case); `Caramel` then adds its own `0.75`, returning `2.74`.

**Q7. Why is Decorator's `getCost()` resolution the same mechanism as Day 8's recursion, not just a similar-looking pattern?** Each decorator's `getCost()` is a recursive case — it delegates to the wrapped object for a partial answer before combining it with its own contribution — and the base component (`Espresso`) is the base case, returning a fixed value with no further delegation. The call chain unwinds exactly like a recursive call stack: work happens on the way back up.

**Q8. Distinguish Proxy from Decorator — both wrap an object behind the same interface.** Decorator adds genuinely new behavior or responsibility the wrapped object never had. Proxy controls access to behavior the object already fully has — logging, lazy construction, or permission checks around a call — without ever changing what the underlying operation fundamentally does.

**Q9. Why is a virtual (lazy) proxy's amortized cost O(1) per call, not O(1) for every call including the first?** The expensive construction happens exactly once, on the first call that actually needs the real object — every call after that just delegates to the already-constructed instance. "Amortized" specifically means the one expensive call is being averaged in with all the cheap ones that follow, not that the first call itself is cheap.

**Q10. Why does Composite's `getSize()` on a Folder need to call `getSize()` on each child rather than maintaining a running total field?** A running total field would need to be kept in sync every time any descendant, at any depth, changes size — a correctness burden with many chances to drift out of sync. Recomputing via recursive delegation is always correct by construction, at the cost of re-walking the subtree on every call; if that recomputation cost mattered in practice, caching with explicit invalidation would be the fix, not a naively-maintained running total.

**Q11. Why does `FileSystemComponent` not define an `add()` method, when `Folder` clearly needs one?** A leaf (`FileItem`) has no children by definition — giving it an `add()` method would either do nothing (silently swallowing a caller's mistake) or throw at runtime, when the type system could instead make "you can't add to a file" a compile-time fact by keeping `add()` off the shared interface and only on `Folder`.

**Q12. Give the concrete, requirement-level signal that should make you reach for each of the five Structural patterns.** Adapter — integrating with an interface you don't control. Decorator — behaviors that combine freely, in combinations the class author can't enumerate ahead of time. Facade — a multi-class subsystem that needs a specific call order hidden behind one entry point. Proxy — controlling access to an object without changing its own code. Composite — a recursive, whole-part structure that needs uniform treatment of single items and groups.

---

## Day 107 — Behavioral Patterns, and TDD Practice

**Q1. Distinguish Observer from Strategy — both involve a class holding a reference to something implementing an interface.** Observer is 1-to-many and reactive: a subject pushes updates to a *set* of observers whenever its own state changes, and the observers don't choose anything. Strategy is 1-to-one and client-directed: a client hands a context exactly *one* algorithm implementation to use, chosen deliberately, not triggered by a state change.

**Q2. Why does `WeatherStation.notifyDisplays()` never need to change when a new `Display` type is added?** It iterates a `List<Display>` and calls `update()` polymorphically — it depends only on the `Display` interface, never on any concrete display class by name. A new display type just needs to implement `Display` and register itself.

**Q3. What's the actual distinguishing question between State and Strategy, in one sentence?** Does the object's own internal lifecycle drive the behavior change (State), or does an external client choose the behavior (Strategy)? Both look structurally similar in code; the difference is *who's in control of the swap and why it happens*.

**Q4. Why is `TrafficLight` itself free of any conditional logic about what state comes next?** Every transition rule lives inside the state class it belongs to (`RedState.next()` creates a `GreenState`, etc.) — `TrafficLight.advance()` only ever calls `currentState.next(this)`, delegating the decision entirely rather than checking `if (currentColor == RED) ...`.

**Q5. How does Command enable undo, queuing, and logging from the same underlying mechanism?** All three come from making a request a stored *object* rather than an immediate method call — a `Deque<Command>` history supports undo (pop and call the inverse), the identical structure as a `List<Command>` supports queuing (process later, in any order), and a command serializing its own state before executing supports logging. One design decision, three payoffs.

**Q6. Why must Template Method's `run()` be declared `final`?** If a subclass could override it, it could reorder or skip steps entirely, defeating the pattern's core guarantee — that the sequence is fixed across every variant, with only specific named steps left to vary.

**Q7. What is the actual purpose of TDD's Red phase, beyond "write a test"?** It forces the API to be designed from the caller's perspective before any implementation exists, since the test is the first piece of code to actually *use* the not-yet-built class — this tends to produce simpler, more caller-friendly interfaces than writing the implementation first and testing it afterward.

**Q8. Why does Green deliberately aim for the *minimum* code to pass, rather than the best implementation right away?** Small, minimal steps keep each change easy to reason about — if the test fails after a tiny change, the cause is almost always that change. Writing a large, "final" implementation before any test has passed reintroduces the exact untested-guesswork risk TDD exists to avoid.

**Q9. In Course Schedule, why does an edge go from the prerequisite to the dependent course (`prereq → course`), not the other way?** Kahn's Algorithm processes nodes with in-degree 0 first — courses with no remaining prerequisites. An edge `prereq → course` correctly increments `course`'s in-degree for each prerequisite it has; reversing the edge direction would compute in-degrees for the wrong relationship and silently produce wrong results.

**Q10. Why does Coin Change's DP allow `dp[i - coin]` to already include uses of the same coin?** The problem allows unlimited reuse of each denomination (unbounded knapsack) — there's no constraint tracking "how many of this coin have I used," only "how much amount remains," so reusing the same coin type multiple times toward the same total is exactly the intended behavior, not a bug.

**Q11. Give a concrete counterexample showing why a greedy approach fails for Coin Change.** `coins = [1, 3, 4]`, `amount = 6`: greedy takes the largest fitting coin first (`4`), then is forced into `1 + 1`, totaling 3 coins; the true optimum is `3 + 3`, totaling 2 coins. Greedy's locally-largest choice isn't always part of the globally optimal solution here.

---

## Day 108 — LLD #1: Tic-Tac-Toe, and Coupling, Cohesion, and the Law of Demeter

**Q1. Why does Tic-Tac-Toe's Step 4 correctly conclude "no pattern needed," and why is that conclusion itself worth stating out loud in an interview?** None of the ten patterns from the last two days solve a problem this system actually has — no incompatible interface, no combinable behaviors, no complex subsystem, no access-control need, no recursive whole-part structure. Stating this explicitly signals the same judgment Day 106 flagged as a common mistake in reverse: recognizing when *not* to apply a pattern is as much a demonstrated skill as applying one correctly.

**Q2. Why does `Board.isWinningMove` check only the row/column/diagonal through the just-placed cell, instead of rescanning the whole board?** It's strictly less work for an equally correct result — a move can only possibly create a win along a line that passes through the cell that just changed; every other line's status is unaffected by this move and doesn't need re-checking.

**Q3. What's the concrete difference between low coupling and high cohesion — they're often confused?** Coupling is about relationships *between* classes — how much one depends on another's internals. Cohesion is about *within* a single class — how focused its own responsibilities are. A class can be low-coupled to everything else and still be low-cohesion internally (a "god class" with a clean external interface hiding an unfocused jumble inside), so they're genuinely separate axes.

**Q4. Why is `game.getBoard().print()` a Law of Demeter violation, and what's the fix?** It chains through `Game` to reach a method on the `Board` object it returns — the caller ends up depending on both `Game` having a `Board` and `Board` having `print()`. The fix is `Game` exposing its own `printBoard()` that delegates internally, so callers only ever depend on `Game`'s contract.

**Q5. Why is `game.getCurrentPlayer().getName()` conventionally *not* treated as a violation, even though it's also a two-hop chain?** The Law of Demeter is conventionally relaxed for simple, immutable data-holder objects with no real internal behavior or invariants to protect — `Player` has nothing a caller could break by reading a field off it, unlike a stateful object like `Board`.

**Q6. In Combination Sum, why does the recursive call pass `i` rather than `i + 1`?** The problem allows reusing the same candidate an unlimited number of times; passing `i` keeps that same index eligible again on the next recursive level, while still using `start` to prevent revisiting *earlier* indices, which is what stops duplicate combinations in a different order from being generated separately.

**Q7. Derive Combination Sum's worst-case time complexity rather than stating it from memory.** With `N` candidates, target `T`, and minimum candidate value `M`: each call branches into at most `N` children, and `remaining` shrinks by at least `M` per level, bounding depth at `⌈T/M⌉`. Branching factor `N` to depth `T/M + 1` gives a loose upper bound of `O(N^(T/M + 1))` total nodes in the recursion tree.

**Q8. Why is omitting Combination Sum's `remaining < 0` prune a correctness bug, not just a slowdown?** Without it, the branch that keeps re-choosing the same smallest reusable candidate never lands exactly on `remaining == 0` and has no other way to stop — it recurses indefinitely down that path, risking a `StackOverflowError` rather than merely doing avoidable extra work.

---

## Day 109 — LLD #2: Vending Machine (State Pattern), and Mock Interview #1

**Q1. What's the two-sentence distinction between State and Strategy?** They share the same code shape — a context delegating to an interface reference — but differ in who drives the swap: State's transitions are triggered by the object's own operations between a small, closed set of internal phases, with states often referencing each other; Strategy's choice is made deliberately by an external client, and strategies are typically unaware of one another.

**Q2. Give the practical constructor-shape tell for distinguishing an ambiguous State-or-Strategy design.** State implementations are often parameterless, stateless, and safely reusable as singletons; Strategy implementations frequently carry their own configuration data in the constructor. Not universal, but a useful, checkable signal when the intent alone doesn't settle it.

**Q3. Why are `VendingMachine`'s four state objects created once as fields, rather than `new`'d on every transition?** They hold no per-transaction data — they're stateless with respect to any individual purchase — so re-instantiating them on every transition allocates memory for no behavioral benefit; reusing fixed singleton instances is the idiomatic version of State whenever the state classes carry no data of their own.

**Q4. Trace what happens if `selectProduct` is called while the machine is in `SoldOutState`.** `SoldOutState.selectProduct` runs — it prints "Machine is sold out" and performs no state change, no balance change, and no dispense. The call is fully absorbed by the current state object; `VendingMachine` itself does nothing state-specific at all.

**Q5. Why does a failed selection (out of stock, or insufficient balance) leave the machine in `HasCoinState` rather than reverting to `NoCoinState`?** The customer's balance is still present and still valid — reverting to `NoCoinState` would either strand that balance or force it to be re-entered; staying in `HasCoinState` correctly allows a different, valid selection using the same already-inserted money.

**Q6. Why does `HasCoinState.selectProduct` transition to `DispensingState` and then immediately call `dispense()` on it, rather than letting a later call trigger dispensing?** Once payment is validated as sufficient, dispensing isn't a separate customer-triggered action — it's a direct, immediate consequence of a successful selection, so the transition and the action happen together rather than waiting for a call that doesn't correspond to anything the customer does.

**Q7. In Longest Substring Without Repeating Characters, why must the duplicate-index check include `>= left`, not just check whether the character was ever seen before?** A character seen earlier but **outside** the current window (before `left`) isn't actually a duplicate *within* the live window — treating it as one would incorrectly move `left` backwards, corrupting the window and producing a wrong (too-small or logically inconsistent) answer.

**Q8. Why is the two-pointer scan for LC 3 O(n) overall rather than O(n²), given `left` moves inside the loop that `right` also drives?** `left` only ever moves forward, never backward, across the algorithm's *entire* run — so even though it moves inside `right`'s loop, the total number of steps `left` takes across every iteration combined is bounded by `n`, not by `n` per iteration; the two pointers together do a bounded `O(n)` amount of total work.

---

## Day 110 — LLD #3: Parking Lot, Part 1 — Design and Core Logic

**Q1. Why must vehicle-to-spot size fit be directional rather than an exact match?** A smaller vehicle can safely occupy a larger spot (a motorcycle fits fine in a compact or large spot), but a larger vehicle cannot occupy a smaller one (a bus cannot fit in a compact spot) — the relationship is an ordering, not an equivalence, and the design needs to reflect that asymmetry directly.

**Q2. Why does `findAvailableSpot` pick the smallest fitting spot instead of the first one found?** Assigning a motorcycle to a free large spot when a free motorcycle spot also exists would strand that large-vehicle capacity unnecessarily — picking the smallest sufficient spot avoids wasting spots that only bigger vehicles can actually use.

**Q3. Trace what happens when a COMPACT vehicle tries to park after only a MOTORCYCLE-sized spot has been freed.** `canFit` checks `vehicle.getSize().ordinal() <= spotSize.ordinal()` — COMPACT's ordinal (1) is not `<=` MOTORCYCLE's ordinal (0), so the check fails; the vehicle does not fit, regardless of the spot being unoccupied. Freeing a smaller spot never helps a larger vehicle.

**Q4. Why was Singleton considered for `ParkingLot` and then explicitly declined?** Realistically only one instance should exist for a given garage, but nothing in the requirements demands the *class itself* enforce that — a single instance created once by the application's own wiring is simpler, and Singleton's real cost (thread-safety machinery, as built in Week 15) is only worth paying when uncontrolled multiple instantiation would be an actual bug the class must actively prevent.

**Q5. Concretely, what breaks in the inheritance-based `ParkingSpot` hierarchy when a new size tier is added?** `LargeSpot.canFit`, which currently returns `true` unconditionally (accepting everything), becomes wrong the moment a new, larger tier exists that should *not* fit in a `LargeSpot` — it must be edited to explicitly exclude the new tier, meaning an existing, already-tested class had to change to accommodate new behavior, a direct Open/Closed violation.

**Q6. State the general tell for when a varying dimension should be a field (composition) versus a subclass (inheritance).** If the difference between variants is a *value* along some naturally ordered or comparable axis, and every variant's behavior is a pure function of that value, favor a field. If variants have *genuinely distinct behavior* that isn't reducible to comparing a shared value, a subclass (or one of Days 106–107's patterns) is the better fit.

**Q7. In Validate BST, construct a tree where comparing each node only to its immediate parent gives the wrong answer, and explain why.** Root `10`, left `5`, right `15` (with `15`'s own left child `6`, right child `20`). Immediate-parent comparisons all individually pass (`5<10`, `15>10`, `6<15`, `20>15`), but `6` sits in root `10`'s right subtree, so it must exceed `10` — it doesn't, making the tree invalid; only bounds propagated from every ancestor, not just the immediate parent, catch this.

**Q8. Why does Validate BST's bound-tracking use `Long` with nullable bounds rather than `int` with `Integer.MIN_VALUE`/`MAX_VALUE` sentinels?** A real node value can legitimately equal `Integer.MIN_VALUE` or `MAX_VALUE`, which would be indistinguishable from the "no bound yet" sentinel under that scheme; a wider type (`Long`) with an explicit `null` for "unbounded" avoids the collision entirely.

---

## Day 111 — LLD #3: Parking Lot, Part 2 — Concurrency

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

## Day 112 — LLD #4: Library Management System, and Mock Interview #2

**Q1. Why are `Book` and `BookItem` two separate classes rather than one?** `Book` represents the catalog-level title, shared across every physical copy; `BookItem` represents one specific, independently-trackable physical copy. A library with 3 copies of one ISBN needs 3 independent statuses, not one shared status for the title — collapsing them into one class would make that impossible to represent correctly.

**Q2. Why is `BookItem`'s lifecycle a transition table rather than a full State pattern, when Vending Machine used State directly?** Vending Machine's states responded *differently* to the *same* triggers — genuinely varying behavior per state. `BookItem`'s statuses don't vary behavior this way; the only thing that changes per status is which transitions are valid, which is a data/validity problem, not a behavioral one — so it's expressed as data (Day 110's technique), not as four additional classes.

**Q3. How does `Catalog` provably have zero dependency on reservation logic, rather than just being described that way?** Its fields and imports contain no reference to `Member`, `Reservation`, or `BookItemStatus`-transition logic at all — the claim is checkable directly by reading the class, not something that has to be taken on faith.

**Q4. `searchAvailableCopiesByTitle` needs both catalog search and status information. Where does that composition happen, and why there specifically?** In `Library`, not `Catalog` — `Library` calls `catalog.searchByTitle`, then separately filters by each `BookItem`'s status. Composing at `Library` keeps `Catalog` itself entirely unaware that availability filtering exists, preserving the one-directional dependency instead of teaching `Catalog` about reservation state to support one specific query.

**Q5. Why is caching a `boolean isAvailable` flag directly on `Book`, updated on every status change, a design mistake here — not just a minor inefficiency?** It would force `Catalog` (which owns `Book`) to become aware of `BookItemStatus` changes happening inside `Library`'s circulation logic — reintroducing exactly the dependency direction the separation was built to prevent, in exchange for an optimization the actual access pattern doesn't clearly need.

**Q6. In Mock Interview #2's escalation, why is narrating a coarse-but-correct lock first, then proposing a refinement, often a stronger live-interview move than jumping straight to the optimized answer?** It demonstrates the reasoning process, not just the destination — an interviewer watching someone silently produce a fine-grained answer learns little about how they think; watching them state a safe baseline and then justify improving on it shows the actual judgment being evaluated.

**Q7. What specifically should be named, out loud, when the "now make it thread-safe" escalation lands — not just "we need to synchronize this"?** The precise check-then-act race (which two steps are non-atomic, and how two threads could interleave through the gap), followed by a specific granularity choice with a stated reason — matching Day 111's own standard, not a vague gesture toward "adding some locking."

---

## Using This Bank

Six questions in ten cover the "why," not just the "what" — reasoning through *why* an approach works, *why* a design choice was made over an alternative, or *why* a common mistake actually breaks something, rather than only recalling a definition. That's deliberate: a tier-1 interviewer pushes on justification, not vocabulary. If any answer above doesn't come immediately, the fix isn't memorizing that specific answer — it's going back to the linked day, rereading the full derivation, and confirming the underlying mechanism is actually understood before moving on.
