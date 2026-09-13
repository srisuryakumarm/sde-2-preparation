# Day 48 — Tree DFS with Global State, and Kafka Fundamentals

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 47 Resource Book](Day47_Resource_Book.md)
**Next ▶:** [Day 49 Resource Book](Day49_Resource_Book.md)
**Companion to:** Day 48 of `Week_07_Revised.md`

---

## Recap

Days 46–47 built and extended a recursive template where a node's answer is computed purely from its own value and what its children's recursive calls *returned*. Today adds a second recursive shape on top of that: a helper that can **short-circuit** the instant it finds a problem anywhere below (Balanced Binary Tree), and a helper that needs to update some **running best-so-far that isn't the value being returned upward at all** (Diameter of Binary Tree). Both problems also expose the same underlying trap — recomputing the same subtree's height over and over — which is worth seeing once, explicitly, since it explains why the naive version of *both* problems is slower than it looks.

## Learning Objectives

By the end of today, without notes:

1. Solve Balanced Binary Tree using the −1 sentinel short-circuit, and explain exactly what redundant work it avoids compared to the naive version.
2. Solve Diameter of Binary Tree, and explain — in Java specifically — why a plain local variable can't hold the running maximum, and what the alternatives are.
3. Solve Minimum Depth of Binary Tree, and state precisely why the naive formula that works for Maximum Depth gives the wrong answer here.
4. Explain what a Kafka partition is for, and why sequential disk appends give Kafka its throughput.

## Concept Dependency Map

```
Day 46/47 — recursive template: a node's return value is built purely
from node.val + children's return values
        │
        ▼
NEW SHAPE 1: short-circuit on a sentinel value
  └─ LC 110 Balanced Binary Tree — helper returns height NORMALLY,
     but returns -1 the instant imbalance is found anywhere below,
     and every caller checks for -1 before doing anything else
        │
        ▼
NEW SHAPE 2: track a running best SEPARATE from the value returned
  └─ LC 543 Diameter of Binary Tree — helper returns HEIGHT upward
     (for the parent's own combination), while updating a value that
     is never returned at all — the running max diameter
        │
        ▼
EXTRA: LC 111 Minimum Depth — Day 46's Maximum Depth, inverted; a
node with only ONE child is NOT a leaf, breaking the naive formula
        │
        ▼
NEW: Kafka Fundamentals — topics as append-only logs, partitions for
parallelism, consumer groups split the work, sequential disk appends
drive throughput
        │
        ▼
Project: todo-api gets a Spring Kafka producer, TaskCreatedEvent
```

---

## Problem 5: Balanced Binary Tree (LeetCode 110, Easy) — Pattern: DFS, Bottom-Up Height Check

**Statement:** Given a binary tree, determine if it is height-balanced — for every node, the height difference between its left and right subtrees is never more than 1.

### Approach 1 — Brute force: recompute height at every node

```java
public static boolean isBalancedBruteForce(TreeNode root) {
    if (root == null) return true;
    int leftHeight = height(root.left);
    int rightHeight = height(root.right);
    if (Math.abs(leftHeight - rightHeight) > 1) return false;
    return isBalancedBruteForce(root.left) && isBalancedBruteForce(root.right);
}

private static int height(TreeNode node) {
    if (node == null) return 0;
    return 1 + Math.max(height(node.left), height(node.right));
}
```

Directly transcribes the definition: check this node's own balance, then recursively check both subtrees the same way. Correct — but `height()` is called independently at *every* node, and each call re-walks that node's entire subtree from scratch. For a tree of height `h`, the root's `height()` call alone costs O(n); then `isBalancedBruteForce` recurses into both children, each calling `height()` again on their own (smaller) subtrees — and this repeats at every level. For a **balanced** tree, this totals **O(n log n)** (each of the O(log n) levels does O(n) combined height-work). For a **skewed** tree, it degrades to **O(n²)** (the same redundant-recomputation shape as an O(n²) nested loop, just expressed recursively) — worth stating both, since "worst case" and "typical case" genuinely diverge here.

