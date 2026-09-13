# Week 17 (Revised): LLD Systems Complete — Six More Systems, Three More Mock Interviews

**What changed:** three more mock interviews land across this week (Days 114, 117, 119) instead of all being crammed into the final week of the whole plan. By the end of today's Week 17, you'll have done five LLD mocks total before HLD even begins — compared to exactly one in the original plan, positioned right before the real interviews started.

---

## Day 113 — LLD #5: ATM Machine

### DSA Block (1 hr)
- Revision: one Heap problem, cold.

### Project Block (3.5 hrs)

**LLD: ATM Machine**
- Requirements to clarify: withdrawal only, or deposits and balance checks too? PIN validation in scope? What denominations does the machine stock?
- Core objects: `ATM` (context), a `State` interface (`Idle`, `HasCard`, `CorrectPIN`, `Dispensing`), `CashDispenser`.
- Task: implement the State pattern for the ATM's lifecycle, and Chain of Responsibility for cash dispensing — a request for ₹4700 should be handled by a chain of denomination handlers (₹2000 → ₹500 → ₹100), each handling what it can and passing the remainder down the chain.
- Definition of done: the machine correctly dispenses cash using the fewest notes possible via the chain, or throws an insufficient-funds exception when it can't.

### Theory Block (1 hr)
- Topic: Chain of Responsibility, Precisely
- Each handler in the chain either handles the request (fully or partially) or passes it to the next handler in line — the caller doesn't know or care how many handlers exist, or which one(s) actually end up acting. This decouples "who is capable of handling this" from "who's asking for it to be handled."
- Coding exercise: none — today's implementation is the exercise.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: continue the targeted connection-building from this week — Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech.

### Daily Deliverable
- [ ] ATM Machine LLD complete — State pattern for lifecycle, Chain of Responsibility for cash dispensing.

---

## Day 114 — LLD #6: Elevator System, and Mock Interview #3

### DSA Block (1 hr)
- Revision: one Trie problem, cold.

### Project Block (2.5 hrs)

**LLD: Elevator System**
- Requirements to clarify: single elevator or a bank of them? Is dispatch optimization (which elevator answers a call) in scope?
- Core objects: `Elevator` (with a `State`: `Idle`, `MovingUp`, `MovingDown`, `DoorsOpen`), `ElevatorController` (the dispatcher), `Request` (floor + direction).
- Task: implement the State pattern for a single elevator's lifecycle, and a SCAN/LOOK-style dispatch algorithm — the elevator services all requests in its current direction before reversing, rather than jumping around floor by floor in request order.
- Definition of done: given a mixed sequence of up/down requests from different floors, the elevator services them in a sensible SCAN order, not naive FIFO.

### Mock Interview #3 — Machine Coding Format (1 hr)
- This one is deliberately different from Mocks #1 and #2. Uber and Atlassian both run a round that isn't LLD-with-a-diagram and isn't LeetCode — it's **Machine Coding**: given a real-world spec, produce fully working, compiling, runnable code in 60-90 minutes, with no partial credit for "the design was right." Today's version: build a simplified in-memory rate limiter (a real Uber-style prompt) from scratch, alone, timed, with your accountability partner just watching the clock and reading the prompt cold — not guiding you. The muscle being built is different from LLD's collaborative back-and-forth: it's working under silence and time pressure toward code that actually runs.
- Debrief question: did the code actually compile and run by the end, or did you run out of time with a design that was right but nothing executable? That gap is exactly what this round tests.

### Theory Block (1 hr)
- Topic: Why SCAN Beats Naive FIFO Here
- Naive FIFO (service requests in the exact order they arrived) can produce absurd physical movement — up to floor 9, then down to floor 1, then back up to floor 8. SCAN treats direction as a first-class constraint: continue in the current direction, picking up anything along the way, only reversing once nothing remains ahead in that direction. This is a real disk-scheduling algorithm, borrowed directly for elevators — worth knowing the name if it comes up.
- Coding exercise: none — this is the design rationale behind today's implementation.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: search "Hiring SDE-2 Backend" on LinkedIn and comment/DM where genuine — and specifically check for open roles at Rippling, Google, Databricks, Stripe, Uber, Atlassian, and Walmart Global Tech.

### Daily Deliverable
- [ ] Elevator System LLD complete — State pattern plus SCAN dispatch verified against a mixed request sequence.
- [ ] Mock Interview #3 (Machine Coding format) completed — working, runnable code produced under time pressure, debriefed.

---

## Day 115 — LLD #7: Splitwise

### DSA Block (1 hr)
- Revision: one Backtracking problem, cold.

### Project Block (3.5 hrs)

