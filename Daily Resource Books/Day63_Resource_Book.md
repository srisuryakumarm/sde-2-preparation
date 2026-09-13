# Day 63 (Sunday) — Consolidation, and Backtracking Continues

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 62 Resource Book](Day62_Resource_Book.md)
**Next ▶:** Day 64 Resource Book (Week 10)
**Companion to:** Day 63 of `Week_09_Revised.md`

---

## Recap

Permutations (yesterday) fixed one position at a time and swapped every remaining candidate into it — a decision structure where each choice depends on what earlier positions already claimed. Today's two problems extend that same family in two different directions: Combinations drops the "every element must be placed somewhere" requirement entirely (order doesn't matter, and not every element appears), while Permutations II reintroduces yesterday's exact swap mechanism but must now also suppress duplicate output when the input itself has repeated values — a genuinely new correctness requirement layered on an already-known technique.

## Learning Objectives

By the end of today, without notes:

1. Solve Combinations using forward-only index recursion, and explain why it naturally avoids duplicate combinations without needing a "used" structure at all.
2. Solve Permutations II, and prove — via a full worked trace, not just a stated rule — exactly why the `!used[i-1]` duplicate-skip condition is correct and why the seemingly-similar `used[i-1]` would be wrong.
3. Give the complete, final accounting of Backtracking's status this week, including the one collision this process caught and avoided.

## Concept Dependency Map

```
Backtracking: include/exclude (Day 61) + swap-based (Day 62)
        │
        ├─ NEW: forward-index recursion (Combinations)
        │    — never revisit an earlier index, so no
        │    duplicate-avoidance structure is needed at all
        │
        └─ NEW: duplicate-value suppression, layered on
             yesterday's swap mechanism (Permutations II)
        │
        ▼
Week 9 closes: Heaps 10/10, Tries 6/6 core, Backtracking 4/12
   (continues Week 10 — already dense, no extras needed)
        │
        ▼
🔗 Week 10, Day 64: Combination Sum — the next backtracking
   variant, reusing forward-index recursion from today,
   with one new wrinkle (elements may repeat)
```

---

## Part 1 — Combinations

### Problem 13: Combinations (LeetCode 77, Medium) — Pattern: Backtracking

