# Day 26 — Meeting Rooms: Intervals Meet the Min-Heap

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 25 Resource Book](Day25_Resource_Book.md)
**Next ▶:** [Day 27 Resource Book](Day27_Resource_Book.md)
**Companion to:** Day 26 of `Week_04_Revised.md`

---

## Recap

The last three days built the Intervals mechanism up from "sort and sweep once" (Day 24, start time) through "sort and sweep, greedy-by-earliest-end" (Day 25, end time). Today's two problems ask a question neither of those directly answers: **how many meeting rooms are needed at once**, given a set of meeting time intervals — which needs a way to track *multiple* simultaneously "in progress" end times, not just one running boundary. That's exactly what Week 3, Day 17's `PriorityQueue` (min-heap) is for — already fully taught, not being re-derived today, only **applied** to intervals for the first time.

Today has no Theory, Project, or Career Block in the plan — DSA only, at 3–3.5 hours.

**A practical note the plan itself flags:** both of today's problems are tagged LeetCode Premium in some regions. If Meeting Rooms (LC 252) is inaccessible, treat its section below as a warm-up read for Meeting Rooms II rather than something you need to submit — the reasoning fully carries over regardless of whether you can click "Submit" on the easier version first.

---

## Learning Objectives

By the end of today, without notes:

1. Solve Meeting Rooms via a direct overlap check after sorting by start time — recognizing it as a simplified special case of Day 25's overlap-checking logic (existence, not counting).
2. Apply a min-heap of end times to Meeting Rooms II, tracking how many rooms are simultaneously occupied, and explain why touching meetings (one starts exactly as another ends) do **not** require a separate room.
3. Explain precisely why the heap's **final** size already equals the maximum concurrent room count — without needing a separately tracked running maximum — via the fact that the heap only ever grows or stays flat, never shrinks independently.

---

## Concept Dependency Map

```
Week 3, Day 17: PriorityQueue (min-heap) — O(log n) insert/poll, O(1) peek — already known
Day 24-25: Intervals — sort, sweep once; overlap condition (start <= end, direction problem-specific)
        │
        ▼
LC 252 Meeting Rooms — sort by start, direct adjacent-overlap check (simplest form: does ANY overlap exist)
        │
        ▼
LC 253 Meeting Rooms II — NEW APPLICATION: min-heap of end times tracks
   "how many rooms are occupied right now" as meetings are processed in start-time order
   (heap size only ever grows or holds steady → final size = the answer, no separate max needed)
        │
        ▼
Tomorrow: Partition Labels (interval reasoning WITHOUT an explicit interval object)
   + Greedy & Intervals pattern closes at 11 required + 2 extra
```

---

## Problem: Meeting Rooms (LeetCode 252, Easy — Premium in some regions) — Pattern: Intervals

**Statement:** given an array of meeting time intervals, determine if a person could attend **all** of them — equivalently, whether any two intervals overlap at all.

### Approach — sort by start time, check each adjacent pair

```java
public static boolean canAttendMeetings(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));   // sort by start time

    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < intervals[i - 1][1]) {
            return false;   // this meeting starts before the previous one ends — conflict
        }
    }
    return true;
}
```

**Why checking only *adjacent* pairs after sorting is sufficient — no need for an all-pairs check:** once sorted by start time, if interval `i` doesn't overlap interval `i-1` (its immediate predecessor in start order), it's guaranteed not to overlap anything *before* `i-1` either, since everything before `i-1` ends no later than `i-1` itself does in the worst realistic case relevant here — more precisely, the only way a conflict can exist at all is between some interval and the one immediately preceding it in sorted order or a later-checked one, because a genuine overlap between two intervals that are *not* adjacent in sorted order would require the intervals between them to also overlap at least one of the two (since start times are monotonically non-decreasing) — meaning that overlap would already have been caught at some adjacent comparison along the way. This is the same "sorting collapses the search to immediate neighbors" argument from Day 24, applied to existence-checking instead of merging.

**Worked trace:** `intervals = [[0,30],[5,10],[15,20]]`. Sort by start (already sorted here).

| i | intervals[i][0] | intervals[i-1][1] | conflict? |
|---|---|---|---|
| 1 | 5 | 30 | 5 < 30 → **yes, return false** |

Matches the known expected output for this exact input — `[5,10]` starts well before `[0,30]` ends.

**Complexity:** Time O(n log n) — dominated by the sort. Space O(1) extra.

