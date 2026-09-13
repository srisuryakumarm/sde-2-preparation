# Week 14 — Consolidated Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Covers:** Day 92 through Day 98 — State Machine DP, Tree DP (Dynamic Programming closes entirely), and Bit Manipulation opens through 8/10 required.
**◀ Previous:** Week 13 Interview Questions
**Next ▶:** Week 15 Interview Questions

Every question below is pulled directly from its source day's Resource Book, in the same order. Use this file for weekly review; use the individual day books for full derivations, worked traces, and code.

---

## Day 92 — State Machine DP Opens: Transaction Fee and Cooldown

**Q1. Define State Machine DP in one sentence, precisely.**
`dp[i][s]` is the best achievable outcome by step `i` given you end step `i` in state `s`; a transition from `s'` to `s` is only included if the action producing it is legal under the problem's rules.

**Q2. Why does LC 122's greedy telescoping sum fail the moment a transaction fee is added?**
The greedy sum implicitly decomposes any multi-day hold into free single-day steps; a fee makes that decomposition non-free (more steps, more fee paid), so the sum silently overstates achievable profit — proven concretely with `prices=[1,3,2], fee=2`, where the true optimum is `0` but naive greedy would report `2`.

**Q3. Why can't Cooldown reuse LC 714's 2-state machine directly?**
Two states can't distinguish "not holding, and free to buy" from "not holding, but sold yesterday, still cooling down" — both are "not holding," but only one permits buying today. A third state is required to carry that one-day distinction forward.

**Q4. In the Cooldown recurrence, what single omission enforces the cooldown rule?**
`hold` transitions only from `prevRest`, never from `prevSold` — the illegal buy-the-day-after-selling transition is structurally absent from the recurrence, not filtered with a conditional.

**Q5. What's the space complexity of both solutions, and why is a full 2D table never needed?**
O(1) — `dp[i][·]` depends only on `dp[i-1][·]`, so rolling variables (one per state) fully replace the table, the same optimization 0/1 Knapsack established for its capacity dimension (Day 85).

**Q6. In the `PaymentState` example, why does the guarded `Authorized` case have to come before the unguarded one?**
`switch` pattern matching checks case labels top to bottom; an unguarded `Authorized` pattern matches every `Authorized` value, so placing it first would shadow the guarded branch beneath it and it would never be reached.

**Q7. Why does the exhaustive switch over `PaymentState` compile with no `default` clause?**
`PaymentState` is `sealed` with a compiler-known, finite `permits` list, and every permitted type has a corresponding `case` — the compiler statically proves every possible input is handled.

---

## Day 93 — State Machine DP Closes: Bounding the Transaction Count

**Q1. What does the new dimension `k` count in `dp[i][k][state]`, and when does it advance?**
The number of transactions started (buys made) so far, capped at `K`; it advances only on a buy, never a sell, since a transaction is charged at the point of commitment.

**Q2. Derive `buy1`'s recurrence from the general form.**
`dp[i][1][hold] = max(dp[i-1][1][hold], dp[i-1][0][cash] - price[i])`; since `dp[·][0][cash]` is always `0` (zero transactions, zero profit), this simplifies to `max(buy1, -price[i])` — exactly the code.

**Q3. Why does `buy2` read the current day's `sell1`, not yesterday's?**
It's deliberate: "at most 2 transactions" permits a same-day sell-then-buy, so `buy2` must be allowed to start from a sale that just happened this same iteration, not one frozen from the prior day.

**Q4. What does the `k ≥ n/2` check in LC 188 protect against, and why `n/2` specifically?**
It avoids running the O(n·k) array path with a `k` that could never actually bind — at most `⌊n/2⌋` non-overlapping transactions fit in `n` days (each needs a distinct buy day and sell day), so once `k` reaches that ceiling, "at most k" and "unlimited" are the same problem, solvable by LC 122's O(n) greedy sum instead.

**Q5. Why is `buy[]` initialized to `Integer.MIN_VALUE` but `sell[]` to `0`?**
`sell[t]` (zero profit, `t` transactions not yet started) is validly reachable with no prices seen; `buy[t]` for `t ≥ 1` (holding, having started transaction `t`) is not reachable at all before any price is seen, and defaulting it to `0` would let `sell[t]` treat an impossible state as if it were freely available.

**Q6. Why does State Machine DP add zero extra practice problems this week?**
The four required problems already span every shape the pattern tests — unconstrained-with-fee (2 states), a structural coupling constraint (3 states), a small fixed transaction budget, and an arbitrary budget with its reduction case — so a fifth problem would necessarily repeat one of the four rather than add a new shape.

