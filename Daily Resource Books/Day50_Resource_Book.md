# Day 50 Resource Book — Binary Search Trees Begin, and Kafka Consumers

**Series:** SDE-2 Interview Prep Resource Books (Week 8) · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 49](./Day49_Resource_Book.md) (Week 7) · **Next ▶:** [Day 51](./Day51_Resource_Book.md)
**Companion to:** Day 50 of `Week_08_Revised.md`

---

## Recap: what today is

Week 7 closed with Trees opened at 8 of an eventual 15 required problems (Maximum Depth, Invert, Same Tree, Symmetric Tree, Balanced, Diameter, Level Order, Right Side View — plus three extras: Subtree of Another Tree, Minimum Depth, Average of Levels). Every one of those problems worked on a **plain** binary tree — no assumption about how values relate to each other beyond "this is a tree." Today adds exactly one new idea on top of that: an ordering invariant. The `TreeNode` shape, the universal recursive template (base case for `null`, combine `left`/`right`), and the depth/height vocabulary from Day 46 are not being re-taught — they're the floor today stands on.

On the theory side, Day 48 built a Kafka **producer** — `todo-api` publishes a `TaskCreatedEvent` when a task is created. Today builds the **consumer** side. The plan calls this "yesterday's `TaskCreatedEvent`," which is worth a small correction: Day 49 was Week 7's consolidation day and touched no Kafka material, so the producer you're consuming from today is actually from Day 48, two days back, not literally yesterday. Nothing about today's work changes because of that — just flagging it so the reference is accurate if you go looking for the code.

## Learning Objectives

By the end of today, without notes:

1. State the BST ordering invariant precisely, and explain why validating it requires more than checking each node against only its immediate children.
2. Solve Validate Binary Search Tree and Kth Smallest Element in a BST, explaining why each reaches for a different traversal shape (bounds-passing DFS vs. inorder).
3. Explain why a plain BST's O(log n) claim is conditional on balance, not guaranteed by the ordering rule alone.
4. Explain Kafka consumer groups, partition-to-consumer assignment, and the auto-commit vs. manual-commit trade-off precisely enough to justify a choice.
5. Write a working `@KafkaListener` method and explain what problem it solves relative to the producer built on Day 48.

## Concept Dependency Map for Today

```
Day 46: TreeNode shape, universal recursive template (base case null, combine left/right)
Day 46: depth/height, O(h) space convention
        │
        ▼
NEW: Binary Search Tree — ordering invariant layered on the SAME shape
        │
        ├──▶ Problem 9: Validate Binary Search Tree (LC 98)
        │     needs: DFS template (Day 46) + a NEW technique — bounds threaded through recursion
        │
        └──▶ Problem 10: Kth Smallest Element in a BST (LC 230)
              needs: inorder traversal (named Day 46, first REAL use of inorder-for-a-reason here)

Extra Practice: BST Iterator (LC 173)
  needs: today's inorder-counting idea, made resumable via an explicit stack

Day 48: Kafka Fundamentals (topics, partitions, sequential disk appends)
Day 48: todo-api Kafka PRODUCER (TaskCreatedEvent)
        │
        ▼
NEW: Kafka CONSUMERS — consumer groups, partition assignment, offset commit strategy
        │
        ▼
Project: @KafkaListener logging receipt of TaskCreatedEvent
```

---

# Part 1 — Binary Search Trees

## Concept Card — Binary Search Trees

**What it is:** a binary tree with exactly one additional rule layered on top of the plain `TreeNode` shape you've had since Day 46: for every node, **every value in its left subtree is smaller than the node's own value, and every value in its right subtree is larger.** Not just the immediate children — the *entire* subtree in each direction.

**Why it matters:** that single global rule is what turns a tree into a search structure. At any node, comparing the target value to `node.val` tells you which subtree could possibly contain it — the other subtree is eliminated entirely, in O(1), with no need to look inside it. That's the same "eliminate half the search space per step" idea from Binary Search (Day 28), now applied to a tree shape instead of a contiguous array.

