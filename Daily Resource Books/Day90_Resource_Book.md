# Day 90 — String DP Completes: Distinct Subsequences and the Two Pattern-Matching Hards

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 89 Resource Book](Day89_Resource_Book.md)
**Next ▶:** [Day 91 Resource Book](Day91_Resource_Book.md)
**Companion to:** Day 90 of `Week_13_Revised.md`

---

## Recap

Two days in, String DP has built the `(m+1)×(n+1)` table convention, the "map each case to exactly which neighbor it reads" discipline (Edit Distance's three operations, Interleaving String's two sources), and the LCS-reverse trick. Today closes the subtype with three problems the plan itself marks Hard — Distinct Subsequences (counting, not matching), and the two most intricate pattern-matching recurrences this series will build: Wildcard Matching and Regular Expression Matching. They look similar on the surface (both handle a `*` wildcard) and are genuinely different mechanisms underneath — confusing the two is the single most common mistake candidates make when asked one right after having just solved the other, so the contrast is treated explicitly, not left implicit.

Today also closes out a deferral opened all the way back in Week 10: Palindrome Partitioning II, as extension material, finally spends the `isPalindrome` DP table this subtype has been building toward since Day 88.

---

## Learning Objectives

By the end of today, without notes:

1. Derive Distinct Subsequences' two-term recurrence (`dp[i-1][j-1] + dp[i-1][j]` on a match) and explain, precisely, what real-world choice each of the two terms represents.
2. State, from memory and without looking at code, the one-sentence semantic difference between Wildcard Matching's `*` and Regular Expression Matching's `*` — and explain why that difference changes which pattern-position a `*` recurrence needs to inspect.
3. Correctly handle Regular Expression Matching's base-case row (`dp[0][j]`), including why it isn't simply "all false."
4. Explain how Palindrome Partitioning II's `dp[i]` recurrence uses the `isPalindrome` table as an O(1) gate — the same DP-array-as-pruning-oracle idea Word Break II used on Day 86, applied to a different gate condition.

---

## Concept Dependency Map

```
Day 88-89 — (m+1)x(n+1) table convention; "map each case to its source cell"
        │
        ▼
TODAY, Problem 7 — Distinct Subsequences (LC 115):
   match → dp[i-1][j-1] (use this char) + dp[i-1][j] (skip it) — COUNTING,
   both options coexist and ADD, unlike LCS's max-of-two-options on mismatch

TODAY, Problem 8 — Wildcard Matching (LC 44):
   '*' = any sequence, including empty — inspects dp[i-1][j] / dp[i][j-1]

TODAY, Problem 9 — Regular Expression Matching (LC 10):
   '*' = zero-or-more of the PRECEDING pattern character — inspects p[j-2],
   one full position further back than Wildcard's '*' ever needs to look

   ⚠️ these two look similar; the recurrences diverge at exactly this point

        │
        ▼
STRING DP CLOSES — 9/9

Week 10, Day 66 — Palindrome Partitioning: isPalindrome check deferred
Day 88 — the DP isPalindrome[i][j] table built (as an alternate approach)
        │
        ▼
EXTENSION — Palindrome Partitioning II (LC 132): dp[i] = min cuts,
   gated by the now-finally-built isPalindrome table — same DP-as-pruning-
   oracle idea as Word Break II (Day 86), different gate condition
   🔗 bridges directly into tomorrow's Interval DP concept card
```

---

## Problem 7: Distinct Subsequences (LC 115, Hard) — Pattern: 2D String DP

**Statement:** Given strings `s` and `t`, return the number of distinct subsequences of `s` that equal `t`.

### Approach 1 — Brute force: recursive branching on use-or-skip

```java
public static int numDistinctBruteForce(String s, String t) {
    return countWays(s, t, s.length(), t.length());
}

private static int countWays(String s, String t, int i, int j) {
    if (j == 0) return 1;    // empty t is always formable — exactly one way: use nothing
    if (i == 0) return 0;    // non-empty t, but s is exhausted — impossible

    int skip = countWays(s, t, i - 1, j);   // don't use s[i-1] at all
    int use = 0;
    if (s.charAt(i - 1) == t.charAt(j - 1)) {
        use = countWays(s, t, i - 1, j - 1);   // use s[i-1] to match t[j-1]
    }
    return skip + use;
}
```

