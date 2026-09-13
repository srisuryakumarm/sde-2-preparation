# SDE-2 Resource Book Series
## Day 20 — Sliding Window Capstone (Monotonic Deque), and Pattern Wrap-Up

**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**← Previous:** [Day 19](Day19_Resource_Book.md) &nbsp;|&nbsp; **Next →:** [Day 21](Day21_Resource_Book.md)
**Companion to:** Day 20 of `Week_03_Revised.md`

---

### Recap

Today's first problem introduces a genuinely new mechanic — the **monotonic deque** — which does not reduce to any shrink-until/while-valid shape used so far this week; it's built on `ArrayDeque`, a data structure you've had since Day 4, applied in a way you haven't seen it used yet. The second problem, Subarrays with K Different Integers, reuses Day 19's `atMostKDistinct` helper directly, twice. Today closes Sliding Window at 14/14 required problems — the largest single pattern family in the series so far.

---

### Learning Objectives

By the end of today, without notes, you should be able to:
1. Explain the monotonic deque invariant precisely — why it's always safe to discard a smaller-or-equal value from the back — and use it to solve Sliding Window Maximum.
2. Derive "exactly K" as `atMost(K) − atMost(K−1)`, and explain why counting valid subarrays *ending at each right* is the correct way to use the at-most-K helper for a counting (not just longest-window) question.
3. Classify any of this week's 14 required Sliding Window problems as fixed- or variable-size from the wording alone.

---

### Concept Dependency Map

```
Day 4 — ArrayDeque (data structure, already known)
        │
        ▼
Day 20: Monotonic Deque (NEW TECHNIQUE — same structure, new discipline)
   decreasing front-to-back; pop smaller-or-equal from back;
   pop expired-index from front
        │
        ▼
  LC 239 — Sliding Window Maximum (fixed-size)
        │
        ▼
  LC 1438 (Extra) — TWO monotonic deques (max + min) together, variable-size

Day 18/19 — atMostKDistinct(nums, k) helper (established, reused verbatim)
        │
        ▼
Day 20: LC 992 — exactly-K distinct = atMost(K) − atMost(K−1)
   (the helper called twice; new idea: COUNTING valid subarrays
    ending at each right, not just tracking a max length)
```

---

## Problem 13: Sliding Window Maximum

**LeetCode #239 — Hard — Pattern: Monotonic Deque**

**Statement:** given `nums` and a fixed window size `k`, return the maximum of each size-`k` window as it slides across the array.

**Brute force:** for each window position, scan all `k` elements for the max. O(n·k).

**Better (but still not optimal):** a max-heap of `(value, index)` pairs — push each new element, pop from the top while its index has fallen out of the window. O(n log n), since heap operations don't let you remove an arbitrary expired element cheaply, only lazily skip it at the top.

