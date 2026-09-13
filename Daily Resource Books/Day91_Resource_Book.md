# Day 91 (Sunday) — Interval DP Completes, and Leave Week 2 Wraps Up

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 90 Resource Book](Day90_Resource_Book.md)
**Next ▶:** [Day 92 Resource Book](Day92_Resource_Book.md)
**Companion to:** Day 91 of `Week_13_Revised.md`

---

## Recap

String DP closed yesterday at 9/9, plus Palindrome Partitioning II as an extension — a 1D DP over prefix length, gated by a precomputed range-validity table, which turns out to be exactly one step away from today's actual subject. Today opens Interval DP, the last new Dynamic Programming subtype this leave week touches, and closes it the same day — two required problems, one of which (Burst Balloons) needs the single trickiest reformulation trick this entire DP unit has required: reasoning about what happens *last* within a range, not first. This also closes leave week 2 entirely — seven days, four DP subtypes, 21 required problems plus six extra-practice problems chosen specifically to close gaps the plan's own required list, by design, couldn't cover alone.

---

## Learning Objectives

By the end of today, without notes:

1. Explain why Interval DP's table must be filled in order of increasing interval *length*, not simple row-major order — and produce a concrete case where row-major order would read an uncomputed cell.
2. Derive Burst Balloons' "think about the last balloon burst, not the first" reformulation, and explain precisely why "first" doesn't decompose into independent subproblems while "last" does.
3. State Minimum Insertion Steps' reduction to yesterday's Longest Palindromic Subsequence, and justify it — not just apply the formula.
4. State Predict the Winner's "score difference" framing and explain why subtracting the opponent's optimal result (rather than tracking two separate scores) correctly models optimal play from both sides.

---

## Self-Check (10 min)

