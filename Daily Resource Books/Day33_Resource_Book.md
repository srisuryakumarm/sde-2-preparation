# Day 33 — Binary Search Capstone: On-the-Answer, Completed and Reviewed

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 32 Resource Book](Day32_Resource_Book.md)
**Next ▶:** [Day 34 Resource Book](Day34_Resource_Book.md)
**Companion to:** Day 33 of `Week_05_Revised.md`

---

## Recap

Yesterday made "binary search on the answer" explicit, tracing it back to Day 28's First Bad Version. Today closes Binary Search at 11 required problems with one more on-the-answer rep, adds two extra reps to make the pattern fully reflexive (the plan itself calls this "the one people miss cold in interviews," which is exactly the kind of signal that justifies going beyond the plan's 1–2 listed reps), and then steps back for a full review of everything the unit covered — five straight days of variations on one template family.

Today's DSA block is only 2 hours against a single required problem — real slack, unlike every other day this week. That slack is spent deliberately on the two extra reps below, both clearly marked as going beyond the strict daily requirement.

---

## Learning Objectives

By the end of today, without notes:

1. Apply binary search on the answer to a feasibility check built from a greedy day-by-day simulation, and explain why it's the same shell as Koko's, with a different check inside.
2. Recognize a feasibility check that needs to track adjacency (Bouquets), not just a running sum, and explain why naive total-count would overcount.
3. Recognize the *maximize*-the-answer flavor of this pattern (Magnetic Force) from its wording alone, and explain exactly how the binary search's shrink direction mirrors, rather than repeats, the minimize-the-answer version.
4. Classify all 11 required (+4 extra) Binary Search problems from this week and last as "on the input" or "on the answer," from memory.

---

## Concept Dependency Map

```
Day 28: LC 278, first "on the answer" appearance
Day 32: LC 875 (Koko) — the framing made explicit, monotonic feasibility over a range
        │
        ▼
Today, Problem 11: Capacity To Ship Packages Within D Days (LC 1011)
   identical skeleton to Koko — binary search over capacity, greedy day-counting
   simulation as the feasibility check
        │
        ├──▶ Extra Practice 1: LC 1482 — feasibility check needs adjacency tracking,
        │        not just a sum (a genuinely different feasibility-check INTERNAL,
        │        same outer shell)
        │
        └──▶ Extra Practice 2: LC 1552 — flips to MAXIMIZE the answer; the binary
                 search's shrink direction mirrors every problem above it
        │
        ▼
Theory: Binary Search, Reviewed — On the Input vs. On the Answer
   (formalizes Day 28's distinction across all 15 problems from both weeks)
```

---

# Part 1 — Binary Search Capstone

### Prerequisites (confirmed)

- Binary search on the answer, and the monotonic-feasibility framing — Day 28, Day 32.
- `long` for overflow-risk accumulators — Day 10, Day 11, Day 32.

## Problem 11: Capacity To Ship Packages Within D Days (LeetCode 1011, Medium) — Pattern: Binary Search on the Answer

**Statement:** `weights[i]` is the weight of the `i`-th package; packages must ship **in order** (no reordering), split across `days` days, on a ship with some fixed weight capacity — each day, load packages in order until the next one would exceed capacity, then start a new day. Return the **minimum** capacity that ships everything within `days` days.

### Approach — binary search on capacity, greedy simulation as the feasibility check

```java
public static int shipWithinDays(int[] weights, int days) {
    int left = 0, right = 0;
    for (int weight : weights) {
        left = Math.max(left, weight);   // capacity must fit the single heaviest package
        right += weight;                  // loosest upper bound: everything in one day
    }
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canShip(weights, mid, days)) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}

private static boolean canShip(int[] weights, int capacity, int days) {
    int daysNeeded = 1;
    long currentLoad = 0;
    for (int weight : weights) {
        if (currentLoad + weight > capacity) {
            daysNeeded++;
            currentLoad = 0;
        }
        currentLoad += weight;
    }
    return daysNeeded <= days;
}
```

