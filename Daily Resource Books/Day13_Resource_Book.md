# Day 13 — Two Pointers Capstone (Hard Tier), and Pattern Consolidation

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 12 Resource Book](Day12_Resource_Book.md)
**Next ▶:** [Day 14 Resource Book](Day14_Resource_Book.md)
**Companion to:** Day 13 of `Week_02_Revised.md`

---

## Overlap Notice

Trapping Rain Water (LC 42) is genuinely new — no prior coverage. Full depth below. This is also the day the 16-problem Two Pointers ladder closes, so today's Theory Block is a full consolidation rather than new problems.

---

## Recap

Yesterday, Sort Colors showed that not every two-pointer problem fits the two categories the plan's own wording emphasizes ("opposite-ends" and "fast-slow") — it needed a third. Today closes the loop on that observation directly: the consolidation below classifies all 16 ladder problems using **every** variant actually used, not just the two most common ones, because forcing a clean binary split would misclassify several of them.

---

## Learning Objectives

1. Prove — not assert — that Trapping Rain Water's two-pointer solution correctly computes trapped water without ever fully scanning the region between the two pointers.
2. Independently classify a two-pointer problem's variant, and justify the classification from the problem's structure, without checking hints.
3. Recite the full 16-problem ladder's variant breakdown from memory, and explain *why* each classification holds — not just *what* it is.

---

## Concept Dependency Map

```
Day 9: opposite-ends over an implicit range (LC 633)
Day 12: opposite-ends, provable greedy (Container recap); 3-pointer partition (Sort Colors)
        │
        └──▶ Today: Trapping Rain Water (LC 42) — opposite-ends,
                     hardest correctness proof in the ladder

Weeks 1–2, all 16 required Two Pointers problems
        │
        └──▶ Today: full pattern consolidation — every variant, named and justified
```

---

## Part 1 — Trapping Rain Water (LC 42)

**Statement:** Given `n` non-negative integers representing an elevation map where each bar has width 1, compute how much water it can trap after raining.

This is the ladder's Hard-tier capstone for a reason — its two-pointer solution is correct, but the *proof* of why is the most subtle argument in the entire 16-problem set. Take the proof section seriously; "it passes the test cases" is not the same as being able to defend it.

### Approach 1 — Brute force

```java
public static int trapBruteForce(int[] height) {
    int n = height.length;
    int water = 0;
    for (int i = 0; i < n; i++) {
        int leftMax = 0;
        for (int j = 0; j <= i; j++) {
            leftMax = Math.max(leftMax, height[j]);
        }
        int rightMax = 0;
        for (int j = i; j < n; j++) {
            rightMax = Math.max(rightMax, height[j]);
        }
        water += Math.min(leftMax, rightMax) - height[i];
    }
    return water;
}
```

For each position `i`, scan left to find the tallest bar in `[0, i]`, scan right to find the tallest bar in `[i, n-1]`, and water at `i` is `min(leftMax, rightMax) - height[i]`. Correct — **O(n²) time, O(1) space** — but redoes the same scanning work from scratch at every position.

### Approach 2 — Prefix/suffix max arrays

```java
public static int trapPrefixSuffix(int[] height) {
    int n = height.length;
    if (n == 0) return 0;

    int[] leftMax = new int[n];
    leftMax[0] = height[0];
    for (int i = 1; i < n; i++) {
        leftMax[i] = Math.max(leftMax[i - 1], height[i]);
    }

    int[] rightMax = new int[n];
    rightMax[n - 1] = height[n - 1];
    for (int i = n - 2; i >= 0; i--) {
        rightMax[i] = Math.max(rightMax[i + 1], height[i]);
    }

    int water = 0;
    for (int i = 0; i < n; i++) {
        water += Math.min(leftMax[i], rightMax[i]) - height[i];
    }
    return water;
}
```

