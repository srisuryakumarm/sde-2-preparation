# Day 85 — 1D DP Continues: Counting Combinations, LIS Two Ways, and 0/1 Knapsack Begins

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 84 Resource Book](Day84_Resource_Book.md)
**Next ▶:** [Day 86 Resource Book](Day86_Resource_Book.md)
**Companion to:** Day 85 of `Week_13_Revised.md`

---

## Recap

Yesterday (Day 84) closed with Coin Change — the pattern's first genuinely *unbounded* problem, and the first time this series' `dp[]` definition didn't track *which* coins built a sub-answer, only how many were used. Day 84 also flagged something explicitly, without resolving it: for **minimizing** (fewest coins to make an amount), the order of the two loops (coins outer vs. amount outer) doesn't matter. It was flagged, at the time, that a future **counting** problem would need a specific order, for the opposite reason. Today is that day — Coin Change II asks "how many *combinations*," and the fix isn't optional polish, it's the difference between a correct answer and a silently wrong one that still compiles and runs.

Today also opens **0/1 Knapsack** — Partition Equal Subset Sum. This is not a variant of yesterday's Unbounded Knapsack framing; it's a genuinely distinct counterpart, last mentioned by name all the way back in Week 4, Day 23, purely as a worked example of *where greedy fails* (no DP solution was built at the time — Dynamic Programming didn't exist yet as a named tool). Today builds the real thing, from zero, and the bounded-vs-unbounded distinction turns out to hinge on a single loop-direction choice — the second loop-order lesson of the day, for a completely different underlying reason than the first.

This week's 21 required problems were checked against the full 202-problem cumulative inventory (Weeks 1–12) before writing began: all confirmed genuinely new, no recaps needed this week. Today's one addition beyond the plan — Combination Sum IV — was checked against `Week_14_Revised.md`'s required list (State Machine DP and Tree DP, eight problems, none overlapping) and against the inventory: clean.

---

## Learning Objectives

By the end of today, without notes:

1. State precisely why Coin Change II's loop order (coins outer, amount inner) is *required* for correctness when counting combinations — not just recite the rule, but produce a concrete input where the wrong order overcounts, and explain exactly what it overcounts.
2. Contrast that combinations-vs-permutations loop-order rule against Combination Sum IV's *opposite* rule, and state in one sentence what property of the counting question flips it.
3. Trace the O(n log n) Longest Increasing Subsequence algorithm by hand on a non-trivial array, including at least one "replace" and one "append" step, and explain why the resulting `tails` array is not, in general, an actual subsequence of the input — while its *length* is still guaranteed correct.
4. Define 0/1 Knapsack precisely, state how its recurrence and required loop direction differ from Unbounded Knapsack, and produce a concrete input where getting the loop direction wrong silently reuses an item that should only be usable once.

---

## Concept Dependency Map

```
Week 12, Day 84 — Coin Change (LC 322): Unbounded Knapsack opens
  ├─ dp[amt] = min coins to make amt; sentinel amt+1, not MAX_VALUE
  ├─ loop order flagged as NOT mattering here (minimization)
  │  ...but flagged as an open question for counting. Today closes it.
  │
  ▼
TODAY, Part A — Coin Change II (LC 518): counting combinations, unbounded
  ├─ resolves yesterday's flag: coins OUTER, amount INNER — proven, not asserted
  └─ EXTRA: Combination Sum IV (LC 377) — same recurrence SHAPE,
     opposite loop order, because it counts PERMUTATIONS, not combinations

Week 12, Day 81 — DP formalized; Day 8 — recursion; Day 3 — Big-O
  │
  ▼
TODAY, Part B — Longest Increasing Subsequence (LC 300)
  ├─ Approach 1: O(n²) — dp[i] = LIS ending exactly at i (new lookback SHAPE:
  │  unbounded look-back distance, gated by a value comparison, not a fixed
  │  distance or a dictionary — a fourth 1D DP shape, after Day 81-84's three)
  └─ Approach 2: O(n log n) — patience sorting + binary search (Week 4-5's
     Binary Search template, reused directly, not re-derived)

Week 4, Day 23 — 0/1 Knapsack NAMED ONLY, as a greedy-failure example
Day 84's Unbounded Knapsack (as the thing to contrast against)
  │
  ▼
TODAY, Part C — Partition Equal Subset Sum (LC 416): 0/1 Knapsack, built fresh
  ├─ NEW: 0/1 Knapsack formalized — each item AT MOST ONCE
  ├─ 1D space optimization requires DECREASING capacity loop —
  │  the day's second, unrelated loop-order rule
  └─ 🔗 forward: Target Sum (Day 86) is the exact same shape, counting instead
     of yes/no
```

---

## Part A — Coin Change II: Resolving Yesterday's Flag

## Problem 9: Coin Change II (LC 518, Medium) — Pattern: DP Combinations (Unbounded Knapsack)

**Statement:** Given an integer `amount` and an array `coins` of distinct denominations (unlimited supply of each), return the number of distinct **combinations** that make up `amount`. Order doesn't distinguish two combinations — `[1,2]` and `[2,1]` are the *same* combination.

### Approach 1 — Brute force: recursion over (coin index, remaining amount)

```java
public static int changeBruteForce(int amount, int[] coins) {
    return countWays(coins, coins.length - 1, amount);
}

private static int countWays(int[] coins, int coinIndex, int remaining) {
    if (remaining == 0) return 1;                 // exact match — one valid combination
    if (remaining < 0 || coinIndex < 0) return 0;  // overshot, or ran out of coin types

    // At each coin, either use it again (stay on the same index — unbounded)
    // or move on and never use it again (index - 1).
    int useThisCoin = countWays(coins, coinIndex, remaining - coins[coinIndex]);
    int skipThisCoin = countWays(coins, coinIndex - 1, remaining);
    return useThisCoin + skipThisCoin;
}
```

Every recursive call either consumes one more of the current coin or permanently retires it — this is exactly what forces each combination to be built with its coins in one fixed, canonical order (by coin index), which is precisely what prevents `[1,2]` and `[2,1]` from being counted as different combinations. **Complexity: Time O(amount^n) in the worst case** (each of the n coin types can be "used again" up to `amount` times before retiring) **— exponential; Space O(amount)** for the recursion depth.

### Approach 2 — Optimized: tabulation, coins outer, amount inner

```java
public static int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1;   // exactly one way to make amount 0: use no coins

    for (int coin : coins) {                       // COINS is the OUTER loop
        for (int amt = coin; amt <= amount; amt++) { // amount is the INNER loop
            dp[amt] += dp[amt - coin];
        }
    }
    return dp[amount];
}
```

**`dp[amt]` represents:** the number of combinations of coins (order-independent) that sum to exactly `amt`, considering only the coin denominations processed *so far* in the outer loop. That last clause is the entire mechanism — hold onto it for the proof below.

### Why coins-outer, amount-inner is required — not a style preference

This is yesterday's flagged question, now answered. Try `amount = 5`, `coins = [1, 2, 5]`, wrong order first:

```java
// WRONG for counting combinations — amount outer, coins inner
int[] dp = new int[amount + 1];
dp[0] = 1;
for (int amt = 1; amt <= amount; amt++) {
    for (int coin : coins) {
        if (amt >= coin) dp[amt] += dp[amt - coin];
    }
}
```

Run this and `dp[5]` comes out to **9**, not the correct **4**. The four genuine combinations are `{5}`, `{1,2,2}`, `{1,1,1,2}`, `{1,1,1,1,1}`. The wrong-order version overcounts because, for a fixed `amt`, it iterates over *every* coin on *every* pass — meaning by the time it considers using a `2`, it's already allowed a `1` to appear before or after that `2` in different passes, effectively distinguishing `1+2+2` from `2+1+2` from `2+2+1` as if they were different combinations, even though the *count* value doesn't track order directly. The amount-outer structure answers "in how many ordered ways" (permutations), not "in how many unordered ways" (combinations) — which is exactly what tomorrow's Combination Sum IV extra practice actually wants, and exactly why swapping the loops solves a *different*, legitimately useful problem instead of a buggy version of this one.

**Why coins-outer fixes it:** processing one coin denomination completely (updating every reachable `dp[amt]` for that coin) *before* moving to the next coin denomination means that, by construction, every combination counted in `dp[amt]` uses coins in one fixed relative order — smallest-processed-coin-first. A combination like `{1,2,2}` gets counted exactly once, when the `2`-coin's pass reaches `amt=5` and adds in whatever `dp[3]` already contained *after* the `1`-coin's pass had already run. There is no way to "revisit" the `1`-coin's contribution after the `2`-coin's pass has started, so no combination can be assembled with its coins in more than one relative order.

**Worked trace:** `amount = 5`, `coins = [1, 2, 5]`.

| After processing coin | dp[0] | dp[1] | dp[2] | dp[3] | dp[4] | dp[5] |
|---|---|---|---|---|---|---|
| (init) | 1 | 0 | 0 | 0 | 0 | 0 |
| coin=1 | 1 | 1 | 1 | 1 | 1 | 1 |
| coin=2 | 1 | 1 | 2 | 2 | 3 | 3 |
| coin=5 | 1 | 1 | 2 | 2 | 3 | **4** |

Row "coin=1": `dp[amt] += dp[amt-1]` for `amt=1..5` — with only 1s available, there's exactly one way to make every amount. Row "coin=2": `dp[2] += dp[0]` → 2; `dp[3] += dp[1]` → 2; `dp[4] += dp[2]` → 1+2=3; `dp[5] += dp[3]` → 1+2=3. Row "coin=5": `dp[5] += dp[0]` → 3+1=4. Final `dp[5] = 4`, matching the four combinations listed above exactly.

**Complexity: Time O(n × amount)** where n is the number of coin denominations (one pass per coin, each pass touching up to `amount` cells) — **Space O(amount)**, the 1D dp array (already space-optimized from a conceptual 2D `dp[coinIndex][amount]` table, the same 2D→1D collapsing move Day 84 used for Coin Change).

**Edge cases:** `amount = 0` → `dp[0] = 1` by definition (the empty combination), correct with no special-casing. `coins` doesn't contain 1 and `amount` isn't reachable (e.g. `coins=[3]`, `amount=5`) → `dp[amount]` stays `0`, correctly. Single coin exactly divides amount (e.g. `coins=[5]`, `amount=15`) → exactly 1 combination, `dp[15]=1`.

> 💡 **Interview Insight:** the strongest opening move here is naming the loop-order requirement *before* writing code, and stating the one-sentence reason: "coins-outer processes each denomination's contribution exactly once across the whole array, which is what forces a canonical order onto every combination counted." An interviewer who has seen candidates silently get 9 instead of 4 will recognize this as real understanding, not memorized code.

---

## Extra Practice: Combination Sum IV (LC 377, Medium) — Pattern: DP Permutations (Unbounded Knapsack, Loop Order Flipped)

**Why this pairs directly with Coin Change II:** same numbers, same "unlimited reuse," nearly identical code — but this problem's name is a well-known trap. Despite being called "Combination Sum," it counts **ordered sequences** (permutations), not combinations. `nums = [1,2]`, `target = 3` counts `[1,1,1]`, `[1,2]`, **and** `[2,1]` as three *separate* answers. This is precisely the "wrong order" version of Coin Change II from above — except here, that's the *correct* behavior, because the problem being asked is genuinely different.

**Statement:** Given a distinct-integer array `nums` (positive integers, unlimited reuse of each) and a target, return the number of possible *ordered* combinations that add up to target.

```java
public static int combinationSum4(int[] nums, int target) {
    int[] dp = new int[target + 1];
    dp[0] = 1;

    for (int t = 1; t <= target; t++) {         // TARGET is the OUTER loop this time
        for (int num : nums) {                  // nums is the INNER loop
            if (t >= num) {
                dp[t] += dp[t - num];
            }
        }
    }
    return dp[target];
}
```

**Why the loop order is flipped, precisely:** target-outer, nums-inner means that for each amount `t`, *every* number gets a chance to be "the number placed last" to reach `t`. Since `dp[t]` accumulates a separate contribution from `dp[t-num]` for every `num` in the array, and `dp[t-num]` itself already contains contributions from every possible predecessor sequence, the same set of numbers reached in a different order (e.g., ending in `1` vs. ending in `2`) gets counted as a distinct sequence — which is exactly "ordered."

**Worked trace:** `nums = [1,2,3]`, `target = 4`.

| t | dp[t] computed as | value |
|---|---|---|
| 0 | (base case) | 1 |
| 1 | dp[0] (using 1) | 1 |
| 2 | dp[1] (using 1) + dp[0] (using 2) | 1+1 = 2 |
| 3 | dp[2] + dp[1] + dp[0] | 2+1+1 = 4 |
| 4 | dp[3] + dp[2] + dp[1] | 4+2+1 = 7 |

The 7 sequences: `(1,1,1,1)`, `(1,1,2)`, `(1,2,1)`, `(2,1,1)`, `(2,2)`, `(1,3)`, `(3,1)` — confirming `(1,2,1)` and `(2,1,1)` and `(1,1,2)` are counted as three separate answers, which is the entire point.

**Complexity: Time O(target × n), Space O(target)** — identical shape to Coin Change II, only the loop order and the question being answered differ.

### 🔑 Key Takeaway — the day's first loop-order table

| Problem | Question | Loop order | Why |
|---|---|---|---|
| Coin Change (Day 84) | min coins (optimize) | either order works | minimization doesn't care how a value was reached, only its size |
| Coin Change II (today) | # combinations (unordered count) | **coins outer** | forces one canonical order per combination — no double-count |
| Combination Sum IV (extra) | # sequences (ordered count) | **amount/target outer** | lets every number be "placed last" at every amount — that's what "ordered" means |

💡 **Interview Insight:** if an interviewer asks "how would you count combinations instead of sequences here?" — or vice versa — the entire answer is "swap which loop is outer," not a new algorithm. Naming that unprompted is a strong signal.

---

## Part B — Longest Increasing Subsequence, Two Ways

## Problem 10: Longest Increasing Subsequence (LC 300, Medium) — Pattern: 1D DP / Binary Search

**Statement:** Given an integer array `nums`, return the length of the longest strictly increasing subsequence (not necessarily contiguous).

### Approach 1 — O(n²): dp[i] = LIS length ending exactly at index i

```java
public static int lengthOfLISQuadratic(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1);      // every single element is an LIS of length 1, by itself

    int longest = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        longest = Math.max(longest, dp[i]);
    }
    return longest;
}
```

**`dp[i]` represents:** the length of the longest increasing subsequence that ends *exactly* at index `i` (not just "somewhere in `nums[0..i]`" — the distinction matters, because it's what makes "look back at every valid `j`" correct instead of only "look at `dp[i-1]`"). This is a new 1D DP shape this series hasn't used before: the lookback distance isn't fixed (Climbing Stairs' always `i-1, i-2`), isn't a single decision (House Robber's take-or-skip), isn't validity-gated by a legality check on one fixed-distance term (Decode Ways), and isn't a dictionary-gated OR (Word Break) — it's a **value-gated OR across every earlier index**, where "gated" means `nums[j] < nums[i]`, not a lookup.

