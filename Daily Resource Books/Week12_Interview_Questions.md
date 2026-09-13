# Week 12 — Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Covers:** [Day 78](Day78_Resource_Book.md) · [Day 79](Day79_Resource_Book.md) · [Day 80](Day80_Resource_Book.md) · [Day 81](Day81_Resource_Book.md) · [Day 82](Day82_Resource_Book.md) · [Day 83](Day83_Resource_Book.md) · [Day 84](Day84_Resource_Book.md)
**Companion to:** `Week_12_Revised.md`

Every question below is pulled verbatim from its day's Resource Book, in the order it was taught. Use this as a single pass for review — if an answer doesn't come immediately, the source day linked above has the full derivation, proof, or trace it was built from.

---

## Day 78 — Dijkstra's Algorithm Begins, and Sharding Strategies

**Q1. State Dijkstra's Algorithm's core loop from memory.** Maintain a min-heap of `(distance, node)`, seeded with `(0, source)`. Repeatedly pop the smallest; if its distance is stale (greater than the current best known), skip it; otherwise finalize the node and relax every outgoing edge, pushing any improved neighbor distance as a new heap entry.

**Q2. Prove why the greedily-finalized node's distance is guaranteed correct.** Any path that could beat it would have to pass through some other not-yet-finalized node first; since all weights are non-negative, that node's own distance-so-far can't exceed the full path's length, meaning it would already sit in the frontier with a smaller tentative distance than the node just popped — contradicting that the popped node had the smallest tentative distance of anything in the frontier.

**Q3. Why does the proof specifically require non-negative weights?** The step "the prefix of a shortest path can't be longer than the whole path" only holds when every edge from that prefix onward adds a non-negative amount — a negative edge could make a longer-hop path cheaper than an already-finalized "shortest" one, and Dijkstra's has no mechanism to revisit a finalized node.

**Q4. Why does the implementation push a new heap entry on every relaxation instead of updating an existing one?** Java's `PriorityQueue` doesn't support `decrease-key`; pushing a new entry and later skipping stale ones (lazy deletion) achieves the same effect without a custom indexed heap.

**Q5. What does skipping a stale heap entry actually cost if you forget to do it?** Not correctness — relaxation only ever improves a distance, so a stale (too-large) entry simply fails its relax checks — but it costs the complexity bound, since the heap can be reprocessed with outdated values instead of being bounded by genuine relaxations.

**Q6. Derive Dijkstra's time complexity.** At most one heap push per successful relaxation, bounded by O(E) edges, plus O(V) initial pushes; each push/pop on a heap of size O(E) costs O(log E) = O(log V) since E ≤ V²; total O(E log V).

**Q7. Why does Path with Maximum Probability use a max-heap and multiplication instead of a min-heap and addition?** Probabilities are all ≤1, so extending a path by another edge can only shrink (or hold) the running product — the "detouring can't help" argument flips from "can't make it smaller" to "can't make it larger," so the greedy pick becomes the frontier's largest tentative value.

**Q8. In Path with Maximum Probability, why is it safe to return the instant `end` is popped?** Because it's popped from a max-heap — nothing else remaining in the heap or reachable through further relaxation can ever produce a larger probability than the current maximum.

**Q9. Why does plain BFS give the wrong answer on Network Delay Time?** BFS locks in a node's distance the first time it's reached, which is only guaranteed cheapest when every edge costs the same; with different weights, a longer-hop path can be cheaper than a shorter-hop one BFS would have already finalized.

**Q10. What's the LeetCode-specific gotcha in Network Delay Time's array sizing?** Nodes are labeled 1 to n, not 0-indexed — the `dist[]` array needs size `n+1`, and the final scan runs from node 1 through node n, not from 0.

**Q11. Distinguish hash-based sharding from range-based sharding, including their opposite failure modes.** Range-based keeps range queries cheap but is vulnerable to hot spots when access skews toward one range; hash-based spreads load evenly regardless of range skew but makes range queries expensive, since consecutive keys scatter across shards.

**Q12. What is the "celebrity problem," and why doesn't any sharding strategy solve it?** One specific key receiving disproportionate traffic — whichever shard it happens to land on gets overloaded regardless of the partitioning scheme, because sharding strategies balance load across the *key space*, not against one key's own popularity.