**The claim that needs a precise caveat:** search/insert/delete are **O(h)**, where h is the tree's height — and h is only O(log n) if the tree is reasonably balanced. Nothing about the BST rule itself guarantees balance. Insert the values `1, 2, 3, 4, 5` into an empty BST in that order, always going right (each new value is larger than everything already present), and you get a right-leaning chain — a linked list wearing a tree's clothes, with h = n. This is a real, common trap: **"BST" alone only buys you O(log n) in the average or balanced case; it does not buy it unconditionally.** Self-balancing variants (AVL trees, Red-Black trees) exist specifically to close this gap by enforcing a balance invariant on every insert/delete — and you've already met one: `TreeMap`, introduced Day 17, is a Red-Black tree internally, which is exactly why its O(log n) claim *doesn't* carry the same asterisk a bare BST's does. Today's BST is the unbalanced, "vanilla" version — worth knowing precisely which one you're claiming a complexity bound for.

**Interview signal:** the word "BST" in the problem statement, or a tree explicitly described as sorted in this left-smaller/right-larger structural sense (as opposed to an array being sorted).

---

## Problem 9: Validate Binary Search Tree (LeetCode 98, Medium) — Pattern: DFS with Boundaries

**Statement:** Given the root of a binary tree, determine if it is a valid BST.

### The wrong instinct, stated first

The tempting first move is to check each node only against its immediate children: `node.left.val < node.val` and `node.right.val > node.val`. This is wrong, and it's worth seeing exactly why before writing the correct version.

Consider this tree:

```
        10
       /  \
      5    15
          /  \
         6    20
```

Checked locally, node-by-node: `5 < 10` ✓. At node 15: `6 < 15` ✓, `20 > 15` ✓. Every *local* parent-child comparison passes. But this is **not** a valid BST — `6` sits in `10`'s right subtree, which means the global rule requires `6 > 10`. It doesn't. A local-only check has no way to catch this, because it never carries forward the constraint that `10`'s right subtree isn't just "greater than 15's immediate value" — it's constrained by *every* ancestor `10` is a right-child-direction away from.

### Approach — DFS with an inherited (min, max) range

```java
public boolean isValidBST(TreeNode root) {
    return validate(root, null, null);
}

private boolean validate(TreeNode node, Integer min, Integer max) {
    if (node == null) return true;   // an empty subtree trivially satisfies any range

    if ((min != null && node.val <= min) || (max != null && node.val >= max)) {
        return false;
    }

    return validate(node.left, min, node.val)      // left subtree: same lower bound, NEW upper bound = node.val
        && validate(node.right, node.val, max);    // right subtree: NEW lower bound = node.val, same upper bound
}
```

Each recursive call carries a valid `(min, max)` range for that node — not just "must be less than my parent," but the full range accumulated from *every* ancestor decision made to reach this point. When we descend left, the upper bound tightens to the parent's value (everything in a left subtree must stay below it); when we descend right, the lower bound tightens instead. `Integer` (boxed, nullable) rather than `int` is deliberate here — it gives a clean way to represent "no bound yet" at the root, without reaching for a sentinel value that might collide with a real node value.

**Trace against the invalid tree above** — `validate(15, min=10, max=null)`: 15 is in range. Recurse left: `validate(6, min=10, max=15)`. Now check: is `6 <= min(10)`? **Yes** → return `false`. The bound inherited from being in `10`'s right subtree (min=10) is exactly what a local-only check would never have seen, and it's exactly what catches the violation.

**Complexity: Time O(n)** — every node visited once. **Space O(h)** — the recursion stack, not a flat O(n): O(log n) for a balanced tree, O(n) for the skewed worst case from this very day's Concept Card.

**Edge cases:** a single node (trivially valid); duplicate values — LeetCode's version of this problem requires *strict* inequality (a node equal to an ancestor bound is invalid), which is exactly why the bounds check above uses `<=`/`>=` rather than `<`/`>`; an entirely null tree (vacuously valid, though LeetCode's constraints guarantee at least one node).

> ⚠️ **Common Mistake:** checking only `node.left.val < node.val < node.right.val` locally. This passes the two-level case above and silently fails on any tree where a violation happens two or more levels down a mixed left/right path — which is most real test cases beyond the trivial ones. If a proposed solution doesn't thread bounds through the recursion, it's the local-only bug.

> 💡 **Interview Insight:** the strongest opening move here is stating the wrong instinct out loud, showing a two-line counterexample, and then explaining why bounds must be *inherited*, not locally computed. This is a problem where showing you understand *why* the naive approach fails is worth more than arriving at working code — it's a very common follow-up question ("what if you only checked immediate children?") to ask even after a candidate produces the correct solution.

