# Week 4 (Revised): Prefix Sum Completes, Greedy & Intervals Completes, Binary Search Begins

**What changed:** this week finishes the leave-week sprint through both brand-new pattern families — Prefix Sum/Kadane's (7 problems total) and Greedy & Intervals (11 problems total), neither of which existed anywhere in the original 17-week plan — then transitions back to normal pace for the (now-expanded) Binary Search pattern.

---

## Day 22 — Prefix Sum Completes

### DSA Block (4-5 hrs at leave-week intensity)
- Problem 4: Contiguous Array — LeetCode #525 — Medium — Pattern: Prefix Sum + HashMap **(new)**
  - Hint: treat every `0` as `-1`. A subarray with equal 0s and 1s now has prefix sum 0 across it — track the *first* index each prefix-sum value occurs at in a HashMap.
  - Complexity: Time O(n) | Space O(n)
- Problem 5: Continuous Subarray Sum — LeetCode #523 — Medium — Pattern: Prefix Sum mod k **(new)**
  - Hint: if two prefix sums share the same remainder mod `k`, the subarray between them is divisible by `k`. Store the *first* index each remainder occurs at.
  - Complexity: Time O(n) | Space O(min(n,k))
- Problem 6: Subarray Sum Equals K — LeetCode #560 — Medium — Pattern: Prefix Sum + HashMap **(new)**
  - Hint: as you compute the running prefix sum, check whether `(prefixSum - k)` has already occurred — if it has, every occurrence marks a valid subarray ending here.
  - Complexity: Time O(n) | Space O(n)
- Problem 7: Maximum Subarray Sum Circular — LeetCode #918 — Medium — Pattern: Kadane's, twice **(new)**
  - Hint: the answer is either a normal (non-wrapping) max subarray, found with regular Kadane's — or a wrapping one, which equals `total sum - minimum subarray sum`. Handle the all-negative edge case separately (it breaks the second branch).
  - Complexity: Time O(n) | Space O(1)

**This closes Prefix Sum & Kadane's: 7 problems — a pattern family that didn't exist anywhere in the original plan.**

### Career Block (30 min)
- LinkedIn: engagement — a quick 10-15 minutes commenting on posts.

### Daily Deliverable
- [ ] Contiguous Array, Continuous Subarray Sum, Subarray Sum Equals K, and Maximum Subarray Sum Circular solved, pushed to `dsa-java/prefix-sum-kadanes/`.
- [ ] Prefix Sum & Kadane's pattern fully closed.

---

## Day 23 — Greedy & Intervals Begins

### DSA Block (4-5 hrs at leave-week intensity)