**Q13. How does naive hash-based sharding relate to Consistent Hashing?** Naive hash-based sharding uses `hash(key) % N`, which remaps nearly every key when N changes; Consistent Hashing is the fix — a hash ring where changing the shard count only remaps a proportional slice of keys, which is why production hash-based sharding is almost always consistent-hash-based in practice.

---

## Day 79 — Dijkstra's: Constrained Variants, and 2PC vs. Saga

**Q1. Construct a concrete example where plain Dijkstra's gives the wrong answer once a stops constraint is added.** A cheap path reaching some intermediate node uses too many stops already; a pricier path to that same node uses fewer stops and has budget left to complete cheaply — Dijkstra's, which finalizes by cost alone, locks onto the cheap-but-stop-exhausted arrival and never explores the valid alternative.

**Q2. Why isn't "mark a node visited the first time it's popped" safe here?** Because the relevant state is `(node, stopsUsed)`, not `node` alone — a rejection for exceeding the stop budget must not block a later, valid arrival at the same node with fewer stops used.

**Q3. State Bellman-Ford's core loop and what each round guarantees.** Relax every edge in the graph once per round, against a frozen snapshot of the previous round's distances; after `r` rounds, every `dist[]` value is correct for "reachable using at most `r` edges."

**Q4. Why must Bellman-Ford relax against a snapshot rather than updating in place?** Updating in place lets one round's later edge relaxations use that same round's already-improved values, silently chaining multiple edges into a single round and breaking the "round `r` = at most `r` edges" guarantee the correctness proof depends on.

**Q5. Why does Bellman-Ford handle negative weights when Dijkstra's can't?** It never permanently finalizes a node mid-algorithm — every value stays open to improvement in a later round — so a negative edge that would undercut an already-"finalized" Dijkstra distance simply gets picked up in a subsequent relaxation round instead of being silently missed.

**Q6. Why does Path With Minimum Effort still admit a Dijkstra-shaped solution despite not summing weights?** The greedy proof only needs "extending a path by one more step can never decrease its cost" — true for both a running sum of non-negative weights and a running maximum, since `max(a,b) ≥ a` always holds.

**Q7. Name the alternative approach to Path With Minimum Effort that doesn't use Dijkstra's at all, and its cost.** Binary search on the answer (candidate effort threshold `T`) combined with Union-Find, unioning every adjacent-cell pair with a height difference `≤ T` and checking start/end connectivity — costs an extra `log(maxHeightDiff)` factor versus the single-pass Dijkstra approach.

**Q8. State Two-Phase Commit's two phases.** Prepare: the coordinator asks every participant to vote yes/no, and a `yes` vote is a binding promise. Commit/Abort: if every vote was yes, the coordinator tells everyone to commit; if any vote was no, everyone aborts.

**Q9. Precisely, when does 2PC block, and why?** When the coordinator crashes after a participant has voted yes but before that participant receives the final commit/abort instruction — the participant can't unilaterally proceed either way, since it's already promised to commit if told to, and has to wait for the coordinator to recover.

**Q10. Connect 2PC's blocking cost to a concept already covered.** The same mechanism as Day 64's cascading-failure argument — a resource (a lock here, a thread there) held while waiting on something unresponsive, stalling everything queued behind it — just at transaction scope instead of thread-pool scope.

**Q11. Why is a Saga's compensating transaction not the same thing as a database rollback?** The earlier local transaction already committed for real; there's nothing left to "roll back" — a compensating transaction is a new, independent, forward-moving operation that reverses the committed effect explicitly.

**Q12. Contrast what 2PC and Saga each trade away.** 2PC trades availability (blocking risk tied to the coordinator) for strong, atomic consistency; Saga trades strong consistency (a visible window of partial completion) for availability — every local step commits independently and immediately.

---

## Day 80 — Dijkstra's Capstone, and Kafka Streams

