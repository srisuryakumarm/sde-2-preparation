# Day 51 Resource Book — BST and Construction, and Spring Cloud Config

**Series:** SDE-2 Interview Prep Resource Books (Week 8) · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 50](./Day50_Resource_Book.md) · **Next ▶:** [Day 52](./Day52_Resource_Book.md)
**Companion to:** Day 51 of `Week_08_Revised.md`

---

## Recap: what today is

Yesterday layered the BST ordering invariant on top of the plain-tree shape and used it for a validity check (both directions, bounded) and a rank query (inorder counting). Today uses that same invariant for a decision — Lowest Common Ancestor of a BST asks "which single direction do I go?" rather than "is everything in range?" — and then deliberately steps away from BSTs entirely for the day's second problem: Construct Binary Tree from Preorder and Inorder Traversal works on a **plain** tree, rebuilt from two flat arrays with no ordering invariant at all. That second problem is where today's real new idea lives — divide-and-conquer, formalized for the first time with a divide step that requires genuine computation, not a free structural split.

## Learning Objectives

By the end of today, without notes:

1. Solve Lowest Common Ancestor of a BST, explaining why the ordering invariant collapses a two-direction search into a single-direction walk.
2. State the three-step divide/conquer/combine shape precisely, and explain what's genuinely new about today's divide step compared to every tree DFS problem since Day 46.
3. Solve Construct Binary Tree from Preorder and Inorder Traversal, including why a HashMap lookup (not a linear scan) is required to keep it O(n).
4. Explain the difference between Spring profiles (`application.yml` + `spring.profiles.active`) and the full Spring Cloud Config Server pattern, and why they solve different-sized versions of the same underlying problem.

## Concept Dependency Map for Today

```
Day 50: BST ordering invariant (left smaller / right larger, globally)
        │
        ▼
Problem 11: Lowest Common Ancestor of a BST (LC 235)
  needs: ordering invariant, used to pick ONE direction instead of validating BOTH

Day 46: TreeNode shape, universal recursive template
Day 35 (light mention only): "divide-and-conquer" named as an LC 23 alternative, not built out
        │
        ▼
NEW: Divide and Conquer, formalized — the divide step now requires COMPUTED work,
     not a free node.left/node.right split
        │
        ▼
Problem 12: Construct Binary Tree from Preorder and Inorder Traversal (LC 105)
  needs: HashMap O(1) lookup (Day 4/5) — required to keep the divide step itself O(1) per call

Extra Practice: Construct Binary Tree from Inorder and Postorder Traversal (LC 106)
  needs: today's Problem 12 skeleton, mirrored — postorder read backward, right built before left

Day 34/36: application.yml already in use (Postgres connection properties)
        │
        ▼
NEW: Spring Profiles — spring.profiles.active, environment-specific override files
```

---

# Part 1 — Lowest Common Ancestor of a BST

## Problem 11: Lowest Common Ancestor of a BST (LeetCode 235, Medium) — Pattern: BST Traversal