**Optimal — a monotonic deque of indices:**

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> deque = new ArrayDeque<>(); // stores INDICES, values strictly decreasing front→back
    int[] result = new int[nums.length - k + 1];

    for (int right = 0; right < nums.length; right++) {
        while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[right]) {
            deque.pollLast();
        }
        deque.offerLast(right);

        if (deque.peekFirst() <= right - k) {
            deque.pollFirst();
        }

        if (right >= k - 1) {
            result[right - k + 1] = nums[deque.peekFirst()];
        }
    }
    return result;
}
```

**Why it works — the domination argument (prove it, don't assert it):** the deque maintains indices in increasing order, with strictly decreasing *values*, and the claim is that it always holds exactly the candidates that could still become some future window's maximum. When a new value `nums[right]` arrives, any earlier candidate with a value `≤ nums[right]` can **never** be the max of any window that still contains `nums[right]` — because `nums[right]` is both more recent (stays in the window at least as long) and at least as large. That earlier candidate is *dominated* and can be safely discarded from the back, permanently — this is what makes popping smaller-or-equal values correctness-preserving, not just a heuristic. Expiring from the front (once the front index falls `≤ right - k`, meaning it's left the window) handles the other failure mode. Since the deque only ever holds genuinely-still-viable candidates in decreasing order, its front is always the current window's maximum.

**Trace:** `nums = [1,3,-1,-3,5,3,6,7]`, `k = 3`.

| right | nums[right] | pops from back | deque (indices) | front expired? | window max (once formed) |
|---|---|---|---|---|---|
| 0 | 1 | — | [0] | — | — |
| 1 | 3 | idx 0 (1≤3) | [1] | — | — |
| 2 | -1 | — | [1,2] | no | **3** |
| 3 | -3 | — | [1,2,3] | no | **3** |
| 4 | 5 | idx 3,2,1 (all ≤5) | [4] | no | **5** |
| 5 | 3 | — | [4,5] | no | **5** |
| 6 | 6 | idx 5,4 (both ≤6) | [6] | no | **6** |
| 7 | 7 | idx 6 (≤7) | [7] | no | **7** |

Final result: **`[3,3,5,5,6,7]`**.

**Complexity:** Time O(n) — each index is pushed once and popped at most once, from either end, across the whole run (the total-movement argument again, applied to a deque instead of two pointers). Space O(k) — the deque never holds more than `k` indices.

**Edge cases & mistakes:**
- ⚠️ Comparing `nums[deque.peekLast()] < nums[right]` instead of `<=` — using strict `<` keeps duplicate values in the deque unnecessarily; it doesn't break correctness (the older duplicate would still expire correctly later) but does waste space and comparisons. `<=` is tighter and standard.
- ⚠️ Storing *values* in the deque instead of *indices* — you lose the ability to tell when the front has expired out of the window, since expiration is a position check, not a value check.
- ⚠️ Checking the front-expiry condition before pushing the new index, in the wrong order — the standard, safest order is: pop invalid backs → push current → pop expired front → record answer, exactly as above.

**💡 Interview framing:** name the invariant immediately: "I'll keep a deque of indices with strictly decreasing values — anything smaller-or-equal to an incoming value can never win again, so I discard it permanently." That sentence *is* the proof, compressed — say it before writing code.

---

## Problem 14: Subarrays with K Different Integers

**LeetCode #992 — Hard — Pattern: Sliding Window (exactly-K via at-most-K)**

**Statement:** given `nums` and integer `k`, return the number of contiguous subarrays with *exactly* `k` distinct integers.

**Brute force:** for every `(start, end)` pair, count distinct integers. O(n²) or O(n³).

**Optimal — "exactly K" = "at most K" − "at most K−1":**

```java
public int subarraysWithKDistinct(int[] nums, int k) {
    return atMostKDistinct(nums, k) - atMostKDistinct(nums, k - 1);
}