**Q1. What changed between Path With Minimum Effort's relaxation rule and Swim in Rising Water's, and what stayed the same?** The compared quantity shifted from an edge property (the height difference between two adjacent cells) to a node property (the destination cell's own elevation) — the min-heap, finalize-on-pop discipline, and correctness proof are all unchanged, since both rely only on "extending a path can't decrease its running maximum."

**Q2. Why does Swim in Rising Water seed `dist[0][0] = grid[0][0]` instead of `0`?** The starting cell's own elevation is already part of the answer — you need the water at least that high just to stand at the start, unlike a sum-based Dijkstra problem where a zero-length path genuinely costs zero.

**Q3. Name the alternative approach to Swim in Rising Water, and what it costs relative to Dijkstra's.** Binary search on the candidate time `T`, combined with Union-Find unioning every adjacent pair with both elevations `≤ T`, checking start/end connectivity — costs an extra `log(maxElevation)` factor and repeated Union-Find construction versus a single Dijkstra pass.

**Q4. Give the full accounting for why Dijkstra's Algorithm received zero extra practice this week.** The required ladder was already deliberately expanded from 3 to 5 problems by this week's own plan revision specifically to close a known gap, and the five problems already span every distinct relaxation shape the pattern tests — sum-minimizing, multiplicative/max-heap, hop-constrained (a different algorithm), and two minimax variants each paired with a distinct alternative approach; a sixth would reinforce, not close, a gap.

**Q5. Define `KStream` and `KTable` precisely, in one sentence each.** `KStream` is an unbounded, event-by-event view of a topic, processed in arrival order within a partition. `KTable` is a changelog — the latest value per key, updated (not appended) as new records for that key arrive.

**Q6. Explain the stream-table duality — why is a `KTable` not a fundamentally different structure from a `KStream`?** A `KTable` is a compacted view of a stream: its changelog topic (retaining only the latest record per key via log compaction) is itself a `KStream` of updates, and any `KStream` can be aggregated into a `KTable` — the same underlying infrastructure serves both views.

**Q7. What actually backs a `KTable` internally?** A compacted changelog topic plus, typically, a local materialized state store (e.g., RocksDB) that Kafka Streams queries directly for fast point lookups, instead of re-scanning the topic on every query.

**Q8. Why does the `mapValues` operator preserve partitioning while `map` might not?** `mapValues` only transforms the value, leaving the key — and therefore the partition a record belongs to — untouched; `map` can change the key, which forces Kafka Streams to assume a repartition might be needed.

**Q9. What does a `null` value mean in a `KStream` feeding a `KTable`, and why does it matter?** A tombstone — an explicit signal to delete that key from the `KTable`'s materialized view, not to set the key's value to null; this only matters when the same topology is also building table semantics from the stream.

**Q10. Does Kafka Streams change Kafka's partition-ordering guarantee from Day 48?** No — ordering is still guaranteed only within a single partition, never across an entire topic; `KStream` processing inherits this unchanged.

---

## Day 81 — Dynamic Programming Begins

**Q1. Trace `fib(5)`'s call tree and state exactly how many times `fib(2)` is recomputed.** Three times — once under `fib(4)`'s `fib(3)`, once directly under `fib(4)`, once under `fib(5)`'s own `fib(3)` — each recomputation producing the identical answer, `1`.

**Q2. State the two properties a problem needs for Dynamic Programming to apply.** Optimal substructure (the optimal answer to the whole problem is built from optimal answers to its subproblems) and overlapping subproblems (the same subproblem recurs multiple times during a naive solve).

**Q3. Give an example of a problem with optimal substructure but no overlapping subproblems, and explain why it doesn't need memoization.** Constructing a binary tree from preorder/inorder traversal (Week 8, Day 51) — each recursive call operates on a disjoint index range, so no subproblem is ever solved twice; there's nothing for a cache to save.

**Q4. Why does memoized `fib(40)` run dramatically faster than naive `fib(40)`?** Naive recursion's call count grows exponentially (the same `T(n)=T(n-1)+T(n-2)+1` shape traced for `fib(5)`, at a much larger scale) — hundreds of millions of calls at `n=40` — while memoization computes each of the 40 distinct states exactly once.

**Q5. What's the space cost difference between memoization and tabulation, precisely?** Both are O(n) in the general case, but memoization pays for a HashMap cache *and* recursion call-stack depth (risking `StackOverflowError` for large `n`), while tabulation is a single iterative array with no stack risk.

**Q6. Why can tabulation be reduced to O(1) space more easily than memoization?** Tabulation's explicit, controlled iteration order makes it clear exactly which past values are still needed (often just the last `k`), so only those need to be kept; memoization's space is tied to cache size and call-stack depth together, both harder to shrink independently.

**Q7. What's the first question to ask before writing any DP code, and why does it matter?** "What does `dp[i]` represent, in one precise sentence?" — if it can't be answered precisely, there isn't a DP solution yet, just a hope that one exists; a vague or wrong definition here produces a recurrence that looks plausible but is silently incorrect.

**Q8. Why does greedy fail on problems Dynamic Programming solves?** Cited from Week 4, Day 23's 0/1 Knapsack counter-example — a greedy algorithm commits to one locally-best choice per step and never reconsiders, and an early choice can block a strictly better later combination with no exchange argument able to recover the loss; DP effectively considers every choice's downstream consequences instead.

**Q9. Prove why `dp[i] = dp[i-1] + dp[i-2]` is correct for Climbing Stairs.** Every way to reach step `i` ends in exactly one of two possible final moves — a 1-step from `i-1` or a 2-step from `i-2` — these cases are disjoint (different final moves) and exhaustive (no other move exists), so by the addition principle the total is their sum.

**Q10. What is Climbing Stairs, in relation to a problem already solved?** `fib(n+1)` — the exact same recurrence and growth pattern as Fibonacci, offset by one index, since both base cases evaluate to `1`.

**Q11. State precisely what `dp[i]` means in Min Cost Climbing Stairs, and why the base cases are `cost[0]` and `cost[1]`, not `0`.** `dp[i]` is the minimum total cost paid by the point you launch from stair `i` — it already includes `cost[i]` itself. Arriving at index 0 or 1 is free, but leaving either still tolls that stair's own cost, so the base cases must include `cost[0]`/`cost[1]`, not be zero.

**Q12. Why is the final answer `min(dp[n-1], dp[n-2])` rather than `dp[n]`?** The "top" is one position past the last real stair and has no cost of its own — you just need the cheaper of the two possible final springboards, not a cost for landing past the array.

**Q13. In the worked trace, why does `dp[6]` end up small despite `dp[5]` being large?** The recurrence takes `min(dp[5], dp[4])`, and `dp[4]=3` is far cheaper than `dp[5]=103` — the algorithm automatically routes around the expensive stair by preferring the 2-step jump from stair 4, with no special-casing needed for "this stair is expensive."

**Q14. What's the general complexity framework for a DP solution, and how does it apply to Climbing Stairs?** Time = (number of distinct states) × (work per state); space = (number of distinct states), reducible with a rolling window. Climbing Stairs has `n` states, O(1) work each → O(n) time; each state depends only on the two previous ones → reducible to O(1) space.

**Q15. Name a concrete wrong-iteration-order bug in tabulation and why it's a correctness issue, not just a performance one.** Filling a `dp[]` array in decreasing index order when the recurrence needs `dp[i-1]`/`dp[i-2]` already computed reads an uninitialized (default-zero) cell instead of a real value — the result is a wrong number, not a slow one.

---

## Day 82 — 1D DP: House Robber Family

**Q1. Prove House Robber's recurrence correct.** At house `i`, exactly two disjoint, exhaustive choices exist: skip it (best is `dp[i-1]`) or rob it (earn `nums[i]` plus the best excluding both `i` and `i-1`, which is `dp[i-2]`) — the answer is the max of the two, since no third option exists and both can't happen together.

**Q2. Why is greedily robbing every other house by position not a valid strategy?** A high-value house at an "odd" position can make it strictly better to skip to it rather than alternate blindly — position-based alternation ignores the actual values, which is exactly why this needs DP rather than a fixed pattern.

**Q3. Prove House Robber II's circular-to-linear reduction.** Every valid circular plan excludes at least one of house 0 or house n-1, since they're adjacent and can't both be robbed; excluding house 0 reduces to an ordinary linear House Robber on houses 1..n-1, and excluding house n-1 reduces to houses 0..n-2 — together these two sub-cases capture every valid plan and never produce an invalid one, so their max is the answer.

**Q4. Why does House Robber II need an explicit `n=1` special case?** With one house, both "exclude house 0" and "exclude house n-1" refer to the same only house, so both linear sub-cases run over an empty range and return 0 — but the true answer is `nums[0]`, since a single house has no adjacency conflict at all; the general reduction only works when there are genuinely two distinct boundary houses to alternate excluding.

**Q5. In the House Robber II trace (`nums=[2,7,9,3]`), why is 11 achievable even though houses 0 and 2 aren't consecutive array indices?** Because circular adjacency only connects house 0 to houses 1 and 3 (not house 2) — houses 0 and 2 are non-adjacent in this specific 4-house circle, so robbing both is a fully valid combination.

**Q6. What is Delete and Earn's core insight, in one sentence?** Picking one occurrence of a value never conflicts with picking another occurrence of the *same* value — only adjacent, *different* values conflict — so the real decision is per-distinct-value, which is exactly House Robber's shape once you bucket total earnable points by value.

**Q7. Why does Delete and Earn's transform use `points[v] = v × count(v)` rather than just counting occurrences?** Each occurrence of value `v` earns `v` points when taken — the total available payoff from fully committing to value `v` is its point value times how many times it appears, not just its frequency.

**Q8. What does a value with zero occurrences do inside Delete and Earn's bucketed array, and why doesn't it need special-casing?** Its `points[]` entry is naturally `0`, which the House Robber recurrence never selects over a positive neighbor — it behaves exactly like a genuinely worthless "house," correctly acting as a gap without any extra logic.

**Q9. Contrast the three DP problems solved across Days 81–82 by what varies between them.** All three share the same two-steps-back lookback shape; Climbing Stairs sums both cases (counting), House Robber takes a max between them (optimizing, take-or-skip), and House Robber II runs that same max-based recurrence twice over two linear slices to handle a circular constraint.

**Q10. Why does Delete and Earn use `long` for its running totals?** With up to 10⁴ elements each worth up to 10⁴, bucketed point totals can approach the edge of comfortable `int` range — the same overflow discipline established Day 10/11 applies to guard against silent wraparound.

---

## Day 83 — 1D DP: Decoding and Products

**Q1. Prove Decode Ways's recurrence correct.** Every valid decoding of a prefix ends in exactly one of two mutually exclusive final groups — a single trailing character or a trailing pair — so the count is the sum of whichever of those two cases is actually a legal code, contributing `dp[i-1]` or `dp[i-2]` respectively.

**Q2. Why is a lone `'0'` never a valid single-digit code?** Codes only run `1`–`26`; there is no code `0`, so a standalone `'0'` can never be decoded on its own.

**Q3. Why does `"06"` fail as a two-digit code without a dedicated leading-zero check?** `"06"` parses to the integer `6`, which is below the required `10`–`26` range — the same numeric range test that validates every other two-digit group already excludes it, with no separate rule needed.

**Q4. In the `"106"` trace, why does `dp[2]` end up as `1` rather than `0`?** Because `'0'` alone fails the single-digit check (contributing 0) but `"10"` succeeds as a valid two-digit code (contributing `dp[0]=1`) — the two-digit path rescues what the single-digit path couldn't cover.

**Q5. What happens if an interior character is `'0'` and it can't form a valid two-digit code with its predecessor (e.g., `"90"`)?** Both terms fail — `'0'` alone is invalid and `90` exceeds the `10`–`26` range — so `dp[i]=0` at that position, and that zero propagates forward through every later `dp[]` value that depends on it, correctly making the whole string undecodable from that point on.

**Q6. Why did Maximum Product Subarray get a recap instead of a full re-teach today?** It was already solved in Week 4, Day 22 as extra practice during Kadane's coverage — the overlap was caught by checking today's required problems against the cumulative problem table before treating anything as new.

**Q7. Why wasn't a substitute problem added today to replace the recapped one?** Only one of today's two required problems overlapped, not both — the rule for adding a fresh replacement problem applies specifically when an entire day's required block turns out to already be solved, which isn't the case here.

**Q8. Why does Maximum Product Subarray need a running minimum, when Kadane's only ever needed a running maximum?** Addition never reorders candidates — adding a negative number only shrinks a running sum. Multiplying by a negative number can flip a very negative running product into the new largest value in a single step, so the algorithm has to track the running minimum too, in case it's about to become the maximum.

**Q9. Why does the code swap `maxEndingHere` and `minEndingHere` specifically when the current number is negative?** A negative multiplier reverses which of the two running values is capable of producing the larger product next — swapping first means the subsequent `max`/`min` calculations combine the correct operands without needing a three-way comparison against both un-swapped values.

**Q10. State the general lesson this recap reinforces about extending a known technique.** Whenever a "best so far" recurrence swaps addition for multiplication (or any operation where a sign or direction can flip), check whether the ordering between your current best and worst candidates can reverse — if it can, tracking only one running extreme is no longer sufficient.

---

## Day 84 — Consolidation, and Unbounded Knapsack Begins

**Q1. Prove Word Break's recurrence correct.** Any valid full segmentation has a specific last word, starting at some position `j`; if `s[0..j)` is itself segmentable (`dp[j]` true) and `s[j..i)` is a dictionary word, then `s[0..i)` is segmentable — trying every possible `j` as the last word's start covers every possible valid segmentation.

**Q2. Why is the commonly-cited O(n²) complexity for Word Break not the full picture in Java?** `s.substring(j,i)` creates a genuine copy costing O(i-j), and hashing a freshly-built String costs another O(i-j) before the HashSet lookup — accounting for this honestly across all (i,j) pairs gives a true worst-case of O(n³), not O(n²).

**Q3. What fixes Word Break's extra complexity factor, and why isn't it built into the core solution here?** A Trie built from the dictionary, or index-based substring comparison avoiding new String allocation — flagged as extension material because it isn't needed to solve the problem as stated, only to tighten the bound under interviewer pressure.

**Q4. Why does Word Break use a Set specifically, and what would a List cost instead?** A HashSet gives O(1) average membership checks; a List would force an O(k) linear scan through the dictionary on every one of the DP's up-to-O(n²) checks, compounding an already-large state count.

**Q5. Prove Coin Change's recurrence correct.** Consider the optimal solution for amount `i` and its last coin used, `c` — removing it leaves a sub-solution for `i-c` that must itself be optimal, or swapping in a cheaper one would produce a cheaper solution for `i`, a contradiction; trying every coin as the candidate last-used coin and taking the min finds the true optimum.

**Q6. Why does Coin Change's recurrence allow unlimited reuse of a coin without any explicit tracking?** Nothing in `dp[i-c]+1` restricts which coins built `dp[i-c]` — if reusing `c` again was already reflected in `dp[i-c]`'s own value, using it again is neither prevented nor specially permitted; the unbounded behavior falls out of simply not tracking which coins were spent.

**Q7. Why is the sentinel value `amount + 1`, not `Integer.MAX_VALUE`?** Any real achievable answer uses at most `amount` coins, so `amount+1` is guaranteed larger than any genuine result while remaining small enough that `dp[i-coin]+1` can never overflow — seeding with `MAX_VALUE` would overflow into a negative number the moment that `+1` is applied to an unset cell.

**Q8. Why doesn't loop order (coins-outer vs. amounts-outer) affect Coin Change's answer?** The recurrence takes a minimum over independent coin choices, and `min()` is order-independent — trying coins in any sequence produces the same smallest value.

**Q9. Will loop order matter for every unbounded-reuse DP problem?** No — it matters specifically when a problem counts distinct *combinations* rather than minimizing a value; counting needs a fixed loop order to avoid treating the same combination as multiple different orderings, a distinction that doesn't apply to a plain minimization like this one.

**Q10. State this week's corrected cumulative problem count, and explain the discrepancy with the plan's own number.** 164, using the required-ladder-only convention the plan itself uses — the plan states 163, continuing a one-off drift first documented at Week 9, Day 63 and never corrected in the plan's own files since.

---

**Total: 80 questions across 7 days.** For the full derivations, proofs, and worked traces behind any answer above, see the linked day.