### Approach 2 — Optimized: combine the height check and the height computation into one pass

```java
public static boolean isBalanced(TreeNode root) {
    return checkHeight(root) != -1;
}

private static int checkHeight(TreeNode node) {
    if (node == null) return 0;

    int leftHeight = checkHeight(node.left);
    if (leftHeight == -1) return -1;          // imbalance already found below — stop immediately

    int rightHeight = checkHeight(node.right);
    if (rightHeight == -1) return -1;          // imbalance found on the right side

    if (Math.abs(leftHeight - rightHeight) > 1) return -1;   // THIS node is unbalanced

    return 1 + Math.max(leftHeight, rightHeight);              // still balanced — return real height
}
```

**The fix:** instead of computing height and checking balance as two separate passes, `checkHeight` does both *at the same time*, on the way back up from a single traversal. Its return value carries two different meanings depending on what happened below: a non-negative number means "still balanced so far, and here's my actual height," while `-1` is a **sentinel** meaning "already broken somewhere below — stop checking, just propagate the failure upward." Every caller checks for `-1` before doing anything else with the returned value, which is exactly what makes this short-circuit — the instant imbalance is found anywhere, no further height computation happens on the rest of the tree at all.

**Trace — an unbalanced tree**, `[1, 2, 2, 3, 3, null, null, 4, 4]` (a left-heavy tree, imbalance four levels deep):

```
              1
            /   \
           2     2
          / \
         3   3
        / \
       4   4
```

`checkHeight` recurses all the way down to the `4`s (leaves, height 1 each) → the `3` nodes: left child height 1, right child height 1, `|1-1|=0`, balanced here, return `2`. Back at the left `2`: its left child (the `3` with the two `4`s) returned height `2`; its right child (the other `3`, a leaf) returned height `1`. `|2 - 1| = 1` — still within tolerance, balanced here too, return `3`. Back at the root: left child (`2`) returned `3`; right child (`2`, a leaf) returned `1`. `|3 - 1| = 2` — **exceeds 1**, return `-1`. `isBalanced` sees `-1` from the top-level call and returns `false`. Correct — and notice the imbalance was only actually detected once, at the root, even though four levels of the tree were walked to get there; nothing was ever *rechecked*.

**Complexity: Time O(n) — every node visited exactly once, Approach 2. Space: O(h)** call stack, same shape as every DFS tree problem so far.

**Edge cases:** empty tree (`checkHeight(null) = 0`, never `-1`, so `isBalanced(null) = true` — vacuously balanced); a single node (trivially balanced); imbalance at the very bottom of a deep tree (the short-circuit still has to walk down to find it before it can propagate back up — short-circuiting saves *redundant recomputation*, not the initial discovery walk itself).

> 💡 **Interview Insight:** If you start with the brute-force version, name its redundant-recomputation cost explicitly *before* being asked to optimize — "this recomputes height at every node instead of computing it once and reusing it" is precisely the diagnosis an interviewer is listening for, and it directly motivates the sentinel fix rather than the fix appearing out of nowhere.

---

## Problem 6: Diameter of Binary Tree (LeetCode 543, Easy) — Pattern: DFS with a Global Max

**Statement:** Given the root of a binary tree, return the length of the diameter — the longest path between any two nodes, measured in edges. This path may or may not pass through the root.

### Approach 1 — Brute force: same redundant-recomputation trap as Problem 5

```java
public static int diameterBruteForce(TreeNode root) {
    if (root == null) return 0;
    int throughRoot = height(root.left) + height(root.right);
    return Math.max(throughRoot,
           Math.max(diameterBruteForce(root.left), diameterBruteForce(root.right)));
}
// height() as defined in Problem 5
```