**🔗 This is Koko's exact skeleton — say so unprompted:** binary search over a range of candidate answers (capacity, not speed), with a monotonic feasibility check (more capacity never hurts — "can ship in time" only gets easier as capacity grows). The only genuinely new piece is what's *inside* the feasibility check: a greedy simulation that walks the packages in order, accumulating a running load and starting a new day exactly when the next package would overflow the current one.

**Why `left` starts at `max(weights)`, not `0`:** capacity below the single heaviest package is infeasible by definition — no package can be split across days. This is the same "tighten the bound using a fact you already know, don't just search from zero" move as yesterday's `right = max(piles)` for Koko, mirrored here as the *lower* bound instead of the upper one.

**⚠️ Common Mistake:** using `>=` instead of `>` in the overflow check (`currentLoad + weight > capacity`). A package that brings the load to *exactly* capacity still belongs on the current day — only a package that would push the load *past* capacity forces a new day. `>=` would incorrectly split off a package that fits perfectly.

**Worked trace:** `weights = [1,2,3,4,5,6,7,8,9,10]`, `days = 5`. `left = 10` (max), `right = 55` (sum).

| left | right | mid | days needed at `mid` | ≤ 5? | action |
|---|---|---|---|---|---|
| 10 | 55 | 32 | 2 | yes | `right = 32` |
| 10 | 32 | 21 | 3 | yes | `right = 21` |
| 10 | 21 | 15 | 5 | yes | `right = 15` |
| 10 | 15 | 12 | 6 | no | `left = 13` |
| 13 | 15 | 14 | 6 | no | `left = 15` |

`left == right == 15`, loop ends → returns `15`. One full simulation for concreteness, at `capacity = 15`: loads accumulate `1,3,6,10,15` (five packages) — the sixth package (`6`) would push `15 + 6 = 21 > 15`, so day 2 starts at `6`, giving `6,13`; the eighth (`8`) would push `13 + 8 = 21 > 15`, day 3 starts at `8`; the ninth (`9`) would push `8+9=17>15`, day 4 starts at `9`; the tenth (`10`) would push `9+10=19>15`, day 5 starts at `10` alone. Five days used, exactly matching the `days` budget — `15` is feasible, and (per the table) `14` isn't, confirming `15` is the true minimum.

**Complexity:** Time O(n log(sum − max)) — the binary search range has width `sum(weights) − max(weights)`, and each of its O(log(range)) iterations runs an O(n) simulation. Space O(1).

**Edge cases:**
- `days == weights.length` — the tightest case; forces capacity `= max(weights)`, one package per day.
- `days == 1` — forces capacity `= sum(weights)`, everything in one day.
- Single package — trivially resolves to that package's own weight.

**💡 Interview Insight:** the strongest possible opening here is exactly the one-liner from yesterday, reused: "the search space isn't the array — it's the range of possible capacities, and the feasibility check is a greedy simulation." Interviewers who've seen this problem before are listening for whether you reach for that framing on your own, or need to be walked to it.

---

# Part 2 — Extra Practice: Two More Reps

Binary search on the answer is the pattern the plan itself flags as commonly missed cold — it gets two extra reps today, both clearly beyond the day's strict 2-hour DSA budget, meant as extension material rather than core requirements.

---

## Extra Practice 1 (Extension): Minimum Number of Days to Make m Bouquets (LeetCode 1482, Medium) — Pattern: On the Answer, Feasibility Needs Adjacency Tracking

**Statement:** `bloomDay[i]` is the day flower `i` blooms. Making one bouquet needs `k` **adjacent**, already-bloomed flowers. Return the minimum day on which `m` bouquets can all be made, or `-1` if it's impossible regardless of how long you wait.

### Approach

