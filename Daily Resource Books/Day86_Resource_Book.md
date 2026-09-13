# Day 86 — 1D DP Completes: Target Sum, Perfect Squares, and Word Break II's Backtracking Fusion

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 85 Resource Book](Day85_Resource_Book.md)
**Next ▶:** [Day 87 Resource Book](Day87_Resource_Book.md)
**Companion to:** Day 86 of `Week_13_Revised.md`

---

## Recap

Yesterday built 0/1 Knapsack from zero — Partition Equal Subset Sum, plus the decreasing-loop proof that keeps items from being reused. Today opens with Target Sum, which the plan itself flags as needing no new recurrence at all: a two-equation reduction lands it on *exactly* yesterday's subset-sum shape, now counting instead of a yes/no. Perfect Squares does the same move on the *other* side of the week — same unbounded-reuse shape as Coin Change (Day 84), different "coins." Both are, deliberately, low-new-content days on the concept side, precisely so today's real weight can go to Word Break II — the first problem this series has posed where two previously-*distinguished* patterns (Dynamic Programming and Backtracking, explicitly contrasted with each other back in Week 9, Day 61) now combine in one solution.

This closes 1D DP entirely at 14 problems — up from the original plan's 12, with two "nice to have" additions (Perfect Squares, Word Break II) now fully scheduled rather than optional.

---

## Learning Objectives

By the end of today, without notes:

1. Derive Target Sum's `P = (target + totalSum) / 2` reduction from the two defining equations, unprompted, and state both edge cases that make an input unsolvable before any DP runs.
2. Recognize Perfect Squares as Coin Change's exact shape with a different "coin list," and generate that coin list (perfect squares ≤ n) correctly at the code level.
3. Explain precisely how Word Break II uses Word Break's `dp[]` boolean array to prune backtracking — not just "it's faster," but which specific branches get cut and why exploring them would otherwise be wasted work.
4. Complete, from memory, the full loop-order comparison across all four DP-with-a-nested-loop problems seen this week (Coin Change, Coin Change II, Combination Sum IV, 0/1 Knapsack) and state which of two independent reasons (bounded-vs-unbounded, or combinations-vs-permutations) drives each rule.

---

## Concept Dependency Map

```
Day 85 — Partition Equal Subset Sum: 0/1 Knapsack, yes/no reachability
        │
        ▼
TODAY, Problem 12 — Target Sum (LC 494): SAME shape, counting instead of yes/no
        (P-N=target, P+N=totalSum → P=(target+totalSum)/2, then count subsets = P)

Day 84 — Coin Change: Unbounded Knapsack, minimize coin count
        │
        ▼
TODAY, Problem 13 — Perfect Squares (LC 279): SAME shape, "coins" = perfect squares

Day 84 — Word Break (LC 139): dp[i] = can s[0..i) be segmented? (boolean, dictionary-gated)
Week 9, Day 61 — Backtracking formalized: choose→explore→un-choose,
   explicitly DISTINGUISHED from DP (enumerate-everything vs. exploit-overlap)
Week 10, Day 66 — Palindrome Partitioning (LC 131): backtrack over string splits
        │
        ▼
TODAY, Problem 14 — Word Break II (LC 140): the two patterns COMBINE —
   Word Break's dp[] gates which backtracking branches are even worth exploring

        │
        ▼
1D DP CLOSES — 14/14. Next new DP subtype: Grid DP, tomorrow.
```

---

## Problem 12: Target Sum (LC 494, Medium) — Pattern: 0/1 Knapsack DP

**Statement:** Given an integer array `nums` and an integer `target`, assign a `+` or `-` sign to every number so the resulting expression evaluates to `target`. Return the number of ways to do this.

### Approach 1 — Brute force: try both signs at every index

```java
public static int findTargetSumWaysBruteForce(int[] nums, int target) {
    return assignSigns(nums, 0, 0, target);
}

private static int assignSigns(int[] nums, int index, int currentSum, int target) {
    if (index == nums.length) {
        return currentSum == target ? 1 : 0;
    }
    return assignSigns(nums, index + 1, currentSum + nums[index], target)
         + assignSigns(nums, index + 1, currentSum - nums[index], target);
}
```

Every index independently branches into `+` or `-` — a full binary decision tree, 2ⁿ leaves. **Complexity: Time O(2ⁿ), Space O(n)** recursion depth.

### Approach 2 — Optimized: reduce to 0/1 Knapsack counting

