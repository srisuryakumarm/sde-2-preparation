# Day 7 Resource Book — Two Pointers Continues, and Week 1 Consolidation

**Series:** Week 1 SDE-2 Prep · [← Curriculum Map](./00_Curriculum_Map.md) · [← Day 6](./Day6_Resource_Book.md)

**Companion to:** Day 7 of `Week_01_Revised.md`

---

## Recap: closing out the week

Two more Two Pointers variants today, then a full Week 1 consolidation. By the end of today you'll have solved **24 DSA problems this week** (12 HashMap/HashSet + 12 Two Pointers, counting this series' extra practice) against your plan's original 12 — the direct result of the "more reps per pattern" requirement this whole series was built around.

## Learning Objectives

By the end of today, without notes:

1. Justify, in one sentence each, when to reach for `ArrayList`, `HashSet`, `HashMap`, and `ArrayDeque`.
2. Solve all six of today's problems, including 3Sum — which deliberately combines Day 5's HashMap-era thinking with Day 6's Two Pointers into one problem.
3. Explain precisely why Container With Most Water's greedy pointer-movement rule is provably correct, not just empirically effective.
4. Summarize, without re-reading, everything Week 1 covered and what Week 2 will assume you already know cold.

## Concept Dependency Map for Today

```
Two Pointers (opposite-ends, from-the-back, same-direction — Day 6)
        │
        ▼
Two more variants:
  4. Single forward pointer EACH, across TWO different structures   (Is Subsequence)
  5. Opposite ends + one allowed "skip" on mismatch                  (Valid Palindrome II)
        │
        ▼
Extra practice, including a genuine pattern-combination:
  Move Zeroes (same-direction) · Container With Most Water (opposite-ends, greedy)
  3Sum (Day 5's "fix one, reduce to Two Sum" + Day 6's Two Pointers, composed)
  Remove Element (same-direction, no sortedness required — a useful contrast)
        │
        ▼
Week 1 Consolidation
```

---

# Self-Check (10 Minutes)

Before today's new material, answer each of these in one sentence, without looking anything up. This is the actual test of whether the last four days' tools are reflexive yet.

- **When do you reach for `ArrayList`?**
- **When do you reach for `HashSet`?**
- **When do you reach for `HashMap`?**
- **When do you reach for `ArrayDeque`?**

<br>

**Model answers** — compare, don't just read:

- **`ArrayList`** — when you need ordered, indexed access to a collection that needs to grow, and you don't need fast membership checks.
- **`HashSet`** — when you need to track uniqueness or answer "have I seen this before?" in average O(1) time, and order doesn't matter.
- **`HashMap`** — when you need to associate *extra information* with each unique item (a count, an index, a running value) — not just a yes/no — in average O(1) time.
- **`ArrayDeque`** — when you need LIFO or FIFO access from one or both ends, and you never need indexed access into the middle.

If any of these took real thought rather than immediate recall, that's worth five more minutes with Day 3-4 before continuing — today assumes this is automatic.

---

# Part 1 — Two More Two Pointers Variants

## Problem 4: Is Subsequence (LeetCode 392, Easy) — Variant: One Forward Pointer Each, Across Two Structures

**Statement:** Given strings `s` and `t`, return `true` if `s` is a subsequence of `t` — `s`'s characters appear in `t` in the same relative order, not necessarily contiguously.

This introduces a genuinely new shape: **two separate structures, each with its own independent pointer**, advancing at different rates — distinct from same-direction fast/slow (Day 6), which operates on *one* structure.

### Approach 1 — Recursive

```java
public static boolean isSubsequenceRecursive(String s, String t) {
    return helper(s, t, 0, 0);
}

private static boolean helper(String s, String t, int i, int j) {
    if (i == s.length()) return true;    // matched every character of s — success
    if (j == t.length()) return false;   // ran out of t before matching all of s
    if (s.charAt(i) == t.charAt(j)) {
        return helper(s, t, i + 1, j + 1);   // match — advance both
    }
    return helper(s, t, i, j + 1);            // no match — advance only t
}
```

Clear, direct translation of "for each character of `t`, either it matches what we're looking for in `s` next, or it doesn't." Correct, and O(s.length() + t.length()) time — but each recursive call adds a frame to the call stack, giving **O(s.length() + t.length()) space**, purely from recursion depth.

### Approach 2 — Optimized: iterative two pointers