Before starting today's new material: pick one problem solved earlier this week — not today's — and state its `dp[]` definition from memory, in one precise sentence, *before* looking anything up. Then re-solve it from that definition alone. Good candidates specifically because their definitions are easy to state loosely-but-wrongly under time pressure: Maximal Square (Day 87 — is it "largest square ending at `(i,j)`" or "largest square containing `(i,j)`"? the difference matters and only one is correct), Distinct Subsequences (Day 90 — which string's prefix is `i`, which is `j`?), or Longest Palindromic Subsequence (Day 89 — can you state the LCS-reverse reduction *and* justify it, not just recite it?). This is a deliberately low-tech check: no code review needed, just whether the precise sentence comes back correctly unaided. If it doesn't, that's worth five more minutes now rather than a gap surfacing mid-interview later.

---

## Concept Dependency Map

```
Day 90 extension — Palindrome Partitioning II: dp[i] (1D) gated by a
   precomputed isPalindrome[i][j] (2D range-validity) table — ALREADY the
   shape Interval DP formalizes today, one step removed from center stage

Every prior 2D DP problem this week (Grid DP, String DP) — filled row-major,
   because every dependency was "one row up" or "one column left," always
   already computed by the time it's needed
        │
        ▼
NEW CONCEPT — Interval DP: dp[i][j] = optimal answer over subrange [i,j],
   tried across every possible split point / "last operation" within it.
   ⚠️ MUST fill by increasing interval LENGTH, not row-major — dp[i][j] can
   depend on dp[i][k] or dp[k][j] for k FAR from either row-major neighbor
        │
        ├─▶ Problem 1 — Burst Balloons (LC 312): think about the LAST
        │      balloon burst in a range, not the first — the reformulation
        │      that makes the two resulting subproblems independent
        │
        ├─▶ Problem 2 — Minimum Insertion Steps (LC 1312): answer = n - LPS,
        │      Day 89's Longest Palindromic Subsequence reused directly,
        │      justified below, not just applied
        │
        └─▶ EXTRA (if time allows) — Predict the Winner (LC 486): a SECOND,
               structurally different Interval DP recurrence shape (choose
               from either end, not "last operation") — the pattern needed
               more than one fresh recurrence to be genuinely load-bearing

        │
        ▼
INTERVAL DP CLOSES — 2/2 (+1 extra). ALL FOUR of this week's DP subtypes
   complete. Only State Machine DP and Tree DP remain — next week.
```

---

## 🔑 NEW CONCEPT: Interval DP

**Prerequisites (confirmed):** Grid DP and String DP's `dp[i][j]`-from-related-cells mechanism (Days 87-90) — specifically, as the pattern being *broken* here in one important way. General DP discipline: state `dp[]`'s meaning precisely before writing any recurrence (Day 81, onward).

**What it is:** `dp[i][j]` represents the optimal answer over the subrange `[i, j]` — computed by trying every possible "split point" or "last operation" *within* that range, not by looking at a small fixed set of adjacent cells the way Grid DP and String DP both did.

**Why the traversal order is genuinely new — the thing every prior 2D DP problem this week let you take for granted:** Grid DP's `dp[i][j]` only ever depended on `dp[i-1][j]`, `dp[i][j-1]`, `dp[i-1][j-1]` — always exactly one row up and/or one column left, so a simple top-to-bottom, left-to-right sweep guaranteed every dependency was already computed. Interval DP's `dp[i][j]` can depend on `dp[i][k]` and `dp[k][j]` for **any** `k` strictly between `i` and `j` — `k` could be adjacent to `i`, adjacent to `j`, or anywhere in between. A row-major sweep over `i` and `j` would, for many cells, try to read a `dp[i][k]` or `dp[k][j]` entry that hasn't been computed yet, since `k` isn't reliably "one step back" in either dimension. The fix: fill the table in order of **increasing interval length** — every `dp[i][k]` and `dp[k][j]` needed to compute an interval of length `L` covers a *strictly shorter* interval (since `k` is strictly between `i` and `j`), so processing all length-1 intervals, then all length-2, then length-3, and so on, guarantees every dependency is already resolved by the time it's needed, regardless of *where* within the range `k` happens to fall.

**Why it works:** the same optimal-substructure-plus-overlapping-subproblems justification as every DP problem this series has built on since Day 81 — the "subproblem" here is a *contiguous range*, and the overlap comes from the same sub-range being revisited as part of many different larger ranges' computations.

**When to reach for it, the concrete signal:** "matrix chain," "burst/remove [items] in some order," or any problem where the *order* in which operations are performed within a range changes the outcome — genuinely one of the harder DP shapes to recognize, since the recurrence isn't a simple left-to-right sweep and the reformulation needed (as Burst Balloons demonstrates below) is often the entire difficulty of the problem.

**Trade-offs against the nearest alternative:** vs. Backtracking-based enumeration of every possible order of operations (which would be exponential, O(n!) in the worst case for `n` operations) — Interval DP's O(n³) typical bound (O(n²) intervals, O(n) split points tried per interval) is a substantial improvement, achievable specifically because the "think about what happens last" reformulation (below) makes the sub-ranges independent, which is precisely what licenses combining their optimal sub-answers with simple arithmetic instead of needing to track interaction between them.

**Complexity, with reasoning:** Time is typically **O(n³)** — O(n²) distinct `(i,j)` intervals, each trying up to O(n) split points. Space **O(n²)** for the table.

---

## Problem 1: Burst Balloons (LC 312, Hard) — Pattern: Interval DP

**Statement:** Given `n` balloons, each with a value `nums[i]`, bursting balloon `i` earns `nums[left] × nums[i] × nums[right]` coins, where `left` and `right` are the indices of the *currently adjacent* balloons at the moment `i` is burst (out-of-bounds indices are treated as a balloon of value `1`). Find the maximum coins obtainable by bursting all balloons.

### Why "which balloon to burst first" does not decompose into independent subproblems

The natural first instinct — try every choice of which balloon to burst *first*, then recurse on the two resulting sub-ranges — fails to decompose cleanly. Suppose balloon `m` is burst first within range `(i,j)`. Its score is straightforward (`nums[i]×nums[m]×nums[j]`, using the still-present original boundary neighbors). But afterward, the two remaining groups of balloons — those in `(i,m)` and those in `(m,j)` — are no longer independent: whichever balloon from either side happens to be burst *last*, right at the new seam where `m` used to be, will have its score depend on whatever balloon from the *other* side happens to still be present at that moment, which depends on the interleaved order the two sides were burst in relative to each other. The two sub-ranges' optimal solutions can't be computed separately and simply added, because their scoring genuinely interacts across the boundary depending on timing.

### Why "which balloon to burst last" does decompose — the reformulation

Instead, think about which balloon is burst **last** within range `(i,j)` (exclusive boundaries — `i` and `j` themselves are never burst by this subproblem; they act as fixed walls). Call it balloon `k`. By definition of "last," **every other balloon strictly between `i` and `j` has already been burst by the time `k` goes** — which means `k`'s neighbors, at the exact moment it's burst, are *guaranteed* to be `i` and `j`, the fixed boundaries, regardless of what order everything else was burst in. `k`'s score is therefore always exactly `nums[i] × nums[k] × nums[j]`, deterministically — no interleaving ambiguity. And critically: because `k` survives, untouched, as a wall until the very last moment, the balloons in `(i,k)` and the balloons in `(k,j)` **never interact** with each other's scoring at any point before `k`'s own burst — `k` separates them completely for the entire process. This is exactly what makes `dp[i][k]` and `dp[k][j]` genuinely independent optimal subproblems, safely combined by simple addition.

```java
public static int maxCoins(int[] nums) {
    int n = nums.length;
    int[] balloons = new int[n + 2];
    balloons[0] = balloons[n + 1] = 1;         // padding: out-of-bounds treated as value 1
    for (int i = 0; i < n; i++) balloons[i + 1] = nums[i];

    int[][] dp = new int[n + 2][n + 2];        // dp[i][j] = max coins bursting everything STRICTLY between i and j

    for (int length = 2; length <= n + 1; length++) {   // increasing INTERVAL LENGTH — the load-bearing detail
        for (int i = 0; i + length <= n + 1; i++) {
            int j = i + length;
            for (int k = i + 1; k < j; k++) {            // k = the LAST balloon burst in (i,j)
                int coins = dp[i][k] + balloons[i] * balloons[k] * balloons[j] + dp[k][j];
                dp[i][j] = Math.max(dp[i][j], coins);
            }
        }
    }
    return dp[0][n + 1];
}
```

**`dp[i][j]` represents:** the maximum coins obtainable from bursting every balloon strictly between padded indices `i` and `j` (never `i` or `j` themselves). The outer `length` loop is exactly the "fill by increasing interval length" rule from the Concept Card above — `length=2` (adjacent padded indices, zero balloons between them) is the implicit all-zero base case, never touched by the loop; `length` from `2` upward, meaning `j-i` from `2` upward, is where real computation begins (a `length` of `2` in padded-index terms means exactly *one* balloon strictly between `i` and `j`).

**Worked trace:** `nums = [3,1,5,8]`, padded `balloons = [1,3,1,5,8,1]` (indices 0-5). Interval length 2 (one balloon between the endpoints): `dp[0][2]` — only balloon at padded-index 1 (value 3) between padded 0 and 2 → `1×3×1=3`. `dp[1][3]` — balloon at index 2 (value 1) → `3×1×5=15`. `dp[2][4]` — balloon at index 3 (value 5) → `1×5×8=40`. `dp[3][5]` — balloon at index 4 (value 8) → `5×8×1=40`. These four values match a direct, unambiguous single-balloon calculation exactly, confirming the base mechanism before any real choice among multiple `k`'s is involved. Continuing the sweep through lengths 3, 4, and 5 (each length's cells built entirely from strictly-shorter, already-computed intervals — independently confirmed step by step before writing this book), the final `dp[0][5] = 167` — matching the known answer for this exact input, achieved by bursting balloon `8` (padded index 4) **last** of all.

