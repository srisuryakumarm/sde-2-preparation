# Day 102 — Sorting Algorithms From Scratch, Quickselect, and the Entire DSA Curriculum in Review

**Series:** SDE-2 Interview Prep · Week 15, Day 102 (of 105 DSA-phase days)
**Curriculum Map:** [00_Curriculum_Map.md](./00_Curriculum_Map.md)
**◀ Previous:** [Day 101](./Day101_Resource_Book.md) · **Next ▶:** [Day 103](./Day103_Resource_Book.md)

**Companion to:** Day 4 of `Week_15_Revised.md`

---

## ⚠️ Overlap Notice — Read This First

Today's required problem, **Kth Largest Element in an Array (LC 215)**, already appears in the curriculum map's problem inventory — solved in **Week 8, Day 55**, via a min-heap of size k. That's not a mistake in today's plan: it's deliberate. Today revisits the same problem specifically to attach a **second, genuinely different technique** to it — Quickselect — which is the entire point of the day. Per this map's own overlap-handling convention:

| Item | Status |
|---|---|
| Kth Largest Element in an Array (LC 215) — the problem itself | Already solved — Week 8, Day 55 (min-heap). **Short recap only, below.** |
| Quickselect — the technique | Genuinely new, full depth. **This is today's real content.** |

The heap solution is not re-derived. Quickselect — and the merge sort / quicksort mechanics it's built directly on top of — receives full treatment.

## Recap

Yesterday closed Segment Trees at 2/2 and implemented Factory Method and Builder in full, with unit tests. Today pivots to a mostly independent thread: **`Collections.sort()` and `Arrays.sort()` have been called throughout this entire series — as far back as Week 1's Group Anagrams sorting a character array into a canonical key — without ever once implementing what runs underneath either call.** Today closes that gap, then uses the mechanism it just built (quicksort's partition step) to revisit Kth Largest Element with a second technique. The day closes with a deliberately different kind of hour: a full recall pass across every pattern this entire DSA phase has covered.

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Implement merge sort and quicksort from scratch, in Java, and verify both against `Arrays.sort()` on a hand-built test array.
2. Prove merge sort's O(n log n) bound holds in every case, and explain precisely why it's stable while quicksort (as commonly implemented) is not.
3. Explain quicksort's average-case O(n log n) versus worst-case O(n²), with a concrete example of an input that triggers the worst case, and state the randomized-pivot mitigation.
4. Explain — correctly, with a mechanism, not just a name — why Java's `Arrays.sort()` uses dual-pivot quicksort for primitives but TimSort for objects.
5. Implement Quickselect by reusing quicksort's own partition step, and prove its O(n) average complexity is genuinely different from quicksort's O(n log n) — not the same argument restated.
6. Recite, from memory, every DSA pattern covered across the whole series and its one-sentence interview signal.

## Concept Dependency Map for Today

```
Week 2, Day 12 (Dutch National Flag —      Week 2 (arrays, recursion base cases)
3-way boundary-pointer partition)                    │
      │                                               │
      ▼                                               ▼
  Quicksort's partition step (NEW: the SAME    Merge Sort (NEW: independent
  boundary-pointer swap mechanism, reduced      of partitioning — divide,
  from 3 zones to 2 — ≤pivot / >pivot)          recurse, then MERGE two
      │                                          already-sorted halves)
      ▼
  Quicksort, complete (NEW: recurse
  BOTH sides of the partition point)
      │
      ▼                                    Week 8, Day 55 (Kth Largest,
  Quickselect (NEW: reuses the SAME        solved via min-heap — cited,
  partition, but recurses into only        not re-derived, per the
  ONE side — the side containing            Overlap Notice above)
  the target index)                                  │
      └───────────────────┬──────────────────────────┘
                           ▼
          Kth Largest Element in an Array (LC 215)
          — short recap (heap) + full depth (Quickselect)
```

---

## Part 1 — Merge Sort, From Scratch

### Mechanism

Divide and conquer: split the array in half, recursively sort each half, then **merge** the two already-sorted halves into one fully sorted array by repeatedly comparing their front elements and taking the smaller.

```java
public static void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return;               // base case: 0 or 1 element — already sorted
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

private static void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) {               // <= , not < — this is what makes the sort STABLE
            temp[k++] = arr[i++];
        } else {
            temp[k++] = arr[j++];
        }
    }
    while (i <= mid)   temp[k++] = arr[i++];  // copy any remaining left-half elements
    while (j <= right) temp[k++] = arr[j++];  // copy any remaining right-half elements
    System.arraycopy(temp, 0, arr, left, temp.length);
}
```