```java
public static int minDays(int[] bloomDay, int m, int k) {
    long flowersNeeded = (long) m * k;
    if (flowersNeeded > bloomDay.length) {
        return -1;   // not enough flowers exist in total — no day ever fixes this
    }
    int left = Integer.MAX_VALUE, right = Integer.MIN_VALUE;
    for (int day : bloomDay) {
        left = Math.min(left, day);
        right = Math.max(right, day);
    }
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canMakeBouquets(bloomDay, mid, m, k)) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}

private static boolean canMakeBouquets(int[] bloomDay, int day, int m, int k) {
    int bouquets = 0, adjacentBloomed = 0;
    for (int bloom : bloomDay) {
        if (bloom <= day) {
            adjacentBloomed++;
            if (adjacentBloomed == k) {
                bouquets++;
                adjacentBloomed = 0;   // this group is used up — start the next run fresh
            }
        } else {
            adjacentBloomed = 0;      // an unbloomed flower breaks the run entirely
        }
    }
    return bouquets >= m;
}
```

**⚠️ Why the `-1` guard has to come *before* any binary search:** every other "on the answer" problem this week has a feasibility check that's guaranteed satisfiable for *some* value in the search range (per each problem's own constraints). This one isn't — if `m × k` exceeds the total flower count, **no day, however large, ever makes enough bouquets possible**, because waiting longer only blooms more flowers, it never creates new ones. Without checking this up front, the binary search would search a range where every candidate is infeasible, with no valid answer to converge on.

**🔑 Key Takeaway — the genuinely new piece is inside the feasibility check, not the outer shell:** Koko and Ship both reduce their feasibility check to a running *sum*. This one needs to track **maximal runs of adjacent bloomed flowers** and take `⌊run_length / k⌋` bouquets from each run, resetting the count whenever an unbloomed flower breaks a run. That's a structurally different feasibility check — proof that "binary search on the answer" is a *family*, unified by the outer monotonic-search shell, not by identical internals.

