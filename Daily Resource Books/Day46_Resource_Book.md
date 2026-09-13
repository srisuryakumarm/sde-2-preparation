# Day 46 — Trees Begin, and Mockito

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 45 Resource Book](Day45_Resource_Book.md)
**Next ▶:** [Day 47 Resource Book](Day47_Resource_Book.md)
**Companion to:** Day 46 of `Week_07_Revised.md`

---

## Recap

Stacks/Monotonic Stack closed yesterday at 14 distinct problems. Today opens an entirely new DSA pattern — Trees — which will run for the rest of this week and into next (closing Week 8, Day 53, at 15 required problems). Nothing about Stacks carries forward directly; what *does* carry forward is recursion (Day 8) and the self-referential node shape Linked Lists have used since Week 5 — today extends that shape from one `next` reference to two, `left` and `right`. On the theory side, Mockito builds directly on yesterday's JUnit 5 foundation.

## Learning Objectives

By the end of today, without notes:

1. Define `TreeNode`, and explain precisely how it differs from the `Node` class Linked Lists have used since Day 34.
2. State the precise, edge-based definitions of depth and height, and explain why LeetCode's "Maximum Depth" is actually computing something closer to the tree's height, expressed as a node count.
3. Apply the universal recursive template — base case for `null`, combine `left`/`right` results — to a tree problem you've never seen before.
4. Solve Maximum Depth of Binary Tree and Invert Binary Tree, and justify the space complexity of a tree recursion precisely (not just "O(n)" — in terms of tree height, with the balanced vs. skewed distinction stated).
5. Write a Mockito-based unit test using `@Mock` and `@InjectMocks`, and explain why it runs without a database.

## Concept Dependency Map

```
Recursion (Day 8) — call stack, base case, recursive case
        │
Self-referential classes — Linked List Node, ONE reference (`next`)
(Weeks 5–6)
        │
        ▼
NEW: TreeNode — the SAME self-referential shape, TWO references
(`left`, `right`) instead of one
        │
        ├─▶ Terminology: root, leaf, depth (edges from root), height
        │    (edges to farthest leaf) — vs. LeetCode's node-counting
        │    "depth" convention
        │
        ├─▶ Traversal orders named: preorder, inorder, postorder (DFS
        │    shapes) and level-order (BFS — deferred to Day 49)
        │
        └─▶ Universal recursive template: null → base case,
             then combine left/right
                    │
                    ▼
        LC 104 Maximum Depth — combine: 1 + max(left, right)
        LC 226 Invert Binary Tree — combine: swap, then recurse
                    │
                    ▼
        (no extra practice today — pattern is OPENING, not closing;
         mirrors Day 34/39/40's precedent)
        │
        ▼
Day 45 — JUnit 5 (@Test, lifecycle annotations)
        │
        ▼
NEW: Mockito — @Mock fakes a dependency entirely; @InjectMocks wires
fakes into the class under test
```

---

# Part 1 — Binary Trees, From Zero

## The Shape

```java
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}
```

Compare this directly to the Linked List `Node` you've been building since Day 34:

```java
class Node {           // Linked List — Weeks 5–6
    int val;
    Node next;          // ONE self-reference
}

class TreeNode {        // Binary Tree — today
    int val;
    TreeNode left;        // TWO self-references
    TreeNode right;
}
```

It's the identical idea — a class that holds a value and one or more references to *another instance of itself* — with one structural change: a linked list node points to exactly one successor, forming a single chain; a tree node points to up to two, forming branches. Every recursive technique that already works on a self-referential single-pointer shape (Day 8's recursion, Weeks 5–6's linked-list traversal and reversal) extends to this shape with no new *mechanism* — only a new shape to apply it to.

## Terminology, Precisely

- **Root** — the single node with no parent; every other node is reached by following `left`/`right` references from it.
- **Leaf** — a node with `left == null` and `right == null`.
- **Depth of a node** — the number of *edges* on the path from the root down to that node. The root itself has depth 0.
- **Height of a node** — the number of *edges* on the longest downward path from that node to a leaf beneath it. A leaf has height 0.
- **Height of the tree** — the height of the root node, equivalently the depth of the tree's deepest leaf.

