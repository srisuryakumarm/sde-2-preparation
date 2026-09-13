# Week 16 (Revised): LLD Framework, Full Pattern Coverage, and the First Four Systems

**What changed:** Proxy, Composite, and Template Method join the pattern lineup (folded into the existing framework days, not extra ones) — three GoF patterns that come up often enough in LLD interviews to be worth having, beyond what the original plan covered. More importantly: mock interviews now start after the *second* system instead of being saved entirely for Week 21. You'll do five of them across the LLD phase instead of one.

---

## Day 106 — The LLD Interview Framework, and Structural Patterns

### Theory Block (3 hrs) — dedicated space, since this framework gets reused for every remaining LLD in this plan

**The LLD Interview Framework**
1. **Clarify requirements** (5-10 min) — what's actually in scope? A Parking Lot LLD might or might not need payment processing, multiple vehicle types, or multi-level support — ask, don't assume. This is the single most skipped step, and skipping it is the most common reason candidates build the wrong system correctly.
2. **Identify core objects** — turn the nouns in the requirements into candidate classes. A "Parking Spot," a "Vehicle," a "Ticket" — each becomes a class; verbs often become methods on those classes.
3. **Define relationships and draw a class diagram** — composition vs. inheritance, associations, multiplicities. This doesn't need to be UML-perfect — a legible sketch showing the classes, their key fields, and how they relate is enough.
4. **Apply patterns deliberately, not decoratively** — a pattern should solve a specific requirement (multiple payment strategies → Strategy; a vending machine's mode-dependent behavior → State), not be bolted on to demonstrate you know its name.
5. **Code the core** — you won't finish every method in a real interview; prioritize the class skeletons, the key relationships, and one or two representative method bodies that show you can actually implement what you designed.

Common mistakes to actively avoid: designing for scale (that's a System Design question, not LLD); over-engineering with patterns nothing requires; skipping requirements clarification and guessing wrong; spending all your time on one class while the rest stay unsketched.

**Why step 5 matters more than it might seem:** Atlassian's actual LLD round ("Code Design") requires a fully working solution, not a diagram — and their "Data Structures" round explicitly isn't traditional LeetCode, it's a real-world scenario where you justify your choice out loud. Every system from here forward gets built to genuinely compile and run, not sketched and abandoned once the class diagram looks right.

- Topic: Structural Patterns — Adapter, Decorator, Facade, Proxy, Composite
- **Adapter** converts one interface into another a client expects, bridging incompatible interfaces without modifying either one. **Decorator** adds behavior to an object dynamically by wrapping it, instead of needing a new subclass for every possible combination of behaviors. **Facade** provides one simplified interface over a complex subsystem with many moving parts underneath. **Proxy** controls access to another object — logging every call, lazily creating an expensive object only when first needed, or checking permissions before letting a call through — while presenting the exact same interface as the real thing. **Composite** lets you treat a single object and a group of objects through the same interface, which is what makes recursive tree structures (a file system, a UI component hierarchy) composable without the client code needing to know whether it's holding one item or a whole subtree.
- Coding exercise: implement the Decorator pattern to price a coffee (base: Espresso; decorators: Milk, Caramel, Whip), where each decorator wraps the previous and adds its own cost. Separately, sketch (no full implementation needed) how Composite would model a file system where a `Folder` can contain both `File`s and other `Folder`s, all accessed through one shared interface.

### Project Block (1.5 hrs)
- Repository: `lld-java`.
- Task: the Coffee Decorator implementation above, in the `design-patterns` module, with a README note on when you'd prefer Decorator over simple subclassing.
- Definition of done: pushed, tested with at least 3 stacked decorators.

### Career Block (1 hr)
- LinkedIn: Post 20 — Month 3-ish recap (problems solved, patterns mastered, the platform shipped so far — a strong milestone post).
- Networking: reply to any recruiter inbound.

### Daily Deliverable
- [ ] Can recite the 5-step LLD framework and the common mistakes without notes.
- [ ] Coffee Decorator implementation pushed. Composite sketch complete.
- [ ] LinkedIn Post 20 published.

---

## Day 107 — Behavioral Patterns, and TDD Practice

### Theory Block (2.5 hrs)
- Topic: Behavioral Patterns — Observer, Strategy, State, Command, Template Method
- **Observer** lets subscribers react to state changes in a subject without tight coupling — a Weather Station pushing updates to multiple Display screens, none of which the station needs to know about specifically. **Strategy** lets you swap an algorithm's implementation at runtime behind a common interface, chosen by the client. **State** lets an object change its own behavior when its internal state changes, without a giant conditional block anywhere. **Command** encapsulates a request as an object, which is what enables queuing it, logging it, or undoing it later. **Template Method** defines the skeleton of an algorithm in a base class, with specific steps deferred to subclasses — the overall sequence stays fixed, but individual steps can vary.
- Coding exercise: implement the Observer pattern with a `WeatherStation` subject pushing temperature updates to multiple `Display` subscribers. Separately, sketch a `DataProcessor` Template Method (`readData()` → `process()` → `writeData()`, with `process()` left abstract for subclasses to fill in).

### DSA Block (1.5 hrs) — lighter today, theory-heavy day
- Revision: solve one Graph and one DP problem from earlier in this plan, cold, without hints — a spaced-repetition check now that the DSA curriculum is fully behind you and LLD/HLD are the focus going forward.

### Project Block (1.5 hrs)
- Repository: `lld-java`.
- Task: the Observer implementation above. Apply TDD retrospectively — write failing tests first, then the code to pass them.
- Definition of done: Red-Green-Refactor practiced and documented; pushed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: schedule a mock interview focused purely on LLD for this weekend — the first of five you'll do across this phase, not the only one.
- **Worth knowing now, even though the full treatment is Week 21:** Google's "Googleyness," Databricks' own leadership principles, and Atlassian's five named values are each evaluated as their own axis in those companies' loops — not generic behavioral questions. Real candidates report being downleveled specifically for treating these casually. Filing that away now so it's not a surprise later.

### Daily Deliverable
- [ ] Observer pattern implemented via TDD. Template Method sketch complete.
- [ ] Two revision problems solved cold.

---

## Day 108 — LLD #1: Tic-Tac-Toe (Warm-Up)

### DSA Block (1 hr)
- Revision: one Backtracking problem, cold.

### Project Block (3.5 hrs) — today's real work is applying the framework end to end for the first time

**LLD: Tic-Tac-Toe**
- Deliberately the simplest system in this plan — no concurrency, no persistence, a small fixed rule set. The point isn't the system's difficulty; it's running the full 5-step framework once, start to finish, before a harder system adds concurrency on top.
- Requirements to clarify (practice asking, even solo): board size fixed at 3x3 or configurable? Two players only, or support for more? How is a win detected — check after every move, or only at the end?
- Core objects: `Board`, `Player`, `Cell` (or a simple grid), a `Game` orchestrator.
- Task: design and implement the full system in `lld-java`, including win-detection logic that checks rows, columns, and both diagonals after every move.
- Definition of done: the game is playable end to end via a simple `main` method loop, correctly detects a win or draw, and the class diagram (even hand-sketched) is committed alongside the code.

### Theory Block (1 hr)
- Topic: Coupling, Cohesion, and the Law of Demeter
- High cohesion (a class does one thing well) and low coupling (classes depend on as little of each other's internals as possible) are the underlying goals every pattern from this week serves. The Law of Demeter — "only talk to your immediate friends" — is a concrete heuristic: avoid chains like `a.getB().getC().doSomething()`, which silently couples you to B and C's internal structure even though you only meant to talk to A.
- Coding exercise: none — apply this lens retroactively to today's Tic-Tac-Toe design; note anywhere you reached through more than one object to get something done, and consider whether that's a smell worth fixing.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: begin the targeted ramp — start connecting with engineers specifically at Rippling, Google, Databricks, Stripe, Uber, Atlassian, and Walmart Global Tech (per `Target_Company_Research_and_Interview_Guide.md`), aiming for roughly 15-20 genuine connections per company by the time applications go out next week. Personalized notes under 300 characters, no ask in the note itself.

### Daily Deliverable
- [ ] Tic-Tac-Toe LLD complete, playable, win-detection correct.
- [ ] Class diagram committed alongside the code.

---

## Day 109 — LLD #2: Vending Machine (State Pattern), and Mock Interview #1

### DSA Block (1 hr)
- Revision: one Sliding Window problem, cold.

### Project Block (2.5 hrs)

**LLD: Vending Machine**
- Requirements to clarify: does it dispense multiple product types, or one? Cash only, or cards too? What happens on insufficient funds, or on a sold-out selection?
- Core objects: `VendingMachine` (the context), a `State` interface implemented by `HasCoinState`, `NoCoinState`, `DispensingState`, `SoldOutState`, and `Product`/`Inventory`.
- Task: implement the full State pattern — the vending machine's behavior should change automatically based on its current state object, with **no giant `if-else` or `switch` block anywhere in the core context class**. That constraint is the actual test of whether you applied the pattern correctly, not just described it.
- Definition of done: a full purchase flow (insert coin → select product → dispense → return to no-coin state) works correctly through actual state transitions, not conditionals.

### Mock Interview #1 (1 hr)
- 45-minute LLD mock with your accountability partner, using **Tic-Tac-Toe** (yesterday's system) as the subject. This is deliberately early and deliberately on an easy system — the goal isn't difficulty, it's building the muscle of narrating your design decisions out loud, under mild time pressure, while someone else is watching and can push back. Note afterward: did you clarify requirements before designing, or jump straight to code?

### Theory Block (1 hr)
- Topic: When to Prefer State vs. Strategy
- Both let you swap behavior at runtime, and the code often looks structurally similar — the distinction is *intent*. State models an object's internal lifecycle, where the object itself transitions between states (today's vending machine literally becomes a different state object as it moves through a purchase). Strategy models an *external* choice of algorithm the client makes and hands in.
- Coding exercise: none — write two sentences distinguishing the two for your own future reference; "when would you use State over Strategy" comes up often enough in LLD interviews to be worth having crisp.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: respond to recruiters.
- **Resume checkpoint — the one right before applications matter most.** The last refresh was Week 6, before any LLD system existed and before the platform had Resilience4j, Feign, the Gateway, Saga, or the Kubernetes/Helm work. Update it now to reflect everything since: the platform's real architecture, problem count, and (once built) the LLD systems. This is the resume that goes out starting next week — it needs to actually reflect where you are, not where you were 10 weeks ago.

### Daily Deliverable
- [ ] Vending Machine LLD complete, state transitions handling the full purchase flow with no conditional blocks in the context class.
- [ ] Mock Interview #1 completed and debriefed.
- [ ] State vs. Strategy distinction written down.
- [ ] Resume updated to reflect current project and problem-count reality.

---

## Day 110 — LLD #3: Parking Lot, Part 1 — Design and Core Logic

### DSA Block (1 hr)
- Revision: one Tree problem, cold.

### Project Block (3.5 hrs)

**LLD: Parking Lot**
- This is your third LLD, not your first — you now have the framework practiced twice on simpler systems, so today adds exactly one new dimension: concurrency (tomorrow).
- Requirements to clarify: multiple levels? Multiple vehicle/spot size types (motorcycle, compact, large)? Payment integration in scope, or out?
- Core objects: `ParkingLot`, `Level`, `ParkingSpot` (sized), `Vehicle` (typed), `Ticket`.
- Task: design the full class diagram and implement the single-threaded version — `assignSpot`, `removeVehicle`, spot-finding logic that respects vehicle/spot size compatibility.
- Definition of done: a vehicle can park (getting an appropriately-sized spot) and leave (freeing it), verified with unit tests, correct in a single-threaded context.

### Theory Block (1 hr)
- Topic: Composition Over Inheritance, Applied
- Refactor a hypothetical badly-designed alternative — a `CompactCarSpot extends ParkingSpot` per size, instead of a `ParkingSpot` with a `size` field and a `Vehicle` with a `size` field that get matched — and articulate in writing why composition (matching two independent size fields) beats an inheritance hierarchy exploding per size/type combination.
- Coding exercise: none — the writing above is the exercise.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: engage heavily — comment on 5 posts today given the milestone project work underway.
- **Reference cultivation, started now rather than scrambled later:** Databricks specifically weighs references heavily in the final decision, and other companies do too even when it's less explicit. Identify 2-3 former colleagues or managers who'd genuinely speak well of your work, and reconnect with a real, warm message now — not a "can you be my reference" ask, just re-establishing the relationship. The ask comes later, naturally, once it's warm again.

### Daily Deliverable
- [ ] Parking Lot single-threaded core complete and unit-tested.
- [ ] Composition-over-inheritance write-up complete.
- [ ] 2-3 potential references reconnected with.

---

## Day 111 — LLD #3: Parking Lot, Part 2 — Concurrency

### DSA Block (1 hr)
- Revision: one Union-Find or Dijkstra problem, cold — deliberately picking from the newer, less-rehearsed patterns.

### Project Block (3.5 hrs)

**LLD: Parking Lot — Thread Safety**
- Task: make `assignSpot` thread-safe. Write a test that spawns 10 threads simultaneously attempting to park, competing for a small number of remaining spots, and prove no two threads are ever assigned the same spot.
- Definition of done: the concurrency test passes reliably across multiple runs (run it at least 5 times to be confident it's not passing by luck); no double-booking under concurrent load.

### Theory Block (1 hr)
- Topic: Where to Put the Lock
- Locking the entire `ParkingLot` object serializes every park/leave operation across the whole system — correct, but a scalability bottleneck at real volume. Locking per-`Level`, or using a concurrent-safe data structure to track free spots (a `ConcurrentHashMap`, or a lock-striped structure), reduces contention while keeping correctness — the same trade-off you'll articulate again in System Design, at far larger scale.
- Coding exercise: none — this is a design discussion to have ready, not a new artifact to build today.

### Career Block (1 hr)
- LinkedIn: Post 21 — "I ran 10 threads at my Parking Lot's last spot — here's how I made sure only one won" (concrete, testable claims make strong LLD-adjacent posts).
- Networking: reflect on progress.
- **Format note:** today's shape — build the single-threaded version, then get pushed on "now make it thread-safe" — is close to exactly what Rippling's onsite does (build something, then discuss how you'd scale/harden it) and close to what a Uber follow-up would look like too. Worth noticing that the practice already matches the real format, not just the content.

### Daily Deliverable
- [ ] Concurrency test passing reliably, no double-booking.
- [ ] Locking-granularity trade-off written down.
- [ ] LinkedIn Post 21 published.

---

## Day 112 (Sunday) — LLD #4: Library Management System, and Mock Interview #2

### Self-Check (15 min)
- [ ] Recite the 5-step LLD framework from memory, unprompted.

### Project Block (2.5 hrs)

**LLD: Library Management System**
- Requirements to clarify: search by title/author/subject in scope? Reservations, or just checkout/return? Fines for late returns?
- Core objects: `Library`, `Book` (the catalog entry) vs. `BookItem` (a physical copy, since a library can hold multiple copies of one title), `Member`, `Reservation`.
- Task: implement the catalog search (by title, author, subject) cleanly separated from the reservation/checkout logic — this separation is the actual design test here, more than any single pattern.
- Definition of done: `BookItem` correctly moves through its lifecycle (`Available` → `Reserved`/`Loaned` → `Available`, or `Lost`), and catalog search logic has zero dependency on reservation logic's internals.

### Mock Interview #2 (1 hr)
- 45-minute LLD mock, using **Parking Lot** (including the concurrency question) as the subject. This time, push for the harder follow-up: "now make it thread-safe" mid-interview, exactly as a real interviewer might, rather than knowing concurrency was coming.

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post — from this week on, prioritize Rippling, Google, Databricks, Stripe, Uber, or Atlassian's own engineering blogs specifically when there's a choice, since these double as LinkedIn content material.
- **Weekly Scorecard:** Day 112, four LLD systems complete (Tic-Tac-Toe, Vending Machine, Parking Lot with concurrency, Library Management), two mock interviews done and debriefed rather than saved for the end, and the pattern lineup now includes Proxy, Composite, and Template Method beyond the original plan's coverage. Six systems remain: ATM, Elevator, Splitwise, BookMyShow, Food Delivery, Hotel Booking. Resume is current, targeted networking is underway (15-20 connections per company, in progress), and reference relationships are being warmed. Applications to the full target list begin next week.

### Daily Deliverable
- [ ] Library Management LLD complete, catalog search and reservation logic cleanly separated.
- [ ] Mock Interview #2 completed and debriefed.
- [ ] Weekly ritual and scorecard complete.
