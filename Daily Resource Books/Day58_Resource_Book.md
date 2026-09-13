# Day 58 — Heaps Capstone: Two-Heap and Merge Patterns

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 57 Resource Book](Day57_Resource_Book.md)
**Next ▶:** [Day 59 Resource Book](Day59_Resource_Book.md)
**Companion to:** Day 58 of `Week_09_Revised.md`

---

## Recap

Yesterday added greedy-scheduling to the heap's repertoire: always act on the single best candidate available right now. Today closes Heaps with two more roles that don't reduce to yesterday's greedy-placement shape at all: a heap pair that maintains a **running statistic** over an unbounded stream (rather than committing to output immediately), and a heap that **coordinates a merge** across several already-sorted sources at once. Both problems build directly on Day 54's core heap mechanism and Day 5's `PriorityQueue`-with-comparator usage (first named Day 17, first used Day 26's Meeting Rooms II) — no new heap machinery, only new *shapes* to apply it to.

## Learning Objectives

By the end of today, without notes:

1. Maintain two heaps in a balanced invariant that supports O(1) median lookup after every insertion, and prove the invariant is actually maintained (not just plausible).
2. Merge k sorted linked lists using a heap, and state the divide-and-conquer alternative that achieves the identical complexity through a completely different mechanism.
3. Give the full, closed accounting of every distinct role a heap has played across Weeks 8–9, and explain why Heaps needs no further extra practice at this point.

## Concept Dependency Map

```
Heap mechanism (Day 54) + PriorityQueue/Comparator (Day 17, Day 26)
        │
        ├─ NEW: Two-heap balance invariant
        │    max-heap (lower half) + min-heap (upper half),
        │    sizes differ by ≤ 1 → O(1) median from the tops alone
        │
        └─ NEW: Heap as k-way merge coordinator
             one entry per source list in the heap;
             pop smallest, advance that source, repeat
             🔗 Day 22/35 (Week 5): Merge Two Sorted Lists — the 2-list case
        │
        ▼
Heaps CLOSE at 10/10 required (13 distinct combined with Week 8's 3 extra)
        │
        ▼
🔗 Day 59: Tries begin — a genuinely new tree shape, not a heap variant
```

---

## Part 1 — Find Median from Data Stream

### Problem 3: Find Median from Data Stream (LeetCode 295, Hard) — Pattern: Two Heaps

**Statement:** Design a data structure supporting `addNum(int)` and `findMedian()`, where `findMedian()` returns the median of every number added so far, at any point, repeatedly.

### Why a single sorted structure isn't the right tool here

The word "stream" is the signal: numbers keep arriving, and the median is needed *repeatedly*, not once at the end. A structure re-sorted on every query, or an array kept sorted via insertion (shifting elements to maintain order), both pay for correctness on every single operation — exactly the kind of repeated, incremental "keep this ordered as it grows" requirement two heaps are built for.

### Approach 1 — Brute force: insert into a sorted list

```java
// O(n) insert (binary search for position is O(log n), but shifting elements is O(n));
// O(1) median lookup once sorted.
```

Simple, correct, and fine for infrequent insertions relative to queries — but a stream implies insertions and queries interleaved constantly, and `O(n)` per insert is the cost being avoided.

### Approach 2 — Optimized: two heaps, size-balanced

```java
class MedianFinder {
    private final PriorityQueue<Integer> lower = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
    private final PriorityQueue<Integer> upper = new PriorityQueue<>(); // min-heap

    public void addNum(int num) {
        lower.offer(num);
        upper.offer(lower.poll());       // move lower's max into upper
        if (upper.size() > lower.size()) {
            lower.offer(upper.poll());   // move upper's min back, restoring balance
        }
    }

    public double findMedian() {
        if (lower.size() > upper.size()) {
            return lower.peek();
        }
        return (lower.peek() + upper.peek()) / 2.0;
    }
}
```

**The invariant, stated precisely:** `lower` (a max-heap) holds the smaller half of all numbers seen so far; `upper` (a min-heap) holds the larger half. `lower.size()` is always either equal to `upper.size()`, or exactly one greater — never the reverse, and never off by more than one. Every number in `lower` is `≤` every number in `upper`.

**Why the "push to lower, then always shuffle across, then rebalance if needed" sequence maintains this — not just asserted, traced:**

