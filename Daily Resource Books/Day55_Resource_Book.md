# Day 55 Resource Book — Heaps Continue: Two-Heap and Kth-Order Patterns

**Series:** SDE-2 Interview Prep Resource Books (Week 8) · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 54](./Day54_Resource_Book.md) · **Next ▶:** [Day 56](./Day56_Resource_Book.md)
**Companion to:** Day 55 of `Week_08_Revised.md`

---

## ⚠️ A note on today's title, before anything else

`Week_08_Revised.md`'s header for today reads "Heaps Continue, and Spring Cloud Config Wrap-Up" — but the plan's actual body for Day 55 contains only a DSA Block and a Career Block; there's no Theory or Project block content describing what a "Spring Cloud Config Wrap-Up" would actually cover. This is worth naming directly rather than either silently ignoring the title or inventing new theory content that isn't in the plan: **today's book follows the plan's actual body**, which means no new backend theory today. If a genuine Spring Cloud Config Server deep-dive (the centrally-managed, Git-backed, `@RefreshScope`-capable version briefly named on Day 51, as distinct from the profiles mechanism actually built that day) is wanted later, that would need to be a deliberately scheduled addition — not something to retrofit into today's already-full DSA agenda based on a title that the plan's own body doesn't back up.

## Recap: what today is

