# Day 75 — Union-Find: Real-World Constraint Problems, and AWS IAM

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 74 Resource Book](Day74_Resource_Book.md)
**Next ▶:** [Day 76 Resource Book](Day76_Resource_Book.md)
**Companion to:** Day 75 of `Week_11_Revised.md`

---

## Recap

Yesterday built `UnionFind` from scratch and used it for the pattern's two most direct applications — detecting a redundant edge, counting components from a matrix. Today's two required problems are less obviously Union-Find on first read: neither hands you an edge list up front. Both require recognizing a connectivity structure hiding inside a differently-shaped problem statement — a real equation-satisfiability puzzle, and a physical removal-ordering puzzle — which is a more realistic test of the pattern than yesterday's more direct setups.

---

## Learning Objectives

By the end of today, without notes:

1. Explain why Satisfiability of Equality Equations must process every `==` equation before checking any `!=` equation, not in one combined pass.
2. Explain the row/column-as-node reduction in Most Stones Removed, and prove — via a spanning-tree argument — why the answer is always `total stones − number of components`.
3. (Extension) Derive why `total connections ≥ n−1` guarantees enough "redundant" cables to reconnect every component in Number of Operations to Make Network Connected.
4. State the difference between an IAM Role and an IAM User, and explain the principle of least privilege concretely.
5. Contrast S3 and EBS by their actual access model, not just "object vs. block" as a memorized label.

---

## Concept Dependency Map

```
Day 74: UnionFind class (find, union, path compression,
        union by rank) — built once, reused everywhere below
        │
        ├─▶ Satisfiability of Equality Equations (LC 990)
        │   two-PASS structure: == first (transitive closure),
        │   != second (contradiction check)
        │
        ├─▶ Most Stones Removed (LC 947)
        │   NEW reduction: union ROW and COLUMN indices,
        │   not stone indices directly — proof via spanning tree
        │
        └─▶ (Extension) Number of Operations to Make Network
            Connected (LC 1319) — deferred here from Week 10's
            Day 69 note, now that Union-Find exists to solve it

Independent theory track:
Day 74: VPC, subnets, IGW, NAT Gateway, Security Groups
        │
        ▼
TODAY: AWS IAM (Roles vs. Users, least privilege),
       S3 (object storage) vs. EBS (block storage)
```

---

# Part 1 — Union-Find, Continued

**Prerequisites, confirmed:** the `UnionFind` class built Day 74 (find with path compression, union with union-by-rank, the `count` field) ✅ — reused directly, unmodified, in every problem below.

## Problem 3: Satisfiability of Equality Equations (LeetCode 990, Medium) — Pattern: Union-Find

**Statement:** given equations like `"a==b"`, `"b!=c"` (variables are single lowercase letters), return `true` if some integer assignment to the variables satisfies every equation, `false` otherwise.

**The reduction:** equality is transitive — if `a==b` and `b==c`, then `a==c` is forced, whether or not it's stated explicitly. That's exactly what Union-Find's `union` operation captures: merging `a` and `b`'s sets, then `b` and `c`'s, automatically places all three in one set, transitivity included for free. Inequality, by contrast, is a **constraint to check against** that transitive closure, not something to union.

**Why `==` must be fully processed before any `!=` is checked — this is not just an implementation convenience, it's required for correctness:** a `!=` equation can be violated by a transitive chain that isn't visible in any single equation. Checking `!=` equations against a *partially built* union structure could miss a contradiction that only becomes visible once every `==` equation has been folded in.

```java
public static boolean equationsPossible(String[] equations) {
    UnionFind uf = new UnionFind(26);   // one slot per lowercase letter

    for (String eq : equations) {
        if (eq.charAt(1) == '=') {
            int a = eq.charAt(0) - 'a';
            int b = eq.charAt(3) - 'a';
            uf.union(a, b);
        }
    }

    for (String eq : equations) {
        if (eq.charAt(1) == '!') {
            int a = eq.charAt(0) - 'a';
            int b = eq.charAt(3) - 'a';
            if (uf.find(a) == uf.find(b)) {
                return false;   // == equations already forced these equal
            }
        }
    }

    return true;
}
```

### Worked Trace

`equations = ["a==b", "b==c", "c!=a"]` — note that no single equation directly states `a==c`; it has to emerge from the chain.

**Pass 1 (`==` only):** `union(a,b)`: different roots, equal rank → `parent[b]=a`, `rank[a]=1`. `union(b,c)`: `find(b)=a` (via the parent pointer just set), `find(c)=c`. Different roots (`a` vs `c`); `rank[a]=1 > rank[c]=0` → `parent[c]=a`. After pass 1: `a`, `b`, and `c` all root to `a`.

