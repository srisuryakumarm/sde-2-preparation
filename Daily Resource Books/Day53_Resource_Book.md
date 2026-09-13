# Day 53 Resource Book — Trees Capstone: The Median Fix, and Divide-and-Conquer Reviewed

**Series:** SDE-2 Interview Prep Resource Books (Week 8) · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 52](./Day52_Resource_Book.md) · **Next ▶:** [Day 54](./Day54_Resource_Book.md)
**Companion to:** Day 53 of `Week_08_Revised.md`

---

## Recap: what today is

This is Trees' capstone day — one problem, genuinely hard, closing the pattern at 15 required problems (up from 11 in the original plan), plus a review exercise that asks you to state, precisely, the one thing that separated yesterday's two Lowest Common Ancestor problems. Median of Two Sorted Arrays is the problem the original plan promised on Day 21 and then never actually delivered anywhere in the remaining 99 days — it sat out until now specifically because it needs the divide-and-conquer intuition Day 51 just built, applied to a binary search that divides on a computed *partition point* rather than an array index.

**No extra practice today**, and this one is worth explaining rather than just noting: the partition-based binary search this problem uses is a genuinely singular technique — there isn't a comparable-difficulty sibling problem that reinforces this *exact* mechanism the way, say, Construct Binary Tree from Inorder/Postorder reinforced Preorder/Inorder yesterday. Reaching for a loosely-related "binary search on the answer" problem just to have an extra would teach pattern-matching to the wrong signal (a different technique wearing a similar name) rather than actually reinforcing this one. Better to leave today at one problem, done rigorously, than pad it with something that doesn't actually build the same muscle.

## Learning Objectives

By the end of today, without notes:

1. Explain why merging both arrays, even efficiently, can't meet this problem's required O(log(min(m,n))) bound — and why that forces a fundamentally different approach.
2. State the partition correctness condition precisely, and prove (not just assert) that satisfying it identifies the median.
3. Solve Median of Two Sorted Arrays with the partition-based binary search, including why the search runs over the *smaller* array specifically.
4. State, in one sentence each, exactly why BST LCA can skip a subtree computationally while general-tree LCA cannot — and explain why that's the entire difference, not a difference in algorithm sophistication.
5. Recite Trees' full 15-problem-plus-5-extra classification, by sub-pattern, without notes.

## Concept Dependency Map for Today

```
Day 28-33: Binary Search — "on the input" vs. "on the answer," both fully closed
Day 51: Divide and conquer, formalized — a divide step requiring real computed work
        │
        ▼
Problem 15: Median of Two Sorted Arrays (LC 4)
  needs: BOTH of the above — binary search over a COMPUTED partition point,
         not an array index and not a value range (a third flavor, distinct from Weeks 4-5's two)

Day 51: LCA of a BST (ordering invariant → compute one direction)
Day 52: LCA of a Binary Tree, general (no invariant → search both, combine)
        │
        ▼
Trees, Reviewed — the ordering invariant is the ENTIRE difference, stated precisely
        │
        ▼
Trees CLOSES: 15/15 required + 5 extra = 20 distinct total
```

---

# Part 1 — Median of Two Sorted Arrays

## Problem 15: Median of Two Sorted Arrays (LeetCode 4, Hard) — Pattern: Binary Search / Divide and Conquer

**Statement:** Given two sorted arrays `nums1` (size `m`) and `nums2` (size `n`), return the median of the combined sorted array. Required time complexity: **O(log(min(m,n)))**.

### Why the obvious approaches don't meet the bound

**Approach 1 — Brute force: concatenate and sort.** Combine both arrays, sort the result, read off the middle element(s).

```java
public double findMedianSortedArraysBruteForce(int[] nums1, int[] nums2) {
    int[] merged = new int[nums1.length + nums2.length];
    System.arraycopy(nums1, 0, merged, 0, nums1.length);
    System.arraycopy(nums2, 0, merged, nums1.length, nums2.length);
    Arrays.sort(merged);

    int mid = merged.length / 2;
    if (merged.length % 2 == 1) {
        return merged[mid];
    }
    return (merged[mid - 1] + merged[mid]) / 2.0;
}
```

**Time O((m+n) log(m+n)), Space O(m+n).** Correct, but ignores that both inputs are already sorted — sorting them again is wasted work, and the complexity is nowhere near the required bound.

