# Day 14 (Sunday) — Consolidation, and Sliding Window Begins

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 13 Resource Book](Day13_Resource_Book.md)
**Next ▶:** Day 15 Resource Book (Week 3)
**Companion to:** Day 14 of `Week_02_Revised.md`

---

## Overlap Notice

Both of today's problems (LC 121, LC 643) are genuinely new — no prior coverage, and neither appears in `Week_03_Revised.md`.

**One deliberate judgment call, flagged explicitly:** no extra Sliding Window practice is added today, even though the pattern is brand new and the prompt generally asks for extra reps on patterns with real interview weight. Reason: Sliding Window is only *opening* today — `Week_03_Revised.md` continues it immediately with 12 more required problems across Days 15–20 (fixed-size, variable-size, at-most-K-distinct, and monotonic-deque variants all represented), closing the pattern at 14 total. Adding extra practice now risks either duplicating a problem Week 3 was always going to require, or spending effort on reps the pattern's own continuation will provide anyway within days. This is the inverse of Days 8–9's situation (a *closing* pattern gets reinforcement) — an *opening* pattern gets full depth on its required problems and nothing more, deliberately deferring extra reps until the pattern's fuller shape is visible.

---

## Self-Check (15 min)

Before starting anything new: pick one Two Pointers problem from earlier this week, cold, no hints, and solve it. If it doesn't come immediately, that's useful information — revisit that specific sub-variant in `00_Curriculum_Map.md`'s terminology section before moving on, rather than pushing ahead on a shaky foundation the week's capstone was supposed to confirm.

---

## Learning Objectives

1. State the Sliding Window mechanism precisely, and explain why it's O(n) despite superficially looking like nested loops.
2. Explain Sliding Window's lineage from the same-direction (fast-slow) Two Pointers variant — what's reused, what's genuinely new.
3. Distinguish fixed-size from variable-size windows, and correctly identify which of today's two problems is which (the answer isn't quite what the plan's own labels might suggest — worth reading carefully).

---

## Concept Dependency Map

```
Week 1: arrays/strings ✅, HashMap/HashSet ✅
Day 13 (yesterday): Same-Direction (Fast-Slow) Two Pointers, fully consolidated
        │
        └──▶ Today: Sliding Window
                 ├─ LC 121 — single-pass min-tracking (an implicit, degenerate window)
                 └─ LC 643 — true fixed-size window, the canonical shape
                          │
                          └──▶ Week 3: variable-size window (still ahead)
```

---

## Sliding Window — Concept Card, in Full Depth

### Prerequisites (confirmed)

- Arrays, HashMap/HashSet (Week 1) — used by later Sliding Window problems, not today's two, but worth having solid regardless since Week 3 leans on both immediately.
- Same-Direction (Fast-Slow) Two Pointers (Week 1 Day 6–7, reinforced through Day 13) — Sliding Window's direct conceptual ancestor.

### What It Is, Mechanistically

**Definition:** a sliding window is a **contiguous** range of a sequence, tracked by two indices — call them `left` and `right` — that only ever move **rightward**, never backward.

As `right` advances, new elements enter the window and get folded into some running aggregate (a sum, a count, a frequency map — whatever the problem needs). When some condition requires it, `left` advances too, removing elements from that same running aggregate as they leave the window.

**Lineage from fast-slow, precisely:** Week 1's fast-slow variant already established "two pointers, both moving only rightward" as a category — `slow` marking a write position, `fast` scanning ahead. Sliding Window keeps that same rightward-only movement but changes what the two pointers *mean*: instead of one scanner and one write-marker, **both** `left` and `right` are meaningful boundaries of an active range, and the algorithm maintains a running aggregate *over that whole range*, not just a single write position. Same directional discipline, a genuinely richer piece of state being tracked.

### Why O(n), Despite Looking Like a Loop Inside a Loop

Code implementing a variable-size window often looks like:

```java
for (right = 0; right < n; right++) {
    // expand: fold nums[right] into the aggregate
    while (/* window invalid */) {
        // shrink: remove nums[left] from the aggregate
        left++;
    }
}
```