### Why it's correct — by induction

**Base case:** an array of 0 or 1 elements is trivially sorted — nothing to do.

**Inductive step:** assume both halves are correctly sorted (this is exactly what the two recursive calls guarantee, by the inductive hypothesis). `merge` then produces a sorted combination of them: at every step, it places whichever of the two current front elements is smaller. Since each half is individually sorted, that front element is always the smallest *remaining* element in its own half — so the smaller of the two fronts is always the smallest remaining element **overall**. Once one half is exhausted, everything left in the other half is, by its own sortedness, still in increasing order and safe to append directly.

### Why it's stable — mechanism, not assertion

Using `arr[i] <= arr[j]` (not `<`) means that when the two front elements are **equal**, the element from the **left** half is taken first. Because the left half's elements originally preceded the right half's elements in the pre-split array, taking the left one first preserves their original relative order. Flip that to strict `<`, and equal elements could be taken from whichever half's pointer happens to be compared first, with no guarantee tied to original position — breaking stability. This is precisely the mechanism, not just the label "merge sort is stable."

### Complexity — via the recurrence

`T(n) = 2T(n/2) + O(n)` — two recursive calls on half the size, plus O(n) merge work. By the Master Theorem: `a = 2, b = 2`, so `n^(log_b a) = n^1 = n`, and `f(n) = O(n) = Θ(n¹)` — this matches Master Theorem Case 2 exactly, giving `T(n) = Θ(n log n)`.

**Crucially, this bound holds in every case — best, average, and worst.** The split is always even (or as close to even as an odd length allows) regardless of the input's actual values, so there's no adversarial input that can push this toward O(n²) the way there is for quicksort below.

**Space:** O(n) auxiliary, for the `temp` array (`[EXTENSION]`: a single O(n) buffer allocated once and reused across every `merge` call, rather than freshly allocated per call, is a common practical optimization — not required for correctness).

---

## Part 2 — Quicksort, From Scratch

### Mechanism

Also divide and conquer, but the work happens **before** recursing, not after. Pick a pivot, **partition** the array so everything ≤ pivot ends up to its left and everything > pivot ends up to its right — placing the pivot in its own final, correct sorted position — then recursively sort the two sides independently. No merge step is needed: once each side is individually sorted, the whole array is sorted, because the partition step already guaranteed everything on the left is ≤ everything on the right.

```java
public static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIndex = partition(arr, low, high);
        quickSort(arr, low, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, high);
    }
}

private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];        // choosing the last element as pivot
    int i = low - 1;              // boundary: everything in arr[low..i] is confirmed <= pivot
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr, i, j);
        }
    }
    swap(arr, i + 1, high);       // place the pivot in its final position
    return i + 1;                 // the pivot's final, correct index
}

private static void swap(int[] arr, int a, int b) {
    int temp = arr[a]; arr[a] = arr[b]; arr[b] = temp;
}
```

> 🔗 **Direct connection to Week 2, Day 12:** this partition scheme — a boundary pointer `i` tracking "everything confirmed to belong in one zone so far," advanced and swapped as a scanning pointer `j` finds more qualifying elements — is the *exact* mechanism Sort Colors' Dutch National Flag algorithm used, reduced from **three** zones (fixed values `0`/`1`/`2`) down to **two** (`≤ pivot` / `> pivot`, with the pivot's value determined by the data itself rather than fixed in advance). If Dutch National Flag's boundary-pointer-and-swap invariant is already reflexive, nothing about *this* mechanism is new — only the two-zone simplification and the data-dependent pivot are.

### Why partition is correct

**Invariant, maintained as `j` scans from `low` to `high−1`:** everything in `arr[low..i]` is `≤ pivot`. Whenever `arr[j] ≤ pivot` is found, the `≤pivot` region is extended by incrementing `i` and swapping `arr[i]` into position — moving that qualifying element into the confirmed region without disturbing the invariant. Elements found `> pivot` are simply skipped over (left where they are, which — by the invariant — is correctly *outside* the confirmed `≤pivot` region). After the loop, swapping the pivot itself (`arr[high]`) into position `i+1` places it exactly between the two zones — everything at or before it is `≤ pivot`, everything after is `> pivot`. That's the pivot's genuine final sorted-array position, unconditionally.

