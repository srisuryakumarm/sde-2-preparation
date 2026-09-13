# Day 83 — 1D DP: Decoding and Products

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 82 Resource Book](Day82_Resource_Book.md)
**Next ▶:** [Day 84 Resource Book](Day84_Resource_Book.md)
**Companion to:** Day 83 of `Week_12_Revised.md`

---

## ⚠️ Overlap Notice — Read This First

| LC # | Problem | Status |
|---|---|---|
| 91 | Decode Ways | New — full depth below |
| 152 | Maximum Product Subarray | **Already solved — Week 4, Day 22, Extra Practice.** Recap only, not re-taught. |

`Week_12_Revised.md` lists Maximum Product Subarray as today's Problem 6, without flagging it as a repeat. Checked against `00_Curriculum_Map.md`'s cumulative problem table: it's already there, added during Week 4's Kadane's-family extra practice, tagged "Kadane's, Running Max AND Min" — the exact technique this problem needs. Per the overlap rule, a problem already solved doesn't get re-taught from scratch, even when the plan lists it as if new.

**No substitute problem added today.** Only one of today's two required problems overlaps — Decode Ways is genuinely new and gets full depth below. The freed time from Maximum Product Subarray's brief recap goes toward that depth, rather than toward a same-day filler problem: the pattern it recaps (Kadane's, extended to track a running min alongside a running max) doesn't have an untaught, comparable-difficulty variant that wouldn't either be redundant with Week 4's own extra practice or reach ahead into `Week_13_Revised.md`'s upcoming 1D DP problems (Longest Increasing Subsequence, Partition Equal Subset Sum). This overlap is also noted below, in place, and will be logged in today's curriculum map update.

---

## Recap

Yesterday's recurrence made a binary take-or-skip decision at each position. Today's new problem, Decode Ways, keeps the same two-steps-back lookback shape but gates each term on a **validity check** first — a digit or digit-pair only contributes to the count if it's a legal code at all, which is a genuinely different kind of condition than "adjacent or not." Today's recap, Maximum Product Subarray, reaches back further — to Kadane's Algorithm (Week 3, Day 21), extended with a second running value to handle multiplication's sign-flipping behavior, something addition never needed.

---

## Learning Objectives

By the end of today, without notes:

1. Prove Decode Ways's recurrence correct, including precisely why a leading zero — in either a single-digit or two-digit position — invalidates a decoding, and why the standard implementation needs no separate check for it.
2. Trace Decode Ways on an input containing a `0` and confirm the recurrence naturally propagates that invalidity forward without special-casing.
3. Recap Maximum Product Subarray's running-max-and-min technique and explain precisely why Kadane's single running value (Week 3, Day 21) isn't sufficient once multiplication is on the table.

---

## Concept Dependency Map

```
Yesterday: dp[i] = max(dp[i-1], dp[i-2]+nums[i])  (take-or-skip DECISION)
        │
        └──▶ same lookback shape, now VALIDITY-GATED: each term only contributes
             if the digit(s) it represents form a legal code at all
                  │
                  ▼
             Problem 5: Decode Ways (LC 91) — NEW, full depth

Kadane's Algorithm (Week 3, Day 21) — running max ending at i, ADDITION only
        │
        └──▶ multiplication flips sign — a very negative running product can become
             the new max after one more negative factor — needs a running MIN too
                  │
                  ▼
             🔗 Recap: Maximum Product Subarray (LC 152) — Week 4, Day 22, Extra Practice
```

---

## Problem 5: Decode Ways (LeetCode 91, Medium) — Pattern: 1D DP, Validity-Gated

**Statement:** A string of digits encodes letters `A`–`Z` as `1`–`26` (`'A'=1` ... `'Z'=26`). Given a digit string, count the number of distinct ways it could have been decoded.

**`dp[i]`, precisely:** the number of valid decodings of the prefix `s[0..i)` (the first `i` characters).

**Why `dp[i] = dp[i-1] + dp[i-2]`, each term gated by a validity check:** every valid decoding of `s[0..i)` ends by decoding either the **last single character** alone, or the **last two characters together** as one two-digit code — no other grouping is possible, and these two cases are mutually exclusive (a specific decoding's final group is either one character or two, never ambiguously both). So: if `s[i-1]` alone is a valid single-digit code (`1`–`9` — **not** `0`, since there is no code `0`), that final-single-character case contributes every valid way to decode everything before it, `dp[i-1]`. If `s[i-2..i)` together form a valid two-digit code (`10`–`26`), that final-pair case contributes `dp[i-2]`. Sum whichever of these two terms is actually valid. Base case: `dp[0] = 1` — there is exactly one way to decode zero characters: decode nothing.

**⚠️ The leading-zero subtlety, resolved precisely:** a lone `'0'` is never a valid single-digit code (codes are `1`–`26`; there's no `0`). This matters in two places — first, if the string's very first character is `'0'`, no decoding can even begin (handled as an upfront special case, `return 0`). Second, and less obvious: a two-digit group like `"06"` is **also** invalid — not because of some separate leading-zero rule, but because `"06"` parses to the integer `6`, which simply fails the `10`–`26` range check on its own. **No dedicated leading-zero check is needed for the two-digit case** — the numeric range test already excludes every `"0X"` pairing automatically, since any string of that shape evaluates below `10`.

```java
public int numDecodings(String s) {
    int n = s.length();
    if (n == 0 || s.charAt(0) == '0') return 0; // can't even start a decoding

    int prev2 = 1; // dp[0] = 1 — one way to decode nothing
    int prev1 = 1; // dp[1] = 1 — first char already confirmed valid (1-9) by the guard above

    for (int i = 2; i <= n; i++) {
        int curr = 0;
        int oneDigit = s.charAt(i - 1) - '0';
        int twoDigit = Integer.parseInt(s.substring(i - 2, i));

        if (oneDigit >= 1 && oneDigit <= 9) {
            curr += prev1;
        }
        if (twoDigit >= 10 && twoDigit <= 26) { // "0X" pairs fail this on numeric value alone
            curr += prev2;
        }

        prev2 = prev1;
        prev1 = curr;
    }

    return prev1;
}
```

(Brute-force recursion and memoization here are structurally identical to yesterday's tiers — recurse on `i-1` and `i-2` with the same validity gates, cache each `i` once. Shown here at the space-optimized tier directly, since that progression is now established.)

**Worked trace 1 — a clean case, `s = "226"`:**

| i | dp[i] computation | dp[i] |
|---|---|---|
| 0 | base | 1 |
| 1 | `'2'` valid (1-9) → `dp[0]` | 1 |
| 2 | `'2'` valid → `+dp[1]`; `"22"` valid (10-26) → `+dp[0]` | 1+1=2 |
| 3 | `'6'` valid → `+dp[2]`; `"26"` valid → `+dp[1]` | 2+1=3 |

**Answer: 3** — `"2,2,6"`, `"22,6"`, `"2,26"`. Exhaustive by hand: no fourth decoding exists.

**Worked trace 2 — the leading-zero case, `s = "106"`, proving the gate works without a special rule:**

| i | dp[i] computation | dp[i] |
|---|---|---|
| 0 | base | 1 |
| 1 | `'1'` valid → `dp[0]` | 1 |
| 2 | `'0'` **not** valid (fails 1-9) → contributes 0; `"10"` valid (10-26) → `+dp[0]` | 0+1=1 |
| 3 | `'6'` valid → `+dp[2]`; `"06"` fails the range check (parses to 6, <10) → contributes 0 | 1+0=1 |

**Answer: 1** — only `"10,6"` is valid. `"1,0,6"` fails because a lone `'0'` decodes nothing; `"1,06"` fails because `"06"` isn't a legal two-digit code. Both invalid paths are excluded automatically by the same two numeric checks, with no dedicated zero-detection logic anywhere in the code.

**Complexity:** Time O(n), Space O(1).

**Edge cases:** a string starting with `'0'` (upfront `return 0`); a string containing an interior `'0'` not preceded by a `'1'` or `'2'` (e.g., `"90"` — `'9'` then `'0'`: `dp[1]=1`; at `i=2`, `'0'` fails the single-digit check, and `"90"` fails the two-digit range check too since `90 > 26` — `dp[2]=0`, correctly propagating total invalidity forward, since `"90"` genuinely has zero valid decodings); a two-character string that's a valid code both ways (e.g., `"12"` → `dp[2] = dp[1] + dp[0] = 1+1 = 2`, correctly counting both `"1,2"` and `"12"`).

**💡 Interview Insight:** stating "a lone zero is never valid, and a leading-zero pair fails the range check on its own — no extra rule needed" unprompted heads off the most common follow-up question on this problem before it's even asked, and demonstrates the check was understood, not just copied from a pattern.

---

## 🔗 Recap: Maximum Product Subarray (LeetCode 152, Medium)

**Original coverage:** Week 4, Day 22, Extra Practice — tagged "Kadane's, Running Max AND Min."

**Why Kadane's single running value (Week 3, Day 21) isn't enough here:** Kadane's tracked one running best-sum-ending-here, because adding a negative number to a running sum only ever *decreases* it — the ordering between "currently best" and "currently worst" never flips. Multiplication breaks that assumption entirely: multiplying a very *negative* running product by one more negative number produces a very *positive* result — the current minimum can become the next maximum in a single step. The fix: track **both** a running max and a running min ending at each position, and when the current number is negative, swap them before combining, since a negative multiplier reverses which one is capable of producing the larger result next.

```java
public int maxProduct(int[] nums) {
    int maxEndingHere = nums[0], minEndingHere = nums[0], result = nums[0];
    for (int i = 1; i < nums.length; i++) {
        int num = nums[i];
        if (num < 0) {
            int temp = maxEndingHere;
            maxEndingHere = minEndingHere;
            minEndingHere = temp; // negative multiplier flips which was bigger
        }
        maxEndingHere = Math.max(num, maxEndingHere * num);
        minEndingHere = Math.min(num, minEndingHere * num);
        result = Math.max(result, maxEndingHere);
    }
    return result;
}
```

**Complexity:** Time O(n), Space O(1) — identical shape to Kadane's itself, just carrying two running values instead of one.

**🔑 Key Takeaway:** whenever a running "best so far" recurrence involves multiplication rather than addition, check whether a sign flip can reorder your candidates — if it can, you need to track both extremes, not just the one you currently care about.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** follow up on Tier B applications submitted this week.

---

## Day 83 — Interview Questions

**Q1. Prove Decode Ways's recurrence correct.** Every valid decoding of a prefix ends in exactly one of two mutually exclusive final groups — a single trailing character or a trailing pair — so the count is the sum of whichever of those two cases is actually a legal code, contributing `dp[i-1]` or `dp[i-2]` respectively.

**Q2. Why is a lone `'0'` never a valid single-digit code?** Codes only run `1`–`26`; there is no code `0`, so a standalone `'0'` can never be decoded on its own.

**Q3. Why does `"06"` fail as a two-digit code without a dedicated leading-zero check?** `"06"` parses to the integer `6`, which is below the required `10`–`26` range — the same numeric range test that validates every other two-digit group already excludes it, with no separate rule needed.

**Q4. In the `"106"` trace, why does `dp[2]` end up as `1` rather than `0`?** Because `'0'` alone fails the single-digit check (contributing 0) but `"10"` succeeds as a valid two-digit code (contributing `dp[0]=1`) — the two-digit path rescues what the single-digit path couldn't cover.

**Q5. What happens if an interior character is `'0'` and it can't form a valid two-digit code with its predecessor (e.g., `"90"`)?** Both terms fail — `'0'` alone is invalid and `90` exceeds the `10`–`26` range — so `dp[i]=0` at that position, and that zero propagates forward through every later `dp[]` value that depends on it, correctly making the whole string undecodable from that point on.

**Q6. Why did Maximum Product Subarray get a recap instead of a full re-teach today?** It was already solved in Week 4, Day 22 as extra practice during Kadane's coverage — the overlap was caught by checking today's required problems against the cumulative problem table before treating anything as new.

**Q7. Why wasn't a substitute problem added today to replace the recapped one?** Only one of today's two required problems overlapped, not both — the rule for adding a fresh replacement problem applies specifically when an entire day's required block turns out to already be solved, which isn't the case here.

**Q8. Why does Maximum Product Subarray need a running minimum, when Kadane's only ever needed a running maximum?** Addition never reorders candidates — adding a negative number only shrinks a running sum. Multiplying by a negative number can flip a very negative running product into the new largest value in a single step, so the algorithm has to track the running minimum too, in case it's about to become the maximum.

**Q9. Why does the code swap `maxEndingHere` and `minEndingHere` specifically when the current number is negative?** A negative multiplier reverses which of the two running values is capable of producing the larger product next — swapping first means the subsequent `max`/`min` calculations combine the correct operands without needing a three-way comparison against both un-swapped values.

**Q10. State the general lesson this recap reinforces about extending a known technique.** Whenever a "best so far" recurrence swaps addition for multiplication (or any operation where a sign or direction can flip), check whether the ordering between your current best and worst candidates can reverse — if it can, tracking only one running extreme is no longer sufficient.

---

## Daily Deliverable Check

- [ ] Decode Ways (LC 91) solved with the leading-zero handling explainable precisely (both the empty-decoding and range-check mechanisms, not just "zeros are special"), pushed.
- [ ] Maximum Product Subarray (LC 152) recapped — running max/min technique explainable, correctly cited back to Week 4, Day 22 and connected to Kadane's (Week 3, Day 21).
- [ ] Today's overlap (LC 152) noted and understood, not silently absorbed.
- [ ] Tier B applications followed up on.

---

## What Tomorrow Assumes You Already Know Cold

Day 84 needs general 1D DP fluency solid across everything built this week — the "state `dp[i]` precisely, then prove the recurrence via exhaustive disjoint cases" discipline is the actual through-line, not any one specific recurrence. Tomorrow's two problems are each genuinely new shapes (a dictionary-gated segmentation problem, and the week's first *unbounded*-reuse problem), so nothing from today's validity-gating or yesterday's take-or-skip logic carries forward directly — but `HashSet` (Week 1, Day 4) does, since Word Break's dictionary lookups depend on it being reflexive.
