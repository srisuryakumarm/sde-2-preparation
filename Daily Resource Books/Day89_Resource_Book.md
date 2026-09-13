# Day 89 — String DP Continues: Counting Palindromes, the LCS-Reverse Trick, and Three-Way Interleaving

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 88 Resource Book](Day88_Resource_Book.md)
**Next ▶:** [Day 90 Resource Book](Day90_Resource_Book.md)
**Companion to:** Day 89 of `Week_13_Revised.md`

---

## Recap

Yesterday opened String DP with two shapes: two-string comparison (LCS, Edit Distance) and a string against itself (Longest Palindromic Substring, via Expand Around Center). Today reuses both directly rather than introducing a third shape: Palindromic Substrings is yesterday's exact expand-around-center technique with a different aggregation (count, not track-longest); Longest Palindromic Subsequence is yesterday's exact LCS recurrence, run against the string and its own reverse — a genuinely non-obvious equivalence that gets a full two-directional argument below, not just an assertion. Interleaving String closes today by extending the `(m+1)×(n+1)` table convention to a third string, `s3`, without adding a third array dimension — worth seeing precisely how that's avoided.

---

## Learning Objectives

By the end of today, without notes:

1. Adapt yesterday's Expand Around Center code to *count* palindromes instead of tracking the longest, changing only the aggregation step.
2. Prove, in both directions, why `LCS(s, reverse(s))` equals the Longest Palindromic Subsequence's length — not just cite the equivalence.
3. State Interleaving String's `dp[i][j]` definition precisely, and explain why `i+j` (not a third independent index) is enough to know the corresponding position in `s3`.
4. Correctly derive Interleaving String's two base-case rows (`dp[i][0]`, `dp[0][j]`), including why they can fail partway through even when every earlier cell in the row was `true`.

---

## Concept Dependency Map

```
Day 88 — Expand Around Center (Longest Palindromic Substring): 2n-1 centers
        │
        ▼
TODAY, Problem 4 — Palindromic Substrings (LC 647): SAME expansion, COUNT not track-longest

Day 88 — LCS recurrence (two strings, match/mismatch branching)
        │
        ▼
TODAY, Problem 5 — Longest Palindromic Subsequence (LC 516):
   LCS(s, reverse(s)) — proven, both directions, below

Day 88 — (m+1)x(n+1) table convention, empty-prefix base case
        │
        ▼
TODAY, Problem 6 — Interleaving String (LC 97): table convention extended to
   a THIRD string s3, without a third dimension — position in s3 is always
   exactly i+j, so no extra index is needed

        │
        ▼
🔗 forward: Day 88's deferred isPalindrome-table payoff (Week 10 Day 66)
   completes tomorrow (Day 90) with an explicit extension problem
```

---

## Problem 4: Palindromic Substrings (LC 647, Medium) — Pattern: Expand Around Center

**Statement:** Given a string, return the total number of palindromic substrings it contains (different start/end positions count separately, even if the substrings have identical content).

### Approach 1 — Brute force: check every substring

```java
public static int countSubstringsBruteForce(String s) {
    int n = s.length(), count = 0;
    for (int i = 0; i < n; i++) {
        for (int j = i; j < n; j++) {
            if (isPalindrome(s, i, j)) count++;
        }
    }
    return count;
}

private static boolean isPalindrome(String s, int left, int right) {
    while (left < right) {
        if (s.charAt(left++) != s.charAt(right--)) return false;
    }
    return true;
}
```

**Complexity: Time O(n³), Space O(1)** — same shape as yesterday's brute-force Longest Palindromic Substring.

### Approach 2 — Optimized: Expand Around Center, counting instead of tracking

```java
public static int countSubstrings(String s) {
    int count = 0;
    for (int center = 0; center < s.length(); center++) {
        count += expandAndCount(s, center, center);          // odd-length centers
        count += expandAndCount(s, center, center + 1);      // even-length centers
    }
    return count;
}

private static int expandAndCount(String s, int left, int right) {
    int count = 0;
    while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
        count++;      // every successful expansion is itself one more valid palindromic substring
        left--;
        right++;
    }
    return count;
}
```

