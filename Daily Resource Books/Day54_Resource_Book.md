# Day 54 Resource Book — Heaps Begin, and Frequency Patterns

**Series:** SDE-2 Interview Prep Resource Books (Week 8) · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 53](./Day53_Resource_Book.md) · **Next ▶:** [Day 55](./Day55_Resource_Book.md)
**Companion to:** Day 54 of `Week_08_Revised.md`

---

## Recap: what today is

Trees closed yesterday. Heaps opens today, completely fresh — no dependency on anything from this week's Tree work, the same clean pattern-boundary shape Greedy & Intervals used back in Week 4 (opened and, in that case, also closed within the same week).

**One thing today is explicitly *not*:** a first exposure to `PriorityQueue`. Day 17 named it, alongside `Comparable`/`Comparator`, and stated its O(log n) insert/poll and O(1) peek as facts. Day 26 actually used one, for Meeting Rooms II's min-heap of end times. What neither of those moments did was explain **why** those complexity numbers hold — that's genuinely new today, and it's the first thing this book builds, before touching a single LeetCode problem.

**No extra practice today** — Heaps is opening fresh, and every pattern that has opened fresh so far in this series (Sliding Window Day 14, Prefix Sum & Kadane's Day 21, Linked Lists Day 34, Stacks Day 39, Monotonic Stack Day 40, Trees Day 46) has correctly received zero extras on its own opening day, with reinforcement landing on a later day once there's something to reinforce. Today follows that same precedent.

**No Theory or Project block today either** — this matches `Week_08_Revised.md`'s own structure exactly, not an omission; the day is DSA-and-mechanism-heavy on purpose, mirroring how Day 41 (Week 6) also ran without a Theory/Project block when Monotonic Stack needed the freed time for depth instead.

## Learning Objectives

By the end of today, without notes:

1. Explain how a binary heap is stored in a plain array — the index math for parent/child, with no pointers involved — and why that array is always a *complete* binary tree by construction.
2. Derive, not just state, why heap insert and remove are O(log n) — from the tree-height argument, the same style of proof used for every other O(log n) claim this series has made.
3. Explain precisely why a heap's O(log n) is unconditional, where a bare BST's (Day 50) was not.
4. Solve Kth Largest Element in a Stream and Last Stone Weight, and explain, for the first one, why a min-heap — not the seemingly more intuitive max-heap — is the correct structure.

## Concept Dependency Map for Today

```
Day 17: PriorityQueue named (Comparable/Comparator), O(log n)/O(1) stated as facts
Day 26: PriorityQueue USED — Meeting Rooms II, min-heap of end times
Day 50: BST — O(log n) is CONDITIONAL on balance, not guaranteed by the structure alone
        │
        ▼
NEW: Heaps, the actual mechanism — array-backed complete binary tree,
     sift-up (insert), sift-down (remove-root), and WHY this is
     unconditionally O(log n) where a bare BST wasn't
        │
        ├──▶ Problem 1: Kth Largest Element in a Stream (LC 703)
        │     needs: fixed-size min-heap, maintained ACROSS calls (a design/class problem)
        │
        └──▶ Problem 2: Last Stone Weight (LC 1046)
              needs: max-heap, repeatedly pop-two-combine-one
```

---

# Part 1 — Heaps: The Actual Mechanism

## Concept Card — Heaps / Priority Queue

**What it is:** a binary tree stored in a plain array, with one structural guarantee and one ordering guarantee.

**Structural guarantee — it's always a *complete* binary tree:** every level is completely filled, except possibly the last, which fills strictly left to right with no gaps. This is what makes array storage work at all: for a node living at array index `i`, its children live at indices `2i+1` and `2i+2`, and its parent lives at index `(i-1)/2` (integer division) — pure index arithmetic, no `left`/`right` pointers anywhere.

**Ordering guarantee (min-heap):** every parent's value is ≤ both of its children's values. This says nothing about how siblings compare to each other — only parent-to-child. One direct consequence: **the minimum element in the entire structure is always at index 0**, the root — which is exactly why `peek()` is O(1).

## Why this is *always* O(log n) — the argument, not just the claim

Day 50's Concept Card flagged a precise caveat about BSTs: the ordering rule alone doesn't guarantee balance, so a bare BST's O(log n) claim depends on an assumption that can fail (a skewed insertion order degrades it to O(n)).

**A heap has no equivalent failure mode**, and here's precisely why: the *complete binary tree* structural guarantee above isn't something a heap merely tends toward — it's enforced on every single insert, by construction. A new element always goes into the **next available array slot** (`array[size]`, then `size++`) — there is no insertion order that produces a gap or an uneven shape, because the array-slot-by-slot filling rule leaves no room for one to occur. And a complete binary tree with `n` nodes has height **exactly `⌊log₂ n⌋`** — not "usually," not "on average" — because each level can hold at most double the previous level's capacity, so reaching `n` nodes forces the height to that precise bound. **Height bounded by log n, unconditionally, is what makes every heap operation that walks root-to-leaf (or leaf-to-root) provably O(log n) — no balancing algorithm required, because there's no imbalance possible in the first place.**

## Insert — "sift up"

Add the new element at the next available array slot (the end). Then repeatedly compare it to its parent: if it violates the heap property (smaller than its parent, for a min-heap), swap them. Repeat until either the property holds or the element reaches the root.

**Worked trace — min-heap `[3, 5, 8, 10, 12]`, inserting `4`:**
- Place `4` at the end: `[3, 5, 8, 10, 12, 4]` (index 5).
- Parent of index 5 is index `(5-1)/2 = 2`, value `8`. Is `4 < 8`? Yes — violates min-heap. Swap: `[3, 5, 4, 10, 12, 8]` (the new `4` is now at index 2).
- Parent of index 2 is index `(2-1)/2 = 0`, value `3`. Is `4 < 3`? No — property holds. **Stop.**
- Final: `[3, 5, 4, 10, 12, 8]`. Check: index 0 (`3`) ≤ its children (`5`, `4`) ✓; index 1 (`5`) ≤ its child (`10`) ✓ — wait, index 1's children are at 3 and 4 (`10`, `12`), both ≥ `5` ✓; index 2 (`4`) ≤ its child (`8`) ✓. Valid heap.

Each comparison moves one level up the tree; the tree has O(log n) levels — so sift-up does **at most O(log n) swaps**.

## Remove-root — "sift down"

The root is always the answer (min, for a min-heap) — that's what gets returned. To restore the array's completeness after removing it, move the **last** element in the array into the now-empty root position (this keeps the array gap-free, which is required for the index math to keep working), then repeatedly compare it to its children: if it violates the heap property, swap with the **smaller** of its two children (for a min-heap). Repeat until the property holds or a leaf is reached.

**Worked trace — continuing from `[3, 5, 4, 10, 12, 8]`, calling poll():**
- Remove root `3` (this is the returned value). Move the last element (`8`) into the root: `[8, 5, 4, 10, 12]` (array shrinks by one).
- At index 0 (`8`): children at indices 1, 2 are `5` and `4`. Smaller child is `4` (index 2). Is `8 > 4`? Yes — violates min-heap. Swap with the smaller child: `[4, 5, 8, 10, 12]`.
- At index 2 (`8`): children would be at indices 5, 6 — both out of bounds (array length 5). No children — **stop.**
- Final: `[4, 5, 8, 10, 12]`. Check: index 0 (`4`) ≤ children (`5`, `8`) ✓; index 1 (`5`) ≤ its child (`10`) ✓. Valid heap, and `4` is indeed the minimum of the remaining elements `{5, 8, 10, 12, 4}`.

Same argument as sift-up, mirrored: each comparison moves one level down a tree with O(log n) levels, so sift-down is **at most O(log n) swaps**.

| Operation | Time | Why |
|---|---|---|
| `peek()` | O(1) | The minimum (or maximum) is always at index 0 by the ordering guarantee. |
| `offer()` / insert | O(log n) | Sift-up traverses at most the tree's height. |
| `poll()` / remove-root | O(log n) | Sift-down traverses at most the tree's height. |

## Java's `PriorityQueue<T>`

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();               // natural ordering = min-heap by default
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());   // reversed comparator = max-heap