**Worked trace:** `nums = [10,9,2,5,3,7,101,18]`.

| i | nums[i] | valid j's (nums[j]<nums[i]) | dp[i] |
|---|---|---|---|
| 0 | 10 | none | 1 |
| 1 | 9 | none | 1 |
| 2 | 2 | none | 1 |
| 3 | 5 | j=2 (2) | dp[2]+1 = 2 |
| 4 | 3 | j=2 (2) | dp[2]+1 = 2 |
| 5 | 7 | j=2,3,4 (2,5,3) | max(dp[2],dp[3],dp[4])+1 = max(1,2,2)+1 = 3 |
| 6 | 101 | j=0..5, all qualify | max of all dp[0..5]+1 = 3+1 = 4 |
| 7 | 18 | j=2,3,4,5 (2,5,3,7) | max(dp[2..5])+1 = 3+1 = 4 |

`longest = max(dp) = 4`, matching the known answer (e.g. `[2,3,7,101]` or `[2,3,7,18]`, both length 4).

**Complexity: Time O(n²)** — for every `i`, scan every earlier `j`. **Space O(n)** for the dp array.

**Edge cases:** empty array → `0` (guard before the loop, or the loop simply never runs and you return the fill value — decide explicitly rather than let it fall out by accident). All-decreasing array → every `dp[i]=1`, answer `1`. All-equal elements → `nums[j] < nums[i]` is strict, so no index ever qualifies against an equal value, correctly giving `1` (the problem asks *strictly* increasing).