These are edge counts, and they matter for a recursive implementation: if you write `height(node) = 1 + max(height(left), height(right))` recursively, the base case for a `null` child must be `height(null) = -1`, so that a leaf — with two `null` children — correctly computes to `1 + max(-1, -1) = 0`.

> ⚠️ **Common Mistake — the "depth" naming trap.** LeetCode's own "Maximum Depth of Binary Tree" (today's Problem 1) is explicitly defined as **the number of *nodes*** along the longest root-to-leaf path, not the number of edges — for a single-node tree, LeetCode's answer is `1`, not the graph-theoretic depth of `0`. That's actually the tree's *height*, expressed as a node count instead of an edge count — which is why the standard recursive solution uses base case `null → 0` (not `-1`) and formula `1 + max(...)`: a leaf then correctly computes to `1 + max(0, 0) = 1`, i.e., "one node on the longest path," not "zero edges to reach it." Both conventions (edge-based, base case `-1`; node-counting, base case `0`) are internally consistent on their own — the mistake is mixing them, which silently produces an answer off by exactly one. This series follows LeetCode's own convention (node-counting, base case `0`) whenever solving a problem phrased the way LeetCode phrases it, and will say so explicitly whenever the distinction matters.

## Traversal Orders — Named Now, Used Throughout

Every way of visiting every node in a tree exactly once falls into one of four named orders:

- **Preorder** — visit the node itself, *then* recurse left, *then* recurse right (root, left, right).
- **Inorder** — recurse left, *then* visit the node, *then* recurse right (left, root, right).
- **Postorder** — recurse left, recurse right, *then* visit the node (left, right, root).
- **Level-order** — visit every node at depth 0, then every node at depth 1, then depth 2, and so on. This is fundamentally different from the other three: it's breadth-first (a queue), not depth-first (recursion/a stack), and it's the entire subject of Day 49.

Today's two problems are both naturally **postorder-shaped**, even though neither literally prints a traversal: both need each subtree's own result *before* they can combine it into the answer for the current node, which is exactly postorder's "children before self" ordering.

## The Universal Recursive Template

Nearly every DFS tree problem — today's and most of the ones ahead — follows one shape:

```java
ReturnType solve(TreeNode node) {
    if (node == null) {
        return /* base case — the "neutral" value for an empty tree */;
    }
    ReturnType leftResult = solve(node.left);
    ReturnType rightResult = solve(node.right);
    return /* combine node.val, leftResult, and rightResult */;
}
```

The entire design work on a new tree problem is answering two questions: **what should the base case return for `null`**, and **how do I combine a node's own value with what its two children already computed**? Once those two answers are fixed, the recursion itself is usually a direct transcription.

## Complexity — Stated Precisely, Not by Default

**Time:** every node is visited exactly once, doing O(1) work per node (outside of the recursive calls themselves) — **O(n)**, for any of today's problems.

**Space — this is where "just say O(n)" undersells what's actually true.** The only space genuinely used by a pure DFS recursion (ignoring the output itself) is the **call stack**, and its depth at any moment is bounded by the tree's **height**, not its node count. That means:

- A **balanced** tree (height ≈ log₂ n) uses **O(log n)** call-stack space.
- A **completely skewed** tree — every node has only one child, degenerating into a linked list shape — has height `n − 1`, using **O(n)** call-stack space.

Today's hints (and most references) state "Space O(n)" for these problems — that's the **worst case** (a skewed tree), stated as a safe upper bound. The fully precise answer, and the one that survives a sharp interviewer pushing on it, is **O(h)** where `h` is the tree's height, which is O(log n) for a balanced tree and O(n) in the worst case. Stating the worst case alone isn't wrong — it's what the plan's own hints do — but naming the O(h) generalization *unprompted* is exactly the kind of depth this series has been building toward.

## Common Mistakes

