# Day 88 — String DP Begins: LCS, Edit Distance, and Expand-Around-Center

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 87 Resource Book](Day87_Resource_Book.md)
**Next ▶:** [Day 89 Resource Book](Day89_Resource_Book.md)
**Companion to:** Day 88 of `Week_13_Revised.md`

---

## Recap

Grid DP opened and closed yesterday in a single day — four problems, all built on `dp[i][j]` combining a small fixed set of spatial neighbors. Today opens String DP, and the concept card's own framing makes the connection explicit: the two dimensions stop being grid coordinates and become *positions in two different strings* (or one string against itself, for palindrome problems). The `dp[i][j]`-from-neighbors mechanism transfers directly; what's new is *why* two cells are related — a character match or mismatch, not spatial adjacency.

String DP is this week's largest subtype at 9 problems across three days — today's three (LCS, Edit Distance, Longest Palindromic Substring) establish the two core shapes (two-string comparison, and single-string palindrome structure) that the remaining six all build on.

---

## Learning Objectives

By the end of today, without notes:

1. State why String DP conventionally uses a `(m+1)×(n+1)` table over 0-indexed strings, rather than an `m×n` table — and explain what row 0 / column 0 represent and why that avoids special-casing an empty prefix.
2. Derive both branches of the LCS recurrence (character match vs. mismatch) and explain why a mismatch takes a `max`, not a fixed fallback.
3. Derive all three branches of the Edit Distance recurrence (insert, delete, replace) and correctly map each to which neighboring cell it reads from.
4. Explain precisely why Longest Palindromic Substring's expand-around-center technique needs `2n-1` centers, not `n`.

---

## Concept Dependency Map

```
Yesterday — Grid DP: dp[i][j] from spatial neighbors (up, left, diagonal)
Week 12, Day 81 — DP discipline: state dp[]'s meaning before writing any recurrence
        │
        ▼
NEW CONCEPT — String DP: dp[i][j] relates a PREFIX of one string to a PREFIX
   of another (or a string to itself, for palindromes) — table convention:
   (m+1)×(n+1) over 0-indexed strings, row/col 0 = "empty prefix"
        │
        ├─▶ Problem 1 — Longest Common Subsequence (LC 1143):
        │      match → diagonal+1; mismatch → max(up, left)
        │      🔗 EXTRA: Delete Operation for Two Strings (LC 583) — direct
        │         application, no new recurrence
        │
        ├─▶ Problem 2 — Edit Distance (LC 72):
        │      match → diagonal (free); mismatch → 1+min(insert,delete,replace)
        │      (three neighbors this time, not two — a genuinely different
        │       branching structure from LCS despite the identical table shape)
        │
        └─▶ Problem 3 — Longest Palindromic Substring (LC 5):
               Expand Around Center — a STRING is compared against ITSELF,
               not against a second string — the other String DP shape.
               2n-1 centers (n single-char + n-1 between-char, for even length)
               🔗 forward: the DP-table version of "is this a palindrome"
               is exactly Week 10 Day 66's deferred optimization — paid off
               tomorrow (Day 89) and the day after (Day 90's extension).
```

---

## 🔑 NEW CONCEPT: String DP

**Prerequisites (confirmed):** Grid DP's `dp[i][j]`-from-neighbors mechanism (yesterday). String indexing and `charAt()` / `substring()` (Week 1-2). The general DP discipline — state `dp[]`'s meaning precisely, verify against a trace (Day 81, onward).

**What it is:** `dp[i][j]` represents a relationship between a **prefix** of one string and a **prefix** of another — or, for palindrome problems, a relationship between a string and itself. This is the same "two-index table" shape as Grid DP, but the two indices no longer mean "row, column of a physical grid" — they mean "how far into string 1, how far into string 2."

