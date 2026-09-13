# Week 17 Interview Questions — Consolidated Review Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Covers:** Day 113 – Day 119 (LLD #5–#10: ATM, Elevator, Splitwise, BookMyShow, Food Delivery, Hotel Booking)

This bank pulls every interview question from all seven of this week's Resource Books into one place, in day order, for spaced review without re-opening each individual file. Each section links back to its source day for full context (code, worked traces, proofs) if an answer needs more than the one-paragraph version below to feel solid again.

---

## Day 113 — ATM Machine (State Reapplied, Chain of Responsibility) · [Full Resource Book](Day113_Resource_Book.md)

**Q1. Does `ATM` contain any conditional logic that inspects which state it's in?** No — every public method is a single-line delegation to `currentState`; the states themselves hold all the branching logic, which is the entire point of the pattern.

**Q2. Why does `ATMState` declare all four methods on every concrete state, including the illegal ones?** So an illegal call fails loudly (an explicit exception) rather than being silently swallowed — a caller invoking `enterPIN` with no card inserted has a real bug, and hiding that is worse than surfacing it immediately.

**Q3. What specifically does Chain of Responsibility decouple, and from what?** The sender (the ATM, requesting a dispense) from the handlers — the sender never knows how many denomination handlers exist or which ones actually end up dispensing anything; it only ever talks to the chain's head.

**Q4. Why is a single method with an if/else per denomination the wrong design, even though it computes the same answer?** It violates Open/Closed — adding a new denomination means editing an existing, already-tested method. Chain of Responsibility makes a new denomination a new class plus one line of wiring, touching nothing that already works.

**Q5. Walk through the exact bug in the naive `dispenseBroken` implementation.** It mutates `availableNotes` as it recurses, before knowing whether the *entire* chain will succeed. If a later handler in the chain can't cover its share, the method correctly returns `false` overall, but earlier handlers have already decremented inventory for notes that were never actually given to anyone — silent, persistent state corruption.

**Q6. How does the `canDispense`/`commitDispense` split fix that bug?** `canDispense` is read-only — it computes the exact same recursive path but never writes `availableNotes`. Only after it confirms success does `commitDispense` run, retracing the identical computation and mutating for real. Since nothing else can modify state between the two calls (single-threaded), `commitDispense` is guaranteed to succeed everywhere `canDispense` said it would.

**Q7. Under what condition would the check-then-commit split itself become unsafe?** Under concurrency — if two withdrawal requests could interleave between the `canDispense` check and the `commitDispense` act, both could pass the check against inventory that's no longer accurate by the time either commits. This is the identical check-then-act shape Day 111's Parking Lot race demonstrated; the fix would be the same, a lock held across both phases.

**Q8. Prove that always dispensing the largest possible denomination first never produces more notes than any other valid combination, for {₹2000, ₹500, ₹100}.** Each denomination is an exact multiple of the next-smaller one (₹2000 = 4×₹500 = 20×₹100). Any solution using fewer large notes than greedy can have some combination of its smaller notes swapped for one larger note of equal value with no change in total value paid, strictly reducing note count — so greedy can never be beaten, only matched.

**Q9. Why does that same greedy approach fail for Coin Change (LC 322)?** Coin Change's denominations are arbitrary and not guaranteed to divide evenly into each other — the counterexample {1,3,4}, target 6: greedy gives 4+1+1 (3 coins), optimal is 3+3 (2 coins). Without the divides-evenly property, there's no guaranteed swap, so only exploring the actual subproblem space (DP) is correct in general.

**Q10. In K Closest Points to Origin, why a max-heap rather than a min-heap?** The heap needs to answer "what's currently the worst of my k best candidates, so I can evict it?" — that's the max-heap's root. A min-heap would need every point inserted and k pops from the front, O(n log n) instead of the bounded max-heap's O(n log k).

**Q11. Why compare squared distances instead of actual distances in K Closest Points?** Squaring preserves ordering for non-negative distances while avoiding a `Math.sqrt()` call entirely — cheaper, and sidesteps floating-point precision issues a square root could introduce.