**Pass 2 (`!=` only):** `"c!=a"`: `find(c)=a`, `find(a)=a` — **same root**. Contradiction found → return `false`.

The contradiction was never stated directly — it emerged purely from the transitive chain `a==b==c`, which is exactly why the two-pass structure (full transitive closure before any inequality check) is required rather than optional.

**Complexity:** the Union-Find structure here has a **fixed** size of 26, independent of the number of equations — so `α(26)` is itself a constant, making each operation genuinely O(1), not merely "small." Time **O(n)**, where `n` is the number of equations (two passes, each O(1) per equation). Space **O(1)** — the 26-element array doesn't grow with input size. This specializes the general `O(n×α(n))` Union-Find bound for the specific case where the element count is capped by a fixed alphabet rather than scaling with the input.

**Edge cases:** `"a==a"` — a no-op union with itself, harmless. `"a!=a"` — `find(a)` always equals `find(a)`, so this is an **immediate, unconditional contradiction**, correctly caught regardless of any other equation.

**💡 Interview Insight:** the two-pass structure is the single thing worth stating out loud before coding — "equality is transitive, so I need every `==` folded in before I can trust any `!=` check against the *full* picture, not a partial one." Naming this up front prevents the common trap of trying to process equations in their original left-to-right order in one pass, which would miss exactly the kind of chained contradiction shown above whenever a `!=` happens to appear before the `==` chain that creates the conflict.

---

## Problem 4: Most Stones Removed with Same Row or Column (LeetCode 947, Medium) — Pattern: Union-Find

**Statement:** stones at grid coordinates `[x, y]`. Two stones are connected if they share a row or a column. Remove stones one at a time, each removal requiring the removed stone to still share a row or column with some *remaining* stone at the moment it's removed. Return the maximum number of stones that can be removed.

**The reduction — union rows and columns themselves, not stone pairs directly:** comparing every pair of stones for a shared row/column would cost O(n²). Instead, treat each **row** and each **column** as a node, and union a stone's row with its column for every stone. Two stones end up in the same connected group exactly when there's a chain of shared rows/columns linking them — captured automatically once rows and columns share a Union-Find structure.

```java
public static int removeStones(int[][] stones) {
    int n = stones.length;
    UnionFind uf = new UnionFind(20002);   // rows 0-9999; columns offset to 10001-19999+

    for (int[] stone : stones) {
        int row = stone[0];
        int col = stone[1] + 10001;   // offset avoids row/column index collision
        uf.union(row, col);
    }

    Set<Integer> distinctRoots = new HashSet<>();
    for (int[] stone : stones) {
        distinctRoots.add(uf.find(stone[0]));
    }

    return n - distinctRoots.size();
}
```

**⚠️ Common Mistake:** unioning stone *indices* directly by checking every pair for a shared row or column. This works, but at O(n²) — the row/column-as-node reduction above is what brings it down to O(n), and it's the actual insight being tested, not an implementation detail.

**⚠️ Common Mistake:** forgetting the offset on column indices, letting a column value collide with a row value that happens to be numerically identical, silently merging two things that share no actual row or column.

### Worked Trace

`stones = [[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]` (the classic 6-stone example).

| Stone | Union call | Effect |
|---|---|---|
| `[0,0]` | `union(row0, col0+off)` | new pair, own set |
| `[0,1]` | `union(row0, col1+off)` | `col1` joins `row0`'s set |
| `[1,0]` | `union(row1, col0+off)` | `col0` is already in `row0`'s set → `row1` joins it too |
| `[1,2]` | `union(row1, col2+off)` | `col2` joins the same growing set |
| `[2,1]` | `union(row2, col1+off)` | `col1` already in the set → `row2` joins too |
| `[2,2]` | `union(row2, col2+off)` | already the same set — no-op |

Every row and column ends up in **one** connected set. Checking `find(stone[0])` for all 6 stones returns the same root every time → `distinctRoots.size() = 1`. **Answer: `6 − 1 = 5`**, matching the known correct result.

### Why the Formula `total − components` Is Correct — Proven, Not Asserted