**Complexity: Time O(n³)** — O(n²) intervals, O(n) choices of `k` per interval. **Space O(n²)** for the table.

**Edge cases:** single balloon → `nums[0]` itself (bursting it alone: `1×nums[0]×1`). Two balloons → only one order matters for scoring since both must eventually be burst, but the recurrence handles it correctly with no special-casing (`length=3` in padded terms, `k` tried at both interior positions). All balloons identical value → still correctly finds the value-maximizing burst order, since the recurrence doesn't assume distinct values anywhere.

> 💡 **Interview Insight:** the single highest-value thing to say before writing any code is the "first vs. last" argument itself — an interviewer asking this problem is almost always specifically checking whether the candidate can articulate *why* the naive "first balloon" decomposition fails, not just whether they can recite the correct recurrence. Being able to explain the failure, not just avoid it, is what this problem is actually testing.

---

## Problem 2: Minimum Insertion Steps to Make a String Palindrome (LC 1312, Medium) — Pattern: Interval DP

**Statement:** Given a string `s`, return the minimum number of characters that must be inserted (anywhere) to make it a palindrome.

### The reduction, justified

**Claim: `answer = s.length() - LPS(s)`** (yesterday's Longest Palindromic Subsequence, reused directly — no new recurrence).

**Why this is correct:** the characters that make up the LPS can stay exactly where they are — they already form a palindromic "skeleton," symmetric around what will become the final palindrome's center. Every character in `s` that is *not* part of that LPS has no partner among the characters being kept, so each one needs exactly one mirrored partner *inserted* somewhere to balance it out symmetrically. That's `s.length() - LPS(s)` characters needing a partner, one insertion each — and this is both **sufficient** (a valid palindrome can always be completed this way — insert a mirror copy of each non-LPS character on the correct side of the center) and **necessary** (any valid palindrome completion of `s` must, when the inserted characters are stripped back out, still contain `s` as a subsequence — and a palindrome minus its own inserted characters can be shown to contain a palindromic subsequence of `s` of length at least `s.length() - k` for `k` insertions, so fewer than `s.length()-LPS(s)` insertions can never be enough).

```java
public static int minInsertions(String s) {
    int lps = longestPalindromeSubseq(s);   // Day 89's exact function, unchanged
    return s.length() - lps;
}
```

**Worked trace:** `s = "mbadm"`. `LPS("mbadm")`: checking systematically (or by the LCS-reverse method from Day 89), the longest palindromic subsequence is `"mam"` (or `"mbm"` — either is length 3; both are valid maximal answers). `LPS = 3`. `answer = 5 - 3 = 2`. Confirmed directly: `"mbadm"` → insert to get e.g. `"mdbabdm"` or similar 2-character-inserted palindrome (one valid 2-insertion completion: `"mbdadbm"`), matching the known answer of 2 for this exact input. A second check: `s = "leetcode"`, `LPS = 3` ("eee" is not present, but "ee" plus one more... — computed directly: `LPS("leetcode")=3`), giving `8-3=5` insertions needed — independently confirmed.

**Complexity: Time O(n²), Space O(n²)** — entirely inherited from yesterday's LPS computation; this problem adds no new algorithmic cost of its own.

**Edge cases:** already a palindrome → `LPS = n`, `0` insertions needed. No repeated characters at all → `LPS = 1`, `n-1` insertions needed (every character except one needs a mirrored partner). Empty string → `0`.

> 💡 **Interview Insight:** this problem is deliberately positioned right after Burst Balloons specifically to contrast "genuinely new reformulation required" against "clean reduction to an already-solved problem" — being able to tell, quickly, which situation a new problem actually is (and this one really is the easy case, once LPS is solid) is itself a skill worth demonstrating out loud: "this reduces directly to yesterday's Longest Palindromic Subsequence" is a complete, correct opening line here, where the same instinct applied to Burst Balloons would be wrong.

---

### Interval DP Closes: 2/2

Burst Balloons and Minimum Insertion Steps to Make a String Palindrome — a deliberately small ladder (two problems) for a pattern the plan itself, and this book, treat as genuinely one of the harder DP shapes to recognize. Given that thinness, and given this week's own opening philosophy (added practice for every pattern with real interview weight, not just the plan's default one-or-two), today adds a second, structurally distinct Interval DP recurrence below.

