# Day 22 — Prefix Sum & Kadane's Completes: The HashMap Half of the Pattern

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 21 Resource Book](Day21_Resource_Book.md)
**Next ▶:** [Day 23 Resource Book](Day23_Resource_Book.md)
**Companion to:** Day 22 of `Week_04_Revised.md`

---

## ⚠️ Overlap Notice — Read This First

One of today's four required problems is a repeat.

| LC # | Problem | Status |
|---|---|---|
| 525 | Contiguous Array | Genuinely new. **Full depth below.** |
| 523 | Continuous Subarray Sum | Genuinely new. **Full depth below.** |
| 560 | Subarray Sum Equals K | Already solved — Week 1, Day 5 (Extra Practice). **Recap only.** |
| 918 | Maximum Subarray Sum Circular | Genuinely new. **Full depth below.** |

There's a real nuance here, flagged in `00_Curriculum_Map.md`'s Week 4 overlap section, worth stating precisely rather than glossing over: **the problem** LC 560 **is a recap, but the *technique* it belongs to is not.** Week 1, Day 5 solved LC 560 as one of twelve HashMap-pattern problems, correctly, but without a formal "prefix sum + HashMap" framing — that framing didn't exist yet, since it needs today's Day 21 prefix-sum foundation, which hadn't been taught back in Week 1. So today does two things at once for LC 560: a short recap of the *problem* (no re-deriving it from scratch), plus the *first formal introduction* of the general technique it's an instance of. That technique also gets a genuinely new sibling variant today (first-occurrence-index tracking, needed for LC 525 and LC 523) that Week 1's version of LC 560 never touched at all.

The freed time from not re-deriving LC 560 goes toward two extra practice problems (below) that fill out both HashMap-flavors of this pattern with a mod-k twist, since this is the pattern's closing day and a high-frequency interview family.

---

## Recap

Day 21 (Week 3) established two things you're building on directly today, both worth having fully reflexive before continuing:

- **Prefix sum:** `prefix[i]` = sum of the first `i` elements, with `prefix[0] = 0` by convention. Any range sum `[i, j]` (inclusive) equals `prefix[j+1] - prefix[i]` — a telescoping difference that isolates exactly the elements in between.
- **Kadane's algorithm:** `best[i] = max(nums[i], best[i-1] + nums[i])` — at every position, either extend the best subarray ending at the previous position, or restart here, whichever is larger. Proven by an exchange argument: extending a *suboptimal* prior subarray is never better than extending the truly optimal one, so tracking only the running optimum loses nothing.

Neither of Day 21's three problems (Range Sum Query, Maximum Subarray, Product of Array Except Self) used a HashMap. Today adds the missing piece: what happens when you store prefix sums not in an array (for direct-index range queries) but in a **HashMap**, so you can ask "have I seen this exact prefix sum — or this exact prefix sum's remainder mod k — before?" That single reframe is the engine behind three of today's four required problems, and both extras.

---

## Learning Objectives

By the end of today, without notes:

1. Recognize when a "have I seen this prefix sum before" HashMap lookup turns an O(n²) subarray-search problem into O(n), and choose correctly between storing a **first-occurrence index** (for "where / how long" questions) versus a **frequency count** (for "how many" questions).
2. Adapt prefix-sum-equality reasoning to prefix-sum-*mod-k* equality, to detect or count subarrays whose sum is divisible by `k` — including correctly normalizing a negative remainder in Java, and stating precisely which of today's problems does and doesn't need that normalization.
3. Prove why running Kadane's algorithm twice (once forward for a max, once inverted for a min) solves Maximum Subarray Sum Circular, and correctly handle the all-negative edge case that breaks the naive version of that idea.
4. Extend Kadane's to track a running max **and** min simultaneously, and explain precisely why a single running max stops being sufficient once negative numbers can flip a product's sign.
5. State, from memory, the full closing shape of Prefix Sum & Kadane's: 7 required problems (3 from Day 21, 4 from today) plus 2 extra, all today — 9 distinct problems, a pattern family that didn't exist anywhere in the original 17-week plan.

---

## Concept Dependency Map