**Q7. What's the actual risk of running a Dead Letter Queue with no consumer watching it?**
Failures still get isolated from the main queue (so throughput/ordering isn't blocked), but silently accumulate unseen — the durability the DLQ provides only pays off if something monitors and acts on what lands there.

---

## Day 94 — Tree DP, and Dynamic Programming Closes Entirely

**Q1. What's genuinely new about Tree DP versus the tree recursion from Week 7?**
Not the postorder-combine shape itself (that's old) — it's that a return value can now **be** the complete DP state (as in House Robber III's pair), the same way `dp[i][state]` was the complete state on Days 92–93, rather than only ever supporting the parent's own separate computation.

**Q2. Why is House Robber III's brute force exponential, and what specifically fixes it?**
Each node gets recomputed from multiple ancestors' calls with no memoization — returning both `{notRobbed, robbed}` from a single postorder visit means a parent never needs to re-descend into a child's subtree to get an answer it might need, eliminating the recomputation entirely.

**Q3. Why can't Binary Tree Maximum Path Sum return everything a parent might need, the way House Robber III does?**
A path can bend at a node (go through both children), but a value handed to a parent can only extend in one direction, since the parent can only attach the node to its own single path — the "bending" answer and the "extendable upward" answer are different questions, and only the second can be a return value.

**Q4. Why does `Math.max(_, 0)` appear in the Max Path Sum solution?**
It encodes that a path may choose not to extend into a child at all; without the clamp, a strongly negative subtree would be added in and actively reduce a sum that would be better off simply stopping there.

**Q5. Reconcile "35 required DP problems" with "34 newly taught."**
35 counts every required *slot* across Weeks 12–14, including one (`LC 152`, Week 12 Day 83) that was fulfilled by recap rather than a fresh teach; 34 counts only the newly-taught problems. Both are correct, answering different questions.

**Q6. Name the one discipline every DP subtype in this series has shared.**
State precisely what `dp[...]` means before writing any recurrence, then prove that recurrence correct via an exhaustive, disjoint-case argument — never assert it.

**Q7. In Binary Tree Cameras, why does a `null` child return `COVERED_NO_CAMERA` instead of `NOT_COVERED`?**
If a missing child forced a camera at its parent, every leaf would appear to have an "uncovered" child and would wastefully force a camera at the leaf itself; treating "no child" as already-covered correctly defers that decision upward instead.

---

## Day 95 — Bit Manipulation Opens: Binary, Two's Complement, and XOR

**Q1. Derive the two's-complement negation formula and prove it.**
`-x = ~x + 1`. Proof: flipping every bit of an n-bit `x` gives `(2ⁿ-1) - x`; adding `1` gives `2ⁿ - x`; under mod-`2ⁿ` wraparound arithmetic, `x + (2ⁿ - x) = 2ⁿ ≡ 0`, which is exactly the defining property `-x` must satisfy.

**Q2. Why does Java need both `>>` and `>>>`, and when do they actually produce different results?**
`>>` sign-extends (preserves the value's sign, useful for arithmetic like floor division by a power of 2); `>>>` always fills with `0` (useful when the bits themselves are the point, not what they represent numerically). They produce identical results for any non-negative operand, since a non-negative number's sign bit is already `0` — they can only diverge when the sign bit being extended is `1`.

**Q3. Prove XOR's self-inverse property and explain why it's the key property for Single Number.**
From the truth table: `1^1=0` and `0^0=0`, so `a^a=0` for any `a`, in every bit position. Combined with associativity (order doesn't matter) and identity (`a^0=a`), XOR-ing an entire array cancels every paired value to `0`, leaving only the unpaired element.

**Q4. Derive `n & (n-1)`'s effect from binary subtraction, don't just state it.**
If `n`'s lowest set bit is at position `k`, subtracting `1` borrows through all `k` trailing zeros (each becomes `1`) until it reaches that `1` bit, which becomes `0`; bits above position `k` are untouched. ANDing the two: position `k` and below all AND to `0`; everything above is identical in both and passes through unchanged — the net effect is exactly `n` with its lowest set bit cleared.

**Q5. Is `hammingWeightNaive` really worse than the optimized version, given both are O(1)?**
Big-O equivalence is technically true and incomplete: the naive version always runs exactly 32 iterations; the optimized version runs exactly `popcount(n)` iterations, strictly fewer whenever `n` has any zero bits — a real, meaningful difference the O(1) label alone doesn't capture.

**Q6. Why does Bit Manipulation add zero extra practice problems on its opening day?**
Consistent with this series' practice for every brand-new top-level pattern (Trees, Heaps, Tries, Backtracking, Graphs, Union-Find, Dijkstra's, Dynamic Programming) — the opening day already carries the full foundational load, and adding a practice problem on top would trade depth on that foundation for shallow repetition.

**Q7. What's the actual difference between Eureka-style and Kubernetes-DNS-style service discovery?**
Eureka is client-side: each client queries a registry and picks an instance itself. Kubernetes/CoreDNS is server-side: a client does an ordinary DNS lookup, and the cluster's networking layer handles routing and load-balancing transparently beneath that lookup — the client never sees an instance list at all.

---

## Day 96 — Bit Manipulation Continues: Power of Two, and DP Fuses In

**Q1. Why does `n & (n-1) == 0` exactly characterize powers of two (and zero)?**
A power of two has exactly one set bit; clearing the lowest set bit (yesterday's proven identity) either empties a number with exactly one set bit, or leaves at least one set bit behind if there were two or more — the check is an iff, not a heuristic.

**Q2. Why is the `n > 0` guard required in Power of Two?**
Without it, `n=0` passes incorrectly: `0-1=-1` (all bits set), and `0 & -1 = 0`, wrongly matching the "cleared to zero" condition even though `0` has zero set bits, not one.

**Q3. What makes Counting Bits' recurrence a genuine DP recurrence, not just a formula?**
`dp[i]` is defined via `dp` at a strictly smaller, already-computed index (`i & (i-1)`, proven smaller by yesterday's identity) plus O(1) extra work — the exact shape every 1D DP recurrence in this series has had since Day 81.

**Q4. Give the alternate recurrence for Counting Bits and explain what it decomposes differently.**
`dp[i] = dp[i>>1] + (i&1)` — drops the lowest bit entirely via a shift, then adds it back only if that dropped bit was itself a `1`; a different but equally valid way of expressing "one smaller, already-known popcount, plus a local correction."

**Q5. Compare the complexity of counting bits independently per-number versus via DP, and justify the difference precisely.**
Independent: O(n log n) — each number's bit count costs time proportional to its own bit-width, and the average bit-width grows as the range grows. DP: O(n) — O(1) work per index by reusing an already-computed smaller result. Genuinely different this time, unlike yesterday's single-number O(1)-vs-O(1) comparison, because a *range* of numbers is involved, not one fixed-width value.

**Q6. Is a Kubernetes `Secret` encrypted?**
Not by default — it's base64-encoded, which is a reversible encoding, not a cipher; anyone with read access to the object can trivially decode it. Genuine encryption-at-rest requires additional configuration beyond using `Secret` alone.

---

## Day 97 — Isolating a Singleton: XOR Revisited, and Counting Mod 3

**Q1. Why is the sum-formula approach to Missing Number a real overflow risk, and what's the fix?**
`n*(n+1)` is computed before dividing by 2; for `n` roughly ≥46,341 this product itself can overflow `int`, silently wrapping (Day 10's exact mechanism). Fix: compute in `long`, matching Day 11's requirement for 4Sum.

**Q2. Prove, with a counterexample, why plain XOR fails on Single Number II.**
`2^2^2 = 2`, not `0` — three copies of the same value XOR to that value itself, since `2^2=0` then `0^2=2` again; a triple's bit contributes an odd count (`3 mod 2 = 1`) to plain XOR, so it doesn't reliably cancel.

**Q3. Derive per-bit frequency counting as a generalization, don't just state the mod-3 rule.**
XOR is really "count each bit's occurrences, take mod 2." Generalizing to groups of three: a tripled number contributes a multiple of 3 to each bit's count, and any multiple of 3 is `0 mod 3` — swap "mod 2" for "mod 3" and tripled bits correctly cancel, leaving only the true singleton's bits.

**Q4. What's the time complexity of the per-bit counting approach, and why?**
O(n) — 32 fixed outer iterations (bit positions) times an O(n) inner scan per bit, and 32 is a constant independent of input size.

**Q5. Why is Hamming Distance a reasonable "cheap" extra to add this week?**
It composes two already-fully-taught mechanisms — XOR (Day 95) to find differing positions, and set-bit counting (Days 95–96) to count them — with no new technique, near-zero marginal teaching cost, matching this series' past practice for exactly this kind of reinforcement addition.

**Q6. When should a Kubernetes workload be a `StatefulSet` instead of a `Deployment`?**
When replicas need a stable, persistent identity and their own dedicated storage that follows that specific replica across restarts — anything where "which instance, and its own data" matters, not just "N interchangeable copies."

---

## Day 98 — Isolating Two Singletons, Addition Without +, and Week 14 Consolidation

**Q1. Derive `n & (-n)` and state what it isolates.**
Writing `n`'s lowest set bit at position `k` as `X 1 0...0`, `-n = ~n+1` works out to `~X 1 0...0` (the same carry-ripple argument as `n-1`, applied to `~n` instead). ANDing: the prefix cancels (`X & ~X = 0` at every position), position `k` gives `1&1=1`, and everything below is `0&0=0` — leaving exactly `n`'s lowest set bit, alone.

**Q2. Why does partitioning by one differing bit correctly separate Single Number III's two answers?**
Every set bit of `unique1^unique2` is, by XOR's definition, a position where the two values differ — so partitioning on any one of them puts the two singletons in different groups by construction, while every paired number's identical copies always agree on that bit and stay together, keeping their cancellation intact within each group.

**Q3. Map XOR and AND to their roles in Sum of Two Integers, precisely.**
XOR's truth table matches "sum digit ignoring carry" exactly (`1^1=0`, matching `1+1`'s digit before carry); AND's truth table matches "generates a carry" exactly (`1&1=1`, the only case addition actually carries) — the loop repeatedly folds that shifted carry back in until none remains.

**Q4. What's the real time-complexity bound for Sum of Two Integers, stated precisely?**
O(1), bounded by the fixed 32-bit word size — the carry can propagate at most ~32 positions before it's shifted entirely out of the register, a bound tied to the type's width, not to the numeric value of the inputs.

**Q5. Reconcile Week 14's "16 total problems" against "14 required."**
14 is exactly `Week_14_Revised.md`'s own required count (6 DP + 8 Bit Manipulation); 16 includes the two extras added this week (`LC 968` on Day 94, `LC 461` on Day 97) — both numbers are correct, describing required-only versus required-plus-extra.

**Q6. What does Week 15's Maximum XOR of Two Numbers problem specifically require from two different earlier weeks?**
This week's Bit Manipulation mechanics (XOR, bit isolation) **and** Week 9's Trie structure, combined into a Bit Trie — a combination deliberately deferred since Week 9 specifically so it could pair with Bit Manipulation once this week existed.

---

## Index — All 46 Questions by Problem/Topic

| # | Topic | Day |
|---|---|---|
| 1 | State Machine DP definition | 92 |
| 2 | Why greedy breaks (LC 122 → LC 714) | 92 |
| 3 | Why Cooldown needs 3 states | 92 |
| 4 | Cooldown's structural constraint enforcement | 92 |
| 5 | State machine space optimization | 92 |
| 6 | Guarded switch case ordering | 92 |
| 7 | Sealed interface exhaustiveness | 92 |
| 8 | Transaction-count dimension `k` | 93 |
| 9 | Deriving `buy1` from the general recurrence | 93 |
| 10 | Same-day chaining in `buy2` | 93 |
| 11 | `k ≥ n/2` reduction | 93 |
| 12 | Sentinel values, `buy[]` vs `sell[]` | 93 |
| 13 | Why State Machine DP has zero extras | 93 |
| 14 | Dead Letter Queue risk with no consumer | 93 |
| 15 | Tree DP vs. Week 7 tree recursion | 94 |
| 16 | House Robber III's exponential brute force | 94 |
| 17 | Why Max Path Sum can't dual-return | 94 |
| 18 | The `Math.max(_, 0)` clamp | 94 |
| 19 | Reconciling 35 vs. 34 required DP problems | 94 |
| 20 | The one shared DP discipline | 94 |
| 21 | Binary Tree Cameras' `null` return value | 94 |
| 22 | Two's-complement negation, derived | 95 |
| 23 | `>>` vs `>>>`, proven | 95 |
| 24 | XOR self-inverse, proven | 95 |
| 25 | `n&(n-1)`, derived | 95 |
| 26 | O(1) vs. O(1)-but-fewer-iterations | 95 |
| 27 | Why zero extras on Bit Manipulation's opening day | 95 |
| 28 | Eureka vs. Kubernetes DNS discovery | 95 |
| 29 | `n&(n-1)==0` as an iff for powers of two | 96 |
| 30 | The `n>0` guard in Power of Two | 96 |
| 31 | Counting Bits as a genuine DP recurrence | 96 |
| 32 | Counting Bits' alternate recurrence | 96 |
| 33 | O(n log n) vs. O(n), precisely justified | 96 |
| 34 | Kubernetes `Secret` encryption misconception | 96 |
| 35 | Missing Number's sum-formula overflow risk | 97 |
| 36 | Counterexample: why XOR fails at triples | 97 |
| 37 | Per-bit frequency counting, derived | 97 |
| 38 | Per-bit counting's complexity | 97 |
| 39 | Why Hamming Distance is a cheap extra | 97 |
| 40 | StatefulSet vs. Deployment | 97 |
| 41 | `n&(-n)`, derived | 98 |
| 42 | Single Number III's partition argument | 98 |
| 43 | XOR/AND mapped to hardware addition | 98 |
| 44 | Sum of Two Integers' real complexity bound | 98 |
| 45 | Reconciling 16 vs. 14 problems this week | 98 |
| 46 | Maximum XOR's two-week prerequisite | 98 |
