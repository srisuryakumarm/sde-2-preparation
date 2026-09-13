# Day 47 — Tree DFS Continues, and TestContainers

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 46 Resource Book](Day46_Resource_Book.md)
**Next ▶:** [Day 48 Resource Book](Day48_Resource_Book.md)
**Companion to:** Day 47 of `Week_07_Revised.md`

---

## Recap

Day 46 built the universal recursive template — base case for `null`, combine `left`/`right` results — against a single tree. Today extends the exact same template to **two** trees at once: Same Tree compares two independent trees directly; Symmetric Tree compares a tree against its own mirror image, which turns out to be Same Tree's identical logic in disguise. Today is also the first day this week to add extra practice, now that Trees is no longer brand new (mirroring Day 41's precedent for Monotonic Stack, which similarly withheld extras on its own opening days).

## Learning Objectives

By the end of today, without notes:

1. Solve Same Tree and Symmetric Tree, and explain precisely why Symmetric Tree's recursion is Same Tree's logic applied to a tree's own two halves.
2. Solve Subtree of Another Tree by composing Same Tree as a subroutine, and state why that composition is correct.
3. Explain what TestContainers adds over Day 45's embedded-H2 approach, and why it needs Day 43's Docker directly.

## Concept Dependency Map

```
Day 46 — TreeNode; universal recursive template (null → base case,
combine left/right); postorder-shaped combination
        │
        ▼
Extended to TWO trees simultaneously:
  ├─ LC 100 Same Tree — combine: are these two roots equal, AND are
  │  their left subtrees the same, AND their right subtrees the same
  ├─ LC 101 Symmetric Tree — Same Tree's exact logic, comparing
  │  root.left against root.right, but MIRRORED: left.left vs.
  │  right.right, left.right vs. right.left
  └─ LC 572 Subtree of Another Tree (EXTRA) — DFS over the main tree,
     calling Same Tree at every node
        │
        ▼
Day 43 — Docker image/container
        │
        ▼
NEW: TestContainers — spins up a REAL Postgres container for
integration tests (needs Docker, directly)
        │
        ▼
Project: todo-api's @DataJpaTest suite now runs against real
TestContainers Postgres, not H2
```

---

## Problem 3: Same Tree (LeetCode 100, Easy) — Pattern: DFS, Two-Tree Comparison

**Statement:** Given the roots of two binary trees, return `true` if they are structurally identical and every corresponding node holds the same value.

### Approach 1 — Brute force: serialize both, then compare

```java
public static boolean isSameTreeBruteForce(TreeNode p, TreeNode q) {
    List<Integer> pSerialized = new ArrayList<>();
    List<Integer> qSerialized = new ArrayList<>();
    serialize(p, pSerialized);
    serialize(q, qSerialized);
    return pSerialized.equals(qSerialized);
}

private static void serialize(TreeNode node, List<Integer> out) {
    if (node == null) {
        out.add(null);            // an explicit marker — WITHOUT it, [1,2] and [1,null,2] would
        return;                    // serialize identically, since both would just produce [1, 2]
    }
    out.add(node.val);
    serialize(node.left, out);
    serialize(node.right, out);
}
```

Convert each tree to a linear preorder sequence — **with an explicit marker for every `null` child**, not just skipping them — then compare the two sequences directly. Correct, and O(n) time, but it allocates two full extra lists to hold the serializations, which the direct comparison below doesn't need at all.

### Approach 2 — Optimized: direct recursive comparison

```java
public static boolean isSameTree(TreeNode p, TreeNode q) {
    if (p == null && q == null) return true;
    if (p == null || q == null) return false;
    return p.val == q.val
        && isSameTree(p.left, q.left)
        && isSameTree(p.right, q.right);
}
```

The universal template, applied to two trees walked in lockstep: the base case now has two sub-cases (both `null` → structurally matching so far; exactly one `null` → a structural mismatch, immediate `false`), and the combine step is "this node's values match, AND the left subtrees match, AND the right subtrees match." Java's `&&` **short-circuits** — the moment any one condition is `false`, the remaining calls never execute, so a mismatch near the root returns immediately without needlessly walking the rest of either tree.

**Trace:** `p = [1, 2, 3]`, `q = [1, 2, 3]` (identical trees).

`isSameTree(1,1)`: values equal → check `isSameTree(2,2)` → both null children, `true` → check `isSameTree(3,3)` → both null children, `true` → overall `true`.

