# Day 25 — Intervals, Sorted by End Time: Non-Overlapping and Balloon-Popping

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 24 Resource Book](Day24_Resource_Book.md)
**Next ▶:** [Day 26 Resource Book](Day26_Resource_Book.md)
**Companion to:** Day 25 of `Week_04_Revised.md`

---

## Recap

Yesterday established the Intervals mechanism — sort, then sweep once — using **start time** as the sort key, for two "combine overlapping ranges" questions (Merge Intervals, Insert Interval). Today asks a genuinely different question — **how many intervals have to go, or how few "hits" can cover everything** — and that different question calls for a different sort key: **end time**. The mechanism (sort, then sweep once, comparing only to what's currently tracked) is unchanged; what sorts by and what gets tracked both shift.

Today has no Theory, Project, or Career Block in the plan — DSA only, tapering to 3.5–4 hours as the leave week's intensity begins to wind down.

---

## Learning Objectives

By the end of today, without notes:

1. Explain, via an exchange argument, why Non-overlapping Intervals sorts by **end** time rather than start time, and why "keep the interval that ends earliest" is provably the right greedy choice.
2. Recognize Minimum Number of Arrows to Burst Balloons as the *identical* greedy-by-end-time shape as Non-overlapping Intervals, reframed from "discard the minimum" to "cover with the minimum."
3. Solve Interval List Intersections (extra) with two pointers advancing across **two separate** interval lists, and distinguish "intersect" from yesterday's "merge."

---

## Concept Dependency Map

```
Yesterday (Day 24): Intervals sorted by START time — combine overlapping ranges
   (Merge Intervals, Insert Interval)
        │
        ▼
TODAY — Intervals sorted by END time — a DIFFERENT greedy key, for a DIFFERENT question
   ("how many must be removed / how few arrows needed to cover everything",
    not "how do overlapping ranges combine")
├─ LC 435 Non-overlapping Intervals — greedy: keep the earliest-ending interval, discard conflicts
├─ LC 452 Min Arrows to Burst Balloons — IDENTICAL shape, reframed as "cover" not "discard"
└─ Extra: LC 986 Interval List Intersections — TWO separate lists, two pointers
   (needs: opposite-ends / same-direction Two Pointers discipline — Week 1, Day 6-7)
        │
        ▼
Tomorrow: Intervals + a min-heap (PriorityQueue, already known — Week 3, Day 17)
```

---

# Part 1 — Why End Time, Not Start Time, This Time

Yesterday's question was "how do overlapping intervals *combine*" — and sorting by start time made the immediate next interval in sorted order the only one that could possibly extend what's currently being built. Today's question is different: **"what's the largest subset of these intervals that can coexist without any overlap"** (equivalently, the *fewest* that must be removed to achieve that). For this question, the interval that **ends earliest** is the one that leaves the most room for everything that comes after it — which makes end time, not start time, the sort key that matters.

**The exchange argument, stated in general form before either problem below:** among any group of mutually overlapping intervals, keeping the one that ends earliest and discarding the rest of that group can never be worse than any other choice. Suppose an optimal solution kept some *other* interval from that overlapping group instead of the earliest-ending one. Since the earliest-ending interval ends no later than the one that was kept, swapping it in — replacing the kept interval with the earliest-ending one — can only leave **more** room (an earlier or equal end time) for whatever intervals come next, never less. So the swap never makes the solution worse, and can only help. This is the general template both of today's required problems apply directly.

---

## Problem: Non-overlapping Intervals (LeetCode 435, Medium) — Pattern: Intervals, Greedy by End Time

**Statement:** given an array of intervals, return the **minimum number that must be removed** so that the rest are pairwise non-overlapping.

### Approach 1 — Brute force (conceptual)

Trying every possible subset of intervals to find the largest non-overlapping one is exponential (2ⁿ subsets) — worth naming as the infeasible baseline, since it clarifies exactly what the greedy approach is shortcutting.

### Approach 2 — Optimized: sort by end time, greedily keep the earliest-ending survivor

```java
public static int eraseOverlapIntervals(int[][] intervals) {
    if (intervals.length == 0) return 0;
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));   // sort by END time

    int removals = 0;
    int lastEnd = intervals[0][1];

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < lastEnd) {
            removals++;              // overlaps the last KEPT interval — discard this one
        } else {
            lastEnd = intervals[i][1];   // no overlap — keep it, update the boundary
        }
    }
    return removals;
}
```

**Why this correctly maximizes the surviving non-overlapping count (equivalently minimizes removals):** this is Part 1's exchange argument applied directly, interval by interval. `lastEnd` always reflects the end time of the most recently *kept* interval — which, because of the sort, is always the earliest-ending interval among everything considered so far that survived. When the next interval's start is before `lastEnd`, it necessarily overlaps that kept interval; discarding the new one (rather than swapping it in for the kept one) is correct because the kept one, having sorted-earlier end time, can only leave *at least as much* room going forward — there's never a reason to swap in an interval that ends later.

**Worked trace:** `intervals = [[1,2],[2,3],[3,4],[1,3]]`. Sort by end time: `[[1,2],[2,3],[1,3],[3,4]]`.

| current | current.start < lastEnd? | action | lastEnd (after) | removals |
|---|---|---|---|---|
| [1,2] (init) | — | keep | 2 | 0 |
| [2,3] | 2<2? no | keep | 3 | 0 |
| [1,3] | 1<3? **yes** | remove | 3 (unchanged) | 1 |
| [3,4] | 3<3? no | keep | 4 | 1 |

Final: `removals = 1` — matches the known expected output for this exact input (removing `[1,3]` leaves `[1,2],[2,3],[3,4]`, pairwise non-overlapping).

**Complexity:** Time O(n log n) — dominated by the sort. Space O(1) extra (excluding sort overhead).

**Edge cases:** empty input (`0` removals, handled by the early return); all intervals identical (all but one get removed, correctly — each subsequent identical interval has `start == lastEnd`... wait, more precisely `start < lastEnd` is false when `start == lastEnd` for genuinely identical *end* times but overlapping ranges — worth tracing carefully: two intervals `[1,3]` and `[1,3]`, sorted, `lastEnd` starts at `3`; second one's start `1 < 3` is true, so it's correctly removed); already pairwise non-overlapping input (`removals` stays `0` throughout, correctly, since no `start < lastEnd` condition ever fires).

⚠️ **Common Mistake:** sorting by **start** time instead of end time, out of habit from yesterday's Merge Intervals — this produces a genuinely wrong answer here, not just a slower one, because the exchange argument specifically depends on "earliest end time leaves the most room," which sorting by start time doesn't guarantee (an interval that starts earliest can still end very late, blocking far more of what follows than a later-starting, earlier-ending alternative would have).

💡 **Interview Insight:** stating "I need to greedily keep the interval that ends earliest, so I'm sorting by end time, not start time" *before* writing code, along with a one-sentence version of the exchange argument, is the single strongest signal on this problem — precisely because sorting by the wrong key is such a common, easy-to-make mistake here.

---

## Problem: Minimum Number of Arrows to Burst Balloons (LeetCode 452, Medium) — Pattern: Intervals, Identical Shape, Reframed

**Statement:** balloons are represented as `[x_start, x_end]` ranges along the x-axis. An arrow shot straight up at position `x` bursts every balloon whose range includes `x`. Return the minimum number of arrows needed to burst every balloon.

**🔑 Key Takeaway, stated before any code:** this is *exactly* Non-overlapping Intervals' mechanism, with the framing flipped. There, overlapping intervals got reduced down to one *kept* survivor per overlapping cluster (counting *removals*). Here, overlapping balloons get reduced down to one shared *arrow* per overlapping cluster (counting *arrows*). Same sort key (end time), same greedy rule (commit to the earliest end within a cluster), different narrative label attached to the same count.

### Approach — sort by end position, greedily place arrows at the earliest end

```java
public static int findMinArrowShots(int[][] points) {
    if (points.length == 0) return 0;
    Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1]));   // sort by END position

    int arrows = 1;
    int lastArrowPos = points[0][1];

    for (int i = 1; i < points.length; i++) {
        if (points[i][0] > lastArrowPos) {
            arrows++;                        // this balloon isn't covered — need a new arrow
            lastArrowPos = points[i][1];
        }
        // else: this balloon's range already includes lastArrowPos — already covered, do nothing
    }
    return arrows;
}
```

**Mapping the reframe precisely, line by line:** Non-overlapping Intervals' `removals++` (discard something that overlaps) becomes here `arrows++` (commit a *new* shared resource, because nothing existing covers this balloon) — the trigger condition even flips sign accordingly: `435`'s `intervals[i][0] < lastEnd` (overlap → discard) becomes `452`'s `points[i][0] > lastArrowPos` (**no** overlap with the last arrow → need a new one). Both conditions are checking the exact same geometric relationship — whether the current range's start falls before or after the tracked boundary — just answering the opposite question about what that relationship implies you should do.

**Why placing the arrow at the earliest end (not anywhere else in the overlap) is optimal:** an arrow placed at the *earliest* end position among a cluster of overlapping balloons is guaranteed to hit every balloon in that cluster (since, by definition of the cluster, every one of them spans at least that far), while leaving the maximum possible room for balloons outside the cluster to require their own, separate arrow only when genuinely necessary — the same exchange argument as Non-overlapping Intervals, restated for "cover" instead of "keep."

**Worked trace:** `points = [[10,16],[2,8],[1,6],[7,12]]`. Sort by end: `[[1,6],[2,8],[7,12],[10,16]]`.

| current | current.start > lastArrowPos? | action | lastArrowPos (after) | arrows |
|---|---|---|---|---|
| [1,6] (init) | — | first arrow | 6 | 1 |
| [2,8] | 2>6? no | already covered | 6 | 1 |
| [7,12] | 7>6? **yes** | new arrow | 12 | 2 |
| [10,16] | 10>12? no | already covered | 12 | 2 |

Final: `arrows = 2` — matches the known expected output for this exact input (one arrow at `x=6` bursts `[1,6]` and `[2,8]`; one arrow at `x=12` bursts `[7,12]` and `[10,16]`).

**Complexity:** Time O(n log n), Space O(1) extra — identical to Non-overlapping Intervals, for the identical underlying reason.

**Edge cases:** empty input (`0`, via the early return); every balloon overlapping at one common point (all covered by a single arrow, correctly — `lastArrowPos` never gets exceeded after the first assignment); no balloons overlapping at all (`arrows` equals the total balloon count, one each); touching balloons like `[1,2]` and `[2,3]` — since the "already covered" condition is `points[i][0] > lastArrowPos` (**strict**), a touch (`start == lastArrowPos`) does **not** trigger a new arrow, meaning touching balloons **share** an arrow — the opposite convention from yesterday's Merge Intervals, where touching intervals were merged, worth noticing precisely because both are "closed-boundary" conventions that could easily be mixed up under pressure.

---

## Extra Practice: Interval List Intersections (LeetCode 986, Medium)

**Why this is worth the extra rep:** every Intervals problem so far this week has operated on **one** list. This one hands you **two separate, independently sorted lists** and asks for their pairwise overlaps — a genuinely different shape that reuses Two Pointers (Week 1, Days 6–7) rather than a single-list sweep, and is common enough in practice (calendar/availability overlap questions) to be worth deliberate practice.

**Statement:** given two lists of closed intervals, `firstList` and `secondList`, each already sorted and internally non-overlapping, return their intersection — every range that is covered by *both* an interval from `firstList` and an interval from `secondList`.

### Approach — two pointers, one per list

```java
public static int[][] intervalIntersection(int[][] firstList, int[][] secondList) {
    List<int[]> result = new ArrayList<>();
    int i = 0, j = 0;

    while (i < firstList.length && j < secondList.length) {
        int start = Math.max(firstList[i][0], secondList[j][0]);
        int end = Math.min(firstList[i][1], secondList[j][1]);

        if (start <= end) {
            result.add(new int[]{start, end});   // valid overlap found
        }

        // advance whichever interval ends first — it can't possibly overlap anything further
        if (firstList[i][1] < secondList[j][1]) {
            i++;
        } else {
            j++;
        }
    }
    return result.toArray(new int[result.size()][]);
}
```

**Why the intersection of two intervals is `[max(starts), min(ends)]`:** a point is covered by *both* intervals exactly when it's at or after both starts (so at or after the *later* start) and at or before both ends (so at or before the *earlier* end). If that computed range is empty (`start > end`), the two current intervals simply don't overlap, and nothing is added.

**Why advancing the interval that ends first is always safe — the key correctness argument:** once `firstList[i]` and `secondList[j]` have been compared, whichever of the two has the **earlier end** cannot possibly overlap any *later* interval in the *other* list — every later interval in the other list starts even later (both lists are individually sorted), so an interval that already ends before the current position in the other list is entirely "used up." Advancing past it loses no future overlap, exactly the same "safe to discard, provably" argument used throughout Two Pointers since Week 1.

**Distinguishing this from yesterday's Merge, precisely:** Merge Intervals **combines** overlapping ranges from a single list into fewer, larger ranges. This problem **extracts** just the overlapping portion between ranges from two separate lists, discarding the non-overlapping remainder of each — a fundamentally different output shape (the intersection can never be larger than either input range, whereas a merge is always at least as large as its largest input).

**Worked trace:** `firstList = [[0,2],[5,10],[13,23],[24,25]]`, `secondList = [[1,5],[8,12],[15,24],[25,26]]`.

| i | j | first[i] | second[j] | start=max | end=min | valid? | advance |
|---|---|---|---|---|---|---|---|
| 0 | 0 | [0,2] | [1,5] | 1 | 2 | yes → [1,2] | first ends 2<5 → i++ |
| 1 | 0 | [5,10] | [1,5] | 5 | 5 | yes → [5,5] | second ends 5<10 → j++ |
| 1 | 1 | [5,10] | [8,12] | 8 | 10 | yes → [8,10] | first ends 10<12 → i++ |
| 2 | 1 | [13,23] | [8,12] | 13 | 12 | no (13>12) | second ends 12<23 → j++ |
| 2 | 2 | [13,23] | [15,24] | 15 | 23 | yes → [15,23] | first ends 23<24 → i++ |
| 3 | 2 | [24,25] | [15,24] | 24 | 24 | yes → [24,24] | second ends 24<25 → j++ |
| 3 | 3 | [24,25] | [25,26] | 25 | 25 | yes → [25,25] | first ends 25<26 → i++ |
| loop ends (i=4=length) | | | | | | | |

Final: `[[1,2],[5,5],[8,10],[15,23],[24,24],[25,25]]` — matches the known expected output for this exact input.

**Complexity:** Time O(m + n) where m, n are the two list lengths — each pointer advances at most its own list's length, total. Space O(m + n) worst case for the output.

**Edge cases:** one list empty (loop condition fails immediately, empty result — correctly, no intersections possible); no overlaps anywhere between the two lists (every computed `start > end`, result stays empty, but the pointer-advancement logic still correctly walks through both lists to completion); one interval fully containing another across lists (correctly produces the fully-contained one as the intersection, since `max(starts)`/`min(ends)` naturally resolves to the smaller range).

💡 **Interview Insight:** if asked why this can't just reuse yesterday's single-list Merge Intervals logic, the answer is structural: merge operates on ranges that need to be *combined* within one collection, while this operates on two *already-clean* collections whose cross-relationships are what's being queried — reaching for Two Pointers instead of a single-list sweep is the correct recognition that this is a different shape of problem entirely, not a smaller version of Merge Intervals.

---

## Day 25 — Interview Questions

**Q1. Why does Non-overlapping Intervals sort by end time instead of start time?** The greedy goal is to keep whichever interval, among any overlapping cluster, leaves the most room for everything that follows — that's the interval with the earliest end time, not necessarily the earliest start time. Sorting by end time is what makes "the next kept interval" always the correct greedy choice.

---

**Q2. State the exchange argument for keeping the earliest-ending interval in an overlapping cluster.** If an optimal solution kept some other interval from the cluster instead, swapping in the earliest-ending one instead can only leave equal or more room for what comes next (since it ends no later), so the swap never makes the solution worse — meaning some optimal solution always agrees with the greedy choice.

---

**Q3. How does Minimum Arrows to Burst Balloons relate to Non-overlapping Intervals?** It's the identical greedy-by-end-time mechanism, reframed — "discard the interval that doesn't survive" becomes "this balloon needs its own new arrow," and the trigger condition flips sign accordingly (overlap-so-discard becomes no-overlap-so-need-new-arrow), but both count the same underlying quantity: the number of independent overlapping clusters.

---

**Q4. In Minimum Arrows, why does a touching pair like `[1,2]` and `[2,3]` share one arrow, while in yesterday's Merge Intervals a touching pair gets merged?** Both are closed-boundary conventions, but they're independently defined by each problem: Minimum Arrows uses a *strict* `>` for "not yet covered" (so a touch, `==`, still counts as covered, sharing the arrow); Merge Intervals uses a *strict* `<` for "no overlap" (so a touch, `==`, does **not** count as "no overlap," triggering a merge). They happen to produce the same practical outcome (touching intervals get combined/shared) via differently-signed conditions — worth being precise about which direction each problem's own inequality goes, rather than assuming they're identical.

---

**Q5. Why does Interval List Intersections use two pointers instead of a single-list sweep?** The input is two *separate*, independently sorted lists whose cross-relationships are being queried, not one list whose internal overlaps need combining — a structurally different question from Merge Intervals, requiring one pointer per list rather than one running "currently building" interval.

---

**Q6. In Interval List Intersections, why is it always safe to advance the pointer belonging to whichever interval ends first?** Both lists are independently sorted, so every remaining interval in the other list starts even later than the current one being compared. An interval that already ends before the current position in the other list cannot overlap anything further in that other list — advancing past it loses no possible future intersection.

---

## Daily Deliverable Check

- [ ] Non-overlapping Intervals and Minimum Number of Arrows to Burst Balloons solved, pushed to `dsa-java/greedy-intervals/`.
- [ ] Interval List Intersections (extra practice) solved, same folder.
- [ ] Can state, from memory, why today's problems sort by end time while yesterday's sorted by start time.
- [ ] Can explain the reframe from Non-overlapping Intervals to Minimum Arrows without looking either solution up.

---

## What Tomorrow Assumes You Already Know Cold

Day 26 assumes both of today's greedy-by-end-time problems are fully reflexive, because Meeting Rooms (tomorrow's first problem) is a direct simplification of the exact same overlap-checking logic. Tomorrow also reaches back further than today — to Week 3, Day 17's `PriorityQueue` (min-heap, O(log n) insert/poll, O(1) peek) — and applies it to interval scheduling for the first time; that mechanism itself won't be re-taught, only its new application. If the sort-by-end-time exchange argument still needs to be looked up rather than stated on demand, that's worth resolving before tomorrow, since Meeting Rooms II leans on a close variant of it.

**Next:** [Day 26 Resource Book](Day26_Resource_Book.md) — Meeting Rooms.