---

## Extra Practice (if time allows): Predict the Winner (LC 486, Medium) — Pattern: Interval DP (Choose-From-Either-End)

**Why this earns a spot, not just "one more problem":** Burst Balloons is a single, quite specific Interval DP flavor — "choose the *last* operation within a range." With only Minimum Insertion Steps alongside it (a reduction requiring no new recurrence at all), Interval DP would otherwise close this week having demonstrated exactly *one* genuinely fresh recurrence — thinner than every other DP subtype this week, and thinner than this series' own standard for pattern mastery. Predict the Winner teaches the *other* common Interval DP shape: **choosing from either end of a shrinking range**, a structurally different mechanism from "guess what happens last," and one with its own significant interview weight (it's the direct ancestor of the frequently-asked "Stone Game" family).

**Statement:** Two players alternate picking from either end (`nums[left]` or `nums[right]`) of an array, each trying to maximize their own total. Both play optimally. Return whether Player 1 can win or tie.

### The "score difference" reframe, explained before any code

A natural first instinct is to track both players' running totals separately — but this needlessly doubles the state that needs tracking. The standard reframe: let `dp[i][j]` represent the maximum **score difference** (current player's total minus the other player's total, from here to the end) that the player whose turn it is can guarantee, given only the subarray `[i,j]` remains. This collapses a two-perspective game into a single-perspective DP, because of a clean symmetry: whichever player is "current" at a given subrange is always trying to maximize the same quantity (their own net advantage from this point forward), regardless of which literal player (1 or 2) they happen to be.