---

## Problem 10: Kth Smallest Element in a BST (LeetCode 230, Medium) — Pattern: Inorder Traversal

**Statement:** Given the root of a BST and an integer `k`, return the kth smallest value in the tree (1-indexed).

### Approach 1 — Brute force

Traverse the entire tree (any order), collect every value into a list, sort it, return the element at index `k - 1`.

```java
public int kthSmallestBruteForce(TreeNode root, int k) {
    List<Integer> values = new ArrayList<>();
    collect(root, values);
    Collections.sort(values);
    return values.get(k - 1);
}

private void collect(TreeNode node, List<Integer> values) {
    if (node == null) return;
    values.add(node.val);
    collect(node.left, values);
    collect(node.right, values);
}
```

**Time O(n log n)** (dominated by the sort), **Space O(n)** for the collected list. This throws away the one fact that makes this problem easy: it's a BST, not just any tree.

### Approach 2 — Optimized: inorder traversal, count as you go

**The fact this exploits:** an inorder traversal (left, node, right) of a BST visits every node in **ascending sorted order** — for free, no sorting step required. This is a direct consequence of the ordering invariant: visiting a node's left subtree first means visiting everything smaller first; visiting its right subtree last means visiting everything larger last. Day 46 named inorder traversal as one of the four standard orders on a plain tree, but on a plain tree it has no special meaning — this is the first problem where inorder's specific output order is the entire point, not just one traversal option among four.

```java
public int kthSmallest(TreeNode root, int k) {
    int[] count = {0};
    int[] result = {-1};
    inorder(root, k, count, result);
    return result[0];
}

private void inorder(TreeNode node, int k, int[] count, int[] result) {
    if (node == null || count[0] >= k) return;

    inorder(node.left, k, count, result);
    count[0]++;
    if (count[0] == k) {
        result[0] = node.val;
        return;
    }
    inorder(node.right, k, count, result);
}
```

A single-element `int[]` is used to hold `count` and `result` because a plain `int` local variable can't be mutated from inside a recursive call the way a field or array cell can — the exact same Java-specific reasoning Day 48 used for Diameter of Binary Tree's running-max. `count[0] >= k` at the top of the method is what lets the traversal **stop early** rather than needlessly visiting the rest of the tree after the answer is already found.

**Complexity: Time O(h + k)** in the best case for early termination, worst case **O(n)** if `k` is large or the tree is skewed such that most of the tree lies along the inorder path before the kth element — commonly stated as O(n) to be safe, since without a guarantee on where the kth-smallest node sits structurally, the traversal may need to touch most nodes. **Space O(h)** for the recursion stack.

**Edge cases:** `k` equal to the total node count (the maximum value); `k = 1` (the minimum — leftmost node); a single-node tree with `k = 1`.

> 🔑 **Key Takeaway:** inorder traversal on a BST is not just "one of four traversal orders" anymore — it's the mechanism that turns "kth smallest" from a sorting problem into a counting problem. Any time a BST problem's phrasing involves rank, order statistics, or "the nth something," inorder traversal is the first tool to reach for.

---

## Extra Practice 1: Binary Search Tree Iterator (LeetCode 173, Medium) — Pattern: Controlled Inorder Traversal

**Statement:** Design an iterator over a BST's nodes in ascending order. Implement `BSTIterator(TreeNode root)`, `boolean hasNext()`, and `int next()`. Average O(1) time per `next()` call; O(h) memory.

**Why this is today's extra:** Kth Smallest just used inorder traversal to answer *one* rank question and then stopped. This problem asks for the same ascending-order visitation, but as a **resumable, externally-controlled sequence** — the caller can interleave `next()` calls with other code, arbitrarily, rather than the traversal running start-to-finish in one call. That's a design problem built directly on top of today's inorder mechanism, not a new algorithm.

### Approach — explicit stack, pushing the left spine

The core idea: don't materialize the whole inorder sequence up front (that's the brute-force-adjacent move and costs O(n) space unconditionally, plus doesn't respect the "average O(1) per call" requirement in spirit). Instead, keep an explicit stack that always has, at its top, the next node to report — mirroring what the *call stack* would be doing inside a recursive inorder traversal at the moment it's about to visit that node.