`addNum(5)`: `lower.push(5)` → `lower=[5]`. `upper.push(lower.pop()=5)` → `lower=[]`, `upper=[5]`. Check: `upper.size()(1) > lower.size()(0)` → true → `lower.push(upper.pop()=5)` → `lower=[5]`, `upper=[]`. **Median = 5** (lower bigger). Correct: median of `[5]` is 5.

`addNum(15)`: `lower.push(15)` → `lower=[15,5]`, top `15`. `upper.push(lower.pop()=15)` → `lower=[5]`, `upper=[15]`. Check: `1 > 1` false → done. **Median = (5+15)/2 = 10**. Correct: median of `[5,15]` is 10.

`addNum(1)`: `lower.push(1)` → `lower={5,1}`, top `5`. `upper.push(lower.pop()=5)` → `lower={1}`, `upper={5,15}`, top `5`. Check: `2 > 1` → true → `lower.push(upper.pop()=5)` → `lower={5,1}`, `upper={15}`. **Median = lower.peek() = 5** (lower bigger, size 2 vs 1). Correct: sorted `[1,5,15]`, median 5.

`addNum(3)`: `lower.push(3)` → `lower={5,1,3}`, top `5`. `upper.push(lower.pop()=5)` → `lower={1,3}`, top `3`; `upper={5,15}`, top `5`. Check: `2 > 2` false → done. **Median = (3+5)/2 = 4**. Correct: sorted `[1,3,5,15]`, median = avg(3,5) = 4.

Every step matches the true median of the numbers seen so far — the "push to lower unconditionally, shuffle the max across, rebalance only if upper overtook lower" sequence is what keeps both the size invariant *and* the ordering invariant (everything in `lower` ≤ everything in `upper`) true after every single call, not just eventually.

**Why push to `lower` first, unconditionally, rather than deciding which heap up front?** Deciding "does this belong in the lower or upper half" before insertion requires comparing against both heaps' current boundary values — extra branching for no benefit. Pushing to `lower` first and then always relaying its new max into `upper` achieves the same correct placement through a single unconditional operation plus one possible rebalancing step; the two-step shuffle is simpler to get right than branch-then-insert, and both cost the same `O(log n)`.

> ⚠️ **Common Mistake:** implementing the "which heap does this go into" logic with an if/else comparing `num` against `lower.peek()`, forgetting the empty-heap edge case (comparing against a `peek()` on an empty `PriorityQueue` throws). The push-then-shuffle version sidesteps this entirely — it never needs to peek before the first element exists.

### Complexity

**`addNum`: O(log n)** — two heap operations, each `O(log n)`, occasionally three if rebalancing fires.
**`findMedian`: O(1)** — both tops are already known.
**Space: O(n)** — every number ever added is stored across the two heaps.

### Edge cases

- First call to `addNum` — no rebalancing needed, single heap gets it, `findMedian` still correct via the size check.
- All numbers identical — heaps still balance correctly on size alone; ordering invariant trivially holds.
- Even vs. odd total count — the `lower.size() > upper.size()` branch is exactly what selects single-value vs. averaged-pair, correctly, every time.

### Interview framing

**Say before coding:** "I'll split the stream across two heaps — a max-heap for the smaller half, a min-heap for the larger half — kept balanced so their tops alone give me the median in O(1)."
**Likely follow-up:** "What if the stream is heavily skewed — say, mostly increasing?" The size-balance invariant is maintained purely by count, independent of value distribution, so skew doesn't break correctness; it's already handled.
**Justify unprompted:** state why `O(1)` reads with `O(log n)` writes is the right trade-off for a *stream* (writes happen once per number; reads can happen arbitrarily often) — the alternative (sorted array) inverts this trade-off in exactly the wrong direction for this access pattern.

---

## Part 2 — Merge k Sorted Lists

### Problem 4: Merge k Sorted Lists (LeetCode 23, Hard) — Pattern: PriorityQueue with Custom Comparator

**Statement:** Given `k` sorted linked lists, merge them into one sorted linked list.

### 🔗 Direct lineage

This is the direct generalization of **Merge Two Sorted Lists** (LC 21, Week 5 Day 35 — dummy-head, two-pointer walk) from exactly 2 lists to k. The 2-list mechanism (always take the smaller of the two current heads, advance that pointer) doesn't change — it's *which structure tracks "the smallest of the current heads"* that has to change once there are more than two candidates to compare at once.

### Approach 1 — Brute force: collect and sort

