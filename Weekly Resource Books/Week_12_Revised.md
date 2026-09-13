# Week 12 (Revised): Dijkstra's Algorithm Completes, Dynamic Programming Begins

**What changed:** Dijkstra's closes at 5 problems (up from 3) — the other thinnest gap from the original audit, now fixed alongside Union-Find last week. Dynamic Programming begins here — the single largest pattern in this plan, and the reason the timeline needed the honest recalculation a few messages back. It stays at its full 33-problem original scope; the depth was never the problem, the schedule around it was.

---

## Day 78 — Dijkstra's Algorithm Begins, and Sharding Strategies

### DSA Block (2.5 hrs)

**Concept Card — Dijkstra's Algorithm (Weighted Shortest Path)**
- What: finds the shortest path from a source node to every other node in a graph, provided every edge weight is *non-negative*. It always expands whichever not-yet-finalized node is currently closest to the source, using a min-heap to efficiently find that node each step.
- Why: BFS only finds shortest paths by *edge count* — the moment edges carry different weights, BFS can give a wrong answer, because "fewest edges" and "lowest total weight" aren't the same thing once weights differ.
- Interview signal: "minimum cost/time/distance," a weighted graph, "cheapest," anything where edges aren't all equal.
- Prerequisites: Heaps ✅, Graph BFS ✅ (both already covered).
- Mechanics: a min-heap holding `(distance, node)` pairs, starting with `(0, source)`. Pop the closest node; if it's already been finalized, skip it; otherwise, relax (attempt to improve) the distance to each of its neighbors.

- Problem 1: Network Delay Time — LeetCode #743 — Medium — Pattern: Dijkstra's Algorithm
  - Hint: close to textbook Dijkstra — build the weighted adjacency list, run from the source, the answer is the maximum finalized distance (or -1 if any node is unreachable).
  - Complexity: Time O(E log V) | Space O(V+E)
- Problem 2: Path with Maximum Probability — LeetCode #1514 — Medium — Pattern: Dijkstra (multiplicative) **(new)**
  - Hint: same algorithm skeleton, but you're maximizing a product of probabilities instead of minimizing a sum — use a *max*-heap instead of a min-heap, and multiply instead of add when relaxing.
  - Complexity: Time O(E log V) | Space O(V+E)

### Theory Block (2 hrs)
- Topic: Sharding Strategies
- As a single database reaches its limits, sharding splits data across multiple database instances. Range-based sharding assigns contiguous key ranges to each shard — simple to reason about, but prone to hot spots when access is skewed toward one range. Hash-based sharding spreads load evenly by hashing the key, but makes range queries expensive since a range query now has to hit every shard. There's a specific failure mode worth naming: the "celebrity problem" — one extremely popular key (a viral post, a bestselling product) can overwhelm whichever single shard it lands on, regardless of which sharding strategy you picked, since neither approach protects against one key getting disproportionate traffic.
- Coding exercise: none — sketch how you'd shard the platform's `orders` table by `userId` using hash-based sharding, and identify what happens if one user creates a disproportionate number of orders.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: add pagination to the platform's GET endpoints using Spring Data's `Pageable`/`PageRequest`.
- Definition of done: `/orders?page=0&size=5` returns exactly 5 items plus page metadata.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: follow up on any pending recruiter or alumni messages.

### Daily Deliverable
- [ ] Network Delay Time and Path with Maximum Probability solved, pushed to `dsa-java/dijkstra/`.
- [ ] Sharding sketch complete. Pagination live on the platform.

---

## Day 79 — Dijkstra's: Constrained Variants, and 2PC vs. Saga

### DSA Block (2.5 hrs)
- Problem 3: Cheapest Flights Within K Stops — LeetCode #787 — Medium — Pattern: Modified Dijkstra / Bellman-Ford
  - Hint: plain Dijkstra doesn't respect the "at most K stops" constraint directly, since it always greedily picks the globally-cheapest next node regardless of how many stops that took — track `(cost, node, stopsUsed)` and only relax when you haven't exceeded K stops, or use a Bellman-Ford-style K-round relaxation instead.
  - Complexity: Time O(K×E) | Space O(V)
- Problem 4: Path With Minimum Effort — LeetCode #1631 — Medium — Pattern: Dijkstra (minimax variant)
  - Hint: instead of summing edge weights along the path, the "distance" you're minimizing is the *maximum* absolute height difference along the path — same algorithm skeleton, a different relaxation rule.
  - Complexity: Time O(E log V) | Space O(V+E)

