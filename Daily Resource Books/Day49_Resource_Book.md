# Day 49 (Sunday) — Consolidation, and Tree BFS

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 48 Resource Book](Day48_Resource_Book.md)
**Next ▶:** [Day 50 Resource Book](Day50_Resource_Book.md)
**Companion to:** Day 49 of `Week_07_Revised.md`

---

## Recap

Days 46–48 covered every DFS shape Trees needs for now — single-tree combination, two-tree comparison, and short-circuit/global-state recursion. Today introduces the pattern's other half entirely: **BFS**, level by level, using Day 4's `ArrayDeque` as a queue instead of the call stack. It doesn't extend Day 48's techniques — it's a fresh, iterative approach applied to the same `TreeNode` shape Day 46 built. Today also closes the week: a self-check, the week's DSA total, and what Week 8 inherits.

## Learning Objectives

By the end of today, without notes:

1. Solve Binary Tree Level Order Traversal, and explain precisely why capturing `queue.size()` before the inner loop is what isolates one level from the next.
2. Solve Binary Tree Right Side View as a direct extension of the same scaffold.
3. State, from memory, the full Trees-so-far ladder and what's deferred to Week 8.
4. Recite this week's actual DSA total against the plan's own claim, and explain any gap.

## Concept Dependency Map

```
Day 4 — Queue (FIFO), via ArrayDeque
        │
Day 46 — TreeNode shape
        │
        ▼
NEW: BFS on trees — a queue holds "every node at the CURRENT level,
and nothing else," achieved by capturing queue.size() BEFORE the
inner loop drains exactly that many nodes, even though their
children are being added to the SAME queue during that drain
        │
        ├─▶ LC 102 Level Order Traversal — collect each level as its
        │    own list
        ├─▶ LC 199 Right Side View — same scaffold, keep only the
        │    LAST node visited per level
        └─▶ LC 637 Average of Levels (EXTRA) — same scaffold,
             aggregate (sum ÷ count) instead of collecting
        │
        ▼
Week 7 Consolidation — Stacks closed (14), Trees opened (11 of 15,
continuing Week 8) — what Week 8 inherits
```

---

### Self-Check (15 min)

Before today's new material, pick one Stack problem from this week — Asteroid Collision, Basic Calculator II, Largest Rectangle in Histogram, or Basic Calculator are all good tests — and solve it cold, no notes, no looking back at Days 43–45. Largest Rectangle in Histogram is the sharpest test of the four: if you can still state its monotonic-increasing invariant and get the boundary-width arithmetic right without checking, the whole week's Stack content is genuinely load-bearing, not just recognized when prompted.

---

# Binary Tree BFS

## Why `queue.size()` Has to Be Captured First

The core trick behind every problem today: capture how many nodes are in the queue *before* draining any of them, and use that captured count — not the queue's live, constantly-changing size — to bound the inner loop.

```java
Queue<TreeNode> queue = new ArrayDeque<>();
queue.offer(root);
while (!queue.isEmpty()) {
    int levelSize = queue.size();     // exactly the current level's node count, captured NOW
    for (int i = 0; i < levelSize; i++) {
        TreeNode node = queue.poll();
        // ... process node ...
        if (node.left != null) queue.offer(node.left);    // adds to the SAME queue
        if (node.right != null) queue.offer(node.right);   // that levelSize was captured from
    }
    // at this exact point, the queue contains ONLY the next level's nodes
}
```

At the top of each `while` iteration, the queue holds *exactly* the nodes of the current level — no more, no less, because every previous iteration only ever added a level's children after that level was already fully accounted for. Capturing `queue.size()` at that moment freezes that count; the `for` loop then processes precisely that many nodes, even though `queue.offer()` calls inside the loop are actively growing the queue with the *next* level's nodes at the same time. Without capturing the count upfront — using `queue.size()` directly as the loop's live bound, or worse, just draining until empty — the newly-added children would get swept into the same pass as their own parents, collapsing every level into one.

## Problem 7: Binary Tree Level Order Traversal (LeetCode 102, Medium) — Pattern: BFS

**Statement:** Given the root of a binary tree, return the values of its nodes as if visiting level by level, left to right, with each level as its own list.

### Approach 1 — Optimized: BFS with a queue

```java
public static List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

The scaffold above, directly: each pass through the outer `while` produces exactly one level's list.

### Approach 2 — A genuinely distinct alternative: DFS with explicit level tracking

```java
public static List<List<Integer>> levelOrderDFS(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    dfs(root, 0, result);
    return result;
}