**Statement:** Given the root of a BST and two nodes `p` and `q` (both guaranteed to exist in the tree), find their lowest common ancestor — the deepest node that has both `p` and `q` as descendants (a node can be its own descendant, for this problem's definition).

### Approach — walk down, using the ordering invariant to pick a direction

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    TreeNode current = root;
    while (current != null) {
        if (p.val < current.val && q.val < current.val) {
            current = current.left;              // both targets are smaller — LCA must be further left
        } else if (p.val > current.val && q.val > current.val) {
            current = current.right;              // both targets are larger — LCA must be further right
        } else {
            return current;                       // targets split here, or current IS one of them
        }
    }
    return null;   // unreachable given the problem's guarantee that both nodes exist
}
```

**The reframe the BST invariant enables:** at any node, if both `p.val` and `q.val` are smaller than the current node's value, the *entire* subtree containing both of them is the left subtree — no need to check the right subtree at all, no need to check both directions the way Validate BST did. The moment `p` and `q` stop being on the same side (one is `≤` current, the other `≥`, or the current node itself is `p` or `q`), the search stops — that node is provably the LCA, because it's the last point where a path to `p` and a path to `q` were still forced to go through the same node.

**Why this is fundamentally easier than the general-tree version of this same question** (arriving tomorrow, Day 52): a plain binary tree gives no way to know which direction a target value is in without actually searching both directions and seeing what comes back. A BST's ordering invariant answers that question in O(1) at every node — you don't need to search both sides, you can *compute* which side to search. That's the entire reason this version can be solved iteratively in O(1) space, while the general version needs recursion to combine results from both children.

**Complexity: Time O(h), Space O(1)** (iterative) — or O(h) space if written recursively instead, which is a legitimate equivalent approach, just without the space savings.

**Edge cases:** `p` or `q` equal to the root (returns the root immediately — correct, since a node is its own ancestor here); `p` is an ancestor of `q` or vice versa (the loop correctly stops at the ancestor itself, since the moment `current` equals one of the targets, the `else` branch fires).

> 🔑 **Key Takeaway:** this is the same "use the invariant to eliminate a whole subtree without searching it" move from Day 50's Validate BST and from Binary Search itself (Day 28) — a different application every time, but the same underlying leverage: an ordering guarantee turns "search both sides" into "compute which one side."

---

# Part 2 — Divide and Conquer, Formalized

## What's actually new here

Every tree DFS problem since Day 46 — Maximum Depth, Diameter, Balanced, all of them — technically already follows a divide/conquer/combine shape: split into `node.left` and `node.right`, solve each recursively, combine the results (`1 + max(...)`, or similar). That's not being presented as new today, because the **divide** step in every one of those problems was **free** — the tree handed you `node.left` and `node.right` directly, no computation required to find them.

**Construct Binary Tree from Preorder and Inorder Traversal is different in exactly one respect:** you're not given a tree at all. You're given two flat arrays — `preorder` and `inorder` — with no left/right pointers anywhere. Figuring out *where the divide even is* — which elements belong to the left subtree and which belong to the right — is itself real, computed work that has to happen **before** either recursive call can be made. This is the first problem in the series where that's true, and it's exactly why it's the natural place to formalize divide-and-conquer as its own named technique, rather than continuing to treat "combine left/right" as something that only ever falls out of a tree's existing shape.

**The three steps, named precisely:**
1. **Divide** — split the problem into independent subproblems. (Today: figure out which slice of `preorder` and which slice of `inorder` belong to the left subtree vs. the right subtree.)
2. **Conquer** — solve each subproblem recursively, with a base case for the trivial size.
3. **Combine** — merge the subproblem solutions into the answer for the original problem. (Today: attach the recursively-built left and right subtrees to the newly-created root node.)

> 🔗 **Backward reference:** "divide-and-conquer" was named once before, in passing, on Day 35 — as one of two valid approaches to Merge k Sorted Lists (the other being a heap). It wasn't built out with real depth there. Today is that build-out.

---

## Problem 12: Construct Binary Tree from Preorder and Inorder Traversal (LeetCode 105, Medium) — Pattern: Divide and Conquer

**Statement:** Given `preorder` and `inorder` traversal arrays of the same binary tree (no duplicate values), reconstruct and return the tree.

### The two facts that make this solvable

1. **Preorder's first element is always the root.** Preorder visits root, then left subtree, then right subtree — so whatever comes first in the array is, by definition, the root of whatever subtree is currently being reconstructed.
2. **Finding that root's value inside `inorder` tells you the size of each subtree.** Inorder visits left subtree, then the node, then right subtree — so everything to the left of the root's position in `inorder` belongs entirely to the left subtree, and everything to the right belongs entirely to the right subtree. The root's index in `inorder` isn't just "where the root is" — it's a **count**: that many elements precede it (left subtree size), and the rest follow it (right subtree size).

### Approach — recursive divide and conquer, HashMap-accelerated

```java
private int preorderIndex;
private Map<Integer, Integer> inorderIndexOf;

public TreeNode buildTree(int[] preorder, int[] inorder) {
    preorderIndex = 0;
    inorderIndexOf = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) {
        inorderIndexOf.put(inorder[i], i);   // value -> index, O(1) lookup instead of a linear scan
    }
    return build(preorder, 0, inorder.length - 1);
}