**LLD: Splitwise**
- Requirements to clarify: equal splits only, or exact/percentage splits too? Multiple groups, or one global ledger?
- Core objects: `User`, `Expense`, `Split` (equal/exact/percentage — a natural Strategy pattern), `Balance` (a debt graph between users).
- Task: implement the balance-minimization algorithm — net every user's overall balance first (who owes/is owed how much in total, collapsing pairwise debts), then greedily match the largest creditor with the largest debtor using two heaps, settling in the fewest possible transactions.
- Definition of done: given a small hardcoded web of transactions across 4-5 users, the algorithm correctly minimizes the total number of settlement transactions — this is the actual point of the exercise, not just tracking who-owes-whom.

### Theory Block (1 hr)
- Topic: When a "Simple" LLD Hides a Real Algorithm
- Splitwise looks like a straightforward CRUD system (expenses, users, splits) until the settlement step, which is genuinely a graph/greedy algorithm problem wearing an LLD costume — directly reusing the Greedy pattern thinking from the DSA phase. This is common enough to watch for: BookMyShow (later this week) hides a concurrency problem the same way. Ask, for every LLD: "is there a non-obvious algorithm buried in one requirement here" before assuming it's pure class design.
- Coding exercise: none — reflection on today's system.

### Career Block (1 hr)
- LinkedIn: Post 22 — "Minimizing Cash Flow: the Splitwise debt algorithm, explained" (walk through the two-heap greedy matching with a small worked example — a strong technical post).
- Networking: continue targeted connection-building; check for referral pathways at the 7 target companies before applications go live in a few days.

### Daily Deliverable
- [ ] Splitwise LLD complete — settlement algorithm correctly minimizes transaction count on a test case.
- [ ] LinkedIn Post 22 published.

---

## Day 116 — LLD #8: BookMyShow, Part 1 — Design

### DSA Block (1 hr)
- Revision: one Graph problem, cold.

### Project Block (3.5 hrs)

**LLD: BookMyShow (Ticket Booking)**
- Requirements to clarify: single theater or multi-city? Seat-level selection, or just quantity? Payment integration in scope?
- Core objects: `Show`, `Seat` (with a status), `Booking`, `Theater`, `Screen`.
- Task: design the full class diagram and implement the single-threaded booking flow — selecting seats, holding them temporarily, confirming a booking.
- Definition of done: a booking correctly moves seats from `Available` to `Held` to `Booked`, verified with unit tests in a single-threaded context.

### Theory Block (1 hr)
- Topic: Why Ticket Booking Is a Concurrency Problem in Disguise
- The entire difficulty of this system isn't the class design, which is fairly ordinary — it's that two users can attempt to book the *same seat* at the *same moment*, and only one can win. Tomorrow's session is entirely about that, deliberately isolated from today's design work the same way Parking Lot separated design from concurrency two weeks ago.
- Coding exercise: none.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: final push on target-company connections before applications go out on Day 118 — aim to have at least one real conversation (not just a connection accept) with someone at 2-3 of your target companies by then.

### Daily Deliverable
- [ ] BookMyShow single-threaded design and booking flow complete.

---

## Day 117 — LLD #8: BookMyShow, Part 2 — Concurrency, and Mock Interview #4

### DSA Block (1 hr)
- Revision: one Dynamic Programming problem, cold.

### Project Block (2.5 hrs)

**LLD: BookMyShow — Seat Locking Under Concurrency**
- Task: write a test that spawns 10 threads simultaneously attempting to book the exact same seat, and prove your locking mechanism guarantees exactly one succeeds and the other nine receive a clean failure — not a corrupted double-booking, and not a silent failure that leaves the seat in an ambiguous state.
- Consider (and be ready to discuss) a pessimistic lock (`SELECT FOR UPDATE` at the DB level, or an in-memory lock per seat) vs. an optimistic approach (a version number on the seat row, retry on conflict) — this is a real trade-off, not just a correctness checkbox, and interviewers often ask you to justify the choice, not just implement one.
- Definition of done: the concurrency test passes reliably across multiple runs; exactly one thread succeeds per seat, every time.

### Mock Interview #4 (1.5 hrs)
- 60-minute LLD mock, using **BookMyShow** (today's system, including the concurrency question) — the hardest system so far, and deliberately the one getting the longest mock. Push specifically on the pessimistic-vs-optimistic locking justification; that's the question most candidates answer weakly on the spot without having pre-formed an opinion. This maps closely to what Uber's coding rounds actually probe — justifying a concurrency choice out loud, not just implementing one.

### Theory Block (1 hr)
- Topic: Pessimistic vs. Optimistic Locking
- Pessimistic locking assumes conflict is likely and blocks other transactions immediately — safe, but throughput suffers under contention. Optimistic locking assumes conflict is rare, lets transactions proceed and only checks for conflict at commit time via a version number, retrying on failure — better throughput when contention is genuinely low, wasted retries when it isn't. High-demand seat booking (a popular movie's opening night) is exactly the scenario where this choice matters and the "obviously correct" pessimistic answer isn't automatically right at real scale.
- Coding exercise: none — this is the design discussion to have ready.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: connect with SDE-2s at companies you've applied to recently.

