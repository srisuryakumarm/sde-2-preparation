# Week 13 — Consolidated Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Source days:** [Day 85](Day85_Resource_Book.md) · [Day 86](Day86_Resource_Book.md) · [Day 87](Day87_Resource_Book.md) · [Day 88](Day88_Resource_Book.md) · [Day 89](Day89_Resource_Book.md) · [Day 90](Day90_Resource_Book.md) · [Day 91](Day91_Resource_Book.md)

This bank pulls every interview question from Week 13's seven Resource Books into one review document — 46 questions total, covering 1D DP's completion, Grid DP, String DP, and Interval DP in full. Organized by day, in the order each topic was taught. For quick pre-interview review, the topical index below groups questions by theme rather than by day; the full Q&A pairs (identical wording to each day's book) follow after.

---

## Topical Review Index

**Loop-order rules (the week's recurring throughline):** Day 85 Q1, Q2, Q3, Q9; Day 86 Q7
**Proof-required recurrences (don't just recite the formula):** Day 85 Q6; Day 87 Q4; Day 89 Q2, Q3; Day 91 Q2
**Pattern contrasts (telling two similar-looking things apart):** Day 90 Q2, Q3; Day 91 Q5
**DP-as-pruning-oracle for backtracking:** Day 86 Q4, Q6; Day 90 Q6
**Reductions to an already-solved problem:** Day 86 Q1, Q3; Day 88 Q4; Day 91 Q3
**New-concept mechanics (0/1 Knapsack, Grid DP syntax, Interval DP traversal):** Day 85 Q8, Q9; Day 87 Q1; Day 91 Q1

---

## Day 85 — 1D DP Continues: Coin Change II, LIS Two Ways, 0/1 Knapsack Begins

---

**1. Why does Coin Change II need coins as the outer loop, but Coin Change (Day 84) didn't care about loop order at all?**

*Answer:* Coin Change minimizes — the smallest number of coins to reach an amount doesn't depend on what order those coins were conceptually added in, only on the final count. Coin Change II counts *combinations*, where order-independence is the actual thing being enforced (`[1,2]` and `[2,1]` must count once, not twice) — coins-outer guarantees every combination is built in one canonical coin order, which is what collapses reorderings into a single count.

---

**2. Give a concrete input where amount-outer, coins-inner overcounts Coin Change II, and explain the overcount.**

*Answer:* `amount=5, coins=[1,2,5]`: correct answer is 4, amount-outer computes 9. The wrong order lets every coin get a chance at every amount on every pass, which effectively counts different orderings of the same multiset of coins as distinct — it answers "how many ordered sequences," not "how many unordered combinations."

---

**3. What single change turns Coin Change II's code into Combination Sum IV's code, and why does that one change flip combinations into permutations?**

*Answer:* Swap which loop is outer — target/amount outer, nums inner. With amount outer, every number gets to be "whichever one is placed last" at every amount, so the same set of numbers reached in a different final order counts as a separate answer — which is exactly what "ordered" (permutations) means.

---

**4. In the O(n²) LIS solution, why is `dp[i]` defined as "LIS ending exactly at i," not "LIS somewhere in nums[0..i]"?**

*Answer:* The "ending exactly at i" definition is what makes the recurrence `dp[i] = max(dp[j]) + 1` for valid `j < i` correct — it lets the algorithm ask, precisely, "which earlier subsequences could this specific number extend?" A "somewhere in the prefix" definition would lose exactly the information (what value the subsequence currently ends on) needed to check whether `nums[i]` can legally extend it.

---

**5. In the O(n log n) LIS approach, what does `tails[k]` represent, and why does a smaller value at a given index make that entry "better"?**

*Answer:* `tails[k]` is the smallest tail value among all increasing subsequences of length `k+1` found so far. Smaller is better because it's a strictly weaker requirement for some future number to extend that length — any future number that could extend a subsequence ending on a larger tail could also extend one ending on a smaller tail, but not necessarily the reverse.

---

**6. Prove that binary search is valid on the `tails` array at every step, not just intuitively "probably sorted."**

*Answer:* By induction: `tails` starts empty (trivially sorted). Each step either appends a value larger than everything currently present (preserving order) or replaces `tails[lo]` with a value that binary search guarantees is both `≤` the old `tails[lo]` and `>` `tails[lo-1]` (since `lo` was the first index failing the `< num` test) — so strictly-increasing order survives every step, which is exactly what licenses the next binary search.

---

**7. Why isn't the final `tails` array necessarily a real subsequence of the input?**

*Answer:* `tails` only ever tracks the best possible tail *value* per achievable length — it never tracks which original indices produced that value, and a later "replace" can overwrite an entry with a value from a different, unrelated part of the array. `nums=[3,4,5,1]` ends with `tails=[1,4,5]`, but `1` occurs after `4` and `5` in the original array, so that's not a valid increasing subsequence, even though the length (3) is correct.

---

**8. State the core mechanism difference between Unbounded Knapsack (Coin Change, Day 84) and 0/1 Knapsack (today).**

*Answer:* Unbounded Knapsack's recurrence never tracks which items built a sub-answer, which is exactly what permits unlimited reuse. 0/1 Knapsack's recurrence is built so every transition reads from "the state before this item was considered" (row `i-1` in the 2D form) — item `i` can only ever be used once because using it moves you off that row entirely, not back onto it.

---

**9. Why must the 1D space-optimized 0/1 Knapsack loop run its capacity dimension in decreasing order? Give a concrete input where increasing order breaks.**

*Answer:* Decreasing order guarantees every cell read during the current item's pass still reflects the state from *before* this item was considered. `nums=[3]`, checking reachability of sum `6`: increasing order lets `dp[3]` (just set true by the single 3) feed into `dp[6]` later in the *same* pass, incorrectly reusing that one `3` twice and reporting `true`; decreasing order evaluates `dp[6]` using the pre-pass value of `dp[3]` (false), correctly reporting `6` unreachable with only one `3` available.

---

**10. Partition Equal Subset Sum asks about splitting into two equal halves — how does that become a single-subset reachability question?**

*Answer:* If the total sum is even and some subset sums to exactly half, everything *not* in that subset automatically sums to the other half (total minus half is half). This turns a two-sided partition question into "does a subset summing to `totalSum/2` exist" — a direct 0/1 Knapsack reachability check.

---

**11. Why is an odd `totalSum` an immediate `false`, with no DP needed?**

*Answer:* Two equal integer subsets must each sum to `totalSum / 2`; an odd total has no integer half, so no valid split can possibly exist regardless of the specific numbers — checked once, up front, before spending any DP work.

---

## Day 86 — 1D DP Completes: Target Sum, Perfect Squares, Word Break II

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

## Day 87 — Grid DP: Unique Paths, Unique Paths II, Minimum Path Sum, Maximal Square

---

**1. What's the one new Java syntax element today, and why hasn't it come up before despite 2D arrays being used since Week 5?**

*Answer:* Declaring and fully populating your own 2D array (`new int[m][n]`) from scratch. Every prior 2D-array problem (e.g. Search a 2D Matrix, Week 5 Day 32) indexed into a matrix handed in as input — today is the first time the array itself needs to be allocated and built, not just read.

---

**2. Why does Unique Paths sum its two neighbors, while Minimum Path Sum takes their minimum?**

*Answer:* Unique Paths counts every distinct way to arrive — the two immediately-prior cells represent mutually exclusive, collectively exhaustive cases, so their counts add. Minimum Path Sum picks one optimal path — only the cheaper of the two prior cells is ever worth continuing from, since any path through the more expensive one can't beat the corresponding path through the cheaper one.

---

**3. In Unique Paths II, why is it wrong to zero out only the exact obstacle cell in the first row/column, leaving every cell after it at 1?**

*Answer:* With movement restricted to right/down and confinement to a single row or column, there is no way around an obstacle — every cell after it in that row/column is genuinely unreachable, not just the obstacle cell itself. The correct implementation makes each first-row/column cell depend on the previous cell's already-computed value, so a zero from an obstacle propagates forward automatically.

---

**4. Prove why Maximal Square's recurrence uses the *minimum* of three neighbors, not their maximum.**

*Answer:* Necessity: if a square of side k exists ending at (i,j), each of the three shifted (k-1)-squares (up, left, diagonal) is entirely contained within it and must also be all-1s, so all three neighbors support at least k-1 — meaning dp[i][j] can be at most min(neighbors)+1. Sufficiency: if all three neighbors support at least m, the three shifted m-squares jointly cover every cell of the (m+1)-square except the single new corner cell, which is confirmed 1 directly — so a real (m+1)-square exists. Using the maximum instead would overclaim a square size no single neighbor can actually support along its own limiting direction.

---

**5. Why does Dungeon Game's DP have to run backwards, when every other Grid DP problem today ran forwards?**

*Answer:* "Minimum health needed entering a cell" depends entirely on the cost of the path still ahead, not anything already traversed — the opposite of every forward problem today, where dp[i][j] only ever needed information about cells already visited. Running the sweep from the destination back to the start makes "the rest of the path" already-known information by the time each cell is computed.

---

**6. In Dungeon Game, why take the minimum of the two forward neighbors' required-health values before subtracting the current cell's value, rather than after?**

*Answer:* The knight will always choose whichever next cell demands less required-entering-health — that's the objectively better move — so the DP should base the current cell's requirement on that better (smaller) option. Subtracting first and comparing afterward would compare two different quantities (post-subtraction values across different next-cell options) rather than choosing the genuinely cheaper path first.

---

## Day 88 — String DP Begins: LCS, Edit Distance, Longest Palindromic Substring

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

## Day 89 — String DP Continues: Palindromic Substrings, Longest Palindromic Subsequence, Interleaving String

---

**1. What's the only real change between Day 88's Longest Palindromic Substring and today's Palindromic Substrings?**

*Answer:* The aggregation step. Both use identical Expand Around Center code with `2n-1` centers; Day 88 tracked the single longest expansion found, today counts every successful expansion as one more valid palindromic substring, incrementing inside the expansion loop rather than comparing against a running maximum afterward.

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

## Day 90 — String DP Completes: Distinct Subsequences, Wildcard Matching, Regular Expression Matching

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

## Day 91 — Interval DP Completes: Burst Balloons, Minimum Insertion Steps, Predict the Winner

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

*46 questions total across seven days. See [00_Curriculum_Map.md](00_Curriculum_Map.md) for the full cumulative problem inventory and concept index this bank draws on.*
