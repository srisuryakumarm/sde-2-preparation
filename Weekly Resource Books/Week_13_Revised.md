# Week 13 (Revised): Leave Week 2 — 1D DP Completes, Grid DP, String DP, and Interval DP All Complete

**This is leave week 2, confirmed: Days 85–91.** Same logic as leave week 1 — no new theory, no project tasks, pure DSA immersion at full-time intensity. This week alone closes four DP subtypes: the rest of 1D DP (including both "nice to have" additions), all of Grid DP, all of String DP (including the Regular Expression Matching addition), and all of Interval DP. That's 21 problems in 7 days — aggressive, but each day builds directly on the one before it, and this is exactly the kind of concentrated stretch a leave week is for.

If you're requesting formal leave from work: Day 91 is a Sunday-equivalent in this schedule, so you'd likely only need to request Days 85–90 — six working days — same logic as leave week 1.

---

## Day 85 — 1D DP: Combinations and Subsequences

### DSA Block (4-5 hrs at leave-week intensity)
- Problem 9: Coin Change II — LeetCode #518 — Medium — Pattern: DP Combinations (Unbounded Knapsack)
  - Hint: unlike Coin Change I, order doesn't matter here — you're counting combinations, not permutations. The outer loop must iterate over coins, with amounts as the inner loop, or you'll end up counting the same combination multiple times in different orders.
  - Complexity: Time O(n×amount) | Space O(amount)
- Problem 10: Longest Increasing Subsequence — LeetCode #300 — Medium — Pattern: 1D DP / Binary Search
  - Hint: the O(n²) version is `dp[i] = max(dp[j]) + 1` for every `j < i` where `nums[j] < nums[i]`. The O(n log n) version maintains a `tails` array (the smallest tail value for an increasing subsequence of each length) and binary searches it — worth learning both, since the second is a common follow-up question after you solve the first.
  - Complexity: Time O(n log n) | Space O(n)
- Problem 11: Partition Equal Subset Sum — LeetCode #416 — Medium — Pattern: 0/1 Knapsack DP
  - Hint: the target is `totalSum / 2` — the question becomes "can a subset sum to exactly this target," a classic 0/1 knapsack shape (each number can be used at most once).
  - Complexity: Time O(n×sum) | Space O(sum)

### Daily Deliverable
- [ ] Coin Change II, Longest Increasing Subsequence, and Partition Equal Subset Sum solved, pushed to `dsa-java/dynamic-programming/`.

---

## Day 86 — 1D DP Completes

### DSA Block (4-5 hrs at leave-week intensity)
- Problem 12: Target Sum — LeetCode #494 — Medium — Pattern: 0/1 Knapsack DP
  - Hint: if `P` is the subset you assign a `+` sign and `N` is the subset you assign a `-` sign, then `P - N = target` and `P + N = totalSum`. Solving those two equations together gives `P = (target + totalSum) / 2` — now you're just counting subsets that sum to `P`, the exact same shape as yesterday's Partition Equal Subset Sum.
  - Complexity: Time O(n×sum) | Space O(sum)
- Problem 13: Perfect Squares — LeetCode #279 — Medium — Pattern: 1D DP (Unbounded Knapsack) **(new)**
  - Hint: `dp[i] = min(dp[i], dp[i - square] + 1)` for every perfect square ≤ `i` — structurally identical to Coin Change, with the "coins" being 1, 4, 9, 16...
  - Complexity: Time O(n×√n) | Space O(n)
- Problem 14: Word Break II — LeetCode #140 — Hard — Pattern: 1D DP + Backtracking **(new)**
  - Hint: Word Break (Day 84) told you *whether* a segmentation exists. This asks you to actually produce every valid segmentation — use the same `dp[i]` boolean array to prune impossible branches early, then backtrack to build the actual sentences only where `dp[i]` is true.
  - Complexity: Time O(n²) plus output size | Space O(n²)

**This closes 1D DP: 14 problems (Climbing Stairs, Min Cost Climbing Stairs, House Robber, House Robber II, Decode Ways, Maximum Product Subarray, Word Break, Coin Change, Coin Change II, Longest Increasing Subsequence, Partition Equal Subset Sum, Target Sum, Perfect Squares, Word Break II) — up from 12 in the original plan.**

### Daily Deliverable
- [ ] Target Sum, Perfect Squares, and Word Break II solved — 1D DP ladder complete at 14 problems.