### Approach 2 — Optimized: O(n log n) via patience sorting + binary search

```java
public static int lengthOfLIS(int[] nums) {
    int[] tails = new int[nums.length];  // tails[k] = smallest tail value among all
    int size = 0;                        // increasing subsequences of length k+1 found so far

    for (int num : nums) {
        int lo = 0, hi = size;           // binary search for the first index where tails[idx] >= num
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (tails[mid] < num) {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        tails[lo] = num;                 // either overwrites an existing (better) tail, or extends
        if (lo == size) size++;
    }
    return size;
}
```

**The reframe:** instead of asking "what's the longest subsequence ending at each index," track, for every achievable *length* `k`, the **smallest possible value** that a length-`k` increasing subsequence could end on. Call this `tails[k]`. Smaller is strictly better — a smaller tail is easier for a future number to extend. `tails` is maintained sorted at all times (provably — see below), which is exactly what makes binary search valid on it.

**Why binary search on `tails` is valid — the invariant, stated and defended:** claim: `tails[0..size)` is always strictly increasing. Proof by induction on the number of elements processed. Base case: after one element, `tails` has one entry — trivially "increasing." Inductive step: suppose `tails` is increasing before processing `num`. Binary search finds the first index `lo` with `tails[lo] >= num`. Two cases: (a) `lo == size` — `num` is larger than everything currently in `tails`, so appending it at the end preserves strictly-increasing order. (b) `lo < size` — `tails[lo]` is being replaced by the strictly smaller-or-equal value `num` (since `num <= tails[lo]` by how `lo` was found, and `num` can't equal `tails[lo]` in a *strictly* increasing subsequence context, so `num < tails[lo]`); `tails[lo-1] < num` holds because `lo` was the *first* index where the old `tails[lo-1] < num` check failed, meaning `tails[lo-1] < num` — so the array remains strictly increasing after the replacement. Either way, the invariant survives, which is exactly what licenses binary search on it at every step, not just the first.

**Why replacing a tail with a smaller value never loses a valid answer:** the length associated with `lo` is `lo+1` (0-indexed). Replacing `tails[lo]` with a smaller `num` doesn't shrink any length already achieved — it only makes that *specific* length easier to extend later, since a smaller tail is a strictly weaker requirement for the next number to beat. No subsequence "disappears"; the algorithm is only ever tracking the *best possible tail* per length, never the count of subsequences or their actual contents.

**Worked trace:** `nums = [10,9,2,5,3,7,101,18]` (numerically re-verified before writing):

| num | binary search result | action | tails after |
|---|---|---|---|
| 10 | not found (size=0) | append | [10] |
| 9 | idx 0 (10≥9) | replace | [9] |
| 2 | idx 0 (9≥2) | replace | [2] |
| 5 | not found | append | [2, 5] |
| 3 | idx 1 (5≥3) | replace | [2, 3] |
| 7 | not found | append | [2, 3, 7] |
| 101 | not found | append | [2, 3, 7, 101] |
| 18 | idx 3 (101≥18) | replace | [2, 3, 7, 18] |

Final `size = 4`. Same answer as Approach 1.

**⚠️ Common Mistake — `tails` is not the actual LIS.** Look at the final row: `tails = [2, 3, 7, 18]`. That *happens* to be a real increasing subsequence of the input here, but that's a coincidence of this particular example. Consider `nums = [3,4,5,1]`: `tails` ends up as `[1,4,5]` — but `1` appears *after* `4` and `5` in the original array, so `"1,4,5"` is not a valid subsequence at all (it's not even in increasing index order). `tails` only ever guarantees the *length* is correct — reconstructing the actual subsequence (if asked) needs a separate parent-pointer array recorded alongside each replace/append, not a read of the final `tails` array itself.

