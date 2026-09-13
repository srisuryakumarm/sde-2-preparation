# Day 31 — Binary Search: Minimum in Rotated Arrays, and Boundary Search

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 30 Resource Book](Day30_Resource_Book.md)
**Next ▶:** [Day 32 Resource Book](Day32_Resource_Book.md)
**Companion to:** Day 31 of `Week_05_Revised.md`

---

## Recap

Day 30 established the rotated-array invariant (one half of any split is always sorted) and then broke it on purpose with duplicates, to show exactly what's lost and what it costs. Today reuses both halves of that lesson on two new shapes: a simpler rotated-array question (just the minimum, not an arbitrary target) that gets its own duplicate-handling extension today rather than next week, and a genuinely different kind of binary search — finding a *boundary* rather than a single match. Two extra-practice reps are added today, since both of today's required problems represent patterns real interviews lean on with more than one rep's worth of weight, and yesterday's problems already covered the "duplicates break things" lesson once — today reinforces it on new ground rather than re-deriving it.

No theory block or project block today — pure DSA, then Career.

---

## Learning Objectives

By the end of today, without notes:

1. Prove why comparing `nums[mid]` to `nums[right]` (not `nums[left]`) is what makes Find Minimum in Rotated Sorted Array's binary search unambiguous.
2. Extend that proof to the duplicate case, and state precisely why `nums[mid] == nums[right]` is irreducibly ambiguous in a way `nums[mid] > nums[right]` and `nums[mid] < nums[right]` are not.
3. Run two independently-biased binary searches to find the first and last occurrence of a value, and explain why one pass can't cleanly do both.
4. Recognize a single-sided boundary search (LC 744) as the same shape as LC 34's leftmost search, simplified.

---

## Concept Dependency Map

```
Day 30: rotated-array invariant (one half always sorted) + duplicate-handling fix (shrink, pay O(n) worst case)
Day 29: Search Insert Position's invariant style (left ends up exactly at the boundary)
        │
        ├──▶ Today, Problem 7: Find Minimum in Rotated Sorted Array (LC 153)
        │        compare nums[mid] to nums[right] — narrower question than Day 30's
        │        (no target to locate, just the rotation point itself)
        │        │
        │        └──▶ Extra Practice 1: LC 154 (duplicates) — same fix shape as Day 30's LC 81
        │
        └──▶ Today, Problem 8: Find First and Last Position (LC 34)
                 NEW: boundary search — two binary searches, each biased to keep
                 narrowing past a match instead of stopping at one
                 │
                 └──▶ Extra Practice 2: LC 744 — same boundary-search shape, one-sided
```

---

# Part 1 — Minimum in Rotated Arrays, and Boundary Search

### Prerequisites (confirmed)

- Rotated-array sortedness invariant, and its duplicate-driven failure mode — Day 30.
- Exact-match template and the "answer is where the pointers converge" reading — Day 28, Day 29.

## Problem 7: Find Minimum in Rotated Sorted Array (LeetCode 153, Medium) — Pattern: Rotated-Array Minimum

**Statement:** `nums` is a rotated sorted array of **distinct** values. Return the minimum element, in O(log n).

### Approach 1 — Brute force

```java
public static int findMinBruteForce(int[] nums) {
    int min = nums[0];
    for (int num : nums) {
        min = Math.min(min, num);
    }
    return min;
}
```

Ignores the rotation structure entirely. Time O(n), Space O(1).

### Approach 2 — Optimized: binary search, comparing `nums[mid]` to `nums[right]`

```java
public static int findMin(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > nums[right]) {
            left = mid + 1;    // the drop (and the minimum) is strictly after mid
        } else {
            right = mid;       // mid could BE the minimum — keep it in range
        }
    }
    return nums[left];   // left == right here
}
```

**Why this works — the invariant, proven both ways:** maintain the claim "`nums[left..right]` contains the array's global minimum" throughout.