---

## Day 87 — Grid (2D) DP Completes in a Single Day

### DSA Block (4-5 hrs at leave-week intensity)

**Concept Card — Grid (2D) DP**
- What: the same subproblem-reuse idea as 1D DP, extended across two dimensions — `dp[i][j]` depends on some combination of `dp[i-1][j]`, `dp[i][j-1]`, and `dp[i-1][j-1]`.
- Why: naturally fits "moving through a grid," and bridges directly into String DP next, where the two dimensions become positions in two different strings instead of grid coordinates.

- Problem 1: Unique Paths — LeetCode #62 — Medium — Pattern: 2D DP
  - Hint: `dp[i][j] = dp[i-1][j] + dp[i][j-1]` — you can only arrive at any cell from directly above or directly to the left. The entire first row and first column are all 1s, since there's only one way to reach any of those cells.
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 2: Unique Paths II — LeetCode #63 — Medium — Pattern: 2D DP with Obstacles
  - Hint: identical recurrence to Unique Paths, except any obstacle cell forces `dp[i][j] = 0` — no paths can pass through it.
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 3: Minimum Path Sum — LeetCode #64 — Medium — Pattern: 2D DP
  - Hint: `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`.
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 4: Maximal Square — LeetCode #221 — Medium — Pattern: 2D Matrix DP
  - Hint: `dp[i][j]` represents the side length of the largest all-1s square with its bottom-right corner at `(i,j)`. It equals `min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1` — the square can only be as large as its tightest-limiting neighbor allows, plus one.
  - Complexity: Time O(m×n) | Space O(m×n)

**This closes Grid DP: 4 problems, all in one day — a good demonstration of how much faster a pattern moves once you've built real fluency in the DP mindset over the past six days.**

### Daily Deliverable
- [ ] Unique Paths, Unique Paths II, Minimum Path Sum, and Maximal Square solved — Grid DP ladder complete at 4 problems.

---

## Day 88 — String DP Begins

### DSA Block (4-5 hrs at leave-week intensity)

**Concept Card — String DP**
- What: `dp[i][j]` now represents a relationship between a prefix of one string and a prefix of another — or one string and itself, for palindrome-related problems.
- Why: string comparison and transformation problems (diffs, spell-checkers, DNA alignment) almost universally reduce to this exact shape.
- Interview signal: two strings being compared, "edit"/"transform one string into another," or anything palindrome-related.

- Problem 1: Longest Common Subsequence — LeetCode #1143 — Medium — Pattern: 2D String DP
  - Hint: `dp[i][j]` is the length of the LCS of `s1[0..i]` and `s2[0..j]`. If the characters at those positions match: `1 + dp[i-1][j-1]`. If they don't: `max(dp[i-1][j], dp[i][j-1])`.
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 2: Edit Distance — LeetCode #72 — Medium — Pattern: 2D String DP
  - Hint: if the characters match, `dp[i][j] = dp[i-1][j-1]` — no edit needed. If they don't match, `1 + min(insert, delete, replace)`, taking the minimum of the three neighboring cells that each represent one of those operations.
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 3: Longest Palindromic Substring — LeetCode #5 — Medium — Pattern: Expand Around Center
  - Hint: treat every character, and every gap between two characters, as a potential palindrome center, and expand outward from each — tracking the longest one found.
  - Complexity: Time O(n²) | Space O(1)

### Daily Deliverable
- [ ] Longest Common Subsequence, Edit Distance, and Longest Palindromic Substring solved, pushed.

---

## Day 89 — String DP: Palindrome Counting and Interleaving

### DSA Block (4-5 hrs at leave-week intensity)
- Problem 4: Palindromic Substrings — LeetCode #647 — Medium — Pattern: Expand Around Center
  - Hint: same expand-around-center technique as yesterday, but instead of tracking the longest one, count every valid palindrome found along the way.
  - Complexity: Time O(n²) | Space O(1)
- Problem 5: Longest Palindromic Subsequence — LeetCode #516 — Medium — Pattern: 2D String DP
  - Hint: this is exactly the Longest Common Subsequence (Day 88) of the string and its own reverse — reuse that logic directly, no new recurrence needed.
  - Complexity: Time O(n²) | Space O(n²)