Precompute `leftMax[i]` = max of `height[0..i]` and `rightMax[i]` = max of `height[i..n-1]` in two linear passes, then a third pass sums `min(leftMax[i], rightMax[i]) - height[i]` across all `i`. **O(n) time, O(n) space.** This eliminates the repeated rescanning — each position's water is now an O(1) lookup — at the cost of two full auxiliary arrays.

### Approach 3 — Optimized: two pointers

```java
public static int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int leftMax = 0, rightMax = 0;
    int water = 0;

    while (left < right) {
        if (height[left] < height[right]) {
            if (height[left] >= leftMax) leftMax = height[left];
            else water += leftMax - height[left];
            left++;
        } else {
            if (height[right] >= rightMax) rightMax = height[right];
            else water += rightMax - height[right];
            right--;
        }
    }
    return water;
}
```

**O(n) time, O(1) space.** The optimization insight: you don't actually need the *entire* precomputed `leftMax[]`/`rightMax[]` arrays — at the moment you're ready to finalize the water at some position, you only need to know, for the **smaller** of the two sides, that its running max is a safe lower bound for the true max on the *other*, not-yet-fully-scanned side. That's a single running variable per side, not a full array.

### The Proof — Why This Is Safe Without Fully Scanning the Middle

This deserves real care, because the worry is legitimate: when the algorithm computes `water += leftMax - height[left]`, it is using `leftMax` — the **exact** true maximum of everything from `0` to `left` (fully scanned, no guessing) — as if it were the smaller of the two bounding maxes. But the *true* maximum of everything to the right of `left` hasn't been fully scanned yet; some of it (the region strictly between the two pointers) is still unknown. Why is it still safe to finalize the water amount at `left` right now?

Two facts, combined:

- `height[right]` — a value already directly observed — is itself one of the values inside the range "everything to the right of `left`" (since `right > left` whenever this branch runs). So the **true** maximum of that whole range is *at least* `height[right]`.
- The branch only runs when `height[left] < height[right]`. Combined with the fact above: the true right-side maximum is at least `height[right]`, which is strictly greater than `height[left]`.

That alone shows the true right-side max exceeds `height[left]` — but the water formula needs more: it needs the true right-side max to be **at least `leftMax`** (not just at least `height[left]`), so that `min(leftMax, true right max) = leftMax` and the formula `leftMax - height[left]` is exactly correct rather than an overestimate.

This holds by an inductive argument on the algorithm's own steps: whenever `leftMax` is updated to a new value (because `height[left] >= leftMax` at that moment), the branch condition guarantees `height[right] > height[left] = ` the new `leftMax`, at that same moment — meaning a bar taller than the new `leftMax` was directly observed on the right side, at a position that remains "to the right of `left`" for every step afterward (the right pointer only ever moves further right-of-`left`-ward relatively, i.e., `left` never overtakes where that tall bar was). That observed taller bar acts as a permanent witness: from the moment `leftMax` is set, there is a provably taller bar somewhere in the not-yet-scanned or already-scanned right region, for as long as `left` hasn't passed it — which, by construction, it never does before that region is accounted for. So `leftMax` never outruns the true right-side maximum, for the entire time it's being used to compute water. Symmetric argument for the `rightMax` branch.

**🔑 Key Takeaway:** the two-pointer version isn't "a shortcut that happens to work" — it's the prefix/suffix-array approach with the observation that you never need the *full* array, only a witness value that's provably good enough at the exact moment you use it. This is a genuinely different flavor of "optimize away an array" than, say, `ArrayDeque` replacing `ArrayList` — there, the optimization was about *how* memory is laid out; here, it's about realizing a weaker, cheaper piece of information (a single running max, not a fully resolved array of them) is provably sufficient.

### Worked Trace

`heights = [4, 2, 0, 3, 2, 5]`.