```java
// Walk every list, collect every value into one array, sort it, rebuild a linked list.
// Time: O(N log N) where N = total nodes across all lists. Space: O(N).
```

Correct, but throws away the fact that each individual list already arrives sorted — sorting from scratch re-does work the input already did for you.

### Approach 2 — Middle ground: merge lists two at a time, sequentially

```java
// mergeTwoLists(list1, list2), then merge that result with list3, then list4, ...
// Time: O(N * k) — each of the k-1 sequential merges can touch up to N nodes.
```

Correct and only uses the already-known 2-list merge — but the early merged results keep getting re-walked as more lists fold in, which is exactly what the next two approaches avoid.

### Approach 3 — Optimized: min-heap of current heads

```java
public static ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> minHeap = new PriorityQueue<>(
        (a, b) -> Integer.compare(a.val, b.val)   // NOT a.val - b.val — see note below
    );
    for (ListNode node : lists) {
        if (node != null) minHeap.offer(node);
    }

    ListNode dummy = new ListNode(-1);
    ListNode tail = dummy;
    while (!minHeap.isEmpty()) {
        ListNode smallest = minHeap.poll();
        tail.next = smallest;
        tail = tail.next;
        if (smallest.next != null) {
            minHeap.offer(smallest.next);
        }
    }
    return dummy.next;
}
```

**Mechanism:** the heap holds exactly one node per *still-active* list — its current head. Popping always yields the smallest value among all current heads; after appending it to the output, that same list's *next* node (if any) is offered in its place. The heap never holds more than `k` nodes at once, no matter how large `N` is.

**⚠️ Common Mistake:** writing the comparator as `(a, b) -> a.val - b.val`. This is a classic subtraction-overflow trap — if `a.val` and `b.val` sit near opposite ends of `int`'s range, the subtraction itself can overflow and silently produce a wrong-sign result, corrupting the heap's ordering. `Integer.compare(a.val, b.val)` never has this problem, since it compares directly without an intermediate subtraction.

### Approach 4 — Equally optimal, different mechanism: divide and conquer

```java
// Pair up the k lists, merge each pair (using the Day 35 2-list merge),
// then repeat on the resulting k/2 lists, halving each round until one list remains.
// Time: O(N log k) — same bound as the heap, log k merge "rounds," O(N) work per round.
```

**Worth stating out loud, unprompted:** this achieves the *identical* `O(N log k)` bound as the heap approach through a completely different mechanism — no heap at all, just repeated pairwise merging using the 2-list technique already known. The heap approach processes one node at a time across all lists simultaneously; divide-and-conquer processes whole lists in merge rounds. Either is a fully correct, optimal answer; naming both and picking one deliberately (rather than only knowing one) is what distinguishes a strong answer here.

### Complexity

**Heap approach — Time: O(N log k)** where N = total nodes across all lists, k = number of lists — every node is pushed and popped exactly once, each operation `O(log k)` since the heap never exceeds size k. **Space: O(k)** for the heap (plus O(N) for the output list itself, which isn't extra auxiliary space in the usual sense since it's the required result).
**Divide and conquer — Time: O(N log k), Space: O(log k)** (recursion depth) or O(1) if done iteratively.

### Edge cases

- `lists` contains empty (`null`) entries mixed with non-empty ones — the initial population loop already skips `null` heads; nothing further needed.
- All lists empty → heap starts empty, loop never runs, `dummy.next` is correctly `null`.
- `k = 1` → degenerates cleanly to a pass-through; no special-casing required.

### Interview framing

**Say before coding:** "I'll keep a min-heap holding the current head of every list; repeatedly pull the global minimum, append it, and push that same list's next node back in."
**Likely follow-up:** "Can you do it without extra space for a heap?" → divide-and-conquer pairwise merging, same complexity, no heap.
**Common mistake to flag unprompted:** the comparator subtraction-overflow trap — mentioning it before being asked signals real experience with this exact gotcha, not just memorized code.

---

## Heaps Close: 10/10 Required — Why No Extra Practice Is Added

Week 8 opened Heaps at 6/10 required + 3 extra (Minimum Cost to Connect Sticks, Kth Smallest in a Sorted Matrix, Sort Characters By Frequency). Today's two problems complete the required ladder at 10/10, for **13 distinct heap problems** solved across Weeks 8–9.