- ⚠️ **`NullPointerException` on a child that doesn't exist** — calling `.left.val` or recursing into `.left` without checking `node.left != null` first, or without the recursive call's own `null` base case catching it. The base case is what makes this safe: `solve(node.left)` when `node.left` is `null` doesn't crash, it hits the base case and returns immediately.
- ⚠️ **Conflating depth and height**, or mixing their base cases (above) — the single most common source of off-by-one tree bugs.
- ⚠️ **Forgetting the `null` base case entirely** for an empty tree (`root == null` as the very first check in the public-facing method) — a surprising number of tree solutions are correct for every non-empty tree and crash immediately on an empty one.

---

## Problem 1: Maximum Depth of Binary Tree (LeetCode 104, Easy) — Pattern: DFS

**Statement:** Given the root of a binary tree, return its maximum depth — the number of nodes along the longest path from the root down to the farthest leaf.

### Approach 1 — Recursive DFS (the natural fit)

```java
public static int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

This *is* the universal template from above: base case `null → 0`, combine by taking the larger of the two children's depths and adding one for the current node. Every node's depth is one more than the deeper of its two children's — exactly what the recursion computes, bottom-up.

### Approach 2 — Iterative BFS (a genuinely distinct alternative)

```java
public static int maxDepthBFS(TreeNode root) {
    if (root == null) return 0;
    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);
    int depth = 0;
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        depth++;
    }
    return depth;
}
```

Instead of recursing, walk the tree level by level with a queue (Day 4's `ArrayDeque`, used here as a FIFO), counting how many levels exist. This previews the exact scaffold Day 49 formalizes — capturing `queue.size()` before draining a level is what isolates one level from the next; that mechanism gets its full explanation then, since it's the star technique for three of Day 49's problems, not just a footnote here.

**Trade-off:** both are O(n) time. The recursive version is shorter and more directly mirrors the problem's own recursive structure; the iterative version avoids any call-stack depth risk on a pathologically skewed tree (recursion depth equal to tree height means a sufficiently deep, unbalanced tree could risk a `StackOverflowError` — Day 8's exact concern, applied here), at the cost of needing an explicit queue.

**Trace (recursive):** tree `[3, 9, 20, null, null, 15, 7]` (root 3, left child 9 — a leaf, right child 20 with children 15 and 7).

```
        3
       / \
      9   20
         /  \
        15   7
```

`maxDepth(3)` → `1 + max(maxDepth(9), maxDepth(20))`
`maxDepth(9)` → `1 + max(maxDepth(null), maxDepth(null)) = 1 + max(0,0) = 1`
`maxDepth(20)` → `1 + max(maxDepth(15), maxDepth(7)) = 1 + max(1,1) = 2`
`maxDepth(3)` → `1 + max(1, 2) = 3`

Final answer: **3**. Correct.

**Complexity: Time O(n) both approaches. Space:** recursive is O(h) call stack (O(log n) balanced, O(n) worst case, per above); iterative BFS is O(w), where `w` is the tree's maximum **width** (the most nodes ever sitting in the queue at once) — for a balanced tree this can be up to O(n/2) at the last level, so both approaches are O(n) in the worst case, but for different structural reasons (stack depth vs. queue width).

**Edge cases:** empty tree (`root == null` → 0, both approaches handle this as the very first check); a single node (→ 1); a completely skewed tree (both still correct — worst-case space, not worst-case correctness).

> 💡 **Interview Insight:** State the recursive solution first — it's the natural fit and takes seconds to write — then volunteer the iterative BFS alternative and its call-stack-safety trade-off *before* being asked. That's a stronger signal than being asked "can you do it without recursion?" and only then producing it.

---

## Problem 2: Invert Binary Tree (LeetCode 226, Easy) — Pattern: DFS

**Statement:** Given the root of a binary tree, invert it (swap every node's left and right children) and return the root.

### Approach 1 — Recursive

```java
public static TreeNode invertTree(TreeNode root) {
    if (root == null) return null;
    TreeNode temp = root.left;
    root.left = invertTree(root.right);
    root.right = invertTree(temp);
    return root;
}
```

Save `root.left` before overwriting it — otherwise the second assignment would read a value that's already been clobbered. `root.left` is assigned the *inverted* right subtree, and `root.right` the *inverted* (saved) left subtree — the recursion and the swap happen together, so every level of the tree ends up both swapped and internally inverted.

### Approach 2 — Iterative (stack- or queue-based; order doesn't matter here)

```java
public static TreeNode invertTreeIterative(TreeNode root) {
    if (root == null) return null;
    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        TreeNode temp = node.left;
        node.left = node.right;
        node.right = temp;
        if (node.left != null) stack.push(node.left);
        if (node.right != null) stack.push(node.right);
    }
    return root;
}
```

Every node needs to be visited exactly once, and swapping one node's children doesn't depend on any other node's swap having happened yet — so unlike Maximum Depth (where a node's answer genuinely depends on its children's answers), the *order* of visiting doesn't matter at all here. A stack (DFS order) or a queue (BFS order) both work identically; this version uses a stack for no reason beyond convention.

**Trace:** tree `[4, 2, 7, 1, 3, 6, 9]`.

```
      4                    4
     / \                  / \
    2   7      -->       7   2
   / \ / \              / \ / \
  1  3 6  9            9  6 3  1