| position | height | true leftMax | true rightMax | water = min-height |
|---|---|---|---|---|
| 0 | 4 | — (edge) | — | 0 |
| 1 | 2 | 4 | 5 | min(4,5)-2 = 2 |
| 2 | 0 | 4 | 5 | min(4,5)-0 = 4 |
| 3 | 3 | 4 | 5 | min(4,5)-3 = 1 |
| 4 | 2 | 4 | 5 | min(4,5)-2 = 2 |
| 5 | 5 | — (edge) | — | 0 |

Total = `2+4+1+2 = 9`. Running the two-pointer algorithm on the same array produces the same total by construction (each position's water is computed identically, just in a different order and without materializing the full `leftMax`/`rightMax` arrays) — worth re-deriving by hand once to see the two approaches land on the same number for a reason, not by coincidence.

### Complexity

Time O(n) — each pointer moves at most `n` times total across the whole run, O(1) work per step. Space O(1) — a genuine improvement over the O(n) auxiliary arrays of the intermediate approach.

### Edge Cases

- Fewer than 3 bars: no water can ever be trapped (need a bar on both sides to hold anything) — the loop's `left < right` bound combined with edges always contributing 0 handles this without special-casing.
- Monotonically increasing or decreasing heights: one side's max always dominates trivially, water is 0 everywhere — a good sanity check to hand-trace.
- A single very tall spike in the middle: correctly acts as a wall that lets both sides fill up to whatever the *shorter* remaining side allows, up to the spike.

**💡 Interview Insight:** if you only remember one thing to say out loud before coding this, say the actual insight — "water at any position is bounded by the *shorter* of its two-sided maxes, and I can track a running max from whichever side is currently smaller without needing the other side's exact value yet." That sentence, stated before writing a line of code, is worth more than a syntactically perfect solution offered without it.

**🔗 Extension (not required for today's deliverable):** a monotonic-stack approach also solves this in O(n) time, O(n) worst-case space, processing left-to-right only and computing water level-by-level as it pops shorter bars off the stack. Worth knowing this alternative exists if asked "is there another way," but the two-pointer approach above is the one to lead with — it strictly dominates on space.

---

## Part 2 — Two Pointers, Reviewed: Opposite-Ends vs. Fast-Slow, Side by Side

The plan's own framing for today's reflection asks specifically about opposite-ends vs. fast-slow. Worth being upfront about something real: those are genuinely the two *most common* variants across the ladder, but they are not the *only* two. Forcing every one of the 16 problems into a binary split would misclassify several of them — Merge Sorted Array isn't really either, and Sort Colors from yesterday clearly isn't. The honest, more valuable version of this exercise classifies against **every** variant actually used.

**Do the reflection yourself first.** Before reading the table below, write down — no looking back at hints, no looking at this table — which variant each of the 16 problems used and *why that specific problem's structure called for it*. This is the highest-leverage 20 minutes in the whole week; the value is in producing the classification yourself, not in reading one. Use the table only to check your own answers afterward.

### The Full Variant Family (7, Not 2)

| Variant | Defining shape |
|---|---|
| **Opposite-ends** | Two pointers start at both ends, converge based on a comparison; sortedness (or a monotonic relationship) guarantees the direction of movement is correct. |
| **From-the-back** | Fills a result from its last index backward, typically merging two already-sorted sources. |
| **Same-direction (fast/slow)** | Both pointers move only rightward; `fast` scans, `slow` marks a write position — used to compact/filter in place. |
| **One-forward-pointer-each** | Two pointers, each scanning its own sequence left-to-right; one advances unconditionally, the other only on a match. |
| **Opposite-ends, one allowed skip** | Opposite-ends convergence with a single permitted exception to the matching rule before failing. |
| **Opposite-ends, greedy pairing** | Opposite-ends convergence where the decision at each step is "pair them, or strand the extreme one" rather than a pure converge-toward-target rule. |
| **Three-way partition (Dutch National Flag)** | Three pointers/regions, not two; doesn't reduce cleanly to either opposite-ends or fast-slow. |

### Answer Key — Check Your Own Reflection Against This

| # | LC | Problem | Variant | Why |
|---|---|---|---|---|
| 1 | 125 | Valid Palindrome | Opposite-Ends | Comparing characters from both ends inward is the direct definition of a palindrome check. |
| 2 | 344 | Reverse String | Opposite-Ends | Swapping from both ends inward is the most direct way to reverse in place. |
| 3 | 88 | Merge Sorted Array | From-the-Back | Merging into the *end* of the array (which has trailing free space) avoids overwriting not-yet-merged elements — impossible if filling from the front. |
| 4 | 392 | Is Subsequence | One-Forward-Pointer-Each | One pointer must examine every character of the longer string; the other only advances on a match — not a converging pair. |
| 5 | 680 | Valid Palindrome II | Opposite-Ends, One Allowed Skip | Same convergence as Valid Palindrome, plus exactly one permitted mismatch before declaring failure. |
| 6 | 26 | Remove Duplicates from Sorted Array | Same-Direction (Fast-Slow) | Compacting an already-sorted array in place needs a write-position marker trailing a scanner, not convergence. |
| 7 | 283 | Move Zeroes | Same-Direction (Fast-Slow) | Same shape as #6, swap instead of overwrite since the displaced values must be preserved elsewhere in the array. |
| 8 | 977 | Squares of a Sorted Array | Opposite-Ends (From-the-Back) | Convergence decides *which* value is next (compare `\|left\|` vs `\|right\|`), from-the-back decides *where* it's written — genuinely both at once. |
| 9 | 167 | Two Sum II | Opposite-Ends | Sortedness guarantees moving `left` up or `right` down moves the sum predictably — the canonical opposite-ends shape. |
| 10 | 15 | 3Sum | Opposite-Ends *(plus an outer fixed-element sweep)* | The core two-pointer mechanism searching for a pair is unchanged from Two Sum II; the outer loop is separate scaffolding. |
| 11 | 16 | 3Sum Closest | Opposite-Ends *(plus outer sweep)* | Identical mechanism to 3Sum; only the objective (closest vs. exact) differs. |
| 12 | 18 | 4Sum | Opposite-Ends *(plus two outer sweeps)* | Same core mechanism, one additional level of outer fixing. |
| 13 | 881 | Boats to Save Most People | Opposite-Ends, Greedy Pairing | Converges from both ends, but the decision at each step is "pair, or strand the heaviest" — a genuinely different rule from pure sum-comparison convergence. |
| 14 | 11 | Container With Most Water | Opposite-Ends | Converges based on which side is the current bottleneck — provably safe to always move the shorter line inward. |
| 15 | 75 | Sort Colors | Three-Way Partition (Dutch National Flag) | Doesn't reduce to either category — three regions and three pointers, one of which (`mid`) scans while the other two (`low`, `high`) mark boundaries. |
| 16 | 42 | Trapping Rain Water | Opposite-Ends | Converges from both ends, tracking a running max on whichever side is currently smaller. |

**Tally:** Opposite-Ends (including its two hybrids and the outer-sweep variants): 11. Same-Direction (Fast-Slow): 2. From-the-Back (standalone): 1. One-Forward-Pointer-Each: 1. Opposite-Ends-with-Skip: 1. Three-Way Partition: 1. Total: 16. ✓

**🔑 Key Takeaway:** if your own reflection matched most of this without having seen it, the pattern recognition genuinely landed. If several entries surprised you — especially Merge Sorted Array or Sort Colors, the two that don't fit the binary framing — that's worth sitting with specifically: the mistake to watch for going forward isn't misclassifying a problem, it's *forcing* an unfamiliar problem into a category it doesn't actually belong to just because it superficially resembles something you've seen. Noticing "this doesn't quite fit either box" is itself a valid, valuable conclusion — precisely what yesterday's Sort Colors and today's Boats-vs-Assign-Cookies distinction (Day 11) were both building toward.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** no new problem today — this slot is for making sure all 16 Two Pointers solutions are actually pushed to `dsa-java/two-pointers/`, cleanly organized.

Practical guidance: use consistent, self-explanatory file naming (e.g., `LC125_ValidPalindrome.java`, not `Solution1.java`) so the folder is browsable at a glance months from now during interview-prep review — the entire point of banking these solutions is being able to find and re-solve any of them quickly under time pressure later, and a folder of ambiguously-named files defeats that. Consider a short `README.md` inside `dsa-java/two-pointers/` listing all 16 (plus the extra-practice reps from both weeks) with a one-line variant tag each — effectively a personal, portable version of today's answer-key table.

**Definition of done:** `dsa-java/two-pointers/` contains all 16 required problems (plus extras), browsable at a glance.

## Career Block Guide (1 hr)

- **LinkedIn engagement (20 min):** 3–5 substantive comments, same standard as prior days.
- **Networking:** reach out to 1 peer or former coworker, staying warm on the relationship — same low-pressure, no-ask framing as Day 11's version of this task.

---

## Day 13 — Interview Questions

**Q1. Walk through why Trapping Rain Water's two-pointer solution is correct without fully scanning the region between the pointers.** Whenever `leftMax` is updated, the branch condition guarantees a strictly taller bar was observed on the right side at that same moment, at a position `left` never overtakes afterward — that observed bar is a permanent witness that the true right-side maximum stays at least as large as `leftMax` for as long as it's being used, making `leftMax - height[left]` exactly correct, not an overestimate.

**Q2. Name three approaches to Trapping Rain Water, in order of optimization.** Brute force (per-position rescanning, O(n²)/O(1)), prefix/suffix max arrays (precompute both directions, O(n)/O(n)), two pointers (running max per side, O(n)/O(1)).

**Q3. Is Merge Sorted Array opposite-ends or fast-slow?** Neither, cleanly — it's From-the-Back: filling the result starting from the last index specifically to avoid overwriting not-yet-merged elements, a different structural need from either named category.

**Q4. Why doesn't Sort Colors reduce to opposite-ends or fast-slow?** It uses three pointers marking three regions rather than two pointers converging or scanning together — `mid` behaves like a scanner, `low`/`high` behave like boundary markers, but no single existing category captures that combination.

**Q5. What's the actual difference between 3Sum's two-pointer mechanism and Boats to Save Most People's?** Both converge from opposite ends, but 3Sum's pointer movement is purely comparison-driven toward an exact or closest sum; Boats adds a genuine branch — pair the two current pointers, or strand the extreme one and move only one pointer — which is a different decision rule layered on the same convergence shape.

**Q6. If a new, unfamiliar two-pointer problem doesn't obviously match any named variant, what's the right move?** Don't force it into an existing box — identify the actual invariant the problem needs (what must stay true as the pointers move, and what determines each pointer's movement), and name it as its own thing if it genuinely is one, the way Sort Colors' three-way partition was named rather than mislabeled.

---

## Daily Deliverable Check

- [ ] Trapping Rain Water solved — Two Pointers ladder complete at 16 problems (5 Week 1 + 11 Week 2).
- [ ] Written opposite-ends vs. fast-slow reflection complete — attempted independently *before* checking the answer key above.
- [ ] `dsa-java/two-pointers/` fully organized, all 16 (plus extras) present with clear, browsable naming.

---

## What Tomorrow Assumes You Already Know Cold

Day 14 assumes the entire Two Pointers family — all 7 variants, not just the two most-cited ones — is fully reflexive, since tomorrow explicitly introduces Sliding Window as *a specific evolution* of the same-direction (fast-slow) variant specifically: two pointers moving only rightward, except now both are meaningful boundaries of a range rather than one scanner and one write-marker, and a running aggregate gets tracked over that range. If that lineage doesn't feel obvious by tomorrow, it's worth re-reading today's fast-slow entries (#6, #7) once more before starting.
