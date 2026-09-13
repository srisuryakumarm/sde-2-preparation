# Day 84 (Sunday) — Consolidation, and Unbounded Knapsack Begins

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 83 Resource Book](Day83_Resource_Book.md)
**Next ▶:** [Day 85 Resource Book](Day85_Resource_Book.md)
**Companion to:** Day 84 of `Week_12_Revised.md`

---

## Recap

This week ran two patterns end to end: Dijkstra's Algorithm, opened and closed within three days, and Dynamic Programming, opened and now four days into a run that continues through next week's entire leave week. Today closes Week 12 with two more genuinely new DP shapes — a dictionary-gated segmentation problem, and the week's first **unbounded** problem, where a single denomination can be reused without limit, a constraint none of this week's earlier problems had.

---

## Self-Check (15 min)

Before adding anything new, state — from memory, without looking back — what `dp[i]` represents for each problem solved so far this week:

- [ ] Climbing Stairs — *the number of distinct ways to reach step `i`.*
- [ ] Min Cost Climbing Stairs — *the minimum total cost paid by the point you launch away from stair `i` (already including `cost[i]` itself).*
- [ ] House Robber — *the maximum money obtainable from houses `0..i`, inclusive.*
- [ ] House Robber II — *(reduction, not a single `dp[i]`) the max of two linear House Robber runs, each excluding one of the two circularly-adjacent boundary houses.*
- [ ] Decode Ways — *the number of valid decodings of the prefix `s[0..i)`.*
- [ ] Maximum Product Subarray (recap) — *the maximum and minimum product of a subarray ending exactly at position `i`.*

If any of these feel shaky rather than immediate, that's worth resolving now — everything below builds on the same "state `dp[i]` precisely first" discipline, and next week runs at leave-week intensity with no slack built in for going back to firm up this week's foundations.

---

## Learning Objectives

By the end of today, without notes:

1. Prove Word Break's recurrence correct, and state precisely — with a worked accounting, not an assertion — why its true worst-case time complexity is a stricter bound than the commonly-cited O(n²).
2. Prove Coin Change's recurrence correct via an exchange-style optimal-substructure argument, and explain why the "unbounded" reuse of a coin denomination falls directly out of the recurrence's structure rather than needing separate logic.
3. Explain precisely why Coin Change's loop order (coins vs. amounts) doesn't affect its answer, and what property of the problem makes that true.
4. Give a complete, corrected account of this week's problem count, catching and explaining any drift between the plan's own internal scorecard and the curriculum map's authoritative running total.

---

## Concept Dependency Map

```
This week's full 1D DP recurrence family (Days 81-83): dp[i] built from a small, FIXED
set of earlier dp[j]'s (always i-1, i-2, or a validity-checked pair)
        │
        ├──▶ dictionary/Set-gated segmentation — the set of valid j's is no longer fixed
        │    at "always i-1 and i-2," it's "every j where a dictionary word bridges j to i"
        │         │         (needs: HashSet, Wk1 D4)
        │         ▼
        │    Problem 7: Word Break (LC 139)
        │
        └──▶ UNBOUNDED reuse allowed for the first time — dp[i] can be built using the
             SAME coin denomination more than once, with no bookkeeping to prevent it
                  │
                  ▼
             Problem 8: Coin Change (LC 322)
                  (Week 13 preview, not taught: Coin Change II needs a DIFFERENT loop-order
                   rule, since it counts combinations rather than minimizing a count — noted
                   below, not built)
```

---

## Problem 7: Word Break (LeetCode 139, Medium) — Pattern: 1D DP + Set

**Statement:** given a string `s` and a dictionary of words `wordDict`, determine whether `s` can be segmented into a space-separated sequence of one or more dictionary words (words may be reused).

**`dp[i]`, precisely:** whether the prefix `s[0..i)` can be fully segmented into dictionary words.

**Why `dp[i] = OR over every valid j<i of (dp[j] AND s[j..i) ∈ dictionary)`:** any valid full segmentation of `s[0..i)` has a specific **last word** in it, starting at some position `j` and running to `i`. If `j` is that last word's start, then `s[0..j)` must itself be validly segmentable by everything before the last word (`dp[j]` true), and `s[j..i)` must itself be a single dictionary word. Trying every possible `j` covers every possible "where does the last word start" position — if *any* valid full segmentation exists, at least one such `j` will find it, and if none exists, no `j` will produce a true result. Base case: `dp[0] = true` — zero characters need zero words, trivially segmentable.