```
Week 3, Day 21 (already covered)
├─ Prefix Sum: prefix[i] = sum of first i elements; range sum = prefix[j+1] - prefix[i]
├─ Kadane's Algorithm: best[i] = max(nums[i], best[i-1] + nums[i]) — "extend or restart"
└─ LC 238 Product of Array Except Self (prefix product × suffix product)

Week 1, Day 4-5 (already covered)
├─ HashMap: O(1) average lookup/insert, bucket mechanism
└─ LC 560 solved as extra practice — correct, but never framed as
   "prefix sum + HashMap" because that framing didn't exist yet
        │
        ▼
TODAY — Prefix Sum + HashMap, formally named for the first time
├─ Flavor A: store FIRST-OCCURRENCE INDEX → answers "where / how long"
│    ├─ LC 525 Contiguous Array         (value, via the 0→-1 trick)
│    └─ LC 523 Continuous Subarray Sum  (value's remainder mod k)
├─ Flavor B: store FREQUENCY COUNT → answers "how many"
│    ├─ LC 560 Subarray Sum Equals K          (RECAP — Week 1, Day 5; value)
│    └─ LC 974 Subarray Sums Divisible by K   (EXTRA; value's remainder mod k)
        │
        ▼
Kadane's, Extended (needs: Day 21's Kadane's, directly)
├─ LC 918 Maximum Subarray Sum Circular  — Kadane's run twice, total − min
└─ LC 152 Maximum Product Subarray (EXTRA) — track running max AND min
        │
        ▼
Prefix Sum & Kadane's CLOSES — 7/7 required + 2 extra = 9 distinct problems
(second pattern family with zero presence in the original 17-week plan —
 Greedy & Intervals, starting tomorrow, is the first day of the other one)
```

---

# Part 1 — Prefix Sum + HashMap: The Missing Piece

Day 21's prefix sum answered range-sum queries by *direct index lookup* into a precomputed array — fast, but only because the query gave you `i` and `j` explicitly. A large class of problems instead asks something like "does *some* subarray sum to exactly `k`?" without telling you where it starts or ends. Checking every `(i, j)` pair directly is O(n²). The fix is the same reframe Day 21's HashMap-pattern week (Week 1) used repeatedly: **trade the search for a lookup.**

**The core identity, restated for this purpose:** if `prefix[j+1] - prefix[i] = k` for some `i < j+1`, then the subarray from `i` to `j` sums to exactly `k`. Rearranged: `prefix[i] = prefix[j+1] - k`. So at each position, instead of searching backward for a valid `i`, ask a HashMap: *"have I already seen a prefix sum equal to `(current prefix sum) - k`?"* That's an O(1) average lookup, turning the whole scan into O(n).

**What you store in the map is where the two flavors diverge, and picking the right one is the actual skill:**

| Question the problem asks | What to store | Why |
|---|---|---|
| "How **long** is the longest such subarray?" / "**Does** one exist?" | **First-occurrence index** of each prefix-sum value | You need the *earliest* matching index to maximize length (or just its existence) — a later, closer match would only shrink or equal the answer, never beat it. |
| "**How many** such subarrays exist?" | **Frequency count** of each prefix-sum value | Every prior occurrence of a matching prefix sum marks a *distinct* valid subarray ending here — you need to count all of them, not just find one. |

**🔑 Key Takeaway:** this is one technique with two accounting strategies, not two techniques. Every problem below is exactly this reframe, applied to whichever of the two questions it's actually asking, sometimes with the equality condition swapped from "same value" to "same value mod k." Recognizing which of the two you need — before you start coding — is worth stating out loud in an interview; it's a stronger signal than arriving at working code by trial and error.

---

## Recap: Subarray Sum Equals K (LeetCode 560, Medium) — Flavor B, Anchor Example

**Original coverage:** Week 1, Day 5, Extra Practice, tagged at the time as "Prefix Sum + Lookup (previews Week 4's formal Prefix Sum pattern)." Below is the solution as originally solved, given fresh because it's the cleanest anchor for Flavor B (frequency count) before extending it to a mod-k version (LC 974, extra, later today).

**Statement:** given an integer array `nums` and an integer `k`, return the total number of contiguous subarrays whose sum equals `k`. `nums` may contain negative numbers.

```java
public static int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixSumCounts = new HashMap<>();
    prefixSumCounts.put(0, 1);   // empty prefix (before index 0) occurs once
    int runningSum = 0, count = 0;

    for (int num : nums) {
        runningSum += num;
        count += prefixSumCounts.getOrDefault(runningSum - k, 0);
        prefixSumCounts.merge(runningSum, 1, Integer::sum);
    }
    return count;
}
```

**Why `prefixSumCounts.put(0, 1)` must be seeded before the loop:** without it, a subarray starting at index 0 that itself sums to exactly `k` would never be found — its `runningSum` would equal `k` exactly, and the lookup `runningSum - k = 0` needs a recorded prefix sum of `0` to match against, which corresponds to "the empty prefix before anything was added." Skipping this seed silently undercounts by exactly the number of such subarrays.

**Why the lookup happens *before* updating the map with this iteration's own sum:** looking up against the map's state from *strictly earlier* iterations prevents a subarray from matching against its own just-inserted entry (which would either wrongly count a zero-length subarray, when `k = 0`, or corrupt the count in more subtle ways). Insert after you've already asked the question, always.