This *looks* like it could be O(n²) — a `for` loop with a `while` loop inside it. It isn't, and the reason is worth being able to state precisely: `left`'s total movement, summed across **every** iteration of the outer loop combined, is bounded by `n` — it never resets backward, so it can advance at most `n` times over the algorithm's *entire* lifetime, not `n` times *per* outer iteration. Each of the `n` elements is added to the aggregate at most once (when `right` reaches it) and removed at most once (when `left` passes it) — total work across the whole run is O(n) + O(n) = O(n), not O(n) × O(n). This is a distinct flavor of reasoning from `ArrayList`'s amortized-doubling argument (Week 1, Day 3) — there, most operations are cheap and a rare one is expensive, averaged out; here, every single pointer step is already cheap, and the bound comes from capping *total* movement, not from averaging occasional spikes.

### Fixed-Size vs. Variable-Size

- **Fixed-size:** the window's length is a given constant `k`. `left` is really just `right - k + 1`, moving in lockstep with `right` — one element enters, one leaves, every single step. LC 643 below is the clean, canonical example.
- **Variable-size:** the window grows and shrinks organically based on the data and a stated constraint (e.g., "longest window with at most K distinct characters," "smallest window whose sum is at least a target"). `right` expands greedily; `left` shrinks only when the constraint is violated, by however much is needed to restore it. **Neither of today's two problems is a true variable-size window in this classic sense** — that's genuinely still ahead, arriving with Week 3's Max Consecutive Ones III and Longest Substring Without Repeating Characters. Today is deliberately narrower: one fixed-size window (LC 643) and one specialized single-pass pattern that's *related to* but not identical to either category (LC 121, discussed below).

### Trade-offs vs. Brute Force

Brute force re-derives each window's aggregate from scratch — O(n·k) for fixed windows, up to O(n²) or worse for variable ones, since re-summing or re-counting a window's contents every time it moves repeats work on the elements shared between consecutive windows. Sliding window's entire value is reusing that shared work: incrementally updating the aggregate as the window's edges move, rather than recomputing it.

### Common Mistakes

