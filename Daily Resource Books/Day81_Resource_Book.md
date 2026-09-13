# Day 81 — Dynamic Programming Begins

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 80 Resource Book](Day80_Resource_Book.md)
**Next ▶:** [Day 82 Resource Book](Day82_Resource_Book.md)
**Companion to:** Day 81 of `Week_12_Revised.md`

---

## Recap

Dijkstra's Algorithm closed yesterday. Today opens Dynamic Programming — by problem count, the single largest pattern in this entire plan (33 required problems, spanning this week and next). It is not, however, an entirely unfamiliar idea being dropped in cold. You've met its central move — "build today's best answer from yesterday's best answer" — twice already, without the formal name: Kadane's Algorithm (Week 3, Day 21) tracked a running best ending at each position, built from the running best one position back; Floyd-Warshall (Week 11, Day 73) built each `k`-th layer's all-pairs distances from the `(k-1)`-th layer's. Today names that shape and gives it a rigorous foundation.

The direct prerequisite is recursion (Week 2, Day 8), specifically the exercise where `fib(5)`'s call tree was traced by hand and showed `fib(2)` being recomputed repeatedly. That redundancy is today's entire motivation, re-derived precisely below — not asserted, traced.

---

## Learning Objectives

By the end of today, without notes:

1. Trace `fib(5)`'s naive recursive call tree by hand, count exactly how many times each subproblem is recomputed, and use that count to explain why memoization helps.
2. State the two properties a problem needs for DP to apply — optimal substructure and overlapping subproblems — and give an example of a problem with the first but not the second (why it doesn't need DP).
3. Implement Fibonacci three ways (naive recursion, top-down memoization, bottom-up tabulation), and state each version's exact time and space complexity, including *why* they differ.
4. State, in one precise sentence, what `dp[i]` represents for both of today's problems, and prove — via a disjoint-and-exhaustive case argument — that each recurrence is correct.

---

## Concept Dependency Map

```
Recursion, call stack, base/recursive case, fib(5) traced by hand (Day 8)
        │
        ├──▶ fib(5)'s call tree re-traced HERE — proves fib(2) is recomputed 3 times,
        │    fib(1) 5 times, fib(0) 3 times — the exact motivating waste
        │
        ▼
Dynamic Programming (NEW, formalized) ── needs: HashMap (Day 4, for the memo cache)
        │        already previewed informally twice: Kadane's (Wk3 D21), Floyd-Warshall (Wk11 D73)
        │        where Greedy fails: 0/1 Knapsack counter-example (Wk4 D23) — cited directly
        │        contrasted with Divide-and-Conquer (Wk8 D51's tree construction) — optimal
        │        substructure WITHOUT overlap, so nothing to cache
        ▼
   ┌────┴────┐
   ▼         ▼
Problem 1: Climbing Stairs (LC 70)      Problem 2: Min Cost Climbing Stairs (LC 746)
  fib(n) wearing a different name          same shape, cost-minimizing instead of way-counting
  dp[i] = dp[i-1] + dp[i-2]                dp[i] = cost[i] + min(dp[i-1], dp[i-2])
```

---

# Part 1 — Dynamic Programming, Formalized

### Concept Card — Dynamic Programming

**Prerequisites, confirmed:** recursion, the call stack, base/recursive cases (Week 2, Day 8). `HashMap` (Week 1, Day 4) — memoization is, mechanically, recursion with a `HashMap` cache bolted on, not a new storage idea. Arrays (Day 2).

**Re-deriving the motivation — `fib(5)`'s call tree, traced in full:**

```
                              fib(5)
                    ┌───────────┴───────────┐
                 fib(4)                    fib(3)
              ┌────┴────┐               ┌────┴────┐
           fib(3)     fib(2)          fib(2)     fib(1)=1
          ┌───┴───┐  ┌───┴───┐       ┌───┴───┐
       fib(2)  fib(1) fib(1) fib(0) fib(1) fib(0)
      ┌───┴───┐  =1     =1    =0     =1     =0
   fib(1)  fib(0)
     =1      =0
```

Count every call by its argument: `fib(5)`→1, `fib(4)`→1, `fib(3)`→**2** (once under `fib(4)`, once directly under `fib(5)`), `fib(2)`→**3** (under `fib(4)`'s `fib(3)`, under `fib(4)` directly, under `fib(5)`'s `fib(3)`), `fib(1)`→**5**, `fib(0)`→**3**. Total: **15 function calls** to compute a single value — and `fib(2)`, called 3 separate times, recomputes the *identical* answer each time, since it's a pure function of its input. This is not a coincidence of `n=5`; the call count itself grows roughly Fibonacci-like (`T(n) = T(n-1) + T(n-2) + 1`), meaning naive recursive `fib(n)` does **exponential** work to answer a question with only `n` genuinely distinct sub-answers.

**What Dynamic Programming is:** a technique for solving a problem by breaking it into subproblems, solving each **exactly once**, and reusing that result every time it's needed again — eliminating exactly the waste just traced above. Two implementation styles:
- **Memoization (top-down):** write the natural recursion, but check a cache before recursing, and populate the cache after computing a new result.
- **Tabulation (bottom-up):** build an array iteratively from the smallest subproblem upward — no recursion at all.

**The two properties a problem needs, precisely:**
- **Optimal substructure:** an optimal solution to the whole problem can be built directly from optimal solutions to its subproblems — you never need a *suboptimal* answer to a subproblem to construct the best answer to the full problem.
- **Overlapping subproblems:** the *same* subproblem recurs multiple times during a naive solve (exactly what the trace above demonstrates for `fib`).

**Both are required, not just the first — a precise counter-example, not a hand-wave:** Divide and Conquer (Week 8, Day 51 — constructing a binary tree from preorder/inorder traversals) has optimal substructure — the correct tree is built directly from the correct left and right subtrees — but **no overlapping subproblems**, because each recursive call operates on a strictly disjoint slice of the input array (the left subtree's index range and the right subtree's never intersect). There is nothing to cache, because nothing repeats. This is exactly why merge sort, quicksort, and Day 51's tree construction never get a "memoized version" — memoizing a function that never receives the same input twice buys nothing but cache-lookup overhead.

**Why greedy sometimes can't substitute for this — cited, not re-derived:** Week 4, Day 23 built a concrete counter-example showing 0/1 Knapsack defeats every greedy strategy tried — an early greedy choice can block a strictly better later combination, with no exchange argument able to recover the loss. That failure is *why* DP exists as a distinct tool: greedy commits to one choice per step and never reconsiders; DP effectively considers every choice's downstream consequence by construction, at the cost of examining more states.

**`fib(n)`, implemented three ways — the vehicle for the coding exercise below:**

```java
// 1. Naive recursion — exponential. Recomputes shared subproblems, exactly as traced above.
public static long fibNaive(int n) {
    if (n <= 1) return n;
    return fibNaive(n - 1) + fibNaive(n - 2);
}

// 2. Memoization (top-down) — recursion + HashMap cache (Day 4).
public static long fibMemo(int n, Map<Integer, Long> cache) {
    if (n <= 1) return n;
    if (cache.containsKey(n)) return cache.get(n);
    long result = fibMemo(n - 1, cache) + fibMemo(n - 2, cache);
    cache.put(n, result);
    return result;
}

// 3. Tabulation (bottom-up) — no recursion; build the array upward from the base case.
public static long fibTabulation(int n) {
    if (n <= 1) return n;
    long[] dp = new long[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

(`long`, not `int` — `fib(40)` alone fits comfortably in `int`, but Day 10/11's overflow habit says build in the safety margin rather than assume `n` never grows past this exercise.)

**Complexity, derived from a general framework — time = (number of distinct states) × (work per state); space = (number of distinct states), reducible if only a bounded lookback is needed:**

| Version | Time | Space | Why |
|---|---|---|---|
| Naive recursion | O(φⁿ) — exponential | O(n) | `n` distinct states, but each is *recomputed* every time it's needed — no caching, so work isn't bounded by state count alone; recursion depth `n` for the stack |
| Memoization | O(n) | O(n) cache + O(n) call stack | `n` distinct states, each computed exactly once (O(1) work per state beyond the two recursive lookups, now cache hits after the first) |
| Tabulation | O(n) | O(n) array | Same n states, each filled exactly once, no recursion overhead |

**Why memoized `fib(40)` is dramatically faster than naive `fib(40)` — the actual timing exercise:** naive recursion's call count for `n=40` runs into the *hundreds of millions* (the same `T(n)=T(n-1)+T(n-2)+1` growth traced above, at `n=40` instead of `n=5`) — a difference of roughly seven orders of magnitude in raw work compared to memoization's 40 cache-backed calls. This is exactly why the naive version should be visibly, uncomfortably slow in the timing exercise below, while both memoization and tabulation return near-instantly.

**Trade-offs — memoization vs. tabulation, honestly:**
- Memoization is **lazy** — it only ever computes states actually reached by the recursion, which matters when a problem's dependency structure means many states are never needed for a given input. Tabulation computes **every** state up to the target, even ones that might not have been strictly necessary.
- Memoization pays real recursion overhead (function-call stack frames) and, for large `n`, risks `StackOverflowError` (Day 8's exact exception, now with a concrete trigger) — tabulation is iterative and has no such risk.
- Tabulation's explicit iteration order is what makes space optimization straightforward (rolling only the last `k` values instead of a full array) — memoization's space is tied to cache size *and* call-stack depth, both harder to shrink below O(n).

**When to reach for DP — the signal, and the discipline:** "minimum/maximum number of ways," "can you reach/make exactly X," optimization over a sequence or grid where greedy is provably insufficient. Before writing any code: **state what `dp[i]` represents, in one precise sentence.** If that sentence can't be stated, there isn't a DP solution yet — only a hope that one exists.

**Common mistakes:**
- **⚠️ Missing or wrong base case.** Every recurrence bottoms out somewhere; get it wrong and every derived value is wrong too, often silently (no exception, just a quietly incorrect number).
- **⚠️ Wrong iteration order in tabulation.** If `dp[i]` depends on `dp[i-1]`, filling the array in any order other than increasing `i` reads an uncomputed (default-zero) cell instead of a real value — a correctness bug, not a performance one.
- **⚠️ Off-by-one in array sizing.** A `dp` array indexed up to `n` needs size `n+1`; this recurs constantly enough to check explicitly every time, not assume from habit.
- **⚠️ Reaching for greedy without checking optimal substructure holds under an exchange argument first** — see the 0/1 Knapsack citation above.

**Edge cases:** the base case(s) themselves (`n=0`, `n=1`) — always verify the recurrence isn't silently relied upon *before* it's valid to apply.

---

## Problem 1: Climbing Stairs (LeetCode 70, Easy) — Pattern: 1D DP

**Statement:** `n` stairs; each move advances 1 or 2 steps; count the number of distinct ways to reach the top.

**`dp[i]`, precisely:** the number of distinct ways to reach step `i`.

**Why `dp[i] = dp[i-1] + dp[i-2]` — a disjoint, exhaustive case argument, not an assertion:** every way to reach step `i` ends with exactly one final move, and only two final moves are possible: a 1-step from `i-1`, or a 2-step from `i-2`. These two cases are **disjoint** (a way ending in a 1-step is a different sequence of moves from one ending in a 2-step, even when both land on `i`) and **exhaustive** (no third kind of move exists). By the addition principle for counting disjoint cases, the total is the sum: `dp[i] = dp[i-1] + dp[i-2]`. Base cases: `dp[0]=1` (one way to be at the ground — take no steps at all), `dp[1]=1` (one way — a single 1-step).

This is `fib(n+1)` wearing a different name — `dp[0..5] = 1,1,2,3,5,8` is exactly `fib[1..6]`.

### Approach 1 — Brute force: naive recursion

```java
public int climbStairsNaive(int n) {
    if (n <= 1) return 1;
    return climbStairsNaive(n - 1) + climbStairsNaive(n - 2);
}
```

Time O(φⁿ), Space O(n) — identical shape to naive `fib`, same exponential redundancy already proven above.

### Approach 2 — Memoization (top-down)

```java
public int climbStairsMemo(int n, Map<Integer, Integer> cache) {
    if (n <= 1) return 1;
    if (cache.containsKey(n)) return cache.get(n);
    int result = climbStairsMemo(n - 1, cache) + climbStairsMemo(n - 2, cache);
    cache.put(n, result);
    return result;
}
```

Time O(n), Space O(n) cache + O(n) stack.

### Approach 3 — Tabulation (bottom-up)

```java
public int climbStairsTabulation(int n) {
    if (n <= 1) return 1;
    int[] dp = new int[n + 1];
    dp[0] = 1;
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

Time O(n), Space O(n).

### Approach 4 — Optimized: O(1) space

```java
public int climbStairs(int n) {
    if (n <= 1) return 1;
    int prev2 = 1, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

**Why O(1) space is valid here:** `dp[i]` only ever depends on the two immediately preceding values — nothing further back is ever read again once `dp[i]` is computed. Keeping the full array is strictly more than the recurrence needs; two rolling variables suffice.

**Worked trace, `n=5`:** `dp[0]=1, dp[1]=1, dp[2]=dp[1]+dp[0]=2, dp[3]=dp[2]+dp[1]=3, dp[4]=dp[3]+dp[2]=5, dp[5]=dp[4]+dp[3]=8`. **Answer: 8** — the 8 distinct 1-and-2-step sequences summing to 5: `1+1+1+1+1`, `1+1+1+2` (×3 orderings: `1121,1211,2111`), `1+2+2` (×3 orderings: `122,212,221`) — total `1+3+3+1=8` (the last `1` being `2+2+1`... let me recount precisely: sequences summing to 5 using parts of size 1 and 2 — by number of 2s used: zero 2s → `11111` (1 way); one 2 → four positions for the single 2 among three 1s and one 2, i.e. `C(4,1)=4` orderings of `{2,1,1,1}` → wait, need total length: one 2 + three 1s = 4 moves summing to `2+1+1+1=5`, arrangements = `4!/3!=4`; two 2s → two 2s + one 1 = 3 moves summing to `2+2+1=5`, arrangements = `3!/2!=3`. Total: `1+4+3=8`.** Matches.

**Complexity (final):** Time O(n), Space O(1).

**Edge cases:** `n=0` isn't in LeetCode's actual constraints (`n≥1`) but the recurrence handles it correctly regardless (`dp[0]=1`); `n=1` and `n=2` are exactly the base/first-derived cases, no special-casing needed beyond what's already there.

**💡 Interview Insight:** naming the `fib(n+1)` connection unprompted, then immediately generalizing ("any problem where you're counting paths through a graph of allowed moves, with a small fixed set of move sizes, reduces to this same lookback-sum shape") signals pattern fluency rather than memorization of one problem.

---

## Problem 2: Min Cost Climbing Stairs (LeetCode 746, Easy) — Pattern: 1D DP

**Statement:** `cost[i]` is the price of stepping on stair `i`. You may start for free at index `0` or index `1`. Each move advances 1 or 2 steps. Minimize total cost to reach "the top" — one step past the last index (`index = cost.length`, which itself has no cost, since it isn't a stair).

**`dp[i]`, precisely — this is the detail worth being exact about:** `dp[i]` is the minimum total cost paid by the point you use stair `i` as a launching point for your next move — i.e., `dp[i]` **already includes `cost[i]` itself.** (This differs subtly from "the cost to arrive at `i`," which would be a different, and here incorrect, quantity — arriving at `0` or `1` is explicitly free, but *leaving* either one still tolls its own `cost[i]`.)

**Why `dp[i] = cost[i] + min(dp[i-1], dp[i-2])`:** to be in a position to launch from stair `i`, you must have arrived via a final move from `i-1` or `i-2` — take whichever predecessor was cheaper to launch from, then add `cost[i]` itself, since standing on `i` and preparing to leave it always costs `cost[i]`, regardless of which predecessor got you there. Base cases: `dp[0] = cost[0]`, `dp[1] = cost[1]` — starting at either is free, so the *only* cost baked into `dp[0]` or `dp[1]` is each one's own launch toll. **Final answer:** `min(dp[n-1], dp[n-2])` — the top itself costs nothing to land on, so the answer is whichever of the two possible final launch points was cheaper to have already reached.

### Approaches 1–3 (brute force, memoization, tabulation)

Structurally identical to Problem 1's three tiers, with the recurrence and base cases swapped in — brute-force recursion re-solves overlapping `minCost(i)` calls exponentially; memoization caches each `i` once; tabulation fills a `dp[]` array bottom-up. Shown here at the optimized, space-reduced tier directly, since the progression is now established.

### Approach 4 — Optimized: O(1) space

```java
public int minCostClimbingStairs(int[] cost) {
    int n = cost.length;
    int prev2 = cost[0], prev1 = cost[1];
    for (int i = 2; i < n; i++) {
        int curr = cost[i] + Math.min(prev1, prev2);
        prev2 = prev1;
        prev1 = curr;
    }
    return Math.min(prev1, prev2);
}
```

**Worked trace:** `cost = [1,100,1,1,1,100,1,1,100,1]` (a well-known, genuinely non-trivial case — the cheap path must "hop over" two expensive stairs, not just alternate blindly).

| i | cost[i] | dp[i] = cost[i] + min(dp[i-1], dp[i-2]) | dp[i] |
|---|---|---|---|
| 0 | 1 | base case | 1 |
| 1 | 100 | base case | 100 |
| 2 | 1 | 1 + min(100, 1) | 2 |
| 3 | 1 | 1 + min(2, 100) | 3 |
| 4 | 1 | 1 + min(3, 2) | 3 |
| 5 | 100 | 100 + min(3, 3) | 103 |
| 6 | 1 | 1 + min(103, 3) | 4 |
| 7 | 1 | 1 + min(4, 103) | 5 |
| 8 | 100 | 100 + min(5, 4) | 104 |
| 9 | 1 | 1 + min(104, 5) | 6 |

**Answer: `min(dp[9], dp[8]) = min(6, 104) = 6`.** Notice `dp[6]` correctly ignores the expensive `dp[5]=103` in favor of the much cheaper `dp[4]=3` — the recurrence "hops over" the costly stair 5 automatically, exactly the behavior the `min()` is there to capture, with no special-casing for "expensive stairs" anywhere in the code.

**Complexity:** Time O(n), Space O(1).

**Edge cases:** `cost.length == 2` (LeetCode's minimum) — the loop never executes, and the answer is `min(cost[1], cost[0])` directly, correctly handled since `prev1`/`prev2` are seeded from the base cases before the loop even starts.

**⚠️ Common Mistake:** defining `dp[i]` as "cost to *arrive* at `i`" instead of "cost through *leaving* `i`" — this swaps which index the base cases and final answer should reference, and is exactly the kind of subtle off-by-one that looks plausible until traced against a concrete example (as done above).

**💡 Interview Insight:** stating the precise `dp[i]` definition — specifically flagging that it includes the *current* stair's own cost, not just the cost to arrive — before writing the recurrence is the single strongest signal on this problem; the recurrence itself is nearly identical to Problem 1's, but silently getting the base case direction backwards produces a plausible-looking, confidently wrong answer.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** `DPFoundations` class implementing the three-Fibonacci exercise above, with timing printed for each.

```java
public class DPFoundations {
    public static void main(String[] args) {
        int n = 40;

        long start = System.nanoTime();
        long naiveResult = fibNaive(n);
        long naiveTime = System.nanoTime() - start;

        start = System.nanoTime();
        long memoResult = fibMemo(n, new HashMap<>());
        long memoTime = System.nanoTime() - start;

        start = System.nanoTime();
        long tabResult = fibTabulation(n);
        long tabTime = System.nanoTime() - start;

        System.out.printf("Naive:  fib(%d)=%d, %d ms, O(n) space (call stack only)%n",
                n, naiveResult, naiveTime / 1_000_000);
        System.out.printf("Memo:   fib(%d)=%d, %d ms, O(n) space (cache + call stack)%n",
                n, memoResult, memoTime / 1_000_000);
        System.out.printf("Tab:    fib(%d)=%d, %d ms, O(n) space (array only)%n",
                n, tabResult, tabTime / 1_000_000);
    }
    // fibNaive, fibMemo, fibTabulation as defined in the Concept Card above
}
```

**Definition of done:** pushed, with a comment on each approach's space complexity (per the `printf` lines above — worth stating explicitly in comments too, not just the console output).

---

## Career Block Guide (1 hr)

**LinkedIn Post 17 — timing naive recursion vs. memoization.** A draft to adapt, not copy verbatim:

> Timed three ways of computing Fibonacci(40) today: naive recursion, memoized recursion, and bottom-up tabulation.
>
> Naive recursion takes a visibly, uncomfortably long time — hundreds of millions of redundant function calls, because it recomputes the exact same sub-answers over and over without remembering anything. Memoization and tabulation both finish essentially instantly — 40 operations instead of hundreds of millions.
>
> The interesting part isn't that caching helps — it's *why*: `fib(5)`'s call tree alone recomputes `fib(2)` three separate times, always getting the identical answer. Multiply that redundancy out to `n=40` and the gap between "remembering what you already solved" and "solving it again every time" becomes the difference between instant and unusably slow. First real Dynamic Programming day of SDE-2 prep — expecting this pattern to show up constantly for the next two weeks.

**Networking:** identify 2 Tier B companies to apply to this week.

---

## Day 81 — Interview Questions

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

## Daily Deliverable Check

- [ ] Climbing Stairs (LC 70) and Min Cost Climbing Stairs (LC 746) solved through all four tiers (brute force → memoization → tabulation → space-optimized), pushed to `dsa-java/dynamic-programming/`.
- [ ] Can state, from memory, the precise `dp[i]` definition for both problems, and prove each recurrence via a disjoint-and-exhaustive case argument.
- [ ] Can trace `fib(5)`'s call tree and state the exact recomputation counts (`fib(2)`×3, `fib(1)`×5, `fib(0)`×3) without notes.
- [ ] `DPFoundations` timing comparison pushed, with space-complexity comments on each approach.
- [ ] LinkedIn Post 17 published.

---

## What Tomorrow Assumes You Already Know Cold

Day 82 needs today's `dp[i] = dp[i-1] + dp[i-2]`-shaped recurrence, and the "state `dp[i]`'s definition before coding" discipline, fully reflexive — House Robber reuses the identical two-steps-back lookback shape tomorrow, with a take-or-skip decision replacing today's always-add. It also assumes today's O(1) space-optimization instinct (rolling variables instead of a full array) is now automatic, since House Robber's own hint leans on it directly without re-deriving why it's valid.