**Why a `Set`, specifically (Week 1, Day 4):** the recurrence checks dictionary membership up to O(n²) times across the full run — a `HashSet` gives O(1) average lookup per check; a `List` would force an O(k) linear scan through the dictionary on *every single check* (`k` = dictionary size), compounding an already-quadratic state count into something far worse.

```java
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    int n = s.length();
    boolean[] dp = new boolean[n + 1];
    dp[0] = true;

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && dict.contains(s.substring(j, i))) {
                dp[i] = true;
                break; // one valid split point is enough for this i
            }
        }
    }
    return dp[n];
}
```

**Worked trace:** `s = "leetcode"`, `wordDict = ["leet", "code"]`.

| i | checks (dp[j] must be true, and s[j..i) ∈ dict) | dp[i] |
|---|---|---|
| 0 | base | **true** |
| 1–3 | no `j` produces both conditions true | false |
| 4 | j=0: dp[0]=true, s[0..4)="leet" ∈ dict → **true** | **true** |
| 5–7 | j=4: dp[4]=true, but s[4..5)="c", s[4..6)="co", s[4..7)="cod" — none in dict; no other dp[j] is true | false |
| 8 | j=4: dp[4]=true, s[4..8)="code" ∈ dict → **true** | **true** |

**Answer: `dp[8] = true`** — `"leet" + "code"`.

**⚠️ Complexity — worth being more precise than the commonly-cited figure:** the DP itself has O(n²) *states and transitions* (n positions `i`, up to n values of `j` each) — this is the figure usually quoted, and it's correct **as a count of dp-table operations, assuming each dictionary check is O(1)**. It isn't, in Java: `s.substring(j, i)` creates a genuinely new `String` (a real copy since Java 7, not a shared-backing-array view), costing O(i-j) to build, and a freshly-built `String`'s `hashCode()` hasn't been cached yet, costing another O(i-j) to compute before the `HashSet` lookup can even begin. Accounting for this honestly: for a fixed `i`, summing the substring-length cost over every `j` from `0` to `i-1` gives O(i²); summing that over every `i` from `1` to `n` gives a true worst-case of **O(n³)**, not O(n²). This doesn't change which algorithm to write — it changes what you say if an interviewer asks "are you sure that's O(n²)?" A Trie built from the dictionary, or index-based (no-copy) substring comparison, removes the extra factor and gets back to a genuine O(n²) — worth naming as a follow-up optimization, flagged here as extension material rather than folded into the core solution, since it isn't needed to pass the problem as stated.

