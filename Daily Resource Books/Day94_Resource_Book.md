# Day 94 — Tree DP, and Dynamic Programming Closes Entirely

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 93 Resource Book](Day93_Resource_Book.md)
**Next ▶:** [Day 95 Resource Book](Day95_Resource_Book.md)
**Companion to:** Day 94 of `Week_14_Revised.md`

---

## Recap

State Machine DP closed yesterday at 4/4, adding a *situation* dimension to `dp[i]`. Today adds a different kind of dimension — not a situation, and not even an index in the usual sense: **the recursion itself is the loop**, and each tree node's postorder return value becomes the DP state.

Two things carry forward directly, one of them cited only in passing:

- **House Robber's take-or-skip recurrence** (Week 12, Day 82) — proven by disjoint-exhaustive cases (rob this element, or don't; those two options cover every possibility, and taking the max over both is therefore always correct). That proof is assumed **fully solid** today, not re-derived — House Robber III extends it directly onto a tree.
- **Diameter of Binary Tree** (Week 7, Day 48) — already built half of today's mechanism: a value computed bottom-up, combined from both children, updating something outside the return value itself. Worth a brief reconnection below, not a re-teach — it was already fully covered.

---

## Learning Objectives

By the end of today, without notes:

1. Define Tree DP precisely, and state exactly what's genuinely new about it versus Day 48's tree recursion.
2. Solve House Robber III by extending House Robber's take-or-skip recurrence onto a tree, returning a *pair* of DP states per node.
3. Solve Binary Tree Maximum Path Sum, and explain why its shape is actually closer to Day 48's Diameter than to House Robber III's — despite both being "Tree DP."
4. Reconcile the plan's "35 required DP problems" figure against this map's distinct-newly-taught count, by hand, from the numbers.

---

## Concept Dependency Map

```
Week 7, Day 48: Diameter of Binary Tree (LC 543)
  "running max OUTSIDE the return value" — height returned, diameter tracked separately
        │
        │  (brief recap — not re-taught; already fully covered)
        ▼
Week 12, Day 82: House Robber (LC 198) — take-or-skip, disjoint-exhaustive proof
        │
        ▼
Tree DP (NEW): dp(node) → the return value(s) themselves ARE the DP state(s),
               combined from left/right via postorder — divide step free (Day 51),
               combine step is what's new
        │
        ├──▶ LC 337 House Robber III — returns a PAIR {robbed, notRobbed}: dual DP states
        │
        └──▶ LC 124 Max Path Sum — returns ONE value + updates an outside global:
                                     closer to Day 48's shape than to LC 337's

        ↓
[Extension] LC 968 Binary Tree Cameras — 3-state greedy tree DP, if time allows

DYNAMIC PROGRAMMING CLOSES: 35/35 required slots, Weeks 12–14
```

---

# Part 1 — Tree Dynamic Programming

## Prerequisites (confirmed)

- The universal recursive tree template — base case for `null`, combine `left`/`right` — and O(h) (not a defaulted O(n)) as the correct space-complexity statement for a tree recursion's call stack (Week 7, Day 46).
- Divide and conquer, formalized: divide, conquer, combine — every tree DFS problem since Day 46 has had a **free** divide step (`node.left`/`node.right`, no computation) (Week 8, Day 51).
- House Robber's take-or-skip recurrence and its disjoint-exhaustive-cases proof (Week 12, Day 82) — assumed solid, cited, not re-derived.

## What it is

Tree DP is what you get when the "loop" a normal DP recurrence would run over is replaced by a **postorder tree traversal**: `dp(node)` is computed from `dp(node.left)` and `dp(node.right)`, exactly the way `dp[i]` is computed from `dp[i-1]` (or further back) in every DP subtype so far — except the "one step back" relation is now "my children," and there can be up to two of them instead of exactly one.

## What's genuinely new versus Day 48

Day 48's Diameter of Binary Tree already had a function returning a value built from both children's returned values — that part isn't new. What's new is **what the return value is allowed to represent**. In Diameter, the returned `height` is a single supporting number whose only job is helping the *parent's own* computation; the actual answer (the diameter) lives in a side-effect variable, entirely outside the return value. In House Robber III below, the return value doesn't just support the parent — **it directly IS the full DP state**, in the same sense `dp[i][state]` was the full state on Days 92–93, just indexed by node instead of by day. That's the precise thing to be able to name if pushed on "isn't this just what we already did in Week 7?"