```java
public static boolean isSubsequence(String s, String t) {
    int i = 0, j = 0;
    while (i < s.length() && j < t.length()) {
        if (s.charAt(i) == t.charAt(j)) {
            i++;
        }
        j++;
    }
    return i == s.length();
}
```

Identical logic, expressed iteratively: `j` advances through `t` unconditionally, every step; `i` advances through `s` only on a match. If `i` reaches the end of `s`, every character was found in order — `s` is a subsequence.

**The actual trade-off between these two, worth stating precisely:** identical time complexity; the iterative version drops the space cost to **O(1)**, since there's no call stack accumulating. Recursion can read more directly as "the problem's own definition," but the space cost is real — for this problem, the iterative version is strictly better on every axis that matters.

**Complexity: Time O(s.length() + t.length()), Space O(1)** for the iterative version — stated with both lengths named, per Day 3's rule, since `s` and `t` aren't guaranteed related sizes.

**Edge cases:** empty `s` (trivially a subsequence of anything — the loop never runs, `i` starts and ends at `0 == s.length()`, correctly `true`); empty `t` with non-empty `s` (loop never runs, `i` stays below `s.length()`, correctly `false`); `s` longer than `t` (can never be a subsequence — `i` will exhaust `t`'s scanning room before matching everything, correctly `false`).

### A genuine, well-known follow-up worth knowing: many queries against the same `t`

This exact problem has a famous follow-up: *what if you need to check a large number of different `s` strings against the same `t`, repeatedly?* Running the O(t.length()) two-pointer scan fresh for every single query becomes wasteful at scale. The better approach: **preprocess `t` once** — build a map from each character to the sorted list of indices where it appears in `t` — then answer each subsequent query by, for every character of `s`, finding the smallest recorded index *greater than* the last index used, via **binary search** into that character's index list.

🔗 **Forward reference:** binary search itself — the mechanism that makes this follow-up efficient — is Week 2's opening topic (previewed back on Day 3 as the canonical O(log n) example). The shape of the optimization is worth having now — preprocess once, then answer each query faster than a fresh linear scan — even before the binary-search mechanics themselves are formally covered.

> 💡 **Interview Insight:** Proactively raising this follow-up — "if this needed to run many times against the same `t`, I'd preprocess `t`'s character positions and binary-search each query instead of rescanning" — even without fully implementing it, signals you're thinking about the problem's *actual* constraints rather than stopping the moment a single correct solution exists.


---

## Problem 5: Valid Palindrome II (LeetCode 680, Easy) — Variant: Opposite Ends, One Allowed Skip

**Statement:** Given a string, return `true` if it can become a palindrome after deleting **at most one** character.

```java
public static boolean validPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) {
            return isPalindromeRange(s, left + 1, right) || isPalindromeRange(s, left, right - 1);
        }
        left++;
        right--;
    }
    return true;   // no mismatch at all — already a palindrome, zero deletions needed
}

private static boolean isPalindromeRange(String s, int left, int right) {
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}
```

**Why exactly two candidates, tried at exactly one point:** scanning inward as usual, everything *before* the first mismatch already agrees correctly — no changes needed there. At the **first** disagreement, there are exactly two possible single-character fixes: the problem is the *left* character (skip it — check whether `left+1..right` is a palindrome) or the problem is the *right* character (skip it — check `left..right-1`). Nothing else needs checking: since only one deletion is allowed total, that single deletion must resolve *this specific* disagreement — if neither of the two possible skips resolves it, no other combination of choices could either, because this exact conflict would remain either way.

**Why you don't need to examine every mismatch, only the first:** any fix must be applied at the earliest point of disagreement, since everything before it is already settled and correct. There's no scenario where skipping *later* in the string retroactively fixes an *earlier* disagreement.

> ⚠️ **Common Mistake:** Attempting to skip *both* the left and right character on a mismatch (checking `left+1..right-1`) — that uses **two** deletions, exceeding the allowed budget of one, and is a genuinely easy slip to make without pausing to count exactly how many characters each candidate check is actually removing.

**Complexity: Time O(n)** — the main scan is O(n), and in the worst case, the two `isPalindromeRange` sub-checks each cost up to O(n), giving O(n) + O(n) + O(n) = O(3n), which simplifies to **O(n)** under Day 3's constant-dropping rule. **Space O(1).**

**Edge cases:** already a palindrome (returns `true` without ever hitting the mismatch branch — zero deletions used, correctly within budget); a string needing exactly one deletion (correctly resolved by one of the two branches); a string needing more than one deletion (correctly `false`, since neither single-skip branch can fully resolve it); single character or empty string (trivially `true`).


---

# Part 2 — Extra Practice: Four More Reps

## Extra Practice 1: Move Zeroes (LeetCode 283, Easy) — Variant: Same Direction (Swap, Not Overwrite)

**Statement:** Given an array, move all zeroes to the end while preserving the relative order of non-zero elements, in place.

```java
public static void moveZeroes(int[] nums) {
    int slow = 0;
    for (int fast = 0; fast < nums.length; fast++) {
        if (nums[fast] != 0) {
            int temp = nums[slow];
            nums[slow] = nums[fast];
            nums[fast] = temp;
            slow++;
        }
    }
}
```

**A worthwhile refinement on Day 6's Remove Duplicates pattern:** same fast/slow shape, but a **swap** instead of an overwrite. Remove Duplicates could safely overwrite because trailing "leftover" values beyond the returned length were explicitly allowed to be garbage. Here, every element must be *preserved somewhere* in the array — zeroes need to end up at the back, not be discarded — so a full swap is required.

**Why the swap is always correct — the invariant worth stating explicitly:** at the moment of any swap, everything in `nums[slow..fast-1]` must be zero. Why: `slow` only fails to advance on exactly the positions `fast` skipped over (the zeroes) — so the gap between `slow` and `fast` is composed entirely of zeroes encountered so far. Swapping the current non-zero value into position `slow` and, in the same motion, moving that guaranteed-zero value into position `fast`, correctly relocates the zero further back — exactly where it needs to go — while preserving it rather than discarding it.

**A worthwhile alternative for the trade-off discussion:**

```java
// New array — O(n) time, O(n) EXTRA space
public static void moveZeroesNewArray(int[] nums) {
    int[] result = new int[nums.length];
    int index = 0;
    for (int num : nums) {
        if (num != 0) {
            result[index++] = num;
        }
    }
    // remaining slots are already 0 by Java's array default
    System.arraycopy(result, 0, nums, 0, nums.length);
}
```

Same time complexity, but O(n) extra space instead of O(1) — worth naming explicitly as the trade-off, even though the in-place swap version is the stronger answer.

**Complexity: Time O(n), Space O(1)** for the optimized version. **Edge cases:** no zeroes present (array unchanged — `slow` tracks `fast` in lockstep, harmlessly swapping each element with itself); all zeroes (`slow` never advances, array unchanged); zeroes already at the end (correctly a no-op in final result).

---

## Extra Practice 2: Container With Most Water (LeetCode 11, Medium) — Variant: Opposite Ends, Provable Greedy

**Statement:** Given an array of heights, choose two lines that, with the x-axis, form a container holding the most water. Area between lines `i` and `j` is `min(height[i], height[j]) × (j - i)`.

### Approach 1 — Brute force

```java
public static int maxAreaBruteForce(int[] height) {
    int maxArea = 0;
    for (int i = 0; i < height.length; i++) {
        for (int j = i + 1; j < height.length; j++) {
            maxArea = Math.max(maxArea, Math.min(height[i], height[j]) * (j - i));
        }
    }
    return maxArea;
}
```

Every pair, checked directly — correct, **O(n²)**.

### Approach 2 — Optimized: two pointers, always shrinking from the shorter side

```java
public static int maxArea(int[] height) {
    int left = 0, right = height.length - 1;
    int maxArea = 0;
    while (left < right) {
        int area = Math.min(height[left], height[right]) * (right - left);
        maxArea = Math.max(maxArea, area);
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    return maxArea;
}
```

**This is worth a full, precise correctness argument — "trust the greedy" isn't good enough, and this is exactly the kind of thing a sharp interviewer will ask you to prove.** Start with `left = 0`, `right = n-1` — the widest possible container. Say `height[left] < height[right]` (left is the shorter side). Claim: **moving `right` inward instead of `left` can never produce a better area than what's already recorded.**

Proof: if `right` moves to some `right' < right`, the width strictly shrinks (`right' - left < right - left`). The new area is `min(height[left], height[right']) × (right' - left)`. Since `height[left]` is unchanged and was *already* the smaller of the two original heights, the new limiting height is **at most** `height[left]` — it can never exceed it, regardless of what `height[right']` turns out to be. So the new area is bounded by `height[left] × (right' - left)`, which is strictly *less* than the current area `height[left] × (right - left)`, since the width term shrank while the height ceiling stayed the same or got smaller. **Moving the taller pointer is therefore provably dominated — it can never beat the current best.** Which means moving the *shorter* pointer is the only move that has any possibility of improving the answer, and it's always safe to make, because the alternative is proven useless.

**Complexity: Time O(n) — each pointer only ever moves inward, so total movement across the whole run is bounded by n; Space O(1).** Versus the brute force's O(n²).

> ⚠️ **Common Mistake:** Moving *both* pointers on the same step (this can skip over the true optimal configuration entirely — only ever move the single shorter-side pointer, one step at a time); moving the taller pointer instead of the shorter one (proven above to never help, and doing so without being able to justify *why* the shorter-pointer rule is correct is a real gap even if the code happens to still work).

**Edge cases:** all equal heights (any pair shares the same limiting height, so the widest pair — checked first — is optimal; the algorithm still correctly explores down to confirm this); exactly two elements (the only possible pair, computed once); strictly increasing or decreasing height sequences (handled correctly by the same general argument, no special-casing needed).


---

## Extra Practice 3: 3Sum (LeetCode 15, Medium) — Synthesis: HashMap-Era Thinking + Two Pointers, Composed

**Statement:** Given an array, return all **unique** triplets `[a, b, c]` such that `a + b + c = 0`.

This problem is included deliberately as a capstone: it's not a new pattern, but a genuine **composition** of two patterns from this week — Day 5's "fix one element, reduce to a smaller lookup problem" thinking, and Day 6's sorted-array Two Pointers — into a single, harder problem. Recognizing that composition is the actual skill this problem is testing.

### Approach 1 — Brute force

```java
public static List<List<Integer>> threeSumBruteForce(int[] nums) {
    Set<List<Integer>> resultSet = new HashSet<>();
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            for (int k = j + 1; k < nums.length; k++) {
                if (nums[i] + nums[j] + nums[k] == 0) {
                    List<Integer> triplet = new ArrayList<>(List.of(nums[i], nums[j], nums[k]));
                    Collections.sort(triplet);
                    resultSet.add(triplet);
                }
            }
        }
    }
    return new ArrayList<>(resultSet);
}
```

Every triplet, checked directly — **O(n³)**, plus the overhead of a `HashSet<List<Integer>>` for deduplication. (This works correctly, worth noting explicitly, because Java's built-in `List` implementations correctly implement `equals()`/`hashCode()` based on *content* — two lists with the same elements in the same order are equal and hash identically, honoring exactly the contract from Day 4.)

### Approach 2 — Optimized: sort, then fix one element and Two-Sum-II the rest

```java
public static List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);   // enables BOTH the two-pointer search AND simple duplicate-skipping
    List<List<Integer>> result = new ArrayList<>();

    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) {
            continue;   // skip duplicate "fixed" values
        }
        if (nums[i] > 0) {
            break;      // sorted + positive: nothing after this can possibly sum to zero
        }

        int left = i + 1, right = nums.length - 1;
        int target = -nums[i];

        while (left < right) {
            int sum = nums[left] + nums[right];
            if (sum == target) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;
                left++;
                right--;
            } else if (sum < target) {
                left++;
            } else {
                right--;
            }
        }
    }
    return result;
}
```

**The composition, stated explicitly:** for each *fixed* `nums[i]`, the remaining question is "find two numbers in the rest of the (sorted) array summing to exactly `-nums[i]`" — which is precisely **Two Sum II** (Day 6), applied to the subarray after index `i`. 3Sum is "fix one element, then solve a smaller Two Sum on what's left," repeated once per possible fixed element. This isn't a coincidental resemblance — it's a direct, deliberate reuse of an already-mastered technique as a subroutine inside a larger problem.

**Why sorting first does double duty:** it *enables* the two-pointer sub-search (which, per Day 6, requires sortedness to be valid at all), **and** it makes duplicate-skipping trivial — sorting guarantees duplicate values become adjacent, so "skip if equal to the value just tried" is sufficient, exactly the same insight Day 6's Remove Duplicates from Sorted Array relied on.

**How the duplicate-skipping actually prevents repeated triplets:** `if (i > 0 && nums[i] == nums[i-1]) continue;` avoids re-using the same value as the "fixed" element twice in a row (which would just re-derive triplets already found). After recording a valid triplet, the two inner `while` loops advance `left`/`right` past any further values identical to the ones just used, preventing the *same* triplet from being recorded again with a merely-shifted-but-equal pointer position.

**Why `if (nums[i] > 0) break;` is a safe, provable early exit:** once sorted, if `nums[i]` is positive, then every value at or after index `i` is also `≥ nums[i] > 0` — so any triplet formed from this point onward would sum three non-negative numbers, at least one strictly positive, which can never equal zero. `break` (not `continue`) is correct specifically because sortedness guarantees *nothing further* can work either, not just this one `i`.

**Complexity: Time O(n²)** — O(n log n) to sort, then n outer iterations each running an O(n) two-pointer inner scan, giving O(n²), which dominates the sort's O(n log n) (n² grows faster, per Day 3's dominant-term rule) — a real, substantial improvement over the brute force's O(n³). **Space: O(log n) to O(n)** depending on the sort's internal implementation, plus output space for the triplets found (conventionally not counted against the algorithm's own space complexity).

**Edge cases:** fewer than 3 elements (the loop bound `nums.length - 2` correctly prevents any iteration — no triplets possible); all zeroes (`[0,0,0,0]` should produce exactly one triplet, `[0,0,0]` — correctly handled by the duplicate-skipping logic, which prevents it from appearing more than once); no valid triplet exists (correctly returns an empty list).

> 💡 **Interview Insight:** Explicitly naming the composition — "this reduces to Two Sum II once I fix one element and sort" — before writing code is a much stronger opening than silently arriving at the two-pointer solution. It demonstrates the ability to decompose a harder, unfamiliar-looking problem into pieces you've already solved, which is a large part of what separates "has memorized this specific problem" from "can actually reason about new ones."


---

## Extra Practice 4: Remove Element (LeetCode 27, Easy) — Variant: Same Direction, No Sortedness Required

**Statement:** Given an array and a value `val`, remove all occurrences of `val` in place; return the new length.

```java
public static int removeElement(int[] nums, int val) {
    int slow = 0;
    for (int fast = 0; fast < nums.length; fast++) {
        if (nums[fast] != val) {
            nums[slow] = nums[fast];
            slow++;
        }
    }
    return slow;
}
```

Structurally identical to Day 6's Remove Duplicates — same fast/slow, overwrite-based shape. **The genuinely useful contrast worth drawing out explicitly:** this problem places **no sortedness requirement on its input at all**, unlike Remove Duplicates. Why the difference is legitimate: Remove Duplicates' keep/discard decision depends on comparing an element to its *neighbor* (`nums[fast] != nums[slow]`), which only reliably catches every duplicate if duplicates are guaranteed adjacent — hence the sortedness requirement. Remove Element's keep/discard decision (`nums[fast] != val`) depends only on a **fixed, external value**, entirely independent of neighboring elements — so it works correctly on any array, sorted or not.

> 🔑 **Key Takeaway:** not every same-direction fast/slow problem requires sorted input — only the ones where the "is this valid to keep" test itself depends on comparing to a *neighboring* element. Recognizing which sub-case a new problem falls into, rather than assuming every fast/slow problem needs pre-sorted data, is the actual generalization this pair of problems is meant to teach.

**Complexity: Time O(n), Space O(1). Edge cases:** `val` never appears (`slow` tracks `fast` in lockstep, full length returned); every element equals `val` (`slow` never advances, `0` returned); empty array (`0`, loop never runs).

---

# Section — Career Block

- **Weekly Industry Awareness Ritual (20 minutes):** clear the TLDR Newsletter backlog, read one engineering blog post — same recurring habit as Day 3.
- **Weekly Scorecard:** seven days in, **24 total DSA problems solved** in this series (12 HashMap/HashSet + 12 Two Pointers) — double your plan's original 12, which was exactly the point of the extra-practice requirement this whole series was built around. Foundations (Java fundamentals through OOP) fully closed in 5 days. Two Pointers alone is now 12 problems deep. Week 2 picks up with more Two Pointers and opens Sliding Window.

---

# Day 7 — Interview Questions

---

**1. What's the Two Pointers variant in Is Subsequence, and how does it differ from Remove Duplicates' fast/slow variant?**

*Answer:* One forward pointer *each*, across two *different* structures (`s` and `t`), advancing at different rates. Remove Duplicates uses two pointers within a *single* structure — a genuinely different shape.

---

**2. Recursive vs. iterative Is Subsequence — what's the actual trade-off?**

*Answer:* Identical time complexity, O(s.length() + t.length()). The recursive version costs O(s.length() + t.length()) space from call-stack depth; the iterative version is O(1) space — a real, meaningful difference favoring iteration here.

---

**3. What's the follow-up optimization for Is Subsequence with many `s` queries against one fixed `t`?**

*Answer:* Preprocess `t` once into a map from character to sorted list of indices, then answer each query via binary search for the next valid index greater than the last one used — avoiding a fresh O(t.length()) scan per query.

---

**4. In Valid Palindrome II, why check only the first mismatch, not every mismatch?**

*Answer:* Everything before the first mismatch is already correct. The single allowed deletion must resolve that first disagreement — if neither of the two possible single-character skips fixes it, no other combination of choices could either.

---

**5. Why would skipping both the left and right character on a mismatch be wrong in Valid Palindrome II?**

*Answer:* That uses two deletions, exceeding the problem's budget of at most one.

---

**6. In Move Zeroes, why swap instead of overwrite, unlike Remove Duplicates?**

*Answer:* Remove Duplicates allows discarding trailing leftover values; Move Zeroes must preserve every element somewhere (zeroes moved to the back, not discarded), so a full swap — not an overwrite — is required.

---

**7. What invariant guarantees Move Zeroes' swap always relocates a zero?**

*Answer:* Everything in `nums[slow..fast-1]` is guaranteed to be zero at the moment of any swap, since `slow` only lags behind `fast` on exactly the positions that were skipped (the zeroes).

---

**8. In Container With Most Water, why is it always correct to move the shorter-side pointer inward?**

*Answer:* Moving the taller pointer instead is provably dominated: the width shrinks, and the limiting height can never exceed the shorter side's height (unchanged), so the resulting area can only be equal or worse. Moving the shorter pointer is the only move with any chance of improving the answer.

---

**9. Why is moving the taller pointer guaranteed to never produce a better area?**

*Answer:* The limiting height is `min(height[left], height[right])`. If the taller side moves, the shorter side's height still caps the area, while the width strictly shrinks — the new area can only be less than or equal to the current one.

---

**10. How does 3Sum reduce to Two Sum II?**

*Answer:* Fixing one element `nums[i]` turns the remaining question into "find two numbers in the rest of the sorted array summing to `-nums[i]`" — exactly Two Sum II, applied to the subarray after index `i`, repeated for every possible fixed element.

---

**11. Why must 3Sum sort the array first? Name both reasons.**

*Answer:* Sorting enables the two-pointer sub-search (which requires sortedness to be valid), and it makes duplicate-skipping trivial, since sorting guarantees duplicate values become adjacent.

---

**12. How does 3Sum's duplicate-skipping logic work?**

*Answer:* Skip a repeated "fixed" value (`nums[i] == nums[i-1]`) to avoid re-deriving the same triplets. After recording a valid triplet, advance `left`/`right` past any further values identical to the ones just used, to avoid recording the same triplet again.

---

**13. Why does `if (nums[i] > 0) break;` safely terminate 3Sum early?**

*Answer:* Once sorted, if `nums[i]` is positive, every value from that point onward is also positive — three non-negative numbers, at least one strictly positive, can never sum to zero. `break` (not `continue`) is valid because sortedness guarantees nothing later could work either.

---

**14. What's 3Sum's overall time complexity, and why does the O(n²) term dominate the O(n log n) sort?**

*Answer:* O(n²) — n outer iterations, each with an O(n) two-pointer scan. n² grows faster than n log n as n increases, so it's the dominant term once both are combined.

---

**15. How does Remove Element's technique differ from Remove Duplicates', in terms of what precondition each needs?**

*Answer:* Remove Duplicates needs sorted input, because its keep/discard check compares an element to its neighbor. Remove Element's check compares against a fixed external value, independent of neighbors, so it works correctly on unsorted input too.

---

**16. [Self-check]** In one sentence each: when do you reach for `ArrayList`, `HashSet`, `HashMap`, `ArrayDeque`?

*Answer:* `ArrayList` — ordered, indexed, growable access without needing fast membership checks. `HashSet` — uniqueness/membership in average O(1), order doesn't matter. `HashMap` — associating extra information with each unique key in average O(1). `ArrayDeque` — LIFO/FIFO access from one or both ends, no indexed access into the middle needed.

---

**17. [Synthesis]** Name a problem from this week that combines two different patterns, and explain the combination.

*Answer:* 3Sum combines Day 5's "fix one element, reduce to a smaller lookup" thinking with Day 6's sorted-array Two Sum via two pointers — for each fixed element, the rest of the problem is solved exactly as Two Sum II was. (Subarray Sum Equals K, from Day 5, is a second valid answer — it combines Two Sum's complement-lookup shape with the new prefix-sum technique.)

---

## Daily Deliverable Check

- [ ] Is Subsequence and Valid Palindrome II (+ the four extra reps) solved and pushed to `dsa-java/two-pointers/`
- [ ] Weekly Industry Awareness Ritual complete
- [ ] Weekly scorecard reviewed — 24 total DSA problems this week


---

# Week 1 Consolidation

## What you actually built this week

Seven days ago, "programming" started at zero — no assumed prior knowledge of variables, loops, or what a compiler even does. Here's the full chain, in the order it was built, each link resting entirely on the ones before it:

**The language itself (Day 1):** how source code becomes a running program (compiler → bytecode → JVM), variables and primitive types, every operator including the short-circuit and integer-division gotchas, control flow, all three loop forms, and methods with correct pass-by-value reasoning for primitives.

**Structure (Day 2):** arrays and *why* indexed access is O(1) — not as a memorized label, but from the base-address-plus-offset mechanism directly. Strings as objects, the full `==`/`.equals()` picture (the single most-referenced gotcha across the entire week), and the complete OOP foundation: classes, constructors, encapsulation, inheritance, interfaces — plus the pass-by-value story finished for objects and arrays.

**Efficiency, formalized (Day 3):** Big-O as a real, precise vocabulary — not intuition, but a tool for analyzing unfamiliar code on sight — and amortized analysis, which explained *why* `ArrayList.add()` genuinely is O(1) on average despite individual calls varying.

**The rest of the toolkit (Day 4):** `HashSet`, `HashMap` (including the hash-function mechanism and the `equals()`/`hashCode()` contract that makes it all actually work), and `ArrayDeque` for stacks and queues.

**Pattern mastery, first family (Day 5):** HashMap/HashSet formalized across **12 problems**, plus the four OOP pillars named and connected to code you'd already written, plus abstract classes.

**Pattern mastery, second family (Days 6-7):** Two Pointers across **12 problems** and three genuinely distinct variants — opposite-ends, from-the-back, same-direction fast/slow — plus SOLID principles grounding *how* to use the OOP tools well, not just that they exist.

## The number that matters most

Your original plan scoped **12 DSA problems** for the week. This series delivered **24** — every required problem at full depth, plus a matched extra problem for every single pattern, specifically because one example of a pattern teaches recognition of *that exact problem*, while three or four teach recognition of the *pattern itself*. That gap — recognizing a shape you've seen once versus a shape you've internalized — is exactly what separates "I've done this problem before" from "I know what to do here" in an actual interview.

## A short diagnostic before moving to Week 2

If any of these would require flipping back through this series rather than answering immediately, that's worth closing first — not as a failure, but because Week 2 is about to build directly on top of every one of these being reflexive:

- Explain why `arr[i]` is O(1), from the memory-address mechanism, not just the label.
- State the `equals()`/`hashCode()` contract and what breaks if it's violated.
- Given an unfamiliar problem statement, name which HashMap/HashSet sub-pattern (or Two Pointers variant) it calls for, before writing any code.
- Explain amortized analysis well enough to justify *why* `ArrayList.add()` is O(1) on average, not just recite the conclusion.
- Distinguish an interface from an abstract class by when you'd reach for each, not just their syntax.

## What Week 2 assumes, and where it's headed

Week 2 finishes Two Pointers (the remaining problems from your plan's original 16-problem scope) and opens **Sliding Window** — a technique that will be introduced, in this same series' style, as a direct evolution of Two Pointers: instead of two independent pointers, a *window* with two edges that expand and contract together over a single pass. Everything about *why* Two Pointers works — sortedness or another structural guarantee licensing a pointer's movement — carries forward directly into *why* a sliding window's edges can move the way they do. Nothing in Week 2 will ask you to already know sliding window; it will absolutely assume Big-O, HashMap/HashSet, and Two Pointers are no longer things you're reasoning through from scratch.

---

**Series complete.** [← Back to Curriculum Map](./00_Curriculum_Map.md) · [Week 1 Interview Question Bank](./Week1_Interview_Questions.md)