**Why sliding window can't replace this, given `nums` may contain negatives:** sliding window's efficiency depends on the window sum changing *monotonically* as you expand or contract it — only guaranteed when every element is non-negative. A single negative value breaks that assumption outright (expanding the window could decrease the sum), so there's no valid "shrink when too big" rule to fall back on. Prefix sum + HashMap doesn't depend on any monotonicity at all, which is exactly why it survives here.

**Complexity:** Time O(n), Space O(n) — up to n distinct prefix sums stored.

**Edge cases:** `k = 0` with a subarray of all zeros (correctly counted — each zero-sum stretch contributes via the seeded `{0:1}` entry and its own repeats); a single element equal to `k` (counted via the seed); all negative numbers (works identically — nothing here assumes a sign).

💡 **Interview Insight:** this is the problem to reach for as your default example when explaining the *frequency-count* flavor of prefix sum + HashMap to an interviewer — it's the cleanest version with no extra transformation layered on top, which is exactly why it's used as the anchor here before LC 525 and LC 523 add one each.

---

## Problem: Contiguous Array (LeetCode 525, Medium) — Flavor A, First-Occurrence Index

**Statement:** given a binary array `nums` (only `0`s and `1`s), find the maximum length of a contiguous subarray with an equal number of `0`s and `1`s.

### Approach 1 — Brute force

```java
public static int findMaxLengthBruteForce(int[] nums) {
    int maxLength = 0;
    for (int i = 0; i < nums.length; i++) {
        int zeros = 0, ones = 0;
        for (int j = i; j < nums.length; j++) {
            if (nums[j] == 0) zeros++; else ones++;
            if (zeros == ones) {
                maxLength = Math.max(maxLength, j - i + 1);
            }
        }
    }
    return maxLength;
}
```

Check every subarray directly, recounting zeros/ones as the window extends. Time O(n²), Space O(1).

### Approach 2 — Optimized: the ±1 transform, plus first-occurrence-index HashMap

**The reframing trick:** treat every `0` as `-1` and every `1` as `+1`. A subarray has an equal count of 0s and 1s **if and only if** its transformed sum is exactly `0` — turning "equal counts of two things" into "prefix sum equality," which is exactly Part 1's core identity with `k = 0`. Since we want the *longest* such subarray, this is a "where/how long" question — **Flavor A**, first-occurrence index.

```java
public static int findMaxLength(int[] nums) {
    Map<Integer, Integer> firstIndexOfSum = new HashMap<>();
    firstIndexOfSum.put(0, -1);   // sum of 0 occurs "before" index 0
    int runningSum = 0, maxLength = 0;

    for (int i = 0; i < nums.length; i++) {
        runningSum += (nums[i] == 0) ? -1 : 1;
        if (firstIndexOfSum.containsKey(runningSum)) {
            maxLength = Math.max(maxLength, i - firstIndexOfSum.get(runningSum));
        } else {
            firstIndexOfSum.put(runningSum, i);   // only the FIRST time this sum occurs
        }
    }
    return maxLength;
}
```

**Why storing only the first occurrence is correct, and required:** if the running sum returns to a value it already hit once, the subarray between the *earliest* recorded index and now is the *longest possible* subarray with sum `0` ending here — any *later* recorded occurrence of the same sum would only produce a shorter subarray. This is precisely why the `else` branch never overwrites an existing entry: overwriting with a later index would silently discard a longer valid answer found later in the scan.

**Worked trace:** `nums = [0, 1, 0]`. Transformed deltas: `-1, +1, -1`. Map starts `{0: -1}`.

| i | nums[i] | delta | runningSum | in map? | action | maxLength |
|---|---|---|---|---|---|---|
| 0 | 0 | -1 | -1 | no | put(-1, 0) | 0 |
| 1 | 1 | +1 | 0 | **yes** (at -1) | maxLength = max(0, 1-(-1)) = 2 | 2 |
| 2 | 0 | -1 | -1 | **yes** (at 0) | maxLength = max(2, 2-0) = 2 | 2 |

Final answer: `2` — the subarray `[0,1]` (or equivalently `[1,0]` starting at index 1), which does have one `0` and one `1`. Matches brute-force verification directly.

**Complexity:** Time O(n), Space O(n).

**Edge cases:** an array of all `0`s or all `1`s (never returns to sum `0` after index `-1`'s seed, so `maxLength` stays `0` — correct, no valid subarray exists); the whole array balanced (`maxLength` = full array length, found when `runningSum` returns to exactly `0`); a single element (`maxLength` stays `0`, correctly — one element alone can't have equal counts of both).