---

## Day 114 — Elevator System (SCAN/LOOK), Mock #3 (Machine Coding) · [Full Resource Book](Day114_Resource_Book.md)

**Q1. What changed about State's usage in the Elevator versus the ATM?** The mechanism is identical — a context delegating to an interface reference, states driving their own transitions — but the Elevator's states are also driven by internal progress (`step()`, arriving at a floor), not purely external events like the ATM's card/PIN input.

**Q2. State the precise difference between SCAN and LOOK.** SCAN travels all the way to the physical boundary (top or bottom floor) before reversing, even with nothing pending there. LOOK reverses as soon as nothing remains pending ahead in the current direction, without traveling to the boundary. Real elevators implement LOOK.

**Q3. Why does `DoorsOpenState.step()` check `getLastDirection()` before deciding where to go next, instead of always checking "up" first?** Checking a fixed direction first would ignore which way the elevator was actually already headed, causing unnecessary direction reversals — genuinely different (worse) behavior than LOOK, even though it would still eventually serve every request correctly.

**Q4. Why is `TreeSet.higher()`/`lower()` the right tool for finding the next stop, instead of iterating over the pending requests?** They run in O(log n) against the tree structure with no manual scanning, and are the same navigable-structure family already used for `TreeMap.ceilingKey()` in Consistent Hashing (Week 9) — a different problem, an identical query shape.

**Q5. What does a Machine Coding interview evaluate that an LLD interview doesn't?** Whether the code actually compiles and runs correctly against the spec, evaluated largely after the candidate works in near-silence — design discussion barely factors in, versus an LLD interview where the real-time trade-off conversation is a large part of the signal.

**Q6. Why can the "think deeply before coding" instinct that serves LLD interviews well actively hurt in Machine Coding?** Spending too much time perfecting a design before writing any code that runs risks a rushed, broken implementation at the end — and an unseen design scores nothing if the code doesn't execute. The discipline that wins here is a compiling skeleton first, incremental implementation second.

**Q7. State Token Bucket's two guarantees precisely.** Bounded burst — never more than `capacity` tokens available at once, regardless of idle time beforehand. Bounded average rate — sustained throughput can't exceed `refillRatePerSecond` long-run, since tokens only arrive that fast.

**Q8. Why does a naive fixed-window counter fail to provide both guarantees simultaneously?** A hard window boundary lets a burst at the end of one window and a burst at the start of the next both succeed, allowing up to roughly `2×capacity` requests in a short span straddling the reset — continuous refill has no boundary to exploit that way.

**Q9. Is `synchronized` on `allowRequest()` a performance optimization or a correctness requirement?** Correctness. Without it, two threads can both read `availableTokens` before either writes back the decrement, both see enough tokens, and both proceed — the same race shape as Week 6's Counter bug, just applied to a different field.

**Q10. In Design Add and Search Words, what specifically happens differently on a `.` versus a normal character?** A normal character descends into exactly one child, deterministically. A `.` branches into all 26 possible children, returning true the instant any one of them leads to a successful match on the rest of the query.

**Q11. What's the worst-case complexity of `search` with wildcards, and why is it usually much better in practice?** O(26^L) worst case, when the query is all wildcards, since every level branches fully with no way to prune. In practice, actual branching at any node is bounded by however many real words share that prefix — almost always far less than 26 — so realistic performance is much closer to O(L).

---

## Day 115 — Splitwise (Strategy Applied For Real, Dual-Heap Greedy Settlement) · [Full Resource Book](Day115_Resource_Book.md)

**Q1. What makes Splitwise's split logic a genuine Strategy application rather than a superficial one?** The client (whoever creates the `Expense`) chooses which concrete strategy to use, and two of the three strategies carry their own constructor-configured data (exact amounts, percentages) — the identical shape as Day 107's `PercentageDiscount(0.10)`, just with real financial consequences.