**Concept Card — Greedy & Intervals**
- What: two closely related ideas. *Greedy* means making the locally-best choice at each step and never revisiting it, trusting that local optimality adds up to global optimality (this only works for problems that actually have this property — proving it's safe is part of recognizing the pattern). *Intervals* problems almost always start with "sort by start time" or "sort by end time," then sweep once.
- Why: this is one of the most commonly tested "you either drilled it or you didn't" pattern families — Merge Intervals specifically shows up constantly, and there's no clever trick to derive it cold if you haven't seen the shape before.
- Where: scheduling, resource allocation, anywhere "process in order, commit to each choice" beats exhaustive search.
- Interview signal: "maximum profit/reach," "minimum number of X," "merge/insert/overlap," "can you attend all," "minimum resources needed."
- Prerequisites: sorting ✅ (used throughout since Two Pointers), arrays ✅.

- Problem 1: Best Time to Buy and Sell Stock II — LeetCode #122 — Easy — Pattern: Greedy **(new)**
  - Hint: unlike Day 14's version (one transaction only), you can buy/sell as many times as you like — just take every positive day-over-day delta.
  - Complexity: Time O(n) | Space O(1)
- Problem 2: Jump Game — LeetCode #55 — Medium — Pattern: Greedy **(new)**
  - Hint: track the farthest index reachable so far as you scan left to right. If you ever reach a position beyond the farthest-reachable mark, it's impossible.
  - Complexity: Time O(n) | Space O(1)
- Problem 3: Jump Game II — LeetCode #45 — Medium — Pattern: Greedy (BFS by levels, in disguise) **(new)**
  - Hint: track the current jump's boundary and the farthest reachable within the *next* jump. When you hit the current boundary, you're forced to take another jump — increment the count.
  - Complexity: Time O(n) | Space O(1)

### Career Block (30 min)
- LinkedIn: engagement — a quick 10-15 minutes commenting on posts.

### Daily Deliverable
- [ ] Best Time to Buy/Sell Stock II, Jump Game, and Jump Game II solved, pushed to `dsa-java/greedy-intervals/`.

---

## Day 24 — Greedy Continues, and Merge Intervals

### DSA Block (4-5 hrs at leave-week intensity)
- Problem 4: Gas Station — LeetCode #134 — Medium — Pattern: Greedy **(new)**
  - Hint: if total gas ≥ total cost, a valid starting point is guaranteed to exist. Track a running tank; whenever it goes negative, the start point can't be anywhere before the *next* station — reset there.
  - Complexity: Time O(n) | Space O(1)
- Problem 5: Merge Intervals — LeetCode #56 — Medium — Pattern: Intervals **(new)**
  - Hint: sort by start time. Walk through once — if the current interval overlaps the last merged one, extend it; otherwise start a new merged interval.
  - Complexity: Time O(n log n) | Space O(n)
- Problem 6: Insert Interval — LeetCode #57 — Medium — Pattern: Intervals **(new)**
  - Hint: three phases — add every interval that ends before the new one starts, merge every interval that overlaps the new one into it, add everything left over.
  - Complexity: Time O(n) | Space O(n)

### Daily Deliverable
- [ ] Gas Station, Merge Intervals, and Insert Interval solved, pushed to `dsa-java/greedy-intervals/`.

---

## Day 25 — Intervals: Non-Overlapping and Balloon-Popping

### DSA Block (3.5-4 hrs at leave-week intensity)
- Problem 7: Non-overlapping Intervals — LeetCode #435 — Medium — Pattern: Intervals (greedy by end time) **(new)**
  - Hint: sort by *end* time this time, not start. Greedily keep the interval that ends earliest, discard anything that overlaps it, repeat — this maximizes how many non-overlapping intervals survive.
  - Complexity: Time O(n log n) | Space O(1)
- Problem 8: Minimum Number of Arrows to Burst Balloons — LeetCode #452 — Medium — Pattern: Intervals (same shape as Non-overlapping Intervals) **(new)**
  - Hint: sort by end position. An arrow shot at the earliest end position pops every balloon whose start is before that point — count how many "resets" you need.
  - Complexity: Time O(n log n) | Space O(1)

### Daily Deliverable
- [ ] Non-overlapping Intervals and Minimum Number of Arrows to Burst Balloons solved, pushed.

---

## Day 26 — Meeting Rooms

### DSA Block (3-3.5 hrs at leave-week intensity)
- Problem 9: Meeting Rooms — LeetCode #252 — Easy — Pattern: Intervals **(new — LeetCode Premium in some regions; if inaccessible, treat it as a warm-up for Problem 10 instead)**
  - Hint: sort by start time. If any interval starts before the previous one ends, at least one conflict exists.
  - Complexity: Time O(n log n) | Space O(1)
- Problem 10: Meeting Rooms II — LeetCode #253 — Medium — Pattern: Intervals (min-heap of end times) **(new — also LeetCode Premium in some regions)**
  - Hint: sort by start time; keep a min-heap of end times for rooms currently in use. For each meeting, if the earliest-ending room frees up before this meeting starts, reuse it — otherwise open a new room.
  - Complexity: Time O(n log n) | Space O(n)

### Daily Deliverable
- [ ] Meeting Rooms and Meeting Rooms II solved (or Meeting Rooms II solved standalone if Premium access is an issue), pushed.

---

## Day 27 — Greedy & Intervals Capstone, and Leave Week Wrap-Up

### DSA Block (2.5-3 hrs)
- Problem 11: Partition Labels — LeetCode #763 — Medium — Pattern: Intervals (implicit, via last-occurrence index) **(new)**
  - Hint: find each character's last occurrence index first. Sweep left to right, extending the current partition's boundary to the max last-occurrence seen so far — close the partition when you reach that boundary.
  - Complexity: Time O(n) | Space O(1)

**This closes Greedy & Intervals: 11 problems — the second pattern family that didn't exist anywhere in the original plan.**

### Theory Block (1 hr) — light, wrapping up the leave week
- Topic: Greedy, Reviewed — Why It Works Here and Where It Would Fail
- Write down, for 3 of this week's 11 problems, a one-sentence argument for *why* the greedy choice is safe (i.e., why committing to the locally-best option can't paint you into a corner later). This is the actual interview skill — "I'll be greedy here" isn't a justification on its own, and interviewers will ask you to defend it.