Take any connected component with `k` stones. Build a spanning tree of that component using the "shares a row or column" relation as edges (any connected structure has at least one spanning tree). A tree with `k ≥ 2` nodes always has at least one **leaf** — a node with exactly one tree-edge — because a tree is acyclic and connected, and a connected acyclic structure with more than one node can't have every node holding two or more tree-edges (that would force a cycle). That leaf stone shares a row or column with its one tree-neighbor, which is still present — so it's always a legal removal. Removing a leaf from a tree leaves a smaller tree, still connected, still satisfying the same argument — so this repeats until exactly one stone remains, which cannot be removed further (nothing left in its component to share a row/column with). This gives exactly `k − 1` legal removals per component of size `k`, for **any** valid removal order, not just one specific lucky sequence — so the total across all components is `Σ(size_i − 1) = totalStones − numComponents`.

**Complexity:** Time O(n × α(n)) — `n` stones, each triggering one union; the final counting pass is O(n) more. Space O(n) — dominated by the `HashSet` of roots and the input itself; the Union-Find array's size (20002) is fixed and coordinate-bounded, not input-scaled.

**Edge cases:** a stone entirely alone in both its row and column (no other stone shares either) forms its own singleton component, contributing `0` to the removal count (`1 stone − 1 component = 0`) — correctly, since it can never be validly removed. All stones on a single row — one giant component, `n−1` removable.

**💡 Interview Insight:** the strongest opening line here is naming the reduction itself before coding — "I'll union rows and columns as nodes, not stones directly, so two stones end up connected exactly when they share a chain of rows/columns" — followed immediately by the formula and a one-sentence version of the spanning-tree argument. Stating the formula without the "why" is a correct answer that sounds like a guess; the leaf-removal argument is what makes it sound like proof.

---

## Extension — Number of Operations to Make Network Connected (LeetCode 1319, Medium) — Pattern: Union-Find

**This is genuinely optional, not part of today's core deliverable — skip it under real time pressure without losing anything essential to today's required problems.** It's included specifically because Week 10's Day 69 curriculum map entry flagged this exact problem, considered it for that week's Graphs extra practice, and deliberately set it aside with a note that "Union-Find opens next week and a connected-components framing is likely to resurface naturally there via a structurally different technique." This is that resurfacing.

**Statement:** `n` computers, `connections[i] = [a, b]` meaning a cable directly between computers `a` and `b`. An operation moves one existing cable to connect any two computers currently *not* connected. Return the minimum number of operations to connect all computers, or `-1` if there aren't enough cables to ever succeed.

**Feasibility check first:** connecting `n` computers requires at least `n − 1` cables, full stop — a spanning tree is the minimum possible connected structure on `n` nodes, and it has exactly `n − 1` edges. Fewer cables than that, and no rearrangement can ever succeed.

**Given enough cables, the answer is exactly `(number of connected components) − 1` — derived, not guessed:** every cable that connects two computers *already* in the same component is "redundant" — it adds no new connectivity, and can be freely repurposed as a bridge elsewhere. If there are `k` components after processing every connection, bridging them into one requires exactly `k − 1` such bridge moves. The question is whether enough redundant cables are guaranteed to exist. They are: if a component has `m` computers, it needs at least `m − 1` cables to already be internally connected, so the minimum total cables needed just to form `k` components across `n` computers is `n − k`. Any cables beyond that minimum are, by definition, redundant. So `redundant = totalConnections − (n − k)`. Given the feasibility check already confirms `totalConnections ≥ n − 1`: `redundant = totalConnections − (n−k) ≥ (n−1) − (n−k) = k − 1`. **The feasibility check alone algebraically guarantees at least `k − 1` redundant cables exist** — exactly enough to bridge every component, never fewer.

```java
public static int makeConnected(int n, int[][] connections) {
    if (connections.length < n - 1) return -1;   // provably impossible

    UnionFind uf = new UnionFind(n);
    for (int[] conn : connections) {
        uf.union(conn[0], conn[1]);
    }

    return uf.getCount() - 1;   // components - 1, using Day 74's running counter directly
}
```

**Worked check:** `n=6, connections=[[0,1],[0,2],[0,3],[1,2]]`. `connections.length=4 < n−1=5` → immediate `-1`, no union work needed. Contrast: `n=6, connections=[[0,1],[0,2],[0,3],[1,2],[1,3]]` (length 5, passes feasibility): unions collapse `{0,1,2,3}` into one component, leaving `{4}` and `{5}` isolated — final `count=3` → answer `3−1=2`.

**Complexity:** Time O(n + E×α(n)) — the feasibility check is O(1) on the input length, the union pass is O(E). Space O(n).