**The only change from yesterday's technique:** yesterday's `expandFromCenter` returned a single width (`right - left - 1`) after the loop finished, then compared it against a running maximum. Today's `expandAndCount` increments a counter *inside* the loop, once per successful expansion — because every time the expansion succeeds one step further, that's a *newly confirmed* palindromic substring (a strictly larger one, centered the same place), not a replacement for a previous answer. The `2n-1` centers argument from yesterday applies identically and needs no re-derivation.

**Worked trace:** `s = "aaa"`. Center `i=0` ('a'): odd-expand from (0,0) → matches once (just "a"), count 1; can't expand further (left would go to -1). Even-expand from (0,1): 'a' vs 'a' → match → count 1 ("aa"); further would need index -1, stop. Center `i=1` ('a'): odd-expand from (1,1) → "a" (count 1); expand to (0,2): 'a'=='a' → match → count 1 more ("aaa"); further needs index -1, stop — subtotal 2. Even-expand from (1,2): 'a'=='a' → count 1 ("aa"); further needs index -1, stop. Center `i=2` ('a'): odd-expand → "a" (count 1); even-expand from (2,3): index 3 out of bounds, count 0. Total: `1+1 + 2+1 + 1+0 = 6` — matching the known answer for `"aaa"` (the six palindromic substrings are `"a","a","a","aa","aa","aaa"`, counted by position, not deduplicated by content).

**Complexity: Time O(n²), Space O(1)** — identical shape to yesterday's Longest Palindromic Substring.

**Edge cases:** every single character is always its own palindromic substring, contributing exactly `n` to the total on its own — a useful sanity-check lower bound (`count ≥ n` always). All-identical-character string (e.g. `"aaaa"`) → maximal count, `n(n+1)/2` (every substring is a palindrome).

> 💡 **Interview Insight:** stating "this is yesterday's Expand Around Center, aggregating with a running count instead of a running max" immediately signals the reuse — worth naming explicitly, since the two problems otherwise look different enough (return an `int` count vs. a `String`) that an interviewer might not assume the connection is obvious to the candidate.

---

## Problem 5: Longest Palindromic Subsequence (LC 516, Medium) — Pattern: 2D String DP