**The reduction, derived, not just stated:** split `nums` into two implicit groups — `P`, the subset assigned `+`, and `N`, the subset assigned `-`. By definition, `P - N = target` (the expression's value) and `P + N = totalSum` (every number is in exactly one group). Two linear equations, two unknowns — add them: `2P = target + totalSum`, so:

```
P = (target + totalSum) / 2
```

The problem is now: **how many subsets of `nums` sum to exactly `P`?** — the *counting* version of yesterday's 0/1 Knapsack reachability check, same decreasing-loop mechanism, `dp[j] += dp[j-num]` instead of `dp[j] = dp[j] || dp[j-num]`.

```java
public static int findTargetSumWays(int[] nums, int target) {
    int totalSum = 0;
    for (int num : nums) totalSum += num;

    // Unsolvable cases, checked BEFORE any DP work:
    if ((target + totalSum) % 2 != 0) return 0;   // P wouldn't be an integer
    if (Math.abs(target) > totalSum) return 0;     // target unreachable even using every number as +

    int P = (target + totalSum) / 2;
    if (P < 0) return 0;   // guards the case totalSum + target is negative-odd in a way P%2 alone wouldn't catch

    int[] dp = new int[P + 1];
    dp[0] = 1;
    for (int num : nums) {
        for (int j = P; j >= num; j--) {   // decreasing — Day 85's proof, reused directly
            dp[j] += dp[j - num];
        }
    }
    return dp[P];
}
```

**Why both guard conditions are necessary, not defensive-but-redundant:** `(target + totalSum) % 2 != 0` catches the case where `P` isn't a whole number at all (e.g. `totalSum=5, target=2` → `P=3.5`, impossible — no assignment of signs to integers can produce a non-integer split). `Math.abs(target) > totalSum` catches a case the parity check alone would miss: even when `target + totalSum` happens to be even, if `|target|` exceeds `totalSum`, no valid split exists because the most extreme achievable sum is `+totalSum` (everything `+`) or `-totalSum` (everything `-`) — anything beyond that range is simply unreachable regardless of parity.

**Worked trace:** `nums = [1,1,1,1,1]`, `target = 3` (the canonical LC example). `totalSum = 5`. `(3+5)%2 = 0` ✓. `P = 8/2 = 4`. Question becomes: how many subsets of five `1`s sum to exactly `4`? That's choosing 4 of the 5 ones — `C(5,4) = 5` ways. Running the DP: after all five `1`s are processed, `dp[4] = 5` (numerically confirmed against a full 2⁵ = 32-assignment brute-force enumeration before writing this book). This matches the known answer of 5 for this exact input.

**Complexity: Time O(n × P), Space O(P)** — identical shape to yesterday's 0/1 Knapsack, `P` in place of `target`.

**Edge cases:** a `0` in `nums` — assigning it `+` or `-` produces the *same* sum either way, so every `0` present doubles the number of ways to reach any given target (confirmed directly: eight zeros plus a single `1`, target `1`, gives `256 = 2⁸` ways — one binary choice per zero, all irrelevant to the sum, all still counted as distinct sign-assignments per the problem's own definition). `target` larger in magnitude than `totalSum` → `0`, caught by the guard before DP even starts.

> 💡 **Interview Insight:** this problem's entire difficulty is the algebraic reduction, not the DP itself — an interviewer watching a candidate immediately reach for two-variable elimination (`P-N=target`, `P+N=totalSum`) rather than fumbling toward it by trial and error is seeing exactly the "reduce to a known shape" instinct this series has been building since Minimum Insertion Steps was previewed as reusing Longest Palindromic Subsequence, and since Isomorphic Strings needed two maps instead of one back in Week 1.

---

## Problem 13: Perfect Squares (LC 279, Medium) — Pattern: 1D DP (Unbounded Knapsack)

**Statement:** Given an integer `n`, return the minimum number of perfect square numbers (`1, 4, 9, 16, ...`) that sum to `n`.

### Approach 1 — Brute force: recursion over every perfect square ≤ remaining

```java
public static int numSquaresBruteForce(int n) {
    if (n == 0) return 0;
    int minCount = Integer.MAX_VALUE;
    for (int i = 1; i * i <= n; i++) {
        int subResult = numSquaresBruteForce(n - i * i);
        if (subResult != Integer.MAX_VALUE) {
            minCount = Math.min(minCount, subResult + 1);
        }
    }
    return minCount;
}
```

At every value, try every perfect square not exceeding it, recurse on the remainder. **Complexity: exponential** — the same shape as Coin Change's brute force, with `√n` "coin" choices instead of a fixed small coin list, so the branching factor itself grows with `n`.

### Approach 2 — Optimized: tabulation, structurally identical to Coin Change

```java
public static int numSquares(int n) {
    int[] dp = new int[n + 1];
    Arrays.fill(dp, n + 1);   // sentinel: n+1 can never be a real answer (worst case is n×"1"s), Day 84's exact trick
    dp[0] = 0;

    for (int i = 1; i <= n; i++) {
        for (int square = 1; square * square <= i; square++) {
            dp[i] = Math.min(dp[i], dp[i - square * square] + 1);
        }
    }
    return dp[n];
}
```

**This is Coin Change (Day 84), unchanged in shape.** The "coin denominations" are simply every perfect square `≤ n` (`1, 4, 9, 16, ...`) instead of the problem's given coin list, and unlimited reuse is exactly correct here too — nothing stops using `4` twice (`4+4+4=12`). The sentinel value `n+1` (not `Integer.MAX_VALUE`) is the identical overflow-avoidance trick from Day 84: relaxing with `dp[i-square*square] + 1` against `Integer.MAX_VALUE` would overflow; against `n+1`, it can't, since the true answer is always `≤ n` (using `n` copies of `1²`).

**Worked trace:** `n = 12`. Relevant squares: `1, 4, 9`. `dp[0]=0`. Building up: `dp[4]=min(dp[3]+1, dp[0]+1)=1` (using one `4`). `dp[8]=dp[4]+1=2` (two `4`s). `dp[12]=min(dp[11]+1, dp[8]+1, dp[3]+1)`; `dp[8]+1 = 3` turns out to be the minimum path. Final `dp[12] = 3` (`4+4+4`), matching the known answer, and separately confirmed against `n=13 → 2` (`4+9`) and `n=100 → 1` (`100` is itself `10²`).

**Complexity: Time O(n × √n)** — for each of `n` values, iterate up to `√n` candidate squares. **Space O(n).**

**Edge cases:** `n` itself a perfect square → answer `1`, falls out of the recurrence naturally (no special case needed). `n=0` → `0` (given as the base case). Small `n` (1, 2, 3) → `1, 2, 3` respectively (`1`, `1+1`, `1+1+1` — no combination of squares beats using `1`s alone for `2` or `3`, correctly found by the DP without hardcoding).

**Extension (not required, worth naming):** a purely number-theoretic solution exists via **Lagrange's Four-Square Theorem** (every natural number is expressible as the sum of at most 4 perfect squares) combined with **Legendre's three-square theorem** (a number is expressible as the sum of 3 or fewer squares unless it has the exact form `4^a(8b+7)`) — this gives an O(√n) check for whether the answer is 1, 2, 3, or (by elimination) 4, with no DP table at all. Not expected to be derived live in an interview, but naming that a closed-form number-theoretic answer exists, distinct from the DP, is a strong signal if an interviewer pushes for a faster-than-O(n√n) solution.

> 💡 **Interview Insight:** stating "this is Coin Change with the coin list replaced by perfect squares" immediately, before writing any code, does double duty — it proves the pattern-recognition this whole DP unit is built around, and it means the interviewer already knows what code is coming before you write it, which reads as confidence rather than uncertainty.

---

## Problem 14: Word Break II (LC 140, Hard) — Pattern: 1D DP + Backtracking

**Statement:** Given a string `s` and a dictionary `wordDict`, return *all* possible sentences where `s` is segmented into a space-separated sequence of dictionary words (each dictionary word may be reused).

### Why this problem is genuinely new, not just "Word Break, but return more"

Week 9, Day 61 formally distinguished Backtracking from Dynamic Programming along one specific line: DP exploits overlapping subproblems to compute a count or an optimum; Backtracking enumerates a full combinatorial solution space, with no overlap to exploit. Word Break (Day 84) is pure DP — *does* a segmentation exist, a single boolean. Word Break II asks for *every* segmentation — that's enumeration, Backtracking's job, not DP's. But naive Backtracking alone is dangerous here: for a string with **no** valid segmentation at all (e.g. `s = "catsandog"`, missing a final "dog" split that completes), unguided backtracking still explores every possible split point combinatorially before giving up, with no way to bail out early. That's exactly where Day 84's `dp[]` boolean array comes back — not re-taught, reused directly, as a pruning oracle.

### Approach 1 — Brute force: pure backtracking, no pruning

```java
public static List<String> wordBreakBruteForce(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    List<String> result = new ArrayList<>();
    backtrackNaive(s, 0, dict, new ArrayList<>(), result);
    return result;
}

private static void backtrackNaive(String s, int start, Set<String> dict,
                                     List<String> path, List<String> result) {
    if (start == s.length()) {
        result.add(String.join(" ", path));
        return;
    }
    for (int end = start + 1; end <= s.length(); end++) {
        String word = s.substring(start, end);
        if (dict.contains(word)) {
            path.add(word);                          // choose
            backtrackNaive(s, end, dict, path, result);  // explore
            path.remove(path.size() - 1);             // un-choose — Week 9's template, unchanged
        }
    }
}
```

This is exactly Week 9's choose → explore → un-choose template, applied to string split-points instead of array elements — structurally the same shape as Palindrome Partitioning (Week 10, Day 66), substituting a dictionary-membership check for a palindrome check. **Complexity: worst case exponential in `s.length()`** — for a string with no valid segmentation, every combination of split points still gets explored before the recursion exhausts itself, since nothing signals "this remainder can never work" ahead of time.

### Approach 2 — Optimized: DP-array-gated backtracking (memoized)

```java
public static List<String> wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    int n = s.length();

    // Phase 1 — Word Break's exact boolean DP (Day 84), reused unchanged as a pruning oracle
    boolean[] reachable = new boolean[n + 1];
    reachable[0] = true;
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            if (reachable[j] && dict.contains(s.substring(j, i))) {
                reachable[i] = true;
                break;
            }
        }
    }
    if (!reachable[n]) return new ArrayList<>();   // whole string unsegmentable — bail immediately, skip backtracking entirely

    // Phase 2 — backtrack, but ONLY into positions reachable[] confirms are worth exploring
    Map<Integer, List<String>> memo = new HashMap<>();
    return backtrack(s, 0, dict, reachable, memo);
}

private static List<String> backtrack(String s, int start, Set<String> dict,
                                        boolean[] reachable, Map<Integer, List<String>> memo) {
    if (memo.containsKey(start)) return memo.get(start);
    if (start == s.length()) {
        List<String> base = new ArrayList<>();
        base.add("");
        return base;
    }

    List<String> sentences = new ArrayList<>();
    for (int end = start + 1; end <= s.length(); end++) {
        if (!reachable[end]) continue;               // THE PRUNE: skip any split point DP already proved is a dead end
        String word = s.substring(start, end);
        if (dict.contains(word)) {
            for (String rest : backtrack(s, end, dict, reachable, memo)) {
                sentences.add(rest.isEmpty() ? word : word + " " + rest);
            }
        }
    }
    memo.put(start, sentences);
    return sentences;
}
```

**Exactly which branches the prune eliminates:** without `reachable[end]` gating the loop, the backtracking would still try every dictionary word starting at `start`, including ones that lead into an `end` position from which *no* valid segmentation of the remaining suffix exists — discovering that dead end only after fully recursing into it. With the gate, any `end` where `reachable[end]` is `false` is skipped in O(1), *before* recursing at all, because Phase 1 already proved — once, for the whole string, in O(n²) — that no segmentation of `s[end..n)` can ever succeed. This is the concrete payoff of Day 84's `dp[]` boolean array: it was built to answer one yes/no question, and today it's repurposed as a cheap oracle that tells backtracking exactly where not to bother looking.

**Complexity, precisely:** the `reachable[]` computation is O(n²) (Word Break's own bound, string-hashing costs aside — see Day 84's note on `substring()`/`hashCode()` cost). The backtracking phase, gated by `reachable[]`, never explores a position that can't eventually succeed, so no work is wasted on dead branches — but for a string with genuinely many valid segmentations (e.g. a long run of `"a"`s against a dictionary of `{"a","aa","aaa",...}`), the number of valid sentences itself can be exponential in `n`, and every one of them has to be materialized into the output. The honest complexity statement is **Time O(n²) for the pruning/traversal backbone, plus O(output size)** for constructing every returned sentence — the plan's own stated bound. The `reachable[]` gate guarantees the algorithm never does *wasted* combinatorial work chasing dead ends; it cannot, and doesn't claim to, make the *output* itself polynomial when the true number of valid segmentations genuinely is exponential.

**Worked trace:** `s = "catsanddog"`, `wordDict = ["cat","cats","and","sand","dog"]`. `reachable[]` computed first: `reachable[3]=true` ("cat"), `reachable[4]=true` ("cats"), `reachable[7]=true` ("sand" reached from index 3, since `s[3:7]`="sand" and `reachable[3]` is already true), `reachable[10]=true` ("dog" from 7). Backtracking from 0: tries "cat" (dict, `reachable[3]`=true) → recurse from 3; tries "cats" (dict, `reachable[4]`=true) → recurse from 4. From 3: tries "sand" (dict, `reachable[7]`=true) → recurse from 7 → "dog" (dict, `reachable[10]`=true) → recurse from 10 = end, return `[""]` → sentence "sand dog" → full: "cat sand dog". From 4: tries "and" (dict, `reachable[7]`=true) → recurse from 7 → "dog" → "and dog" → full: "cats and dog". Final result: `["cat sand dog", "cats and dog"]` — matches the known LC example exactly (independently verified before writing this book). Separately confirmed: `s="catsandog"` (no valid final split) returns `[]` immediately via the Phase 1 `reachable[n]` check, with zero backtracking performed.

**Edge cases:** no valid segmentation at all → `reachable[n]` is `false`, return empty list immediately, no wasted backtracking (this is precisely the case the naive Approach 1 handles worst). Empty dictionary → `reachable[]` stays all-false past index 0, empty result. Every character individually in the dictionary → many valid segmentations, output size itself can be large — the "plus output size" term in the complexity is what's actually being paid for there, not wasted search.

> 💡 **Interview Insight:** if asked "how would you speed up your first backtracking attempt," the strongest answer is naming the exact reuse: "run Word Break's own boolean DP first, in O(n²), then gate the backtracking's branch choices on it" — not a vague "add memoization." Being able to say *which* prior problem's exact array is being repurposed, and *why* it's valid to reuse it unchanged, is what separates "knows the trick" from "has internalized why patterns compose."

**🔗 Backward reference — the same shape, one deferred detail resolved:** Palindrome Partitioning (Week 10, Day 66) backtracks over the same kind of "every split point" search, checking palindrome-validity fresh at every call, with no precomputed table — DP-based `isPalindrome` memoization was named at the time as a known optimization and *explicitly deferred*. That deferral gets paid off in full on Day 88 (tomorrow's tomorrow), when Palindromic Substrings builds exactly that table. Today's Word Break II is the same *category* of payoff one week early, for a dictionary-membership gate instead of a palindrome gate.

---

### 1D DP Closes: 14/14

Climbing Stairs, Min Cost Climbing Stairs, House Robber, House Robber II, Decode Ways, Maximum Product Subarray *(recapped, Week 4)*, Word Break, Coin Change, Coin Change II, Longest Increasing Subsequence, Partition Equal Subset Sum, Target Sum, Perfect Squares, Word Break II — 14 problems, four distinct lookback shapes (always-sum, max-take-or-skip, validity-gated, dictionary/value-gated), two knapsack flavors (unbounded, 0/1), and — as of today — DP-gated Backtracking. Every loop-order rule this pattern needed is now on the table:

### 🔑 The Complete Loop-Order Reference (all four cases, side by side)

| Problem | Reuse? | Counting what? | Loop order | The one-sentence reason |
|---|---|---|---|---|
| Coin Change (Day 84) | Unbounded | Minimum count (optimize) | Doesn't matter | A minimum doesn't care what order values were considered in |
| Coin Change II (Day 85) | Unbounded | Combinations (unordered) | Coins outer | Forces one canonical coin-order per combination |
| Combination Sum IV (Day 85, extra) | Unbounded | Permutations (ordered) | Amount outer | Lets every number be "placed last" at every amount |
| Partition Equal Subset Sum / Target Sum (Day 85-86) | **Bounded (0/1)** | Reachability / count | Capacity **decreasing** | Guarantees each item's own pass never reads a cell it just wrote |

Two of these four rules exist because of *what's being counted* (combinations vs. permutations); one exists because of *reuse rules* (bounded vs. unbounded) — genuinely independent axes, easy to conflate under time pressure, worth being able to name apart on sight.

---

## Day 86 — Interview Questions

---

**1. Derive Target Sum's `P = (target + totalSum) / 2` from scratch.**

*Answer:* Let `P` be the subset assigned `+`, `N` the subset assigned `-`. The expression's value is `P - N = target`; every number is in exactly one group, so `P + N = totalSum`. Adding the two equations eliminates `N`: `2P = target + totalSum`, giving `P = (target + totalSum) / 2`.

---

**2. Name Target Sum's two unsolvable-input guards, and explain why both are needed (not just one).**

*Answer:* `(target + totalSum)` odd → `P` isn't an integer, no valid split exists regardless of the numbers. `|target| > totalSum` → even when the parity check passes, the achievable range of sums is `[-totalSum, totalSum]`; a target outside that range is unreachable no matter how signs are assigned. The two catch different failure modes and neither implies the other.

---

**3. Why is Perfect Squares "the same problem" as Coin Change, mechanically?**

*Answer:* Both are Unbounded Knapsack minimizing a count of "items" summing to a target — Coin Change's items are the given coin denominations, Perfect Squares' items are every perfect square ≤ n. The recurrence, the sentinel-value trick, and the complexity shape are all identical; only the item list generation differs.

---

**4. Why does Word Break II need Word Break's `dp[]` array at all — why not just backtrack directly?**

*Answer:* Unguided backtracking on a string with no valid segmentation still explores every combination of split points before exhausting itself, with nothing to signal early that a given remainder can never work. Word Break's boolean array answers exactly that question — computed once in O(n²) — letting the backtracking skip any branch leading into a position it already knows is a dead end, in O(1) per check instead of a full wasted recursion.

---

**5. State Word Break II's complexity precisely, including why "plus output size" is an honest term, not a hedge.**

*Answer:* O(n²) for the DP-gated traversal backbone (no wasted exploration of dead branches), plus the cost of constructing every valid sentence returned — because the true *number* of valid segmentations can itself be exponential for some inputs (e.g. a string breakable many overlapping ways), and every one of them has to be materialized into the output regardless of how efficiently the search itself is pruned.

---

**6. How does Word Break II connect to Palindrome Partitioning (Week 10, Day 66), and what's different?**

*Answer:* Both backtrack over every way to split a string, choose → explore → un-choose. Palindrome Partitioning checks palindrome-validity fresh at every call with no precomputed table (a deferred optimization at the time). Word Break II checks dictionary-membership, and additionally gates the search with a precomputed boolean DP array — the dictionary case needed the gate to avoid combinatorial dead-end exploration in a way Palindrome Partitioning, checking a cheap local condition, didn't strictly require to still be considered efficient.

---

**7. Complete the sentence: "0/1 Knapsack's loop-order rule and Coin Change II's loop-order rule both involve reordering a loop, but they fix two different bugs — namely..."**

*Answer:* "...0/1 Knapsack's decreasing-capacity rule prevents a single item from being reused within its own pass (a bounded-vs-unbounded correctness bug); Coin Change II's coins-outer rule prevents the same combination from being counted once per reordering (a combinations-vs-permutations counting bug). Getting one fix confused for the other, or applying either rule to the wrong problem, produces a different kind of wrong answer in each case."

---

## Daily Deliverable Check

- [ ] Target Sum solved via the two-equation reduction, with both unsolvable-input guards explained and tested.
- [ ] Perfect Squares solved, explicitly identified as Coin Change's shape with perfect squares as the coin list.
- [ ] Word Break II solved with the `reachable[]` pruning gate — not pure unguided backtracking — and the `"catsandog"` no-solution case confirmed to bail out immediately rather than search exhaustively.
- [ ] Can state, from memory, all four loop-order rules from this week's table and which of the two underlying reasons (bounded/unbounded vs. combinations/permutations) drives each.
- [ ] **1D DP ladder complete at 14/14 — pushed to `dsa-java/dynamic-programming/`.**

---

## What Tomorrow Assumes You Already Know Cold

Day 87 opens Grid DP — a brand-new pattern, four problems, closing entirely in one day. It assumes 2D array *indexing* is already comfortable (established since Week 5, Day 32's `Search a 2D Matrix`), but introduces, for the first time in this series, *declaring and fully populating your own* 2D array from scratch (`new int[m][n]`) rather than indexing into one handed to you — flagged explicitly as tomorrow's one small new syntax wrinkle, not assumed silently. It also assumes today's general DP discipline (state `dp[i][j]`'s meaning in one precise sentence before writing any recurrence, the rule established Day 81) transfers automatically to two dimensions, since tomorrow won't re-derive that discipline from scratch.