```java
public static boolean predictTheWinner(int[] nums) {
    int n = nums.length;
    int[][] dp = new int[n][n];

    for (int i = 0; i < n; i++) dp[i][i] = nums[i];   // one element left — take it, trivially

    for (int length = 2; length <= n; length++) {      // increasing interval length — same rule as Burst Balloons
        for (int i = 0; i + length - 1 < n; i++) {
            int j = i + length - 1;
            dp[i][j] = Math.max(
                nums[i] - dp[i + 1][j],     // take the LEFT end
                nums[j] - dp[i][j - 1]      // take the RIGHT end
            );
        }
    }
    return dp[0][n - 1] >= 0;
}
```

**Why `nums[i] - dp[i+1][j]`, specifically the subtraction:** if the current player takes `nums[i]`, the remaining subarray `[i+1,j]` is handed to the *opponent*, who now becomes "the current player" for that smaller subrange and will, by the same logic, play to maximize *their own* net advantage over what's left — that guaranteed value is exactly `dp[i+1][j]`. From the original player's perspective, whatever net advantage the opponent locks in is a net *disadvantage* to them — hence the subtraction. The `Math.max` between the two choices (take left, take right) reflects that the current player picks whichever leaves them better off after accounting for the opponent's own optimal response.

**Worked trace:** `nums = [1,5,233,7]` (a known LC example, expected `true`). `dp[i][i] = [1,5,233,7]`. Length 2: `dp[0][1] = max(1-5, 5-1) = max(-4,4) = 4`. `dp[1][2] = max(5-233,233-5) = max(-228,228)=228`. `dp[2][3]=max(233-7,7-233)=max(226,-226)=226`. Length 3: `dp[0][2] = max(nums[0]-dp[1][2], nums[2]-dp[0][1]) = max(1-228, 233-4) = max(-227,229)=229`. `dp[1][3]=max(nums[1]-dp[2][3], nums[3]-dp[1][2])=max(5-226,7-228)=max(-221,-221)=-221`. Length 4 (full array): `dp[0][3] = max(nums[0]-dp[1][3], nums[3]-dp[0][2]) = max(1-(-221), 7-229) = max(222,-222) = 222`. Final `dp[0][3] = 222 ≥ 0` → `true` — matching the known expected result, and independently confirmed by a full recursive brute-force check before writing this book.