At every node, compute the diameter "through" it as `height(left) + height(right)`, then recurse to check every other node the same way, taking the global max. Structurally identical to Problem 5's brute force — `height()` gets recomputed from scratch at every node — and it degrades the same way: **O(n log n)** for a balanced tree, **O(n²)** worst case for a skewed one.

### Approach 2 — Optimized: one pass, height and diameter computed together

```java
public class Solution {
    private int maxDiameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        height(root);
        return maxDiameter;
    }

    private int height(TreeNode node) {
        if (node == null) return 0;
        int leftHeight = height(node.left);
        int rightHeight = height(node.right);
        maxDiameter = Math.max(maxDiameter, leftHeight + rightHeight);
        return 1 + Math.max(leftHeight, rightHeight);
    }
}
```

**The key distinction — what gets returned vs. what gets tracked:** `height()` still *returns* height upward, exactly as in Problem 5, because that's what a parent call needs to compute its own height. But the diameter *through* the current node — `leftHeight + rightHeight` — is a different quantity that no parent needs returned to it; it only ever needs to be compared against a single running best-so-far. That's tracked in `maxDiameter`, updated as a side effect on the way back up, entirely separate from the value the function returns.

**Why a local variable can't hold `maxDiameter` in Java, specifically:** if `maxDiameter` were declared as a local variable inside `diameterOfBinaryTree` and `height` were a separate method, `height` would have no way to reach back and update it — Java doesn't give a nested method access to modify a caller's local variable the way some languages' closures do. The two idiomatic Java fixes are: (1) an **instance field**, as above — simplest, and what most interview solutions use; or (2) threading a **mutable holder** through the recursion explicitly, most commonly a single-element array (`int[] maxDiameter = new int[1];`, updated via `maxDiameter[0] = ...`), which avoids adding a field to the class at the cost of slightly noisier code. Both are correct; the instance-field version is simpler and is what to reach for by default.

**Trace:** `root = [1, 2, 3, 4, 5]` (a left-heavy tree — `2` has children `4` and `5`; `3` is a leaf).

```
        1
       / \
      2   3
     / \
    4   5
```

`height(4)`: leaf, `leftHeight=0, rightHeight=0`, `maxDiameter = max(0, 0+0) = 0`, returns `1`.
`height(5)`: same, returns `1`.
`height(2)`: `leftHeight=height(4)=1`, `rightHeight=height(5)=1`, `maxDiameter = max(0, 1+1) = 2`, returns `1+max(1,1)=2`.
`height(3)`: leaf, `maxDiameter` unchanged (`0+0=0 < 2`), returns `1`.
`height(1)`: `leftHeight=height(2)=2`, `rightHeight=height(3)=1`, `maxDiameter = max(2, 2+1) = 3`, returns `1+max(2,1)=3`.

Final `maxDiameter = 3` — the path `4 → 2 → 1 → 3` (or `5 → 2 → 1 → 3`), three edges. Correct. Note that the diameter-defining path does **not** pass through the node with the single highest individual height contribution in isolation — it's found by comparing `leftHeight + rightHeight` at *every* node, which is exactly why the check has to happen at every single node, not just the root.

**Complexity: Time O(n) — Approach 2, one pass. Space: O(h)** call stack, plus O(1) for the instance field (constant regardless of tree size).

**Edge cases:** a single node (diameter `0` — no path exists between two *different* nodes); a tree where the longest path doesn't pass through the root at all (the trace above — the check at *every* node, not just the root, is what catches this); a completely skewed tree (diameter equals `n - 1`, correctly, since the "longest path" degenerates to the single chain).

> 💡 **Interview Insight:** The single most common wrong-but-close answer here is returning `leftHeight + rightHeight` directly from the root only, instead of checking it at every node and tracking a separate max. Stating explicitly, before coding, that the diameter-defining path can be anywhere in the tree — not necessarily through the root — heads that mistake off before it happens.

---

## Extra Practice: Minimum Depth of Binary Tree (LeetCode 111, Easy) — Pattern: DFS