### Career Block (1 hr) — back to normal cadence as the leave week ends
- LinkedIn: Post 7 — "Two patterns I'd never touched before this week: Greedy/Intervals and Kadane's Algorithm" (a good honest milestone post — naming a real gap you closed reads better than pretending you always knew it).
- Networking: catch up on any outreach that slipped during the focused leave-week stretch.

### Daily Deliverable
- [ ] Partition Labels solved — Greedy & Intervals ladder complete at 11 problems.
- [ ] Greedy-safety reflection written. LinkedIn Post 7 published.
- [ ] **Leave week complete.** Prefix Sum/Kadane's and Greedy/Intervals — both entirely new pattern families — are now fully closed, 18 problems total, without pushing the overall 120-day timeline by a single day.

---

## Day 28 (Sunday) — Consolidation, and Binary Search Begins

### Self-Check (15 min)
- [ ] Pick one problem each from Prefix Sum/Kadane's and Greedy/Intervals and re-solve cold, without hints. This is the real test of whether the leave week's intensity actually stuck, versus just being completed.

### DSA Block (2 hrs)

**Concept Card — Binary Search**
- What: repeatedly halving a *sorted* search space by comparing the middle element to your target.
- Why: O(log n) instead of O(n) — for a billion elements, ~30 comparisons instead of a billion.
- Where: any sorted structure, and any problem where the *answer itself* is monotonic — letting you binary search on the answer instead of the input.
- Interview signal: "sorted array," or "minimum/maximum value such that condition holds" (the on-answer variant).
- Prerequisites: arrays ✅. `mid = left + (right - left) / 2` avoids integer overflow.

- Problem 1: Binary Search — LeetCode #704 — Easy
  - Hint: classic template — narrow `[left, right]` based on comparing `nums[mid]` to `target`.
  - Complexity: Time O(log n) | Space O(1)
- Problem 2: First Bad Version — LeetCode #278 — Easy — Pattern: Binary Search on Answer
  - Hint: binary search on a boolean condition, not a value — once `isBadVersion(mid)` is true, the answer is `mid` or earlier.
  - Complexity: Time O(log n) | Space O(1)

### Theory Block (1 hr)
- Topic: Records and Sealed Classes
- `record` gives you an immutable data carrier in one line — constructor, getters, `equals`/`hashCode`/`toString`, no boilerplate. `sealed interface ... permits A, B, C` restricts implementers, letting a `switch` over the sealed type be exhaustive (compiler-guaranteed, no `default` needed) — the compiler knows every possible implementer up front and will refuse to compile a switch that misses one, catching an entire category of bug before the code ever runs.
- Coding exercise: `sealed interface PaymentState permits Pending, Success, Failed`, each a `record`. Write an exhaustive `switch` expression returning a status string per exact type.

### Project Block (1 hr)
- Repository: `java-fundamentals`.
- Task: `ModernJava` package implementing the state machine above.
- Definition of done: compiles on Java 17+, pure `record` syntax, exhaustive switch with no `default`; pushed.

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 28, four weeks in, **57 total DSA problems solved** — 25 through Day 14, plus 12 finishing Sliding Window (Days 15–20), plus 18 from the leave week (7 Prefix Sum/Kadane's + 11 Greedy/Intervals, both patterns now fully closed), plus 2 Binary Search starters today. This checkpoint covers less pattern-breadth than the original plan's Day 28 (which was already mid-Stacks by now) — that's the direct, honest cost of inserting two entire pattern families that the original never covered at all. What you have instead: complete, no-gaps coverage of every pattern touched so far, including two that didn't exist in the original plan anywhere.

### Daily Deliverable
- [ ] Binary Search and First Bad Version solved, pushed to `dsa-java/binary-search/`.
- [ ] `ModernJava` state machine pushed.
- [ ] Weekly ritual and scorecard complete. Leave week fully absorbed — back to standard daily pace from tomorrow.