**Complexity: Time O(n log n)** — n elements, O(log n) binary search per element. **Space O(n)** for the tails array.

**Edge cases:** same as Approach 1 — empty array, strictly decreasing, all-equal.

> 💡 **Interview Insight:** this is one of the most commonly-asked DP-to-binary-search upgrades in tier-1 interviews specifically *because* most candidates can produce the O(n²) version but stall on the O(n log n) one. State the O(n²) version first, explicitly, then say "this can be improved to O(n log n) using patience sorting" before diving in — naming the technique by name signals you've seen this exact upgrade before, not that you're improvising it live.

---

## Extension (if time allows): Number of Longest Increasing Subsequence (LC 673, Medium)

A direct extension of Approach 1 above, not a new mechanism — worth a fast pass while the O(n²) shape is fresh, but safe to skip under time pressure and return to later; nothing on Day 86 depends on it.

**Statement:** Return not just the length of the LIS, but *how many* distinct longest increasing subsequences exist.

```java
public static int findNumberOfLIS(int[] nums) {
    int n = nums.length;
    int[] length = new int[n];   // length[i] = LIS length ending at i (same as before)
    int[] count = new int[n];    // count[i] = number of LIS's of that length ending at i
    Arrays.fill(length, 1);
    Arrays.fill(count, 1);

    int maxLength = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                if (length[j] + 1 > length[i]) {
                    length[i] = length[j] + 1;   // strictly longer path found — reset, inherit j's count
                    count[i] = count[j];
                } else if (length[j] + 1 == length[i]) {
                    count[i] += count[j];        // an equally-long path found — accumulate, don't replace
                }
            }
        }
        maxLength = Math.max(maxLength, length[i]);
    }

    int total = 0;
    for (int i = 0; i < n; i++) {
        if (length[i] == maxLength) total += count[i];
    }
    return total;
}
```