minHeap.offer(5);     // insert
int smallest = minHeap.poll();   // remove and return the root
int peekAtMin = minHeap.peek();  // look at the root without removing it
```

`Collections.reverseOrder()` is a static utility that returns a `Comparator` reversing natural ordering — it's not new syntax (no anonymous class needed for this specific case), just a ready-made comparator. A **custom** ordering (comparing by something other than a value's own natural order — distance, frequency, and so on) needs a `Comparator` written by hand; that resurfaces properly on Day 55, reusing the exact anonymous-inner-class technique from Day 35's custom `Comparator` for Merge k Sorted Lists.

**Prerequisite confirmed:** `Comparable`/`Comparator`, needed for any custom ordering, was covered in full on Day 17 (Week 3) — nothing new to build there before today's problems.

---

# Part 2 — First Two Heap Problems

## Problem 1: Kth Largest Element in a Stream (LeetCode 703, Easy) — Pattern: Fixed-Size Min-Heap

**Statement:** Design a class that, given an integer `k` and an initial array of numbers, supports `add(val)` — insert a new value into the stream, and return the current kth largest value in the stream so far.

### The counterintuitive part, addressed directly

The instinct many people reach for first: "I want the kth **largest** — shouldn't that mean a **max**-heap?" It's a reasonable instinct, and it's backward. Walk through why.

### Approach — a min-heap, capped at size k

```java
class KthLargest {
    private final int k;
    private final PriorityQueue<Integer> minHeap;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        this.minHeap = new PriorityQueue<>();
        for (int num : nums) {
            add(num);
        }
    }

    public int add(int val) {
        minHeap.offer(val);
        if (minHeap.size() > k) {
            minHeap.poll();   // evict the smallest — it can't be the kth largest anymore
        }
        return minHeap.peek();
    }
}
```

**The reframe:** don't track the whole stream. Track only the **k largest values seen so far** — a fixed-size window of "the current top k." Once that window has exactly k elements, its **smallest** member is, by definition, exactly the kth largest overall — there are precisely `k - 1` elements in the window larger than it, and everything *not* in the window is smaller than everything in it. A min-heap is the right structure specifically because the question this problem keeps asking, over and over, is "what's the smallest thing currently in my top-k set?" — and that's exactly what a min-heap's O(1) peek answers, with O(log k) maintenance cost each time the set changes. A max-heap of the *entire* stream would answer a completely different question (what's the single largest value ever seen) and would need O(n) work or repeated polls to dig down to the kth position — it doesn't naturally maintain a bounded top-k window at all.

**Complexity: each `add()` call is O(log k)** — the heap never holds more than `k + 1` elements before an eviction. **Space O(k).** The constructor's initial loop over `nums` costs `O(n log k)` for `n` initial values.

**Edge cases:** `add()` called fewer than `k` times total, before the heap has ever reached size `k` (the heap just keeps growing without eviction — `peek()` still returns the smallest value seen so far, which is a well-defined, if not-yet-"final," answer to "the kth largest of what's been seen"); duplicate values (handled correctly with no special case — the heap doesn't care about uniqueness, only relative order).

> ⚠️ **Common Mistake:** reaching for a max-heap because "kth largest" sounds like it should pair with "largest." The min-heap-of-size-k pattern is the correct structure precisely because the question is about the boundary of a bounded top-k set, not about the single overall maximum — this exact confusion is worth being ready to preempt out loud.

---

## Problem 2: Last Stone Weight (LeetCode 1046, Easy) — Pattern: Max-Heap

**Statement:** Given an array of stone weights, repeatedly take the two heaviest stones and smash them together: if they're equal, both are destroyed; otherwise, the lighter is destroyed and the heavier becomes `heavier - lighter`. Return the weight of the last stone remaining, or `0` if none remain.

### Approach 1 — Brute force: re-sort every round

```java
public int lastStoneWeightBruteForce(int[] stones) {
    List<Integer> remaining = new ArrayList<>();
    for (int s : stones) remaining.add(s);

    while (remaining.size() > 1) {
        Collections.sort(remaining, Collections.reverseOrder());
        int first = remaining.remove(0);
        int second = remaining.remove(0);
        if (first != second) {
            remaining.add(first - second);
        }
    }
    return remaining.isEmpty() ? 0 : remaining.get(0);
}
```

Re-sorting descending every round and taking the top two is correct, but wasteful — **O(n) rounds × O(n log n) sort each = O(n² log n)**, Space O(n).

### Approach 2 — Optimized: max-heap

```java
public int lastStoneWeight(int[] stones) {
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
    for (int stone : stones) {
        maxHeap.offer(stone);
    }

    while (maxHeap.size() > 1) {
        int first = maxHeap.poll();
        int second = maxHeap.poll();
        if (first != second) {
            maxHeap.offer(first - second);
        }
    }

    return maxHeap.isEmpty() ? 0 : maxHeap.peek();
}
```

The heap only ever needs to answer "what are the two current largest values" — exactly a max-heap's specialty, at O(log n) per pop/push instead of an O(n log n) full re-sort every round.

**Complexity: each round does 2 polls + at most 1 offer, each O(log n); at most n − 1 rounds** (every round removes at least one stone net — two removed, at most one added back). **Time O(n log n), Space O(n).**

**Edge cases:** a single stone (loop never executes, that stone is returned directly); two equal-weight stones (both destroyed, heap empties, return `0`); the process naturally terminates with either 0 or 1 stones remaining, never gets "stuck," since every round strictly reduces the count.

> 💡 **Interview Insight:** this problem is a clean, low-stakes way to demonstrate the "simulate directly with the right structure" instinct — the brute-force version isn't wrong, it's just paying an O(log n) sorting cost repeatedly for information a heap maintains incrementally. Naming that trade-off before writing the heap version is worth doing explicitly.

---

# Section — Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** identify 3 Target Tier B companies — same tracker as Day 50's, building toward this week's application and outreach targets.

---

# Day 54 — Interview Questions

**1. How is a binary heap stored in memory, and what index formulas connect a node to its children and parent?**

*Answer:* As a plain array, with no pointers. For a node at index `i`: children live at `2i+1` and `2i+2`; the parent lives at `(i-1)/2` (integer division).

---

**2. What are the two guarantees a heap makes, and which one is structural versus which is about ordering?**

*Answer:* Structural: it's always a complete binary tree — every level full except possibly the last, filled left to right with no gaps. Ordering: every parent's value is ≤ (min-heap) or ≥ (max-heap) both of its children's values.

---

**3. Why is a heap's O(log n) guarantee unconditional, where a bare BST's (Day 50) was not?**

*Answer:* A heap's complete-binary-tree structure is enforced on every insert by construction — a new element always goes into the next array slot, with no insertion order able to produce an uneven shape. A complete tree with n nodes has height exactly ⌊log₂n⌋, always — so operations bounded by tree height are provably O(log n), with no balancing algorithm needed because no imbalance is possible in the first place. A bare BST's shape depends entirely on insertion order, with no such enforcement.

---

**4. Walk through why sift-up (insert) is O(log n).**

*Answer:* The new element starts at the last array slot and is compared to its parent; if it violates the heap property, it swaps with the parent and the comparison repeats one level up. Each step moves exactly one level toward the root, and the tree has at most O(log n) levels, so at most O(log n) swaps occur.

---

**5. During sift-down (remove-root), why does the LAST array element get moved to the root, rather than promoting one of the root's children directly?**

*Answer:* Moving the last element preserves the array's completeness (no gaps), which the index-math formulas for parent/child depend on. Promoting a child directly would leave a hole that breaks that structural guarantee.

---

**6. During sift-down, why swap with the SMALLER of the two children (for a min-heap), not either child?**

*Answer:* Swapping with the smaller child guarantees the heap property is satisfied between the new parent and the child that wasn't swapped (since it's ≥ the smaller one, and the smaller one is now placed above it in the correct position) — swapping with the larger child could leave the smaller child violating the property relative to its new parent.

---

**7. In Kth Largest Element in a Stream, why is a min-heap correct, when the question asks about the LARGEST elements?**

*Answer:* The heap tracks only a bounded window of the k largest values seen so far, not the whole stream. Once that window has k elements, its own smallest member is, by definition, the kth largest overall — exactly what a min-heap's O(1) peek answers directly, with O(log k) maintenance per update.

---

**8. Why can't a max-heap of the entire stream answer "what's the kth largest" as efficiently?**

*Answer:* A max-heap's peek gives the single overall maximum, not the kth-ranked value — reaching the kth position would require repeated polls (destroying the heap's other contents) or O(n) work, rather than maintaining a bounded top-k window incrementally the way the min-heap-of-size-k approach does.

---

**9. In Last Stone Weight, why does the brute-force re-sorting approach cost O(n² log n) overall?**

*Answer:* Each of up to n rounds re-sorts the remaining stones at O(n log n) cost, to extract just the top two values — repeating a full sort every round when only the two largest values actually matter each time.

---

**10. Why does Last Stone Weight's simulation always terminate, never loop indefinitely?**

*Answer:* Every round removes exactly two stones and adds back at most one, so the total stone count strictly decreases by at least one every round — the process necessarily reaches 0 or 1 remaining stones in a bounded number of rounds.

---

## Daily Deliverable Check

- [ ] Kth Largest Element in a Stream and Last Stone Weight solved, pushed to `dsa-java/heaps/`.
- [ ] Heap mechanism — array index math, sift-up, sift-down, and the unconditional O(log n) argument — explainable from scratch, including the precise contrast with why a bare BST's O(log n) claim needed a balance caveat and a heap's doesn't.
- [ ] 3 Target Tier B companies identified and logged.

---

## What Tomorrow Assumes You Already Know Cold

Day 55 assumes today's array-backed heap mechanism and the min-heap-of-size-k pattern are both solid — Kth Largest Element in an Array reuses the identical shape from today's Kth Largest Element in a Stream, just without the "across multiple calls" statefulness, and K Closest Points to Origin extends it with a custom `Comparator` (distance, not natural value order) — reusing the exact anonymous-inner-class technique from Day 35, not new syntax.

**Next:** [Day 55 Resource Book](./Day55_Resource_Book.md) — Heaps Continue.
