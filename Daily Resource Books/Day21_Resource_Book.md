# SDE-2 Resource Book Series
## Day 21 (Sunday) — Leave Week Begins: Prefix Sum & Kadane's Algorithm

**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**← Previous:** [Day 20](Day20_Resource_Book.md) &nbsp;|&nbsp; **Next →:** Day 22 (Week 4)
**Companion to:** Day 21 of `Week_03_Revised.md`

**This is Day 1 of the leave week** and the last day of Week 3. This book does two jobs: today's material, and the full Week 3 Consolidation at the end.

---

### Recap

Sliding Window is now fully closed (Day 20). Today opens a new pattern family with no dependency on it at all — Prefix Sum & Kadane's Algorithm reaches back only to Day 2 (arrays) and Day 4/5 (HashMap), both long-since solid. This is genuinely fresh ground.

---

### Learning Objectives

By the end of today, without notes, you should be able to:
1. Build a prefix sum array and answer any range-sum query in O(1), and explain the telescoping argument for why the subtraction works.
2. State and *prove* Kadane's recurrence — not just apply it — including why it handles all-negative arrays correctly without a special case.
3. Solve Product of Array Except Self without division, and explain precisely why division is disallowed (not just that it is).

---

### Concept Dependency Map

```
Day 2 — Arrays                    Day 4/5 — HashMap (not used today, but the
        │                            "prefix sum + HashMap" sub-variant is
        ▼                            coming in Week 4 — see the Week 3
Day 21: Prefix Sum                  Consolidation below)
   prefix[i] = sum(nums[0..i-1])
   range[i,j] = prefix[j+1] - prefix[i]
        │
        ├──▶ LC 303 — direct application
        │
        ▼
   Kadane's Algorithm (running-max special case)
   currentSum = max(nums[i], currentSum + nums[i])
        │
        ├──▶ LC 53 — direct application
        │      🔗 Week 4, Day 22 (LC 918) extends this to circular arrays
        │
        ▼
   Two-directional accumulation (prefix AND suffix, same technique twice)
        │
        └──▶ LC 238 — Product of Array Except Self
```

---

### Self-Check (10 min)

Before today's new material, pick one Sliding Window problem from this week — not the one you found easiest — and solve it cold, no hints, no looking back at your own prior solution. Good candidates specifically *because* they have a non-trivial correctness argument attached: Fruit Into Baskets (LC 904) or Longest Repeating Character Replacement (LC 424). This is the actual test of whether the week's intensity stuck, versus just having been completed once under guidance.

---

## Concept Card — Prefix Sum & Kadane's Algorithm

**What:** precompute cumulative sums so any range's total is a single subtraction away, instead of a fresh scan every time. Kadane's Algorithm is the running-maximum special case of this idea: track the best subarray sum *ending at* each position, rather than a sum over a fixed, queried range.