### Complexity

**Average case: O(n log n).** If the pivot splits the array reasonably evenly on average (true for random or non-adversarial input), the recursion behaves like merge sort's — `T(n) = 2T(n/2) + O(n)`.

**Worst case: O(n²).** Occurs when partitioning is maximally unbalanced *every single call* — e.g., an already-sorted array with "always pick the last element" pivot selection: each partition splits into a group of size `n−1` and a group of size `0`, doing O(n) work but only shrinking the problem by 1 each time. Total work: `n + (n-1) + (n-2) + ... + 1 = O(n²)` — the same arithmetic-series shape this series has already proven for other naive O(n²) approaches.

**The randomized-pivot mitigation:** swap `arr[high]` with a *randomly chosen* index in `[low, high]` before partitioning. This doesn't change the theoretical worst case (an adversary could still get catastrophically unlucky) — it makes that worst case **vanishingly unlikely for any fixed input**, since no input can be crafted in advance to reliably trigger it once the pivot choice depends on randomness rather than position. This gives **expected O(n log n)** regardless of input arrangement, which is the standard, citable answer to "how do you avoid quicksort's worst case in practice."

**Space:** O(log n) auxiliary for the recursion stack in the balanced (average) case, degrading to O(n) in the unbalanced (worst) case — but **no auxiliary array** is needed, unlike merge sort's O(n) buffer. This in-place property is quicksort's most concrete practical advantage.

### Stability — and why quicksort (as implemented here) doesn't have it

Quicksort is **not stable**. The `swap` calls inside `partition` can and do reorder equal elements relative to each other — a swap moves whatever value currently sits at index `i` to index `j` and vice versa, with no logic protecting the relative order of two elements that happen to be equal in value. Contrast directly with merge sort's `merge`, which never swaps — it only *copies*, in an order explicitly controlled by the `<=` comparison, which is precisely what gives merge sort its stability guarantee and quicksort's swap-based partition its lack of one.

| | Merge Sort | Quicksort |
|---|---|---|
| Worst case | O(n log n) — guaranteed | O(n²) |
| Extra space | O(n) | O(log n) avg / O(n) worst — in-place otherwise |
| Stable? | Yes | No |
| Typical practical speed | Slower (allocation, less cache-friendly) | Faster (in-place, better cache locality) |

**Common mistakes:** forgetting the base case (`low < high`) and recursing infinitely or out of bounds; picking a fixed pivot (always first or always last element) on data that's likely to already be sorted or reverse-sorted, silently walking straight into the O(n²) case without realizing it; assuming quicksort is stable by analogy with merge sort, when the two algorithms' correctness mechanisms (swap-based partitioning vs. copy-based merging) are fundamentally different on exactly this axis.

---

## Part 3 — Why Java's `Arrays.sort()` Uses Two Different Algorithms

This is the actual payoff the plan names explicitly: knowing *both* algorithms well enough to explain the engineering trade-off between them, not just recite two names.

- **`Arrays.sort(int[])`** (and other primitive-array overloads) uses **Dual-Pivot Quicksort** (adopted since Java 7, based on work by Yaroslavskiy, Bentley, and Bloch) — an in-place, generally faster-in-practice variant of the quicksort just built above, using two pivots instead of one to partition into three regions per pass instead of two.
- **`Arrays.sort(Object[])`** and **`Collections.sort()`** use **TimSort** — a highly optimized, *adaptive* hybrid, fundamentally built on merge sort's core idea (merging sorted runs) combined with insertion sort for small runs and specific optimizations for data that's already partially ordered.

**Why this split, precisely — not just "one's for primitives, one's for objects":**

1. **Stability matters for objects, not for primitives.** Two equal `int` values are genuinely indistinguishable — there's no notion of "identity" beyond the value itself, so reordering two equal ints has no observable effect on anything. Two `Person` objects that happen to compare equal on `lastName` are *not* interchangeable — a caller sorting a list of people by last name has a reasonable expectation that people sharing a last name keep their original relative order. Quicksort's in-place, swap-based mechanism can't offer that guarantee (as shown in Part 2); merge sort's copy-based mechanism can, cleanly.