💡 **Interview Insight:** naming the ±1 transform *before* writing any code is the strongest opening move here — it converts an unfamiliar-looking "equal counts of two categories" problem into a problem you already know how to solve (prefix sum equals a target), which is exactly the kind of reduction interviewers want to see you notice unprompted.

---

## Problem: Continuous Subarray Sum (LeetCode 523, Medium) — Flavor A, Extended to Mod K

**Statement:** given an integer array `nums` and an integer `k`, return `true` if `nums` has a subarray of length at least 2 whose sum is a multiple of `k`. Per LeetCode's current constraints, `0 <= nums[i]`, so every value in this problem is non-negative — worth flagging now, because today's extra practice problem (LC 974) relaxes exactly this constraint, and the fix for that is a genuinely different line of code.

### Approach 1 — Brute force

```java
public static boolean checkSubarraySumBruteForce(int[] nums, int k) {
    for (int i = 0; i < nums.length; i++) {
        int sum = nums[i];
        for (int j = i + 1; j < nums.length; j++) {
            sum += nums[j];
            if (sum % k == 0) return true;   // (j - i + 1) is already >= 2 here
        }
    }
    return false;
}
```

Every subarray of length ≥ 2, checked directly. Time O(n²), Space O(1).

### Approach 2 — Optimized: prefix sum mod k, first-occurrence index

**The extension from LC 525:** instead of asking "have I seen this exact prefix sum before," ask "have I seen this prefix sum's **remainder mod k** before." If two prefix sums share the same remainder mod `k`, their difference — the subarray sum between them — is exactly divisible by `k`, since `(a - b) mod k = 0` whenever `a mod k = b mod k`. Still a "does one exist / where" question, so still **Flavor A** — first-occurrence index — but the equality condition is now on the remainder, not the raw value.

```java
public static boolean checkSubarraySum(int[] nums, int k) {
    Map<Integer, Integer> firstIndexOfRemainder = new HashMap<>();
    firstIndexOfRemainder.put(0, -1);   // remainder 0 occurs "before" index 0
    int runningSum = 0;

    for (int i = 0; i < nums.length; i++) {
        runningSum += nums[i];
        int remainder = runningSum % k;
        if (firstIndexOfRemainder.containsKey(remainder)) {
            if (i - firstIndexOfRemainder.get(remainder) >= 2) {
                return true;   // length requirement satisfied
            }
            // remainder seen before, but too close — do NOT overwrite (see below)
        } else {
            firstIndexOfRemainder.put(remainder, i);
        }
    }
    return false;
}
```

**Why the map is never overwritten, even when a remainder repeats without satisfying length ≥ 2:** keeping the *earliest* index for each remainder gives every future match the best possible chance of reaching a length-2 gap. If you overwrote with a closer, later index instead, a genuinely valid pair further apart could be missed entirely — the earliest index can only help future comparisons, never hurt them.

**Why no negative-mod normalization is needed here:** Java's `%` operator can return a negative result when its left operand is negative (`-7 % 3` is `-1` in Java, not `2`) — but since `0 <= nums[i]` is guaranteed for this problem, `runningSum` is always non-negative, so `runningSum % k` is always in `[0, k-1]` already. This stops being true the moment negative values enter the picture — exactly the situation LC 974 (below) puts you in.

**Worked trace:** `nums = [23, 2, 4, 6, 7]`, `k = 6`. Map starts `{0: -1}`.

| i | nums[i] | runningSum | remainder | seen before? | gap | result |
|---|---|---|---|---|---|---|
| 0 | 23 | 23 | 5 | no | — | put(5, 0) |
| 1 | 2 | 25 | 1 | no | — | put(1, 1) |
| 2 | 4 | 29 | 5 | yes (at 0) | 2-0=2 | **≥2 → return true** |

Matches the known expected output for this exact input (`[2, 4]` sums to `6`, a multiple of `6`, length 2). Confirmed correct.

**Complexity:** Time O(n), Space O(min(n, k)) — at most `k` distinct remainders can ever exist.