**Why this problem specifically, and not a different pick:** it reinforces the *same* "count components, answer is components-minus-something" shape Most Stones Removed just used, but through a genuinely different lens — there, the formula came from a removal-ordering argument; here, it comes from a redundant-resource-counting argument. Seeing the same structural family solved two different ways in one sitting is worth more than a second problem that just repeats the leaf-removal logic verbatim.

---

**Note on extra practice: exactly one problem added today (above), consistent with the plan set on Day 74** — Union-Find's true opening day carries zero extras, per this series' established precedent; today, one day past opening, is where deferred reinforcement lands instead.

---

# Part 2 — AWS IAM, and S3 vs. EBS

### Prerequisites (confirmed)

VPC, subnets, and Security Groups (Day 74) ✅ — IAM is a separate AWS concern (identity and permissions, not networking), but sits inside the same "how does this platform actually run in AWS" thread.

## IAM: Roles vs. Users

**IAM User:** a persistent identity with its own long-lived credentials (an access key/secret pair, or a console password). Suited to an actual human operator who needs to log in repeatedly over time.

**IAM Role:** **not** tied to a permanent identity at all — it's a set of permissions that something else **assumes temporarily**. An EC2 instance, a Lambda function, or a federated identity can assume a Role and receive short-lived, auto-rotating credentials for the duration it needs them, with nothing long-lived to ever leak.

**🔑 Key Takeaway:** for anything that isn't a human sitting down to work — an application, a service, an instance — a Role is the correct choice, not a User. Giving an application a User's permanent access keys means those keys sit somewhere in configuration or environment variables indefinitely, a standing liability a Role's temporary-credential model avoids entirely by construction.

## Least Privilege

Grant only the specific permissions something actually needs to do its job — nothing broader "to be safe," nothing granted preemptively for a future need that hasn't arrived yet. The direct payoff: if a credential is ever compromised, the damage it can do is bounded by exactly what it was allowed to do, not by whatever else happened to be available under the same identity.

## S3 vs. EBS — Contrasted by Actual Access Model, Not Just a Label

**S3 (object storage):** accessed over HTTP(S) via API calls (`GET`, `PUT`, `DELETE` against a bucket/key), effectively unlimited in scale, and objects are **immutable** — you replace an object wholesale, you don't edit part of it in place. Good for files, backups, static assets, anything accessed as a whole unit.

**EBS (block storage):** behaves like a raw virtual hard drive, formatted with an actual filesystem (ext4 and similar) on top, and is attached to **exactly one EC2 instance at a time** under the standard usage model. This is where a database's actual data files live — a database needs to read and write small regions of a file in place, repeatedly, which is precisely what block storage is built for and object storage's whole-object model is not.

**⚠️ Common Mistake:** describing the difference as just "object vs. block" without being able to say *why* that distinction determines the use case. The load-bearing fact is mutability and access granularity: S3 objects are replaced wholesale over HTTP; EBS volumes support in-place, byte-range reads and writes the way a mounted filesystem requires — which is exactly why a running database sits on EBS, never S3.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`.

**Task:** write a minimal IAM policy JSON granting read-only access to a specific S3 bucket, saved as `iam-policy.json` with a comment explaining each permission.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListBucket",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::ecommerce-platform-assets"
    },
    {
      "Sid": "AllowGetObjects",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::ecommerce-platform-assets/*"
    }
  ]
}
```

**Why two separate statements, not one:** `s3:ListBucket` is a permission on the **bucket itself** (its ARN has no trailing `/*`) — it's what lets you enumerate what's inside. `s3:GetObject` is a permission on **objects within** the bucket (its ARN ends in `/*`, covering any key path) — it's what lets you read an object's actual content once you know its key. Granting only `GetObject` without `ListBucket` is a genuinely common misconfiguration: an application can fetch a specific, already-known object but can't browse the bucket's contents at all, which is easy to mistake for a broken permission when it's actually a *missing* one.

**Definition of done:** `iam-policy.json` pushed, both statements commented inline explaining what each permission covers.

---

## Career Block Guide (1 hr)

**LinkedIn Post 16 — a preview of tomorrow's Dijkstra territory, written a day ahead:**

> Quick preview of something I'll be deep in soon: why BFS — the algorithm I've leaned on all week for shortest paths — actually gives the *wrong* answer the moment edges stop being equal.
>
> BFS finds the shortest path by edge count. That's only the same thing as the lowest-cost path when every edge costs exactly one step. The instant edges carry different weights, "fewest edges" and "cheapest total path" can point to two different routes entirely — a 1-edge path costing 100 can lose to a 3-edge path costing 15.
>
> Weighted shortest-path needs a different algorithm, one that always expands whichever unfinished node is currently cheapest to reach rather than whichever was reached in fewest hops. That's Dijkstra's — starting soon, and it's a genuinely different way of thinking about "shortest" than anything BFS ever required.