The curriculum map already tracked four distinct roles a heap had played through Day 56: min-heap for "kth largest," max-heap for "keep the k best," min-heap for "minimize combination cost," and heap-as-candidate-generator. Today adds two more that don't reduce to any of those four: **max-heap-driven greedy scheduling** (yesterday, Reorganize String / Task Scheduler) and **two structurally distinct multi-source coordination roles** (today — balance-invariant statistics tracking, and k-way merge coordination). That's six genuinely different roles a single data structure plays, spanning Easy through Hard.

No extra practice is added today. Reasoning, explicitly: this is exactly the "biggest single fix from the audit" the plan's own header called out (6 → 10 required problems) — the gap this series exists to catch was already closed by the plan's own revision, not something left for this book to patch. Padding a pattern that already demonstrates six distinct roles across 13 problems would add repetition, not new pattern-recognition reps. (Contrast this with Tries and Backtracking, both opening this week — see Day 61's closing note and the curriculum map's Week 9 section for why those get the *opposite* treatment during their opening arcs.)

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.
**Networking:** no specific outreach today, per the plan.

---

## Day 58 — Interview Questions

**Q1. State the two-heap invariant for Find Median from Data Stream precisely.** `lower` (max-heap) holds the smaller half of all numbers seen; `upper` (min-heap) holds the larger half; `lower.size()` equals `upper.size()` or is exactly one more; every value in `lower` is ≤ every value in `upper`.

**Q2. Why push to `lower` unconditionally rather than deciding up front which heap a new number belongs in?** It avoids comparing against a heap that might currently be empty (which would throw on `peek()`), and achieves correct placement through one unconditional push plus at most one rebalancing step — simpler to get right than branch-then-insert, same `O(log n)` cost either way.

**Q3. Given the invariant, derive `findMedian()`'s two cases.** If `lower` has one more element than `upper`, the median is `lower`'s top alone (the middle element in an odd-length stream). If they're equal in size, the median is the average of both tops (the two middle elements in an even-length stream).

**Q4. Why is Merge k Sorted Lists' comparator written with `Integer.compare(a.val, b.val)` instead of `a.val - b.val`?** Subtracting two `int`s that sit near opposite ends of the representable range can overflow and silently flip the sign, corrupting heap ordering; `Integer.compare` never has this failure mode since it doesn't compute an intermediate difference.

**Q5. Name two genuinely different algorithms that both solve Merge k Sorted Lists in O(N log k), and explain the mechanism difference.** Min-heap of current list-heads (processes one node at a time across all lists simultaneously) and divide-and-conquer pairwise merging (processes whole lists per round, halving the list count each round) — same asymptotic bound, structurally different approaches.

**Q6. Why does the heap in Merge k Sorted Lists never exceed size k, regardless of how large N is?** It holds exactly one node per still-active input list at any time — popping a node and immediately offering its successor keeps the count bounded by the number of lists, never by total node count.

**Q7. Across Weeks 8–9, name the distinct roles a heap has played, and why that variety matters for interview prep.** Min-heap for kth-largest queries, max-heap for keeping the k best, min-heap for minimizing combination cost, heap as a candidate generator, max-heap for greedy scheduling, and two-heap balance-invariant statistics plus k-way merge coordination — six genuinely different applications of one structure, which is why the pattern needs no further padding: the goal was recognizing *when* to reach for a heap and *how* to shape it, not memorizing 13 individual solutions.

---

## Daily Deliverable Check

- [ ] Find Median from Data Stream solved — invariant traced through at least 4 sequential `addNum` calls by hand, matching this book's worked trace.
- [ ] Merge k Sorted Lists solved via the heap approach, **and** can state the divide-and-conquer alternative's complexity without looking it up.
- [ ] Both pushed to `dsa-java/heaps/`. **Heaps ladder complete at 10/10 required, 13/13 distinct.**
- [ ] Can state, from memory, why no extra Heap practice was added today.

---

## What Tomorrow Assumes You Already Know Cold

Day 59 leaves heaps behind entirely and opens Tries — a new tree shape, not a heap variant, so nothing about sift-up/sift-down or comparator ordering carries forward directly. What *does* carry forward: recursion (Day 8) as a load-bearing tool, and the general habit — now exercised six distinct ways across two weeks — of asking "what structure does this access pattern actually need" before reaching for code. Tomorrow applies that same habit to a brand-new structure.

**Next:** [Day 59 Resource Book](Day59_Resource_Book.md) — Tries Begin, and the CAP Theorem.