### Daily Deliverable
- [ ] BookMyShow concurrency test passing reliably, no double-booking.
- [ ] Pessimistic vs. optimistic locking trade-off written down, ready to discuss either way.
- [ ] Mock Interview #4 completed and debriefed.

---

## Day 118 — LLD #9: Food Delivery System, and the System Design Framework Preview

### DSA Block (1 hr)
- Revision: one String DP problem, cold.

### Project Block (2.5 hrs)

**LLD: Food Delivery System (Swiggy/Zomato-style)**
- Requirements to clarify: matching delivery partners in scope, or just order placement? Multiple restaurants per order, or one?
- Core objects: `Order`, `Restaurant`, `DeliveryPartner`, a `PartnerMatchingStrategy` interface.
- Task: implement the Strategy pattern for partner assignment — `NearestPartnerStrategy` vs. `HighestRatedPartnerStrategy` — cleanly decoupled from the order-processing logic, so a new matching strategy can be added without touching order processing at all.
- Definition of done: swapping strategies at runtime changes which partner gets assigned, with zero changes to the order-processing code path.

### Theory Block (1.5 hrs)
- Topic: The System Design Framework — Preview
- Next week, System Design begins properly. The 5-step framework: **Requirements** (functional and non-functional — read/write ratio, scale, latency needs), **Estimation** (back-of-envelope: QPS, storage, bandwidth), **High-Level Design** (the boxes-and-arrows architecture), **Detailed Design** (deep-diving 1-2 components the interviewer cares most about), **Bottlenecks** (identifying and resolving them, usually where the real signal is). This mirrors the LLD framework's shape — clarify, structure, go deep, evaluate — at a different altitude, with scale as the central concern instead of class design.
- Coding exercise: none — read through this framework once; you'll apply it for real starting next week.

### Career Block (1.5 hrs)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- **Applications launch today.** Apply to your full target list — Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech — in this same window rather than staggering them over following weeks. The reason: real negotiating leverage comes from multiple offers landing close together, not from being excellent at one company in isolation. If you apply to company #1 this week and company #4 a month from now, #1 will likely have already decided before #4 even starts. Where you have a warm connection from this week's networking, ask for a referral submission specifically instead of the cold-apply portal — referred candidates are hired at roughly 4x the rate of cold applications.

### Daily Deliverable
- [ ] Food Delivery LLD complete — Strategy pattern cleanly decoupling partner matching from order processing.
- [ ] System Design framework reviewed, ready to apply next week.
- [ ] Applications submitted to the full target company list.

---

## Day 119 (Sunday) — LLD #10: Hotel Booking System, Mock Interview #5, and LLD Complete

### Self-Check (15 min)
- [ ] Without notes, list all 10 LLD systems built this and last week, and the primary pattern each one centers on.

### Project Block (2.5 hrs)

**LLD: Hotel Booking System**
- Requirements to clarify: overbooking tolerance (hotels sometimes deliberately overbook against no-show rates — is that in scope, or strict 1:1)? Search/filter by room type and date range in scope?
- Core objects: `Hotel`, `Room` (typed), `RoomBooking`, a `Search` component.
- Task: implement the booking lifecycle cleanly, and compare — in writing — this system's concurrency needs against BookMyShow's from earlier this week: hotel inventory typically syncs asynchronously across channels (your own site, third-party OTAs) since a few minutes of staleness is tolerable, where movie ticket booking requires strict real-time locking since a seat is a much more immediately contested resource.
- Definition of done: the booking lifecycle (search → reserve → confirm) works correctly, and the concurrency-model comparison is written down.

### Mock Interview #5 (1 hr)
- 45-minute LLD mock, your choice of subject from anything built across the last two weeks — pick whichever system you personally feel least confident defending under questioning, since that's the one worth the practice most right now, not the one that would go smoothest.

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post — prioritize your target companies' own engineering blogs.
- **Weekly Scorecard, and a real milestone: all 10 LLD systems are complete, five mock interviews are done and debriefed (including one in the Machine Coding format Uber and Atlassian actually use), and applications are live at Rippling, Google, Databricks, Stripe, Uber, Atlassian, and Walmart Global Tech** — sequenced from a genuine warm-up through two dedicated concurrency-focused systems, with the interview framework taught once, properly, up front. Compare this to the original plan's single LLD mock and applications starting cold with no rehearsal behind them — this phase now has real, targeted rehearsal built in throughout. System Design begins tomorrow, in parallel with your first real interview responses starting to arrive.

### Daily Deliverable
- [ ] Hotel Booking LLD complete.
- [ ] Async-sync concurrency-model comparison against BookMyShow written down.
- [ ] Mock Interview #5 completed and debriefed.
- [ ] Weekly ritual and scorecard complete — **LLD phase closed, five mocks deep.**
