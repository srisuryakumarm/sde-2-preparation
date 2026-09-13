# Day 30 — Binary Search: Rotated Arrays

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 29 Resource Book](Day29_Resource_Book.md)
**Next ▶:** [Day 31 Resource Book](Day31_Resource_Book.md)
**Companion to:** Day 30 of `Week_05_Revised.md`

---

## Recap

Yesterday closed with a proof that binary search only needs *some* rule for safely discarding half the search space — a sorted array is one such rule, a local slope comparison is another. Today's rule is a third: after an array has been sorted and then rotated at an unknown pivot, splitting it at any `mid` guarantees that **at least one of the two halves is still normally sorted** — and that's enough structure to binary search on, even though the array as a whole no longer is.

No new theory today — this is a pure DSA day, followed by a written reflection instead of a coding project.

---

## Learning Objectives

By the end of today, without notes:

1. Prove why splitting a rotated sorted array at any index always leaves at least one half in normal sorted order.
2. Apply that guarantee to search a rotated array in O(log n), including correctly checking whether the target actually falls inside the sorted half's range before recursing into it.
3. Produce a concrete counter-example showing exactly how a single duplicate value breaks that guarantee, and state precisely what the fix costs in complexity.

---

## Concept Dependency Map

```
Day 28: exact-match binary search template
Day 29: proof style — invariant preserved each step, forces one correct answer
        │
        ▼
Today, Problem 5: Search in Rotated Sorted Array (LC 33)
   NEW invariant: splitting at mid always leaves ≥1 half normally sorted
        │
        ▼
Today, Problem 6: Search in Rotated Sorted Array II (LC 81)
   duplicates break "which half is sorted" — same skeleton, one added branch,
   and the O(log n) guarantee is the casualty
```

---

## Problem 5: Search in Rotated Sorted Array (LeetCode 33, Medium) — Pattern: Modified Binary Search on a Rotated Array

**Statement:** `nums` was sorted in ascending order, then rotated at some unknown pivot (e.g., `[0,1,2,4,5,6,7]` → `[4,5,6,7,0,1,2]`). All values are **distinct**. Given `target`, return its index, or `-1` if absent. Required in O(log n).

### Approach 1 — Brute force

```java
public static int searchRotatedBruteForce(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] == target) return i;
    }
    return -1;
}
```

Ignores the rotation structure entirely. Time O(n), Space O(1) — correct, but throws away exactly the information that makes O(log n) possible.

### Approach 2 — Optimized: identify the sorted half, then check range membership

```java
public static int searchRotated(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        }
        if (nums[left] <= nums[mid]) {
            // left half [left..mid] has no wraparound — it's normally sorted
            if (nums[left] <= target && target < nums[mid]) {
                right = mid - 1;   // target is inside the sorted left half
            } else {
                left = mid + 1;    // target must be on the other side
            }
        } else {
            // right half [mid..right] is the one with no wraparound
            if (nums[mid] < target && target <= nums[right]) {
                left = mid + 1;    // target is inside the sorted right half
            } else {
                right = mid - 1;
            }
        }
    }
    return -1;
}
```

**Why one half is always guaranteed sorted:** the array is one sorted sequence that's been cut at a single "wrap" point and the two pieces swapped. Any contiguous slice that doesn't straddle that wrap point is, by construction, still in plain ascending order. Splitting the current range at `mid` produces two halves — the wrap point can fall inside at most *one* of them, so the other is guaranteed wrap-free, i.e., normally sorted. The check `nums[left] <= nums[mid]` tests exactly this: if true, `[left, mid]` contains no wrap (a genuinely sorted slice would never have its first element exceed its last); if false, the wrap must be somewhere in `[left, mid]`, which means `[mid, right]` is the wrap-free, sorted side instead.