**Q2. Why does the sum-validation for exact amounts and percentages live inside each strategy rather than in `Expense`?** Each strategy alone knows what "valid input" means for its own algorithm. Pushing validation up into `Expense` would force it to understand every strategy's internal rules, defeating the interface boundary's purpose.

**Q3. Why net balances instead of settling the pairwise ledger directly?** A pairwise ledger can have redundant opposing debts (Bob owes Alice ₹1000 while Alice owes Bob ₹500 from a different expense) that settle more efficiently as one payment once netted — settling every pairwise entry separately wastes transactions.

**Q4. How is the dual-heap technique here different from Week 9's Two Heaps (running median)?** Two Heaps splits one dataset into two *balanced*, size-tracked halves for O(1) median access. This uses two entirely *independent* max-heaps — creditors and debtors — with no balancing relationship between their sizes at all. Structurally different roles that happen to share the word "two heaps."

**Q5. Prove the algorithm never needs more than (n − 1) transactions.** Every transaction settles `min(creditor, debtor)`, which fully zeroes out whichever side was smaller — so every transaction reduces the count of people with nonzero balance by at least one. Since all balances sum to zero, the last two remaining must exactly cancel, so the final transaction always retires two at once: at most (n − 2) single-retirement transactions plus one double-retirement transaction gives (n − 2) + 1 = n − 1.

**Q6. Is (n − 1) always the true theoretical minimum number of transactions?** No — when balances split into separable zero-summing sub-groups, the true minimum can be lower, since each sub-group settles independently. Finding that true minimum in general is LeetCode 465 (Optimal Account Balancing), NP-hard, solved via backtracking — a fundamentally different, harder problem than this greedy approach was built to solve.

**Q7. Why use integer minor-units or `BigDecimal` instead of `double` for money in a real system?** `double` accumulates floating-point rounding error across enough operations, which surfaces as impossible-looking residual balances in production. `double` is used here only to keep the algorithm's logic visible without `BigDecimal`'s verbosity.

**Q8. In Generate Parentheses, why does the close-paren branch check `close < open` rather than `close <= open`?** Allowing closes to equal or exceed opens would let a `)` appear before a matching `(` somewhere earlier in the string — an invalid prefix. `close < open` is the entire correctness guarantee for every prefix, not just the finished string.

**Q9. What does `deleteCharAt` accomplish in the backtracking recursion?** It's the "undo" step — after a branch has been fully explored, it restores `current` to exactly its state before that branch was tried, so the next branch (if any) starts clean rather than carrying over the previous attempt's characters.

**Q10. If asked to count valid parenthesis sequences instead of generating them, what's the fast answer?** The nth Catalan number, `C(n) = (2n)! / ((n+1)! · n!)` — a closed form, requiring no backtracking tree to actually be built.

---

## Day 116 — BookMyShow, Part 1 (Design, and the Race Traced) · [Full Resource Book](Day116_Resource_Book.md)

**Q1. Why can't `SeatStatus` live directly on `Seat`?** The same physical seat is reused across every show a screen hosts in a day — booking it for one showtime must not affect its availability for a different showtime on the same screen. Status has to be scoped per-`Show`, tracked in a map keyed by seat ID, not stored as a field on the seat itself.

**Q2. Does the single-threaded `BookingService` above have any logic bugs?** No — every method is correct for one caller at a time, and passes every single-threaded test that could be written against it. The problem isn't a logic bug; it's a gap between reads and writes that isn't visible from any single-threaded test at all.

**Q3. Trace the exact interleaving that lets two users successfully hold the same seat.** Both threads' availability checks (`getSeatStatus != AVAILABLE`) can read `AVAILABLE` before either thread's write (`setSeatStatus(HELD)`) has happened — so both checks pass, and both writes then succeed, each overwriting the other with no error raised.

**Q4. Name the two other races this week that share this exact shape.** Parking Lot's `findAvailableSpot()`/`assignVehicle()` split (Day 111) and the ATM's `canDispense`/`commitDispense` split (Day 113) — both read state, then separately write to it, with nothing holding the gap between the two operations shut.