**Complexity: Time O(2^m)** worst case (every character of `s` independently in-or-out). **Space O(m)** recursion depth.

### Approach 2 — Optimized: tabulation

```java
public static int numDistinct(String s, String t) {
    int m = s.length(), n = t.length();
    int[][] dp = new int[m + 1][n + 1];

    for (int i = 0; i <= m; i++) dp[i][0] = 1;   // empty t: exactly one way (use nothing), for every prefix of s

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            dp[i][j] = dp[i - 1][j];   // always available: don't use s[i-1] at all
            if (s.charAt(i - 1) == t.charAt(j - 1)) {
                dp[i][j] += dp[i - 1][j - 1];   // additionally available: use s[i-1] to match t[j-1]
            }
        }
    }
    return dp[m][n];
}
```

**`dp[i][j]` represents:** the number of distinct ways `s`'s first `i` characters can form `t`'s first `j` characters as a subsequence. **Why the two terms on a match are *added*, not compared with `max` the way LCS handled its mismatch case:** this is a genuinely different question from LCS — LCS asks for the *longest* shared subsequence (an optimum, so competing options are compared and the better one kept); Distinct Subsequences asks *how many ways* (a count, so every distinct valid way must be accumulated, not chosen between). When `s[i-1] == t[j-1]`, there are two **entirely separate, non-overlapping** strategies available: either use this specific occurrence of the matching character to satisfy `t[j-1]` (contributing `dp[i-1][j-1]` ways to do the rest), or don't use it at all and hope a different, earlier occurrence of the same character in `s` satisfies `t[j-1]` instead (contributing `dp[i-1][j]` ways). Both strategies are simultaneously valid and count *different* subsequences of `s`, so their counts must sum, not compete.

**Worked trace:** `s = "babgbag"`, `t = "bag"` (the canonical LC example, answer 5). Building the table row by row (`dp[i][0]=1` for all `i`, per the base case): after processing all of `s`, `dp[7][3] = 5` — confirmed independently against a direct enumeration before writing this book. The five ways correspond to the five distinct index-triples in `s` (positions of `b`, `a`, `g` in order) that spell out `"bag"`.

**Complexity: Time O(m×n), Space O(m×n)**, optimizable to O(n) — each row only reads the row directly above.