**Edge cases:** `k = 1` (every sum is trivially a multiple of 1, so any subarray of length ≥ 2 qualifies — the algorithm still needs *some* remainder collision to fire, which it always will since there are only `k=1` possible remainder buckets, guaranteeing an immediate collision by index 1); array shorter than length 2 (loop can't produce a length-≥2 gap, correctly returns `false`); all zeros (remainder stays `0` throughout, first real collision fires at `i=1`, correctly `true`).

⚠️ **Common Mistake:** checking `sum % k == 0` directly on a *running* sum without the remainder-matching reframe, and trying to track "the sum since the last reset" — this either misses subarrays that don't start at a reset point, or requires re-scanning, losing the O(n) guarantee. The remainder-collision reframe is what avoids ever needing to recompute a sum from scratch.

---

## Extra Practice: Subarray Sums Divisible by K (LeetCode 974, Medium) — Flavor B, Extended to Mod K

**Why this is worth the extra rep:** it combines *both* of today's ideas into one problem — Flavor B's frequency counting (from LC 560) applied to a mod-k equality condition (from LC 523) — and it's the one place today where negative input values actually force a real code difference, not just a caveat.

**Statement:** given an integer array `nums` (which **may contain negative values**, `-10^4 <= nums[i] <= 10^4`) and an integer `k`, return the **number** of non-empty subarrays whose sum is divisible by `k`.

### Approach — prefix sum mod k, frequency count

```java
public static int subarraysDivByK(int[] nums, int k) {
    Map<Integer, Integer> remainderCounts = new HashMap<>();
    remainderCounts.put(0, 1);   // empty prefix, remainder 0, occurs once
    int runningSum = 0, count = 0;

    for (int num : nums) {
        runningSum += num;
        int remainder = ((runningSum % k) + k) % k;   // normalize a possibly-negative remainder
        count += remainderCounts.getOrDefault(remainder, 0);
        remainderCounts.merge(remainder, 1, Integer::sum);
    }
    return count;
}
```

**Why `((runningSum % k) + k) % k` is necessary here but wasn't for LC 523:** Java's `%` follows the sign of the *dividend*, not the divisor — `-7 % 6` evaluates to `-1` in Java, mathematically a valid remainder in some conventions, but not one that will ever match the `[0, k-1]` remainders produced by a *positive* running sum elsewhere in the same map. Left unnormalized, `-1` and the "true" remainder `5` (since `-7 ≡ 5 (mod 6)`) would be tracked as two different bucket keys, silently splitting what should be one bucket into two and undercounting. Adding `k` before the second `%k` shifts any negative result back into `[0, k-1]` without changing its mathematical meaning — `((-1 % 6) + 6) % 6 = (-1 + 6) % 6 = 5`, the correct remainder.

**Why frequency count, not first-occurrence index, this time:** the question is "how many," not "how long" or "does one exist" — every earlier index sharing the current remainder marks one additional valid subarray ending here, so all of them need to be counted, not just the earliest.

**Worked trace:** `nums = [4, 5, 0, -2, -3, 1]`, `k = 5`. Map starts `{0: 1}`.

| element | runningSum | raw `% k` | normalized remainder | count added | count so far | map after |
|---|---|---|---|---|---|---|
| 4 | 4 | 4 | 4 | 0 | 0 | {0:1, 4:1} |
| 5 | 9 | 4 | 4 | 1 (matches prior 4) | 1 | {0:1, 4:2} |
| 0 | 9 | 4 | 4 | 2 (matches both prior 4s) | 3 | {0:1, 4:3} |
| -2 | 7 | 2 | 2 | 0 | 3 | {0:1, 4:3, 2:1} |
| -3 | 4 | 4 | 4 | 3 (matches all three prior 4s) | 6 | {0:1, 4:4, 2:1} |
| 1 | 5 | 0 | 0 | 1 (matches seeded 0) | 7 | {0:2, 4:4, 2:1} |

Final count: `7` — matching the known expected output for this exact input. Confirmed correct.

**Complexity:** Time O(n), Space O(min(n, k)).

**Edge cases:** all elements identical and summing to a multiple of `k` at every prefix (large counts accumulate correctly via repeated map hits, as traced above at remainder `4`); `k` larger than any possible running sum (remainders never repeat except through the seeded `0`, so only subarrays summing to exactly `0` get counted); negative `k` — not possible here, since LeetCode's constraint fixes `k >= 1`.

🔗 **Direct connection:** this problem is the single clearest illustration of Part 1's whole framing — same HashMap reframe as LC 560, same mod-k equality condition as LC 523, frequency count (not first-occurrence) because it's a "how many," and a genuinely new line of code (the double-`% k` normalization) that neither required problem today needed. If an interviewer asks "how would Subarray Sum Equals K change if I also gave you a divisor instead of a target sum," this problem *is* that question.

---

# Part 2 — Kadane's, Extended

Day 21's Kadane's finds the maximum sum of any *contiguous, non-wrapping* subarray. Today extends it in two directions: allowing the subarray to *wrap around* the array's end (LC 918), and switching the objective from sum to *product*, where a single negative number can flip everything (LC 152, extra).

---

## Problem: Maximum Subarray Sum Circular (LeetCode 918, Medium)

**Statement:** given a **circular** integer array `nums` (the end connects back to the beginning), return the maximum possible sum of a non-empty subarray, where a subarray may wrap around from the end back to the start.

### Approach — Kadane's, run twice

**The key insight:** exactly one of two cases holds for the true maximum:

1. **The optimal subarray doesn't wrap.** Then it's just Day 21's ordinary Kadane's result on the array as-is.
2. **The optimal subarray wraps around the end.** Then the elements it *excludes* form a single contiguous, non-wrapping block in the *middle* of the array. Maximizing the wrapping sum is therefore equivalent to **minimizing** that excluded middle block, and then computing `total sum − (minimum subarray sum)`. Finding the minimum-sum subarray is Kadane's algorithm run with `min` in place of `max` — the identical recurrence, inverted.

```java
public static int maxSubarraySumCircular(int[] nums) {
    int totalSum = 0;
    int maxEndingHere = 0, maxSoFar = Integer.MIN_VALUE;
    int minEndingHere = 0, minSoFar = Integer.MAX_VALUE;

    for (int num : nums) {
        totalSum += num;
        maxEndingHere = Math.max(num, maxEndingHere + num);
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
        minEndingHere = Math.min(num, minEndingHere + num);
        minSoFar = Math.min(minSoFar, minEndingHere);
    }

    if (maxSoFar < 0) {
        return maxSoFar;   // all-negative edge case — see below
    }
    return Math.max(maxSoFar, totalSum - minSoFar);
}
```

**⚠️ The all-negative edge case, and exactly why it breaks the naive version:** if every element is negative, the "minimum subarray" that minimizes the excluded block is the *entire array* — which would make the wrapping candidate `total − total = 0`. But `0` corresponds to an **empty** subarray, which the problem doesn't allow (a non-empty subarray of all-negative numbers must have a genuinely negative sum). The fix is the guard at the top: if the ordinary (non-wrapping) Kadane's maximum is already negative, every element is negative, so skip the wrapping computation entirely and return the non-wrapping answer directly — the correct answer in that case is simply the least-negative single element.

**Worked trace:** `nums = [5, -3, 5]`.

| num | maxEndingHere | maxSoFar | minEndingHere | minSoFar | totalSum |
|---|---|---|---|---|---|
| 5 | 5 | 5 | 5 | 5 | 5 |
| -3 | max(-3,5-3=2)=2 | 5 | min(-3,5-3=2)=-3 | -3 | 2 |
| 5 | max(5,2+5=7)=7 | 7 | min(5,-3+5=2)=2 | -3 | 7 |

`maxSoFar = 7` (not negative, guard doesn't fire). `totalSum − minSoFar = 7 − (−3) = 10`. Answer: `max(7, 10) = 10`.

Verify by hand: the wrapping subarray `[5, 5]` (last element + first element, wrapping past the `-3` in the middle) sums to `10` — matches, and correctly beats the best non-wrapping subarray (`7`, from `[5,-3,5]` itself). Confirmed correct, and matches the known expected output for this exact input.

**Complexity:** Time O(n) — one pass computes both Kadane's runs and the total simultaneously. Space O(1).

**Edge cases:** single-element array (both `maxSoFar` and `minSoFar` equal that one element; `totalSum - minSoFar = 0`, but the guard only fires if `maxSoFar < 0`, and if the single element is negative, `maxSoFar` *is* that negative element, so the guard correctly returns it directly rather than the invalid `0`); all-positive array (wrapping never helps, since excluding *any* positive block only shrinks the sum — the algorithm still computes both candidates and correctly picks the non-wrapping one, since `total - minSoFar` will never exceed `maxSoFar` when nothing is negative to exclude profitably — no special-case needed, the formula handles it automatically).

💡 **Interview Insight:** state the two-case split ("optimal subarray either wraps or it doesn't, and the wrapping case reduces to minimizing the excluded middle") *before* writing any code — this is the entire insight, and an interviewer who hears it stated cleanly already knows you understand the problem, independent of whether the code that follows is perfect on the first try. The all-negative guard is the detail that separates a mostly-correct solution from a fully-correct one; naming it unprompted is a strong signal.

---

## Extra Practice: Maximum Product Subarray (LeetCode 152, Medium)

**Why this is worth the extra rep:** it's the single most common Kadane's variant asked in interviews beyond the sum case, and it breaks the "just track one running best" assumption in a genuinely instructive way.

**Statement:** given an integer array `nums`, find the contiguous, non-empty subarray with the **largest product**, and return that product. `nums` may contain negative numbers and zeros.

### Why a single running max (ordinary Kadane's, unmodified) fails here

Kadane's recurrence for *sum* works because extending with a positive addition always helps and extending with a negative one might still beat restarting. Product doesn't have that property: **a large negative running product, multiplied by one more negative number, can become the new largest positive product** — something a single "running max" would have already discarded as bad, having no memory of how negative it was.

### Approach — track running max AND running min simultaneously

```java
public static int maxProduct(int[] nums) {
    int maxEndingHere = nums[0], minEndingHere = nums[0];
    int result = nums[0];

    for (int i = 1; i < nums.length; i++) {
        int num = nums[i];
        if (num < 0) {
            int temp = maxEndingHere;
            maxEndingHere = minEndingHere;
            minEndingHere = temp;   // negative num flips which running value CAN become the new max
        }
        maxEndingHere = Math.max(num, maxEndingHere * num);
        minEndingHere = Math.min(num, minEndingHere * num);
        result = Math.max(result, maxEndingHere);
    }
    return result;
}
```

**Why the swap-before-multiply step is exactly correct:** multiplying by a negative number reverses order — whatever was the *largest* product ending at the previous position becomes a candidate for the new *smallest* once multiplied by a negative, and vice versa. Swapping `maxEndingHere` and `minEndingHere` right before the multiply means the same two lines of arithmetic (`max(num, maxEndingHere*num)` and `min(num, minEndingHere*num)`) stay correct regardless of the current number's sign, rather than needing a separate branch for the negative case.

**Worked trace:** `nums = [2, 3, -2, 4]`.

| i | num | swap? | maxEndingHere | minEndingHere | result |
|---|---|---|---|---|---|
| 0 (init) | 2 | — | 2 | 2 | 2 |
| 1 | 3 | no | max(3, 2·3=6)=6 | min(3, 2·3=6)=3 | 6 |
| 2 | -2 | **yes**, swap(6,3)→(3,6) | max(-2, 3·-2=-6)=-2 | min(-2, 6·-2=-12)=-12 | 6 |
| 3 | 4 | no | max(4, -2·4=-8)=4 | min(4, -12·4=-48)=-48 | 6 |

Final `result = 6` — the subarray `[2,3]`. Matches the known expected output for this exact input. Confirmed correct.

**Complexity:** Time O(n), Space O(1).

**Edge cases:** a single zero in the array (both `maxEndingHere` and `minEndingHere` reset toward `0` at that position, since `max(0, anything*0)` and `min(0, anything*0)` both involve `0` as a candidate — correctly "resets" the running product without special-casing, the same way ordinary Kadane's naturally handles a very negative running sum by restarting); all-negative array with an odd count (careful trace needed — an odd number of negatives means the full-array product is negative, so the true best answer is *some* even-length sub-stretch, which the min/max tracking finds automatically, without ever explicitly counting negatives); single-element array (loop doesn't execute, `result` stays the initial value — correct, trivially).

⚠️ **Common Mistake:** attempting to solve this by counting negative numbers and slicing at zero boundaries (a common but fragile first instinct) instead of tracking running max/min — it's *possible* to make a zero-boundary-slicing approach work, but it requires careful extra bookkeeping (tracking the first and last negative index within each zero-free segment) that the max/min-tracking version gets for free, in fewer lines, with a proof that's easier to state cleanly under interview pressure.

---

## Prefix Sum & Kadane's — Pattern Closes

**7 required problems** (LC 303, 53, 238 from Week 3 Day 21; LC 525, 523, 560, 918 from today) **+ 2 extra practice problems** (LC 974, 152, both today) **= 9 distinct problems total.** A second pattern family — after Sliding Window — with zero presence anywhere in the original 17-week plan, now fully closed without slipping the overall 120-day timeline.

**The two-flavor split, stated once more for retrieval:** prefix sum + HashMap always reduces to "have I seen a matching prefix sum (or remainder) before" — store a **first-occurrence index** when the question is "where / how long" (LC 525, 523), store a **frequency count** when the question is "how many" (LC 560, 974). Kadane's extends from a single running max (sum) to a max-and-min pair (product, LC 152) or a max-and-min *pair of full passes* (circular sum, LC 918) whenever a sign flip or a wraparound breaks the assumption that "bigger is always better to keep."

---

## Career Block Guide (30 min)

**LinkedIn: engagement.** 10–15 minutes commenting on posts in your network or feed — genuinely engaging with 3–5 posts (a specific technical opinion, a question, or a concrete reaction) reads far better than a generic "Great post!" and costs about the same time. Worth prioritizing comments on posts from people at companies you're targeting; it's a low-friction way to stay visible to them before you ever send a connection request.

---

## Day 22 — Interview Questions

**Q1. What's the core reframe that lets a HashMap answer "does a subarray summing to k exist" in O(n) instead of O(n²)?** Rearrange `prefix[j+1] - prefix[i] = k` to `prefix[i] = prefix[j+1] - k`, then ask a HashMap "have I already seen a prefix sum equal to `(current prefix sum) - k`" at each position — an O(1) average lookup replacing an O(n) backward search.

---

**Q2. When should the HashMap store a first-occurrence index versus a frequency count?** First-occurrence index when the question is "where" or "how long" (a later, closer match can only shrink or equal the best answer, never beat it) — frequency count when the question is "how many" (every prior matching occurrence marks a separate valid subarray, so all of them must be counted).

---

**Q3. In Contiguous Array, why does treating every 0 as -1 make this a prefix-sum problem?** Equal counts of 0s and 1s in a subarray is equivalent to that subarray's transformed sum being exactly 0 — turning "equal counts of two categories" into "prefix sum equals a target," which is the same identity already used for exact-sum matching.

---

**Q4. Why must Contiguous Array's map keep only the *first* occurrence of each running sum, never overwrite it?** The longest valid subarray ending at the current index pairs with the *earliest* index sharing the same running sum — overwriting with a later index would silently discard a longer answer found afterward.

---

**Q5. Why doesn't Continuous Subarray Sum (LC 523) need the `((x % k) + k) % k` negative-remainder normalization, while Subarray Sums Divisible by K (LC 974) does?** LC 523 guarantees `nums[i] >= 0`, so every running sum stays non-negative and Java's `%` already returns a value in `[0, k-1]`. LC 974 allows negative elements, so the running sum can go negative, and Java's `%` follows the sign of the dividend — producing a negative "remainder" that would silently split one true remainder bucket into two unless normalized.

---

**Q6. Why does Maximum Subarray Sum Circular's wrapping case reduce to `total sum − minimum subarray sum`?** A wrapping subarray's excluded elements form one contiguous non-wrapping block in the middle of the array; maximizing what's included is equivalent to minimizing what's excluded, and the minimum-sum contiguous block is found by Kadane's algorithm run with `min` in place of `max`.

---

**Q7. Why does an all-negative array break the naive `max(maxKadane, total - minKadane)` formula, and how is it fixed?** If every element is negative, the minimum-sum subarray is the entire array, making the wrapping candidate `total - total = 0` — but `0` implies an empty subarray, which isn't a valid answer for an all-negative, non-empty-required input. The fix is to check whether the ordinary (non-wrapping) Kadane's maximum is already negative; if so, skip the wrapping computation and return that value directly.

---

**Q8. In Maximum Product Subarray, why is tracking a single running max insufficient?** Multiplying a large-magnitude *negative* running product by one more negative number can produce the new largest *positive* product — information a single running max, having already discarded very negative values as "bad," would have no way to recover.

---

**Q9. Why does swapping `maxEndingHere` and `minEndingHere` before multiplying by a negative number keep the recurrence correct?** Multiplying by a negative number reverses order — what was the largest product ending at the previous position becomes a candidate for the new smallest, and vice versa. Swapping first means the same `max(num, maxEndingHere*num)` / `min(num, minEndingHere*num)` lines stay correct regardless of the current number's sign.

---

**Q10. What closes today, and what's the final count?** Prefix Sum & Kadane's — 7 required (3 from Day 21, 4 from today) + 2 extra (LC 974, LC 152, both today) = 9 distinct problems. The second pattern family in this series with zero presence anywhere in the original 17-week plan.

---

## Daily Deliverable Check

- [ ] Contiguous Array, Continuous Subarray Sum, and Maximum Subarray Sum Circular solved fresh; Subarray Sum Equals K confirmed solid via recap (no re-solve needed) — all pushed to `dsa-java/prefix-sum-kadanes/`.
- [ ] Maximum Product Subarray and Subarray Sums Divisible by K (extra practice) solved, same folder.
- [ ] Prefix Sum & Kadane's pattern fully closed — can state the first-occurrence-index vs. frequency-count distinction from memory, and explain the circular-subarray all-negative edge case unprompted.
- [ ] LinkedIn engagement done (10–15 min).

---

## What Tomorrow Assumes You Already Know Cold

Day 23 opens Greedy & Intervals — a pattern family with no dependency on today's prefix-sum/HashMap material at all, so nothing here is a hard prerequisite for tomorrow. What tomorrow *does* lean on is the general habit today reinforced one more time: naming which of two closely related sub-techniques a problem calls for (first-occurrence index vs. frequency count, today; several more such forks are coming in Greedy & Intervals) *before* writing code, rather than discovering the right one by trial and error. Tomorrow also revisits two problems you've already solved without a formal name for what you were doing — LC 881 (Boats to Save Most People, Week 2 Day 11) and LC 11 (Container With Most Water, Week 2 Day 12) were both already tagged "provable greedy" in the curriculum map — today's prefix-sum-and-HashMap fluency isn't what's being called on tomorrow; that informal greedy instinct is.

**Next:** [Day 23 Resource Book](Day23_Resource_Book.md) — Greedy & Intervals Begins.