**Approach 2 — A meaningfully better approach: two-pointer merge, without building the full array.** Both arrays are already sorted, so a proper merge (Day 35's Merge Two Sorted Lists technique, adapted from linked lists to arrays) reaches the median position directly, without a sort step and without materializing the whole merged array — just walk both arrays with two pointers, advancing whichever is currently smaller, until the target position(s) are reached.

```java
public double findMedianSortedArraysLinear(int[] nums1, int[] nums2) {
    int m = nums1.length, n = nums2.length;
    int total = m + n;
    int targetIndex = total / 2;

    int i = 0, j = 0;
    int prev = -1, curr = -1;

    for (int count = 0; count <= targetIndex; count++) {
        prev = curr;
        if (i < m && (j >= n || nums1[i] <= nums2[j])) {
            curr = nums1[i++];
        } else {
            curr = nums2[j++];
        }
    }

    return (total % 2 == 1) ? curr : (prev + curr) / 2.0;
}
```

**Time O(m+n), Space O(1)** — a real improvement, no sort, no auxiliary array. **Still not O(log(min(m,n))).** This is worth sitting with for a moment: O(m+n) is linear in the combined input size; the required bound is logarithmic in the *smaller* array's size alone — a fundamentally different growth rate, not just a constant-factor improvement. Reaching it requires **not looking at most of the data at all**, which a merge — by definition, touching every element up to the target position — structurally cannot do.

### Approach 3 — Optimized: binary search on a partition point

**The reframe:** don't merge anything. Instead, ask: is there a way to split `nums1` into a left part and a right part, and `nums2` into a left part and a right part, such that combining *both* left parts gives exactly the left half of what the full merge would have produced — without ever performing that merge?

**The two conditions a correct partition must satisfy:**

1. **Size condition:** the combined left part must have `⌈(m+n)/2⌉` elements — enough that, for an odd total, the left part's maximum *is* the median, and for an even total, the left part's maximum and the right part's minimum straddle it.
2. **Value condition:** every element in the combined left part must be ≤ every element in the combined right part. This is what makes it a *valid* partition of the true merged order, not just an arbitrary split.

**Binary search over `i`, the partition point in the smaller array.** Let `A` be the smaller array (size `m'`), `B` the larger (size `n'`) — swap if needed so this always holds. For a candidate split `i` elements of `A` into the left part, the partner split `j` in `B` is **forced**, not independently searched: `j = ⌈(m'+n')/2⌉ - i`. This is exactly why the search only needs to range over `A`'s size, not both arrays independently — fixing `i` fully determines `j`.

Define, for a candidate `(i, j)`:
- `left1 = A[i-1]` if `i > 0`, else `-∞`
- `right1 = A[i]` if `i < m'`, else `+∞`
- `left2 = B[j-1]` if `j > 0`, else `-∞`
- `right2 = B[j]` if `j < n'`, else `+∞`

The partition is correct exactly when **`left1 ≤ right2` AND `left2 ≤ right1`**. If `left1 > right2`, `A`'s left part reaches too far right (contains something too large) — `i` needs to shrink. If `left2 > right1`, the opposite — `i` needs to grow. This monotonic relationship (moving `i` in one direction always trends toward satisfying the condition) is exactly what makes binary search valid here, the same "provably monotonic, therefore binary-searchable" argument from every Binary Search problem back in Weeks 4–5, now applied to a *partition index* rather than an array value.

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    if (nums1.length > nums2.length) {
        return findMedianSortedArrays(nums2, nums1);   // ensure nums1 (A) is the SMALLER array
    }

    int m = nums1.length, n = nums2.length;
    int left = 0, right = m;
    int halfLen = (m + n + 1) / 2;

    while (left <= right) {
        int i = left + (right - left) / 2;   // partition point in the SMALLER array
        int j = halfLen - i;                  // FORCED, not independently searched

        int left1  = (i == 0) ? Integer.MIN_VALUE : nums1[i - 1];
        int right1 = (i == m) ? Integer.MAX_VALUE : nums1[i];
        int left2  = (j == 0) ? Integer.MIN_VALUE : nums2[j - 1];
        int right2 = (j == n) ? Integer.MAX_VALUE : nums2[j];

        if (left1 <= right2 && left2 <= right1) {
            if ((m + n) % 2 == 1) {
                return Math.max(left1, left2);
            }
            return (Math.max(left1, left2) + Math.min(right1, right2)) / 2.0;
        } else if (left1 > right2) {
            right = i - 1;   // i too big — shrink
        } else {
            left = i + 1;    // i too small — grow
        }
    }

    throw new IllegalStateException("Input arrays must be sorted");   // unreachable for valid sorted input
}
```

**Worked trace:** `nums1 = [1, 3]`, `nums2 = [2]`. `nums1.length(2) > nums2.length(1)`, so swap: `A = [2]` (m=1), `B = [1, 3]` (n=2). `halfLen = (1+2+1)/2 = 2`. `left=0, right=1`.

- `i = 0`: `j = 2 - 0 = 2`. `left1 = -∞` (i=0). `right1 = A[0] = 2`. `left2 = B[1] = 3` (j=2>0). `right2 = +∞` (j=2=n). Check: `left1(-∞) ≤ right2(+∞)` ✓, but `left2(3) ≤ right1(2)`? **False.** `i` too small → `left = 1`.
- `i = 1`: `j = 2 - 1 = 1`. `left1 = A[0] = 2` (i=1>0). `right1 = +∞` (i=1=m). `left2 = B[0] = 1` (j=1>0). `right2 = B[1] = 3` (j=1<2). Check: `left1(2) ≤ right2(3)` ✓, `left2(1) ≤ right1(+∞)` ✓. **Both hold — valid partition.**
- Total `m+n = 3`, odd → median = `max(left1, left2) = max(2, 1) = 2`.

**Confirm directly:** merged `[1,2,3]` → median `2`. Matches.

**Why binary searching `A` (the smaller array) specifically gives O(log(min(m,n))):** the search range is `[0, m]` where `m` is the smaller array's size — the loop's iteration count is bounded by `log(m)`, i.e., `log(min(m,n))`. Searching over the larger array instead would still be *correct* (the same partition logic works symmetrically), just slower: `O(log(max(m,n)))`. The swap-to-ensure-`nums1`-is-smaller step at the top isn't a minor tidiness choice — it's what the required complexity bound actually depends on.

**A subtlety worth having ready:** could a real array value that happens to equal `Integer.MIN_VALUE` or `Integer.MAX_VALUE` be confused with the sentinel? No — even if a genuine array value equals `Integer.MIN_VALUE`, treating it as "acts like `-∞`" for comparison purposes is harmless, since nothing in a valid `int` array can be smaller than it anyway; the sentinel and a genuine boundary value are behaviorally indistinguishable in every comparison this algorithm performs.

**Complexity: Time O(log(min(m,n))), Space O(1)** — no auxiliary array, no recursion, a fixed handful of integer comparisons per iteration.

**Edge cases:** one array completely empty (handled correctly — `i` or `j` can legitimately reach `0` or the array's full length, both of which the sentinel logic accounts for); arrays of very different sizes (exactly why the swap-to-smaller step exists); duplicate values across the two arrays (the `≤`/`≥` comparisons throughout are non-strict specifically to handle this correctly).

> ⚠️ **Common Mistake:** binary searching over the *larger* array "because it doesn't matter which one, they're symmetric." The algorithm is still correct either way — but the required O(log(min(m,n))) bound specifically depends on searching the smaller one. On a problem this size-sensitive, stating that choice out loud, unprompted, is worth doing before writing any code.

> 💡 **Interview Insight:** this is a famously hard problem, and being visibly proceeding-by-approach — brute force, then the O(m+n) improvement, then naming *why* a merge structurally can't reach O(log(min(m,n))) before introducing the partition idea — demonstrates the reasoning process an interviewer is actually evaluating far more than arriving at the final code quickly. If the partition proof is shaky under pressure, walking through the worked trace above out loud is a legitimate, honest way to demonstrate the mechanism even while still building full confidence in the general argument.

---

# Part 2 — Trees, Reviewed: BST Traversal vs. General Tree DFS

The plan's own reflection prompt: *why does BST's Lowest Common Ancestor (Day 51) get to skip checking both subtrees, while general Binary Tree LCA (Day 52) can't?* Write one sentence each — here's what those two sentences should actually say.

**BST LCA (Day 51):** because the BST's ordering invariant lets you *compute*, in O(1) at every node, which single subtree could possibly contain both target values — so the other subtree is skipped without ever being searched.

**General LCA (Day 52):** because a plain tree carries no information about which subtree a target value lives in, so both children must actually be searched, and their results combined, to determine where the paths to each target diverge.

**The synthesis, worth being able to state as crisply as the two sentences above:** this isn't a difference in algorithmic sophistication — general-tree LCA isn't "the harder version" of the same idea. It's that a BST carries *extra information* (the ordering invariant) that a plain tree simply doesn't have, and that extra information is what turns a search into a computation. Being able to state that — not just that the two solutions look different, but precisely *why* they have to — is worth more than having independently solved both problems, which is exactly the plan's own framing for today.

---

# Part 3 — Trees Closes: The Full 15 + 5

Trees opened Day 46 at 0/15 and closes today — 15 required problems (up from 11 in the original plan) plus 5 extra practice problems, 20 distinct total, spanning DFS, BFS, BST traversal, construction, general-tree LCA, and serialization.

| Day | LC # | Problem | Type | Sub-pattern |
|---|---|---|---|---|
| 46 | 104 | Maximum Depth of Binary Tree | Required | DFS — Postorder Combine |
| 46 | 226 | Invert Binary Tree | Required | DFS — Swap and Recurse |
| 47 | 100 | Same Tree | Required | DFS — Two-Tree Comparison |
| 47 | 101 | Symmetric Tree | Required | DFS — Mirrored Comparison |
| 47 | 572 | Subtree of Another Tree | Extra | DFS — Composes isSameTree |
| 48 | 110 | Balanced Binary Tree | Required | DFS — Sentinel Short-Circuit |
| 48 | 543 | Diameter of Binary Tree | Required | DFS — Running Max Outside Return |
| 48 | 111 | Minimum Depth of Binary Tree | Extra | DFS — One-Child-Not-a-Leaf Trap |
| 49 | 102 | Binary Tree Level Order Traversal | Required | BFS — queue.size() Level Isolation |
| 49 | 199 | Binary Tree Right Side View | Required | BFS — Last Node per Level |
| 49 | 637 | Average of Levels in Binary Tree | Extra | BFS — Aggregate Instead of Collect |
| 50 | 98 | Validate Binary Search Tree | Required | BST — DFS with Boundaries |
| 50 | 230 | Kth Smallest Element in a BST | Required | BST — Inorder Traversal |
| 50 | 173 | BST Iterator | Extra | BST — Controlled Inorder (Stack) |
| 51 | 235 | Lowest Common Ancestor of a BST | Required | BST — Ordering-Based Direction |
| 51 | 105 | Construct Binary Tree from Preorder/Inorder | Required | Divide and Conquer |
| 51 | 106 | Construct Binary Tree from Inorder/Postorder | Extra | Divide and Conquer, Mirrored |
| 52 | 236 | Lowest Common Ancestor of a Binary Tree | Required | DFS — Postorder Combine (general) |
| 52 | 297 | Serialize and Deserialize Binary Tree | Required | DFS Preorder + Null Markers |
| 53 | 4 | Median of Two Sorted Arrays | Required | Binary Search / Divide and Conquer |

**This also closes the loop the original plan left open on Day 21** — Median of Two Sorted Arrays was promised there and never delivered anywhere across the original plan's remaining 99 days. It landed here, deliberately, once the recursive divide-and-conquer intuition from Construct Binary Tree (Day 51) existed to draw on — not as a delay, but as the correct dependency order.

**Deferred on purpose, not forgotten:** House Robber III and Binary Tree Maximum Path Sum — both genuinely Tree problems by shape — are reserved for Dynamic Programming, later in the series, once DP's own "best answer here, built from the best answer one step back" thinking is in place to layer on top of tree recursion. Pulling either forward into today's ladder would mean re-deriving them from scratch later anyway, once DP framing exists, rather than getting the full value of both techniques combined.

---

# Section — Project Block

**Repository:** none new. Confirm all 15 Tree solutions are pushed and organized in `dsa-java/trees/`, one folder per problem, including today's Median of Two Sorted Arrays and the week's extras (BST Iterator, Construct from Inorder/Postorder).

# Section — Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** casual check-in with your accountability partner — a good moment for this specifically, given today closes a major pattern; a quick "here's what I closed out this week" message is genuine, low-effort momentum-sharing, not just a check-the-box task.

---

# Day 53 — Interview Questions

**1. Why can't an efficient merge of the two arrays meet this problem's required O(log(min(m,n))) bound, even done optimally at O(m+n)?**

*Answer:* O(m+n) is linear in the combined size; the required bound is logarithmic in the smaller array alone — a fundamentally different growth rate. Reaching it requires not examining most of the input at all, which a merge, by definition touching every element up to the target position, structurally cannot do.

---

**2. State the two conditions a correct partition of both arrays must satisfy.**

*Answer:* A size condition — the combined left part must hold exactly `⌈(m+n)/2⌉` elements. A value condition — every element in the combined left part must be ≤ every element in the combined right part.

---

**3. Why does fixing the partition index `i` in the smaller array also fully determine `j`, the partition index in the larger array?**

*Answer:* The size condition fixes the total number of elements required in the combined left part. Once `i` elements are taken from the smaller array, the remaining slots needed to reach that total are forced — `j = halfLen - i` — leaving nothing left to search independently.

---

**4. What does it mean if `left1 > right2` during the search, and what's the correct response?**

*Answer:* It means the smaller array's left part reaches too far right — it contains a value larger than something that should be in the combined right part. `i` needs to shrink, so the search moves its right boundary down.

---

**5. Why does the algorithm binary search over the smaller array specifically, rather than either array?**

*Answer:* The search range is bounded by the array being searched — searching the smaller array bounds the iteration count by `log(min(m,n))`, matching the required complexity. Searching the larger array would still be correct, just slower: `O(log(max(m,n)))`.

---

**6. For an odd combined length, why is `max(left1, left2)` the median, with no need to look at either right value?**

*Answer:* The partition is sized so the combined left part has exactly one more element than the right part when the total is odd — meaning the left part's own maximum value IS the middle element of the full combined order.

---

**7. Could a real array value equal to `Integer.MIN_VALUE` or `MAX_VALUE` break the sentinel logic?**

*Answer:* No — even if a genuine value equals the sentinel, treating it as behaving like -∞ or +∞ is harmless, since nothing in a valid int array can be smaller than `Integer.MIN_VALUE` or larger than `Integer.MAX_VALUE` anyway; every comparison the algorithm performs behaves identically either way.

---

**8. In one sentence: why does BST LCA get to skip checking both subtrees, while general-tree LCA can't?**

*Answer:* The BST's ordering invariant lets you compute, in O(1), which single subtree could contain both targets, so the other is never searched; a plain tree carries no such information, so both children must actually be searched and combined.

---

**9. Is general-tree LCA a "harder algorithm" than BST LCA?**

*Answer:* No — it's not a difference in sophistication. A BST simply carries extra information (the ordering invariant) that a plain tree doesn't have, and that extra information is what turns a search into a computation. Without it, searching both sides is the correct and necessary approach, not an inferior one.

---

**10. Why does today add no extra practice problem, when every other "thin" pattern this week got one?**

*Answer:* The partition-based binary search here is a genuinely singular technique with no comparable-difficulty sibling that reinforces the exact same mechanism — reaching for a loosely-related "binary search on the answer" problem would reinforce pattern-matching on surface similarity rather than the actual technique, which does more harm than good.

---

## Daily Deliverable Check

- [ ] Median of Two Sorted Arrays solved — all three approaches (brute force, linear merge, partition binary search) explainable, with the partition correctness proof stated without hesitation. Trees ladder fully complete at 15 required + 5 extra = 20 distinct.
- [ ] BST vs. general-tree LCA reflection written — both one-sentence answers and the synthesis, not just the mechanics of each algorithm.
- [ ] All 15 Tree solutions confirmed pushed and organized in `dsa-java/trees/`.

---

## Week 8 So Far

Trees closes today at 15 required + 5 extra = 20 distinct problems, spanning DFS, BFS, BST traversal, construction (divide and conquer), general-tree LCA, and serialization. Tomorrow opens an entirely new pattern — Heaps — starting completely fresh, the same "close one pattern, open the next, same week" shape Week 4 used for Greedy & Intervals.

## What Tomorrow Assumes You Already Know Cold

Day 54 assumes `PriorityQueue`'s basic API (`.offer()`, `.poll()`, `.peek()`) is already familiar — it was named on Day 17 and actually used on Day 26 (Meeting Rooms II's min-heap of end times) — but does **not** assume the underlying array-backed binary heap mechanism has ever been taught in depth, because it hasn't; that's tomorrow's genuinely new material. It does not depend on anything from today's Median of Two Sorted Arrays or the Trees pattern generally — Heaps is a clean break, the first pattern this series has opened immediately after fully closing the previous one within the same week.

**Next:** [Day 54 Resource Book](./Day54_Resource_Book.md) — Heaps Begin, and Frequency Patterns.