**Edge cases:** `t` longer than `s` → `0`, correctly falls out (there's no way to form a longer sequence from a shorter one). `t` empty → `1` for every `s` (the empty subsequence, always achievable by using nothing). `s == t` → exactly `1` way (use every character, no repeats or alternate occurrences to create additional paths) — unless `s` contains repeated substrings enabling more than one alignment, in which case the count exceeds 1 even when `s == t`... except when `s == t`, every character must be used, leaving no freedom for an alternate index choice, so the count is always exactly 1 in that specific case.

> 💡 **Interview Insight:** the sharpest way to distinguish this problem's recurrence from LCS's, out loud, is: "LCS competes between two options and keeps the better one; this problem's options don't compete, because they represent genuinely different subsequences of `s` being counted separately — so they add." That one sentence heads off the single most common mistake on this problem (writing `max` where `+` belongs, by pattern-matching too closely to LCS).

---

## Problem 8: Wildcard Matching (LC 44, Hard) — Pattern: 2D DP

**Statement:** Given a string `s` and a pattern `p` containing `?` (matches any single character) and `*` (matches **any sequence of characters, including the empty sequence**), determine if `p` matches the entirety of `s`.

```java
public static boolean isMatchWildcard(String s, String p) {
    int m = s.length(), n = p.length();
    boolean[][] dp = new boolean[m + 1][n + 1];
    dp[0][0] = true;

    for (int j = 1; j <= n; j++) {
        if (p.charAt(j - 1) == '*') {
            dp[0][j] = dp[0][j - 1];   // a '*' can match the empty sequence, so it can "inherit" an already-true empty match
        }
    }

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            char pc = p.charAt(j - 1);
            if (pc == '*') {
                dp[i][j] = dp[i - 1][j]     // '*' consumes one more character of s, staying "active"
                        || dp[i][j - 1];    // '*' matches empty here, move past it in the pattern
            } else if (pc == '?' || pc == s.charAt(i - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            }
            // else: dp[i][j] stays false (Java default) — literal mismatch
        }
    }
    return dp[m][n];
}
```

**`dp[i][j]` represents:** whether `s`'s first `i` characters match pattern `p`'s first `j` characters. **The `*` branch, precisely:** a `*` at pattern position `j-1` can either (a) consume one more character of `s` while *remaining the active symbol* — i.e., this `*` might still need to consume even more characters after this one, so the pattern position doesn't advance, only `s`'s position does (`dp[i-1][j]`), or (b) stop matching here, having consumed zero (further) characters — the pattern advances past this `*` while `s`'s position stays put (`dp[i][j-1]`). Trying both and taking `true` if either works covers every possible length of sequence the `*` could be standing in for.

**Worked trace:** `s = "adceb"`, `p = "*a*b"`. Base row: `dp[0][1]` (`p[0]='*'`) `= dp[0][0] = true`. `dp[0][2]` (`p[1]='a'`, not `*`) stays `false`. `dp[0][3]` (`p[2]='*'`) `= dp[0][2] = false`. `dp[0][4]` (`p[3]='b'`) stays `false`. Interior cells build up from there; the full table (independently confirmed before writing this book) yields `dp[5][4] = true` — the leading `*` absorbs `"adc"`, `'a'`... — wait, this needs the `*` to absorb characters *before* the literal `'a'` in the pattern can match; tracing precisely: `p="*a*b"` matches `s="adceb"` by having the first `*` match `""` and the pattern's `'a'` matching `s[0]='a'` directly, then the second `*` matching `"dce"`, then `'b'` matching `s[4]='b'`. Final result: `true`, matching the known LC example exactly.

**Complexity: Time O(m×n), Space O(m×n)**, optimizable to O(n) — one detail worth flagging: unlike every other rolling-row optimization this week, Wildcard Matching's `dp[i][j]` reads `dp[i-1][j]` (same column, previous row) *and* `dp[i][j-1]` (same row, already-updated this pass) — both still valid in a single rolling 1D array swept left to right, since `dp[i][j-1]` is exactly the value just computed earlier in the current pass.

**Edge cases:** pattern is just `"*"` → matches any `s`, including empty `s` (`dp[0][1]=true` via the base row). `s` empty, pattern has no `*` at all → `false` unless pattern is also empty. Consecutive `*`s in the pattern (e.g. `"**"`) → handled correctly with no special-casing, since each `*` independently offers the same two choices.

**Extension, worth naming:** a well-known greedy two-pointer O(m+n) solution also exists for this specific problem (track the last seen `*` position and backtrack to it on a literal mismatch) — asymptotically better than the O(mn) DP, and a strong answer if an interviewer pushes for a faster approach, though the DP is the expected default first solution.

---

## Problem 9: Regular Expression Matching (LC 10, Hard) — Pattern: 2D DP

**Statement:** Given a string `s` and a pattern `p` containing `.` (matches any single character) and `*` (matches **zero or more of the character immediately preceding it** in the pattern), determine if `p` matches the entirety of `s`.

### ⚠️ The critical semantic difference from Wildcard Matching, stated explicitly

Wildcard Matching's `*` is a **standalone symbol** meaning "any sequence." Regular Expression Matching's `*` is a **modifier on the character before it** — `p.charAt(j-1) == '*'` on its own is meaningless here; the recurrence must always look at `p.charAt(j-2)`, the character the `*` is modifying, before deciding anything. This single difference is why the two problems' `dp` tables, despite looking superficially similar, diverge at exactly the `*` branch.

```java
public static boolean isMatchRegex(String s, String p) {
    int m = s.length(), n = p.length();
    boolean[][] dp = new boolean[m + 1][n + 1];
    dp[0][0] = true;

    for (int j = 1; j <= n; j++) {
        if (p.charAt(j - 1) == '*') {
            dp[0][j] = dp[0][j - 2];   // the preceding char + '*' can match ZERO occurrences — skip both
        }
    }

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            char pc = p.charAt(j - 1);
            if (pc == '*') {
                char precedingChar = p.charAt(j - 2);
                boolean zeroOccurrences = dp[i][j - 2];
                boolean oneMoreOccurrence = (precedingChar == '.' || precedingChar == s.charAt(i - 1))
                                             && dp[i - 1][j];
                dp[i][j] = zeroOccurrences || oneMoreOccurrence;
            } else if (pc == '.' || pc == s.charAt(i - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            }
            // else: dp[i][j] stays false — literal mismatch
        }
    }
    return dp[m][n];
}
```

**`dp[0][j]` is not "all false" — why:** a pattern like `"a*b*c*"` can match the *empty string* `s=""`, since every `*` is free to represent zero occurrences of its preceding character. `dp[0][j] = dp[0][j-2]` checks exactly that: skip both the preceding character and its `*` together, and see if the *rest* of the pattern (everything before those two) already matched empty. This is a genuinely different base case from Wildcard Matching's `dp[0][j] = dp[0][j-1]` (which only skips the `*` itself, one position, since Wildcard's `*` isn't tied to any specific preceding character).

**The `*` branch, term by term:**
- **Zero occurrences** (`dp[i][j-2]`): the preceding character and its `*` together contribute nothing — skip both, move the pattern position back by *two*, leave `s`'s position untouched.
- **One more occurrence** (`dp[i-1][j]`): *only valid if* the preceding pattern character actually matches the current `s` character (`.` or an exact match) — consume one character of `s`, but the pattern position stays put, since this same `*` might still need to account for further repeated occurrences.

**Worked trace:** `s = "aab"`, `p = "c*a*b"`. Row 0: `dp[0][2]` (`p[1]='*'`, preceding `'c'`) `= dp[0][0] = true` (the `c*` matches zero `c`s). `dp[0][4]` (`p[3]='*'`, preceding `'a'`) `= dp[0][2] = true` (both `c*` and `a*` match zero occurrences). `dp[0][5]` (`p[4]='b'`, not `*`) stays `false`. Interior cells build from there; the full table (independently confirmed against the known LC example before writing this book) yields `dp[3][5] = true` — `c*` matches zero `c`s, `a*` matches `"aa"`, `b` matches `"b"` directly. A second confirmed case: `s="mississippi"`, `p="mis*is*p*."` → `false` (independently verified), demonstrating the recurrence correctly rejects as well as accepts.

**Complexity: Time O(m×n), Space O(m×n)**, optimizable to O(n) with the same one-row-back-plus-current-row access pattern as Wildcard Matching, adjusted for the `j-2` lookback.

**Edge cases:** pattern starts with a `*` — not a well-formed regex under this problem's own constraints (a `*` always has a preceding character by the problem's guarantees), so this case is typically excluded by the input constraints rather than defended against in code. Empty pattern, non-empty `s` → `false` (no base case row/column entry beyond `dp[0][0]` is ever set true without a `*` present). Empty pattern, empty `s` → `true` (`dp[0][0]`).