**Why the `else if` (not just `if length[j]+1 >= length[i]`) matters:** finding a strictly *longer* path means every previous way of reaching `dp[i]` is now irrelevant — `count[i]` must be *replaced* with `count[j]`, not added to. Finding an *equally long* path means it's a genuinely different way to reach the same best length — `count[i]` must *accumulate* `count[j]`. Conflating these two into one `>=` branch silently inflates the count by keeping stale contributions around after a strictly-better path invalidates them.

**Complexity: Time O(n²), Space O(n)** — same shape as Approach 1 above, with a second array tracked alongside.

---

## Part C — 0/1 Knapsack, Built From Zero

### 🔑 NEW CONCEPT: 0/1 Knapsack

**Prerequisites (confirmed):** Unbounded Knapsack's mechanism (Day 84) — specifically, as the thing being contrasted against. The named-but-unbuilt 0/1 Knapsack greedy-failure example from Week 4, Day 23 (a worked example of why an early greedy choice can block a strictly better later combination, with no swap able to recover the loss — that's *why* greedy fails here, not *how* to solve it correctly; today builds the actual solution). 1D DP tabulation and space-optimization from every DP day so far.

**What it is:** given a set of items, each with a weight and a value, and a knapsack with a fixed weight capacity, choose a subset of items — **each usable at most once** — maximizing total value without exceeding capacity. Today's specific problem (Partition Equal Subset Sum) is the simplest possible flavor: every "value" equals its own "weight" (so maximizing value and hitting an exact target weight become the same question), and the question is yes/no reachability rather than an optimum.

**Why it's a genuinely different mechanism from Unbounded Knapsack, not a variant:** Unbounded Knapsack's entire mechanism (Day 84) falls out of *not tracking which items built a sub-answer* — that's precisely what allows unlimited reuse. 0/1 Knapsack needs the opposite guarantee: once an item is used, it must become unavailable for the *rest of the current item's own consideration*. That requires the DP to distinguish "the state before this item was considered" from "the state after" — which a naive 1D array, updated in the same direction as Unbounded Knapsack, cannot do, because it would let the just-updated cell feed back into itself within the same item's pass.

