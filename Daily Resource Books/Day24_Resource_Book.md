# Day 24 — Greedy Continues, and Intervals Begins

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 23 Resource Book](Day23_Resource_Book.md)
**Next ▶:** [Day 25 Resource Book](Day25_Resource_Book.md)
**Companion to:** Day 24 of `Week_04_Revised.md`

---

## Recap

Yesterday formalized greedy (definition, exchange-argument proof shape, the 0/1 Knapsack counter-example showing where it fails) and applied it to three problems: unlimited stock trading (telescoping decompositions), Jump Game (frontier domination), and Jump Game II (BFS-by-levels in disguise). Today opens with one more, harder greedy problem — Gas Station, which needs a genuinely different flavor of exchange argument, a *prefix-elimination* argument rather than a domination argument — and then introduces **Intervals** as a sibling pattern: a completely different mechanism (sort, then sweep once) that happens to combine naturally with greedy reasoning throughout the rest of this week.

Today has no Theory, Project, or Career Block in the plan — DSA only, at the full 4–5 hour leave-week intensity.

---

## Learning Objectives

By the end of today, without notes:

1. Prove Gas Station's greedy reset-point rule via a prefix-elimination argument: why failing at some station `i` after starting from `start` means every station between `start` and `i` can be eliminated as a candidate too, not just `start` itself.
2. State the Intervals pattern's core mechanism — sort by start time, sweep once, merge on overlap — and identify the exact overlap condition (`current.start <= last.end`, inclusive).
3. Solve Insert Interval in O(n), exploiting the problem's already-sorted input, rather than defaulting to Merge Intervals' O(n log n) general algorithm.
4. Recognize when a greedy strategy needs a **second** sort key to be well-defined (Queue Reconstruction by Height), not just a single comparator field.

---

## Concept Dependency Map

```
Yesterday (Day 23): Greedy formalized + exchange argument + frontier domination (LC 55, 45)
        │
        ▼
LC 134 Gas Station — greedy, PREFIX-ELIMINATION argument (needs: running-total tracking,
        the same shape as prefix sum's running accumulation, Day 21-22 — but a genuinely
        different proof shape than yesterday's domination argument)
        │
        ▼
NEW — Intervals: sort by START time, sweep once, merge on overlap
   (needs: sorting — Week 1 Day 3; arrays — Week 1 Day 2. No new data structure.)
├─ LC 56 Merge Intervals — the base mechanism
├─ LC 57 Insert Interval — same mechanism, exploiting pre-sorted input for O(n)
└─ Extra: LC 406 Queue Reconstruction by Height — greedy with a SECOND sort key
        │
        ▼
Tomorrow: Intervals sorted by END time instead — a different greedy key, different question
```

---

# Part 1 — Gas Station: A Different Flavor of Greedy Proof

## Problem: Gas Station (LeetCode 134, Medium) — Pattern: Greedy

**Statement:** there are `n` gas stations arranged in a circle. `gas[i]` is the amount of gas at station `i`; `cost[i]` is the gas needed to travel from station `i` to station `i+1`. Starting at some station with an empty tank, determine the starting station index that allows completing the full circuit, or return `-1` if no such station exists. If a solution exists, it's guaranteed unique.

### Approach 1 — Brute force

```java
public static int canCompleteCircuitBruteForce(int[] gas, int[] cost) {
    int n = gas.length;
    for (int start = 0; start < n; start++) {
        int tank = 0;
        boolean madeIt = true;
        for (int steps = 0; steps < n; steps++) {
            int i = (start + steps) % n;
            tank += gas[i] - cost[i];
            if (tank < 0) { madeIt = false; break; }
        }
        if (madeIt) return start;
    }
    return -1;
}
```

Try every possible starting station, simulate the full circuit from each. Time O(n²), Space O(1).

### Approach 2 — Optimized: greedy, one pass with a reset rule