Yesterday built the heap mechanism from the array up and used it two ways: a fixed-size min-heap tracking a live stream (Kth Largest Element in a Stream), and a max-heap repeatedly combining extremes (Last Stone Weight). Today's two required problems extend both of those shapes — Kth Largest Element in an Array is the same min-heap-of-size-k idea from yesterday's stream problem, minus the "across multiple calls" statefulness, and it's also where a debt from Week 2 finally comes due: Day 12's Sort Colors noted that its Dutch National Flag partitioning technique "resurfaces with Quickselect later." Today is later. K Closest Points to Origin extends the size-k heap pattern with a **custom** ordering (distance, not a value's natural order) — the first time this week's heap problems need a hand-written `Comparator`, reusing the exact anonymous-inner-class technique from Day 35.

Two extras land today: Minimum Cost to Connect Sticks — deferred from yesterday specifically because Heaps' opening day correctly got no extras — mirrors Last Stone Weight's shape with one flipped detail worth understanding precisely, not just noting. Kth Smallest Element in a Sorted Matrix reuses today's own kth-order-statistic theme and, as a bonus, resolves another old thread: Day 32 mentioned a "staircase" O(m+n) technique for a related matrix problem (LC 240) but explicitly didn't give it full treatment. It gets used for real here.

## Learning Objectives

By the end of today, without notes:

1. Solve Kth Largest Element in an Array three ways — sort, heap, and Quickselect — and explain why Quickselect is O(n) average while ordinary quicksort is O(n log n), despite both being built on the same partition step.
2. Explain precisely how Day 12's Dutch National Flag partition and today's Quickselect partition are the same idea and how they differ.
3. Solve K Closest Points to Origin with a custom-comparator max-heap, and explain why comparing squared distances (not true distance) is both correct and preferable.
4. Solve Minimum Cost to Connect Sticks, and explain — precisely, via a cost-accounting argument — why combining the two *smallest* sticks each round is optimal, in contrast to Last Stone Weight's two-*largest* rule.
5. Solve Kth Smallest Element in a Sorted Matrix two ways (heap; binary search on value + staircase count), and connect the second approach explicitly back to Day 32's unfinished LC 240 mention and to Weeks 4–5's "binary search on the answer" framing.

## Concept Dependency Map for Today

```
Day 12: Dutch National Flag partition (three-way, around fixed 0/1/2 values) — "resurfaces with Quickselect later"
Day 54: min-heap-of-size-k (Kth Largest Element in a Stream)
        │
        ▼
Problem 3: Kth Largest Element in an Array (LC 215)
  needs: (a) yesterday's heap-of-size-k shape, restated without statefulness
         (b) NEW — Quickselect, a two-way partition variant of Day 12's technique

Day 35: custom Comparator via anonymous inner class (Merge k Sorted Lists' heap approach)
        │
        ▼
Problem 4: K Closest Points to Origin (LC 973)
  needs: max-heap of size k, ordered by a CUSTOM comparator (squared distance)

Extra 1: Minimum Cost to Connect Sticks (LC 1167)
  needs: today's heap mechanism, MIN-heap this time — contrast directly with Day 54's Last Stone Weight

Day 32: LC 240 "staircase" technique NAMED, not solved with full treatment
Day 28-33: Binary Search "on the answer" — fully closed pattern
        │
        ▼
Extra 2: Kth Smallest Element in a Sorted Matrix (LC 378)
  needs: heap-of-size-k (approach 1) OR binary search on value + Day 32's staircase count (approach 2, resolved for real)
```

---

# Part 1 — Kth Largest Element in an Array: Three Approaches

## Problem 3: Kth Largest Element in an Array (LeetCode 215, Medium) — Pattern: Min-Heap of Size k / Quickselect

**Statement:** Given an unsorted array and an integer `k`, return the kth largest element — the kth largest *in sorted order*, not the kth distinct value.

### Approach 1 — Brute force: sort

```java
public int findKthLargestSort(int[] nums, int k) {
    Arrays.sort(nums);
    return nums[nums.length - k];
}
```

Sort ascending, read off the element `k` positions from the end. **Time O(n log n), Space O(log n)–O(n)** depending on the sort implementation's internal overhead. Simple, correct, and does more work than necessary — full sorted order is never actually needed, only the one boundary value.

### Approach 2 — Min-heap of size k (yesterday's shape, restated)

```java
public int findKthLargestHeap(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) {
            minHeap.poll();
        }
    }
    return minHeap.peek();
}
```

Identical reasoning to Kth Largest Element in a Stream: keep a bounded window of the k largest values seen; its smallest member is the answer. **Time O(n log k), Space O(k).**

### Approach 3 — Quickselect: the Day 12 debt, paid

**The connection, stated precisely:** Day 12's Sort Colors rearranged an array into three zones in one pass, around three *known, fixed* values (0, 1, 2) — the Dutch National Flag technique. Quickselect's partition step is the *same underlying idea* — rearrange an array in one pass relative to a pivot — with two changes: **two** zones instead of three (≤ pivot, > pivot), and the pivot's value comes from **an element already in the array**, not one of several known constants.

```java
public int findKthLargest(int[] nums, int k) {
    int targetIndex = nums.length - k;   // kth largest = this index, in ascending sorted order
    int left = 0, right = nums.length - 1;

    while (left < right) {
        int pivotIndex = partition(nums, left, right);
        if (pivotIndex == targetIndex) {
            return nums[pivotIndex];
        } else if (pivotIndex < targetIndex) {
            left = pivotIndex + 1;
        } else {
            right = pivotIndex - 1;
        }
    }
    return nums[left];
}

private int partition(int[] nums, int left, int right) {
    int pivot = nums[right];
    int i = left;
    for (int j = left; j < right; j++) {
        if (nums[j] <= pivot) {
            swap(nums, i, j);
            i++;
        }
    }
    swap(nums, i, right);
    return i;   // pivot's FINAL, correct sorted position
}

private void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

**Why this is correct:** after `partition`, the pivot sits at its true sorted-order index — everything before it is ≤ it, everything after is > it. That index is either exactly the target, in which case the answer is found immediately, or it tells you unambiguously which side of the array the target must be in (Binary Search's "eliminate half" logic, Day 28, applied to an array position instead of a search key) — so the next partition only needs to recurse into that one side.

**Worked trace:** `nums = [3,2,1,5,6,4]`, `k=2` → `targetIndex = 6-2 = 4` (0-indexed position of the 2nd-largest in ascending sorted order `[1,2,3,4,5,6]`, which is `5`).

- `partition(nums, 0, 5)`: pivot = `nums[5] = 4`. Scanning: `3,2,1 ≤ 4` all swap into place at `i=0,1,2`; `5,6 > 4`, skipped. Final swap places pivot at index `3`: array becomes `[3,2,1,4,6,5]`. Returns `3`.
- `3 < targetIndex(4)` → search right half: `left = 4`.
- `partition(nums, 4, 5)`: pivot = `nums[5] = 5`. `nums[4]=6 ≤ 5`? No. Final swap places pivot at index `4`: array becomes `[3,2,1,4,5,6]`. Returns `4`.
- `4 == targetIndex(4)` → **return `nums[4] = 5`.** Correct.

**Why Quickselect is O(n) average, while quicksort — built on the identical partition step — is O(n log n):** quicksort recurses into **both** sides of every partition, so its total work follows `T(n) = 2T(n/2) + O(n)`, which resolves to `O(n log n)`. Quickselect recurses into only **one** side — whichever one the target index falls in — so its work follows (on average) `T(n) = T(n/2) + O(n)`, a geometric series that sums to `O(n)` total, not `O(n log n)`, because each successive level does proportionally less work and there's no second branch to add a `log n` factor.

**Worst case: O(n²).** If the pivot (here, always `nums[right]`) is consistently the smallest or largest remaining element — which happens on already-sorted or reverse-sorted input with this specific pivot choice — each partition only shrinks the search space by one element, giving `O(n)` partitions of `O(n)` work each. **The standard mitigation:** choose a random index as the pivot (swap it into the `right` position before partitioning) — this makes the worst case astronomically unlikely rather than eliminating it, turning the *expected* case into O(n) regardless of input order.

| Approach | Time | Space |
|---|---|---|
| Sort | O(n log n) | O(log n)–O(n) |
| Min-heap, size k | O(n log k) | O(k) |
| Quickselect | O(n) average, O(n²) worst case | O(1) extra (in-place) |

**Edge cases:** `k = 1` (the maximum — every approach handles this without special-casing); `k = n` (the minimum); duplicate values (the `≤` comparison in `partition` handles duplicates correctly, keeping the algorithm's correctness intact regardless of how many elements tie).

> 💡 **Interview Insight:** naming all three approaches, and specifically being able to state *why* Quickselect beats the heap approach asymptotically (O(n) vs. O(n log k)) while also being honest about its worst case and the randomized-pivot mitigation, is a strong, complete answer to this extremely common question. The heap approach is a perfectly good fallback if Quickselect's partition logic isn't fully solid under pressure — naming the trade-off explicitly is better than attempting Quickselect and leaving a buggy partition function.

---

# Part 2 — Custom-Comparator Heaps

## Problem 4: K Closest Points to Origin (LeetCode 973, Medium) — Pattern: Max-Heap of Size k, Custom Comparator

**Statement:** Given an array of `[x, y]` points, return the `k` points closest to the origin `(0, 0)`, in any order.

### Approach 1 — Brute force: sort by distance

```java
public int[][] kClosestSort(int[][] points, int k) {
    Arrays.sort(points, new Comparator<int[]>() {
        @Override
        public int compare(int[] a, int[] b) {
            int distA = a[0] * a[0] + a[1] * a[1];
            int distB = b[0] * b[0] + b[1] * b[1];
            return distA - distB;
        }
    });
    return Arrays.copyOfRange(points, 0, k);
}
```

**Time O(n log n), Space O(n)** for the copy (plus sort overhead). Correct, and — as with Kth Largest Element in an Array above — does more sorting work than the question actually requires.

### Approach 2 — Optimized: max-heap of size k, ordered by distance

```java
public int[][] kClosest(int[][] points, int k) {
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>(new Comparator<int[]>() {
        @Override
        public int compare(int[] a, int[] b) {
            int distA = a[0] * a[0] + a[1] * a[1];
            int distB = b[0] * b[0] + b[1] * b[1];
            return distB - distA;   // REVERSED: larger distance sorts "first" — makes this a max-heap by distance
        }
    });

    for (int[] point : points) {
        maxHeap.offer(point);
        if (maxHeap.size() > k) {
            maxHeap.poll();   // evict the current FARTHEST point
        }
    }

    int[][] result = new int[k][2];
    for (int i = 0; i < k; i++) {
        result[i] = maxHeap.poll();
    }
    return result;
}
```

**This is the first heap problem this week needing a hand-written `Comparator`** — natural integer ordering doesn't apply to an `int[]` point; "closeness" is a computed property. The anonymous-inner-class syntax here is not new: it's the exact technique Day 35 used for a custom `Comparator` on Merge k Sorted Lists' heap approach, applied to a different comparison rule.

**Why squared distance, not true Euclidean distance:** true distance requires a square root; comparing squared distances produces the **identical ordering**, because squaring (for non-negative inputs) and its inverse, square root, are both monotonically increasing — if `a² < b²` for non-negative `a, b`, then `a < b`, unconditionally. Skipping the square root avoids both unnecessary computation and floating-point imprecision, for zero cost in correctness. This is a generally useful technique any time only the *ordering* of distances matters, not their exact values.

**Why max-heap, not min-heap:** the goal is to keep the `k` **closest** points — so the heap needs to efficiently identify and evict the current **farthest** point among the ones being kept, every time the kept set exceeds size `k`. That's a max-heap's specialty (`peek()`/`poll()` on the largest), the mirror image of Day 54's Kth-Largest-via-min-heap logic, where the goal was keeping the largest values and evicting the smallest.

**Complexity: Time O(n log k), Space O(k).**

> 🔗 **Forward note, not required today:** the same Quickselect idea from Problem 3 applies here too — partition points by squared distance instead of by value, with target index `k` instead of `n-k` — reaching O(n) average. The mechanism is identical to what was just fully derived above; not repeated in full here since nothing about it changes beyond the comparison key.

**Edge cases:** `k` equal to the total number of points (every point is "closest," heap approach still works correctly, just never evicts); points genuinely equidistant from the origin (no tie-breaking is specified or needed — any valid set of k closest points is accepted).

---

# Part 3 — Extra Practice

## Extra Practice 1: Minimum Cost to Connect Sticks (LeetCode 1167, Medium) — Pattern: Min-Heap, Repeated Combine

**Statement:** Given stick lengths, repeatedly connect any two sticks into one (cost = sum of their lengths, and the new stick's length is that same sum) until one stick remains. Return the minimum total cost.

**Deferred here on purpose:** this problem mirrors Day 54's Last Stone Weight almost exactly — "repeatedly combine two, using a heap" — but Heaps' actual opening day correctly received no extras, so this landed here instead, once there's something to compare it against directly.

### Approach — min-heap, always combine the two smallest

```java
public int connectSticks(int[] sticks) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int stick : sticks) {
        minHeap.offer(stick);
    }

    int totalCost = 0;
    while (minHeap.size() > 1) {
        int first = minHeap.poll();
        int second = minHeap.poll();
        int combined = first + second;
        totalCost += combined;
        minHeap.offer(combined);
    }
    return totalCost;
}
```

### The contrast with Last Stone Weight, precisely

Last Stone Weight (Day 54) used a **max**-heap, combining the two **largest** stones each round. This problem uses a **min**-heap, combining the two **smallest** sticks each round. Both "repeatedly combine two via a heap" — the direction flips, and it's worth being able to say exactly why, not just that it does.

**Why smallest-first is optimal here — a cost-accounting argument:** every combination's cost gets **paid again** every time that combined stick is itself combined with something later — a stick combined early, and then combined again three more times, has its original length effectively counted four times in the total cost. Combining the two currently-smallest sticks first keeps small values "out of the way" — merged and folded into future combinations as early as possible, so their cost is paid the fewest additional times. Combining large sticks early, by contrast, means a large value gets carried forward and re-counted in every subsequent combination it participates in — needlessly inflating the total. This is a distinct exchange-argument shape from the three named back in Week 4 (domination, interval-swap, prefix-elimination) — call it a **cost-accounting exchange**: swapping any non-smallest-first choice for the smallest-first one can be shown to never increase the total, by tracking how many times each element's value ends up counted.

**Why Last Stone Weight doesn't use this same logic:** it isn't minimizing a total cost at all — it's simulating a specific destructive process (smash the two heaviest, discard the smaller, keep the difference) where the problem itself specifies which two stones interact each round. There's no optimization question to answer; the max-heap there is just efficiently maintaining "the two current largest," which is what the simulation's own rules require, not what a cost-minimization proof selects.

**Complexity: Time O(n log n), Space O(n)** — n-1 combinations, each O(log n) for the two polls and one offer.

**Edge cases:** a single stick (cost `0`, loop doesn't execute — no connections needed); two sticks (one combination, cost = their sum); all sticks equal length (correctly handled with no special case).

---

## Extra Practice 2: Kth Smallest Element in a Sorted Matrix (LeetCode 378, Medium) — Pattern: Heap of Size k / Binary Search on Value

**Statement:** Given an `n × n` matrix where every row and every column is sorted in ascending order, find the kth smallest element in the matrix.

**Why this is today's second extra:** it directly reinforces today's kth-order-statistic theme with a heap — but it also resolves an old thread. Day 32 introduced "Search a 2D Matrix II" (LC 240) specifically for contrast, mentioning an O(m+n) "staircase" technique without giving it full treatment or code. Approach 2 below is that exact technique, finally built out for real, applied to counting instead of searching for one value.

### Approach 1 — Min-heap seeded with each row's first element

```java
public int kthSmallest(int[][] matrix, int k) {
    int n = matrix.length;
    PriorityQueue<int[]> minHeap = new PriorityQueue<>(new Comparator<int[]>() {
        @Override
        public int compare(int[] a, int[] b) {
            return matrix[a[0]][a[1]] - matrix[b[0]][b[1]];
        }
    });

    for (int row = 0; row < n; row++) {
        minHeap.offer(new int[]{row, 0});   // (row, col) — each row's smallest element
    }

    int result = -1;
    for (int i = 0; i < k; i++) {
        int[] cell = minHeap.poll();
        result = matrix[cell[0]][cell[1]];
        if (cell[1] + 1 < n) {
            minHeap.offer(new int[]{cell[0], cell[1] + 1});   // this row's next-smallest becomes a new candidate
        }
    }
    return result;
}
```

**Why seeding with one entry per row (not the whole matrix) is enough:** each row is independently sorted, so a row's smallest *unconsidered* element is always its current leftmost cell — the heap only ever needs to hold each row's current "frontier" candidate, not every cell up front. **Complexity: O(n) to seed the heap, O(k log n) for the k extractions — O((n+k) log n) total, Space O(n).**

### Approach 2 — Binary search on value + Day 32's staircase count, resolved

**The reframe:** instead of generating candidates in order, binary search over the **range of possible answer values** (from `matrix[0][0]` to `matrix[n-1][n-1]`), and for each candidate value, count how many matrix elements are ≤ it. This is exactly Weeks 4–5's "binary search on the answer" framing (Day 28, formalized Day 33) — the tell is the same: "find the smallest/kth value such that a condition holds," with a monotonic feasibility check standing in for a sorted array to search directly.

**The count, in O(m+n) — Day 32's staircase, finally built:**

```java
public int kthSmallest(int[][] matrix, int k) {
    int n = matrix.length;
    int lo = matrix[0][0], hi = matrix[n - 1][n - 1];

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        int count = countLessOrEqual(matrix, mid);
        if (count < k) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    return lo;
}