**Q5. Why is "just add `synchronized` to the whole method" an incomplete answer, even though it would technically work?** It would serialize every booking attempt across the *entire show*, not just the specific seats actually in contention — a real throughput cost at scale. The stronger answer names that only the seats actually involved need locking, echoing the lock-granularity conversation Day 111 already had about parking spots.

**Q6. In Clone Graph, why must the clone be registered in `visited` before recursing into its neighbors, not after?** A cyclic graph can lead back to the original node before the neighbor loop finishes. If the clone weren't registered yet, that cycle would trigger infinite recursion instead of finding the already-created clone and stopping.

**Q7. What's the time and space complexity of Clone Graph, and why?** O(V + E) time — every node and every edge is visited exactly once, guarded by the visited map. O(V) space for the map, plus O(V) worst-case recursion depth on a graph shaped like a long path.

---

## Day 117 — BookMyShow, Part 2 (Concurrency Fixed), Mock #4 · [Full Resource Book](Day117_Resource_Book.md)

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

## Day 118 — Food Delivery (Strategy, Second Application), System Design Preview · [Full Resource Book](Day118_Resource_Book.md)

**Q1. Why is `NearestPartnerStrategy` still genuinely Strategy despite carrying no constructor configuration, unlike Splitwise's split strategies?** The definitional test (Day 109) is who chooses which implementation runs — the client, here whoever constructs `DeliveryAssignmentService` — not whether the implementation happens to carry constructor data. Constructor config is a common tell, not the definition; both config-carrying and stateless implementations can be equally valid Strategy usages.

**Q2. What capability does `setStrategy()` demonstrate that Splitwise's usage never did?** Runtime swapping — Splitwise fixed one strategy per `Expense` for that expense's entire lifetime, while Food Delivery can change its matching approach mid-run with zero changes to any calling code.

**Q3. Why would replacing this Strategy design with an `enum` and a `switch` be a mistake, even though it would work with exactly two options today?** It would violate Open/Closed the moment a third matching approach is needed — adding it would mean editing the existing switch statement, exactly the kind of modification-of-working-code Strategy (and Chain of Responsibility, Day 113) exist to avoid.

**Q4. Name the System Design framework's five steps, in order.** Requirements, Estimation, High-Level Design, Detailed Design, Bottlenecks and Trade-offs.

**Q5. What is the "Estimation" step actually for?** Establishing the real scale being designed for — expected request volume, storage growth, bandwidth — since a system built for a thousand users and one built for a hundred million require genuinely different designs, and every later decision depends on getting this roughly right first.

**Q6. In Edit Distance, why does the "characters match" case cost zero additional operations?** Because the cost of aligning everything before these two matching characters is already captured in `dp[i-1][j-1]` — a matching character requires no insert, delete, or replace, so the answer simply carries forward unchanged.

**Q7. What are the three options considered when characters don't match, and what does each correspond to?** Replace (`dp[i-1][j-1]`, both prefixes advance together), delete from `word1` (`dp[i-1][j]`, only `word1`'s prefix advances), and insert into `word1` (`dp[i][j-1]`, only `word2`'s prefix advances) — the minimum of the three, plus one operation, is the answer for that cell.

**Q8. How does Edit Distance relate to Longest Common Subsequence?** Same 2D table shape and base structure — LCS asks how much can be kept with zero edits allowed; Edit Distance asks the minimum cost to force a full match, which needs insert and delete options LCS doesn't have, plus a replace option LCS's problem statement doesn't permit at all.

---

## Day 119 — Hotel Booking, Mock #5, LLD Complete · [Full Resource Book](Day119_Resource_Book.md)

**Q1. Of the ten LLD systems built across Weeks 16–17, how many chose a GoF pattern as their headline decision, and what does that split signal?** Four (Vending Machine, ATM, Elevator, and Strategy across Splitwise/Food Delivery) — the other six deliberately didn't, either because no pattern genuinely fit or because the real problem was elsewhere (BookMyShow's concurrency). Recognizing when *not* to reach for a pattern was as much the point of this phase as recognizing when to.