Contrast: `p = [1, 2]` (left child only), `q = [1, null, 2]` (right child only). At the root, values match (`1 == 1`), so recurse: `isSameTree(p.left=2, q.left=null)` — one is `null`, the other isn't → immediately `false`, short-circuiting the rest of the check. **This is exactly why the brute-force serialization above must mark `null`s explicitly** — without that marker, both of these trees would serialize to the identical sequence `[1, 2]`, incorrectly reporting them as the same tree despite having completely different shapes.

**Complexity: Time O(n) — every node visited once (assuming the trees don't mismatch early; O(min(n,m)) if they do, due to short-circuiting). Space: O(h) recursive** (call stack only) **vs. O(n) for the serialization approach** (two full lists) — the direct comparison's real advantage over brute force isn't a better time bound, it's avoiding that extra space entirely.

**Edge cases:** both trees empty (`true`); one empty, one not (`false` immediately); same values but different shapes (the explicit-`null`-marker point above — this is the case that actually separates a correct solution from a subtly wrong one).

> 💡 **Interview Insight:** If you reach for serialization first, say out loud *why* a null marker is non-negotiable — dropping it is the single most common way to submit a solution that passes a few tests and then fails on exactly this shape-mismatch case. It's also worth naming this serialize-and-compare idea explicitly now: it's the seed of Day 52's Serialize/Deserialize Binary Tree, a much larger problem in its own right, not something to build out further today.

---

## Problem 4: Symmetric Tree (LeetCode 101, Easy) — Pattern: DFS, Mirrored Comparison

**Statement:** Given the root of a binary tree, check whether it is a mirror of itself around its center.

### Approach 1 — Brute force: build the mirror, then compare with Same Tree

```java
public static boolean isSymmetricBruteForce(TreeNode root) {
    if (root == null) return true;
    TreeNode mirrorOfRight = deepCopyAndInvert(root.right);
    return isSameTree(root.left, mirrorOfRight);
}
```

Explicitly build an inverted copy of the right subtree (Day 46's Invert Binary Tree, applied to a *copy* rather than mutating the original), then compare it against the left subtree using yesterday's — today's earlier — `isSameTree`. Correct, and it makes the "mirror" idea completely explicit, but it allocates an entire extra copy of half the tree just to throw it away after one comparison.

### Approach 2 — Optimized: mirrored comparison, no extra tree built

```java
public static boolean isSymmetric(TreeNode root) {
    return root == null || isMirror(root.left, root.right);
}

private static boolean isMirror(TreeNode left, TreeNode right) {
    if (left == null && right == null) return true;
    if (left == null || right == null) return false;
    return left.val == right.val
        && isMirror(left.left, right.right)
        && isMirror(left.right, right.left);
}
```

**This is Same Tree's exact logic — compare two structures for equality — with one change: instead of comparing `p` against `q` position-for-position, it compares `left` against `right` *crossed*: `left.left` against `right.right`, and `left.right` against `right.left`.** That crossing is the entire definition of a mirror — a reflection swaps positions, so what should match isn't "same side to same side," it's "left side to right side, reversed at every level." No extra tree is ever built; the mirroring is entirely in *which pair of children get compared next*, not in any data structure.

**Trace:** `root = [1, 2, 2, 3, 4, 4, 3]`.

```
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

`isMirror(2, 2)` [comparing root's left and right]: values equal → `isMirror(3, 3)` [left.left vs. right.right] → both leaves, equal, `true` → `isMirror(4, 4)` [left.right vs. right.left] → both leaves, equal, `true` → overall `true`. Symmetric, correctly.

**Complexity: Time O(n). Space: O(h) recursive** (call stack) **vs. O(n) for the brute-force copy-and-compare** (an entire extra half-tree allocated).

**Edge cases:** empty tree (`true` — symmetric vacuously); single node (`true` — `isMirror(null, null)`); a tree symmetric in *values* but not *shape* — e.g., `[1,2,2,null,3,null,3]` — must return `false`, and does, because the very first `left == null || right == null` check (comparing `left.right=3`'s position against `right.left=null`) catches the shape mismatch before values are ever compared at that pair.

> 💡 **Interview Insight:** Naming the connection to Same Tree unprompted — "this is the same comparison, just crossed" — is a stronger signal than independently re-deriving mirrored recursion from scratch, because it demonstrates pattern transfer, which is the actual skill this whole series is training.

---

## Extra Practice: Subtree of Another Tree (LeetCode 572, Easy) — Pattern: DFS, composing Same Tree

**Statement:** Given the roots of two binary trees `root` and `subRoot`, return `true` if there is a subtree of `root` with the same structure and node values as `subRoot`.

### Approach — DFS over `root`, calling `isSameTree` at every node

```java
public static boolean isSubtree(TreeNode root, TreeNode subRoot) {
    if (root == null) return false;
    if (isSameTree(root, subRoot)) return true;
    return isSubtree(root.left, subRoot) || isSubtree(root.right, subRoot);
}
```

This directly reuses Problem 3's `isSameTree` as a subroutine: at every node of `root`, ask "does the tree rooted *here* exactly match `subRoot`?" If not, the answer might still be `false` — for the moment — but there could be a match somewhere further down, so recurse into both children and ask the same question there.

**Why a plain value-only check isn't enough — the trap this problem is really testing:** it's tempting to just check whether `subRoot`'s values appear somewhere in `root`, ignoring structure. Consider `root = [3, 4, 5, 1, 2, null, null, null, null, 0]` and `subRoot = [4, 1, 2]` — every value in `subRoot` genuinely does appear in `root`'s left branch, but `root`'s node `4` has an *extra* grandchild (`0`) that `subRoot`'s node `4` doesn't have, so it is **not** a true subtree match. This is exactly why the check has to be full structural equality (`isSameTree`, which compares shape *and* values together, including `null` markers) rather than a values-only membership check.

**Complexity: Time O(n × m)**, where `n = size of root`, `m = size of subRoot` — in the worst case, `isSameTree` is attempted at every one of `root`'s n nodes, and each attempt costs up to O(m). **Space: O(h_root)** call stack for the outer DFS, plus O(h_subRoot) for whichever `isSameTree` call is active at any moment — both bounded by the respective trees' heights.

*(A further optimization exists — serializing both trees with explicit null markers, the same idea Problem 3's brute force introduced, and running a linear-time string-matching algorithm like KMP for O(n + m) overall — but KMP hasn't been covered in this series and is out of scope here; it's worth knowing the better bound exists, not worth detouring into building it today.)*

**Edge cases:** `subRoot` matching at the very root of `root` (the first `isSameTree` call catches it immediately); `subRoot` appearing nowhere (every recursive branch returns `false`, propagating up through the `||` chain); values matching but structure differing, as traced above — the case this problem is specifically designed to catch.

> 💡 **Interview Insight:** State the composition explicitly before coding: "this is Same Tree, called at every node." That single sentence is the entire algorithm; the code is just the DFS scaffold needed to try it everywhere.

---

# Part 2 — TestContainers

### Prerequisites (confirmed)

Docker — image vs. container, Day 43, directly: TestContainers' entire mechanism is "spin up a real Docker container, programmatically, for the duration of one test run." Day 45's `@DataJpaTest` + embedded H2 is also assumed, since today's project directly replaces H2 with this.

### Why H2 Isn't Enough

Day 45's `@DataJpaTest` runs against H2 — fast, zero setup, but **not actually Postgres**. H2 doesn't enforce Postgres-specific behavior: SQL dialect differences, specific constraint-enforcement timing, JSON column types, and other Postgres-only features can all behave differently — or not exist at all — in H2. A test suite that's entirely green against H2 can still fail the moment it runs against real Postgres in production, because H2 never actually verified the thing that matters: does this code work against *the database you're actually deploying to*.

### What TestContainers Does

```java
@Testcontainers
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class TaskRepositoryIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Test
    void savesAndRetrievesATask() {
        // runs against a REAL, throwaway Postgres container
    }
}
```

`@Testcontainers` and `@Container` tell JUnit 5 to start a real `postgres:16` Docker container (via Day 43's Docker daemon, running locally) before the test class runs, and tear it down after. Postgres in a container doesn't get a fixed, predictable port — Docker assigns one dynamically to avoid clashing with anything else running locally — so `@DynamicPropertySource` reads that container's actual assigned URL, username, and password *at test-run time* and injects them into Spring's configuration, overriding whatever's in `application.yml`. `@AutoConfigureTestDatabase(replace = NONE)` tells `@DataJpaTest` **not** to fall back to its default embedded-database behavior, since a real Postgres container is being supplied instead.

The result: the exact same `@DataJpaTest` structure from Day 45, now running against genuine Postgres, in a fresh, disposable container, every test run — catching the dialect and feature gaps H2 could never have caught, while still being fully automated and requiring no manually-managed shared database.

### Common Mistakes

- ⚠️ **Assuming TestContainers replaces `@DataJpaTest` entirely** — it doesn't; it replaces *what database `@DataJpaTest` runs against*, and still needs `replace = NONE` to stop Spring from silently substituting its own embedded default underneath it.
- ⚠️ **Hardcoding a port** — Postgres's container port is dynamic by design; `@DynamicPropertySource` exists specifically because the URL isn't known until the container has actually started.
- ⚠️ **Running TestContainers without Docker available** (e.g., in some CI environments without Docker-in-Docker configured) — the tests will fail to even start the container, which reads as a confusing infrastructure failure rather than an obvious "Docker isn't available here."

---

# Project Block Guide (1.5 hrs)

**Repository:** `todo-api`. Replace Day 45's embedded H2 with TestContainers Postgres in the `@DataJpaTest` suite, using `@DynamicPropertySource` as shown above.

**Definition of done:** `mvn test` spins up a real Postgres container automatically and runs the existing repository tests against it — confirm by checking `docker ps` *during* a test run and seeing a live, temporary Postgres container appear.

---

# Career Block Guide (1 hr)

### LinkedIn

Engagement day — 20 minutes commenting on 3–5 posts in your network.

### Networking

Review 3 engineering manager profiles for outreach tomorrow.

---

# Day 47 — Interview Questions

**Q1. Why must a tree serialization include an explicit marker for `null` children, not just omit them?**
Without an explicit marker, structurally different trees can produce identical serialized sequences — e.g., `[1, 2]` (left child only) and `[1, null, 2]` (right child only) both serialize to `[1, 2]` if `null`s are simply skipped, incorrectly making two differently-shaped trees compare as equal.

**Q2. Walk through why Symmetric Tree's `isMirror` compares `left.left` against `right.right`, rather than `left.left` against `right.left`.**
A mirror reflection swaps positions at every level, not just at the top — the left subtree's *left* child sits in the position that mirrors the right subtree's *right* child, and vice versa. Comparing same-side to same-side (`left.left` vs. `right.left`) would check for two identical subtrees, not a mirrored pair.

**Q3. What's the actual advantage of the direct recursive comparison in Same Tree / Symmetric Tree over serializing both structures first and comparing the results?**
It avoids allocating extra data structures (full serialized lists, or in Symmetric Tree's brute force, an entire copied-and-inverted subtree) purely to hold an intermediate representation — the direct comparison only needs O(h) call-stack space, versus O(n) for the serialized approach.

**Q4. In Subtree of Another Tree, why is checking whether `subRoot`'s values merely *appear* somewhere in `root` not sufficient?**
Value membership ignores structure — a node in `root` can contain every value from `subRoot` while having additional children `subRoot` doesn't have, which is not a true subtree match. The check needs full structural equality (matching shape and values together, via `isSameTree`), not just value presence.

**Q5. State the time complexity of Subtree of Another Tree and justify it.**
O(n × m), where n is the size of `root` and m is the size of `subRoot`: in the worst case, `isSameTree` is attempted at every one of `root`'s n nodes, and each attempt costs up to O(m) to potentially walk all of `subRoot`.

**Q6. Why doesn't H2 fully substitute for Postgres in a persistence-layer test, even though both are relational databases?**
H2 doesn't enforce Postgres-specific SQL dialect behavior, constraint timing, or Postgres-only features (like native JSON column types) — a test suite fully passing against H2 can still fail against real Postgres in production, because H2 never actually exercised the database engine the application is really deployed against.

**Q7. Why does TestContainers need `@DynamicPropertySource` specifically, rather than a fixed connection string in `application.yml`?**
The container's Postgres instance is assigned a port dynamically by Docker at startup, to avoid clashing with anything else running locally — the actual connection URL isn't known until the container has already started, so it has to be read and injected at test-run time rather than hardcoded in advance.

**Q8. What does `@AutoConfigureTestDatabase(replace = Replace.NONE)` do, and why is it necessary alongside TestContainers?**
It tells `@DataJpaTest` not to fall back to its own default embedded-database substitution — without it, `@DataJpaTest`'s default behavior would silently override the real TestContainers Postgres connection with its own embedded database, defeating the entire point of using TestContainers.

---

# Daily Deliverable Check

- [ ] Same Tree (LC 100) and Symmetric Tree (LC 101) solved, with the "Same Tree logic, crossed" connection stated explicitly — pushed.
- [ ] Subtree of Another Tree (LC 572) solved as extra practice, with the value-vs-structure trap understood.
- [ ] `todo-api`'s `@DataJpaTest` suite running against a real TestContainers Postgres container, confirmed via `docker ps` during a test run.
- [ ] 20 minutes of LinkedIn engagement completed.
- [ ] 3 engineering manager profiles reviewed.

---

## What Tomorrow Assumes You Already Know Cold

Day 48 assumes today's two-tree comparison template is solid, but introduces a genuinely new recursive shape on top of Day 46's base template: a sentinel value that short-circuits a recursion early (Balanced Binary Tree), and a running maximum tracked *outside* the value being returned upward (Diameter of Binary Tree). Neither of today's problems used either technique — today assumed only single-return-value combination, which is exactly what tomorrow extends.