**The mechanism, precisely — 2D first, then space-optimized:**

Conceptually, `dp[i][j]` = *can (or how) the first `i` items achieve a subset summing to exactly `j`*, built as:

```
dp[i][j] = dp[i-1][j]                              // don't use item i
           OR dp[i-1][j - weight[i]]                // use item i (only if j >= weight[i])
```

Note the `i-1` on *both* branches — every transition reads only from the **previous item's row**, never the current one. That's the entire bounded-ness of the recurrence: item `i` genuinely cannot be used twice, because using it moves you to row `i-1`, not row `i`.

Space-optimizing `dp[i][j]` down to a 1D `dp[j]` (dropping the item dimension, the same 2D→1D collapse Day 84 used) is where the real subtlety lives:

```java
public static boolean canPartition(int[] nums) {
    int totalSum = 0;
    for (int num : nums) totalSum += num;
    if (totalSum % 2 != 0) return false;      // an odd total can never split into two equal halves
    int target = totalSum / 2;

    boolean[] dp = new boolean[target + 1];
    dp[0] = true;                             // sum 0 is always reachable — use no items

    for (int num : nums) {
        for (int j = target; j >= num; j--) {  // DECREASING — the load-bearing detail
            dp[j] = dp[j] || dp[j - num];
        }
    }
    return dp[target];
}
```

**Why the inner loop must run in DECREASING order — proved with a minimal, checkable counterexample:** consider `nums = [3]` and (for illustration, ignoring the even-total guard for a moment) a target of `6`. There is exactly one `3` available, so `6` should be **unreachable** — `false`. Run the loop *increasing* instead (`for j = num to target`) and trace it: `dp[3] = dp[3] || dp[0] = true` (correct so far — one `3` reaches sum 3). Then, in the **same pass, for the same single item**, `dp[6] = dp[6] || dp[3]` — but `dp[3]` was *just* set to `true` moments earlier in this exact pass, by this exact item. The increasing loop lets the single `3` "reach back and use itself again," silently reusing one item twice and reporting `true` for a target that's genuinely impossible with only one `3` available. Running the loop **decreasing** instead evaluates `dp[6]` (using the old, pre-this-item value of `dp[3]`, which is `false`) *before* `dp[3]` itself gets overwritten later in the same pass — so `dp[6]` correctly stays `false`. (Verified directly: increasing-loop `dp[6]` with `nums=[3]` computes `true`; decreasing-loop computes `false`, the correct answer.)

This is the day's **second** loop-order rule — unrelated to the coins-outer/amount-inner rule from Part A. That one was about combinations vs. permutations; this one is about bounded vs. unbounded reuse. Confusing the two is a real, common mistake — the fix for a "used more than once" bug and the fix for a "double-counted as different orders" bug look superficially similar (both are "change a loop") but address entirely different failures.

**When to reach for 0/1 Knapsack, the concrete signal:** "each item/element can be used **at most once**," combined with a target sum, capacity, or optimal-value framing over a **fixed, finite set** — contrast directly with Unbounded Knapsack's "unlimited supply" framing (Coin Change) and with straightforward subset-generation Backtracking (Week 9-10), which 0/1 Knapsack replaces specifically when the question only needs a count, a yes/no, or an optimum — not the actual subsets themselves.

**Trade-offs against the nearest alternatives:** vs. brute-force subset enumeration (Backtracking, `2ⁿ` subsets) — 0/1 Knapsack DP is exponentially faster whenever the target/capacity is polynomially bounded, at the cost of only answering "does a qualifying subset exist / what's the best value," not enumerating every subset (if the problem needs actual subsets back, as some variants do, Backtracking — with the DP table used purely to *prune* — is the right combination, previewed by tomorrow's Word Break II). Vs. Unbounded Knapsack — identical-looking recurrence shape, opposite loop direction when space-optimized, because the two make opposite promises about reuse.

**Complexity, with reasoning:** Time **O(n × target)** — `n` items, each doing O(target) work in its own pass. Space **O(target)** for the 1D array (down from a conceptual O(n × target) for the un-optimized 2D table — the same kind of space optimization already applied to every 1D DP problem this week).

---

## Problem 11: Partition Equal Subset Sum (LC 416, Medium) — Pattern: 0/1 Knapsack DP

**Statement:** Given an array of positive integers, determine whether it can be partitioned into two subsets with equal sums.

### Approach 1 — Brute force: subset enumeration (Backtracking, Week 9-10's technique)

```java
public static boolean canPartitionBruteForce(int[] nums) {
    int totalSum = 0;
    for (int num : nums) totalSum += num;
    if (totalSum % 2 != 0) return false;
    return canReachSum(nums, 0, totalSum / 2);
}

private static boolean canReachSum(int[] nums, int index, int remaining) {
    if (remaining == 0) return true;
    if (index == nums.length || remaining < 0) return false;
    // include nums[index], or exclude it — the same include/exclude template as Subsets (Week 9, Day 61)
    return canReachSum(nums, index + 1, remaining - nums[index])
        || canReachSum(nums, index + 1, remaining);
}
```

Reframes the problem the same way it'll be reframed for DP: "can some subset sum to exactly `totalSum / 2`?" (if the two halves are equal, they're each half the total — and if a subset summing to half exists, everything *not* in it automatically sums to the other half). **Complexity: Time O(2ⁿ)** — every element independently in or out. **Space O(n)** recursion depth.