```java
public static int canCompleteCircuit(int[] gas, int[] cost) {
    int totalTank = 0, currentTank = 0, start = 0;

    for (int i = 0; i < gas.length; i++) {
        int delta = gas[i] - cost[i];
        totalTank += delta;
        currentTank += delta;
        if (currentTank < 0) {
            start = i + 1;    // no station from [old start .. i] can work either — see proof
            currentTank = 0;
        }
    }
    return totalTank >= 0 ? start : -1;
}
```

**Claim 1 — a solution exists if and only if `totalTank >= 0` (total gas at least covers total cost).** If total gas is less than total cost, completing any full circuit is arithmetically impossible regardless of starting point — immediate `-1`. The more interesting direction — that `totalTank >= 0` **guarantees** some valid start exists — follows from Claim 2 below, since Claim 2 shows the algorithm always finds a candidate whenever one is possible.

**Claim 2 — the prefix-elimination argument, proven, not asserted:** suppose starting from `start`, the running tank first goes negative at station `i`. Then **no station `j` with `start <= j <= i` can be a valid starting point either.** Here's why: for any such `j`, the running tank accumulated from `start` up through `j` is non-negative (if it had gone negative before `j`, the failure would have been detected earlier, at that point, not at `i`) — call this accumulated surplus `S >= 0`. Starting fresh at `j` instead begins with tank `0`, strictly less than or equal to the `S` that continuing from `start` would have already banked by the time it reaches `j`. Since continuing from `start` (with its head start of `S`) *still* goes negative by station `i`, starting fresh at `j` (with no head start at all) goes negative **at least as early** — it cannot possibly do better. This holds for *every* `j` in `[start, i]` simultaneously, so the entire range is eliminated in one shot, and the algorithm is justified in jumping the search directly to `i + 1` without individually re-testing any station in between.

**Why this guarantees termination at a valid answer whenever `totalTank >= 0`:** each reset moves `start` strictly forward, and the total number of resets is bounded by `n` (there are only `n` stations to ever become the new `start`). If the tank never goes negative again after the final reset, the loop completes with `start` pointing at a station from which the running tank (restarted at `0` from that point) never dips below zero for the remainder of the array — and because `totalTank >= 0` overall, the *wraparound* portion (from the end of the array back to the original `start`) is guaranteed not to undo that, since the full circuit's total surplus is non-negative and everything before the final `start` has already been shown incapable of sustaining a full circuit.

**Worked trace:** `gas = [1, 2, 3, 4, 5]`, `cost = [3, 4, 5, 1, 2]`. Deltas: `-2, -2, -2, +3, +3`.

| i | delta | totalTank | currentTank | currentTank < 0? | start (after) |
|---|---|---|---|---|---|
| 0 | -2 | -2 | -2 | yes | 1, reset currentTank=0 |
| 1 | -2 | -4 | -2 | yes | 2, reset currentTank=0 |
| 2 | -2 | -6 | -2 | yes | 3, reset currentTank=0 |
| 3 | +3 | -3 | 3 | no | 3 |
| 4 | +3 | 0 | 6 | no | 3 |

`totalTank = 0 >= 0`, so a solution exists: `start = 3`. Verify by simulating from station 3: tank after station 3: `4-1=3`; after station 4 (wrapping): `3+5-2=6`; after station 0: `6+1-3=4`; after station 1: `4+2-4=2`; after station 2: `2+3-5=0`. Never negative — valid circuit. Matches the known expected output for this exact input.

**Complexity:** Time O(n) — one pass; the elimination argument is precisely what avoids the brute force's re-testing of every individual station. Space O(1).

**Edge cases:** `totalTank == 0` exactly (still valid — "at least covers" includes equality, as traced above); a single station (`gas[0] >= cost[0]` required and sufficient — the loop's single iteration either resets or doesn't, correctly); every station's delta negative except one large surplus station (the algorithm correctly resets past every failing station until it reaches the one true valid start, without needing to distinguish "large surplus" from "small surplus" — the elimination logic is agnostic to magnitude).