2. **Comparisons are cheap and fixed for primitives, expensive and arbitrary for objects.** Primitive comparison is a single machine instruction. Object comparison invokes a user-supplied `compareTo`/`Comparator` — potentially expensive, and, worse, potentially **adversarial or buggy**. A guaranteed-worst-case algorithm (merge sort's O(n log n) in every case) is the safer choice when the comparison logic isn't fully trusted or controlled; quicksort's O(n²) worst case is a real risk specifically when comparisons come from arbitrary user code.

3. **In-place matters more for primitives.** No allocation overhead and strong cache locality are a bigger practical win for large primitive arrays (stored contiguously, compared cheaply) than for arrays of object references (already one level of indirection away from their actual data, so the memory-locality advantage of in-place swapping is smaller in practice).

`[EXTENSION — worth naming, not required to defend in depth]`: TimSort isn't literally textbook merge sort — it's adaptive, detecting existing sorted "runs" in the input and merging those directly (very fast on partially-sorted, real-world data like log files or resubmitted mostly-sorted lists), and uses insertion sort for small runs since insertion sort genuinely outperforms merge sort's overhead on small inputs. Its core mechanism — merging sorted subsequences — is still fundamentally the merge sort idea, which is why "a variant of merge sort" is an accurate description rather than a loose one.

> 💡 **Interview Insight:** "explain quicksort's worst case" and "why does Java use different sort algorithms for primitives vs. objects" are both real, if less frequently asked, SDE-2 questions — and both are now answerable with an actual mechanism, not a memorized one-liner.

---

## Part 4 — Kth Largest Element in an Array (LeetCode #215, Medium)

**Statement:** given an unsorted array, find the k-th largest element.

### 🔗 Recap: already solved, Week 8 Day 55 (min-heap)

Maintain a min-heap of size `k` over the array; after processing every element, the heap's minimum (its root) is the k-th largest overall. **O(n log k)** time, **O(k)** space. Not re-derived here — see Week 8, Day 55 for the full treatment. What's new today is a second, structurally unrelated technique for the exact same problem.

### New technique, full depth: Quickselect

**The reframe:** reuse quicksort's own `partition` step, unmodified, from Part 2. After one partition call, the pivot sits in its true, final sorted-array position — `pivotIndex`. Quicksort's *inefficiency* (relative to what this problem actually needs) is that it then recurses into **both** sides, sorting the entire array, when all this problem needs is **one specific position**. Quickselect's entire idea: recurse into only whichever side actually contains the target index, and ignore the other side completely.

**Converting "k-th largest" into a target index:** if the array were fully sorted ascending, the k-th largest element sits at index `n − k` (0-indexed). So the target is `n − k`, and the loop partitions repeatedly until the pivot lands exactly there.

```java
public static int findKthLargest(int[] nums, int k) {
    int targetIndex = nums.length - k;
    int low = 0, high = nums.length - 1;
    Random rand = new Random();
    while (true) {
        // randomize the pivot first — same worst-case mitigation as quicksort
        int randomIndex = low + rand.nextInt(high - low + 1);
        swap(nums, randomIndex, high);

        int pivotIndex = partition(nums, low, high);   // the SAME partition method from Part 2
        if (pivotIndex == targetIndex) {
            return nums[pivotIndex];
        } else if (pivotIndex < targetIndex) {
            low = pivotIndex + 1;      // target must be in the right side
        } else {
            high = pivotIndex - 1;     // target must be in the left side
        }
    }
}
```

**Why this is correct:** `partition` guarantees the pivot ends up in its true final sorted-array position on every call (proven in Part 2 — this doesn't need to be re-proven, only cited). So: if `pivotIndex == targetIndex`, the answer is found exactly. If `pivotIndex < targetIndex`, everything at or before `pivotIndex` is `≤` the pivot, meaning the target (a *larger* index, hence a value that would sit further right in sorted order) cannot be anywhere in the left region — it must be in the right side. The symmetric argument holds for `pivotIndex > targetIndex`. Each iteration **eliminates one entire side** of the current search range — conceptually the same halving intuition as Binary Search (Week 4–5), except the split point here is **data-dependent** (wherever the pivot happens to land) rather than a fixed midpoint.

### Complexity — the actual argument, not an assertion

**Average case: O(n).** This needs to be argued, because it looks like it should match quicksort's O(n log n) and it doesn't. Quicksort's recurrence is `T(n) = 2T(n/2) + O(n)` — it recurses into **both** halves. Quickselect's recurrence is `T(n) = T(n/2) + O(n)` — it recurses into only **one** half, since the other side is now irrelevant and thrown away entirely. Expanding this out as a sum:

`T(n) = n + n/2 + n/4 + n/8 + ... + 1`

This is a **geometric series** with ratio ½, and a geometric series is dominated entirely by its **first** term — the sum converges to `2n = O(n)`. This is a fundamentally different shape from merge sort/quicksort's recurrence, which has O(log n) levels each contributing a *full-sized* O(n) of work (giving O(n log n) total) — here, each level's work *shrinks* geometrically, so the total is bounded by a small constant multiple of the very first level's work alone.

> 🔑 **Key Takeaway:** "Quickselect recurses into one side instead of both" isn't just a minor implementation detail — it's the entire reason its complexity class is different (O(n), not O(n log n)) from quicksort's, despite both being built on the identical partition step.

**Worst case: O(n²)** — same adversarial-input risk as quicksort, mitigated the same way (randomized pivot selection, included in the code above).

**Space: O(1) extra**, using the iterative `while`-loop form shown above — a deliberate choice. A recursive version would use O(log n) average call-stack space (each recursive call handles roughly half the remaining range); writing it iteratively avoids that entirely, which is a genuine, citable advantage over the recursive formulation.

**Comparing the two full solutions head-to-head:**

| | Min-Heap (Week 8) | Quickselect (today) |
|---|---|---|
| Time | O(n log k) — guaranteed | O(n) average, O(n²) worst |
| Space | O(k) | O(1) |
| Streaming-friendly (don't need the full array upfront)? | Yes | No — needs random access to the whole array |
| Mutates the input array? | No | **Yes** — partitioning rearranges elements in place |
| Best when... | k is small relative to n, or data arrives as a stream | The whole array is already in memory and k is a significant fraction of n |

> ⚠️ **Common Mistake:** using Quickselect when the caller needs the original array order preserved afterward. Partitioning rearranges `nums` in place — if that's not acceptable, either copy the array first (adding back O(n) space, eroding Quickselect's main advantage) or use the heap approach instead. This is a genuine, practical trade-off worth stating unprompted.

**Edge cases:** `k = 1` (the maximum element, target index `n-1`); `k = n` (the minimum, target index `0`); many duplicate values (`partition`'s `≤` comparison handles duplicates correctly, though — `[EXTENSION]` — Lomuto-style partitioning can degrade in performance specifically on inputs with many values equal to the pivot; a three-way partition, à la Dutch National Flag's original three zones, is the standard fix if that's flagged as a concern, not required here).

**Interview framing:** name both approaches — heap and Quickselect — and their trade-offs *before* picking one to implement; that comparison, unprompted, is the strongest signal in this problem. Likely follow-ups: *"what if this needs to run many times as the array changes?"* — neither approach is ideal for that; a Segment Tree (Day 100) or an order-statistics structure would be the right direction to name. *"What if the data doesn't fit in memory?"* — the heap wins, and being able to say precisely why (streaming, bounded O(k) space, no need for random access) is the expected depth.

---

## Part 5 — Theory Block: The Entire DSA Curriculum, Looking Back

This hour is spent differently on purpose: recall every pattern from memory, name its one-sentence interview signal, before checking anything against notes. What follows is the reference version of that exercise — useful to build *after* attempting it cold, not instead of attempting it cold.

| Pattern | First taught | Interview signal — the phrase that says "use this" |
|---|---|---|
| HashMap / HashSet | Week 1 | "have I seen this before," "count occurrences," "find a pair/complement" — anything needing O(1) lookup |
| Two Pointers | Week 1–2 | Sorted (or sortable) array/string, looking for a pair/triplet, or in-place partitioning |
| Sliding Window | Week 2 | "contiguous subarray/substring" + a size, sum, or distinct-count constraint |
| Prefix Sum & Kadane's | Week 3 | Repeated range-sum queries on a *static* array; "maximum subarray sum" |
| Greedy & Intervals | Week 4 | "maximum/minimum number of..." with a proof that the locally-best choice never needs revisiting; overlapping start/end ranges |
| Binary Search | Week 4–5 | Sorted data, OR a monotonic yes/no feasibility check across a range of candidate answers ("search on the answer") |
| Linked Lists | Week 5–6 | In-place list manipulation, cycle detection, needing O(1) insert/delete without shifting |
| Stacks / Monotonic Stack | Week 6 | "matching/balanced/nested," "most recent," or "next greater/smaller element" |
| Trees (general, BST) | Week 7 | Hierarchical data; BST specifically signals an ordering property usable for pruning |
| Heaps / Priority Queue | Week 8 | "k-th largest/smallest," "top k," "merge k sorted things," anything needing repeated access to a running min/max |
| Tries | Week 9 | Prefix-based lookup over a set of strings — "starts with," "autocomplete," "word search over a fixed dictionary" |
| Backtracking | Week 9–10 | "all possible," "every combination/permutation/subset," with a need to prune invalid partial choices early |
| Graphs (BFS/DFS) | Week 10–11 | Explicit or implicit connections between entities; needs an explicit `visited` set once cycles are possible |
| Union-Find | Week 11 | "are these connected," dynamic connectivity queries, redundant-edge detection |
| Dijkstra's / Bellman-Ford | Week 12 | Weighted shortest path; Bellman-Ford specifically for negative weights or a bounded number of hops |
| Dynamic Programming (6 subtypes) | Week 12–14 | "count the number of ways," "min/max cost/value," with overlapping subproblems and optimal substructure — 1D, Grid, String, Interval, State Machine, and Tree DP each have their own further-specific signal |
| Bit Manipulation | Week 14–15 | Values naturally described in binary, single-number-among-duplicates problems, or explicit O(1)-per-bit constraints |
| Segment Trees | Week 15 | Range sum/min/max query **combined with** the array also being frequently updated |
| Sorting (from scratch) | Week 15 | "implement X sort," or needing to explain *why* a language's built-in sort behaves the way it does |

**Recall exercise, done properly, also means being able to say — for a genuinely new, unseen problem — which *one or two* of the above it most resembles, and why, before writing any code.** That recognition, not the count of problems solved, is what this hour is actually building.

---

## Project Block Guide (1 hr)

**Repository:** `dsa-java`. No new content today — this slot is a **final-pass cleanup**, not a coding task: confirm every pattern folder is present, correctly named, and easy to navigate at a glance, since this repository is about to shift from "a place solutions went" to "the resource leaned on constantly during real interview review." Concretely: check folder names match pattern names consistently (no stray abbreviations or typos), confirm every problem file within a folder is named recognizably (LC number + short title), and add or fix a top-level README index if one doesn't already exist or has drifted out of date.

## Career Block Guide (1 hr)

- **LinkedIn:** engagement — 20 minutes commenting substantively on 3–5 posts.
- **Networking:** verify every Tier B and Tier C application from the last month is actually **submitted**, not just drafted — a real, easy-to-let-slip failure mode worth an explicit checklist pass.

---

## Day 102 — Interview Questions

**Q1. Prove merge sort's O(n log n) bound — don't just cite it.**

*Answer:* The recurrence is `T(n) = 2T(n/2) + O(n)` — two half-sized recursive calls plus O(n) merge work. By the Master Theorem, `a=2, b=2` gives `n^(log_b a) = n`, matching `f(n) = O(n)` exactly (Case 2), so `T(n) = Θ(n log n)`. This holds in every case — best, average, worst — because the split is always even regardless of input values, unlike quicksort's data-dependent partitioning.

---

**Q2. Why is merge sort stable, mechanically — not just "because it is"?**

*Answer:* The merge step compares with `<=`, so when two front elements are equal, the element from the left half is always taken first. Since the left half's elements originally preceded the right half's in the pre-split array, this preserves their relative order. The mechanism is copy-based (never swaps), which is what makes this guarantee possible in the first place.

---

**Q3. Give a concrete input that triggers quicksort's O(n²) worst case with "always pick the last element" pivot selection, and explain why.**

*Answer:* An already-sorted array. Every partition call picks the largest remaining element as pivot, splitting into a group of size `n-1` and a group of size `0` — O(n) work per call, but the problem only shrinks by 1 each time. Total work is `n + (n-1) + ... + 1 = O(n²)`, the same arithmetic-series shape as other naive O(n²) approaches in this series.

---

**Q4. How does randomizing the pivot fix quicksort's worst case?**

*Answer:* It doesn't change the theoretical worst case — an unlucky sequence of random choices is still possible. It removes the ability for any *fixed input* to reliably trigger that worst case, since the pivot no longer depends on array position. This gives expected O(n log n) regardless of input arrangement, which is the standard mitigation.

---

**Q5. Is quicksort, as implemented today, stable? Why or why not?**

*Answer:* No. The partition step swaps elements to maintain its boundary invariant, and a swap can reorder two equal elements relative to each other with no logic protecting their original order — fundamentally different from merge sort's copy-based, order-preserving merge.

---

**Q6. Why does Java's `Arrays.sort()` use dual-pivot quicksort for primitives but TimSort for objects — give the actual mechanism, not just the names.**

*Answer:* Three reasons: (1) stability matters for objects (two `Person`s with the same last name have a reasonable expectation of keeping relative order) but is meaningless for primitives (two equal ints are indistinguishable); (2) object comparisons are user-supplied, potentially expensive or even adversarial, making merge sort's guaranteed-worst-case O(n log n) the safer choice, while primitive comparisons are cheap, fixed hardware operations where quicksort's average-case speed and in-place nature are a bigger practical win; (3) in-place swapping's cache-locality advantage matters more for contiguous primitive data than for arrays of object references, which are already one level of indirection removed from their actual data.

---

**Q7. What is TimSort, precisely — is it accurate to call it "a variant of merge sort"?**

*Answer:* Yes, accurately — its core mechanism is merging sorted subsequences, the same fundamental idea as merge sort. It's adaptive on top of that: it detects existing sorted "runs" in the input and merges those directly (fast on partially-sorted real-world data), and uses insertion sort for small runs since insertion sort's low overhead beats merge sort's on small inputs.

---

**Q8. Quickselect and quicksort both use the exact same `partition` method. Why is Quickselect's average complexity O(n) and not O(n log n) like quicksort's?**

*Answer:* Quicksort recurses into *both* sides of each partition — `T(n) = 2T(n/2) + O(n)`, giving O(n log n). Quickselect recurses into only *one* side, discarding the other entirely, since only one side can contain the target index — `T(n) = T(n/2) + O(n)`. Expanded out, this is the geometric series `n + n/2 + n/4 + ... ≈ 2n = O(n)`, dominated by its first term — a fundamentally different shape from quicksort's recurrence, not just a smaller constant on the same shape.

---

**Q9. Why does Quickselect's loop convert "k-th largest" into `n - k` as a target index?**

*Answer:* If the array were fully sorted in ascending order, the k-th largest element would sit at index `n - k` (0-indexed) — e.g., the 1st largest (the maximum) is at index `n-1`. Partitioning repeats until the pivot's own final position exactly equals that target index.

---

**Q10. What are the real, practical trade-offs between the heap approach and Quickselect for this exact problem?**

*Answer:* The heap is O(n log k) time, O(k) space, guaranteed worst case, and works on a stream without needing the full array in memory upfront. Quickselect is O(n) average time, O(1) extra space, but has a real (if unlikely, with randomization) O(n²) worst case, requires random access to the whole array upfront, and mutates the input array in place — a genuine problem if the caller needs the original order preserved.

---

## Daily Deliverable Check

- [ ] Merge sort implemented from scratch; verified against `Arrays.sort()` on a hand-built array.
- [ ] Quicksort implemented from scratch; verified against `Arrays.sort()`; randomized pivot included.
- [ ] O(n log n) proven for merge sort (every case) and argued for quicksort (average case), with a concrete O(n²) trigger case for quicksort's worst case.
- [ ] The dual-pivot-quicksort-vs-TimSort explanation reproducible with the actual mechanism, not just the two names.
- [ ] Kth Largest Element (LC 215) solved via Quickselect, reusing the day's own `partition` method; the O(n)-vs-O(n log n) recurrence contrast with quicksort explainable precisely.
- [ ] Heap-vs-Quickselect trade-off table reproducible without notes.
- [ ] Full pattern-and-signal recall exercise attempted from memory first, then checked.
- [ ] `dsa-java` repository final pass complete — folders named consistently, README index current.
- [ ] LinkedIn engagement completed. All Tier B/C applications from the last month confirmed actually submitted.

## What Tomorrow Assumes You Already Know Cold

Tomorrow (Day 103) opens an entirely new track — SQL — with essentially no dependency on today's sorting material. It does assume today's retrospective actually happened: the whole point of tomorrow's SQL problems (and the rest of this series' remaining interview prep) is being able to recognize which *pattern* a new problem needs quickly, which is exactly what today's hour was for. Tomorrow also leans on Week 6, Day 40's SQL fundamentals — joins, keys, indexes, the NULL/`NOT IN` trap — as already-solid background, extending them with subqueries, correlated subqueries, `GROUP BY`/`HAVING`, and self-joins for the first time.

**Next ▶:** [Day 103](./Day103_Resource_Book.md)