**Edge cases:** empty input or single meeting (loop doesn't execute, correctly returns `true` — trivially no conflict possible); touching meetings, `[1,5]` and `[5,10]` (condition is `intervals[i][0] < intervals[i-1][1]`, strict — `5 < 5` is false, so touching meetings do **not** count as a conflict, matching the everyday intuition that one meeting can start the instant another ends); fully identical intervals repeated (immediately flagged as a conflict on the first comparison, correctly).

💡 **Interview Insight:** this problem is worth explicitly framing as "the existence version" of a question Meeting Rooms II (next) answers in "counting" form — naming that relationship before moving to the harder problem sets up the heap-based solution as a natural generalization rather than an unrelated new technique.

---

## Problem: Meeting Rooms II (LeetCode 253, Medium — Premium in some regions) — Pattern: Intervals, Min-Heap of End Times

**Statement:** given an array of meeting time intervals, return the **minimum number of conference rooms required** so that every meeting can be held without any two overlapping in the same room.

### Approach 1 — Brute force (conceptual)

For each meeting, check it against every room currently in use to see if any room's current meeting has already ended; if so, reuse that room, otherwise open a new one. Implemented naively (scanning every open room for every meeting), this is O(n²) — worth naming, but not the expected final answer.

### Approach 2 — Optimized: sort by start time, min-heap of end times

```java
public static int minMeetingRooms(int[][] intervals) {
    if (intervals.length == 0) return 0;
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));   // sort by start time

    PriorityQueue<Integer> endTimes = new PriorityQueue<>();   // min-heap, default ordering

    for (int[] interval : intervals) {
        if (!endTimes.isEmpty() && endTimes.peek() <= interval[0]) {
            endTimes.poll();   // the earliest-ending occupied room has freed up — reuse it
        }
        endTimes.offer(interval[1]);   // this meeting now occupies a room (reused or new)
    }
    return endTimes.size();
}
```

**What the heap represents at every point in the loop:** each value in `endTimes` is the end time of a meeting currently "occupying" a room, among meetings whose start time is `<=` the meeting currently being processed. The heap's **size**, at any point, is exactly the number of rooms simultaneously in use for meetings processed so far.

**Why `endTimes.peek() <= interval[0]` is the correct reuse condition:** `peek()` returns the *smallest* end time currently in the heap — the room that will free up (or already has) the soonest. If that soonest-freeing room's end time is at or before the current meeting's start time, that room is genuinely available (a room vacated at time `5` can host a meeting starting at time `5`, matching Meeting Rooms' own "touching is fine" convention from the problem above) — reuse it. If even the *soonest*-freeing room isn't free in time, then **no** room is free, since every other room's end time in the heap is at least as large.

**Why the final heap size — not a separately tracked running maximum — is already the answer, proven:** look at what happens to the heap's size on each iteration. Exactly one of two things occurs: **(a)** a room is reused — `poll()` immediately followed by `offer()`, net size change zero — or **(b)** no room is available — only `offer()` happens, size increases by exactly one. There is **no** operation that decreases the heap's size on its own; every `poll()` is immediately paired with an `offer()` in the very next line. This means the heap's size is **monotonically non-decreasing** across the entire loop — it only ever grows or holds steady, never shrinks. A sequence that never decreases reaches its maximum value exactly at its final value. So the size after the last meeting is processed is, by construction, the largest size the heap ever reached — no separate `maxRooms` variable is needed.

**Worked trace:** `intervals = [[0,30],[5,10],[15,20]]`. Sort by start (already sorted).

| interval | endTimes before | peek <= start? | action | endTimes after |
|---|---|---|---|---|
| [0,30] | {} | — (empty) | offer(30) | {30} |
| [5,10] | {30} | 30<=5? no | offer(10) | {10,30} |
| [15,20] | {10,30} | 10<=15? **yes** | poll() → {30}, offer(20) | {20,30} |

Final `endTimes.size() = 2` — matches the known expected output for this exact input (meeting `[0,30]` needs its own room the entire time; `[5,10]` and `[15,20]` can share a second room, since `[5,10]` finishes before `[15,20]` begins).

**Complexity:** Time O(n log n) — the sort is O(n log n); each of the `n` meetings triggers at most one `poll()` and one `offer()`, each O(log n) on a heap of size up to n, giving O(n log n) total for the loop as well. Space O(n) worst case, for the heap.

**Edge cases:** empty input (`0`, via the early return); every meeting mutually overlapping (heap grows to size `n` and never shrinks — every room stays occupied simultaneously, correctly requiring `n` rooms); every meeting sequential with no overlap at all (the reuse condition fires every single time after the first, heap size stays at `1` throughout, correctly needing only one room); meetings that only touch (`[1,5]`, `[5,10]`) — reuse fires correctly via the non-strict `<=`, matching Meeting Rooms' own touching-is-fine convention from the problem above, worth noticing this is the *opposite* inequality direction from Day 25's Non-overlapping Intervals (which used strict `<` for "counts as overlap") — two different problems, two different conventions for what "touching" means, and getting them mixed up under pressure is a genuine risk worth guarding against explicitly.

⚠️ **Common Mistake:** tracking a separate `maxRooms = Math.max(maxRooms, endTimes.size())` variable inside the loop, out of an instinct that "size at the end might not be the max." It's unnecessary work here specifically *because* of the monotonic-growth proof above — stating that proof explicitly, rather than defensively tracking a maximum you don't need, is the stronger interview answer, though tracking it defensively isn't *wrong*, just redundant once the monotonicity is understood.

💡 **Interview Insight:** if asked "why a min-heap and not, say, a sorted list of end times," the answer is about *which* operations matter: you only ever need the single *smallest* end time (to check "is the soonest-freeing room actually free"), never the full sorted order — a min-heap gives O(log n) access to exactly that one value on both insert and removal, which is strictly what's needed and no more, whereas maintaining a fully sorted structure would do unnecessary extra work to preserve an ordering nothing here actually queries beyond its minimum.

---

## Day 26 — Interview Questions

**Q1. Why does Meeting Rooms only need to check adjacent pairs after sorting by start time, not every pair?** Sorting collapses the search: if interval `i` doesn't conflict with its immediate predecessor `i-1` in start-time order, any conflict further back would already have propagated forward and been caught at some adjacent comparison along the way, since start times only increase moving forward through the sorted array.

---

**Q2. What does the min-heap in Meeting Rooms II represent at any point during the loop?** The end times of all meetings currently occupying a room, among meetings processed so far — its size is exactly the number of rooms simultaneously in use at that point in the scan.

---

**Q3. Why is `endTimes.peek() <= interval[0]` the correct condition for reusing a room?** `peek()` gives the soonest-freeing occupied room's end time. If even that soonest one isn't free by the current meeting's start, no room is free, since every other end time in the heap is at least as large — so checking only the minimum is sufficient to decide reusability.

---

**Q4. Why does the heap's final size equal the maximum number of rooms ever needed, without tracking a separate maximum?** Every loop iteration either reuses a room (poll immediately followed by offer, net size change zero) or opens a new one (offer only, size increases by one) — there's no operation that decreases the heap's size independently. A sequence that never decreases is at its maximum exactly at its final value.

---

**Q5. Why a min-heap specifically, rather than a fully sorted structure of end times?** Only the single smallest end time is ever queried (to check whether the soonest-freeing room is actually free) — never the full ordering. A min-heap gives O(log n) access to exactly that one value on both insert and removal, without paying for a stronger ordering guarantee nothing here uses.

---

**Q6. Contrast the "touching" convention in Meeting Rooms II with Day 25's Non-overlapping Intervals.** Meeting Rooms II treats a touch (`peek() == start`) as **freeing up** the room — reuse is allowed, via a non-strict `<=`. Non-overlapping Intervals treats a touch as **not** overlapping (via a strict `<` for its own "overlap" check) — both conventions ultimately agree that touching endpoints don't create a conflict, but they're implemented with opposite-feeling inequality directions relative to what each problem is checking, worth verifying explicitly rather than assuming.

---

## Daily Deliverable Check

- [ ] Meeting Rooms and Meeting Rooms II solved (or Meeting Rooms II solved standalone, with Meeting Rooms treated as a reasoning warm-up, if Premium access is unavailable), pushed to `dsa-java/greedy-intervals/`.
- [ ] Can state the monotonic-heap-growth proof for Meeting Rooms II from memory, without needing to trace through an example to re-derive it.
- [ ] Can explain why a min-heap is the right structure here, specifically because only the minimum end time is ever queried.

---

## What Tomorrow Assumes You Already Know Cold

Day 27 assumes today's heap-based room-tracking is solid, but doesn't build on it directly — Partition Labels (tomorrow's sole new problem) uses interval-style reasoning (a "last occurrence" sweep) without an explicit `[start, end]` object or a heap at all, so today's specific mechanism isn't a hard prerequisite. What tomorrow does lean on is the full accumulated Greedy & Intervals vocabulary from this entire week — sort-by-start vs. sort-by-end, the exchange argument, the "which quantity are we tracking" instinct — all of which needs to be genuinely automatic by tomorrow's capstone and pattern-closing consolidation.

**Next:** [Day 27 Resource Book](Day27_Resource_Book.md) — Greedy & Intervals Capstone, and Leave Week Wrap-Up.