### Approach 2 — Optimized: 0/1 Knapsack tabulation

(Full code and the decreasing-loop proof are in the Concept Card immediately above — reusing it directly here rather than repeating it.)

**Worked trace:** `nums = [1, 5, 11, 5]`. `totalSum = 22`, even → `target = 11`.

| After item | dp[0] | dp[1] | dp[2..4] | dp[5] | dp[6] | dp[7..10] | dp[11] |
|---|---|---|---|---|---|---|---|
| (init) | T | F | F | F | F | F | F |
| num=1 | T | T | F | F | F | F | F |
| num=5 | T | T | F | T | T | F | F |
| num=11 | T | T | F | T | T | F | **T** |
| num=5 | T | T | F | T | T | T(dp7=T via dp2? no — see below) | T |

Tracing the last row precisely (decreasing `j` from 11 down to 5, using the state *after* num=11's row): `dp[11] = dp[11] || dp[6] = T || T = T` (unchanged, already true). `dp[10] = dp[10] || dp[5] = F || T = T`. `dp[9]=dp[9]||dp[4]=F||F=F`. `dp[8]=F||F=F`. `dp[7]=F||F=F`. `dp[6]=T||T=T` (unchanged — but note this uses `dp[1]`, the *pre-this-pass* value, correctly, since decreasing order hasn't touched `dp[1]` yet at this point). `dp[5]=T||T=T` (unchanged). Final `dp[11] = true` — confirmed: `{11}` and `{1,5,5}` are the two equal subsets (11 and 11).

**Complexity:** as derived in the Concept Card — Time O(n × sum), Space O(sum).

**Edge cases:** odd `totalSum` → immediately `false`, no DP needed (checked before building the array at all — cheap short-circuit). Single element → only reachable target is `0` or the element itself; a single positive element can never equal half of itself unless it's `0`, so single-element arrays are always `false` unless the array is empty (a degenerate case most problem constraints exclude). All elements identical (e.g. `[2,2,2,2]`) → works correctly with no special handling, since the DP only cares about sums, not which specific indices contributed.

> 💡 **Interview Insight:** naming the reduction — "equal-subset-partition is really just subset-sum-to-half" — *before* writing any code is the single highest-leverage thing to say out loud here; it's what turns an apparently two-sided partitioning problem into the single well-known 0/1 Knapsack shape, and most interviewers are specifically listening for that reframe.

---

## Day 85 — Interview Questions

---

**1. Why does Coin Change II need coins as the outer loop, but Coin Change (Day 84) didn't care about loop order at all?**

*Answer:* Coin Change minimizes — the smallest number of coins to reach an amount doesn't depend on what order those coins were conceptually added in, only on the final count. Coin Change II counts *combinations*, where order-independence is the actual thing being enforced (`[1,2]` and `[2,1]` must count once, not twice) — coins-outer guarantees every combination is built in one canonical coin order, which is what collapses reorderings into a single count.

---

**2. Give a concrete input where amount-outer, coins-inner overcounts Coin Change II, and explain the overcount.**

*Answer:* `amount=5, coins=[1,2,5]`: correct answer is 4, amount-outer computes 9. The wrong order lets every coin get a chance at every amount on every pass, which effectively counts different orderings of the same multiset of coins as distinct — it answers "how many ordered sequences," not "how many unordered combinations."

---

**3. What single change turns Coin Change II's code into Combination Sum IV's code, and why does that one change flip combinations into permutations?**

*Answer:* Swap which loop is outer — target/amount outer, nums inner. With amount outer, every number gets to be "whichever one is placed last" at every amount, so the same set of numbers reached in a different final order counts as a separate answer — which is exactly what "ordered" (permutations) means.

---

**4. In the O(n²) LIS solution, why is `dp[i]` defined as "LIS ending exactly at i," not "LIS somewhere in nums[0..i]"?**

*Answer:* The "ending exactly at i" definition is what makes the recurrence `dp[i] = max(dp[j]) + 1` for valid `j < i` correct — it lets the algorithm ask, precisely, "which earlier subsequences could this specific number extend?" A "somewhere in the prefix" definition would lose exactly the information (what value the subsequence currently ends on) needed to check whether `nums[i]` can legally extend it.