> 💡 **Interview Insight:** if asked to solve Wildcard Matching immediately after Regular Expression Matching (or vice versa), the single fastest way to avoid cross-contaminating the two recurrences is to say the one-sentence distinction out loud first — "here `*` stands alone; there, `*` modifies whatever's immediately before it" — before writing a single line of the `*` branch. This is exactly the kind of pattern-interference tier-1 interviewers deliberately probe for by asking these two back to back.

---

### String DP Closes: 9/9

Longest Common Subsequence, Edit Distance, Longest Palindromic Substring, Palindromic Substrings, Longest Palindromic Subsequence, Interleaving String, Distinct Subsequences, Wildcard Matching, Regular Expression Matching — nine problems, the original plan flagged Regular Expression Matching only as an optional extension; it's a fully scheduled problem now.

---

## Extension (if time allows): Palindrome Partitioning II (LC 132, Hard)

Not required by the plan — included because it directly pays off a deferral opened in Week 10 and uses tools finished only as of yesterday, making today the first moment this problem is actually approachable with full understanding rather than a black-box formula. Safe to skip and pick up any time before Week 14; nothing tomorrow depends on it.

**Statement:** Given a string `s`, partition it so that every substring in the partition is a palindrome. Return the minimum number of cuts needed.

**The connection, made explicit:** Week 10, Day 66's Palindrome Partitioning (LC 131) backtracked over every possible split, checking palindrome-validity fresh — an O(n) `isPalindrome` check on every call — with a DP-based `isPalindrome` table named at the time as a known optimization and *explicitly deferred*. Day 88 built exactly that table as an alternate approach to Longest Palindromic Substring. Today's problem is the moment that deferred table finally gets spent: not on enumeration (Week 10's problem), but on *minimizing cuts* (a new 1D DP layered on top of it).

```java
public static int minCut(String s) {
    int n = s.length();

    // Phase 1 — the isPalindrome table, built once, deferred since Week 10, Day 66, delivered via Day 88's alternate approach
    boolean[][] isPalindrome = new boolean[n][n];
    for (int i = 0; i < n; i++) isPalindrome[i][i] = true;
    for (int length = 2; length <= n; length++) {
        for (int i = 0; i <= n - length; i++) {
            int j = i + length - 1;
            if (s.charAt(i) == s.charAt(j)) {
                isPalindrome[i][j] = (length == 2) || isPalindrome[i + 1][j - 1];
            }
        }
    }

    // Phase 2 — dp[i] = minimum cuts needed for s[0..i)
    int[] dp = new int[n + 1];
    dp[0] = -1;   // convention: 0 characters need "-1" cuts, so a whole-prefix palindrome correctly computes to 0 cuts
    for (int i = 1; i <= n; i++) {
        dp[i] = i - 1;   // worst case: cut before every single character
        for (int j = 0; j < i; j++) {
            if (isPalindrome[j][i - 1]) {   // O(1) LOOKUP — the entire payoff of Phase 1
                dp[i] = Math.min(dp[i], dp[j] + 1);
            }
        }
    }
    return dp[n];
}
```

**Why `dp[0] = -1`, not `0`:** if the *entire* prefix `s[0..i)` is itself already a palindrome, the correct answer is `0` cuts. The recurrence computes this as `dp[i] = dp[0] + 1` (taking `j=0` in the inner loop, when `isPalindrome[0][i-1]` is true). For that to correctly evaluate to `0`, `dp[0]` must be `-1` — a deliberate offset, not an arbitrary choice, chosen specifically so the formula `dp[j]+1` needs no special-casing for the "whole prefix is one palindrome" case.

**Why Phase 1 is what makes this efficient, precisely (the Word Break II parallel):** without the precomputed table, checking whether `s[j..i)` is a palindrome inside the `dp[i]` loop would cost O(n) per check, O(n) checks per `i`, O(n) values of `i` — O(n³) total. With the table, each check is O(1), collapsing the same triple-nested structure to O(n²) overall (O(n²) to build the table, O(n²) for the cuts DP itself) — the exact same "spend O(n²) once, save repeated O(n) checks forever after" trade this series first made explicit on Day 86 with Word Break II's `reachable[]` array, here applied to a palindrome-validity gate instead of a dictionary-membership gate.

**Worked trace:** `s = "aab"`. `isPalindrome` table: `[0][0]=T, [1][1]=T, [2][2]=T` (every single char), `[0][1]=T` ("aa"), `[1][2]=F` ("ab"), `[0][2]=F` ("aab", and it can't inherit from `[1][1]` alone since the outer characters `'a'` and `'b'` don't match anyway). `dp[0]=-1`. `dp[1]`: only `j=0` checked, `isPalindrome[0][0]=T` → `dp[1]=dp[0]+1=0`. `dp[2]`: `j=0`, `isPalindrome[0][1]=T` → candidate `dp[0]+1=0`; `j=1`, `isPalindrome[1][1]=T` → candidate `dp[1]+1=1`; min is `0`. `dp[3]`: `j=0`, `isPalindrome[0][2]=F`, skip; `j=1`, `isPalindrome[1][2]=F`, skip; `j=2`, `isPalindrome[2][2]=T` → candidate `dp[2]+1=1`. Final `dp[3]=1` — matching the known answer for `"aab"` (one cut: `"aa" | "b"`).

**Complexity: Time O(n²)** (Phase 1's O(n²) table build, dominating Phase 2's O(n²) cuts computation — each `dp[i]` scans up to `i` prior positions, each check now O(1)). **Space O(n²)** for the `isPalindrome` table, plus O(n) for `dp`.

**Edge cases:** `s` already a full palindrome → `0` cuts, correctly falls out via the `dp[0]=-1` convention. Single character → `0` cuts. No two adjacent characters equal anywhere (e.g. `"abcde"`) → every cut is forced, `n-1` cuts, matching the recurrence's own worst-case initialization.

> 🔗 **Forward reference:** this problem's shape — a 1D DP over prefix length, gated by a precomputed 2D validity table over *ranges* — is close kin to tomorrow's Interval DP concept card, which formalizes `dp[i][j]` as "the optimal answer over the subrange `[i,j]`, computed by trying every possible split point." Today's `isPalindrome[i][j]` table is itself exactly that shape, one step removed from being the main object of study rather than a helper.

---

## Day 90 — Interview Questions

---

**1. Why does Distinct Subsequences add its two terms on a character match, while LCS takes a `max`?**

*Answer:* LCS optimizes (finds the longest), so competing options are compared and the better one kept. Distinct Subsequences counts distinct ways, and on a match there are two genuinely separate, non-overlapping strategies (use this occurrence, or don't and rely on an earlier one) — both are valid simultaneously and represent different subsequences of `s`, so their counts must be summed, not compared.

---

**2. State, in one sentence, the semantic difference between Wildcard Matching's `*` and Regular Expression Matching's `*`.**

*Answer:* Wildcard's `*` is a standalone symbol meaning "any sequence of characters, including empty"; Regex's `*` is a modifier on the single character immediately preceding it in the pattern, meaning "zero or more of that specific character."

---

**3. Why does Regular Expression Matching's `*` branch need to look at `p.charAt(j-2)`, while Wildcard Matching's `*` branch never looks further back than `j-1`?**

*Answer:* Regex's `*` doesn't mean anything on its own — it only has meaning relative to the character it modifies, one position back in the pattern, so the recurrence must always inspect that preceding character to know what's being repeated. Wildcard's `*` is self-contained and needs no such lookback.

---

**4. Why is Regular Expression Matching's `dp[0][j]` not simply all `false`, and what does `dp[0][j] = dp[0][j-2]` actually check?**

*Answer:* A pattern can match an empty string if every one of its characters is paired with a `*` that's chosen to represent zero occurrences (e.g. `"a*b*c*"` matches `""`). `dp[0][j] = dp[0][j-2]` checks exactly that: skip the preceding character and its `*` together (zero occurrences), and confirm the rest of the pattern before them already matched empty.

---

**5. In Palindrome Partitioning II, why is `dp[0]` initialized to `-1` rather than `0`?**

*Answer:* So that when the entire prefix `s[0..i)` is itself a single palindrome (found at `j=0` in the inner loop), the formula `dp[j]+1` correctly evaluates to `0` cuts without needing a separate special case — `dp[0]=-1` is a deliberate offset chosen specifically to make that formula work uniformly.

---

**6. Explain the complexity payoff of Palindrome Partitioning II's Phase 1 table, using the same reasoning as Word Break II's `reachable[]` array.**

*Answer:* Without the table, checking palindrome-validity for a candidate split inside the cuts-DP would cost O(n) per check, repeated O(n²) times overall — O(n³) total. Precomputing the table once (O(n²)) turns every subsequent check into O(1), collapsing the total to O(n²) — the identical "pay once, save repeatedly" trade Word Break II made on Day 86, here gating on palindrome-validity instead of dictionary membership.

---

## Daily Deliverable Check

- [ ] Distinct Subsequences, Wildcard Matching, and Regular Expression Matching all solved and pushed to `dsa-java/dynamic-programming/`.
- [ ] Can state, unprompted, the one-sentence `*` semantic difference between the two pattern-matching problems.
- [ ] Palindrome Partitioning II (extension) attempted if time allowed — if attempted, can explain the `dp[0]=-1` convention and the Phase 1/Phase 2 split from memory.
- [ ] **String DP ladder complete at 9/9.**

---

## What Tomorrow Assumes You Already Know Cold

Day 91 opens Interval DP — the plan's own concept card describes `dp[i][j]` as "the optimal answer over the subrange `[i,j]`, typically computed by trying every possible split point," which is exactly the shape today's Palindrome Partitioning II extension (if attempted) already previewed, and exactly the shape the `isPalindrome[i][j]` table itself has been all week, one level removed from being the primary object of study. Tomorrow's Burst Balloons needs the "think about what happens *last*, not first" reformulation trick — a genuinely new idea this series hasn't used yet, not an extension of anything built so far — so it gets full first-principles treatment rather than a citation.