private int countLessOrEqual(int[][] matrix, int target) {
    int n = matrix.length;
    int count = 0;
    int row = n - 1, col = 0;   // start at the BOTTOM-LEFT corner

    while (row >= 0 && col < n) {
        if (matrix[row][col] <= target) {
            count += row + 1;   // everything ABOVE this cell in the same column is also <= target
            col++;               // move right — nothing more to learn from this row
        } else {
            row--;                // move up — this row's remaining cells rightward are only bigger
        }
    }
    return count;
}
```

**Why the bottom-left starting point works:** columns are sorted ascending top-to-bottom, so if `matrix[row][col] <= target`, then every cell *above* it in that same column is also `<= target` (a smaller row index means a smaller value) — that's `row + 1` elements counted in one step, not one at a time. Moving right after that is safe because this row has nothing further to contribute at this or any earlier column. If instead `matrix[row][col] > target`, moving up is the only useful direction — this row's remaining cells to the right are all *larger* still (rows are sorted ascending left-to-right too), so there's nothing left to check in this row.

**Quick trace confirming the count:** matrix `[[1,5,9],[10,11,13],[12,13,15]]`, `target = 10`. Start `row=2,col=0` (`12`): `12 ≤ 10`? No → `row=1`. `row=1,col=0` (`10`): `10 ≤ 10`? Yes → `count += 2` (rows 0,1 in col 0: `1, 10`, both ≤ 10) → `count=2`, `col=1`. `row=1,col=1` (`11`): `> 10` → `row=0`. `row=0,col=1` (`5`): `≤ 10` → `count += 1` → `count=3`, `col=2`. `row=0,col=2` (`9`): `≤10` → `count += 1` → `count=4`, `col=3`, loop ends (`col=n=3`). **Final count = 4** — matching `{1, 5, 9, 10}`, exactly the four matrix values `≤ 10`.

**Complexity: O(n log(max−min)) time** — binary search over the value range, O(n) count per iteration (`m=n` here since the matrix is square). **Space O(1)**, beyond the matrix itself — no heap, no auxiliary storage. This asymptotically beats Approach 1 for large `k` (close to `n²`), since Approach 1's cost grows with `k` while this approach's cost depends only on the value range and `n`.

**Edge cases:** `k = 1` (the matrix's minimum, `matrix[0][0]`); `k = n²` (the maximum); duplicate values across the matrix (the binary search correctly converges on a value that's actually present, since `lo`/`hi` only ever move to matrix values, never an in-between non-existent value, by construction of the loop's convergence).

> 🔗 **Backward reference, resolved:** this is the O(m+n) "staircase" technique Day 32 named for LC 240 and explicitly declined to give full treatment to at the time. Today it gets used for real, adapted from "find one target" to "count everything ≤ a candidate" — same corner-walking mechanism, different question being asked of it.

---

# Section — Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** apply to 2 Tier B companies — from the target lists built on Days 50 and 54.

---

# Day 55 — Interview Questions

**1. State precisely what Day 12's Dutch National Flag partition and today's Quickselect partition share, and what differs.**

*Answer:* Both rearrange an array into ordered zones in a single pass around a pivot value. They differ in zone count (three, around fixed 0/1/2 values, for Dutch National Flag; two, around an element drawn from the array itself, for Quickselect) and in where the pivot comes from (a known constant vs. an actual array element).

---

**2. Why is Quickselect O(n) average while quicksort, built on the same partition step, is O(n log n)?**

*Answer:* Quicksort recurses into both sides of every partition (`T(n) = 2T(n/2) + O(n)`, resolving to O(n log n)). Quickselect only recurses into the one side containing the target index (`T(n) = T(n/2) + O(n)` on average), a geometric series summing to O(n) with no second branch to add a log n factor.

---

**3. What causes Quickselect's O(n²) worst case, and what's the standard mitigation?**

*Answer:* Consistently poor pivot choices (e.g., always picking the smallest or largest remaining element, which happens with a fixed last-element pivot on sorted or reverse-sorted input) shrink the search space by only one element per partition. Randomizing the pivot choice makes this worst case vanishingly unlikely rather than eliminating it, giving an expected O(n) regardless of input order.

---

**4. Why does K Closest Points to Origin need a hand-written Comparator, where earlier heap problems this week didn't?**

*Answer:* "Closeness" is a computed property of an `int[]` point, not something Java's natural integer ordering applies to directly — a custom comparison rule is required, using the same anonymous-inner-class technique introduced for a custom Comparator back on Day 35.

---

**5. Why is comparing squared distance instead of true distance both correct and preferable?**

*Answer:* Squaring and square-rooting are both monotonically increasing for non-negative numbers, so comparing squared values produces identical ordering to comparing true distances — while avoiding an unnecessary square root computation and any floating-point imprecision it could introduce.

---

**6. Why does K Closest Points to Origin need a max-heap, while Kth Largest Element in an Array's heap approach needs a min-heap?**

*Answer:* K Closest needs to efficiently find and evict the current farthest point whenever the kept set exceeds size k — a max-heap's specialty. Kth Largest needs to efficiently find and evict the current smallest of the kept top-k values — a min-heap's specialty. Both maintain a bounded window of size k; which extreme needs evicting determines which heap type is correct.

---

**7. Minimum Cost to Connect Sticks and Last Stone Weight both repeatedly combine two heap elements. Why does one use a min-heap and the other a max-heap?**

*Answer:* Minimum Cost to Connect Sticks is minimizing a total cost, where each combination's cost is paid again every time that combined value is combined further — combining the smallest values first (min-heap) minimizes how many times any value's cost gets re-counted. Last Stone Weight isn't optimizing a cost at all; it's simulating a specific process the problem defines (smash the two heaviest), which requires a max-heap purely to efficiently track the two current largest values as the rules dictate.

---

**8. In Kth Smallest Element in a Sorted Matrix's staircase count, why does moving right after a "≤ target" match count `row + 1` elements at once, rather than checking them individually?**

*Answer:* Columns are sorted ascending top-to-bottom, so if the current cell is ≤ target, every cell above it in the same column (a smaller row index, hence a smaller or equal value) is also guaranteed ≤ target — all `row + 1` of them can be counted in one step without individually checking each.

---

**9. How does the "binary search on value" approach for Kth Smallest in a Sorted Matrix connect to Weeks 4–5's Binary Search framing?**

*Answer:* It's a direct instance of "binary search on the answer" — searching over a range of candidate answer values (not array positions) using a monotonic feasibility check (count of elements ≤ candidate) rather than directly indexing into a sorted array, the same shape formalized on Day 33.

---

**10. Which of the two Kth Smallest in a Sorted Matrix approaches scales better as k approaches n², and why?**

*Answer:* The binary-search-on-value approach — its cost depends on the value range and n (via the O(n) count step), not on k at all, while the heap approach's cost grows directly with k (O(k log n) for the extractions).

---

## Daily Deliverable Check

- [ ] Kth Largest Element in an Array and K Closest Points to Origin solved, pushed — all three approaches to LC 215 (sort, heap, Quickselect) explainable, including the O(n) vs. O(n log n) distinction from quicksort.
- [ ] Extra Practice: Minimum Cost to Connect Sticks solved — the cost-accounting argument for why smallest-first is optimal stated precisely, in contrast to Last Stone Weight's largest-first rule.
- [ ] Extra Practice: Kth Smallest Element in a Sorted Matrix solved both ways — the staircase count mechanism, and its connection back to Day 32's unfinished LC 240 mention, both explainable.

---

## What Tomorrow Assumes You Already Know Cold

Day 56 assumes today's HashMap-plus-heap combination instinct is ready to click into place — Top K Frequent Elements is the canonical version of "count with a HashMap, then bound with a heap," a shape today's problems have been building toward without naming it explicitly as its own combination. It also assumes the min-heap-generates-candidates-in-order idea from Day 54's mechanism section (values coming out of a min-heap in sorted order, one at a time) is solid, since Ugly Number II reuses that directly, generating its own candidates rather than being handed a fixed input array.

**Next:** [Day 56 Resource Book](./Day56_Resource_Book.md) — Consolidation, and Heaps: Frequency and Scheduling.
