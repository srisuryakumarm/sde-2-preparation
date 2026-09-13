# Day 82 — 1D DP: House Robber Family

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 81 Resource Book](Day81_Resource_Book.md)
**Next ▶:** [Day 83 Resource Book](Day83_Resource_Book.md)
**Companion to:** Day 82 of `Week_12_Revised.md`

---

## Recap

Yesterday's recurrence shape — `dp[i]` built from `dp[i-1]` and `dp[i-2]` — always *added* both possibilities together (every way to reach step `i` counts). Today keeps the exact same two-steps-back lookback but replaces "add both" with a **decision**: at each position, either take it or skip it, and only one of those two choices survives into the final answer. The mechanics (base cases, O(1) space rolling, the "state the recurrence, then prove it" discipline) transfer directly from yesterday; what's new is the `max()` standing in for what was `+` in Climbing Stairs.

Today is also past Dynamic Programming's own opening day, which — per this series' established precedent — is exactly when extra practice becomes fair game again; one extra problem is added below, past the point where padding would have blunted the pattern's genuine first exposure.

---

## Learning Objectives

By the end of today, without notes:

1. Prove `dp[i] = max(dp[i-1], dp[i-2]+nums[i])` correct via an exhaustive, disjoint two-case argument (rob it or don't).
2. Prove House Robber II's circular-to-linear reduction correct, and state precisely why `n=1` needs an explicit special case the general argument doesn't cover.
3. Explain why Delete and Earn is House Robber "in disguise," including exactly what gets lost and what's preserved by bucketing individual elements into per-value totals.

---

## Concept Dependency Map

```
Yesterday: dp[i] = dp[i-1] + dp[i-2]  (Climbing Stairs — ADD both cases, both always count)
        │
        └──▶ same lookback, now a DECISION: dp[i] = max(dp[i-1], dp[i-2] + nums[i])
                  │
                  ▼
             Problem 3: House Robber (LC 198)
                  │
                  └──▶ same recurrence, run TWICE on two linear slices — a circular
                       constraint reduced to two ordinary sub-cases
                            │
                            ▼
                       Problem 4: House Robber II (LC 213)
                            │
                            └──▶ same recurrence again, reached via a bucket-by-VALUE
                                 transform instead of a direct index mapping
                                      │
                                      ▼
                                 Extra Practice: Delete and Earn (LC 740)
```

---

## Problem 3: House Robber (LeetCode 198, Medium) — Pattern: 1D DP

**Statement:** `nums[i]` is the money in house `i`, houses in a line. No two *adjacent* houses can both be robbed. Maximize total money.

**`dp[i]`, precisely:** the maximum money obtainable from houses `0..i`, inclusive, under the no-two-adjacent constraint.

**Why `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — exhaustive, disjoint case argument:** at house `i`, exactly two choices exist, and no third: **skip it** (the best achievable is whatever the best was without house `i` at all — `dp[i-1]`) or **rob it** (earn `nums[i]`, but house `i-1` is now forbidden, so the best achievable for everything before is `dp[i-2]`, the best excluding both `i` and `i-1`). These two cases can't both happen and cover every possibility, so the answer is their max. Base cases: `dp[0] = nums[0]` (a single house, rob it), `dp[1] = max(nums[0], nums[1])` (two adjacent houses, take the better one).

### Approaches 1–2 (brute force, memoization) — concise, per the tiers already fully derived Day 81

```java
// Brute force — exponential, recomputes overlapping suffixes.
public int robBruteForce(int[] nums, int i) {
    if (i < 0) return 0;
    if (i == 0) return nums[0];
    return Math.max(robBruteForce(nums, i - 1), robBruteForce(nums, i - 2) + nums[i]);
}

// Memoization — cache each i once. O(n) time, O(n) cache + stack.
public int robMemo(int[] nums, int i, Map<Integer, Integer> cache) {
    if (i < 0) return 0;
    if (i == 0) return nums[0];
    if (cache.containsKey(i)) return cache.get(i);
    int result = Math.max(robMemo(nums, i - 1, cache), robMemo(nums, i - 2, cache) + nums[i]);
    cache.put(i, result);
    return result;
}
```

### Approach 3 — Optimized: O(1) space

```java
public int rob(int[] nums) {
    int prev2 = 0, prev1 = 0; // dp[i-2], dp[i-1], generalized so the loop needs no special-casing for i=0
    for (int num : nums) {
        int curr = Math.max(prev1, prev2 + num);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

(Seeding both rolling variables at `0` and folding every `nums[i]` through the same loop body — rather than hand-writing `dp[0]`/`dp[1]` as separate lines — is a clean way to avoid restating the base cases explicitly; `curr` at the first iteration correctly reduces to `max(0, 0+nums[0]) = nums[0]`.)

**Worked trace:** `nums = [2,7,9,3,1]`.

| i | nums[i] | dp[i]=max(dp[i-1], dp[i-2]+nums[i]) | dp[i] |
|---|---|---|---|
| 0 | 2 | base | 2 |
| 1 | 7 | max(2, 7) | 7 |
| 2 | 9 | max(7, 2+9) | 11 |
| 3 | 3 | max(11, 7+3) | 11 |
| 4 | 1 | max(11, 11+1) | 12 |

**Answer: 12** — robbing houses `0, 2, 4` (`2+9+1=12`). Confirm by exhaustion over this small case: the only other competitive combination is `1, 3` (`7+3=10`) — `12 > 10`, matches.

**Complexity:** Time O(n), Space O(1).

**Edge cases:** a single house (`dp[0]=nums[0]`, loop body handles it via the seeded `0,0` rolling pair); two houses (`max` of the pair, same mechanism); all houses worth `0` (answer `0`, no special case needed).

**⚠️ Common Mistake:** trying to greedily take every other house by position, rather than computing the actual DP — position-based alternation (`0,2,4,...`) is *not* always optimal; a high-value house at an "odd" position can make skipping to it strictly better, which is exactly why this needs DP rather than a fixed pattern.

**💡 Interview Insight:** stating the `dp[i]` definition and the two-case proof out loud before coding is what separates "recognizes this as House Robber" from "actually understands why the recurrence is correct" — interviewers on this problem specifically probe whether a candidate can justify skipping vs. robbing as a genuine either/or, not just recite the formula.

---

## Problem 4: House Robber II (LeetCode 213, Medium) — Pattern: 1D DP, Circular

**Statement:** Same rules, but the houses are arranged in a **circle** — house `0` and house `n-1` are now adjacent to each other too.

**Reduction to two linear sub-cases, proven, not assumed:** any valid circular robbery plan either robs house `0` or it doesn't — those are the only two possibilities. **If house `0` isn't robbed**, the circular adjacency between house `0` and house `n-1` is irrelevant (house `0` is out of play entirely), so the remaining houses `1..n-1` form an *ordinary linear* House Robber problem — solve it with yesterday's exact recurrence. **If house `n-1` isn't robbed**, symmetric argument, houses `0..n-2` form a linear subproblem. Since every valid circular solution excludes *at least one* of house `0` or house `n-1` (they're adjacent, so both can never be robbed together), every valid solution is captured by at least one of these two linear sub-cases — and neither sub-case can ever produce an *invalid* circular solution, since each is itself a fully valid ordinary House Robber run within its own linear range. Taking the max of the two sub-case results is therefore both safe (never counts an invalid plan) and complete (never misses a valid one).

```java
public int rob(int[] nums) {
    int n = nums.length;
    if (n == 1) return nums[0]; // see below — the general reduction breaks here
    return Math.max(
        robLinear(nums, 0, n - 2),   // exclude the LAST house
        robLinear(nums, 1, n - 1)    // exclude the FIRST house
    );
}

private int robLinear(int[] nums, int start, int end) {
    int prev2 = 0, prev1 = 0;
    for (int i = start; i <= end; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

**⚠️ The `n=1` edge case — a genuine gotcha, not a defensive formality:** with a single house, "exclude the last house" and "exclude the first house" both refer to *the same, only* house — both calls to `robLinear` would run over an **empty range** (`start > end`), correctly returning `0` from each. `max(0, 0) = 0` — but the true answer for one house, with no adjacency conflict possible at all, is `nums[0]`. The general two-case reduction silently breaks precisely because "excluding house 0" and "excluding house n-1" collapse into excluding the *only* house that exists, leaving nothing for either sub-case to actually rob. This needs an explicit `if (n == 1) return nums[0];` **before** the general logic runs — not a case the reduction handles on its own.

**Worked trace:** `nums = [2,7,9,3]` (circular: `2-7-9-3-back to 2`; house 0 and house 2 are *not* adjacent to each other, only each to their immediate circular neighbors).

- Case A, exclude last (`robLinear(nums, 0, 2)` on `[2,7,9]`): `dp: 2 → max(2,7)=7 → max(7,2+9)=11`. Result **11**.
- Case B, exclude first (`robLinear(nums, 1, 3)` on `[7,9,3]`): `dp: 7 → max(7,9)=9 → max(9,7+3)=10`. Result **10**.

**Answer: `max(11, 10) = 11`** — robbing houses `0` and `2` (`2+9=11`), which is valid since those two are not circularly adjacent (only `0↔1↔2↔3↔0` are direct neighbors).

**Complexity:** Time O(n) (two linear passes, each O(n/2)-ish, still O(n) total), Space O(1).

**Edge cases:** `n=1` (handled explicitly, above); `n=2` (the two houses are mutually adjacent both ways around the circle — both sub-cases correctly degrade to a single-house comparison, `max(nums[0], nums[1])`, matching House Robber's own two-house base case).

**💡 Interview Insight:** the `n=1` special case is exactly the kind of detail a tier-1 interviewer expects flagged *before* being asked "does this handle a single house correctly?" — naming it unprompted, with the precise reason the general argument breaks down, is stronger than only fixing it after being pushed.

---

## Extra Practice: Delete and Earn (LeetCode 740, Medium) — Pattern: House Robber, via a Value-Bucket Transform

**Why this is today's extra practice:** House Robber's shape — take-or-skip, no two adjacent — recurs constantly in interviews under disguises that don't look like a row of houses at all. This is the canonical disguise: the "adjacency" here is between **numeric values**, not array positions.

**Statement:** pick some `nums[i]`, earn `nums[i]` points, and **delete every element equal to `nums[i]-1` and `nums[i]+1`** (all occurrences, not just one). Repeat until no elements remain (or you choose to stop). Maximize total points.

**Checked clean:** Delete and Earn does not appear anywhere in the cumulative problem table through Week 11, and does not appear in `Week_13_Revised.md`'s required list either — a genuinely new, non-colliding addition.

**The transformation, justified precisely, not just asserted:** picking one occurrence of value `v` doesn't conflict with picking *another* occurrence of that same value `v` — only *different, adjacent* values (`v-1` or `v+1`) conflict. That means the real decision isn't "which individual elements to pick," it's "which **distinct values** to fully commit to" — for each distinct value present, you either take **every** occurrence of it (earning `v × count(v)`) or **none** of it, and two adjacent values can't both be fully taken. Build `points[v] = v × count(v)` for every value `v` from `0` to `max(nums)`, and the problem becomes *exactly* House Robber over the `points[]` array, indexed by value instead of by house position — "no two adjacent indices" becomes "no two adjacent values," the identical constraint shape.

```java
public int deleteAndEarn(int[] nums) {
    int maxVal = Arrays.stream(nums).max().getAsInt();
    long[] points = new long[maxVal + 1];
    for (int num : nums) {
        points[num] += num; // bucket every occurrence's earned value by its own value
    }

    long prev2 = 0, prev1 = 0; // House Robber, unmodified, over points[] instead of nums[]
    for (long p : points) {
        long curr = Math.max(prev1, prev2 + p);
        prev2 = prev1;
        prev1 = curr;
    }
    return (int) prev1;
}
```

(`long` for the running sums — `nums[i] ≤ 10⁴` and up to `10⁴` elements per LeetCode's constraints means the bucketed totals can approach the edge of comfortable `int` range; the same overflow discipline from Day 10/11 applies.)

**Worked trace:** `nums = [2,2,3,3,3,4]`.

Bucket: value `2` appears twice → `points[2] = 2×2 = 4`. Value `3` appears three times → `points[3] = 3×3 = 9`. Value `4` appears once → `points[4] = 4×1 = 4`. `points = [0, 0, 4, 9, 4]` (indices `0`,`1` empty).

Run House Robber over `[0,0,4,9,4]`: `dp: 0 → max(0,0)=0 → max(0,0+4)=4 → max(4,0+9)=9 → max(9,4+4)=9`.

**Answer: 9** — take every `3` (three of them, `3×3=9`), which forces deleting every `2` and every `4`; the alternative (take the `2`s and `4`s together, `4+4=8`) is strictly worse. Matches the known result for this exact input.

**Complexity:** Time O(n + maxVal) — O(n) to bucket, O(maxVal) to run House Robber over the bucketed array. Space O(maxVal).

**Edge cases:** all elements identical (a single non-zero bucket, no adjacent-value conflict possible, House Robber's recurrence takes it in full automatically — no special-casing needed, since `points[v-1]` and `points[v+1]` are simply `0` and never win the `max`); a value with zero occurrences sitting between two populated values (its `points` entry is `0`, correctly acting as a "free" gap in the House Robber recurrence rather than needing to be skipped explicitly in the array construction).

**⚠️ Common Mistake:** trying to simulate the deletions directly (removing elements from a live collection, re-scanning to find the next-best pick) — correct in principle but needlessly complex and easy to get wrong around repeated values; the bucket-then-House-Robber transform sidesteps simulation entirely by recognizing the *value*-level structure underneath.

**💡 Interview Insight:** naming "this is House Robber over bucketed values" unprompted, before writing any code, is the single highest-leverage thing to say on this problem — it demonstrates the actual transferable skill this whole extra-practice block exists to build: recognizing a familiar shape underneath unfamiliar problem dressing, not having memorized 740 specifically.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** identify 2 Tier B companies to apply to this week.

---

## Day 82 — Interview Questions

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

## Daily Deliverable Check

- [ ] House Robber (LC 198) and House Robber II (LC 213) solved, with both correctness proofs (the two-case argument, and the circular-to-linear reduction with its `n=1` exception) explainable from memory, pushed.
- [ ] Delete and Earn (LC 740) solved as extra practice, with the value-bucket transform explainable and justified, not just applied.
- [ ] 2 Tier B companies identified for application this week.

---

## What Tomorrow Assumes You Already Know Cold

Day 83 needs today's take-or-skip recurrence shape fully reflexive — Decode Ways extends it to a *validity-gated* decision (whether a 1-digit or 2-digit slice is even a legal code) rather than a raw rob/skip choice, and tomorrow's book won't re-derive the general "disjoint, exhaustive cases summed or maxed" proof technique established across today and yesterday. It also assumes Kadane's Algorithm (Week 3, Day 21) is solid, since tomorrow's second problem is a direct, previously-taught extension of it, recapped rather than re-taught.