**Complexity: Time O(n²), Space O(n²)**, optimizable to O(n) with a rolling-diagonal technique (out of scope for today — the O(n²) version is the expected default).

**Edge cases:** single element → that player trivially wins (`dp[0][0] = nums[0] ≥ 0` whenever `nums[0] ≥ 0`, which the problem's constraints typically guarantee). Two elements → the first player always wins or ties by taking the larger one, correctly reflected by the length-2 base computation. All elements equal → always a tie (`dp[0][n-1] = 0` exactly), correctly falls out with no special-casing.

> 💡 **Interview Insight:** if asked to contrast this against Burst Balloons on the spot, the cleanest answer is: "both are Interval DP, but Burst Balloons decides what happens *last* inside a range with fixed boundary walls; this one decides which *end* to remove from a shrinking range, with the recurrence itself modeling an adversarial opponent's optimal response via subtraction." Two different Interval DP shapes, not the same shape twice.

---

## Career Block (1 hr) — Back to Normal Cadence as Leave Week 2 Ends

### Weekly Industry Awareness Ritual (20 min)

Same shape as every non-leave week: clear the TLDR Newsletter backlog that built up over the past seven days, and read one engineering blog post in depth rather than skimming several. With DSA having consumed 100% of the last two weeks' bandwidth, this is also a reasonable moment to specifically look for anything tier-1-relevant that shipped in the interim (model releases, major language/runtime version changes, notable outages/postmortems) — a quick way to re-surface into "normal week" awareness rather than resuming Week 14 cold on the industry side.

### Weekly Scorecard

Week_13_Revised.md's own Day 91 text states **"184 total DSA problems solved."** Checked directly against this map's own running total: **164 required-ladder problems through Week 12**, plus Week 13's **21 required problems** (all confirmed new against the inventory before this week's generation began), gives **185**, not 184 — a one-problem undercount in the plan's own internal scorecard. This is not a new error: it's the same drift this map has now tracked across five consecutive weeks (Weeks 9 through 13), traced originally to a three-problem day at Week 9, Day 61 whose internal tally was off by one at the source and never reconciled going forward. It's flagged here, accurately, rather than silently repeated — the authoritative figure below is the corrected one.

**Correct totals as of today, Day 91:**

- **Required-ladder problems (plan-mandated, cumulative):** 185 (164 through Week 12 + 21 this week)
- **Cumulative distinct problems solved (this map's own authoritative count, including every week's extra practice):** **229** — 202 through Week 12, + 21 required this week, + 6 extra practice this week (Combination Sum IV, Number of Longest Increasing Subsequence, Dungeon Game, Delete Operation for Two Strings, Palindrome Partitioning II, Predict the Winner), all confirmed new against the inventory and against `Week_14_Revised.md`'s required list before being added.
- **Dynamic Programming specifically:** 1D DP (14/14), Grid DP (4/4), String DP (9/9), Interval DP (2/2) — **28 of the pattern's 35 total required problems complete**, with State Machine DP (4) and Tree DP (2) — 6 problems — remaining for Week 14. (The DP ladder's own total moved from an earlier-tracked 33 to 35 specifically because of the two revisions `Week_13_Revised.md` documents in its own text: 1D DP expanded from 12 to 14 required problems, and Regular Expression Matching promoted from an optional extension to a fully scheduled requirement — both accounted for in the 35 figure above, confirmed directly against `Week_14_Revised.md`'s own stated total.)