**The table convention, and why it's `(m+1)×(n+1)`, not `m×n`:** every problem below uses `dp[i][j]` to mean "considering `s1`'s first `i` characters and `s2`'s first `j` characters" — a **1-indexed count over a 0-indexed string**. `dp[0][j]` and `dp[i][0]` represent an **empty prefix** on one side, which is a real, meaningful base case (e.g., the LCS of "anything" and "nothing" is length 0) — not a special case to guard against. This convention is deliberate: it avoids negative-index checks entirely (`s1[i-1]` is always valid when `i ≥ 1`, since the loop never runs `i=0` in the main recurrence body), which is exactly the kind of off-by-one trap this problem family is famous for in interviews.

**Why it works:** identical justification to Grid DP's — optimal substructure (a prefix-pair's best answer is built from smaller prefix-pairs' best answers) plus overlapping subproblems (the same prefix-pair recurs across many recursive paths in a naive solution). The only change from yesterday is what "smaller" means: instead of moving toward `(0,0)` spatially, the table moves toward `(0,0)` by shrinking one or both prefixes by one character at a time.

**When to reach for it, the concrete signal:** two strings being compared, "edit" or "transform one string into another," or anything palindrome-related on a single string. This is a much broader trigger phrase set than Grid DP's "moving through a grid" — String DP is one of the most frequently-tested DP shapes at the tier-1 level specifically because so many real string-processing problems (diffs, spell-checkers, DNA sequence alignment, autocomplete) reduce to exactly this table shape.

**Trade-offs against the nearest alternative:** vs. a two-pointer sweep (Week 1-2) — two-pointer techniques work when a *single* optimal alignment can be found greedily and confirmed correct without exploring alternatives; String DP is needed the moment multiple candidate alignments must be compared against each other for an optimum or a count, which is most of today's and tomorrow's problems.

**Complexity, with reasoning:** Time **O(m×n)** — one O(1)-work cell per pair of prefix lengths. Space **O(m×n)**, optimizable to **O(min(m,n))** via the same rolling-row idea as Grid DP — most String DP recurrences below only ever read the row directly above and the cell directly to the left, so a full 2D table is a correctness-first default, not a hard requirement.

---

## Problem 1: Longest Common Subsequence (LC 1143, Medium) — Pattern: 2D String DP

**Statement:** Given two strings, return the length of their longest common subsequence (characters in the same relative order, not necessarily contiguous).

### Approach 1 — Brute force: recursive branching on match/no-match

```java
public static int lcsBruteForce(String s1, String s2) {
    return lcsHelper(s1, s2, s1.length(), s2.length());
}

private static int lcsHelper(String s1, String s2, int i, int j) {
    if (i == 0 || j == 0) return 0;                      // empty prefix — no common subsequence possible
    if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
        return 1 + lcsHelper(s1, s2, i - 1, j - 1);       // characters match — both must be part of the LCS
    }
    return Math.max(lcsHelper(s1, s2, i - 1, j), lcsHelper(s1, s2, i, j - 1));
}
```