**Networking:** apply to 2 Tier B companies.

---

## Day 75 — Interview Questions

**Q1. Why must every `==` equation be processed before any `!=` equation is checked?**
*A:* Equality is transitive, and a `!=` violation can come from a transitive chain that isn't visible in any single equation. Checking against a partially built union structure risks missing exactly that kind of chained contradiction.

**Q2. In Most Stones Removed, why union rows and columns as nodes instead of comparing stone pairs directly?**
*A:* Pairwise comparison costs O(n²). Treating rows and columns as the nodes being unioned reduces it to O(n) unions — one per stone — while still correctly capturing "connected via a chain of shared rows/columns."

**Q3. Prove the `total stones − components` formula, don't just state it.**
*A:* Any component with `k` stones has a spanning tree via the row/column relation; a tree with `k≥2` nodes always has a leaf, which is always a legal removal (it shares a row/column with its still-present tree-neighbor); removing a leaf leaves a smaller tree, so this repeats until exactly one stone remains — giving exactly `k−1` removals per component, for any valid order.

**Q4. In Number of Operations to Make Network Connected, why does `connections.length ≥ n−1` guarantee enough redundant cables to bridge every component?**
*A:* Minimum cables needed to form `k` components across `n` computers is `n−k`; anything beyond that is redundant, so `redundant = total − (n−k)`. If `total ≥ n−1`, then `redundant ≥ (n−1)−(n−k) = k−1` — exactly the number of bridges needed.

**Q5. Why does Satisfiability of Equality Equations use a fixed-size-26 Union-Find, and what does that change about its complexity?**
*A:* Variables are single lowercase letters, so the structure's size is capped at 26 regardless of input size — making each operation genuinely O(1) rather than merely near-constant, and making overall space O(1) instead of scaling with the number of equations.

**Q6. What's the actual difference between an IAM Role and an IAM User?**
*A:* A User is a persistent identity with long-lived credentials, suited to a human. A Role has no permanent identity of its own — it's assumed temporarily by something else (an instance, a function), receiving short-lived, auto-rotating credentials with nothing long-lived to leak.

**Q7. State the principle of least privilege in one sentence, and its direct security payoff.**
*A:* Grant only the specific permissions something actually needs, nothing broader — so that if a credential is ever compromised, the damage is bounded by exactly what it was allowed to do.

**Q8. Why does a running database sit on EBS rather than S3?**
*A:* A database needs in-place, byte-range reads and writes to its data files, which block storage (a mounted filesystem on a virtual disk) supports directly. S3 objects are immutable and replaced wholesale over HTTP — not a fit for a live database's access pattern.

**Q9. What's the difference between `s3:ListBucket` and `s3:GetObject`, and why do you usually need both?**
*A:* `ListBucket` permits enumerating what's inside the bucket (a permission on the bucket itself); `GetObject` permits reading a specific object's content (a permission on objects within it). An application with only `GetObject` can fetch a known key but can't browse the bucket's contents.

**Q10. What's the shared structural idea connecting today's two required problems, despite looking unrelated on the surface?**
*A:* Both reduce to "count connected components, then derive the answer from that count" — Satisfiability via a direct same-root contradiction check, Most Stones Removed via a size-minus-components removal formula proven by a spanning-tree argument.

---

## Daily Deliverable Check

- [ ] Satisfiability of Equality Equations and Most Stones Removed with Same Row or Column solved, pushed — Union-Find ladder now at 4/7 required.
- [ ] (Extension, optional) Number of Operations to Make Network Connected solved.
- [ ] Can prove the `total − components` formula from a spanning-tree argument, not just recite it.
- [ ] `iam-policy.json` pushed, both statements commented.
- [ ] LinkedIn Post 16 published. Applied to 2 Tier B companies.

---

## What Tomorrow Assumes You Already Know Cold

Day 76 assumes today's row/column-as-node reduction is solid — tomorrow's Smallest String With Swaps uses the same "union the abstract thing that connects elements, not the elements pairwise" instinct, just with index positions instead of coordinates. It also assumes the `UnionFind` class itself needs no further explanation at this point; tomorrow is the third consecutive day calling it as a settled tool.

**Next:** [Day 76 Resource Book](Day76_Resource_Book.md) — Union-Find: String Grouping, and Observability with Prometheus.