```java
class BSTIterator {
    private final Deque<TreeNode> stack;

    public BSTIterator(TreeNode root) {
        stack = new ArrayDeque<>();
        pushLeftSpine(root);
    }

    private void pushLeftSpine(TreeNode node) {
        while (node != null) {
            stack.push(node);
            node = node.left;
        }
    }

    public int next() {
        TreeNode node = stack.pop();
        if (node.right != null) {
            pushLeftSpine(node.right);   // the next smallest values live down this subtree's left spine
        }
        return node.val;
    }

    public boolean hasNext() {
        return !stack.isEmpty();
    }
}
```

**Why this produces ascending order:** the leftmost unvisited node is always exactly what an inorder traversal would visit next, and it's always sitting at the top of the stack — either because it was pushed during initialization (the tree's true leftmost node), or because, after popping a node with a right child, that right child's own left spine was just pushed, making *its* leftmost descendant the new top.

**Complexity — the amortized argument:** a single `next()` call can, in the worst case, do O(h) work (if the popped node's right child has a long left spine to push). But across the iterator's **entire lifetime**, every node is pushed exactly once and popped exactly once — total work across all `next()` calls combined is O(n) for n nodes, which averages to O(1) per call. This is the same amortized-cost shape as `ArrayList.add()` (Day 3) and the total-pointer-movement argument for variable-size Sliding Window (Day 15) — a single expensive-looking operation is fine as long as the *total* cost across all operations stays linear.

**Space: O(h)** — the stack never holds more than one full root-to-leaf path's worth of nodes at a time.

**Edge cases:** calling `next()` when `hasNext()` is false is undefined by the problem (don't guard for it unless asked); a completely skewed tree (stack depth reaches O(n), the same worst case this whole book's Concept Card flagged).

> 💡 **Interview Insight:** this problem is frequently used to check whether "inorder traversal" is understood as a *mechanism* (an explicit stack simulating what recursion does implicitly) or just a memorized recursive snippet. Being able to write the iterative version unprompted, and explain why the amortized cost is O(1) rather than claiming a false worst-case O(1) per call, is the actual signal being tested.

---

# Part 2 — Kafka Consumers

## Consumer Groups and Partition Assignment

Day 48 established that a Kafka topic is split into **partitions** for parallelism, and that Kafka guarantees ordering *only within a single partition*, never across an entire topic. Today's new piece: a **consumer group** is a named set of consumer instances that coordinate to read a topic together. Kafka's guarantee is precise and worth stating exactly: **within a single consumer group, each partition is read by exactly one consumer at a time.** Two consumers in the *same* group never both process the same partition simultaneously — this is what makes horizontal scaling safe: add more consumer instances to a group (up to the partition count) and each one picks up a disjoint subset of partitions, with no coordination logic to write yourself. Add a *second, differently-named* consumer group, though, and it reads the entire topic independently, from the start — group membership, not the topic itself, is what determines who's "already seen" a message.

## Offset Committing: What Actually Breaks on a Crash

An **offset** is a partition-local sequence number marking how far a consumer has read. Committing an offset is how a consumer records "I'm done through here" — so that if it restarts, it resumes from the right place instead of reprocessing everything or skipping ahead.

**Auto-commit** (the default): the consumer periodically commits the latest offset it has *read*, on a timer, independent of whether processing that message actually finished. This is where a specific, concrete failure mode lives: if the consumer crashes **after** an auto-commit fires but **before** it finishes processing the message that commit covered, that message is gone on restart — the consumer believes it already handled everything up to that offset, and resumes past it. This is a silent data loss risk, not a crash or an exception; nothing about it announces itself.

**Manual commit**: the consumer explicitly commits only after processing genuinely completes. This closes the gap above — a crash mid-processing means the offset was never committed, so the same message gets redelivered on restart. That fixes the loss, but introduces a different requirement: **the processing logic must be idempotent** (safe to run twice with the same input), because "redelivered and reprocessed" is now a real, expected scenario rather than an edge case to dismiss.

| | Auto-commit | Manual commit |
|---|---|---|
| Risk on crash | Can silently **lose** a message (committed before processing finished) | Can **redeliver** a message (processing ran, crash happened before commit) |
| Requires idempotent processing? | Not strictly, but doesn't protect against loss either way | **Yes** — redelivery is a designed-for case, not a bug |
| Simplicity | Simpler, fewer moving parts | More control, more responsibility |

**Neither is unconditionally "safer"** — auto-commit trades a loss risk for simplicity; manual commit trades that loss risk for a duplicate-delivery risk that the application itself must now handle correctly. Today's `todo-api` listener uses the framework default (effectively auto-commit-equivalent behavior for a simple logging consumer) since nothing here is processing anything non-idempotent yet — worth knowing this is a deliberate simplification, not the production-grade choice for every future listener this project might add.

## `@KafkaListener`

Spring Kafka's `@KafkaListener` annotation is the consumer-side mirror of the `KafkaTemplate.send()` call Day 48's producer used. Annotating a method and specifying a topic is enough for the framework to wire up a consumer, subscribe it to that topic, and invoke the method once per message received — the same reflection-based wiring mechanism (Day 34) that powers `@RestController`/`@GetMapping`, applied here to message consumption instead of HTTP routing.

```java
@Component
public class TaskEventListener {

    private static final Logger log = LoggerFactory.getLogger(TaskEventListener.class);

    @KafkaListener(topics = "task-created-events", groupId = "todo-api-consumer-group")
    public void handleTaskCreated(TaskCreatedEvent event) {
        log.info("Received TaskCreatedEvent: taskId={}, title={}", event.getTaskId(), event.getTitle());
    }
}
```

`groupId` is what determines the partition-assignment behavior described above — every instance of `todo-api` you might ever run with this same `groupId` coordinates as one logical consumer group, splitting partition reads between them rather than each independently reprocessing everything.

> 🔗 **Backward reference:** this is the direct consumer-side counterpart to Day 48's `TaskCreatedEvent` producer — same event class, same topic, opposite direction. If Day 48's producer isn't publishing successfully, today's listener has nothing to receive; verify the producer side first if `handleTaskCreated` never fires.

---

# Section — Project Block

**Repository:** `todo-api`. **Task:** implement the `@KafkaListener` from the exercise above.

**Definition of done:** creating a task (via the existing endpoint) logs **both** the producer's send confirmation (Day 48) and the listener's receipt log line — confirm both appear, in order, in the application log for a single task-creation request. This is the concrete verification that the full produce → consume loop actually works end to end, not just that each half compiles independently.

# Section — Career Block

**LinkedIn Post 12:** Kafka partitions and consumer groups, with a simple diagram. A draft to adapt into your own voice:

> Built out the consumer side of a Kafka pipeline today — the part that actually reads what gets published. The detail that stuck with me: partitions, not the topic as a whole, are Kafka's unit of both parallelism *and* ordering. Two consumers in the same group split the partitions between them for throughput; but that only works because each individual partition still gets read by exactly one consumer at a time, which is also exactly what keeps ordering intact *within* that partition. Scale and ordering aren't in tension here — they're solved by the same mechanism.

**Networking:** identify 3 Target Tier B companies — add them to whatever tracker you've been using since Week 1's networking blocks, ready for the applications and outreach this plan schedules over the coming days.

---

# Day 50 — Interview Questions

**1. State the BST ordering invariant precisely — not "left is smaller," but the full rule.**

*Answer:* For every node, every value in its entire left subtree is smaller than the node's value, and every value in its entire right subtree is larger — not just the immediate children, the whole subtree in each direction.

---

**2. Why does checking only `node.left.val < node.val < node.right.val` fail to validate a BST correctly?**

*Answer:* It only checks one level down. A node can locally look fine against its immediate parent while still violating a constraint from a higher ancestor (e.g., a node in a right subtree that's smaller than the root two levels up) — a purely local check has no way to see that inherited bound.

---

**3. Why does the bounds-passing DFS for Validate BST use `Integer` instead of `int` for the min/max parameters?**

*Answer:* `Integer` can be `null`, giving a clean way to represent "no bound yet" at the root (and along the unbounded side at every level) without needing a sentinel value that might collide with a legitimate node value.

---

**4. Is a BST's O(log n) search guaranteed by the ordering rule alone?**

*Answer:* No. The ordering rule alone permits a fully skewed tree (e.g., inserting strictly increasing values) with height O(n). O(log n) requires the tree to also be balanced — a separate property a plain BST doesn't enforce on its own. Self-balancing variants like Red-Black trees (which back `TreeMap`, Day 17) exist specifically to guarantee it.

---

**5. Why does inorder traversal of a BST visit nodes in ascending order, while it has no particular meaning on a plain binary tree?**

*Answer:* Inorder visits left subtree, then the node, then right subtree. On a BST specifically, "everything in the left subtree" is guaranteed smaller and "everything in the right subtree" is guaranteed larger by the ordering invariant — so visiting left-node-right necessarily visits values in increasing order. A plain tree has no such guarantee, so the same traversal order carries no sorted meaning there.

---

**6. In Kth Smallest Element in a BST, why is a single-element `int[]` used for `count` and `result` instead of plain `int` locals?**

*Answer:* A plain local `int` can't be mutated by an inner recursive call in a way that's visible to the caller — the same limitation Diameter of Binary Tree (Day 48) hit with its running max. A single-element array (or an instance field) gives a mutable reference the recursion can update.

---

**7. What does BST Iterator's `next()` do differently from Kth Smallest's inorder traversal, mechanically?**

*Answer:* Kth Smallest runs one recursive inorder pass to completion (or until the kth element is found) in a single call. BST Iterator uses an explicit stack that mirrors what the recursive call stack would look like mid-traversal, so the "next" node can be produced one at a time across arbitrarily many separate `next()` calls, rather than all at once.

---

**8. Why is BST Iterator's `next()` described as "average O(1)," not worst-case O(1)?**

*Answer:* A single call can do up to O(h) work if the popped node has a right child with a long left spine to push. But across the iterator's entire lifetime, every node is pushed and popped exactly once — total work is O(n), which averages to O(1) per call even though individual calls vary.

---

**9. In Kafka, what exactly does "consumer group" control?**

*Answer:* It determines partition assignment and duplicate-avoidance: within one consumer group, each partition is read by exactly one consumer at a time, so instances in the same group split the work. A different, separately-named consumer group reads the entire topic independently from scratch — group identity, not the topic, determines what's "already been read."

---

**10. What specifically can go wrong with Kafka auto-commit, and when?**

*Answer:* If the consumer crashes after an offset auto-commits but before it finishes processing the message that commit covers, that message is effectively lost — on restart, the consumer resumes past an offset it never actually finished handling, with no error or exception marking the loss.

---

**11. If you switch to manual commit to fix that risk, what new requirement does it introduce?**

*Answer:* Processing must become idempotent. Manual commit fixes message loss by only committing after processing finishes, but a crash between finishing processing and committing causes the same message to be redelivered on restart — the application must handle reprocessing the same message safely.

---

**12. What mechanism does `@KafkaListener` share with `@RestController`/`@GetMapping` from Day 34?**

*Answer:* Both are wired up via reflection at startup — the framework inspects annotated methods at runtime and connects them to their trigger (an HTTP route for `@GetMapping`, a topic subscription for `@KafkaListener`) without either being called directly by name anywhere in your own code.

---

## Daily Deliverable Check

- [ ] Validate Binary Search Tree and Kth Smallest Element in a BST solved, pushed to `dsa-java/trees/`, with the bounds-passing technique and the inorder-for-rank technique both explainable from scratch.
- [ ] Extra Practice: BST Iterator solved — the amortized O(1) argument can be stated without hesitation.
- [ ] `@KafkaListener` live in `todo-api`, verified end to end — both the Day 48 producer log line and today's consumer log line appear for a single task creation.
- [ ] LinkedIn Post 12 published. 3 Target Tier B companies identified and logged.

---

## What Tomorrow Assumes You Already Know Cold

Day 51 assumes today's bounds-passing technique is fully reflexive — Lowest Common Ancestor of a BST reuses the same "the ordering invariant tells you which direction to go" instinct, just applied to a decision (which single direction to recurse into) rather than a validity check (both directions, bounded). It also assumes the plain-tree DFS template from Day 46 is solid enough to extend into a genuinely new technique: Construct Binary Tree from Preorder and Inorder Traversal introduces divide-and-conquer for the first time with real depth, building directly on the recursive comfort this week has been reinforcing.

**Next:** [Day 51 Resource Book](./Day51_Resource_Book.md) — BST and Construction, and Spring Cloud Config.