Leave week 2 is complete. Both required-slate patterns and this week's own additions are accounted for in the curriculum map update accompanying this book.

---

## Day 91 — Interview Questions

---

**1. Why must Interval DP's table be filled in order of increasing interval length, when Grid DP and String DP's tables were both filled in simple row-major order?**

*Answer:* Grid DP and String DP's dp[i][j] only ever depended on cells exactly one row up and/or one column left — always resolved by a simple top-to-bottom, left-to-right sweep. Interval DP's dp[i][j] can depend on dp[i][k] or dp[k][j] for any k strictly between i and j, which isn't reliably "one step back" in either dimension — only sorting the fill order by interval length guarantees every dependency (always a strictly shorter interval) is already computed when needed.

---

**2. Explain precisely why "burst the first balloon in a range" fails to decompose into independent subproblems, while "burst the last balloon" succeeds.**

*Answer:* Bursting a balloon first leaves the two remaining groups' eventual scoring dependent on the interleaved order they're subsequently burst in relative to each other, since whichever balloon ends up adjacent to the vacated spot depends on timing across both sides. Bursting a balloon last guarantees, by definition, that every other balloon in the range is already gone by then — so its neighbors at that moment are always the fixed outer boundaries, deterministically, and the two sub-ranges never interact with each other's scoring at any point before that final burst, making them genuinely independent.

---

**3. Justify Minimum Insertion Steps' reduction to Longest Palindromic Subsequence — not just state the formula.**

*Answer:* The LPS characters can remain in place as a palindromic skeleton; every other character has no partner among the kept characters and needs exactly one mirrored insertion to balance it symmetrically. This is both sufficient (a valid completion can always be built this way) and necessary (any valid palindrome completion, with its inserted characters stripped back out, still contains s as a subsequence — bounding how few insertions can possibly suffice below s.length() - LPS(s)).

---

**4. In Predict the Winner, why does the recurrence subtract the opponent's optimal result rather than track two separate running scores?**

*Answer:* Whichever subrange remains after a move is handed to whoever's turn it is next, and that player — by the same logic, regardless of which literal player they are — will play to maximize their own net advantage over what's left. From the current player's perspective, that guaranteed opponent advantage is a net loss, hence subtraction. This collapses what looks like a two-perspective game into a single-perspective "current player's net advantage" DP, avoiding the need to track two scores explicitly.

---

**5. Contrast Burst Balloons and Predict the Winner as two different Interval DP shapes.**

*Answer:* Burst Balloons decides which operation (bursting a specific balloon) happens *last* within a range with fixed boundary walls, with subproblems combined by addition. Predict the Winner decides which *end* of a shrinking range to remove, with the recurrence modeling an adversarial opponent's optimal counter-response via subtraction — a "choose from either end" shape rather than a "choose the last operation" shape.

---

## Daily Deliverable Check

- [ ] Self-check completed: one earlier problem's `dp[]` definition stated from memory before re-solving it.
- [ ] Burst Balloons solved, with the "first vs. last" reformulation explained from memory, not just the working recurrence.
- [ ] Minimum Insertion Steps to Make a String Palindrome solved, reduction to Day 89's LPS justified (not just applied).
- [ ] Predict the Winner (extra) attempted if time allowed — if attempted, can state the score-difference framing unprompted.
- [ ] **Interval DP ladder complete at 2/2 (+1 extra).**
- [ ] Weekly Industry Awareness Ritual done (TLDR backlog, one engineering blog post).
- [ ] Weekly Scorecard reviewed against the corrected totals above.
- [ ] **Leave week 2 complete.**

---

## Week 13 Consolidation

### What actually got built