- **⚠️ Recomputing the aggregate from scratch on every slide instead of updating it incrementally.** Defeats the entire point — silently degrades back to the brute-force complexity while still *looking* like a sliding-window solution.
- **⚠️ Forgetting to shrink the window when a variable-size problem's constraint is violated** — produces a window that's simply wrong, not just slow. (Not a risk for today's two fixed/implicit cases, but worth flagging now since it's the single most common bug once Week 3's variable windows begin.)
- **⚠️ Off-by-one on the fixed-window slide index** — the loop that slides a fixed-size window should start at index `k` (the first index *after* the initial window), not `k-1` or `0`; starting in the wrong place either reprocesses part of the initial window or skips the first legitimate slide.

---

## Problem 1: Best Time to Buy and Sell Stock (LC 121)

**Statement:** `prices[i]` is the stock's price on day `i`. Choose a single day to buy and a later single day to sell, maximizing profit. Return `0` if no profit is possible.

### Approach 1 — Brute force

```java
public static int maxProfitBruteForce(int[] prices) {
    int maxProfit = 0;
    for (int buyDay = 0; buyDay < prices.length; buyDay++) {
        for (int sellDay = buyDay + 1; sellDay < prices.length; sellDay++) {
            maxProfit = Math.max(maxProfit, prices[sellDay] - prices[buyDay]);
        }
    }
    return maxProfit;
}
```

Check every pair `(buyDay, sellDay)` with `buyDay < sellDay`, track the best `prices[sellDay] - prices[buyDay]`. Time O(n²), Space O(1).

### Approach 2 — Optimized: single-pass min-tracking

```java
public static int maxProfit(int[] prices) {
    if (prices.length == 0) return 0;
    int minPrice = prices[0];
    int maxProfit = 0;
    for (int i = 1; i < prices.length; i++) {
        maxProfit = Math.max(maxProfit, prices[i] - minPrice);
        minPrice = Math.min(minPrice, prices[i]);
    }
    return maxProfit;
}
```

At each day, ask "if I sold today, using the best buy day seen *so far*, what's my profit?" — then update the best-buy-day tracker for future days to use.

**How this relates to Sliding Window, precisely (and where the label is a little loose):** the plan calls this "Sliding Window (fixed-start tracking)." It's worth being precise about what that means and doesn't mean. There's no explicit window boundary or running aggregate-over-a-range being maintained here the way LC 643 has — this is really a specialized one-pass pattern in its own right. The useful connection is this: you can think of it as an *implicit* window `[minPriceIndex, i]`, where the left edge doesn't creep forward by one each step the way a textbook window would — it can jump forward arbitrarily far, straight to wherever the new minimum was found, the moment a new minimum appears. What it genuinely shares with Sliding Window is the core efficiency idea: never re-scan from scratch: carry forward one running piece of state (the minimum) instead of re-deriving it. The *shape* of the technique (implicit, jumping left edge, no real aggregate) is different enough from LC 643 that calling both of them "the same pattern" would overstate it — better to say they're both instances of the broader "carry state forward instead of rescanning" idea that Sliding Window formalizes more fully.

**Worked trace:** `prices = [7,1,5,3,6,4]`. `minPrice=7, maxProfit=0`.

| i | prices[i] | profit = prices[i]-minPrice | maxProfit | minPrice after |
|---|---|---|---|---|
| 1 | 1 | 1-7=-6 | 0 | 1 |
| 2 | 5 | 5-1=4 | 4 | 1 |
| 3 | 3 | 3-1=2 | 4 | 1 |
| 4 | 6 | 6-1=5 | 5 | 1 |
| 5 | 4 | 4-1=3 | 5 | 1 |

Final `maxProfit = 5` (buy at 1, sell at 6). Matches manual verification — no other buy/sell pair in this array beats a profit of 5.

**Complexity:** Time O(n), Space O(1) — versus the brute force's O(n²)/O(1).

**Edge cases:**
- Prices strictly decreasing throughout: `maxProfit` never updates from `0`, correctly reporting "no profit possible" rather than a negative number.
- Single price: loop never runs (starts at index 1), `maxProfit` stays `0` — correct, since one day alone can't produce both a buy and a sell.
- All identical prices: every profit computed is `0`, `maxProfit` stays `0` — correct.

**⚠️ Common Mistake:** computing `minPrice` for *today* before computing today's profit, rather than after. Doing so would let a day "buy and sell on itself" in a way that miscounts — though because that specific case always yields exactly `0` profit either way, it doesn't actually break this particular problem's correctness; it's still worth doing in the right order (profit first, then update the minimum) as a matter of precise habit, since the wrong order *does* cause real bugs in adjacent variants of this problem (e.g., versions requiring at least one full day of holding).

---

## Problem 2: Maximum Average Subarray I (LC 643)

**Statement:** Given an integer array and integer `k`, find the contiguous subarray of length **exactly** `k` with the maximum average. Return that average.

### Approach 1 — Brute force

```java
public static double findMaxAverageBruteForce(int[] nums, int k) {
    int maxSum = Integer.MIN_VALUE;
    for (int start = 0; start <= nums.length - k; start++) {
        int sum = 0;
        for (int i = start; i < start + k; i++) {
            sum += nums[i];
        }
        maxSum = Math.max(maxSum, sum);
    }
    return (double) maxSum / k;
}
```

For each of the `n-k+1` valid starting positions, sum its `k` elements directly. Time O(n·k), Space O(1).

### Approach 2 — Optimized: true fixed-size sliding window

```java
public static double findMaxAverage(int[] nums, int k) {
    long windowSum = 0;
    for (int i = 0; i < k; i++) {
        windowSum += nums[i];
    }
    long maxSum = windowSum;
    for (int i = k; i < nums.length; i++) {
        windowSum += nums[i] - nums[i - k];   // one enters, one leaves
        maxSum = Math.max(maxSum, windowSum);
    }
    return (double) maxSum / k;
}
```

This is the canonical fixed-size window: build the first window's sum directly, then for every subsequent position, update the running sum by adding exactly the one new element entering on the right and subtracting exactly the one element leaving on the left — never re-summing the whole window. (`long` for the running sum is defensive good practice here too, same reasoning as Days 10–11: with enough elements at the constraint boundary, an `int` sum could in principle overflow.)

**Why this is the optimization, precisely:** consecutive windows of size `k` share `k-1` elements. Brute force re-adds all `k-1` shared elements every single slide, redoing work that didn't need to change. The incremental update touches exactly the 2 elements that actually changed (one leaving, one entering) — turning O(k) work per window into O(1), and the total from O(n·k) into O(n).

**Worked trace:** `nums = [1, 12, -5, -6, 50, 3]`, `k = 4`. Initial window sum (indices 0–3): `1+12-5-6 = 2`. `maxSum = 2`.

| i (entering) | nums[i] | leaving = nums[i-k] | windowSum update | windowSum | maxSum |
|---|---|---|---|---|---|
| 4 | 50 | nums[0]=1 | 2+50-1 | 51 | 51 |
| 5 | 3 | nums[1]=12 | 51+3-12 | 42 | 51 |

Final `maxSum = 51`, average = `51/4 = 12.75`.

**Complexity:** Time O(n) — exactly linear, not merely amortized: every step does exactly one addition and one subtraction, with no occasional expensive rebuild the way `ArrayList`'s doubling has. Space O(1) — only the running sum is kept, never the window's individual elements.

**Edge cases:**
- `k == n`: exactly one valid window (the whole array) — the second loop never executes, `maxSum` is just the total sum, correct without special-casing.
- Negative numbers throughout: works identically — sum-tracking doesn't care about sign, only the final average comparison does.
- `k == 1`: every single element is its own "window"; the algorithm correctly reduces to "find the maximum element."

**⚠️ Common Mistake:** starting the sliding loop at index `k-1` or `0` instead of `k` — the first `k` indices already constitute the *initial* window (summed directly, before the loop), so sliding must begin from the very next index, `k`, or the window either double-counts part of itself or slides one step early.

---

## Project Block Guide — (folded into Career Block today per the plan; no separate new coding task beyond the two problems above)

Push both solutions to `dsa-java/sliding-window/`. Start the folder now with the same naming discipline established for `two-pointers/` yesterday — it's about to grow quickly over the next six days.

## Career Block Guide (1 hr)

- **Weekly Industry Awareness Ritual (20 min):** clear the TLDR Newsletter backlog, read one engineering blog post. Practical framing: skim for anything that changes how you'd answer a "what have you been reading/following lately" interview question — that's the actual return on this ritual, not just staying generally informed.
- **Weekly Scorecard:** see the full consolidation below — the plan's own official count and the actual count (including all extra practice) now diverge meaningfully, and both numbers are worth understanding, not just the official one.

---

## Day 14 — Interview Questions

**Q1. Why is a sliding window O(n) even though the code often has a loop nested inside another loop?** The inner pointer's total movement across the *entire* run, not per outer iteration, is bounded by n — each element is added to the window's aggregate at most once and removed at most once overall, so total work is O(n), not O(n²).

**Q2. What's the actual mechanistic relationship between Sliding Window and the fast-slow Two Pointers variant?** Both move two pointers only rightward, never backward. Fast-slow uses one pointer as a scanner and the other purely as a write-position marker; Sliding Window promotes both pointers to meaningful range boundaries and maintains a running aggregate over everything between them.

**Q3. Is LC 121 (Best Time to Buy/Sell Stock) a true sliding window?** Not in the strict sense — there's no explicit window boundary or aggregate-over-a-range being tracked, just a running minimum and an implicit left edge that can jump forward arbitrarily. It shares Sliding Window's core "carry state forward, never rescan" idea without matching its textbook shape.

**Q4. What distinguishes a fixed-size from a variable-size window?** Fixed-size: length is a given constant, both edges move in lockstep, one element enters and one leaves every step (LC 643). Variable-size: the window grows and shrinks based on whether a stated constraint currently holds — not covered by either of today's problems, arriving in Week 3.

**Q5. In LC 643, why is updating the sum incrementally (`windowSum += nums[i] - nums[i-k]`) better than resumming each window?** Consecutive windows share k-1 elements; incremental update touches only the two elements that actually changed, turning O(k) per-window work into O(1) and the overall complexity from O(n·k) to O(n).

**Q6. Why wasn't any extra Sliding Window practice added this week, when every other pattern got extra reps?** The pattern is only opening today — Week 3 immediately continues it with 12 more required problems spanning every major variant, so extra practice now risks duplicating problems Week 3 was always going to require rather than adding genuinely new value.

---

## Daily Deliverable Check

- [ ] Best Time to Buy/Sell Stock and Maximum Average Subarray I solved, pushed to `dsa-java/sliding-window/`.
- [ ] Self-check (cold Two Pointers problem) attempted before starting new material.
- [ ] Weekly ritual and scorecard complete (see below).

---

# Week 2 Consolidation

## What Actually Got Built

- 7 daily Resource Books (Day 8 → Day 14), each following the same structure established in Week 1.
- Two full theory arcs completed: the Two Pointers pattern family closed at 16 problems (opposite-ends, from-the-back, fast-slow, one-forward-pointer-each, opposite-ends-with-a-skip, greedy pairing, and three-way partition — 7 named variants total, not the 2 the plan's own reflection prompt names); Sliding Window opened with its first 2 problems and a full mechanism-level concept card.
- Five new Java theory topics, each built strictly on confirmed prior material: Recursion (Day 8), the JVM Memory Model (Day 9), Primitives & the Integer Cache (Day 10), String Internals (Day 11), Collections Internals (Day 12).
- `Week2_Interview_Questions.md` consolidating every Q&A pair from all 7 days.
- `00_Curriculum_Map.md` extended (not replaced) with Week 2's dependency map, problem inventory, terminology, and a new overlap check against Week 3.

## Planned vs. Actual — Two Numbers, Both Worth Knowing

The plan's own Day 14 scorecard states **25** total DSA problems solved at this checkpoint, matching the *original* plan's count at the same point. That number is real, but it's a **required-only** count — it doesn't include any extra practice added by this resource-book process, in either week.

| | Official (required only) | Actual (required + all extra practice) |
|---|---|---|
| Stack (preview) | — *(not part of the plan's own pattern tally)* | 4 |
| HashMap/HashSet | 7 | 12 |
| Two Pointers | 16 (5 Week 1 + 11 Week 2) | 19 (16 + 3 genuine extras: LC 27, 80, 633) |
| Sliding Window | 2 | 2 *(no extras added this week, by design)* |
| **Total** | **25** | **37** |

Both numbers are correct simultaneously — they're just answering different questions. 25 is "how many problems did the plan itself require." 37 is "how many distinct problems have actually been solved and banked," which is the number that matters for interview readiness, since extra practice is what turns "solved this exact problem once" into "recognizes this pattern on sight." The gap (12 problems) is entirely accounted for by extra practice: 5 in HashMap/HashSet, 7 in Two Pointers (3 recapped-into-required-status is not double-counted here, since those are the *same* problems, not additional ones — see `00_Curriculum_Map.md`'s inventory table for the exact accounting).

## Short Diagnostic — Check Before Moving On

- [ ] Can you name all 7 Two Pointers sub-variants and, for an unfamiliar problem, identify which one applies without checking hints?
- [ ] Can you explain why `StackOverflowError` is an `Error`, and why naive recursive Fibonacci is exponential in time but only linear in space?
- [ ] Can you explain pass-by-value for objects precisely enough that "Java is pass-by-reference for objects" sounds obviously wrong, not just memorized-wrong?
- [ ] Can you predict, without running it, whether `Integer.valueOf(x) == Integer.valueOf(y)` is `true` or `false` for any given `x, y`?
- [ ] Can you explain why `String` concatenation in a loop is O(n²) *even accounting for* the compiler's automatic per-statement `StringBuilder` rewrite?
- [ ] Can you justify, from the memory-layout mechanism (not just the Big-O label), why `ArrayDeque` beats both `ArrayList` and `LinkedList` for stack/queue use?
- [ ] Can you state why a sliding window is O(n) despite the nested-loop *appearance*, in terms of total pointer movement rather than per-iteration cost?

If any of these aren't immediate, that's exactly what this checklist is for — better to find it now than mid-interview.

## What Week 3 Assumes

Per `Week_03_Revised.md`: Day 15 opens with **Max Consecutive Ones III** and **Longest Substring Without Repeating Characters** — the first *true* variable-size windows in this series, where `left` shrinks only in response to a violated constraint rather than moving in lockstep with `right`. This assumes today's fixed-size mechanism (LC 643) and the general "carry an aggregate forward, never rescan" principle are both solid; the *new* piece Week 3 introduces is the shrink-on-violation logic itself, which today deliberately didn't cover.

Day 15's theory block is **HashMap Internals** — hashing into buckets, collision handling, treeification — a direct deepening of Week 1 Day 4–5's HashMap/HashSet coverage. It assumes the `.equals()`/`hashCode()` contract (already established in Week 1) is solid enough to build on, not re-teach.

No Two Pointers content carries forward as "still open" — the pattern is fully closed as of today, confirmed via Day 13's consolidation.