💡 **Interview Insight:** the prefix-elimination proof is the entire difference between "I tried the greedy reset and it happened to work" and demonstrating real understanding. Stating it as "if starting from X fails at Y, nothing between X and Y can succeed either, because continuing from X only ever has *more* banked surplus at every intermediate point than starting fresh there would" is the exact sentence a tier-1 interviewer is listening for — notice this is a **different shape** of exchange argument than yesterday's Jump Game domination argument (which eliminated *closer* points in favor of a single *farthest* one); here, an entire contiguous *range* of candidates gets eliminated at once, in favor of skipping past all of them together.

🔗 **Backward reference:** the running-total-with-reset shape here echoes the running-prefix-sum accumulation from Days 21–22, though the goal (find a safe starting point) and the reset trigger (going negative) are new — worth noticing the shared "accumulate as you scan" mechanic even though the actual technique (greedy elimination vs. HashMap lookup) is entirely different.

---

# Part 2 — Intervals: Sort, Then Sweep Once

**The mechanism, stated once, reused for the rest of the week:** an "interval" problem gives you a collection of `[start, end]` ranges and asks something about how they relate to each other — do they overlap, should they be combined, how many can coexist. Almost every one of these problems starts with the same move: **sort by start time** (occasionally by end time instead — tomorrow's problems do exactly that, and the difference matters), then **sweep through once**, comparing each interval only to whatever you're currently building, never re-scanning from the beginning.

**Why sorting first is what makes a single sweep sufficient:** once intervals are ordered by start time, the *only* interval that could possibly overlap the one you're currently building is the *next* one in sorted order — anything further ahead starts even later, and anything behind has already been fully accounted for. Without sorting, an overlap could hide anywhere in the array, forcing an O(n²) all-pairs check. Sorting collapses the search space to "just look at your immediate neighbor," which is exactly what turns this into an O(n log n) algorithm (dominated by the sort itself) instead of O(n²).

**The overlap condition, stated precisely:** two intervals `[a, b]` and `[c, d]`, sorted so `a <= c`, overlap (or touch) if and only if `c <= b` — the second interval's start is not strictly past the first interval's end. This is a **closed, inclusive** comparison; whether touching endpoints count as "overlapping" is a detail every interval problem states explicitly, and getting the inequality direction (`<=` vs `<`) backwards is one of the most common small bugs in this entire pattern family.

---

## Problem: Merge Intervals (LeetCode 56, Medium) — Pattern: Intervals

**Statement:** given an array of intervals, merge all overlapping intervals and return the resulting non-overlapping set, covering the same total range as the input.

### Approach — sort by start, sweep and merge

```java
public static int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));   // sort by start time

    List<int[]> merged = new ArrayList<>();
    for (int[] current : intervals) {
        if (merged.isEmpty() || merged.get(merged.size() - 1)[1] < current[0]) {
            merged.add(current);   // no overlap with the last merged interval — start a new one
        } else {
            merged.get(merged.size() - 1)[1] =
                Math.max(merged.get(merged.size() - 1)[1], current[1]);   // extend the last one
        }
    }
    return merged.toArray(new int[merged.size()][]);
}
```

**Why `Math.max`, not a direct overwrite, when extending:** the current interval's end isn't necessarily *later* than the interval it's merging into — a short interval fully contained inside a longer already-merged one (e.g., merging `[1,10]` then `[2,3]`) must leave the merged end at `10`, not shrink it to `3`. Taking the max is what makes this safe regardless of the relative sizes of the two intervals being merged.

**Worked trace:** `intervals = [[1,3],[2,6],[8,10],[15,18]]`. Already sorted by start.

| current | merged so far | overlap check | action |
|---|---|---|---|
| [1,3] | [] | — | add: [[1,3]] |
| [2,6] | [[1,3]] | last.end(3) < current.start(2)? No → overlap | extend: [[1,6]] |
| [8,10] | [[1,6]] | last.end(6) < current.start(8)? Yes → no overlap | add: [[1,6],[8,10]] |
| [15,18] | [[1,6],[8,10]] | last.end(10) < current.start(15)? Yes | add: [[1,6],[8,10],[15,18]] |

Final: `[[1,6],[8,10],[15,18]]` — matches the known expected output for this exact input.

**Complexity:** Time O(n log n) — dominated by the sort; the sweep itself is O(n). Space O(n) for the output (O(log n) to O(n) additional for the sort's own overhead, depending on the sort algorithm used internally).

**Edge cases:** a single interval (returned unchanged, no merging possible); completely non-overlapping intervals (every one becomes its own entry, output size equals input size); one interval fully swallowing another (handled correctly by the `Math.max`, as traced above); touching-but-not-overlapping intervals like `[1,4]` and `[4,5]` — since the condition uses `<` for "no overlap" (i.e., `last.end < current.start`), a touch (`last.end == current.start`) does **not** satisfy "no overlap" and so **does** get merged — matching this problem's convention that touching endpoints count as overlapping.

⚠️ **Common Mistake:** getting the overlap inequality backwards (`<=` where `<` belongs, or vice versa) without checking it against the problem's own stated convention for touching intervals — this is exactly the kind of off-by-one-in-spirit bug the "Insert Interval" problem below will test again, deliberately.

---

## Problem: Insert Interval (LeetCode 57, Medium) — Pattern: Intervals

**Statement:** given a list of non-overlapping intervals **already sorted by start time**, and a new interval, insert the new interval into the list, merging any overlaps as needed, and return the resulting sorted, non-overlapping list.

### Approach 1 — Fall back to Merge Intervals

Append `newInterval` to the given list, then run Day 24's `merge` function above unchanged. Correct — but it re-sorts an array that's already sorted, paying O(n log n) for work the problem statement already handed you for free.

### Approach 2 — Optimized: three phases, exploiting the given sort, O(n)

```java
public static int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0, n = intervals.length;

    // Phase 1: every interval ending strictly before newInterval starts
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i]);
        i++;
    }

    // Phase 2: every interval overlapping newInterval — merge them all into it
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.add(newInterval);

    // Phase 3: everything left over, unaffected
    while (i < n) {
        result.add(intervals[i]);
        i++;
    }
    return result.toArray(new int[result.size()][]);
}
```

**Why three explicit phases, rather than one general merge loop:** the input's guaranteed pre-sorted, non-overlapping structure means there's a clean, provable three-way split — intervals entirely before the new one, intervals that overlap it, and intervals entirely after — and each phase can be handled with a simple, single-purpose `while` loop rather than the more general "compare to whatever's currently accumulating" logic Merge Intervals needs when it can't assume anything about the input beyond "unsorted."

**Why this is O(n), strictly better than falling back to the general merge's O(n log n):** no sort is performed at all — the single pass through `intervals` (split across the three phases, but each index visited exactly once, total) is the only work done, and the "total pointer movement across all three while-loops combined is bounded by n" argument (the same amortized shape used throughout this series since Week 1's `ArrayList` analysis) applies directly.

**Worked trace:** `intervals = [[1,3],[6,9]]`, `newInterval = [2,5]`.

- Phase 1: `intervals[0] = [1,3]`, is `3 < 2`? No — phase 1 adds nothing, `i` stays `0`.
- Phase 2: `intervals[0][0]=1 <= newInterval[1]=5`? Yes → merge: `newInterval = [min(2,1), max(5,3)] = [1,5]`, `i=1`. Next: `intervals[1][0]=6 <= newInterval[1]=5`? No — phase 2 stops. Add `[1,5]` to result.
- Phase 3: `intervals[1] = [6,9]` added unchanged.

Final: `[[1,5],[6,9]]` — matches the known expected output for this exact input.

**Complexity:** Time O(n), Space O(n) for the output.

**Edge cases:** `newInterval` overlapping nothing (phase 2's loop body never executes even once, `newInterval` is added to the result completely unchanged); `newInterval` overlapping *every* existing interval (phase 1 and phase 3 both contribute nothing, phase 2 consumes the entire input); `newInterval` inserted into an empty list (phases 1 and 3 trivially do nothing, phase 2's loop condition `i < n` is immediately false since `n=0`, `newInterval` is added as-is).

💡 **Interview Insight:** naming the complexity contrast unprompted — "I could fall back to Merge Intervals' general algorithm at O(n log n), but since this input is already sorted, I can do this in O(n) instead by exploiting that structure directly" — is exactly the kind of "don't reach for the general tool when the specific structure of this input lets you do better" observation that separates a strong answer from a merely correct one.

---

## Extra Practice: Queue Reconstruction by Height (LeetCode 406, Medium)

**Why this is worth the extra rep:** every greedy problem so far this week has sorted by a *single* key. This one needs **two** — and the reasoning for *why* the second key matters, and in which direction, is exactly the kind of subtlety worth deliberately practicing before it costs you in an interview.

**Statement:** given `people`, an array where `people[i] = [h, k]` means a person of height `h` has exactly `k` people of height `>= h` standing in front of them in the desired final queue, reconstruct and return the queue.

### Approach — sort by height descending, k ascending; insert at index k

```java
public static int[][] reconstructQueue(int[][] people) {
    Arrays.sort(people, (a, b) -> {
        if (a[0] != b[0]) return b[0] - a[0];   // height DESCENDING
        return a[1] - b[1];                      // k ASCENDING, for ties in height
    });

    List<int[]> result = new ArrayList<>();
    for (int[] person : people) {
        result.add(person[1], person);   // insert at index k
    }
    return result.toArray(new int[result.size()][]);
}
```

**Why sort by height *descending* first:** placing the tallest people first means that every person placed afterward is guaranteed to be shorter-or-equal to everyone already in the queue. A shorter person's placement can **never** invalidate an already-placed taller person's `k` count, because `k` only counts people who are taller-or-equal — a shorter person inserted anywhere, before or after, simply doesn't contribute to that count at all. This is what makes each person's own `k` value directly usable as an insertion index the moment they're placed: among people at least as tall as them (which, by the descending order, is exactly "everyone already placed"), `k` is precisely how many should be in front — so inserting at index `k` satisfies their constraint immediately and permanently, with no future insertion able to disturb it.

**Why the *second* key (`k`, ascending) is needed for ties in height:** among people of the *same* height, each one's `k` counts people taller **or equal** — meaning people of the same height do count toward each other's `k`. Processing same-height people in ascending-`k` order and inserting each at their own `k` index correctly builds up their relative order too: the person who should have zero same-height people in front of them (`k` smallest) gets placed first among their height group, and each subsequent same-height insertion naturally lands after the ones that should precede it.

**Worked trace:** `people = [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]`. Sort descending height, ascending k for ties: `[[7,0],[7,1],[6,1],[5,0],[5,2],[4,4]]`.

| insert | at index | result after |
|---|---|---|
| [7,0] | 0 | [[7,0]] |
| [7,1] | 1 | [[7,0],[7,1]] |
| [6,1] | 1 | [[7,0],[6,1],[7,1]] |
| [5,0] | 0 | [[5,0],[7,0],[6,1],[7,1]] |
| [5,2] | 2 | [[5,0],[7,0],[5,2],[6,1],[7,1]] |
| [4,4] | 4 | [[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]] |

Final: `[[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]` — matches the known expected output for this exact input.

**Complexity:** Time O(n²) — sorting is O(n log n), but each `ArrayList.insert(index, ...)` shifts every element after it, and up to `n` insertions each costing up to O(n) gives O(n²) overall. Space O(n). *(Extension, beyond what's needed to solve this at interview pace: the insertion cost can be reduced to O(n log n) total using a structure with efficient rank-based insertion, such as a Binary Indexed Tree over available slots — worth mentioning as a follow-up if asked "can you do better," but the ArrayList version is the expected first-pass answer.)*

**Edge cases:** all people the same height (the ascending-`k` tiebreak alone determines full ordering, exactly as if height weren't a factor); a single person (`k` must be `0`, trivially satisfied, inserted at index `0`); `k` values that would be inconsistent with any valid queue — not possible here, since the problem guarantees valid input.

⚠️ **Common Mistake:** sorting by height *ascending* instead of descending, then trying to make the insertion logic work anyway — it doesn't, because a taller person inserted *after* shorter people would need their `k` count adjusted retroactively for however many shorter people ended up in front of them by coincidence, which defeats the entire point of choosing an insertion order where nothing needs retroactive adjustment.

---

## Day 24 — Interview Questions

**Q1. State Gas Station's prefix-elimination argument precisely.** If starting from `start` first fails at station `i`, then for any station `j` with `start <= j <= i`, the accumulated surplus from `start` to `j` is non-negative — so continuing from `start` has already banked at least as much by the time it reaches `j` as starting fresh at `j` would have. If continuing from `start` still fails by `i`, starting fresh at `j` fails at least as early. Every station in `[start, i]` is eliminated in one shot.

---

**Q2. Why does Gas Station only need to check `totalTank >= 0` once, rather than per-candidate-start?** If total gas is less than total cost, no starting point can possibly complete the circuit, regardless of where you start — it's a global, arithmetic impossibility, not something that varies by starting station.

---

**Q3. What's the core mechanism every Intervals problem shares?** Sort by start time (sometimes end time instead), then sweep once, comparing each interval only to whatever's currently being built — sorting collapses the search for a possible overlap down to "check the immediate neighbor," which is what allows a single O(n) sweep after the O(n log n) sort.

---

**Q4. In Merge Intervals, why does extending the last merged interval's end use `Math.max` instead of a direct overwrite?** A later interval in sorted-by-start order isn't guaranteed to have a later end — one interval can be fully contained inside another already-merged one. Taking the max preserves the correct (larger) end regardless of which interval happens to be "current."

---

**Q5. Why is Insert Interval solvable in O(n), while Merge Intervals needs O(n log n)?** Insert Interval's input is already sorted and non-overlapping by the problem's own guarantee, so no sort is needed at all — a single three-phase pass (before / overlapping / after) suffices. Merge Intervals can't assume any pre-existing order, so it must sort first.

---

**Q6. In Queue Reconstruction by Height, why does sorting tallest-first make each person's own `k` directly usable as an insertion index?** Processing tallest-first guarantees that everyone already placed when a given person is inserted is at least as tall as them — and `k` only counts people taller-or-equal. So inserting at index `k` places exactly the right number of taller-or-equal people in front, and no future (necessarily shorter-or-equal) insertion can disturb that count, since shorter people never count toward anyone's `k`.

---

**Q7. Why does Queue Reconstruction by Height need a second sort key for same-height ties?** People of equal height count toward each other's `k` (it counts "taller or equal"), so processing same-height people in ascending-`k` order and inserting each at their own index correctly builds their relative order too — the person needing zero same-height people in front gets placed first among that height group.

---

## Daily Deliverable Check

- [ ] Gas Station, Merge Intervals, and Insert Interval solved, pushed to `dsa-java/greedy-intervals/`.
- [ ] Queue Reconstruction by Height (extra practice) solved, same folder.
- [ ] Can state Gas Station's prefix-elimination argument from memory, and explain why it's a different proof shape than yesterday's frontier-domination argument.
- [ ] Can state the Intervals sort-and-sweep mechanism and its overlap condition from memory.

---

## What Tomorrow Assumes You Already Know Cold

Day 25 assumes today's Intervals mechanism — sort, then sweep once, comparing only to what's currently being built — is fully reflexive, because tomorrow immediately varies it: sorting by **end** time instead of start time, for a **different** kind of question (how many intervals must be discarded, rather than how to combine them). If the overlap condition (`current.start <= last.end`, closed) still needs to be looked up rather than recalled, that's worth resolving before tomorrow, since tomorrow's proof leans on being able to state the condition instantly and adapt it to a new sort key without re-deriving it from scratch.

**Next:** [Day 25 Resource Book](Day25_Resource_Book.md) — Intervals: Non-Overlapping and Balloon-Popping.