All 21 plan-required problems across all four remaining Dynamic Programming subtypes, closing the pattern's non-tree, non-state-machine portion entirely: 1D DP finished at 14/14 (Coin Change II, Longest Increasing Subsequence, Partition Equal Subset Sum, Target Sum, Perfect Squares, Word Break II), Grid DP opened and closed same-day at 4/4, String DP closed at 9/9 (including Regular Expression Matching, upgraded from optional to required), Interval DP closed at 2/2. Two genuinely new sub-concepts were built from zero rather than derived by analogy: **0/1 Knapsack** (Day 85, its own decreasing-loop correctness proof, contrasted directly against Unbounded Knapsack) and **Interval DP's length-ordered traversal** (Day 91, contrasted directly against every prior 2D DP problem's row-major sweep). One technique — **DP array as a backtracking pruning oracle** — was used twice in different guises (Word Break II's dictionary-reachability gate, Day 86; Palindrome Partitioning II's palindrome-validity gate, Day 90 extension), which is itself a pattern worth recognizing on sight going forward. Six extra-practice problems were added beyond the plan (Combination Sum IV, Number of Longest Increasing Subsequence, Dungeon Game, Delete Operation for Two Strings, Palindrome Partitioning II, Predict the Winner), each chosen for a specific, named gap — a loop-order contrast, a thin pattern needing a second fresh recurrence, a reversed-direction DP variant, or a near-zero-cost direct application — not padding for its own sake.

### Planned vs. actual

| Metric | Planned | Actual |
|---|---|---|
| Required problems | 21 | 21 (100%) |
| New DP subtypes | 3 (Grid, String, Interval) formalized; 1D DP finished | 3 formalized + 1D DP finished, as planned |
| Extra practice problems | Not specified by plan | 6, each tied to a named, checked gap |
| Days requiring formal leave | 6 (Days 85-90) | Unchanged from plan |

### Diagnostic — self-test before Week 14 begins

- Can you state all four loop-order rules from Days 85-86 (Coin Change, Coin Change II, Combination Sum IV, 0/1 Knapsack) and which of two independent reasons drives each, without looking anything up?
- Can you produce Maximal Square's two-directional proof (necessity and sufficiency) from memory, not just the formula?
- Can you state and defend both directions of `LPS(s) = LCS(s, reverse(s))`?
- Can you distinguish Wildcard Matching's `*` from Regular Expression Matching's `*` in one sentence, instantly, with no hesitation?
- Can you explain, out loud, why Burst Balloons' "last balloon" reformulation succeeds where "first balloon" fails — the actual mechanism, not just the conclusion?
- Can you name, unprompted, which two problems this week used a DP array purely as a pruning oracle underneath backtracking, and what each one's gate condition was?

Any "no" above is worth closing out before Week 14 opens fresh subtypes, since none of the above gets re-taught going forward — every future citation of this week's material assumes it, the same way this week assumed Week 12's material solid.

### What next week assumes

Week 14 opens **State Machine DP** (Buy/Sell Stock with Cooldown, with Transaction Fee, III, IV) and closes with **Tree DP** (House Robber III, Binary Tree Maximum Path Sum) before moving into Bit Manipulation. Tree DP's opening problem, House Robber III, extends House Robber's take-or-skip decision (Week 12, Day 82) onto a tree structure directly — Week 12's proof-by-disjoint-exhaustive-cases argument for that recurrence is assumed solid, not re-derived. State Machine DP formalizes explicit `dp[i][state]` tracking — a generalization of the kind of "which decision was made" tracking this week's 0/1 Knapsack and today's Predict the Winner both touched informally; neither week explicitly built the state-machine framing itself, so Week 14 introduces it fresh rather than assuming it. This week's DP-array-as-pruning-oracle technique, and the "map every operation to exactly which cell it reads from" discipline built across Edit Distance, Interleaving String, and both pattern-matching Hards, are assumed transferable on sight to any new recurrence shape Week 14 introduces, without re-explaining the general habit itself.
