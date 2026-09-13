# Week 4 — Consolidated Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Companion to:** `Week_04_Revised.md`, Days 22–28

Every question and answer from this week's seven Resource Books, pulled into one review document — 54 questions total, organized by day. Use this for spaced review rather than re-reading each day's book in full; anywhere an answer doesn't come immediately is worth returning to that day's book for the fuller worked treatment.

**Jump to:** [Day 22](#day-22--prefix-sum--kadanes-completes) · [Day 23](#day-23--greedy--intervals-begins) · [Day 24](#day-24--greedy-continues-and-intervals-begins) · [Day 25](#day-25--intervals-sorted-by-end-time) · [Day 26](#day-26--meeting-rooms) · [Day 27](#day-27--greedy--intervals-capstone) · [Day 28](#day-28--consolidation-and-binary-search-begins)

---

## Day 22 — Prefix Sum & Kadane's Completes

*Full treatment: [Day22_Resource_Book.md](Day22_Resource_Book.md)*

**Q1. What's the core reframe that lets a HashMap answer "does a subarray summing to k exist" in O(n) instead of O(n²)?** Rearrange `prefix[j+1] - prefix[i] = k` to `prefix[i] = prefix[j+1] - k`, then ask a HashMap "have I already seen a prefix sum equal to `(current prefix sum) - k`" at each position — an O(1) average lookup replacing an O(n) backward search.

---

**Q2. When should the HashMap store a first-occurrence index versus a frequency count?** First-occurrence index when the question is "where" or "how long" (a later, closer match can only shrink or equal the best answer, never beat it) — frequency count when the question is "how many" (every prior matching occurrence marks a separate valid subarray, so all of them must be counted).

---

**Q3. In Contiguous Array, why does treating every 0 as -1 make this a prefix-sum problem?** Equal counts of 0s and 1s in a subarray is equivalent to that subarray's transformed sum being exactly 0 — turning "equal counts of two categories" into "prefix sum equals a target," which is the same identity already used for exact-sum matching.

---

**Q4. Why must Contiguous Array's map keep only the *first* occurrence of each running sum, never overwrite it?** The longest valid subarray ending at the current index pairs with the *earliest* index sharing the same running sum — overwriting with a later index would silently discard a longer answer found afterward.

---

**Q5. Why doesn't Continuous Subarray Sum (LC 523) need the `((x % k) + k) % k` negative-remainder normalization, while Subarray Sums Divisible by K (LC 974) does?** LC 523 guarantees `nums[i] >= 0`, so every running sum stays non-negative and Java's `%` already returns a value in `[0, k-1]`. LC 974 allows negative elements, so the running sum can go negative, and Java's `%` follows the sign of the dividend — producing a negative "remainder" that would silently split one true remainder bucket into two unless normalized.

---

**Q6. Why does Maximum Subarray Sum Circular's wrapping case reduce to `total sum − minimum subarray sum`?** A wrapping subarray's excluded elements form one contiguous non-wrapping block in the middle of the array; maximizing what's included is equivalent to minimizing what's excluded, and the minimum-sum contiguous block is found by Kadane's algorithm run with `min` in place of `max`.

---

**Q7. Why does an all-negative array break the naive `max(maxKadane, total - minKadane)` formula, and how is it fixed?** If every element is negative, the minimum-sum subarray is the entire array, making the wrapping candidate `total - total = 0` — but `0` implies an empty subarray, which isn't a valid answer for an all-negative, non-empty-required input. The fix is to check whether the ordinary (non-wrapping) Kadane's maximum is already negative; if so, skip the wrapping computation and return that value directly.

---

**Q8. In Maximum Product Subarray, why is tracking a single running max insufficient?** Multiplying a large-magnitude *negative* running product by one more negative number can produce the new largest *positive* product — information a single running max, having already discarded very negative values as "bad," would have no way to recover.

---

**Q9. Why does swapping `maxEndingHere` and `minEndingHere` before multiplying by a negative number keep the recurrence correct?** Multiplying by a negative number reverses order — what was the largest product ending at the previous position becomes a candidate for the new smallest, and vice versa. Swapping first means the same `max(num, maxEndingHere*num)` / `min(num, minEndingHere*num)` lines stay correct regardless of the current number's sign.

---

**Q10. What closes today, and what's the final count?** Prefix Sum & Kadane's — 7 required (3 from Day 21, 4 from today) + 2 extra (LC 974, LC 152, both today) = 9 distinct problems. The second pattern family in this series with zero presence anywhere in the original 17-week plan.

---

## Day 23 — Greedy & Intervals Begins

*Full treatment: [Day23_Resource_Book.md](Day23_Resource_Book.md)*

**Q1. What's the difference between "greedy" as a description of an algorithm's behavior and "greedy" as a justified, correct strategy?** Any algorithm that makes a locally-best choice at each step without reconsidering it is *describable* as greedy — that alone says nothing about correctness. It becomes a justified strategy only once an exchange argument (or equivalent proof) shows the locally-best choice can always be swapped into an optimal solution without making it worse.

---

**Q2. Give an example where greedy provably fails, and say exactly why.** 0/1 Knapsack with capacity 10 and items (6,12), (5,10), (5,10) — greedy-by-value-density picks the first item (density 2, same as the others), leaving capacity 4, insufficient for either remaining item, for a total value of 12. The true optimum takes the second and third items together (weight 10 exactly, value 20). Greedy's early choice blocks a better later combination, and no exchange argument can recover the lost value.

---

**Q3. Why is Best Time to Buy/Sell Stock II's optimal strategy fundamentally different from Day 14's single-transaction version, not just a small variation?** Removing the one-transaction limit means any longer uphill price run can be freely decomposed into consecutive single-day transactions without losing profit (a telescoping identity: `(b-a)+(c-b) = c-a`), so capturing every positive day-over-day delta independently becomes optimal — a strategy that would be actively wrong under a one-transaction limit, where you must commit to a single buy/sell pair.

---

**Q4. In Jump Game, why is it safe to discard *how* a position was reached and track only the single farthest-reachable index?** The farthest reachable index strictly dominates every closer reachable index — anything reachable from a closer point remains reachable from the farthest point too, since the farthest point has at least as much jump range available at every subsequent step. Any strategy tracking a closer point can be replaced by one tracking the farthest point without ever losing reachability.

---

**Q5. In Jump Game II, what does `currentJumpEnd` represent, and why does reaching it force a jump count increment?** It's the farthest index reachable using the jumps already counted. Once the scan reaches that exact index, continuing forward requires a jump has already been used to arrive there, so committing to the next jump (incrementing the count, extending the boundary to `farthestReachable`) is forced, not a choice.

---

**Q6. Why does Jump Game II's loop stop one index before the array's end?** If the loop reached the final index and that index happened to equal `currentJumpEnd`, the code would increment `jumps` one unnecessary time for a "jump" that's never actually needed, since the destination is already reached.

---

**Q7. Why is Jump Game II's greedy strategy described as "BFS by levels, in disguise"?** Each jump corresponds to one BFS level — the set of indices reachable in exactly that many jumps. `currentJumpEnd` marks the boundary of the current level; `farthestReachable`, updated continuously while scanning the current level, becomes the next level's boundary the moment the current one is exhausted. This mirrors BFS's level-by-level exploration without needing an explicit queue, and is exactly why the strategy finds the *minimum* jump count, not just *a* valid one.

---

## Day 24 — Greedy Continues, and Intervals Begins

*Full treatment: [Day24_Resource_Book.md](Day24_Resource_Book.md)*

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

## Day 25 — Intervals, Sorted by End Time

*Full treatment: [Day25_Resource_Book.md](Day25_Resource_Book.md)*

**Q1. Why does Non-overlapping Intervals sort by end time instead of start time?** The greedy goal is to keep whichever interval, among any overlapping cluster, leaves the most room for everything that follows — that's the interval with the earliest end time, not necessarily the earliest start time. Sorting by end time is what makes "the next kept interval" always the correct greedy choice.

---

**Q2. State the exchange argument for keeping the earliest-ending interval in an overlapping cluster.** If an optimal solution kept some other interval from the cluster instead, swapping in the earliest-ending one instead can only leave equal or more room for what comes next (since it ends no later), so the swap never makes the solution worse — meaning some optimal solution always agrees with the greedy choice.

---

**Q3. How does Minimum Arrows to Burst Balloons relate to Non-overlapping Intervals?** It's the identical greedy-by-end-time mechanism, reframed — "discard the interval that doesn't survive" becomes "this balloon needs its own new arrow," and the trigger condition flips sign accordingly (overlap-so-discard becomes no-overlap-so-need-new-arrow), but both count the same underlying quantity: the number of independent overlapping clusters.

---

**Q4. In Minimum Arrows, why does a touching pair like `[1,2]` and `[2,3]` share one arrow, while in Merge Intervals a touching pair gets merged?** Both are closed-boundary conventions, but they're independently defined by each problem: Minimum Arrows uses a *strict* `>` for "not yet covered" (so a touch, `==`, still counts as covered, sharing the arrow); Merge Intervals uses a *strict* `<` for "no overlap" (so a touch, `==`, does **not** count as "no overlap," triggering a merge). They happen to produce the same practical outcome (touching intervals get combined/shared) via differently-signed conditions.

---

**Q5. Why does Interval List Intersections use two pointers instead of a single-list sweep?** The input is two *separate*, independently sorted lists whose cross-relationships are being queried, not one list whose internal overlaps need combining — a structurally different question from Merge Intervals, requiring one pointer per list rather than one running "currently building" interval.

---

**Q6. In Interval List Intersections, why is it always safe to advance the pointer belonging to whichever interval ends first?** Both lists are independently sorted, so every remaining interval in the other list starts even later than the current one being compared. An interval that already ends before the current position in the other list cannot overlap anything further in that other list — advancing past it loses no possible future intersection.

---

## Day 26 — Meeting Rooms

*Full treatment: [Day26_Resource_Book.md](Day26_Resource_Book.md)*

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

**Q6. Contrast the "touching" convention in Meeting Rooms II with Non-overlapping Intervals.** Meeting Rooms II treats a touch (`peek() == start`) as **freeing up** the room — reuse is allowed, via a non-strict `<=`. Non-overlapping Intervals treats a touch as **not** overlapping (via a strict `<` for its own "overlap" check) — both conventions ultimately agree that touching endpoints don't create a conflict, but they're implemented with opposite-feeling inequality directions relative to what each problem is checking.

---

## Day 27 — Greedy & Intervals Capstone

*Full treatment: [Day27_Resource_Book.md](Day27_Resource_Book.md)*

**Q1. How does Partition Labels turn a string-partitioning problem into interval reasoning without ever building an explicit interval object?** Each character implicitly defines a range from its first to its last occurrence — no two occurrences of the same character can be split across partitions, so a single pass tracking the running maximum "last occurrence seen so far" plays the same role Merge Intervals' explicit end-time tracking does, just derived on the fly instead of given upfront.

---

**Q2. Why is the `lastOccurrence` lookup array O(1) space, regardless of the input string's length?** It's always exactly 26 entries (one per lowercase letter), a fixed, bounded key space independent of `s`'s length — the same reasoning used for every fixed-alphabet frequency structure since Week 1's Valid Anagram.

---

**Q3. In Partition Labels, what does `i == end` signify, and what earlier problem's "commit" moment does it mirror?** It signifies that every character seen since the current partition's start has had its last occurrence fully accounted for — nothing still open requires the partition to stay open. This mirrors Jump Game II's `i == currentJumpEnd` moment: the boundary has been reached, so committing (closing the partition / taking the jump) is forced, not optional.

---

**Q4. State Jump Game's safety argument in one sentence.** The farthest reachable index dominates every closer one, since anything reachable from a closer point remains reachable from the farthest point too — tracking only the farthest value loses no information about ultimate reachability.

---

**Q5. State Non-overlapping Intervals' safety argument in one sentence.** Among overlapping intervals, the earliest-ending one leaves at least as much room for what follows as any alternative choice would, so swapping it into any optimal solution never makes that solution worse.

---

**Q6. State Gas Station's safety argument in one sentence.** If starting from `X` fails at `Y`, every station between them already has a non-negative banked surplus by the time it's reached — at least as much as a fresh start there would have — so they all fail at least as early too, and the entire range is eliminated at once.

---

**Q7. What do all three of these arguments have in common, structurally?** Each shows the greedy choice can be substituted into any alternative solution without making it worse — the specific mechanism differs (domination of a reachable set, an interval swap, elimination of a candidate range), but the underlying template ("show the swap is always safe") is the same one used throughout this series since Container With Most Water, back in Week 2.

---

**Q8. What is the final closing count for Greedy & Intervals, and how does it split between required and extra?** 11 required problems (LC 122, 55, 45, 134, 56, 57, 435, 452, 252, 253, 763) + 2 extra practice (LC 406, 986) = 13 distinct problems — the second pattern family in this series with zero presence anywhere in the original 17-week plan.

---

## Day 28 — Consolidation, and Binary Search Begins

*Full treatment: [Day28_Resource_Book.md](Day28_Resource_Book.md)*

**Q1. Why is `mid = left + (right - left) / 2` preferred over `mid = (left + right) / 2`?** The latter can overflow `int` when `left` and `right` are both large, silently wrapping to a garbage (possibly negative) value with no exception, per Week 2, Day 10's overflow mechanics. `(right - left)` is bounded by the size of the remaining search space, not by how large `left` and `right` individually are, so it can't overflow the same way.

---

**Q2. Why does LC 704's binary search use `left <= right` as its loop condition, not `left < right`?** When `left == right`, there's still one unchecked candidate — `nums[left]` itself — that must be examined before concluding the target is absent. `left < right` would skip that final check in some inputs.

---

**Q3. How does First Bad Version's binary search differ mechanically from LC 704's, and why?** It searches for a *boundary* in a monotonic true/false sequence, not an exact match against a known value. Since `mid` might itself be the answer when `isBadVersion(mid)` is true, it must stay in the search range (`right = mid`, not `mid - 1`), and the loop uses `left < right` since the scheme converges to `left == right` pointing at the answer, with no "not found" outcome to signal.

---

**Q4. What does "binary search on the answer" mean, and which of today's two problems previews it?** Instead of searching a given sorted array, you search a *range of candidate answers*, using a monotonic feasibility check to decide which half to keep — there's no array involved at all. First Bad Version previews this: the "array" is conceptually the range `[1, n]`, and `isBadVersion` is the monotonic feasibility check.

---

**Q5. What exactly does declaring `record Point(int x, int y) {}` generate?** A canonical constructor, private final fields `x` and `y`, accessor methods `x()` and `y()` (not `getX()`/`getY()`), `equals()`/`hashCode()` based on all components, a `toString()`, and an implicitly `final` class that can never be extended.

---

**Q6. Why is a record's auto-generated `equals()`/`hashCode()` pair guaranteed to satisfy the Week 1, Day 4 contract?** Both are generated from the exact same set of components, by construction — there's no way to end up with a record whose `equals()` and `hashCode()` disagree with each other, unlike a hand-written class where the two could be implemented inconsistently by mistake.

---

**Q7. What does `sealed interface PaymentState permits Pending, Success, Failed` guarantee, and what must each permitted type declare?** Only the three named types may ever implement `PaymentState`, checked at compile time — nothing else, anywhere in the codebase. Each permitted type must itself be declared `final`, `sealed`, or `non-sealed`, explicitly.

---

**Q8. Why do records satisfy a sealed interface's "must be final/sealed/non-sealed" requirement automatically?** Records are always implicitly `final` — they can never be extended — so this requirement is met with zero extra syntax whenever every permitted implementer is a record.

---

**Q9. The plan states this exercise "compiles on Java 17+." What's imprecise about that, exactly?** Records (Java 16+) and sealed interfaces (Java 17+) are each individually accurate. But exhaustive switch pattern matching over a sealed hierarchy — a `switch` using type patterns with no `default`, checked for completeness against the `permits` list — was a preview feature across Java 17–20 and became standard only in Java 21 (JEP 441). Compiling the exercise exactly as specified, with no `default` and no `--enable-preview` flag, requires Java 21 or later.

---

**Q10. What closed this week, and what opened, in one sentence each?** Prefix Sum & Kadane's closed at 9 distinct problems (7 required + 2 extra); Greedy & Intervals closed at 13 distinct problems (11 required + 2 extra) — both entirely new pattern families with zero presence in the original 17-week plan; Binary Search opened at 2/11 required, with extra practice deliberately deferred until it closes in Week 5.

---

## Week 4 Summary Table

| Day | Focus | Required Problems | Extra Practice |
|---|---|---|---|
| 22 | Prefix Sum & Kadane's completes | LC 525, 523, 560*, 918 | LC 974, 152 |
| 23 | Greedy begins | LC 122, 55, 45 | — |
| 24 | Greedy continues, Intervals begins | LC 134, 56, 57 | LC 406 |
| 25 | Intervals, sorted by end time | LC 435, 452 | LC 986 |
| 26 | Meeting Rooms (min-heap) | LC 252, 253 | — |
| 27 | Greedy & Intervals capstone | LC 763 | — |
| 28 | Binary Search begins; Records/Sealed Classes | LC 704, 278 | — |

*\*LC 560 recapped from Week 1, Day 5 — not a new solve.*

**Patterns closed this week:** Prefix Sum & Kadane's (9 distinct), Greedy & Intervals (13 distinct).
**Pattern opened this week:** Binary Search (2/11 required so far).