**Statement:** Given a string, return the length of its longest palindromic **subsequence** (not necessarily contiguous — contrast directly with yesterday's *substring* version).

### The claim, and why it needs a real proof, not just a citation

**Claim:** `LPS(s) = LCS(s, reverse(s))`. This is *not* obvious on its face — LCS finds characters common to two strings in the same relative order; nothing in that definition obviously talks about palindromes. Both directions are argued below.

**Direction 1 — `LPS(s) ≤ LCS(s, reverse(s))`, proven:** let `P` be any palindromic subsequence of `s`, found at increasing indices `i₁ < i₂ < ... < iₖ` in `s`. Because `P` is a palindrome, `P` read backward is identical to `P` read forward. Reading `s` from the *back* (i.e., reading `reverse(s)` from the front) at the mirrored positions `(n-1-iₖ) < (n-1-i_{k-1}) < ... < (n-1-i₁)` (a valid increasing sequence of indices into `reverse(s)`, since reversing a decreasing sequence of original-string positions gives an increasing one) produces exactly `s[iₖ], s[i_{k-1}], ..., s[i₁]` — which is `P` read backward, and since `P` is a palindrome, that's the same string `P` again. So `P` is *also* a valid subsequence of `reverse(s)`, at those mirrored indices. `P` is therefore a common subsequence of `s` and `reverse(s)`, meaning `LCS(s, reverse(s)) ≥ |P|` for every palindromic subsequence `P` — in particular for the longest one, giving `LCS(s,reverse(s)) ≥ LPS(s)`.

**Direction 2 — `LCS(s, reverse(s)) ≤ LPS(s)`, demonstrated via a concrete backtrack (the standard, and most instructive, way to see this):** rather than a fully general algebraic argument, trace an actual case and observe what the alignment forces. `s = "bbbab"`, `reverse(s) = "babbb"`.

| | "" | b | a | b | b | b |
|---|---|---|---|---|---|---|
| "" | 0 | 0 | 0 | 0 | 0 | 0 |
| b | 0 | 1 | 1 | 1 | 1 | 1 |
| b | 0 | 1 | 1 | 2 | 2 | 2 |
| b | 0 | 1 | 1 | 2 | 3 | 3 |
| a | 0 | 1 | 2 | 2 | 3 | 3 |
| b | 0 | 1 | 2 | 3 | 3 | 4 |

`dp[5][5] = 4`. Backtracking from the bottom-right corner (standard LCS reconstruction: follow a diagonal step whenever the current characters match, otherwise step toward whichever neighbor holds the equal-or-larger value) produces the sequence **"bbbb"** — which *is* a palindrome, and is genuinely the longest palindromic subsequence of `"bbbab"` (confirmed independently against a full 2⁵-subsequence brute-force check before writing this book).

**Why the backtrack is *forced* into a symmetric result, not just lucky here:** every matched pair in the alignment pairs a position `i` in `s` (read left to right) against a position `j` in `reverse(s)` (also read left to right) — but reading `reverse(s)` left-to-right *is* reading `s` right-to-left. So every matched character-pair in the LCS alignment corresponds to a pair of positions in the *original* `s` — one approaching from the left, one from the right — that hold equal characters. Building the matched sequence up from *both ends inward simultaneously* is exactly the structural definition of a palindrome: for every character placed at position `k` from the front of the resulting subsequence, the character at position `k` from the *back* is forced to be identical, because that's precisely what "matched against the reverse" means at every step of the alignment.

**Complexity: Time O(n²), Space O(n²)** — the LCS table run on `s` and `reverse(s)`, both length `n`; optimizable to O(n) with the same rolling-row technique.

**Edge cases:** single character → itself, length 1 (trivially a palindrome). Entire string already a palindrome → `LPS = n`, the full string. No repeated characters at all (e.g. `"abcde"`) → `LPS = 1` (any single character, the only guaranteed palindrome).

> 💡 **Interview Insight:** if asked "why does this work" and only "it's a known trick" comes to mind, that's a gap worth closing before the interview, not during it — the "≥" direction above is a clean, fully general proof worth having ready verbatim; the "≤" direction is best defended by walking through exactly this kind of concrete backtrack and pointing at the both-ends-inward structure it reveals.

---

## Problem 6: Interleaving String (LC 97, Medium) — Pattern: 2D String DP

**Statement:** Given three strings `s1`, `s2`, `s3`, determine whether `s3` can be formed by interleaving `s1` and `s2` — preserving each string's own internal character order, but freely interleaving the two sources together.

### Approach 1 — Brute force: recursive branching on which source provides the next character

```java
public static boolean isInterleaveBruteForce(String s1, String s2, String s3) {
    if (s1.length() + s2.length() != s3.length()) return false;
    return interleaveHelper(s1, s2, s3, 0, 0);
}

private static boolean interleaveHelper(String s1, String s2, String s3, int i, int j) {
    int k = i + j;
    if (k == s3.length()) return true;   // both sources exhausted, s3 fully matched
    boolean fromS1 = i < s1.length() && s1.charAt(i) == s3.charAt(k)
                      && interleaveHelper(s1, s2, s3, i + 1, j);
    boolean fromS2 = j < s2.length() && s2.charAt(j) == s3.charAt(k)
                      && interleaveHelper(s1, s2, s3, i, j + 1);
    return fromS1 || fromS2;
}
```

**Complexity: Time O(2^(m+n))** worst case — massive overlapping recomputation of identical `(i,j)` states, the same redundancy pattern this entire subtype keeps re-deriving DP from.

### Approach 2 — Optimized: tabulation

```java
public static boolean isInterleave(String s1, String s2, String s3) {
    int m = s1.length(), n = s2.length();
    if (m + n != s3.length()) return false;   // fast reject before any table work

    boolean[][] dp = new boolean[m + 1][n + 1];
    dp[0][0] = true;

    for (int i = 1; i <= m; i++) {
        dp[i][0] = dp[i - 1][0] && s1.charAt(i - 1) == s3.charAt(i - 1);
    }
    for (int j = 1; j <= n; j++) {
        dp[0][j] = dp[0][j - 1] && s2.charAt(j - 1) == s3.charAt(j - 1);
    }

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            dp[i][j] = (dp[i - 1][j] && s1.charAt(i - 1) == s3.charAt(i + j - 1))
                    || (dp[i][j - 1] && s2.charAt(j - 1) == s3.charAt(i + j - 1));
        }
    }
    return dp[m][n];
}
```

**`dp[i][j]` represents:** whether `s1`'s first `i` characters and `s2`'s first `j` characters can interleave to form exactly `s3`'s first `i+j` characters. **Why no third index is needed for the position in `s3`:** every character consumed from *either* source advances the interleaved result by exactly one character — so after consuming `i` characters from `s1` and `j` from `s2`, *however* they were interleaved, exactly `i+j` characters of `s3` have necessarily been produced. The position in `s3` is therefore always a deterministic function of `i` and `j`, never an independent choice — which is precisely why a 2D table suffices for what looks, at first glance, like a fundamentally three-string problem.

**Why the recurrence checks two *sources*, not two *directions*:** at cell `(i,j)`, the character `s3[i+j-1]` (the most recently placed character of `s3`) must have come from *either* the end of `s1`'s consumed prefix or the end of `s2`'s consumed prefix — never both, never neither, if `s3` is actually formed by this specific interleaving. So `dp[i][j]` is true exactly when at least one of those two sourcing stories checks out **and** the state *before* that last character was placed was itself valid (`dp[i-1][j]` or `dp[i][j-1]` respectively).

**⚠️ Common Mistake — the base-case row/column isn't "all true up to the first mismatch, unconditionally provable in isolation."** `dp[i][0]` requires *both* `dp[i-1][0]` (the previous position was validly built) *and* the current character matching — a single mismatch anywhere in the run makes every subsequent `dp[i][0]` in that row `false`, exactly the same "propagate the failure forward" shape as Unique Paths II's obstacle rule (Day 87). Writing `dp[i][0] = (s1.charAt(i-1) == s3.charAt(i-1))` alone, without the `&& dp[i-1][0]` term, would let a *later* coincidental character match after an earlier failure incorrectly "revive" the row.

**Worked trace:** `s1 = "aabcc"`, `s2 = "dbbca"`, `s3 = "aadbbcbcac"`. Lengths: `5+5=10=len(s3)` ✓. Building the table (independently confirmed against a full recursive check before writing this book): `dp[0][0]=true`. First column: `dp[1][0] = ('a'=='a')=true`; `dp[2][0] = dp[1][0] && ('a'=='a') = true`; `dp[3][0] = dp[2][0] && ('b'=='d')` → `'b'≠'d'` → `false`, and every `dp[i][0]` for `i≥3` stays `false`. First row: `dp[0][1] = ('d'=='a')=false`, and every `dp[0][j]` for `j≥1` stays `false`. From here the interior cells combine contributions from both directions; the final `dp[5][5] = true` — confirming `s3` genuinely is a valid interleaving. A second check, `s1="aabcc", s2="dbbca", s3="aadbbbaccc"` (same lengths, different `s3`), returns `false` — confirmed directly.

**Complexity: Time O(m×n), Space O(m×n)**, optimizable to O(min(m,n)) — the rolling-row technique applies identically, since each cell only ever reads the row directly above and the cell directly to the left.

**Edge cases:** all three strings empty → `dp[0][0]=true` trivially, no interleaving needed. `s1` or `s2` empty → `s3` must exactly equal the other one, correctly reduced to a single base-case row/column check. Length mismatch (`m+n ≠ len(s3)`) → rejected immediately, no table work needed at all.

> 💡 **Interview Insight:** the single highest-value thing to say unprompted here is the "why only two dimensions, when three strings are involved" justification — that the position in `s3` is fully determined by `i+j`, never a free variable. Interviewers use this problem specifically to check whether a candidate notices that reduction, versus reaching immediately (and unnecessarily) for a 3D table.

---

## Day 89 — Interview Questions

---

**1. What's the only real change between yesterday's Longest Palindromic Substring and today's Palindromic Substrings?**

*Answer:* The aggregation step. Both use identical Expand Around Center code with `2n-1` centers; yesterday tracked the single longest expansion found, today counts every successful expansion as one more valid palindromic substring, incrementing inside the expansion loop rather than comparing against a running maximum afterward.

---

**2. Prove the "≥" direction of `LPS(s) = LCS(s, reverse(s))`.**

*Answer:* Any palindromic subsequence `P` of `s`, found at increasing indices in `s`, is — because `P` equals its own reverse — also extractable from `reverse(s)` at the mirrored (also increasing) index positions, producing the same string `P`. So `P` is a common subsequence of `s` and `reverse(s)`, meaning `LCS(s,reverse(s)) ≥ |P|` for every palindromic `P`, hence `≥ LPS(s)`.

---

**3. Why does a matched pair in the `LCS(s, reverse(s))` alignment correspond to a symmetric pair of positions in the original `s`?**

*Answer:* Reading `reverse(s)` left to right is the same as reading `s` right to left. Every matched character-pair in the alignment therefore pairs a position approaching from `s`'s front with a position approaching from `s`'s back — building the result from both ends inward simultaneously, which is exactly the defining structure of a palindrome.

---

**4. In Interleaving String, why does `dp[i][j]` fully determine the corresponding position in `s3`, with no third index needed?**

*Answer:* Every character consumed from either source advances the interleaved output by exactly one character, regardless of which source it came from — so after consuming `i` from `s1` and `j` from `s2`, exactly `i+j` characters of `s3` have been produced, deterministically. The position in `s3` is a function of `i+j`, never an independent choice.

---

**5. Why can't Interleaving String's first row/column be computed independently at each cell — why does a mismatch early in the row affect every cell after it?**

*Answer:* `dp[i][0]` requires both the current character to match **and** `dp[i-1][0]` to already be true — a single mismatch breaks the chain, and since there's only one possible source (`s1` alone) for the entire first column, nothing can "recover" after a break the way the two-source interior cells sometimes can. Every subsequent cell in that row/column inherits the `false`, the same forward-propagating-failure shape as Unique Paths II's obstacle rule.

---

## Daily Deliverable Check

- [ ] Palindromic Substrings, Longest Palindromic Subsequence, and Interleaving String all solved and pushed to `dsa-java/dynamic-programming/`.
- [ ] Can state and defend both directions of the LPS = LCS(s, reverse(s)) proof, not just cite the equivalence.
- [ ] Interleaving String's base-case propagation edge case tested explicitly (a mismatch partway through the first row or column).
- [ ] Can explain why Interleaving String needs only two dimensions despite three strings being involved.

---

## What Tomorrow Assumes You Already Know Cold

Day 90 closes String DP with three Hard problems (Distinct Subsequences, Wildcard Matching, Regular Expression Matching), all extending today's and yesterday's `(m+1)×(n+1)` table convention with progressively more intricate branching on what a `*` or a mismatch is allowed to mean. It assumes the "map each operation/case to exactly which neighboring cell it reads from" discipline (used today for Interleaving String's two sourcing stories, yesterday for Edit Distance's three operations) is now automatic — tomorrow's two pattern-matching problems each have four or more distinct cases per cell, and the book won't re-explain the general mapping discipline itself, only the new cases.