- If `nums[mid] > nums[right]`: since `mid < right` always holds here (proof below), a normally-sorted slice from `mid` to `right` would need `nums[mid] <= nums[right]` — so `nums[mid] > nums[right]` proves the rotation's "drop" happens strictly after `mid`. The minimum can't be at or before `mid`, so it's safe to discard `mid` entirely: `left = mid + 1`.
- If `nums[mid] <= nums[right]`: this proves `[mid, right]` contains no drop — a genuinely sorted-and-rotation-free slice — which means `mid` is at or after the rotation point (since a pre-rotation value can never be `<=` a post-rotation value in a true rotation of distinct values; if it were, `mid` couldn't still be in the "high" segment). If the rotation point is at or before `mid`, the true minimum — which sits exactly at that rotation point — must be in `[left, mid]`. So `right = mid` (mid is kept, not excluded, since it might be the minimum itself).

**⚠️ Why compare to `nums[right]`, not `nums[left]` — this is a genuine correctness reason, not a style preference:** with this midpoint formula, `mid` is *always* strictly less than `right` whenever `left < right` (check it: `right - left >= 1`, so `mid = left + floor((right-left)/2)` never reaches `right`). But `mid` **can** equal `left` — in a two-element range (`right = left + 1`), `mid = left` exactly. Comparing `nums[mid]` to `nums[left]` would then sometimes compare an element to *itself*, collapsing the two-way branch into a meaningless `nums[left] > nums[left]` (always false) right at the corner case that most needs a real signal. Comparing to `nums[right]` sidesteps this entirely, since `mid` never coincides with `right`.

**Worked trace:** `nums = [4, 5, 6, 7, 0, 1, 2]`.

| left | right | mid | nums[mid] | nums[right] | comparison | action |
|---|---|---|---|---|---|---|
| 0 | 6 | 3 | 7 | 2 | 7 > 2 | `left = 4` |
| 4 | 6 | 5 | 1 | 2 | 1 ≤ 2 | `right = 5` |
| 4 | 5 | 4 | 0 | 1 | 0 ≤ 1 | `right = 4` |
| 4 | 4 | — | — | — | `left == right` | loop ends |

Returns `nums[4] = 0`. Correct.

**Complexity:** Time O(log n), Space O(1).

**Edge cases:**
- Array not rotated at all — every comparison finds `nums[mid] <= nums[right]`, and `right` walks down to `0`, correctly returning the first element.
- Single element — loop condition `left < right` is false immediately, returns `nums[0]`.
- Two elements — traced implicitly above by the general argument; both orderings (`[1,2]` and `[2,1]`) resolve correctly in one iteration.

**⚠️ Common Mistake:** using `left <= right` as the loop condition instead of `left < right`. With `<=`, once the range narrows to a single element (`left == right`), `mid` also equals that same index, `nums[mid] > nums[right]` is comparing a value to itself (always false), so the algorithm falls into the `else` branch and sets `right = mid` — which doesn't change anything, since `mid` already equaled `right`. The loop never terminates. This is a genuine infinite-loop bug, not a style nitpick — worth tracing once by hand to see it happen.

**💡 Interview Insight:** this problem is frequently asked as a warm-up *specifically* to set up the duplicate-handling follow-up below — treat the two as a pair in how you present them, the same way Day 30's LC 33 and LC 81 were a pair.

---

## Problem 8: Find First and Last Position of Element in Sorted Array (LeetCode 34, Medium) — Pattern: Binary Search, Boundary Search (NEW)

**Statement:** `nums` is sorted and **may contain duplicates**. Given `target`, return `[firstIndex, lastIndex]` of its occurrences, or `[-1, -1]` if absent. Required in O(log n).

### Approach 1 — Brute force

```java
public static int[] searchRangeBruteForce(int[] nums, int target) {
    int first = -1, last = -1;
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] == target) {
            if (first == -1) first = i;
            last = i;
        }
    }
    return new int[]{first, last};
}
```

Time O(n), Space O(1).

### Approach 2 — Optimized: two binary searches, each biased to keep narrowing on a match

```java
public static int[] searchRange(int[] nums, int target) {
    int first = findBound(nums, target, true);
    if (first == -1) {
        return new int[]{-1, -1};   // absent entirely — no point running the second search
    }
    int last = findBound(nums, target, false);
    return new int[]{first, last};
}

private static int findBound(int[] nums, int target, boolean findFirst) {
    int left = 0, right = nums.length - 1;
    int result = -1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            result = mid;
            if (findFirst) {
                right = mid - 1;   // record it, then keep hunting further LEFT
            } else {
                left = mid + 1;    // record it, then keep hunting further RIGHT
            }
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return result;
}
```

**🔑 Key Takeaway — this is the genuinely new mechanism this week:** every binary search so far, on finding a match, has stopped and returned. Boundary search does the opposite: on a match, it *records* the candidate but keeps searching, deliberately narrowing toward one side, because there might be an even better (further left, or further right) match still ahead. The result only becomes final once the loop actually exhausts that side.

**Mnemonic worth internalizing exactly, not approximately:** hunting for the **first** occurrence → after a match, look further **left** (`right = mid - 1`). Hunting for the **last** → after a match, look further **right** (`left = mid + 1`). Getting these backwards is the single easiest way to fail this problem while still "understanding the idea."

**Worked trace:** `nums = [5, 7, 7, 8, 8, 10]`, `target = 8`.

*Finding first:* `left=0,right=5 → mid=2,nums[2]=7<8 → left=3` · `left=3,right=5 → mid=4,nums[4]=8==8 → result=4, right=3` (keep looking left) · `left=3,right=3 → mid=3,nums[3]=8==8 → result=3, right=2` · `left=3 > right=2`, loop ends → returns `3`.

*Finding last:* `left=0,right=5 → mid=2,nums[2]=7<8 → left=3` · `left=3,right=5 → mid=4,nums[4]=8==8 → result=4, left=5` (keep looking right) · `left=5,right=5 → mid=5,nums[5]=10>8 → right=4` · `left=5 > right=4`, loop ends → returns `4`.

Result: `[3, 4]`. Check against the array: indices 3 and 4 both hold `8`; index 2 holds `7` and index 5 holds `10` — correct boundaries on both sides.

**Complexity:** Time O(log n) — two independent binary searches back to back is `2 × O(log n)`, which is still O(log n) (the constant factor doesn't change the complexity class). Space O(1).

**Edge cases:**
- Target entirely absent (e.g., `target = 6` in the array above) — `findBound(..., true)` never sets `result`, returns `-1`, and the second search is skipped entirely (a real, worthwhile optimization, not just a formality — no point running a second O(log n) search once absence is confirmed).
- Exactly one occurrence — `first == last`.
- Target at the very first or very last index of the array.
- Empty array — `left = 0, right = -1`, loop body never runs, `findBound` correctly returns `-1` with no out-of-bounds access.

**⚠️ Common Mistake:** running both searches unconditionally, even when the first already proves absence — correct, but wasteful, and it signals not having thought through the control flow.

**💡 Interview Insight:** a very likely follow-up: *"can this be done in a single pass?"* The honest, strong answer is: not cleanly — you could try to interleave both searches, but it doesn't change the O(log n) complexity class and meaningfully hurts readability for no asymptotic gain. Naming that trade-off directly (rather than trying to force a "clever" one-pass version under pressure) reads as better engineering judgment, not less cleverness.

---

# Part 2 — Extra Practice: Two More Reps

Both of today's required problems represent patterns with real, recurring interview weight, and each only gets one plan-required rep — exactly the situation the whole point of adding extra practice is for.

---

## Extra Practice 1: Find Minimum in Rotated Sorted Array II (LeetCode 154, Medium) — Pattern: Rotated-Array Minimum, With Duplicates

**Statement:** Same as Problem 7, except `nums` **may contain duplicates**.

### Approach — same skeleton, one new branch for the ambiguous case

```java
public static int findMinWithDuplicates(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > nums[right]) {
            left = mid + 1;
        } else if (nums[mid] < nums[right]) {
            right = mid;
        } else {
            // nums[mid] == nums[right]: genuinely ambiguous — drop right by one and retry
            right--;
        }
    }
    return nums[left];
}
```

**⚠️ Why `nums[mid] == nums[right]` is irreducibly ambiguous — not just "harder," actually undecidable from that information alone:** the two clean branches above each rely on a *strict* inequality proving something about where the rotation's drop is. When the two values are equal, neither proof goes through — `mid` could be a value from the "high" pre-rotation segment that just happens to coincide numerically with a "low" post-rotation duplicate at `right`, or `mid` could already be inside the low segment alongside `right`. No comparison of just these two values can tell those cases apart.

**The fix — and why it loses no information:** drop `right` by one (`right--`) rather than making a directional decision. This is safe specifically *because* `nums[right] == nums[mid]`: whatever role that value might have played in locating the minimum, an identical value still exists at `mid`, which remains inside the search range — so nothing reachable is lost, only one redundant data point is discarded.

**Worked trace:** `nums = [10, 1, 10, 10, 10]`. `left=0,right=4,mid=2,nums[2]=10,nums[right]=nums[4]=10` → equal → `right-- → right=3`. `left=0,right=3,mid=1,nums[1]=1,nums[right]=nums[3]=10` → `1 < 10` → `right=mid=1`. `left=0,right=1,mid=0,nums[0]=10,nums[right]=nums[1]=1` → `10 > 1` → `left=mid+1=1`. `left=1,right=1`, loop ends → returns `nums[1]=1`. Correct.

**Complexity:** Time O(log n) average, **O(n) worst case** — Space O(1). The worst case is exactly the same shape as Day 30's LC 81: an all-duplicate array (e.g., `[2,2,2,2,2,2]`) forces the ambiguous branch on every single iteration, each time discarding only one element.

**🔗 Direct connection:** this is the identical shape as yesterday's LC 33 → LC 81 jump. Duplicates remove the ability to always cleanly halve the range; the fix is to give up a constant amount of certainty per step instead of a clean half; the cost is the same O(log n) → O(n) worst-case degradation. If this is starting to feel like a repeatable move rather than a new trick each time, that's exactly the point — recognizing "duplicates in a rotated-array problem always cost the same specific thing" is a stronger interview signal than re-deriving the fix from scratch each time it comes up.

---

## Extra Practice 2: Find Smallest Letter Greater Than Target (LeetCode 744, Easy) — Pattern: Boundary Search, Single-Sided

**Statement:** `letters` is a sorted (and effectively circular) array of characters, possibly with repeats. Given `target`, return the smallest letter in `letters` **strictly greater** than `target`. If `target` is `>=` every letter, wrap around and return the first letter.

### Approach — boundary search, one side only

```java
public static char nextGreatestLetter(char[] letters, char target) {
    int left = 0, right = letters.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (letters[mid] <= target) {
            left = mid + 1;    // not good enough (equal doesn't count) — must be further right
        } else {
            right = mid - 1;
        }
    }
    return letters[left % letters.length];   // wraps if target >= every letter
}
```

**Why this converges correctly — same invariant style as Day 29's Search Insert Position:** everything before `left` ends up `<= target` (disqualified — strictly greater is required), everything from `left` onward is `> target`. So `left` lands exactly on the first letter strictly greater than target. `% letters.length` handles the one genuinely new wrinkle: if `target` is `>=` every letter, `left` walks all the way to `letters.length` — one past the end — and the modulo wraps that back to index `0`, exactly the required wraparound behavior.

**Worked trace:** `letters = ['c','f','j']`, `target = 'j'` (equal to the max — should wrap). `left=0,right=2,mid=1,'f'<='j'→left=2`. `left=2,right=2,mid=2,'j'<='j'→left=3`. `left=3>right=2`, loop ends → `letters[3 % 3] = letters[0] = 'c'`. Correct — wraps to the first letter.

**Complexity:** Time O(log n), Space O(1).

**Edge cases:**
- Target smaller than every letter → converges to `left = 0` directly, no wraparound needed.
- Target equal to or larger than every letter → `left` reaches `letters.length`, modulo wraps to `0`.
- Duplicate letters (e.g., `['c','c','c']`, `target='c'`) → the `<=` branch correctly steps past every copy of `'c'` before landing on the first strictly-greater letter (or wrapping, if none exists).
- Single-letter array.

**⚠️ Common Mistake:** using `<` instead of `<=` in the main branch. Since the problem needs *strictly* greater, an element equal to `target` must be treated as "not good enough, keep going right" — exactly what `letters[mid] <= target` captures by folding the equal case into the "move right" branch.

**🔗 Direct connection:** this is the same boundary-search shape as LC 34's leftmost search from Problem 8 — one binary search, biased to converge on the first index past a threshold — simplified here to a single one-sided condition (`<=` vs. `>`) rather than needing two independently-biased passes, since there's only one boundary being asked for, not two.

---

## Career Block Guide (1 hr)

**LinkedIn (20 min):** comment meaningfully on 3–5 posts.

**Networking:** comment meaningfully on 5 posts from existing connections — deeper engagement with people already in your network, rather than expanding it today.

---

## Day 31 — Interview Questions

**Q1. Why compare `nums[mid]` to `nums[right]` rather than `nums[left]` in Find Minimum in Rotated Sorted Array?** Because `mid` can equal `left` in a two-element range (comparing a value to itself, losing the signal at exactly the corner case that needs it most), but `mid` never equals `right` given this midpoint formula — so comparing to `right` always compares two genuinely distinct elements.

**Q2. In that same problem, when `nums[mid] > nums[right]`, why is it safe to discard `mid` entirely?** Because that inequality proves the rotation's "drop" happens strictly after `mid` — a normally sorted slice from `mid` to `right` would require `nums[mid] <= nums[right]`, so violating that means the minimum can't be at or before `mid`.

**Q3. What real information does `nums[mid] == nums[right]` fail to give you, in the duplicate version?** It can't distinguish "mid is a high pre-rotation value that happens to numerically match a low post-rotation duplicate at right" from "mid is already inside the low segment alongside right" — both are consistent with the same equality, so no directional decision can be made safely.

**Q4. Why is discarding `right` (not `mid`, not `left`) the safe move in that ambiguous case?** Because an identical value still exists at `mid`, which stays inside the search range — so nothing reachable is lost by dropping the one redundant duplicate at `right`.

**Q5. In Find First and Last Position, why doesn't the algorithm just return immediately on the first match, like ordinary binary search does?** Because a match doesn't guarantee it's the *boundary* match — there could be more occurrences further in the direction being searched, so the match is recorded as a candidate and the search keeps narrowing until that's ruled out.

**Q6. State the exact rule for which direction to narrow after a match, for "first" vs. "last."** Finding the first occurrence: after a match, narrow left (`right = mid - 1`). Finding the last: after a match, narrow right (`left = mid + 1`).

**Q7. Why skip the second search entirely when the first one returns `-1`?** A `-1` from the first search means the target isn't in the array at all — running a second search to find a "last occurrence" of something proven absent would just re-derive the same `-1`, at real (if asymptotically harmless) extra cost.

**Q8. How does Find Smallest Letter Greater Than Target relate to Find First and Last Position?** Same boundary-search shape — one binary search converging on the first index past a threshold — but simplified to a single condition (`<=` moves right, else moves left) instead of two independently-biased searches, since only one boundary is needed here, not both a first and a last.

---

## Daily Deliverable Check

- [ ] Find Minimum in Rotated Sorted Array and Find First/Last Position solved, pushed.
- [ ] Extra practice: Find Minimum in Rotated Sorted Array II and Find Smallest Letter Greater Than Target solved, pushed — both checked against `00_Curriculum_Map.md` and `Week_06_Revised.md` beforehand; neither collides with anything already solved or about to be required.

---

## What Tomorrow Assumes You Already Know Cold

Day 32 assumes today's rotated-array and boundary-search mechanics are both solid, but doesn't extend either directly — tomorrow introduces two more genuinely distinct binary-search shapes (searching a 2D structure, and searching over a range of *possible answers* rather than array indices). What does carry forward directly is the proof discipline from today and Day 29: state the invariant, show each branch preserves it, show it forces a unique correct answer at convergence — tomorrow's "on the answer" problems need that same rigor applied to a monotonic *feasibility check* instead of a monotonic array.