**Q2. Why can't room availability be a boolean field on `Room`?** A room is available for some date ranges and unavailable for others simultaneously — a room booked next week is still available today. Availability must be checked against existing bookings for a specific date range, not stored as one global flag — the same underlying mistake as putting seat status directly on `Seat` (Day 116), on a different axis.

**Q3. Why is `checkOut` treated as exclusive in the overlap check?** A guest checking out on a given day is gone by check-in time, so a new guest checking in that same day doesn't conflict with them — standard same-day hotel turnover. An inclusive `checkOut` would incorrectly block that, treating standard practice as a conflict.

**Q4. Is the `overlaps()` check genuinely new logic?** No — it's the identical interval-intersection test from Week 4's merge-intervals-style problems, applied to date ranges instead of a generic array of intervals.

**Q5. Why does BookMyShow need strict, real-time locking while Hotel Booking can tolerate eventual consistency?** BookMyShow has high contention concentrated on one show's small seat inventory, in a tight time window, with users expecting immediate confirmation, and no cheap way to fix an oversold seat before showtime. Hotel Booking sells the same rooms across many independently-operated channels that can't easily share a real-time lock, on a much longer booking horizon, with a real operational fallback (room upgrade, guest relocation) for the rare oversold case.

**Q6. Is "which system needs strict locking" a question with one universally correct answer?** No — it depends on how concentrated contention is, how much latency users will tolerate, how many independent parties need to coordinate, and how expensive the failure mode is to absorb operationally. BookMyShow and Hotel Booking land on different, both-defensible points on that same trade-off.

**Q7. Why did this week's six DSA blocks add zero new problems to the curriculum?** They were spaced-repetition revision of patterns already fully mastered with extensive original and extra practice weeks earlier — a single cold retrieval check serves that purpose; padding already-reflexive patterns with more reps would spend the plan's 1-hour daily time-box on repetition that wasn't needed, not genuine new learning.

---

## Index — All 65 Questions by Topic

| Topic | Where |
|---|---|
| State pattern (reapplied) | D113 Q1–Q2, D114 Q1, Q3 |
| Chain of Responsibility | D113 Q3–Q7 |
| Greedy exchange arguments / Coin Change contrast | D113 Q8–Q9 |
| Heap (K Closest Points) | D113 Q10–Q11 |
| SCAN/LOOK dispatch | D114 Q2, Q4 |
| Machine Coding interview format | D114 Q5–Q6 |
| Token Bucket rate limiter | D114 Q7–Q9 |
| Trie with wildcards | D114 Q10–Q11 |
| Strategy (first + second application) | D115 Q1–Q2, D118 Q1–Q3 |
| Balance netting | D115 Q3 |
| Dual-heap greedy settlement + proof + honest limit | D115 Q4–Q6 |
| Money-as-double pitfall | D115 Q7 |
| Backtracking (Generate Parentheses) | D115 Q8–Q10 |
| Per-show seat scoping / the race, traced | D116 Q1–Q4 |
| Lock granularity | D116 Q5 |
| Clone Graph | D116 Q6–Q7 |
| Pessimistic locking (in-memory + DB) | D117 Q1–Q2 |
| Optimistic locking + CAS + ABA | D117 Q3–Q4 |
| CountDownLatch | D117 Q5–Q6 |
| Pessimistic vs. optimistic trade-off | D117 Q7–Q9 |
| Longest Increasing Subsequence | D117 Q10–Q11 |
| System Design framework (preview) | D118 Q4–Q5 |
| Edit Distance | D118 Q6–Q8 |
| All 10 LLD systems | D119 Q1 |
| Date-range availability / interval overlap | D119 Q2–Q4 |
| BookMyShow vs. Hotel Booking concurrency | D119 Q5–Q6 |
| Why zero new DSA problems this week | D119 Q7 |