- Problem 6: Interleaving String — LeetCode #97 — Medium — Pattern: 2D String DP
  - Hint: `dp[i][j]` is true if `s1[0..i]` and `s2[0..j]` can interleave to form exactly `s3[0..i+j]` — at each cell, check both possible sources for the current character of `s3`.
  - Complexity: Time O(m×n) | Space O(m×n)

### Daily Deliverable
- [ ] Palindromic Substrings, Longest Palindromic Subsequence, and Interleaving String solved, pushed.

---

## Day 90 — String DP Completes

### DSA Block (4-5 hrs at leave-week intensity)
- Problem 7: Distinct Subsequences — LeetCode #115 — Hard — Pattern: 2D String DP
  - Hint: `dp[i][j]` counts the number of ways `s[0..i]` can form `t[0..j]` as a subsequence. If the characters match, you have two choices — use this match, or skip it: `dp[i-1][j-1] + dp[i-1][j]`. If they don't match, only the skip option exists: `dp[i-1][j]`.
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 8: Wildcard Matching — LeetCode #44 — Hard — Pattern: 2D DP
  - Hint: `dp[i][j]` is true if `s[0..i]` matches pattern `p[0..j]`. Handle `*` carefully — it can match an empty sequence (`dp[i][j-1]`) or extend to consume one more character of `s` while still matching (`dp[i-1][j]`).
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 9: Regular Expression Matching — LeetCode #10 — Hard — Pattern: 2D DP **(new)**
  - Hint: nearly the same skeleton as Wildcard Matching, but `*` here means "zero or more of the *preceding* character," not "any sequence" — so `*` needs to look one pattern-position back before deciding what it's repeating.
  - Complexity: Time O(m×n) | Space O(m×n)

**This closes String DP: 9 problems (Longest Common Subsequence, Edit Distance, Longest Palindromic Substring, Palindromic Substrings, Longest Palindromic Subsequence, Interleaving String, Distinct Subsequences, Wildcard Matching, Regular Expression Matching) — the original plan flagged Regular Expression Matching only as an optional extension; it's now a fully scheduled problem instead of a suggestion.**

### Daily Deliverable
- [ ] Distinct Subsequences, Wildcard Matching, and Regular Expression Matching solved — String DP ladder complete at 9 problems.

---

## Day 91 (Sunday) — Interval DP Completes, and Leave Week 2 Wraps Up

### Self-Check (10 min)
- [ ] Pick one problem from earlier this week and state its `dp[]` definition from memory before re-solving it.

### DSA Block (3-4 hrs)

**Concept Card — Interval DP**
- What: `dp[i][j]` represents the optimal answer over the subrange `[i, j]`, typically computed by trying every possible "split point" or "last operation" within that range.
- Why: problems where an operation on a range depends on how you first divide or order the range — genuinely one of the harder DP shapes, since the recurrence isn't a simple left-to-right sweep.
- Interview signal: "matrix chain," "burst/remove in some order," any problem where the *order* in which you perform operations within a range changes the outcome.

- Problem 1: Burst Balloons — LeetCode #312 — Hard — Pattern: Interval DP
  - Hint: think backwards — pick the *last* balloon to burst within range `[i, j]`, not the first. `dp[i][j]` is the maximum coins obtainable from bursting every balloon strictly between `i` and `j`.
  - Complexity: Time O(n³) | Space O(n²)
- Problem 2: Minimum Insertion Steps to Make a String Palindrome — LeetCode #1312 — Medium — Pattern: Interval DP
  - Hint: the answer is `string.length() - (length of the Longest Palindromic Subsequence)` — Day 89's problem, reused directly with no new recurrence needed.
  - Complexity: Time O(n²) | Space O(n²)

**This closes Interval DP: 2 problems.**

### Career Block (1 hr) — back to normal cadence as leave week 2 ends
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 91, thirteen weeks in, **184 total DSA problems solved.** Leave week 2 is complete — 1D DP fully closed (14), Grid DP fully closed (4), String DP fully closed (9, including the Regular Expression Matching upgrade), and Interval DP fully closed (2). Only State Machine DP (4 problems) and Tree DP (2 problems) remain to close out Dynamic Programming entirely — both manageable at normal pace over the next few days.

### Daily Deliverable
- [ ] Burst Balloons and Minimum Insertion Steps to Make a String Palindrome solved — Interval DP ladder complete at 2 problems.
- [ ] Leave week 2 complete. Weekly ritual and scorecard done.