private TreeNode build(int[] preorder, int inStart, int inEnd) {
    if (inStart > inEnd) return null;   // this subtree's inorder range is empty

    int rootVal = preorder[preorderIndex++];   // DIVIDE step: the next unconsumed preorder value is always the next root
    TreeNode root = new TreeNode(rootVal);

    int rootIndex = inorderIndexOf.get(rootVal);   // DIVIDE step: this index tells us exactly where left ends and right begins

    root.left = build(preorder, inStart, rootIndex - 1);     // CONQUER: left subtree
    root.right = build(preorder, rootIndex + 1, inEnd);      // CONQUER: right subtree
                                                                // COMBINE: attach both to root, implicitly, by returning root
    return root;
}
```

**Why `preorderIndex` can be a single shared counter, not a separately-tracked range per call:** preorder visits root, *then the entire left subtree*, then the entire right subtree. As long as the left recursive call is made **before** the right one — which it is, on the line above — the shared, incrementing index naturally lines up with what preorder would visit next, no matter how deep the recursion or which subtree is currently being built. This only works because the calls happen in exactly the order preorder itself would visit them; swapping the order of the `root.left =` / `root.right =` lines would desynchronize the index and silently build the wrong tree.

**Why the HashMap, specifically:** without it, finding `rootVal`'s position in `inorder` requires a linear scan every time a new root is placed — O(n) work, n times, giving O(n²) overall. Precomputing every value's index once, up front, turns that into an O(1) lookup per root — the same complement-lookup leverage from Day 5's Two Sum, applied here to avoid repeated scanning rather than repeated pairwise checking.

**Complexity: Time O(n)** with the HashMap (O(n²) without it — worth stating both, since "why the HashMap" is a natural follow-up). **Space O(n)** for the map, plus **O(h)** for the recursion stack.

**Edge cases:** a single-node tree (`inStart == inEnd`, immediately returns a leaf); an empty tree (both arrays empty — LeetCode guarantees at least one node, but the `inStart > inEnd` base case handles it correctly regardless); the problem's own guarantee of no duplicate values is load-bearing — with duplicates, a value's position in `inorder` wouldn't uniquely identify which occurrence is the root, and the HashMap-lookup approach would break.

> ⚠️ **Common Mistake:** building the right subtree before the left. It compiles, it looks harmless, and it silently produces a wrong tree — `preorderIndex` gets consumed out of order, so later "roots" are actually values that belonged to a different part of the tree entirely. If a Construct Binary Tree solution is producing a structurally-plausible-looking but wrong tree, this ordering is the first thing to check.

---

## Extra Practice 1: Construct Binary Tree from Inorder and Postorder Traversal (LeetCode 106, Medium) — Pattern: Divide and Conquer, Mirrored

**Statement:** Same task as Problem 12, given `inorder` and `postorder` instead of `preorder` and `inorder`.

**Why this is today's extra:** identical divide-and-conquer shape, same HashMap-for-O(1)-lookup optimization — but postorder visits **left, right, root**, the reverse of preorder's root-first order. That single change flips two things that are easy to get backward if you're pattern-matching from yesterday's code rather than re-deriving it.

```java
private int postorderIndex;
private Map<Integer, Integer> inorderIndexOf;

public TreeNode buildTree(int[] inorder, int[] postorder) {
    postorderIndex = postorder.length - 1;   // start from the END — postorder's LAST element is the root
    inorderIndexOf = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) {
        inorderIndexOf.put(inorder[i], i);
    }
    return build(postorder, 0, inorder.length - 1);
}

private TreeNode build(int[] postorder, int inStart, int inEnd) {
    if (inStart > inEnd) return null;

    int rootVal = postorder[postorderIndex--];   // walking BACKWARD
    TreeNode root = new TreeNode(rootVal);

    int rootIndex = inorderIndexOf.get(rootVal);

    // Build RIGHT before LEFT — postorder is [left..., right..., root], so reading
    // backward from the root encounters the right subtree's nodes before the left's.
    root.right = build(postorder, rootIndex + 1, inEnd);
    root.left = build(postorder, inStart, rootIndex - 1);

    return root;
}
```

**The two flips, stated explicitly:**
1. **Start position:** preorder's root is at the *front* (index 0, walking forward); postorder's root is at the *back* (last index, walking backward).
2. **Recursion order:** Problem 12 builds left before right, because preorder lists the left subtree before the right subtree. This extra builds **right before left**, because reading postorder backward from the root encounters the right subtree's (reversed) elements first, then the left subtree's.

**Trace, to make the flip concrete:** `inorder = [9,3,15,20,7]`, `postorder = [9,15,7,20,3]`. `postorderIndex` starts at 4 (value `3`) — root. Its `inorder` index is 1, so left subtree is `inorder[0..0] = [9]` (size 1) and right subtree is `inorder[2..4] = [15,20,7]` (size 3). Building right first: `postorderIndex` decrements to 3 (value `20`) — correctly the right subtree's root, *not* `15`, because postorder's last three entries before the final `3` are `[9, 15, 7, 20]`'s tail — specifically `20` is the most recently-visited node before `3` itself, consistent with `20` being visited last within the right subtree (postorder visits a subtree's own root last, always). Continuing this consumes `7`, then `15`, correctly finishing the right subtree before `postorderIndex` ever reaches `9` — the left subtree's sole node — confirming the right-before-left order is required for the pointer to land on the correct values at each step.

**Complexity: Time O(n), Space O(n) + O(h)** — identical to Problem 12, only the traversal direction and build order differ.

**Edge cases:** identical to Problem 12's list, plus this one specific new failure mode — building left before right here doesn't just misplace nodes, it can desynchronize `postorderIndex` badly enough to throw an `ArrayIndexOutOfBoundsException` or `NullPointerException` on some inputs, rather than merely producing a wrong-but-valid-looking tree, since the index can walk past the values that actually belong to the current subtree.

> 💡 **Interview Insight:** being asked "now do it with postorder instead" right after solving the preorder version is a very common follow-up. The strong answer isn't re-deriving from scratch — it's naming the two things that flip (start position, build order) and explaining *why* each flips, in one or two sentences, before touching the keyboard.

---

# Part 3 — Spring Profiles and Externalized Configuration

## The problem being solved

`todo-api` has had a Postgres connection configured in `application.yml` since Day 36. As written, that configuration is one fixed set of values — a database URL, credentials, maybe a `ddl-auto` setting — baked into the build. Deploying the exact same build artifact to a local dev environment, a staging environment, and production, each of which needs different values for at least the database URL, means either maintaining separate builds per environment (defeating the point of "build once, deploy everywhere") or hand-editing the config file before each deploy (fragile, and a direct invitation for a dev credential to end up in production by mistake).

**Externalized configuration** solves this by separating "what varies per environment" from the build itself, so **one build artifact** behaves differently depending on which environment it's told it's running in.

## Spring Profiles

A **profile** is a named configuration variant. `application.yml` holds whatever's shared across every environment; profile-specific files hold only the values that differ.

```yaml
# application.yml — shared, environment-independent defaults
spring:
  application:
    name: todo-api