private int atMostKDistinct(int[] nums, int k) {
    if (k < 0) return 0;
    Map<Integer, Integer> window = new HashMap<>();
    int left = 0, count = 0;
    for (int right = 0; right < nums.length; right++) {
        window.merge(nums[right], 1, Integer::sum);
        while (window.size() > k) {
            int leftVal = nums[left];
            window.put(leftVal, window.get(leftVal) - 1);
            if (window.get(leftVal) == 0) window.remove(leftVal);
            left++;
        }
        count += right - left + 1; // valid subarrays ENDING exactly at `right`
    }
    return count;
}
```

**Two things need proving here, not just asserting.**

**First — why the subtraction gives exactly-K.** "At most K distinct" is a superset of "at most K−1 distinct" (every subarray with ≤K−1 distinct values also has ≤K). The subarrays counted by `atMost(K)` but *not* by `atMost(K−1)` are precisely those with distinct-count strictly greater than K−1 and at most K — i.e., exactly K. Set subtraction on nested sets.

**Second — why `count += right - left + 1` correctly counts subarrays, not just tracks a length.** For a fixed `right`, if `[left, right]` has at most K distinct values, then **every** `[left', right]` with `left' ≥ left` also has at most K distinct values — shrinking the start can only remove elements from consideration, never add a new distinct value. So the number of valid subarrays ending at this exact `right` is exactly the number of valid starting points, `right − left + 1`, where `left` is the smallest one (maintained by the shrink loop, which stops the instant validity is restored — it never shrinks further than necessary). Summing this over every `right` counts every valid subarray exactly once.

**Trace:** `nums = [1,2,1,2,3]`, `k = 2`.

`atMostKDistinct(nums, 2)`: window grows cleanly through `right=0..3` (`{1:2,2:2}`, size 2 throughout), contributing counts `1,2,3,4` (running total **10**). At `right=4` (value `3`), size hits 3 → shrink to `left=3` (`{2:1,3:1}`) → contributes `4-3+1=2` more. **Total: 12.**

`atMostKDistinct(nums, 1)`: every step immediately forces a shrink back to a single distinct value (since two different consecutive values always exceed 1), contributing `1` each time across all 5 positions. **Total: 5.**

`subarraysWithKDistinct = 12 − 5 = 7`.

**Complexity:** Time O(n) — `atMostKDistinct` is O(n) by the usual argument, called twice, so O(n) overall (constant factor of 2 doesn't change the order). Space O(n) worst case (the window map).

**Edge cases & mistakes:**
- ⚠️ `k = 1`: `atMostKDistinct(nums, 0)` must return `0` cleanly (guarded by the `if (k < 0)` — wait, note `k-1=0` here is still valid, not negative; the `k<0` guard exists specifically for when the *outer* call passes `k=0`, making the second helper call `atMostKDistinct(nums, -1)`).
- ⚠️ Assuming you need a fundamentally different algorithm for "exactly K" — the insight that it decomposes into two at-most-K calls is the entire trick; resist the urge to hand-roll a direct "exactly K" shrink condition, which is significantly harder to get right (a window can be "exactly K" and then adding one more element makes it K+1, but shrinking from the left might drop it to K-1 without ever passing back through "exactly K" — the direct approach has awkward corner cases the subtraction trick sidesteps entirely).

**💡 Interview framing:** "counting-exactly-K problems are usually easier as a subtraction of two at-most-K problems than as a direct 'exactly K' window" is a reusable insight worth stating up front — it signals you're not about to attempt the harder direct approach.

---

## Extra Practice: Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit

**LeetCode #1438 — Medium — Pattern: Sliding Window (variable), TWO monotonic deques**

✅ **Overlap check:** absent from `00_Curriculum_Map.md`'s inventory and from `Week_04_Revised.md`. Added specifically to reinforce today's new monotonic-deque mechanic a second time, in a variable-size setting — LC 239 above is fixed-size and uses one deque; this is variable-size and uses two at once.

**Statement:** return the length of the longest subarray where the absolute difference between any two elements is at most `limit`.

**Brute force:** for every subarray, track min and max. O(n²).

**Optimal — a max-deque and a min-deque, both maintained exactly as in LC 239, shrinking the window when they disagree by more than `limit`:**

```java
public int longestSubarray(int[] nums, int limit) {
    Deque<Integer> maxDeque = new ArrayDeque<>(); // decreasing, front = window max
    Deque<Integer> minDeque = new ArrayDeque<>(); // increasing, front = window min
    int left = 0, best = 0;

    for (int right = 0; right < nums.length; right++) {
        while (!maxDeque.isEmpty() && nums[maxDeque.peekLast()] <= nums[right]) {
            maxDeque.pollLast();
        }
        maxDeque.offerLast(right);

        while (!minDeque.isEmpty() && nums[minDeque.peekLast()] >= nums[right]) {
            minDeque.pollLast();
        }
        minDeque.offerLast(right);

        while (nums[maxDeque.peekFirst()] - nums[minDeque.peekFirst()] > limit) {
            if (maxDeque.peekFirst() == left) maxDeque.pollFirst();
            if (minDeque.peekFirst() == left) minDeque.pollFirst();
            left++;
        }

        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Why it works:** "valid" here means `max − min ≤ limit` over the whole window — exactly the kind of condition a monotonic deque is built for, since it gives O(1) access to the current max (LC 239's deque) and, symmetrically, the current min (a mirror-image increasing deque). The window is shrink-until-valid (Day 15's shape), just with a validity check built from two deques' fronts instead of a counter.

**Trace:** `nums = [8,2,4,7]`, `limit = 4`. At `right=1` (`nums[1]=2`), the window `[0,1]` has max=8, min=2, diff=6>4 → shrink: `left` advances to 1 (both deques' fronts get evicted since the expiring index is `0` in both). From there the window grows again: `[1,2]` (max=4,min=2,diff=2, valid, length 2), `[1,3]`→ diff between 7 and 2 is 5>4 → shrink again to `left=2`, giving `[2,3]` (max=7,min=4,diff=3, valid, length 2). Final answer: **2**.

**Complexity:** Time O(n) — each index enters and leaves each deque at most once; two deques don't change the asymptotic bound, just the constant factor. Space O(n) worst case.

**Edge cases & mistakes:**
- ⚠️ Only checking `maxDeque.peekFirst() == left` (or only `minDeque`'s) when advancing `left` during a shrink — both deques need the check independently, since the expiring index might currently be at the front of one, both, or (if it was already evicted as dominated) neither.
- ⚠️ All-equal-value input: both deques degrade to holding a single index each; the algorithm still handles this correctly since diff is always 0.

**💡 Interview framing:** "same monotonic-deque discipline as Sliding Window Maximum, run twice — once for max, once for min — because validity here depends on both ends of the window's range, not just one extreme."

---

## Theory: Sliding Window, Reviewed — Fixed vs. Variable, Side by Side

**This closes Sliding Window: 14/14 required problems — up from 10 in the original plan — plus 4 extra this week (18 distinct Sliding Window problems total across Weeks 2–3).**

| # | Problem | Fixed / Variable | Signal in the wording |
|---|---|---|---|
| LC 121 | Best Time to Buy/Sell Stock | *Implicit* (single-pass min-tracking, not a true two-edge window) | "maximum profit," single pass — no explicit window size |
| LC 643 | Maximum Average Subarray I | **Fixed** | "of length k" |
| LC 1004 | Max Consecutive Ones III | Variable | "at most k" (violation budget) |
| LC 3 | Longest Substring w/o Repeating Chars | Variable | "longest substring," no fixed length given |
| LC 424 | Longest Repeating Character Replacement | Variable | "longest substring... replace up to k" |
| LC 567 | Permutation in String | **Fixed** | "a permutation of s1" — fixed length = `len(s1)` |
| LC 438 | Find All Anagrams in a String | **Fixed** | "anagrams" — fixed length = `len(p)` |
| LC 209 | Minimum Size Subarray Sum | Variable | "minimal length... sum ≥ target" |
| LC 904 | Fruit Into Baskets | Variable | "at most two types" |
| LC 1493 | Longest Subarray of 1's After Deleting One | Variable | "longest... after deleting" |
| LC 340 | Longest Substring At Most K Distinct | Variable | "at most k distinct" |
| LC 76 | Minimum Window Substring | Variable | "minimum window... contains" |
| LC 239 | Sliding Window Maximum | **Fixed** | "sliding window of size k" |
| LC 992 | Subarrays with K Different Integers | Variable (called twice) | "exactly k different" |
| *Extra:* LC 1695 | Maximum Erasure Value | Variable | "subarray with all unique elements" |
| *Extra:* LC 1052 | Grumpy Bookstore Owner | **Fixed** | "for minutes minutes" |
| *Extra:* LC 1838 | Frequency of the Most Frequent Element | Variable | "at most k operations" |
| *Extra:* LC 1438 | Longest Subarray, Abs Diff ≤ Limit | Variable | "longest subarray... limit" |

🔑 **Key Takeaway — the pattern in the signal itself:** a *number given directly* ("of size k," "a permutation of s1," "for minutes minutes") means fixed-size. A *constraint to satisfy* ("at most k," "longest/shortest such that," "no more than") means variable-size, because the window's size is exactly what you're solving for.

---

## Project Block Guide (1 hr)

No new task. Confirm all 14 required Sliding Window solutions, plus the 4 extras, are pushed and organized under `dsa-java/sliding-window/`, with filenames matching their LeetCode numbers for easy cross-reference against the table above.

---

## Career Block Guide (1 hr)

- **LinkedIn Post 6 — Sliding Window pattern guide:** structure it exactly like the table above — fixed vs. variable, with a short code snippet of each skeleton (Day 14's fixed template, Day 15's variable template). A guide that teaches the *recognition signal*, not just "here's a problem I solved," is what gets saved and shared.
- **Networking — 2 college alumni at target companies:** same personalization principle as Day 15 — lead with the specific shared context.

---

## Day 20 — Interview Questions

**Q1. Why is it safe to permanently discard a value from the back of the monotonic deque when a larger (or equal) one arrives?**
A: The discarded value is dominated — the new value is both more recent (stays in the window at least as long) and at least as large, so the discarded value can never be the maximum of any window that still contains the new one.

**Q2. Why does Sliding Window Maximum store indices in the deque, not values?**
A: Expiration depends on *position* (has this index fallen outside the current window?), which a raw value can't tell you — you need the index to compare against the window's boundary.

**Q3. Derive "exactly K distinct" from "at most K distinct."**
A: `exactly(K) = atMost(K) − atMost(K−1)`, since "at most K−1" is a strict subset of "at most K," and the difference is precisely the subarrays whose distinct count is greater than K−1 and at most K — i.e., exactly K.

**Q4. In the at-most-K helper used for counting, why does `count += right - left + 1` correctly count subarrays rather than just track a window length?**
A: For a fixed `right`, every subarray `[left', right]` with `left' ≥ left` is also valid (shrinking the start can only remove distinct values, never add one), so the number of valid subarrays ending at `right` equals the number of valid starting points, `right - left + 1`.

**Q5. What's the one concrete signal that tells you a Sliding Window problem is fixed-size rather than variable-size?**
A: A number given directly as the window's size in the problem statement ("of size k," "a permutation of," "for k minutes") signals fixed; a constraint to satisfy ("at most k," "longest/shortest such that") signals variable, since the size itself is what's being solved for.

**Q6. Why does Longest Subarray With Absolute Diff ≤ Limit need two deques instead of one?**
A: Its validity check depends on both the window's maximum and minimum simultaneously (`max − min ≤ limit`); a single monotonic deque only gives you one extreme at a time.

---

## Daily Deliverable Check

- [ ] Sliding Window Maximum (LC 239) and Subarrays with K Different Integers (LC 992) solved, pushed — **Sliding Window ladder complete at 14/14 required.**
- [ ] **Extra:** Longest Continuous Subarray With Absolute Diff ≤ Limit (LC 1438) solved, same folder.
- [ ] Fixed vs. variable reflection table completed (above) — every one of this week's 18 problems classified with its wording signal.
- [ ] LinkedIn Post 6 published.

---

### What Tomorrow Assumes You Already Know Cold

Day 21 opens an entirely new pattern family — Prefix Sum & Kadane's Algorithm — with no dependency on Sliding Window's mechanics at all; its prerequisites are just arrays (Day 2) and HashMap (Day 4/5), both long-since solid. What today's closing table *is* still worth for tomorrow and beyond: the general skill of reading a problem's wording for its structural signal, which you'll do again for Greedy & Intervals in Week 4.