---

**5. In the O(n log n) LIS approach, what does `tails[k]` represent, and why does a smaller value at a given index make that entry "better"?**

*Answer:* `tails[k]` is the smallest tail value among all increasing subsequences of length `k+1` found so far. Smaller is better because it's a strictly weaker requirement for some future number to extend that length — any future number that could extend a subsequence ending on a larger tail could also extend one ending on a smaller tail, but not necessarily the reverse.

---

**6. Prove that binary search is valid on the `tails` array at every step, not just intuitively "probably sorted."**

*Answer:* By induction: `tails` starts empty (trivially sorted). Each step either appends a value larger than everything currently present (preserving order) or replaces `tails[lo]` with a value that binary search guarantees is both `≤` the old `tails[lo]` and `>` `tails[lo-1]` (since `lo` was the first index failing the `< num` test) — so strictly-increasing order survives every step, which is exactly what licenses the next binary search.

---

**7. Why isn't the final `tails` array necessarily a real subsequence of the input?**

*Answer:* `tails` only ever tracks the best possible tail *value* per achievable length — it never tracks which original indices produced that value, and a later "replace" can overwrite an entry with a value from a different, unrelated part of the array. `nums=[3,4,5,1]` ends with `tails=[1,4,5]`, but `1` occurs after `4` and `5` in the original array, so that's not a valid increasing subsequence, even though the length (3) is correct.

---

**8. State the core mechanism difference between Unbounded Knapsack (Coin Change, Day 84) and 0/1 Knapsack (today).**

*Answer:* Unbounded Knapsack's recurrence never tracks which items built a sub-answer, which is exactly what permits unlimited reuse. 0/1 Knapsack's recurrence is built so every transition reads from "the state before this item was considered" (row `i-1` in the 2D form) — item `i` can only ever be used once because using it moves you off that row entirely, not back onto it.

---

**9. Why must the 1D space-optimized 0/1 Knapsack loop run its capacity dimension in decreasing order? Give a concrete input where increasing order breaks.**

*Answer:* Decreasing order guarantees every cell read during the current item's pass still reflects the state from *before* this item was considered. `nums=[3]`, checking reachability of sum `6`: increasing order lets `dp[3]` (just set true by the single 3) feed into `dp[6]` later in the *same* pass, incorrectly reusing that one `3` twice and reporting `true`; decreasing order evaluates `dp[6]` using the pre-pass value of `dp[3]` (false), correctly reporting `6` unreachable with only one `3` available.

---

**10. Partition Equal Subset Sum asks about splitting into two equal halves — how does that become a single-subset reachability question?**

*Answer:* If the total sum is even and some subset sums to exactly half, everything *not* in that subset automatically sums to the other half (total minus half is half). This turns a two-sided partition question into "does a subset summing to `totalSum/2` exist" — a direct 0/1 Knapsack reachability check.

---

**11. Why is an odd `totalSum` an immediate `false`, with no DP needed?**

*Answer:* Two equal integer subsets must each sum to `totalSum / 2`; an odd total has no integer half, so no valid split can possibly exist regardless of the specific numbers — checked once, up front, before spending any DP work.

---

## Daily Deliverable Check

- [ ] Coin Change II solved with coins-outer/amount-inner order, and the overcount from the wrong order demonstrated and explained (not just avoided).
- [ ] Combination Sum IV (extra) solved, and the one-line loop-order diff from Coin Change II stated from memory.
- [ ] Longest Increasing Subsequence solved both ways: O(n²) dp[] and O(n log n) tails[]/binary search — both traced by hand on `[10,9,2,5,3,7,101,18]`.
- [ ] Can state, unprompted, why the final `tails` array isn't generally a real subsequence.
- [ ] Number of Longest Increasing Subsequence attempted if time allowed (safe to defer).
- [ ] 0/1 Knapsack's mechanism explained from memory, including the decreasing-loop proof with the `nums=[3]`, target `6` counterexample.
- [ ] Partition Equal Subset Sum solved, pushed to `dsa-java/dynamic-programming/`.

---

## What Tomorrow Assumes You Already Know Cold

Day 86 opens with Target Sum, which the plan itself describes as "the exact same shape" as today's Partition Equal Subset Sum — a two-equation reduction (`P - N = target`, `P + N = totalSum`) landing back on subset-sum counting. Today's 0/1 Knapsack mechanism, and specifically the decreasing-loop requirement, needs to be reflexive by tomorrow, not re-derived — tomorrow's book will cite today's proof rather than repeat it. Word Break II (also tomorrow) reuses Word Break's boolean `dp[]` array from Day 84 directly as a pruning structure underneath fresh backtracking — if Word Break's exact recurrence isn't solid from memory, that's worth a thirty-second refresher before starting Day 86, since tomorrow builds on it without re-explaining it.