## Why it works

Optimal substructure: the best outcome at `node` is fully determined by the best outcomes at `node.left` and `node.right`, plus whatever `node` itself contributes — nothing outside `node`'s subtree can change that, since a binary tree has no cross-links back up or sideways. Overlapping subproblems: for a general graph this would need memoization to avoid recomputation, but a *tree* has no shared substructure between siblings by definition — each node is visited exactly once in a postorder pass, so the "overlap" that justified memoization everywhere else in DP is structurally absent here. **The DP win in a tree isn't avoiding recomputation (there was never any to avoid) — it's avoiding the exponential *blow-up* a brute-force version gets from recomputing a child's answer once per each of the parent's own branching choices**, shown concretely below.

## When to reach for it

The signal: a problem defines a value **per node** that depends on a **bounded, small combination** of its children's own values — "the best X in this subtree," where X composes cleanly bottom-up. Distinguish from plain tree DFS (Days 46–52): if the return value is only ever a simple, single fact (a height, a boolean, an equality check) with no *choice* being optimized over, it's still just tree recursion, not Tree DP. The moment a node has to choose between mutually exclusive options — and the best choice depends on what the children's own best choices were — it's Tree DP.

## Trade-offs against the nearest alternative (naive recomputation)

| | Recompute each subtree's answer per parent decision | Tree DP (postorder, return the state) |
|---|---|---|
| Correctness | Fine, but see complexity | Fine |
| Complexity | Exponential — see House Robber III's brute force below | O(n) — every node visited once |
| What it needs | Nothing extra | Return value(s) must carry every state a parent could need — decide this up front |

## Complexity, with reasoning

Every node is visited exactly once in the postorder pass, doing O(1) work combining its (already-computed) children's states — **O(n) time**, total, across the whole tree. Space is the recursion call stack: **O(h)**, where `h` is the tree's height — O(log n) if balanced, O(n) worst case for a fully skewed tree (the same distinction Day 46 established as the precise statement, not a defaulted O(n)).

---

## Problem: House Robber III (LeetCode 337, Medium)

**Statement:** Houses are arranged as a binary tree; robbing two directly-connected houses (parent and child) triggers an alarm. Maximize total value robbed.

### Approach 1 — Brute force (naive recursion, no state carried)

```java
public static int robBruteForce(TreeNode node) {
    if (node == null) return 0;

    // Option 1: rob this node — cannot rob its direct children, but CAN rob grandchildren
    int robThis = node.val;
    if (node.left != null) {
        robThis += robBruteForce(node.left.left) + robBruteForce(node.left.right);
    }
    if (node.right != null) {
        robThis += robBruteForce(node.right.left) + robBruteForce(node.right.right);
    }

    // Option 2: skip this node — children are each free to be robbed or not, take their best
    int skipThis = robBruteForce(node.left) + robBruteForce(node.right);

    return Math.max(robThis, skipThis);
}
```