**Statement:** Given a binary tree, return its minimum depth — the number of nodes along the shortest path from the root down to the nearest leaf.

### The naive formula — and exactly where it breaks

Day 46's Maximum Depth used `1 + max(maxDepth(left), maxDepth(right))`. The tempting, symmetric-looking guess for minimum depth:

```java
// WRONG for a node with only one child
public static int minDepthWrong(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.min(minDepthWrong(root.left), minDepthWrong(root.right));
}
```

Trace this against a right-skewed chain `[2, null, 3, null, 4, null, 5, null, 6]` (each node has only a right child, five levels deep): at the root, `root.left == null`, so `minDepthWrong(root.left) = 0` — and `1 + min(0, minDepthWrong(root.right))` evaluates to `1 + min(0, anything) = 1`, since `0` wins the `min` no matter what the right side computes. This claims the minimum depth is **1**, as if the root itself were a leaf. **It is not** — the root has a child, so by the problem's own definition (a leaf has *no* children), the root doesn't qualify, and the true minimum depth here is 5, straight down the only chain that exists.

**The bug:** treating a `null` child exactly like Maximum Depth does (as contributing `0`, then letting `min` pick it) silently treats "this side doesn't exist" as "this side is a zero-cost path to a leaf" — which is true for *maximum* depth (an absent side can never be the *longest* path, so contributing 0 and losing to `max` is harmless) but false for *minimum* depth (an absent side would incorrectly *win* the `min` comparison, even though it isn't a real path to an actual leaf at all).

### Correct approach

```java
public static int minDepth(TreeNode root) {
    if (root == null) return 0;
    if (root.left == null) return 1 + minDepth(root.right);
    if (root.right == null) return 1 + minDepth(root.left);
    return 1 + Math.min(minDepth(root.left), minDepth(root.right));
}
```

A node with exactly one child is explicitly **not** a leaf and must continue down its *only* existing side — the two special-case checks handle that directly, before ever reaching the general `min` case, which now only fires when both children genuinely exist and both are real candidates.

**Trace:** the same right-skewed chain, `2 → 3 → 4 → 5 → 6` (five nodes, each with only a right child except the last).
`minDepth(2)`: `left==null` → `1 + minDepth(3)`.
`minDepth(3)`: `left==null` → `1 + minDepth(4)`.
`minDepth(4)`: `left==null` → `1 + minDepth(5)`.
`minDepth(5)`: `left==null` → `1 + minDepth(6)`.
`minDepth(6)`: both `null` → base case, `0`.
Unwinding: `minDepth(6)=0` (wait — `6` is a leaf, both children null, general case fires: `1+min(minDepth(null),minDepth(null))=1+min(0,0)=1`) → `minDepth(5)=1+1=2` → `minDepth(4)=1+2=3` → `minDepth(3)=1+3=4` → `minDepth(2)=1+4=5`.

Final: **5** — correctly the full chain length, not `1`.

**Complexity: Time O(n) worst case** (a skewed tree must be walked in full, same as Maximum Depth) **— though unlike Maximum Depth, a BFS version of this problem can stop the instant the first leaf is found at any level, which for a tree where the shallowest leaf is much closer than the deepest can be meaningfully faster in practice, despite an identical O(n) worst-case bound.** **Space: O(h)** recursive, same shape as every DFS problem this week.

**Edge cases:** the one-child trap above (the entire point of this problem); an empty tree (`0`); a perfectly balanced tree (minimum and maximum depth coincide, and the naive formula would have accidentally given the right answer — which is exactly why this trap is easy to miss in casual testing against "normal-looking" trees).

> 💡 **Interview Insight:** This problem is a near-guaranteed follow-up to Maximum Depth precisely because the naive formula looks so plausible. Volunteering the one-child counterexample unprompted, the moment you write `min` instead of `max`, is a strong, specific signal — far stronger than writing the correct version without ever mentioning why the "obvious" version is wrong.

---

# Part 2 — Kafka Fundamentals

### Prerequisites (confirmed)

None directly — a fresh theory thread, like Docker and JUnit 5 were. It does assume `todo-api`'s existing structure (TaskService/TaskController, live since Day 34) as the thing today's producer gets attached to.

### Topics, Partitions, and Why Kafka Is Fast

A Kafka **topic** is an append-only log — records are only ever added to the end, never modified or deleted in place. A topic is split into **partitions**, each an independent, ordered log of its own; splitting a topic across partitions is what allows multiple consumers to read (and multiple producers to write) **in parallel**, since each partition can be handled independently.

**Producers** write records to a topic (Kafka assigns — or the producer specifies — which partition a given record lands in, often by hashing a key, so related records consistently land in the same partition and preserve their relative order). **Consumers** read records from a topic; a **consumer group** is a set of consumers cooperatively splitting a topic's partitions between them, so each partition is read by only one consumer *within that group* at a time — this is what lets you scale message processing horizontally by adding more consumer instances to a group, up to one per partition.

**Why sequential disk appends drive Kafka's throughput:** a mechanical or even SSD-backed disk is dramatically faster at sequential writes (appending to the end of an existing file) than at random-access writes (seeking to arbitrary positions) — Kafka's append-only design means every write is sequential by construction, which is the direct mechanical reason it can sustain very high write throughput despite persisting every message to disk. This is the same sequential-vs-random-access performance gap Day 40's B-tree/index discussion touched on from the read side; here it's the same physical principle applied to writes.

### Common Mistakes

- ⚠️ **Assuming more partitions is always better** — more partitions means more parallelism, but also more open file handles and more overhead per partition; partition count is a real capacity-planning decision, not a "set it to something large and forget it" default.
- ⚠️ **Assuming Kafka guarantees global ordering across an entire topic** — it only guarantees ordering **within a single partition**. Two records in different partitions have no guaranteed relative order, which is exactly why a producer's partitioning key choice matters for anything where order is significant.
- ⚠️ **Confusing a consumer group with a broadcast** — within one consumer group, each message goes to exactly one consumer (the partitions are divided up). Multiple *different* consumer groups, on the other hand, each independently receive every message — that's how Kafka supports both "distribute the work" and "notify every interested subsystem" from the same topic.

---

# Project Block Guide (1.5 hrs)

**Repository:** `todo-api`. Run Kafka locally via Docker (Day 43's mechanism, applied to a new image), use the CLI to create a topic, and confirm messages flow with a console producer and console consumer before touching any application code. Then add Spring Kafka to the project and configure a producer that publishes a `TaskCreatedEvent` whenever a task is created via the existing `TaskController`/`TaskService` flow.

**Definition of done:** creating a task through the API produces a message visible in the console consumer, confirming the event actually reaches Kafka end to end.

---

# Career Block Guide (1 hr)

### LinkedIn

Engagement day — 20 minutes commenting on 3–5 posts in your network.

### Networking

Send connection requests to the 3 engineering managers you reviewed yesterday.

---

# Day 48 — Interview Questions

**Q1. What specifically does Balanced Binary Tree's brute-force approach recompute redundantly, and what's its complexity?**
It calls `height()` independently at every node, and each call re-walks that node's entire subtree from scratch — the same subtree's height ends up computed many times over. This is O(n log n) for a balanced tree and degrades to O(n²) for a skewed one.

**Q2. What does the `-1` sentinel mean in the optimized Balanced Binary Tree solution, and why does every caller need to check for it before proceeding?**
`-1` means "an imbalance was already found somewhere below this point" — as opposed to a non-negative return, which is a genuine height with no imbalance found yet. Every caller checks for `-1` first specifically so the check can short-circuit: once imbalance is found anywhere, the failure just propagates straight up without computing any further heights on the rest of the tree.

**Q3. In Diameter of Binary Tree, what does the `height` helper return, versus what does it track as a side effect — and why are those two different things?**
It returns the node's height upward, because that's what the parent's own height computation needs. It separately tracks `maxDiameter` — the best `leftHeight + rightHeight` seen at any node — as a side effect, because no parent ever needs that value returned to it; it only ever needs comparing against a single running best.

**Q4. Why can't a plain local variable hold Diameter's running max in Java, and what are the two standard fixes?**
A local variable declared in the outer method isn't reachable for a separate recursive helper method to update — Java doesn't give a nested method write access to a caller's local variable. The two fixes are an instance field on the class (simplest, most common), or explicitly threading a mutable holder — typically a single-element array — through the recursive calls.

**Q5. Why does the diameter-defining path not necessarily pass through the root, and what does that imply about where the check has to happen?**
The longest path can lie entirely within one subtree, never touching the root at all. That's why `leftHeight + rightHeight` must be checked and compared against the running max at *every* node during the traversal, not just once at the root.

**Q6. Walk through exactly why `1 + min(minDepth(left), minDepth(right))` gives the wrong answer for a node with only one child.**
A `null` child contributes `0` to the `min` comparison, and since `0` is the smallest possible value, it wins the `min` regardless of what the real (non-null) side computes — incorrectly reporting depth `1`, as if the current node itself were a leaf. But a node with one child is explicitly not a leaf by the problem's own definition, so its minimum depth must continue down its one existing side instead.

**Q7. Why does the naive minimum-depth formula happen to give the correct answer on a perfectly balanced tree, even though it's wrong in general?**
A perfectly balanced tree has no nodes with exactly one child — every internal node has either zero or two children — so the specific case the naive formula mishandles (a single-child node) never actually occurs, and the bug never surfaces on that shape of input.

**Q8. What is a Kafka partition for, and what guarantee does Kafka make — and not make — about ordering?**
A partition is an independent, ordered log that a topic is split into, enabling parallel reads and writes across multiple consumers/producers. Kafka guarantees ordering only *within* a single partition — there's no guaranteed relative order between records in different partitions of the same topic.

**Q9. Why are sequential disk appends the mechanical reason Kafka achieves high throughput?**
Sequential writes (appending to the end of an existing file) are dramatically faster than random-access writes on both mechanical and SSD-backed disks. Kafka's append-only log design means every write is sequential by construction, which is what lets it sustain high throughput while still durably persisting every message to disk.

**Q10. What does a Kafka consumer group actually split among its members, and what happens if two separate consumer groups read the same topic?**
A consumer group splits a topic's *partitions* among its member consumers, so each partition is read by only one consumer within that group at a time. Two independent consumer groups each receive every message on the topic independently — one group's consumption has no effect on what another group sees.

---

# Daily Deliverable Check

- [ ] Balanced Binary Tree (LC 110) and Diameter of Binary Tree (LC 543) solved — the redundant-recomputation trap in the brute force, and the sentinel/running-max fixes, explained out loud — pushed.
- [ ] Minimum Depth of Binary Tree (LC 111) solved as extra practice, with the one-child trap explicitly demonstrated against Maximum Depth.
- [ ] Kafka running locally via Docker; topic created and verified end-to-end via console producer/consumer before touching application code.
- [ ] Spring Kafka producer live in `todo-api`, publishing `TaskCreatedEvent` on task creation, verified in the console consumer.
- [ ] 20 minutes of LinkedIn engagement completed.
- [ ] Connection requests sent to yesterday's 3 EMs.

---

## What Tomorrow Assumes You Already Know Cold

Day 49 assumes Day 4's `ArrayDeque`-as-`Queue` and today's overall comfort with the `TreeNode` shape, but explicitly **does not** depend on today's short-circuit-sentinel or running-max techniques — Binary Tree Level Order Traversal is a fresh, iterative, queue-driven approach rather than a further extension of today's recursive machinery. It's the first genuinely new *traversal strategy* since Day 46 introduced DFS.