**Complexity: Time O(2^(m+n))** worst case — every mismatch branches two ways, with massive overlapping recomputation of identical `(i,j)` pairs (the exact redundancy Day 81's `fib(5)` retracing argument demonstrated, now over a 2D subproblem space). **Space O(m+n)** recursion depth.

### Approach 2 — Optimized: tabulation

```java
public static int longestCommonSubsequence(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];   // dp[0][*] and dp[*][0] default to 0 — the empty-prefix base case, for free

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}
```

**`dp[i][j]` represents:** the LCS length of `s1`'s first `i` characters and `s2`'s first `j` characters. **Why a match takes the diagonal `+1`, unconditionally:** if `s1[i-1] == s2[j-1]`, this shared character can always be safely included in *some* LCS of the two full prefixes — there's never a reason to discard a matching character in favor of a strictly-worse alternative, so the answer is exactly one more than the best answer for both prefixes shortened by one. **Why a mismatch takes `max`, not a fixed fallback:** when the last characters differ, at least one of them cannot be part of the LCS ending at both these exact prefix lengths — but *which* one is discardable isn't knowable without comparison, so the recurrence tries both ("discard `s1`'s last char" = `dp[i-1][j]`; "discard `s2`'s last char" = `dp[i][j-1]`) and keeps whichever is better. This is the same "try both, keep the better one" shape Grid DP's Minimum Path Sum used yesterday, applied to a character-comparison branch instead of a spatial one.

**Worked trace:** `s1 = "abcde"`, `s2 = "ace"`.

| | "" | a | c | e |
|---|---|---|---|---|
| "" | 0 | 0 | 0 | 0 |
| a | 0 | 1 | 1 | 1 |
| b | 0 | 1 | 1 | 1 |
| c | 0 | 1 | 2 | 2 |
| d | 0 | 1 | 2 | 2 |
| e | 0 | 1 | 2 | **3** |

At `(a,a)`: match → `dp[0][0]+1=1`. At `(c,c)` (row "c", column "c"): match → `dp[2][1]+1=1+1=2`. At `(e,e)` (row "e", column "e"): match → `dp[4][2]+1=2+1=3`. Final `dp[5][3]=3` — LCS is `"ace"`, length 3, matching the known answer for this classic example.

**Complexity: Time O(m×n), Space O(m×n)**, optimizable to O(min(m,n)) via rolling rows.

**Edge cases:** either string empty → `0` immediately, falls out of the base case with no special-casing. No characters in common at all → `0`. One string is a subsequence of the other → LCS equals the shorter string's full length.

> 💡 **Interview Insight:** LCS is the single most-reused recurrence shape in this entire subtype — naming, unprompted, that a new problem "reduces to LCS" (as several problems this week and next explicitly will) is worth more than re-deriving the table from scratch each time.

---

## Extra Practice (light): Delete Operation for Two Strings (LC 583, Medium) — Pattern: 2D String DP (Direct LCS Application)

**Why this is worth the few extra minutes:** essentially free, given LCS is already built — a clean, fast confirmation that the "reduce to a known shape" reflex (explicitly named as the goal in this week's Target Sum and Minimum Insertion Steps problems) applies here too, with zero new recurrence to learn.

**Statement:** Given two strings, find the minimum number of characters to delete (from either string) to make them equal.

```java
public static int minDistance(String word1, String word2) {
    int lcs = longestCommonSubsequence(word1, word2);   // Problem 1's exact function, unchanged
    return (word1.length() - lcs) + (word2.length() - lcs);
}
```

**The reduction:** whatever the LCS is, it can stay untouched in both strings — every character in each string that is *not* part of that shared subsequence has to go, since it's precisely the presence of those extra characters that makes the two strings unequal in the first place. Deleting exactly `(word1.length() - lcs)` characters from `word1` and `(word2.length() - lcs)` from `word2` leaves both strings equal to the LCS itself, and no smaller number of deletions can work, since any common result the two strings could be reduced to is, by definition, some common subsequence of both — and the LCS is the *longest* one, minimizing the total characters that must be discarded.

**Complexity: Time O(m×n), Space O(m×n)** — identical to LCS, since this problem is LCS plus O(1) arithmetic.

**Edge cases:** identical strings → LCS equals both lengths, `0` deletions needed. No characters in common → LCS `0`, delete everything from both (`word1.length() + word2.length()` total).

---

## Problem 2: Edit Distance (LC 72, Medium) — Pattern: 2D String DP

**Statement:** Given two strings, return the minimum number of operations (insert, delete, or replace one character) to convert one into the other.

### Approach 1 — Brute force: recursive branching over three operations

```java
public static int editDistanceBruteForce(String s1, String s2) {
    return editHelper(s1, s2, s1.length(), s2.length());
}

private static int editHelper(String s1, String s2, int i, int j) {
    if (i == 0) return j;   // s1 exhausted — insert all of s2's remaining characters
    if (j == 0) return i;   // s2 exhausted — delete all of s1's remaining characters
    if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
        return editHelper(s1, s2, i - 1, j - 1);   // characters already match — no operation needed here
    }
    int replace = editHelper(s1, s2, i - 1, j - 1);
    int delete  = editHelper(s1, s2, i - 1, j);
    int insert  = editHelper(s1, s2, i, j - 1);
    return 1 + Math.min(replace, Math.min(delete, insert));
}
```

**Complexity: Time O(3^(m+n))** worst case — three-way branching on every mismatch. **Space O(m+n)** recursion depth.

### Approach 2 — Optimized: tabulation

```java
public static int minDistanceEdit(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];

    for (int i = 0; i <= m; i++) dp[i][0] = i;   // convert s1's first i chars to "" — i deletions
    for (int j = 0; j <= n; j++) dp[0][j] = j;   // convert "" to s2's first j chars — j insertions

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];      // matching characters cost nothing
            } else {
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1],   // replace
                                Math.min(dp[i - 1][j],       // delete from s1
                                         dp[i][j - 1]));     // insert into s1
            }
        }
    }
    return dp[m][n];
}
```

**`dp[i][j]` represents:** the minimum number of operations to convert `s1`'s first `i` characters into `s2`'s first `j` characters. **Mapping each operation to exactly which cell it reads, precisely:**
- **Replace** `s1[i-1]` with `s2[j-1]`: after replacing, both "last characters" effectively match, so the remaining work is converting the two prefixes *before* these last characters — `dp[i-1][j-1]`.
- **Delete** `s1[i-1]`: `s1`'s prefix shrinks by one character, `s2`'s doesn't — remaining work is `dp[i-1][j]`.
- **Insert** a character into `s1` to match `s2[j-1]`: conceptually, `s1`'s prefix is unchanged but now "aligned" one character further into `s2` — remaining work is `dp[i][j-1]`.

Each of the three operations costs exactly `1`, plus whatever the corresponding smaller subproblem costs — take the cheapest of the three.

**Worked trace:** `s1 = "horse"`, `s2 = "ros"`.

| | "" | r | o | s |
|---|---|---|---|---|
| "" | 0 | 1 | 2 | 3 |
| h | 1 | 1 | 2 | 3 |
| o | 2 | 2 | 1 | 2 |
| r | 3 | 2 | 2 | 2 |
| s | 4 | 3 | 3 | 2 |
| e | 5 | 4 | 4 | **3** |

At (h,r): mismatch → `1+min(dp[0][0],dp[0][1],dp[1][0]) = 1+min(0,1,1)=1`. At (o,o): match → `dp[1][1]=1`. At (e,s) — the final cell: mismatch → `1+min(dp[4][2],dp[4][3],dp[5][2]) = 1+min(3,2,4)=3`. Final `dp[5][3]=3`, matching the well-known answer (replace 'h'→'r', delete 'r', delete 'e' — or an equivalent 3-operation sequence).

**Complexity: Time O(m×n), Space O(m×n)**, optimizable to O(min(m,n)).

**Edge cases:** one string empty → answer equals the other string's full length (all inserts or all deletes), correctly given by the base-case row/column. Identical strings → `0`. Completely disjoint character sets → every position needs a replace, giving `max(m,n)` typically (with any length difference needing inserts/deletes on top).

> 💡 **Interview Insight:** this is one of the most common "compute the value, then reconstruct the actual operations" follow-ups in tier-1 interviews — being ready to say "backtrack from `dp[m][n]`, checking which of the three source cells the current value could have come from" (the same reconstruction idea LIS's O(n log n) approach needed a parent-pointer array for, Day 85) shows the DP-table-as-reconstructable-history mental model is solid, not just the forward computation.

---

## Problem 3: Longest Palindromic Substring (LC 5, Medium) — Pattern: Expand Around Center

**Statement:** Given a string, return its longest palindromic **substring** (contiguous, unlike a subsequence).

### Approach 1 — Brute force: check every substring

```java
public static String longestPalindromeBruteForce(String s) {
    int n = s.length();
    String longest = "";
    for (int i = 0; i < n; i++) {
        for (int j = i; j < n; j++) {
            String candidate = s.substring(i, j + 1);
            if (isPalindrome(candidate) && candidate.length() > longest.length()) {
                longest = candidate;
            }
        }
    }
    return longest;
}

private static boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left++) != s.charAt(right--)) return false;
    }
    return true;
}
```

**Complexity: Time O(n³)** — O(n²) substrings, each palindrome-checked in O(n). **Space O(n)** for the current candidate substring.

### Approach 2 — Optimized: Expand Around Center

```java
public static String longestPalindrome(String s) {
    if (s == null || s.length() < 1) return "";
    int start = 0, maxLength = 1;

    for (int center = 0; center < s.length(); center++) {
        int len1 = expandFromCenter(s, center, center);         // odd-length palindromes: single-char center
        int len2 = expandFromCenter(s, center, center + 1);     // even-length palindromes: between-char center
        int longerLen = Math.max(len1, len2);
        if (longerLen > maxLength) {
            maxLength = longerLen;
            start = center - (longerLen - 1) / 2;
        }
    }
    return s.substring(start, start + maxLength);
}

private static int expandFromCenter(String s, int left, int right) {
    while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
        left--;
        right++;
    }
    return right - left - 1;   // width of the palindrome found (right, left have both stepped one PAST the boundary)
}
```

**Why `2n-1` centers, not `n`:** a palindrome of **odd** length has a single middle character as its center (e.g. `"aba"`'s center is `'b'`) — there are `n` such single-character centers. A palindrome of **even** length has no single middle character — its center sits *between* two characters (e.g. `"abba"`'s center is between the two `'b'`s) — there are `n-1` such between-character gaps. Every palindrome in the string, odd or even length, has exactly one of these `n + (n-1) = 2n-1` centers, so checking all of them (by treating `center` and `center+1` as the two possible "left/right starting points" to expand outward from) is exhaustive — missing either category would silently miss every palindrome of that length parity.

**Worked trace:** `s = "babad"`. Center `i=0` ('b'): odd-expand → "b" alone (len 1). Center `i=1` ('a'): odd-expand → checks 'b','a','b' → matches → "bab" (len 3); further expand would need index -1, stop. Center `i=2` ('b'): odd-expand → checks 'a','b','a' → matches → "aba" (len 3); further would need s[4] vs s[0], stop. Center `i=3` ('a'), `i=4` ('d'): shorter results. Even-centers between each pair: none produce anything longer than 3 for this string. `maxLength` ends at 3, first found at center `i=1` → returns `"bab"` (LeetCode also accepts `"aba"` as an equally valid answer — the problem allows either when a tie exists).

**Complexity: Time O(n²)** — `2n-1` centers, each expansion taking up to O(n) in the worst case (e.g. a string of all identical characters). **Space O(1)** extra (excluding the returned substring itself) — notably *not* O(n²), unlike the DP-table alternative below.

**Edge cases:** single character → itself, length 1. Entire string is one palindrome (e.g. `"aaaa"`) → correctly found via the widest expansion. No palindrome longer than length 1 exists → any single character is returned (the trivial length-1 palindrome always exists).

**Alternate approach, worth naming (not required today):** a DP-table formulation also exists — `isPalin[i][j] = true` if `s[i..j]` is a palindrome, built as `isPalin[i][j] = (s[i]==s[j]) && (j-i<2 || isPalin[i+1][j-1])`. This costs O(n²) time **and** O(n²) space (strictly worse than Expand Around Center's O(1) extra space for this specific problem), but the *table itself* becomes reusable infrastructure — tomorrow's Palindromic Substrings reuses the same expand-around-center idea directly, but this exact DP table is what finally pays off the optimization Palindrome Partitioning (Week 10, Day 66) named and explicitly deferred: an O(1) "is this substring a palindrome" lookup, instead of a fresh check on every call.

> 💡 **Interview Insight:** naming *both* approaches unprompted — "Expand Around Center for O(1) extra space, or a DP table if I'll need repeated palindrome-substring lookups later" — signals awareness that the "optimal" approach depends on what happens *after* this one problem, not just this problem in isolation. There's also Manacher's Algorithm, an O(n) linear-time solution — worth naming as existing if pushed for sub-quadratic, though it's rarely expected to be derived live in an interview.

---

## Day 88 — Interview Questions

---

**1. Why does String DP use a `(m+1)×(n+1)` table instead of `m×n`, over 0-indexed strings?**

*Answer:* Row 0 and column 0 represent an empty prefix on one side — a real, meaningful base case, not a special case. This convention means `s1.charAt(i-1)` is always a valid access whenever the main recurrence body runs (since it only ever runs for `i ≥ 1`), avoiding negative-index checks entirely.

---

**2. In LCS, why does a matching character always take `dp[i-1][j-1]+1` unconditionally, with no comparison needed?**

*Answer:* A character shared by both strings at the current prefix boundary can always be safely included in some LCS of those two prefixes — there's never a scenario where discarding a genuine match produces a strictly better answer, so no comparison against the alternative is needed.

---

**3. In Edit Distance, map each of the three operations (insert, delete, replace) to exactly which neighboring cell it reads from, and explain why.**

*Answer:* Replace reads `dp[i-1][j-1]` — both prefixes shrink together, since replacing the last character makes it match, letting the remaining work be judged on the prefixes before it. Delete reads `dp[i-1][j]` — only `s1`'s prefix shrinks, since a character was removed from `s1`. Insert reads `dp[i][j-1]` — only `s2`'s prefix effectively shrinks (from `s2`'s perspective, one more of its characters has now been accounted for).

---

**4. Delete Operation for Two Strings reduces directly to LCS — state the reduction and justify it.**

*Answer:* `answer = (len1 - lcs) + (len2 - lcs)`. The LCS can remain untouched in both strings; every other character, by definition, isn't part of any shared subsequence and must be deleted from wherever it appears. Since the LCS is the *longest* possible shared subsequence, this minimizes total deletions — any smaller number of deletions would have to leave behind a longer common subsequence than the LCS, which is impossible by definition.

---

**5. Why does Longest Palindromic Substring's Expand Around Center need `2n-1` centers?**

*Answer:* Odd-length palindromes center on a single character (`n` such centers); even-length palindromes center on a gap between two characters (`n-1` such gaps, since there are `n-1` adjacent pairs in a string of length `n`). Together, `n + (n-1) = 2n-1` centers cover every possible palindrome regardless of length parity — checking only `n` centers would silently miss every even-length palindrome.

---

**6. Compare Expand Around Center's space complexity to the DP-table alternative for the same problem, and state when the DP table is worth its extra cost anyway.**

*Answer:* Expand Around Center uses O(1) extra space; the DP table uses O(n²). The table is worth building anyway when repeated "is this substring a palindrome" queries are needed elsewhere (as with Palindrome Partitioning, Week 10 Day 66, whose brute-force palindrome check this table is built specifically to replace) — a single O(n²) precomputation replacing many repeated O(n) checks.

---

## Daily Deliverable Check

- [ ] Longest Common Subsequence, Edit Distance, and Longest Palindromic Substring all solved and pushed to `dsa-java/dynamic-programming/`.
- [ ] Delete Operation for Two Strings (extra) solved as a direct LCS application.
- [ ] Can state the `(m+1)×(n+1)` table convention and why it avoids negative-index checks, from memory.
- [ ] Can map each of Edit Distance's three operations to its source cell without looking it up.
- [ ] Can explain the `2n-1` centers requirement for Expand Around Center, unprompted.

---

## What Tomorrow Assumes You Already Know Cold

Day 89 continues directly from today's Expand Around Center technique (Palindromic Substrings reuses it to count instead of track-longest) and from today's LCS recurrence (Longest Palindromic Subsequence is explicitly "the LCS of the string and its own reverse — reuse that logic directly, no new recurrence needed"). Both citations assume today's two techniques are solid enough to reuse without re-deriving. Interleaving String (also tomorrow) extends the two-pointer-prefix table idea to *three* strings at once — a genuinely new twist on the `(m+1)×(n+1)` convention that tomorrow's book builds from today's foundation rather than restating it.