**Why this is exponential:** `skipThis` calls `robBruteForce(node.left)` and `robBruteForce(node.right)` — and `robThis` **also** separately recomputes `node.left`'s and `node.right`'s subtrees (by reaching into the grandchildren directly). Every node ends up re-explored from multiple different ancestors' calls, with no memoization — the same overlapping-recomputation signature fib(5)'s call tree demonstrated on Day 81, just shaped like a tree instead of a line. **Time: O(2ⁿ)**-ish (exact bound depends on tree shape, but it's exponential in the worst case). **Space:** O(h) call stack, ignoring the wasted recomputation itself.

### Approach 2 — Optimized: return both states from every node

```java
public static int rob(TreeNode root) {
    int[] result = robHelper(root);            // {maxIfNotRobbed, maxIfRobbed}
    return Math.max(result[0], result[1]);
}

private static int[] robHelper(TreeNode node) {
    if (node == null) return new int[]{0, 0};

    int[] left = robHelper(node.left);
    int[] right = robHelper(node.right);

    int notRobbed = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);  // children free to be robbed or not
    int robbed = node.val + left[0] + right[0];                                  // children FORCED not-robbed

    return new int[]{notRobbed, robbed};
}
```

**Why this eliminates the recomputation:** each node now returns *both* answers a parent could ever need — "best if you don't rob me" and "best if you do" — in a single postorder visit. A parent deciding whether to rob itself never needs to re-descend into a child's subtree to find out; both possible answers are already sitting in that child's returned pair. This is precisely House Robber's take-or-skip recurrence (Day 82) — `notRobbed` mirrors "skip," taking each child's own best regardless of its state; `robbed` mirrors "take," forcing both children into their not-robbed state — extended onto two children instead of one linear predecessor.

**Worked trace:** tree `[3,2,3,null,3,null,1]` (LeetCode's own example 1; expected answer `7`):

```
        3
       / \
      2   3
       \    \
        3    1
```

- Leaf `3` (child of `2`): `left=right={0,0}` → `notRobbed=0`, `robbed=3+0+0=3` → returns `{0, 3}`.
- Leaf `1` (child of the right `3`): same shape → returns `{0, 1}`.
- Node `2`: `left={0,0}` (no left child), `right={0,3}` (the leaf `3` above) → `notRobbed = max(0,0) + max(0,3) = 0+3 = 3`; `robbed = 2 + 0 + 0 = 2` → returns `{3, 2}`.
- Node `3` (root's right child): `left={0,0}`, `right={0,1}` (the leaf `1`) → `notRobbed = 0 + 1 = 1`; `robbed = 3 + 0 + 0 = 3` → returns `{1, 3}`.
- Root `3`: `left = {3,2}` (node `2`'s result), `right = {1,3}` (node `3`'s result) → `notRobbed = max(3,2) + max(1,3) = 3+3 = 6`; `robbed = 3 + left[0] + right[0] = 3 + 3 + 1 = 7` → returns `{6, 7}`.

Final: `max(6, 7) = 7`. Matches. (Reconstruction: rob the root `3`, plus the two leaf `3` and leaf `1` grandchildren, skipping both middle `2`/`3` children — `3+3+1=7`.)

**Complexity:** Time **O(n)**, Space **O(h)** call stack — same bounds as any tree DFS (Day 46).

**Edge cases:**
- `null` root: `robHelper` returns `{0,0}` for a `null` node by definition, so `rob(null)` correctly yields `0`.
- Single node: `left=right={0,0}` → `notRobbed=0`, `robbed=node.val` → correctly returns `node.val`.
- A long single chain (effectively a linked list dressed as a tree): the recurrence degenerates to exactly House Robber's original 1D recurrence, one node "deep" at a time — worth stating explicitly if asked how this generalizes the original problem.

**💡 Interview Insight:** Say, before coding, "each node needs to return two numbers, not one — the best answer assuming I'm robbed, and the best answer assuming I'm not" — naming *why* a single return value is insufficient here is the actual signal; jumping straight to a two-element array with no stated justification looks memorized. Likely follow-up: *"what changes for an n-ary tree instead of binary?"* — answer: sum every child's `max(notRobbed, robbed)` for the not-robbed case, and every child's `notRobbed` alone for the robbed case, generalizing cleanly since nothing about the argument depended on exactly two children.

---

## Problem: Binary Tree Maximum Path Sum (LeetCode 124, Hard)

**Statement:** A "path" is any sequence of nodes connected by edges, without revisiting a node; it need not pass through the root, and need not end at a leaf. Find the maximum sum of node values along any path.

### Why this is NOT the same shape as House Robber III

House Robber III's return value pair **is** the complete DP state — nothing is tracked outside it. Here, a path can **bend** at a node (go up through the left child, through the node itself, and back down through the right child) — but a value **returned to a parent** can only continue **straight through** the node in one direction, because the parent can only attach the node to *its own* single path in one direction, not fork it. So the "best path bending through this node" and "the best extendable-upward value this node can offer its parent" are two different questions, and only the second one can be a return value. The first has to be tracked separately, **exactly like Day 48's diameter** — a running value maintained outside the return, updated as a side effect at every node.

### Approach — postorder, single return value, outside global for the real answer

```java
public class MaxPathSumSolution {
    private int maxSum = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        maxGain(root);
        return maxSum;
    }

    private int maxGain(TreeNode node) {
        if (node == null) return 0;

        // Clamp at 0: a negative contribution should never be included — skipping
        // a child entirely is always at least as good as adding a negative sum.
        int leftGain = Math.max(maxGain(node.left), 0);
        int rightGain = Math.max(maxGain(node.right), 0);

        // The path BENDING through this node (both children) — candidate for the real answer,
        // never returned upward, since a parent can only extend ONE direction.
        maxSum = Math.max(maxSum, node.val + leftGain + rightGain);

        // What THIS node offers its own parent: itself plus the better single branch only.
        return node.val + Math.max(leftGain, rightGain);
    }
}
```

**Why the `Math.max(_, 0)` clamp is required, precisely:** without it, a strongly negative subtree would still get added into `leftGain`/`rightGain`, actively *reducing* a sum that would have been better off simply not extending into that child at all. Clamping to `0` encodes "you may choose not to extend the path into a child," which is always a legal choice for this problem (a path can start and end anywhere) — the clamp is what makes that choice available to the recurrence.

**Worked trace:** `[-10, 9, 20, null, null, 15, 7]` (LeetCode's own example; expected answer `42`):

```
        -10
        /  \
       9    20
           /  \
          15   7
```

- `maxGain(15)`: leaf. `leftGain=rightGain=0`. `maxSum = max(−∞, 15+0+0) = 15`. Returns `15 + max(0,0) = 15`.
- `maxGain(7)`: leaf. `maxSum = max(15, 7) = 15` (unchanged). Returns `7`.
- `maxGain(9)`: leaf (no children). `maxSum = max(15, 9) = 15` (unchanged). Returns `9`.
- `maxGain(20)`: `leftGain = max(15,0)=15`, `rightGain = max(7,0)=7`. `maxSum = max(15, 20+15+7) = max(15,42) = 42`. Returns `20 + max(15,7) = 35`.
- `maxGain(−10)` (root): `leftGain = max(9,0)=9`, `rightGain = max(35,0)=35`. `maxSum = max(42, −10+9+35) = max(42,34) = 42` (unchanged — the root's own bending path, `34`, loses to `20`'s bending path, `42`). Returns `−10+35=25` (irrelevant — nothing above the root consumes it).

Final `maxSum = 42`. Matches.

**Complexity:** Time **O(n)**, Space **O(h)** — identical shape to House Robber III and to Day 48's Diameter.

**Edge cases:**
- All-negative tree: the answer is still the single **largest** node value (a "path" of length one is legal) — the clamp-to-`0` logic handles this correctly, since every `leftGain`/`rightGain` collapses to `0` and `maxSum` ends up tracking the best single `node.val` seen.
- Single node: `maxSum` becomes exactly `node.val`, correctly.
- The optimal path never touching the root: fully handled — `maxSum` is updated at **every** node during the traversal, not only at the root.

**💡 Interview Insight:** State the "why a single return value can't be the whole answer here" reasoning *before* writing the outside `maxSum` variable — this is the single most-tested piece of understanding on this exact problem, precisely because House Robber III (just solved) primes the wrong instinct ("return everything the parent could need"). Explicitly flagging that this problem breaks that instinct, and why, is a strong, specific signal.

**🔗 Connects to:** Diameter of Binary Tree (Week 7, Day 48) — same "running value outside the return" shape; today's version adds a clamp for negative contributions, which Diameter (heights are never negative) never needed.

---

## [Extension — Time Permitting] Binary Tree Cameras (LeetCode 968, Hard)

*Marked as extension, not core material — the two required problems above already deliver Tree DP's central lesson. This is here because it's a genuinely different flavor (a greedy argument layered onto postorder tree DP, 3 states instead of 2) that neither required problem covers, not because today's required list was thin.*

**Statement:** Each camera monitors itself, its parent, and its direct children. Place the minimum number of cameras so every node is monitored.

**The greedy insight:** process postorder (leaves first). A leaf should **never** get a camera — placing one instead at the leaf's *parent* covers strictly more (the parent, the parent's other children, and the leaf itself). Generalizing: place a camera at a node **only when forced** — specifically, the moment any child is "not yet covered by anything."

```java
public class CamerasSolution {
    private int cameras = 0;
    private static final int NOT_COVERED = 0, COVERED_NO_CAMERA = 1, HAS_CAMERA = 2;

    public int minCameraCover(TreeNode root) {
        if (dfs(root) == NOT_COVERED) cameras++;   // root itself still uncovered — must place one
        return cameras;
    }

    private int dfs(TreeNode node) {
        if (node == null) return COVERED_NO_CAMERA;   // a null child needs nothing and forces nothing

        int left = dfs(node.left);
        int right = dfs(node.right);

        if (left == NOT_COVERED || right == NOT_COVERED) {
            cameras++;
            return HAS_CAMERA;
        }
        if (left == HAS_CAMERA || right == HAS_CAMERA) {
            return COVERED_NO_CAMERA;
        }
        return NOT_COVERED;
    }
}
```

**Why `null` returns `COVERED_NO_CAMERA`, not `NOT_COVERED`:** if a missing child forced its (nonexistent) parent to place a camera, every leaf would be treated as having an uncovered child and would immediately force a camera at itself — exactly the wasteful placement the greedy argument above says to avoid. Treating "no child" as "already covered" is what correctly defers the decision to the parent instead.

**Trace:** `[0,0,null,0,0]` (LeetCode's example 1; expected `1`) — a root with one left child, which itself has two leaf children. Both leaves: `dfs` sees two `COVERED_NO_CAMERA` children (both `null`), returns `NOT_COVERED`. Their parent: sees two `NOT_COVERED` children → places a camera (`cameras=1`), returns `HAS_CAMERA`. Root: sees one `HAS_CAMERA` child → returns `COVERED_NO_CAMERA`, no further camera needed. `minCameraCover` checks the root's own return (`COVERED_NO_CAMERA`, not `NOT_COVERED`) — no extra camera. Final: `1`. Matches.

**Complexity:** Time O(n), Space O(h) — same shape as every Tree DP problem today.

**Why this is genuinely a different flavor, not a repeat:** House Robber III and Max Path Sum both optimize a **sum**; this optimizes a **count under a covering constraint**, and the correctness argument is a **greedy exchange argument** (placing lower is never better than placing at the first forced point) layered on top of the postorder combine — not a pure "take the max over disjoint cases" argument like the other two. Skippable under time pressure; not skippable if aiming to recognize this specific 3-state covering-DP shape cold in an actual interview.

---

## Dynamic Programming Closes: 35/35 Required Slots, Weeks 12–14

Reconciling the numbers directly, since the plan states "35" and a straight newly-taught count gives a different-looking number:

- **Week 12:** 8 required slots (`70, 746, 198, 213, 91, 152, 139, 322`) — 7 newly taught, 1 (`LC 152`, Day 83) fulfilled by recap rather than a fresh teach.
- **Week 13:** 21 required slots, all newly taught.
- **Week 14:** 6 required slots (`714, 309, 123, 188, 337, 124`), all newly taught.

`8 + 21 + 6 = 35` total required slots — matching the plan's own figure exactly. Of those, `7 + 21 + 6 = 34` were newly taught, and exactly `1` (the Day 83 recap) was not. Both numbers are correct; they answer slightly different questions (total required slots vs. distinct newly-taught problems), and neither contradicts the other once stated this precisely.

**Cumulative distinct problems solved through today: 229 (through Week 13) + 6 (today's and yesterday's DP required) + 1 (`LC 968`, today's extension) = 236.** (Bit Manipulation, opening tomorrow, adds to this total starting Day 95 — see Day 98's Week 14 Consolidation for the full week's final tally.)

---

# Part 2 — Theory Block: Dynamic Programming, Fully Reviewed

Six subtypes, each a genuinely distinct answer to "what does the DP state actually index by":

| Subtype | State shape | Representative problem | The transferable skill |
|---|---|---|---|
| 1D | `dp[i]` — one index | House Robber (Day 82) | What varies between always-sum (counting), max take-or-skip (optimizing), validity-gated sum, and Set-gated OR recurrences |
| Grid | `dp[r][c]` — two indices, a position | Unique Paths (Day 87) | 1D's lookback generalized across two independent directions at once |
| String | `dp[i][j]` — two indices, usually two strings (or one against itself) | LCS, Edit Distance (Day 88) | Table movement direction encodes the operation (match / insert / delete / substitute) |
| Interval | `dp[i][j]` — two indices, one array/string, filled by increasing **length** | Burst Balloons (Day 91) | Fill order becomes a correctness requirement, not just implementation detail |
| State Machine | `dp[i][state]` (+ optional budget dimension) | Cooldown, at-most-k transactions (Days 92–93) | A *situation*, not just a position, can be part of what "the subproblem" means |
| Tree | `dp(node)`, one or more return values | House Robber III, Max Path Sum (today) | The "index" is a node, combined via postorder — and a return value can either **be** the full state (House Robber III) or merely **support** a side-tracked answer (Max Path Sum, Diameter) |

**The one constant across all six, worth being able to state as a single sentence:** define precisely what `dp[...]` means *before* writing any recurrence, then prove the recurrence correct by an exhaustive, disjoint-case argument — never assert it and move on. Every subtype above has, at some point in this series, had a specific counterexample built to show what breaks when that discipline is skipped (0/1 Knapsack's loop-order counterexample, Day 85; Interval DP's fill-order counterexample, Day 91; today's brute-force House Robber III blowing up from *not* carrying state). That discipline — not any single recurrence shape — is the actual thing six weeks of Dynamic Programming were building.

---

## Project Block Guide

**Repository:** `scalable-ecommerce-platform`. Lighter day by design — use the freed time to review and clean up the DP-adjacent code written across the last two weeks (state machine transitions, grid/string table-filling helpers if any made it into the actual project) rather than adding new project surface area.

## Career Block Guide

Continue outreach cadence. If a technical post is due today, the DP-synthesis table above (six subtypes, one transferable discipline) is strong post material — it demonstrates breadth across a topic most candidates only show narrow, single-problem familiarity with.

---

## Day 94 — Interview Questions

**Q1. What's genuinely new about Tree DP versus the tree recursion from Week 7?** Not the postorder-combine shape itself (that's old) — it's that a return value can now **be** the complete DP state (as in House Robber III's pair), the same way `dp[i][state]` was the complete state on Days 92–93, rather than only ever supporting the parent's own separate computation.

**Q2. Why is House Robber III's brute force exponential, and what specifically fixes it?** Each node gets recomputed from multiple ancestors' calls with no memoization — returning both `{notRobbed, robbed}` from a single postorder visit means a parent never needs to re-descend into a child's subtree to get an answer it might need, eliminating the recomputation entirely.

**Q3. Why can't Binary Tree Maximum Path Sum return everything a parent might need, the way House Robber III does?** A path can bend at a node (go through both children), but a value handed to a parent can only extend in one direction, since the parent can only attach the node to its own single path — the "bending" answer and the "extendable upward" answer are different questions, and only the second can be a return value.

**Q4. Why does `Math.max(_, 0)` appear in the Max Path Sum solution?** It encodes that a path may choose not to extend into a child at all; without the clamp, a strongly negative subtree would be added in and actively reduce a sum that would be better off simply stopping there.

**Q5. Reconcile "35 required DP problems" with "34 newly taught."** 35 counts every required *slot* across Weeks 12–14, including one (`LC 152`, Week 12 Day 83) that was fulfilled by recap rather than a fresh teach; 34 counts only the newly-taught problems. Both are correct, answering different questions.

**Q6. Name the one discipline every DP subtype in this series has shared.** State precisely what `dp[...]` means before writing any recurrence, then prove that recurrence correct via an exhaustive, disjoint-case argument — never assert it.

**Q7. In Binary Tree Cameras, why does a `null` child return `COVERED_NO_CAMERA` instead of `NOT_COVERED`?** If a missing child forced a camera at its parent, every leaf would appear to have an "uncovered" child and would wastefully force a camera at the leaf itself; treating "no child" as already-covered correctly defers that decision upward instead.

---

## Daily Deliverable Check

- [ ] LC 337 (House Robber III) solved with the dual-state postorder return, pushed to `dsa-java/dynamic-programming/tree-dp/`.
- [ ] LC 124 (Max Path Sum) solved, with the outside-global/return-value distinction stated explicitly in comments or commit notes.
- [ ] Can explain, unprompted, why 337 and 124 — both "Tree DP" — actually have different return-value shapes.
- [ ] [Optional] LC 968 (Binary Tree Cameras) attempted if time allowed.
- [ ] Dynamic Programming's 6-subtype synthesis table reproducible from memory, one sentence per subtype.

---

## What Tomorrow Assumes You Already Know Cold

Day 95 leaves Dynamic Programming behind entirely and opens **Bit Manipulation** — a genuinely new top-level pattern, built from zero the same way Trees, Heaps, Tries, Backtracking, Graphs, Union-Find, and Dijkstra's Algorithm each were. Nothing about DP transfers directly into tomorrow's material; what carries forward instead is Day 10's exact primitive-type mechanics — sizes, ranges, and precisely how two's complement makes overflow wrap silently — which tomorrow formalizes into deliberate bitwise tools rather than an incidental side effect of arithmetic.