```

At the root: swap `2` and `7`. Recurse into (now) `root.left = 7` (originally the right subtree): swap its children `6` and `9`. Recurse into (now) `root.right = 2` (originally the left subtree): swap its children `1` and `3`. Every subtree is a mirror image of its original.

**Complexity: Time O(n) — every node visited once, both approaches. Space: O(h)** recursive (call stack), **O(w)** iterative (explicit stack/queue holding at most one level's worth of unprocessed nodes at a time in the worst case) — same shape of trade-off as Problem 1.

**Edge cases:** empty tree (`null` returned immediately); a single node (swapping two `null` children is a no-op, correctly); an already-symmetric tree (still fully processed — the algorithm has no way to know in advance that a swap will be a no-op at some nodes).

---

*(No extra practice added today — Trees is opening, not closing, matching this series' established precedent from Linked Lists (Day 34) and Stacks/Monotonic Stack (Days 39–40): the first day of a new pattern gets exactly its required problems, with reinforcement reps added once the pattern is no longer brand new. Day 47 onward will add extra practice as it becomes appropriate.)*

---

# Part 2 — Mockito

### Prerequisites (confirmed)

Day 45's JUnit 5 structure (`@Test`, the lifecycle annotations) — directly. Mockito doesn't replace JUnit 5's test-running mechanism; it slots into it.

### What Mockito Actually Does

`TaskService` (built up over prior weeks) depends on `TaskRepository`. Testing `TaskService`'s own logic — validation, business rules, whatever it does beyond a pass-through — shouldn't require a real database to be running; it should be testable in complete isolation. Mockito creates a **fake implementation** of `TaskRepository` that does nothing on its own except what you explicitly tell it to:

```java
@ExtendWith(MockitoExtension.class)
class TaskServiceTest {

    @Mock
    private TaskRepository taskRepository;      // a fake — no real database behind it at all

    @InjectMocks
    private TaskService taskService;             // TaskService, with the fake TaskRepository
                                                    // automatically wired into its constructor/fields