server:
  port: 8080
```

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/todo_dev
    username: dev_user
    password: dev_pass
logging:
  level:
    root: DEBUG
```

```yaml
# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://prod-db-host:5432/todo_prod
    username: ${DB_USERNAME}    # pulled from an environment variable, never hardcoded
    password: ${DB_PASSWORD}
logging:
  level:
    root: WARN
```

**Which profile is active** is controlled by `spring.profiles.active`, set at startup — not baked into any file that ships with the build:

```bash
java -jar todo-api.jar -Dspring.profiles.active=dev
java -jar todo-api.jar -Dspring.profiles.active=prod
```

At startup, Spring loads `application.yml` first, then layers the active profile's file on top, with profile-specific values taking precedence wherever a key appears in both. The result: the exact same JAR file, unmodified, produces different runtime behavior purely based on a flag supplied at launch — which is precisely the property needed once separate Dev, Staging, and Production environments exist to support.

**`@Profile`** extends this to entire Spring beans, not just property values — a bean annotated `@Profile("dev")` is only registered when the `dev` profile is active, useful for swapping an entire component (a mock email sender in dev, a real one in prod) rather than just a config value.

## What this is *not*: Spring Cloud Config Server

Worth being precise about scope: what's being built today is **Spring profiles** — a mechanism for one service to hold multiple named configuration variants, selected at its own startup. This solves the single-service, multi-environment problem cleanly.

**Spring Cloud Config Server** is a different, heavier tool for a different-shaped problem: a *separate microservice* that centrally serves configuration to many other services at once, typically backed by a Git repository (so config changes are versioned and auditable like code), with `@RefreshScope` allowing a running service to pick up a config change *without restarting*. That's the right tool once an organization has many services that need centrally-managed, dynamically-updatable configuration — a meaningfully bigger problem than one service picking between `dev` and `prod` at its own launch. Today's work is a real, complete, and correct solution to today's actual problem; it's just a different (and smaller) tool than the name "Spring Cloud Config" might suggest if the two are conflated.

---

# Section — Project Block

**Repository:** `todo-api`. **Task:** move hardcoded config into `application.yml` profiles (`dev`, `prod`), selected via `spring.profiles.active`.

**Definition of done:** running with `-Dspring.profiles.active=dev` vs. `-Dspring.profiles.active=prod` visibly changes behavior — at minimum, confirm the datasource URL actually differs (log it at startup, or check which database the app connects to) between the two runs.

# Section — Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts. No new post today; keeping the visibility-without-authorship rhythm going between the more substantial posts.

**Networking:** send connection requests to the 3 Tier B targets identified yesterday. Reference something specific from each profile — a recent post, a shared connection, the actual team they're on — rather than a generic request.

---

# Day 51 — Interview Questions

**1. Why does Lowest Common Ancestor of a BST only need to check ONE direction at each step, unlike Validate BST?**

*Answer:* The BST ordering invariant means if both target values are smaller than the current node, the entire subtree containing both must be the left subtree — no need to also check the right subtree. Validate BST needs to bound both directions because it's confirming a global property holds everywhere, not making a single navigational decision.

