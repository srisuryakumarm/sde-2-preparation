# SDE-2 Resource Book Series — Curriculum Map & Cross-Week Handoff (Through Week 21 — Series Complete)

> **Flag, not a silent fix:** this title and the companion line below were frozen at "Through Week 16" across two subsequent extensions (Weeks 17 and 18 both appended content below without updating this pointer) — corrected here as part of the Week 19 extension. The "Role of this document" paragraph immediately below still cites Week 16-era specifics (resource-book and interview-bank counts, a Week 17 example) that were accurate when written and were not kept current through those same two extensions; current counts live in the File Index addendum notes further down rather than restated here, consistent with this document's own extend-don't-edit convention for narrative content.

**Companion to:** `Week_01_Revised.md` through `Week_21_Revised.md` (your day-wise task plans) — Week 21 is the series' final week; there is no `Week_22_Revised.md`.

**Role of this document:** this file does two jobs at once. It's the human-readable index for the whole series — what was taught, in what order, and where. It's *also* the complete handoff a future week's generation needs to build on everything so far correctly, without re-uploading any of the one hundred and twelve Resource Books or any of the sixteen interview question banks. If you're generating Week 17 (or any later week), this file plus that week's plan file is the full input — nothing else from Weeks 1–16 is needed. **Week 15 closed the entire DSA phase of this series** — everything from Week 16 onward is LLD/HLD systems, mock interviews, and behavioral prep, built on top of the DSA foundation this file indexes in full. **Week 16 was the phase's first week**, four complete LLD systems and ten design patterns deep, with zero new DSA problems added — see the new LLD System Inventory below, tracked separately from the DSA problem table it deliberately does not extend.

**Update log:** originally built after Week 1 (7 days, 28 problems). Extended after Week 2 (7 more days, Two Pointers closed at 16 required problems, Sliding Window opened). Extended after Week 3 (7 more days, Sliding Window closed at 14 required + 4 extra = 18 total; Prefix Sum & Kadane's opened at 3/7). Extended after Week 4 (7 more days, Prefix Sum & Kadane's closed at 9 distinct, Greedy & Intervals opened *and* closed in the same week at 13 distinct, Binary Search opened at 2/11). Extended after Week 5 (7 more days, Binary Search closed at 11 required + 4 extra = 15 distinct across Weeks 4–5, Linked Lists opened at 4/11 required + 2 extra). Extended after Week 6 (7 more days, Linked Lists closed at 11 required + 4 extra = 15 distinct across Weeks 5–6, Stacks/Monotonic Stack opened at 7/12 required — including 2 recapped from Week 1 — + 2 extra). Extended after Week 7 (7 more days, Stacks/Monotonic Stack closed at 12 required + 2 extra = 14 distinct across Weeks 1/6/7, Trees opened at 8/15 required + 3 extra = 11 distinct). Extended after Week 8 (7 more days, Trees closed at 15 required + 5 extra = 20 distinct across Weeks 7–8 — including the Day 21-promised Median of Two Sorted Arrays, finally delivered — Heaps opened at 6/10 required + 3 extra = 9 distinct). Extended after Week 9 (7 more days, Heaps closed at 10 required + 3 extra = 13 distinct across Weeks 8–9, Tries opened *and* closed its own core at 6/6 required + 0 extra — the first pattern in the series to open and close within the same week without ever receiving extra practice — Backtracking opened at 4/12 required + 0 extra). Extended after Week 10 (7 more days, Backtracking closed at 12 required + 0 extra = 12 distinct across Weeks 9–10 — the first pattern in the series to run its entire opening-through-close arc, seven days, without ever receiving extra practice — Graphs BFS/DFS opened at 6/12 required + 2 extra = 8 distinct, the first pattern requiring an explicit `visited` structure and the first to demonstrate that requirement is conditional, not absolute). Extended after Week 11 (7 more days, Graphs BFS/DFS closed at 12 required + 2 extra (Week 10) = 14 distinct across Weeks 10–11, the second-thinnest gap from the original audit fixed; Union-Find opened *and* closed within the same week at 7 required + 1 extra = 8 distinct — the first-thinnest gap from the original audit, now also fixed; Minimum Spanning Trees taught as a direct payoff, reusing the `UnionFind` class as a live subroutine the same day the pattern closed, rather than as a new structure). Extended after Week 12 (7 more days, Dijkstra's Algorithm opened *and* closed within the same week at 5 required + 0 extra = 5 distinct — the required ladder itself was expanded from 3 to 5 by this week's own plan revision specifically to close a known gap, and correctly received no extra practice on top of that revision, on its opening day or any day after; Dynamic Programming opened at 8/33 required — including 1 recap (LC 152, originally Week 4, Day 22) — + 1 extra = 8 distinct, continues into Week 13. Week 12 also caught its own required-problem overlap for the first time in the series: a plan-required problem (Maximum Product Subarray, Day 83) turned out to already be solved as an earlier week's extra practice, exactly the failure mode the overlap-check system exists to catch, on the *opposite* side from every prior catch — see the Week 12 Overlap section below). Extended after Week 13 (7 more days, all four of Dynamic Programming's remaining subtypes closed entirely within the week: 1D DP closed at 14/14 required — up from the original plan's 12 — + 2 extra, Grid DP opened *and* closed within a single day (Day 87) at 4/4 required + 1 extra, String DP opened and closed across three days at 9/9 required — Regular Expression Matching upgraded from optional extension to fully scheduled, per the week's own plan revision — + 3 extra, and Interval DP opened *and* closed within a single day (Day 91) at 2/2 required + 1 extra, the pattern's thinnest required ladder in the series, deliberately reinforced with a second, structurally distinct recurrence shape rather than left at one; 21 required + 6 extra = 27 new problems this week, zero recaps needed in either direction — the first Dynamic Programming week in the series to close that way; two genuinely new sub-concepts were built fully from zero rather than derived by analogy, 0/1 Knapsack (Day 85) and Interval DP's length-ordered table-fill requirement (Day 91); Dynamic Programming now stands at 28/35 required across Weeks 12–13, with only State Machine DP and Tree DP remaining for Week 14). Extended after Week 14 (7 more days, State Machine DP opened *and* closed within the week at 4/4 required + 0 extra, the required ladder already spanning every shape the pattern tests in interviews — unconstrained-with-fee, a structural coupling constraint, a small fixed transaction budget, and an arbitrary budget with its own reduction case — correctly receiving no extra practice on its opening day (Day 92) or the day after (Day 93), the reasoning stated explicitly rather than left a silent zero, exactly Dijkstra's Algorithm's Week 12 treatment repeated; one deliberate reordering flagged where it happened — Transaction Fee taught before Cooldown on Day 92, reversed from the plan's own stated sequence, so the state count escalates from the pattern's simplest case outward; Tree DP opened *and* closed within a single day (Day 94), joining Grid DP's and Interval DP's Week 13 precedent for single-day patterns, at 2/2 required + 1 extra (Binary Tree Cameras, placed on that same day and marked extension material) — one old problem, Diameter of Binary Tree (Week 7, Day 48), correctly resurfaced as a citation rather than re-taught; Dynamic Programming now closes entirely at 35/35 required across Weeks 12–14 (34 newly taught + 1 recap, LC 152, fully reconciled on Day 94); Bit Manipulation opened at Day 95, correctly receiving no extra practice that day either — the eighth consecutive genuinely-new top-level pattern in the series to receive that treatment — and reached 8/10 required + 1 extra (Hamming Distance) by week's end, that extra landing on Day 97, two days past opening rather than the usual one, since Day 96 was judged to already carry sufficient new material (a DP/Bit-Manipulation fusion problem) without adding practice on top; two required problems (Reverse Bits, Maximum XOR of Two Numbers in an Array) deferred to Week 15 to close the pattern at 10, per the plan's own stated target; zero recaps needed in either direction this week, the second consecutive week to close that way; 14 required + 2 extra = 16 new problems, bringing the series to 245 distinct problems across Weeks 1–14). Extended after Week 15 (7 more days, Bit Manipulation closed entirely at 10/10 required — 11 distinct total including Week 14's 1 extra — its two final required problems, Reverse Bits and Maximum XOR of Two Numbers in an Array, arriving exactly as deferred from Week 14; Maximum XOR simultaneously closed Tries at 7/7 required, 0 extra, 7 distinct total, the first problem in the series built as a genuine two-pattern fusion — a Bit Trie, combining Week 14's bit mechanics with Week 9's Trie structure — rather than belonging cleanly to one pattern; Segment Trees opened *and* closed within the week (Days 100–101) at 2/2 required + 0 extra, the plan's own explicitly-stated light-touch scope for a rare-but-real topic; Sorting algorithms (merge sort, quicksort) were built from scratch for the first time in the series, closing a gap `Arrays.sort()`/`Collections.sort()` usage had left open since Week 1's Group Anagrams, with Quickselect delivered in full depth against a recapped problem (Kth Largest Element in an Array, LC 215, originally Week 8 Day 55 via min-heap); a 10-problem SQL practice track (Days 103–104) was added entirely outside the original plan's scope, extending Week 6's SQL fundamentals with subqueries, correlated subqueries, GROUP BY/HAVING, two structurally distinct self-join shapes, and the full window-function toolkit (ROW_NUMBER/RANK/DENSE_RANK, LAG/LEAD, PARTITION BY, CASE expressions and conditional aggregation); Design Patterns opened as an entirely new non-DSA track, all three Creational patterns (Singleton, Factory Method, Builder) taught and implemented in full with unit tests, including `volatile` and the Java Memory Model reordering bug Double-Checked Locking needs it to fix — the series' first genuinely new concurrency primitive since Week 6's `synchronized`/`ReentrantLock`; Kubernetes was introduced from zero (Pod, ReplicaSet, Deployment, Service, the declarative reconcile-loop model) as a real, explicitly-flagged prerequisite gap ahead of the Horizontal Pod Autoscaler the plan's own Day 99 assumed as background; zero extra practice was added anywhere this week, across every pattern touched, each zero reasoned through and stated explicitly rather than defaulted to silently; this week's own DSA-required count — 197, by this map's row-by-row table — diverges from the plan's own stated "203," flagged here as the same class of running-total drift this map has caught and noted before, rather than silently propagated; **the entire DSA phase of the series closes as of Day 105**, at 197 required + 53 extra = 249 distinct DSA problems across Weeks 1–15, plus the 10-problem SQL track, 259 distinct problems in total). Extended after Week 16 (7 more days, Days 106–112, **the series' first non-DSA-pattern week**: Structural patterns (Adapter, Decorator, Facade, Proxy, Composite) and Behavioral patterns (Observer, Strategy, State, Command, Template Method) taught at full depth across Days 106–107, the GoF 3-category taxonomy — Creational/Structural/Behavioral — named explicitly for the first time, building directly on Week 15's three Creational patterns; the 5-step LLD interview framework introduced Day 106 and then applied live, end-to-end, against four complete systems — Tic-Tac-Toe (Day 108, correctly requiring no pattern, the framework's first live run), Vending Machine (Day 109, State genuinely earned, contrasted against Strategy the same day), Parking Lot (Days 110–111, single-threaded then made thread-safe via per-`ParkingSpot` locking, also correctly requiring no pattern), and Library Management System (Day 112, a data-driven transition table deliberately preferred over a full State implementation, the choice justified directly against Day 109's genuine State usage rather than defaulted to); TDD's Red-Green-Refactor cycle (Day 107), Coupling/Cohesion/the Law of Demeter (Day 108), Composition over Inheritance (Day 110), and lock-granularity trade-offs (Day 111, whole-lot vs. per-level vs. per-spot, generalized to `ConcurrentHashMap`'s own lock striping from Week 6) each taught as a dedicated theory block, three of the four applied retroactively to that same day's own freshly-written code rather than against a separate illustrative example; two mock interviews run and debriefed (Day 109 against Tic-Tac-Toe, Day 112 against Parking Lot with a forced "now make it thread-safe" mid-interview escalation); six DSA revision problems — Course Schedule (LC 207), Coin Change (LC 322), Combination Sum (LC 39), Longest Substring Without Repeating Characters (LC 3), Validate Binary Search Tree (LC 98), and Redundant Connection (LC 684), one each from Graph, DP, Backtracking, Sliding Window, Tree, and Union-Find — solved cold as spaced-repetition checks, newly selected for this purpose and logged in the new DSA Revision Log below rather than added to the required/extra problem table, since re-solving an already-counted problem is a different activity from solving a new one; **zero new required or extra DSA problems added this week**, the DSA phase's own closing totals (197 required + 53 extra = 249 distinct, 259 including SQL) consequently unchanged; a new LLD System / Design Exercise Inventory table opened below, alongside the existing DSA problem table rather than folded into DSA-phase conventions built for a different kind of content — the open design question this map flagged ahead of Week 16 resolved this way, the reasoning stated explicitly in the Day 106 Resource Book rather than decided silently; one imprecision in this map's own prior Week 16 preview corrected here rather than silently propagated — the "Known Overlap With Week 16" section below states the revision blocks fall on "Days 108–109," but `Week_16_Revised.md` itself places them across five days, 107 through 111, not two; flagged in a new note immediately after that section, not edited in place, per this map's own extend-don't-edit rule). Extended after Week 17 (7 more days, Days 113–119, **closing the LLD phase at ten systems total**: State reapplied twice more without re-derivation (ATM, Day 113; Elevator, Day 114, the latter also driven by internal progress rather than purely external events); Chain of Responsibility taught as this series' eleventh pattern (Day 113), a naive mutate-as-you-go dispenser deliberately shown broken before being fixed via a canDispense/commitDispense read-then-write split, largest-first dispensing proven safe by an exchange argument and proven unsafe in general via a worked {1,3,4}-target-6 counterexample, connected directly to why Coin Change needed DP instead of greedy; SCAN vs. LOOK distinguished precisely and LOOK implemented with genuine direction persistence, proven more than 2x more efficient than naive FIFO on a worked trace (Day 114); Mock Interview #3 introducing an entirely new format, Machine Coding, with a from-scratch Token Bucket rebuild as its task, recapped rather than re-taught from Week 10, Day 68; Strategy moving from taught-but-unapplied to two full system-level applications — Splitwise's constructor-configured split strategies (Day 115) and Food Delivery's stateless partner-matching strategies (Day 118), the second deliberately chosen to sharpen Day 109's own State-vs-Strategy test toward its actual definitional signal (who chooses) rather than its common but non-defining tell (constructor configuration); Splitwise's dual-heap greedy settlement algorithm taught as a genuinely new, seventh heap role, proven to guarantee at most (n−1) transactions via an exchange-style counting argument, paired with an explicit, honest limit — that guarantee is not always the true theoretical minimum, which is LeetCode 465, Optimal Account Balancing, NP-hard, out of scope; BookMyShow split deliberately across two days, Day 116 building a single-threaded booking flow that passes every single-threaded test before its exact check-then-act race is traced concretely in the theory block and named explicitly as the same shape Parking Lot's race (Week 16, Day 111) and the ATM's canDispense/commitDispense assumption (Day 113) already demonstrated, the fix deliberately deferred; Day 117 closing that race two ways, pessimistic (a direct in-memory extension of Day 111's per-object locking, plus a genuinely new database-level SELECT FOR UPDATE variant motivated specifically by multi-process deployment) and optimistic (a new name for the same CAS mechanism already used by AtomicInteger, now guarding a version column, with the ABA problem named and ruled out by the counter's monotonicity), both proven deterministic via a CountDownLatch-gated ten-thread test — the series' one genuinely new concurrency primitive this week, introduced solely as a start gate, with the completion wait deliberately left on Day 111's already-established .join() rather than replaced; Mock Interview #4 run against BookMyShow with concurrency pushed on specifically; the System Design framework's five steps previewed at deliberately shallow depth (Day 118), explicitly not taught ahead of Week 18's own planned full treatment; Hotel Booking closing the LLD phase at ten systems (Day 119) with no new pattern, its date-range availability modeling citing Week 4's interval-overlap logic directly and its concurrency model compared against BookMyShow's via Week 9, Day 61's synchronous-vs-asynchronous replication trade-off, grounded in concrete factors — contention concentration, latency tolerance, number of independent coordinating parties, cost of the failure mode — rather than an abstract restatement; a Day 119 self-check opening the day confirming all ten LLD systems and their patterns nameable cold, the four-of-ten pattern-adoption ratio itself flagged as a deliberate lesson; Mock Interview #5 run as the phase's fifth and final mock, candidate's choice of subject; six DSA revision problems — K Closest Points to Origin (LC 973), Design Add and Search Words Data Structure (LC 211), Generate Parentheses (LC 22), Clone Graph (LC 133), Longest Increasing Subsequence (LC 300), and Edit Distance (LC 72) — solved cold, the Backtracking/Graph/DP picks (LC 22/133/300) explicitly confirmed distinct from Week 16's three (LC 39/207/322) per the constraint that week's own extension flagged in advance; **zero new required or extra DSA problems added this week**, deliberately, all six blocks being spaced-repetition revision rather than new-pattern acquisition, the reasoning stated explicitly rather than left a silent zero; the LLD System / Design Exercise Inventory extended with Week 17's six systems (BookMyShow counted once across its two days), closing the LLD phase at ten systems / seven deliverables total across Weeks 16–17; a new discrepancy found in the prior week's own closing paragraph — "What Week 16 Leaves You Knowing Cold" mislabels Food Delivery as Day 117 and Hotel Booking as Day 118, when `Week_17_Revised.md` itself places them on Days 118 and 119 respectively — flagged in a new note within the new "How to Use This Document for Week 18" section rather than edited in place, the same treatment the prior "Days 108–109" imprecision received during the Week 16 extension). Extended after Week 19 (7 more days, Days 127–133, **closing the HLD phase entirely at sixteen systems total across Weeks 18–19**: WhatsApp's connection-management layer as the series' first genuinely stateful-connection design (Day 127); geospatial indexing — geohashing, quadtrees, Redis Geo — defended under Uber-format mock pressure (Day 128, HLD Mock #3), including a correctly-scoped CAP-theorem anecdote; Netflix and Instagram covered in one day, the latter finally resolving "the celebrity problem" named three appearances ago on Day 78 (Day 129); idempotency keys and SETNX-based distributed locks given their full, first-time treatment — including the unsafe-lock-release bug traced step by step — closing the thread deliberately left open since Day 121's Lua-atomicity pattern, defended a second time under deliberate race-condition pressure in HLD Mock #4 (Days 130–131); at-least-once execution and SQS Visibility Timeout recognized as the identical TTL-lease shape one layer down from Day 130's own lock (Day 131); the Skip List built fully from scratch after three uses of Sorted Sets as a black box (Days 123, 128, 131), and Redis Cluster precisely distinguished from Consistent Hashing rather than conflated with it (Day 132); and content-addressable deduplication, content-defined-vs-fixed-size chunking (proven, not asserted, via a worked byte-shift example), sync-conflict handling, and HLD Mock #5 — candidate's choice, matching the LLD phase's own Day 119 precedent exactly — closing the phase (Day 133); five HLD mocks across Weeks 18–19 in total, an exact match for the LLD phase's own five and four more than the original plan's single HLD mock; two DSA cold-revision problems (LC 721 Accounts Merge, LC 1143 Longest Common Subsequence), both confirmed absent from every prior revision log before selection, continuing Weeks 16–18's zero-new-required/extra-DSA-problem precedent; no extra HLD systems added beyond the plan's own nine, the same "extra practice doesn't transfer from a LeetCode pattern to a system-design system" reasoning Day 106 gave for the LLD phase, reapplied rather than re-derived; this map's own title and companion-line pointers corrected from a two-extension-stale "Through Week 16" to "Through Week 19," flagged in a new note rather than silently fixed) — nothing from the Week 1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16/17/18 version below was removed or shortened, only appended to. Extended after Week 20 (7 more days, Days 134–140, **the series shifts from designing systems to operating one — zero new required or extra DSA problems, zero new LLD/HLD systems, confirmed directly against `Week_20_Revised.md`'s full content**: Day 134 opens with real Kubernetes Deployment/Service manifests for all four of `scalable-ecommerce-platform`'s modules plus a k6 load-testing baseline (50 VUs, ≈50 req/s, p95 ≈62ms), both citing Week 15 Day 99's from-zero Kubernetes theory and Week 7 Days 43–44's Docker/Compose work rather than re-deriving either; Day 135 templates those exact manifests into a Helm chart, with Kubernetes Ingress explicitly disambiguated from Week 10 Day 66's Spring Cloud Gateway — two "gateway-shaped" things kept deliberately distinct rather than left to blur; Day 136 layers three environment-specific values files on top of that one chart, citing Week 8 Day 51's Spring Profiles as the same externalized-config pattern one layer up the stack, and proving (not asserting) that a Kubernetes Secret is base64-encoded rather than encrypted; Day 137 adds Micrometer Tracing and Zipkin alongside the metrics pipeline already live since Week 11 Days 76–77, naming the one genuine blind spot — trace context does not automatically survive a hop through Kafka the way it does over HTTP, a real gap given Week 10 Day 70's Saga runs on exactly that transport — plus a Stripe-format API-design pass (versioning, cursor-vs-offset pagination proven via a concrete concurrent-insert trace, Week 19 Day 130's idempotency-key mechanism applied to Order creation, a structured error contract); Day 138 runs three chaos experiments against the Kubernetes-deployed platform under real k6 load (a sustained Payment-kill correctly done via scaling to zero replicas rather than a single `kubectl delete pod`, since the latter is just Day 99's own reconcile loop healing itself within seconds; Toxiproxy/Pumba latency injection against Product observing Gateway timeout behavior; connection pooling taught from zero and then saturated, its cascade-via-thread-pool-exhaustion failure mode distinguished explicitly from a dependency simply being down), a light Istio/Linkerd service-mesh sidecar-injection-plus-traffic-split demonstration explicitly framed as moving resilience out of application code and into infrastructure rather than a fancier Resilience4j, and — the Databricks-specific add-on — a hand-rolled generic `BoundedBlockingQueue<T>` built from `ReentrantLock`/`Condition` alone, directly extending Week 6 Day 38's Producer-Consumer exercise rather than introducing new concurrency primitives, with a broken `if`-guarded version's exact race traced by hand before the correct `while`-guarded version's own invariants are proven under genuine concurrent contention using the identical `CountDownLatch`-gated pattern first built for Week 17 Days 116–117's BookMyShow race fix; Day 139 closes the loop opened by Week 7 Day 47/Week 8 Day 52's TestContainers-vs-WireMock realism-vs-control distinction by proving code coverage isn't correctness (a 100%-covered, zero-assertion divide-by-zero counterexample) and naming mutation testing (PIT) as the honest alternative before wiring JaCoCo as an actually build-failing gate rather than a mere report, finalizes a GitHub Actions pipeline traced end to end including its failure path, and adds liveness/readiness probes with the well-known dependency-in-liveness anti-pattern named explicitly and its correct home (readiness) justified by Day 134's own Endpoints-removal mechanism; Day 140 closes the week with zero new material — a self-check citing Week 10 Day 70 vs. Week 12 Day 79 (Saga vs. 2PC), Week 10 Days 66/68 plus Week 18 Day 121 (why rate limiting centralizes at the Gateway), and Week 7 Day 44 vs. Week 15 Day 99 plus Day 135 (why Kubernetes/Helm over docker-compose); a bottleneck analysis reading Day 134's baseline against Day 138's three chaos results; and the platform's own first estimation-and-bottleneck pass, explicitly citing all sixteen Weeks 18–19 HLD systems as the precedent being extended to a system this series actually built rather than only designed, naming Week 12 Day 78's Sharding Strategies and Week 9 Day 60's Consistent Hashing by name as exactly what a genuine 100x fix would require. **Two discrepancies flagged, not silently resolved, both in the new "How to Use This Document for Week 21" section below:** `Week_20_Revised.md`'s own claim that `todo-api` and "the original plan's `order-management-api`" "both got real Kubernetes deployments" earlier in the series is unsupported by this map's history, which shows `todo-api` as Docker/Compose-only, frozen since Day 62, and shows (per `Week_21_Revised.md`'s own later note) that `order-management-api` never existed as a separate repository at all; and the plan's Day 139 claim that readiness/liveness probes were "already configured back in the DSA phase for `todo-api`" rests on that same unsupported premise. Neither claim changed what Day 134 or Day 139 actually needed to teach, so both were built as genuinely new material regardless — nothing from the Week 1/2/3/4/5/6/7/8/9/10/11/12/13/14/15/16/17/18/19 version below was removed or shortened by this extension either, only appended to; noted here rather than silently fixed: that trailing tally itself already fell one week behind before this extension, since the Week 19 clause immediately above ends "...only appended to" listing versions only through 18, not 19 — left exactly as originally written, per this map's own extend-don't-edit rule, rather than corrected in place.

---

## How this series works

1. **One Resource Book per day** (`Day1_Resource_Book.md` → `Day119_Resource_Book.md`, so far — the DSA phase's full run, Days 1–105, plus the now-complete ten-system LLD phase, Days 106–119: Week 16's four-system opening, Days 106–112, and Week 17's six-system close, Days 113–119). Each is self-contained for that day's *new* material, but explicitly builds on every earlier day, across week boundaries.
2. **Strict dependency ordering.** Nothing is used before it's taught.
3. **Every problem gets the full treatment:** brute force → optimized → alternative approaches → why one wins → complexity → common mistakes/edge cases → what a tier-1 interviewer is actually evaluating.
4. **Every pattern gets more reps than the plan lists**, clearly labeled **Extra Practice** so it's always clear what's from the plan vs. added — except when a pattern is still *opening* rather than closing, in which case extra reps are deliberately deferred rather than risking a collision with the next week's required problems (see Week 2, Day 14; Week 3, Day 21; also Week 4, Day 23 — Greedy & Intervals opening the same way, though this particular pattern went on to close within the same week, so its extras landed on Days 24–25 instead of being pushed to a later week; also Week 5 — Binary Search was already open going into Day 29, not opening fresh, so its extras were fair game mid-pattern and landed on Days 31 and 33, while Linked Lists genuinely opened on Day 34 and correctly got none that day, landing on Day 35 instead; also Week 6 — Stacks opened Day 39 and Monotonic Stack opened Day 40, each correctly receiving no extras on its own opening day, with Monotonic Stack's deferred reps landing on Day 41 instead, while Linked Lists — already open, not opening — got two more extras on Day 36, mid-pattern, the same way Binary Search did the week before; also Week 7 — Trees opened Day 46 and correctly received no extras that day, with its first three extras spread across Days 47, 48, and 49 instead of stacked on one day, since none of Day 46's two problems, being the pattern's very first exposure, had anything yet to reinforce; also Week 8 — Heaps opened Day 54 and correctly received no extras that day either, with its first three extras spread across Days 55 and 56 instead, one of them — Minimum Cost to Connect Sticks — specifically deferred from a natural Day 54 pairing with Last Stone Weight for exactly this reason; also Week 11 — Union-Find opened Day 74 and correctly received no extras that day, with its one deferred extra (LC 1319, itself earmarked back in Week 10's own Day 69 note) landing on Day 75 instead, once the pattern was one day past opening — the same precedent, held for the sixth consecutive genuinely-new pattern in the series; also Week 12 — Dijkstra's Algorithm opened Day 78 and correctly received no extras that day, but unlike every prior pattern, no *deferred* extra landed later in the pattern's run either: its required ladder had already been expanded from 3 to 5 problems by the plan's own revision specifically to close a known gap, and the five problems already span every distinct relaxation shape the pattern tests in interviews, so the usual "extras land once the pattern is past opening" step was deliberately skipped this time, with the reasoning stated explicitly on Day 80 rather than left as a silent zero; Dynamic Programming opened Day 81 and correctly received none that day either, with its one extra (Delete and Earn, reinforcing the House Robber family with a third, differently-shaped variant) landing on Day 82 instead, once the pattern was one day past opening — the same precedent, held for the seventh consecutive genuinely-new pattern in the series; also Week 13 — **a deliberate, flagged departure from this precedent**: Grid DP (Day 87) and Interval DP (Day 91) are the first two patterns in the series whose entire opening-through-close arc fits inside a single calendar day, leaving no later, non-opening day within the pattern to defer extras to the way every multi-day pattern above could. Rather than force an extra into some other, unrelated day, both extras (Dungeon Game, Day 87; Predict the Winner, Day 91) were placed on the pattern's own opening-and-closing day itself, but explicitly marked as optional "if time allows" extension material — a different mechanism serving the same underlying purpose this precedent protects (keeping the day's true required pacing intact), adapted for a pattern shape none of the prior single-week patterns had); also Week 14 — State Machine DP opened Day 92 and correctly received no extras that day *or* the day after (Day 93), its four required problems already spanning every shape the pattern tests, the reasoning stated explicitly rather than left a silent zero, exactly Dijkstra's Algorithm's Week 12 treatment repeated; Tree DP opened and closed within a single day (Day 94), joining Grid DP's and Interval DP's Week 13 precedent for single-day patterns, its one extra (Binary Tree Cameras) placed on that same opening-and-closing day and marked extension material, the same mechanism Week 13 established; Bit Manipulation opened Day 95 and correctly received no extras that day either — the eighth consecutive genuinely-new pattern in the series to receive that treatment — with its one deferred extra (Hamming Distance) landing on Day 97, two days past opening rather than the usual one, since Day 96 was judged to already carry sufficient new material (a DP-fusion problem) without adding practice on top; also Week 15 — Segment Trees opened and closed within two days (Days 100–101) and, unlike Grid DP's and Interval DP's Week 13 precedent of placing one marked-extension extra on a pattern's own single-day arc, received **zero** extras on either day — its two-problem required ladder judged, by the plan's own explicit reasoning, to already be sufficient for a deliberately-light topic, the same "ladder already sufficient, stated explicitly" mechanism Dijkstra's Algorithm (Week 12) and State Machine DP (Week 14) used, rather than the single-day-token-extra mechanism Grid/Interval DP used; Bit Manipulation closed on Day 99 with no new extra added this week specifically (its one extra, Hamming Distance, had already landed the week before), continuing to close an already-open pattern rather than opening a fresh one; the newly-added SQL practice track (Days 103–104) and the Sorting-from-scratch exercise (Day 102) were both treated as deliberately fixed-scope tracks from the outset — 10 problems and one from-scratch implementation exercise respectively — rather than open-ended patterns, so neither received or was ever expected to receive extra practice.
5. **Every day ends with an interview question set**; `Week1_Interview_Questions.md`, `Week2_Interview_Questions.md`, `Week3_Interview_Questions.md`, `Week4_Interview_Questions.md`, `Week5_Interview_Questions.md`, `Week6_Interview_Questions.md`, `Week7_Interview_Questions.md`, `Week8_Interview_Questions.md`, `Week9_Interview_Questions.md`, `Week10_Interview_Questions.md`, `Week11_Interview_Questions.md`, `Week12_Interview_Questions.md`, `Week13_Interview_Questions.md`, `Week14_Interview_Questions.md`, `Week15_Interview_Questions.md`, `Week16_Interview_Questions.md`, and `Week17_Interview_Questions.md` each consolidate their week's questions.
6. **Overlap is checked in both directions, every week.** Before adding extra practice, check it isn't about to be required next week. Before re-teaching a "required" problem, check it wasn't already solved as extra practice in an earlier week. Week 2 was the first real test of this — see the Overlap section below.

---

## Full Concept-Dependency Map (Weeks 1–17)

```
DAY 1 — Absolute Foundations (no prior programming knowledge assumed)
├─ Program model: source → compiler → bytecode → JVM
├─ Dev environment: JDK, IDE, command line basics
├─ Variables & primitive types (int, double, boolean, char)
├─ Operators incl. the "= vs ==" trap
├─ Control flow: if/else if/else, switch (classic + arrow syntax)
├─ Loops: for, while, do-while, break/continue
├─ Methods: declaration, parameters, return, overloading, pass-by-value (primitives)
├─ Practice: FizzBuzz · Celsius→Fahrenheit · Prime Checker
└─ Git basics · Career block guide
        ▼
DAY 2 — Arrays, Strings, and Object-Oriented Programming (needs: Day 1)
├─ Arrays: fixed-size, contiguous memory, O(1) indexed access — why
├─ Strings as objects: immutability, == vs .equals()
├─ Practice: max-in-array · reverse-in-place · char frequency · palindrome check
├─ OOP: class vs. object, constructors, `this`, encapsulation, inheritance, interfaces
├─ Pass-by-value for objects/arrays — completes Day 1's story
└─ Practice: Book class · Shape/Circle/Rectangle hierarchy
        ▼
DAY 3 — Big-O Notation (full treatment) and ArrayList (needs: Days 1-2)
├─ Big-O formalized: O(1), O(log n), O(n), O(n log n), O(n²)
├─ Amortized analysis (needed for ArrayList's growth)
├─ ArrayList: dynamic resizing, doubling strategy, autoboxing
└─ Practice: redo Day 2 array exercises with ArrayList
        ▼
DAY 4 — HashSet, HashMap, Stack, Queue (needs: Big-O — Day 3; .equals() — Day 2)
├─ HashSet: uniqueness/membership, O(1) average, built on HashMap internally
├─ HashMap: key→value, hash function → bucket intuition, equals()/hashCode() contract
├─ Stack (LIFO) & Queue (FIFO) via ArrayDeque
└─ Practice: word frequency · dedup with HashSet · Valid Parentheses + 3 extra stack reps
        ▼
DAY 5 — HashMap/HashSet as a Formal Interview Pattern (12 problems) + OOP's Four Pillars
│  (needs: HashMap/HashSet — Day 4; OOP basics — Day 2)
├─ Pattern family: complement lookup, membership, frequency counting,
│  canonical-key grouping, two-way mapping, smart-starting-point traversal
├─ 7 plan problems + 5 extra practice problems (12 total)
└─ OOP Four Pillars formalized · enum with per-constant behavior · Account hierarchy
        ▼
DAY 6 — Two Pointers Begins (6 problems) + SOLID Principles
│  (needs: arrays — Day 2)
├─ Pattern family: opposite-ends convergence, from-the-back merging, same-direction (fast/slow)
├─ 3 plan problems + 3 extra practice problems (6 total)
└─ SOLID principles (needs interfaces — Day 2; polymorphism — Day 5)
        ▼
DAY 7 — Two Pointers Continues (6 problems) + Week 1 Consolidation
   (needs: Two Pointers — Day 6; HashMap — Day 5, for one bridge problem)
   ├─ Pattern family: one forward pointer per structure, opposite-ends with one allowed skip,
   │  same-direction (swap, not overwrite), same-direction without sortedness
   ├─ 2 plan problems + 4 extra practice problems (6 total)
   └─ Week 1 wrap-up, self-check, what Week 2 assumes you already know cold
        ▼
DAY 8 — Two Pointers Continues (recap LC 26, 283 + 1 new: LC 80) + Recursion
   (needs: fast-slow variant — Day 6-7; method calls — Day 1)
   ├─ LC 26, LC 283 recapped (already solved Week 1, Day 6/7 Extra) — see Overlap section
   ├─ 1 new fast-slow rep: LC 80 (Remove Duplicates II)
   └─ Recursion: call stack made explicit, base/recursive case, StackOverflowError,
      fib(5) traced by hand, previews memoization
        ▼
DAY 9 — Two Pointers Continues (recap LC 977, 167 + 1 new: LC 633) + JVM Memory Model
   (needs: opposite-ends variant — Day 6; stack frames — Day 8)
   ├─ LC 977, LC 167 recapped (already solved Week 1, Day 6 Extra)
   ├─ 1 new opposite-ends rep: LC 633 (Sum of Square Numbers — implicit range, not an array)
   └─ JVM Memory Model: stack vs. heap, WHY pass-by-value holds (mechanism, not just rule)
        ▼
DAY 10 — 3Sum recap + 3Sum Closest (new) + Primitives, Precisely
   (needs: 3Sum skeleton — Day 7; stack/heap — Day 9)
   ├─ LC 15 recapped (already solved Week 1, Day 7 Extra)
   ├─ LC 16 (3Sum Closest) — new, full depth
   └─ Primitives: exact ranges, overflow wraparound, Integer cache (-128..127),
      reconnects to == vs .equals() (Week 1 Day 2)
        ▼
DAY 11 — 4Sum + Boats to Save Most People (both new) + String Internals
   (needs: 3Sum/3Sum-Closest skeleton — Day 10; overflow — Day 10)
   ├─ LC 18 (4Sum) — new; requires `long` for sums (Day 10's overflow, applied)
   ├─ LC 881 (Boats to Save Most People) — new; opposite-ends, greedy pairing (new sub-variant)
   │  (no extra practice added — Assign Cookies considered and rejected, see Day 11 book)
   └─ String Internals: immutability, string pool, StringBuilder,
      += in a loop is O(n²) even accounting for compiler's per-statement optimization
        ▼
DAY 12 — Container recap + Sort Colors (new) + Collections Internals
   (needs: opposite-ends provable-greedy — Day 7; amortized doubling — Day 3)
   ├─ LC 11 recapped (already solved Week 1, Day 7 Extra)
   ├─ LC 75 (Sort Colors) — new; Three-Way Partition / Dutch National Flag (new sub-variant)
   │  (no extra practice added — narrow technique, resurfaces with Quickselect later)
   └─ Collections Internals: ArrayList vs. LinkedList vs. ArrayDeque,
      contiguous array vs. linked nodes vs. circular array, cache locality
        ▼
DAY 13 — Trapping Rain Water (Hard, capstone) + Two Pointers Full Consolidation
   (needs: every Two Pointers variant taught so far — Days 6-12)
   ├─ LC 42 (Trapping Rain Water) — new; hardest correctness proof in the ladder
   └─ Two Pointers, Reviewed: all 16 required problems classified across
      7 named variants (not just opposite-ends/fast-slow) — full answer key
   **Two Pointers closes here: 16/16 required problems (5 Week 1 + 11 Week 2).**
        ▼
DAY 14 (Sun) — Consolidation, Sliding Window Begins
   (needs: same-direction fast-slow — Day 6-7, direct ancestor of Sliding Window)
   ├─ LC 121 (Best Time to Buy/Sell Stock) — new; single-pass min-tracking
   ├─ LC 643 (Maximum Average Subarray I) — new; true fixed-size window
   │  (no extra practice added — pattern is opening, not closing; see Overlap section)
   ├─ Sliding Window: full mechanism, fixed vs. variable size, lineage from fast-slow
   └─ Week 2 Consolidation: planned-vs-actual count, diagnostic, what Week 3 assumes
        ▼
DAY 15 — Sliding Window Continues (Variable-Size, NEW mechanism), HashMap Internals
   (needs: fixed-size window — Day 14; HashMap/equals-hashCode contract — Day 4)
   ├─ LC 1004 (Max Consecutive Ones III) — new; variable-size window, violation counter
   ├─ LC 3 (Longest Substring Without Repeating Characters) — new; variable-size, HashSet
   ├─ LC 1695 (Maximum Erasure Value) — extra; HashSet uniqueness + running sum
   └─ HashMap Internals: hashCode() → spread → bucket index, treeification (threshold 8,
      min capacity 64), resize/amortized cost, constant-hashCode vs. contract-violation bugs
        ▼
DAY 16 — Sliding Window with Anagrams, Generics
   (needs: variable-size window — Day 15; fixed-size window — Day 14; classes — Day 2)
   ├─ LC 424 (Longest Repeating Character Replacement) — new; stale-maxFreq proof
   ├─ LC 567 (Permutation in String) — new; fixed-size, frequency-array match
   ├─ LC 1052 (Grumpy Bookstore Owner) — extra; fixed-size, gain-maximization
   └─ Generics: <T>, bounded type parameters, PECS wildcards, type erasure (why new T[]
      doesn't compile)
        ▼
DAY 17 — Sliding Window with HashMap, Comparable vs. Comparator
   (needs: LC 567's skeleton — Day 16; shrink-while-valid — new today)
   ├─ LC 438 (Find All Anagrams in a String) — new; LC 567's skeleton, collect all matches
   ├─ LC 209 (Minimum Size Subarray Sum) — new; shrink-WHILE-valid (minimize), new loop shape
   └─ Comparable vs. Comparator; TreeMap (Red-Black tree, O(log n)); PriorityQueue
      (binary heap array, O(log n) insert/poll, O(1) peek)
        ▼
DAY 18 — Sliding Window: At-Most-K-Distinct, Exception Handling
   (needs: shrink-until-valid — Day 15; StackOverflowError IS-A Error — Day 8/9)
   ├─ LC 904 (Fruit Into Baskets) — new; at-most-2-distinct via HashMap.size()
   ├─ LC 1493 (Longest Subarray of 1's After Deleting One Element) — new; −1 mandatory-deletion proof
   ├─ LC 1838 (Frequency of the Most Frequent Element) — extra; sort + budget-constrained window
   └─ Exception Handling: Throwable hierarchy, checked vs. unchecked, try-with-resources
      / AutoCloseable (the suppressed-exception fix over manual try/finally)
        ▼
DAY 19 — Sliding Window: Two-Pointer Hard Tier (no new theory)
   (needs: LC 904's at-most-K shape — Day 18; LC 209's shrink-while-valid — Day 17)
   ├─ LC 340 (Longest Substring with At Most K Distinct Characters) — new; LC 904 generalized
   └─ LC 76 (Minimum Window Substring) — new, Hard; required/formed counter pair
        ▼
DAY 20 — Sliding Window Capstone (Monotonic Deque), Pattern Wrap-Up
   (needs: ArrayDeque — Day 4; LC 340's at-most-K helper — Day 19)
   ├─ LC 239 (Sliding Window Maximum) — new, Hard; monotonic deque, domination argument
   ├─ LC 992 (Subarrays with K Different Integers) — new, Hard; exactly-K = atMost(K)−atMost(K−1)
   ├─ LC 1438 (Longest Subarray, Abs Diff ≤ Limit) — extra; TWO monotonic deques (max+min)
   └─ Sliding Window, Reviewed: all 18 problems (14 required + 4 extra) classified
      fixed vs. variable — full answer key
   **Sliding Window closes here: 14/14 required problems (2 Week 2 + 12 Week 3) + 4 extra
   (all Week 3) = 18 distinct problems total.**
        ▼
DAY 21 (Sun) — Leave Week Begins: Prefix Sum & Kadane's Algorithm; Week 3 Consolidation
   (needs: arrays — Day 2; HashMap — Day 4/5. NOT dependent on Sliding Window.)
   ├─ LC 303 (Range Sum Query - Immutable) — new; prefix sum, telescoping argument
   ├─ LC 53 (Maximum Subarray) — new; Kadane's, full "extend or restart" proof
   ├─ LC 238 (Product of Array Except Self) — new; prefix product × suffix product, no division
   │  (no extra practice added — pattern is opening, not closing; mirrors Day 14's precedent)
   └─ Week 3 Consolidation: planned-vs-actual count, diagnostic, what Week 4 assumes
        ▼
DAY 22 — Prefix Sum & Kadane's Completes (HashMap half of the pattern)
   (needs: prefix sum/Kadane's — Day 21; HashMap — Day 4/5)
   ├─ LC 525 (Contiguous Array) — new; ±1 transform, first-occurrence-index HashMap
   ├─ LC 523 (Continuous Subarray Sum) — new; prefix sum mod k, first-occurrence-index
   ├─ LC 560 (Subarray Sum Equals K) — RECAPPED (Week 1, Day 5 Extra); frequency-count HashMap,
   │  formally framed for the first time as this pattern's "how many" flavor
   ├─ LC 918 (Maximum Subarray Sum Circular) — new; Kadane's run twice, all-negative edge case
   ├─ LC 974 (Subarray Sums Divisible by K) — extra; frequency-count + mod k, negative-remainder fix
   └─ LC 152 (Maximum Product Subarray) — extra; Kadane's extended to running max AND min
   **Prefix Sum & Kadane's closes here: 7/7 required (3 Week 3 + 4 Week 4) + 2 extra
   (both Week 4) = 9 distinct problems total.**
        ▼
DAY 23 — Greedy & Intervals Begins
   (needs: arrays — Day 2; sorting — Day 3. Informally already used, unnamed, Day 11/Day 12.)
   ├─ Greedy, Formalized: definition, the exchange-argument proof template,
   │  a worked 0/1 Knapsack counter-example showing exactly where greedy fails
   ├─ LC 122 (Best Time to Buy/Sell Stock II) — new; telescoping decomposition, unlimited txns
   ├─ LC 55 (Jump Game) — new; frontier-domination argument
   └─ LC 45 (Jump Game II) — new; BFS-by-levels in disguise, frontier domination extended
   (no extra practice added — pattern is opening, not closing; mirrors Day 14/21's precedent)
        ▼
DAY 24 — Greedy Continues, Intervals Begins (sort by START time)
   (needs: Day 23's exchange argument; arrays/sorting — Days 2-3)
   ├─ LC 134 (Gas Station) — new; prefix-elimination argument (a THIRD distinct exchange-argument shape)
   ├─ Intervals: sort-by-start, sweep-once mechanism, formally named for the first time
   ├─ LC 56 (Merge Intervals) — new; combine overlapping ranges
   ├─ LC 57 (Insert Interval) — new; 3-phase, O(n) by exploiting given sorted input
   └─ LC 406 (Queue Reconstruction by Height) — extra; greedy with a SECOND sort key
        ▼
DAY 25 — Intervals Continues (sort by END time — a different key, a different question)
   (needs: Day 24's Intervals mechanism)
   ├─ LC 435 (Non-overlapping Intervals) — new; greedy-by-end-time, exchange argument stated in general form
   ├─ LC 452 (Min Arrows to Burst Balloons) — new; identical shape to LC 435, reframed discard→cover
   └─ LC 986 (Interval List Intersections) — extra; TWO separate lists, two pointers (not a single sweep)
        ▼
DAY 26 — Meeting Rooms (Intervals + Min-Heap)
   (needs: PriorityQueue — Day 17; Day 24-25's Intervals mechanism)
   ├─ LC 252 (Meeting Rooms) — new; existence check, sort by start, adjacent-pair comparison
   └─ LC 253 (Meeting Rooms II) — new; min-heap of end times, counting; heap size grows
      monotonically so the FINAL size already equals the answer, no separate max needed
        ▼
DAY 27 — Greedy & Intervals Capstone, Leave Week Wrap-Up
   (needs: every Greedy & Intervals variant taught so far — Days 23-26)
   ├─ LC 763 (Partition Labels) — new; interval reasoning via last-occurrence index,
   │  NO explicit interval object — Merge Intervals' mechanism, derived on the fly
   └─ Greedy, Reviewed: 3 of the week's exchange arguments restated as one-sentence proofs
      (domination — Jump Game; interval-swap — Non-overlapping Intervals;
       prefix-elimination — Gas Station) — 3 distinct argument SHAPES, not repetitions
   **Greedy & Intervals closes here: 11/11 required (all Week 4) + 2 extra
   (both Week 4) = 13 distinct problems total.**
        ▼
DAY 28 (Sun) — Consolidation, Binary Search Begins
   (needs: arrays/sorting — Days 2-3; int overflow — Day 10, directly, for mid-calculation)
   ├─ LC 704 (Binary Search) — new; exact-match template, ON THE INPUT
   ├─ LC 278 (First Bad Version) — new; boundary search on a monotonic condition, ON THE ANSWER
   │  (no extra practice added — pattern is opening, not closing; mirrors Day 14/21/23's precedent)
   ├─ Records: compact immutable data carrier, standard since Java 16 — auto constructor/
   │  accessors (`.x()` not `.getX()`)/equals()/hashCode()/toString(), implicitly final
   ├─ Sealed interfaces: `permits` clause, standard since Java 17 — restricted, compiler-known hierarchy
   ├─ Exhaustive switch via type patterns over a sealed hierarchy — standard only since Java 21
   │  (JEP 441); plan's "Java 17+" corrected precisely — records/sealed themselves are fine on 17+,
   │  the no-default exhaustiveness check specifically needs 21+
   └─ Week 4 Consolidation: planned-vs-actual count (76 cumulative distinct), diagnostic,
      what Week 5 assumes
        ▼
DAY 29 — Binary Search Continues, Threads and the JVM Concurrency Model
   (needs: exact-match template + overflow-safe midpoint — Day 28)
   ├─ LC 35 (Search Insert Position) — new; exact-match template, answer read off
   │  where the pointers converge, not off a found flag
   ├─ LC 162 (Find Peak Element) — new; binary search on an UNSORTED array —
   │  local slope comparison replaces global sortedness as the elimination rule
   │  (LC 852 mentioned as a strictly-easier variant, not given full treatment)
   ├─ Threads and the JVM Concurrency Model — entirely fresh theory thread, no
   │  dependency on anything from Week 4; DIRECTLY extends Day 9 (one stack per
   │  thread, one shared heap per process)
   ├─ Thread vs. Runnable — needs single inheritance, Day 2
   └─ NEW SYNTAX: anonymous inner classes (needed for Runnable; reused Day 35
      for a custom Comparator)
        ▼
DAY 30 — Binary Search: Rotated Arrays
   (needs: Day 29's proof style — invariant preserved each step, forces a unique answer)
   ├─ LC 33 (Search in Rotated Sorted Array) — new; splitting at mid always
   │  leaves ≥1 half normally sorted
   │  (no extra practice added — both reps stay this day; the natural
   │  duplicate-handling extension needs Day 31's LC 153 taught first)
   └─ LC 81 (Search in Rotated Sorted Array II) — new; duplicates break "which
      half is sorted," fix costs the O(log n) guarantee, degrades to O(n) worst case
        ▼
DAY 31 — Binary Search: Minimum in Rotated Arrays, Boundary Search
   (needs: Day 30's rotated-array invariant and duplicate-handling fix)
   ├─ LC 153 (Find Minimum in Rotated Sorted Array) — new; compare nums[mid] to
   │  nums[right], not nums[left] — mid can equal left, never right
   ├─ LC 154 (Find Minimum in Rotated Sorted Array II) — EXTRA; duplicate
   │  extension of LC 153, same shape as Day 30's LC 33→81 jump
   ├─ LC 34 (Find First and Last Position) — new; BOUNDARY SEARCH — on a match,
   │  keep narrowing instead of stopping, biased left (first) or right (last)
   └─ LC 744 (Find Smallest Letter Greater Than Target) — EXTRA; same boundary
      shape, single-sided
        ▼
DAY 32 — Binary Search: 2D Matrices, Binary Search on the Answer
   (needs: Day 28's LC 704 template; Day 10/11's long-for-overflow-risk habit)
   ├─ LC 74 (Search a 2D Matrix) — new; row-major flatten to 1D, valid only
   │  because each row's first value exceeds the previous row's last
   ├─ LC 240 (Search a 2D Matrix II) — discussed for CONTRAST only, not solved
   │  with full treatment (no code/trace); different guarantee (rows+cols
   │  individually sorted, not globally), needs a different technique
   │  (staircase from top-right, O(m+n)) — NOT counted in the Problem
   │  Inventory below
   └─ LC 875 (Koko Eating Bananas) — new; BINARY SEARCH ON THE ANSWER, formally
      reused from Day 28's LC 278 — monotonic feasibility over a range of
      candidate SPEEDS, not array indices
        ▼
DAY 33 — Binary Search Capstone: On-the-Answer Completed and Reviewed
   (needs: Day 32's "on the answer" framing, made explicit)
   ├─ LC 1011 (Capacity To Ship Packages Within D Days) — new; identical shell
   │  to Koko, greedy day-counting simulation as the feasibility check
   ├─ LC 1482 (Minimum Number of Days to Make m Bouquets) — EXTRA; feasibility
   │  check needs ADJACENCY tracking, not just a sum — same shell, different
   │  internals; needs an upfront impossibility guard no other problem this
   │  week needed
   ├─ LC 1552 (Magnetic Force Between Two Balls) — EXTRA; MAXIMIZES the answer
   │  instead of minimizing — shrink direction flips relative to every other
   │  on-the-answer problem this week
   └─ Binary Search, Reviewed: full "on the input" vs. "on the answer"
      classification across all 15 distinct problems (Weeks 4–5 combined) —
      formalizes the distinction Day 28 first drew with only 2 data points
   **Binary Search closes here: 11/11 required (9 Week 5 + 2 Week 4) + 4 extra
   (all Week 5: LC 154, 744, 1482, 1552) = 15 distinct problems total.**
        ▼
DAY 34 — Linked Lists Begin, Spring Boot Initialization
   (needs: OOP/classes — Day 2; heap references — Day 9, DIRECTLY, no new
   memory mechanism, only a new SHAPE; ArrayList trade-offs — Day 3/12,
   PREVIEWED on Day 12's Collections Internals, PROVEN here)
   ├─ LC 206 (Reverse Linked List) — new; iterative (3-pointer) AND recursive
   │  (needs: recursion — Day 8; space cost O(n) call stack, not O(1))
   ├─ LC 876 (Middle of the Linked List) — new; fast/slow — SAME variant as
   │  Day 6/7's Two Pointers, applied where no index arithmetic is possible
   │  (no extra practice added — pattern is opening, not closing; mirrors
   │  Day 14/21/23/28's precedent)
   ├─ REST APIs and Spring Boot — entirely fresh theory thread; HTTP
   │  verbs/status codes, statelessness as a real scaling trade-off
   ├─ NEW: reflection (a running program inspecting its own classes/methods/
   │  annotations at RUNTIME) — how Spring's @RestController/@GetMapping
   │  differ mechanically from Day 2/5's compile-time-only @Override
   └─ ResponseWrapper<T> — Day 16's Generics, simplest case: T fully unbounded
   **⚠️ `Week_05_Revised.md`'s own header states Linked Lists "grows... to 12";
   this conflicts with `Week_06_Revised.md`'s header ("closes at 11") and with
   Week 5's OWN Day 35 scorecard text ("expanded 11-problem set"). 11 is used
   throughout as correct — verified directly by arithmetic (4 Week 5 + 7 Week 6
   required = 11) and by 2-of-3 independent sources agreeing.**
        ▼
DAY 35 (Sun) — Linked Lists Continue, Week 5 Consolidation
   (needs: Day 34's fast/slow AND iterative reversal, COMBINED — no new
   mechanism, only a new combination)
   ├─ LC 234 (Palindrome Linked List) — new; fast/slow + reversal combined;
   │  odd-length middle node ends up compared to itself, falls out for free
   ├─ LC 21 (Merge Two Sorted Lists) — new; NEW TECHNIQUE — dummy head, avoids
   │  special-casing an empty result list
   ├─ LC 92 (Reverse Linked List II) — EXTRA; bounded reversal, reuses today's
   │  dummy head to avoid special-casing left=1
   ├─ LC 23 (Merge k Sorted Lists) — EXTRA; TWO distinct O(N log k) approaches
   │  — heap (Day 17 PriorityQueue + Day 29 anonymous Comparator, O(k) space)
   │  vs. divide-and-conquer (reuses today's own merge routine, O(log k) space)
   └─ Week 5 Consolidation: planned-vs-actual count (95 cumulative distinct;
      70 required-ladder-only, matching the plan's own convention — see Running
      Totals below), diagnostic, what Week 6 assumes
   **Linked Lists opens here: 4/11 required + 2 extra (both Day 35: LC 92, 23)
   = 6 distinct problems solved so far; continues in Week 6.**
        ▼
DAY 36 — Linked Lists: Cycles, and Spring Data JPA
   (needs: Day 34's fast/slow mechanism — SAME two pointers, stopping
   condition changes from "fast reaches end" to "slow==fast")
   ├─ LC 141 (Linked List Cycle) — new; Floyd's Cycle Detection —
   │  closing-gap proof (gap shrinks by exactly 1/iteration once both
   │  pointers are inside the cycle)
   ├─ LC 142 (Linked List Cycle II) — new; Floyd's, extended — phase-2
   │  reset-to-head derived from a = (n-1)c + (c-b), not memorized
   ├─ LC 202 (Happy Number) — EXTRA; Floyd's generalized beyond linked
   │  lists — next(n)=sum of squared digits treated as a "next pointer"
   ├─ LC 287 (Find the Duplicate Number) — EXTRA; Floyd's applied to an
   │  array read as implicit pointers (nums[i] = "next"); O(1) space
   │  constraint is the tell
   ├─ Spring Data JPA — entirely fresh theory thread; @Entity/@Id/@Column
   │  read via reflection (Day 34's mechanism, new work); JpaRepository
   │  generated at runtime via the SAME reflection mechanism
   └─ todo-api gets its first real persistence — Task entity + repo,
      Postgres via application.yml, ddl-auto: update
   **(no extra practice withheld here — Linked Lists is continuing, not
   opening; mirrors Binary Search's Day 31/33 precedent, not Day 34's)**
        ▼
DAY 37 — Linked Lists: Harder Pointer Manipulation, and Explicit Locks
   (needs: Day 35's dummy head, reused for BOTH problems today)
   ├─ LC 19 (Remove Nth Node From End) — new; fixed-offset two pointers,
   │  gap = n+1 derived from a worked trace, not asserted
   ├─ LC 2 (Add Two Numbers) — new; math simulation on a linked list,
   │  carry != 0 loop condition catches the final-digit-overflow edge case
   ├─ NEW: `synchronized` — brief primer, injected here (not in the
   │  original plan's text) as a required baseline before ReentrantLock's
   │  comparison means anything; intrinsic locks are ALREADY reentrant —
   │  flagged explicitly as a common misconception ReentrantLock's own
   │  name invites
   ├─ NEW: ReentrantLock — tryLock/timeout, fairness, Condition support
   │  (needed directly, not optionally, by Day 38)
   └─ FIRST actual data-corruption demo in this series (count++ under
      100 threads) — Day 29 showed non-deterministic ORDERING only;
      today shows a genuinely WRONG final answer, then fixes it
   **Counter (java-fundamentals) — pushed, verified under 100 threads**
        ▼
DAY 38 — Linked Lists: Advanced Combinations, and Wait/Notify
   (needs: Day 34 fast/slow + reversal + Day 35 dummy-head merge, ALL
   THREE combined with no new pointer mechanism; needs Day 37's
   ReentrantLock directly — Condition is created FROM a Lock)
   ├─ LC 143 (Reorder List) — new; three known techniques chained:
   │  find middle → reverse second half → merge by ALTERNATING (not
   │  sorted order, unlike Day 35's merge)
   ├─ LC 138 (Copy List with Random Pointer) — new; HashMap old-node→
   │  new-node mapping; NEW node shape (`random`, distinct from Day 39's
   │  `prev`) — map.get(null) safely returns null, no branch needed
   ├─ NEW: wait()/notify()/notifyAll() — one wait-set per object; the
   │  while-not-if rule (spurious wakeups ARE real, notifyAll() wakes
   │  irrelevant threads too)
   ├─ NEW: Condition (lock.newCondition()) — MULTIPLE wait-sets from ONE
   │  lock; this is what actually fixes the single-wait-set inefficiency
   └─ Producer-Consumer (java-fundamentals) — ReentrantLock + two
      Conditions (notFull, notEmpty); correctness argued, not just run
        ▼
DAY 39 — Linked Lists Capstone, Stacks Begin, and ConcurrentHashMap
   (needs: NEW — doubly linked list, taught here from scratch — every
   prior linked-list day used only `next`; needs Day 15 HashMap
   internals + Day 37 synchronized/locks for ConcurrentHashMap)
   ├─ NEW: doubly linked list + dummy-head-AND-tail sentinel — extends
   │  Day 35's single dummy head to both ends simultaneously
   ├─ LC 146 (LRU Cache) — new; HashMap (O(1) lookup) + DLL (O(1)
   │  reordering) — CLOSES LINKED LISTS
   ├─ 🔗 LC 20 (Valid Parentheses) — RECAP, originally Week 1 Day 4
   │  (required then, required again here) — no re-teach, cited only
   ├─ Concept Card — Stacks — formal pattern intro; ArrayDeque already
   │  in informal use since Week 1, Day 4 (Stack vs. legacy
   │  java.util.Stack's unneeded synchronized overhead, explained)
   ├─ NEW: ConcurrentHashMap internals — CAS (defined precisely) +
   │  bucket-level synchronized, contrasted against
   │  Collections.synchronizedMap()'s single whole-map lock
   └─ ConcurrentMapBenchmark (java-fundamentals) — measured, not just
      asserted
   **Linked Lists CLOSES: 11/11 required (4 Wk5 + 7 Wk6) + 4 extra
   (2 Wk5: LC 92, 23 + 2 Wk6: LC 202, 287) = 15 distinct total.**
   **Stacks OPENS here — no extras today, mirrors Day 34's precedent.**
        ▼
DAY 40 — Stacks Continue, and SQL Fundamentals
   (needs: Day 39's Stacks formal identity; needs Day 36's Task entity
   fields for the Flyway migration to match exactly)
   ├─ 🔗 LC 232 (Implement Queue using Stacks) — RECAP, originally
   │  Week 1 Day 4 (extra practice then, required here)
   ├─ LC 496 (Next Greater Element I) — new; Monotonic Stack OPENS —
   │  decreasing-invariant PROVEN (not observed), amortized O(n) proof
   │  (pushed once/popped at most once) — freed recap time reinvested
   │  in depth here, not a bonus problem
   ├─ NEW: SQL Fundamentals — keys, normalization (1NF/2NF/3NF), JOIN
   │  types (LEFT fills NULL vs. INNER excludes — made concrete with an
   │  actual example table), indexes/B-trees; NOT IN vs. NOT EXISTS
   │  NULL-propagation trap
   └─ Flyway migration (todo-api) — V1__create_tasks_table.sql,
      ddl-auto: update → validate; column names verified against
      Day 36's entity, snake_case conversion explained
   **Monotonic Stack OPENS here — no extras today either (freed time
   went to depth on LC 496, not a bonus problem); deferred to Day 41.**
        ▼
DAY 41 — Stacks: Circular Variants, Min Stack, and Monotonic Stack Practice
   (needs: Day 40's Monotonic Stack invariant, reused unchanged; NO
   Theory/Project block today — matches `Week_06_Revised.md` exactly,
   not an omission; freed schedule slack used for extra practice instead)
   ├─ LC 503 (Next Greater Element II) — new; circular wrap (2n
   │  iterations, i%n) + index-based storage (values may repeat here,
   │  unlike LC 496) — SAME invariant, two new pieces layered on top
   ├─ LC 155 (Min Stack) — new; two stacks again, different roles;
   │  <= not < proven via an explicit buggy-vs-correct counter-trace
   ├─ LC 901 (Online Stock Span) — EXTRA; SAME monotonic mechanism,
   │  persisted ACROSS separate calls (streaming, not one array pass)
   └─ LC 402 (Remove K Digits) — EXTRA; MIRROR invariant — increasing
      stack, greedy digit removal
   **Monotonic Stack: 7/12 required (2 Wk1-recapped) + 2 extra (LC 901,
   402) = 9 distinct so far; continues into Week 7.**
        ▼
DAY 42 (Sun) — Consolidation, and Monotonic Stack Continues
   (needs: Day 40/41's index-based Monotonic Stack, reused unchanged;
   self-check reuses Days 36/38/39 cold, no new material there)
   ├─ Self-Check: one Linked List problem from this week, attempted cold
   ├─ LC 739 (Daily Temperatures) — new; SAME index-based Monotonic
   │  Stack as LC 503, recording a day-gap instead of a value
   ├─ LC 150 (Evaluate Reverse Polish Notation) — new; general-purpose
   │  stack (LIFO IS the algorithm, same role as LC 20) — NOT monotonic;
   │  pop-order proof via a division/subtraction counter-example
   └─ Week 6 Consolidation: planned-vs-actual count (111 cumulative
      distinct; 84 required-ladder-only, matching the plan's own Day 42
      claim exactly), diagnostic, what Week 7 assumes
        ▼
DAY 43 — Stack Simulation, and Docker Basics
   (needs: Day 39's Stack identity, reused unchanged; Day 40/41's
   Monotonic Stack invariant + amortized "pushed/popped once" proof —
   cited, not re-derived, since none of Week 7's five remaining Stack
   problems re-derive either from scratch)
   ├─ LC 735 (Asteroid Collision) — new; Stack Simulation — push
   │  right-movers, resolve collisions against the stack top on each
   │  left-mover (while, not if — a chain of destroyed right-movers)
   ├─ LC 227 (Basic Calculator II) — new; Stack — general-purpose
   │  LIFO, lastSign trick: push signed operands on +/-, pop-compute-
   │  push on */÷
   ├─ NEW: Docker Basics — image (static layered snapshot) vs.
   │  container (running instance); Dockerfile layer caching — an
   │  unchanged layer is reused, a changed layer invalidates itself
   │  and everything after it
   └─ todo-api: multi-stage Dockerfile (Stage 1 Maven build, Stage 2
      slim JRE runtime)
        ▼
DAY 44 — Stacks: The Hard Tier, and Docker Compose
   (needs: Day 43's Stack Simulation; Docker Basics — Day 43, directly,
   for Compose's multi-container orchestration)
   ├─ LC 84 (Largest Rectangle in Histogram) — new, Hard; Monotonic
   │  Stack (strictly increasing, indices not heights) — pop-and-
   │  compute-area on a shorter bar, new top is left boundary; also
   │  covered: divide-and-conquer alternative, O(n log n) average,
   │  O(n²) worst case on sorted input
   ├─ LC 85 (Maximal Rectangle) — new, Hard; row-by-row reduction to
   │  LC 84 — per-row "consecutive 1s" histogram, n applications of
   │  today's earlier algorithm
   ├─ NEW: Docker Compose — one docker-compose.yml orchestrates
   │  multiple services; same-network containers resolve each other
   │  by SERVICE NAME (DNS), not localhost, not manual IP; depends_on
   │  guarantees start order only, NOT readiness
   └─ todo-api: docker-compose.yml wiring app + Postgres + Redis,
      named volume for Postgres persistence
        ▼
DAY 45 — Stacks Capstone, and JUnit 5/AssertJ
   (needs: Day 43/44's Stack-as-state-holder shape, extended once
   more; JUnit 5 — entirely fresh theory thread)
   ├─ LC 224 (Basic Calculator) — new, Hard; Stack — push (result,
   │  sign) CONTEXT before each '(', restore after each ')'; broken-
   │  naive-attempt-first pedagogy (strip parens → wrong by exactly
   │  the nested-sign-propagation bug), mirroring Day 5's Isomorphic
   │  Strings treatment
   **Stacks/Monotonic Stack CLOSES: 12/12 required (7 Wk6 + 5 Wk7)
   + 2 extra (both Wk6: LC 901, 402) = 14 distinct total. Full two-
   family review given (general-purpose LIFO vs. true monotonic
   invariant).**
   ├─ NEW: JUnit 5 — @Test, @BeforeEach/@AfterEach, @ParameterizedTest;
   │  AssertJ fluent assertions chain multi-condition checks
   └─ todo-api: @DataJpaTest suite for TaskRepository (plan named
      TaskController; corrected in-place — @DataJpaTest doesn't load
      the web layer, so it verifies the persistence layer beneath the
      controller, not the controller itself), AssertJ assertions
        ▼
DAY 46 — Trees Begin, and Mockito
   (needs: NEW — TreeNode, taught here from scratch as a self-
   referential shape — DIRECTLY extends Linked Lists' Node (Weeks 5–6,
   ONE `next` reference) to TWO references (`left`, `right`); needs
   recursion — Day 8, DIRECTLY, no new call-stack mechanism, only a
   new shape to recurse over; needs Day 45's JUnit 5 for Mockito)
   ├─ NEW: Binary Trees — TreeNode shape; depth vs. height (precise,
   │  edge-based definitions, base case -1) vs. LeetCode's node-
   │  counting "Maximum Depth" convention (base case 0) — the two
   │  base cases named explicitly as a common off-by-one trap;
   │  preorder/inorder/postorder/level-order all named; universal
   │  recursive template (null → base case, combine left/right);
   │  space complexity stated as O(h), not defaulted to O(n)
   ├─ LC 104 (Maximum Depth of Binary Tree) — new; DFS, postorder-
   │  style combine: `1 + max(depth(left), depth(right))`; BFS
   │  level-counting given as a genuinely distinct alternative,
   │  previewing Day 49's scaffold
   ├─ LC 226 (Invert Binary Tree) — new; DFS, swap then recurse
   │  (no extra practice added — pattern is opening, not closing;
   │  mirrors Day 34/39/40's precedent)
   ├─ NEW: Mockito — @Mock/@InjectMocks fake a dependency out
   │  entirely, no DB/Spring context at all; DIRECTLY extends Day 45's
   │  JUnit 5 structure; unit (Mockito) vs. integration (@DataJpaTest)
   │  distinction named explicitly
   └─ todo-api: Mockito-based TaskService unit test, mocking
      TaskRepository, confirmed to run in milliseconds
   **Trees OPENS here — no extras today, matching Day 34/39/40's
   precedent; deferred to Days 47–49.**
        ▼
DAY 47 — Tree DFS Continues, and TestContainers
   (needs: Day 46's base recursive template, extended to TWO trees
   simultaneously)
   ├─ LC 100 (Same Tree) — new; DFS, two-tree structural comparison;
   │  brute-force serialize-and-compare given, WITH explicit null
   │  markers (seeds, but does not build out, Day 52's Serialize/
   │  Deserialize idea)
   ├─ LC 101 (Symmetric Tree) — new; DFS, mirrored comparison — Same
   │  Tree's exact logic, crossed (left.left vs. right.right)
   ├─ LC 572 (Subtree of Another Tree) — EXTRA; composes isSameTree
   │  as a subroutine, called at every node via DFS; value-vs-
   │  structure trap named explicitly (KMP mentioned as a further
   │  bound, out of scope, not taught)
   ├─ NEW: TestContainers — H2 doesn't enforce Postgres-specific
   │  behavior; spins up a REAL Postgres container (needs Docker —
   │  Day 43, directly); @DynamicPropertySource reads the container's
   │  dynamically-assigned port at test-run time
   └─ todo-api: @DataJpaTest suite now runs against real
      TestContainers Postgres, replacing H2, via @DynamicPropertySource
      + @AutoConfigureTestDatabase(replace = NONE)
        ▼
DAY 48 — Tree DFS with Global State, and Kafka Fundamentals
   (needs: Day 46/47's base template; NEW recursive shape — a
   sentinel short-circuit (Balanced) and a running max tracked
   OUTSIDE the returned value (Diameter), both introduced against the
   SAME redundant-recomputation trap named once and reused)
   ├─ LC 110 (Balanced Binary Tree) — new; DFS bottom-up, -1 sentinel
   │  short-circuits on first imbalance found anywhere below; brute-
   │  force redundant-height-recomputation cost stated as O(n log n)
   │  balanced / O(n²) skewed
   ├─ LC 543 (Diameter of Binary Tree) — new; DFS with a running max
   │  tracked separately from the height value returned upward — Java-
   │  specific note on WHY a local variable can't hold it (instance
   │  field vs. mutable-holder-array, both given)
   ├─ LC 111 (Minimum Depth of Binary Tree) — EXTRA; Day 46's Maximum
   │  Depth, inverted — a one-child node is NOT a leaf, breaking the
   │  naive `1+min(...)` formula; traced against a right-skewed chain
   ├─ NEW: Kafka Fundamentals — topics as append-only logs, partitions
   │  for parallelism, consumer groups split partition reads; sequential
   │  disk appends drive throughput (same sequential-vs-random-access
   │  principle as Day 40's B-tree discussion, applied to writes)
   └─ todo-api: Spring Kafka producer, TaskCreatedEvent on task
      creation
        ▼
DAY 49 (Sun) — Consolidation, and Tree BFS
   (needs: Queue — Day 4, directly, for BFS; Day 46's TreeNode shape;
   explicitly NOT dependent on Day 48's global-state technique — a
   fresh iterative approach)
   ├─ Self-Check: one Stack problem from this week, attempted cold
   ├─ LC 102 (Binary Tree Level Order Traversal) — new; BFS — capture
   │  `queue.size()` before the inner loop to isolate exactly one
   │  level per pass; DFS-with-level-tracking given as a genuinely
   │  distinct alternative, with a skewed-tree space-complexity
   │  reversal noted (BFS cheaper than DFS on that specific shape)
   ├─ LC 199 (Binary Tree Right Side View) — new; BFS, same scaffold,
   │  keep only the last node visited per level
   ├─ LC 637 (Average of Levels in Binary Tree) — EXTRA; same BFS
   │  scaffold, aggregate (sum ÷ count, long accumulator) instead of
   │  collect
   └─ Week 7 Consolidation: planned-vs-actual count (127 cumulative
      distinct; 97 required-ladder-only, matching the plan's own
      Day 49 claim exactly), diagnostic, what Week 8 assumes

DAY 50 — Binary Search Trees Begin, and Kafka Consumers
   (needs: Day 46's TreeNode shape + universal recursive template,
   NOT re-derived — BST layers one new ordering invariant on top;
   Day 48's Kafka producer, for the consumer side)
   ├─ NEW: Binary Search Trees — ordering invariant; O(log n) claim
   │  flagged as CONDITIONAL on balance, contrasted with Day 17's
   │  TreeMap (Red-Black tree, unconditional)
   ├─ LC 98 (Validate Binary Search Tree) — new; DFS with inherited
   │  (min, max) bounds, not a local-only parent-child check
   ├─ LC 230 (Kth Smallest Element in a BST) — new; inorder traversal
   │  used for its sorted-order property for the first time
   ├─ LC 173 (BST Iterator) — EXTRA; controlled/resumable inorder via
   │  explicit stack, amortized O(1) argument
   ├─ NEW: Kafka consumer groups, partition assignment, auto- vs.
   │  manual-commit trade-off
   └─ `todo-api`: `@KafkaListener` completing the Day 48 produce/
      consume loop

DAY 51 — BST and Construction, and Spring Cloud Config
   (needs: Day 50's ordering invariant, for LCA; Day 46's template +
   Day 4/5's HashMap, for divide-and-conquer construction)
   ├─ LC 235 (Lowest Common Ancestor of a BST) — new; ordering
   │  invariant computes ONE direction, no bounds threading needed
   ├─ NEW: Divide and Conquer, formalized — the divide step requiring
   │  real computed work, first time (vs. every free left/right split
   │  since Day 46); Day 35's LC 23 mention named as the light preview
   ├─ LC 105 (Construct Binary Tree from Preorder and Inorder) — new;
   │  HashMap-accelerated O(n), preorderIndex shared-counter technique
   ├─ LC 106 (Construct Binary Tree from Inorder and Postorder) —
   │  EXTRA; mirrored, two flips named (start position, build order)
   └─ NEW: Spring Profiles (`spring.profiles.active`) — explicitly
      scoped as distinct from, and lighter than, the full Spring
      Cloud Config Server pattern (named, not built)

DAY 52 — Trees Beyond BST: General LCA and Serialization, and WireMock
   (needs: Day 46's combine-both-children template, for general LCA
   without an ordering invariant; Day 47's Same Tree brute force,
   which seeded but did not build out the null-marker idea)
   ├─ LC 236 (Lowest Common Ancestor of a Binary Tree) — new;
   │  postorder combine, contrasted directly with Day 51's BST version
   ├─ LC 297 (Serialize and Deserialize Binary Tree) — new; preorder +
   │  explicit null markers; ambiguity argument for why ONE traversal
   │  suffices here vs. TWO for Day 51's construction problems
   ├─ NO extra today — day already carries a Hard problem + new theory
   └─ NEW: WireMock — contrasted directly with Day 47's TestContainers
      (real dependency for realism vs. fake dependency for control);
      forward-referenced to Resilience4j, still weeks out

DAY 53 — Trees Capstone: The Median Fix, and Divide-and-Conquer Reviewed
   (needs: Day 51's divide-and-conquer formalization; Weeks 4–5's
   Binary Search, both "on the input" and "on the answer" flavors)
   ├─ LC 4 (Median of Two Sorted Arrays) — new; binary search on a
   │  COMPUTED PARTITION POINT — a third Binary Search flavor,
   │  distinct from both of Weeks 4–5's; fulfills the Day 21 promise,
   │  99 days late in the original plan
   ├─ NO extra — explicitly judged a singular technique, not padded
   ├─ Trees Reviewed: BST LCA vs. general LCA, one sentence each,
   │  written out in full (not left as an open exercise)
   └─ Trees CLOSES here: 15/15 required + 5 extra = 20 distinct total

DAY 54 — Heaps Begin, and Frequency Patterns
   (needs: Day 17's PriorityQueue/Comparable-Comparator mention; Day
   26's actual PriorityQueue use (Meeting Rooms II); Day 50's BST
   balance caveat, for direct contrast)
   ├─ NEW: Heaps — full mechanism built for the first time: array
   │  index math, sift-up, sift-down, and WHY the O(log n) bound is
   │  UNCONDITIONAL here where a bare BST's was not (complete-tree
   │  structure enforced by construction, not by insertion order)
   ├─ LC 703 (Kth Largest Element in a Stream) — new; fixed-size
   │  min-heap maintained ACROSS calls; why min- not max-heap,
   │  addressed directly
   ├─ LC 1046 (Last Stone Weight) — new; max-heap, repeatedly
   │  pop-two-combine-one
   ├─ NO extra (Heaps' own opening day) — no Theory/Project block,
   │  matches `Week_08_Revised.md` exactly
   └─ Heaps OPENS here: 2/10 required

DAY 55 — Heaps Continue: Two-Heap and Kth-Order Patterns
   (needs: Day 12's Dutch National Flag partition, for Quickselect;
   Day 35's custom-Comparator-via-anonymous-class, for distance
   ordering; Day 32's unresolved LC 240 staircase mention)
   ├─ LC 215 (Kth Largest Element in an Array) — new; THREE
   │  approaches — sort, heap, Quickselect — Quickselect explicitly
   │  named as Day 12's promised resurfacing; O(n) vs. quicksort's
   │  O(n log n) derived, not just asserted
   ├─ LC 973 (K Closest Points to Origin) — new; max-heap, FIRST
   │  custom-Comparator heap problem this week (squared distance)
   ├─ LC 1167 (Minimum Cost to Connect Sticks) — EXTRA; deferred from
   │  Day 54; min-heap, contrasted with Last Stone Weight via a named
   │  "cost-accounting exchange" argument (smallest-first optimal)
   ├─ LC 378 (Kth Smallest Element in a Sorted Matrix) — EXTRA; heap
   │  approach AND binary-search-on-value + Day 32's staircase count,
   │  resolved for real
   ├─ ⚠️ Plan discrepancy flagged: title says "Spring Cloud Config
   │  Wrap-Up," plan body has no Theory/Project content for it — no
   │  new theory invented; body followed as written
   └─ Heaps: 4/10 required + 2 extra so far

DAY 56 (Sun) — Consolidation, and Heaps: Frequency and Scheduling
   (needs: Day 4/5's HashMap frequency counting, combined explicitly
   with a heap for the first time; Day 54's root-is-always-current-
   min guarantee, reused as a candidate GENERATOR rather than a
   fixed-collection query)
   ├─ Self-Check: one Tree problem from this week, attempted cold
   ├─ LC 347 (Top K Frequent Elements) — new; HashMap + min-heap,
   │  named explicitly; O(n) bucket-sort alternative given, with the
   │  heap-vs-bucket trade-off (streaming vs. fixed-input) stated
   ├─ LC 264 (Ugly Number II) — new; min-heap GENERATES its own
   │  candidates (2x/3x/5x a confirmed-ugly number) rather than
   │  testing arbitrary integers; `long`, not `int`, for overflow
   │  (Day 10/11's habit, reapplied)
   ├─ LC 451 (Sort Characters By Frequency) — EXTRA, marked optional/
   │  light given Sunday's shorter block; direct sibling of LC 347
   └─ Week 8 Consolidation: planned-vs-actual count (145 cumulative
      distinct; 110 required-ladder-only, matching the plan's own
      Day 56 claim exactly), diagnostic, what Week 9 assumes

DAY 57 — Heaps: Scheduling Patterns
   (needs: Day 54's max-heap mechanism; Day 5/56's frequency
   counting, applied here to drive a greedy placement decision
   rather than a query)
   ├─ NEW role: max-heap driving GREEDY SCHEDULING — "act on the
   │  best candidate right now," a genuinely different use than any
   │  of Week 8's four heap roles
   ├─ LC 767 (Reorganize String) — new; hold-the-just-placed-entry-
   │  out-for-one-round; pigeonhole feasibility check, (n+1)/2 not
   │  n/2, boundary-verified
   ├─ LC 621 (Task Scheduler) — new; heap+cooldown-queue simulation
   │  AND the O(1)-space closed-form alternative, both derived, with
   │  the n-as-cooldown vs. n-as-input-size notation collision flagged
   ├─ NO extra (mid-pattern, but no Theory/Project block this day
   │  per `Week_09_Revised.md`, so none added) — matches the plan
   └─ Heaps: 8/10 required

DAY 58 — Heaps Capstone: Two-Heap and Merge Patterns
   (needs: Day 54's heap mechanism; Day 17/26's PriorityQueue +
   Comparator; Day 35's Merge Two Sorted Lists, direct 2-list
   ancestor of today's k-list merge)
   ├─ NEW role: two-heap BALANCE INVARIANT for O(1) running-median
   │  queries over an unbounded stream — traced through 4 sequential
   │  inserts, not just asserted
   ├─ LC 295 (Find Median from Data Stream) — new; push-then-shuffle-
   │  then-rebalance sequence, fully traced
   ├─ NEW role: heap as K-WAY MERGE COORDINATOR — one entry per
   │  active source; divide-and-conquer named as an equal-complexity
   │  alternative mechanism
   ├─ LC 23 (Merge k Sorted Lists) — new; Integer.compare vs. a.val-
   │  b.val subtraction-overflow trap flagged
   ├─ NO extra — full reasoning given: six now-distinct heap roles
   │  across Weeks 8–9 already demonstrated; this is the plan's own
   │  audit-driven 6→10 fix, not a gap left for this series to patch
   └─ Heaps CLOSES here: 10/10 required + 3 extra (Week 8) = 13
      distinct total

DAY 59 — Tries Begin, and the CAP Theorem
   (needs: Day 8's recursion; Day 46's TreeNode node-with-pointers
   shape, generalized from 2 named children to 26 indexed ones; Day
   4's HashMap, for the direct Trie-vs-HashSet trade-off)
   ├─ NEW structure: TrieNode — up to 26 children, isEndOfWord flag;
   │  insert/search/startsWith all O(m), independent of n stored words
   ├─ LC 208 (Implement Trie) — new; why search and startsWith differ
   │  by exactly one check, traced against a "app" prefix-of-"apple"
   │  counterexample
   ├─ LC 677 (Map Sum Pairs) — new; delta-propagation-along-the-path
   │  technique for O(m) prefix-sum queries with correct key-overwrite
   │  semantics; full insert/insert/overwrite trace verified by hand
   ├─ NO extra (Tries' own opening day, matching every prior pattern's
   │  true-opening-day precedent — Trees Day 46, Heaps Day 54)
   ├─ NEW (System Design, independent track): CAP Theorem — precise
   │  formulation (not "pick 2 of 3"); partition tolerance isn't
   │  optional for a real distributed system; PACELC named as an
   │  extension; MongoDB/Cassandra framed as DEFAULTS, not fixed
   │  classifications
   └─ Tries OPENS here: 2/6 core required
        ▼
DAY 60 — Tries Continue, and Consistent Hashing
   (needs: Day 59's Trie walk, extended in two directions without
   re-deriving the base mechanism)
   ├─ NEW: constrained descent — only recurse into a child marked
   │  isEndOfWord (a traversal CONSTRAINT, not just a final check)
   ├─ LC 720 (Longest Word in Dictionary) — new; Trie+DFS, with a
   │  sort+HashSet alternative of comparable complexity also named;
   │  lexicographic tie-break shown to fall out of child-index-order
   │  traversal for free
   ├─ NEW: branching descent — '.' tries all 26 children instead of
   │  following one
   ├─ LC 211 (Design Add and Search Words) — new; complexity CORRECTED
   │  from the plan's shorthand "O(m)" to the precise O(26^k×(m-k))
   │  worst case, with the fast-path/worst-case distinction stated
   │  explicitly rather than left implicit
   ├─ NEW (System Design): Consistent Hashing — TreeMap-backed ring,
   │  ceilingKey() + wraparound via firstEntry(); WHY it remaps only
   │  a proportional key share vs. naive %N's near-total remap; virtual
   │  nodes named as the standard unevenness fix
   └─ Tries: 4/6 core required

DAY 61 — Tries Core Complete, Backtracking Begins, and Replication
Models
   (needs: Day 60's two traversal styles, both reused; Day 8's
   recursion, extended into a THIRD structurally new shape this
   series has built on it)
   ├─ LC 648 (Replace Words) — new; constrained descent applied to
   │  dictionary-root substitution; shortest-match-via-walk-order
   │  argued, not assumed
   ├─ LC 212 (Word Search II) — new, Hard; Trie construction + full
   │  matrix backtracking combined for the first time; visited-via-
   │  overwrite instead of a separate array, argued as the backtrack
   │  undo step itself
   ├─ Tries CLOSES here: 6/6 core required + 0 extra — full two-part
   │  reasoning given (already-comprehensive required set; pattern too
   │  short to ever reach a non-opening reinforcement day)
   ├─ NEW: Backtracking, formalized — choose/explore/un-choose
   │  template; WHY it's distinct from plain tree DFS (shared mutable
   │  state requiring explicit restoration) and from DP (enumeration
   │  of a combinatorial space, not overlapping-subproblem counting)
   ├─ LC 78 (Subsets) — new; include/exclude decision tree, full
   │  8-leaf trace for a 3-element input
   ├─ NO extra (Backtracking's own opening day, same precedent as
   │  Tries Day 59, Trees Day 46, Heaps Day 54)
   ├─ NEW (System Design): Replication Models — single-leader/multi-
   │  leader/leaderless; sync-vs-async durability/latency trade-off;
   │  explicitly reconnected to Day 59's CAP framing (single-leader-
   │  sync leans CP, leaderless-async leans AP — same trade-off, two
   │  angles)
   └─ Backtracking OPENS here: 1/12 required

DAY 62 — Backtracking Continues, and the Flagship Platform
Initializes
   (needs: Day 61's choose/explore/un-choose template, applied to a
   STRUCTURALLY DIFFERENT decision shape than Subsets; Day 34's
   Spring Boot/REST; Day 43-44's Docker, for multi-module reasoning)
   ├─ NEW variant: swap-based choice, position-dependent (not
   │  independent per element like Subsets) — explicitly flagged as
   │  NOT a repeat of Day 61's shape
   ├─ LC 46 (Permutations) — new; full 6-permutation trace for a
   │  3-element input, showing every "swap back" as load-bearing, not
   │  cosmetic cleanup
   ├─ NO extra (still within Backtracking's opening arc; Week 10's
   │  already-dense required ladder makes padding unnecessary — full
   │  reasoning deferred to Day 63's closing note)
   ├─ NEW: Spring AOP — Aspect/Pointcut/Advice; proxy mechanism (JDK
   │  dynamic proxy vs. CGLIB); the self-invocation limitation, named
   │  as a real, commonly-tested gotcha, not a footnote
   ├─ NEW: Multi-module Maven — parent `pom.xml` (packaging=pom,
   │  <modules>, <dependencyManagement>) vs. child POMs; the exact
   │  <dependencies>-vs-<dependencyManagement> distinction flagged as
   │  a common, confusing-to-debug mistake
   └─ `scalable-ecommerce-platform` initializes — 4 independently-
      compiling modules (Product/Order/Payment/Notification), 5 weeks
      ahead of the original plan's equivalent project; `todo-api`
      frozen, feature-complete

DAY 63 (Sun) — Consolidation, and Backtracking Continues
   (needs: Day 62's swap mechanism, reused; Day 61's forward-index
   idea previewed via Combinations' never-revisit rule)
   ├─ Self-Check: one Heap or Trie problem, attempted cold
   ├─ NEW: forward-index recursion — never revisit an earlier index;
   │  shown to be the ENTIRE duplicate-avoidance mechanism for
   │  Combinations, with nothing extra needed, and explicitly
   │  contrasted with why it would be WRONG for Permutations
   ├─ LC 77 (Combinations) — new; full 6-combination trace, C(4,2)=6
   │  confirmed; remaining-elements pruning named as extension
   ├─ LC 47 (Permutations II) — new; duplicate-handling layered on
   │  Day 62's swap mechanism — full [1,1,2] trace PROVING !used[i-1]
   │  correct and used[i-1] wrong, both skip points traced exhaustively
   ├─ NO extra — full three-part reasoning given: opening-arc timing,
   │  Week 10's already-dense ladder, AND a specific caught collision
   │  (LC 17, Letter Combinations of a Phone Number, confirmed to be
   │  Week 10 Day 65's required problem before being ruled out)
   └─ Week 9 Consolidation: planned-vs-actual (14/14 required, 0
      extra — first week in the series with zero extra practice
      added, across three independently-justified decisions), 159
      cumulative distinct, diagnostic, what Week 10 assumes

DAY 64 — Combination Sum, Its Duplicate-Handling Twin, and
Resilience4j
   (needs: Day 61's template; Day 63's forward-index rule, extended
   with one new wrinkle)
   ├─ LC 39 (Combination Sum) — new; forward-index + repetition
   │  allowed — recurse with i, not i+1; the one-line delta from Day
   │  63 named explicitly, full [2,3,6,7]/target-7 trace
   ├─ LC 40 (Combination Sum II) — new; duplicate-skip TRANSLATED
   │  into forward-index form (i > start, not Day 63's !used[i-1]) —
   │  full [1,1,2]/target-4 trace PROVING i>start correct and the
   │  i>0 analog wrong, run and confirmed by direct execution
   ├─ NO extra — full reasoning given: Backtracking closes this week
   │  at a dense, comprehensive 12-problem ladder already spanning
   │  every canonical variant; padding it trades pacing realism for
   │  repetition with no corresponding gap (mirrors Tries' Day 61
   │  precedent more than it mirrors a genuinely thin pattern)
   ├─ NEW: Resilience4j Circuit Breaker — CLOSED/OPEN/HALF_OPEN;
   │  cascading failure via thread-pool exhaustion named as the
   │  specific mechanism it prevents, not just "handles failures";
   │  proxy-based — explicitly the SAME mechanism as Day 62's Spring
   │  AOP, including the identical self-invocation bypass gotcha
   └─ Backtracking continues: 6/12 required

DAY 65 — Backtracking: Phone Letters and Parentheses, and Feign
   (needs: Day 61's template, applied to a THIRD and FOURTH choice
   model this series has now used — external per-position lookup,
   and counter-gated validity)
   ├─ LC 17 (Letter Combinations of a Phone Number) — new; resolves
   │  Week 9's flagged near-miss, confirmed exactly once in the
   │  series, here; empty-input → [] (not [""]) edge case required
   ├─ LC 22 (Generate Parentheses) — new; closeCount<openCount
   │  PROVEN sufficient via a two-part invariant argument (balance
   │  never negative; length forces open=close=n), not asserted;
   │  the closeCount<n bug named as this problem's i>0-analog
   ├─ NO extra — same week-level reasoning as Day 64
   ├─ NEW: Feign Clients — declarative interface, proxy-generated
   │  HTTP client via reflection (same mechanism family as Day 34's
   │  @RestController); fallback shares Resilience4j's "same
   │  signature + trailing param" convention but at CLASS scope, not
   │  method scope; calls the platform's REAL Product module, not a
   │  mock — tied to Day 52's TestContainers-vs-WireMock realism
   │  argument
   └─ Backtracking continues: 8/12 required

DAY 66 — Backtracking on Grids and Palindromes, and the Gateway
   (needs: Week 9 Day 61's Word Search II overwrite-restore
   mechanism, CITED not re-taught; Day 63's forward-index, extended
   over string positions instead of array indices)
   ├─ LC 79 (Word Search) — RECAP-HEAVY: explicitly the single-word
   │  subset of Week 9 Day 61's Word Search II; near-zero new
   │  mechanism, only the no-Trie complexity delta (no shared-prefix
   │  pruning across multiple words)
   ├─ LC 131 (Palindrome Partitioning) — new; forward-index-over-
   │  string-positions + a palindrome-validity gate; DP-based
   │  isPalindrome memoization named as a known optimization and
   │  explicitly DEFERRED — not yet taught, same pattern as earlier
   │  DP deferrals
   ├─ NO extra — same week-level reasoning
   ├─ NEW: Spring Cloud Gateway — single external entry point;
   │  centralizes cross-cutting concerns (routing today; auth Day
   │  67; rate limiting Day 68) that would otherwise be duplicated
   │  and drift across every module independently
   └─ Backtracking continues: 10/12 required

DAY 67 — Backtracking Capstone (Closes 12/12), and JWT at the
Gateway
   (needs: Day 64's i>start translation, cited DIRECTLY as the
   mechanical ancestor — not Week 9's Permutations II swap-based
   version; Day 4's HashSet)
   ├─ LC 90 (Subsets II) — new; Day 61's include/exclude shape +
   │  Day 64's EXACT skip condition reused verbatim, not re-derived
   ├─ LC 51 (N-Queens) — new, Hard; row-by-row placement eliminates
   │  the row constraint by construction; row±col diagonal-identity
   │  PROVEN via the constant-difference/constant-sum argument, not
   │  stated; solution counts verified by direct execution (n=4→2,
   │  n=8→92, matching known values)
   ├─ Backtracking CLOSES here: 12/12 required + 0 extra = 12
   │  distinct across Weeks 9–10 — full 6-choice-model classification
   │  table given (include/exclude, swap-based, forward-index bare/
   │  with-repetition/with-duplicate-skip, string-building via
   │  lookup, string-building via counter, grid-DFS, row-by-row with
   │  derived constraints)
   ├─ NEW: JWT Authentication at the Gateway — self-contained signed
   │  tokens, no DB round-trip to validate; explicitly distinguished
   │  from OAuth 2.0 (a token FORMAT vs. an authorization PROTOCOL —
   │  complementary, frequently conflated, not synonymous)
   └─ Backtracking: CLOSED (12/12 required, 0 extra, Weeks 9–10)

DAY 68 — Graphs Begin, and Rate Limiting
   (needs: Day 8's recursion, Day 4's Queue, Day 46/49's tree
   traversal — CITED DIRECTLY as the mechanism transferring
   UNCHANGED, plus exactly one new requirement)
   ├─ NEW: Concept Card — Graphs (BFS/DFS) — adjacency list vs.
   │  matrix trade-off (O(V+E) vs. unconditional O(V²)); "a grid is
   │  a graph in disguise"; explicit `visited` requirement PROVEN
   │  necessary via the tree-has-exactly-one-path argument, not just
   │  asserted; BFS-vs-DFS interview signal (shortest/minimum/fewest
   │  → BFS; does-a-path-exist/connectivity → either, DFS simpler)
   ├─ LC 200 (Number of Islands) — new; matrix DFS PROVEN to be
   │  plain graph traversal, NOT backtracking — no un-choose step,
   │  because marking is permanent with nothing ever needing
   │  restoration; explicit contrast drawn against Day 66's Word
   │  Search's superficially-similar recursive shape
   ├─ LC 695 (Max Area of Island) — new; DFS-returns-a-value shape,
   │  cited directly to Day 46's Maximum Depth of Binary Tree — same
   │  "1 + combine(neighbor results)" recursion, sum instead of max,
   │  up to 4 neighbors instead of a tree's fixed 2
   ├─ NO extra (Graphs' own true opening day — same precedent as
   │  every prior pattern's genuine opening day: Trees D46, Heaps
   │  D54, Tries D59, Backtracking D61)
   ├─ NEW: Token Bucket rate limiting — TWO independent guarantees
   │  named separately (bounded burst via capacity; bounded average
   │  rate via refill); naive fixed-window counter's boundary-
   │  clustering failure named as the specific gap this closes;
   │  `synchronized` requirement cited directly to Day 37's Counter
   │  corruption demonstration — a correctness bug, not just a
   │  performance concern
   └─ Graphs OPENS here: 2/12 required (of a 12-problem target
      spanning Weeks 10–11)

DAY 69 — Graph Traversal Continues, and Kafka Schema Registry
   (needs: Day 68's visited set, generalized from a boolean to
   carrying real information; Day 4/5's HashMap)
   ├─ LC 133 (Clone Graph) — new; visited generalizes to
   │  HashMap<Node,Node>; a naive unguarded DFS PROVEN to genuinely
   │  infinite-loop on a 2-node cycle (walked through concretely,
   │  not just asserted); map-populated-before-recursing ordering
   │  justified by showing the reversed order still loops
   ├─ LC 785 (Is Graph Bipartite?) — new; visited generalizes to a
   │  2-color array; the odd-cycle characterization named as the
   │  underlying theory (bipartite iff no odd cycle), not just "run
   │  BFS with colors"; disconnected-components outer loop justified
   │  via direct parallel to Day 68's Number of Islands outer scan
   ├─ ONE extra added (Graphs past its own opening day): LC 802
   │  (Find Eventual Safe States) — checked against
   │  `Week_11_Revised.md` and the problem table below, confirmed
   │  clean; 3-state coloring (unvisited/visiting/safe) explicitly
   │  framed as today's 2-coloring generalized for cycle detection
   │  in a DIRECTED graph; flagged as a direct preview of Week 11's
   │  topological-sort cycle detection (Course Schedule)
   ├─ NEW: Kafka Schema Registry — versioned event contracts between
   │  independently-evolving services; the silent-drift failure mode
   │  named concretely (a field rename breaking a downstream
   │  consumer with no compile-time signal, surfacing far from its
   │  actual cause); Avro chosen specifically because it requires a
   │  registrable schema, unlike plain JSON; the JPA entity (Day 36)
   │  and the Avro event schema deliberately kept as TWO separate
   │  things — persistence detail vs. public contract
   └─ Graphs continues: 4/12 required + 1 extra

DAY 70 (Sun) — Multi-Source BFS, a DAG's Missing Visited Set, and
Week 10 Consolidation
   (needs: Day 49's queue.size() level-isolation trick, extended to
   many simultaneous sources; Day 68's visited rule, tested directly
   against its own boundary condition)
   ├─ Self-Check: one Backtracking problem, attempted cold
   ├─ NEW: multi-source BFS — seed the queue with EVERY starting
   │  node before the first iteration; "minutes elapsed" shown to BE
   │  Day 49's level counter, unchanged, just counted from many roots
   │  at once instead of one
   ├─ LC 994 (Rotting Oranges) — new; full multi-source BFS trace,
   │  two independent rot sources converging, confirmed by direct
   │  execution (4 minutes for the traced example)
   ├─ LC 797 (All Paths From Source to Target) — new; the FIRST
   │  problem in the series where `visited` is correctly ABSENT —
   │  Day 68's rule shown to be conditioned on cycles being
   │  *possible*, not unconditional, and this problem's DAG guarantee
   │  rules cycles out entirely; a SECOND, independent reason given
   │  (a global visited would incorrectly block legitimate multi-path
   │  revisits of the same node); explicitly reclassified as genuine
   │  backtracking (shared mutable path, real un-choose step) in
   │  direct contrast to Day 68's Number of Islands — two graph
   │  problems, two opposite correct classifications, both justified
   ├─ ONE extra added: LC 542 (01 Matrix) — checked against
   │  `Week_11_Revised.md`, confirmed clean; named explicitly as
   │  Rotting Oranges' exact multi-source shape minus the shared
   │  counter, computing each cell's own distance instead
   ├─ NEW: Saga Choreography — local per-service transactions chained
   │  by events, no central coordinator; compensating transaction
   │  explicitly distinguished from database rollback (the earlier
   │  local transaction already committed — nothing left to roll
   │  back); 2PC's availability cost tied directly back to Day 64's
   │  cascading-failure/thread-exhaustion argument
   └─ Week 10 Consolidation: planned-vs-actual (14/14 required + 2
      extra = 16 distinct), 175 cumulative distinct, diagnostic, what
      Week 11 assumes

DAY 71 (Mon) — Topological Sort, and Networking Fundamentals I
├─ NEW: Topological Sort — Kahn's Algorithm (BFS + in-degree)
│  AND DFS 3-state coloring, the latter a direct escalation of
│  Day 69's Find Eventual Safe States extension, cited not re-derived
│  ├─ LC 207 (Course Schedule) — required; existence check
│  └─ LC 210 (Course Schedule II) — required; construction,
│     identical algorithm, wider output
├─ Graphs BFS/DFS: 8/12 → 10/12 required
└─ NEW theory thread: TCP/UDP, DNS resolution hierarchy —
   independent of the DSA track, continues Day 72

DAY 72 (Tue) — Multi-Source Traversal, Reversed: a New Role
├─ NEW role for multi-source seeding: REACHABILITY, distinguished
│  explicitly from Day 70's DISTANCE role — doesn't require BFS
│  specifically, since there's no level-order guarantee needed
│  ├─ LC 130 (Surrounded Regions) — required; DFS/BFS from every
│  │  border 'O', two-phase mark-then-sweep
│  └─ LC 417 (Pacific Atlantic Water Flow) — required; REVERSED
│     comparison (expand on height ≥, not ≤) — proven, not
│     asserted, via the reversed-edge-is-the-same-edge argument
├─ Graphs BFS/DFS: 10/12 → 12/12 required — CLOSES
└─ Networking Fundamentals II — HTTP/HTTPS (built on Day 71's TCP),
   Load Balancers (L4 vs. L7) — the Gateway (Day 66) identified
   explicitly as an L7 balancer by definition, not analogy

DAY 73 (Wed) — Graphs Capstone: the Pattern's Own Edge Cases
├─ LC 127 (Word Ladder, Hard) — required; BFS shortest path on an
│  IMPLICIT graph (adjacency generated per-word, never materialized);
│  explicitly SINGLE-source, distinguished from Days 70/72
├─ NEW: Floyd-Warshall — all-pairs shortest path, NOT a traversal;
│  a DP-flavored preview (same spirit as Week 3's Kadane's, formal
│  DP not yet named); `k` proven outermost-loop-required via the
│  layer-completion argument
│  └─ LC 1334 (Find the City...) — required; tie-break via `<=`
├─ Graphs BFS/DFS: CLOSES at 12/12 required — up from 9 in the
│  original plan; 14 distinct combined with Week 10's 2 extras
└─ 1-hr reflection: BFS vs. DFS vs. neither, all 6 of this week's
   Graph problems classified precisely (ordering/cycle-detection is
   its own third category, not a BFS-or-DFS binary)

DAY 74 (Thu) — Union-Find Begins (New Structure, from Zero)
├─ NEW STRUCTURE: Union-Find (Disjoint Set Union)
│  ├─ Naive: O(n) worst case — proven via a fixed-direction-union
│  │  chain example
│  ├─ Union by rank alone: O(log n) — proven via the rank-doubling
│  │  induction (a rank-r tree has ≥2^r nodes)
│  ├─ Path compression: flattens on every `find`, illustrated via
│  │  a concrete chain-collapse example
│  └─ Combined: O(α(n)) amortized — practically constant
│  ├─ LC 684 (Redundant Connection) — required
│  └─ LC 547 (Number of Provinces) — required; Union-Find AND DFS
│     shown side by side, trade-off made concrete (static matrix
│     erases Union-Find's usual incremental-query advantage)
├─ Union-Find: 0/7 → 2/7 required — opens; ZERO extras today,
│  per this series' own true-opening-day precedent (Trees Day 46,
│  Heaps Day 54, Tries Day 59, Backtracking Day 61, Graphs Day 68)
└─ NEW theory thread: AWS Networking — VPC, public/private subnets
   (defined by ROUTE TABLE, not location), IGW vs. NAT Gateway
   (bidirectional vs. outbound-only), Security Groups (stateful)

DAY 75 (Fri) — Union-Find: Constraint Problems, Not Direct Edges
├─ LC 990 (Satisfiability of Equality Equations) — required;
│  two-PASS structure (== fully unioned before any != checked) —
│  proven necessary via a transitive-chain contradiction example;
│  fixed-size-26 structure — O(1) per op, not just O(α(n))
├─ LC 947 (Most Stones Removed with Same Row or Column) — required;
│  unions ROWS and COLUMNS as nodes, not stone pairs; `total −
│  components` formula PROVEN via a spanning-tree leaf-removal
│  argument, not asserted
├─ Union-Find: 2/7 → 4/7 required
├─ EXTENSION (optional): LC 1319 (Number of Operations to Make
│  Network Connected) — the exact problem Week 10's Day 69 note
│  deferred here; feasibility (edges ≥ n-1) algebraically
│  guarantees enough redundant cables to bridge every component,
│  derived not assumed; answer = components − 1, reusing the
│  `UnionFind.count` field directly
└─ AWS IAM (Roles vs. Users — temporary-assumed vs. persistent;
   least privilege) and S3 (object, immutable, HTTP) vs. EBS
   (block, mounted filesystem, single-instance-attached)

DAY 76 (Sat) — Union-Find: the "Union the Abstraction" Instinct,
Formalized Across Two More Surfaces
├─ LC 1202 (Smallest String With Swaps) — required; unions INDEX
│  POSITIONS; full-permutation-within-a-component PROVEN via a
│  path-based character-movement construction, citing the connected-
│  graph-generates-the-symmetric-group fact
├─ LC 721 (Accounts Merge) — required; unions EMAILS, not accounts
│  — the transitive-chain justification stated explicitly (A–B–C
│  sharing no direct pair); final grouping cites Day 5's canonical-
│  key HashMap pattern directly, same shape, different key source
├─ Union-Find: 4/7 → 6/7 required
└─ Observability: Prometheus (pull-based scraping — contrasted with
   push, including the Pushgateway exception), Micrometer (a facade,
   explicitly not a monitoring system itself), `/actuator/prometheus`

DAY 77 (Sun) — Union-Find Capstone, and Minimum Spanning Trees
├─ LC 261 (Graph Valid Tree, Premium in some regions) — required;
│  `n-1` edges + acyclic ⟹ connected, PROVEN via the forest-edge-
│  count argument (n−c edges for c components), not asserted —
│  no separate connectivity scan needed
├─ Union-Find: CLOSES at 7/7 required — up from 3–4 in the original
│  plan; 8 distinct combined with Day 75's one extra
├─ NEW: Minimum Spanning Trees — Kruskal's (reuses `UnionFind`
│  AS-IS, unmodified, as a live cycle-detection filter) and Prim's
│  (reuses the `PriorityQueue` machinery from Days 17/26/54);
│  correctness via the Cut Property, a full exchange argument in
│  the same style Week 4's Greedy Algorithms established
└─ Week 11 Consolidation: planned-vs-actual (13/13 required + 1
   extra = 14 distinct), 189 cumulative distinct, diagnostic, the
   150-vs-151 plan-scorecard discrepancy traced and flagged (same
   Week-9-origin drift this map already caught once), what
   Week 12 assumes

DAY 78 (Mon) — Dijkstra's Algorithm Begins, and Sharding Strategies
├─ LC 743 (Network Delay Time) — required; textbook Dijkstra, min-
│  heap of (distance,node), non-negative-weight greedy proof stated
│  as a domination argument, same shape as the Cut Property (Day 77)
├─ LC 1514 (Path with Maximum Probability) — required; max-heap +
│  multiplicative relaxation, the FLIP proving Dijkstra's is a
│  shape, not a fixed recipe
├─ Dijkstra's Algorithm: OPENS at 2/5 required (ladder pre-expanded
│  from 3→5 by this week's own plan revision)
└─ Sharding Strategies — range-based vs. hash-based, the celebrity
   problem; hash-based sharding tied directly back to Consistent
   Hashing (Day 60) as the naive-%N problem it was built to fix

DAY 79 (Tue) — Dijkstra's: Constrained Variants, and 2PC vs. Saga
├─ LC 787 (Cheapest Flights Within K Stops) — required; a genuine
│  Dijkstra failure case, proven via counterexample, not asserted —
│  fixed via NEW: Bellman-Ford (K-round relaxation against a frozen
│  snapshot, induction-proved), primary approach; modified-Dijkstra
│  with (node,stopsUsed) state given as the alternative, its exact
│  trap (a rejected over-budget state must not poison future
│  revisits) explained rather than only avoided
├─ LC 1631 (Path With Minimum Effort) — required; minimax Dijkstra,
│  EDGE-weighted; alt. approach cites Binary Search on the Answer
│  (Wk5) + Union-Find (Wk11) together for the first time
├─ Dijkstra's Algorithm: 2/5 → 4/5 required
└─ NEW: Two-Phase Commit — full mechanism (prepare/vote, commit-or-
   abort, the coordinator-crash blocking window tied directly to
   Day 64's cascading-failure argument), contrasted with Saga —
   Choreography/Orchestration RECAPPED from Week 10, Day 70, not
   re-taught

DAY 80 (Wed) — Dijkstra's Capstone, and Kafka Streams
├─ LC 778 (Swim in Rising Water) — required, Hard; minimax Dijkstra
│  again, NODE-weighted this time (destination's own elevation, not
│  an edge difference) — the precise distinction from Day 79 stated
│  explicitly; alt. approach reuses the same Binary-Search+Union-
│  Find pairing a second time
├─ Dijkstra's Algorithm: CLOSES at 5/5 required — up from 3 in the
│  original plan, both thinnest gaps from the original audit now
│  closed; ZERO extra practice added anywhere in the pattern's run,
│  reasoning stated explicitly (ladder already deliberately
│  expanded; five problems already span every relaxation shape the
│  pattern tests)
└─ NEW: Kafka Streams — KStream (event-by-event) vs. KTable (latest-
   value-per-key changelog), the stream-table duality explained as
   the actual mechanism, not just terminology; built directly on
   Kafka Fundamentals (Day 48), Consumers (Day 50), Schema Registry
   (Day 69)

DAY 81 (Thu) — Dynamic Programming Begins
├─ LC 70 (Climbing Stairs) — required; fib(n+1) wearing a different
│  name, all four tiers shown (brute → memo → tabulation → O(1)
│  space) to establish the progression explicitly
├─ LC 746 (Min Cost Climbing Stairs) — required; same shape, cost-
│  minimizing instead of way-counting; dp[i]'s exact meaning (cost
│  through LEAVING i, not arriving) verified against a 10-element
│  trace, not assumed
├─ NEW: Dynamic Programming, formalized — fib(5)'s call tree RE-
│  TRACED (fib(2) recomputed 3×, fib(1) 5×, fib(0) 3×, 15 calls
│  total), proving the motivation rather than asserting it; optimal
│  substructure + overlapping subproblems defined, with Divide-and-
│  Conquer (Wk8 D51) cited as optimal-substructure-WITHOUT-overlap;
│  greedy's failure cited from Wk4 D23's 0/1 Knapsack counter-
│  example; informally previewed twice already without the name —
│  Kadane's (Wk3 D21), Floyd-Warshall (Wk11 D73) — both cited
│  directly per this map's own Week-11 handoff note
├─ Dynamic Programming: OPENS at 2/33 required
└─ (Theory Block folded into the Concept Card + Project Block —
   fib(n) three ways, timed, IS the DPFoundations exercise)

DAY 82 (Fri) — 1D DP: House Robber Family
├─ LC 198 (House Robber) — required; take-or-skip decision replaces
│  Day 81's always-add, proven via disjoint-exhaustive case argument
├─ LC 213 (House Robber II) — required; circular-to-linear reduction
│  PROVEN (every valid plan excludes ≥1 boundary house); the n=1
│  edge case where the general reduction silently breaks, isolated
│  and explained, not just patched
├─ LC 740 (Delete and Earn) — EXTRA; House Robber via a bucket-by-
│  VALUE transform, the transform's validity argued (same-value
│  picks never conflict, only adjacent DIFFERENT values do), not
│  just applied; checked clean against the full problem table and
│  Week_13_Revised.md
├─ Dynamic Programming: 2/33 → 4/33 required + 1 extra = 5 distinct
└─ (extras land here, one day past DP's opening — same precedent as
   every prior genuinely-new pattern in the series)

DAY 83 (Sat) — 1D DP: Decoding and Products
├─ LC 91 (Decode Ways) — required; validity-GATED lookback (a new
│  shape — earlier problems always looked back a fixed distance
│  unconditionally); the leading-zero subtlety resolved via the
│  numeric range check alone, no dedicated zero-detection rule
├─ ⚠️ LC 152 (Maximum Product Subarray) — listed as required by
│  Week_12_Revised.md, but ALREADY IN THIS TABLE — Week 4, Day 22,
│  Extra Practice. RECAPPED, not re-taught — see the Week 12 Overlap
│  section below. The first required-problem-turns-out-to-be-an-
│  earlier-extra catch in the series, the mirror image of every
│  earlier overlap catch (which ran the other direction: an extra
│  turning out to collide with a later week's required list)
├─ Dynamic Programming: 4/33 → 5/33 required (+1 recap slot) + 1
│  extra = 6 distinct new (LC 152 not counted twice)
└─ No substitute added for the recapped slot — only one of two
   required problems overlapped, not the whole day's block

DAY 84 (Sun) — Consolidation, and Unbounded Knapsack Begins
├─ LC 139 (Word Break) — required; needs HashSet (Day 4) for O(1)
│  dictionary lookups; the commonly-cited O(n²) bound corrected to
│  a rigorous O(n³) once Java's actual substring()/hashCode() cost
│  is accounted for, with the Trie-based fix named as extension
├─ LC 322 (Coin Change) — required; UNBOUNDED reuse, the first of
│  the series — proven to fall directly out of not tracking which
│  coins built a sub-answer, not a rule bolted on; sentinel value
│  (amount+1, not MAX_VALUE) chosen specifically to avoid overflow
│  on the `+1` relaxation; loop-order independence for MINIMIZING
│  explained, with a flag (not a teach) that COUNTING will need a
│  fixed order next week
├─ Dynamic Programming: 5/33 → 7/33 required + 1 extra = 8 distinct
│  new this week, continues into Week 13
└─ Week 12 Consolidation: planned-vs-actual (13/13 required + 1
   extra = 13 new distinct, +1 recap not double-counted), 202
   cumulative distinct, diagnostic, the 163-vs-164 plan-scorecard
   discrepancy traced and flagged (same Week-9-origin drift this
   map has now caught three times), what Week 13 assumes

DAY 85 — Coin Change II Resolves the Loop-Order Flag; LIS Two Ways;
0/1 Knapsack Begins
├─ LC 518 (Coin Change II) — required; resolves Day 84's flagged
│  loop-order question for COUNTING combinations — coins outer,
│  amount inner, proven (not asserted) via a concrete overcount at
│  amount=5,coins=[1,2,5]: wrong order gives 9, correct gives 4
├─ LC 377 (Combination Sum IV) — extra; same recurrence shape, loop
│  order FLIPPED (amount outer) because it counts permutations, not
│  combinations — the day's first loop-order contrast, checked clean
│  against the full inventory and against Week 14's required list
│  before adding
├─ LC 300 (Longest Increasing Subsequence) — required; BOTH
│  approaches taught in full: O(n²) dp[i]=LIS ending exactly at i (a
│  new lookback shape — value-gated across every earlier index, not
│  fixed-distance or dictionary-gated); O(n log n) patience-sorting
│  tails[] + binary search, with the strictly-increasing invariant
│  on tails[] proven by induction (not assumed), and the common
│  misconception that tails[] IS the actual subsequence explicitly
│  corrected via a concrete counterexample (nums=[3,4,5,1])
├─ LC 673 (Number of LIS) — extra, marked skippable/extension;
│  direct extension of the O(n²) dp[] shape, tracking a parallel
│  count[] array
├─ NEW CONCEPT — 0/1 Knapsack, built fully fresh (not derived from
│  Unbounded Knapsack by analogy) — named-only since Week 4 Day 23's
│  greedy-failure example, actually solved for the first time today;
│  1D space-optimization requires the capacity loop to run
│  DECREASING, proven via a minimal counterexample (nums=[3],
│  target=6: increasing order incorrectly reuses the single 3 twice)
└─ LC 416 (Partition Equal Subset Sum) — required; reduces two-sided
   partition to single-subset reachability at totalSum/2

DAY 86 — 1D DP Closes at 14/14; Word Break II Fuses DP and Backtracking
├─ LC 494 (Target Sum) — required; two-equation reduction
│  (P-N=target, P+N=totalSum → P=(target+totalSum)/2) derived, not
│  just applied; two distinct unsolvable-input guards (parity, and
│  |target|>totalSum) shown to catch different failure modes
├─ LC 279 (Perfect Squares) — required; Coin Change's exact
│  Unbounded Knapsack shape, "coins" = perfect squares ≤ n;
│  Lagrange's four-square theorem named as a non-DP extension, not
│  built
├─ LC 140 (Word Break II) — required; the first problem this series
│  combines two patterns explicitly DISTINGUISHED from each other at
│  Week 9 Day 61 (DP vs. Backtracking) — Word Break's dp[] boolean
│  array (Day 84) reused UNCHANGED as an O(1) pruning oracle gating
│  fresh backtracking; complexity stated honestly as O(n²) traversal
│  backbone PLUS output size, not a false polynomial bound
├─ 🔗 backward reference, not a new problem: connects to Palindrome
│  Partitioning (Week 10 Day 66) as the same backtracking shape with
│  a different gate condition — the isPalindrome DP-table deferral
│  named at Week 10 Day 66 is flagged here as still open, to be paid
│  off Day 88
├─ 1D DP CLOSES: 14/14 (up from the original plan's 12)
└─ 🔑 the week's complete loop-order table finalized: 4 rules across
   Coin Change / Coin Change II / Combination Sum IV / 0/1 Knapsack,
   two independent underlying reasons (bounded-vs-unbounded;
   combinations-vs-permutations) — first time this series has needed
   to hold two unrelated loop-order rules apart in the same sitting

DAY 87 — Grid DP Opens and Closes Same Day
├─ NEW SYNTAX — declaring/populating a 2D array from scratch
│  (`new int[m][n]`), distinguished explicitly from indexing into a
│  GIVEN matrix (Week 5 Day 32, LC 74) — the latter was already
│  familiar, the former was not
├─ NEW CONCEPT — Grid (2D) DP formalized: dp[i][j] from spatial
│  neighbors (up/left/diagonal), row-major traversal order justified
│  (every dependency is always already computed under that sweep)
├─ LC 62 (Unique Paths), LC 63 (Unique Paths II), LC 64 (Minimum
│  Path Sum) — required; sum-vs-min neighbor-combination contrast
│  explained (counting adds, optimizing takes the better one);
│  Unique Paths II's obstacle-propagation-past-the-obstacle-cell
│  edge case flagged as a common mistake, not just handled silently
├─ LC 221 (Maximal Square) — required; the week's most demanding
│  proof — BOTH directions of dp[i][j]=min(3 neighbors)+1 derived
│  (necessity via containment of shifted (k-1)-squares; sufficiency
│  via the three shifted m-squares jointly covering the full
│  (m+1)-region with no gap), verified independently against 200
│  random-grid brute-force trials before writing
└─ LC 174 (Dungeon Game) — extra, marked extension; the DP direction
   REVERSES (bottom-right to top-left) — a deliberate counterexample
   to the "DP always sweeps forward" assumption every required
   problem today reinforced

DAY 88 — String DP Begins
├─ NEW CONCEPT — String DP formalized: dp[i][j] relates a PREFIX of
│  one string to a prefix of another (or a string to itself);
│  (m+1)x(n+1) table convention over 0-indexed strings justified
│  specifically as avoiding negative-index checks, not just "by
│  convention"
├─ LC 1143 (Longest Common Subsequence) — required;
│  match→diagonal+1 unconditional, mismatch→max(up,left) — proven,
│  not just stated
├─ LC 583 (Delete Operation for Two Strings) — extra, light; direct
│  LCS application, near-zero marginal teaching cost
├─ LC 72 (Edit Distance) — required; THREE-neighbor branching
│  (insert/delete/replace), each operation mapped to its exact
│  source cell with justification, not just memorized
└─ LC 5 (Longest Palindromic Substring) — required; Expand Around
   Center, 2n-1 centers justified (n odd-centers + n-1 even-gap-
   centers); DP-table alternative named as the thing that will pay
   off Week 10 Day 66's deferred isPalindrome optimization, not
   built yet today

DAY 89 — Counting Palindromes, the LCS-Reverse Proof, Three-Way
Interleaving
├─ LC 647 (Palindromic Substrings) — required; yesterday's exact
│  expand-around-center code, aggregation changed from track-max to
│  count
├─ LC 516 (Longest Palindromic Subsequence) — required; LPS(s) =
│  LCS(s,reverse(s)) — genuinely proven both directions, not cited:
│  "≥" via a general argument (any palindromic subsequence is
│  automatically a common subsequence of s and reverse(s)), "≤" via
│  a concrete backtrack trace (s="bbbab") showing the alignment is
│  structurally forced into a both-ends-inward (palindromic) shape
├─ LC 97 (Interleaving String) — required; (m+1)x(n+1) table
│  extended to a THIRD string s3 with no third dimension, since i+j
│  always determines the exact position in s3 — proven, not assumed;
│  base-case forward-failure-propagation flagged as the same shape
│  as Unique Paths II's obstacle rule (Day 87)
└─ 🔗 sets up tomorrow's isPalindrome-table payoff explicitly

DAY 90 — String DP Closes at 9/9; Two Pattern-Matching Hards Contrasted
├─ LC 115 (Distinct Subsequences) — required; match→ADD two terms
│  (not max), justified against LCS's max-of-two by the count-vs-
│  optimize distinction — the day's sharpest "don't pattern-match
│  too closely to the last problem" lesson
├─ LC 44 (Wildcard Matching) and LC 10 (Regular Expression Matching)
│  — required; the '*' semantic difference (standalone-any-sequence
│  vs. modifier-on-preceding-character) stated as the ONE fact that
│  prevents cross-contaminating the two recurrences; Regex's
│  dp[0][j]=dp[0][j-2] base case explained precisely (why it isn't
│  all-false)
├─ STRING DP CLOSES: 9/9 (Regular Expression Matching upgraded from
│  optional extension to fully scheduled, per Week 13's own plan
│  revision)
└─ LC 132 (Palindrome Partitioning II) — extra, extension; PAYS OFF
   the isPalindrome DP-table deferral opened Week 10 Day 66 and
   built as an alternate approach Day 88 — same DP-array-as-pruning-
   oracle idea Word Break II used Day 86, applied to a palindrome-
   validity gate instead of dictionary membership; dp[0]=-1
   convention explained, not just stated; explicitly flagged as the
   shape Interval DP formalizes tomorrow

DAY 91 (Sun) — Interval DP Completes; Leave Week 2 Ends
├─ NEW CONCEPT — Interval DP formalized: dp[i][j] over subrange
│  [i,j], MUST be filled by increasing interval LENGTH (not
│  row-major) — the first genuinely new traversal-order rule since
│  Grid DP's row-major sweep, proven necessary via a concrete case a
│  row-major sweep would read uncomputed
├─ LC 312 (Burst Balloons) — required; the week's hardest
│  reformulation — "burst LAST, not first" proven to be what makes
│  the two sub-ranges independent (a last-burst balloon's neighbors
│  are deterministically the fixed boundaries; a first-burst
│  balloon's remaining groups still interact across the boundary
│  depending on timing)
├─ LC 1312 (Minimum Insertion Steps to Make a String Palindrome) —
│  required; reduces directly to Day 89's LPS, justified
│  (sufficiency + necessity sketch), not just applied
├─ LC 486 (Predict the Winner) — extra; added specifically because
│  Interval DP's required ladder (2 problems, one of which is a pure
│  reduction) left only ONE genuinely fresh recurrence — a second,
│  structurally distinct Interval DP shape (choose-from-either-end,
│  score-difference framing) was judged necessary for real pattern
│  mastery, not padding
├─ INTERVAL DP CLOSES: 2/2 (+1 extra) — ALL FOUR of this week's DP
│  subtypes complete
└─ Week 13 Consolidation: 21/21 required (100%), +6 extra (each tied
   to a named, checked gap), DP now 28/35 required complete (7
   through Wk12 + 21 this week; the 35-total denominator itself
   confirmed directly against Week 14's own stated count and a full
   recount across three weeks' plan documents, superseding this
   map's earlier-tracked 33, which predated Week 13's own plan
   revisions), 229 cumulative distinct, the 184-vs-185 plan-
   scorecard discrepancy traced and flagged (same Week-9-origin
   drift this map has now caught across five consecutive weeks),
   what Week 14 assumes

DAY 92 (Mon) — State Machine DP Opens: Transaction Fee, Cooldown
├─ NEW CONCEPT — State Machine DP formalized: dp[i][state], a
│  situation dimension (not just position) added to the DP index for
│  the first time; 0/1 Knapsack (Day 85) and Predict the Winner (Day
│  91) each informally previewed state-style thinking without ever
│  naming it
├─ LC 714 (Best Time to Buy/Sell Stock with Transaction Fee) —
│  required; 2-state (hold/cash) machine; taught BEFORE Cooldown,
│  reversed from the plan's own stated order, so state count
│  escalates from the pattern's simplest case outward (flagged
│  explicitly in the Resource Book, not silent)
├─ LC 309 (Best Time to Buy/Sell Stock with Cooldown) — required;
│  extends to 3 states, proven necessary (2 states cannot distinguish
│  "free to buy" from "cooling down after yesterday's sale")
├─ Direct citation, not re-teach: LC 121 (Wk2 D14, Sliding Window)
│  and LC 122 (Wk4 D23, Greedy) reframed as unconstrained special
│  cases of today's machine — greedy's exchange argument shown to
│  fail once fee/cooldown couples consecutive days
└─ Java 21 Pattern Matching (record patterns + guarded switch),
   building on sealed interfaces/records (Wk4 D28); exhaustive-switch
   Java-21-not-just-17+ distinction reused precisely as established

DAY 93 (Tue) — State Machine DP Closes: Bounded Transaction Count
├─ EXTENDS Day 92's machine with a transaction-count dimension:
│  dp[i][k][state], k a budget (advances only on buy, never sell)
├─ LC 123 (Best Time to Buy/Sell Stock III) — required; k fixed at 2,
│  unrolled into 4 named variables, derived from the general form
├─ LC 188 (Best Time to Buy/Sell Stock IV) — required; k generalized
│  to array-indexed states; k≥n/2 reduction to LC 122's unconstrained
│  greedy proven from the non-overlap bound on transaction count
└─ STATE MACHINE DP CLOSES: 4/4 (+0 extra) — reasoning for the zero
   stated explicitly (ladder already spans every shape the pattern
   tests), matching Dijkstra's Wk12 treatment

DAY 94 (Wed) — Tree DP Opens and Closes; Dynamic Programming Complete
├─ NEW CONCEPT — Tree DP: dp(node) via postorder combine, where a
│  return value can itself BE the full DP state (not merely support
│  the parent's own computation, Diameter's Wk7 D48 shape) —
│  distinguished explicitly, problem by problem, below
├─ Recap only, not re-taught: LC 543 (Diameter of Binary Tree, Wk7
│  D48) cited as the mechanism's first half already built
├─ LC 337 (House Robber III) — required; extends House Robber's
│  take-or-skip recurrence (Wk12 D82's disjoint-exhaustive proof,
│  assumed solid) onto a tree; returns a PAIR — both notRobbed and
│  robbed ARE the DP state
├─ LC 124 (Binary Tree Maximum Path Sum) — required; single return
│  value + outside global, shown to match Day 48's shape MORE closely
│  than LC 337's, despite both being "Tree DP" — explicit contrast
│  drawn as the day's key lesson
├─ [Extension] LC 968 (Binary Tree Cameras) — extra; 3-state greedy
│  tree DP, a genuinely third flavor (covering-constraint + greedy
│  exchange argument, not a pure disjoint-max argument)
└─ DYNAMIC PROGRAMMING CLOSES: 35/35 required across Wks 12–14 (34
   newly taught + 1 recap, LC 152 — reconciled explicitly, both
   figures correct, answering different questions); 6-subtype
   synthesis table (1D/Grid/String/Interval/State Machine/Tree)
   delivered as the day's theory block; 236 cumulative distinct
   through today

DAY 95 (Thu) — Bit Manipulation Opens: Binary, Two's Complement, XOR
├─ NEW PATTERN — built fully from zero: place value; two's
│  complement derived (not stated) from Wk2 D10's overflow-wraps
│  proof, negation formula -x = ~x+1 proven via ~x=(2ⁿ-1)-x; all
│  seven bitwise operators (& | ^ ~ << >> >>>) defined precisely;
│  >> vs >>> proven to diverge only for negative operands, concrete
│  example (n=-8)
├─ Recap only, not re-taught: Wk3 D15's HashMap Internals (h^(h>>>16),
│  (n-1)&hash) cited as an informal glimpse of today's operators
├─ XOR's algebraic properties proven by truth table (commutative,
│  associative, identity, self-inverse)
├─ LC 136 (Single Number) — required; XOR-fold
├─ n & (n-1) derived from binary-subtraction borrow mechanics
│  (clears the lowest set bit) — not asserted
├─ LC 191 (Number of 1 Bits) — required; naive 32-iteration shift-
│  and-check vs. n&(n-1)'s popcount-bounded loop, both correctly
│  identified as O(1) with the meaningful-but-non-asymptotic
│  distinction stated precisely, not glossed as "same thing"
└─ BIT MANIPULATION OPENS: 2/10 (+0 extra) — no extras on opening
   day, the EIGHTH consecutive genuinely-new top-level pattern in the
   series to receive that treatment

DAY 96 (Fri) — Power of Two; Bit Manipulation Fuses With DP
├─ LC 231 (Power of Two) — required; n&(n-1)==0 as an iff (not
│  heuristic) for "exactly one set bit"; n>0 guard proven necessary
│  (n=0 passes without it)
├─ LC 338 (Counting Bits) — required; FUSION — dp[i]=dp[i&(i-1)]+1,
│  yesterday's identity supplying the DP transition directly, well-
│  foundedness proven (i&(i-1) < i always); alternate recurrence
│  dp[i]=dp[i>>1]+(i&1) given; independent-per-number O(n log n) vs.
│  DP's O(n) correctly distinguished from yesterday's single-number
│  O(1)-vs-O(1) case (a genuine asymptotic gap this time, a range of
│  numbers, not one fixed-width value)
└─ 0 extra — two required problems already meaningfully distinct
   (direct application vs. cross-pattern fusion); one extra deferred
   to Day 97 for maximum reinforcement value

DAY 97 (Sat) — Missing Number; Per-Bit Frequency Counting (NEW)
├─ LC 268 (Missing Number) — required; XOR cancellation reused
│  unextended; sum-formula alternative's overflow risk (n≈46,341+)
│  tied directly back to Wk2 D10/D11's overflow and long-casting
│  lessons
├─ Counterexample proven first: 2^2^2=2 (not 0) — plain XOR fails
│  once groups are size 3, not 2
├─ NEW TECHNIQUE — per-bit frequency counting, derived as XOR's mod-2
│  cancellation generalized to mod-3
├─ LC 137 (Single Number II) — required; per-bit counting applied
└─ [Extra] LC 461 (Hamming Distance) — extra; near-zero marginal
   cost, composes Day 95's XOR + Days 95–96's bit-counting with no
   new technique, same mold as Wk13 D88's LC 583

DAY 98 (Sun) — Isolating Two Singletons; Addition Without +; Week Ends
├─ n & (-n) derived by combining Day 95's negation formula with the
│  n&(n-1)-style borrow argument applied to -n instead of n-1 —
│  isolates (not clears) the lowest set bit
├─ LC 260 (Single Number III) — required; four-step derivation (XOR-
│  all → isolate one differing bit → partition → XOR each half),
│  partition-correctness argued from XOR's own differing-bit
│  definition, not asserted
├─ LC 371 (Sum of Two Integers) — required; XOR/AND truth tables
│  mapped exactly onto a hardware full-adder's sum/carry columns;
│  termination bound (~32 iterations, fixed word size, NOT input-
│  value-dependent) argued precisely, "O(log(a+b))" flagged as an
│  imprecise first answer
├─ BIT MANIPULATION: 8/10 (+1 extra = 9 distinct this week), closes
│  at 10 required next week (LC 190, LC 421 — the latter also closing
│  Tries' deferred 7th problem, a two-prerequisite-chain dependency
│  flagged explicitly for Week 15)
└─ Week 14 Consolidation: 14/14 required (100%), +2 extra, DP CLOSED
   ENTIRELY at 35/35 (Wks 12–14), Bit Manipulation at 8/10, 245
   cumulative distinct, one reordering flagged (Day 92), zero recaps
   needed for the second consecutive week, what Week 15 assumes
   (this week's bit identities + Wk9's Trie mechanism, both needed
   for Maximum XOR)

DAY 99 (Mon) — Bit Manipulation's Final Two, Tries Closes, Kubernetes Opens
├─ LC 190 (Reverse Bits) — required; 32-bit shift-and-append, `>>>`
│  not `>>` (Day 95's sign-extension proof, now load-bearing — a
│  signed shift would corrupt the bit pattern being read)
├─ NEW STRUCTURE — Bit Trie: Wk9's TrieNode shape (insert/traverse),
│  alphabet swapped from 26 letters to 2 bits (children[2])
├─ LC 421 (Maximum XOR of Two Numbers in an Array) — required; Bit
│  Trie, greedy opposite-bit walk proved optimal via place-value
│  dominance (2^i > sum of every lower bit combined); closes BOTH
│  Bit Manipulation (10/10, +1 extra Wk14 = 11 distinct) AND Tries
│  (7/7, 0 extra, 7 distinct) — the deferred fusion flagged at Wk14's
│  close
├─ NEW TRACK — Kubernetes, from zero: Pod → ReplicaSet → Deployment →
│  Service, the declarative reconcile-loop model (contrasted directly
│  with Docker's imperative `docker run`); flagged prerequisite gap,
│  since the plan assumed this background without it existing yet
├─ Metrics Server + HPA (`autoscaling/v2`): averageUtilization measured
│  against CPU *requests*, not node capacity; scale-up/down asymmetry
│  (thrashing avoidance) named
└─ BIT MANIPULATION CLOSED 10/10 (+1 = 11 distinct). TRIES CLOSED 7/7
   (0 extra, 7 distinct). Zero extra added to either pattern this
   week, reasoned explicitly (see Overlap/consolidation notes).

DAY 100 (Tue) — Segment Trees Open; Creational Patterns: Singleton
├─ NEW STRUCTURE — Segment Tree: array-backed binary tree, Wk8 Heap's
│  EXACT index math reused (children 2i+1/2i+2, 0-indexed), node =
│  range aggregate not priority; 4n over-allocation
├─ Trade-off table derived: brute force (O(1) update/O(n) query) vs.
│  Prefix Sum (O(n) update/O(1) query) vs. Segment Tree (O(log n)
│  both) — the gap neither prior structure closes alone
├─ LC 307 (Range Sum Query - Mutable) — required; build/update/query
│  all traced on a concrete 3-element example, O(log n) query bound
│  proven via "≤2 straddling nodes per level" argument
├─ NEW TRACK — Design Patterns (Creational): Singleton, Factory
│  Method, Builder introduced at one-sentence depth each, motivated
│  via Wk1's SOLID (Open/Closed, Dependency Inversion)
├─ Singleton, full depth: private constructor + static accessor;
│  naive lazy version's race condition proved (same shape as Wk6
│  D37's Counter); Double-Checked Locking
├─ NEW CONCEPT — `volatile`: DCL's reordering bug proved precisely
│  (construction's 3 conceptual steps, steps 2/3 reorderable without
│  it) — first new concurrency primitive since Wk6's
│  synchronized/ReentrantLock
├─ Enum Singleton: classloader thread-safety guarantee; reflection
│  immunity (`Constructor.newInstance()`'s hard-coded enum check) and
│  serialization immunity (`Enum.valueOf()`, no constructor called)
│  both proved, not asserted
└─ `lld-java` repo initialized. SEGMENT TREES: 1/2 so far (opens,
   correctly 0 extra). SINGLETON complete (DCL + Enum, both variants).

DAY 101 (Wed) — Segment Trees Close; Factory Method & Builder
├─ Segment Tree, NEW MENTAL MODEL — indexed by VALUE not position;
│  leaf = count of insertions, not a raw element
├─ Coordinate compression taught as the general technique (sort,
│  dedupe, rank) — not the narrower same-problem-specific offset trick
├─ LC 315 (Count of Smaller Numbers After Self) — required, Hard;
│  right-to-left invariant proved (only right-of-i elements inserted
│  when i is queried); full trace on nums=[5,2,6,1] → [2,1,1,0]
├─ [Extension] Fenwick Tree / BIT — i & (-i) DIRECTLY reuses Wk14
│  D98's isolate-lowest-set-bit identity; invertibility (needs
│  subtraction) named as why it can't generalize to min/max the way
│  Segment Tree does
├─ SEGMENT TREES CLOSED 2/2 (0 extra, 2 distinct) — plan's own
│  "deliberately light" framing honored explicitly, matching
│  Dijkstra's/State-Machine-DP's "ladder already sufficient" precedent
├─ Factory Method, full depth: Simple Factory (violates OCP) formally
│  distinguished from true GoF Factory Method (abstract creator +
│  subclasses, respects OCP) — a real, commonly-conflated distinction
├─ Builder, full depth: telescoping-constructor anti-pattern shown
│  directly; immutability (final fields) tied explicitly to thread-
│  safety-for-free, not just readability
└─ Both patterns implemented with JUnit tests, pushed to
   `lld-java/design-patterns`.

DAY 102 (Thu) — Sorting From Scratch; Quickselect; DSA Retrospective
├─ NEW ALGORITHM — Merge Sort: divide-conquer, correctness by
│  induction, stability mechanism proved (<= in merge, not <, left-
│  half-first); O(n log n) EVERY case via Master Theorem (a=b=2)
├─ NEW ALGORITHM — Quicksort: Lomuto partition — Wk2 D12's Dutch
│  National Flag boundary-pointer-and-swap, reduced 3 zones → 2;
│  O(n²) worst case traced on sorted input (arithmetic-series shape,
│  same family as Day5's naive-approach proofs); randomized-pivot
│  mitigation
├─ Java's real `Arrays.sort()` reconciled: dual-pivot quicksort
│  (primitives — stability moot, comparisons cheap/trusted) vs.
│  TimSort (objects — stability matters, comparisons
│  expensive/arbitrary) — verified via web search, not assumed
├─ 🔗 LC 215 (Kth Largest Element in an Array) — RECAP ONLY (Wk8 D55,
│  min-heap); Quickselect delivered FULL DEPTH as the new technique,
│  reusing today's own partition method; T(n)=T(n/2)+O(n) geometric-
│  series argument contrasted explicitly against quicksort's
│  T(n)=2T(n/2)+O(n) — proves the O(n)-vs-O(n log n) split isn't the
│  same argument restated
├─ THEORY — full DSA pattern-and-signal retrospective, ~19 patterns,
│  built as a standing reference table
└─ `dsa-java` final-pass cleanup (no new content). SORTING: from-
   scratch exercise + 1 recap (LC 215) — no LC-numbered "required"
   problems of its own, by design.

DAY 103 (Fri) — SQL Opens: Subqueries, Aggregation, Self-Joins
├─ NEW TRACK — SQL practice (10 problems total, Days 103–104),
│  entirely outside the original plan, extending Wk6 D40's
│  fundamentals (JOINs, keys, indexes, NOT IN/NULL trap — NOT
│  re-taught)
├─ NEW CONCEPT — SQL logical processing order: FROM→WHERE→GROUP BY→
│  HAVING→SELECT→ORDER BY→LIMIT — the foundation for every WHERE-vs-
│  HAVING and (Day 104) window-function rule this week
├─ LC 176 (Second Highest Salary) — required; MAX-subquery's native
│  NULL-on-empty-result behavior proved; LIMIT/OFFSET alternative's
│  DISTINCT requirement + scalar-wrapping requirement both traced
├─ LC 184 (Department Highest Salary) — required; correlated subquery
│  mechanism defined precisely (re-evaluated per outer row); ties
│  handled correctly by construction, contrasted against a naive
│  GROUP BY/MAX that would silently drop them; 🔗 forward ref to
│  Day 104's PARTITION BY as the generalization
├─ LC 182 (Duplicate Emails) — required; GROUP BY/HAVING taught
│  properly, tied directly to the logical processing order above
├─ LC 197 (Rising Temperature) — required; self-join #1 (date-offset
│  via DATEDIFF, NOT row-adjacency — gap-safety argued explicitly)
├─ LC 181 (Employees Earning More Than Their Managers) — required;
│  self-join #2 (hierarchical, managerId=id); INNER JOIN's NULL-
│  exclusion tied DIRECTLY back to Wk6's NOT IN/three-valued-logic
│  material
└─ SQL: 5/10 so far. No Project Block today (SQL Block doubles as the
   day's full deliverable) — flagged explicitly, not silently omitted.

DAY 104 (Sat) — SQL Closes: Window Functions
├─ NEW CONCEPT — Window functions: the core GROUP BY contrast proved
│  precisely (rows NOT collapsed, unlike GROUP BY); OVER/PARTITION
│  BY/ORDER BY anatomy; the "can't filter a window function in the
│  same query's WHERE" rule, extending Day 103's processing order by
│  one clause
├─ ROW_NUMBER/RANK/DENSE_RANK precisely distinguished on a tied
│  dataset — the classic trap, resolved with a worked table, not
│  asserted
├─ LC 177 (Nth Highest Salary) — required; parameterized LIMIT/OFFSET
│  (extends Day 103 D176 directly) + DENSE_RANK, the latter EXPLICITLY
│  tied back to Day 103's DISTINCT as "the same intent, two
│  mechanisms"
├─ LC 180 (Consecutive Numbers) — required; LAG×2, self-join named as
│  the pre-window-function alternative for contrast
├─ LC 262 (Trips and Users) — required, Hard; Users joined twice
│  (client role + driver role) distinguished from a true self-join;
│  CASE expressions + conditional aggregation (SUM(CASE...)) taught
│  from zero
├─ LC 626 (Exchange Seats) — required; CASE + parity, full 5-row trace
│  proving the odd-total edge case
├─ LC 185 (Department Top Three Salaries) — required, Hard;
│  PARTITION BY given full treatment, explicitly generalizing Day
│  103's correlated-subquery "one group at a time" limit; DENSE_RANK
│  proved correct over RANK/ROW_NUMBER via a concrete tie counter-
│  example (90k/90k/85k/80k) — the definitive synthesis of the whole
│  2-day SQL arc
└─ SQL PRACTICE TRACK CLOSED 10/10 (0 extra) — entirely additive to
   the original plan, spanning subqueries through window functions.

DAY 105 (Sun) — Consolidation: The Entire DSA Curriculum Closes
├─ No new content — self-check (full pattern recall, 3-random-pattern
│  cold-solve) + Weekly Scorecard
├─ Numeric reconciliation flagged: plan's own "203 DSA problems"
│  vs. this map's row-by-row 197 required-only DSA — same drift class
│  already caught before, map's table held authoritative
└─ Week 15 Consolidation: 15/15 planned problems delivered exactly
   (4 new DSA + 1 recap + 10 SQL), 0 extra anywhere — a clean planned-
   equals-actual week. DSA PHASE CLOSED ENTIRELY: 197 required + 53
   extra = 249 distinct DSA problems (Wks 1–15) + 10 SQL = 259 total.
   What Week 16 assumes: every pattern above, on recall, plus all
   three Creational patterns and this week's SQL toolkit, all as
   settled background — Week 16 opens Structural design patterns and
   real LLD systems.

DAY 106 (Mon) — LLD Interview Framework, and Structural Patterns
├─ 5-step LLD framework: clarify requirements → identify core objects →
│  define relationships/class diagram → apply patterns deliberately →
│  code the core (needs: nothing new — a live-interview process, not a
│  data structure); reused unchanged for every remaining system this
│  phase, Week 16 and Week 17 both
├─ GoF's 3-category taxonomy named explicitly for the first time —
│  Creational (Week 15) / Structural / Behavioral (Day 107) —
│  "Creational" itself had been used as a track label since Week 15
│  without the full taxonomy being named
├─ Structural patterns, full depth (needs: interfaces — Wk1 D2; SOLID's
│  Open/Closed — Wk1 D6; recursion — Wk2 D8 for Composite specifically):
│  Adapter, Decorator (full implementation — Coffee pricing, 3+ stacked
│  decorators, `lld-java/design-patterns/decorator/`), Facade, Proxy
│  (virtual/lazy + logging variants shown), Composite (sketch — file
│  system, `getSize()` shown structurally identical to Wk7 D46's
│  postorder DFS combine step)
└─ Deliberate constraint stated explicitly: lambdas/functional
   interfaces still not taught anywhere in the series (flagged Wk5
   D29, reconfirmed Wk8 D54) — every pattern this week implemented via
   named concrete classes, matching the plan's own phrasing throughout

DAY 107 (Tue) — Behavioral Patterns, and TDD Practice
├─ Behavioral patterns, full depth (needs: Day 106's Structural
│  foundation and 3-category taxonomy; interfaces — Wk1 D2): Observer
│  (full implementation, WeatherStation/Display, built test-first),
│  Strategy (DiscountStrategy example, forward-referenced to Wk17
│  D115/D118), State (small TrafficLight example, forward-referenced
│  to Day 109's full system), Command (remote-control/undo example,
│  full depth — no later day in the plan reinforces it, so it isn't
│  deferred the way State's full treatment is), Template Method
│  (sketch — DataProcessor, mechanism tied directly to Wk15 D101's
│  Factory Method abstract-method shape)
├─ Test-Driven Development — Red/Green/Refactor (NEW methodology,
│  needs: JUnit — Wk7 D45), applied to build Observer from a failing
│  test through to a refactored, multi-observer-tested implementation
└─ DSA Revision Block opens (needs: every pattern below already
   closed): Course Schedule (LC 207, Graph, orig. Wk11 D71) + Coin
   Change (LC 322, DP, orig. Wk12 D84), both solved cold — see the new
   DSA Revision Log below

DAY 108 (Wed) — LLD #1: Tic-Tac-Toe, and Coupling/Cohesion/Law of Demeter
├─ Framework applied live, end-to-end, for the first time (needs: Day
│  106's 5 steps; enums — established since Wk1) — Board/Player/Cell/
│  Game/Mark; Step 4 correctly concludes no pattern is warranted, the
│  judgment itself treated as a checkable skill, not a coverage gap
├─ Coupling / Cohesion / Law of Demeter (NEW, needs: encapsulation —
│  Wk1 D5) — applied retroactively to the Tic-Tac-Toe code just
│  written, including one genuine LoD violation (`game.getBoard()
│  .print()`) found and fixed live, and one deliberate relaxation
│  (chaining through the immutable `Player` DTO) justified explicitly
│  rather than applied dogmatically
└─ DSA Revision: Combination Sum (LC 39, Backtracking w/ reuse, orig.
   Wk10 D64), solved cold

DAY 109 (Thu) — LLD #2: Vending Machine (State Pattern), and Mock #1
├─ Framework applied live a second time (needs: Day 108's live run) —
│  Product/Inventory/VendingMachine/VendingMachineState w/ 4 concrete
│  states (NoCoin/HasCoin/Dispensing/SoldOut); State now genuinely
│  earned, zero state-dispatch conditionals in the context class,
│  verified directly against the code rather than asserted
├─ Mock Interview #1 run and debriefed — Tic-Tac-Toe as subject
└─ Theory: State vs. Strategy, the intent distinction (needs: both
   patterns already built — Day 107's DiscountStrategy, today's State)
   + DSA Revision: Longest Substring Without Repeating Characters
   (LC 3, Sliding Window, orig. Wk3 D15), solved cold

DAY 110 (Fri) — LLD #3: Parking Lot, Part 1 — Design and Core Logic
├─ Framework applied a third time — ParkingLot/Level/ParkingSpot/
│  Vehicle/Ticket, both Vehicle and ParkingSpot sized via a SHARED
│  ordered enum (`VehicleSize`), not subclassing; Singleton considered
│  for `ParkingLot` and explicitly declined, with reasoning — distinct
│  from Days 108/109's "no pattern fits" conclusions in that this one
│  was actively considered and ruled out, not simply never raised
├─ Composition Over Inheritance (NEW, formalized — needs: Wk1 D6's
│  Open/Closed; the instinct itself traces to Wk1 D5's Runnable-over-
│  extends-Thread preference) — a flawed inheritance-based spot
│  hierarchy built and critiqued directly (what breaks when a new size
│  tier is added), then the composition version already in use
│  justified retroactively against it
└─ DSA Revision: Validate Binary Search Tree (LC 98, Tree DFS w/
   propagated bounds, orig. Wk8 D50), solved cold

DAY 111 (Sat) — LLD #3: Parking Lot, Part 2 — Concurrency
├─ The Day 110 check-then-act race (in `findAvailableSpot`/
│  `assignVehicle`) found, proven, and fixed (needs: synchronized/
│  ReentrantLock — Wk6 D37; wait/notify — Wk6 D38; ConcurrentHashMap's
│  CAS — Wk6 D39; volatile/JMM — Wk15 D100) — per-`ParkingSpot` locking
│  via synchronized `tryAssign()`/`removeVehicle()`; `AtomicInteger`
│  introduced (NEW class, built on Wk6 D39's already-taught CAS
│  primitive, not a new mechanism) for ticket-ID generation;
│  `parkedVehicle` confirmed to need no `volatile` given synchronized
│  already covers its visibility; a 10-thread concurrency test proving
│  no double-booking, built from already-taught primitives only (raw
│  `Thread`, `wait`/`notifyAll`, `join` — deliberately not
│  `ExecutorService`/`CountDownLatch`, untaught material)
└─ Theory: Where to Put the Lock — whole-lot vs. per-level vs. per-spot
   granularity, generalized to `ConcurrentHashMap`'s own lock striping
   (Wk6 D39) + DSA Revision: Redundant Connection (LC 684, Union-Find,
   orig. Wk11 D74 — chosen over Dijkstra as the "newer, less-rehearsed"
   pick the plan itself called for)

DAY 112 (Sun) — LLD #4: Library Management System, and Mock #2
├─ Framework applied a fourth time — `Book` vs. `BookItem` (catalog
│  entry vs. physical copy, a NEW distinction), `Catalog` (search only)
│  and `Library` (search + circulation) kept genuinely separate —
│  `Catalog` provably has zero `Reservation`/`Member`/`BookItemStatus`
│  references anywhere in its own code
├─ `BookItem`'s status lifecycle built as a data-driven transition
│  table (Day 110's technique, reused), NOT a full State
│  implementation — contrasted directly against Day 109's genuine
│  State usage, the distinction being varying behavior (State) vs. a
│  validity check only (data table)
├─ Mock Interview #2 run and debriefed — Parking Lot + concurrency,
│  with a forced "now make it thread-safe" mid-interview escalation
└─ Week 16 Consolidation: 4/4 planned LLD systems delivered, 6/6
   planned DSA revision problems solved cold, 0 new DSA problems (none
   planned — the series' first non-DSA-pattern week). What Week 17
   assumes: all ten patterns + the framework + this week's four
   judgment calls (pattern vs. no pattern; State vs. data table;
   Singleton considered and declined; lock granularity chosen and
   justified), all fully reflexive, with none of them re-explained.

DAY 113 (Mon) — LLD #5: ATM Machine
├─ State pattern REAPPLIED, zero new mechanism — Idle/HasCard/
│  CorrectPIN/Dispensing, verified against zero state-dispatch
│  conditionals exactly as Day 109 was
├─ Chain of Responsibility taught (NEW — this series' 11th pattern) —
│  ₹2000→₹500→₹100 denomination chain; a naive mutate-as-you-go
│  dispenseBroken() shown to silently corrupt inventory on partial
│  failure, fixed with a canDispense()/commitDispense() read-then-
│  write split
├─ Largest-first dispensing proven safe via an exchange argument (each
│  denomination an exact multiple of the next), proven UNSAFE in
│  general via the {1,3,4}, target 6 counterexample — directly
│  connected to why Coin Change (Wk12 D84) needed DP instead of greedy
└─ DSA revision: K Closest Points to Origin (LC 973, Heap) — bounded
   max-heap of size k, cited to Wk8 D55's Kth Largest lineage

DAY 114 (Tue) — LLD #6: Elevator System, and Mock #3
├─ State pattern reapplied a THIRD time — Idle/MovingUp/MovingDown/
│  DoorsOpen; unlike Days 109/113, driven partly by internal progress
│  (step()) not only external events
├─ SCAN vs. LOOK distinguished precisely (NOT a GoF pattern) — LOOK
│  (reverse once nothing remains ahead) is what's actually built, not
│  pure SCAN (travel to the physical boundary regardless); direction
│  persistence via a lastDirection field, using TreeSet.higher()/
│  lower() (same navigable-structure family as Wk9 D60's
│  TreeMap.ceilingKey() for Consistent Hashing)
├─ Worked trace proves LOOK beats FIFO more than 2x on an identical
│  request set (8 floors vs. 17)
├─ Mock Interview #3 — first Machine Coding-format mock (NEW interview
│  skill): a compiling-skeleton-first discipline, contrasted directly
│  against the "think before coding" instinct that serves LLD
│  interviews
├─ Token Bucket (Wk10 D68) rebuilt cold as the mock's task — RECAP
│  only, no new algorithm; the naive-fixed-window contrast and the
│  synchronized correctness requirement (Wk6 D37) both cited, not
│  re-derived
└─ DSA revision: Design Add and Search Words Data Structure (LC 211,
   Trie with wildcard branching) — cited to Wk9 D60

DAY 115 (Wed) — LLD #7: Splitwise
├─ Strategy's FIRST full system-level application (taught Wk16 D107,
│  never yet the primary pattern for an entire system) — EqualSplit/
│  ExactSplit/PercentageSplit, the latter two constructor-configured
│  exactly like D107's PercentageDiscount(0.10)
├─ BalanceSheet — pairwise ledger netted into signed per-person
│  balances
├─ Dual-Heap Greedy Settlement taught (NEW — 7th distinct heap role
│  this series has built, explicitly distinguished from Wk9 D58's Two
│  Heaps balanced-median technique, which splits ONE dataset rather
│  than pairing two independent heaps)
├─ Proven, by an exchange-style counting argument: at most (n−1)
│  transactions for n people with nonzero balance — AND the honest
│  limit stated explicitly: not always the true theoretical minimum,
│  which is LeetCode 465 (Optimal Account Balancing), NP-hard,
│  backtracking-solved, out of scope
├─ LinkedIn Post 22 published — the settlement algorithm, including
│  the honest caveat, not just the confident version
└─ DSA revision: Generate Parentheses (LC 22, Backtracking, counter-
   gated) — DELIBERATELY DIFFERENT from Wk16's Combination Sum (LC 39)
   per this map's own flagged constraint

DAY 116 (Thu) — LLD #8: BookMyShow, Part 1 — Design
├─ NO new pattern — pure 5-step framework application; Seat/Show/
│  Screen/Theater/Booking, with seat status deliberately scoped
│  PER-SHOW (a map on Show), not as a global field on Seat
├─ Single-threaded booking flow (hold→confirm→cancel) built and
│  passes every single-threaded test
├─ Theory: the exact check-then-act race traced concretely — two
│  threads' availability reads both succeeding before either write
│  lands — named explicitly as the SAME shape as Parking Lot's race
│  (Wk16 D111) and the ATM's canDispense/commitDispense assumption
│  (Wk17 D113), set up for Day 117's fix, deliberately not fixed yet
└─ DSA revision: Clone Graph (LC 133, Graph DFS + visited map) —
   DELIBERATELY DIFFERENT from Wk16's Course Schedule (LC 207) per
   this map's own flagged constraint

DAY 117 (Fri) — LLD #8: BookMyShow, Part 2 — Concurrency, and Mock #4
├─ Pessimistic locking: in-memory per-seat lock (DIRECT extension of
│  Wk16 D111's per-ParkingSpot technique, zero new mechanism) PLUS
│  SELECT FOR UPDATE (NEW — DB-level row lock, motivated specifically
│  by multi-process deployment, which an in-memory lock structurally
│  cannot reach)
├─ Optimistic locking taught (NEW NAME, OLD MECHANISM — the identical
│  CAS idea already used by AtomicInteger, Wk16 D111/Wk6 D39, now
│  guarding a version column instead of an in-memory integer);
│  @Version connected to @Entity/@Id (Wk6 D36); the ABA problem named
│  and ruled out (version is monotonic, never reused)
├─ CountDownLatch taught (NEW — exactly one new primitive, used only
│  as a start gate; completion-wait still uses Wk16 D111's .join(),
│  deliberately not replaced) — the 10-thread test proven
│  deterministic (exactly 1 success, every run) for both fixes,
│  contrasted against the broken version's non-deterministic
│  multi-success behavior
├─ Pessimistic vs. optimistic given a concrete trade-off (contention
│  level, not a coin flip) — BookMyShow's actual answer: optimistic
│  by default, pessimistic/a queue for identifiable high-contention
│  hot spots
├─ Mock Interview #4 run — BookMyShow + concurrency, pushed on
│  pessimistic-vs-optimistic justification specifically
└─ DSA revision: Longest Increasing Subsequence (LC 300, 1D DP, both
   O(n²) and O(n log n) forms) — DELIBERATELY DIFFERENT from Wk16's
   Coin Change (LC 322) per this map's own flagged constraint

DAY 118 (Sat) — LLD #9: Food Delivery System, and System Design Preview
├─ Strategy's SECOND full system-level application — NearestPartner/
│  HighestRatedPartner, both stateless/parameterless (unlike Day
│  115's constructor-configured strategies) — used explicitly to
│  sharpen Wk16 D109's State-vs-Strategy test: the definitional
│  signal is WHO CHOOSES (the client), not whether constructor config
│  is carried, which is a common tell, not the definition
├─ Runtime strategy-swapping demonstrated (setStrategy()) — a
│  capability Day 115's fixed-per-Expense usage never exercised
├─ System Design's 5-step framework PREVIEWED ONLY (Requirements→
│  Estimation→HLD→Detailed Design→Bottlenecks) — deliberately
│  shallow, full depth deferred to Wk18 D120; explicitly not taught
│  ahead of Week 18's plan
├─ Career: all 7 target-company applications submitted (Rippling,
│  Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech)
└─ DSA revision: Edit Distance (LC 72, 2D String DP) — cited directly
   as LCS's table shape (Wk13) with insert/delete added and replace
   permitted

DAY 119 (Sun) — LLD #10: Hotel Booking System, Mock #5, LLD Phase Closed
├─ Self-check opened the day: all ten LLD systems + primary pattern
│  (or deliberate absence of one) listed cold — only 4 of 10 chose a
│  GoF pattern as the headline decision, flagged explicitly as itself
│  the lesson
├─ NO new pattern — Room availability scoped by DATE RANGE (the same
│  per-show-not-per-seat lesson from Day 116, applied to a different
│  axis), overlap check cited directly as Week 4's interval-
│  intersection logic, checkOut modeled as EXCLUSIVE (same-day
│  turnover correctness)
├─ Concurrency model compared against BookMyShow — grounded
│  concretely in Wk9 D61's sync-vs-async replication trade-off, NOT a
│  re-application of Day 117's locking machinery: contention
│  concentration, latency tolerance, number of independent
│  coordinating parties, and cost of the failure mode all named as
│  the actual deciding factors
├─ Mock Interview #5 run (candidate's choice of subject) — LLD
│  phase's fifth and final mock
├─ Weekly Industry Awareness Ritual + Weekly Scorecard closed the LLD
│  phase: 10/10 systems, 11 patterns (10 GoF + the framework), 5
│  mocks
└─ Week 17 Consolidation: 6/6 planned LLD systems delivered, 3/3
   mocks, 6/6 DSA revision problems solved cold (Backtracking/Graph/DP
   picks confirmed distinct from Week 16's, per this map's own
   flagged constraint), 0 new DSA problems (deliberate — all six
   blocks were spaced-repetition revision of already-fully-mastered
   patterns, not new-pattern acquisition needing extra reps), 1/1
   LinkedIn post, 7/7 applications submitted. What Week 18 assumes:
   today's System Design preview genuinely in hand as a shape to fill
   in; Token Bucket (Wk10 D68) solid enough to anchor Wk18 D121's
   fuller rate-limiting landscape; this week's concrete,
   factor-grounded trade-off reasoning (Day 117's locking choice, Day
   119's sync-vs-async choice) expected to reapply at HLD scale.
```

---


### Week 18 Additions (Days 120–126) — System Design (HLD) Begins

Everything below is newly taught starting Day 120; nothing here was covered in Weeks 1–17. Listed in the order each concept's prerequisites are satisfied.

| Concept | First Taught | Depends On (confirmed prior) |
|---|---|---|
| System Design (HLD) 5-Step Framework — Requirements, Estimation, HLD, Detailed Design, Bottlenecks | Day 120 | Distinct from the LLD 5-Step Framework (Day 106); previewed shallowly only, Day 118 |
| Base62 Encoding | Day 120 | Java: `StringBuilder`, char arithmetic, div/mod (Week 1) |
| `SELECT ... FOR UPDATE SKIP LOCKED` | Day 120 | `SELECT ... FOR UPDATE` (Week 17, Day 117) |
| Leaking Bucket | Day 121 | — |
| Fixed Window Counter (+ its 2x boundary flaw) | Day 121 | — |
| Sliding Window Log | Day 121 | — |
| Distributed Rate Limiting via Redis + Lua atomicity | Day 121 | Token Bucket (Week 10, Day 68); Spring Cloud Gateway (Week 10, Day 66); Load Balancers (Week 10, Day 72) |
| Notification System architecture — per-channel queues, Template Service | Day 122 | DLQ pattern (Week 14, Day 93, cited not re-taught) |
| Redis, full picture — data structures, Memcached contrast, persistence, pub/sub | Day 123 | Redis borrowed narrowly, Days 121–122; infra since Week 6, Day 44 |
| Thundering Herd (+ repopulation-mutex, conceptual only; stale-while-revalidate, built) | Day 123 | — |
| Cache-Aside pattern (`@Cacheable`/`@CacheEvict`) | Day 123 | Spring AOP/proxy interception (Week 9, Day 62); Resilience4j (Week 10, Day 64) |
| Consistent Hashing, applied to cache-node membership | Day 123 (recap) | Week 9, Day 60 — not re-taught |
| UUID (structure, and why it isn't sortable) | Day 124 | — |
| Twitter Snowflake ID generation (64-bit layout) | Day 124 | Java bitwise ops, `long`, `synchronized` (Week 1, Week 4) |
| DB ID Range Allocation | Day 124 | Conceptually parallel to the KGS pattern, Day 120 |
| Quorum (general majority-overlap tool) | Day 125 | Replication's leaderless R+W>N (Week 9, Day 61) — same tool, new application |
| Split-brain | Day 125 | CAP Theorem (Week 9, Day 59) — direct connection |
| Paxos (conceptual only — not implemented) | Day 125 | — |
| Raft — leader election, log replication, safety | Day 125 | Replication Models (Week 9, Day 61) — closes its open gap |
| etcd (Raft-backed) | Day 125 | Kubernetes Control Plane (Week 15, Day 99) — names a mechanism already relied on |
| BookMyShow at Scale — city-sharding, seat-hold (Redis TTL + Lua), proactive cache warming | Day 126 | Combines Days 120–125; BookMyShow LLD locking (Week 17, Days 116–117) unchanged, cited |

### Week 19 Additions (Days 127–133) — System Design (HLD) Closes

Everything below is newly taught starting Day 127; nothing here was covered in Weeks 1–18. Listed in the order each concept's prerequisites are satisfied. **This closes the HLD phase at sixteen systems total, Weeks 18–19.**

| Concept | First Taught | Depends On (confirmed prior) |
|---|---|---|
| Persistent WebSockets | Day 127 | HTTP request-response (Week 10, Day 72); TCP (Week 10, Day 71) |
| Connection-Management Registry | Day 127 | Persistent WebSockets, above; Redis as infra (Week 6, Day 44) |
| Wide-Column Store (Cassandra-style) | Day 127 | B-tree write cost (Week 6, Day 40); CAP Theorem (Week 9, Day 59); Replication Models (Week 9, Day 61) |
| Geohashing | Day 128 | — |
| Quadtrees | Day 128 | Contrasted directly against Geohashing, above |
| Redis Geo (`GEOADD`/`GEOSEARCH`) | Day 128 | Redis's data-structure survey (Week 18, Day 123 — Sorted Sets named, not built); Geohashing, above |
| Transcoding Pipeline | Day 129 | Kafka async pipeline pattern (Week 7, Day 48) |
| Adaptive Bitrate Streaming | Day 129 | Transcoding Pipeline, above |
| CDN | Day 129 | Cache-Aside (Week 18, Day 123); Thundering Herd + proactive warming (Week 18, Days 123, 126) |
| Push / Pull / Hybrid Fan-out | Day 129 | "The celebrity problem" (Week 12, Day 78) |
| Idempotency Keys | Day 130 | The check-then-act shape (Days 121, 122, 124, 126) |
| Distributed Locks (Redis `SETNX`) | Day 130 | Lua atomicity (Week 18, Day 121); Quorum (Week 18, Day 125 — for Redlock) |
| Double-Entry Ledger | Day 130 | Optimistic/pessimistic locking precedent (Week 17, Day 117) |
| SQS Visibility Timeout | Day 131 | Distributed Locks, above — the identical TTL-lease shape |
| Quartz Scheduler — Cron & Misfire Handling | Day 131 | — |
| At-Least-Once vs. Exactly-Once ("Effectively-Once") Execution | Day 131 | Idempotency Keys, above |
| Skip List | Day 132 | Linked lists (Week 1–2); B-tree write cost (Week 6, Day 40 — the contrast) |
| Redis Cluster | Day 132 | Consistent Hashing (Week 9, Day 60) — contrasted directly, not conflated |
| Composite-Score Tie-Handling | Day 132 | Skip List / Sorted Sets, above |
| Distributed, Asynchronously-Rebuilt Trie | Day 132 | Tries, full depth (Week 9, Days 59–61) |
| Chunking — Fixed-Size vs. Content-Defined | Day 133 | — |
| Content-Addressable Deduplication | Day 133 | Chunking, above; hashing/HashMap fundamentals (Week 1–2) |
| Sync Conflicts — LWW vs. Conflict-Copy | Day 133 | Synchronous vs. asynchronous replication trade-offs (Week 9, Day 61) |
| Multipart Upload to S3 | Day 133 | Distinguished directly from Chunking, above — transport-level, not storage-level |

### Week 20 Additions (Days 134–140) — From Designing Systems to Operating One

| Concept | Day | Prerequisite(s) Cited |
|---|---|---|
| Load Testing Fundamentals (VUs, throughput, p50/p95/p99 proven on real numbers) | Day 134 | — |
| k6 (script anatomy, thresholds) | Day 134 | Load Testing Fundamentals, above |
| Kubernetes Deployment/Service manifests, written for real | Day 134 | Kubernetes from zero (Week 15, Day 99) |
| Resource Requests vs. Limits (scheduler bin-packing vs. enforced ceiling; OOMKill vs. CPU throttle) | Day 134 | Kubernetes from zero (Week 15, Day 99) — directly extends the HPA `averageUtilization` note |
| Helm (`Chart.yaml`, `values.yaml`, `templates/`, `_helpers.tpl`) | Day 135 | Day 134's raw manifests |
| Kubernetes Ingress | Day 135 | Spring Cloud Gateway (Week 10, Day 66) — contrasted directly, not conflated |
| Minikube | Day 135 | Docker (Week 7, Day 43) |
| Multi-Environment Values Layering (`-f` precedence; maps deep-merge, lists replace wholesale) | Day 136 | Helm, above |
| ConfigMap vs. Secret (Secret = base64-encoded, proven not encrypted) | Day 136 | Spring Profiles (Week 8, Day 51) — same externalized-config pattern, one layer up |
| "Build Once, Deploy Many Times" | Day 136 | ConfigMap/Secret, above |
| Micrometer Tracing (Trace ID/Span ID, automatic HTTP context propagation, sampling) | Day 137 | Micrometer-as-a-facade (Week 11, Days 76–77) |
| Zipkin | Day 137 | Micrometer Tracing, above |
| Tracing's Kafka Blind Spot (context does not auto-propagate across a message the way it does over HTTP) | Day 137 | Kafka pub/sub (Week 7, Day 48); Saga Choreography (Week 10, Day 70) |
| API Design Practice — Versioning, Cursor-vs-Offset Pagination, Structured Error Contracts | Day 137 | Idempotency Keys (Week 19, Day 130) |
| Chaos Engineering, as formal practice | Day 138 | Resilience4j Circuit Breaker (Week 10, Day 64) |
| Connection Pooling (HikariCP) | Day 138 | — |
| Service Mesh (sidecar injection, weighted traffic split) | Day 138 | Resilience4j (Week 10, Day 64) — contrasted explicitly as infrastructure vs. library |
| `BoundedBlockingQueue<T>`, hand-rolled | Day 138 | Producer-Consumer / `ReentrantLock` / `Condition` (Week 6, Days 37–38); `CountDownLatch`-gated testing (Week 17, Days 116–117) |
| Mutation Testing (concept, PIT named) | Day 139 | TestContainers vs. WireMock realism-vs-control distinction (Week 7, Day 47; Week 8, Day 52) |
| JaCoCo wired as an enforced, build-failing gate | Day 139 | Mutation Testing, above |
| GitHub Actions CI/CD Pipeline, traced end to end including its failure path | Day 139 | Docker (Week 7, Day 43); Helm (Day 135) |
| Liveness vs. Readiness Probes (restart vs. Endpoints-removal; dependency-in-liveness anti-pattern) | Day 139 | Actuator/Micrometer (Week 11, Days 76–77); Service Endpoints mechanism (Day 134) |
| Platform At-Scale Analysis (10x/100x, applied to a built system) | Day 140 | HLD 5-Step Framework's Estimation/Bottlenecks steps, already applied to all 16 Weeks 18–19 systems |

**Zero new DSA problems, zero new LLD/HLD systems this week** — confirmed directly against `Week_20_Revised.md`'s full content, matching the shape already set by Weeks 16–19.

### Week 21 Additions (Days 141–148) — Behavioral Mastery, Domain Knowledge, and Final Polish (Series Closes)

| Concept | Day | Prerequisite(s) Cited |
|---|---|---|
| The STAR Framework, rigorously (timing budget, quantification without an obvious number) | Day 141 | — (opens the behavioral track from zero) |
| Company-Agnostic Competency Map (6 competencies) | Day 141 | STAR Framework, above |
| UPI's NPCI/PSP/bank-node architecture, applied as concrete 2PC roles | Day 141 | Two-Phase Commit (Week 12, Day 79) — recap only; Saga (Week 10, Day 70) — recap only |
| Resolving an ambiguous UPI transaction after a mid-transfer network failure | Day 141 | Idempotency Keys (Week 19, Day 130) — reused, not re-taught |
| High Availability (Active-Active vs. Active-Passive, load-balancer health-check failover) | Day 142 | Kubernetes readiness probes/Endpoints (Week 20, Day 134); Chaos Engineering (Week 20, Day 138); Quorum/split-brain (Week 18, Day 125) |
| High-Volume, Low-Margin Transaction Systems (cost-at-scale reframing) | Day 143 | Sharding (Week 12, Day 78); Distributed Cache (Week 18, Day 123); Rate Limiting (Week 10/18, Day 68/121) |
| Multi-Tenant SaaS at Scale (isolation spectrum, noisy neighbor, RBAC at scale) | Day 144 | Sharding (Week 12, Day 78); Rate Limiting (Week 18, Day 121); Connection Pooling (Week 20, Day 138) |
| Coverage-gap audit methodology (8 stories × 6 competencies) | Day 145 | All 8 STAR stories, Days 141–144 |
| Google's Googleyness & Emergent Leadership | Day 145 | Competency Map, above |
| Databricks' Leadership Principles (6, verified against current public sourcing — the plan itself names only 2) | Day 145 | Competency Map, above |
| Atlassian's Five Values | Day 145 | Competency Map, above |
| 2-Minute Cold-Recall self-check structure | Day 146 | — |
| Resume bullet construction (technique + target + quantified outcome) | Day 146 | STAR Result-quantification discipline (Day 141) |
| GitHub portfolio presentation (pinning strategy, README anatomy) | Day 147 | — |
| Job search strategy (referral mechanism, follow-up cadence, status tracking) | Day 147 | — |

**Zero new DSA problems, zero new LLD/HLD systems this week** — confirmed directly against `Week_21_Revised.md`'s full content, matching the shape already set by Weeks 16–20. This is the series' final week; there is no Week 22 Additions subsection to follow this one.

## Complete Problem Inventory — Everything Solved So Far

This is the table a future week's generation checks before teaching or adding anything. It will keep growing — every future week appends to it rather than starting a new one.

| Week | Day | LC # | Problem | Type | Pattern / Variant |
|---|---|---|---|---|---|
| 1 | 4 | 20 | Valid Parentheses | Required (plan) | Stack — LIFO (previews Week 6's formal Stack pattern) |
| 1 | 4 | 682 | Baseball Game | Extra Practice | Stack |
| 1 | 4 | 232 | Implement Queue using Stacks | Extra Practice | Stack — Two-Stack |
| 1 | 4 | 1047 | Remove All Adjacent Duplicates In String | Extra Practice | Stack |
| 1 | 5 | 1 | Two Sum | Required (plan) | HashMap — Complement Lookup |
| 1 | 5 | 217 | Contains Duplicate | Required (plan) | HashSet — Membership |
| 1 | 5 | 242 | Valid Anagram | Required (plan) | Frequency Counting |
| 1 | 5 | 383 | Ransom Note | Required (plan) | Frequency Counting |
| 1 | 5 | 205 | Isomorphic Strings | Required (plan) | Two-Way Mapping |
| 1 | 5 | 49 | Group Anagrams | Required (plan) | HashMap Keyed by Canonical Form |
| 1 | 5 | 128 | Longest Consecutive Sequence | Required (plan) | HashSet, Smart Starting Point |
| 1 | 5 | 169 | Majority Element | Extra Practice | Frequency Counting |
| 1 | 5 | 349 | Intersection of Two Arrays | Extra Practice | Membership |
| 1 | 5 | 387 | First Unique Character in a String | Extra Practice | Frequency Counting |
| 1 | 5 | 290 | Word Pattern | Extra Practice | Two-Way Mapping |
| 1 | 5 | 560 | Subarray Sum Equals K | Extra Practice | Prefix Sum + Lookup (previews Week 4's formal Prefix Sum pattern) |
| 1 | 6 | 125 | Valid Palindrome | Required (plan) | Two Pointers — Opposite Ends |
| 1 | 6 | 344 | Reverse String | Required (plan) | Two Pointers — Opposite Ends |
| 1 | 6 | 88 | Merge Sorted Array | Required (plan) | Two Pointers — From the Back |
| 1 | 6 | 167 | Two Sum II (Input Array Is Sorted) | Extra Practice | Two Pointers — Opposite Ends |
| 1 | 6 | 977 | Squares of a Sorted Array | Extra Practice | Two Pointers — Opposite Ends (From the Back) |
| 1 | 6 | 26 | Remove Duplicates from Sorted Array | Extra Practice | Two Pointers — Same Direction, Different Speeds |
| 1 | 7 | 392 | Is Subsequence | Required (plan) | Two Pointers — One Forward Pointer Each |
| 1 | 7 | 680 | Valid Palindrome II | Required (plan) | Two Pointers — Opposite Ends, One Allowed Skip |
| 1 | 7 | 283 | Move Zeroes | Extra Practice | Two Pointers — Same Direction (Swap) |
| 1 | 7 | 11 | Container With Most Water | Extra Practice | Two Pointers — Opposite Ends, Provable Greedy |
| 1 | 7 | 15 | 3Sum | Extra Practice | Synthesis — HashMap-Era Thinking + Two Pointers |
| 1 | 7 | 27 | Remove Element | Extra Practice | Two Pointers — Same Direction, No Sortedness Required |
| 2 | 8 | 80 | Remove Duplicates from Sorted Array II | Extra Practice | Two Pointers — Same Direction (Fast-Slow, Bounded Duplicates) |
| 2 | 9 | 633 | Sum of Square Numbers | Extra Practice | Two Pointers — Opposite Ends (Implicit Range) |
| 2 | 10 | 16 | 3Sum Closest | Required (plan) | Sorting + Two Pointers |
| 2 | 11 | 18 | 4Sum | Required (plan) | Sorting + Two Pointers |
| 2 | 11 | 881 | Boats to Save Most People | Required (plan) | Two Pointers — Opposite Ends, Greedy Pairing |
| 2 | 12 | 75 | Sort Colors | Required (plan) | Two Pointers — Three-Way Partition (Dutch National Flag) |
| 2 | 13 | 42 | Trapping Rain Water | Required (plan) | Two Pointers — Opposite Ends (Running-Max Witness Argument) |
| 2 | 14 | 121 | Best Time to Buy and Sell Stock | Required (plan) | Sliding Window — Single-Pass Min-Tracking (Implicit Window) |
| 2 | 14 | 643 | Maximum Average Subarray I | Required (plan) | Sliding Window — Fixed Size |
| 3 | 15 | 1004 | Max Consecutive Ones III | Required (plan) | Sliding Window — Variable Size (Violation Counter) |
| 3 | 15 | 3 | Longest Substring Without Repeating Characters | Required (plan) | Sliding Window — Variable Size (HashSet) |
| 3 | 15 | 1695 | Maximum Erasure Value | Extra Practice | Sliding Window — Variable Size (HashSet + Running Sum) |
| 3 | 16 | 424 | Longest Repeating Character Replacement | Required (plan) | Sliding Window — Variable Size (Frequency Array, Stale Max) |
| 3 | 16 | 567 | Permutation in String | Required (plan) | Sliding Window — Fixed Size (Frequency Array Match) |
| 3 | 16 | 1052 | Grumpy Bookstore Owner | Extra Practice | Sliding Window — Fixed Size (Gain Maximization) |
| 3 | 17 | 438 | Find All Anagrams in a String | Required (plan) | Sliding Window — Fixed Size (Frequency Array, Collect All) |
| 3 | 17 | 209 | Minimum Size Subarray Sum | Required (plan) | Sliding Window — Variable Size (Shrink While Valid) |
| 3 | 18 | 904 | Fruit Into Baskets | Required (plan) | Sliding Window — At-Most-K-Distinct (K=2) |
| 3 | 18 | 1493 | Longest Subarray of 1's After Deleting One Element | Required (plan) | Sliding Window — Variable Size (Mandatory-Deletion Adjustment) |
| 3 | 18 | 1838 | Frequency of the Most Frequent Element | Extra Practice | Sliding Window — Variable Size (Sort + Budget Constraint) |
| 3 | 19 | 340 | Longest Substring with At Most K Distinct Characters | Required (plan) | Sliding Window — At-Most-K-Distinct (General K) |
| 3 | 19 | 76 | Minimum Window Substring | Required (plan) | Sliding Window — Variable Size (Required/Formed Coverage) |
| 3 | 20 | 239 | Sliding Window Maximum | Required (plan) | Sliding Window — Monotonic Deque (Fixed Size) |
| 3 | 20 | 992 | Subarrays with K Different Integers | Required (plan) | Sliding Window — Exactly-K (At-Most-K Subtraction) |
| 3 | 20 | 1438 | Longest Continuous Subarray With Absolute Diff ≤ Limit | Extra Practice | Sliding Window — Variable Size (Two Monotonic Deques) |
| 3 | 21 | 303 | Range Sum Query - Immutable | Required (plan) | Prefix Sum |
| 3 | 21 | 53 | Maximum Subarray | Required (plan) | Kadane's Algorithm |
| 3 | 21 | 238 | Product of Array Except Self | Required (plan) | Prefix Product × Suffix Product |
| 4 | 22 | 525 | Contiguous Array | Required (plan) | Prefix Sum + HashMap, First-Occurrence Index (±1 transform) |
| 4 | 22 | 523 | Continuous Subarray Sum | Required (plan) | Prefix Sum + HashMap, First-Occurrence Index (mod k) |
| 4 | 22 | 918 | Maximum Subarray Sum Circular | Required (plan) | Kadane's, Run Twice (total − min) |
| 4 | 22 | 974 | Subarray Sums Divisible by K | Extra Practice | Prefix Sum + HashMap, Frequency Count (mod k, negative-remainder fix) |
| 4 | 22 | 152 | Maximum Product Subarray | Extra Practice | Kadane's, Running Max AND Min |
| 4 | 23 | 122 | Best Time to Buy and Sell Stock II | Required (plan) | Greedy — Telescoping Decomposition |
| 4 | 23 | 55 | Jump Game | Required (plan) | Greedy — Frontier Domination |
| 4 | 23 | 45 | Jump Game II | Required (plan) | Greedy — BFS by Levels (Frontier Domination Extended) |
| 4 | 24 | 134 | Gas Station | Required (plan) | Greedy — Prefix-Elimination Argument |
| 4 | 24 | 56 | Merge Intervals | Required (plan) | Intervals — Sort by Start, Combine Overlapping |
| 4 | 24 | 57 | Insert Interval | Required (plan) | Intervals — Sort by Start, 3-Phase O(n) |
| 4 | 24 | 406 | Queue Reconstruction by Height | Extra Practice | Greedy — Two-Key Sort |
| 4 | 25 | 435 | Non-overlapping Intervals | Required (plan) | Intervals — Sort by End, Greedy Keep Earliest |
| 4 | 25 | 452 | Minimum Number of Arrows to Burst Balloons | Required (plan) | Intervals — Sort by End, Identical Shape to LC 435 |
| 4 | 25 | 986 | Interval List Intersections | Extra Practice | Intervals — Two Pointers, Two Separate Lists |
| 4 | 26 | 252 | Meeting Rooms | Required (plan) | Intervals — Existence Check (Sort by Start) |
| 4 | 26 | 253 | Meeting Rooms II | Required (plan) | Intervals — Min-Heap of End Times (Monotonic Growth) |
| 4 | 27 | 763 | Partition Labels | Required (plan) | Intervals, Implicit — Last-Occurrence Index |
| 4 | 28 | 704 | Binary Search | Required (plan) | Binary Search — Exact Match, On the Input |
| 4 | 28 | 278 | First Bad Version | Required (plan) | Binary Search — Boundary Search, On the Answer |
| 5 | 29 | 35 | Search Insert Position | Required (plan) | Binary Search — Exact Match, On the Input (Insertion Point) |
| 5 | 29 | 162 | Find Peak Element | Required (plan) | Binary Search — On the Input, Local Slope Rule (No Global Sort) |
| 5 | 30 | 33 | Search in Rotated Sorted Array | Required (plan) | Binary Search — Rotated Array, One Half Always Sorted |
| 5 | 30 | 81 | Search in Rotated Sorted Array II | Required (plan) | Binary Search — Rotated Array, With Duplicates |
| 5 | 31 | 153 | Find Minimum in Rotated Sorted Array | Required (plan) | Binary Search — Rotated Array Minimum |
| 5 | 31 | 154 | Find Minimum in Rotated Sorted Array II | Extra Practice | Binary Search — Rotated Array Minimum, With Duplicates |
| 5 | 31 | 34 | Find First and Last Position of Element in Sorted Array | Required (plan) | Binary Search — Boundary Search (Two-Sided) |
| 5 | 31 | 744 | Find Smallest Letter Greater Than Target | Extra Practice | Binary Search — Boundary Search (One-Sided) |
| 5 | 32 | 74 | Search a 2D Matrix | Required (plan) | Binary Search — 2D Flattened to 1D |
| 5 | 32 | 875 | Koko Eating Bananas | Required (plan) | Binary Search — On the Answer, Feasibility (Minimize) |
| 5 | 33 | 1011 | Capacity To Ship Packages Within D Days | Required (plan) | Binary Search — On the Answer, Feasibility (Minimize) |
| 5 | 33 | 1482 | Minimum Number of Days to Make m Bouquets | Extra Practice | Binary Search — On the Answer, Feasibility (Adjacency-Aware) |
| 5 | 33 | 1552 | Magnetic Force Between Two Balls | Extra Practice | Binary Search — On the Answer, Feasibility (Maximize) |
| 5 | 34 | 206 | Reverse Linked List | Required (plan) | Linked List — Iterative Pointer Reversal (+ Recursive Alternative) |
| 5 | 34 | 876 | Middle of the Linked List | Required (plan) | Linked List — Fast/Slow Pointers |
| 5 | 35 | 234 | Palindrome Linked List | Required (plan) | Linked List — Fast/Slow + Reversal, Combined |
| 5 | 35 | 21 | Merge Two Sorted Lists | Required (plan) | Linked List — Two Pointers with Dummy Head |
| 5 | 35 | 92 | Reverse Linked List II | Extra Practice | Linked List — Bounded Reversal (Dummy Head) |
| 5 | 35 | 23 | Merge k Sorted Lists | Extra Practice | Linked List — k-Way Merge (Heap OR Divide-and-Conquer) |
| 6 | 36 | 141 | Linked List Cycle | Required (plan) | Linked List — Floyd's Cycle Detection |
| 6 | 36 | 142 | Linked List Cycle II | Required (plan) | Linked List — Floyd's Cycle Detection, Extended |
| 6 | 36 | 202 | Happy Number | Extra Practice | Floyd's Cycle Detection, Generalized (No Linked List) |
| 6 | 36 | 287 | Find the Duplicate Number | Extra Practice | Floyd's Cycle Detection, Applied to an Array as Implicit Pointers |
| 6 | 37 | 19 | Remove Nth Node From End of List | Required (plan) | Linked List — Two Pointers (Fixed Offset) |
| 6 | 37 | 2 | Add Two Numbers | Required (plan) | Linked List — Math Simulation |
| 6 | 38 | 143 | Reorder List | Required (plan) | Linked List — Fast/Slow + Reversal + Merge (Combined) |
| 6 | 38 | 138 | Copy List with Random Pointer | Required (plan) | Linked List — HashMap Node Mapping |
| 6 | 39 | 146 | LRU Cache | Required (plan) | HashMap + Doubly Linked List (closes Linked Lists) |
| 6 | 40 | 496 | Next Greater Element I | Required (plan) | Monotonic Stack (Decreasing) — opens the pattern |
| 6 | 41 | 503 | Next Greater Element II | Required (plan) | Monotonic Stack, Circular Array |
| 6 | 41 | 155 | Min Stack | Required (plan) | Stack — Two Stacks (Running Min) |
| 6 | 41 | 901 | Online Stock Span | Extra Practice | Monotonic Stack, Streaming/Stateful (Persists Across Calls) |
| 6 | 41 | 402 | Remove K Digits | Extra Practice | Monotonic Stack (Increasing), Greedy |
| 6 | 42 | 739 | Daily Temperatures | Required (plan) | Monotonic Stack (Decreasing), Index-Based |
| 6 | 42 | 150 | Evaluate Reverse Polish Notation | Required (plan) | Stack — General Purpose (LIFO Is the Algorithm) |
| 7 | 43 | 735 | Asteroid Collision | Required (plan) | Stack Simulation |
| 7 | 43 | 227 | Basic Calculator II | Required (plan) | Stack — General Purpose (lastSign Trick) |
| 7 | 44 | 84 | Largest Rectangle in Histogram | Required (plan) | Monotonic Stack (Strictly Increasing), Index-Based |
| 7 | 44 | 85 | Maximal Rectangle | Required (plan) | Monotonic Stack, Applied Row-by-Row (Reduces to LC 84) |
| 7 | 45 | 224 | Basic Calculator | Required (plan) | Stack — General Purpose (Saved Context Across Parens) |
| 7 | 46 | 104 | Maximum Depth of Binary Tree | Required (plan) | Tree DFS — Postorder-Style Combine |
| 7 | 46 | 226 | Invert Binary Tree | Required (plan) | Tree DFS — Swap and Recurse |
| 7 | 47 | 100 | Same Tree | Required (plan) | Tree DFS — Two-Tree Structural Comparison |
| 7 | 47 | 101 | Symmetric Tree | Required (plan) | Tree DFS — Mirrored Comparison |
| 7 | 47 | 572 | Subtree of Another Tree | Extra Practice | Tree DFS — Composes isSameTree as a Subroutine |
| 7 | 48 | 110 | Balanced Binary Tree | Required (plan) | Tree DFS — Bottom-Up, Sentinel Short-Circuit |
| 7 | 48 | 543 | Diameter of Binary Tree | Required (plan) | Tree DFS — Running Max Outside Return Value |
| 7 | 48 | 111 | Minimum Depth of Binary Tree | Extra Practice | Tree DFS — One-Child-Is-Not-a-Leaf Trap |
| 7 | 49 | 102 | Binary Tree Level Order Traversal | Required (plan) | Tree BFS — queue.size() Level Isolation |
| 7 | 49 | 199 | Binary Tree Right Side View | Required (plan) | Tree BFS — Last Node per Level |
| 7 | 49 | 637 | Average of Levels in Binary Tree | Extra Practice | Tree BFS — Aggregate Instead of Collect |
| 8 | 50 | 98 | Validate Binary Search Tree | Required (plan) | BST — DFS with Inherited (min,max) Boundaries |
| 8 | 50 | 230 | Kth Smallest Element in a BST | Required (plan) | BST — Inorder Traversal for Rank |
| 8 | 50 | 173 | BST Iterator | Extra Practice | BST — Controlled Inorder via Explicit Stack |
| 8 | 51 | 235 | Lowest Common Ancestor of a BST | Required (plan) | BST — Ordering-Based Single Direction |
| 8 | 51 | 105 | Construct Binary Tree from Preorder and Inorder Traversal | Required (plan) | Divide and Conquer — HashMap-Accelerated |
| 8 | 51 | 106 | Construct Binary Tree from Inorder and Postorder Traversal | Extra Practice | Divide and Conquer, Mirrored |
| 8 | 52 | 236 | Lowest Common Ancestor of a Binary Tree | Required (plan) | Tree DFS — Postorder Combine (General, No Invariant) |
| 8 | 52 | 297 | Serialize and Deserialize Binary Tree | Required (plan) | DFS Preorder + Explicit Null Markers |
| 8 | 53 | 4 | Median of Two Sorted Arrays | Required (plan) | Binary Search on a Computed Partition Point |
| 8 | 54 | 703 | Kth Largest Element in a Stream | Required (plan) | Fixed-Size Min-Heap, Maintained Across Calls |
| 8 | 54 | 1046 | Last Stone Weight | Required (plan) | Max-Heap, Repeated Combine |
| 8 | 55 | 215 | Kth Largest Element in an Array | Required (plan) | Min-Heap of Size k / Quickselect |
| 8 | 55 | 973 | K Closest Points to Origin | Required (plan) | Max-Heap of Size k, Custom Comparator (Squared Distance) |
| 8 | 55 | 1167 | Minimum Cost to Connect Sticks | Extra Practice | Min-Heap, Repeated Combine (Cost-Accounting Exchange) |
| 8 | 55 | 378 | Kth Smallest Element in a Sorted Matrix | Extra Practice | Heap of Size k / Binary Search on Value + Staircase Count |
| 8 | 56 | 347 | Top K Frequent Elements | Required (plan) | HashMap + Min-Heap (Bucket Sort Alternative Noted) |
| 8 | 56 | 264 | Ugly Number II | Required (plan) | Min-Heap as Candidate Generator |
| 8 | 56 | 451 | Sort Characters By Frequency | Extra Practice | HashMap + Max-Heap, Frequency Output |
| 9 | 57 | 767 | Reorganize String | Required (plan) | Max-Heap by Frequency, Greedy Scheduling |
| 9 | 57 | 621 | Task Scheduler | Required (plan) | Max-Heap + Cooldown Queue (+ O(1)-Space Closed Form) |
| 9 | 58 | 295 | Find Median from Data Stream | Required (plan) | Two Heaps, Size-Balanced |
| 9 | 58 | 23 | Merge k Sorted Lists | Required (plan) | PriorityQueue, k-Way Merge Coordinator |
| 9 | 59 | 208 | Implement Trie (Prefix Tree) | Required (plan) | Trie — Core Operations |
| 9 | 59 | 677 | Map Sum Pairs | Required (plan) | Trie — Cumulative Value via Delta Propagation |
| 9 | 60 | 720 | Longest Word in Dictionary | Required (plan) | Trie + DFS, Constrained Descent |
| 9 | 60 | 211 | Design Add and Search Words Data Structure | Required (plan) | Trie + DFS, Branching Descent (Wildcard) |
| 9 | 61 | 648 | Replace Words | Required (plan) | Trie, Constrained Descent (Dictionary Substitution) |
| 9 | 61 | 212 | Word Search II | Required (plan) | Trie + Matrix Backtracking |
| 9 | 61 | 78 | Subsets | Required (plan) | Backtracking — Include/Exclude |
| 9 | 62 | 46 | Permutations | Required (plan) | Backtracking — Swap-Based, Position-Dependent |
| 9 | 63 | 77 | Combinations | Required (plan) | Backtracking — Forward-Index Recursion |
| 9 | 63 | 47 | Permutations II | Required (plan) | Backtracking with Duplicate Handling |
| 10 | 64 | 39 | Combination Sum | Required (plan) | Backtracking — Forward-Index + Repetition |
| 10 | 64 | 40 | Combination Sum II | Required (plan) | Backtracking — Forward-Index + Duplicate Skip (i>start) |
| 10 | 65 | 17 | Letter Combinations of a Phone Number | Required (plan) | Backtracking — String-Building, External Lookup |
| 10 | 65 | 22 | Generate Parentheses | Required (plan) | Backtracking — String-Building, Counter-Gated |
| 10 | 66 | 79 | Word Search | Required (plan) | Backtracking — Matrix DFS, Overwrite-Restore (recap of Wk9 D61's mechanism) |
| 10 | 66 | 131 | Palindrome Partitioning | Required (plan) | Backtracking — Forward-Index Over String Positions |
| 10 | 67 | 90 | Subsets II | Required (plan) | Backtracking — Include/Exclude + Duplicate Skip (i>start) |
| 10 | 67 | 51 | N-Queens | Required (plan) | Backtracking — Row-by-Row + Derived Constraints |
| 10 | 68 | 200 | Number of Islands | Required (plan) | Graph DFS/BFS — Matrix Flood Fill |
| 10 | 68 | 695 | Max Area of Island | Required (plan) | Graph DFS — Matrix Flood Fill, Aggregated Return |
| 10 | 69 | 133 | Clone Graph | Required (plan) | Graph DFS + HashMap (Cycle-Safe) |
| 10 | 69 | 785 | Is Graph Bipartite? | Required (plan) | Graph BFS — 2-Coloring |
| 10 | 69 | 802 | Find Eventual Safe States | Extra Practice | Graph DFS — 3-State Coloring (Directed Cycle Detection) |
| 10 | 70 | 994 | Rotting Oranges | Required (plan) | Graph — Multi-Source BFS |
| 10 | 70 | 797 | All Paths From Source to Target | Required (plan) | Graph DFS on a DAG (Backtracking, No Visited Needed) |
| 10 | 70 | 542 | 01 Matrix | Extra Practice | Graph — Multi-Source BFS |
| 11 | 71 | 207 | Course Schedule | Required (plan) | Topological Sort — Kahn's Algorithm / Cycle Detection |
| 11 | 71 | 210 | Course Schedule II | Required (plan) | Topological Sort — Construction |
| 11 | 72 | 130 | Surrounded Regions | Required (plan) | Multi-Source DFS/BFS — Reachability From Border |
| 11 | 72 | 417 | Pacific Atlantic Water Flow | Required (plan) | Multi-Source DFS/BFS — Reversed Reachability |
| 11 | 73 | 127 | Word Ladder | Required (plan) | BFS Shortest Path — Implicit Graph, Single-Source |
| 11 | 73 | 1334 | Find the City With the Smallest Number of Neighbors at a Threshold Distance | Required (plan) | Floyd-Warshall — All-Pairs Shortest Path |
| 11 | 74 | 684 | Redundant Connection | Required (plan) | Union-Find |
| 11 | 74 | 547 | Number of Provinces | Required (plan) | Union-Find / DFS |
| 11 | 75 | 990 | Satisfiability of Equality Equations | Required (plan) | Union-Find — Two-Pass (Equality, Then Inequality) |
| 11 | 75 | 947 | Most Stones Removed with Same Row or Column | Required (plan) | Union-Find — Row/Column as Node |
| 11 | 75 | 1319 | Number of Operations to Make Network Connected | Extra Practice | Union-Find — Component Counting |
| 11 | 76 | 1202 | Smallest String With Swaps | Required (plan) | Union-Find — Index Position as Node |
| 11 | 76 | 721 | Accounts Merge | Required (plan) | Union-Find + HashMap Grouping |
| 11 | 77 | 261 | Graph Valid Tree | Required (plan) | Union-Find |
| 12 | 78 | 743 | Network Delay Time | Required (plan) | Dijkstra's Algorithm |
| 12 | 78 | 1514 | Path with Maximum Probability | Required (plan) | Dijkstra — Multiplicative, Max-Heap |
| 12 | 79 | 787 | Cheapest Flights Within K Stops | Required (plan) | Bellman-Ford — K-Round Relaxation |
| 12 | 79 | 1631 | Path With Minimum Effort | Required (plan) | Dijkstra — Minimax, Edge-Weighted |
| 12 | 80 | 778 | Swim in Rising Water | Required (plan) | Dijkstra — Minimax, Node-Weighted |
| 12 | 81 | 70 | Climbing Stairs | Required (plan) | 1D DP |
| 12 | 81 | 746 | Min Cost Climbing Stairs | Required (plan) | 1D DP |
| 12 | 82 | 198 | House Robber | Required (plan) | 1D DP — Take-or-Skip |
| 12 | 82 | 213 | House Robber II | Required (plan) | 1D DP — Circular Reduction |
| 12 | 82 | 740 | Delete and Earn | Extra Practice | 1D DP — House Robber via Value-Bucket Transform |
| 12 | 83 | 91 | Decode Ways | Required (plan) | 1D DP — Validity-Gated |
| 12 | 83 | 152 | Maximum Product Subarray | Required (plan) — **recapped, not newly solved; see Problems Recapped in Week 12 below** | Kadane's, Running Max AND Min |
| 12 | 84 | 139 | Word Break | Required (plan) | 1D DP + Set |
| 12 | 84 | 322 | Coin Change | Required (plan) | 1D DP — Unbounded Knapsack |
| 13 | 85 | 518 | Coin Change II | Required (plan) | 1D DP — Unbounded Knapsack, Combinations |
| 13 | 85 | 377 | Combination Sum IV | Extra Practice | 1D DP — Unbounded Knapsack, Permutations |
| 13 | 85 | 300 | Longest Increasing Subsequence | Required (plan) | 1D DP / Binary Search |
| 13 | 85 | 673 | Number of Longest Increasing Subsequence | Extra Practice | 1D DP — LIS + Count Tracking |
| 13 | 85 | 416 | Partition Equal Subset Sum | Required (plan) | 0/1 Knapsack DP |
| 13 | 86 | 494 | Target Sum | Required (plan) | 0/1 Knapsack DP — Counting |
| 13 | 86 | 279 | Perfect Squares | Required (plan) | 1D DP — Unbounded Knapsack |
| 13 | 86 | 140 | Word Break II | Required (plan) | 1D DP + Backtracking (DP-Gated) |
| 13 | 87 | 62 | Unique Paths | Required (plan) | 2D (Grid) DP |
| 13 | 87 | 63 | Unique Paths II | Required (plan) | 2D DP with Obstacles |
| 13 | 87 | 64 | Minimum Path Sum | Required (plan) | 2D (Grid) DP |
| 13 | 87 | 221 | Maximal Square | Required (plan) | 2D Matrix DP |
| 13 | 87 | 174 | Dungeon Game | Extra Practice | 2D DP — Reverse-Direction Sweep |
| 13 | 88 | 1143 | Longest Common Subsequence | Required (plan) | 2D String DP |
| 13 | 88 | 583 | Delete Operation for Two Strings | Extra Practice | 2D String DP — Direct LCS Application |
| 13 | 88 | 72 | Edit Distance | Required (plan) | 2D String DP |
| 13 | 88 | 5 | Longest Palindromic Substring | Required (plan) | Expand Around Center |
| 13 | 89 | 647 | Palindromic Substrings | Required (plan) | Expand Around Center |
| 13 | 89 | 516 | Longest Palindromic Subsequence | Required (plan) | 2D String DP — LCS(s, reverse(s)) |
| 13 | 89 | 97 | Interleaving String | Required (plan) | 2D String DP — Three-String |
| 13 | 90 | 115 | Distinct Subsequences | Required (plan) | 2D String DP — Counting |
| 13 | 90 | 44 | Wildcard Matching | Required (plan) | 2D DP — Pattern Matching |
| 13 | 90 | 10 | Regular Expression Matching | Required (plan) | 2D DP — Pattern Matching |
| 13 | 90 | 132 | Palindrome Partitioning II | Extra Practice | 1D DP Gated by Range-Validity Table |
| 13 | 91 | 312 | Burst Balloons | Required (plan) | Interval DP — Last-Operation Reformulation |
| 13 | 91 | 1312 | Minimum Insertion Steps to Make a String Palindrome | Required (plan) | Interval DP — Reduces to LPS |
| 13 | 91 | 486 | Predict the Winner | Extra Practice | Interval DP — Choose-From-Either-End |
| 14 | 92 | 714 | Best Time to Buy and Sell Stock with Transaction Fee | Required (plan) | State Machine DP — 2-State (Hold/Cash) |
| 14 | 92 | 309 | Best Time to Buy and Sell Stock with Cooldown | Required (plan) | State Machine DP — 3-State (Hold/Sold/Rest) |
| 14 | 93 | 123 | Best Time to Buy and Sell Stock III | Required (plan) | State Machine DP — Transaction-Count Dimension |
| 14 | 93 | 188 | Best Time to Buy and Sell Stock IV | Required (plan) | State Machine DP — Transaction-Count Dimension, Generalized |
| 14 | 94 | 337 | House Robber III | Required (plan) | Tree DP — Dual Return-Value States |
| 14 | 94 | 124 | Binary Tree Maximum Path Sum | Required (plan) | Tree DP — Running Max Outside Return Value |
| 14 | 94 | 968 | Binary Tree Cameras | Extra Practice | Tree DP — 3-State Greedy Covering |
| 14 | 95 | 136 | Single Number | Required (plan) | Bit Manipulation — XOR Cancellation |
| 14 | 95 | 191 | Number of 1 Bits | Required (plan) | Bit Manipulation — n&(n-1) |
| 14 | 96 | 231 | Power of Two | Required (plan) | Bit Manipulation — n&(n-1)==0 |
| 14 | 96 | 338 | Counting Bits | Required (plan) | Bit Manipulation Fused With 1D DP |
| 14 | 97 | 268 | Missing Number | Required (plan) | Bit Manipulation — XOR Cancellation |
| 14 | 97 | 137 | Single Number II | Required (plan) | Bit Manipulation — Per-Bit Frequency Counting (Mod 3) |
| 14 | 97 | 461 | Hamming Distance | Extra Practice | Bit Manipulation — XOR + Bit-Counting Composition |
| 14 | 98 | 260 | Single Number III | Required (plan) | Bit Manipulation — Isolate, Partition, XOR |
| 14 | 98 | 371 | Sum of Two Integers | Required (plan) | Bit Manipulation — Ripple-Carry Simulation |
| 15 | 99 | 190 | Reverse Bits | Required (plan) | Bit Manipulation — 32-bit Shift-and-Append |
| 15 | 99 | 421 | Maximum XOR of Two Numbers in an Array | Required (plan) | Bit Trie — Greedy Opposite-Bit Walk (closes Bit Manip + Tries) |
| 15 | 100 | 307 | Range Sum Query - Mutable | Required (plan) | Segment Tree — Point Update / Range Sum |
| 15 | 101 | 315 | Count of Smaller Numbers After Self | Required (plan) | Segment Tree (value-indexed) / Fenwick Tree — Coordinate Compression |
| 15 | 103 | 176 | Second Highest Salary | Required (plan) | SQL — Subquery / LIMIT-OFFSET |
| 15 | 103 | 184 | Department Highest Salary | Required (plan) | SQL — Correlated Subquery |
| 15 | 103 | 182 | Duplicate Emails | Required (plan) | SQL — GROUP BY / HAVING |
| 15 | 103 | 197 | Rising Temperature | Required (plan) | SQL — Self-Join (Date Offset) |
| 15 | 103 | 181 | Employees Earning More Than Their Managers | Required (plan) | SQL — Self-Join (Hierarchical) |
| 15 | 104 | 177 | Nth Highest Salary | Required (plan) | SQL — Window Function (DENSE_RANK) |
| 15 | 104 | 180 | Consecutive Numbers | Required (plan) | SQL — Window Function (LAG) |
| 15 | 104 | 262 | Trips and Users | Required (plan) | SQL — Conditional Aggregation (CASE) |
| 15 | 104 | 626 | Exchange Seats | Required (plan) | SQL — CASE / Parity Logic |
| 15 | 104 | 185 | Department Top Three Salaries | Required (plan) | SQL — Window Function (DENSE_RANK, Partitioned) |

**Week 2 total: 9 newly-solved problems** — 7 required by the plan, 2 added as extra practice — across Two Pointers (7) and Sliding Window (2).

**Week 3 total: 19 newly-solved problems** — 15 required by the plan, 4 added as extra practice — across Sliding Window (16, closing the pattern) and Prefix Sum & Kadane's (3, opening the pattern). Zero problems recapped — see the Week 3 Overlap section below for the confirmation.

**Week 4 total: 20 newly-solved problems** — 16 required by the plan, 4 added as extra practice — across Prefix Sum & Kadane's (5: LC 525, 523, 918 required + LC 974, 152 extra — closing the pattern), Greedy & Intervals (13: 11 required + LC 406, 986 extra — opening *and* closing the pattern in the same week), and Binary Search (2 required, opening the pattern). One additional required slot (LC 560) was fulfilled by recap, not a new solve — see the Problems Recapped in Week 4 table below.

**Week 5 total: 19 newly-solved problems** — 13 required by the plan, 6 added as extra practice — across Binary Search (13: 9 required + 4 extra — closing the pattern at 11 required / 15 distinct combined with Week 4) and Linked Lists (6: 4 required + 2 extra — opening the pattern, continues in Week 6). Zero problems recapped — see the Week 5 Overlap section below for the confirmation. LC 240 was discussed for contrast on Day 32 but deliberately not given full problem treatment (no code, trace, or edge-case list), so it is **not** counted in this table or in any total above.

**Week 6 total: 16 newly-solved problems** — 12 required by the plan, 4 added as extra practice — across Linked Lists (9: 7 required + 2 extra — closing the pattern at 11 required / 15 distinct combined with Week 5) and Stacks/Monotonic Stack (7: 5 required + 2 extra — opening the pattern at 7/12 required, continues in Week 7). Two additional required slots (LC 20, LC 232) were fulfilled by recap, not new solves — see the Problems Recapped in Week 6 table below.

**Week 7 total: 16 newly-solved problems** — 13 required by the plan, 3 added as extra practice — across Stacks/Monotonic Stack (5: all required — closing the pattern at 12 required / 14 distinct combined with Weeks 1 and 6) and Trees (11: 8 required + 3 extra — opening the pattern at 8/15 required, continues in Week 8). Zero problems recapped — see the Week 7 Overlap section below for the confirmation.

**Week 8 total: 18 newly-solved problems** — 13 required by the plan, 5 added as extra practice — across Trees (7 required + 2 extra — closing the pattern at 15 required / 20 distinct combined with Week 7, including the Day 21-promised Median of Two Sorted Arrays finally delivered) and Heaps (6 required + 3 extra — opening the pattern at 6/10 required, continues in Week 9). Zero problems recapped — see the Week 8 Overlap section below for the confirmation.

**Week 9 total: 14 newly-solved problems — all 14 required by the plan, zero added as extra practice** — across Heaps (4 required, Days 57–58, closing the pattern at 10/10 — 13/13 distinct combined with Week 8's 3 extra), Tries (6 required, Days 59–61, opening *and* closing its own core within the same week — the first pattern in the series to do this), and Backtracking (4 required, Days 61–63 — Subsets, Permutations, Combinations, Permutations II — opening the pattern, continues in Week 10). Zero problems recapped — see the Week 9 Overlap section below for the confirmation. This is the first week in the series where **no pattern received any extra practice at all**; each of the three zero-extra outcomes has its own independent justification (see Day 58, Day 61, and Day 63's closing notes respectively) rather than a single blanket policy.

**Week 10 total: 16 newly-solved problems — 14 required by the plan, 2 added as extra practice** — across Backtracking (8 required, Days 64–67 — Combination Sum, Combination Sum II, Letter Combinations of a Phone Number, Generate Parentheses, Word Search, Palindrome Partitioning, Subsets II, N-Queens — **closing the pattern at 12/12 required, 0 extra, 12 distinct combined with Week 9**) and Graphs BFS/DFS (6 required + 2 extra — LC 802 Day 69, LC 542 Day 70, both checked against `Week_11_Revised.md` and confirmed absent — **opening the pattern at 6/12 required, continues in Week 11**). Zero problems recapped — every one of Week 10's 14 required problems was confirmed absent from the cumulative table above before being taught in full; see the Week 10 Overlap section below for the confirmation. Backtracking's zero-extra outcome (Days 64–67) has its own explicit justification (Day 67's closing note: a dense, already-comprehensive 12-problem required ladder spanning six distinct choice models, with no thin spot left to fill) rather than defaulting to the no-extra pattern by habit.

**Week 11 total: 14 newly-solved problems — 13 required by the plan, 1 added as extra practice** — across Graphs BFS/DFS (6 required, Days 71–73 — Course Schedule, Course Schedule II, Surrounded Regions, Pacific Atlantic Water Flow, Word Ladder, Find the City With the Smallest Number of Neighbors at a Threshold Distance — **closing the pattern at 12/12 required, 14 distinct combined with Week 10**) and Union-Find (7 required + 1 extra — LC 1319, Day 75, checked against `Week_12_Revised.md` and confirmed absent — **opening *and* closing the pattern within the same week, at 7/7 required, 8 distinct**). Zero problems recapped — every one of Week 11's 13 required problems was confirmed absent from the cumulative table above before being taught in full; see the Week 11 Overlap section below for the confirmation. Union-Find's zero-extra opening day (Day 74) has its own explicit justification, matching every prior pattern's true-opening-day precedent exactly (see Day 74's closing note); its one deferred extra landed Day 75 instead, once the pattern was a day past opening.

**Week 12 total: 13 newly-solved problems — 12 required by the plan, 1 added as extra practice** — across Dijkstra's Algorithm (5 required, Days 78–80 — Network Delay Time, Path with Maximum Probability, Cheapest Flights Within K Stops, Path With Minimum Effort, Swim in Rising Water — **opening *and* closing the pattern within the same week, at 5/5 required, 5 distinct, zero extra**) and Dynamic Programming (8 required — including 1 recap — + 1 extra, Days 81–84 — **opening the pattern at 7/33 genuinely-new-required + 1 extra = 8 distinct, continues in Week 13**). One required slot (LC 152) was fulfilled by recap, not a new solve — see the Problems Recapped in Week 12 table below. This is the first week in the series where a *required* problem, not an extra, turned out to already be solved — caught by checking `Week_12_Revised.md`'s required list against this table's cumulative row-by-row content before treating anything as new, the same check direction Week 4 and Week 6's recaps used, just landing on the opposite side (there, an extra collided with a later required list; here, a required problem collided with an earlier extra). Dijkstra's zero-extra outcome across its entire run has its own explicit justification (see Day 80's closing note: the required ladder was already deliberately expanded by this week's own plan revision specifically to close a known gap, and the five problems already span every distinct relaxation shape the pattern tests) rather than defaulting to the opening-day-only precedent every other pattern has followed.

**Week 13 total: 27 newly-solved problems — 21 required by the plan, 6 added as extra practice** — entirely within Dynamic Programming, Days 85–91: 1D DP finishes (6 required — Coin Change II, LIS, Partition Equal Subset Sum, Target Sum, Perfect Squares, Word Break II — +2 extra, Combination Sum IV and Number of LIS — closing the pattern at **14/14 required**, up from the original plan's 12), Grid DP opens and closes the same day (4 required + 1 extra, Dungeon Game — **4/4 required**), String DP opens and closes across three days (9 required — including Regular Expression Matching, upgraded from optional extension to fully scheduled — + 2 extra, Delete Operation for Two Strings and Palindrome Partitioning II — **9/9 required**), and Interval DP opens and closes on the final day (2 required + 1 extra, Predict the Winner — **2/2 required**). Zero recaps needed anywhere this week — every one of Week 13's 21 required problems was confirmed new against the full inventory before being taught, and all six extras were checked against both the inventory and `Week_14_Revised.md`'s required list before being added (see the Week 13 and Week 14 Overlap sections below). This is the first week in the series where every extra-practice pick's specific purpose is named individually rather than defaulting to "one more rep of the same pattern" — each of the six closes a distinct, identified gap (a loop-order contrast, a thin-pattern second recurrence shape, a reversed-direction variant, or a near-zero-cost direct reduction), per this week's own generation prompt explicitly asking for judgment over padding.

**Week 14 total: 16 newly-solved problems — 14 required by the plan, 2 added as extra practice** — across Dynamic Programming and Bit Manipulation, Days 92–98: State Machine DP opens and closes within the week (4 required, Days 92–93 — Transaction Fee, Cooldown, then Stock III and Stock IV — **4/4 required, 0 extra**, reasoning for the zero stated explicitly rather than left silent) and Tree DP opens and closes the same day (2 required + 1 extra, Binary Tree Cameras, Day 94 — **2/2 required**, joining Grid DP's and Interval DP's Week 13 precedent for single-day patterns) — together **closing Dynamic Programming entirely at 35/35 required across Weeks 12–14** — and Bit Manipulation opens (Days 95–98: Single Number, Number of 1 Bits, Power of Two, Counting Bits, Missing Number, Single Number II, Single Number III, Sum of Two Integers — 8 required — + 1 extra, Hamming Distance, Day 97 — **8/10 required, continues in Week 15**). Zero recaps needed anywhere this week, the second consecutive week to close that way — every one of Week 14's 14 required problems was confirmed new against the full inventory before being taught, and both extras were checked against the inventory and `Week_15_Revised.md`'s required list before being added (see the Week 14 and Week 15 Overlap sections below). One deliberate reordering, flagged where it happened, unlike any prior week: Transaction Fee taught before Cooldown on Day 92, reversed from the plan's own stated sequence, so the state count escalates from the pattern's simplest case outward.

**Week 15 total: 14 newly-solved problems — all 14 required by the plan, zero added as extra practice** — across Bit Manipulation, Tries, Segment Trees, and SQL, Days 99–104: Bit Manipulation's final two required problems (Reverse Bits, Maximum XOR of Two Numbers in an Array, Day 99 — **closing the pattern at 10/10 required, 11 distinct combined with Week 14's 1 extra**), with Maximum XOR simultaneously closing Tries (**7/7 required, 0 extra, 7 distinct**, its 7th problem deferred from Week 9 exactly as flagged); Segment Trees opens and closes within the week (2 required, Days 100–101 — Range Sum Query - Mutable, Count of Smaller Numbers After Self — **2/2 required, 0 extra, 2 distinct**); and the SQL practice track, entirely additive to the original plan (10 required, Days 103–104 — Second Highest Salary, Department Highest Salary, Duplicate Emails, Rising Temperature, Employees Earning More Than Their Managers, Nth Highest Salary, Consecutive Numbers, Trips and Users, Exchange Seats, Department Top Three Salaries — **10/10 required, 0 extra, track complete**). One required slot (LC 215, Kth Largest Element in an Array, Day 102) was fulfilled by a short recap, not a new solve — Quickselect was taught in full depth as the day's genuinely new content instead; see the Problems Recapped in Week 15 table below. This is the third week in the series (after Week 9 and, functionally, Week 12's Dijkstra's-only portion) where **zero extra practice was added anywhere** — every pattern's zero has its own independent, explicitly-stated justification rather than a blanket policy: Bit Manipulation's is a closing-day non-event (its one extra already landed in Week 14); Segment Trees' and the SQL track's are both the plan's own deliberately-fixed-scope framing, honored directly. This week's own DSA-required count (197, this map's row-by-row table) diverges from `Week_15_Revised.md`'s own stated "203" — flagged here per this map's established convention rather than silently propagated; see the Week 15 Consolidation note below for the full reconciliation.

### Problems Recapped in Week 2 (Not Newly Solved — Already in the Table Above)

These six required slots in `Week_02_Revised.md` were fulfilled by a short recap, not a re-solve, because the problem was already done in Week 1. Listed here so the trail is visible; **not** counted again in any total.

| LC # | Problem | Originally solved | Recapped at |
|---|---|---|---|
| 26 | Remove Duplicates from Sorted Array | Week 1, Day 6 (Extra) | Week 2, Day 8 |
| 283 | Move Zeroes | Week 1, Day 7 (Extra) | Week 2, Day 8 |
| 977 | Squares of a Sorted Array | Week 1, Day 6 (Extra) | Week 2, Day 9 |
| 167 | Two Sum II | Week 1, Day 6 (Extra) | Week 2, Day 9 |
| 15 | 3Sum | Week 1, Day 7 (Extra) | Week 2, Day 10 |
| 11 | Container With Most Water | Week 1, Day 7 (Extra) | Week 2, Day 12 |

### Problems Recapped in Week 4 (Not Newly Solved — Already in the Table Above)

One required slot in `Week_04_Revised.md` was fulfilled by a recap, flagged in advance in the Week 4 Overlap section below (originally noted back when Week 2 was generated) and confirmed during Week 4's actual generation. **Not** counted again in any total.

| LC # | Problem | Originally solved | Recapped at |
|---|---|---|---|
| 560 | Subarray Sum Equals K | Week 1, Day 5 (Extra) | Week 4, Day 22 |

### Problems Recapped in Week 6 (Not Newly Solved — Already in the Table Above)

Two required slots in `Week_06_Revised.md` were fulfilled by recap, not a re-solve — both flagged in advance in the Week 6 Overlap section below (originally noted back when Week 1 was generated) and confirmed during Week 6's actual generation. **Not** counted again in any total.

| LC # | Problem | Originally solved | Recapped at |
|---|---|---|---|
| 20 | Valid Parentheses | Week 1, Day 4 (Required) | Week 6, Day 39 |
| 232 | Implement Queue using Stacks | Week 1, Day 4 (Extra) | Week 6, Day 40 |

### Problems Recapped in Week 12 (Not Newly Solved — Already in the Table Above)

One required slot in `Week_12_Revised.md` was fulfilled by a recap, not a re-solve — this one was **not** flagged in advance, unlike the Week 4 and Week 6 recaps above, which were both earmarked in an earlier week's Overlap section before the colliding week was even generated. This is the first recap in the series caught only during the colliding week's own generation, by checking the required list directly against this table's cumulative contents, per the standard "before treating a required problem as new" step. **Not** counted again in any total.

| LC # | Problem | Originally solved | Recapped at |
|---|---|---|---|
| 152 | Maximum Product Subarray | Week 4, Day 22 (Extra) | Week 12, Day 83 |

### Problems Recapped in Week 15 (Not Newly Solved — Already in the Table Above)

One required slot in `Week_15_Revised.md` was fulfilled by a recap, flagged in advance — this map's own Week 14 entry named LC 215 explicitly as an already-solved required problem before Week 15 was generated (see "the Tries entry above" cross-reference in Week 14's Bit Manipulation bullet, and the Week 15 Overlap section below), continuing the Week 4/Week 6 precedent of an advance flag rather than Week 12's catch-during-generation. Unlike every prior recap in this table, this one received **full depth on a second technique** (Quickselect) applied to the same problem, rather than a bare short recap alone — a deliberate asymmetric treatment stated explicitly in Day 102's own Overlap Notice. **Not** counted again in any total.

| LC # | Problem | Originally solved | Recapped at |
|---|---|---|---|
| 215 | Kth Largest Element in an Array | Week 8, Day 55 (Required) | Week 15, Day 102 |

**Week 16 total: 0 newly-solved DSA problems — 0 required, 0 extra.** The series' first week to close this way, and correctly so: Week 16 introduced no new DSA pattern (the DSA phase closed entirely as of Day 105), so there was nothing for the plan to require and nothing thin enough to need extra reps. What Week 16 *did* add — four complete LLD systems and six cold-solve DSA revisions of already-taught problems — is tracked in the two new sections immediately below, deliberately kept separate from this table rather than forced into its required/extra columns, which were built for a different kind of content.

### DSA Revision Log — Week 16 (Cold Spaced-Repetition Re-Solves, Not Newly Solved — Already in the Table Above)

New this week: five of Week 16's seven days each carried a "solve one problem cold, without hints" DSA revision block, drawing on already-closed patterns for spaced-repetition practice rather than new pattern coverage. Distinct from a **recap** (a plan-required problem that turns out to already be solved, handled above) — a revision is a *deliberate*, planned re-solve of a problem the plan never required again, chosen by the Resource Book generation itself. Logged here, explicitly, so a future week's generation can avoid re-selecting the same problems for its own revision or extra-practice slots — Week 17's own plan (`Week_17_Revised.md`) carries revision blocks in three of these same six patterns (Backtracking, Graph, DP — see the new Known Overlap With Week 17 section below), making this log directly load-bearing, not just historical record. **Not** counted again in any total.

| Day | LC # | Problem | Pattern | Originally solved |
|---|---|---|---|---|
| 107 | 207 | Course Schedule | Graph — Topological Sort (Kahn's) | Week 11, Day 71 |
| 107 | 322 | Coin Change | DP — Unbounded Knapsack | Week 12, Day 84 |
| 108 | 39 | Combination Sum | Backtracking — Forward-Index + Repetition | Week 10, Day 64 |
| 109 | 3 | Longest Substring Without Repeating Characters | Sliding Window, Variable-Size | Week 3, Day 15 |
| 110 | 98 | Validate Binary Search Tree | Tree DFS — Inherited (min,max) Boundaries | Week 8, Day 50 |
| 111 | 684 | Redundant Connection | Union-Find | Week 11, Day 74 |

### DSA Revision Log — Week 17 (Cold Spaced-Repetition Re-Solves, Not Newly Solved)

Six of Week 17's seven days each carried a revision block (Day 119 ran a self-check instead — see its entry above). **The Backtracking, Graph, and DP picks below are confirmed different from Week 16's three** (Combination Sum/LC 39, Course Schedule/LC 207, Coin Change/LC 322) — the constraint this map's own Week 16 extension flagged ahead of time, resolved as follows.

| Day | LC # | Problem | Pattern | Originally solved |
|---|---|---|---|---|
| 113 | 973 | K Closest Points to Origin | Heap — Bounded Max-Heap of Size k | Week 8, Day 55 |
| 114 | 211 | Design Add and Search Words Data Structure | Trie — Wildcard Branching | Week 9, Day 60 |
| 115 | 22 | Generate Parentheses | Backtracking — Counter-Gated | Week 9, Day 61 |
| 116 | 133 | Clone Graph | Graph DFS + Visited Map | Weeks 10–11 |
| 117 | 300 | Longest Increasing Subsequence | 1D DP (O(n²) and O(n log n) forms) | Week 12 |
| 118 | 72 | Edit Distance | 2D String DP | Week 13 |

**No collision with Week 16's revision picks, by LC number or by pattern-repeat-of-the-same-problem** — confirmed directly against the table above. Whoever generates Week 18 should treat both this table and Week 16's as the full "already used for revision" pool — Week 18's own Day 125 self-check names Backtracking-or-Trie explicitly, which touches two of this week's six patterns directly (see the new Known Overlap With Week 18 section below).


### DSA Revision Log — Week 18 (Cold Spaced-Repetition Re-Solve, Not Newly Solved — Already in the Table Above)

| Day | LC # | Problem | Pattern | Originally solved |
|---|---|---|---|---|
| 126 | 40 | Combination Sum II | Backtracking — Forward-Index + Duplicate-Skip | Week 10, Day 64 |

> ⚠️ **Discrepancy flagged, not silently resolved either way:** this map's own pre-Week-18 forward-looking notes (the "Known Overlap With Week 18" and "How to Use This Document for Week 18" sections, both dated before `Week_18_Revised.md` reached its final form) refer to this self-check as **"Day 125's."** `Week_18_Revised.md` itself places the self-check on **Day 126**, immediately before the BookMyShow-at-Scale theory block. Day 126's Resource Book follows the actual plan file and flags the discrepancy explicitly on the page; this entry does the same — consistent with how this map has handled its own past day-labeling discrepancies (Week 16's Days 108–109 vs. actual 107–111; the Day 95–97 Kubernetes-adjacent question) rather than silently picking a side.
>
> Confirmed clear against all three prior revision picks in these two patterns: not LC 22 (Generate Parentheses, drawn Week 17, Day 115), not LC 211 (Design Add and Search Words, drawn Week 17, Day 114), not LC 39 (Combination Sum, drawn Week 16, Day 108). Also confirmed clear against `Week_19_Revised.md`, which contains no LeetCode-numbered problems at all — Week 19 continues the HLD phase directly, so there was no risk of this pick colliding with anything Week 19 requires.

### DSA Revision Log — Week 19 (Cold Spaced-Repetition Re-Solves, Not Newly Solved — Already in the Table Above)

| Day | LC # | Problem | Pattern | Originally solved |
|---|---|---|---|---|
| 131 | 721 | Accounts Merge | Union-Find + HashMap Grouping | Week 11, Day 76 |
| 131 | 1143 | Longest Common Subsequence | 2D String DP | Week 13, Day 88 |

> Both picks respond directly to `Week_19_Revised.md`'s own stated Uber-specific advice ("practice DSU (Union-Find), DP, and classic CS problems, be ruthless about time complexity") rather than being chosen arbitrarily. Both confirmed absent from the DSA Revision Log — Weeks 16, 17, and 18 above before selection (Union-Find's only prior revision pick was LC 684; Dynamic Programming's were LC 322, LC 300, and LC 72 — none overlapping with today's picks), and both confirmed present in the Complete Problem Inventory table above as genuinely already-taught problems (LC 721: Week 11, Day 76; LC 1143: Week 13, Day 88) before being recommended for cold revision, per this map's own standing rule.

**No DSA Revision Log entry for Week 20 or Week 21 — for two different reasons, worth distinguishing.** Week 20 had none because its content was infrastructure/operations work with no DSA angle to revise. Week 21 is different: Days 146 and 148 both include a cold-recall self-check that explicitly covers "one DSA pattern... at random," but `Week_21_Revised.md` deliberately leaves the specific pattern unpicked — an open, candidate-selects-at-random exercise rather than a predetermined revision pick baked into the day's content the way every entry in this log above was. Day 146's Resource Book offers a reasoned menu of strong candidates (every pattern absent from this log: Two Pointers, Prefix Sum & Kadane's, Greedy & Intervals, Binary Search, Linked Lists, Stacks/Monotonic Stack, Dijkstra's, Bit Manipulation, Segment Trees, and Sorting-from-scratch) rather than a single fixed pick, so no new row is added here — logging a specific problem as "the Week 21 revision pick" would misrepresent an open self-check as a scripted one.

### Running Totals

- **Week 1:** 28 problems — 15 required by the plan, 13 extra practice — across Stack (preview), HashMap/HashSet, Two Pointers.
- **Week 2:** 9 newly-solved problems (above) + 6 recapped (already counted in Week 1's 28, not added again).
- **Week 3:** 19 newly-solved problems (above) + 0 recapped (none found — see the Week 3 Overlap section below).
- **Week 4:** 20 newly-solved problems (above) + 1 recapped (LC 560, already counted in Week 1's 28, not added again).
- **Week 5:** 19 newly-solved problems (above) + 0 recapped (none found — see the Week 5 Overlap section below).
- **Week 6:** 16 newly-solved problems (above) + 2 recapped (LC 20, LC 232, both already counted in Week 1's 28, not added again).
- **Week 7:** 16 newly-solved problems (above) + 0 recapped (none found — see the Week 7 Overlap section below).
- **Week 8:** 18 newly-solved problems (above) + 0 recapped (none found — see the Week 8 Overlap section below).
- **Cumulative distinct problems solved, Weeks 1–8: 145.** (28 + 9 + 19 + 20 + 19 + 16 + 16 + 18. See Day 56's Week 8 Consolidation for the full planned-vs-actual breakdown, including why this differs from `Week_08_Revised.md`'s own internal "110 problems" Day-56 scorecard figure — that figure tracks only the plan's required ladder (97 through Week 7, per Day 49's own note, + 13 in Week 8) and doesn't include any extra practice from this series, exactly the same convention every prior week's own scorecard figure used. The two figures are cross-checked directly against each other in Day 56's Consolidation: 97 + 13 required slots = 110, matching the plan's own claim exactly.)
- **Cumulative distinct problems solved, Weeks 1–9: 159.** (145 + Week 9's 14, all required, zero extra. See Day 63's Week 9 Consolidation for the full breakdown. **A genuine discrepancy, flagged rather than silently absorbed:** `Week_09_Revised.md`'s own Day 63 scorecard states "123 total DSA problems solved," which — following the same required-ladder-only convention every prior week's internal scorecard has used (110 through Week 8, confirmed above) — implies Week 9 contributed 13 required problems. Counting Week 9's own day-by-day problem list directly (Days 57–63: 2+2+2+2+3+1+2) gives **14**, not 13; Day 61 alone carries three problems (Replace Words, Word Search II, Subsets), one more than every other day this week, which is the most likely place a manually-maintained running total drifted by one during the plan's own revision. This map's row-by-row table above is the authoritative count — 14 required rows for Week 9 — and every total in this document uses that figure; the plan's own "123" is one problem short of what its own day-by-day content actually lists, and is noted here rather than propagated.)
- **Cumulative distinct problems solved, Weeks 1–10: 175.** (159 + Week 10's 16 — 14 required + 2 extra. See Day 70's Week 10 Consolidation for the full breakdown. This specific bullet was itself missing from this list prior to the Week 11 extension — the 175 figure was already in circulation via Day 70's own Consolidation note and the Week 11 Overlap section, just not previously restated here; added now so the sequence is unbroken.)
- **Cumulative distinct problems solved, Weeks 1–11: 189.** (175 + Week 11's 14 — 13 required + 1 extra. See Day 77's Week 11 Consolidation for the full breakdown, including the flagged 150-vs-151 discrepancy in `Week_11_Revised.md`'s own internal scorecard — traced back to the same Week 9 Day 63 drift documented immediately above, evidently carried forward uncorrected through both Week 10's and Week 11's own sequentially-incremented tallies rather than recomputed from this map's authoritative figures.)
- **Cumulative distinct problems solved, Weeks 1–12: 202.** (189 + Week 12's 13 — 12 required + 1 extra; the week's 13th required slot, LC 152, was a recap and isn't counted twice. See Day 84's Week 12 Consolidation for the full breakdown, including the flagged 163-vs-164 discrepancy in `Week_12_Revised.md`'s own internal scorecard — the same Week 9 Day 63 drift, now carried forward through four consecutive weeks' worth of internal tallies without ever being reconciled against this map's authoritative row-by-row count. Note this cumulative-distinct figure and the required-ladder-only figure diverge for the first time by more than a fixed offset this week specifically because of the recap: 164 required-ladder slots vs. 202 cumulative-distinct solves is not a typo — the two conventions are answering different questions, and Day 84's Consolidation states both side by side rather than picking one.)
- **Two Pointers, fully closed:** 16/16 required (5 Week 1 + 11 Week 2) + 3 genuine extras beyond the required ladder (LC 27, LC 80, LC 633) = 19 distinct Two Pointers problems solved in total.
- **Sliding Window, fully closed:** 14/14 required (2 Week 2 + 12 Week 3) + 4 genuine extras, all Week 3 (LC 1695, LC 1052, LC 1838, LC 1438) = 18 distinct Sliding Window problems solved in total.
- **Prefix Sum & Kadane's, fully closed:** 7/7 required (3 Week 3 + 4 Week 4) + 2 genuine extras, both Week 4 (LC 974, LC 152) = 9 distinct problems solved in total.
- **Greedy & Intervals, fully closed:** 11/11 required (all Week 4) + 2 genuine extras, both Week 4 (LC 406, LC 986) = 13 distinct problems solved in total. Opened and closed within the same week — the first pattern in this series to do so.
- **Binary Search, fully closed:** 11/11 required (2 Week 4 + 9 Week 5) + 4 genuine extras, all Week 5 (LC 154, LC 744, LC 1482, LC 1552) = 15 distinct Binary Search problems solved in total. Split close to evenly between the two framings established Day 28 and formalized Day 33 — 9 "on the input," 6 "on the answer" (counting LC 278) — see Day 33's full 15-problem classification table if a refresher is ever needed.
- **Linked Lists, fully closed:** 11/11 required (4 Week 5 + 7 Week 6) + 4 extras (2 Week 5 Day 35: LC 92, LC 23 — bounded reversal, k-way merge; 2 Week 6 Day 36: LC 202, LC 287 — Floyd's Cycle Detection generalized beyond linked lists) = 15 distinct Linked List problems solved in total. Extras were deliberately withheld on Day 34, the pattern's actual opening day — mirroring exactly how Sliding Window (Week 2, Day 14), Prefix Sum & Kadane's (Week 3, Day 21), and Binary Search (Week 4, Day 28) each opened — then added on Day 35 and again on Day 36, since Binary Search's own precedent (extras landing mid-pattern, not on an opening day) confirms extras are fair game once a pattern is no longer opening fresh.
- **Stacks/Monotonic Stack, fully closed:** 12/12 required (7 Week 6 — including 2 recapped from Week 1, LC 20 and LC 232 — + 5 Week 7) + 2 extras, both Week 6 Day 41 (LC 901, LC 402) = 14 distinct Stack problems solved in total. Stacks opened Day 39 and Monotonic Stack (its sub-pattern) opened Day 40 — each correctly received no extras on its own opening day, the same treatment Linked Lists received Day 34; Monotonic Stack's deferred extras landed Day 41 instead, and no further extras were added once Week 7 closed the pattern out at its Hard tier. Day 45 gives a full two-family classification (general-purpose LIFO vs. true monotonic invariant) across all 14.
- **Trees, fully closed:** 15/15 required (8 Week 7 + 7 Week 8) + 5 extras (3 Week 7 — Day 47: LC 572; Day 48: LC 111; Day 49: LC 637 — + 2 Week 8 — Day 50: LC 173; Day 51: LC 106) = 20 distinct Tree problems solved in total. Trees opened Day 46 and correctly received no extras on its own opening day, the same treatment every prior pattern's true opening day has received (Sliding Window Day 14, Prefix Sum & Kadane's Day 21, Linked Lists Day 34, Stacks Day 39, Monotonic Stack Day 40); its five extras spread across Days 47, 48, 49, 50, and 51 rather than stacked on any single day. Closed Day 53, spanning DFS, BFS, BST traversal and validation, divide-and-conquer construction, general-tree LCA, and serialization — and separately closing the Median of Two Sorted Arrays gap the original plan's Day 21 left open, 99 days late in that plan's own numbering. See Day 53's full 20-problem classification table if a refresher is ever needed.
- **Heaps, fully closed:** 10/10 required (6 Week 8 + 4 Week 9) + 3 extras, all Week 8 (Day 55: LC 1167, LC 378; Day 56: LC 451) = **13 distinct Heap problems solved in total.** Heaps opened Day 54 and correctly received no extras on its own opening day, the same treatment every prior pattern's true opening day has received; its deferred extras landed Days 55–56 instead, and none were added in Week 9 — by the time Week 9 picked the pattern back up, six distinct heap roles were already demonstrated (kth-largest, keep-the-k-best, minimize-combination-cost, candidate-generator — all Week 8 — plus greedy-scheduling and two-heap-balance/k-way-merge — both Week 9), which Day 58's own closing note treats as sufficient without argued need for a seventh. Two old debts resolved while opening this pattern: Quickselect (Day 55), named as a future destination back on Day 12's Sort Colors, and the O(m+n) "staircase" technique (Day 55's Kth Smallest in a Sorted Matrix extra), named for LC 240 on Day 32 and explicitly left without full treatment at the time. Closed Week 9, Day 58 (LC 767, 621, 295, 23 — Days 57–58). See the Week 8 and Week 9 Overlap sections below.
- **Tries, core opened and closed within Week 9, pattern fully closed as of Week 15:** 6/6 core required (all Week 9) + 0 extra, plus a 7th and final required problem, deliberately deferred, delivered in Week 15 (Maximum XOR of Two Numbers in an Array, LC 421, Day 99 — a Bit Trie, fusing this pattern with Bit Manipulation) = **7/7 required, 0 extra, 7 distinct Trie problems solved in total — the pattern's full and final scope.** Week 9's run was the first pattern in this series both to open and close its own core inside a single week, and the first to receive zero extra practice at any point within that week — Day 61's closing note gave the two-part reasoning at the time (the required six already span every major Trie role; the pattern was too short-lived within Week 9 to ever reach a genuine non-opening reinforcement day, unlike Trees or Heaps). The 7th problem was correctly held back rather than pulled forward despite fitting thematically at the time — mirroring exactly how House Robber III and Binary Tree Maximum Path Sum were deliberately held back from Trees for Dynamic Programming (see the Trees entry above) — and delivered exactly where flagged, once Bit Manipulation's own mechanics were far enough along to support it. See Day 99's Part 2 for the full Bit Trie treatment.
- **Backtracking, fully closed:** 12/12 required (4 Week 9: Subsets, Permutations, Combinations, Permutations II + 8 Week 10: Combination Sum, Combination Sum II, Letter Combinations of a Phone Number, Generate Parentheses, Word Search, Palindrome Partitioning, Subsets II, N-Queens) + 0 extra = **12 distinct Backtracking problems solved in total.** Backtracking opened Day 61 and correctly received no extras on its own opening day, the same treatment every prior pattern's true opening day has received — and, unusually among this series' patterns, received **no extras on any day at all**, opening through close (Days 61–67), a seven-day run of zero-extra outcomes each independently justified rather than defaulted to (Day 63's three-part reasoning while still mid-opening; Day 67's closing reasoning once the full 12-problem ladder was visible: six distinct choice models — include/exclude, swap-based, forward-index bare/with-repetition/with-duplicate-skip, string-building via lookup, string-building via counter, grid-DFS, row-by-row with derived constraints — already comprehensively covered, no thin spot left to fill). One real collision was caught and avoided during this run, not hypothetical: LC 17 was seriously considered as a Week 9 extra, checked against `Week_10_Revised.md`, and confirmed to already be Week 10 Day 65's required problem. Closed Day 67. See Day 67's full classification table for a refresher, and the Week 9 and Week 10 Overlap sections below.
- **Graphs BFS/DFS, fully closed:** 12/12 required (6 Week 10: Number of Islands, Max Area of Island, Clone Graph, Is Graph Bipartite?, Rotting Oranges, All Paths From Source to Target + 6 Week 11: Course Schedule, Course Schedule II, Surrounded Regions, Pacific Atlantic Water Flow, Word Ladder, Find the City With the Smallest Number of Neighbors at a Threshold Distance) + 2 extras, both Week 10 (Day 69: LC 802 Find Eventual Safe States; Day 70: LC 542 01 Matrix) = **14 distinct Graphs problems solved in total.** Graphs opened Day 68 and correctly received no extras on its own opening day, the same treatment every prior pattern's true opening day has received; its two extras landed Days 69–70 instead, once the pattern was past opening — mirroring Heaps' Day 54→55–56 precedent and Monotonic Stack's Day 40→41 precedent exactly. Both extras were checked against `Week_11_Revised.md` and confirmed absent from its required list before being added. This is the first pattern in the series requiring an explicit `visited` structure (a tree cannot cycle; a graph can) — and, within the same week it opened, the first to demonstrate that rule is *conditioned* on cycles being possible rather than unconditional (Day 70's All Paths From Source to Target correctly omits `visited` entirely, given a guaranteed-acyclic input). Week 11 closed the pattern across four genuinely distinct final roles: topological ordering/cycle-detection (Kahn's and DFS 3-state coloring, Day 71), multi-source reachability both forward and reversed (Day 72), single-source BFS on an implicit graph (Word Ladder, Day 73), and a deliberate, bounded exposure to a non-traversal algorithm family entirely (Floyd-Warshall, Day 73). Closed Day 73. See Day 73's Theory Block for the full BFS-vs-DFS-vs-neither classification across all 12 required problems, and the Week 10 and Week 11 Overlap sections below.
- **Union-Find, opened and closed within the same week:** 7/7 required (all Week 11: Redundant Connection, Number of Provinces — Day 74; Satisfiability of Equality Equations, Most Stones Removed with Same Row or Column — Day 75; Smallest String With Swaps, Accounts Merge — Day 76; Graph Valid Tree — Day 77) + 1 extra, Day 75 (LC 1319, Number of Operations to Make Network Connected — deliberately earmarked back in Week 10's own Day 69 note, picked up here rather than introduced early) = **8 distinct Union-Find problems solved in total.** Union-Find opened Day 74 and correctly received no extras on its own opening day, the same treatment every prior pattern's true opening day has received; its one deferred extra landed Day 75 instead, once the pattern was one day past opening. A brand-new data structure taught fully from scratch — naive O(n) worst case, union-by-rank alone proven O(log n) via a rank-doubling induction, path compression illustrated via a concrete chain-collapse example, combined O(α(n)) amortized — then applied across seven genuinely distinct surfaces: redundant-edge detection, static component counting (contrasted directly with DFS), two-pass equation satisfiability, a row/column-as-node removal-ordering proof, index-position rearrangement (with a full permutation-achievability proof), email-identity merging via a HashMap grouping pattern shared structurally with Day 5, and a tree-validity check reusing the `n-1`-edges-plus-acyclic graph theory fact. Immediately reused, unmodified, as Kruskal's Algorithm's cycle-detection subroutine the same day it closed (Day 77) — the pattern's close and its first real-world reuse landing on the same day. Closed Day 77. See Day 77's Concept Card and the Week 11 Overlap section below.
- **Dijkstra's Algorithm, opened and closed within the same week:** 5/5 required (all Week 12: Network Delay Time, Path with Maximum Probability — Day 78; Cheapest Flights Within K Stops, Path With Minimum Effort — Day 79; Swim in Rising Water — Day 80) + 0 extra = **5 distinct Dijkstra's problems solved in total.** Dijkstra's opened Day 78 and correctly received no extras on its own opening day, the same treatment every prior pattern's true opening day has received — and, unusually, received no *deferred* extras once past opening either, unlike every other pattern in the series: the required ladder itself was expanded from 3 to 5 problems by this week's own plan revision specifically to close a gap the original audit flagged, and the five problems already span the pattern's full range of relaxation shapes — a textbook sum-minimizing shortest path, a multiplicative/max-heap flip of the same shape, a hop-constrained variant needing an entirely different algorithm (Bellman-Ford), and two minimax variants (one edge-weighted, one node-weighted), each independently paired with the same genuinely distinct alternative approach (Binary Search on the Answer + Union-Find). Reused Prim's Algorithm's exact loop shape from the day before it opened (Day 77), and reused the `PriorityQueue` machinery from Days 17, 26, and 54 unmodified. Closed Day 80. See Day 80's closing note for the full zero-extra reasoning, and the Week 12 Overlap section below.
- **Dynamic Programming, opened — the largest pattern in the plan, continues into Week 13:** 7/33 genuinely-new-required (+ 1 recapped slot, LC 152) + 1 extra = **8 distinct Dynamic Programming problems solved so far** (Climbing Stairs, Min Cost Climbing Stairs — Day 81; House Robber, House Robber II — Day 82; Delete and Earn, extra — Day 82; Decode Ways — Day 83; Maximum Product Subarray, recapped from Week 4 — Day 83; Word Break, Coin Change — Day 84). Dynamic Programming opened Day 81 and correctly received no extras on its own opening day, the same treatment every prior pattern's true opening day has received; its one extra (Delete and Earn, reinforcing House Robber's family with a third, differently-shaped variant) landed Day 82 instead, once the pattern was one day past opening. The formal introduction was built on two concepts already informally previewing the same shape without the name — Kadane's Algorithm (Week 3, Day 21) and Floyd-Warshall (Week 11, Day 73) — both cited directly rather than re-derived, per this map's own Week 11 handoff note. Coin Change (Day 84) is the pattern's first genuinely *unbounded* problem — unlimited reuse of a single "item" (a coin denomination), a constraint no problem before it in the series has had. Continues into Week 13 (leave week 2), which closes four more DP subtypes entirely: the rest of 1D DP, Grid DP, String DP, and Interval DP. See Day 81's Concept Card, Day 84's Week 12 Consolidation, and the Week 12 Overlap section below.
- **Cumulative distinct problems solved, Weeks 1–13: 229.** (202 + Week 13's 27 — 21 required + 6 extra. See Day 91's Week 13 Consolidation for the full breakdown. **This bullet was itself missing from this list prior to the Week 14 extension** — the 229 figure was already in circulation via Day 91's own Consolidation note and the Week 14 Overlap section, just not previously restated here; added now so the sequence is unbroken, the same gap this list caught and fixed once before — Week 11 catching Week 10's own missing bullet, above.)
- **Cumulative distinct problems solved, Weeks 1–14: 245.** (229 + Week 14's 16 — 14 required + 2 extra. See Day 98's Week 14 Consolidation for the full breakdown.)
- **Dynamic Programming, closed entirely as of Week 14 — continuing the Day 84 bullet above rather than replacing it:** Week 13 closed the remaining three-quarters of 1D DP (14/14 required total, up from the original plan's 12) plus Grid DP (4/4 + 1 extra, opened and closed Day 87), String DP (9/9 + 3 extra, Days 88–90), and Interval DP (2/2 + 1 extra, opened and closed Day 91) — 21 required + 6 extra = 27 new problems, zero recaps needed. Week 14 closed the final two subtypes: State Machine DP (4/4 + 0 extra, Days 92–93, reasoning for the zero stated explicitly) and Tree DP (2/2 + 1 extra, Day 94) — 6 required + 1 extra = 7 new problems. **Dynamic Programming's final total: 35/35 required across Weeks 12–14 (34 newly taught + 1 recap, LC 152) + 8 extra (Delete and Earn, Combination Sum IV, Number of LIS, Dungeon Game, Delete Operation for Two Strings, Palindrome Partitioning II, Predict the Winner, Binary Tree Cameras) = 43 distinct Dynamic Programming problems solved in total**, across six genuinely distinct subtypes (1D, Grid, String, Interval, State Machine, Tree), each opened with its state defined precisely before any recurrence was written, and each recurrence proven by exhaustive, disjoint cases rather than asserted — the single discipline Day 94's own closing synthesis names as the actual throughline running under all six. See Day 94's 6-subtype synthesis table for the full breakdown, and the Week 14 Overlap section below.
- **Bit Manipulation, fully closed as of Week 15:** 10/10 required (8 Week 14 + 2 Week 15: Reverse Bits, Maximum XOR of Two Numbers in an Array — Day 99) + 1 extra, Week 14 Day 97 (LC 461, Hamming Distance) = **11 distinct Bit Manipulation problems solved in total.** Bit Manipulation opened Week 14 Day 95 and correctly received no extras on its own opening day, the eighth consecutive genuinely-new top-level pattern in the series to receive that treatment; its one deferred extra landed Day 97, two days past opening, since Day 96 was judged to already carry sufficient new material (Counting Bits, a Bit-Manipulation/DP fusion) without adding practice on top; no further extras were added upon closing in Week 15, reasoned through explicitly rather than defaulted to (see Day 99's closing note). Built fully from zero: two's complement derived from Week 2, Day 10's overflow-wraps proof, all seven bitwise operators, XOR's algebraic properties proven by truth table, `n&(n-1)` and `n&(-n)` both derived rather than stated, per-bit frequency counting generalized past mod-2, and closing with full 32-bit reversal (`>>>`'s sign-extension-avoidance now load-bearing rather than just proven) and a genuine two-pattern fusion (Bit Trie) rather than a same-pattern reinforcement. See Day 99's Part 1–2 for the closing two problems.
- **Segment Trees, opened and closed within the same week:** 2/2 required (both Week 15: Range Sum Query - Mutable, Day 100; Count of Smaller Numbers After Self, Day 101) + 0 extra = **2 distinct Segment Tree problems solved in total — the pattern's full and deliberately-light scope**, per the plan's own explicit framing (genuine exposure to a rare-but-real topic, not exhaustive coverage). A brand-new data structure taught fully from scratch — build/update/query all O(log n), the array-backed index math reused directly from Week 8's Heaps — demonstrated across its two genuinely distinct usage shapes: aggregation indexed by array position (Day 100) and aggregation indexed by value, via coordinate compression (Day 101). A Fenwick Tree / BIT alternative was covered as explicitly-marked extension material on Day 101, connecting directly to Week 14's `n & (-n)` identity, without counting toward the pattern's required or extra tallies. Closed Day 101. See Day 100's Part 1 and Day 101's Part 1 and Fenwick Tree extension.
- **Sorting algorithms, built from scratch, opened and closed within the same day (Week 15, Day 102):** no LC-numbered required problems of its own — the deliverable was implementing merge sort and quicksort directly, closing a gap `Arrays.sort()`/`Collections.sort()` usage had left open since Week 1's Group Anagrams (Day 5). One required slot, Kth Largest Element in an Array (LC 215), was fulfilled by recap (see Problems Recapped in Week 15 above) with Quickselect delivered as the day's actual new technique in full depth, reusing the day's own quicksort partition method directly. Not tracked as a "distinct problems" pattern the way the DSA patterns above are, since its practice vehicle was implementation-from-scratch rather than a required LeetCode ladder — see Day 102's Parts 1–4 for the full treatment.
- **SQL practice track, opened and closed within the same week, entirely additive to the original plan:** 10/10 required (5 Week 15 Day 103: Second Highest Salary, Department Highest Salary, Duplicate Emails, Rising Temperature, Employees Earning More Than Their Managers; 5 Week 15 Day 104: Nth Highest Salary, Consecutive Numbers, Trips and Users, Exchange Seats, Department Top Three Salaries) + 0 extra = **10 distinct SQL problems solved in total — the track's full and deliberately-fixed scope**, matching Segment Trees' precedent of an explicitly-bounded, non-open-ended treatment. Extends Week 6, Day 40's SQL fundamentals (JOINs, keys, indexes, the NULL/`NOT IN` trap — not re-taught) with subqueries, correlated subqueries, `GROUP BY`/`HAVING`, two structurally distinct self-join shapes, and the full window-function toolkit (`ROW_NUMBER`/`RANK`/`DENSE_RANK` precisely distinguished on a tied dataset, `LAG`/`LEAD`, `PARTITION BY`, `CASE` expressions and conditional aggregation). Tracked in the same cumulative problem table as the DSA patterns above, but reported separately in totals per the plan's own DSA-vs-SQL framing. Closed Day 104. See Days 103–104 in full.
- **Cumulative distinct DSA problems solved, Weeks 1–15: 249.** (245 + Week 15's 4 new DSA — 2 Bit Manipulation, 1 Segment Tree Day 100, 1 Segment Tree Day 101 — with LC 215's Day 102 slot a recap, not counted again. **A numeric reconciliation, flagged here rather than silently absorbed:** this map's row-by-row table gives **197** required-only DSA problems through Week 15 (192 through Week 14, confirmed by direct re-sum of every week's own "required by the plan" figure above, + 5 Week 15 DSA required slots, including the LC 215 recap slot counted once as required) — not the "203" `Week_15_Revised.md`'s own Day 105 text states. This is very likely the same class of running-total drift already documented for Weeks 9 through 12 above, propagating silently through a manually-maintained figure rather than being recomputed from the actual day-by-day content each time; this map's table is held authoritative here, exactly as in every earlier instance. Both the required-only (197) and cumulative-distinct (249) figures are correct answers to different questions, reported side by side rather than collapsed into one, matching Day 84's established precedent for exactly this situation. **A second, smaller pre-existing discrepancy found while re-summing for this reconciliation, also flagged rather than silently fixed:** the raw row count in the Complete Problem Inventory table above through Week 14 is 246, one more than the stated 245 — traced to LC 23 (Merge k Sorted Lists), which appears twice (Week 5, Day 35, Extra Practice; Week 9, Day 58, Required (plan)) without the second occurrence carrying the same explicit "recapped, not newly solved" annotation LC 152's own duplicate row carries at Week 12, Day 83. Both Week 5's and Week 9's stated week-totals appear to count it as a genuine solve, which is very likely a one-problem overcount somewhere in that pair — not resolved here, since correcting it would mean editing Week 5's or Week 9's own historical row rather than extending forward, contrary to this document's own extend-don't-edit convention; left as a standing open item for whoever next has reason to touch those two weeks' figures directly, the same treatment the still-open Week 13 terminology gap above received.)
- **SQL practice track total (tracked separately from the DSA figure above, per the plan's own framing): 10 distinct problems, entirely additive.**
- **Grand total, every distinct problem solved across the whole series, Weeks 1–15: 259** (249 DSA + 10 SQL). **The DSA phase of this series is complete as of Day 105** — see Day 105's Week 15 Consolidation and Career Block for the full milestone accounting (every pattern closed, every originally-flagged gap fixed, the one broken promise resolved, both non-original-plan pattern families closed). Week 16 onward is LLD/HLD systems, mock interviews, and behavioral prep, built on this foundation rather than extending it further.
- **Grand total, Weeks 1–16: 259 — unchanged.** The series' first week to add zero new DSA or SQL problems, and correctly so: Week 16 introduced no new DSA pattern, so there was nothing for this count to extend. What Week 16 added instead — six cold-solve revisions of already-counted problems, and four complete LLD systems — is tracked in the new DSA Revision Log and LLD System / Design Exercise Inventory sections, deliberately kept outside this DSA/SQL problem count rather than forced into it.
- **Grand total, Weeks 1–17: 259 — unchanged, for the same reason as Week 16.** Week 17 introduced no new DSA pattern either — its six DSA blocks were all spaced-repetition revision of patterns closed weeks 8–13, deliberately, not an oversight (see Day 119's Week 17 Consolidation for the reasoning stated in full). What Week 17 added instead — six more cold-solve revisions and six complete LLD systems, closing the LLD phase at ten systems total — is tracked in the new DSA Revision Log — Week 17 subsection above and the LLD System / Design Exercise Inventory extension below.

---


### Running Totals — After Week 18

DSA problem totals are **unchanged** from the Week 17 running totals above — Week 18 added zero newly-solved DSA problems (it is not a DSA-pattern week, consistent with Weeks 16–17's own precedent) and one cold-revision re-solve (LC 40, Combination Sum II — logged above, not counted as newly solved, per this table's own standing rule).

New this week — System Design (HLD) phase, Week 18 only:
- **HLD systems designed:** 7 (URL Shortener, Rate Limiter formalized, Notification System, Distributed Cache, Distributed ID Generation, Distributed Consensus, BookMyShow at Scale)
- **HLD mocks completed:** 2 (Mock #1 — subject URL Shortener, Day 122; Mock #2 — subject Rate Limiter, Day 125)

Combined LLD + HLD phase totals, Weeks 15–18:
- **Systems designed, LLD + HLD combined:** 17 (10 LLD, Weeks 16–17; 7 HLD, Week 18)
- **Mock interviews, LLD + HLD combined:** 7 (5 LLD-phase mocks, Weeks 16–17; 2 HLD-phase mocks, Week 18)

### Running Totals — After Week 19

DSA problem totals are **unchanged** from the Week 18 running totals above — Week 19 added zero newly-solved DSA problems (continuing Weeks 16–18's own precedent; Week 19 has no DSA-pattern content at all) and two cold-revision re-solves (LC 721, LC 1143 — logged above, not counted as newly solved, per this table's own standing rule).

New this week — System Design (HLD) phase, Week 19 only:
- **HLD systems designed:** 9 (WhatsApp/Chat, Uber Driver Location Tracking, Netflix/Video Streaming, Instagram Feed, Payment System, Distributed Job Scheduler, Leaderboard, Search Autocomplete, Google Drive/Object Storage)
- **HLD mocks completed:** 3 (Mock #3 — subject Distributed Cache, Day 128; Mock #4 — subject Payment System, Day 131; Mock #5 — candidate's choice, Day 133)

Combined LLD + HLD phase totals, Weeks 15–19:
- **Systems designed, LLD + HLD combined:** 26 (10 LLD, Weeks 16–17; 16 HLD, Weeks 18–19)
- **Mock interviews, LLD + HLD combined:** 10 (5 LLD-phase mocks, Weeks 16–17; 5 HLD-phase mocks, Weeks 18–19)

**The HLD phase is complete as of Day 133** — matching the LLD phase's own five-mock close exactly, and closing the combined LLD+HLD design-interview arc at ten mocks and twenty-six systems across Weeks 15–19. See Day 133's Week 19 Consolidation and Self-Check for the full sixteen-system HLD list with each system's distinctive concept.

### Running Totals — After Week 20

DSA problem totals are **unchanged** from the Week 19 running totals above — Week 20 added zero newly-solved DSA problems and zero cold-revision re-solves, confirmed directly against `Week_20_Revised.md`'s full content, which contains no LeetCode-numbered problems at all (the first week in the series of which that's true in this exact total sense — Weeks 16–19 each still logged cold-revision re-solves even while adding zero newly-solved problems; Week 20 has neither).

LLD/HLD systems-designed and mocks-completed totals are likewise **unchanged** from Week 19 above — Week 20 introduces no new HLD system and runs no mock interview; where it touches one of the sixteen already-designed systems (Day 140's at-scale exercise cites several by name), it is explicitly an application of something already taught, not new system-design teaching, per this map's own "How to Use This Document for Week 20" guidance.

New this week — Capstone Hardening phase, Week 20 only (see the new Platform Operations / Infrastructure Deliverable Inventory table below for the full day-by-day breakdown):
- **Kubernetes deployment artifacts delivered:** Deployment + Service manifests for all four modules (Day 134); a Helm chart replacing them (Day 135); three environment-specific values files (Day 136).
- **Chaos experiments run and documented:** 3 (Payment-kill/circuit-breaker, Day 138a; latency-injection/Gateway-timeout, Day 138b; connection-pool saturation, Day 138c) — three times the plan's own original single-experiment baseline, matching this map's established practice of naming the multiplier explicitly rather than leaving it implicit.
- **Service mesh demonstrations:** 1 (sidecar injection + a 90/10 weighted traffic split, Day 138).
- **Hand-rolled concurrency components:** 1 (`BoundedBlockingQueue<T>`, Day 138 — `java-fundamentals`).
- **CI/CD pipelines finalized:** 1, including Helm chart validation as one of its own steps (Day 139).
- **Formal at-scale analyses of a built (not merely designed) system:** 1 — the platform itself (Day 140), the first system in the series to receive the HLD 5-step framework's Estimation/Bottlenecks treatment as a genuinely built artifact rather than a whiteboard design.

### Running Totals — After Week 21 (Series Final)

DSA, SQL, LLD, and HLD problem/system totals are all **unchanged** from the Week 20 running totals above — confirmed directly against `Week_21_Revised.md`'s full content, which contains no LeetCode-numbered problems and designs no new system, matching Week 20's own shape. **Final, closing figures for the entire 21-week series:** 197 required DSA problems, 249 distinct DSA problems including all extra practice (Weeks 1–15), plus a 10-problem SQL track — 259 combined. 10 LLD systems (Weeks 16–17) and 16 HLD systems (Weeks 18–19), each backed by six mock interviews apiece across the full series (five per phase through Week 19, plus one more of each in Week 21 — Day 145's LLD mock and Day 146's System Design mock).

New this week — the behavioral/domain-knowledge/portfolio-finalization content that closes the series (see the new "Week 21 Additions" subsection above, in the Concept-Dependency Map, for the full day-by-day breakdown): the STAR framework and six-competency map, all eight STAR stories (rehearsed and mapped against Google's Googleyness, Databricks' six leadership principles, and Atlassian's five values), four new domain-knowledge topics (UPI's real-world 2PC application, High Availability, high-volume/low-margin economics, Multi-Tenant SaaS at scale), a full technical retention self-check, a finalized resume, and a polished, honestly-presented five-repository portfolio. None of this is DSA-, LLD-, or HLD-table material in the sense those tables track, so no new rows were added to any of the three — its record lives in the Concept-Dependency Map's Week 21 Additions subsection and the Terminology section's own Week 21 Additions subsection, below.

## LLD System / Design Exercise Inventory (Week 16 Onward)

**Why this table exists, stated explicitly rather than resolved silently:** the "How to Use This Document for Week 16" note (superseded below by the current Week 17 version, but preserved per this document's own extend-don't-edit rule) flagged this as a genuine open question ahead of Week 16's generation — the DSA Problem Inventory above tracks LeetCode-style problems by number, which doesn't fit LLD content at all (a system like "Parking Lot" has no LC number, and "solving" it isn't a single event the way solving a problem is). Rather than force LLD content into that table's shape, or silently skip tracking it, this is a **new, parallel table**, structurally similar in spirit — day, deliverable, required-vs-not, pattern used — but adapted for what LLD content actually is. Extended the same way the DSA table is: every future week appends to it, never replaces or shortens it.

| Week | Day | System / Exercise | Primary Pattern(s) Used | Type | Notes |
|---|---|---|---|---|---|
| 16 | 106 | Coffee Decorator | Decorator | Required (plan) | Full implementation + JUnit, 3+ stacked decorators |
| 16 | 106 | File System (sketch) | Composite | Required (plan) | Sketch depth, no full test suite, per the plan's own framing |
| 16 | 107 | WeatherStation / Display | Observer | Required (plan) | Built test-first via TDD; Red-Green-Refactor documented |
| 16 | 107 | DataProcessor (sketch) | Template Method | Required (plan) | Sketch depth |
| 16 | 108 | Tic-Tac-Toe | None (deliberately) | Required (plan) | 5-step framework's first live, end-to-end run |
| 16 | 109 | Vending Machine | State | Required (plan) | Zero state-dispatch conditionals in the context class |
| 16 | 110 | Parking Lot, Part 1 (single-threaded) | None (Singleton considered, declined) | Required (plan) | Composition over subclassing for spot/vehicle sizing |
| 16 | 111 | Parking Lot, Part 2 (concurrency) | None | Required (plan) | Per-`ParkingSpot` locking; 10-thread test, zero double-booking |
| 16 | 112 | Library Management System | None (data-driven transition table, not State) | Required (plan) | Catalog/circulation kept provably separate |
| 17 | 113 | ATM Machine | State (reapplied) + Chain of Responsibility (new, 11th pattern) | Required (plan) | Naive dispenser shown broken, fixed via check/commit split |
| 17 | 114 | Elevator System | State (reapplied, 3rd time) + SCAN/LOOK dispatch (not GoF) | Required (plan) | LOOK proven >2x more efficient than FIFO on a worked trace |
| 17 | 115 | Splitwise | Strategy (1st full system application) | Required (plan) | Dual-heap greedy settlement; ≤(n−1) proof + honest NP-hard caveat |
| 17 | 116 | BookMyShow, Part 1 (design) | None (deliberately) | Required (plan) | Per-show seat scoping; check-then-act race traced, not yet fixed |
| 17 | 117 | BookMyShow, Part 2 (concurrency) | None (pessimistic + optimistic locking) | Required (plan) | `SELECT FOR UPDATE`, version-based CAS, `CountDownLatch`-proven 10-thread test |
| 17 | 118 | Food Delivery System | Strategy (2nd full system application) | Required (plan) | Stateless strategies — sharpens the who-chooses test vs. constructor-config heuristic |
| 17 | 119 | Hotel Booking System | None (deliberately) | Required (plan) | Date-range availability; concurrency model compared against BookMyShow |

**Week 16 total: 9 LLD deliverables — 9 required by the plan, 0 added as extra.** Unlike the DSA table, "extra practice" doesn't have a clean analog here — Day 106's Resource Book reasoned through this directly: a design pattern's mastery doesn't come from repeating the *same* pattern against many small variations the way a LeetCode pattern does; it comes from applying the framework to genuinely different systems, which the plan's own system count already provides. No extra LLD exercises were added this week on that reasoning, stated here rather than left as an unexplained zero.

**Week 17 total: 7 LLD deliverables (6 distinct systems — BookMyShow spans Days 116–117 as one system) — 7 required by the plan, 0 added as extra**, for the identical reasoning Week 16 gave: the plan's own six-system count already provides genuine variety (four systems chose a pattern, two deliberately didn't), which is what LLD mastery actually needs more of, not repeated small variations on one already-built system. **This closes the LLD phase at ten systems total across Weeks 16–17** — see Day 119's Self-Check and Week 17 Consolidation for the full ten-system list with primary patterns.

**Mock interviews, tracked here rather than in either problem table, since they aren't a deliverable in either table's sense:**

| Week | Day | Subject | Format | Debriefed |
|---|---|---|---|---|
| 16 | 109 | Tic-Tac-Toe | 45-min, full framework | ✅ |
| 16 | 112 | Parking Lot + concurrency | 45-min, forced "make it thread-safe" escalation mid-interview | ✅ |
| 17 | 114 | Rate limiter (Token Bucket, recapped from Week 10) | 60–90 min, Machine Coding format (new format) | ✅ |
| 17 | 117 | BookMyShow + concurrency | 60-min, pushed on pessimistic-vs-optimistic justification | ✅ |
| 17 | 119 | Candidate's choice | 45-min | ✅ |

**LLD phase total: five mocks across Weeks 16–17** (two LLD-framework mocks, one Machine Coding mock, two more LLD mocks with concurrency emphasis) — matching `Week_17_Revised.md`'s own Day 119 framing of the phase closing "five mocks deep."

---


## System Design / HLD Deliverable Inventory (Week 18 Onward)

The LLD table above tracks one-process, class-level design (Weeks 16–17). Week 18 opens a genuinely different kind of deliverable — multi-machine architecture, not class diagrams — so it gets its own table here rather than being forced into the LLD table's shape, per this map's own standing guidance (see "How to Use This Document for Week 18," below) to reason through the right shape explicitly rather than copy one built for something else.

| Week | Day | System / Topic | Core New Concepts | Mock? | Notes |
|---|---|---|---|---|---|
| 18 | 120 | URL Shortener | HLD 5-step framework (full depth, first time); Base62; `FOR UPDATE SKIP LOCKED` | — | Day 118's preview superseded by full teaching |
| 18 | 121 | Rate Limiter, Formalized | Leaking Bucket; Fixed Window (+ 2x flaw); Sliding Window Log; Redis+Lua distributed limiting | — | Token Bucket (Week 10, Day 68) recapped, not re-taught |
| 18 | 122 | Notification System | Per-channel queues; Template Service; Redis dedup (best-effort, deliberately not the full idempotency-key mechanism) | **HLD Mock #1** — subject: URL Shortener (Day 120); focus: narrating the 5 steps under time pressure | DLQ (Week 14, Day 93) cited, not re-taught |
| 18 | 123 | Distributed Cache | Redis full picture (data structures, Memcached contrast, persistence, pub/sub); Thundering Herd + 2 mitigations; Cache-Aside | — | Consistent Hashing (Week 9, Day 60) recapped, not re-taught |
| 18 | 124 | Distributed ID Generation | UUID; Twitter Snowflake (64-bit); DB ID range allocation | — | — |
| 18 | 125 | Distributed Consensus | Quorum; split-brain; Paxos (conceptual only); Raft (election, log replication, safety); etcd | **HLD Mock #2** — subject: Rate Limiter (Day 121); focus: defending a choice under "why not X" follow-ups | Closes the gap Replication Models (Week 9, Day 61) left open |
| 18 | 126 | BookMyShow at Scale | City-sharding; seat-hold (Redis TTL + Lua, reusing Day 121's atomicity pattern); proactive cache warming (a genuine extension of Day 123's reactive mitigations, for when spike timing is known in advance) | Self-Check: LC 40, cold (see DSA Revision Log — Week 18, above; discrepancy flagged there) | BookMyShow's original LLD locking (Week 17, Days 116–117) unchanged — cited, not re-derived |
| 19 | 127 | WhatsApp / Chat System | Persistent WebSockets; connection-management registry; message routing via Kafka (applied, not re-taught); wide-column store (Cassandra-style) for offline storage | — | Media sharing named, deliberately deferred to Day 133 |
| 19 | 128 | Uber Driver Location Tracking | Geohashing; Quadtrees; Redis Geo (`GEOADD`/`GEOSEARCH`) | **HLD Mock #3** — subject: Distributed Cache (Day 123); Uber format, think-aloud + "why not simpler" pushback | CAP theorem correctly scoped away from a single-machine cache |
| 19 | 129 | Netflix / Video Streaming | Transcoding pipeline; adaptive bitrate streaming; CDN (Cache-Aside relocated to the edge) | — | Thundering Herd (Day 123) and proactive warming (Day 126) both reapplied at the edge |
| 19 | 129 | Instagram Feed | Push / pull / hybrid fan-out | — | Resolves "the celebrity problem" (Day 78), three appearances in |
| 19 | 130 | Payment System | Idempotency keys (full); `SETNX`-based distributed locks (full, incl. unsafe-release trace); double-entry ledger | — | Closes the loop deliberately left open since Days 121/122/124/126 |
| 19 | 131 | Distributed Job Scheduler | SQS Visibility Timeout; Redis Sorted Set delayed queue; Quartz cron + misfire handling; at-least-once vs. exactly-once | **HLD Mock #4** — subject: Payment System (Day 130); race-condition pressure format | DSA cold-revision aside: LC 721, LC 1143 (see DSA Revision Log — Week 19, above) |
| 19 | 132 | Leaderboard System | Skip List (full mechanism, first time); Redis Cluster (slot-based, contrasted with Consistent Hashing); composite-score tie-handling | — | Sorted Sets' O(log n) contract finally explained — named Day 123, used Days 128/131 |
| 19 | 132 | Search Autocomplete | Distributed, asynchronously-rebuilt Trie; top-K-per-node precomputation | — | Trie structure itself cited from Week 9, not re-taught |
| 19 | 133 | Google Drive / Object Storage | Chunking (fixed-size vs. content-defined); content-addressable deduplication; sync conflicts (LWW vs. conflict-copy); multipart upload | **HLD Mock #5** — candidate's choice (same precedent as LLD's Day 119) | Self-Check: all 16 HLD systems named cold; HLD phase closes |

**Deliberately deferred past Week 18, flagged here so Week 19's generation doesn't need to rediscover why:** the formal **`SETNX`-based distributed lock** pattern and **idempotency keys** — both named as Week 19, Day 130 content in `Week_19_Revised.md` — were not taught this week even though Day 122's notification deduplication and Day 126's seat-hold both needed *some* check-and-set mechanism. Both were solved instead by reusing Day 121's already-taught Lua-atomicity pattern (a plain existence-check-then-set, wrapped in Lua for atomicity, with an explicitly accepted best-effort race window where the stakes were low enough to tolerate it). This keeps Week 19's own new content genuinely new when it arrives, rather than a rehash of a lighter version already covered.

**The HLD phase closes here, at sixteen systems total across Weeks 18–19** — matching the LLD phase's own ten-system close (Day 119) in spirit, if not in count. Every deferral flagged immediately above is now resolved: `SETNX`-based distributed locks and idempotency keys received their full, first-time treatment on Day 130, including the unsafe-lock-release bug traced as a concrete failure mode rather than asserted, closing a thread deliberately left open since Day 121. Sorted Sets' own O(log n) contract — named Day 123, used operationally on Days 128 and 131 without explanation — is finally opened up on Day 132 via the Skip List, following the identical `TreeMap`-since-Week-1 precedent for using a structure by its proven complexity before its internals are taught. Five HLD mocks ran across the two weeks (Days 122, 125, 128, 131, 133) — a genuine multiple of the original plan's single HLD mock, and an exact match for the LLD phase's own five. See Day 133's Week 19 Consolidation and Self-Check for the full sixteen-system list with each system's distinctive concept.

## Platform Operations / Infrastructure Deliverable Inventory (Week 20 Onward)

**Why this table exists, reasoned through rather than forced, the same way the LLD and HLD tables above each were:** the DSA table tracks LeetCode-numbered problems; the LLD and HLD tables above track discrete *systems designed*. Week 20's content is neither — no LeetCode number, and not a system designed from a blank page, but real infrastructure and operational artifacts (Kubernetes manifests, a Helm chart, a CI/CD pipeline, one hand-rolled concurrency component, one written analysis) produced *for* a system already under construction since Week 9. Forcing these into the HLD table's "System / Topic" column would misrepresent them as new systems being designed, which they aren't. This is accordingly a third, parallel table — extended the same way the other two are: every future week appends to it, never replaces or shortens it.

| Day | Deliverable | Core New Concepts | Notes |
|---|---|---|---|
| 134 | k6 load-test baseline; Deployment + Service manifests for all 4 modules | Load testing fundamentals (VUs, p50/p95/p99); k6; real Kubernetes manifests; resource requests vs. limits | Kubernetes theory (Week 15, Day 99) applied for the first time in writing, not re-taught |
| 135 | Helm chart (`helm-chart/`) | `Chart.yaml`/`values.yaml`/`templates/`; Kubernetes Ingress (explicitly disambiguated from Spring Cloud Gateway); Minikube | Templates Day 134's exact manifests — no new K8s object types introduced |
| 136 | Three environment-specific values files + ConfigMaps/Secrets | Values-file override precedence (maps merge, lists replace); ConfigMap vs. Secret (base64 ≠ encryption, proven); "build once, deploy many times" | Spring Profiles (Week 8, Day 51) cited as the identical externalized-config pattern one layer up the stack |
| 137 | Distributed tracing integration (all 4 modules); `docs/api-design-notes.md` | Micrometer Tracing; Zipkin; MDC log correlation; sampling trade-off; API versioning/pagination/idempotency/error-contract | Tracing's Kafka blind spot named explicitly and tied directly to Day 70's Saga running on that exact transport |
| 138 | 3 documented chaos experiments; service mesh sidecar + traffic-split demo; `BoundedBlockingQueue<T>` (`java-fundamentals`) | Chaos engineering as formal practice; connection pooling (HikariCP), taught from zero; service mesh (library-vs-infrastructure distinction); hand-rolled bounded blocking queue | Queue directly extends Producer-Consumer (Week 6, Day 38) and reuses the `CountDownLatch`-gated test pattern (Week 17, Days 116–117) — no new concurrency primitives |
| 139 | CI/CD pipeline; liveness/readiness probes (all 4 modules); finalized `README.md` | Mutation testing (concept, PIT named); JaCoCo wired as an enforced gate; GitHub Actions pipeline traced end to end; liveness-vs-readiness distinction + dependency-in-liveness anti-pattern | Coverage-isn't-correctness proven via a real 100%-covered, zero-assertion counterexample, not asserted |
| 140 | `docs/load-test-results.md`; `docs/at-scale-analysis.md` | Bottleneck-diagnosis methodology (saturation vs. failure signature); 10x/100x breakdown applied to a built system | Explicitly cites all sixteen Weeks 18–19 HLD systems as the precedent this exact treatment extends |

**Zero new HLD systems and zero mock interviews this week** — Day 140's at-scale exercise cites specific already-designed HLD systems by name (Days 120, 121, 124, 126 among them) purely as precedent for a methodology being reapplied, not as new system-design teaching, consistent with this map's own standing guidance on that distinction. This closes the **Capstone Hardening phase** at seven deliverables across Week 20 — the platform moves from "designed and built" to "designed, built, deployed on real infrastructure, chaos-tested, and formally analyzed at scale," the single biggest structural gap the original audit had left open, per `Week_20_Revised.md`'s own closing scorecard.

## Terminology and Framing Already Established

Reuse these exact framings rather than inventing new labels for the same idea:
- **Two Pointers variants — the full family, 7 named variants, not 2:** *opposite-ends*, *from-the-back*, *same-direction (fast/slow)*, *one-forward-pointer-each*, *opposite-ends-with-one-allowed-skip*, *opposite-ends-with-greedy-pairing* (new, Week 2 Day 11 — Boats to Save Most People; distinct from one-forward-pointer-each even though both are "sorted + greedy," see Day 11's Assign Cookies aside), *three-way partition / Dutch National Flag* (new, Week 2 Day 12 — Sort Colors; doesn't reduce to either of the two most-common variants). Week 2, Day 13 has the full 16-problem classification table if a refresher is ever needed.
- **HashMap/HashSet variants**: *complement lookup*, *membership*, *frequency counting*, *canonical-key grouping*, *two-way mapping*, *smart-starting-point traversal*.
- **Sliding Window — fully closed as of Week 3, Day 20:** *fixed-size* (both edges move in lockstep, e.g. LC 643, LC 567, LC 438, LC 239, LC 1052) vs. *variable-size* (edges move independently based on a constraint; introduced Week 3, Day 15). Framed explicitly as an evolution of *same-direction (fast/slow)*: same rightward-only pointer discipline, but both pointers become meaningful range boundaries with a running aggregate over everything between them, instead of one scanner and one write-marker. Within variable-size, two distinct loop shapes are both established and must not be conflated: **shrink-until-valid** (Day 15 onward — shrinks only when currently invalid, solves for the *longest* valid window) vs. **shrink-while-valid** (Day 17 onward — shrinks precisely because it's currently valid, solves for the *shortest* valid window). Day 20's full 18-problem fixed/variable answer key is the reference table if a refresher is ever needed.
- **Monotonic deque** (new, Week 3 Day 20): a deque of indices held in strictly-decreasing-value order (front-to-back, for a max), where a smaller-or-equal incoming value permanently dominates and evicts everything smaller at the back, and expired (out-of-window) indices are evicted from the front. Distinct from every prior Sliding Window shape — it's not a shrink-loop at all.
- **At-most-K-distinct, and its counting extension** (Week 3, Days 18–20): the shrink-until-valid shape with a HashMap-size validity check, first as a hardcoded K=2 (LC 904), then generalized to a parameter (LC 340), then reused to derive an *exactly-K* count via `atMost(K) − atMost(K−1)` (LC 992) — cite this exact three-step generalization lineage rather than re-deriving any of it.
- **HashMap Internals** (deepening, Week 3 Day 15): `hashCode()` → spread (`h ^ (h>>>16)`) → bucket index (`(n-1)&hash`) → collision chaining → treeification (`TREEIFY_THRESHOLD=8`, `MIN_TREEIFY_CAPACITY=64`, `UNTREEIFY_THRESHOLD=6`). The equals()/hashCode() contract (Week 1, Day 4, stated as a rule) now has its mechanism: violating it causes `get()` to search the *wrong bucket entirely* — distinct from a merely-constant hashCode(), which only degrades a bucket's chain length (a performance bug, not a correctness bug). Don't conflate these two when citing "a bad hashCode()" later.
- **Generics** (new, Week 3 Day 16): `<T>`, bounded type parameters, PECS (**P**roducer **E**xtends, **C**onsumer **S**uper) for wildcards, and type erasure as the reason `new T[]` doesn't compile (arrays are reified at runtime; erased generics have no runtime type to give them).
- **Comparable vs. Comparator** (new, Week 3 Day 17): one natural ordering defined inside a class (`Comparable`/`compareTo`) vs. any number of external orderings defined outside it (`Comparator`/`compare`). `TreeMap` (Red-Black tree, O(log n), sorted keys) and `PriorityQueue` (binary heap array, O(log n) insert/poll, O(1) peek) both cited as consumers of one or the other.
- **`==` vs `.equals()`** is established as "the single most-referenced gotcha across the week" (Week 1, Day 2) — the Integer cache trap (Week 2, Day 10), the String pool (Week 2, Day 11), and array-reference comparison in `Arrays.equals()` vs `==` (Week 3, Day 16) are all this same gotcha resurfacing in sneakier forms via reference-reuse or reference-vs-content mechanisms; cite this lineage rather than re-deriving any instance of it from scratch.
- **Amortized analysis** was taught specifically to justify `ArrayList.add()`'s O(1) average (Week 1, Day 3) — reused verbatim for `StringBuilder`'s buffer doubling (Week 2, Day 11), `ArrayDeque`'s resize (Week 2, Day 12), `HashMap`'s resize/rehash (Week 3, Day 15), and the "total pointer movement ≤ n" argument that makes every variable-size Sliding Window solution O(n) despite a nested loop (Week 3, Day 15 onward). Reuse this exact justification each time rather than re-deriving it.
- **Stack vs. heap** (Week 2, Day 9) is the mechanism-level explanation for pass-by-value, established in Week 1 (Day 1 primitives, Day 2 objects) as a rule. Later weeks can cite "the stack/heap split" as the *why* behind pass-by-value without re-deriving it.
- **Prefix Sum & Kadane's** (Week 3 Day 21, **fully closed** Week 4 Day 22): `prefix[i] = sum of the first i elements`, `prefix[0]=0` by convention (avoids an index-0 special case), range sum `[i,j] = prefix[j+1]-prefix[i]` by telescoping. Kadane's is the running-max special case: `best[i] = max(nums[i], best[i-1]+nums[i])`, proven by an exchange argument (extending a suboptimal prior subarray is never better than extending the optimal one) — lightly flagged as a preview of Dynamic Programming's "best answer here, built from the best answer one step back" shape, not yet formally named as DP. Extended Week 4 with a HashMap layered on top: store a **first-occurrence index** when the question is "where/how long" (LC 525, 523) vs. a **frequency count** when it's "how many" (LC 560, 974) — the same fork every prefix-sum-plus-lookup problem eventually reduces to. Kadane's itself extended two ways: run twice (once inverted, for a min) to handle a circular array (LC 918), or track a running max *and* min together to survive a sign-flipping product (LC 152).
- **Greedy Algorithms** (new, Week 4 Day 23, closed Day 27): a locally-best choice at each step, never revisited — a *description*, not by itself a *justification*. Correctness requires an **exchange argument**: show the greedy choice can always be swapped into any optimal solution without making it worse. Three distinct exchange-argument shapes were named this week, and are cited by name rather than re-derived going forward: **domination** (a reachable/achievable set fully contains what a worse choice could achieve — Jump Game, LC 55), **interval-swap** (the chosen element leaves at least as much room as any alternative — Non-overlapping Intervals, LC 435), and **prefix-elimination** (an entire contiguous range of candidates is ruled out at once, not just the one just tried — Gas Station, LC 134). Where greedy provably fails: 0/1 Knapsack (worked counter-example, Day 23) — an early choice can block a strictly better later combination, with no swap able to recover the loss; this needs Dynamic Programming, not greedy.
- **Intervals** (new, Week 4 Day 24, closed Day 27): sort, then sweep once, comparing each interval only to whatever's currently being built. **Sort by start time** for "combine overlapping ranges" questions (Merge Intervals, Insert Interval, Day 24) — the overlap condition is `current.start <= last.end`. **Sort by end time** instead for "how many survive / how few resources needed to cover everything" questions (Non-overlapping Intervals, Min Arrows, Day 25) — keep or cover via the earliest-ending choice in each overlapping cluster. A min-heap of end times (reusing `PriorityQueue`, Week 3 Day 17) extends this to counting concurrent resource needs (Meeting Rooms II, Day 26); an implicit-interval variant needs no `[start,end]` object at all, deriving ranges on the fly from a last-occurrence sweep (Partition Labels, Day 27). Different problems define "touching" endpoints inconsistently (compare LC 56 vs. LC 435 vs. LC 253) — verify each problem's own inequality direction rather than assuming.
- **Records and Sealed Classes** (new, Week 4 Day 28): `record` (standard since Java 16, JEP 395) generates a canonical constructor, `.x()`-style accessors (not `.getX()`), `equals()`/`hashCode()`/`toString()` from all components, and is implicitly `final`. `sealed interface ... permits A, B, C` (standard since Java 17, JEP 409) restricts implementers to a compiler-known, finite list; each permitted type must be `final`, `sealed`, or `non-sealed` (records satisfy this automatically, being always implicitly final). Exhaustive `switch` via type patterns over a sealed hierarchy, with no `default` clause, needs **Java 21** specifically (JEP 441, Pattern Matching for `switch` — preview across 17–20, standard only at 21) — cite this precise version distinction rather than the looser "Java 17+" whenever this feature is referenced again.
- **Binary Search — "on the input" vs. "on the answer"** (Day 28, formalized Day 33): *on the input* binary searches directly over array positions, with the array itself supplying the monotonic elimination rule (exact match, rotated-array structure, boundary search, 2D-flattened-to-1D — 9 of the pattern's 15 distinct problems). *On the answer* binary searches over a range of candidate answer values with no array being searched at all, using an actively-constructed monotonic feasibility function instead (First Bad Version, Koko, Ship Capacity, Bouquets, Magnetic Force — 6 of the 15). The tell for the second framing is almost always the phrasing "minimum/maximum X such that Y is achievable" — cite this exact classification (Day 33's full table) rather than re-deriving it.
- **Rotated sorted array invariant** (Day 30, extended Day 31): splitting a rotated sorted array at any index always leaves at least one half normally sorted (no duplicates) — `nums[left] <= nums[mid]` tests which half. Duplicates break this specific check; the fix (shrink from both ends, or drop one boundary, depending on the exact problem) always costs the same thing: O(log n) average degrades to O(n) worst case. This exact cost — not just "duplicates make it harder" — is the fact to cite (LC 33→81 and LC 153→154 both demonstrate it identically).
- **Boundary search** (Day 31): on a match, keep narrowing instead of stopping — biased left (`right = mid - 1`) to find a first/leftmost occurrence, biased right (`left = mid + 1`) for last/rightmost. Two independently-biased passes for a two-sided search (LC 34); one pass with a single-sided condition when only one boundary is needed (LC 744).
- **Threads and the JVM Concurrency Model** (new, Week 5 Day 29): one call stack *per thread*, one heap *shared by every thread in the process* — a direct extension of Day 9's stack/heap split from one thread to N, not a new memory model. `implements Runnable` over `extends Thread`, for the same single-inheritance reason (Day 2) that motivates preferring composition/interfaces generally elsewhere. The JVM's actual `Thread.State` enum has six values (`NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED`), not five — `RUNNABLE` covers both "eligible" and "actually executing," and the conceptual model's "Blocked/Waiting" bucket actually splits three ways. Week 5's demo showed non-deterministic *ordering* only, not data corruption — corruption from an unsynchronized shared object is reserved for Week 6, Day 37's `Counter`/`ReentrantLock` exercise; don't conflate the two failure modes when citing this again.
- **Anonymous inner classes** (new syntax, Week 5 Day 29): a class declared and instantiated in one expression, with no name, for a one-off implementation of an interface or superclass used in exactly one place. Introduced for `Runnable`, reused Day 35 for a custom `Comparator`. This is the general-purpose version of the same unnamed-subclass mechanism Week 1 Day 5's enum-constant bodies already used narrowly — cite that connection rather than presenting it as unrelated. Deliberately used in place of lambda syntax throughout this series so far, since lambdas/functional interfaces haven't been formally taught yet; don't introduce lambda syntax in future weeks without first confirming it's actually been taught by then.
- **Linked Lists** (new, Week 5 Day 34, opened at 4/11): a node's `next` field is a heap reference — mechanically identical to any object reference (Day 9), not a new memory concept, only a new shape built from one already in hand. O(1) insertion/deletion *at a known node* (rewiring references) vs. O(n) *to find* that node in the first place if you don't already hold a reference to it — these are frequently conflated and shouldn't be. Trade-off against `ArrayList`, previewed Day 12's Collections Internals, proven here: no memory-address formula for `arr[i]` exists on a linked list, so random access is O(n); insertion at a known position avoids `ArrayList`'s O(n) shift entirely.
- **Dummy head** (new technique, Week 5 Day 35): a throwaway node that gives a list-building loop something valid to attach to from the very first real node, removing the need to special-case "is the result still empty." Reused immediately for a bounded reversal (LC 92) to avoid separately special-casing a reversal that starts at the true head.
- **REST APIs and Spring Boot** (new, Week 5 Day 34): HTTP verbs map to CRUD, status codes grouped by leading digit (`2xx`/`4xx`/`5xx`, with `400` vs. `404` vs. `422` distinguished precisely — malformed request vs. resource doesn't exist vs. well-formed but semantically invalid). Statelessness is a real trade-off (any server instance can handle any request, at the cost of clients resending context every time), not an arbitrary rule. `@RestController`/`@GetMapping` are resolved via **reflection** — a running program inspecting its own classes/methods/annotations at *runtime*, introduced here for the first time — mechanically distinct from `@Override`'s compile-time-only role (Day 2/5); don't conflate the two kinds of annotation when citing either again. `ResponseWrapper<T>` reuses Day 16's Generics at their simplest: `T` fully unbounded, since the class never calls a `T`-specific method.
- **Floyd's Cycle Detection, extended** (Week 6 Day 36): the closing-gap proof (once both pointers are inside a cycle, the gap between them shrinks by exactly 1 every iteration, so it must hit 0) is the fact to cite, not just "fast catches slow." The phase-2 reset-to-head step (Cycle II) is derived from `a = (n-1)c + (c-b)`, not memorized. Generalizes to anything with a deterministic "next state" and a bounded state space — Happy Number (`next(n)` = sum of squared digits) and Find the Duplicate Number (array values read as implicit pointers, `nums[i]` = "next") both reuse the identical two-phase mechanism with zero changes to the core logic.
- **Spring Data JPA / ORM mechanics** (new, Week 6 Day 36): JPA is a specification, Hibernate the common implementation, Spring Data JPA a further layer removing boilerplate — cite all three by name, not interchangeably. `@Entity`/`@Id`/`@Column` are read via **reflection** (Day 34's mechanism, reused for new work); `JpaRepository` implementations are generated at runtime via that same mechanism, not hand-written. `@Enumerated(EnumType.STRING)` vs. the `ORDINAL` default (fragile under reordering) is a specific, citable gotcha. `ddl-auto: update` (convenient, silently destructive) vs. `validate` (Day 40, once Flyway owns the schema) — two different tools for two different project phases, not "one is better."
- **`synchronized` and `ReentrantLock`** (Week 6 Day 37): both are **already reentrant** — that is not a distinguishing feature of `ReentrantLock` despite its name, and citing reentrancy as ReentrantLock's advantage is a flagged misconception. The real differences: `tryLock()`/timeouts, configurable fairness, interruptible acquisition, and — critically for Day 38 — multiple `Condition` objects per lock. `synchronized` guarantees release on any exit path; `ReentrantLock` does not, making `try`/`finally` a correctness requirement, not a style choice. Week 6 Day 37 is this series' first demonstration of actual data *corruption* from a race condition (a concrete lost-update interleaving trace) — Week 5 Day 29's demo showed non-deterministic ordering only; don't conflate the two again.
- **`wait()`/`notify()` and `Condition`** (Week 6 Day 38): both require holding the associated lock, or the JVM throws `IllegalMonitorStateException`. The `while`-not-`if` rule around any `wait()`/`await()` call has two independent justifications, both citable: documented spurious wakeups, and `notifyAll()`/`signalAll()` waking threads whose specific condition still doesn't hold. `Condition` (`lock.newCondition()`) gives multiple independent wait-sets from one `Lock` — this is what makes `signal()` (not `signalAll()`) safe for a specific waiter group, since every thread on a given `Condition` is, by construction, waiting for the same thing.
- **Doubly linked list, and dummy head + dummy tail** (new, Week 6 Day 39): adding a `prev` pointer turns removal-of-a-known-node from O(n) (must traverse to find the predecessor) into O(1) (`node.prev` already is it). Two permanent sentinels — extending Day 35's single dummy head to both ends — eliminate every "is this the first/last real element" branch uniformly. Built specifically as LRU Cache's (LC 146) second structure, paired with a HashMap for O(1) lookup; neither alone satisfies both of that problem's requirements.
- **Stacks, formalized** (new pattern, Week 6 Day 39, opened at 0/12): LIFO — informally in use via `ArrayDeque` since Week 1 Day 4 (`ArrayDeque` preferred over legacy `java.util.Stack` specifically because `Stack` extends `Vector` and pays unneeded `synchronized` overhead on every call). Interview signal: "matching," "balanced," "nested," "most recent," "evaluate an expression." **Monotonic Stack** (sub-pattern, opened Day 40): a stack whose ordering invariant is *actively enforced by the algorithm itself* on every push — provably so, not just observed — with an amortized O(n) argument (every element pushed exactly once, popped at most once) that must be reproduced, not asserted, when a nested loop makes O(n²) look plausible at a glance. Established across four shapes so far, cite by name rather than re-deriving: plain array (LC 496), circular array via `2n`/`i%n` + index-based storage (LC 503), a persistent/streaming variant spanning separate method calls rather than one array pass (LC 901), and the mirror-image *increasing* variant for greedy digit removal (LC 402).
- **`ConcurrentHashMap` internals** (new, Week 6 Day 39): the comparison against `Collections.synchronizedMap()` is about *locking scope*, not correctness — both are equally correct under concurrent access; `synchronizedMap` serializes everything behind one lock, `ConcurrentHashMap` locks at the level of individual buckets and uses **CAS** (Compare-And-Swap — an atomic hardware read-check-write with no interruptible "middle," defined precisely, not just named) to avoid locking entirely for some operations. A plain, unwrapped `HashMap` under concurrent modification is a third, genuinely *incorrect* option, not merely a slower one — don't present the choice as a two-way trade-off between correct-but-slow and correct-but-fast.
- **SQL Fundamentals** (new, Week 6 Day 40): `LEFT JOIN` **includes** a non-matching row with `NULL`-filled columns; `INNER JOIN` **excludes** it entirely — cite this as two different outcomes, not two strengths of one operation. `NOT IN` silently returns zero rows for every case if its subquery can produce even one `NULL` (`UNKNOWN` propagates through `AND`, read as `false`); `NOT EXISTS` has no equivalent failure mode and is the safer default. Indexes trade write speed for read speed via a B-tree, O(log n) vs. O(n) — every index must also be updated on every write, not just the underlying row.
- **Stacks/Monotonic Stack, fully closed** (Week 7 Days 43–45, 12/12 required + 2 extra = 14 distinct): closed with a two-family classification, cite by name rather than re-deriving — **general-purpose LIFO** (the stack holds state that needs correcting or restoring, no ordering invariant on the values: LC 20, 232, 155, 150, 735, 227, 224) vs. **true monotonic stack** (an enforced increasing/decreasing invariant answers a positional-neighbor question: LC 496, 503, 901, 402, 739, 84, 85). The signal that sorts a new problem into one or the other: does it ask about a value's relationship to neighbors by *position* (monotonic) or ask you to evaluate/simulate/reverse an *ordered sequence of operations* (general-purpose). Largest Rectangle in Histogram (LC 84) extended the monotonic-stack shape from values to **indices** specifically — the first time this series' monotonic stack stored positions rather than the values themselves, needed because width requires position information a bare value can't carry.
- **Docker** (new, Week 7 Day 43): image (static, layered, read-only snapshot) vs. container (a running instance, with a thin writable layer discarded on removal) — cite as the same class/object relationship established Week 1 Day 2, not a new concept. `Dockerfile` layer caching: each instruction is a layer, hashed on its inputs; the instant one layer's inputs change, that layer *and every layer after it* rebuild regardless of cache — this is the entire justification for ordering dependency-resolution steps before source-copy steps. Multi-stage builds (Day 43) separate a build stage (full JDK, Maven) from a minimal runtime stage (JRE only), crossing only the compiled artifact via `COPY --from=`.
- **Docker Compose** (new, Week 7 Day 44): same-network containers resolve each other by **service name** via DNS, automatically — `localhost` from inside one service refers to that container itself, not to any other service in the file, a common source of "connection refused" the first time a `localhost`-wired app is containerized. `depends_on` guarantees container **start order only**, not dependency **readiness** — a real production setup closes this gap with a `healthcheck` + `condition: service_healthy`, not `depends_on` alone. Named volumes persist independently of a container's own lifecycle, needed for any stateful service (Postgres) but not a stateless one (the app itself).
- **JUnit 5 and AssertJ** (new, Week 7 Day 45): `@BeforeEach`/`@AfterEach` exist to guarantee test independence — no test may rely on state left behind by another, since execution order isn't guaranteed. AssertJ's fluent chains (`assertThat(x).hasSize(3).first().matches(...)`) aren't just shorter than JUnit's built-in assertions; a failure reports specifically which condition on which element broke, which separate `assertEquals`/`assertTrue` calls can't do in isolation.
- **Mockito, and unit vs. integration testing** (new, Week 7 Day 46): `@Mock` fakes a dependency entirely (no I/O, no real implementation behind it at all) — genuinely different from `@DataJpaTest` (Day 45), which is a real embedded database and Spring context. This is the testing-pyramid distinction, cite by name going forward: a **unit test** (Mockito) verifies one class's logic in total isolation, fast; an **integration test** (`@DataJpaTest`, TestContainers) verifies real collaborating pieces actually work together, slower but closer to production reality. Neither replaces the other.
- **Binary Trees** (new, Week 7 Day 46, opened at 0/15): `TreeNode` is Linked Lists' self-referential `Node` shape (Week 5 Day 34) with **two** references (`left`, `right`) instead of one (`next`) — cite this lineage rather than presenting recursion-over-trees as a new mechanism; only the shape being recursed over is new. **Depth** (edges from root) and **height** (edges to farthest leaf) are graph-theory standard with base case `null → -1`; LeetCode's own "Maximum Depth" convention counts *nodes*, not edges, with base case `null → 0` — the two base cases must never be mixed, flagged explicitly as this pattern's signature off-by-one trap. The **universal recursive template** — base case for `null`, combine `left`/`right` results — is the single sentence nearly every DFS tree problem this series will pose reduces to; cite it by name rather than re-deriving it per problem. Space complexity is stated as **O(h)** (tree height: O(log n) balanced, O(n) worst-case skewed), not defaulted to a loose "O(n)."
- **TestContainers** (new, Week 7 Day 47): closes a real gap Day 45's `@DataJpaTest` + H2 leaves open — H2 doesn't enforce Postgres-specific dialect behavior, so a suite fully green against H2 can still fail against real Postgres in production. Spins up a real, disposable Docker container (needs Day 43's Docker directly); `@DynamicPropertySource` exists specifically because the container's port is assigned dynamically and isn't known until the container has already started.
- **Kafka Fundamentals** (new, Week 7 Day 48): a topic's partitions enable parallelism; Kafka guarantees ordering **only within a single partition**, never across an entire topic — don't overstate this guarantee when citing it later. Sequential disk appends are the specific mechanical reason for Kafka's throughput (sequential writes dramatically outperform random-access writes on both mechanical and SSD storage) — the same sequential-vs-random-access principle Day 40's B-tree/index discussion established from the read side, cited here applied to writes.
- **Binary Search Trees** (new concept, Week 8 Day 50, within the already-open Trees pattern): `TreeNode` shape and the universal recursive template (Day 46) are **not** re-derived — a BST layers exactly one new invariant (left subtree entirely smaller, right subtree entirely larger, globally) on top. O(log n) is explicitly **conditional** on balance, not guaranteed by the ordering rule alone (a skewed insertion order degrades to O(n)) — contrast directly with `TreeMap` (Day 17, Red-Black tree, unconditional O(log n)) whenever this resurfaces.
- **Kafka Consumers** (new, Week 8 Day 50): consumer **group**, not the topic itself, determines what's "already read" — within one group, each partition is read by exactly one consumer at a time; a differently-named group reads the entire topic independently from scratch. Auto-commit's specific, concrete failure mode: a crash after an offset commits but before processing finishes silently loses that message. Manual commit fixes that but requires idempotent processing, since redelivery on crash becomes a designed-for case, not a bug.
- **Divide and Conquer, formalized** (new, Week 8 Day 51): three named steps — divide, conquer, combine. Every tree DFS problem since Day 46 already technically followed this shape, but with a **free** divide step (`node.left`/`node.right`, no computation). Construct Binary Tree from Preorder and Inorder (Day 51) is the first problem where the divide step itself requires real computed work (an inorder-index lookup) before recursion can begin — that's the specific new element, not the recurse-and-combine shape, which isn't new. Day 35's LC 23 mention was the light, unbuilt preview.
- **Spring Profiles** (new, Week 8 Day 51): `spring.profiles.active` selects which profile-specific file's values layer on top of the shared `application.yml`, at startup — one build artifact, different runtime behavior per environment. Explicitly scoped as **distinct from, and lighter than**, the full Spring Cloud Config Server pattern (a separate, centrally-serving, Git-backed microservice with `@RefreshScope` for live reload) — the latter is named but not built; don't conflate the two if "Spring Cloud Config" resurfaces later.
- **Serialize/Deserialize null-marker technique** (Week 8 Day 52): Day 47's Same Tree brute force seeded this (serialize both trees using explicit null markers, compare the resulting strings) but only used it one-directionally, for comparison. Day 52 builds the reverse direction — reconstructing a tree *from* a serialized string. The core argument: a bare preorder sequence with no null markers is ambiguous (multiple distinct tree shapes can share the same real-node-only preorder sequence); explicit null markers remove that ambiguity, which is why **one** traversal suffices here where Construct Binary Tree needed **two**.
- **WireMock** (new, Week 8 Day 52): contrast directly with TestContainers (Day 47) — TestContainers provides a **real** dependency you own, for realism (H2 wasn't Postgres-realistic enough); WireMock provides a **fake** stand-in for a dependency you don't own, for control (a real external API can't reliably be forced into specific failure states on demand). Forward-referenced to Resilience4j, still weeks out in the plan.
- **Median of Two Sorted Arrays / binary search on a partition point** (Week 8 Day 53): a third Binary Search flavor, distinct from both "on the input" and "on the answer" (Day 28, formalized Day 33) — binary searches over a **partition index** in the smaller array, with the partner index in the larger array forced by a size condition, not independently searched. Correctness requires two conditions (size, value), both citable directly rather than re-derived. Fulfills the Day 21 promise the original plan's own numbering left open for 99 days.
- **Trees, fully closed** (Week 7 Days 46–49 + Week 8 Days 50–53, 15/15 required + 5 extra = 20 distinct): closed spanning DFS, BFS, BST traversal and validation, divide-and-conquer construction, general-tree LCA, and serialization. The single sentence distinguishing BST LCA from general-tree LCA, worth citing verbatim rather than re-deriving: a BST's ordering invariant lets you *compute* which single subtree to search, where a plain tree carries no such information and must search both, combining the results. House Robber III and Binary Tree Maximum Path Sum remain deliberately deferred to Dynamic Programming, per the plan's own note — not pulled forward despite fitting thematically.
- **Heaps / Priority Queue, the actual mechanism** (new pattern, Week 8 Day 54, opened at 0/10): `PriorityQueue` itself is **not** new — named Day 17 (alongside Comparable/Comparator), actually used Day 26 (Meeting Rooms II's min-heap of end times). What's new: the array-backed complete-binary-tree mechanism itself — index math (children at `2i+1`/`2i+2`, parent at `(i-1)/2`), sift-up (insert), sift-down (remove-root), and the precise argument for why a heap's O(log n) is **unconditional** where a bare BST's is not (complete-tree structure enforced by construction on every insert, not dependent on insertion order). Custom `Comparator` ordering (Day 55 onward) reuses the anonymous-inner-class technique from Day 35 — lambdas still not formally taught, so this is not new syntax.
- **Quickselect** (Week 8 Day 55): Day 12's Sort Colors named this as a future destination ("resurfaces with Quickselect later") — delivered here. Same one-pass-partition-around-a-pivot lineage as Day 12's Dutch National Flag, reduced from three zones (fixed 0/1/2 values) to two (≤pivot, >pivot), with the pivot drawn from the array itself. O(n) average vs. quicksort's O(n log n) — both built on the identical partition step — because Quickselect recurses into only *one* side of each partition, not both; O(n²) worst case, with randomized-pivot selection as the standard, citable mitigation.
- **Heaps, greedy-scheduling role** (new heap role, Week 9 Day 57): a max-heap driving "act on the best candidate right now," distinct from every Week 8 heap role (all of which were either fixed-collection queries or candidate generation, not a placement decision with an ordering constraint). The specific mechanism — hold the just-placed entry out of the heap for exactly one round (Reorganize String) or park it in a cooldown structure until a computed re-entry time (Task Scheduler) — is the citable technique if a "greedy placement with a no-immediate-repeat constraint" problem resurfaces later.
- **Two Heaps** (new heap role, Week 9 Day 58): a max-heap (smaller half) and min-heap (larger half) kept balanced within one element of each other, giving O(1) access to a running median over an unbounded stream. The citable mechanism: push unconditionally to one designated heap, shuffle its extreme value across to the other, then rebalance only if that shuffle overshot — not a branch-then-insert decision.
- **k-Way Merge via Heap** (new heap role, Week 9 Day 58): a min-heap holding exactly one "current head" entry per active source, bounded at size k regardless of total element count N — the direct generalization of Day 35's two-list merge (LC 21) to k sources. Divide-and-conquer pairwise merging is the equal-complexity (O(N log k)), heap-free alternative, citable by name if asked "can you avoid the heap."
- **Heaps, fully closed** (Week 8 Days 54–56 + Week 9 Days 57–58, 10/10 required + 3 extra = 13 distinct): six genuinely distinct roles across the two weeks — kth-largest query, keep-the-k-best, minimize-combination-cost, candidate-generator, greedy-scheduling, and two-heap-balance/k-way-merge-coordination. Cite this six-role enumeration directly if a "why no more heap practice" question resurfaces, rather than re-deriving it.
- **Trie** (new structure, Week 9 Day 59, opened and closed within the week at 6/6 core): generalizes `TreeNode` (Day 46) from 2 named children to 26 indexed ones — the recursion pattern over the structure is unchanged, only branching factor and edge meaning differ. `insert`/`search`/`startsWith` are all O(m), independent of how many words n are stored — contrast directly with `HashSet<String>` (O(1) average exact-match, but no efficient prefix query at all) whenever the choice between them resurfaces.
- **Trie, constrained descent** (Week 9 Day 60, reused Day 61): only recurse into a child where `isEndOfWord` is true — a traversal *constraint*, not a final check, first used to validate that every prefix of a candidate word is itself a complete word (Longest Word in Dictionary), then reused directly for dictionary-root substitution (Replace Words, Day 61).
- **Trie, branching descent** (Week 9 Day 60): a wildcard character tries all 26 possible children recursively instead of following one — worst case O(26^k × (m−k)) for k wildcards in a length-m word, simplifying to O(26^m) when fully wildcarded; only O(m) in the wildcard-free fast path. Cite the precise bound, not the fast-path shorthand, if this resurfaces.
- **Trie + Matrix Backtracking** (Week 9 Day 61): a single Trie shared across all target words prunes a board backtrack the instant no remaining word matches the current path's prefix — words sharing a prefix share that exploration exactly once, rather than once per word. Visited-marking via overwriting the board cell (restored on backtrack) is the established technique going forward for any grid-backtracking problem, avoiding a separate `visited` array.
- **Backtracking, formalized** (new pattern, Week 9 Day 61, opened at 0/12): the choose → explore → un-choose template, built on recursion (Day 8) as its only real prerequisite. Distinguished from plain tree DFS (Days 46+) by the explicit "un-choose" step — necessary because backtracking typically mutates *one shared structure* across the whole decision tree, where tree DFS recurses into structurally independent subtrees that never need restoring. Distinguished from Dynamic Programming (not yet taught) by enumerating a full combinatorial solution space rather than exploiting overlapping subproblems for a count or optimum.
- **Backtracking, include/exclude variant** (Week 9 Day 61): each element gets one independent binary choice, producing a complete binary decision tree of depth n, 2ⁿ leaves — used for Subsets. Contrast directly with the swap-based variant below whenever the choice between them resurfaces.
- **Backtracking, swap-based variant** (Week 9 Day 62): fixes one position at a time, swapping every remaining candidate into it in turn, recursing, then swapping back — "which elements are placed" is encoded in the array's own layout via the swap, needing no separate `used` structure. Used for Permutations; explicitly NOT a repeat of the include/exclude shape, since each position's choice depends on what earlier positions already claimed.
- **Backtracking, forward-index variant** (Week 9 Day 63): recursion only ever advances the starting index forward, never revisiting — since order doesn't matter for a combination, this alone guarantees every combination is built in exactly one canonical increasing form, with no separate duplicate-avoidance structure needed. Used for Combinations; explicitly the wrong choice for Permutations, where order matters and every position must be reachable from every earlier one.
- **Backtracking, duplicate-value suppression** (Week 9 Day 63): sort first, then at each recursion depth skip a candidate if it equals the previous candidate **and** the previous one is *not* currently part of the active path (`!used[i-1]`, not `used[i-1]`) — `!used[i-1]` true means the previous identical value was already fully explored and backtracked past at this exact depth, so choosing this one now would just re-explore an equivalent sibling branch; `used[i-1]` true means the previous copy is legitimately part of the *current* path, and reusing a duplicate value within one path is required, not forbidden. Proven via a full `[1,1,2]` trace (Day 63) rather than stated as a rule — cite that trace directly if this condition resurfaces, rather than re-deriving it under time pressure.
- **CAP Theorem** (new, Week 9 Day 59): during a network partition, choose Consistency (every read reflects the latest write, or errors) or Availability (every request gets a response, possibly stale) — not an unconditional "pick 2 of 3," since partition tolerance isn't a real design choice for a genuinely distributed system. PACELC named as the extension covering the no-partition case (latency vs. consistency). MongoDB/Cassandra cited as CP/AP **defaults**, explicitly not fixed classifications — both expose tunable consistency levels.
- **Consistent Hashing** (new, Week 9 Day 60): servers and keys hashed onto one ring; a key belongs to the next server clockwise, found via `TreeMap.ceilingKey()` (O(log S)) with wraparound via `firstKey()`. Remaps only the keys in the ring segment adjacent to a changed server, vs. naive `hash % N`'s near-total remap on any server-count change. Virtual nodes are the citable fix for uneven ring distribution with few physical servers.
- **Replication Models** (new, Week 9 Day 61): single-leader (simple, leader is a bottleneck/SPOF), multi-leader (needs conflict resolution), leaderless/quorum-based (`R+W>N` for strong consistency). Synchronous replication trades latency for durability; asynchronous trades durability for latency/throughput. Explicitly reconnected to Day 59's CAP framing: single-leader-sync leans CP, leaderless-async leans AP — cite this direct connection rather than treating the two topics as unrelated.
- **Spring AOP** (new, Week 9 Day 62): Aspect (what), Pointcut (where), Advice (when — Before/After/Around/etc.). Proxy-based (JDK dynamic proxy for interfaces, CGLIB otherwise) — the proxy sits *outside* the real bean, which is exactly why self-invocation (`this.method()`) bypasses any Advice entirely; this is the citable explanation if an `@Transactional`-not-firing question resurfaces, not just "self-invocation doesn't work."
- **Multi-module Maven** (new, Week 9 Day 62): parent POM (`packaging=pom`, `<modules>`, `<dependencyManagement>` for centralized *version* control only) vs. child POMs (each declaring its own `<parent>` and its own `<dependencies>` entries — `dependencyManagement` alone adds nothing to a child that doesn't also declare the dependency itself). `mvn clean install` from the root resolves the reactor build order automatically.
- **Backtracking, forward-index + repetition** (Week 10 Day 64): Day 63's forward-index template (Combinations), with exactly one line changed — recurse with `i`, not `i+1` — leaving the current index eligible to be chosen again. Used for Combination Sum. The entire "repetition allowed" rule expressed as a single changed recursive argument.
- **Backtracking, duplicate-skip translated to forward-index form** (Week 10 Day 64): `i > start && nums[i]==nums[i-1]` — skip a repeated value only when it's *not* the first choice offered at the current recursion depth. Mechanically distinct from Day 63's `!used[i-1]` (which inspects a `used[]` array's live membership state, only meaningful in a swap-based model) despite solving the same category of problem — the idea transfers across backtracking's choice models, the code does not. Proven via a `[1,1,2]`/target-4 trace showing `i > 0` in place of `i > start` silently drops a valid combination. First taught Combination Sum II; cited directly (not re-derived) by Subsets II, Week 10 Day 67.
- **Resilience4j Circuit Breaker** (new, Week 10 Day 64): CLOSED (normal) → OPEN (failure-rate threshold crossed; real calls skipped entirely, fallback fires immediately) → HALF_OPEN (trial calls after a wait period) → CLOSED or back to OPEN. Exists to stop cascading failure — one slow dependency exhausting the caller's own thread pool, starving unrelated requests of any thread to run on. Proxy-based, the exact same mechanism as Day 62's Spring AOP; self-invocation bypasses it identically. Fallback method matched by reflection: same signature + trailing `Throwable`.
- **Feign Clients** (new, Week 10 Day 65): a plain interface + HTTP mapping annotations; Spring generates a real implementing HTTP client via reflection at startup (same annotation-driven-codegen family as `@RestController`). Fallback is a whole class implementing the interface, not a single method — broader scope than Resilience4j's method-level fallback, though the two compose.
- **Spring Cloud Gateway** (new, Week 10 Day 66): single external entry point; centralizes cross-cutting concerns (routing, then auth, then rate limiting) that would otherwise be independently implemented and drift across every module. Does NOT sit in the path of internal service-to-service calls (e.g., Day 65's Feign call, Order→Product) — only external traffic passes through it.
- **JWT vs. OAuth 2.0** (new, Week 10 Day 67): JWT is a self-contained, signed **token format** — claims verifiable via signature + expiry with no DB round-trip. OAuth 2.0 is a separate **authorization protocol** governing how a token gets issued to a third party. Frequently paired (OAuth flows often issue JWTs as access tokens) but not synonymous — a system can use JWTs with no OAuth involved at all, which is exactly what Day 67 builds.
- **Graphs, why `visited` is now mandatory** (new pattern, Week 10 Day 68, opened at 0/12): a tree is a connected, acyclic graph with exactly one path between any two nodes — structurally incapable of a traversal revisiting a node. A general graph carries no such guarantee; an unguarded traversal on a graph containing a cycle can revisit nodes indefinitely. Every graph traversal from Day 68 forward carries an explicit `visited` structure **for this reason specifically** — conditioned on cycles being *possible*, not an unconditional rule (see Day 70's entry below for the case where it correctly doesn't apply).
- **Adjacency list vs. adjacency matrix** (Week 10 Day 68): list — O(V+E) space, O(degree) neighbor/edge-existence lookup, default for sparse graphs (most interview graphs). Matrix — unconditional O(V²) space regardless of actual edge count, O(1) specific-edge-existence check, right only for dense graphs or edge-existence-heavy workloads. "A grid is a graph in disguise" — adjacency is computed from coordinates (`r±1,c`/`r,c±1`), never materialized.
- **BFS vs. DFS, the interview signal** (Week 10 Day 68): "shortest path / minimum steps / fewest moves" on an unweighted graph → BFS, unconditionally (level-by-level order guarantees the first arrival is via fewest edges). "Does a path exist / explore everything reachable / is this connected" → DFS (or BFS; DFS is often simpler code for pure reachability).
- **Graph DFS vs. Backtracking, the actual test** (Week 10 Day 68, sharpened Day 70): NOT "is it recursive" or "does it explore neighbors" — both look alike. The test is whether an un-choose step is needed: backtracking mutates one shared structure across sibling branches that must be restored (Day 70's All Paths From Source to Target — real `path.remove()` needed); plain graph traversal marks permanently, with nothing to restore (Day 68's Number of Islands — no un-choose exists). Two visually similar recursive graph problems this week reached opposite, individually-justified classifications — treat this as a per-problem question, not a lookup table.
- **Multi-source BFS** (Week 10 Day 70): seed the queue with every starting node before the first iteration, instead of one. Everything else about Day 49's `queue.size()` level-isolation trick is unchanged — the level counter simply now measures simultaneous distance from many sources rather than one. Used for Rotting Oranges (a shared counter, one global answer) and, as extra practice, 01 Matrix (each cell's own distance, computed directly from whichever source reached it first — no shared counter needed).
- **Multi-source traversal's two distinct roles — distance vs. reachability** (Week 10 Day 70 vs. Week 11 Day 72): seeding multiple sources at once answers two structurally different questions depending on what's asked. *Distance* (Day 70: Rotting Oranges, 01 Matrix) needs BFS specifically, for its level-order guarantee. *Reachability* (Day 72: Surrounded Regions, Pacific Atlantic Water Flow) only needs "can this be reached at all," which DFS answers equally well — there is no level-order requirement to preserve. Conflating the two into one undifferentiated "multi-source BFS" habit is the exact trap Day 72 was built to correct.
- **The reversed-traversal proof technique** (Week 11 Day 72, Pacific Atlantic Water Flow): when a forward condition (`height(neighbor) ≤ height(current)` for real flow) is expensive to check from every cell, search backward from the small target set instead, using the *logically identical* reversed condition (`height(neighbor) ≥ height(current)`) — the two inequalities are the same statement read in opposite directions, so a reversed path found this way, read backward, is a genuine forward-valid path, not an approximation. The same "search from the small target set outward instead of from every point inward" idea also drives Surrounded Regions the same day, from a different (definitional, not inequality-reversal) angle.
- **Topological Sort, two approaches, one already-taught** (Week 11 Day 71): Kahn's Algorithm (BFS + in-degree tracking, new this week) and DFS 3-state coloring (white/gray/black) — the latter is not new material, it's Day 69's Find Eventual Safe States cycle-detection technique (unvisited/visiting/safe) generalized from "is this node eventually safe" to "does any cycle exist, and if not, what's a valid order." Topological order is proven to be the *reverse* of DFS finish order: a node can't finish before everything it points to has finished, so for any edge `u→v`, `v` always finishes first.
- **Union-Find's complexity progression, each step proven, not just stated** (Week 11 Day 74): naive (fixed-direction unions can build a degenerate chain) → O(n) worst case per operation. Union by rank alone → O(log n) worst case, proven via a doubling induction: rank only increases when merging equal-rank trees, and each such merge at least doubles the resulting tree's minimum node count, so a rank-`r` tree needs ≥2^r nodes, bounding rank (and height) by log₂(n). Path compression added → O(α(n)) amortized combined with rank, where α is the inverse Ackermann function, ≤4 for any practically-occurring `n`.
- **"Union the abstraction, not the raw input"** (Week 11 Days 75–76, recurring across four problems): the elements being unioned are frequently not the problem's literal input objects. Rows and columns, not stones (Most Stones Removed). Equation variables, not equations (Satisfiability). Index positions, not characters (Smallest String With Swaps). Emails, not accounts (Accounts Merge). In each case, identifying the correct abstraction to union is the actual insight; the union-find mechanics themselves are unchanged from Day 74's class.
- **The Cut Property, and its exchange-argument proof** (Week 11 Day 77): for any partition of a graph's vertices into two non-empty sets, the minimum-weight edge crossing that partition belongs to some MST. Proven the same way Week 4's Greedy Algorithms proved its own exchange arguments: assume the minimum crossing edge `e` is absent from some MST `T`; adding `e` to `T` creates a cycle that must cross the cut an even number of times (it starts and ends on the same side), so some other edge `e'` in that cycle also crosses it, with `weight(e) ≤ weight(e')` by `e`'s minimality; swapping `e'` for `e` yields a spanning tree no heavier than `T` that contains `e`. This single property is what licenses both Kruskal's (global sort) and Prim's (grow-from-a-node) as correct, not merely reasonable-sounding, greedy choices.
- **Token Bucket rate limiting** (new, Week 10 Day 68): two independent, simultaneously-enforced guarantees — bounded burst (capacity) and bounded long-run average rate (continuous refill). A naive fixed-window counter fails the burst guarantee specifically (allows up to 2N requests clustered across a window boundary); continuous refill avoids that discrete-jump gap. Requires `synchronized` under concurrency — a real correctness bug without it (Day 37's Counter-corruption shape), not just a performance concern.
- **Kafka Schema Registry** (new, Week 10 Day 69): central, versioned event-schema store; enforces a compatibility mode (commonly backward) at publish time, rejecting a change that would break existing consumers *before* it ships, rather than discovering the break later inside some unrelated downstream service. Requires Avro (or similar) specifically because compatibility-checking needs an actual registrable schema to compare against — plain JSON carries no enforced contract at all. The JPA entity (persistence detail) and the Avro event schema (public contract) are deliberately kept as two separate shapes, not one.
- **Saga, Choreography vs. Orchestration** (new, Week 10 Day 70): a multi-service business transaction broken into independent local transactions, chained by events, since no single database transaction can span independently-deployed services. Choreography: no central coordinator, each service reacts to events it's subscribed to (this week's platform, built directly on Day 48's Kafka pub/sub). Orchestration: one coordinator explicitly directs every step (full flow visible in one place, at the cost of a central dependency). Compensating transaction ≠ database rollback — the earlier local transaction already committed; undoing it needs a new, explicit, forward-moving operation. 2PC's availability cost (locks held, blocked on the slowest participant) connects directly to Day 64's cascading-failure argument, at transaction scope instead of single-call scope.
- **Dijkstra's Algorithm** (new, Week 12 Day 78): single-source shortest path, non-negative weights only. Min-heap of `(distance,node)`, finalize-on-pop, relax neighbors, skip stale heap entries (lazy deletion — Java's `PriorityQueue` has no `decrease-key`). Correctness is a domination argument, same *shape* as the Cut Property (Day 77) though a different specific property: any path that could beat the greedily-finalized node would have to pass through a currently-farther frontier node first, and non-negative weights mean that detour can only add distance, never recover it. Structurally a five-line diff from Prim's (Day 77) — same loop, same heap discipline; only what's being minimized changes (cumulative source distance vs. one edge's weight into a growing tree). Breaks outright under negative weights, silently (wrong answer, not a crash), since a negative edge can undercut an already-"finalized" node the algorithm never revisits. O(E log V) with a lazy-deletion binary heap.
- **Dijkstra's Algorithm, adapted relaxation rules** (Week 12 Days 78–80): the mechanism is a *shape*, not a fixed recipe — three variants taught, each with the same proof skeleton surviving a different combining operator. Sum-minimizing (Day 78, textbook). Multiplicative/max-heap (Day 78, Path with Maximum Probability — probabilities ≤1 mean extending a path can only shrink the running product, flipping "smallest wins" to "largest wins"). Minimax, both edge-weighted (Day 79, Path With Minimum Effort) and node-weighted (Day 80, Swim in Rising Water) — `candidate = max(running, newWeight)` in place of a sum, valid because `max(a,b) ≥ a` always, the same non-decreasing-extension property the proof actually needs.
- **Bellman-Ford** (new, Week 12 Day 79): relax every edge in the graph, `k+1` times, against a **frozen snapshot** of the previous round's distances — never update in place mid-round, or a round can silently chain multiple edges together and break the "round `r` = at most `r` edges" guarantee the correctness proof (induction on round number) depends on. Handles negative weights and bounded-hop-count constraints Dijkstra's structurally cannot, at a strictly worse O(k×E) vs. O(E log V). Used for Cheapest Flights Within K Stops specifically because plain Dijkstra's greedy finalization can lock onto a cheap-but-stop-exhausted path and never explore a pricier-but-fewer-stops alternative — proven via a concrete counterexample, not asserted.
- **Sharding Strategies** (new, Week 12 Day 78): splits *different* data across multiple database instances — the write/storage-scaling complement to Replication (Day 61), which copies the *same* data for redundancy/read-scaling; the two are layered together, not alternatives. Range-based: cheap range queries, vulnerable to hot spots when access skews toward one range. Hash-based: even load regardless of range skew, expensive range queries (fan-out to every shard). Naive hash-based sharding (`hash(key) % N`) is exactly the scheme Consistent Hashing (Day 60) was built to fix — production hash-based sharding is almost always consistent-hash-based in practice, specifically to avoid a near-total remap when the shard count changes. The "celebrity problem" (one key receiving disproportionate traffic) is solved by neither strategy, by construction — both balance the key *space*, not one key's own popularity.
- **Two-Phase Commit** (new, Week 12 Day 79): atomic commit across multiple participants, coordinated by one designated coordinator. Prepare (every participant votes yes/no, a yes vote is binding) then Commit-or-Abort (unanimous yes → commit; any no → abort). Blocks specifically when the coordinator crashes between a participant's yes vote and receiving the final instruction — that participant can't unilaterally proceed either way, and is stuck holding its locks. The same mechanism as Day 64's cascading-failure argument (a held resource stalling everything queued behind it), just at transaction/lock scope instead of thread-pool scope. Contrasted directly with Saga (Day 70, recapped not re-taught): 2PC trades availability for strong atomic consistency; Saga trades strong consistency (a visible partial-completion window) for availability.
- **Kafka Streams — KStream vs. KTable, the stream-table duality** (new, Week 12 Day 80): `KStream` — an unbounded, event-by-event view of a topic, processed in arrival order within a partition (Day 48's ordering guarantee, unchanged). `KTable` — a changelog: the latest value per key, updated (not appended) as new records for that key arrive, backed internally by a compacted changelog topic plus typically a local materialized state store (e.g. RocksDB). The duality, precisely: a `KTable`'s changelog is itself a `KStream` of updates, and any `KStream` can be aggregated into a `KTable` — the same infrastructure serves both views, which is *why* a `KTable` works, not just what it's called. Built directly on Kafka Fundamentals (Day 48), Consumer Groups (Day 50), and Schema Registry (Day 69).
- **Dynamic Programming, formalized** (new pattern, Week 12 Day 81, opened at 2/33): solving a problem by breaking it into subproblems, solving each exactly once, reusing the result every time it recurs. Requires both **optimal substructure** (the whole problem's optimal answer is built from its subproblems' optimal answers) and **overlapping subproblems** (the same subproblem recurs during a naive solve) — Divide-and-Conquer (Day 51's tree construction) has the first without the second, which is exactly why it's never memoized. Two implementations: memoization (top-down, recursion + `HashMap` cache — Day 4's structure, Day 8's recursion) and tabulation (bottom-up, iterative array, no recursion or `StackOverflowError` risk). Motivated by re-tracing `fib(5)`'s call tree from Day 8 and counting exact redundancy (`fib(2)`×3, `fib(1)`×5, `fib(0)`×3, 15 calls total for one answer) — proven, not asserted. Informally previewed twice already without the name: Kadane's Algorithm (Week 3, Day 21) and Floyd-Warshall (Week 11, Day 73), both cited directly per this map's own Week 11 handoff note. The discipline before writing any recurrence: state what `dp[i]` represents, in one precise sentence — verified against a concrete traced example every time this series does it, not assumed correct on inspection (Min Cost Climbing Stairs, Day 81, is the first case where an imprecise definition would have silently produced the wrong base case).
- **1D DP, the shared lookback family** (Week 12 Days 81–84): every problem this week builds `dp[i]` from a small set of earlier `dp[j]` values, varying only in *how* they're combined and *which* earlier values qualify. Always-sum (Climbing Stairs, Day 81 — counting, both cases always valid). Max, take-or-skip (House Robber family, Day 82 — optimizing, a real decision). Validity-gated sum (Decode Ways, Day 83 — each term must first pass a legality check before it counts at all). Dictionary/Set-gated OR across every valid split point, not a fixed lookback distance at all (Word Break, Day 84 — the first 1D DP problem this series where the set of relevant earlier states isn't fixed at "always `i-1` and `i-2`").
- **Unbounded Knapsack, opened** (Week 12 Day 84, Coin Change): the first DP problem in the series allowing unlimited reuse of a single "item" (a coin denomination), and the first time this series' state definition doesn't track which items built a sub-answer, only how many were used — the unbounded property falls directly out of that omission, not a rule bolted on. Sentinel value chosen as `amount+1` specifically (not `Integer.MAX_VALUE`) to avoid overflow on the recurrence's `+1` relaxation step. Loop order (coins-outer vs. amounts-outer) proven not to matter for *this* problem, since minimization is order-independent — flagged, not built, that a future combination-*counting* unbounded problem will need a specific loop order for the opposite reason. **0/1 Knapsack** (bounded, each item at most once) remains a distinct, not-yet-introduced counterpart, expected in Week 13.

**A gap found while extending this section for Week 14:** this list's last entry prior to today's additions was Day 84's Unbounded Knapsack — Week 13's own new terminology (0/1 Knapsack, Grid DP, String DP, Interval DP, and the length-ordered-fill-order requirement, among others) was never added here, even though the concept-dependency map, problem inventory, and "What Week 13 Leaves You Knowing Cold" sections were all properly extended. The same category of omission as the Weeks 1–10 cumulative-count bullet caught during the Week 11 extension (Running Totals section, above) — flagged here rather than silently left, but not backfilled in this pass, since doing so properly means re-deriving Week 13's own specific phrasing from its actual Resource Books, not reconstructing it secondhand from this map alone. Whoever next has reason to touch this section should treat Week 13's terminology as a standing open item.

- **State Machine DP, formalized** (new sub-pattern, Week 14 Day 92, opened at 0/4): `dp[i][state]` — a *situation* dimension added to the DP index for the first time, not just a position. 0/1 Knapsack (Day 85) and Predict the Winner (Day 91) each informally previewed state-style thinking without the name, the same "preview without the name" precedent Kadane's and Floyd-Warshall set for DP itself (Day 81). Two problems already solved via other techniques resurface as unconstrained special cases: LC 121 (Week 2, Day 14, Sliding Window) and LC 122 (Week 4, Day 23, Greedy) — the latter's exchange argument proven to fail the moment a fee or cooldown couples consecutive days.
- **State Machine DP, the transaction-count dimension** (Week 14 Day 93): `dp[i][k][state]`, `k` a budget advancing only on commitment (a buy), never completion (a sell) — a genuinely different kind of dimension from Day 92's situational one. The `k ≥ n/2` reduction (LC 188) is this series' first case of a DP problem's own parameter range collapsing it back to an earlier, simpler problem (LC 122) once the constraint stops binding.
- **Tree DP vs. plain tree recursion, the precise distinction** (new sub-pattern, Week 14 Day 94, opened and closed at 0/2): not the postorder-combine shape itself (Week 7, Day 46 already had that) — the distinction is whether a return value merely *supports* the parent's computation (Diameter of Binary Tree, Week 7 Day 48; also Binary Tree Maximum Path Sum, Day 94) or *is itself* the full DP state (House Robber III's `{notRobbed, robbed}` pair, Day 94). Two required problems this week land on opposite sides of that distinction despite both being labeled "Tree DP" — treated as the day's central lesson, not an aside.
- **Two's complement, derived, not stated** (new pattern, Week 14 Day 95, opened at 0/10): `-x = ~x+1`, proven from `~x=(2ⁿ-1)-x` and mod-`2ⁿ` wraparound arithmetic — the same wraparound Week 2, Day 10 already proved governs `int` overflow, now explained rather than only demonstrated. Chosen specifically so addition/subtraction need one circuit, with no sign-based special-casing.
- **`>>` vs `>>>`, proven, not asserted** (Week 14 Day 95): identical for any non-negative operand (a non-negative number's sign bit is already `0`, matching `>>>`'s constant `0`-fill exactly); diverge only when sign-extending a `1` — proven with `n=-8` (`>>` → `-4`, `>>>` → `2147483644`), not stated as a rule to memorize.
- **XOR's cancellation, and its limits** (Week 14 Days 95, 97): proven by truth table (commutative, associative, identity, self-inverse) — the mechanism is really "count each bit's occurrences, take mod 2." Generalizes cleanly to mod `k` for any group size (Single Number II, Day 97, mod 3) — but does not degrade gracefully on its own; a concrete counterexample (`2^2^2=2`, not `0`) is required to show plain XOR actively fails on groups of three, not merely "doesn't help."
- **`n&(n-1)` and `n&(-n)`, two derived identities, precisely distinguished** (Week 14 Days 95, 98): both built from the same lowest-set-bit borrow/negation mechanics — `n&(n-1)` *clears* the lowest set bit (derived from `n-1`'s borrow ripple), `n&(-n)` *isolates* it (derived by applying that same ripple argument to `-n=~n+1` instead) — complementary operations on the same bit position, not two unrelated tricks.
- **Bit Manipulation fuses with 1D DP** (Week 14 Day 96, Counting Bits): `dp[i]=dp[i&(i-1)]+1`, a bit-manipulation identity supplying a DP transition directly — the same cross-pattern fusion this series named once before (Word Break II fusing DP and Backtracking, Week 13 Day 86). Well-foundedness (`i&(i-1) < i`, always) proven from the same identity, not assumed.
- **Bit Trie, a Trie whose alphabet is bits, not letters** (new, Week 15 Day 99): Week 9's exact TrieNode/insert/traverse mechanism, `children[26]` (or a `HashMap`) swapped for `children[2]`. The greedy opposite-bit walk (Maximum XOR of Two Numbers in an Array) proven optimal via place-value dominance: for k-bit numbers, `2^i > 2^(i-1)+...+2^0`, so maximizing a higher bit is always safe to lock in first regardless of what it forces at lower bits — the same "prove the greedy choice, don't assert it" discipline this series has applied to every other greedy claim.
- **Kubernetes, from zero** (new track, Week 15 Day 99): Pod (wraps one-or-more containers, Week 7's Docker container as the direct prerequisite) → ReplicaSet (maintains a `replicas` count, the first concrete instance of the **declarative reconcile loop** — declare desired state, a controller continuously closes the gap to it, forever, contrasted directly with `docker run`'s one-shot imperative model) → Deployment (manages ReplicaSets, adds rolling updates) → Service (a stable address across a Pod set whose individual members and IPs are disposable). HPA (`autoscaling/v2`) is explicitly framed as *another instance of the same reconcile loop*, just computing its own desired `replicas` from live Metrics-Server data instead of a human-typed number; `averageUtilization` is measured against a Pod's CPU **request**, not node capacity — the single most common reason a first HPA silently does nothing.
- **Segment Tree, the series' first range-aggregation tree** (new structure, Week 15 Days 100–101, opened and closed at 2/2): Week 8 Heap's exact array-backed index math reused (`2i+1`/`2i+2`, 0-indexed) — only what's *stored* per node differs (a range aggregate, not a priority). Closes the gap neither Brute Force (O(1) update/O(n) query) nor Prefix Sum (O(n) update/O(1) query) can close alone, landing both operations at O(log n). Query's O(log n) bound proven via "at most two straddling nodes per level, every other node resolves in O(1)," not asserted. Demonstrated in two structurally distinct modes: indexed by array **position** (Day 100, Range Sum Query - Mutable) and indexed by **value**, via coordinate compression (Day 101, Count of Smaller Numbers After Self) — the second a genuinely different mental model of what a leaf represents, not the same technique restated. Trade-off against Fenwick Tree/BIT (extension, Day 101 — `i&(-i)` directly reusing Week 14 Day 98's isolate-lowest-set-bit identity): BIT is leaner but relies on invertibility (`rangeSum = prefixSum(r) − prefixSum(l−1)`), so it naturally covers sum/XOR but not min/max, which Segment Tree handles uniformly.
- **`volatile`, the series' first new concurrency primitive since Week 6** (new, Week 15 Day 100): fixes a specific, provable reordering bug in Double-Checked Locking, not a generic "thread safety" habit. `instance = new X()` is three conceptual steps (allocate, construct, assign); without `volatile`, the JVM may legally reorder the constructor's writes and the reference assignment, letting a second thread observe a non-null but partially-constructed object. `volatile` forbids exactly this reordering and establishes a happens-before edge on read/write of that field. Explicitly distinguished from `synchronized` (Week 6, Day 37): `synchronized` gives mutual exclusion, `volatile` gives visibility/ordering — DCL needs both, neither substitutes for the other.
- **Singleton, the spectrum** (new pattern, Week 15 Day 100): private constructor + static accessor is the shared shell; naive lazy (race condition, same shape as Week 6 Day 37's `Counter`), fully-`synchronized`-method (correct, permanent per-call overhead), Double-Checked Locking with `volatile` (correct, cheap after first call), and Enum Singleton (thread-safe via the JVM's own classloader-initialization guarantee, no lock needed) all taught as points on one spectrum, not independent facts. Enum Singleton's two attack-immunities both proven mechanistically: reflection (`Constructor.newInstance()` hard-codes an `IllegalArgumentException` check for enum types — verified against Oracle's own documentation) and serialization (enum deserialization resolves via `Enum.valueOf()` against the existing constant, calling no constructor at all).
- **Factory Method, Simple Factory formally distinguished from the true GoF pattern** (new pattern, Week 15 Day 101): a static method with a type-branching conditional ("Simple Factory," not one of the 23 GoF patterns, violates Open/Closed per Week 1 Day 6's SOLID) vs. an abstract creator with an abstract method overridden per concrete subclass (true Factory Method, respects Open/Closed — adding a type adds a subclass, touches no existing code). A commonly-conflated distinction, treated here as the day's central point rather than an aside.
- **Builder, immutability as the substantive benefit, not just fluency** (new pattern, Week 15 Day 101): solves the telescoping-constructor anti-pattern (shown directly, not just named) via fluent, named setter calls plus atomic, validated construction in `build()`. The built object's `final` fields are explicitly tied to thread-safety-for-free — a "Builder" over a still-mutable target keeps the readability benefit but loses this one, flagged as a common, substantive implementation mistake, not a style nitpick.
- **Merge Sort and Quicksort, built from scratch for the first time** (new algorithms, Week 15 Day 102): `Arrays.sort()`/`Collections.sort()` had been used as a black box since Week 1 Day 5's Group Anagrams without either underlying algorithm ever being implemented. Merge Sort: O(n log n) in *every* case (Master Theorem, `a=b=2`), stability proven mechanistically (`<=` in the merge, left-half-first on ties — copy-based, no swaps). Quicksort: Lomuto partition is Week 2 Day 12's Dutch National Flag boundary-pointer-and-swap mechanism, reduced from three zones to two; O(n²) worst case traced on sorted input as the same arithmetic-series shape this series has proven for other naive approaches; not stable, by direct contrast with merge sort's copy-vs-swap distinction. Java's real split — dual-pivot quicksort for primitives, TimSort (a genuine merge-sort variant) for objects — reconciled via three concrete reasons (stability's relevance, comparison cost/trust, in-place's locality payoff), verified against current sources rather than assumed from training data.
- **Quickselect, why its complexity class genuinely differs from quicksort's, not just its constant** (Week 15 Day 102): reuses quicksort's own partition method unmodified, but recurses into only the side containing the target index. `T(n)=T(n/2)+O(n)` (Quickselect) vs. `T(n)=2T(n/2)+O(n)` (quicksort) — the first is a geometric series dominated by its first term (`O(n)`), the second has O(log n) full-sized levels (`O(n log n)`) — presented side by side specifically so the O(n)-vs-O(n log n) split is derived from the recurrences, not asserted as a fact to memorize. Delivered as full-depth new content against a recapped problem (Kth Largest Element in an Array, LC 215, Week 8 Day 55 via min-heap) — the series' first case of a recap receiving a second full-depth technique rather than a bare short recap alone.
- **SQL logical query processing order** (new, Week 15 Day 103): `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`, established as the mechanical reason `WHERE` can't reference an aggregate but `HAVING` can — the foundation Day 104 extends by one clause for window functions.
- **Correlated subquery, precisely defined** (new, Week 15 Day 103): a subquery referencing a column from the outer query's current row, conceptually re-evaluated per outer row — distinguished directly from Day 103's own earlier independent subquery (Second Highest Salary), which needs no such re-evaluation.
- **Self-Join, two structurally distinct shapes** (new, Week 15 Day 103): a date-offset join (`DATEDIFF(...) = 1`, Rising Temperature — row-adjacency explicitly rejected as unsafe against date gaps) and a hierarchical self-reference join (`managerId = id`, Employees Earning More Than Their Managers — `INNER JOIN`'s NULL-exclusion tied directly back to Week 6's `NOT IN`/three-valued-logic material, not a new rule).
- **Window functions, the core GROUP BY contrast** (new, Week 15 Day 104): rows are never collapsed, unlike `GROUP BY` — every input row survives, gaining an added computed column. `ROW_NUMBER`/`RANK`/`DENSE_RANK` precisely distinguished on a tied dataset (unique-always / gap-after-tie / no-gap-after-tie respectively); `DENSE_RANK` proven — via a concrete tie counter-example on Day 104's Department Top Three Salaries, not asserted — as the only one of the three correct for "top N distinct values." `LAG`/`LEAD` access a neighboring row's value without a self-join. `PARTITION BY` explicitly framed as the generalization of Day 103's correlated-subquery "recompute per group" idea into full per-group ranking. A window function's own result cannot be filtered in the same query's `WHERE` (it's evaluated after `WHERE`/`GROUP BY`/`HAVING`) — requires a wrapping subquery, extending Day 103's logical processing order by one clause.
- **`CASE` expressions and conditional aggregation** (new, Week 15 Day 104): `CASE WHEN...THEN...ELSE...END` as a value-producing expression usable inside an aggregate; `SUM(CASE WHEN cond THEN 1 ELSE 0 END)` proven mechanistically (not just idiomatically) to count matching rows within a group, since the `CASE` contributes exactly 1 per matching row and 0 otherwise.
- **GoF's 3-category taxonomy, named explicitly for the first time** (new, Week 16 Day 106): Creational (how objects get created — Week 15's Singleton/Factory Method/Builder), Structural (how objects are composed into larger structures — Day 106's five), Behavioral (how objects communicate and vary behavior at runtime — Day 107's five). "Creational" and "GoF" had both been used as labels since Week 15 without the full 3-way split being named; framed as the fastest sanity check for whether a pattern is even being reached for on the right kind of problem.
- **Structural patterns, all five, told apart by the concrete problem each solves** (new, Week 16 Day 106): Adapter (two interfaces don't match — bridges without touching either); Decorator (behaviors need to combine freely, in combinations the class author can't enumerate ahead of time — wraps, doesn't subclass per combination); Facade (a multi-class subsystem needs a specific call order hidden behind one entry point); Proxy (controls access to an object without changing its own code — virtual/lazy, protection, and logging variants share one mechanism, differing only in what runs between "receive the call" and "delegate"); Composite (a recursive, whole-part structure needs uniform treatment of a single item and a group — leaf returns a base value directly, composite recurses-then-combines, structurally identical to Week 7 Day 46's Tree DFS postorder shape). Proxy vs. Adapter: Adapter changes the *interface*; Proxy keeps the same interface and controls access to it. Proxy vs. Decorator: Decorator adds genuinely new behavior the wrapped object never had; Proxy controls access to behavior it already fully has.
- **Behavioral patterns, all five, told apart the same way** (new, Week 16 Day 107): Observer (an unknown, possibly-growing set of things need to react to a change — 1-to-many, reactive, subject never named by type); Strategy (the *client* chooses which algorithm runs — 1-to-one, client-directed, implementations typically carry their own configuration data); State (an object's *own* operations drive transitions between a small, closed set of internal phases — states typically reference each other and are typically parameterless, reusable singletons); Command (a request needs to become a storable, undoable, queueable object — one mechanism, three payoffs: a `Deque`/`List` of stored commands supports undo, queuing, and logging alike); Template Method (several variants share one fixed, `final` sequence, differing in one or two `abstract` steps — the exact "abstract creator + abstract method" shape Week 15 Day 101's Factory Method used, aimed at varying a step instead of varying what gets created).
- **State vs. Strategy, the practical tell beyond the definitions** (new, Week 16 Day 109): both share identical code shape — a context delegating to an interface reference — differing in who drives the swap. A secondary, checkable signal drawn directly from the week's own code: State implementations are frequently parameterless and safely reusable as singletons (Day 109's `NoCoinState`, etc.); Strategy implementations frequently carry constructor configuration (Day 107's `PercentageDiscount(0.10)`). Not universal, but useful when intent alone doesn't settle it.
- **Test-Driven Development — Red/Green/Refactor** (new methodology, Week 16 Day 107, needs: JUnit — Week 7 Day 45): Red (a failing test, written before the implementation it calls exists, forcing the API to be designed from the caller's side) → Green (the minimum code to pass, kept small so a failure has one obvious cause) → Refactor (restructure with the passing test as a safety net). Applied to build Observer (Week 16 Day 107) from a single failing test through a multi-observer, removal-tested implementation — the added tests in Refactor were specifically ones only worth writing *because* Observer's Open/Closed guarantee makes them meaningful, not incidental coverage.
- **Coupling, Cohesion, and the Law of Demeter** (new, Week 16 Day 108, needs: encapsulation — Week 1 Day 5): Coupling — how much one class depends on another's internals (low is good); Cohesion — how focused a single class's own responsibilities are (high is good; a clean external interface can still hide low internal cohesion — the two are genuinely separate axes, not the same thing measured two ways). Law of Demeter — a method calls only its own fields, its parameters, objects it creates, or its direct fields, never a "train wreck" chain through an object another call returned; conventionally relaxed for simple, immutable data-holder objects with no real internals to leak (a two-hop chain reading a field off a DTO isn't the same risk as one reaching through a stateful service object). Applied retroactively against freshly-written code, not a separate illustrative example — Day 108 found and fixed one genuine violation (`game.getBoard().print()`) live in its own Tic-Tac-Toe implementation.
- **Composition Over Inheritance, formalized** (new, Week 16 Day 110, needs: Week 1 Day 6's Open/Closed; the instinct itself traces to Week 1 Day 5's `Runnable`-over-`extends Thread` preference): the tell is whether a varying dimension is a *value* along a natural order (favor a field — Parking Lot's `VehicleSize` enum, shared by both `Vehicle` and `ParkingSpot`) or a *genuinely distinct behavior* (favor a subclass, or a Days 106–107 pattern — Week 15's Creational hierarchy is the correct counter-example, not a contradiction). Proven, not just asserted: a hypothetical inheritance-based spot hierarchy (`MotorcycleSpot`/`CompactSpot`/`LargeSpot extends ParkingSpot`) was built and shown to require editing an *existing, already-tested* class (`LargeSpot`) the moment a new size tier is added — a direct, concrete Open/Closed violation, not a hypothetical one.
- **Lock granularity — whole-lot vs. per-level vs. per-spot** (new, Week 16 Day 111, needs: `synchronized`/`ReentrantLock` — Week 6 Day 37; `wait`/`notify` — Week 6 Day 38; `ConcurrentHashMap`'s CAS — Week 6 Day 39; `volatile`/JMM — Week 15 Day 100): a lock should scope to genuinely shared mutable state, no further — Parking Lot's per-`ParkingSpot` locking (via `synchronized tryAssign`/`removeVehicle`) contends only between threads targeting the *same* spot, since no state is actually shared between different spots. Generalizes directly to `ConcurrentHashMap`'s own lock striping (Week 6 Day 39): per-object locking is lock striping's limiting case, affordable exactly when the key set is small and fixed. `AtomicInteger` introduced as new API surface on an already-understood mechanism (Compare-And-Swap, already opened up inside `ConcurrentHashMap`), not a new concurrency primitive from zero — distinguished directly from `volatile`, which becomes redundant (not incorrect) once every read/write of a field already goes through the same `synchronized` block.


### Terminology and Framing Established — Week 18 Additions

- **System Design (HLD) 5-Step Framework** (Requirements → Estimation → High-Level Design → Detailed Design → Bottlenecks) — Day 120. Explicitly distinct from the **LLD 5-Step Framework** (Day 106); both named "5-step," different questions, different interview formats — the two are never used interchangeably going forward.
- **Base62 Encoding** — Day 120. A positional numeral system, 62-symbol alphabet (`0–9A–Za–z`).
- **Thundering Herd** — Day 123. Many concurrent requests missing an expired/cold cache key simultaneously, spiking redundant database load at once.
- **Cache-Aside Pattern** — Day 123. Read: check cache, populate on miss. Write: update DB, then evict (not update-in-place) the cache entry.
- **Quorum** — Day 125. A strict majority (more than N/2) of nodes in an N-node cluster; the same majority-overlap tool as Replication's R+W>N (Week 9, Day 61), applied here to leader election.
- **Split-brain** — Day 125. Two nodes simultaneously believing they are leader; prevented structurally by quorum's overlap guarantee.
- **Raft** — Day 125. Consensus decomposed into leader election, log replication, and safety; the mechanism underneath etcd, and therefore underneath Kubernetes' Control Plane (Week 15, Day 99), the whole time.

### Terminology and Framing Established — Week 19 Additions

- **Persistent WebSockets** — Day 127. A full-duplex, long-lived connection upgraded from an initial HTTP handshake (`101 Switching Protocols`); contrasted directly with HTTP's request-response model (Week 10, Day 72) and with long-polling/SSE.
- **Connection-Management Registry** — Day 127. A shared lookup (Redis hash) mapping a connected user to their specific server instance and connection — necessary because a WebSocket, unlike a stateless HTTP request, is pinned to one server for its entire lifetime; distinct from load-balancer sticky sessions, which answer a different question.
- **Wide-Column Store (Cassandra-style)** — Day 127. Partition key (node ownership) + clustering key (on-disk sort order within a partition); an LSM-tree write path defers rebalancing cost to background compaction, contrasted directly with a B-tree index's immediate write cost (Week 6, Day 40).
- **Geohashing** — Day 128. Interleaved-bit encoding of `(lat, long)` into a string where shared prefixes imply proximity; the boundary problem (nearby points can still land in different cells) requires checking neighboring cells, not just an exact-prefix match.
- **Quadtrees** — Day 128. Recursive quadrant subdivision that adapts to point density, contrasted directly with Geohashing's fixed-precision grid, above.
- **Redis Geo (`GEOADD`/`GEOSEARCH`)** — Day 128. Geohashing implemented as a Sorted Set keyed by a 52-bit interleaved score — the first operational payoff of Sorted Sets, named but not built on Day 123; the Skip List mechanism itself still deferred to Day 132.
- **Adaptive Bitrate Streaming & CDN** — Day 129. Chunked, manifest-driven quality switching at the client; a CDN is explicitly recognized as Cache-Aside (Week 18, Day 123) relocated to a geographically distributed edge tier, and shown to inherit Thundering Herd (Day 123) and its proactive-warming fix (Week 18, Day 126) as a direct consequence of that reframing.
- **Push / Pull / Hybrid Fan-out** — Day 129. Fan-out-on-write (cheap reads, write cost ∝ follower count) vs. fan-out-on-read (cheap writes, read cost ∝ follow count) vs. hybrid (push for ordinary accounts, pull for celebrity accounts) — the actual resolution of "the celebrity problem," first named Week 12, Day 78.
- **Idempotency Keys** — Day 130. A client-generated key representing one logical intent, not one HTTP attempt; the server checks for prior completion before acting, returning the stored result on a duplicate rather than re-executing — the general form of four earlier informal appearances (Days 121, 122, 124, 126).
- **Distributed Locks (Redis `SETNX`)** — Day 130. Atomic test-and-set with a TTL; safe release requires an atomic "check my token, then delete" (Lua), or a crashed-then-recovered holder can delete a different process's still-valid lock — traced as a concrete failure mode, not asserted.
- **Double-Entry Ledger** — Day 130. Every transaction recorded as a balanced debit/credit pair; the ledger sums to zero by construction, making corruption self-evidently detectable rather than something that must be separately discovered.
- **At-Least-Once vs. "Effectively-Once" Execution** — Day 131. True exactly-once requires distributed consensus most systems don't build; at-least-once delivery (SQS Visibility Timeout — the identical TTL-lease shape as Day 130's lock) plus idempotent processing (Day 130) produces an outcome indistinguishable from exactly-once from outside the system.
- **Skip List** — Day 132. Multiple levels of sorted linked lists, each a random subset of the level below; random node height (not deterministic promotion) makes insertion a local splice needing no rebalancing, giving expected O(log n) search/insert — the mechanism finally explaining Sorted Sets, named Day 123 and used operationally on Days 128 and 131.
- **Redis Cluster** — Day 132. Fixed 16,384 hash slots, rebalanced by explicit slot migration — a genuinely different mechanism from Consistent Hashing's ring (Week 9, Day 60), despite solving a similar-sounding problem; not to be conflated.
- **Content-Addressable Deduplication & Content-Defined Chunking** — Day 133. Storing data by a hash of its own content; content-defined (rolling-hash) chunk boundaries, unlike fixed-size ones, survive a small edit without breaking every downstream chunk's hash.
- **Sync Conflicts — LWW vs. Conflict-Copy** — Day 133. Last-Write-Wins risks silently discarding real work (and is vulnerable to clock skew); conflict-copy preserves both versions for manual reconciliation — the deliberate choice for a file-sync system specifically, not a universal default.
- **Percentile Latency (p50/p95/p99)** — Day 134. The Nth percentile of k sorted values is the value at position `ceil(N/100 × k)`; proven, on real constructed numbers, that average and even p95 can look healthy while p99 reveals a real, meaningful tail of slow requests.
- **Resource Requests vs. Limits** — Day 134. `requests` is a scheduler bin-packing input compared against node capacity; `limits` is a runtime-enforced ceiling — exceeding the memory limit gets a container killed (OOMKilled, memory isn't compressible), exceeding the CPU limit gets it throttled instead (CPU time is compressible) — two different enforcement mechanisms, not one.
- **Helm** — Day 135. `Chart.yaml` (metadata) + `values.yaml` (defaults) + `templates/` (Go-template YAML); `helm install` renders templates against values into plain Kubernetes YAML, submits it via the same API calls `kubectl apply` would use, and additionally stores the result as a numbered release, enabling `helm rollback`.
- **Kubernetes Ingress** — Day 135. A cluster object, interpreted by a separately-installed Ingress Controller, routing external traffic by host/path to internal Services — explicitly not the same layer as Spring Cloud Gateway (Week 10, Day 66), which does business-aware routing/auth/rate-limiting once traffic is already inside the application tier.
- **ConfigMap vs. Secret** — Day 136. Both inject externalized config, typically as environment variables; a Secret's value is base64-*encoded* by default, proven (not asserted) to be trivially reversible with no key required — real protection needs etcd encryption-at-rest or an external secret manager, neither built here.
- **"Build Once, Deploy Many Times"** — Day 136. One image promoted unchanged across environments, varying only externalized config around it; baking environment differences into the image itself breaks the core premise of testing, since the tested artifact and the shipped artifact would no longer be the same bytes.
- **Distributed Tracing (Trace ID / Span ID)** — Day 137. One Trace ID per originating request, shared across every hop it causes; one Span ID per unit of work, forming a parent-child tree. Propagates automatically across a synchronous HTTP call (instrumentation at the framework boundary) but *not* automatically across a Kafka message — Saga Choreography's (Week 10, Day 70) one genuine tracing blind spot, needing deliberate header propagation.
- **Cursor-Based vs. Offset-Based Pagination** — Day 137. Offset-based pagination can duplicate or skip items under concurrent inserts, traced concretely on real ranked positions; cursor-based pagination anchors to a specific item's own sort position, immune to that failure, at the cost of not supporting "jump to page N."
- **Chaos Engineering** — Day 138. Deliberately inducing a real failure (not a mocked one) to verify a configured resilience mechanism actually behaves as configured under it — the gap between "passes a unit test" and "survives a real, induced failure" is exactly what this closes.
- **Connection Pooling (HikariCP)** — Day 138. A fixed-size set of reused, already-authenticated database connections, avoiding a fresh TCP+auth handshake per request; saturating it can fail gracefully (clean queuing/timeout) or cascade (blocked threads exhaust the module's own HTTP thread pool, taking down requests that never needed the database at all) — a genuinely different failure shape from a dependency simply being down.
- **Service Mesh** — Day 138. A sidecar proxy injected into every Pod, transparently intercepting all network traffic; moves retry/timeout/routing behavior *out of application code and into infrastructure* — a categorically different mechanism than a library like Resilience4j (Week 10, Day 64), not merely a fancier version of it.
- **Liveness vs. Readiness Probes** — Day 139. Liveness failure kills and restarts the container (assumes a restart fixes it); readiness failure removes the Pod from its Service's Endpoints (Day 134's mechanism) without touching the container (assumes the problem is transient). Checking a downstream dependency in *liveness* is a well-known anti-pattern — a dependency's brief outage would restart every Pod simultaneously, making recovery worse, not better.
- **Mutation Testing** — Day 139. Deliberately introduces a small bug into production code and checks whether the existing test suite catches it; a surviving mutant proves the suite doesn't actually verify that logic, regardless of what a line/branch coverage percentage claims.

> ⚠️ **Small labeling flag, not silently fixed:** every entry above from Day 134 onward (Percentile Latency through Mutation Testing) is Week 20 content, not Week 19 — it was appended under the existing "Week 19 Additions" heading rather than getting its own "Week 20 Additions" subsection when Week 20 was generated. The content itself is accurate; only the heading it sits under is off. Left as-is here per this document's own extend-don't-edit convention; the Week 21 entries below correctly get their own heading, and any future reader relying on headings alone to find Week 20's terminology should check here instead.

### Terminology and Framing Established — Week 21 Additions (Series Final)

- **The STAR Framework, rigorously** — Day 141. Situation and Task each capped near one sentence; Action is first-person and decision-by-decision, not team-activity description; Result is quantified wherever a real number exists. A 90-second budget splits roughly 15/15/50/20 across the four parts, weighted toward Action since that's the part actually being evaluated.
- **Company-Agnostic Competency Map** — Day 141. Six competencies (Ownership, Navigating Ambiguity, Conflict & Disagreement, Failure & Learning, Technical Leadership, Cross-Functional Pushback) distilled as the common structure underlying Google's, Databricks', and Atlassian's own differently-labeled rubrics (Day 145 makes the mapping explicit). Ownership and Technical Leadership are distinguished by whose deliverable moved (yours vs. someone else's); Conflict & Disagreement and Cross-Functional Pushback are distinguished by which side of an org boundary the disagreement crossed.
- **UPI's NPCI/PSP/Bank-Node Architecture as Concrete 2PC Roles** — Day 141. NPCI is the coordinator, each bank's core banking node is a participant/resource manager, PSPs relay messages without holding the resource themselves — Two-Phase Commit's abstract roles (Week 12, Day 79), applied rather than re-derived.
- **Resolving an Ambiguous Transaction After a Mid-Transfer Network Failure** — Day 141. A participant left blocked by 2PC's own coordinator-crash failure mode (Week 12, Day 79) is resolved in practice by an idempotent retry (Week 19, Day 130's mechanism, reused) rather than a smarter commit protocol — the same consistency-vs-availability trade-off as Saga vs. 2PC (Week 10/12), recognized one layer down.
- **High Availability — Active-Active vs. Active-Passive** — Day 142. Active-Active serves live traffic from multiple instances simultaneously (no failover delay, higher cost); Active-Passive keeps a standby idle (a real detection-plus-promotion gap, lower cost). The actual failover trigger is a load balancer's health check — mechanically identical to Kubernetes removing a Pod from a Service's Endpoints on a failed readiness probe (Week 20, Day 134), not a separate mechanism.
- **High-Volume, Low-Margin Transaction Systems** — Day 143. The same tools (Caching, Sharding, Rate Limiting — Weeks 9/12/18) reframed through a cost-per-transaction lens rather than a pure-latency one; an inefficiency that's a minor annoyance at high margin becomes a direct, multiplicative cost against thin per-unit margin at volume.
- **Multi-Tenant Isolation Spectrum** — Day 144. Shared database/shared schema (lowest isolation and cost, highest blast radius) → shared database/separate schema → separate database per tenant (highest isolation and cost, lowest blast radius) — a genuine three-tier spectrum, not the plan's own binary framing.
- **Noisy Neighbor Problem** — Day 144. One tenant's heavy usage consumes shared infrastructure every other tenant also depends on; mitigated by per-tenant rate limiting (Week 18, Day 121, re-keyed by tenant) and connection-pool partitioning (extending Week 20, Day 138's HikariCP coverage) — rate limiting alone is insufficient since it caps request volume, not per-request cost.
- **Google's Googleyness & Emergent Leadership** — Day 145. One of four distinct Google hiring attributes (alongside general cognitive ability, leadership, and role-related knowledge), evaluated separately from technical skill — comfort with ambiguity, intellectual humility, collaborative instinct, and curiosity.
- **Databricks' Six Leadership Principles** — Day 145. Customer obsession, raising the bar, truth-seeking, operating from first principles, a bias for action, and putting the company first — the plan itself names only two ("seeking truth," "first principles"); the full six were independently verified against current public sourcing rather than assumed.
- **Atlassian's Five Values** — Day 145. Open Company, No BS; Build With Heart and Balance; Don't [Mess With] the Customer; Play, as a Team; Be the Change You Seek — given verbatim by the plan itself, evaluated in a dedicated round with real weight, not a token formality.

## What Week 1 Leaves You Knowing Cold

From Day 7's self-check and closing diagnostic — treat all of this as solid, not something later weeks need to re-establish:
- When to reach for `ArrayList` / `HashSet` / `HashMap` / `ArrayDeque`, without hesitation.
- Why `arr[i]` is O(1) from the memory-address mechanism, not just the label.
- The `equals()`/`hashCode()` contract and what breaks if it's violated.
- Naming which HashMap/HashSet sub-pattern or Two Pointers variant an unfamiliar problem calls for, before writing code.
- Why `ArrayList.add()` is O(1) on average, via amortized analysis, not just the conclusion.
- Interface vs. abstract class — by when you'd reach for each, not just the syntax.

## What Week 2 Leaves You Knowing Cold

From Day 14's self-check and Week 2 Consolidation — treat all of this as solid, not something Week 3 needs to re-establish:
- All 7 Two Pointers sub-variants, named and justified without checking hints — the pattern is **fully closed** at 16/16 required problems.
- What a stack frame is, why `StackOverflowError` is an `Error` not a `RuntimeException`, and why naive recursive Fibonacci is exponential in time but only linear in space.
- Pass-by-value for objects, precisely enough to explain *why* "pass-by-reference" is the wrong description — not just that it is.
- The Integer cache's exact range (-128 to 127) and why it's a disguised, sneakier form of `==` vs. `.equals()`.
- Why `String` concatenation in a loop is O(n²) even accounting for the compiler's automatic per-statement `StringBuilder` rewrite — and why `StringBuilder.append()` in a loop isn't.
- `ArrayList` vs. `LinkedList` vs. `ArrayDeque`, at the memory-layout level (contiguous array vs. linked nodes vs. circular array) — including why cache locality can matter more than the Big-O label.
- Sliding Window's mechanism and its lineage from same-direction (fast/slow) Two Pointers — but **not yet** variable-size window mechanics (shrink-on-violation), which Week 3 introduces fresh.

Week 3 continues Sliding Window immediately (12 more required problems, closing the pattern at 14 total) and deepens HashMap Internals (hashing into buckets, collision handling, treeification) as a direct extension of Week 1's HashMap/HashSet coverage. It assumes today's fixed-size window mechanism and the `.equals()`/`hashCode()` contract are both reflexive — nothing about variable-size windows or hashing internals themselves is assumed yet.

## What Week 3 Leaves You Knowing Cold

From Day 21's self-check and Week 3 Consolidation — treat all of this as solid, not something Week 4 needs to re-establish:
- Both Sliding Window loop shapes, named and distinguished without hesitation — **shrink-until-valid** (longest window) vs. **shrink-while-valid** (shortest window) — and the pattern is **fully closed** at 14/14 required + 4 extra (18 distinct total).
- Why a variable-size window's total runtime is O(n) despite a while-loop nested in a for-loop — the total-pointer-movement argument, not just the conclusion.
- The monotonic deque's domination argument (why a smaller-or-equal value can be permanently discarded from the back), and how to run two of them together when a window's validity depends on both its max and its min.
- How to derive "exactly K" from two calls to an "at most K" helper, and why summing `right-left+1` at each step correctly counts subarrays rather than just tracking a length.
- The HashMap bucket mechanism precisely enough to defend it — `hashCode()` → spread → `(n-1)&hash` → bucket, when and into what a bucket treeifies — and the difference between a merely-constant hashCode() (a performance bug) and an equals()/hashCode() contract violation (a correctness bug that causes silent `get()` failures).
- Type erasure precisely enough to explain why `new T[]` doesn't compile, and PECS applied correctly to a fresh wildcard example.
- `Comparable` vs. `Comparator` in one sentence, and why `TreeMap`/`PriorityQueue` are each O(log n) from their underlying structures (a balanced tree; an array-backed binary heap), not from memorized labels.
- The checked-vs-unchecked exception split, the full `Throwable` hierarchy, and the specific bug `try-with-resources` fixes over manual `try/finally` (a masked exception, not just "less boilerplate").
- The prefix-sum telescoping argument and Kadane's "extend or restart" recurrence, both proven from scratch, not just applied — **but** Prefix Sum & Kadane's is **not yet closed** (3/7 required); the "prefix sum + HashMap, track first-occurrence index" sub-variant has not been taught yet at all (see below).

Week 4 (`Week_04_Revised.md`) finishes Prefix Sum & Kadane's (4 more required problems, closing the pattern at 7/7) and opens an entirely fresh pattern, Greedy & Intervals, before transitioning to Binary Search. It assumes today's prefix-sum telescoping and Kadane's recurrence are both solid, and that sorting (Week 1, Day 3) and array fundamentals (Week 1, Day 2) — Greedy & Intervals' only stated prerequisites — are fully automatic. It does **not** yet assume the "prefix sum + HashMap, first-occurrence index" technique (needed for LC 525 and LC 523) — none of Week 3's three Prefix Sum problems used a HashMap, so this sub-variant needs fresh teaching in Week 4, Day 22. One thing *is* already in place for it, easy to miss: LC 560 (Subarray Sum Equals K), required at Week 4 Day 22, was already solved back in Week 1, Day 5, as extra practice — Week 4's generation should recap it from that prior solve rather than re-teach it from zero, while still formally introducing the technique itself for the first time.

## What Week 4 Leaves You Knowing Cold

From Day 28's self-check and Week 4 Consolidation — treat all of this as solid, not something Week 5 needs to re-establish:
- The first-occurrence-index vs. frequency-count fork in prefix-sum-plus-HashMap problems, chosen correctly without hesitation — and Prefix Sum & Kadane's is **fully closed** at 7/7 required + 2 extra (9 distinct total).
- Maximum Subarray Sum Circular's all-negative edge case — not just that a guard is needed, but *why* the naive `total - minKadane` formula produces an invalid empty-subarray result there.
- The formal definition of greedy, the exchange-argument proof template, and a concrete worked example of where greedy fails (0/1 Knapsack) — recognition of *when* greedy applies is as important as applying it.
- All three of this week's distinct exchange-argument shapes, named and matched to their problem without hesitation — domination (Jump Game), interval-swap (Non-overlapping Intervals), prefix-elimination (Gas Station) — and Greedy & Intervals is **fully closed** at 11/11 required + 2 extra (13 distinct total), opened and closed within the same week.
- The Intervals sort-and-sweep mechanism in both its forms — sort by start (combine) vs. sort by end (discard/cover) — including which each of this week's problems used and why.
- Why Meeting Rooms II's min-heap size, at the end of the scan, already equals the answer — the monotonic-growth argument, not a separately tracked maximum.
- The overflow-safe midpoint calculation (`left + (right-left)/2`), and the mechanical difference between an exact-match binary search (LC 704) and a boundary search on a monotonic condition (LC 278, "on the answer") — though Binary Search itself is **not yet closed** (2/11 required); Week 5 develops the "on the answer" variant in much greater depth.
- What a `record` generates, precisely, and why its `equals()`/`hashCode()` pair is contract-correct by construction; what a `sealed interface permits` clause guarantees; and the precise Java-21 requirement (JEP 441) for exhaustive switch pattern matching over a sealed hierarchy — a correction to the plan's own stated "Java 17+."

Week 5 (`Week_05_Revised.md`) continues Binary Search immediately and intensively (9 more required problems across Days 29–33, closing the pattern at 11/11 — deliberately *not* including Median of Two Sorted Arrays, LC 4, Hard, which stays reserved for right after Trees complete), then pivots to Linked Lists (growing to 12 problems) and initializes the `todo-api` Spring Boot project. It assumes today's exact-match template and overflow-safe midpoint calculation are fully automatic, since Week 5 varies the template repeatedly (rotated arrays, 2D matrices, boundary search, on-the-answer feasibility checks) without re-deriving the base mechanism each time. It also introduces Threads and the JVM's concurrency model (Day 29) as an entirely fresh theory thread, with no dependency on anything from Week 4. Greedy & Intervals and Prefix Sum & Kadane's are both fully closed and need no further teaching — only citation, if either resurfaces later in the series.

---

## What Week 5 Leaves You Knowing Cold

From Day 33's Binary Search Reviewed section and Day 35's Week 5 Consolidation — treat all of this as solid, not something Week 6 needs to re-establish:
- The full "on the input" vs. "on the answer" classification across all 15 distinct Binary Search problems (Weeks 4–5 combined), sorted correctly without hesitation — and Binary Search is **fully closed** at 11/11 required + 4 extra (15 distinct total).
- Why comparing to `nums[right]`, not `nums[left]`, is what makes rotated-array-minimum search unambiguous (the mid-can-equal-left, never-equal-right argument) — and the exact, reusable cost duplicates impose on every rotated-array variant (O(log n) average → O(n) worst case), demonstrated twice (LC 33→81, LC 153→154).
- Boundary search's exact narrowing rule for "first" vs. "last" occurrence, and why on-the-answer problems need an actively-constructed feasibility function rather than an array to search — recognizing the "minimum/maximum X such that Y is achievable" phrasing as the tell.
- A linked list node's `next` field as a heap reference (Day 9's mechanism, not a new one), the ArrayList-vs-LinkedList trade-off with its "still O(n) if you have to search first" nuance, and both reversal techniques (iterative O(1) space, recursive O(n) call-stack space) — though Linked Lists itself is **not yet closed** (4/11 required); Week 6 closes it with cycle detection, harder pointer manipulation, and LRU Cache.
- The dummy-head technique, and why it removes empty-result special-casing — reused once already this week (LC 92) and worth recognizing on sight going forward.
- The JVM's stack-per-thread/shared-heap model, the six real `Thread.State` values (not just the five-stage conceptual model), and the distinction between non-deterministic *ordering* (demonstrated this week) and actual data *corruption* (not yet demonstrated — reserved for Week 6, Day 37).
- HTTP verbs/status codes, REST's statelessness trade-off, and Spring's reflection-based startup routing — mechanically distinct from `@Override`'s compile-time-only role — plus `todo-api`, live with its first endpoint.

**⚠️ One correction worth flagging explicitly:** the paragraph above (written when Week 4 closed, before Week 5 existed) anticipated Linked Lists "growing to 12 problems," echoing `Week_05_Revised.md`'s own header. Once Week 5 was actually generated and cross-checked against `Week_06_Revised.md`, **11** turned out to be the correct total — confirmed by direct arithmetic (4 required in Week 5 + 7 required in Week 6 = 11) and by Week 5's own Day 35 scorecard text independently agreeing. Left uncorrected above per this document's own extend-don't-edit rule; corrected here, going forward.

Week 6 (`Week_06_Revised.md`) closes Linked Lists (7 more required problems across Days 36–39, closing the pattern at 11/11) before opening Stacks. It assumes today's fast/slow and reversal mechanics are fully reflexive, since Day 36's Linked List Cycle reuses fast/slow with no re-explanation, exactly as this week reused Day 34's without one. It assumes `todo-api` is live and reachable, since Day 36 adds Spring Data JPA and a real database-backed entity directly on top of what was initialized this week. It assumes Day 29's Threads/JVM concurrency model is solid, since Day 37 builds `ReentrantLock` and a concurrent `Counter` directly on it — the first time this series demonstrates actual data corruption from a race condition, not just non-deterministic ordering. **Two known overlaps need explicit handling when Week 6 is generated** — see the Week 6 Overlap section below: LC 20 (Valid Parentheses, required Week 1 Day 4, required again Week 6 Day 39) and LC 232 (Implement Queue using Stacks, extra practice Week 1 Day 4, required Week 6 Day 40), both flagged since Week 1 and carried forward through every intervening overlap check, only now reaching the week they actually matter for.

---

## What Week 6 Leaves You Knowing Cold

From Day 39's Linked Lists Capstone and Day 42's Week 6 Consolidation — treat all of this as solid, not something Week 7 needs to re-establish:
- Floyd's Cycle Detection in full, including the phase-2 distance-math proof (not the memorized trick), and its generalization beyond linked lists (Happy Number, Find the Duplicate Number) — Linked Lists is **fully closed** at 11/11 required + 4 extra (15 distinct total).
- Every Linked List technique this series built: fast/slow pointers (middle-finding *and* cycle detection — two different stopping conditions on the same mechanism), iterative/recursive reversal, dummy heads (now extended to dummy-head-and-tail for a doubly linked list), HashMap-based node mapping, and combining three techniques in one problem (Reorder List) — all reflexive, none needing re-derivation.
- `synchronized`'s intrinsic locks are already reentrant — not a distinguishing `ReentrantLock` feature, a flagged misconception if cited otherwise. `ReentrantLock`'s real value is `tryLock()`/timeouts, fairness, and `Condition` support. The lost-update interleaving trace (why `count++` isn't atomic) is this series' first demonstrated data *corruption*, not just reordering.
- `wait()`/`notify()`'s single-wait-set limitation, and why `Condition`'s multiple-wait-sets-per-lock fixes it — plus the `while`-not-`if` rule's two independent justifications (spurious wakeups; `notifyAll()` waking irrelevant threads).
- Stacks, formalized as LIFO (informally in use via `ArrayDeque` since Week 1), and Monotonic Stack as its own sub-pattern — the decreasing-invariant proof, the amortized O(n) argument, and all four shapes it's been applied to so far (plain array, circular array, streaming/stateful, and the mirror-image increasing variant) — though the pattern is **not yet closed** (7/12 required); Week 7 closes it with the Hard tier.
- `ConcurrentHashMap`'s bucket-level locking and CAS, contrasted precisely against `Collections.synchronizedMap()`'s single whole-map lock — a performance distinction between two equally-correct options, not a correctness one.
- SQL fundamentals: keys, normalization, the `LEFT JOIN`-fills-`NULL` vs. `INNER JOIN`-excludes distinction, the `NOT IN`/`NULL` trap, and index trade-offs — plus `todo-api`, now Flyway-managed with `ddl-auto: validate` replacing Day 36's convenience default.

Week 7 (`Week_07_Revised.md`) closes Stacks/Monotonic Stack at its Hard tier (Asteroid Collision, Basic Calculator II, Largest Rectangle in Histogram, Maximal Rectangle, Basic Calculator) before opening Trees. It assumes this week's Monotonic Stack invariant and amortized argument are fully reflexive, since none of Week 7's five remaining Stack problems re-derive either from scratch. It assumes Week 2's recursion fundamentals and this series' now-extensive comfort with self-referential node classes (Linked Lists, Weeks 5–6) are both solid, since Trees is "the same self-referential shape, with two children instead of one `next` pointer" from its very first problem. It assumes `todo-api` is Flyway-managed and JPA-backed, since Week 7 wraps it in Docker, Docker Compose, and a full testing stack (JUnit 5, AssertJ, Mockito, TestContainers) with no further schema changes of its own. **No known overlaps were found** checking Week 6's extra-practice picks (LC 202, 287, 901, 402) against Week 7's required list — see the Week 7 Overlap section below for the confirmation.

---

## What Week 7 Leaves You Knowing Cold

From Day 45's Stacks Capstone and Day 49's Week 7 Consolidation — treat all of this as solid, not something Week 8 needs to re-establish:
- Stacks/Monotonic Stack is **fully closed** at 12/12 required + 2 extra (14 distinct total), with the full two-family classification (general-purpose LIFO vs. true monotonic invariant) reflexive — including the Hard tier's three calculator-family problems and the index-based monotonic stack applied to Largest Rectangle in Histogram / Maximal Rectangle.
- The `TreeNode` shape, as Linked Lists' `Node` with two self-references instead of one; the precise edge-based depth/height definitions *and* LeetCode's node-counting "Maximum Depth" convention, named as two distinct, non-interchangeable base cases; all four traversal orders (preorder, inorder, postorder, level-order); the universal recursive template (base case for `null`, combine `left`/`right`); and O(h) — not a defaulted O(n) — as the precise space-complexity statement for a DFS tree recursion.
- Every DFS shape this series has posed so far: single-tree combination (Maximum Depth, Invert), two-tree comparison including mirrored comparison (Same Tree, Symmetric Tree, and composing `isSameTree` as a subroutine for Subtree of Another Tree), sentinel short-circuiting (Balanced Binary Tree), and a running best tracked separately from the value returned upward, with the Java-specific reasoning for why an instance field or mutable holder is required rather than a plain local variable (Diameter of Binary Tree) — plus the one-child-is-not-a-leaf trap that breaks a naively-adapted Maximum Depth formula (Minimum Depth).
- BFS on trees: the `queue.size()`-captured-before-the-inner-loop scaffold that isolates exactly one level per pass, and its three applications so far (collect a level, keep the last node per level, aggregate a level) — plus the DFS-with-level-tracking alternative, and the skewed-tree case where BFS's space bound actually beats DFS's, reversing the usual intuition.
- `todo-api` is now Docker/Compose-managed (multi-stage build, app + Postgres + Redis wired by service name) with a complete testing pyramid in place: Mockito-based unit tests against fully faked dependencies, and `@DataJpaTest` integration tests running against a real, disposable TestContainers Postgres rather than H2 — plus a working Kafka producer publishing `TaskCreatedEvent`.
- Kafka's partition-level (not topic-wide) ordering guarantee, and the sequential-disk-append mechanism behind its throughput.

Week 8 (`Week_08_Revised.md`) closes Trees at 15 required problems — Lowest Common Ancestor (both the BST-ordering-exploiting variant and the general two-child-DFS variant), Construct Binary Tree from Preorder/Inorder, Serialize/Deserialize Binary Tree, and finally Median of Two Sorted Arrays, fixing the gap the original plan's Day 21 left open — before opening Heaps. It assumes this week's `TreeNode` shape and universal recursive template are fully reflexive, since Binary Search Trees (Day 50) layer a new ordering invariant on top of the plain-tree shape rather than re-deriving the shape itself, and Serialize/Deserialize (Day 52) directly extends the preorder-with-explicit-null-markers idea Day 47's Same Tree brute force already introduced, without re-deriving it. It assumes this week's Kafka producer and full Docker/Compose/testing setup are stable, since Week 8 adds a Kafka *consumer* (`@KafkaListener`) and Spring Cloud Config on top, with no further changes to Docker, Compose, or the testing stack itself. **No known overlaps were found** checking Week 7's extra-practice picks (LC 572, 111, 637) against Week 8's required list — see the Week 8 Overlap section below for the confirmation.

---

## What Week 8 Leaves You Knowing Cold

From Day 53's Trees Capstone and Day 56's Week 8 Consolidation — treat all of this as solid, not something Week 9 needs to re-establish:
- Trees is **fully closed** at 15/15 required + 5 extra (20 distinct total), with the BST-ordering-invariant-vs-general-tree distinction for Lowest Common Ancestor fully reflexive — the ordering invariant is what lets one computed direction replace searching both children, not a difference in algorithmic sophistication.
- Divide-and-conquer, formalized: the three named steps, and specifically what was new about Construct Binary Tree's divide step (computed, not free) versus every earlier tree DFS problem's free left/right split.
- The Median of Two Sorted Arrays partition proof — both correctness conditions (size, value), why the search runs over the smaller array specifically, and the full worked trace.
- The heap mechanism end to end: array index math, sift-up, sift-down, and the unconditional-vs-conditional O(log n) contrast with a bare BST from Day 50 — assumed completely solid, not re-derivable from a `PriorityQueue` API call alone.
- Quickselect's partition step (the same lineage as Day 12's Dutch National Flag), its O(n)-average-vs-O(n log n) distinction from quicksort, and its worst case plus the randomized-pivot mitigation.
- Every heap-of-size-k role played so far: min-heap for "kth largest" (Days 54–55), max-heap for "k closest"/"k largest values to keep" (Day 54's Last Stone Weight, Day 55's K Closest Points), min-heap for "minimize combination cost" (Day 55's Minimum Cost to Connect Sticks), and a heap used as a **candidate generator** rather than a fixed-collection query (Day 56's Ugly Number II) — four genuinely different roles the same structure plays, not one pattern with cosmetic variations.
- `todo-api` now has a complete Kafka produce-consume loop, environment-specific configuration via Spring profiles, and a WireMock stub in place — and is explicitly **done** receiving new features from Day 62 onward, per Week 9's own plan.

Week 9 (`Week_09_Revised.md`) closes Heaps at 10 required problems (Reorganize String, Task Scheduler, Find Median from Data Stream, Merge k Sorted Lists — Days 57–58), closes Tries' core 6 problems, opens Backtracking, and initializes `scalable-ecommerce-platform`, five weeks earlier than the original plan had it. It assumes this week's full heap mechanism is completely reflexive, since none of Week 9's four remaining Heap problems re-derive it from scratch. It assumes recursion (Day 8) is solid enough to extend into two structurally new shapes — Tries (each node holding up to 26 children, not 2) and Backtracking (explore-recurse-undo, with no canonical "Easy" problem to ease into). It assumes `todo-api`'s Docker, Compose, and full testing stack are stable and untouched, since Week 9 adds no further infrastructure changes there at all before active project work moves entirely to the new flagship platform. **No known overlaps were found** checking Week 8's extra-practice picks (LC 173, 106, 1167, 378, 451) against Week 9's required list — see the Week 9 Overlap section below for the confirmation.

---

## What Week 9 Leaves You Knowing Cold

From Day 58's Heaps closing note, Day 61's Tries closing note, and Day 63's Week 9 Consolidation — treat all of this as solid, not something Week 10 needs to re-establish:
- Heaps is **fully closed** at 10/10 required + 3 extra (13 distinct total), with all six distinct roles reflexive: kth-largest query, keep-the-k-best, minimize-combination-cost, candidate-generator, greedy-scheduling (hold-out-one-round / cooldown-queue), and two-heap-balance / k-way-merge-coordination.
- The two-heap balance invariant end to end: push-unconditionally-then-shuffle-then-rebalance-if-needed, and both `findMedian()` cases derived from the size relationship, not memorized as separate rules.
- Tries' full toolkit, **fully closed** at 6/6 core + 0 extra: `insert`/`search`/`startsWith`, delta-propagated cumulative values, constrained descent (only into `isEndOfWord` children), branching descent (wildcard, with the precise O(26^k×(m-k)) worst case, not the wildcard-free-only "O(m)" shorthand), dictionary-root substitution, and Trie-pruned matrix backtracking with overwrite-based visited marking.
- Backtracking's choose/explore/un-choose template, and — critically — that it is **not** one shape: include/exclude (Subsets), swap-based/position-dependent (Permutations), and forward-index/never-revisit (Combinations) are three structurally different mechanisms under one template, each chosen for a different reason (independent per-element choice vs. position-dependent choice vs. order-doesn't-matter), not interchangeable defaults.
- The Permutations II duplicate-skip condition (`!used[i-1]`, proven via the full `[1,1,2]` trace, not just stated) — assumed fully reflexive, since Week 10's Subsets II and Combination Sum II both reuse the identical skip-logic shape without re-deriving it.
- CAP Theorem, Consistent Hashing, and Replication Models as one connected three-day arc — the same underlying consistency-vs-availability trade-off viewed from three angles (a formal theorem, a specific resharding mechanism, and a write-propagation strategy) — not three unrelated topics.
- Spring AOP's proxy mechanism and the self-invocation limitation specifically, since Week 10's Resilience4j and Feign work both build advice-bearing code inside modules that will need this same proxy-boundary awareness.
- `scalable-ecommerce-platform`'s four-module skeleton (Product/Order/Payment/Notification), the parent/child POM structure, and the `<dependencyManagement>`-vs-`<dependencies>` distinction — assumed stable, since Week 10 builds functionality *inside* this structure (Resilience4j in Order, Feign clients calling Product, a Gateway module) rather than re-architecting it. `todo-api` remains frozen, feature-complete, receiving no further changes.

Week 10 (`Week_10_Revised.md`) continues Backtracking to closure at 12/12 required (Combination Sum, Combination Sum II, Letter Combinations of a Phone Number, Generate Parentheses, Word Search, Palindrome Partitioning, Subsets II, N-Queens — Days 64–67), then opens Graphs — genuinely new anywhere in this series, the first pattern requiring an explicit `visited` set since, unlike every tree structure taught so far, a graph can contain cycles. It assumes today's forward-index and duplicate-skip mechanisms transfer directly (Combination Sum reuses forward-index recursion with exactly one new wrinkle — elements may repeat — rather than re-teaching the mechanism; Subsets II reuses the exact `!used[i-1]`-style skip logic from Permutations II, adapted to include/exclude rather than swap-based choice). **One near-collision was caught and avoided, not silently absorbed:** Letter Combinations of a Phone Number (LC 17) was seriously considered as Week 9 extra practice and confirmed, against `Week_10_Revised.md` itself, to already be Week 10 Day 65's required problem — see the Week 9 Overlap section immediately below for the full account, and the Week 10 Overlap section (to be added when Week 10 is generated) for confirmation it wasn't duplicated there either.

## What Week 10 Leaves You Knowing Cold

From Day 67's Backtracking closing note (full 12-problem, 6-choice-model classification table), Day 68's Graphs Concept Card, and Day 70's Week 10 Consolidation — treat all of this as solid, not something Week 11 needs to re-establish:
- Backtracking is **fully closed** at 12/12 required + 0 extra (12 distinct total, Weeks 9–10), with all six distinct choice models reflexive: include/exclude, swap-based, forward-index (bare, with-repetition, with-duplicate-skip), string-building (external-lookup and counter-gated variants), grid-DFS with overwrite-restore marking, and row-by-row placement with derived O(1) constraint tracking.
- The `i > start` duplicate-skip condition (Combination Sum II, proven via a `[1,1,2]`/target-4 trace against the `i > 0` bug) — assumed fully reflexive, since it's the exact mechanism Subsets II cited rather than re-derived, and the same category of reasoning (which choice model does a given duplicate-skip translate into) may resurface again.
- Graphs' foundational Concept Card in full: adjacency list vs. matrix trade-offs, "a grid is a graph in disguise," the BFS-vs-shortest-path / DFS-vs-reachability interview signal, and — most load-bearing — **why `visited` is mandatory** (a tree cannot cycle, a graph can) **and that the rule is conditional, not absolute** (Day 70's All Paths From Source to Target correctly has none, given a guaranteed DAG).
- The graph-DFS-vs-backtracking classification test (is there a shared mutable structure needing an un-choose step, or is marking permanent with nothing to restore) — proven on two visually similar recursive traversals this week (Number of Islands: no; All Paths From Source to Target: yes) reaching opposite, individually-justified answers. Treat this as a per-problem question, not a memorized default in either direction.
- Multi-source BFS (seed the queue with every starting node at once; Day 49's level-isolation trick otherwise unchanged) — assumed fully reflexive, since Week 11's Word Ladder and the threshold-distance shortest-path problem both sit in the same BFS-on-an-implicit-or-weighted-adjacency family.
- The full resilience/gateway/messaging arc built into `scalable-ecommerce-platform` this week — Resilience4j circuit breakers, Feign clients, Spring Cloud Gateway routing, JWT validation, Token Bucket rate limiting, Kafka Schema Registry, and Saga choreography (including its compensating-transaction path, exercised end to end, not just its happy path) — assumed stable and live, since Week 11 builds the next layer of distributed-systems work directly on top of this foundation rather than revisiting it from scratch.

Week 11 (`Week_11_Revised.md`) continues Graphs to closure at 12/12 required (Course Schedule, Course Schedule II, Surrounded Regions, Pacific Atlantic Water Flow, Word Ladder, and a threshold-distance shortest-path problem), then opens Union-Find as a new structure entirely. It assumes this week's Concept Card is fully solid — particularly the conditional nature of the `visited` rule, since Course Schedule's cycle detection in a *directed* graph is a direct escalation of exactly the reasoning Day 69's Find Eventual Safe States extension previewed (3-state coloring, unvisited/visiting/safe, generalizing Is Graph Bipartite's 2-coloring). **Two extra-practice picks were checked against `Week_11_Revised.md` before being added, both confirmed clean:** LC 802 (Find Eventual Safe States, Day 69) and LC 542 (01 Matrix, Day 70) — neither appears anywhere in Week 11's required list. See the Week 10 Overlap section immediately below for the full confirmation, including every required Week 10 problem checked against the cumulative table above before being taught.

**One small correction, worth recording here rather than silently letting it stand:** the paragraph above's own note two paragraphs up (in the Week 10 section, "Multi-source BFS... assumed fully reflexive, since Week 11's Word Ladder and the threshold-distance shortest-path problem both sit in the same BFS-on-an-implicit-or-weighted-adjacency family") turned out to be imprecise once Week 11 was actually taught. Word Ladder is BFS on an implicit graph, but it is **single-source**, not multi-source — there is exactly one `beginWord`. The threshold-distance problem (Find the City With the Smallest Number of Neighbors at a Threshold Distance) is **not BFS at all** — it's Floyd-Warshall, a non-traversal, all-pairs algorithm family, a distinction Day 73's own Theory Block makes explicitly. The historical sentence above is left as originally written, per this document's extend-don't-edit convention for its dependency-map and knowing-cold narrative; this note exists so the imprecision doesn't propagate into any future week's assumptions.

## What Week 11 Leaves You Knowing Cold

From Day 73's Theory Block (the full BFS-vs-DFS-vs-neither classification), Day 74's Union-Find Concept Card, and Day 77's Week 11 Consolidation — treat all of this as solid, not something Week 12 needs to re-establish:
- Graphs BFS/DFS is **fully closed** at 12/12 required + 2 extra (14 distinct total, Weeks 10–11), with four genuinely distinct final roles reflexive: topological ordering/cycle-detection (Kahn's and DFS 3-state coloring), multi-source reachability (forward and reversed), single-source BFS on an implicit graph, and Floyd-Warshall as a bounded, deliberate exposure to a non-traversal shortest-path family.
- Union-Find, in full: naive O(n) → union-by-rank-alone O(log n) (proven via the rank-doubling induction) → combined with path compression O(α(n)) amortized — assumed fully reflexive, since Day 77's Kruskal's Algorithm calls the `UnionFind` class as a live subroutine without re-deriving or re-justifying any of it.
- The "union the abstraction, not the raw input" instinct (rows/columns, equation variables, index positions, emails — Days 75–76) — treat this as the default first question for any new Union-Find problem going forward: *what* should actually be getting unioned, not just *whether* Union-Find applies.
- The Cut Property and its exchange-argument proof (Day 77) — assumed fully reflexive, since it's the correctness justification for both Kruskal's and Prim's, and the same exchange-argument *style* (not the same property) may resurface for Dijkstra's greedy correctness next week.
- The distinction between multi-source *reachability* (Day 72, DFS or BFS interchangeable) and multi-source *distance* (Day 70, BFS required for its level-order guarantee) — this is now a settled, two-way-tested distinction, not a one-off note.
- Networking, in full: TCP/UDP/DNS (Day 71), HTTP/HTTPS and L4-vs-L7 load balancing (Day 72), AWS VPC/subnets/IAM/S3/EBS (Days 74–75), and observability end to end — Micrometer instrumentation through `/actuator/prometheus` through Prometheus scraping through Grafana visualization (Days 76–77) — a complete, working pipeline on `scalable-ecommerce-platform`, assumed live and stable, the same treatment Week 10's resilience/gateway/messaging arc received going into this week.

Week 12 (`Week_12_Revised.md`) closes Dijkstra's Algorithm (opened fresh, reusing the `PriorityQueue` mechanics from Days 17/26/54 and Day 77's Prim's directly) and opens Dynamic Programming — the single largest pattern in the entire plan. It assumes today's Cut Property exchange-argument style transfers by analogy to Dijkstra's own greedy-correctness argument (a different property, the same proof shape), and that the informal DP-flavored reasoning previewed twice without the formal name (Week 3's Kadane's, and this week's own Floyd-Warshall, Day 73) is available to cite directly rather than re-introduced from zero. **One extra-practice pick was checked against `Week_12_Revised.md` before being added, confirmed clean:** LC 1319 (Number of Operations to Make Network Connected, Day 75) — it does not appear anywhere in Week 12's required list. See the Week 11 Overlap section immediately below for the full confirmation, including every required Week 11 problem checked against the cumulative table above before being taught, and the flagged 150-vs-151 plan-scorecard discrepancy.

## What Week 12 Leaves You Knowing Cold

From Day 80's Dijkstra's closing note, Day 81's Dynamic Programming Concept Card, and Day 84's Week 12 Consolidation — treat all of this as solid, not something Week 13 needs to re-establish:
- Dijkstra's Algorithm is **fully closed** at 5/5 required, 0 extra, 5 distinct total, spanning every relaxation shape the pattern tests in interviews: sum-minimizing, multiplicative/max-heap, hop-constrained (Bellman-Ford), and minimax (both edge- and node-weighted). The greedy-correctness domination proof, and precisely where it depends on non-negative weights, is assumed fully reflexive.
- Bellman-Ford, in full: K-round relaxation against a frozen snapshot, the induction proof for why each round bounds hops by exactly one more edge, and why relaxing in place instead breaks that guarantee — assumed solid, not re-derivable from scratch if it resurfaces.
- Dynamic Programming's foundational discipline — state `dp[i]` precisely before writing any recurrence, then prove the recurrence via an exhaustive, disjoint-case argument — is the actual through-line of everything taught this week, not any single recurrence. Memoization vs. tabulation's honest trade-offs (laziness vs. computing every state; recursion/stack-overflow risk vs. iteration; harder vs. easier space optimization) are assumed known, not re-explained.
- The 1D DP lookback family, in its full range so far: always-sum (counting), max-based take-or-skip (optimizing), validity-gated sum (a legality check per term), and dictionary/Set-gated OR across every valid split point (Word Break — the first time the set of relevant earlier states isn't a fixed distance back at all). Treat "what varies between these recurrences" as the transferable skill, not each one memorized independently.
- Unbounded Knapsack has opened (Coin Change, Day 84) — unlimited reuse of a single item, falling directly out of not tracking which items built a sub-answer. 0/1 Knapsack (bounded, each item at most once) has **not** been introduced yet and is a genuinely distinct counterpart Week 13 will need to build fresh, not derive from this week's unbounded framing by analogy alone.
- The overlap-checking discipline caught its first **required**-problem collision this week (Maximum Product Subarray, Day 83, already solved as Week 4 extra practice) — the mirror image of every prior catch, which ran the other direction (an extra colliding with a later week's required list). Both directions of the check are now proven to matter in practice, not just in principle.

Week 13 (`Week_13_Revised.md`, leave week 2) closes four DP subtypes entirely across seven days at full-time intensity: the rest of 1D DP (Coin Change II, Longest Increasing Subsequence, Partition Equal Subset Sum, Target Sum, Perfect Squares, Word Break II), all of Grid DP, all of String DP, and all of Interval DP. It assumes today's unbounded-knapsack mechanism transfers directly to Coin Change II (Day 85) — with the loop-order distinction flagged today (minimization is order-independent; counting combinations is not) actually resolved this time, not just noted — and that Word Break's `dp[]` boolean array (Day 84) is reused directly as a pruning structure before Word Break II (Day 86) backtracks to build real output. It will need to formally introduce 0/1 Knapsack (Partition Equal Subset Sum, Target Sum) as a distinct counterpart to this week's Unbounded Knapsack — flagged here as coming, not built. **No extra-practice picks from Week 12 required checking against `Week_13_Revised.md`, since Week 12 added exactly one extra (Delete and Earn, LC 740) and it was already checked and confirmed clean during Week 12's own generation** — see the Week 12 Overlap section below for that confirmation, plus the required-problem overlap (LC 152) this week caught in the other direction.

## What Week 13 Leaves You Knowing Cold

From Day 85's 0/1 Knapsack Concept Card, Day 87's Grid DP Concept Card, Day 88's String DP Concept Card, Day 91's Interval DP Concept Card, and Day 91's Week 13 Consolidation — treat all of this as solid, not something Week 14 needs to re-establish:
- **1D DP is fully closed at 14/14 required** (16 distinct including the two extras), spanning six lookback shapes total across Weeks 12–13: always-sum, max-based take-or-skip, validity-gated sum, dictionary/Set-gated OR, value-gated OR across every earlier index (LIS), and DP-gated Backtracking (Word Break II). The complete four-rule loop-order table (Coin Change / Coin Change II / Combination Sum IV / 0/1 Knapsack) is assumed reflexive, including which of two independent reasons — bounded-vs-unbounded reuse, or combinations-vs-permutations counting — drives each rule.
- **0/1 Knapsack, in full**: built from zero (Day 85), the recurrence reading only from the previous item's row, and the 1D space-optimization's decreasing-capacity requirement — proven via a minimal counterexample (`nums=[3]`, target `6`), not just stated. Assumed as reflexive a mechanism as Unbounded Knapsack was going into this week.
- **Grid DP is fully closed at 4/4 required** (5 distinct including Dungeon Game). The "which neighbors, combined how" question (sum for counting, min for optimizing, three-way min for square-growth) is the transferable skill; Maximal Square's two-directional proof is assumed defensible from memory, not just its formula. Declaring/populating a 2D array from scratch (`new int[m][n]`) is now established syntax, distinguished from indexing into a given matrix (Week 5, Day 32).
- **String DP is fully closed at 9/9 required** (12 distinct including three extras), the week's largest subtype. The `(m+1)×(n+1)` table convention over 0-indexed strings, the "map every operation/case to exactly which neighboring cell it reads from" discipline (used for Edit Distance's three operations, Interleaving String's two sources, both pattern-matching Hards' four-plus cases), and the fully-proven `LPS(s) = LCS(s, reverse(s))` equivalence are all assumed reflexive. The Wildcard-vs-Regex `*` semantic distinction (standalone-symbol vs. modifier-on-preceding-character) is assumed instant, not re-derivable under pressure.
- **Interval DP is fully closed at 2/2 required** (3 distinct including Predict the Winner), the pattern's thinnest ladder by design, deliberately reinforced with a second, structurally distinct recurrence shape (choose-from-either-end, alongside Burst Balloons' choose-the-last-operation). The length-ordered traversal requirement — the first genuinely new 2D-DP fill-order rule since Grid DP's row-major sweep — is assumed solid, including a concrete case for why row-major order would fail here.
- **DP array as a backtracking pruning oracle** was used twice this week in different guises (Word Break II's dictionary-reachability gate, Day 86; Palindrome Partitioning II's palindrome-validity gate, Day 90) — this is now a recognizable technique in its own right, not two unrelated one-off tricks, worth naming on sight if a similar shape resurfaces.
- **Dynamic Programming overall stands at 28/35 required** (7 through Week 12 + 21 this week), with State Machine DP (4) and Tree DP (2) — 6 problems — the only remainder, confirmed directly against `Week_14_Revised.md`'s own stated 35-problem total.

Week 14 (`Week_14_Revised.md`) closes Dynamic Programming entirely: State Machine DP (Days 92–93, formalizing explicit `dp[i][state]` tracking for the first time — a generalization this week's 0/1 Knapsack and Predict the Winner both touched informally without ever building the state-machine framing itself, so it is introduced fresh, not assumed) and Tree DP (Day 94, opening with House Robber III, which extends House Robber's take-or-skip recurrence — Week 12, Day 82's disjoint-exhaustive-cases proof — onto a tree structure directly, assumed solid rather than re-derived), before moving into Bit Manipulation. **All six of Week 13's extra-practice picks were checked against `Week_14_Revised.md`'s required list before being added, confirmed clean:** LC 377, 673, 174, 583, 132, and 486 — none appear anywhere in Week 14's fourteen required problems (LC 309, 714, 123, 188, 337, 124, plus the eight Bit Manipulation problems beginning Day 95: LC 136, 191, 231, 338, 268, 137, 260, 371). See the Week 14 Overlap section below for the full confirmation, including every one of Week 14's own required problems checked against the cumulative table above before being taught, and the flagged 184-vs-185 plan-scorecard discrepancy.

## What Week 14 Leaves You Knowing Cold

From Day 92's and 93's State Machine DP material, Day 94's Tree DP material and DP-wide synthesis, Day 95's bit-manipulation foundations, and Day 98's Week 14 Consolidation — treat all of this as solid, not something Week 15 needs to re-establish:
- **State Machine DP is fully closed at 4/4 required, 0 extra.** `dp[i][state]` (a situation dimension) and `dp[i][k][state]` (a budget dimension, advancing only on commitment) are assumed as two distinct, nameable kinds of DP dimension, not variations on one idea. The `k ≥ n/2` reduction back to an unconstrained earlier problem (LC 122) is assumed reflexive as a general pattern — "does this constraint's own parameter range ever stop binding" — not just as this one specific bound.
- **Tree DP is fully closed at 2/2 required + 1 extra (3 distinct).** The precise distinction between "a return value that IS the DP state" (House Robber III's pair) and "a return value that merely supports an outside-tracked answer" (Max Path Sum, and Diameter of Binary Tree before it, Week 7 Day 48) is assumed instantly producible, not something to re-derive under pressure — two required problems in the same day landed on opposite sides of it on purpose.
- **Dynamic Programming overall is now fully closed, 35/35 required across Weeks 12–14** (34 newly taught + 1 recap, LC 152 — both figures reconciled explicitly, Day 94). The six-subtype synthesis (1D, Grid, String, Interval, State Machine, Tree — see Day 94's table) and its one shared discipline — state `dp[...]` precisely before writing any recurrence, then prove it by exhaustive disjoint cases — are assumed fully internalized. Nothing further from Dynamic Programming is expected to need reinforcement going forward; Week 15 does not touch it.
- **Bit Manipulation's foundations are assumed fully reflexive, not just "seen once":** binary place value; two's complement, including the derived (not stated) negation formula `-x=~x+1`; all seven operators, with `>>` vs `>>>` provably identical for non-negative operands and divergent only when sign-extending a `1`; XOR's four algebraic properties, proven by truth table; `n&(n-1)` (clears the lowest set bit) and `n&(-n)` (isolates it), both derived from binary subtraction/negation mechanics; and per-bit frequency counting as XOR's mod-2 cancellation generalized to mod-`k`, motivated by a concrete counterexample (`2^2^2=2`) rather than asserted as a rule.
- **Bit Manipulation stands at 8/10 required + 1 extra** (Hamming Distance, Day 97) — 9 distinct. Two required problems remain (Reverse Bits, Maximum XOR of Two Numbers in an Array), both deferred to Week 15.

Week 15 (`Week_15_Revised.md`) closes Bit Manipulation at Day 99 with Reverse Bits and Maximum XOR of Two Numbers in an Array — the latter combining this week's bit mechanics **and** Week 9's Trie structure into a Bit Trie, a two-prerequisite-chain dependency worth being explicit about: Week 9's Trie mechanism needs to be exactly as reflexive as this week's bit identities, not assumed solid by default just because it's older material. Week 9's own closing note already flagged LC 421 as deliberately deferred for exactly this pairing, so nothing about this dependency is new information — it's simply now due. **Both of Week 14's extra-practice picks were checked against `Week_15_Revised.md`'s required list before being added, confirmed clean:** LC 968 (Binary Tree Cameras) and LC 461 (Hamming Distance) — neither appears anywhere in Week 15's required list (LC 190, 421, 307, 315, 215, plus ten SQL problems, Days 103–104). See the Week 15 Overlap section below for the full confirmation, including every one of Week 14's own required problems checked against the cumulative table above before being taught.

## What Week 15 Leaves You Knowing Cold

From Day 99's Bit Trie and Kubernetes material, Day 100–101's Segment Tree and Creational Patterns material, Day 102's sorting/Quickselect material, Days 103–104's SQL material, and Day 105's Week 15 Consolidation — treat all of this as solid, not something Week 16 needs to re-establish:

- **Bit Manipulation is fully closed at 10/10 required + 1 extra (11 distinct).** No further reinforcement is expected; if the pattern resurfaces later in the series, treat it as a short recap plus new problems, not a full re-teach, per this map's own standing convention.
- **Tries is fully closed at 7/7 required, 0 extra (7 distinct)** — its Week 9 core plus the Week 15 Bit Trie fusion. The Bit Trie's specific mechanism (children indexed by bit, not letter; the greedy opposite-bit walk; the place-value-dominance proof of its optimality) is assumed reflexive, not something to re-derive if a future bit-manipulation-adjacent problem needs it again.
- **Segment Trees are fully closed at 2/2 required, 0 extra (2 distinct) — the pattern's full, deliberately-light scope, not expected to expand further.** The build/update/query mechanism, the O(log n) query proof, and the position-indexed-vs-value-indexed distinction are assumed solid background if a future pattern needs range-query-with-updates again. The Fenwick Tree extension (and its direct connection to `n&(-n)`) is assumed known by name and mechanism, not necessarily by heart.
- **Sorting fundamentals are assumed genuinely internalized, not just "the two algorithms were shown once":** merge sort's guaranteed-every-case O(n log n) and stability mechanism; quicksort's average/worst-case split and why randomized pivots mitigate it; the dual-pivot-quicksort-vs-TimSort split in Java's real `Arrays.sort()`, with the actual mechanism defensible, not just the two names. Quickselect's O(n)-average argument, and specifically *why* its recurrence differs from quicksort's rather than merely having a smaller constant, is assumed reflexive.
- **All three Creational design patterns are fully taught and implemented — Singleton (both DCL-with-`volatile` and Enum variants), Factory Method (Simple Factory vs. true GoF, precisely distinguished), and Builder (immutability tied to thread-safety, not just fluency).** `volatile` and its specific reordering-bug justification are assumed solid background for any future concurrency material. Week 16 builds directly on top of these three — Structural patterns (Adapter, Decorator, Facade, Proxy, Composite) and Behavioral patterns are taught assuming "private constructor plus static accessor" and "delegate creation to a subclass" are already fluent, not re-derived.
- **The SQL practice track is fully closed at 10/10 required, 0 extra — subqueries, correlated subqueries, `GROUP BY`/`HAVING`, two self-join shapes, and the full window-function toolkit (`ROW_NUMBER`/`RANK`/`DENSE_RANK`, `LAG`/`LEAD`, `PARTITION BY`, `CASE`/conditional aggregation), all assumed durable background.** The SQL logical processing order (`FROM→WHERE→GROUP BY→HAVING→SELECT→ORDER BY→LIMIT`, extended by window functions' own evaluation position) is assumed reflexive if any future system-design or data-modeling context needs it.
- **Kubernetes fundamentals — Pod, ReplicaSet, Deployment, Service, the declarative reconcile-loop model, and the Horizontal Pod Autoscaler as a live instance of that same loop — are assumed solid**, taught as a flagged, necessary prerequisite insertion rather than assumed background the way the original plan's Day 99 treated it.
- **The entire DSA phase of this series is closed as of Day 105 — 197 required + 53 extra = 249 distinct DSA problems across Weeks 1–15, plus the 10-problem SQL track, 259 distinct problems in total.** Every pattern from HashMap/HashSet through Sorting is assumed fully reflexive on recall, not something Week 16's LLD systems or later mock interviews should ever need to re-teach from scratch — only cite. A numeric reconciliation was flagged (this map's row-by-row 197 vs. `Week_15_Revised.md`'s own stated 203 required-only DSA figure) and resolved in this map's favor, consistent with this document's established handling of the same class of drift.

Week 16 (`Week_16_Revised.md`) opens Structural design patterns (Adapter, Decorator, Facade, Proxy, Composite) directly on top of Week 15's Creational foundation, and its DSA-adjacent content is exclusively **revision** of already-closed patterns (Sliding Window, Backtracking, Trees, Union-Find/Dijkstra's, Graphs/DP) — no new DSA patterns, no new required LeetCode problems in Bit Manipulation, Tries, Segment Trees, Sorting, or SQL. **Every one of Week 15's extra-practice picks (there were none — see the Week 15 total above) and every Week 15 required problem were checked against `Week_16_Revised.md`'s own content before this map was finalized; no collision in either direction.** See the Week 16 Overlap section below for the full confirmation.

## What Week 16 Leaves You Knowing Cold

From Day 106's LLD framework and Structural patterns, Day 107's Behavioral patterns and TDD, Day 108's live framework run and Coupling/Cohesion/Law of Demeter, Day 109's State pattern and State-vs-Strategy theory, Day 110's Composition over Inheritance, Day 111's concurrency work, and Day 112's Library Management System and Week 16 Consolidation — treat all of this as solid, not something Week 17 needs to re-establish:

- **The 5-step LLD interview framework (clarify requirements → identify core objects → define relationships/class diagram → apply patterns deliberately → code the core) is assumed fully automatic** — introduced Day 106, then run live four separate times (Tic-Tac-Toe, Vending Machine, Parking Lot, Library Management) without any of those four runs re-explaining what a step is for. Week 17's six remaining systems assume the same.
- **All ten Structural and Behavioral patterns are fully taught, with working implementations for the ones the plan named as coding exercises** (Decorator — Coffee, full JUnit-tested implementation; Observer — WeatherStation/Display, built test-first; State — the Vending Machine's four concrete states). The other seven (Adapter, Facade, Proxy, Composite, Strategy, Command, Template Method) have solid conceptual depth plus illustrative code, not full graded projects — a deliberate, stated pacing choice, not a depth gap. GoF's 3-category taxonomy (Creational/Structural/Behavioral) is assumed known by name and by the concrete "what problem does this category solve" test.
- **The judgment to recognize when *no* pattern is warranted is assumed as solid as the judgment to apply one correctly.** Tic-Tac-Toe (Day 108) and Parking Lot (Day 110) both correctly required none; Singleton was actively considered for `ParkingLot` and explicitly declined, with reasoning — three independent, differently-reasoned data points, not a single lucky case.
- **State vs. Strategy is assumed genuinely distinguishable on a new example, not just recitable as two sentences** — the code-shape similarity, the who-drives-the-swap distinction, and the parameterless-singleton-vs-configured-instance practical tell are all assumed reflexive. Day 112's `BookItem` transition table adds a third contrast point: a data-driven validity rule is not automatically a State pattern just because something has named phases.
- **TDD's Red-Green-Refactor cycle is assumed internalized as a workflow, not just a vocabulary term** — why Red comes before any implementation, why Green stays deliberately minimal, and why Refactor's safety net specifically comes from the already-passing test, not from care alone.
- **Coupling, Cohesion, and the Law of Demeter are assumed applicable as a live code-review lens, not just definitions** — including the Law of Demeter's own necessary nuance (conventionally relaxed for simple, immutable data-holder objects), which is assumed understood as a genuine exception with a stated reason, not a loophole.
- **Composition over Inheritance's actual tell — value along a natural order (field) vs. genuinely distinct behavior (subclass/pattern) — is assumed applicable to a new design, not just recognizable in the Parking Lot example it was built against.** The concrete Open/Closed cost of the inheritance alternative (an existing class needing edits when a new variant is added) is assumed provable on a fresh example, not just remembered as an assertion.
- **Concurrency reasoning is assumed transferable to a different system, not anchored to Parking Lot specifically** — identifying a check-then-act race, choosing a locking granularity and justifying the choice by what's actually shared, and knowing precisely when `volatile` is redundant versus load-bearing, all assumed reflexive. Mock Interview #2's live "now make it thread-safe" escalation is the actual evidence this held, not just a feeling of familiarity.
- **Four complete LLD systems exist as working, tested reference implementations** (Tic-Tac-Toe, Vending Machine, Parking Lot single-threaded and concurrent, Library Management) in `lld-java`, alongside `dsa-java`'s DSA work from Weeks 1–15 — cite these by system name going forward rather than re-deriving their designs.
- **Six DSA revision problems were solved cold this week — Course Schedule, Coin Change, Combination Sum, Longest Substring Without Repeating Characters, Validate BST, Redundant Connection** — confirming Graph, DP, Backtracking, Sliding Window, Tree, and Union-Find are all still genuinely reflexive, not just previously covered. See the DSA Revision Log above for exactly which problems, so Week 17's own revision picks (it carries three more, in overlapping patterns — see the Overlap section immediately below) select differently.

Week 17 (`Week_17_Revised.md`) continues LLD systems directly — ATM (Day 113, introducing Chain of Responsibility, this series' eleventh pattern), Elevator (Day 114), Splitwise (Day 115, Strategy's first full system-level application, as forward-referenced from Day 107), BookMyShow (Day 116), Food Delivery (Day 117, Strategy's second full application), and Hotel Booking (Day 118) — plus its own DSA revision blocks in Heap, Trie, Backtracking, Graph, DP, and String DP. **Every one of Week 16's four LLD systems and every one of its six DSA revision picks were checked against `Week_17_Revised.md`'s own content before this map was finalized; no collision in either direction.** See the new Week 17 Overlap section immediately below for the full confirmation, including the specific cross-pattern revision-collision check three of Week 17's own revision days need.

## What Week 17 Leaves You Knowing Cold

From Day 113's ATM (State reapplied, Chain of Responsibility new), Day 114's Elevator (State reapplied a third time, SCAN/LOOK, Mock #3's Machine Coding format), Day 115's Splitwise (Strategy's first full application, dual-heap greedy settlement with an honest NP-hard caveat), Day 116's BookMyShow Part 1 (design, the race traced precisely), Day 117's BookMyShow Part 2 (pessimistic + optimistic locking, Mock #4), Day 118's Food Delivery (Strategy's second full application, System Design previewed), and Day 119's Hotel Booking (concurrency model compared, LLD phase closed) — treat all of this as solid, not something Week 18 needs to re-establish:

- **Chain of Responsibility (this series' eleventh pattern) is assumed fully earned** — what it decouples (the sender from an unknown number of handlers), the naive mutate-as-you-go failure mode and its check-then-commit fix, and the connection to Open/Closed, all assumed reflexive on a new example, not just recitable against the ATM specifically.
- **State's generalization across three genuinely different systems (Vending Machine, ATM, Elevator) is assumed to have proven the pattern transfers, not just repeated well** — including the Elevator's added wrinkle (internal-progress-driven transitions, not purely external events).
- **Strategy is assumed to have moved from "taught conceptually" to "reflexively applicable," across two structurally different applications** — Splitwise's constructor-configured strategies and Food Delivery's stateless ones. The actual definitional test (who chooses/drives the swap, Day 109) is assumed to now clearly outrank the constructor-config heuristic, not sit alongside it as equally load-bearing.
- **The check-then-act race is assumed recognizable on sight, in a genuinely new system, without prompting** — Day 116 traced it precisely in BookMyShow; this is now the third system this series has demonstrated the identical shape in (Parking Lot, the ATM's assumption, BookMyShow), and Week 18 will not re-explain what the shape looks like.
- **Pessimistic and optimistic locking are both assumed implementable from a cold start, with a concrete, contention-based justification for choosing between them** — not the abstract definitions alone. `SELECT FOR UPDATE`'s multi-process motivation and optimistic locking's CAS lineage (not a new mechanism, a new domain) are both assumed understood at the mechanism level, not just the name level.
- **`CountDownLatch` as a start-gate, and *why* a naive `.start()` loop doesn't reliably prove a concurrency fix, are both assumed understood** — testing methodology, not just the fix itself, is assumed to now be part of what "proving concurrency-safe" means.
- **The System Design framework's five steps (Requirements, Estimation, HLD, Detailed Design, Bottlenecks) are assumed held as a shape, explicitly NOT as learned material** — Day 118 previewed only; Week 18 is where each step actually gets taught, and should not assume more familiarity than a name-level preview provides.
- **All ten LLD systems and their patterns (or deliberate absence of one) are assumed listed correctly from memory** — Day 119's self-check is the actual evidence this holds, not a feeling of familiarity; only four of ten chose a GoF pattern as the headline decision, and that ratio is itself assumed understood as a lesson, not a coincidence.
- **Six more DSA revision problems were solved cold this week — K Closest Points to Origin, Design Add and Search Words, Generate Parentheses, Clone Graph, Longest Increasing Subsequence, Edit Distance** — confirming Heap, Trie, Backtracking, Graph, DP, and String DP are all still genuinely reflexive. See the DSA Revision Log — Week 17 above for exactly which problems, so Week 18's own revision picks (Day 125 names Backtracking-or-Trie explicitly) select differently.

Week 18 (`Week_18_Revised.md`) opens System Design in earnest — URL Shortener (Day 120, the 5-step framework applied for real, not previewed), the full rate-limiting landscape (Day 121, building on Token Bucket from Week 10, recapped here Day 114), and onward. **Every one of Week 17's six LLD systems and every one of its six DSA revision picks were checked against `Week_18_Revised.md`'s own content before this map was finalized; no collision in either direction, with one same-pattern revision-collision risk worth naming explicitly.** See the new Week 18 Overlap section immediately below for the full confirmation.


## What Week 18 Leaves You Knowing Cold

The System Design (HLD) 5-step framework, applied cold to three full systems this week (URL Shortener, Rate Limiter, BookMyShow at Scale) and defended under adversarial follow-up in HLD Mock #2 — distinct from the LLD framework, never conflated with it. All four rate-limiting algorithms, compared on demand, including Fixed Window's boundary flaw with the exact worked numbers. Distributed coordination as a recurring, recognizable shape — the identical check-then-act race solved four separate times this week (Day 121's Gateway, Day 122's dedup, Day 124's `synchronized nextId()`, Day 126's seat-hold) using two different mechanisms (a JVM lock when the problem was single-process, Redis+Lua when it wasn't), chosen deliberately rather than reflexively. Redis's full picture — data structures, Memcached contrast, Thundering Herd and its two mitigations, Cache-Aside via `@Cacheable`/`@CacheEvict`, and the self-invocation caveat that's now the same fact recognized for a third time (Spring AOP, Resilience4j, and now Redis caching, all one proxy-based mechanism). Consistent Hashing (Week 9, Day 60) and Sharding Strategies (Week 12, Day 78), each reused unmodified for the second time this series (first at Week 9/12, now again at Day 120 and Day 126). Twitter Snowflake's 64-bit layout, built and traced by hand. Quorum, split-brain, and Raft's three-part decomposition — with the concrete, satisfying reveal that Kubernetes' Control Plane (Week 15, Day 99) has depended on exactly this mechanism, via etcd, since it was first taught. Two HLD mocks completed, each testing a genuinely different skill (Mock #1: narrating steps under time pressure; Mock #2: defending a choice under "why not X").

Week 18 introduces no newly-solved DSA problems, by design (Weeks 16–17's own established precedent for non-DSA phases) — one cold-revision problem only (LC 40, Combination Sum II; see the Day 125-vs-126 discrepancy note in the DSA Revision Log above). Two deliberate sequencing choices, both flagged in place rather than silently absorbed: Redis's full theoretical picture landed Day 123, not Day 121, with Day 121 borrowing only the narrow slice of Redis it actually needed; and the formal SETNX-based distributed-lock pattern and idempotency keys — both reserved for Week 19, Day 130 — were deliberately not taught this week, even though two separate problems (Day 122's dedup, Day 126's seat-hold) could have used a lighter version of exactly that mechanism, solved instead by reusing Day 121's already-taught Lua-atomicity approach.

Week 19 (`Week_19_Revised.md`) opens by combining this week's mechanisms directly rather than introducing them one at a time: fan-out strategies for Instagram's feed (Day 129 — the harder version of Day 122's own Confluence reframe, now solved in full); geospatial indexing for Uber's driver tracking (Day 128 — one of the richer Redis data structures named, not built, on Day 123); and, closing the loop this week deliberately left open, idempotency keys and SETNX-based distributed locks in full (Day 130). Checked directly against `Week_19_Revised.md`'s full content: it contains no LeetCode-numbered problems at all, so there was no risk of Week 18's one revision pick (LC 40) colliding with anything Week 19 requires, and no DSA-pattern content in Week 19 to avoid teaching ahead of.

## What Week 19 Leaves You Knowing Cold

Nine more HLD systems, closing the phase at sixteen total across Weeks 18–19. WhatsApp's connection-management layer as the first genuinely stateful-connection design in the series (Day 127), including the specific reason a load balancer's sticky sessions don't solve the same problem a connection registry does. Geospatial indexing — geohashing, quadtrees, Redis Geo — defended cold under Uber-format pressure in HLD Mock #3 (Day 128), including the correctly-scoped CAP-theorem anecdote. Hybrid fan-out (Day 129) as the actual resolution of "the celebrity problem," a shape this series named on Day 78 and touched twice more (Day 122, Day 127) before finally solving it, with the underlying arithmetic — why even 50,000 writes/sec isn't fast enough at 100 million followers — shown, not asserted. Idempotency keys and SETNX-based distributed locks (Day 130) — the formal, general version of four separate informal appearances (Days 121, 122, 124, 126) — including the unsafe-lock-release bug traced step by step, and defended a second time under deliberate race-condition pressure in HLD Mock #4 (Day 131). At-least-once execution and SQS Visibility Timeout (Day 131) recognized as the identical TTL-lease shape one layer down the stack from Day 130's own lock. The Skip List (Day 132), finally built from scratch after three uses of Sorted Sets as a black box (Days 123, 128, 131) — random height replacing rebalancing as the actual engineering insight, proven by a worked search trace, not asserted. HLD Mock #5 (Day 133), candidate's choice, matching the LLD phase's own Day 119 precedent exactly.

Week 19 introduces no newly-solved DSA problems either, continuing Weeks 16–18's own precedent — two cold-revision problems only, LC 721 (Accounts Merge, Union-Find) and LC 1143 (Longest Common Subsequence, DP), both chosen specifically because neither had appeared in any prior week's DSA Revision Log, and both tied directly to Uber's own stated interview advice (Day 131). One deliberate scope decision, stated explicitly rather than left a silent gap: no extra HLD systems were added beyond the plan's own nine this week, since "extra practice" doesn't transfer cleanly from a LeetCode pattern (many small variations of one shape) to a system-design system (each one already a genuinely different problem) — the same reasoning Day 106 gave for the LLD phase, reapplied here rather than re-derived. The HLD phase closes at five mocks total across Weeks 18–19 (Days 122, 125, 128, 131, 133) — an exact match for the LLD phase's own five, and four more than the original plan's single HLD mock.

Week 20 (`Week_20_Revised.md`) shifts from designing systems to operating one — Kubernetes, Helm, distributed tracing, chaos engineering, and a service mesh, closing with a full estimation-and-bottleneck pass applied back across all sixteen HLD systems from these two weeks, treating the Estimation and Bottlenecks steps taught and applied throughout Weeks 18–19 as genuinely reflexive rather than re-teaching them. Checked directly against `Week_20_Revised.md`'s full content: it contains no LeetCode-numbered problems either, so there is no risk of Week 19's two revision picks (LC 721, LC 1143) colliding with anything Week 20 requires, and no new HLD system or concept in Week 20 to avoid teaching ahead of.

## What Week 20 Leaves You Knowing Cold

Real Kubernetes manifests for all four of `scalable-ecommerce-platform`'s modules, written by hand before ever being templated (Day 134) — the label/selector mismatch failure mode and the CPU-throttle-vs-memory-OOMKill distinction both traced concretely, not asserted — alongside a k6 load-testing baseline (≈50 req/s, p95 ≈62ms at 50 VUs) that Day 140 later treats as real evidence, not a placeholder number. Those same manifests templated into a Helm chart (Day 135), with Kubernetes Ingress deliberately, explicitly distinguished from Spring Cloud Gateway (Week 10, Day 66) — two "gateway-shaped" things that do not collapse into one just because they share a name pattern. Three environment-specific values files layered on top of that one chart (Day 136), with a Kubernetes Secret's base64 encoding proven, not asserted, to be reversible in one command, and Spring Profiles (Week 8, Day 51) cited directly as the same externalized-config idea one layer up the stack. Distributed tracing added alongside the metrics pipeline already live since Week 11 (Day 137), with its one genuine blind spot — Kafka doesn't carry trace context the way HTTP does — named explicitly rather than glossed over, given the platform's own Saga runs on exactly that transport; plus a full Stripe-format API-design pass, including a concurrent-insert pagination failure traced on real ranked positions rather than described abstractly.

Three chaos experiments, not one (Day 138) — a sustained Payment failure correctly induced by scaling to zero rather than a single pod delete, since the latter is just the reconcile loop healing itself; connection pooling taught from zero specifically to make sense of a saturation experiment that has no earlier prerequisite in this series; and the cascade-via-thread-pool-exhaustion failure mode named as a genuinely different shape from a dependency simply being down. A light service mesh demonstration, framed explicitly as moving resilience out of application code and into infrastructure — a different *kind* of thing than Resilience4j, not a fancier version of it. And the week's own hardest problem: a hand-rolled, generic `BoundedBlockingQueue<T>`, built from `ReentrantLock`/`Condition` alone, with a broken `if`-guarded version's exact race traced by hand — two producers both waiting, one `notifyAll()` waking both, neither re-checking — before the correct `while`-guarded version's invariants are proven under real concurrent contention using the identical `CountDownLatch`-gated pattern Week 17 built for BookMyShow's booking race, now proving a data structure instead of a business-logic fix. A full CI/CD pipeline (Day 139), with coverage-isn't-correctness proven via a genuine 100%-covered, zero-assertion counterexample before JaCoCo is wired as an actually build-failing gate, and the well-known dependency-in-liveness anti-pattern named and correctly routed to readiness instead, justified by Day 134's own Endpoints-removal mechanism rather than asserted as a rule to memorize.

And the platform's own first real estimation-and-bottleneck pass (Day 140) — the exact treatment all sixteen Weeks 18–19 HLD systems already received, now applied to a system this series actually built rather than only ever designed on paper, naming Sharding Strategies (Week 12, Day 78) and Consistent Hashing (Week 9, Day 60) by name as precisely what a genuine 100x fix would require, rather than a generic "add more servers." Two claims in the plan's own framing — that `todo-api` and "the original plan's `order-management-api`" both received real Kubernetes deployments earlier in the series — were checked directly against this map's history and found unsupported (`todo-api` is Docker/Compose-only, frozen since Day 62; `order-management-api` never existed as a separate repository, per `Week_21_Revised.md`'s own later note); flagged in the Day 134 and Day 139 Resource Books and in the new "How to Use This Document for Week 21" section below, not silently repeated or silently corrected, exactly this map's established handling of the same class of drift.

**Week 20 introduces zero newly-solved DSA problems and zero cold-revision re-solves** — the first week in the series where that's true in *both* senses simultaneously; Weeks 16–19 each still logged cold-revision re-solves even while adding nothing new. It introduces zero new LLD/HLD systems and runs no mock interview, consistent with the Capstone Hardening phase being a genuinely different kind of week, tracked in the new Platform Operations / Infrastructure Deliverable Inventory table above rather than forced into either existing table's shape.

`Week_21_Revised.md` (available and read in full ahead of its own generation) closes the technical-preparation phase entirely — behavioral and domain-knowledge depth, portfolio finalization, mock-interview marathons, negotiation preparation, no LeetCode problems and no new HLD systems, confirming zero overlap risk with anything Week 20 produced. It cites this week's work directly and repeatedly as settled fact rather than something to re-derive: Day 141's `docs/architecture.md` finalization explicitly continues "the move to Kubernetes/Helm" from this week; Day 142's High Availability theory explicitly cites chaos engineering as something "practiced for real against a Kubernetes deployment two weeks ago"; and the plan's own closing scorecard (Day 148) restates this week's deliverables — real Kubernetes/Helm across three environments, genuine chaos-test data, the hand-rolled concurrency component, the Stripe-format API design work, and the formal at-scale analysis — as fixed, completed facts the final week is built on top of, not aspirations still in progress. *(One small correction to this paragraph's own framing, found while actually generating Week 21 rather than while previewing it: "negotiation preparation" above turns out to be inaccurate — checked directly against `Week_21_Revised.md`'s full content, negotiation is named exactly once, in Day 148's closing paragraph, explicitly as part of the interview-execution phase that begins *after* this series ends, not as content Week 21 itself teaches. Left in place per this document's own extend-don't-edit rule; see "Series Complete," above, for the flag stated plainly.)*

## What Week 21 Leaves You Knowing Cold (Series Final)

The STAR framework held to a real standard — Situation and Task capped near a sentence each, Action as a first-person sequence of named decisions rather than team-activity description, Result quantified wherever a genuine number exists — proven against a worked illustrative example (Day 141) before being applied to eight real, individually-workshopped stories (Days 141–144), each mapped to one of six competencies with a deliberate fix for the one story the plan itself left unlabeled (Story 6, steered to Navigating Ambiguity specifically to close what a running tally showed was otherwise the map's only gap). All eight then reframed, without rebuilding any of them, against three real, independently-verified company frameworks — Google's Googleyness, Databricks' six leadership principles (the plan names two; four more were confirmed against current public sourcing), and Atlassian's five values, given verbatim by the plan and treated with the same rehearsal discipline as a technical round.

Two-Phase Commit (Week 12, Day 79) and Saga (Week 10, Day 70) recapped, not re-taught, then applied concretely to UPI's NPCI/PSP/bank-node architecture, with the one genuinely new piece — resolving an ambiguous transaction after a mid-transfer network failure — built directly on Idempotency Keys (Week 19, Day 130) rather than invented fresh. High Availability's Active-Active/Active-Passive distinction, with its failover trigger traced to the identical mechanism already built for Kubernetes readiness probes (Week 20, Day 134), not a separate concept requiring its own new mental model. The same caching/sharding/rate-limiting toolkit from Weeks 9, 12, and 18, reframed through a cost-at-scale lens for high-volume, low-margin systems, with a worked number proving the reframing rather than asserting it. Multi-tenancy's real three-tier isolation spectrum, the noisy-neighbor mechanism and its two concrete mitigations, and RBAC's role-based indirection, including the tenant-scoping subtlety a flat single-tenant model never has to consider.

A fully reconciled account of the entire series' own numbers — 197 required and 249 distinct DSA problems, 259 including the SQL track, verified directly against this map's own row-by-row data rather than any one week's internal scorecard claim, with `Week_21_Revised.md`'s own unreconciled "213" identified, explained, and deliberately kept off the finalized resume in favor of the two figures that are actually traceable. Ten LLD and sixteen HLD systems, each backed by six mock interviews by the time this week closed. A five-repository portfolio, pinned in a deliberately-reasoned order with every README honest about what it actually represents — including `todo-api`'s early, Docker/Compose-only status, correctly *not* inflated to match the capstone's later Kubernetes hardening.

This is the final entry in this section. There is no Week 22.

## ✅ Known Overlap With Week 2 — Resolved

This is the overlap flagged after Week 1 and resolved during Week 2's generation. Kept here, marked resolved, rather than deleted — the resolution approach is itself useful precedent for the next time this happens.

Two Pointers was still open at the end of Week 1 (5 of 16 required problems done), which is exactly the situation where this kind of overlap happens — the extra practice added in Week 1 to reinforce Two Pointers, and the problems Week 2's own plan scheduled to finish the pattern, were chosen independently and landed on some of the same, very canonical, problems.

**Collisions that were confirmed, and how each was actually resolved in Week 2:**

| LC # | Problem | Already done | Week 2 required it at | Resolution |
|---|---|---|---|---|
| 26 | Remove Duplicates from Sorted Array | Week 1, Day 6 (Extra) | Day 8, Problem 6 | Recapped; freed time → LC 80 (new) |
| 283 | Move Zeroes | Week 1, Day 7 (Extra) | Day 8, Problem 7 | Recapped; freed time → LC 80 (new) |
| 977 | Squares of a Sorted Array | Week 1, Day 6 (Extra) | Day 9, Problem 8 | Recapped; freed time → LC 633 (new) |
| 167 | Two Sum II | Week 1, Day 6 (Extra) | Day 9, Problem 9 | Recapped; freed time → LC 633 (new) |
| 15 | 3Sum | Week 1, Day 7 (Extra) | Day 10, Problem 10 | Recapped; LC 16 got full depth |
| 11 | Container With Most Water | Week 1, Day 7 (Extra) | Day 12, Problem 14 | Recapped; LC 75 got full depth |

All 6 are now reflected in the Problem Inventory's "Recapped in Week 2" table above, not double-counted in any total.

**Further out, informational only — still no action needed yet, unchanged from the Week 1 version of this note:**
- LC 232 (Implement Queue using Stacks): done as Week 1 Day 4 extra practice; also required at Week 6, Day 40. Week 6's generation will catch this automatically via the inventory table above, once it exists.
- LC 560 (Subarray Sum Equals K): done as Week 1 Day 5 extra practice; also required at Week 4, Day 22. Same — self-resolving when that week is generated.

---

## ✅ Known Overlap With Week 3 — Confirmed Clean

Checked `Week_03_Revised.md` (Days 15–21) against this map's full inventory before finalizing Week 2's extra-practice picks (LC 80, LC 633) and before deciding not to add extra Sliding Window practice on Day 14. Result at the time: **no collisions.**

- Neither LC 80 nor LC 633 (Week 2's extras) appeared anywhere in Week 3's required list.
- Week 3's Sliding Window problems (LC 1004, LC 3, LC 424, LC 567, LC 438, LC 209, LC 904, LC 1493, LC 340, LC 76, LC 239, LC 992 — 12 problems, Days 15–20) were all distinct from Week 2's two (LC 121, LC 643), confirming those two really were the *first* 2 of the pattern's eventual 14, not an accidental repeat.
- Week 3 Day 21 opened Prefix Sum & Kadane's Algorithm (LC 303, LC 53, LC 238) — a fresh pattern branch with no Two-Pointers-or-Sliding-Window dependency.

**Confirmed during Week 3's actual generation:** every one of Week 3's 15 required problems was independently re-checked against the full Weeks 1–2 inventory before being taught, and the pre-check above held exactly — zero recaps were needed anywhere in Week 3. This section is now closed out; no further action needed.

---

## ✅ Known Overlap With Week 4 — Resolved

Checked `Week_04_Revised.md` (Days 22–28) against this map's full inventory (including Week 3's own additions) before finalizing Week 3's extra-practice picks (LC 1695, LC 1052, LC 1838, LC 1438) and before deciding not to add extra Prefix Sum/Kadane's practice on Day 21. Pre-check result: **no collisions among required problems; one confirmed recap.**

- None of Week 3's 4 extras appeared anywhere in Week 4's required list (Week 4 required LC 525, 523, 560, 918, 122, 55, 45, 134, 56, 57, 435, 452, 252, 253, 763, 704, 278 — none matched).
- Week 3's 15 required problems were likewise all distinct from Week 4's required list.
- Week 3 Day 21 opened Prefix Sum & Kadane's at 3/7 required (LC 303, 53, 238); Week 4 Day 22 closed it with the remaining 4 (LC 525, 523, 560, 918) — confirmed non-overlapping with what Week 3 already taught, aside from the one flagged recap below.
- **The one confirmed collision:** LC 560 (Subarray Sum Equals K), required at Week 4 Day 22, had already been solved back in Week 1, Day 5, as extra practice, flagged at the time as previewing this exact pattern. **Resolution, as actually carried out in Week 4's generation:** recapped from that prior solve (not re-taught from zero) on Day 22, while the underlying "prefix sum + HashMap, first-occurrence index" technique was still formally introduced for the first time via LC 525 and LC 523, since Week 3's three Prefix Sum problems never used a HashMap at all. The time freed by not re-deriving LC 560 from scratch went toward two extra-practice problems on the same closing day (LC 974, LC 152) rather than being left unused.
- **Still further out, informational only, unchanged:** LC 232 (Implement Queue using Stacks), done as Week 1 Day 4 extra practice, is also required at Week 6, Day 40 — self-resolving whenever that week is generated.

**Confirmed during Week 4's actual generation:** every one of Week 4's 17 required problem-slots was independently re-checked against the full Weeks 1–3 inventory before being taught; the pre-check above held exactly, with the one LC 560 recap resolved as predicted and zero other collisions found. This section is now closed out; no further action needed.

---

## ✅ Known Overlap With Week 5 — Confirmed Clean

Checked `Week_05_Revised.md` (Days 29–35) against this map's full inventory (including Week 4's own additions above) before finalizing Week 4's extra-practice picks (LC 974, 152, 406, 986) and before deciding not to add extra Binary Search practice on Day 28. Result: **no collisions.**

- None of Week 4's 4 extras appears anywhere in Week 5's required list (Week 5 requires LC 35, 162, 33, 81, 153, 34, 74, 875, 1011, 206, 876, 234, 21 — none match).
- Week 4's 17 required problem-slots (16 newly solved + the LC 560 recap) are likewise all distinct from Week 5's required list.
- Week 4 Day 28 opened Binary Search at 2/11 required (LC 704, 278); Week 5 continues it with the remaining 9 (LC 35, 162, 33, 81, 153, 34, 74, 875, 1011), closing the pattern at 11/11 — confirmed non-overlapping with what Week 4 already taught. Week 5 explicitly reserves Median of Two Sorted Arrays (LC 4, Hard) for after Trees complete, rather than folding it into this Binary Search ladder — noted here so a future week doesn't need to re-discover that decision.
- **Nothing carried forward as an open action item this time** — unlike LC 560 (originally flagged back at the Week 1→2 overlap check, resolved here at Week 4) and LC 232 (flagged the same way, still pending for Week 6), no problem solved so far as extra practice appears anywhere in Week 5's required list.

**Confirmed during Week 5's actual generation:** every one of Week 5's 13 required problem-slots was independently re-checked against the full Weeks 1–4 inventory before being taught, and the pre-check above held exactly — zero recaps were needed anywhere in Week 5. All 6 of Week 5's own extra-practice picks (LC 154, 744, 1482, 1552, 92, 23) were separately checked against both this map and `Week_06_Revised.md` before being finalized, with no collisions found either way (see the Week 6 Overlap section immediately below, which also surfaced two *pre-existing* collisions unrelated to Week 5's own choices). This section is now closed out; no further action needed.

---

## ✅ Known Overlap With Week 6 — Resolved

Checked `Week_06_Revised.md` (Days 36–42) against this map's full inventory (Weeks 1–5, 95 distinct) before finalizing Week 5's extra-practice picks (LC 154, 744, 1482, 1552, 92, 23) and before deciding to withhold extras on Day 34, Linked Lists' actual opening day. Result: **no collisions caused by anything Week 5 itself chose, but two pre-existing collisions confirmed — both dating back to Week 1, Day 4, and now directly actionable.**

- None of Week 5's 6 extras appears anywhere in Week 6's required list (Week 6 requires LC 141, 142, 19, 2, 143, 138, 146, 20, 232, 496, 503, 155, 739, 150 — none match).
- Week 5's 19 newly-solved problems are likewise all distinct from Week 6's required list.
- Week 5 Days 34–35 opened Linked Lists at 4/11 required (LC 206, 876, 234, 21); Week 6 continues it with the remaining 7 (LC 141, 142, 19, 2, 143, 138, 146), closing the pattern at 11/11 — confirmed non-overlapping with what Week 5 already taught.
- **Two confirmed collisions, both carried forward since Week 1, Day 4 — flagged there at the time, now finally reaching the week they matter for:**
  - **LC 20 (Valid Parentheses):** solved **required** at Week 1, Day 4, with that table entry explicitly noting even then that it "previews Week 6's formal Stack pattern." Required again at Week 6, Day 39, Problem 1. When Week 6 is generated, this should be recapped (cite Week 1, Day 4) rather than re-taught from zero — Day 39's other required problem (LC 146, LRU Cache) is genuinely new and hard, and should get the full depth this frees up time for. **Note for whoever generates Week 6:** Day 39 is also the Stack pattern's actual opening day (the Concept Card is introduced there) — per this document's own established precedent (Day 14, 21, 23, 28, and 34 all correctly withheld extras on a pattern's opening day), the recap's freed time should go toward LC 146's depth, not toward a new Stack-pattern extra that day; if Stack extras are warranted, they belong on a later Week 6 day once the pattern is no longer opening fresh, mirroring exactly how Week 5 handled Linked Lists (opened Day 34 with none, added Days 35).
  - **LC 232 (Implement Queue using Stacks):** solved as **extra practice** at Week 1, Day 4. Required at Week 6, Day 40. Same resolution — recap, cite the origin day, don't re-teach; use the freed time for a fresh Two-Stack-pattern problem if warranted. This exact flag appears in both the Week 1→2 and Week 3→4 overlap sections above, each time noted as "still pending for Week 6" or "self-resolving whenever that week is generated" — this is that week.

**Action needed for Week 6's generation:** both collisions above need explicit handling on Days 39 and 40 respectively — recap from the cited origin day, note the overlap directly in that day's Resource Book, and reflect it in this map's next update, per this series' own standing rule that overlap gets made visible rather than silently absorbed.

**Confirmed during Week 6's actual generation:** both flagged collisions were handled exactly as prescribed — LC 20 recapped on Day 39 (citing Week 1, Day 4), with the freed time spent on LC 146's full depth rather than a same-day Stack extra, per this document's opening-day precedent; LC 232 recapped on Day 40 (citing Week 1, Day 4), with the freed time spent on LC 496's full depth instead of a same-day extra, since Day 40 turned out to be Monotonic Stack's own opening day (a distinction this section's original note didn't anticipate, since Monotonic Stack hadn't been named yet when Week 5 wrote it) — the "fresh Two-Stack-pattern problem if warranted" option this section floated was therefore deliberately not used on Day 40 itself; Monotonic Stack's deferred extras (LC 901, LC 402) landed Day 41 instead, once the pattern was no longer opening fresh. Both recaps are reflected in the Problems Recapped in Week 6 table above, and both flagged concept-level insertions this collision-handling surfaced (a `synchronized` primer before `ReentrantLock`, Day 37; doubly linked lists before LRU Cache, Day 39) are documented in the Concept-Dependency Map above. This section is now closed out; no further action needed.

---

## ✅ Known Overlap With Week 7 — Confirmed Clean

Checked `Week_07_Revised.md` (Days 43–49) against this map's full inventory (Weeks 1–6, 111 distinct) before finalizing Week 6's extra-practice picks (LC 202, 287, 901, 402) and before deciding to withhold extras on Days 39 and 40, Stacks' and Monotonic Stack's respective opening days. Result: **no collisions.**

- None of Week 6's 4 extras appears anywhere in Week 7's required list (Week 7 requires LC 735, 227, 84, 85, 224, 104, 226, 100, 101, 110, 543, 102, 199 — none match).
- Week 6's 16 newly-solved problems are likewise all distinct from Week 7's required list.
- Week 6 Days 39–41 opened Stacks/Monotonic Stack at 7/12 required (LC 20, 232, 496, 503, 155, 739, 150 — 2 of which are recaps, not new solves); Week 7 continues it with the remaining 5 (LC 735, 227, 84, 85, 224), closing the pattern at 12/12 — confirmed non-overlapping with what Week 6 already taught.
- **Nothing carried forward as an open action item this time** — unlike LC 20 and LC 232 (flagged back at the Week 1→2 overlap check, both resolved above in Week 6), no problem solved so far as extra practice appears anywhere in Week 7's required list.

**Confirmed during Week 7's actual generation:** the pre-check held exactly as predicted — no recap was needed anywhere in Week 7, and this is reflected above by the simple absence of a "Problems Recapped in Week 7" table (only Weeks 2, 4, and 6 have one, since only those weeks had an actual collision to resolve). Week 7's own three extra-practice picks (LC 572, 111, 637) were chosen with Week 8's required list in hand and checked against it directly — see the Week 8 section immediately below for that forward check. This section is now closed out; no further action needed.

---

## ✅ Known Overlap With Week 8 — Confirmed Clean

Checked `Week_08_Revised.md` (Days 50–56) against this map's full inventory (Weeks 1–7, 127 distinct) before finalizing Week 7's extra-practice picks (LC 572, 111, 637) and before deciding to withhold extras on Day 46, Trees' actual opening day. Result: **no collisions.**

- None of Week 7's 3 extras appears anywhere in Week 8's required list (Week 8 requires LC 98, 230, 235, 105, 236, 297, 4, 703, 1046, 215, 973, 347, 264 — none match). Specifically: LC 572 (Subtree of Another Tree) and LC 111 (Minimum Depth) are both Tree-DFS problems distinct from every Tree problem Week 8 requires; LC 637 (Average of Levels) is a Tree-BFS problem distinct from Week 8's required list, which contains no further BFS-on-trees problems at all — Week 8 stays entirely within BST traversal, construction, general-tree LCA, serialization, and (separately) Heaps.
- Week 7's 16 newly-solved problems are likewise all distinct from Week 8's required list.
- Week 7 Days 46–49 opened Trees at 8/15 required (LC 104, 226, 100, 101, 110, 543, 102, 199); Week 8 continues it with the remaining 7 (LC 98, 230, 235, 105, 236, 297, 4), closing the pattern at 15/15 — confirmed non-overlapping with what Week 7 already taught.
- **One item worth flagging explicitly, though it is not a collision:** `Week_08_Revised.md` itself notes that Binary Tree Maximum Path Sum and House Robber III are deliberately deferred to Dynamic Programming, later in the series, rather than being folded into either week's Tree ladder. Week 7 respected this — neither problem was used as extra practice — and Week 8's generation did the same; this is a sequencing decision the plan authors made explicitly, not an oversight to "fix" by pulling either problem forward.
- **Nothing carried forward as an open action item** — no problem solved so far as extra practice appears anywhere in Week 8's required list.

**Confirmed during Week 8's actual generation:** the pre-check held exactly as predicted — no recap was needed anywhere in Week 8, reflected above by the absence of a "Problems Recapped in Week 8" table (only Weeks 2, 4, and 6 have one). One additional discrepancy surfaced during generation that the pre-check couldn't have caught, since it's not an overlap issue: `Week_08_Revised.md`'s own Day 55 header names "Spring Cloud Config Wrap-Up," but the plan's actual Day 55 body contains no Theory or Project block content for it. Day 55's Resource Book follows the body as written (no new theory invented) and flags the discrepancy directly rather than silently absorbing it either way. Week 8's own five extra-practice picks (LC 173, 106, 1167, 378, 451) were chosen with Week 9's required list in hand and checked against it directly — see the Week 9 section immediately below for that forward check. This section is now closed out; no further action needed.

---

## ✅ Known Overlap With Week 9 — Resolved

This is the check performed during Week 8's generation, looking ahead into `Week_09_Revised.md`. Kept here, now marked resolved during Week 9's own generation, rather than deleted — the same "flag prospectively, confirm retrospectively" pattern every prior week's overlap section has used.

Checked `Week_09_Revised.md` (Days 57–63) against this map's full inventory (Weeks 1–8, 145 distinct) before finalizing Week 8's extra-practice picks (LC 173, 106, 1167, 378, 451) and before deciding to withhold extras on Day 54, Heaps' actual opening day. Result: **no collisions**, confirmed.

- None of Week 8's 5 extras appears anywhere in Week 9's required list (Week 9 requires LC 767, 621, 295, 23, 208, 677, 720, 211, 648, 212, 78, 46, 77, 47 — none match). Specifically: LC 1167 (Minimum Cost to Connect Sticks) and LC 378 (Kth Smallest Element in a Sorted Matrix) are both Heap problems distinct from Week 9's four required Heap problems (LC 767 Reorganize String, LC 621 Task Scheduler, LC 295 Find Median from Data Stream, LC 23 Merge k Sorted Lists — none match); LC 173 (BST Iterator) and LC 106 (Construct Binary Tree from Inorder/Postorder) are Tree problems, and Week 9 has no further Tree problems at all, the pattern having fully closed in Week 8; LC 451 (Sort Characters By Frequency) is a Heap/frequency problem distinct from every Week 9 requirement.
- Week 8's 18 newly-solved problems are likewise all distinct from Week 9's required list.
- Week 8 Days 54–56 opened Heaps at 6/10 required (LC 703, 1046, 215, 973, 347, 264); Week 9 continued it with the remaining 4 (LC 767, 621, 295, 23), closing the pattern at 10/10 — confirmed non-overlapping with what Week 8 already taught.
- **Confirmed during Week 9's own generation:** zero extra practice was added anywhere in Week 9 (Heaps and Tries both closed exactly at their required counts; Backtracking added none while opening), so there was never a live extra-practice pick this week that needed re-checking against this section — the pre-check held trivially, by there being no Week-9-side extras to collide with anything.

---

## ✅ Known Overlap With Week 10 — Resolved

Checked `Week_10_Revised.md` (Days 64–70) against this map's full inventory before finalizing Week 9's own extra-practice decisions. Result: **one real collision found and avoided; otherwise clean.**

- **The collision:** LC 17 (Letter Combinations of a Phone Number) was under serious consideration as Week 9 extra Backtracking practice — high interview frequency, and a distinct "build a string from per-position choices" flavor unlike any of Week 9's four required Backtracking problems (Subsets, Permutations, Combinations, Permutations II). Checking `Week_10_Revised.md` directly showed it is Week 10 **Day 65's required Problem 7**. **Not added** to Week 9. This is the concrete instance, not a hypothetical one, of this series' overlap-check process doing its job — see Day 63's closing note and this map's Backtracking entry above for the full account.
- Every other Week 10 Backtracking requirement (LC 39 Combination Sum, LC 40 Combination Sum II, LC 22 Generate Parentheses, LC 79 Word Search, LC 131 Palindrome Partitioning, LC 90 Subsets II, LC 51 N-Queens) is confirmed distinct from everything solved through Week 9 — none appears anywhere in the problem table above.
- Week 9's four required Heap problems (LC 767, 621, 295, 23) and six required Trie problems (LC 208, 677, 720, 211, 648, 212) do not appear anywhere in `Week_10_Revised.md` — both patterns are fully closed and Week 10 doesn't revisit either.
- Week 9 added **zero** extra practice, so there was no live Week-9-side pick besides the LC 17 near-miss above that needed checking against Week 10 at all — the search for collisions was thorough (every problem seriously considered was checked, not just the ones ultimately used), but the actual surface area was small precisely because this week's own judgment calls landed on adding nothing.

**Confirmed during Week 10's own generation:** LC 17 landed exactly where predicted — Day 65's required Problem 7, taught in full, appearing exactly once in the problem table above — and nowhere else in Week 10. The pre-check held completely; no correction was needed.

---

## ✅ Known Overlap With Week 11 — Checked, Two Extras Confirmed Clean

Checked `Week_11_Revised.md` (Days 71–77) against this map's full inventory before finalizing Week 10's own extra-practice decisions. Result: **clean — both of Week 10's extra-practice picks confirmed absent from Week 11's required list; no collision found in either direction.**

- **Week 10's two extras, checked against Week 11 before being added:** LC 802 (Find Eventual Safe States, added Day 69) and LC 542 (01 Matrix, added Day 70). Neither appears anywhere in `Week_11_Revised.md`'s required list (Course Schedule LC 207, Course Schedule II LC 210, Surrounded Regions LC 130, Pacific Atlantic Water Flow LC 417, Word Ladder LC 127, and the threshold-distance shortest-path problem LC 1334). Both stand as genuine extra practice, not accidental early duplicates of next week's own required work.
- **Every Week 10 required Backtracking and Graphs problem** (LC 39, 40, 17, 22, 79, 131, 90, 51, 200, 695, 133, 785, 994, 797) is confirmed absent from `Week_11_Revised.md`'s required list — Backtracking is fully closed and doesn't resurface; Graphs continues with six genuinely different required problems, none overlapping this week's six.
- **One deliberate non-choice, worth recording explicitly:** LC 1319 (Number of Operations to Make Network Connected) and similar connected-components-via-DFS/BFS problems were considered as possible Graphs extra practice this week, then set aside — not because of a table or plan collision, but because Union-Find opens next week (Week 11) and a connected-components framing is likely to resurface naturally there via a structurally different technique; introducing it early via DFS/BFS risked blunting that structure's own motivating example rather than genuinely conflicting with anything required. Noted here for transparency, since the overlap-avoidance principle is about protecting genuine new reps, not just mechanically checking two lists against each other.

**Action needed for Week 11's generation:** none beyond the standard confirm-during-actual-generation step — this section can be closed out once Week 11 is actually generated and the pre-check above is confirmed to have held.

## ✅ Known Overlap With Week 12 — Checked in Advance, Confirmed During Actual Generation, Plus One New Finding

Checked `Week_12_Revised.md` (Days 78–84) against this map's full inventory before finalizing Week 11's own extra-practice decisions. Result: **clean — Week 11's one extra-practice pick confirmed absent from Week 12's required list; no collision found in either direction, at the time this check ran.**

- **Week 11's one extra, checked against Week 12 before being added:** LC 1319 (Number of Operations to Make Network Connected, added Day 75). Does not appear anywhere in `Week_12_Revised.md`'s required list (Network Delay Time LC 743, Path with Maximum Probability LC 1514, Cheapest Flights Within K Stops LC 787, Path With Minimum Effort LC 1631, Swim in Rising Water LC 778, Climbing Stairs LC 70, Min Cost Climbing Stairs LC 746, House Robber LC 198, House Robber II LC 213, Decode Ways LC 91, Maximum Product Subarray LC 152, Word Break LC 139, Coin Change LC 322) — Week 12 is entirely Dijkstra's Algorithm and Dynamic Programming, an unrelated pattern pair, so the collision surface was small by construction.
- **Every Week 11 required Graphs and Union-Find problem** (LC 207, 210, 130, 417, 127, 1334, 684, 547, 990, 947, 1202, 721, 261) is confirmed absent from `Week_12_Revised.md`'s required list — Graphs is fully closed and doesn't resurface; Union-Find is fully closed and doesn't resurface either, though its `UnionFind` class is reused directly as Kruskal's cycle-detection subroutine (Day 77) — that's code reuse, not a problem-content collision.
- **One deliberate non-choice, worth recording for the same reason Week 10's LC 1319 note was:** LC 1631 (Path With Minimum Effort) is solvable via Union-Find plus binary search on the answer, a structurally valid alternative to the Dijkstra-minimax approach `Week_12_Revised.md` requires it for (Day 79). It was not considered as Week 11 Union-Find extra practice specifically because it's already required next week under a different technique — using it here would have meant re-teaching it from scratch seven days later under a different lens, not genuine reinforcement. (It was, in the end, presented exactly this way on Day 79 — as the named alternative approach, not fully coded, alongside the Dijkstra-minimax solution that got full treatment.)

**Confirmed during Week 12's actual generation:** the pre-check above held — LC 1319 does not appear in `Week_12_Revised.md`, exactly as predicted.

**One new finding this section's own pre-check did not, and could not, have caught:** this section only ever checked *Week 11's extra* against *Week 12's required list* — one direction. It never checked whether any of *Week 12's own required problems* had *already been solved in an earlier week*, because that check is specifically the responsibility of the week being generated, run against the full historical table, not something a prior week's forward-looking pre-check covers. Running that check during Week 12's actual generation surfaced a genuine collision: **LC 152 (Maximum Product Subarray)**, required by `Week_12_Revised.md` for Day 83, was already solved in **Week 4, Day 22** as extra practice. This is the first required-problem collision the series has hit — the mirror image of Week 4 and Week 6's recaps, which each caught an *extra* colliding with a *later* week's required list; this one caught a *required* problem colliding with an *earlier* week's extra. Handled the same way those were: a short recap on Day 83, not a re-teach, logged in the Problems Recapped in Week 12 table above. See Day 83's Overlap Notice for the full handling.

**Action needed for Week 13's generation:** none beyond the standard confirm-during-actual-generation step for Week 12's own extra — see the new Week 13 section immediately below.

## ✅ Known Overlap With Week 13 — Checked in Advance, Confirmed During Actual Generation

Checked `Week_13_Revised.md` (Days 85–91) against this map's full inventory before finalizing Week 12's own extra-practice decisions. Result: **clean — Week 12's one extra-practice pick confirmed absent from Week 13's required list; no collision found in either direction.**

- **Week 12's one extra, checked against Week 13 before being added:** LC 740 (Delete and Earn, added Day 82). Does not appear anywhere in `Week_13_Revised.md`'s required list (Coin Change II LC 518, Longest Increasing Subsequence LC 300, Partition Equal Subset Sum LC 416, Target Sum LC 494, Perfect Squares LC 279, Word Break II LC 140, Unique Paths LC 62, Unique Paths II LC 63, Minimum Path Sum LC 64, Maximal Square LC 221, Longest Common Subsequence LC 1143, Edit Distance LC 72, Longest Palindromic Substring LC 5, Palindromic Substrings LC 647, Longest Palindromic Subsequence LC 516, Interleaving String LC 97, Distinct Subsequences LC 115, Wildcard Matching LC 44, Regular Expression Matching LC 10, Burst Balloons LC 312, Minimum Insertion Steps to Make a String Palindrome LC 1312) — no collision.
- **Every Week 12 required Dijkstra's and Dynamic Programming problem** (LC 743, 1514, 787, 1631, 778, 70, 746, 198, 213, 91, 152, 139, 322) is confirmed absent from `Week_13_Revised.md`'s required list — Dijkstra's is fully closed and doesn't resurface; Dynamic Programming continues, but every Week 13 required problem is a genuinely new 1D/Grid/String/Interval DP problem, not a repeat of anything Week 12 already taught.
- **Two deliberate non-choices, worth recording for the same reason every prior week's non-choices were:** first, no extra was added in the Word Break / Coin Change family on Day 84, specifically because Word Break II, Coin Change II, and Perfect Squares are all required next week under the same or an adjacent technique — adding a same-family extra here would have created exactly the kind of collision this check exists to prevent. Second, Maximum Subarray (LC 53, Kadane's own original problem, Week 3 Day 21) was **not** re-added as a Week 12 extra alongside its recapped extension (Maximum Product Subarray, Day 83) — it was already required and taught in Week 3, and citing it directly as the recap's ancestor served the same purpose without a redundant re-solve.

**Confirmed during Week 13's actual generation:** the pre-check above held exactly as predicted — LC 740 does not appear anywhere in `Week_13_Revised.md`, and none of Week 13's own 21 required problems turned out to already be solved in an earlier week either (checked directly against the full inventory before teaching each one). **This is the first Dynamic Programming week in the series with zero recaps needed in either direction** — no "Problems Recapped in Week 13" table was needed above, unlike Week 12's LC 152 collision immediately before it. Week 13's own six extra-practice picks (LC 377, 673, 174, 583, 132, 486) were each checked against the full inventory *and* against `Week_14_Revised.md`'s required list before being added — see the new Week 14 section immediately below for that forward confirmation.

**Action needed for Week 14's generation:** none beyond the standard confirm-during-actual-generation step — this section can be closed out once Week 14 is actually generated and the pre-check below is confirmed to have held.

## ✅ Known Overlap With Week 14 — Checked in Advance, Confirmed During Actual Generation

Checked `Week_14_Revised.md` (Days 92–98) against this map's full inventory before finalizing Week 13's own extra-practice decisions. Result: **clean — all six of Week 13's extra-practice picks confirmed absent from Week 14's required list; no collision found in either direction.**

- **Week 13's six extras, checked against Week 14 before being added:** LC 377 (Combination Sum IV, Day 85), LC 673 (Number of Longest Increasing Subsequence, Day 85), LC 174 (Dungeon Game, Day 87), LC 583 (Delete Operation for Two Strings, Day 88), LC 132 (Palindrome Partitioning II, Day 90), LC 486 (Predict the Winner, Day 91). None appear anywhere in `Week_14_Revised.md`'s required list (Best Time to Buy and Sell Stock with Cooldown LC 309, with Transaction Fee LC 714, III LC 123, IV LC 188, House Robber III LC 337, Binary Tree Maximum Path Sum LC 124, Single Number LC 136, Number of 1 Bits LC 191, Power of Two LC 231, Counting Bits LC 338, Missing Number LC 268, Single Number II LC 137, Single Number III LC 260, Sum of Two Integers LC 371) — no collision.
- **Every Week 13 required 1D/Grid/String/Interval DP problem** (LC 518, 300, 416, 494, 279, 140, 62, 63, 64, 221, 1143, 72, 5, 647, 516, 97, 115, 44, 10, 312, 1312) is confirmed absent from `Week_14_Revised.md`'s required list — Week 14 moves to genuinely different DP subtypes (State Machine, Tree) and then an entirely different pattern (Bit Manipulation), so the collision surface was small by construction, the same reasoning the Week 12 → 13 check relied on.
- **One deliberate non-choice, worth recording for the same reason every prior week's non-choices were:** Predict the Winner (LC 486) was seriously considered as required-depth material rather than "extension," given it fills a genuine thinness in Interval DP's required ladder (only one other genuinely fresh recurrence, Burst Balloons) — it was ultimately kept as extra/if-time-allows specifically because `Week_13_Revised.md` itself allots Day 91 only 3–4 hours against a self-check, career block, and two required problems already, and promoting a third problem to mandatory would have broken that day's own stated pacing.

**Confirmed during Week 14's actual generation:** the pre-check above held exactly as predicted — none of Week 13's six extras collide with Week 14's required list, and none of Week 14's own fourteen required problems turned out to already be solved in an earlier week either, independently re-verified by direct search against every LC number (`309, 714, 123, 188, 337, 124, 136, 191, 231, 338, 268, 137, 260, 371`, each returning zero matches against the full inventory before being taught). **This is the second consecutive week in the series with zero recaps needed in either direction** — no "Problems Recapped in Week 14" table was needed above. One old problem did resurface, but correctly as a citation rather than a re-teach: Diameter of Binary Tree (LC 543, Week 7 Day 48) was cited when introducing Tree DP (Day 94), not re-solved — it does not appear as a Week 14 row in the problem-inventory table and is not counted toward Week 14's 16 distinct new problems. Week 14's own two extra-practice picks (LC 968, LC 461) were each checked against the full inventory *and* against `Week_15_Revised.md`'s required list before being added — see the new Week 15 section immediately below for that forward confirmation.

## ✅ Known Overlap With Week 15 — Checked in Advance, Confirmed During Actual Generation

Checked `Week_15_Revised.md` (Days 99–105) against this map's full inventory before finalizing Week 14's own extra-practice decisions. Result: **clean — both of Week 14's extra-practice picks confirmed absent from Week 15's required list; no collision found in either direction.**

- **Week 14's two extras, checked against Week 15 before being added:** LC 968 (Binary Tree Cameras, Day 94), LC 461 (Hamming Distance, Day 97). Neither appears anywhere in `Week_15_Revised.md`'s required list (Reverse Bits LC 190, Maximum XOR of Two Numbers in an Array LC 421, Range Sum Query – Mutable LC 307, Count of Smaller Numbers After Self LC 315, Kth Largest Element in an Array LC 215, plus ten SQL problems across Days 103–104) — no collision.
- **Every Week 14 required State Machine DP, Tree DP, and Bit Manipulation problem** (LC 714, 309, 123, 188, 337, 124, 136, 191, 231, 338, 268, 137, 260, 371) is confirmed absent from `Week_15_Revised.md`'s required list — Week 15 moves to closing Bit Manipulation's final two problems (new, not repeats), then to Segment Trees, a full Sorting review, and SQL, an essentially disjoint surface by construction.
- **One dependency, flagged in advance and handled exactly as anticipated:** LC 215 (Kth Largest Element in an Array), required in `Week_15_Revised.md` Day 102, was already solved once before — Week 8, Day 55, via a min-heap of size `k`, with Quickselect named at the time as a future destination. Week 15's Day 102 gave LC 215 a short recap and reserved full depth for Quickselect as the genuinely new technique, reusing the day's own quicksort partition method directly — the same asymmetric handling Week 14 gave LC 121/122 on Day 92, exactly as this section anticipated.

**Confirmed during Week 15's actual generation:** the pre-check above held exactly as predicted — none of Week 14's two extras collide with Week 15's required list, and every one of Week 15's own fourteen new required problems (LC 190, 421, 307, 315, plus ten SQL problems) was confirmed absent from the full inventory before being taught, independently re-verified rather than assumed clean from the pre-check alone. **Week 15 added zero extra practice anywhere**, so there is nothing from this week to check against Week 16 in the "extra collides with a later required problem" direction — the only carry-forward item is the LC 215 recap already resolved above. Week 15's own required problems were also checked directly against `Week_16_Revised.md` (available at the time of this map's own extension) before finalizing this map — see the new Week 16 section immediately below.

## ✅ Known Overlap With Week 16 — Checked, Confirmed Clean

Checked `Week_16_Revised.md` (Days 106–112) against this map's full inventory, and against Week 15's own required list, before finalizing this map's Week 15 extension. Result: **clean — no collision in either direction, and a structurally simple check besides.**

- **Week 15 added zero extra-practice problems anywhere**, so the usual "does an extra collide with next week's required list" check has nothing to check this time — there is no Week 15 extra that could possibly collide with anything in Week 16.
- **Week 16 introduces no new required LeetCode problems in any of Week 15's patterns.** Its DSA-adjacent content (Days 108–109, per `Week_16_Revised.md`) is explicitly framed as **revision** — re-solving already-closed problems from earlier patterns (Sliding Window, Backtracking, Trees, Union-Find/Dijkstra's, Graphs, Dynamic Programming) cold, without hints, rather than introducing new LC-numbered required problems. None of these revision slots touch Bit Manipulation, Tries, Segment Trees, Sorting, or SQL — Week 15's five patterns — so there is no possibility of a required-problem collision against this week's inventory in either direction.
- **Week 16's actual new content is non-DSA:** Structural design patterns (Adapter, Decorator, Facade, Proxy, Composite, Day 106) and Behavioral patterns (Day 107), both building directly on Week 15's three Creational patterns without re-deriving them — confirmed as a direct dependency, not an overlap risk, since design patterns aren't tracked in the LeetCode-problem inventory table this overlap-check process governs.

**Action needed for Week 17's generation:** none yet — no `Week_17_Revised.md` was available at the time of this check. Whoever generates Week 16 should run this same pre-check against Week 17's plan once it exists, the same way this section was produced against Week 16's.

**Retrospective correction, added once Week 16 was actually generated day by day — not edited into the bullet above, per this map's own extend-don't-edit rule:** the second bullet above states Week 16's revision content falls on "Days 108–109." Once every day was actually built, the revision blocks were confirmed to span **five** days, not two — Day 107 (one Graph + one DP problem), Day 108 (Backtracking), Day 109 (Sliding Window), Day 110 (Tree), and Day 111 (Union-Find/Dijkstra's) — matching the five *patterns* this section already named correctly, just not the day range. Most likely written against an earlier draft of `Week_16_Revised.md`, before Mock Interviews and the fuller revision-block spread were added in its own "Revised" pass. Flagged here rather than silently corrected above, the same treatment this map gives every other discrepancy it finds in its own prior entries.

---

## ✅ Known Overlap With Week 17 — Checked, Confirmed Clean

Checked `Week_17_Revised.md` (Days 113–119) against this map's full inventory — including the new LLD System Inventory and DSA Revision Log above — before finalizing this map's Week 16 extension. Result: **clean — no collision in either direction**, with one specific cross-pattern check worth stating explicitly rather than leaving implicit.

- **Week 16 added zero extra-practice DSA problems and zero extra LLD systems**, so the usual "does an extra collide with next week's required list" check has nothing to check on that front — there is nothing from Week 16's own "extra" column, in either table, that could collide with Week 17.
- **Week 17 introduces no new required LeetCode problems that collide with anything in Week 16's DSA Revision Log.** Week 17's own DSA-adjacent content is, like Week 16's, framed as revision of already-closed patterns — Heap (Day 113), Trie (Day 114), Backtracking (Day 115), Graph (Day 116), DP (Day 117), and String DP (Day 118) — checked directly against the six specific problems Week 16 logged (Course Schedule, Coin Change, Combination Sum, Longest Substring Without Repeating Characters, Validate BST, Redundant Connection): no shared LC numbers.
- **A same-pattern revision collision is possible in three patterns, even without a shared LC number, and is worth naming explicitly rather than only checking numbers:** Week 17 carries its own revision blocks in **Backtracking** (Day 115), **Graph** (Day 116), and **DP** (Day 117) — the same three patterns Week 16 already drew a cold-solve pick from (Combination Sum, Course Schedule, Coin Change respectively). Since neither week's revision picks are named by the plan itself — both are chosen by the Resource Book generation, from each pattern's full pool of already-taught problems — there's a real risk Week 17's own generation reaches for the same "obvious" pick a second time within two weeks, which would defeat the point of spaced repetition rather than serve it. **Whoever generates Week 17 should pick a *different* Backtracking, Graph, and DP problem than Week 16's three** (i.e., not Combination Sum, Course Schedule, or Coin Change again) — the DSA Revision Log above is the direct reference for what to avoid. Week 17's other three revision patterns (Heap, Trie, String DP) don't overlap with anything Week 16 touched, so no equivalent check is needed there.
- **Week 17's actual new content is non-DSA, like Week 16's:** six more LLD systems (ATM, Elevator, Splitwise, BookMyShow, Food Delivery, Hotel Booking), plus Chain of Responsibility as an eleventh pattern and Strategy's first two full system-level applications (Splitwise, Food Delivery) — both confirmed as direct, cited dependencies on Week 16's own pattern catalog and framework, not overlap risks, since LLD systems and patterns aren't tracked in the LeetCode-problem inventory this overlap-check process primarily governs.

**Action needed for Week 18's generation:** none yet — no `Week_18_Revised.md` was available at the time of this check. Whoever generates Week 17 should run this same pre-check against Week 18's plan once it exists, paying particular attention to whichever three specific problems get chosen for Week 17's own Backtracking/Graph/DP revision slots, so a third week doesn't reach for the same picks a third time.

## ✅ Known Overlap With Week 18 — Checked, Confirmed Clean

Checked `Week_18_Revised.md` (Days 120–126, its full content) against this map's full inventory — including the LLD System Inventory and both weeks' DSA Revision Logs above — before finalizing this map's Week 17 extension. Result: **clean — no collision in either direction**, with one specific cross-pattern risk worth stating explicitly, the same category of check the Week 16→17 transition already needed once.

- **Week 17 added zero extra-practice DSA problems and zero extra LLD systems**, exactly like Week 16 — nothing from Week 17's own "extra" column, in either table, exists to collide with Week 18.
- **Week 18 introduces no new required LeetCode problems that collide with anything in Week 17's DSA Revision Log.** Checked directly against this week's six specific picks (K Closest Points to Origin/LC 973, Design Add and Search Words/LC 211, Generate Parentheses/LC 22, Clone Graph/LC 133, Longest Increasing Subsequence/LC 300, Edit Distance/LC 72): no shared LC numbers anywhere in `Week_18_Revised.md`.
- **A same-pattern revision collision is possible in two patterns, worth naming explicitly rather than only checking numbers — the same category of risk the Week 16→17 transition flagged for Backtracking/Graph/DP:** Day 125's self-check names **"one Backtracking or Trie problem, cold"** explicitly, without specifying which. Week 17 already drew a cold-solve pick from both: **Backtracking** (Generate Parentheses, LC 22, Day 115) and **Trie** (Design Add and Search Words Data Structure, LC 211, Day 114) — and Week 16, before that, also drew from Backtracking (Combination Sum, LC 39). **Whoever generates Week 18 should pick a problem for Day 125 that is neither LC 22 nor LC 211 nor LC 39**, regardless of which of the two patterns Day 125 ends up drawing from — the DSA Revision Log — Week 17 (and Week 16's, above it) is the direct reference for what to avoid.
- **Week 18's actual new content is non-DSA, continuing Week 16–17's pattern:** System Design's five-step framework taught in full for the first time (Day 120, previewed only as of Day 118 — confirmed no full-depth teaching happened early, respecting the boundary Day 118 deliberately held), the complete rate-limiting landscape (Day 121 — Token Bucket, Leaking Bucket, Fixed Window, Sliding Window Log — building on, not colliding with, Day 114's Token Bucket recap, itself sourced from Week 10 Day 68), and further HLD systems (Days 122–126, including a direct citation back to BookMyShow's object design and concurrency work, Days 116–117, rather than re-deriving it) — confirmed as cited dependencies on Weeks 16–17's work, not overlap risks.

**Action needed for Week 19's generation:** run this same pre-check against Week 19's plan once it exists. Pay particular attention to whichever specific problem Day 125 actually draws (Backtracking or Trie, and which LC number) — log it in a Week 18 DSA Revision Log entry when Week 18 is generated, so Week 19 (and any later week revisiting these two patterns) has the complete, current "already used" list rather than just Weeks 16–17's.

---


## ✅ Known Overlap With Week 19 — Checked, Confirmed Clean

Checked `Week_19_Revised.md` in full (all of Days 127–133) against Week 18's one DSA revision pick and against everything Week 18 taught, in both directions.

**DSA overlap: not applicable, cleanly.** `Week_19_Revised.md` contains no LeetCode-numbered problems anywhere in its content — it continues the HLD phase directly, with no DSA-pattern reinforcement block of its own. Week 18's single revision pick (LC 40, Combination Sum II, Day 126) has no possible collision with Week 19's content, in either direction: it isn't required there, and nothing in Week 19 needed to be avoided when it was chosen.

**Teaching-ahead check, both directions:** Week 19 opens with several topics that could plausibly have been touched early by an over-eager Week 18 — checked directly, none were:
- **Fan-out strategies (push/pull/hybrid)**, Week 19's Day 129 topic — Day 122's Confluence reframe this week deliberately stopped at *recognizing* the fan-out shape ("10,000 watchers, edited every minute"), explicitly deferring push-vs-pull-vs-hybrid to Day 129, by name, rather than solving it early.
- **Redis Sorted Sets** (Week 19, Day 132, Leaderboard) and **Redis geospatial indexing** (Week 19, Day 128, Uber) — both named in Day 123's Redis survey as data structures that exist, with their concrete payoff problems explicitly deferred to Week 19 by day number, not built or exercised this week.
- **SETNX-based distributed locks and idempotency keys** (Week 19, Day 130) — as detailed in the System Design / HLD Deliverable Inventory table above, Week 18 twice needed *some* check-and-set mechanism (Day 122's dedup, Day 126's seat-hold) and both times deliberately reused Day 121's already-taught Lua-atomicity pattern instead of building a lighter version of Day 130's actual content early.

**What Week 19 inherits directly, as reflexive building blocks rather than material to re-teach:** the HLD 5-step framework (three full applications this week, one successfully defended under adversarial follow-up in Mock #2); all four rate-limiting algorithms; Redis's full picture including Thundering Herd and Cache-Aside; Consistent Hashing and Sharding Strategies, each now applied twice; Snowflake ID generation; and quorum/Raft/split-brain, including etcd's concrete role under Kubernetes. Week 19's own generation should cite these by day number exactly as this week cited Weeks 9, 10, 12, 14, and 17 — not re-teach any of them from zero.

## ✅ Known Overlap With Week 20 — Checked, Confirmed Clean

Checked `Week_20_Revised.md` in full (all of Days 134–140) against Week 19's two DSA revision picks and against everything Week 19 taught, in both directions.

**DSA overlap: not applicable, cleanly.** `Week_20_Revised.md` contains no LeetCode-numbered problems anywhere in its content — it shifts from designing the platform to deploying and operating it (Kubernetes, Helm, distributed tracing, chaos engineering, service mesh, CI/CD), with no DSA-pattern reinforcement block of its own. Week 19's two revision picks (LC 721, LC 1143, Day 131) have no possible collision with Week 20's content, in either direction: neither is required there, and nothing in Week 20 needed to be avoided when they were chosen.

**Teaching-ahead check, both directions:** Week 20 has no HLD-system or systems-design-concept content that could plausibly have been touched early by Week 19 — the two weeks operate at genuinely different layers (Week 19: what to build; Week 20: how to run what's already built). No collision found in either direction.

**What Week 20 inherits directly, as reflexive building blocks rather than material to re-teach:** the HLD 5-step framework's Estimation and Bottlenecks steps specifically, applied to all sixteen HLD systems across Weeks 18–19, not merely sketched — `Week_20_Revised.md`'s own Day 140 content explicitly assumes this happened for every one of the sixteen ("what breaks first at 10x, what breaks at 100x"), and it is confirmed true here for all nine of Week 19's systems (see each day's own Estimation and Bottlenecks sections above). Week 20's own generation should cite specific HLD systems by name and day number when its capstone-hardening work touches them, not re-describe any of them from scratch.

## ✅ Known Overlap With Week 21 — Checked, Confirmed Clean

Checked `Week_21_Revised.md` in full (all of Days 141–148) against everything Week 20 taught and built, in both directions.

**DSA overlap: not applicable, cleanly.** `Week_21_Revised.md` contains no newly-assigned LeetCode-numbered problems anywhere — its only DSA-adjacent content is spot-check review (Days 141, 148: "pick one DSA pattern... explain it for 2 minutes, cold"), reusing already-taught material for retrieval practice, not new problem-solving. No collision with Week 20's own zero DSA problems is possible in either direction.

**HLD/LLD overlap: not applicable, cleanly.** Day 138's own 60-minute System Design Mock Interview explicitly picks an existing system from the sixteen already designed (naming Payment System, Day 130, or Distributed Job Scheduler, Day 131, as good choices) — a sixth HLD-adjacent mock reusing prior design work, not a new system. No new HLD or LLD system is introduced anywhere in Week 21.

**Teaching-ahead check, both directions:** Week 21 is entirely behavioral, domain-knowledge, portfolio-finalization, and negotiation content — no infrastructure, deployment, or operations material that could plausibly overlap with Week 20's Kubernetes/Helm/tracing/chaos/CI-CD work. No collision found in either direction.

**What Week 21 inherits directly from Week 20, as settled fact rather than material to re-derive:** Day 141's `docs/architecture.md` finalization explicitly continues "the move to Kubernetes/Helm" as an already-completed fact; Day 142's High Availability theory explicitly cites chaos engineering as "practiced for real against a Kubernetes deployment two weeks ago"; Day 148's closing scorecard restates real Kubernetes/Helm across three environments, genuine chaos-test data, the hand-rolled concurrency component, the Stripe-format API design work, and the formal at-scale analysis as fixed facts about the platform, not aspirations. Week 21's own generation should cite these as completed capstone-hardening work by day number (Days 134–140) rather than re-describe any of it.

**⚠️ A numeric discrepancy worth flagging now, ahead of Week 21's own generation, rather than after:** `Week_21_Revised.md` states the DSA curriculum twice as **"213 problems"** (Day 144's resume-pass task; Day 148's closing scorecard). This map's own row-by-row table holds the DSA phase's authoritative closing count at **197 required + 53 extra = 249 distinct** (Weeks 1–15), plus the 10-problem SQL track = **259 distinct total** — and separately, `Week_15_Revised.md`'s own stated figure of "203" required-only was already flagged as drift against this map's 197 during the Week 15 extension (see the standing note in "How to Use This Document," below). "213" is now a **third** number in circulation, matching neither 197, 203, nor 249/259. Not expected to block anything in Week 21's own generation, since Week 21 doesn't re-derive this count, only states it in prose — but whoever finalizes Day 144's actual resume language should trace which figure (if any) the resume itself should use, rather than let a third unreconciled number propagate into a document a real interviewer might read.

## File Index

| File | Status | Covers |
|---|---|---|
| `00_Curriculum_Map.md` | ✅ Done | This file |
| `Day1_Resource_Book.md` | ✅ Done | Program model, JVM, variables, operators, control flow, loops, methods, Git basics |
| `Day2_Resource_Book.md` | ✅ Done | Arrays, Strings, OOP fundamentals |
| `Day3_Resource_Book.md` | ✅ Done | Big-O notation, ArrayList |
| `Day4_Resource_Book.md` | ✅ Done | HashSet, HashMap, Stack/Queue |
| `Day5_Resource_Book.md` | ✅ Done | HashMap/HashSet pattern (12 problems), OOP 4 pillars |
| `Day6_Resource_Book.md` | ✅ Done | Two Pointers pattern (6 problems), SOLID |
| `Day7_Resource_Book.md` | ✅ Done | Two Pointers pattern (6 problems), Week 1 wrap-up |
| `Week1_Interview_Questions.md` | ✅ Done | Every interview question from every Week 1 day, consolidated |
| `Day8_Resource_Book.md` | ✅ Done | Two Pointers recap (LC 26, 283) + LC 80, Recursion |
| `Day9_Resource_Book.md` | ✅ Done | Two Pointers recap (LC 977, 167) + LC 633, JVM Memory Model |
| `Day10_Resource_Book.md` | ✅ Done | 3Sum recap + 3Sum Closest, Primitives & Integer Cache |
| `Day11_Resource_Book.md` | ✅ Done | 4Sum, Boats to Save Most People, String Internals |
| `Day12_Resource_Book.md` | ✅ Done | Container recap + Sort Colors, Collections Internals |
| `Day13_Resource_Book.md` | ✅ Done | Trapping Rain Water, full Two Pointers consolidation (16-problem answer key) |
| `Day14_Resource_Book.md` | ✅ Done | Sliding Window begins (LC 121, 643), Week 2 Consolidation |
| `Week2_Interview_Questions.md` | ✅ Done | Every interview question from every Week 2 day, consolidated |
| `Day15_Resource_Book.md` | ✅ Done | Sliding Window continues (variable-size, LC 1004, 3 + extra LC 1695), HashMap Internals |
| `Day16_Resource_Book.md` | ✅ Done | Sliding Window (LC 424, 567 + extra LC 1052), Generics |
| `Day17_Resource_Book.md` | ✅ Done | Sliding Window (LC 438, 209), Comparable vs. Comparator, TreeMap/PriorityQueue |
| `Day18_Resource_Book.md` | ✅ Done | Sliding Window (LC 904, 1493 + extra LC 1838), Exception Handling |
| `Day19_Resource_Book.md` | ✅ Done | Sliding Window Hard tier (LC 340, 76), no new theory |
| `Day20_Resource_Book.md` | ✅ Done | Sliding Window capstone (LC 239, 992 + extra LC 1438), pattern closes at 14+4, fixed/variable review |
| `Day21_Resource_Book.md` | ✅ Done | Prefix Sum & Kadane's begins (LC 303, 53, 238), Week 3 Consolidation |
| `Week3_Interview_Questions.md` | ✅ Done | Every interview question from every Week 3 day, consolidated |
| `Day22_Resource_Book.md` | ✅ Done | Prefix Sum & Kadane's closes (LC 525, 523, 918 + recap LC 560 + extra LC 974, 152) |
| `Day23_Resource_Book.md` | ✅ Done | Greedy & Intervals begins (LC 122, 55, 45) — exchange argument formalized |
| `Day24_Resource_Book.md` | ✅ Done | Greedy continues (LC 134), Intervals begins (LC 56, 57 + extra LC 406) |
| `Day25_Resource_Book.md` | ✅ Done | Intervals, sort by end time (LC 435, 452 + extra LC 986) |
| `Day26_Resource_Book.md` | ✅ Done | Meeting Rooms (LC 252, 253) — min-heap applied to intervals |
| `Day27_Resource_Book.md` | ✅ Done | Greedy & Intervals capstone (LC 763), pattern closes at 11+2, Theory + Career blocks return |
| `Day28_Resource_Book.md` | ✅ Done | Binary Search begins (LC 704, 278), Records/Sealed Classes (+ Java 21 correction), Week 4 Consolidation |
| `Week4_Interview_Questions.md` | ✅ Done | Every interview question from every Week 4 day, consolidated |
| `Day29_Resource_Book.md` | ✅ Done | Binary Search continues (LC 35, 162), Threads and the JVM Concurrency Model |
| `Day30_Resource_Book.md` | ✅ Done | Binary Search: Rotated Arrays (LC 33, 81) |
| `Day31_Resource_Book.md` | ✅ Done | Binary Search: Min in Rotated Arrays, Boundary Search (LC 153, 34 + extra LC 154, 744) |
| `Day32_Resource_Book.md` | ✅ Done | Binary Search: 2D Matrices, On the Answer (LC 74, 875; LC 240 discussed for contrast, not solved) |
| `Day33_Resource_Book.md` | ✅ Done | Binary Search capstone (LC 1011 + extra LC 1482, 1552), pattern closes at 11+4=15, on-input/on-answer review |
| `Day34_Resource_Book.md` | ✅ Done | Linked Lists begins (LC 206, 876), REST APIs and Spring Boot Initialization, `todo-api` live |
| `Day35_Resource_Book.md` | ✅ Done | Linked Lists continues (LC 234, 21 + extra LC 92, 23), Week 5 Consolidation |
| `Week5_Interview_Questions.md` | ✅ Done | Every interview question from every Week 5 day, consolidated |
| `Day36_Resource_Book.md` | ✅ Done | Linked List Cycles (LC 141, 142 + extra LC 202, 287), Spring Data JPA, `todo-api` persistence |
| `Day37_Resource_Book.md` | ✅ Done | Linked Lists (LC 19, 2), `synchronized` primer + Explicit Locks (`ReentrantLock`), first data-corruption demo |
| `Day38_Resource_Book.md` | ✅ Done | Linked Lists (LC 143, 138), Wait/Notify and `Condition`, Producer-Consumer |
| `Day39_Resource_Book.md` | ✅ Done | Doubly Linked Lists + LRU Cache (LC 146) closes Linked Lists; recap LC 20; Concept Card — Stacks; `ConcurrentHashMap`/CAS |
| `Day40_Resource_Book.md` | ✅ Done | Recap LC 232; Monotonic Stack begins (LC 496); SQL Fundamentals; Flyway migration |
| `Day41_Resource_Book.md` | ✅ Done | Monotonic Stack: circular variant + Min Stack (LC 503, 155 + extra LC 901, 402); no Theory/Project block, per plan |
| `Day42_Resource_Book.md` | ✅ Done | Self-check; Monotonic Stack (LC 739, 150); Week 6 Consolidation |
| `Week6_Interview_Questions.md` | ✅ Done | Every interview question from every Week 6 day, consolidated |
| `Day43_Resource_Book.md` | ✅ Done | Stack Simulation (LC 735, 227), Docker Basics, `todo-api` multi-stage `Dockerfile` |
| `Day44_Resource_Book.md` | ✅ Done | Stacks Hard tier (LC 84, 85), Docker Compose, `todo-api` + Postgres + Redis |
| `Day45_Resource_Book.md` | ✅ Done | Stacks capstone (LC 224), pattern closes at 12+2=14, full two-family review; JUnit 5/AssertJ |
| `Day46_Resource_Book.md` | ✅ Done | Trees begins from zero (LC 104, 226); Mockito |
| `Day47_Resource_Book.md` | ✅ Done | Trees continues (LC 100, 101 + extra LC 572); TestContainers |
| `Day48_Resource_Book.md` | ✅ Done | Trees: global-state DFS (LC 110, 543 + extra LC 111); Kafka Fundamentals |
| `Day49_Resource_Book.md` | ✅ Done | Self-check; Tree BFS (LC 102, 199 + extra LC 637); Week 7 Consolidation |
| `Week7_Interview_Questions.md` | ✅ Done | Every interview question from every Week 7 day, consolidated |
| `Day50_Resource_Book.md` | ✅ Done | BST begins (LC 98, 230 + extra LC 173); full heap-preview citations to Day 17/26; Kafka Consumers |
| `Day51_Resource_Book.md` | ✅ Done | LCA of BST + tree construction (LC 235, 105 + extra LC 106); Divide and Conquer formalized; Spring Profiles |
| `Day52_Resource_Book.md` | ✅ Done | General-tree LCA + Serialize/Deserialize (LC 236, 297); WireMock |
| `Day53_Resource_Book.md` | ✅ Done | Trees capstone (LC 4), pattern closes at 15+5=20; BST vs. general LCA review, written out in full |
| `Day54_Resource_Book.md` | ✅ Done | Heaps begins from zero (LC 703, 1046); full array-backed heap mechanism, unconditional-vs-BST contrast |
| `Day55_Resource_Book.md` | ✅ Done | Heaps continues (LC 215, 973 + extra LC 1167, 378); Quickselect delivered, Day 32's staircase resolved |
| `Day56_Resource_Book.md` | ✅ Done | Self-check; Heaps frequency/generation (LC 347, 264 + extra LC 451); Week 8 Consolidation |
| `Week8_Interview_Questions.md` | ✅ Done | Every interview question from every Week 8 day, consolidated |
| `Day57_Resource_Book.md` | ✅ Done | Heaps: greedy scheduling (LC 767, 621) |
| `Day58_Resource_Book.md` | ✅ Done | Heaps capstone (LC 295, 23), pattern closes at 10+3=13; six-role accounting, no extra added |
| `Day59_Resource_Book.md` | ✅ Done | Tries begins from zero (LC 208, 677); CAP Theorem |
| `Day60_Resource_Book.md` | ✅ Done | Tries continues (LC 720, 211 — complexity corrected from plan shorthand); Consistent Hashing |
| `Day61_Resource_Book.md` | ✅ Done | Tries closes (LC 648, 212), pattern closes at 6+0=6; Backtracking begins (LC 78); Replication Models |
| `Day62_Resource_Book.md` | ✅ Done | Backtracking continues (LC 46); Spring AOP; `scalable-ecommerce-platform` initializes |
| `Day63_Resource_Book.md` | ✅ Done | Backtracking continues (LC 77, 47); Week 9 Consolidation, including the LC 17 overlap catch |
| `Week9_Interview_Questions.md` | ✅ Done | Every interview question from every Week 9 day, consolidated |
| `Day64_Resource_Book.md` | ✅ Done | Backtracking continues (LC 39, 40 — duplicate-skip translated to forward-index form); Resilience4j |
| `Day65_Resource_Book.md` | ✅ Done | Backtracking continues (LC 17, 22 — closeCount<openCount proven); Feign Clients |
| `Day66_Resource_Book.md` | ✅ Done | Backtracking continues (LC 79 — recap-heavy, cites Wk9 D61 directly; LC 131); Spring Cloud Gateway |
| `Day67_Resource_Book.md` | ✅ Done | Backtracking closes (LC 90, 51), pattern closes at 12+0=12, full classification table; JWT at the Gateway |
| `Day68_Resource_Book.md` | ✅ Done | Graphs begins from zero (LC 200, 695); Concept Card; Token Bucket rate limiting |
| `Day69_Resource_Book.md` | ✅ Done | Graphs continues (LC 133, 785 + extra LC 802); Kafka Schema Registry |
| `Day70_Resource_Book.md` | ✅ Done | Graphs continues (LC 994, 797 + extra LC 542); Saga Choreography; Week 10 Consolidation |
| `Week10_Interview_Questions.md` | ✅ Done | Every interview question from every Week 10 day, consolidated |
| `Day71_Resource_Book.md` | ✅ Done | Topological Sort (LC 207, 210); TCP/UDP, DNS |
| `Day72_Resource_Book.md` | ✅ Done | Multi-source reachability, reversed traversal (LC 130, 417); HTTP/HTTPS, Load Balancers |
| `Day73_Resource_Book.md` | ✅ Done | Graphs capstone (LC 127, 1334), pattern closes 12/12 + 2 extra = 14 distinct; Floyd-Warshall; BFS-vs-DFS-vs-neither review |
| `Day74_Resource_Book.md` | ✅ Done | Union-Find opens from zero (LC 684, 547); AWS Networking (VPC, subnets, IGW/NAT, Security Groups) |
| `Day75_Resource_Book.md` | ✅ Done | Union-Find continues (LC 990, 947 + extra LC 1319); AWS IAM, S3 vs. EBS |
| `Day76_Resource_Book.md` | ✅ Done | Union-Find continues (LC 1202, 721); Prometheus, Micrometer |
| `Day77_Resource_Book.md` | ✅ Done | Union-Find closes (LC 261), pattern closes 7/7 + 1 extra = 8 distinct; Minimum Spanning Trees (Kruskal's, Prim's); Grafana; Week 11 Consolidation |
| `Week11_Interview_Questions.md` | ✅ Done | Every interview question from every Week 11 day, consolidated |
| `Day78_Resource_Book.md` | ✅ Done | Dijkstra's Algorithm opens (LC 743, 1514); Sharding Strategies |
| `Day79_Resource_Book.md` | ✅ Done | Dijkstra's continues (LC 787, 1631), NEW: Bellman-Ford; Two-Phase Commit vs. Saga (recapped) |
| `Day80_Resource_Book.md` | ✅ Done | Dijkstra's capstone (LC 778), pattern closes 5/5 + 0 extra = 5 distinct; Kafka Streams (KStream/KTable) |
| `Day81_Resource_Book.md` | ✅ Done | Dynamic Programming opens from zero (LC 70, 746); fib(5) call tree re-traced, memoization vs. tabulation |
| `Day82_Resource_Book.md` | ✅ Done | House Robber family (LC 198, 213 + extra LC 740) |
| `Day83_Resource_Book.md` | ✅ Done | Decode Ways (LC 91) + Maximum Product Subarray recapped (LC 152, originally Week 4 Day 22) |
| `Day84_Resource_Book.md` | ✅ Done | Word Break, Coin Change (LC 139, 322), NEW: Unbounded Knapsack; Week 12 Consolidation |
| `Week12_Interview_Questions.md` | ✅ Done | Every interview question from every Week 12 day, consolidated |
| `Day85_Resource_Book.md` | ✅ Done | Coin Change II + extra Combination Sum IV (LC 518, 377); LIS two ways (LC 300) + extra Number of LIS (LC 673); NEW: 0/1 Knapsack begins (Partition Equal Subset Sum, LC 416) |
| `Day86_Resource_Book.md` | ✅ Done | Target Sum, Perfect Squares (LC 494, 279); Word Break II (LC 140), NEW: DP-gated Backtracking fusion; 1D DP closes 14/14 |
| `Day87_Resource_Book.md` | ✅ Done | NEW: Grid DP opens and closes same day — Unique Paths, Unique Paths II, Minimum Path Sum, Maximal Square (LC 62, 63, 64, 221) + extra Dungeon Game (LC 174) |
| `Day88_Resource_Book.md` | ✅ Done | NEW: String DP opens — LCS, Edit Distance, Longest Palindromic Substring (LC 1143, 72, 5) + extra Delete Operation for Two Strings (LC 583) |
| `Day89_Resource_Book.md` | ✅ Done | Palindromic Substrings, Longest Palindromic Subsequence, Interleaving String (LC 647, 516, 97); LPS = LCS(s,reverse(s)) proven both directions |
| `Day90_Resource_Book.md` | ✅ Done | Distinct Subsequences, Wildcard Matching, Regular Expression Matching (LC 115, 44, 10); String DP closes 9/9; extra Palindrome Partitioning II (LC 132) pays off Week 10 Day 66 deferral |
| `Day91_Resource_Book.md` | ✅ Done | NEW: Interval DP opens and closes same day — Burst Balloons, Minimum Insertion Steps (LC 312, 1312) + extra Predict the Winner (LC 486); Week 13 Consolidation |
| `Week13_Interview_Questions.md` | ✅ Done | Every interview question from every Week 13 day, consolidated |
| `Day92_Resource_Book.md` | ✅ Done | NEW: State Machine DP opens — Transaction Fee, Cooldown (LC 714, 309, reordered from the plan's stated sequence) + 0 extra; Java 21 Pattern Matching (record patterns, guarded switch) |
| `Day93_Resource_Book.md` | ✅ Done | State Machine DP closes — Stock III, Stock IV (LC 123, 188) + 0 extra = 4/4; Dead Letter Queue pattern |
| `Day94_Resource_Book.md` | ✅ Done | NEW: Tree DP opens and closes same day — House Robber III, Binary Tree Maximum Path Sum (LC 337, 124) + extra Binary Tree Cameras (LC 968); Dynamic Programming closes entirely, 35/35; 6-subtype DP synthesis |
| `Day95_Resource_Book.md` | ✅ Done | NEW: Bit Manipulation opens from zero — binary, two's complement (derived), all seven operators, XOR proven — Single Number, Number of 1 Bits (LC 136, 191) + 0 extra; Service Discovery (Eureka, CoreDNS) |
| `Day96_Resource_Book.md` | ✅ Done | Power of Two, Counting Bits (LC 231, 338, fused with 1D DP) + 0 extra; ConfigMaps and Secrets |
| `Day97_Resource_Book.md` | ✅ Done | Missing Number, Single Number II (LC 268, 137, NEW: per-bit frequency counting) + extra Hamming Distance (LC 461); StatefulSets and DaemonSets |
| `Day98_Resource_Book.md` | ✅ Done | Single Number III, Sum of Two Integers (LC 260, 371, NEW: n&(-n)) + 0 extra; Bit Manipulation at 8/10; Week 14 Consolidation |
| `Week14_Interview_Questions.md` | ✅ Done | Every interview question from every Week 14 day, consolidated |
| `Day99_Resource_Book.md` | ✅ Done | NEW: Reverse Bits, Maximum XOR of Two Numbers (LC 190, 421, Bit Trie) + 0 extra; Bit Manipulation CLOSED 10/10, Tries CLOSED 7/7; Kubernetes fundamentals + HPA (flagged prerequisite insertion) |
| `Day100_Resource_Book.md` | ✅ Done | NEW: Segment Tree opens — Range Sum Query - Mutable (LC 307) + 0 extra; NEW: Singleton (DCL + `volatile`, Enum) |
| `Day101_Resource_Book.md` | ✅ Done | Count of Smaller Numbers After Self (LC 315, value-indexed Segment Tree) + 0 extra; Segment Trees CLOSED 2/2; Fenwick Tree extension; NEW: Factory Method, Builder |
| `Day102_Resource_Book.md` | ✅ Done | NEW: Merge Sort, Quicksort from scratch; 🔗 Kth Largest Element recap (LC 215, Wk8) + Quickselect full depth; full DSA pattern retrospective |
| `Day103_Resource_Book.md` | ✅ Done | NEW: SQL opens — subqueries, correlated subqueries, GROUP BY/HAVING, self-joins (LC 176, 184, 182, 197, 181), 5/10 SQL |
| `Day104_Resource_Book.md` | ✅ Done | NEW: SQL window functions — ROW_NUMBER/RANK/DENSE_RANK, LAG/LEAD, PARTITION BY, CASE (LC 177, 180, 262, 626, 185); SQL CLOSED 10/10 |
| `Week15_Interview_Questions.md` | ✅ Done | Every interview question from every Week 15 day, consolidated |
| `Day105_Resource_Book.md` | ✅ Done | No new content — full pattern self-check, Weekly Scorecard; **the entire DSA phase closes: 249 distinct DSA + 10 SQL = 259 total, Weeks 1–15** |
| `Day106_Resource_Book.md` | ✅ Done | NEW: 5-step LLD framework; GoF 3-category taxonomy named; Structural patterns — Adapter, Decorator (full impl. + JUnit), Facade, Proxy, Composite (sketch) |
| `Day107_Resource_Book.md` | ✅ Done | NEW: Behavioral patterns — Observer (full impl., built via TDD), Strategy, State, Command, Template Method (sketch); NEW: TDD Red-Green-Refactor; DSA revision: Course Schedule, Coin Change |
| `Day108_Resource_Book.md` | ✅ Done | LLD #1 Tic-Tac-Toe — framework applied live end-to-end, correctly needs no pattern; NEW: Coupling/Cohesion/Law of Demeter (applied retroactively, one violation found + fixed); DSA revision: Combination Sum |
| `Day109_Resource_Book.md` | ✅ Done | LLD #2 Vending Machine — State genuinely earned, zero state-dispatch conditionals; Mock Interview #1 (Tic-Tac-Toe); State vs. Strategy theory; DSA revision: Longest Substring Without Repeating Characters |
| `Day110_Resource_Book.md` | ✅ Done | LLD #3 Parking Lot Part 1 (single-threaded; Singleton considered and declined); NEW: Composition Over Inheritance (flawed hierarchy built and critiqued directly); DSA revision: Validate BST |
| `Day111_Resource_Book.md` | ✅ Done | LLD #3 Parking Lot Part 2 — concurrency: check-then-act race found/fixed, per-spot locking, `AtomicInteger`, 10-thread no-double-booking test; NEW: lock-granularity theory; DSA revision: Redundant Connection |
| `Day112_Resource_Book.md` | ✅ Done | LLD #4 Library Management System — Catalog/circulation provably separated; data-driven transition table contrasted against Day 109's State; Mock Interview #2 (Parking Lot + forced thread-safety escalation); Week 16 Consolidation |
| `Week16_Interview_Questions.md` | ✅ Done | Every interview question from every Week 16 day, consolidated (64 questions) |
| `Day113_Resource_Book.md` | ✅ Done | LLD #5 ATM Machine — State reapplied; Chain of Responsibility (11th pattern), naive dispenser shown broken then fixed via canDispense/commitDispense; greedy largest-first proven safe + Coin Change contrast; DSA revision: K Closest Points to Origin |
| `Day114_Resource_Book.md` | ✅ Done | LLD #6 Elevator System — State reapplied a third time; SCAN vs. LOOK distinguished, LOOK proven >2x more efficient than FIFO; Mock #3 (Machine Coding format) via a cold Token Bucket rebuild; DSA revision: Design Add and Search Words Data Structure |
| `Day115_Resource_Book.md` | ✅ Done | LLD #7 Splitwise — Strategy's first full system application (Equal/Exact/Percentage split); dual-heap greedy settlement proven ≤(n−1) with an honest NP-hard caveat (LC 465); LinkedIn Post 22; DSA revision: Generate Parentheses |
| `Day116_Resource_Book.md` | ✅ Done | LLD #8 BookMyShow Part 1 — single-threaded design, seat status scoped per-show; the check-then-act race traced precisely, deliberately not yet fixed; DSA revision: Clone Graph |
| `Day117_Resource_Book.md` | ✅ Done | LLD #8 BookMyShow Part 2 — pessimistic (in-memory + SELECT FOR UPDATE) and optimistic (version-based CAS) locking, both proven deterministic via a CountDownLatch-gated 10-thread test; Mock #4; DSA revision: Longest Increasing Subsequence |
| `Day118_Resource_Book.md` | ✅ Done | LLD #9 Food Delivery System — Strategy's second full application (stateless strategies sharpen the who-chooses test); System Design's 5-step framework previewed only; all 7 applications submitted; DSA revision: Edit Distance |
| `Day119_Resource_Book.md` | ✅ Done | LLD #10 Hotel Booking System — all ten LLD systems self-checked; date-range availability via Week 4's interval-overlap logic; concurrency model compared against BookMyShow via Week 9's replication trade-offs; Mock #5; LLD phase closed; Week 17 Consolidation |
| `Week17_Interview_Questions.md` | ✅ Done | Every interview question from every Week 17 day, consolidated (65 questions) |

| `Day120_Resource_Book.md` | ✅ Done | HLD #1 URL Shortener — System Design 5-step framework taught in full for the first time (Day 118's preview superseded); Base62 encoder/decoder; KGS uniqueness via `FOR UPDATE SKIP LOCKED` |
| `Day121_Resource_Book.md` | ✅ Done | HLD #2 Rate Limiter, Formalized — Leaking Bucket, Fixed Window (2x boundary flaw proven), Sliding Window Log; Token Bucket (Week 10) made genuinely distributed via Redis + Lua atomicity |
| `Day122_Resource_Book.md` | ✅ Done | HLD #3 Notification System — per-channel queues, Template Service, best-effort Redis dedup contrasted against payment idempotency (deferred to Week 19); HLD Mock #1 (URL Shortener, step-narration focus) |
| `Day123_Resource_Book.md` | ✅ Done | HLD #4 Distributed Cache — Redis's full picture; Thundering Herd + two mitigations; Cache-Aside (`@Cacheable`/`@CacheEvict`); Consistent Hashing (Week 9) recapped and applied |
| `Day124_Resource_Book.md` | ✅ Done | HLD #5 Distributed ID Generation — UUID's sortability cost proven; Twitter Snowflake built and traced by hand; DB ID range allocation connected back to Day 120's KGS |
| `Day125_Resource_Book.md` | ✅ Done | HLD #6 Distributed Consensus — Quorum's majority-overlap proof; split-brain tied to CAP (Week 9); Raft's three parts; etcd's role under Kubernetes (Week 15) revealed; HLD Mock #2 (Rate Limiter, "why not X" defense focus) |
| `Day126_Resource_Book.md` | ✅ Done | HLD #7 BookMyShow at Scale — city-sharding; seat-hold via Redis TTL + Lua; proactive cache warming as a known-timing extension of Day 123's Thundering Herd; Self-Check LC 40 (see Day 125-vs-126 discrepancy note); Week 18 Consolidation |
| `Week18_Interview_Questions.md` | ✅ Done | Every interview question from every Week 18 day, consolidated (100 questions) |

| `Day127_Resource_Book.md` | ✅ Done | HLD #8 WhatsApp/Chat System — Persistent WebSockets; connection-management registry; message routing via Kafka (applied); wide-column store for offline storage; media sharing deliberately deferred to Day 133 |
| `Day128_Resource_Book.md` | ✅ Done | HLD #9 Uber Driver Location Tracking — Geohashing, Quadtrees, Redis Geo; CAP theorem correctly scoped away from a single-machine cache; HLD Mock #3 (Distributed Cache, Uber format) |
| `Day129_Resource_Book.md` | ✅ Done | HLD #10 Netflix/Video Streaming (transcoding, adaptive bitrate, CDN as Cache-Aside at the edge) + HLD #11 Instagram Feed (push/pull/hybrid fan-out resolves "the celebrity problem"); LinkedIn Post 25 |
| `Day130_Resource_Book.md` | ✅ Done | HLD #12 Payment System — idempotency keys and SETNX-based distributed locks in full (unsafe-release bug traced), double-entry ledger; closes the loop deferred since Day 121 |
| `Day131_Resource_Book.md` | ✅ Done | HLD #13 Distributed Job Scheduler — SQS Visibility Timeout, Quartz cron + misfire handling, at-least-once vs. exactly-once; HLD Mock #4 (Payment System, race-condition focus); DSA revision: Accounts Merge, Longest Common Subsequence |
| `Day132_Resource_Book.md` | ✅ Done | HLD #14 Leaderboard (Skip List built from scratch, Redis Cluster distinguished from Consistent Hashing, composite-score tie-handling) + HLD #15 Search Autocomplete (distributed, asynchronously-rebuilt Trie) |
| `Day133_Resource_Book.md` | ✅ Done | HLD #16 Google Drive/Object Storage — chunking (fixed-size vs. content-defined, proven via worked example), content-addressable dedup, sync conflicts; all 16 HLD systems self-checked; HLD Mock #5 (candidate's choice); HLD phase closed; Week 19 Consolidation |
| `Week19_Interview_Questions.md` | ✅ Done | Every interview question from every Week 19 day, consolidated (100 questions) |

| `Day134_Resource_Book.md` | ✅ Done | Load testing fundamentals (p50/p95/p99 proven), k6; real Kubernetes Deployment/Service manifests for all 4 modules; resource requests vs. limits |
| `Day135_Resource_Book.md` | ✅ Done | Helm chart templating Day 134's manifests; Kubernetes Ingress disambiguated from Spring Cloud Gateway (Week 10); Minikube |
| `Day136_Resource_Book.md` | ✅ Done | Three environment-specific values files; ConfigMap vs. Secret (base64 ≠ encryption, proven); "build once, deploy many times" |
| `Day137_Resource_Book.md` | ✅ Done | Micrometer Tracing + Zipkin; tracing's Kafka blind spot named; Stripe-format API design (versioning, cursor pagination, idempotency, error contract) |
| `Day138_Resource_Book.md` | ✅ Done | Three chaos experiments (Payment-kill, latency-injection, connection-pool saturation); light service mesh; `BoundedBlockingQueue<T>` hand-rolled and proven under contention |
| `Day139_Resource_Book.md` | ✅ Done | Coverage-vs-correctness proven; JaCoCo as an enforced gate; GitHub Actions pipeline traced end to end; liveness/readiness probes; README finalized |
| `Day140_Resource_Book.md` | ✅ Done | Self-check (Saga vs. 2PC, Gateway rate limiting, K8s/Helm vs. docker-compose); bottleneck analysis from real data; platform's own 10x/100x at-scale analysis; Week 20 Consolidation; capstone complete |
| `Week20_Interview_Questions.md` | ✅ Done | Every interview question from every Week 20 day, consolidated (66 questions) |
| `Day141_Resource_Book.md` | ✅ Done | STAR framework, rigorously; 6-competency map; Stories 1–2 (Ownership, Conflict & Disagreement); UPI's 2PC application and ambiguous-transaction resolution |
| `Day142_Resource_Book.md` | ✅ Done | Stories 3–4 (Failure & Learning, Ownership); High Availability (Active-Active/Passive, health-check failover, chaos-as-verification) |
| `Day143_Resource_Book.md` | ✅ Done | Stories 5–6 (Failure & Learning, Navigating Ambiguity — Story 6 deliberately steered to close the map's one gap); High-Volume/Low-Margin Systems; `lld-java` README (10-system table) |
| `Day144_Resource_Book.md` | ✅ Done | Stories 7–8 (Cross-Functional Pushback, Technical Leadership — all 8 stories complete); Multi-Tenant SaaS at Scale (isolation spectrum, noisy neighbor, RBAC); `todo-api` README |
| `Day145_Resource_Book.md` | ✅ Done | Full 8-story competency-coverage audit; Googleyness, Databricks' 6 principles, Atlassian's 5 values, each mapped against real stories; 6th LLD mock |
| `Day146_Resource_Book.md` | ✅ Done | Cold-recall self-check (DSA/LLD/HLD candidate menus); resume finalized — DSA count reconciled to 249, plan's own "213" identified as unverifiable and not used; 6th System Design mock |
| `Day147_Resource_Book.md` | ✅ Done | GitHub portfolio — 5 repos pinned in reasoned order, honest README framing; referral mechanism and follow-up cadence; application status reviewed across 7 companies |
| `Day148_Resource_Book.md` | ✅ Done | Full unaided self-check; Weekly Scorecard with fully reconciled series totals; Week 21 Consolidation doubling as full-series consolidation; technical preparation phase closes |
| `Week21_Interview_Questions.md` | ✅ Done | Every interview question from every Week 21 day, consolidated (47 questions) — the series' final interview bank |

These one hundred and nineteen Resource Books and all seventeen interview banks are the full archive — kept for your own reading and review. **They are not needed to generate Week 18 or any later week; this map is built to stand in for them.**

> ⚠️ **A discrepancy found while extending this section for Week 15, flagged rather than silently resolved either way:** the file-index one-line summaries above for `Day95_Resource_Book.md`, `Day96_Resource_Book.md`, and `Day97_Resource_Book.md` mention theory topics — "Service Discovery (Eureka, CoreDNS)," "ConfigMaps and Secrets," "StatefulSets and DaemonSets" — that read as Kubernetes-adjacent, but are **not corroborated anywhere else in this document**: not in the detailed Concept-Dependency Map's Day 95–98 entries (Bit Manipulation only), not in the Terminology section's Week 14 entries (Bit Manipulation only), and not in "What Week 14 Leaves You Knowing Cold" (Bit Manipulation only) — despite that same Terminology section demonstrably tracking non-DSA infrastructure topics for other weeks when they were genuinely taught (Week 12's Sharding, Two-Phase Commit, and Kafka Streams entries, for comparison). Given the conflict, Day 99's Kubernetes material was built **from zero, treating Pod/ReplicaSet/Deployment/Service and the reconcile-loop model as genuinely new** — the lower-risk assumption regardless of which side of this discrepancy is correct: if these file-index entries are accurate and this material really was covered in Weeks 13–14, Day 99's treatment is a redundant but harmless refresher directly feeding into HPA; if they're stale leftovers from an earlier plan draft (the more likely explanation, given the lack of corroboration anywhere else), Day 99's treatment is the first and only correct exposure. **This is worth a direct human check** — if Service Discovery/CoreDNS, ConfigMaps/Secrets, or StatefulSets/DaemonSets genuinely were taught in Weeks 13–14, this row-index text is accurate and this note can be removed; if not, these three file-index entries should be corrected to whatever those days' theory blocks actually covered.

> 📌 **Count updated after Week 18, addendum only — the sentence above is left as originally written, describing the archive as it stood through Week 17.** Add Week 18's 7 new Resource Books and 1 new interview bank: the archive now stands at **126 Resource Books and 18 interview banks.**

> 📌 **Count updated after Week 19, a second addendum, building on the one immediately above rather than editing it.** Add Week 19's 7 new Resource Books and 1 new interview bank: the archive now stands at **133 Resource Books and 19 interview banks.** This closes the LLD + HLD design-interview arc's own resource-book count at Days 106–133 (28 Resource Books, 4 interview banks) on top of the DSA phase's 105.

> 📌 **Count updated after Week 20, a third addendum, building on the two immediately above rather than editing either.** Add Week 20's 7 new Resource Books and 1 new interview bank: the archive now stands at **140 Resource Books and 20 interview banks.** This closes the Capstone Hardening phase's own resource-book count at Days 134–140 (7 Resource Books, 1 interview bank) — the first phase since the LLD/HLD transition to run exactly one week rather than two.

> 📌 **Count updated after Week 21, a fourth and final addendum, building on the three immediately above rather than editing any of them.** Add Week 21's 8 new Resource Books (Days 141–148, an 8-day week rather than 7) and 1 new interview bank: the archive stands, finally, at **148 Resource Books and 21 interview banks.** This closes the entire series. There is no Week 22 addendum to follow this one.

---


## Series Complete — Week 21 Generated, No Week 22 to Hand Off To

This document's active handoff role ends here. `Week_21_Revised.md` is the series' last plan file — Day 148's own text closes the technical preparation phase explicitly, and there is no `Week_22_Revised.md` for this map to prepare anyone for. What follows is this update's record of what actually happened when Week 21 was generated, written the same way every other week's update has been: additively, with discrepancies flagged rather than silently fixed.

**What Week 21 actually added, in full:** zero new DSA problems, zero new LLD or HLD systems — confirmed directly against `Week_21_Revised.md`'s complete content, exactly as the pre-check below predicted. In their place: the entire behavioral/STAR track built from zero (framework, six-competency map, all eight stories); four new pieces of domain knowledge (UPI's 2PC application and its ambiguous-transaction resolution, High Availability, high-volume/low-margin economics, Multi-Tenant SaaS); a full technical retention self-check; resume finalization; and portfolio/job-search closure. Full detail in the new "Week 21 Additions" subsections added below to the Concept-Dependency Map and the Terminology section, and in the new "What Week 21 Leaves You Knowing Cold" entry.

**The pre-check below was correct, and this generation pass confirms it retrospectively** — no DSA collision, no LLD/HLD collision. It also correctly identified that Two-Phase Commit needed to be a recap rather than fresh teaching (Week 12, Day 79); Week 21's own Day 141 recapped it accordingly rather than re-teaching it, and additionally recapped Saga (Week 10, Day 70) and reused Idempotency Keys (Week 19, Day 130) rather than re-deriving either.

**Two small corrections to this section's own text, and the "How to Use" section below, flagged rather than edited in place, per this document's own convention:**
1. Both sections below refer to the "213 problems" language and the resume-finalization task as landing on **"Day 144"** of `Week_21_Revised.md`. Checked directly against the plan file: both actually appear on **Day 146**, not Day 144 (Day 144 covers Stories 7–8 and Multi-Tenant SaaS; the resume pass and both "213" mentions are on Day 146 and Day 148). The substance of the flag — that "213" doesn't reconcile against this map's verified 197/249/259 — is otherwise accurate and was carried into Day 146's Resource Book in full, with clear guidance on which verified figure to actually use.
2. A different, earlier part of this document (the "What Week 20 Leaves You Knowing Cold" entry, above) characterizes `Week_21_Revised.md` as including "negotiation preparation" among its content. Checked directly against the plan: negotiation is mentioned exactly once, in Day 148's closing paragraph, explicitly as part of the **not-yet-done** interview-execution phase that begins after this series ends — not as content Week 21 itself teaches. Worth knowing if anyone reads that earlier characterization at face value.

One more, smaller and not really worth a numbered flag but noted for completeness: the "Known Overlap With Week 21" section's own summary line below states the DSA phase's closing count as "197 required + 53 extra = 249 distinct." 197 + 53 is 250, not 249 — the arithmetic doesn't quite close. The two headline numbers themselves (197 and 249) are independently correct, directly re-verified against the Running Totals section's own week-by-week additions during this pass; the "53" specifically should read 52 (249 − 197). This document's Day 146 Resource Book uses 197 and 249 directly rather than repeating the "53" figure, so nothing downstream was affected by it.

**Archive count as of this update:** 148 Resource Books (Days 1–148) and 21 consolidated interview question banks (Weeks 1–21). Full row-by-row detail in the File Index update below.

---

## How to Use This Document for Week 21 (and Beyond)

Read the Concept-Dependency Map's "Week 20 Additions" subsection (near its end) and the new "Platform Operations / Infrastructure Deliverable Inventory" table above for exactly what Week 20 built and where each piece was first taught. There is no DSA Revision Log entry for Week 20 and no new HLD/LLD system to check against — the only overlap-relevant items Week 20 leaves behind are infrastructure/operational, tracked in that new table rather than the DSA or HLD tables. Given `Week_21_Revised.md` (already read in full while extending this map for Week 20 — see "Known Overlap With Week 21," above) runs no new DSA problems and designs no new system either, this is the first consecutive pair of weeks in the entire series where the standard "check the next week's plan for overlap" step surfaces nothing to check in either direction, in either table.

Five things worth carrying forward explicitly, each already detailed above with its own reasoning:

1. **The Capstone Hardening phase is fully closed as of Day 140** — seven deliverables, one week, no mock interview (a genuinely different shape than every phase since Week 16, each of which ran two weeks and multiple mocks). `Week_21_Revised.md`'s own content treats this as settled, completed fact — Day 141's `docs/architecture.md` finalization, Day 142's chaos-engineering citation, and Day 148's closing scorecard all build directly on top of it rather than re-describing any of it. If Week 21 references Kubernetes, Helm, tracing, chaos engineering, the service mesh, or the hand-rolled queue, it should be by day number (134–140), as an application of or reflection on something already built, not new teaching.

2. **Two discrepancies flagged during Week 20's generation, neither resolved, both still open for a direct human check:** `Week_20_Revised.md`'s claim that `todo-api` and "the original plan's `order-management-api`" both received real Kubernetes deployments earlier in the series (contradicted by this map's own history — `todo-api` is Docker/Compose-only, frozen since Day 62; `order-management-api` never existed as a separate repo, per `Week_21_Revised.md`'s own note that it was absorbed into the platform back in Week 9); and the related claim, resurfacing on Day 139, that readiness/liveness probes were "already configured back in the DSA phase for `todo-api`." Both were treated as the lower-risk case (build as genuinely new material) regardless of which side is correct, following the exact precedent the Day 95–97 Kubernetes-adjacent discrepancy already set during Week 15's extension. Worth a direct human check against the actual `todo-api` repository if anyone wants this fully settled rather than flagged.

3. **A new numeric discrepancy, found this pass rather than inherited:** `Week_21_Revised.md` states the DSA curriculum as **"213 problems"** twice (Days 144, 148) — a third figure alongside this map's authoritative **197 required (249 distinct including extras, 259 with SQL)** and `Week_15_Revised.md`'s own already-flagged **"203."** Full detail in "Known Overlap With Week 21," above. Not expected to block Week 21's own generation, but worth resolving before Day 144's actual resume language gets finalized, since that document may reach a real interviewer.

4. **Standing item, repeated from every prior week's version of this section since Week 15 — still not backfilled, still not blocking anything:** the 197-vs-203 required-only-DSA reconciliation (Week 15's Running Totals entry, Day 105's own Career Block note) — this map's row-by-row table (197) remains held authoritative. Now joined by item 3, above, rather than resolved by it.

5. **Standing convention, reconfirmed a second time:** where a discrepancy or imprecision is found, flag it in a new note rather than editing historical content in place. This extension followed that convention twice over (items 2 and 3, above) rather than silently resolving either one or treating the second as a reason to handle the first differently.

**What Week 20 leaves as available, reflexive building blocks, not material to re-teach:** real, working Kubernetes manifests and a Helm chart for the platform, across three environments; a live distributed-tracing pipeline (with its Kafka blind spot known and named); three chaos experiments' worth of evidence about how the platform actually fails under real conditions, not how it's configured to fail; a light but real service-mesh demonstration; a hand-rolled, formally-tested `BoundedBlockingQueue<T>`; a CI/CD pipeline that enforces its own quality gate; and a formal, evidence-backed 10x/100x analysis of the platform's own architecture. Cite these by day number the way this week cited Weeks 6, 7, 8, 9, 10, 11, 12, 15, 17, 18, and 19 — Week 21's own generation should extend this document exactly as this update did: append only, flag discrepancies in a new note rather than editing historical content in place, never edit or shorten what's already here.

---


## How to Use This Document for Week 20 (and Beyond)

Read the Concept-Dependency Map's "Week 19 Additions" subsection (near its end) and the "System Design / HLD Deliverable Inventory" table above for exactly what Week 19 built and where each piece was first taught — HLD Mock subjects and formats included, since all five HLD mocks (and all five LLD mocks before them) are now spent; if Week 20 runs a further mock of any kind, it has ten prior subject/format pairings to check against, not just Week 19's three. The DSA Revision Log — Week 19 entries above (LC 721, LC 1143) are the only DSA items Week 19 touched; if Week 20's own plan calls for a further cold-revision pick, check it against the full cumulative Problem Inventory table, not just this week's small addition to it.

Four things worth carrying forward explicitly, each already detailed above with its own reasoning:

- **The HLD phase is fully closed as of Day 133** — sixteen systems, five mocks. `Week_20_Revised.md`'s own content (Kubernetes, Helm, distributed tracing, chaos engineering, service mesh, CI/CD) introduces no new HLD system — if Week 20 references one of the sixteen, it should be by name, as an application of something already taught, not new system-design teaching.
- **Every deferred thread this map has ever flagged is now closed.** SETNX-based distributed locks and idempotency keys (Day 130); Sorted Sets' Skip-List mechanism (Day 132, after three operational-only uses on Days 123, 128, and 131). Nothing is left deliberately incomplete heading into Week 20.
- **`Week_20_Revised.md`'s own Day 140 explicitly assumes** every one of the sixteen HLD systems from Weeks 18–19 received a real estimation pass and bottleneck analysis, not just a design sketch — confirmed true for all nine of Week 19's systems specifically (see each day's own Estimation and Bottlenecks sections), continuing Week 18's own three. This is not a formality: Week 20's capstone-hardening exercise only makes sense if this genuinely holds.
- **Standing convention, reconfirmed**: where a discrepancy or imprecision is found, flag it in a new note rather than editing historical content in place. This map corrected its own stale title and companion-line (frozen at "Through Week 16" across two subsequent extensions) during this same Week 19 update — see the flag immediately under the title — following the identical treatment every previous in-place discrepancy on this map has received, rather than being made an exception.

---


## How to Use This Document for Week 19 (and Beyond)

Read the Concept-Dependency Map's "Week 18 Additions" subsection (near its end) and the "System Design / HLD Deliverable Inventory" table above for exactly what Week 18 built and where each piece was first taught — HLD Mock subjects and focuses included, since Week 19 likely runs further mocks and shouldn't reuse a subject/focus pairing already exercised without a reason to. The DSA Revision Log — Week 18 entry above (LC 40) is the only DSA item Week 18 touched; if Week 19's own plan calls for a further cold-revision pick, check it against the full cumulative Problem Inventory table, not just this week's small addition to it.

Three things worth carrying forward explicitly, each already detailed above with its own reasoning:

1. **The Day 125-vs-126 self-check discrepancy** (DSA Revision Log — Week 18, above) — this map's own pre-Week-18 notes said "Day 125"; the actual plan file placed it on Day 126. If `Week_19_Revised.md`'s day numbers ever appear to drift from what a prior week's forward-looking note assumed, follow the actual plan file, exactly as Week 18's generation did, and flag the discrepancy rather than silently picking a side — this map's own established convention (Week 16's Days 108–109, the Day 95–97 Kubernetes question, and now this) is to flag, not resolve, when the two sources disagree.

2. **Two deliberate deferrals**, both explained in full in the HLD Deliverable Inventory table above: Redis's full theoretical picture landed Day 123, not Day 121 (Day 121 borrowed only what it needed); and SETNX-based distributed locks / idempotency keys were kept out of Week 18 entirely, twice, in favor of reusing Day 121's Lua-atomicity pattern — both specifically so Week 19, Day 130's actual new content stays genuinely new rather than a rehash. Week 19's generation should teach the full SETNX-based lock and idempotency-key mechanisms at Day 130 as first-time material, not a recap — Week 18 never gave them a shallow version to build on.

3. **`Week_19_Revised.md` contains no LeetCode-numbered problems** — confirmed directly, in full, during Week 18's generation (see "Known Overlap With Week 19," above). If this holds when Week 19 is actually generated, no DSA-collision check against Week 20 (or beyond) is needed for problems, only for HLD topics/systems, the same way this week's check against Week 19 worked.

**What Week 18 leaves as available, reflexive building blocks, not material to re-teach:** the HLD 5-step framework (three full applications, one defended live under follow-up); all four rate-limiting algorithms; Redis's complete picture including Thundering Herd, Cache-Aside, and the self-invocation caveat (now established a third time — Spring AOP, Resilience4j, Redis caching); Consistent Hashing and Sharding Strategies, each applied twice across the series; Snowflake ID generation; and quorum/Raft/split-brain, with etcd's concrete role under Kubernetes now named. Cite these by day number the way this week cited Weeks 9, 10, 12, 14, and 17 — Week 19's own generation should extend this document exactly as this update did: append only, flag discrepancies in place, never edit or shorten what's already here.

## How to Use This Document for Week 18 (and Beyond)

To generate Week 18's resource books in a fresh chat, upload exactly four things:
1. The current version of the generation prompt
2. `Week_18_Revised.md`
3. This file (`00_Curriculum_Map.md`)
4. `Week_19_Revised.md`, if available — used only to keep Week 18's own DSA revision picks (Day 125's Backtracking-or-Trie self-check, specifically) and any extra-practice picks from colliding with Week 19's required problems, the same overlap check now run seventeen times (Week 1 → 2 through Week 16 → 17, plus Week 17 → 18, confirmed clean). No `Week_19_Revised.md` was available at the time this map was updated.

**No flagged collisions carry into Week 18** — the Week 18 Overlap section above confirms a clean check: Week 17 added zero extra-practice DSA problems and zero extra LLD systems, so there's nothing to check for collision in that direction, and none of Week 18's own required content introduces an LC number already sitting in either week's DSA Revision Log. **The Backtracking/Graph/DP revision-pick risk Week 17 carried is now fully resolved** — Day 115 drew Generate Parentheses (LC 22), Day 116 drew Clone Graph (LC 133), and Day 117 drew Longest Increasing Subsequence (LC 300), all confirmed distinct from Week 16's three (Combination Sum, Course Schedule, Coin Change). **A new, analogous constraint takes its place:** Week 18's own Day 125 self-check names "one Backtracking or Trie problem, cold" without specifying which — whoever generates Week 18 must pick a problem that is neither LC 22 (this week's Backtracking pick) nor LC 211 (this week's Trie pick) nor LC 39 (Week 16's Backtracking pick), regardless of which of the two patterns Day 125 ends up drawing from. What Week 18 inherits directly otherwise: all ten LLD systems and eleven patterns (Days 106–119) as the explicit, unrestated foundation for Detailed Design steps that will need this same design judgment at a larger scale; the System Design framework's five steps held only as a preview shape (Day 118), with Day 120 doing the actual first full-depth teaching, not a re-explanation of something already learned; and Token Bucket (Week 10, Day 68, recapped Day 114) solid enough to anchor Day 121's fuller rate-limiting landscape without re-deriving it.

**Four open items worth carrying forward explicitly, all flagged above rather than silently resolved:**
- **The Day 95–97 file-index Kubernetes-adjacent discrepancy** (see the note directly above the File Index's Week 15 rows) — still unresolved, still worth a direct human check against what those days' theory blocks actually covered.
- **The 197-vs-203 required-only-DSA numeric reconciliation** (Week 15's Running Totals entry) — this map's row-by-row table (197) remains held authoritative; not expected to block anything, but worth being aware of if a future plan file states a cumulative DSA count that should trace back to 197, not 203.
- **A new discrepancy found while extending this section for Week 17, flagged rather than silently corrected:** the closing paragraph of "What Week 16 Leaves You Knowing Cold" (above) mislabels two of Week 17's own days — it names "BookMyShow (Day 116), Food Delivery (Day 117, Strategy's second full application), and Hotel Booking (Day 118)," but `Week_17_Revised.md` itself places Food Delivery on **Day 118** (Day 117 is actually BookMyShow Part 2/Mock #4) and Hotel Booking on **Day 119**, not Day 118 — not edited in place, per this map's own extend-don't-edit rule.
- **The Backtracking/Trie revision-pick collision risk for Week 18's Day 125** (new this extension, see the Week 18 Overlap section above) — Week 18's own generation must select a problem outside {LC 22, LC 211, LC 39} for this slot; not a blocking issue, but an active constraint on a choice that hasn't been made yet.

When Week 18 is generated, its own resource-book generation should extend this same file — adding Week 18's own concepts to the dependency map, opening whatever new HLD/System-Design-specific inventory table its content actually needs (this map's own Week 16 precedent — reasoning through the right shape explicitly rather than forcing new content into a table built for something else — is the model to follow, not a specific table structure to copy blindly), logging exactly which pattern and LC number Day 125's revision draws from, updating "what Week 18 leaves you knowing cold," and re-running the overlap check against Week 19. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended seventeen times (Week 1 → Week 2 through Week 16 → Week 17) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues.

*(Everything from this point down is kept exactly as originally written, per this document's own extend-don't-edit rule — each paragraph below describes a generation step that has since happened; the current, up-to-date instructions are the paragraphs directly above.)*

## How to Use This Document for Week 17 (and Beyond)

To generate Week 17's resource books in a fresh chat, upload exactly four things:
1. The current version of the generation prompt
2. `Week_17_Revised.md`
3. This file (`00_Curriculum_Map.md`)
4. `Week_18_Revised.md`, if available — used only to keep Week 17's own DSA revision picks and any extra-practice picks from colliding with Week 18's required problems, the same overlap check now run sixteen times (Week 1 → 2 through Week 14 → 15, plus Week 15 → 16 and Week 16 → 17, both confirmed clean). No `Week_18_Revised.md` was available at the time this map was updated.

**No flagged collisions carry into Week 17** — the Week 17 Overlap section above confirms a clean check: Week 16 added zero extra-practice DSA problems and zero extra LLD systems, so there's nothing to check for collision in that direction, and none of Week 17's own required content introduces an LC number already sitting in Week 16's DSA Revision Log. **One thing does carry forward as an active constraint, not just a clean bill of health:** Week 17 carries its own DSA revision blocks in Backtracking (Day 115), Graph (Day 116), and DP (Day 117) — the same three patterns Week 16 already drew a cold-solve pick from. Whoever generates Week 17 **must pick different specific problems** than Week 16's three (Combination Sum, Course Schedule, Coin Change) for these slots — the DSA Revision Log above has the full list of everything already picked, across all patterns, so far. What Week 17 inherits directly otherwise: all ten Structural and Behavioral patterns (Days 106–107) and the 5-step LLD framework, applied live four times across Days 108–112, are the explicit, unrestated foundation — Week 17's ATM (Day 113) introduces Chain of Responsibility as an eleventh pattern, and Splitwise (Day 115) and Food Delivery (Day 118) are Strategy's first two full system-level applications, both forward-referenced from Day 107 and assumed to land on genuinely solid ground, not a re-derivation.

**Three open items worth carrying forward explicitly, all flagged above rather than silently resolved:**
- **The Day 95–97 file-index Kubernetes-adjacent discrepancy** (see the note directly above the File Index's Week 15 rows) — still unresolved, still worth a direct human check against what those days' theory blocks actually covered.
- **The 197-vs-203 required-only-DSA numeric reconciliation** (Week 15's Running Totals entry) — this map's row-by-row table (197) remains held authoritative; not expected to block anything, but worth being aware of if a future plan file states a cumulative DSA count that should trace back to 197, not 203.
- **The Backtracking/Graph/DP revision-pick collision risk** (new this extension, see the Week 17 Overlap section above) — Week 17's own generation must select different specific problems than Week 16's three for these three patterns' revision slots; not a blocking issue, but an active constraint on a choice that hasn't been made yet.

When Week 17 is generated, its own resource-book generation should extend this same file — adding Week 17's concepts (Chain of Responsibility; Strategy's two full system applications) to the dependency map, appending its six new LLD systems to the LLD System / Design Exercise Inventory above, logging exactly which DSA revision problems Days 113–118 actually drew from (explicitly confirming the Backtracking/Graph/DP picks differ from Week 16's three), updating "what Week 17 leaves you knowing cold," and re-running the overlap check against Week 18. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended sixteen times (Week 1 → Week 2 through Week 15 → Week 16) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues.

*(Everything from this point down is kept exactly as originally written, per this document's own extend-don't-edit rule — each paragraph below describes a generation step that has since happened; the current, up-to-date instructions are the paragraphs directly above.)*

## How to Use This Document for Week 16 (and Beyond)

To generate Week 16's resource books in a fresh chat, upload exactly four things:
1. The current version of the generation prompt
2. `Week_16_Revised.md`
3. This file (`00_Curriculum_Map.md`)
4. `Week_17_Revised.md`, if available — used only to keep Week 16's own extra-practice picks (if any) from colliding with Week 17's required problems, the same overlap check now run fifteen times (Week 1 → 2 through Week 14 → 15, plus Week 15 → 16, confirmed clean). No `Week_17_Revised.md` was available at the time this map was updated.

**No flagged collisions carry into Week 16** — the Week 16 Overlap section above confirms a clean check: Week 15 added zero extra-practice problems anywhere, so there is nothing to check for collision in that direction, and none of Week 16's own content introduces new required LeetCode problems in any of Week 15's five patterns (Bit Manipulation, Tries, Segment Trees, Sorting, SQL) — Week 16's DSA-adjacent material is explicitly revision of already-closed patterns. **The DSA phase of this series is complete** — Week 16 is the first week whose primary content is not a new DSA pattern at all, so the usual "does this week open/close/continue a pattern, and does that affect the extra-practice-timing precedent" analysis does not apply the way it has for every prior week. What Week 16 does inherit directly: all three Creational design patterns (Singleton, Factory Method, Builder), taught in full Week 15 Days 100–101, are the explicit, unrestated foundation for Week 16's Structural patterns (Adapter, Decorator, Facade, Proxy, Composite, Day 106) and Behavioral patterns (Day 107) — neither re-derives "private constructor plus static accessor" or "delegate creation to a subclass," both assumed fluent. Week 16's revision blocks (Days 108–109) draw cold-solve problems from Sliding Window, Backtracking, Trees, Union-Find/Dijkstra's, and Graphs/DP — all closed patterns this map fully indexes; no new teaching is needed for any of them, only citation if a specific problem's original mechanism needs to be pointed back to.

**Two open items worth carrying forward explicitly, both flagged above rather than silently resolved:**
- **The Day 95–97 file-index Kubernetes-adjacent discrepancy** (see the note directly above the File Index's closing line) — worth a direct human check against what those days' theory blocks actually covered, since this map could not resolve it definitively from internal evidence alone.
- **The 197-vs-203 required-only-DSA numeric reconciliation** (Week 15's Running Totals entry, and Day 105's own Career Block note) — this map's row-by-row table (197) is held authoritative over `Week_15_Revised.md`'s own stated figure (203), consistent with this document's established handling of the same class of drift; not expected to block Week 16's generation, but worth being aware of if a future week's own plan file states a cumulative DSA count that should trace back to 197, not 203.

When Week 16 is generated, its own resource-book generation should extend this same file — adding Week 16's concepts (the Structural and Behavioral design patterns, and whatever LLD systems Week 16 builds) to the dependency map, noting which specific revision problems Days 108–109 actually drew from (so a future week can confirm they don't silently re-select the same cold-solve problems again), updating "what Week 16 leaves you knowing cold," and re-running the overlap check against Week 17. Since Week 16 is the first LLD-focused week, whoever generates it should also decide, explicitly, how this map's own DSA-oriented structure (the Concept-Dependency Map, the Problem Inventory table) should adapt for a phase of the series where "problems" may mean LLD systems and mock-interview scenarios rather than LeetCode numbers — that's a real, open design question for this document itself, not something to resolve silently by pattern-matching the DSA-phase conventions onto content they weren't built for. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended fourteen times (Week 1 → Week 2 through Week 14 → Week 15) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues.

*(Everything from this point down is kept exactly as originally written, per this document's own extend-don't-edit rule — each paragraph below describes a generation step that has since happened; the current, up-to-date instructions are the paragraphs directly above.)*

To generate Week 15's resource books in a fresh chat, upload exactly three things:
1. The current version of the generation prompt
2. `Week_15_Revised.md`
3. This file (`00_Curriculum_Map.md`)

Optionally, include a `Week_16_Revised.md` too, if available — it would only be used to keep Week 15's own extra-practice picks from colliding with Week 16's required problems, the same overlap check now run fourteen times (Week 1 → 2 through Week 13 → 14, plus Week 14 → 15, confirmed clean — both of Week 14's extras checked directly against Week 15's own required list before being added). No `Week_16_Revised.md` was available at the time this map was updated.

**No flagged collisions carry into Week 15** — the Week 15 Overlap section above confirms a clean check against the full inventory (245 distinct problems, Weeks 1–14), and neither of Week 14's two extra-practice picks (LC 968, LC 461) collides with Week 15's required list. **As with Week 14's own transition, there is no lingering required-problem collision to stay aware of this time** — Week 14 closed with zero recaps needed in either direction, the second consecutive week in the series to do so (Diameter of Binary Tree, LC 543, resurfaced only as a citation, not a required-problem collision); every one of Week 14's 14 required problems was confirmed new before being taught. Week 15 has its own internal sequencing note worth re-reading before generating: it closes Bit Manipulation at Day 99 (Reverse Bits, Maximum XOR of Two Numbers in an Array) — the latter specifically needs BOTH Week 14's bit-manipulation mechanics AND Week 9's Trie structure, combined into a Bit Trie, a two-prerequisite-chain dependency this map's Week 14 section flags explicitly; Week 9's own closing note already deferred this exact problem for this exact pairing, so nothing here is new information, only now due. Segment Trees then open fresh (Day 100) — per this series' own precedent, its own true opening day should receive no extras, the same treatment every prior pattern's genuine opening day has received (Trees Day 46, Heaps Day 54, Tries Day 59, Backtracking Day 61, Graphs Day 68, Union-Find Day 74, Dijkstra's Day 78, Dynamic Programming Day 81, Bit Manipulation Day 95). Day 102's Kth Largest Element in an Array (LC 215) is a required problem that already appears in the inventory (Week 8, Day 55, via a min-heap) — not a collision needing resolution, since `Week_15_Revised.md`'s own plan is already aware of it and frames the day around teaching Quickselect as a second, genuinely distinct approach; see the Week 15 Overlap section above for how to handle it (short recap for the problem itself, full depth reserved for the new technique).

When Week 15 is generated, its own resource-book generation should extend this same file — adding Week 15's concepts to the dependency map, appending Week 15's problems (required and extra) to the inventory table above, updating "what Week 15 leaves you knowing cold," and re-running the overlap check against Week 16. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended thirteen times (Week 1 → Week 2 through Week 13 → Week 14) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues.

*(Everything from this point down is kept exactly as originally written, per this document's own extend-don't-edit rule — each paragraph below describes a generation step that has since happened; the current, up-to-date instructions are the paragraphs directly above.)*

To generate Week 14's resource books in a fresh chat, upload exactly three things:
1. The current version of the generation prompt
2. `Week_14_Revised.md`
3. This file (`00_Curriculum_Map.md`)

Optionally, include a `Week_15_Revised.md` too, if available — it would only be used to keep Week 14's own extra-practice picks from colliding with Week 15's required problems, the same overlap check now run thirteen times (Week 1 → 2 through Week 12 → 13, plus Week 13 → 14, confirmed clean — all six of Week 13's extras checked directly against Week 14's own required list before being added). No `Week_15_Revised.md` was available at the time this map was updated.

**No flagged collisions carry into Week 14** — the Week 14 Overlap section above confirms a clean check against the full inventory (229 distinct problems, Weeks 1–13), and none of Week 13's six extra-practice picks (LC 377, 673, 174, 583, 132, 486) collide with Week 14's required list. **Unlike Week 13's own transition, there is no lingering required-problem collision to stay aware of this time** — Week 13 closed with zero recaps needed in either direction, the first Dynamic Programming week in the series to do so; every one of Week 13's 21 required problems was confirmed new before being taught, and none of Week 14's own required problems collide with anything solved through Week 13 either (checked directly against the full inventory: LC 309, 714, 123, 188, 337, 124, 136, 191, 231, 338, 268, 137, 260, 371 — all absent). Week 14 has its own internal sequencing note worth re-reading before generating: it closes Dynamic Programming entirely — State Machine DP (Days 92–93, formalizing explicit `dp[i][state]` tracking for the first time, a generalization Week 13's 0/1 Knapsack and Predict the Winner both touched informally without ever building the state-machine framing itself, so it needs full, fresh treatment) and Tree DP (Day 94, opening with House Robber III, which extends House Robber's take-or-skip recurrence — Week 12, Day 82's disjoint-exhaustive-cases proof — onto a tree structure directly, assumed solid rather than re-derived) — before moving into Bit Manipulation. Week 13's DP-array-as-pruning-oracle technique (Word Break II's dictionary gate, Day 86; Palindrome Partitioning II's palindrome gate, Day 90) and its "map every operation to exactly which cell it reads from" discipline are assumed transferable on sight to whatever new recurrence shapes State Machine DP and Tree DP introduce, not re-explained from scratch.

When Week 14 is generated, its own resource-book generation should extend this same file — adding Week 14's concepts to the dependency map, appending Week 14's problems (required and extra) to the inventory table above, updating "what Week 14 leaves you knowing cold," and re-running the overlap check against Week 15. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended twelve times (Week 1 → Week 2 through Week 12 → Week 13) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues.

*(Everything from this point down is kept exactly as originally written, per this document's own extend-don't-edit rule — each paragraph below describes a generation step that has since happened; the current, up-to-date instructions are the paragraphs directly above.)*

To generate Week 13's resource books in a fresh chat, upload exactly three things:
1. The current version of the generation prompt
2. `Week_13_Revised.md`
3. This file (`00_Curriculum_Map.md`)

Optionally, include a `Week_14_Revised.md` too, if you have it — it only gets used to keep Week 13's own extra-practice picks from colliding with Week 14's required problems, the same overlap check already run twelve times now (Week 1 → 2 through Week 11 → 12, plus Week 12 → 13, confirmed clean — Week 12's one extra checked directly against Week 13's own required list before being added).

**No flagged collisions carry into Week 13** — the Week 13 Overlap section above confirms a clean check against the full inventory (202 distinct problems, Weeks 1–12), and Week 12's one extra-practice pick (LC 740) was confirmed absent from Week 13's required list before being taught. **Unlike every prior transition, one recap does carry forward as something to be aware of, not resolve** — Week 12 itself caught and handled a required-problem collision (LC 152, recapped Day 83); that's fully resolved and needs no further action, but it's the first time this document's own overlap machinery caught a collision in that direction, so it's worth knowing the mechanism exists when reviewing Week 13's own required list. Week 13 has its own internal sequencing note worth re-reading before generating: it is leave week 2 — no new theory or project content, full-time DSA immersion across all seven days — closing four Dynamic Programming subtypes (the rest of 1D DP, Grid DP, String DP, Interval DP) at a pace this map's own "What Week 12 Leaves You Knowing Cold" section assumes is already fully automatic, not something to re-derive mid-week. Coin Change II (Day 85) builds directly on Week 12's Unbounded Knapsack framing (Day 84) but needs the opposite loop-order rule, for counting rather than minimizing — that distinction was flagged in Week 12's own material but deliberately not built out, so it needs full, fresh treatment here, not a citation. Partition Equal Subset Sum and Target Sum (Days 85–86) will need 0/1 Knapsack introduced formally for the first time in the series — a genuinely new concept, not a variant of anything already taught, despite sounding adjacent to Unbounded Knapsack.

When Week 13 is generated, its own resource-book generation should extend this same file — adding Week 13's concepts to the dependency map, appending Week 13's problems (required and extra) to the inventory table above, updating "what Week 13 leaves you knowing cold," and re-running the overlap check against Week 14. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended eleven times (Week 1 → Week 2 through Week 11 → Week 12) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues.

*(Everything from this point down is kept exactly as originally written, per this document's own extend-don't-edit rule — each paragraph below describes a generation step that has since happened; the current, up-to-date instructions are the paragraphs directly above.)*

To generate Week 12's resource books in a fresh chat, upload exactly three things:
1. The current version of the generation prompt
2. `Week_12_Revised.md`
3. This file (`00_Curriculum_Map.md`)

Optionally, include a `Week_13_Revised.md` too, if you have it — it only gets used to keep Week 12's own extra-practice picks from colliding with Week 13's required problems, the same overlap check already run eleven times now (Week 1 → 2 through Week 10 → 11, plus Week 11 → 12, confirmed clean — Week 11's one extra checked directly against Week 12's own required list before being added).

**No flagged collisions carry into Week 12** — the Week 12 Overlap section above confirms a clean check against the full inventory (189 distinct problems, Weeks 1–11), and Week 11's one extra-practice pick (LC 1319) was confirmed absent from Week 12's required list before being taught. As with Week 11, there's no recap to resolve before teaching any specific day this time — every one of Week 11's 13 required problems was confirmed new against the table before being taught, and none of Week 12's own required problems collide with anything solved through Week 11 either. Week 12 has its own internal sequencing note worth re-reading before generating: Dijkstra's Algorithm opens fresh (Day 78), reusing the `PriorityQueue` mechanics from Days 17/26/54 and Day 77's Prim's directly rather than re-deriving heap-based shortest-path selection from scratch — per this series' own precedent, its own true opening day should receive no extras, the same treatment every prior pattern's genuine opening day has received (Trees Day 46, Heaps Day 54, Tries Day 59, Backtracking Day 61, Graphs Day 68, Union-Find Day 74). Dynamic Programming then opens mid-week (Day 81) — the single largest pattern in the entire plan — building directly on the informal DP-flavored reasoning already previewed twice without the formal name (Week 3's Kadane's Algorithm, and this week's own Floyd-Warshall, Day 73); that connection is worth citing directly rather than re-introducing memoization/tabulation as if from nothing.

When Week 12 is generated, its own resource-book generation should extend this same file — adding Week 12's concepts to the dependency map, appending Week 12's problems (required and extra) to the inventory table above, updating "what Week 12 leaves you knowing cold," and re-running the overlap check against Week 13. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended ten times (Week 1 → Week 2 through Week 10 → Week 11) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues.

*(The paragraph immediately below this one is kept exactly as originally written, per this document's own extend-don't-edit rule — it describes Week 11's generation, which has since happened; the current, up-to-date instructions are the paragraphs directly above.)*

To generate Week 11's resource books in a fresh chat, upload exactly three things:
1. The current version of the generation prompt
2. `Week_11_Revised.md`
3. This file (`00_Curriculum_Map.md`)

Optionally, include a `Week_12_Revised.md` too, if you have it — it only gets used to keep Week 11's own extra-practice picks from colliding with Week 12's required problems, the same overlap check already run ten times now (Week 1 → 2 through Week 9 → 10, plus Week 10 → 11, confirmed clean — both of Week 10's extras checked directly against Week 11's own required list before being added).

**No flagged collisions carry into Week 11** — the Week 11 Overlap section above confirms a clean check against the full inventory (175 distinct problems, Weeks 1–10), and both of Week 10's extra-practice picks (LC 802, LC 542) were confirmed absent from Week 11's required list before being taught. As with Week 10, there's no recap to resolve before teaching any specific day this time — every one of Week 10's 14 required problems was confirmed new against the table before being taught, and none of Week 11's own required problems collide with anything solved through Week 10 either. Week 11 has its own internal sequencing note worth re-reading before generating: Graphs continues and closes (6 more required problems, closing at 12/12), and its Course Schedule problems specifically extend Day 69's Find Eventual Safe States 3-state-coloring extension into full topological-sort cycle detection — that connection is worth citing directly rather than re-deriving directed-cycle detection from scratch. Union-Find then opens fresh later in the week — per this series' own precedent, its own true opening day should receive no extras, the same treatment every prior pattern's genuine opening day has received (Trees Day 46, Heaps Day 54, Tries Day 59, Backtracking Day 61, Graphs Day 68).

When Week 11 is generated, its own resource-book generation should extend this same file — adding Week 11's concepts to the dependency map, appending Week 11's problems (required and extra) to the inventory table above, updating "what Week 11 leaves you knowing cold," and re-running the overlap check against Week 12. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended nine times (Week 1 → Week 2 through Week 9 → Week 10) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues.

*(The paragraph immediately below this one is kept exactly as originally written, per this document's own extend-don't-edit rule — it describes Week 10's generation, which has since happened; the current, up-to-date instructions are the paragraph directly above.)*

When Week 10 is generated, its own resource-book generation should extend this same file — adding Week 10's concepts to the dependency map, appending Week 10's problems (required and extra) to the inventory table above, updating "what Week 10 leaves you knowing cold," and re-running the overlap check against Week 11. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended eight times (Week 1 → Week 2 through Week 8 → Week 9) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues. Week 10's generation should also specifically confirm the LC 17 resolution held: Letter Combinations of a Phone Number should appear exactly once in the extended table, on Day 65, marked `Required (plan)` — if it appears anywhere else, or twice, that's a signal this week's own generation introduced the exact duplication the Week 9 → 10 check was meant to prevent.

*(The paragraph immediately below this one is kept exactly as originally written, per this document's own extend-don't-edit rule — it describes Week 9's generation, which has since happened; the current, up-to-date instructions are the paragraph directly above.)*

When Week 9 is generated, its own resource-book generation should extend this same file — adding Week 9's concepts to the dependency map, appending Week 9's problems (required and extra) to the inventory table above, updating "what Week 9 leaves you knowing cold," and re-running the overlap check against Week 10. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended seven times (Week 1 → Week 2, Week 2 → Week 3, Week 3 → Week 4, Week 4 → Week 5, Week 5 → Week 6, Week 6 → Week 7, Week 7 → Week 8) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues. Week 9's generation should note that Backtracking opens fresh mid-week (Day 61) alongside Tries closing — a genuinely new pattern, so per this series' own precedent, it should receive the same "no extras on the opening day" treatment every prior pattern's true opening day has received; any deferred Backtracking extras should land on a later Backtracking day instead, the same way Heaps' landed on Days 55–56 this time.

*(The paragraph immediately below this one is kept exactly as originally written, per this document's own extend-don't-edit rule — it describes Week 7's generation, which has since happened; the current, up-to-date instructions are the paragraph directly above.)*

When Week 7 is generated, its own resource-book generation should extend this same file — adding Week 7's concepts to the dependency map, appending Week 7's problems (required and extra) to the inventory table above, updating "what Week 7 leaves you knowing cold," and re-running the overlap check against Week 8. Keep this as one continuously-growing document across the whole series rather than starting a fresh one each week. This file has now been extended five times (Week 1 → Week 2, Week 2 → Week 3, Week 3 → Week 4, Week 4 → Week 5, Week 5 → Week 6) without anything being removed or shortened — the same discipline should hold indefinitely as the series continues. Week 7's generation should note that Stacks/Monotonic Stack closes mid-week (Day 45, at 12/12) before Trees opens on Day 46 — a genuinely new pattern, so per this series' own precedent, it should receive the same "no extras on the opening day" treatment Linked Lists (Day 34) and Stacks/Monotonic Stack (Days 39 and 40, respectively) each received; any deferred Tree extras should land on a later Tree day instead, the same way Monotonic Stack's landed on Day 41.