### Theory Block (2 hrs)
- Topic: Two-Phase Commit vs. the Saga Pattern
- Two-Phase Commit coordinates a distributed transaction with a "prepare, then commit" handshake across every participant — every participant must agree it's ready before anyone actually commits. The problem: if the coordinator crashes between the prepare and commit phases, every participant is left blocked, holding locks, unable to proceed either way. This is exactly why 2PC doesn't scale well to microservices. The Saga pattern takes a different approach entirely: break the transaction into a sequence of local transactions, each with its own compensating action that can undo it if a later step in the sequence fails. Choreography means each service reacts to other services' events with no central coordinator (this is what you already built into the platform's Order→Payment flow); Orchestration means a central coordinator explicitly directs each step instead.
- Coding exercise: none — write 150 words on what your platform's Saga-based order flow does, specifically, if the payment step fails after inventory has already been reserved — what's the compensating action?

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: implement the compensating action from the exercise above — if payment fails, the Order module should listen for a `PaymentFailedEvent` and release/compensate whatever the Order step already committed.
- Definition of done: simulating a payment failure correctly triggers the compensating action, verified by checking the Order's final state.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: apply to 1 Tier B target company.

### Daily Deliverable
- [ ] Cheapest Flights Within K Stops and Path With Minimum Effort solved, pushed.
- [ ] Saga compensating-action write-up complete and implemented on the platform.

---

## Day 80 — Dijkstra's Capstone, and Kafka Streams

### DSA Block (2 hrs)
- Problem 5: Swim in Rising Water — LeetCode #778 — Hard — Pattern: Dijkstra (minimax variant) **(new)**
  - Hint: same minimax idea as Path With Minimum Effort — the "cost" of a path is the maximum elevation encountered along it, not a sum. A min-heap ordered by that running maximum gets you there.
  - Complexity: Time O(n² log n) | Space O(n²)

**This closes Dijkstra's Algorithm: 5 problems — up from 3 in the original plan. Combined with last week's Union-Find fix, both of the thinnest gaps identified in the original audit are now closed.**

### Theory Block (2 hrs)
- Topic: Kafka Streams
- `KStream` represents an unbounded, record-by-record stream of events as they arrive. `KTable` represents a changelog — the latest known value for each key, updated as new records for that key arrive (think of it as a continuously-updating table built from a stream of changes). These two abstractions are the core building blocks for processing streams directly inside your own application, rather than only producing/consuming raw messages.
- Coding exercise: sketch a KStream topology reading from a `raw-order-events` topic, mapping values to a normalized format, writing to a `processed-order-events` topic.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: implement the KStream topology above as a standalone demo module.
- Definition of done: pushed, running, verified via console consumer.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: second mock-interview swap with your accountability partner — include one Graph or Union-Find problem, since those are the freshest patterns right now.

### Daily Deliverable
- [ ] Swim in Rising Water solved — Dijkstra's ladder complete at 5 problems.
- [ ] KStream demo pushed. Mock-interview swap completed.

---

## Day 81 — Dynamic Programming Begins

### DSA Block (2.5 hrs)

**Concept Card — Dynamic Programming**
- What: solving a problem by breaking it into overlapping subproblems, solving each one exactly once, and reusing that result every time it's needed again — instead of recomputing it from scratch each time, which is what plain recursion does.
- Why: think back to Week 2's Fibonacci recursion exercise, where tracing `fib(5)`'s call tree showed `fib(2)` getting recomputed multiple times, wastefully. DP fixes exactly this waste, in one of two ways: **memoization** (top-down — write the recursion normally, but cache results the first time each one is computed) or **tabulation** (bottom-up — build an array from the smallest subproblem upward, no recursion needed at all).
- Where: resource allocation, sequence alignment, scheduling — anywhere "the optimal solution to the whole problem is built from optimal solutions to its smaller pieces" holds true.
- Interview signal: "minimum/maximum number of ways," "can you reach/make exactly X," optimization over a sequence or grid where a greedy approach provably fails. The single most important question to ask yourself before writing any code: *what does `dp[i]` represent, in one precise sentence?* If you can't answer that, you don't actually have a DP solution yet — you have a hope that one exists.
- Prerequisites: recursion ✅.

- Problem 1: Climbing Stairs — LeetCode #70 — Easy — Pattern: 1D DP
  - Hint: `dp[i] = dp[i-1] + dp[i-2]` — Fibonacci wearing a different name. Optimize to O(1) space by keeping only the last two values instead of a full array.
  - Complexity: Time O(n) | Space O(1)