**Edge cases:** `s` that cannot be segmented at all (e.g., LeetCode's own second example, `s="catsandog"` against a dictionary that almost, but not quite, covers it — correctly resolves to `dp[n]=false`, with no special-casing needed beyond the recurrence itself); a dictionary word longer than `s` (never matches any substring, contributes nothing, no crash); `s` itself present verbatim in the dictionary (`j=0` alone resolves `dp[n]=true` immediately).

**💡 Interview Insight:** volunteering the O(n²)-vs-O(n³) distinction unprompted, along with the Trie-based fix, is a strong signal on this specific problem — it's commonly asked at exactly the depth of "can you push past the number everyone quotes."

---

## Problem 8: Coin Change (LeetCode 322, Medium) — Pattern: 1D DP, Unbounded Knapsack (NEW framing)

**Statement:** given coin denominations (unlimited supply of each) and a target `amount`, find the minimum number of coins summing to exactly `amount` — or `-1` if impossible.

**Why this is genuinely new — "unbounded" reuse, introduced now because it's needed now:** every DP problem this week so far has drawn from a **fixed, small set of earlier states** (`dp[i-1]`, `dp[i-2]`, or a validity-checked pair) — no problem allowed reusing the same "item" an unlimited number of times. Coin Change does: the same coin denomination can be used as many times as helpful, with nothing tracking how many times it's already been used.

**`dp[i]`, precisely:** the minimum number of coins needed to make amount `i` exactly.

**Why `dp[i] = min over every coin c ≤ i of (dp[i-c] + 1)` — an exchange-style optimal-substructure proof, not an assertion:** consider the optimal (fewest-coin) solution for amount `i`, and look at its **last coin used**, call it `c`. Removing that one coin leaves a sub-solution making amount `i-c`. That sub-solution must itself be optimal for `i-c` — if a cheaper way to make `i-c` existed, swapping it in (keeping the same final coin `c`) would produce a strictly cheaper solution for `i`, contradicting that the original was optimal. So the true minimum for `i` is exactly `1 +` the true minimum for `i-c`, for whichever `c` makes that smallest — trying every coin as the candidate "last coin used" and taking the min over all of them finds it.

**Why this recurrence allows unlimited reuse automatically, with no extra bookkeeping:** nothing in `dp[i-c] + 1` restricts which coins were used to build `dp[i-c]` — if the cheapest way to make `i-c` already used denomination `c` (possibly more than once), that's simply reflected in `dp[i-c]`'s value already, and using `c` again to reach `i` is neither prevented nor specially handled. The unbounded property isn't a rule that had to be added; it's a direct consequence of *not* tracking which coins were spent, only how many.

```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1); // sentinel — see below for why this exact value, not MAX_VALUE
    dp[0] = 0;

    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }

    return dp[amount] > amount ? -1 : dp[amount];
}
```

**⚠️ Why the sentinel is `amount + 1`, specifically, and not `Integer.MAX_VALUE`:** any genuinely achievable answer uses at most `amount` coins (the coarsest possible valid combination — one coin of value 1 repeated `amount` times, if a 1-value coin exists; any actual valid combination uses no more coins than that, since every coin is worth at least 1). `amount + 1` is therefore guaranteed larger than any real answer, cleanly signaling "unreached" — while still being small enough that `dp[i-coin] + 1` can **never overflow**. Seeding with `Integer.MAX_VALUE` instead would make that same `+1` operation overflow into a negative number the moment it's reached from an already-unset cell, corrupting the comparison silently. This is the same overflow discipline from Day 10/11, applied to sentinel selection specifically.

**Why loop order — coins-outer vs. amounts-outer — doesn't matter here, and why that's about to matter next week:** this problem takes a **minimum** over independent candidate coins, and `min()` doesn't care what order its candidates are considered in — trying coin `5` before coin `1` or after produces the same final minimum either way. This will *not* be true for every unbounded-reuse problem: a future problem that **counts distinct combinations**, rather than minimizing a count, needs a specific loop order to avoid counting the same combination once per possible ordering of its coins — flagged here as a distinction that's coming, not one this problem needs.

**Worked trace:** `coins = [1,2,5]`, `amount = 11`.

| i | dp[i] = min(dp[i-1]+1, dp[i-2]+1, dp[i-5]+1) [only where the coin ≤ i] | dp[i] |
|---|---|---|
| 0 | base | 0 |
| 1 | dp[0]+1 | 1 |
| 2 | min(dp[1]+1, dp[0]+1) = min(2,1) | 1 |
| 3 | min(dp[2]+1, dp[1]+1) = min(2,2) | 2 |
| 4 | min(dp[3]+1, dp[2]+1) = min(3,2) | 2 |
| 5 | min(dp[4]+1, dp[3]+1, dp[0]+1) = min(3,3,1) | 1 |
| 6 | min(dp[5]+1, dp[4]+1, dp[1]+1) = min(2,3,2) | 2 |
| 7 | min(dp[6]+1, dp[5]+1, dp[2]+1) = min(3,2,2) | 2 |
| 8 | min(dp[7]+1, dp[6]+1, dp[3]+1) = min(3,3,3) | 3 |
| 9 | min(dp[8]+1, dp[7]+1, dp[4]+1) = min(4,3,3) | 3 |
| 10 | min(dp[9]+1, dp[8]+1, dp[5]+1) = min(4,4,2) | 2 |
| 11 | min(dp[10]+1, dp[9]+1, dp[6]+1) = min(3,4,3) | 3 |

**Answer: 3** (`5+5+1=11`) — note `dp[11]` is reached via the `c=1` branch off `dp[10]=2`, and `dp[10]=2` itself was built from two `5`s — the same denomination reused twice in the same final answer, with nothing in the algorithm needing to know or track that.

**Complexity:** Time O(amount × numCoins), Space O(amount).

**Edge cases:** `amount = 0` (`dp[0]=0` directly, no coins needed, correctly returned without the loop body ever needing to run); no valid combination exists (e.g., `coins=[2]`, `amount=3` — every `dp[i]` for odd `i` stays at the sentinel, `dp[3]` never improves, `dp[3] > amount` → `-1`); a single coin denomination exactly equal to `amount`.

**💡 Interview Insight:** stating the sentinel-value reasoning (`amount+1`, not `MAX_VALUE`) unprompted is a small but real signal of overflow discipline; stating the loop-order independence — and *why* it's specific to minimization rather than a general property of unbounded problems — heads off a common follow-up question before it's asked.

---

## Career Block Guide (1 hr)

**Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.

**Weekly Scorecard, corrected:** Day 84, twelve weeks in — **164 total DSA problems solved** (required-ladder count, matching the plan's own convention). The plan's own internal scorecard states 163; this continues the same one-off drift the curriculum map has flagged twice already, first at Week 9, Day 63 (123 vs. the authoritative 124) and again at Week 11 (150 vs. 151) — the discrepancy has never been corrected inside the plan's own files, only tracked here. Full accounting, including the cumulative-distinct figure (which differs from the required-ladder figure because it excludes today's recapped problem and includes this week's one extra), is in the Week 12 Consolidation section below. Dijkstra's fully closed at 5 (up from 3) — both thin-pattern gaps from the original audit are now fixed. Dynamic Programming is 8 problems into its 33-problem run.

---

## Day 84 — Interview Questions

**Q1. Prove Word Break's recurrence correct.** Any valid full segmentation has a specific last word, starting at some position `j`; if `s[0..j)` is itself segmentable (`dp[j]` true) and `s[j..i)` is a dictionary word, then `s[0..i)` is segmentable — trying every possible `j` as the last word's start covers every possible valid segmentation.

**Q2. Why is the commonly-cited O(n²) complexity for Word Break not the full picture in Java?** `s.substring(j,i)` creates a genuine copy costing O(i-j), and hashing a freshly-built String costs another O(i-j) before the HashSet lookup — accounting for this honestly across all (i,j) pairs gives a true worst-case of O(n³), not O(n²).

**Q3. What fixes Word Break's extra complexity factor, and why isn't it built into the core solution here?** A Trie built from the dictionary, or index-based substring comparison avoiding new String allocation — flagged as extension material because it isn't needed to solve the problem as stated, only to tighten the bound under interviewer pressure.

**Q4. Why does Coin Change use a `Set`... wait, why does Word Break use a Set specifically, and what would a List cost instead?** A HashSet gives O(1) average membership checks; a List would force an O(k) linear scan through the dictionary on every one of the DP's up-to-O(n²) checks, compounding an already-large state count.

**Q5. Prove Coin Change's recurrence correct.** Consider the optimal solution for amount `i` and its last coin used, `c` — removing it leaves a sub-solution for `i-c` that must itself be optimal, or swapping in a cheaper one would produce a cheaper solution for `i`, a contradiction; trying every coin as the candidate last-used coin and taking the min finds the true optimum.

**Q6. Why does Coin Change's recurrence allow unlimited reuse of a coin without any explicit tracking?** Nothing in `dp[i-c]+1` restricts which coins built `dp[i-c]` — if reusing `c` again was already reflected in `dp[i-c]`'s own value, using it again is neither prevented nor specially permitted; the unbounded behavior falls out of simply not tracking which coins were spent.

**Q7. Why is the sentinel value `amount + 1`, not `Integer.MAX_VALUE`?** Any real achievable answer uses at most `amount` coins, so `amount+1` is guaranteed larger than any genuine result while remaining small enough that `dp[i-coin]+1` can never overflow — seeding with `MAX_VALUE` would overflow into a negative number the moment that `+1` is applied to an unset cell.

**Q8. Why doesn't loop order (coins-outer vs. amounts-outer) affect Coin Change's answer?** The recurrence takes a minimum over independent coin choices, and `min()` is order-independent — trying coins in any sequence produces the same smallest value.

**Q9. Will loop order matter for every unbounded-reuse DP problem?** No — it matters specifically when a problem counts distinct *combinations* rather than minimizing a value; counting needs a fixed loop order to avoid treating the same combination as multiple different orderings, a distinction that doesn't apply to a plain minimization like this one.

**Q10. State this week's corrected cumulative problem count, and explain the discrepancy with the plan's own number.** 164, using the required-ladder-only convention the plan itself uses — the plan states 163, continuing a one-off drift first documented at Week 9, Day 63 and never corrected in the plan's own files since.

---

## Daily Deliverable Check

- [ ] Word Break (LC 139) and Coin Change (LC 322) solved, with both correctness proofs explainable from memory, pushed.
- [ ] Can state precisely why Word Break's true complexity is O(n³) in the worst case, and name the fix.
- [ ] Can explain why Coin Change's sentinel is `amount+1` rather than `MAX_VALUE`, and why loop order doesn't matter for this specific problem.
- [ ] Weekly ritual and corrected scorecard complete.
- [ ] Self-check above completed — every `dp[i]` definition from this week restated without notes.

---

## What Tomorrow Assumes You Already Know Cold

Day 85 opens leave week 2 — pure DSA immersion, no new theory, no project tasks, at full-time intensity — and its very first problem, Coin Change II, needs today's unbounded-knapsack mechanism cited directly, not re-derived, along with the loop-order distinction flagged above actually built out for real this time. General 1D DP fluency (the `dp[i]`-definition-first discipline, exhaustive-case proofs) needs to be fully automatic across all eight of this week's problems, since next week runs at a pace that assumes no time spent re-deriving anything from this week. `HashSet` (Word Break, today) and `HashMap` (memoization, all week) both need to stay reflexive — Word Break II (Day 86) reuses today's exact `dp[]` boolean array as a pruning structure before backtracking to build actual output.

---

# Week 12 Consolidation

**What actually got built:** Dijkstra's Algorithm, opened and closed within three days (Days 78–80) — 5/5 required, 0 extra, both thin-pattern gaps from the original audit now closed. Dynamic Programming opened (Day 81) and is four days into its run — 8/33 required across Days 81–84 (7 genuinely new + 1 recap, Maximum Product Subarray/LC 152, caught against Week 4, Day 22 and not re-taught), plus 1 extra (Delete and Earn/LC 740, Day 82, checked clean against both the cumulative problem table and `Week_13_Revised.md`).

**Planned vs. actual:** the plan called for 13 required problems this week (5 Dijkstra + 8 DP); all 13 slots were filled exactly as planned, with one of them (LC 152) delivered as a recap rather than a fresh teach once the overlap was caught. One extra was added (Delete and Earn), against a default of zero — justified specifically because House Robber's family benefits from a third, differently-shaped variant, and Dijkstra's already-expanded required ladder needed nothing further. Net new-distinct problems taught this week: **13** (12 new-required + 1 new-extra; the recap doesn't count twice).

**Corrected running totals:**

| Convention | Through Week 11 | Week 12 | Through Week 12 |
|---|---|---|---|
| Required-ladder-only (matches plan's own convention) | 151 | +13 | **164** |
| Cumulative distinct (all real solves, extras included, no double-counting recaps) | 189 | +13 | **202** |

The plan's own Day 84 scorecard states "163" — one below the authoritative 164, continuing the identical one-off drift already flagged twice in this document (Week 9, Day 63: 123 vs. 124; Week 11: 150 vs. 151). The drift appears to originate from a single uncorrected miscount several weeks back, carried forward additively without ever being reconciled inside the plan's own files. Noted here again for the same reason it was worth noting the first two times: the gap is small, consistent, and easy to keep propagating silently if nobody checks the arithmetic against the row-by-row table.

**Short diagnostic list — resolve any gaps here before Monday, not during leave week itself:**
- Can you state Dijkstra's greedy-correctness proof, including exactly where non-negative edge weights are used, without notes?
- Can you explain Bellman-Ford's snapshot-vs-in-place relaxation distinction and why skipping it breaks the round-bounded guarantee?
- Can you state, precisely, why House Robber II needs an explicit `n=1` special case that its general two-case reduction doesn't cover?
- Can you explain why Word Break's true worst-case complexity is O(n³), not the commonly-quoted O(n²)?
- Can you explain why Coin Change's loop order doesn't matter for minimization but will matter for a future combination-counting variant?
- Is the "state `dp[i]` precisely before writing any recurrence" discipline fully automatic, or still something you have to consciously remember to do?

**What Week 13 assumes:** Week 13 is leave week 2 (Days 85–91) — no new theory, no project tasks, full-time DSA immersion, closing four DP subtypes across seven days (the rest of 1D DP, all of Grid DP, all of String DP, all of Interval DP). Day 85's Coin Change II needs today's unbounded-knapsack mechanism solid, plus the loop-order distinction actually resolved this time (counting combinations, not minimizing a value — the reason loop order suddenly matters). Day 86's Word Break II needs today's Word Break `dp[]` array reused directly as a pruning structure. Week 13 will also need to formally introduce **0/1 Knapsack** (Partition Equal Subset Sum, Target Sum) — bounded, each item usable at most once — as a distinct counterpart to this week's **Unbounded Knapsack** (Coin Change, unlimited reuse); that distinction is flagged here as coming, not built, since it belongs to next week's own generation. Everything else — recursion, HashMap, HashSet, the exhaustive-case-proof habit — carries forward as load-bearing prerequisite, not optional review.