**Why:** this pattern is genuinely absent from a lot of DSA prep plans despite Maximum Subarray (Kadane's) arguably being *the* canonical array interview problem.

**Where:** any "range sum/product query," "maximum/minimum subarray," or "does some subarray satisfy X" problem where the brute force is "try every subarray" — prefix sum or Kadane's usually collapses that to O(n).

**Interview signal:** "contiguous subarray," "range sum," "maximum sum subarray."

**Prerequisites:** arrays ✅ (Day 2), HashMap ✅ (Day 4/5 — not needed for today's three problems specifically, but for the pattern family as a whole; see the Week 3 Consolidation for exactly where that piece is still missing).

---

## Problem 1: Range Sum Query - Immutable

**LeetCode #303 — Easy — Pattern: Prefix Sum**

**Statement:** given an integer array, answer multiple queries of "what's the sum of `nums[left..right]`?" efficiently.

**Brute force:** sum the range directly on each query. O(n) per query, O(1) preprocessing.

**Optimal — precompute a prefix sum array once:**

```java
class NumArray {
    private final int[] prefix;

    public NumArray(int[] nums) {
        prefix = new int[nums.length + 1];
        for (int i = 0; i < nums.length; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }
    }

    public int sumRange(int left, int right) {
        return prefix[right + 1] - prefix[left];
    }
}
```

**Why it works:** define `prefix[i]` as the sum of the first `i` elements (`prefix[0] = 0` by convention — an empty prefix sums to zero, which is exactly what avoids needing a special case for ranges starting at index 0). Then `prefix[j+1] - prefix[i]` = (sum of the first `j+1` elements) − (sum of the first `i` elements) = sum of elements `i` through `j` — the first `i` elements are common to both sums and cancel out exactly, a telescoping argument.

**Trace:** `nums = [2,4,6,8]` → `prefix = [0,2,6,12,20]`. `sumRange(1,2)` (i.e., `nums[1]+nums[2] = 4+6`) `= prefix[3] - prefix[1] = 12 - 2 = 10`. ✓. `sumRange(0,0)` (`nums[0]` alone) `= prefix[1] - prefix[0] = 2 - 0 = 2`. ✓ — confirming the `prefix[0]=0` convention correctly handles the leftmost-range edge case without a branch.

**Complexity:** Time O(n) to build, O(1) per query thereafter. Space O(n) for the prefix array.

**Edge cases & mistakes:**
- ⚠️ **The single most common bug in prefix sum code:** using `prefix[j] - prefix[i-1]` (indices aligned directly to the array) instead of the `prefix[i]`-means-"first-i-elements" convention above — this works, but forces an awkward special case when `i=0` (since `i-1=-1` is out of bounds). Building the prefix array one element longer than the input, with `prefix[0]=0`, eliminates that special case entirely — prefer this convention consistently going forward.
- ⚠️ Recomputing the prefix array on every query instead of once in the constructor — defeats the entire point of precomputation.

**💡 Interview framing:** "I'll precompute once, in the constructor, so each query is O(1) — this is the standard trade-off when a structure will be queried many times but built once." Naming that trade-off explicitly is what separates "solved it" from "understood why this shape of solution exists."

---

## Problem 2: Maximum Subarray

**LeetCode #53 — Medium — Pattern: Kadane's Algorithm**

**Statement:** given an integer array (possibly containing negatives), find the contiguous subarray with the largest sum, and return that sum.

**Brute force:** for every `(start, end)` pair, sum the subarray. O(n²) directly, or O(n³) if each sum is recomputed from scratch rather than extended.

**Better (not yet optimal):** prefix sums turn any range sum into O(1), so brute force with prefix sums is O(n²) time, O(n) space — better constants, same order.

**Optimal — Kadane's Algorithm:**

```java
public int maxSubArray(int[] nums) {
    int currentSum = nums[0];
    int maxSum = nums[0];
    for (int i = 1; i < nums.length; i++) {
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
```

**Why it works — the proof, not just the recurrence.** Define `best[i]` = the maximum sum of any subarray *ending exactly at* index `i`. Any such subarray either (a) consists of `nums[i]` alone, or (b) extends some subarray that ended at `i-1` by exactly one more element. If it extends a prior subarray, the best possible extension always uses the *best* subarray ending at `i-1` — using anything less than the best would only give a smaller-or-equal sum after adding the same `nums[i]`, so there's never a reason to extend a suboptimal one. This gives the recurrence `best[i] = max(nums[i], best[i-1] + nums[i])`, exactly what `currentSum` computes at each step; `maxSum` just tracks the running best across all `i`. 🔗 This "best answer at position i, built from the best answer one position back" shape is a preview of Dynamic Programming, formalized in a later week — for now, it's specifically Kadane's own trick, not yet a named general technique.

**Why all-negative arrays need no special case:** because the recurrence compares against `nums[i]` *alone* at every step (not against zero), the algorithm never pretends a negative running sum is "worth keeping." If every element is negative, `currentSum` resets to the (less-negative) single element every single step, and `maxSum` correctly converges to the single largest (least negative) element — never zero, and never an empty subarray, both of which would be wrong answers here.

**Trace:** `nums = [-2,1,-3,4,-1,2,1,-5,4]`.

| i | nums[i] | currentSum = max(nums[i], currentSum+nums[i]) | maxSum |
|---|---|---|---|
| 0 | -2 | -2 (init) | -2 |
| 1 | 1 | max(1, -2+1=-1) = 1 | 1 |
| 2 | -3 | max(-3, 1-3=-2) = -2 | 1 |
| 3 | 4 | max(4, -2+4=2) = 4 | 4 |
| 4 | -1 | max(-1, 4-1=3) = 3 | 4 |
| 5 | 2 | max(2, 3+2=5) = 5 | 5 |
| 6 | 1 | max(1, 5+1=6) = 6 | 6 |
| 7 | -5 | max(-5, 6-5=1) = 1 | 6 |
| 8 | 4 | max(4, 1+4=5) = 5 | 6 |

Final: **6** — the subarray `[4,-1,2,1]`.

**Complexity:** Time O(n) — single pass. Space O(1) — two running variables, no array needed (unlike the brute-force-with-prefix-sums approach above).

**Edge cases & mistakes:**
- ⚠️ Initializing `maxSum` (or `currentSum`) to `0` instead of `nums[0]` — silently wrong for all-negative arrays, since it would let "an empty subarray summing to 0" beat every real (negative) option, which isn't a valid answer to this problem.
- ⚠️ Single-element array: the loop body never executes; `maxSum` is correctly just `nums[0]` from initialization — verify this rather than assume it.

**💡 Interview framing:** "at each position, I either extend the best subarray ending just before me, or start fresh here — whichever is bigger." Say the proof's core sentence, not just the code — "extending a suboptimal prior subarray is never better than extending the optimal one" is the one-line version of the exchange argument above, and it's usually exactly what's being probed for when an interviewer asks "why does this work?"

🔗 **Forward reference:** Week 4, Day 22's Maximum Subarray Sum Circular (LC 918) extends this directly — the answer there is either a normal Kadane's result, or `(total array sum) − (minimum subarray sum)` for the wraparound case (found by running this same algorithm inverted, for the minimum instead of the maximum). That's new material for Week 4, not something to derive today.

---

## Problem 3: Product of Array Except Self

**LeetCode #238 — Medium — Pattern: Prefix Product × Suffix Product**

**Statement:** given an array, return an array where each element is the product of all the *other* elements. No division allowed.

**Brute force:** for each index, multiply every other element. O(n²).

**Why division is disallowed — worth understanding, not just accepting:** the tempting O(n) shortcut is `output[i] = totalProduct / nums[i]`. This breaks the moment any `nums[i] == 0` (division by zero — and if there are *two* or more zeros, every output should be 0, which the division approach can't even express coherently), and the problem forbids it explicitly for exactly this reason — it's not an arbitrary restriction, it's ruling out a shortcut that's fundamentally unsound on valid input.

**Optimal — prefix products from the left, suffix products from the right, combined in one output array:**

```java
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] output = new int[n];

    output[0] = 1;
    for (int i = 1; i < n; i++) {
        output[i] = output[i - 1] * nums[i - 1]; // prefix product, one pass left→right
    }

    int suffixProduct = 1;
    for (int i = n - 1; i >= 0; i--) {
        output[i] *= suffixProduct;               // fold in the suffix product
        suffixProduct *= nums[i];
    }

    return output;
}
```

**Why it works:** "product of everything except index i" is, by definition, `(product of everything before i) × (product of everything after i)`. The first pass fills `output[i]` with exactly the first factor (product of `nums[0..i-1]`); the second pass multiplies in the second factor (product of `nums[i+1..n-1]`), maintained as a single running variable instead of a second array — this is what keeps extra space at O(1), excluding the output array itself, which the problem doesn't count against space complexity by convention.

**Trace:** `nums = [1,2,3,4]`.

*Pass 1 (prefix, into `output`):* `output = [1, 1, 2, 6]` (each `output[i]` = product of everything strictly before index `i`).

*Pass 2 (suffix, right→left, `suffixProduct` starts at 1):*
- `i=3`: `output[3] = 6×1 = 6`; `suffixProduct = 1×4 = 4`.
- `i=2`: `output[2] = 2×4 = 8`; `suffixProduct = 4×3 = 12`.
- `i=1`: `output[1] = 1×12 = 12`; `suffixProduct = 12×2 = 24`.
- `i=0`: `output[0] = 1×24 = 24`; `suffixProduct = 24×1 = 24`.

Final: **`[24, 12, 8, 6]`** — check by hand: index 0 excludes `1`, so `2×3×4=24` ✓; index 3 excludes `4`, so `1×2×3=6` ✓.

**Complexity:** Time O(n) — two linear passes. Space O(1) extra (excluding the required output array) — this is the actual point of the two-pass technique over a naive "build a separate prefix array and a separate suffix array" version, which would be O(n) extra space for the two auxiliary arrays.

**Edge cases & mistakes:**
- ⚠️ Exactly one zero in the input: every output *except* the zero's own position should be 0, and the zero's own position should hold the product of everything else — the algorithm above handles this correctly with no special case, since a zero anywhere in a running product just zeroes out the running product from that point onward, exactly as it should.
- ⚠️ Building two separate O(n) arrays (a full prefix array and a full suffix array, then multiplying them pointwise) — correct, but misses the O(1)-extra-space optimization that folding the suffix pass directly into the output array achieves.

**💡 Interview framing:** "no division" is your cue to immediately say why division would be unsound here (zeros), before being asked — then present the two-pass technique as "the direct algebraic definition of the answer, computed in two sweeps instead of one nested loop."

---

## Career Block Guide (30 min)

Leave-week days stay light on career tasks by design. LinkedIn: engagement only — 10–15 minutes commenting on posts, no new content to draft today.

---

## Day 21 — Interview Questions

**Q1. Why does the prefix sum array use `prefix[i] = sum of the first i elements` (with `prefix[0]=0`), rather than aligning `prefix[i]` directly to `nums[i]`?**
A: The `prefix[0]=0` convention means a range starting at index 0 needs no special case — `sumRange(0, j) = prefix[j+1] - prefix[0] = prefix[j+1] - 0`, which is already correct without a branch.

**Q2. Prove Kadane's recurrence — don't just state it.**
A: Let `best[i]` be the max sum of any subarray ending exactly at `i`. Such a subarray either is `nums[i]` alone, or extends the best subarray ending at `i-1` (extending anything less than the best there can only give an equal-or-smaller result after adding the same `nums[i]`). So `best[i] = max(nums[i], best[i-1] + nums[i])`, and the answer is the max of `best[i]` over all `i`.

**Q3. Why doesn't Kadane's need a special case for all-negative arrays?**
A: The recurrence compares the extension against `nums[i]` alone, never against zero — so it never treats "keep a negative running sum" as better than "restart," and correctly converges to the single largest (least negative) element rather than an invalid empty-subarray sum of zero.

**Q4. Why is division disallowed in Product of Array Except Self, precisely?**
A: `total / nums[i]` breaks whenever any element is zero (undefined), and can't correctly express the case of two or more zeros (every output should be 0) — it's not an arbitrary rule, it rules out an approach that's unsound on valid input.

**Q5. Why is the two-pass prefix/suffix technique O(1) extra space, when it still touches every element twice?**
A: The suffix pass folds directly into the existing `output` array via one running variable (`suffixProduct`), rather than allocating a second full-size array — only the required output array and O(1) extra bookkeeping are used.

---

## Daily Deliverable Check

- [ ] Self-check: one Sliding Window problem solved cold.
- [ ] Range Sum Query - Immutable (LC 303), Maximum Subarray (LC 53), and Product of Array Except Self (LC 238) solved, pushed to `dsa-java/prefix-sum-kadanes/`.
- [ ] Leave week officially underway.

---
---

# Week 3 Consolidation

## What actually got built

**Sliding Window — CLOSED at 14/14 required, plus 4 extra (18 distinct total across Weeks 2–3):**
- 12 required problems this week (Days 15–20): Max Consecutive Ones III (1004), Longest Substring Without Repeating Characters (3), Longest Repeating Character Replacement (424), Permutation in String (567), Find All Anagrams in a String (438), Minimum Size Subarray Sum (209), Fruit Into Baskets (904), Longest Subarray of 1's After Deleting One Element (1493), Longest Substring with At Most K Distinct Characters (340), Minimum Window Substring (76), Sliding Window Maximum (239), Subarrays with K Different Integers (992).
- 4 extra practice problems added this week: Maximum Erasure Value (1695, Day 15), Grumpy Bookstore Owner (1052, Day 16), Frequency of the Most Frequent Element (1838, Day 18), Longest Continuous Subarray With Absolute Diff ≤ Limit (1438, Day 20) — each checked against the curriculum map and `Week_04_Revised.md` before being added; none collided.
- Zero recaps needed — every required problem this week was verified absent from the Weeks 1–2 inventory before being taught, confirming the map's own pre-check (see its "Known Overlap With Week 3" section) held.

**Prefix Sum & Kadane's — OPENED at 3/7 required, 0 extra (deliberately deferred):** Range Sum Query - Immutable (303), Maximum Subarray (53), Product of Array Except Self (238). No extra practice added — the pattern is still open, not closing, exactly the same reasoning the map applied to Sliding Window when *it* opened on Week 2, Day 14. Week 4 closes this pattern; extra practice, if warranted, belongs to whichever week actually closes it.

**Theory:** HashMap Internals (deepening Week 1's HashMap/HashSet — hashing into buckets, collision chaining, treeification thresholds), Generics (type parameters, bounded wildcards, PECS, type erasure), Comparable vs. Comparator (plus TreeMap and PriorityQueue as heap-adjacent structures), Exception Handling (checked vs. unchecked, try-with-resources), and Sliding Window's own closing review (fixed vs. variable, all 18 problems classified).

**Projects:** `HashCodeContractDemo` (Day 15), `ResponseWrapper<T>` + `Pair<A,B>` (Day 16), `Transaction` + `TransactionSorting` (Day 17), `CacheMissException` + try-with-resources demo (Day 18).

**Two corrections made along the way, flagged here for visibility rather than silently absorbed:**
1. **Day 15:** the plan's hashCode-always-returns-0 exercise, read literally, only demonstrates a *performance* bug (every key still findable, just slower), not the *silent-failure* bug its own description promised. Both were built and explicitly distinguished — the constant-hashCode demo for the performance case, plus a genuine equals()/hashCode() contract-violation demo for the actual silent-failure case the exercise was really after.
2. **Day 17:** the plan's project block calls the practice class a "Transaction record" — but `record` isn't taught until Week 4, Day 28. `Transaction` was built as a plain hand-written class instead (fields, constructor, getters), with a forward note that Day 28's `record` keyword automates exactly what was hand-written here.

No day-ordering changes were needed beyond these two syntax/framing substitutions — Week 3's plan order already respects every prerequisite dependency as written.

## Planned vs. actual

| | Required (plan) | Extra practice | Total |
|---|---|---|---|
| Sliding Window (this week's portion) | 12 | 4 | 16 |
| Prefix Sum & Kadane's (this week's portion) | 3 | 0 | 3 |
| **Week 3 total** | **15** | **4** | **19** |

**Cumulative distinct problems, Weeks 1–3: 56** (37 through Week 2, per the curriculum map, + 19 this week).

⚠️ **A number worth flagging now, so it doesn't look like a discrepancy later:** `Week_04_Revised.md`'s own Day 28 scorecard states "57 total DSA problems solved" as of Day 28, built from the plan's own internal count (25 through Day 14, +12, +18, +2). That figure tracks only the *plan's required ladder* — it doesn't include any of this series' extra practice. This document's 56-through-Day-21 (and whatever Week 4 adds on top) is deliberately a different, larger number, because it's counting *distinct problems actually solved*, extras included. Both are correct; they're just answering different questions. Use this map's cumulative total as the authoritative one going forward.

## Diagnostic — what to check yourself against

- [ ] Can you write the variable-size window template from Day 15 from a blank editor, no reference?
- [ ] Can you state, out loud, *why* Longest Repeating Character Replacement's stale `maxFreq` doesn't break correctness — not just that it doesn't?
- [ ] Can you derive "exactly K distinct" from two calls to an at-most-K helper, and explain why summing `right-left+1` per step counts subarrays correctly?
- [ ] Can you explain the monotonic deque's domination argument (why discarding a smaller-or-equal value from the back is always safe) without hand-waving?
- [ ] Can you prove Kadane's recurrence from scratch, including why all-negative arrays need no special case?
- [ ] Given a new, unseen Sliding Window problem, can you classify it fixed vs. variable from the wording alone, in under 30 seconds?

If any of these feel shaky, that's exactly what Day 21's self-check and this list are for — better to find out now than mid-interview.

## What Week 4 assumes

Week 4 (`Week_04_Revised.md`) finishes Prefix Sum & Kadane's (4 more required problems: Contiguous Array #525, Continuous Subarray Sum #523, Subarray Sum Equals K #560, Maximum Subarray Sum Circular #918 — closing the pattern at 7/7), then opens an entirely fresh pattern, Greedy & Intervals (11 problems), before transitioning to Binary Search.

**What's genuinely solid going in:** the prefix-sum telescoping argument and Kadane's extend-or-restart recurrence, both proven (not just applied) today. Sorting (Day 3) and array fundamentals (Day 2), both long-since automatic, are Greedy & Intervals' only stated prerequisites per its own Concept Card. Kadane's own "locally optimal choice, provably safe" reasoning is a small-scale preview of the exact skill Greedy & Intervals formalizes — Week 4's theory block leans on being able to *justify* why a greedy choice is safe, and today's Kadane's proof is the first time that flavor of reasoning showed up in this series.

**What's genuinely NOT covered yet, and needs fresh teaching in Week 4:** none of today's three problems used a HashMap. The "prefix sum + HashMap, track the first index each running value occurred at" sub-variant — needed for Contiguous Array (525) and Continuous Subarray Sum (523) — hasn't been formally taught. There is one prior, easy-to-miss exposure worth surfacing explicitly when Week 4 is generated: Subarray Sum Equals K (LC 560) was already solved back in Week 1, Day 5, as extra practice, flagged in the curriculum map at the time as previewing exactly this pattern. Week 4, Day 22 should recap LC 560 from that prior solve (not re-teach it from zero) while still formally introducing the prefix-sum-plus-HashMap technique itself for the first time, since the technique — as opposed to that one specific problem — is genuinely new.

**Maximum Subarray Sum Circular (LC 918)** builds directly on today's Kadane's, adding the "total sum minus minimum subarray sum" wraparound case — new material, but resting on a fully solid foundation.

---

*(Week 3 index for reference: [Day 15](Day15_Resource_Book.md) · [Day 16](Day16_Resource_Book.md) · [Day 17](Day17_Resource_Book.md) · [Day 18](Day18_Resource_Book.md) · [Day 19](Day19_Resource_Book.md) · [Day 20](Day20_Resource_Book.md) · Day 21 (this file). Consolidated Q&A: [Week3_Interview_Questions.md](Week3_Interview_Questions.md).)*