- Problem 2: Min Cost Climbing Stairs — LeetCode #746 — Easy — Pattern: 1D DP
  - Hint: `dp[i] = cost[i] + min(dp[i-1], dp[i-2])`; you're allowed to start from either step 0 or step 1.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Dynamic Programming, Practiced — Memoization vs. Tabulation, Side by Side
- Coding exercise: implement Fibonacci three separate ways — naive recursion, top-down with a `HashMap` cache, and bottom-up tabulation — and time all three for `n=40`. The naive version should take visibly, painfully long, which is the entire point of seeing all three side by side.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: the three-Fibonacci-implementations exercise above, in a `DPFoundations` class, with timing printed for each.
- Definition of done: pushed, with a comment on each approach's space complexity.

### Career Block (1 hr)
- LinkedIn: Post 17 — "I timed naive recursion vs. memoization on Fibonacci(40), and the difference is absurd."
- Networking: identify 2 Tier B companies to apply to this week.

### Daily Deliverable
- [ ] Climbing Stairs and Min Cost Climbing Stairs solved, pushed to `dsa-java/dynamic-programming/`.
- [ ] `DPFoundations` timing comparison pushed. LinkedIn Post 17 published.

---

## Day 82 — 1D DP: House Robber Family

### DSA Block (2.5 hrs)
- Problem 3: House Robber — LeetCode #198 — Medium — Pattern: 1D DP
  - Hint: `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — either skip this house (take yesterday's best) or rob it (two houses back's best, plus this one).
  - Complexity: Time O(n) | Space O(1)
- Problem 4: House Robber II — LeetCode #213 — Medium — Pattern: 1D DP (circular)
  - Hint: the houses are arranged in a circle now, so the first and last are adjacent. Run House Robber's exact logic twice — once excluding the first house, once excluding the last — and take the max of the two results.
  - Complexity: Time O(n) | Space O(1)

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 2 Tier B companies to apply to this week.

### Daily Deliverable
- [ ] House Robber and House Robber II solved, pushed.

---

## Day 83 — 1D DP: Decoding and Products

### DSA Block (2.5 hrs)
- Problem 5: Decode Ways — LeetCode #91 — Medium — Pattern: 1D DP
  - Hint: `dp[i] = dp[i-1]` (if the single digit at position `i` is valid on its own, meaning 1-9) `+ dp[i-2]` (if the two digits ending at `i` together form a valid 10-26 code).
  - Complexity: Time O(n) | Space O(1)
- Problem 6: Maximum Product Subarray — LeetCode #152 — Medium — Pattern: 1D DP
  - Hint: a negative number flips the meaning of "max" and "min" instantly — track both `maxSoFar` and `minSoFar` ending at each position, since today's minimum could become tomorrow's maximum after one more negative multiplication.
  - Complexity: Time O(n) | Space O(1)

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: follow up on Tier B applications.

### Daily Deliverable
- [ ] Decode Ways and Maximum Product Subarray solved, pushed.

---

## Day 84 (Sunday) — Consolidation, and Unbounded Knapsack Begins

### Self-Check (15 min)
- [ ] State, from memory, what `dp[i]` represents for each of this week's problems so far. If any answer feels shaky, that's worth another look before adding more problems on top.

### DSA Block (2 hrs)
- Problem 7: Word Break — LeetCode #139 — Medium — Pattern: 1D DP + Set
  - Hint: `dp[i]` is true if the substring `s[0..i]` can be fully segmented into dictionary words. For every `j < i`, if `dp[j]` is true and `s[j..i]` is itself a dictionary word, then `dp[i]` is true.
  - Complexity: Time O(n²) | Space O(n)
- Problem 8: Coin Change — LeetCode #322 — Medium — Pattern: 1D DP (Unbounded Knapsack)
  - Hint: `dp[i] = min(dp[i], dp[i - coin] + 1)` for every coin denomination — initialize the array with a dummy high value (`amount + 1`) representing "not yet reachable."
  - Complexity: Time O(n×amount) | Space O(amount)

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 84, twelve weeks in, **163 total DSA problems solved.** Dijkstra's fully closed at 5 (up from 3) — both thin-pattern gaps from the original audit are now fixed. Dynamic Programming is 8 problems into its 33-problem run. This is the largest remaining block in the plan, and where leave week 2 will land — more on the exact dates in the next update.

### Daily Deliverable
- [ ] Word Break and Coin Change solved, pushed.
- [ ] Weekly ritual and scorecard complete.