**Statement:** Given integers `n` and `k`, return every possible combination of `k` numbers chosen from `1` to `n` (order doesn't matter — `[1,2]` and `[2,1]` are the same combination and must appear only once).

```java
public static List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(1, n, k, new ArrayList<>(), result);
    return result;
}

private static void backtrack(int start, int n, int k, List<Integer> path, List<List<Integer>> result) {
    if (path.size() == k) {
        result.add(new ArrayList<>(path));
        return;
    }
    for (int i = start; i <= n; i++) {
        path.add(i);
        backtrack(i + 1, n, k, path, result); // i + 1, never start over — this is the whole mechanism
        path.remove(path.size() - 1);
    }
}
```

**Why forward-only recursion (`i + 1`, never revisiting anything ≤ `i`) is the entire duplicate-avoidance mechanism, with nothing extra needed:** since order doesn't matter, every valid combination has exactly one representation as an *increasing* sequence. By only ever choosing the *next* number forward from wherever the last choice left off, the recursion can only ever produce numbers in increasing order — meaning it can only ever produce each combination in its one canonical (increasing) form, never in any other order that would count as a duplicate. Contrast this directly with Permutations (yesterday), where order *does* matter and every element must eventually be tried in every position — forward-only recursion would have been *wrong* there, since it would have silently skipped every reordering.

### Extension: pruning when not enough elements remain

```java
// If (n - i + 1) < (k - path.size()), stop the loop early —
// not enough remaining numbers to ever complete this combination.
if (n - i + 1 < k - path.size()) break;
```

Not required for correctness (the base loop already terminates correctly without it), but a well-known, easy-to-state optimization — worth naming if an interviewer asks "can you prune this further," since it shows the search-space-shape isn't just accepted as given.

### Worked trace

`n = 4, k = 2`.

`backtrack(1, ...)`: `i=1`: path=`[1]`. `backtrack(2,...)`: `i=2`: path=`[1,2]` → size 2 = k → **add `[1,2]`**. undo. `i=3`: path=`[1,3]` → **add `[1,3]`**. undo. `i=4`: path=`[1,4]` → **add `[1,4]`**. undo. Return, undo path=`[1]`→`[]`.
`i=2`: path=`[2]`. `backtrack(3,...)`: `i=3`: path=`[2,3]` → **add `[2,3]`**. undo. `i=4`: path=`[2,4]` → **add `[2,4]`**. undo. undo→`[]`.
`i=3`: path=`[3]`. `backtrack(4,...)`: `i=4`: path=`[3,4]` → **add `[3,4]`**. undo. undo→`[]`.
`i=4`: path=`[4]`. `backtrack(5,...)`: loop `i=5` to `4` — empty range, nothing happens. undo→`[]`.

**Result: `[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]`** — 6 combinations, matching `C(4,2) = 6` exactly.

### Complexity

**Time: O(C(n,k) × k)** — `C(n,k)` combinations produced, each costing `O(k)` to copy into the result.
**Space: O(k)** auxiliary — recursion depth equals `k` at most.

### Edge cases

- `k = 0` → the empty combination `[]` is the single valid answer (base case hits immediately at the top level; not exercised by this problem's usual constraints, but worth knowing the code handles it correctly regardless).
- `k = n` → exactly one combination exists (every number), and the loop structure naturally produces only that one path.
- `k > n` → no valid combinations; the loop structure naturally produces none (every branch runs out of remaining numbers before `path.size()` reaches `k`), no special-case check required.

### Interview framing

**Say before coding:** "Since order doesn't matter, I'll only ever recurse forward from the index just chosen — that alone guarantees every combination is built in exactly one canonical increasing order, so no duplicate-avoidance structure is needed beyond the forward-only recursion itself."
**Likely follow-up:** "Can you prune the search further?" → the remaining-elements pruning shown above.

---

## Part 2 — Permutations II

### Problem 14: Permutations II (LeetCode 47, Medium) — Pattern: Backtracking with Duplicate Handling

**Statement:** Given an array of integers that **may contain duplicates**, return every distinct permutation.

### Approach — sort, used-array, skip-condition

```java
public static List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums); // duplicates must be adjacent for the skip-check below to work
    List<List<Integer>> result = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), result);
    return result;
}

private static void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> result) {
    if (path.size() == nums.length) {
        result.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue; // the duplicate-skip condition
        used[i] = true;
        path.add(nums[i]);
        backtrack(nums, used, path, result);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

### Why `!used[i-1]` — not `used[i-1]` — is the correct condition, proven by trace

This is the single most confusing line in this week's material, and it deserves a complete, exhaustive trace rather than a stated rule.

`nums = [1, 1, 2]` (already sorted). `used = [F,F,F]`, `path = []`.

**Top level, `i=0`:** `used[0]=false`. Dedup check: `i>0`? No (`i=0`) → check doesn't apply. **Choose `nums[0]=1`**: `used[0]=true`, `path=[1]`. Recurse.
&nbsp;&nbsp;**`i=0`:** used → skip. **`i=1`:** `used[1]=false`. Dedup: `nums[1]==nums[0]` (1==1, yes) **and** `!used[0]`? `used[0]` is currently `true` → `!used[0]` is `false` → condition is **false overall** (needs both to be true) → **don't skip.** Choose `nums[1]=1`: `path=[1,1]`. Recurse.
&nbsp;&nbsp;&nbsp;&nbsp;**`i=2`:** `used[2]=false`. Dedup: `nums[2]==nums[1]`? `2==1`? No → proceed. Choose `nums[2]=2`: `path=[1,1,2]` → size 3 → **add `[1,1,2]`**. Undo → `path=[1,1]`, `used[2]=false`.
&nbsp;&nbsp;Undo choice at `i=1` → `path=[1]`, `used[1]=false`.
&nbsp;&nbsp;**`i=2`:** `used[2]=false`. Dedup: `nums[2]==nums[1]`? No → proceed. Choose `nums[2]=2`: `path=[1,2]`. Recurse.
&nbsp;&nbsp;&nbsp;&nbsp;**`i=1`:** `used[1]=false`. Dedup: `nums[1]==nums[0]` (yes) **and** `!used[0]`? `used[0]` still `true` (outer choice active) → `!used[0]=false` → don't skip. Choose `nums[1]=1`: `path=[1,2,1]` → **add `[1,2,1]`**. Undo.
&nbsp;&nbsp;Undo choice at `i=2` → `path=[1]`, `used[2]=false`.
Undo top-level choice at `i=0` → `path=[]`, `used[0]=false`.

**Top level, `i=1`:** `used[1]=false`. Dedup: `i>0` (yes), `nums[1]==nums[0]` (1==1, yes) **and** `!used[0]`? `used[0]` is now `false` (just undone above) → `!used[0]=true` → condition is **true** → **SKIP.** This correctly prevents exploring "start with the *second* copy of 1" as if it were a distinct choice from "start with the first copy of 1" — every permutation reachable from this branch would exactly duplicate one already found starting from `i=0`.

**Top level, `i=2`:** `used[2]=false`. Dedup: `nums[2]==nums[1]`? No → proceed. Choose `nums[2]=2`: `path=[2]`. Recurse.
&nbsp;&nbsp;**`i=0`:** `used[0]=false`. Dedup: `i>0`? No → proceed regardless. Choose `nums[0]=1`: `path=[2,1]`. Recurse.
&nbsp;&nbsp;&nbsp;&nbsp;**`i=1`:** `used[1]=false`. Dedup: `nums[1]==nums[0]` (yes) **and** `!used[0]`? `used[0]` is `true` (just chosen) → `!used[0]=false` → don't skip. Choose `nums[1]=1`: `path=[2,1,1]` → **add `[2,1,1]`**. Undo.
&nbsp;&nbsp;Undo → `path=[2]`, `used[0]=false`.
&nbsp;&nbsp;**`i=1`:** `used[1]=false`. Dedup: `nums[1]==nums[0]` (yes) **and** `!used[0]`? `used[0]` now `false` (undone) → `!used[0]=true` → **SKIP** — same reasoning as the top-level skip: this would duplicate the branch just explored starting with `nums[0]`.
&nbsp;&nbsp;**`i=2`:** `used[2]=true` → skip.

**Final result: `[1,1,2], [1,2,1], [2,1,1]`** — exactly 3 distinct permutations, matching `3!/2! = 3` (3 elements, one duplicate pair).

**The rule, stated precisely now that it's proven:** `!used[i-1]` being true means "the previous copy of this same value is *not* currently part of the active path" — which can only happen if we already fully explored the branch that used it *at this same recursion depth*, then backtracked past it. Choosing this copy now would just re-explore an equivalent branch. When `used[i-1]` **is** true (the previous copy *is* part of the currently-active path, one level up), choosing this copy is using a *second, distinct* instance of the duplicate value within one single permutation — which is required, not forbidden (see `[1,1,2]` itself, which legitimately uses both 1's).

> ⚠️ **Common Mistake:** writing `used[i-1]` instead of `!used[i-1]`, reasoning "skip if the previous identical value was already used." This is backwards — it would incorrectly block legitimate within-path reuse of a duplicate value (breaking `[1,1,2]` itself) while failing to block the actual sibling-branch duplicate this check exists to catch.

### Complexity

**Time: O(n × n!)** worst case — same order as ordinary Permutations (bounded by the total decision tree size), even though duplicate values reduce the actual *output* count below `n!`.
**Space: O(n)** — recursion depth plus the `used` array.

### Edge cases

- All elements identical — dedup logic collapses the entire tree down to exactly one output permutation, correctly.
- No duplicates at all — the dedup condition never fires (`nums[i] == nums[i-1]` never true), and behavior is identical to ordinary Permutations.
- Exactly one duplicate pair (traced above) — the minimal case that actually exercises the skip logic.

### Interview framing

**Say before coding:** "I'll sort first so duplicates are adjacent, then at each recursion depth, skip a candidate if it equals the previous one *and* the previous one isn't currently part of the active path — that specific condition prevents re-exploring an equivalent sibling branch, without blocking legitimate reuse of a duplicate value within one path."
**Likely follow-up:** "Walk me through why `!used[i-1]`, not `used[i-1]`." — have the `[1,1,2]` trace above ready; this is one of the few points this week worth having fully memorized rather than re-derived live, given how easy it is to get backwards under pressure.

---

## Backtracking's Status — Full Accounting, Including the Overlap Catch

4 of 12 required Backtracking problems are done: Subsets, Permutations, Combinations, Permutations II. The remaining 8 (Combination Sum, Combination Sum II, Letter Combinations of a Phone Number, Generate Parentheses, Word Search, Palindrome Partitioning, Subsets II, N-Queens) are already fully scoped in `Week_10_Revised.md`, closing the pattern at 12/12 on Day 67.

**No extra practice was added to Backtracking this week**, for reasons that compound rather than any single one being sufficient alone:

1. **The pattern is still in its genuine opening arc.** Per this series' established convention (no extras on an opening day, deferred reps land on a later day of the *same* pattern), Subsets (Day 61, opening) correctly received none. Permutations (Day 62) and today's two problems are still early reps of a pattern that only reaches its halfway point today — there isn't yet a natural "reinforcement" day the way Trees or Heaps had multiple non-opening days within their own week to place extras on.
2. **Week 10's required ladder is already dense.** Every Backtracking day in `Week_10_Revised.md` carries 2 required problems, with no gaps — Combination Sum + Combination Sum II, Letter Combinations + Generate Parentheses, Word Search + Palindrome Partitioning, Subsets II + N-Queens. A 12-problem required ladder spanning warm-up through Hard already covers essentially every canonical variant (include/exclude, permutation with and without duplicates, combination with and without reuse, string-building from per-position choices, matrix backtracking, palindrome partitioning, and N-Queens as the constraint-satisfaction capstone) — there's no thin coverage here to patch.
3. **A specific collision was caught and avoided.** Letter Combinations of a Phone Number (LC 17) was seriously considered as extra practice today — it's exactly the kind of high-frequency, distinct-flavored backtracking problem (string-building from per-position choices, unlike anything in this week's four) that would normally be a strong pick. Checking `Week_10_Revised.md` first showed it's already Day 65's **required** Problem 7. Adding it here would have meant Day 65's generation either re-teaching a problem already fully covered, or silently absorbing the duplication — precisely the failure mode the overlap-check process exists to catch. It was not added.

This gets flagged explicitly, per the standing instruction to surface overlap findings rather than silently resolving them: **the near-miss on LC 17 is the concrete example this week of the overlap-check mechanism doing its job**, not just a theoretical safeguard.

---

## Career Block (1 hr)

**Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.

**Weekly Scorecard:** Day 63, nine weeks in. `Week_09_Revised.md`'s own running total states **123 total DSA problems solved** — but that figure is one short of what this week's own day-by-day problem list actually adds up to, and it's worth flagging rather than quietly repeating. Weeks 1–8 closed at **110 required problems** (97 through Week 7 + 13 in Week 8, per `00_Curriculum_Map.md`). Counting this week's own problems day by day — Day 57 (2), Day 58 (2), Day 59 (2), Day 60 (2), Day 61 (3 — Replace Words, Word Search II, *and* Subsets), Day 62 (1), Day 63 (2) — gives **14**, not 13; Day 61 alone carries one more problem than every other day this week, the likely spot a manually-tracked running total drifted by one. **110 + 14 = 124 required problems through Week 9**, all solved, zero added as extra practice this week specifically.

Across the full series, Weeks 1–8 also added **35 extra-practice problems** beyond the required ladder (145 distinct through Week 8, per `00_Curriculum_Map.md`'s Running Totals, minus the 110 required = 35 extra). Week 9 adds its 14 required problems on top, with no extra practice of its own — **145 + 14 = 159 distinct problems solved, cumulative, through Day 63.**

Heaps fully closed at 10/10 required (13/13 distinct with Week 8's 3 extra) — the single biggest gap fix from the original audit, genuinely delivered. Tries' core fully closed at 6/6, deliberately lean, its 7th problem correctly deferred to Bit Manipulation. Backtracking is 4/12 into an already well-scoped, comprehensive run that closes cleanly in Week 10 with no padding needed on either side. `scalable-ecommerce-platform` is live with a real four-module skeleton and working AOP — five weeks earlier than the original plan's equivalent project, and it's the only backend project you'll be building from here forward.

---

## Week 9 Consolidation

### What actually got built

- **Heaps closed** at 10/10 required problems (Reorganize String, Task Scheduler, Find Median from Data Stream, Merge k Sorted Lists), 13/13 distinct combined with Week 8 — six genuinely distinct heap roles demonstrated across the two weeks (kth-largest, keep-the-k-best, minimize-combination-cost, candidate-generator, greedy-scheduling, and this week's two-heap-balance/k-way-merge).
- **Tries opened and closed its core** in exactly 3 days (Days 59–61) at 6/6 required problems, spanning basic operations, cumulative value aggregation, constrained traversal, branching traversal, dictionary substitution, and Trie-pruned matrix backtracking. Its 7th canonical problem remains correctly deferred to Bit Manipulation.
- **Backtracking opened** at Day 61 and reached 4/12 required problems by week's end (Subsets, Permutations, Combinations, Permutations II) — on track to close cleanly in Week 10 with no gaps needing to be patched later.
- **System design theory:** CAP Theorem, Consistent Hashing, Replication Models — a coherent three-day arc, each topic explicitly building on the previous (CAP's C-vs-A trade-off → how a ring topology avoids one specific A-cost of naive resharding → how replication strategy is the same underlying trade-off applied to write propagation).
- **Spring AOP** — Aspect/Pointcut/Advice, the proxy mechanism, and the self-invocation limitation, applied via a working `@LogExecutionTime` aspect.
- **`scalable-ecommerce-platform` initialized** — four independently-compiling Maven modules (Product, Order, Payment, Notification) sharing version management through a parent POM, five weeks ahead of the original plan's schedule for the equivalent project. `todo-api` is now feature-complete and stable, receiving no further changes.

### Planned vs. actual problem count

| | Planned (required) | Actual (required + extra) |
|---|---|---|
| Heaps (this week's share) | 4 | 4 (0 extra — see Day 58's reasoning) |
| Tries | 6 | 6 (0 extra — see Day 61's reasoning) |
| Backtracking (this week's share) | 4 | 4 (0 extra — see today's reasoning) |
| **Week 9 total** | **14** | **14** |

Every problem this week was plan-required; zero extra practice was added — the first week in this series where that's true across the board, and each of the three zero-extra decisions has its own distinct, explicitly-stated justification rather than being a single blanket policy.

### Short diagnostic list — confirm before moving on

- [ ] Can you state, without looking, the four heap-of-size-k roles from Week 8 *plus* the two new roles from this week — six total, each with a one-line "why this shape" justification?
- [ ] Can you explain why a Trie beats a HashSet for prefix queries specifically, in one sentence, without hedging?
- [ ] Can you write the choose/explore/un-choose backtracking template from memory, and explain in your own words why the un-choose step is structurally necessary (not habit)?
- [ ] Can you reproduce the Permutations II `[1,1,2]` trace and correctly explain `!used[i-1]` vs. `used[i-1]` without re-deriving it from first principles under time pressure?
- [ ] Can you state the CAP theorem's precise formulation (not the "pick 2 of 3" shorthand) and name one system that defaults CP and one that defaults AP, correctly hedged as defaults, not fixed classifications?
- [ ] Can you explain Spring AOP's self-invocation limitation and why it happens mechanistically (proxy placement), not just that it happens?

If any of these require re-deriving from scratch rather than stating directly, that's worth another pass before Week 10 — which assumes every item above is fully reflexive.

### What Week 10 assumes

Week 10 assumes the full heap mechanism, the complete Trie toolkit, and today's swap-based/forward-index backtracking mechanisms are all fully reflexive — none of Week 10's 8 remaining Backtracking problems re-derive the choose/explore/un-choose template from scratch, and Combination Sum specifically extends today's Combinations forward-index recursion with exactly one new wrinkle (elements may be reused) rather than teaching forward-index recursion again. It also assumes today's multi-module Maven skeleton and working AOP aspect are stable, since Week 10 builds Resilience4j, Feign, and a Gateway directly inside this week's module structure without restructuring it.

---

## Day 63 — Interview Questions

**Q1. Why does forward-only recursion (`i+1`, never revisiting) fully solve Combinations' duplicate-avoidance problem with no extra structure needed?** Since order doesn't matter, every valid combination has exactly one increasing-order representation; only ever choosing forward from the last pick means the recursion can only ever produce that one canonical increasing form, so no duplicate can ever arise in the first place.

**Q2. Why would forward-only recursion be the *wrong* choice for Permutations?** Order matters there — every element must eventually be tried in every position, including positions "behind" where an earlier choice landed. Forward-only recursion would silently skip every reordering, undercounting the true permutation set.

**Q3. State the Permutations II duplicate-skip condition precisely, and explain why it's `!used[i-1]`, not `used[i-1]`.** Skip candidate `i` if `nums[i] == nums[i-1]` **and** `!used[i-1]` (the previous identical value is *not* currently part of the active path). `!used[i-1]` true means we already fully explored and backtracked past using the previous copy at this exact depth — choosing this copy now would just re-explore an equivalent sibling branch. `used[i-1]` being true instead means the previous copy is actively part of the *current* path one level up, and choosing this copy now is legitimate reuse of a duplicate value within one permutation, not a forbidden repeat.

**Q4. Give the complete reasoning for why Backtracking received zero extra practice this week.** The pattern is still in its genuine opening arc with no natural non-opening reinforcement day yet; Week 10's required ladder is already dense (2 problems/day, no gaps) and comprehensively spans every canonical variant; and a specific candidate (Letter Combinations of a Phone Number) was checked against `Week_10_Revised.md` and found to already be its required Day 65 problem — confirming the overlap-check process catching a real collision, not just a theoretical safeguard.

**Q5. What's the cumulative distinct-problem count through Day 63, and how does it reconcile against the plan's own "123" figure?** 145 distinct problems through Week 8 (110 required + 35 extra across the series, per the curriculum map's running totals) plus this week's 14 newly-solved (Heaps' remaining 4 + Tries' 6 + Backtracking's 4, zero extra) — 159 distinct total. The plan's own "123" figure (Day 63's scorecard in `Week_09_Revised.md`) undercounts by one: it implies 13 required problems for Week 9, but the plan's own day-by-day list for this week sums to 14 (Day 61 alone carries three — Replace Words, Word Search II, and Subsets — one more than any other day this week, the likely source of the drift). 110 + 14 = 124 required through Week 9, and 145 + 14 = 159 distinct overall — this book uses the verified row-by-row count rather than propagating the plan's own scorecard total.

**Q6. Name the six distinct heap roles demonstrated across Weeks 8–9.** Min-heap for kth-largest queries; max-heap for keeping the k best; min-heap for minimizing combination cost; heap as a candidate generator; max-heap for greedy scheduling (this week, Day 57); and two-heap balance-invariant statistics tracking plus heap-as-merge-coordinator (this week, Day 58).

**Q7. Why did Tries close in exactly 3 days with zero extra practice, while Backtracking — also opening this week — gets a full 2 weeks and still adds none either?** Different reasons for the same outcome: Tries' required set (6) already spans every major role and the pattern is inherently short (no non-opening day exists within its own arc to place extras on). Backtracking's required set spans 2 full weeks and 12 problems specifically *because* it's a larger, higher-interview-weight pattern — but that larger required ladder is itself already comprehensive enough that no padding is needed; the absence of extras reflects thoroughness of the required set in both cases, via two different mechanisms (short-and-complete vs. long-and-complete).

---

## Daily Deliverable Check

- [ ] Combinations and Permutations II solved, pushed to `dsa-java/backtracking/`.
- [ ] Can reproduce the full `[1,1,2]` Permutations II trace from memory, including both skip points and why each fires.
- [ ] Weekly ritual and scorecard complete — cumulative distinct-problem count (159) reconciled against the plan's own required-only figure (123, one short of this week's own verified day-by-day count of 124).
- [ ] Week 9 Consolidation diagnostic checklist above completed honestly — any unchecked item revisited before Week 10.
- [ ] `00_Curriculum_Map.md` and `Week9_Interview_Questions.md` reviewed (see below) as this week's permanent reference.

---

## Week 9 → Week 10

Week 10 continues Backtracking directly from today's stopping point (Combination Sum, Day 64) and, once it closes at Day 67, opens Graphs — a genuinely new pattern, first appearance anywhere in this series, built on the same recursion-and-explicit-state foundation as this week's two patterns but requiring a `visited` set for the first time (unlike trees, graphs can contain cycles). On the platform side, Week 10 builds Resilience4j, Feign, an API Gateway, JWT, rate limiting, Kafka Schema Registry, and a full Saga choreography flow — all directly inside the module skeleton this week initialized.

**This week's permanent artifacts:** [`00_Curriculum_Map.md`](00_Curriculum_Map.md) (extended below) and [`Week9_Interview_Questions.md`](Week9_Interview_Questions.md) (consolidating every Q&A pair from Days 57–63).

**Next:** Day 64 Resource Book (Week 10) — Backtracking: Combination Sum and Its Duplicate-Handling Twin, and Resilience4j.