**Why naive "just count total bloomed flowers, divide by k" would be wrong:** `bloomDay = [1, 5, 1, 5, 1]`, `k = 2`, `day = 1`. Three flowers have bloomed by day 1 (indices 0, 2, 4) — naive counting gives `⌊3/2⌋ = 1` bouquet. But none of those three bloomed flowers are *adjacent* to each other (index 1 and index 3 haven't bloomed, breaking every potential pair) — the true, adjacency-aware answer is **0** bouquets. Tracing the actual algorithm confirms it: `adjacentBloomed` resets to `0` at both unbloomed indices, so it never reaches `k = 2` anywhere, and `bouquets` stays `0`.

**Worked trace (primary, `k = 1` so adjacency is trivially satisfied by every single flower):** `bloomDay = [1,10,3,10,2]`, `m = 3`, `k = 1`. `flowersNeeded = 3 ≤ 5`, proceed. `left = 1, right = 10`.

| left | right | mid | bloomed by `mid` | bouquets | ≥ 3? | action |
|---|---|---|---|---|---|---|
| 1 | 10 | 5 | `[1,_,3,_,2]` | 3 | yes | `right = 5` |
| 1 | 5 | 3 | `[1,_,3,_,2]` | 3 | yes | `right = 3` |
| 1 | 3 | 2 | `[1,_,_,_,2]` | 2 | no | `left = 3` |

`left == right == 3`, loop ends → returns `3` (matches the known answer for this input).

**Complexity:** Time O(n log(max(bloomDay) − min(bloomDay))). Space O(1).

**Edge cases:**
- `m × k == bloomDay.length` exactly — every single flower must eventually be used; the last flower to bloom sets the answer.
- `k == 1` — adjacency becomes irrelevant, degenerating to simple counting, as in the trace above.
- `m × k > bloomDay.length` — caught by the upfront guard, returns `-1` immediately.

---

## Extra Practice 2 (Extension): Magnetic Force Between Two Balls (LeetCode 1552, Medium) — Pattern: On the Answer, *Maximizing* Instead of Minimizing

**Statement:** `position[i]` gives basket locations (not necessarily sorted). Place `m` balls into baskets so that the **minimum** distance between any two placed balls is as **large** as possible. Return that maximum possible minimum distance.

### Approach — binary search on distance, greedy placement as the feasibility check

```java
public static int maxDistance(int[] position, int m) {
    Arrays.sort(position);
    int left = 1, right = position[position.length - 1] - position[0];
    int best = 0;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (canPlace(position, m, mid)) {
            best = mid;
            left = mid + 1;    // this distance worked — try for something even LARGER
        } else {
            right = mid - 1;   // too ambitious — need a smaller minimum distance
        }
    }
    return best;
}

private static boolean canPlace(int[] position, int m, int minDist) {
    int count = 1;                  // first ball always goes in the lowest-position basket
    int lastPosition = position[0];
    for (int i = 1; i < position.length; i++) {
        if (position[i] - lastPosition >= minDist) {
            count++;
            lastPosition = position[i];
        }
    }
    return count >= m;
}
```

**⚠️ Common Mistake, and the single most planted trap in this exact problem:** `position` is **not** guaranteed sorted by the problem statement. Skipping `Arrays.sort(position)` produces a greedy placement that means nothing, since "place the next ball at the next basket that's far enough away" only makes sense when scanning baskets in position order.

**🔑 Key Takeaway — this is the mirror image of every "on the answer" problem so far, and the wording is the tell:** Koko, Ship, and Bouquets all ask for a **minimum** value subject to feasibility — larger candidate values only make feasibility *easier*, so the feasible region is `[threshold, ∞)`, and on a successful check the search pulls the upper bound down (`right = mid`) hunting for something smaller. Here, feasibility runs the other way: a **larger** minimum distance is *harder* to satisfy with only `m` balls (if a spacing of `d` works, any smaller spacing `d' < d` trivially also works using the same placement — but the reverse doesn't hold). So the feasible region is `(0, threshold]`, and on a successful check the search pushes the *lower* bound up (`left = mid + 1`), hunting for something larger, tracking the best feasible value seen in a separate `best` variable rather than relying on `left == right` convergence alone. Recognizing this flip from the problem's own wording — "maximize the *minimum*" is the giveaway phrase — before writing a single line of code, is the actual skill being tested.

**Worked trace:** `position = [1,2,3,4,7]` (already sorted), `m = 3`. `left = 1, right = 7 - 1 = 6`.

| left | right | mid | greedy placement | balls placed | ≥ 3? | action |
|---|---|---|---|---|---|---|
| 1 | 6 | 3 | `1 → 4 (gap 3) → 7 (gap 3)` | 3 | yes | `best=3, left=4` |
| 4 | 6 | 5 | `1 → 7 (gap 6)` | 2 | no | `right=4` |
| 4 | 4 | 4 | `1 → 7 (gap 6)` | 2 | no | `right=3` |

`left=4 > right=3`, loop ends → returns `best = 3` (matches the known answer for this input).

**Complexity:** Time O(n log n) for the sort, plus O(n log(range)) for the binary search itself — the sort typically dominates for large `n`. Space O(1) beyond the sort.

**Edge cases:**
- `m == 2` — reduces to simply the distance between the two extreme baskets.
- `m == position.length` — every basket must be used; the answer becomes the smallest gap between consecutive sorted positions.
- Duplicate positions — handled correctly since a gap of `0` is never `>=` any `minDist >= 1`, so a duplicate basket is naturally skipped by the greedy placement.

---

## This Closes Binary Search: 11 Required Problems, Easy Through Medium — Up From 8 in the Original Plan

**Median of Two Sorted Arrays (LeetCode 4, Hard) is deliberately *not* part of this ladder.** It's scheduled right after Trees complete, once divide-and-conquer intuition is properly established — exactly what the original plan promised on its own Day 21 and then never actually delivered. Flagging it again here, explicitly, so it doesn't get lost a second time.

---

# Part 3 — Binary Search, Reviewed: "On the Input" vs. "On the Answer"

### Prerequisites (confirmed)

- Every binary search problem from Day 28 through today.

Day 28 first drew this line — LC 704 (exact match, on the input) against LC 278 (boundary search, on the answer) — but with only two data points, it was easy to file away as a minor variation rather than the deepest fork in the whole topic. Five days and thirteen more problems later, it's worth stating precisely and testing against everything covered.

**On the input:** the binary search operates directly over **positions in a given array** (or a well-defined slice of it). The array itself — sorted, locally well-behaved, or rotated-but-structured — supplies the monotonic property being exploited. The output is typically an index, or a value read directly from the array.

**On the answer:** the binary search operates over a **range of candidate answer values that may have nothing to do with any array index** — a speed, a capacity, a day number, a distance. A **feasibility function**, which has to be actively constructed rather than read off a data structure, supplies the monotonic property instead: "does this candidate answer work?" evaluated across an ordered range, transitioning from infeasible to feasible (or vice versa) exactly once.

**🔑 Key Takeaway — why "on the answer" is the one that gets missed:** "on the input" problems announce themselves — there's a sorted (or nearly-sorted) array sitting right there. "On the answer" problems don't look like search problems at all on first read; they look like optimization or simulation problems. The tell is almost always in the phrasing: *"minimum X such that Y is achievable"* or *"maximum X such that Y remains achievable."* Recognizing that phrasing, and then actively building the feasibility function yourself, is the entire skill — nothing about the input data structure hints at it the way a sorted array does.

### Full Classification — Every Binary Search Problem, Two Weeks Combined

| Day | LC # | Problem | Framing |
|---|---|---|---|
| 28 | 704 | Binary Search | On the input — exact match |
| 28 | 278 | First Bad Version | On the answer — boundary/feasibility (first appearance) |
| 29 | 35 | Search Insert Position | On the input — exact match, insertion point |
| 29 | 162 | Find Peak Element | On the input — local slope rule, not global sort |
| 30 | 33 | Search in Rotated Sorted Array | On the input — rotated, one half always sorted |
| 30 | 81 | Search in Rotated Sorted Array II | On the input — rotated, with duplicates |
| 31 | 153 | Find Minimum in Rotated Sorted Array | On the input — rotated minimum |
| 31 | 154 *(extra)* | Find Minimum in Rotated Sorted Array II | On the input — rotated minimum, with duplicates |
| 31 | 34 | Find First and Last Position | On the input — boundary search, two-sided |
| 31 | 744 *(extra)* | Find Smallest Letter Greater Than Target | On the input — boundary search, one-sided |
| 32 | 74 | Search a 2D Matrix | On the input — 2D flattened to 1D |
| 32 | 875 | Koko Eating Bananas | On the answer — feasibility (speed), minimize |
| 33 | 1011 | Capacity To Ship Packages Within D Days | On the answer — feasibility (capacity), minimize |
| 33 | 1482 *(extra)* | Minimum Number of Days to Make m Bouquets | On the answer — feasibility (day), minimize, adjacency-aware check |
| 33 | 1552 *(extra)* | Magnetic Force Between Two Balls | On the answer — feasibility (distance), **maximize** |

**11 required + 4 extra = 15 distinct Binary Search problems solved across Weeks 4–5.** 9 "on the input," 6 "on the answer" (counting LC 278) — roughly even, which is itself worth noting: this isn't a rare edge case worth a footnote, it's a full half of the pattern's real interview weight.

---

## Project Block Guide (1 hr)

**Repository:** `dsa-java/binary-search/`. No new problem today — confirm all 11 required solutions (plus the 4 extras, kept clearly labeled as such) are pushed and organized, one folder per problem, matching the structure used all week. Definition of done: a `git log` or directory listing showing all 15 present and correctly labeled required vs. extra.

---

## Career Block Guide (1 hr)

**LinkedIn (20 min):** comment meaningfully on 3–5 posts.

**Networking:** identify 3 companies with strong engineering blogs (general list, same as Day 29 — building the pool of targets ahead of when outreach actually starts).

---

## Day 33 — Interview Questions

**Q1. What's the one-sentence definition of "binary search on the answer"?** Searching over a range of candidate answer values (not array positions) using a monotonic feasibility function, rather than searching over positions in a given array.

**Q2. What phrasing in a problem statement should make you suspect an "on the answer" problem?** "Minimum/maximum X such that Y is achievable" — the answer being optimized isn't sitting in any array; it has to be searched over as a value.

**Q3. Ship Capacity and Koko Eating Bananas share almost the entire algorithm. What's the one part that's genuinely different between them?** Only the feasibility check's internals — Koko sums ceiling-divided hours per pile; Ship simulates a greedy day-by-day loading process. The outer binary-search-on-a-monotonic-condition shell is identical.

**Q4. Why does Minimum Days to Make m Bouquets need an upfront `-1` check that none of the other on-the-answer problems this week needed?** Because feasibility there can be impossible in principle — if `m × k` exceeds the total number of flowers, no day, however large, ever creates enough of them, since waiting doesn't create new flowers. Every other problem this week is guaranteed feasible for some value in its search range.

**Q5. Why would counting total bloomed flowers and dividing by `k` give a wrong answer for the Bouquets problem?** Because bouquets need `k` *adjacent* bloomed flowers, not just `k` bloomed flowers anywhere — three isolated bloomed flowers with no two adjacent can make zero bouquets of size two, even though naive division would suggest one.

**Q6. What's different about Magnetic Force Between Two Balls compared to every other "on the answer" problem this week?** It maximizes the answer instead of minimizing it — a larger minimum distance is harder to achieve with a fixed number of balls, so the feasible region and the binary search's shrink direction both flip relative to the minimize-type problems.

**Q7. In Magnetic Force, why does a successful feasibility check push `left` up instead of pulling `right` down?** Because the goal is the *largest* feasible distance — on success, the algorithm records the current candidate as the best answer so far and searches strictly above it for something even larger, rather than searching below it for something smaller.

**Q8. Classify Search a 2D Matrix and Koko Eating Bananas: on the input, or on the answer?** Search a 2D Matrix is on the input — the flattened matrix supplies the sorted structure being searched. Koko is on the answer — the search is over eating speeds, using a constructed feasibility check, with no array being searched at all.

**Q9. Across all 15 Binary Search problems from Weeks 4 and 5, roughly how evenly split are the two framings?** Close to even — 9 on the input, 6 on the answer — which is itself worth remembering: "on the answer" isn't a rare exception, it accounts for close to half the pattern's real interview weight.

---

## Daily Deliverable Check

- [ ] Capacity To Ship Packages Within D Days solved — Binary Search ladder complete at 11 required problems.
- [ ] Extra practice: Minimum Number of Days to Make m Bouquets and Magnetic Force Between Two Balls solved, pushed, clearly labeled as extension material — both checked against `00_Curriculum_Map.md` and `Week_06_Revised.md` beforehand; no collisions found.
- [ ] "On the input vs. on the answer" full classification internalized — can sort all 15 problems into the right bucket without notes.
- [ ] All 11 required + 4 extra Binary Search solutions confirmed pushed and organized in `dsa-java/binary-search/`.

---

## What Tomorrow Assumes You Already Know Cold

Binary Search is done — Day 34 doesn't extend it and won't reference it again except by citation if it resurfaces later in the series (Median of Two Sorted Arrays, after Trees). What Day 34 *does* assume, from further back: OOP fundamentals (Day 2) are fully solid, since Linked Lists' `ListNode` is nothing more than a class with a value field and a reference field, and Day 9's stack/heap mechanism is fully solid too, since that reference field is about to be explained as exactly the same kind of heap pointer Day 9 already covered — Linked Lists doesn't introduce a new memory concept, only a new *shape* built from one already in hand.
