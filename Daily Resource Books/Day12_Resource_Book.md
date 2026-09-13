# Day 12 — Two Pointers Near-Capstone, and Collections Internals

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 11 Resource Book](Day11_Resource_Book.md)
**Next ▶:** [Day 13 Resource Book](Day13_Resource_Book.md)
**Companion to:** Day 12 of `Week_02_Revised.md`

---

## Overlap Notice

| LC # | Problem | Status |
|---|---|---|
| 11 | Container With Most Water | Already solved — Week 1, Day 7 (Extra Practice). **Recap only.** |
| 75 | Sort Colors | Genuinely new. **Full depth below.** |

Same partial-overlap shape as Day 10. Sort Colors introduces the **Dutch National Flag** three-way partition — a genuinely new sub-variant of Two Pointers that doesn't cleanly reduce to opposite-ends or fast-slow. **No extra practice problem is added for it today**, deliberately: it's a narrow, self-contained technique at this point in the course (its next real appearance is alongside Quicksort/Quickselect's partitioning logic, still ahead in this plan), and forcing a second rep now would either repeat Sort Colors' exact shape under a different name or reach for a problem that doesn't actually use three-way partitioning. Full depth on the one canonical example is the right call here, flagged per the same judgment the prompt asks for.

---

## Recap

Yesterday's Boats to Save Most People showed a new *decision rule* within the opposite-ends family. Today's Sort Colors goes further — a new *pointer arrangement* entirely, three pointers instead of two, that doesn't fit cleanly into either of Week 1's two named categories. Recognizing when a problem needs a genuinely new mental model, instead of forcing it into an existing box, is itself the lesson.

---

## Learning Objectives

1. Recap Container With Most Water's greedy-pointer argument from memory.
2. State and *prove* the three-region invariant that makes Sort Colors' single-pass Dutch National Flag partition correct.
3. Name the exact off-by-one mistake that breaks Sort Colors for most people who get it wrong, and explain precisely why it breaks.
4. Explain, at the mechanism level, why `ArrayDeque` beats both `ArrayList` and `LinkedList` for stack/queue use — not just cite the conclusion.

---

## Concept Dependency Map

```
Week 1 Day 7: Container With Most Water (LC 11) — opposite-ends, provable greedy
        │
        └──▶ Today: Sort Colors (LC 75) — three pointers,
                     a genuinely new arrangement (not opposite-ends, not fast-slow)

Week 1 Day 3: amortized analysis (ArrayList doubling)
Week 1 Day 4: ArrayDeque introduced (as Stack/Queue backing)
        │
        └──▶ Today: Collections Internals — WHY ArrayList/LinkedList/ArrayDeque
                     perform the way they do, at the memory-layout level
```

---

## Part 1 — Two Pointers: Recap and Sort Colors

### 🔗 Recap: Container With Most Water (LC 11)

**Original coverage:** Week 1, Day 7, Extra Practice. Optimized solution only, for reference.

```java
public static int maxArea(int[] height) {
    int left = 0, right = height.length - 1;
    int best = 0;
    while (left < right) {
        int width = right - left;
        int area = Math.min(height[left], height[right]) * width;
        best = Math.max(best, area);
        if (height[left] < height[right]) {
            left++;      // the shorter line is the bottleneck — only moving it can possibly improve area
        } else {
            right--;
        }
    }
    return best;
}
```

`left = 0`, `right = n-1`; area is capped by the shorter of the two lines, so the taller line can never be the bottleneck — always move the pointer at the **shorter** line inward, since keeping it fixed can only ever be matched or beaten by moving the taller one, never improved. Time O(n), Space O(1). This is the same "provable greedy" flavor as yesterday's Boats problem — worth naming that connection out loud if it comes up.

---

### New Problem: Sort Colors (LC 75)

**Statement:** Given an array containing only `0`, `1`, and `2`, sort it in place, in one pass, using O(1) extra space. (The classic framing: sort red/white/blue — the "Dutch National Flag" problem, after Dijkstra's original formulation.)

**Brute force:** any general-purpose sort (`Arrays.sort`) — O(n log n), doesn't exploit the fact that there are only three distinct values.

### Approach 1 — Intermediate: counting, two passes

```java
public static void sortColorsCounting(int[] nums) {
    int[] count = new int[3];
    for (int num : nums) {
        count[num]++;
    }
    int index = 0;
    for (int color = 0; color < 3; color++) {
        for (int i = 0; i < count[color]; i++) {
            nums[index++] = color;
        }
    }
}
```

Worth naming explicitly as a real, correct, easier-to-derive O(n) solution: count each value's frequency, then overwrite the array with that many 0s, then 1s, then 2s. Genuinely O(n) time, O(1) space. The problem's **one-pass** constraint is specifically what rules this out and forces the three-pointer technique below — say this out loud if you present the counting version first, so it reads as a deliberate stepping stone rather than a miss.

### Approach 2 — Optimized: Dutch National Flag, one pass, three pointers

```java
public static void sortColors(int[] nums) {
    int low = 0, mid = 0, high = nums.length - 1;

    while (mid <= high) {
        if (nums[mid] == 0) {
            swap(nums, low, mid);
            low++;
            mid++;
        } else if (nums[mid] == 1) {
            mid++;
        } else {                    // nums[mid] == 2
            swap(nums, mid, high);
            high--;                 // do NOT advance mid here — see the common mistake below
        }
    }
}

private static void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

Maintain the invariant, at all times during the algorithm: `[0, low)` is all 0s, `[low, mid)` is all 1s, `[mid, high]` is unprocessed/unknown, `(high, n-1]` is all 2s.

**Why it's safe to advance both `low` and `mid` in the `0` case (proof, not assertion):** at any point, `low <= mid` always holds (they only ever move together in this branch, and `mid` never falls behind `low`). So `nums[low]`, before the swap, is either itself unprocessed (if `low == mid`) or is a `1` (if `low < mid`, since `[low, mid)` is defined by the invariant to hold only 1s). Either way, swapping it into position `mid` is safe: if it was a `1`, it now correctly sits inside the (about-to-grow) `[low, mid)` region once `mid` advances too; if `low == mid`, the swap is a same-index no-op and the reasoning collapses trivially. The `0` that was at `mid` is now at `low`, correctly extending the `[0, low)` region by exactly one. This is why `low` and `mid` can safely move together here — the value being displaced from `low` is never anything the invariant doesn't already account for.

**⚠️ Common Mistake — the single most common bug in this algorithm.** Advancing `mid` after the `2`-swap. **Don't.** The value swapped *into* position `mid` from `high` is unexamined — it could be a `0`, a `1`, or another `2` — so it must be re-checked on the *next* iteration, not skipped. Advancing `mid` here silently skips checking that value, which can leave a `0` or `2` stranded in the wrong region. This is worth internalizing as a specific, nameable trap, not just "remember to be careful" — the `0` and `1` branches both advance `mid` because the value landing there (or already there) is *known*; the `2` branch's incoming value is *unknown*, which is exactly why it's the one branch that must not advance `mid`.

**Termination and full coverage:** the loop ends when `mid > high`. At that point `[0, low)` = 0s, `[low, mid)` = 1s, and — because `mid` and `high` have met (`mid == high + 1`) — `(high, n-1]` = 2s covers everything from `mid` onward with no gap. Every index has been classified, none skipped, none double-counted.

**Complexity:** Time O(n) — despite three pointers, each array index is touched a bounded number of times overall (visited once as `mid`'s current position, plus at most once more as a swap target from a `0`- or `2`-swap), giving true linear time, not the "looks like nested pointer juggling, must be worse than O(n)" impression it can give at first glance. Space O(1) — in-place swaps only.

**Worked trace:** `nums = [2,0,2,1,1,0]`. `low=0, mid=0, high=5`.

| low | mid | high | nums[mid] | action | array after |
|---|---|---|---|---|---|
| 0 | 0 | 5 | 2 | swap(mid,high); high-- | [0,0,2,1,1,2] |
| 0 | 0 | 4 | 0 | swap(low,mid); low++; mid++ | [0,0,2,1,1,2] |
| 1 | 1 | 4 | 0 | swap(low,mid) *(no-op, same index)*; low++; mid++ | [0,0,2,1,1,2] |
| 2 | 2 | 4 | 2 | swap(mid,high); high-- | [0,0,1,1,2,2] |
| 2 | 2 | 3 | 1 | mid++ | [0,0,1,1,2,2] |
| 2 | 3 | 3 | 1 | mid++ | [0,0,1,1,2,2] |
| — | 4 | 3 | — | mid > high, stop | **[0,0,1,1,2,2]** ✓ |

**Edge cases:**
- Already sorted: every comparison lands in the branch that just advances `mid` (or a harmless self-swap) — no correctness issue, just fewer effective swaps.
- All one value: `mid` sweeps straight through, one branch fires every time, terminates cleanly.
- Empty or single-element array: loop body never executes or executes trivially — correct without special-casing, provided the initial pointer bounds are set correctly.

**💡 Interview Insight:** if asked "what if there were more than 3 distinct values?" — the honest answer is that this exact technique doesn't generalize cleanly past three partitions with a single pass; problems with more categories typically fall back to counting sort (if the value range is small and known) or a comparison sort. Saying this directly, instead of trying to force-fit a "four-way Dutch flag," is the stronger answer.

---

## Part 2 — Collections Internals: `ArrayList`, `LinkedList`, `ArrayDeque`

### Prerequisites (confirmed)

- `ArrayList`'s resizing/doubling and the amortized argument for `add()` (Week 1, Day 3).
- Stack (LIFO) / Queue (FIFO) introduced via `ArrayDeque` (Week 1, Day 4) — today explains *why* `ArrayDeque` was the right choice then, at the mechanism level.

### `ArrayList` — Contiguous Array

**Definition:** a resizable, indexed collection backed by a single contiguous `Object[]` array.

`get(i)` / `set(i)` are O(1) — direct address arithmetic (base address + `i × slot size`), no searching. `add()` at the **end** is O(1) amortized (Week 1's doubling argument). Insertion or removal at the **front or middle** is O(n): every subsequent element must physically shift one slot over (`System.arraycopy` under the hood), because the array's contiguous layout has no "gap" to insert into without moving everything after it.

### `LinkedList` — Doubly-Linked Nodes

**Definition:** a collection of individually-allocated nodes, each holding data plus a pointer to the *next* node and the *previous* node, with no requirement that nodes sit near each other in memory.

Insertion or removal at the **front or back** is O(1) — *given* you already hold a reference to the relevant node (or it's literally the head/tail), it's just a few pointer relinks, no shifting. But there is no random access: `get(i)` requires walking `i` links from whichever end is closer, making it O(n) — and inserting "at index i" is therefore also O(n) overall, since finding position i dominates the cost of the O(1) relink once you're there.

**🔑 Key Takeaway — Big-O isn't the whole story here.** `LinkedList` nodes are scattered across the heap wherever the allocator happened to place them, with no guarantee of proximity. `ArrayList`'s backing array is one contiguous block. CPU caches fetch memory in contiguous chunks (cache lines); scanning an `ArrayList` tends to pull several useful elements into cache per fetch, while walking a `LinkedList` tends to cause a cache miss on nearly every single node, since consecutive nodes aren't near each other in actual memory. The practical consequence: `LinkedList` can be **slower in real measured time** than an array-backed structure even in scenarios where its Big-O bound looks equal or better — a genuinely important, precise, and often-missing piece of "why" beyond the asymptotic label.

### `ArrayDeque` — Circular Array

**Definition:** a resizable array used as a **ring buffer** — a `head` index and a `tail` index track the deque's current ends, and "wrapping around" the array's boundary is handled with modular arithmetic (in practice, a bitmask, since the JDK implementation keeps capacity at a power of two, making `index & (capacity - 1)` equivalent to — and faster than — `index % capacity`).

**The key mechanism, precisely, since it's easy to assume `ArrayDeque` "shifts everything" the way `ArrayList` would:** adding to the *front* does **not** shift any existing elements. It decrements `head` by one position (wrapping to `capacity - 1` if `head` was at `0`) and writes the new element there. Adding to the *back* increments `tail` similarly. Both are O(1) — genuinely, not just amortized for this specific step, though *occasionally* (when the ring buffer fills) a resize-and-copy into a larger array is needed, which is where the O(1) becomes "amortized" overall, via the identical doubling argument as `ArrayList` and `StringBuilder`.

```
Conceptual layout (capacity 8, currently holding 4 elements, wrapped):
index:   0    1    2    3    4    5    6    7
        [C]  [D]  [ ]  [ ]  [ ]  [ ]  [A]  [B]
                                      ▲head      ▲tail wraps to end
front of deque: A (idx 6) → B (idx 7) → C (idx 0) → D (idx 1) : back of deque
```

This gives `ArrayDeque` the best of both structures for stack/queue use: O(1) (amortized) insertion/removal at *both* ends, like `LinkedList`, **plus** the contiguous-array cache-friendliness `LinkedList` lacks — which is precisely why the JDK's own documentation recommends `ArrayDeque` over both `Stack` and `LinkedList` for stack and queue use respectively.

```java
Deque<Integer> deque = new ArrayDeque<>();
deque.addFirst(1);   // [1]              — decrements head, no shifting
deque.addLast(2);    // [1, 2]           — increments tail
deque.addFirst(0);   // [0, 1, 2]        — decrements head again
deque.pollFirst();   // returns 0 → [1, 2]
deque.pollLast();    // returns 2 → [1]
```

Every one of those five calls is O(1) — none of them touch any element other than the one being added or removed, which is exactly the property `ArrayList`'s `add(0, x)` (front-insertion) cannot offer.

### Complexity Summary

| Operation | ArrayList | LinkedList | ArrayDeque |
|---|---|---|---|
| Get by index | O(1) | O(n) | O(n) *(not designed for index access)* |
| Add/remove at end | O(1) amortized | O(1) | O(1) amortized |
| Add/remove at front | O(n) | O(1) | O(1) amortized |
| Cache locality | Good (contiguous) | Poor (scattered nodes) | Good (contiguous) |

### Common Mistakes

- **⚠️ Choosing `LinkedList` by default "because insertion is O(1)."** True only at the ends, and only given a direct reference to the relevant node — `LinkedList`'s middle-insertion is *also* O(n) once you count the traversal needed to get there, and its real-world constant factors (allocation per node, pointer-chasing, cache misses) are usually worse than `ArrayList`'s in practice for most workloads.
- **⚠️ `ArrayDeque` does not permit `null` elements** — it throws `NullPointerException` on `add(null)`, unlike `ArrayList` and `LinkedList`, which both permit `null` freely. This is a real, concrete gotcha if migrating code between them.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** `CollectionsBenchmark` class — benchmark inserting 100,000 elements at index 0 of an `ArrayList` vs. an `ArrayDeque`, with comments on *why* the difference exists.

Practical guidance: use `System.nanoTime()` around each loop; expect the `ArrayList` front-insertion case to be dramatically slower (quadratic-shaped: each of the 100,000 inserts shifts everything already present) while `ArrayDeque`'s should look close to linear. If you want a third data point, add `LinkedList` front-insertion too (should be fast, since it's genuinely O(1) per insert at the front) — a good way to see the cache-locality point land concretely: `LinkedList` should *win* on Big-O here but the comments should explain what you'd need to change to actually see its real-world weakness (e.g., middle-index access instead of front-insertion).

## Career Block Guide (1 hr)

- **LinkedIn engagement (20 min):** 3–5 substantive comments, same standard as prior days.
- **Networking:** identify 5 Backend Engineers at Tier B companies. Practically: Tier B here typically means strong-but-less-saturated-with-applicants than the very top-tier names — worth deliberately widening the net beyond only the most obvious companies, since response rates tend to be meaningfully higher there. Save names today; Day 15 (next week) is when outreach based on today's research typically continues, per the ongoing networking cadence — check next week's plan once you have it for the exact follow-up day.

---

## Day 12 — Interview Questions

**Q1. Justify Container With Most Water's greedy pointer movement.** Area is bounded by the shorter of the two current lines; keeping the shorter one fixed and moving the taller one inward can never produce a *larger* area than moving the shorter one would have, since the bottleneck (the shorter line) is unchanged or worse either way — so the shorter pointer is always the one to move.

**Q2. State Sort Colors' three-region invariant.** `[0, low)` holds only 0s, `[low, mid)` holds only 1s, `[mid, high]` is unprocessed, `(high, n-1]` holds only 2s — maintained at every step of the single pass.

**Q3. Why is it safe to advance both `low` and `mid` together in the `0` branch, but not `mid` alone in the `2` branch?** In the `0` branch, the value being displaced from `low` is always either unprocessed (if `low==mid`) or a known `1` (if `low<mid`) — either way, the invariant already accounts for it. In the `2` branch, the value swapped in from `high` is completely unexamined and must be re-checked, so `mid` cannot safely advance past it yet.

**Q4. Is Sort Colors an opposite-ends or a fast-slow two-pointer problem?** Neither, cleanly — it's a three-pointer partition (Dutch National Flag) that borrows elements of both: `mid` scans like a fast pointer, while `low`/`high` behave more like boundary markers than either classic category. Worth naming as its own category rather than forcing it into an existing one.

**Q5. Why is `get(i)` O(1) on `ArrayList` but O(n) on `LinkedList`?** `ArrayList` computes a direct memory address from the index; `LinkedList` has no such formula — it must walk node-to-node from whichever end is closer, since nodes have no positional relationship to their index.

**Q6. Why can `LinkedList` be slower in practice than `ArrayList`, even in a scenario where their Big-O bounds are equal or favor `LinkedList`?** Cache locality — `ArrayList`'s contiguous backing array lets the CPU prefetch multiple elements per cache line; `LinkedList`'s nodes are scattered across the heap, so traversal tends to cause a cache miss on nearly every node.

**Q7. How does `ArrayDeque` achieve O(1) insertion at the front without shifting elements, the way `ArrayList` would need to?** It's a circular array: the `head` index simply decrements (wrapping around the array's boundary via modular/bitmask arithmetic) rather than requiring every existing element to move over.

---

## Daily Deliverable Check

- [ ] Container With Most Water confirmed solid from Week 1 (recap only, no re-solve needed).
- [ ] Sort Colors solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain precisely why `ArrayDeque` beats `LinkedList` for stack/queue use — mechanism, not just the conclusion.
- [ ] `CollectionsBenchmark` pushed, with commentary on *why* the timing difference exists.
- [ ] 5 Backend Engineers at Tier B companies identified.

---

## What Tomorrow Assumes You Already Know Cold

Day 13 — the Two Pointers capstone — assumes every variant taught so far (opposite-ends, from-the-back, fast-slow, one-forward-pointer-each, opposite-ends-with-a-skip, greedy pairing, and today's three-way partition) is name-and-justify-ready, since tomorrow's reflection exercise asks you to classify the entire 16-problem ladder without looking back at hints. Trapping Rain Water tomorrow also assumes today's invariant-proof style of reasoning (state the invariant, show it's maintained, show termination implies full coverage) is comfortable — it's the same proof shape, one notch harder.