Once the sorted half is identified, the only remaining question is whether `target` actually falls within *that half's own value range* — if it does, recurse into it (standard binary search, nothing rotation-specific left); if it doesn't, the target must be in the other half (even though that half isn't sorted itself, the search continues into it and re-applies the same "identify the sorted piece" logic one level deeper).

**Worked trace:** `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 0`.

| left | right | mid | nums[mid] | which half sorted | target in that range? | action |
|---|---|---|---|---|---|---|
| 0 | 6 | 3 | 7 | `nums[0]=4 <= nums[3]=7` → left `[0,3]` sorted | is `4 <= 0 < 7`? No | `left = 4` (go right) |
| 4 | 6 | 5 | 1 | `nums[4]=0 <= nums[5]=1` → left `[4,5]` sorted | is `0 <= 0 < 1`? Yes | `right = 4` |
| 4 | 4 | 4 | 0 | `nums[4] == target` | — | return `4` |

Correct — `nums[4] = 0`.

A second, shorter trace to see the *right*-half-sorted branch fire: `nums = [6, 7, 0, 1, 2, 3, 4, 5]`, `target = 3`. `left=0, right=7, mid=3, nums[3]=1`. Is `nums[0]=6 <= nums[3]=1`? No — so the **right** half `[3,7]` is the sorted one. Is `nums[3]=1 < 3 <= nums[7]=5`? Yes → `left = 4`. Now `left=4, right=7, mid=5, nums[5]=3 == target` → return `5`. Correct in two steps.

**Complexity:** Time O(log n) — each step still halves the range; the extra work per step (deciding which half is sorted, checking range membership) is O(1). Space O(1).

**Edge cases:**
- Array not actually rotated (pivot at index 0) — `nums[left] <= nums[mid]` is true every time, and the algorithm degrades gracefully into plain binary search.
- Single-element array — loop runs once, direct match-or-miss.
- Target equal to `nums[left]` or `nums[right]` exactly — the `<=`/`<=` boundaries in the range checks are written to include these correctly (traced above: `nums[left] <= target` uses `<=`, not `<`).
- Rotation point directly at `mid` — one of the two half-checks still correctly identifies a sorted side; the algorithm never needs to know *where* the pivot is, only which side is currently clean.

**⚠️ Common Mistake:** writing the sortedness check as `nums[left] < nums[mid]` (strict) instead of `<=`. With a two-element range where `left == mid`, `nums[left] < nums[mid]` is false even though that trivial one-element-wide "half" is (trivially) sorted — using strict `<` here can misroute the search. `<=` handles the degenerate single-element case correctly.

**⚠️ Common Mistake:** checking only one side of the range (`target < nums[mid]` without also confirming `target >= nums[left]`) — a target smaller than everything in the sorted half would incorrectly be routed into it.

**💡 Interview Insight:** state the invariant *before* writing code: "splitting a rotated sorted array anywhere always leaves at least one side normally sorted — I'll check which side that is first, then decide whether the target's value range falls inside it." Naming the invariant unprompted, rather than arriving at the branching logic by trial and error, is the strongest possible opening here.

---

## Problem 6: Search in Rotated Sorted Array II (LeetCode 81, Medium) — Pattern: Modified Binary Search, Duplicates Break the Clean Guarantee (NEW)

**Statement:** Identical setup to Problem 5, except `nums` **may contain duplicates**. Return `true` if `target` exists, `false` otherwise (this variant asks for existence, not an index).

### Approach — same skeleton, one new branch

```java
public static boolean searchRotatedWithDuplicates(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return true;
        }
        if (nums[left] == nums[mid] && nums[mid] == nums[right]) {
            // can't tell which side is wrap-free — shrink both ends and retry
            left++;
            right--;
        } else if (nums[left] <= nums[mid]) {
            if (nums[left] <= target && target < nums[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        } else {
            if (nums[mid] < target && target <= nums[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }
    return false;
}
```

**⚠️ What exactly breaks, shown concretely — this is the flip from yesterday's problem:** yesterday's `nums[left] <= nums[mid]` check was a reliable signal for "left half is wrap-free." With duplicates, it can lie. Take `nums = [1, 0, 1, 1, 1]`, `target = 0`, and run **Problem 5's** logic (no duplicate handling) on it:

`left=0, right=4, mid=2, nums[2]=1` (not target). Check `nums[left]=1 <= nums[mid]=1`? **True** — so Problem 5's logic concludes "left half `[0,2]` is sorted." But `[1, 0, 1]` is *not* sorted — the check passed only because the first and last values of that slice happen to coincide, not because the slice is actually wrap-free. Trusting it: is `1 <= 0 < 1`? False → go right, `left = 3`. From there the search only ever explores indices `3` and `4` (both value `1`) and returns `false` — even though `target = 0` is sitting right there at index `1`. **A real, silent wrong answer**, not a hypothetical one.

**The fix, and why it works:** when `nums[left] == nums[mid] == nums[right]`, those three values genuinely can't distinguish "this side is flat-and-sorted" from "this side contains the wrap, and by coincidence the endpoints match." The only safe move is to give up trying to reason about this range and shrink it from both ends by one (`left++; right--`), discarding one element from consideration without discarding any information the equal endpoints weren't providing anyway (if `nums[left]` held the *only* copy of its value, it — being equal to `nums[mid]` and `nums[right]` — clearly wasn't the target we're comparing against at `mid`, and symmetrically for `nums[right]`).

**Verifying the fix on the same input:** `nums = [1, 0, 1, 1, 1]`, `target = 0`. `left=0, right=4, mid=2, nums[2]=1` (not target). `nums[0]=1 == nums[2]=1 == nums[4]=1` → **flat case** → `left=1, right=3`. Now `mid = 1 + (3-1)/2 = 2`, `nums[2]=1` (not target). Check flat case: `nums[1]=0 == nums[2]=1`? False — not flat. `nums[left]=0 <= nums[mid]=1`? True → left half `[1,2]` sorted. Is `0 <= target(0) < 1`? Yes → `right = mid - 1 = 1`. Now `left=1, right=1, mid=1, nums[1]=0 == target` → return `true`. Correct.

**Complexity:** Time O(log n) **average**, but **O(n) worst case** — Space O(1). The worst case is exactly an array where the flat-triple case fires on every single iteration (e.g., `nums = [1,1,1,1,1,1,1]`, searching for a value that isn't `1`): each step only shrinks the range by one element from each end (net two), so it degrades to a linear scan wearing a binary-search costume.

**Edge cases:**
- All elements identical, target absent — hits the O(n) worst case, still correctly returns `false` eventually.
- All elements identical, target present — matches on the first `nums[mid] == target` check it happens to land on, often well before O(n) work is needed.
- No duplicates at all — the flat-triple branch simply never fires, and this degrades to exactly Problem 5's algorithm.

**⚠️ Common Mistake:** shrinking only one end (`left++` alone, or `right--` alone) instead of both. Moving only one boundary doesn't resolve the ambiguity in general — the safe move is always to give up exactly one element of information from *each* side.

**💡 Interview Insight:** this is one of the most common natural follow-ups to LC 33, so treat it as an expected continuation, not a surprise. The strongest answer names three things unprompted: exactly *which* invariant breaks (the "one half is always cleanly sorted" guarantee — shown concretely above, not just asserted), the fix (shrink both ends on the ambiguous flat case), and the cost (the worst-case complexity guarantee is gone — O(log n) average degrades to O(n) worst case, even though correctness is preserved). Naming the cost unprompted is what separates "I patched it" from "I understand what the patch actually gives up."

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** no new code today — write a short comparison note, saved alongside your solutions, addressing exactly two questions:

1. **What, precisely, does a single duplicate value break** about yesterday's rotated-array logic? A strong answer names the exact failure mode: `nums[left] <= nums[mid]` stops being a reliable signal for "wrap-free," because a flat run of equal values can make the check pass by coincidence even when the slice actually contains the rotation point — use the `[1,0,1,1,1]` example above (or build your own) to show it concretely, not just assert it.
2. **Why does the fix cost the O(log n) guarantee?** Because the fallback (shrinking both ends by one) only ever discards a constant number of elements per step in the worst case, rather than halving the range — an array of all-equal values forces that fallback on every iteration.

Definition of done: a few sentences per question, concrete rather than abstract — if you can't point to a specific input that breaks the naive check, the note isn't done yet.

---

## Career Block Guide (1 hr)

**LinkedIn (20 min):** comment meaningfully on 3–5 posts.

**Networking:** identify 3 companies known for strong **backend** engineering blogs specifically (as opposed to yesterday's general list) — these start mattering concretely once `todo-api` goes live on Day 34, a couple of days from now, and having a short list ready means outreach can start from something you've actually read rather than a cold search.

---

## Day 30 — Interview Questions

**Q1. Why is at least one half of a rotated sorted array always guaranteed to be normally sorted, no matter where you split it?** The array is a single ascending sequence cut once and swapped. A contiguous slice only fails to be sorted if it straddles that cut point, and any split at `mid` can put the cut in at most one of the two resulting halves — so the other one is guaranteed clean.

**Q2. What does `nums[left] <= nums[mid]` actually test?** Whether the slice `[left, mid]` is free of the rotation's wraparound. A genuinely sorted slice never has its first element exceed its last, so if this holds, the left half is safely sorted; if it fails, the wraparound must be inside `[left, mid]`, meaning the right half is the sorted one instead.

**Q3. Once you know which half is sorted, what's the second check, and why is it needed?** Whether `target` actually falls within that sorted half's own value range. Knowing a half is sorted doesn't mean the target is in it — the algorithm still has to compare `target` against that half's own min/max before deciding to recurse there.

**Q4. Give a concrete input where trusting `nums[left] <= nums[mid]` without duplicate-handling produces a wrong answer.** `nums = [1, 0, 1, 1, 1]`, `target = 0`: `nums[0] <= nums[2]` is `1 <= 1`, true, incorrectly signaling the left slice `[1,0,1]` is sorted — it isn't — which routes the search away from index 1, where the actual `0` lives, producing a false negative.

**Q5. What's the fix for that failure, and what does it cost?** When `nums[left] == nums[mid] == nums[right]`, neither half can be trusted, so shrink the range from both ends by one and retry. The cost is the complexity guarantee: this fallback can fire on every iteration for an all-duplicate array, degrading from O(log n) average to O(n) worst case.

**Q6. Does the O(81) version ever return an actually wrong answer, or just a slower one, compared to LC 33's algorithm run on the same duplicate-free-adjusted case?** Just slower in the worst case — the fallback never discards a range that could contain the target, only elements it's already directly confirmed (via the triple-equality check) can't be a useful match at that boundary, so correctness holds throughout; only the time bound is lost.

**Q7. If the array turns out not to be rotated at all (pivot at index 0), what happens to this algorithm?** It degrades gracefully — the "which half is sorted" check finds the entire remaining range sorted at every step, so it behaves identically to plain binary search, with no special-casing needed.

**Q8. Why does LC 81 return a `boolean` while LC 33 returns an `int` index?** That's simply how each problem is specified on LeetCode — with duplicates present, a target's index isn't even well-defined if it's stated to only require existence; the algorithm itself barely changes either way, only the return type and the final `-1`-vs-`false` convention.

---

## Daily Deliverable Check

- [ ] Search in Rotated Sorted Array and Search in Rotated Sorted Array II solved, pushed.
- [ ] Comparison note written, with a concrete counter-example (not just an abstract description) showing what duplicates break.

---

## What Tomorrow Assumes You Already Know Cold

Day 31 assumes today's "identify the sorted half, then check range membership" reasoning is fully automatic — tomorrow's Find Minimum in Rotated Sorted Array reuses the exact same rotated-array intuition, just simplified to a `nums[mid]` vs. `nums[right]` comparison instead of a full range check. It also carries today's duplicate lesson forward directly: Day 31 pairs LC 153 with its own duplicate-handling extension, using precisely the same "can't tell which side is clean, shrink and retry" fix taught here — so today's counter-example-driven understanding, not just the working code, is what tomorrow builds on.