---

**2. What makes LCA of a BST solvable iteratively in O(1) space, while the general-tree version (tomorrow) typically needs recursion?**

*Answer:* The ordering invariant lets you *compute* which single direction to go at every step, with no need to search both sides. A plain tree gives no such shortcut — you have to actually search both children and see what each returns, which is naturally expressed as a recursive combine.

---

**3. State the three steps of divide-and-conquer precisely.**

*Answer:* Divide — split the problem into independent subproblems. Conquer — solve each recursively, with a base case. Combine — merge the subproblem results into the overall answer.

---

**4. Every tree DFS problem since Day 46 already followed a divide/conquer/combine shape. What's actually new about Construct Binary Tree from Preorder and Inorder?**

*Answer:* In every earlier problem, the divide step was free — the tree handed you `node.left` and `node.right` directly. Here, there's no tree yet, only two flat arrays; figuring out which elements belong to the left subtree and which to the right requires real computed work (finding the root's index in `inorder`) before recursion can even begin.

---

**5. Why is preorder's first element always the root, and why does that fact alone not tell you the subtree sizes?**

*Answer:* Preorder visits root, then left subtree, then right subtree — so the first element is always whatever root is currently being placed. But it says nothing about how many of the remaining elements belong to the left subtree vs. the right; that split has to come from a second source — inorder's root position, in this problem.

---

**6. Why must the left recursive call happen before the right one when using a shared `preorderIndex` counter?**

*Answer:* The shared counter only stays synchronized with what preorder would visit next if the recursive calls consume it in the same order preorder itself lists — root, then the entire left subtree, then the entire right subtree. Building right before left would consume index positions meant for the left subtree while building the right one, silently producing a wrong tree.

---

**7. Why does a HashMap turn this problem from O(n²) into O(n)?**

*Answer:* Without it, finding a root value's position in `inorder` requires an O(n) linear scan, repeated once per node placed — O(n²) total. Precomputing every value's index once up front makes each lookup O(1), for O(n) total across all n placements.

---

**8. Going from preorder+inorder to inorder+postorder (today's extra), what two things flip, and why?**

*Answer:* The root's position flips from the front of the array (preorder lists root first) to the back (postorder lists root last), so the pointer must start at the end and walk backward. The build order flips from left-then-right to right-then-left, because reading postorder backward from the root encounters the right subtree's elements before the left subtree's.

---

**9. Why does the problem's "no duplicate values" guarantee matter for the HashMap-lookup approach?**

*Answer:* The technique relies on a value's position in `inorder` uniquely identifying it. With duplicate values, a value could appear at more than one index, and a single HashMap entry couldn't correctly distinguish which occurrence is the actual root being placed.

---

**10. What's the practical difference between Spring profiles and Spring Cloud Config Server?**

*Answer:* Spring profiles let one service hold multiple named configuration variants, chosen at its own startup via `spring.profiles.active` — solving the single-service, multi-environment problem. Spring Cloud Config Server is a separate microservice that centrally serves configuration to many services at once, typically Git-backed and capable of live updates via `@RefreshScope` — a different tool for centrally managing config across many services, not just switching one service between environments.

---

**11. How does `@Profile` differ from a profile-specific property in `application-dev.yml`?**

*Answer:* A profile-specific property overrides a configuration *value*. `@Profile` controls whether an entire Spring *bean* is registered at all — useful for swapping a whole component (e.g., a mock vs. a real implementation) based on the active profile, not just a setting on one shared component.

---

## Daily Deliverable Check

- [ ] Lowest Common Ancestor of a BST and Construct Binary Tree from Preorder/Inorder solved, pushed, with the divide-and-conquer shape explainable precisely — including what's actually new about it versus prior tree DFS.
- [ ] Extra Practice: Construct Binary Tree from Inorder/Postorder solved — the two flips (start position, build order) stated correctly without re-deriving from scratch.
- [ ] `dev`/`prod` profiles working in `todo-api`, confirmed via an actually-different datasource URL between the two runs.

---

## What Tomorrow Assumes You Already Know Cold

Day 52 assumes today's divide-and-conquer distinction is solid — Serialize/Deserialize Binary Tree is a genuinely different technique (preorder with explicit null markers), but general-tree Lowest Common Ancestor tomorrow builds directly on today's LCA-of-a-BST intuition, this time *without* an ordering invariant to lean on, which is exactly what makes it need both children searched and combined via recursion rather than a single computed direction.

**Next:** [Day 52 Resource Book](./Day52_Resource_Book.md) — Trees Beyond BST: General LCA and Serialization, and WireMock.