    @Test
    void throwsWhenTaskNotFound() {
        when(taskRepository.findById(1L)).thenReturn(Optional.empty());   // stub the fake's behavior

        assertThatThrownBy(() -> taskService.getById(1L))
            .isInstanceOf(TaskNotFoundException.class);

        verify(taskRepository).findById(1L);      // confirm the fake was actually called as expected
    }
}
```

`@Mock` creates the fake. `@InjectMocks` constructs `TaskService` and automatically wires any `@Mock`-annotated fields into it (via constructor or field injection, whichever `TaskService` uses) — no manual `new TaskService(taskRepository)` wiring needed. `when(...).thenReturn(...)` **stubs** the fake's behavior for a specific call; `verify(...)` asserts the fake was actually invoked the way you expected, which is a genuinely different kind of assertion than checking a return value — it's checking that an *interaction happened*, not just that a value came back correct.

### Why This Is a Fundamentally Different Kind of Test Than Yesterday's

Yesterday's `@DataJpaTest` spins up a real (if embedded) database and a real Spring context around your JPA layer — it's verifying that your persistence mapping actually works against something database-shaped. Today's Mockito-based test has **no database, no Spring context, no I/O at all** — `TaskRepository` is entirely fake, so calling it costs nothing beyond a plain Java method call. This is exactly why the plan's own definition of done — "the test passes in milliseconds" — is a real, measurable signal, not just an expectation: there's genuinely nothing slow left in the test to wait on.

This is the classic **unit vs. integration** distinction in the testing pyramid: a unit test (today) verifies one class's logic in complete isolation, fast and focused; an integration test (yesterday's `@DataJpaTest`, and Day 47's TestContainers) verifies that real collaborating pieces actually work together, slower but closer to production reality. Neither replaces the other — a healthy test suite has many of the first and fewer of the second.

### Common Mistakes

- ⚠️ **Forgetting to stub a method the code under test actually calls** — an un-stubbed mock method returns Java's default value for its return type (`null` for objects, `0` for numeric types, `false` for `boolean`) rather than throwing, which can silently mask what should have been a test failure.
- ⚠️ **Mocking the class under test itself** — `@Mock` belongs on `TaskService`'s *dependencies* (`TaskRepository`), never on `TaskService` itself; mocking the very thing you're trying to test verifies nothing about its real behavior.
- ⚠️ **Over-verifying** — asserting `verify()` on every single interaction, including ones incidental to what the test actually cares about, makes tests brittle and coupled to implementation details that should be free to change.

---

# Project Block Guide (1.5 hrs)

**Repository:** `todo-api`. Write a Mockito-based unit test for `TaskService`, mocking `TaskRepository` — cover at least one "happy path" (successful creation or retrieval) and one failure path (not-found, or a validation rejection, depending on what `TaskService` currently enforces).

**Definition of done:** the test suite passes, and specifically passes in milliseconds — time it, and confirm there's no observable delay, which is your proof the repository layer is genuinely, completely mocked out rather than accidentally hitting a real database.

---

# Career Block Guide (1 hr)

### LinkedIn — Post 11

> Today I hit the pattern that shows up in more interviews than almost anything else: trees.
>
> The core idea, once you see it, is almost suspiciously simple — for any tree problem, ask two questions: what should an empty subtree contribute (the base case), and how do I combine a node's own value with what its two children already figured out? That's the whole recursive template.
>
> DFS (recursion, depth-first) and BFS (a queue, level-by-level) solve different shapes of tree question — DFS for anything about a single root-to-leaf path or combining subtree results; BFS for anything about "level N" specifically. [Attach a simple side-by-side diagram: a small tree with DFS's visit order (root→left→right, following one branch down before backing up) next to BFS's visit order (all of depth 0, then all of depth 1, then depth 2) — the visual is the entire point of this post.]
>
> Day 46 of SDE-2 prep. #buildinpublic #dsa #datastructures

### Networking

Send 5 connection requests to Senior Engineers at Tier A companies.

---

# Day 46 — Interview Questions

**Q1. How does `TreeNode` differ structurally from the Linked List `Node` class, and what stays exactly the same?**
`TreeNode` has two self-references (`left`, `right`) instead of one (`next`), forming branches instead of a single chain. What stays the same is the core idea — a class holding a value plus one or more references to another instance of itself — and every recursive technique for handling `null`, defining a base case, and combining results carries over unchanged; only the number of children to combine changes.

**Q2. Give the precise, edge-based definitions of depth and height, and state each one's base case for a `null` node.**
Depth of a node is the number of edges on the path from the root down to it (root has depth 0). Height of a node is the number of edges on the longest downward path from it to a leaf (a leaf has height 0). Using the recursive formula `height = 1 + max(height(left), height(right))`, the base case for a `null` child must be `-1`, so a leaf — with two `null` children — correctly computes to `1 + max(-1,-1) = 0`.

**Q3. LeetCode's "Maximum Depth of Binary Tree" uses base case `null → 0`, not `-1`. What is it actually computing, and why does that base case make sense for it?**
It's computing the tree's height, expressed as a count of *nodes* on the longest path rather than *edges* — for a single-node tree it returns 1, not the graph-theoretic depth of 0. With base case `0`, a leaf computes to `1 + max(0,0) = 1`, correctly counting "one node," which is exactly LeetCode's own stated definition for this problem.

**Q4. What are the four named traversal orders, and which one shares Maximum Depth and Invert Binary Tree's underlying shape?**
Preorder (root, left, right), inorder (left, root, right), postorder (left, right, root), and level-order (breadth-first, by depth). Both of today's problems are postorder-shaped: each needs both children's results before it can combine them into its own answer, even though neither literally performs a traversal that prints values.

**Q5. Precisely state the space complexity of a recursive DFS tree traversal — not just "O(n)."**
O(h), where h is the tree's height: O(log n) for a balanced tree, degrading to O(n) in the worst case of a completely skewed tree (every node with only one child). "O(n)" is a correct but loose upper bound that's really describing the worst case only.

**Q6. Why is Maximum Depth's iterative BFS alternative worth knowing, given the recursive version is shorter?**
It avoids the call-stack depth risk the recursive version carries on a pathologically skewed tree — recursion depth equals tree height, and a sufficiently deep, unbalanced tree risks a `StackOverflowError`. BFS's space bound is O(w), the tree's maximum width, a structurally different risk than O(h) stack depth.

**Q7. In Invert Binary Tree's recursive solution, why must `root.left` be saved to a temp variable before either assignment happens?**
The two assignments overwrite `root.left` and `root.right` in sequence; if `root.left` isn't saved first, the second assignment (`root.right = invertTree(temp)`) would need `temp` to still hold the *original* left child, but without saving it, that value would already have been overwritten by the first assignment.

**Q8. Why doesn't the order of traversal (stack vs. queue, DFS vs. BFS) matter for Invert Binary Tree, when it clearly does matter for Maximum Depth?**
Swapping one node's children is a fully local operation that doesn't depend on any other node's swap having already happened. Maximum Depth, by contrast, needs each child's *result* before it can compute its own — a genuine data dependency that traversal order must respect (children before parent), which Invert Binary Tree simply doesn't have.

**Q9. What does `@Mock` actually create, and what does `@InjectMocks` do with it?**
`@Mock` creates a fake implementation of a dependency (e.g., `TaskRepository`) that does nothing unless explicitly stubbed. `@InjectMocks` constructs the class under test (`TaskService`) and automatically wires any `@Mock` fields into it, without needing manual `new TaskService(taskRepository)` wiring.

**Q10. Why does a Mockito-based `TaskService` test run in milliseconds, while yesterday's `@DataJpaTest` does not?**
The Mockito test has no database and no Spring context at all — `TaskRepository` is a plain fake object, so calling it is just a regular method call with no I/O. `@DataJpaTest` spins up a real (if embedded) database and Spring context, which carries genuine startup and I/O cost even though it's not a full production database.

---

# Daily Deliverable Check

- [ ] Maximum Depth of Binary Tree (LC 104) and Invert Binary Tree (LC 226) solved — both approaches for each, and the depth-vs-height distinction stated precisely — pushed to `dsa-java/trees/`.
- [ ] Mockito-based `TaskService` test passing, mocking `TaskRepository`, confirmed to run in milliseconds.
- [ ] LinkedIn Post 11 published, with the DFS-vs-BFS diagram attached.
- [ ] 5 connection requests sent to Senior Engineers at Tier A companies.

---

## What Tomorrow Assumes You Already Know Cold

Day 47 assumes today's `TreeNode` shape and the universal recursive template (base case for `null`, combine `left`/`right`) are fully solid, since Same Tree and Symmetric Tree extend that exact template to comparing *two* trees — or a tree against its own mirror — simultaneously, without re-deriving the base recursive scaffold from scratch.