private static void dfs(TreeNode node, int level, List<List<Integer>> result) {
    if (node == null) return;
    if (level == result.size()) {
        result.add(new ArrayList<>());     // first time reaching this level — create its list
    }
    result.get(level).add(node.val);
    dfs(node.left, level + 1, result);
    dfs(node.right, level + 1, result);
}
```

This is a real, correct alternative, not a novelty: pass the current depth down through the recursion, and grow `result` by one list the first time each new depth is reached. `result.size() == level` is precisely "have we seen this depth before" — the moment it's false, a preorder (Day 46's naming) DFS still produces the correct level-grouped output, because every node still gets placed at the depth its own recursive call carries. **This confirms level-order grouping isn't inherently tied to BFS** — it's tied to knowing each node's depth, which DFS can track just as well via a parameter. BFS is the more natural fit because a queue's processing order *is* already level-by-level; DFS needs the extra `level` parameter to reconstruct the same grouping explicitly.

**Trace (BFS):** tree `[3, 9, 20, null, null, 15, 7]`.

| queue at top of iteration | `levelSize` | level collected | queue after |
|---|---|---|---|
| `[3]` | 1 | `[3]` | `[9, 20]` |
| `[9, 20]` | 2 | `[9, 20]` | `[15, 7]` |
| `[15, 7]` | 2 | `[15, 7]` | `[]` |

Result: `[[3], [9, 20], [15, 7]]`. Correct.

**Complexity: Time O(n) — every node enqueued and dequeued exactly once, both approaches. Space: O(w)** for BFS, where `w` is the tree's maximum width (the most nodes the queue ever holds at once — up to ~n/2 at the last level of a balanced tree); **O(h)** call stack for the DFS alternative, plus O(n) for the output either way (unavoidable — the output itself has one entry per node).

**Edge cases:** empty tree (`root == null` → empty list, both approaches' first check); a single node (one level, one-element list); a completely skewed tree (BFS's queue never holds more than 1 node at a time — its width is 1 — so BFS space drops to O(1) extra beyond the output on this shape, while DFS's call stack still costs O(n), an interesting reversal of the usual DFS-favors-skewed-trees intuition from Day 46).

> 💡 **Interview Insight:** Volunteering the DFS-with-level-tracking alternative unprompted — after leading with BFS as the natural fit — signals that you understand level-order grouping is about depth-tracking, not about BFS specifically. That distinction is exactly what the edge case above is testing: a skewed tree flips which approach is actually cheaper in practice.

---

## Problem 8: Binary Tree Right Side View (LeetCode 199, Medium) — Pattern: BFS

**Statement:** Given the root of a binary tree, imagine standing on its right side — return the values of the nodes you can see, ordered top to bottom.

### Approach — Same scaffold, keep only the last node per level

```java
public static List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            if (i == levelSize - 1) result.add(node.val);   // the LAST node processed at this level
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
    }
    return result;
}
```

**Why "the last node processed at each level" is exactly "the rightmost visible node":** the inner loop always processes a level's nodes in left-to-right order, because children are offered left-then-right, which means they're dequeued in that same left-to-right order at the next level down. The last node processed at any given level is therefore always the rightmost one at that level — precisely what someone standing to the right of the tree would see first (and only) at that depth. This is Problem 7's identical scaffold with one line changed: instead of collecting every value into a level list, only the value at `i == levelSize - 1` is kept.

**Trace:** same tree, `[3, 9, 20, null, null, 15, 7]`. Level `[3]` → last (and only) is `3`. Level `[9, 20]` → last is `20`. Level `[15, 7]` → last is `7`. Result: `[3, 20, 7]`.

**Complexity: Time O(n). Space: O(w)**, identical shape to Problem 7 — this problem changes *what's kept*, not the traversal itself.

**Edge cases:** a tree that's left-heavy at some level (e.g., a node with only a *left* child at the rightmost position of its level) — the scaffold still correctly reports that left child, since it's still the last one *processed* at that level, even though it's structurally on the "left" side of its own immediate parent; empty tree (empty result); a single node (`[root.val]`).

---

## Extra Practice: Average of Levels in Binary Tree (LeetCode 637, Easy) — Pattern: BFS, Aggregation

**Statement:** Given the root of a binary tree, return the average value of the nodes at each level.

### Approach — Same scaffold again, aggregate instead of collect

```java
public static List<Double> averageOfLevels(TreeNode root) {
    List<Double> result = new ArrayList<>();
    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        long sum = 0;                    // long — guards against overflow summing many/large node values
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            sum += node.val;
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add((double) sum / levelSize);
    }
    return result;
}
```

This is the third problem today built on the identical BFS scaffold — Problem 7 collected a list per level, Problem 8 kept one value per level, this one reduces a level to a single running sum, divided by the level's node count once the level is fully drained. Given how directly it reuses the scaffold, it's genuinely fast to add once Level Order Traversal is already solid — worth noting explicitly, since the marginal cost of extra reps drops sharply once a scaffold this reusable is in hand.

`sum` is declared `long`, not `int`, for the same overflow-awareness reason established back on Day 10/11: a level with many nodes, each near `Integer.MAX_VALUE`, could overflow a 32-bit accumulator silently.

**Trace:** `[3, 9, 20, null, null, 15, 7]`. Level `[3]`: sum `3`, avg `3.0`. Level `[9, 20]`: sum `29`, avg `14.5`. Level `[15, 7]`: sum `22`, avg `11.0`. Result: `[3.0, 14.5, 11.0]`.

**Complexity: Time O(n). Space: O(w)** for the queue, O(1) extra for the running sum (versus O(levelSize) if a level's values were collected into a list first and averaged afterward — a small but real reason to aggregate inline rather than reusing Problem 7's list-collecting version and post-processing it).

**Edge cases:** a single node (`[root.val.toDouble()]`); a level whose sum doesn't divide evenly by its count (the explicit `(double)` cast before dividing is what avoids Java's integer-division truncation here — dropping it would silently floor every average).

---

# Career Block

### Weekly Industry Awareness Ritual (20 min)

Clear the week's TLDR Newsletter backlog; read one engineering blog post in full, not skimmed.

### Weekly Scorecard

**Day 49, seven weeks in.** The plan's own required-ladder count: **97 total DSA problems solved** — this matches exactly: 84 required-only through Week 6, plus this week's 13 required (5 Stack + 8 Tree). Including this series' extra practice on top of the required ladder, the fuller count is **127 distinct problems solved** (111 through Week 6, plus this week's 16 — 13 required + 3 extra: Subtree of Another Tree, Minimum Depth, Average of Levels).

Stacks/Monotonic Stack is **fully closed at 12 required (up from 10) + 2 extra = 14 distinct.** Trees is underway with **8 of its expanded 15-problem set done (+ 3 extra = 11 distinct)**, BFS now covered alongside DFS. Docker, Docker Compose, JUnit 5/AssertJ/Mockito/TestContainers, and Kafka fundamentals are all live in `todo-api`.

---

# Day 49 — Interview Questions

**Q1. Why does the BFS level-order scaffold capture `queue.size()` into a variable before the inner loop, instead of just checking `queue.isEmpty()` inside it?**
At the start of each outer iteration, the queue holds exactly the current level's nodes. Capturing that count freezes it, so the inner loop processes precisely those nodes — even though it's simultaneously adding the *next* level's children to the same queue. Checking `isEmpty()` (or using the queue's live, changing size) instead would sweep newly-added children into the same pass as their own parents, collapsing every level into one.

**Q2. Describe the DFS-based alternative to Level Order Traversal, and explain what makes it correct.**
Pass the current depth down through a standard DFS, and the first time a given depth is reached, append a new list to the result at that index. It's correct because level-order grouping only requires knowing each node's depth — which a DFS can track just as well via a parameter — it isn't inherently tied to breadth-first traversal at all.

**Q3. In Right Side View, why does keeping "the last node processed in the inner loop" correctly give the rightmost node at each level?**
Children are always offered left child before right child, so nodes are dequeued in left-to-right order at every level. The last node processed in a given level's inner loop is therefore always the rightmost one at that depth, which is exactly the node visible from the right side.

**Q4. Which is cheaper on a completely skewed (linked-list-shaped) tree — BFS or DFS-based level order — and why does that reverse the usual intuition?**
BFS is cheaper here: its queue never holds more than one node at a time on a skewed tree (width 1), so its extra space drops close to O(1) beyond the output. DFS's call stack, by contrast, still costs O(n) — equal to the tree's height. This is a reversal of the usual pattern, where BFS is normally the one with worse worst-case space (up to O(n) width on a balanced tree) and DFS the safer one (O(log n) on a balanced tree).

**Q5. Why is `sum` declared as a `long` in Average of Levels rather than an `int`?**
To guard against silent overflow — a level with many nodes, each holding a value near `Integer.MAX_VALUE`, could overflow a 32-bit accumulator before the average is even computed, corrupting the result without throwing any error.

**Q6. What would happen in Average of Levels if the division `sum / levelSize` were computed without the `(double)` cast?**
Java would perform integer division, truncating any fractional part and silently flooring every level's average — e.g., a true average of `14.5` would compute as `14`. The cast to `double` on at least one operand is what forces floating-point division instead.

**Q7. State this week's DSA total two ways, and explain why there are two different correct numbers.**
The plan's own required-ladder-only count is 97, matching its own Day 49 claim exactly. Including this series' added extra practice, the fuller distinct-problem count is 127. The gap (30) is exactly the extra practice problems added on top of the required ladder across all seven weeks so far, including this week's three (Subtree of Another Tree, Minimum Depth, Average of Levels).

---

# Daily Deliverable Check

- [ ] One Stack problem from this week solved cold, without notes.
- [ ] Level Order Traversal (LC 102) and Right Side View (LC 199) solved — the `queue.size()` capture explained precisely, not just used — pushed.
- [ ] Average of Levels (LC 637) solved as extra practice.
- [ ] Weekly Industry Awareness Ritual completed (newsletter backlog + one blog post).
- [ ] Weekly Scorecard reviewed and understood — both the 97 (required-only) and 127 (cumulative distinct) figures.

---

## What Tomorrow Assumes You Already Know Cold

Day 50 assumes today's BFS scaffold and Day 46's `TreeNode` shape are both solid, but opens Binary Search Trees — a genuinely new invariant (left subtree smaller, right subtree larger) layered on top of the plain-tree shape this week established, not something today's material derives on its own.

---

# Week 7 Consolidation

**What actually got built:** Stacks/Monotonic Stack closed out at its Hard tier — Asteroid Collision, Basic Calculator II, Largest Rectangle in Histogram, Maximal Rectangle, Basic Calculator — finishing at 12 required (up from the original plan's 10) + 2 extra practice from Week 6 = **14 distinct problems total**, with a full two-family review (general-purpose LIFO vs. true monotonic invariant) on Day 45. Trees opened from zero: the `TreeNode` shape, precise depth/height definitions (and LeetCode's own node-counting convention, named explicitly), all four traversal orders, and the universal recursive template, followed by 8 required problems across DFS (single-tree, two-tree comparison, short-circuit sentinels, global-max tracking) and BFS — plus 3 extra practice problems (Subtree of Another Tree, Minimum Depth, Average of Levels) once the pattern was no longer brand new. On the engineering side, `todo-api` gained a multi-stage Docker build, a full Compose stack (app + Postgres + Redis), a complete JUnit 5/AssertJ/Mockito/TestContainers testing pyramid (unit tests against fakes, integration tests against a real containerized Postgres), and its first Kafka producer.

**Planned vs. actual:** the original (pre-revision) plan allotted 10 problems to Stacks and 11 to Trees; the revised plan expanded both (12 and 15 respectively) specifically because both patterns carry outsized real interview weight. This week delivered its full share of that expansion: Stacks fully closed at its new target, and Trees is 8 of its expanded 15 in, on schedule to close Week 8, Day 53 — the same day the Median of Two Sorted Arrays gap from the original plan's Day 21 finally gets fixed. Required-ladder problem count matches the plan's own Day 49 claim exactly (97); including extra practice, 127 distinct problems solved across seven weeks.

**Diagnostic — if any of these feel shaky, that's this week's actual signal, not a reason for concern on its own:**
- Can you state Largest Rectangle in Histogram's monotonic invariant and get the width arithmetic right without looking it up?
- Can you write the universal tree recursion template (base case for `null`, combine `left`/`right`) for a tree problem you've genuinely never seen before?
- Can you explain, unprompted, why Minimum Depth's naive formula breaks on a one-child node?
- Can you explain the difference between what `@DataJpaTest` and a Mockito-based unit test each actually verify?

**What Week 8 assumes:** that this week's Monotonic Stack invariant and amortized argument are fully reflexive (Week 8 doesn't touch Stacks again, but nothing here gets re-taught if a future pattern needs it). That the `TreeNode` shape, the universal recursive template, and BFS's level-order scaffold are all solid — Week 8 builds Binary Search Trees directly on top of the plain-tree shape (adding an ordering invariant, not a new shape), and Serialize/Deserialize Binary Tree (Day 52) directly extends the preorder-with-null-markers idea this week's Same Tree brute force already gestured at, without re-deriving it. That `todo-api` is now Docker/Compose-managed with a full testing pyramid in place and a working Kafka producer — Week 8 adds a Kafka *consumer* on top (`@KafkaListener`) and Spring Cloud Config, with no further changes to this week's Docker or testing setup. Median of Two Sorted Arrays (Day 53) will lean on the recursive divide-and-conquer intuition Construct Binary Tree from Preorder/Inorder (Day 51) builds — a dependency internal to Week 8 itself, not something this week needed to prepare.
