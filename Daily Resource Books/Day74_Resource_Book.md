# Day 74 — Union-Find Begins, and AWS Networking Fundamentals

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 73 Resource Book](Day73_Resource_Book.md)
**Next ▶:** [Day 75 Resource Book](Day75_Resource_Book.md)
**Companion to:** Day 74 of `Week_11_Revised.md`

---

## Recap

Graphs closed yesterday at 12/12 required, and its final lesson was explicit: not every question about a graph is best answered by traversing it. Today is the direct payoff of that lesson. Union-Find answers exactly one question — "are these two things connected, directly or transitively" — dramatically faster than re-running BFS/DFS every time that question is asked again, especially as connections are added incrementally over time. This is a **brand new data structure**, taught fully from scratch; nothing about it is inherited from Graphs beyond the motivating question.

---

## Learning Objectives

By the end of today, without notes:

1. Implement Union-Find (`find` and `union`) from memory, including both path compression and union by rank.
2. Explain, with a real argument, why an unoptimized Union-Find can degrade to O(n) per operation, and prove why union by rank alone already bounds that to O(log n).
3. State the combined complexity (O(α(n)) amortized) and explain in plain terms why α(n) is "practically constant."
4. Justify, unprompted, why Union-Find beats re-running DFS/BFS for repeated connectivity queries — and name the one thing DFS/BFS can do that Union-Find genuinely can't.
5. Solve Redundant Connection and Number of Provinces, including Number of Provinces' DFS alternative.
6. Explain a VPC's subnet model — public vs. private, Internet Gateway vs. NAT Gateway — and what "stateful" means for a Security Group.

---

## Concept Dependency Map

```
Day 1-2: Arrays              Day 68-73: Graphs (closed) —
        │                    "are these connected" was
        │                    answered by traversal every time
        └──────────┬─────────────────┘
                    ▼
          TODAY: Union-Find (Disjoint Set Union) — NEW STRUCTURE
          ├─ find(x) — walk parent[] pointers to the root
          ├─ union(x,y) — merge two sets
          ├─ Optimization 1: path compression
          ├─ Optimization 2: union by rank
          ├─ Combined: O(α(n)) amortized — practically constant
          ├─ Redundant Connection (LC 684)
          └─ Number of Provinces (LC 547) — Union-Find AND DFS,
             both shown, deliberately contrasted

Independent theory track:
AWS Networking Fundamentals — VPC, subnets, IGW, NAT Gateway,
Security Groups (builds on Days 71-72's general TCP/HTTP vocabulary,
now applied to a specific cloud provider's implementation of it)
```

---

# Part 1 — Union-Find (Disjoint Set Union)

## Concept Card — Union-Find

**Prerequisites, confirmed:** arrays (Day 1) ✅, basic recursion (Day 2/Week 2) ✅. Nothing from Graphs is a hard prerequisite — Union-Find is presented here as an alternative to graph traversal for one specific kind of question, not as something built on top of it.

**What it is:** a data structure that partitions a set of elements into disjoint (non-overlapping) groups, and answers two questions efficiently: `find(x)` — which group does `x` belong to (returned as a representative element, the group's "root"), and `union(x, y)` — merge `x`'s group and `y`'s group into one.

**The representation:** a single array, `parent[]`, where `parent[i]` holds the index of `i`'s parent in an implicit tree. A **root** is a node that is its own parent (`parent[i] == i`). Every element starts in its own singleton group: `parent[i] = i` for all `i`. Finding which group `x` belongs to means walking `parent[x] → parent[parent[x]] → ...` until reaching a node that's its own parent — that root **is** the group's identity; two elements are in the same group exactly when `find` returns the same root for both.

```
Initial state, n=5:           After union(0,1) and union(2,3):

parent: [0,1,2,3,4]           parent: [0,0,2,2,4]
  0  1  2  3  4                  0      2      4
  (each its own root)            │      │
                                  1      3
```

### The Naive Version, and Why It Can Degrade

```java
// NAIVE — no optimizations, shown to motivate what comes next
public static int findNaive(int[] parent, int x) {
    while (parent[x] != x) x = parent[x];
    return x;
}

public static void unionNaive(int[] parent, int x, int y) {
    int rootX = findNaive(parent, x);
    int rootY = findNaive(parent, y);
    if (rootX != rootY) parent[rootY] = rootX;   // always attach y's root under x's root
}
```

**⚠️ Common Mistake / the actual failure mode:** always attaching in a fixed direction (say, `y`'s root under `x`'s root, regardless of which tree is bigger) can build a long chain. Union `0-1`, then `1-2`, then `2-3`, then `3-4` in that fixed-direction style, and `parent` becomes `[0,0,1,2,3]` — a straight line, not a shallow tree. `find(4)` now has to walk `4→3→2→1→0`, four hops for five elements. In the worst case, this makes `find`/`union` **O(n)** per call — no better than a linked list, and no better than just re-scanning for connectivity directly.

### Optimization 1 — Union by Rank

**The fix:** always attach the **shorter** tree under the **taller** tree's root, tracked via a `rank[]` array (an upper bound on each tree's height, not its exact size). When merging two trees of equal rank, either can become the root, but the new root's rank increases by 1; when merging trees of unequal rank, the taller root's rank is unchanged (attaching something shorter underneath it doesn't grow its height).

**Why this bounds height to O(log n) — proven:** rank only ever increases when two **equal-rank** trees merge, and each such merge produces a tree with at least as many nodes as the sum of the two merged trees — meaning at least *double* the smaller one's minimum node count. By induction: a rank-0 tree has ≥1 node (the base case). A rank-`(r+1)` tree is formed by merging two rank-`r` trees, each with ≥2ʳ nodes by the inductive hypothesis, so the result has ≥2×2ʳ = 2^(r+1) nodes. A tree of rank `r`, therefore, requires at least `2^r` nodes to exist at all — so for `n` total elements, rank (and therefore height) can never exceed `log₂(n)`.

### Optimization 2 — Path Compression

**The fix:** every time `find(x)` walks up to the root, re-point every node visited along the way **directly** to that root, flattening the path for every future call.

```java
public int find(int x) {
    if (parent[x] != x) {
        parent[x] = find(parent[x]);   // recurse to the root, then compress on the way back
    }
    return parent[x];
}
```

**A concrete illustration of the flattening:** suppose (without union by rank, purely to show this effect in isolation) a chain exists: `parent[2]=1, parent[3]=2, parent[4]=3`, root `1`. Calling `find(4)`: the recursion walks `4→3→2→1` to find the root, then, unwinding, sets `parent[3]=1`, `parent[2]=1`, and `parent[4]=1` directly. A **second** call to `find(4)` — or `find(3)`, or `find(2)` — is now a single O(1) direct lookup, not a multi-hop walk. This is what "amortized" means concretely here: the *first* traversal of a long chain pays to flatten it; every later call on any node along that same chain gets the benefit for free.

### Combined: O(α(n)) Amortized

With **both** optimizations together, the amortized cost per `find`/`union` operation is **O(α(n))**, where α is the inverse Ackermann function — a function that grows so slowly that `α(n) ≤ 4` for any `n` up to values far larger than the number of atoms in the observable universe. For every practical purpose, this is constant time.

```java
public class UnionFind {
    private final int[] parent;
    private final int[] rank;
    private int count;   // number of distinct sets remaining

    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        count = n;
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);   // path compression
        }
        return parent[x];
    }

    public boolean union(int x, int y) {
        int rootX = find(x), rootY = find(y);
        if (rootX == rootY) return false;   // already connected — no-op

        if (rank[rootX] < rank[rootY]) {
            parent[rootX] = rootY;
        } else if (rank[rootX] > rank[rootY]) {
            parent[rootY] = rootX;
        } else {
            parent[rootY] = rootX;
            rank[rootX]++;
        }
        count--;
        return true;
    }

    public int getCount() { return count; }
}
```

### Why It Wins Over Re-Running BFS/DFS — Trade-offs Against the Nearest Alternative

| | Union-Find | BFS/DFS re-run per query |
|---|---|---|
| Repeated "are X and Y connected" queries | O(α(n)) amortized per query, after O(n) setup | O(V+E) per query, every time |
| Adding a new connection incrementally | O(α(n)) — just `union()` the new edge | Full re-traversal needed to stay current |
| Counting connected components | O(1) — maintained as a running counter | O(V+E) per count, one full traversal |
| Getting the **actual path** between two connected nodes | **Cannot** — only tells you *whether* they're connected, not *how* | Yes — a traversal naturally reconstructs a path |
| Handling an edge being **removed** | **Cannot**, without rebuilding from scratch | Yes — just re-traverse the updated graph |

**🔑 Key Takeaway:** Union-Find is the right tool exactly when the question is "connected or not, possibly asked many times, possibly as edges keep arriving" — and the wrong tool the moment you need an actual path, or need to handle deletions. Reaching for it without checking which of those you actually need is a real interview mistake, not just a style choice.

**Interview signal:** "are these connected," "how many groups/provinces/clusters," "does adding this edge create a cycle" — and, more subtly, any problem describing connections arriving one at a time where you're asked about connectivity *as you go*.

---

## Problem 1: Redundant Connection (LeetCode 684, Medium) — Pattern: Union-Find

**Statement:** a graph started as a tree of `n` nodes, then one extra edge was added, creating exactly one cycle. Given the edges in the order they were added, return the edge that can be removed to restore a tree — if multiple such edges exist, return the one that appears **last** in the input.

**Approach:** process edges in order; for each `[u, v]`, check `find(u)` vs. `find(v)` **before** unioning. If they're already the same root, this edge connects two nodes already connected by an earlier edge — adding it creates the cycle, and since edges are processed in input order, the first such edge found is guaranteed to be the last one in the *original* order that creates a cycle (any answer found this way is automatically "last," since nothing after it has been examined yet, and nothing before it could have created a cycle — the graph was a valid tree until this exact edge).

```java
public static int[] findRedundantConnection(int[][] edges) {
    int n = edges.length;
    UnionFind uf = new UnionFind(n + 1);   // nodes are 1-indexed

    for (int[] edge : edges) {
        int u = edge[0], v = edge[1];
        if (!uf.union(u, v)) {   // union() returns false when already connected
            return edge;
        }
    }
    return new int[0];   // unreachable given the problem's guarantees
}
```

### Worked Trace

`edges = [[1,2],[1,3],[2,3]]`. `parent = [0,1,2,3]`, `rank = [0,0,0,0]` initially (index 0 unused).

| Edge | `find(u)` | `find(v)` | Same root? | Action | State after |
|---|---|---|---|---|---|
| `[1,2]` | 1 | 2 | No | union: equal rank → `parent[2]=1`, `rank[1]=1` | `parent=[0,1,1,3]` |
| `[1,3]` | 1 | 3 | No | union: rank[1]=1 > rank[3]=0 → `parent[3]=1` | `parent=[0,1,1,1]` |
| `[2,3]` | `find(2)=1` | `find(3)=1` | **Yes** | redundant — return `[2,3]` | — |

Matches the known correct answer for this exact classic input.

**Complexity:** Time O(n × α(n)) — `n` edges, each union near-constant amortized. Space O(n) for the `parent`/`rank` arrays.

**Edge cases:** the redundant edge could connect two nodes that are each the root of their own tree at the time it's processed but were already connected through a longer chain built earlier — the algorithm doesn't care about chain length, only current root equality, so this is handled correctly without special-casing. A self-loop (`[u, u]`) — `find(u) == find(u)` trivially, correctly flagged as redundant immediately.

**💡 Interview Insight:** the phrase worth saying out loud before coding: "process edges in order, and the first edge that connects two already-connected nodes is the answer — I don't need to detect the cycle's full shape, just the one edge that closes it." That's a stronger opening than describing Union-Find mechanics first; it shows the *reduction* was understood, not just the tool.

---

## Problem 2: Number of Provinces (LeetCode 547, Medium) — Pattern: Union-Find / DFS

**Statement:** `isConnected[i][j] = 1` means cities `i` and `j` are directly connected. A "province" is a group of cities connected directly or indirectly. Return the total number of provinces.

### Approach 1 — Union-Find

Union every directly-connected pair from the matrix; the final answer is simply the running `count` of distinct sets remaining.

```java
public static int findCircleNum(int[][] isConnected) {
    int n = isConnected.length;
    UnionFind uf = new UnionFind(n);
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            if (isConnected[i][j] == 1) {
                uf.union(i, j);
            }
        }
    }
    return uf.getCount();
}
```

### Approach 2 — DFS (a meaningfully distinct alternative, not just a stylistic swap)

For every unvisited city, DFS across the matrix to mark its entire connected component, incrementing a counter once per component discovered.

```java
public static int findCircleNumDFS(int[][] isConnected) {
    int n = isConnected.length;
    boolean[] visited = new boolean[n];
    int provinces = 0;
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfs(isConnected, visited, i);
            provinces++;
        }
    }
    return provinces;
}

private static void dfs(int[][] isConnected, boolean[] visited, int city) {
    visited[city] = true;
    for (int neighbor = 0; neighbor < isConnected.length; neighbor++) {
        if (isConnected[city][neighbor] == 1 && !visited[neighbor]) {
            dfs(isConnected, visited, neighbor);
        }
    }
}
```

**Worked trace** (Union-Find version): `isConnected = [[1,1,0],[1,1,0],[0,0,1]]`, `n=3`. `count` starts at 3. `i=0,j=1`: `isConnected[0][1]=1` → `union(0,1)`: different roots, equal rank → `parent[1]=0`, `count` → 2. `i=0,j=2`: `isConnected[0][2]=0` → skip. `i=1,j=2`: `isConnected[1][2]=0` → skip. Final `count = 2` — city 2 stands alone; cities 0 and 1 form the other province. Matches the known correct answer.

**Complexity — both approaches:** Time O(n²) (the matrix itself has n² entries, and both approaches must read all of it). Space: Union-Find O(n); DFS O(n) for `visited` plus O(n) recursion stack worst case.

**Why show both, side by side, rather than picking one:** the complexities are nearly identical here, precisely *because* the input is a static matrix handed over all at once. The real argument for Union-Find only shows up under a different input shape — if connections arrived one at a time as a stream (`addConnection(i, j)` calls over time, with "how many provinces right now" queried in between), Union-Find answers each query in O(α(n)) using only the new edge, while DFS would need a **full O(n²) re-scan** after every single addition. This problem's static-matrix framing doesn't force that distinction to matter — but recognizing when it *would* matter is exactly the trade-off judgment from the Concept Card above, applied concretely.

**⚠️ Common Mistake:** in the Union-Find version, looping `j` from `0` instead of `i+1`, which redundantly checks every pair twice (and checks `i==j`, always trivially connected) — not incorrect, since redundant unions are harmless no-ops, but wasteful and a sign the symmetry of the matrix wasn't noticed.

**💡 Interview Insight:** if asked "which would you actually use here" — the honest answer is DFS, precisely *because* the input is a fixed matrix with no incremental structure; naming that trade-off, rather than defaulting to whichever pattern is freshest, is what today's Concept Card was actually building toward.

---

**Note on extra practice: none added today.** This is Union-Find's true opening day, and every pattern this series has introduced follows the same precedent — zero extras on the day a new structure is first taught, since the day's own cognitive load is already at capacity absorbing the structure itself (Trees, Day 46; Heaps, Day 54; Tries, Day 59; Backtracking, Day 61; Graphs, Day 68). Extra practice for Union-Find is deliberately deferred to Day 75.

---

# Part 2 — AWS Fundamentals: Networking

### Prerequisites (confirmed)

General TCP/IP and HTTP vocabulary (Days 71–72) ✅ — today applies that vocabulary inside one specific cloud provider's implementation of it, rather than introducing new transport-layer concepts.

## VPC: Your Own Isolated Network

**What it is:** a VPC (Virtual Private Cloud) is an isolated, logically separate network within AWS's infrastructure, defined by a CIDR block (e.g., `10.0.0.0/16` — roughly 65,000 addresses). Everything deployed inside it — servers, databases — gets an address from that range, and nothing outside the VPC can see into it by default.

## Subnets: Public vs. Private

A VPC gets subdivided into **subnets**, and what makes a subnet "public" or "private" is entirely about its **route table**, not some inherent property of the subnet itself.

- **Public subnet:** its route table sends `0.0.0.0/0` (i.e., "anywhere on the internet") traffic to an **Internet Gateway (IGW)**. Instances here can have a public IP and be reached directly from the internet.
- **Private subnet:** has **no** route to an IGW. It cannot be reached directly from outside. If it needs *outbound* access (say, to download a package), its route table instead sends `0.0.0.0/0` to a **NAT Gateway**.

## Internet Gateway vs. NAT Gateway — the asymmetry is the entire point

**Internet Gateway:** allows **bidirectional** traffic — inbound requests from the internet can reach instances in the public subnet (if a Security Group allows it), and those instances can also initiate outbound traffic.

**NAT Gateway:** lives in a **public** subnet itself (it needs its own path to the IGW), and allows only **outbound-initiated** traffic from the private subnet it serves. It performs source NAT — rewriting the private instance's address to its own public one for outbound requests — but nothing on the internet can *initiate* a connection back in through it. A private-subnet database can reach out to download a security patch; nothing on the internet can reach in to touch that database directly, even if it somehow had the address.

**🔑 Key Takeaway:** "public" and "private" describe **routing**, not physical location or inherent security — a subnet is public specifically because its route table points to an IGW, and nothing more fundamental than that.

## Security Groups: A Stateful Firewall

**What it is:** a Security Group is a firewall attached to individual instances (technically, to their network interfaces), controlling what traffic is allowed in and out.

**"Stateful" — what this precisely means:** if an outbound request is allowed by the Security Group's rules, the **response** to that request is automatically allowed back in, without needing a separate, matching inbound rule for it. The Security Group tracks connection state and recognizes "this inbound packet is a reply to a connection I already approved outbound." This is a genuinely different model from a stateless firewall (like a Network ACL, evaluated at the subnet level), which would require explicit inbound *and* outbound rules for both halves of any conversation — including replies.

**⚠️ Common Mistake:** assuming a Security Group needs an explicit inbound rule to allow a response to come back after an approved outbound request — that's exactly the stateless model, and it's not how Security Groups work. Writing an unnecessary inbound rule "just to be safe" for something that's already covered statefully is a sign this distinction wasn't fully internalized, not a harmless extra precaution.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`.

**Task:** diagram an AWS deployment for the platform: EC2 instances in a public subnet behind the Gateway (Day 66), RDS in a private subnet, Security Groups restricting the database to only the application instances.

```
                    ┌───────────────────────────────────────────────┐
                    │              VPC — 10.0.0.0/16                  │
                    │                                                  │
     Internet ──────┼──▶ [IGW] ─────┐                                 │
                    │               │                                  │
                    │  ┌────────────▼─────────────────────────────┐  │
                    │  │   PUBLIC SUBNET — 10.0.1.0/24               │  │
                    │  │   EC2: Gateway (Day 66) — SG: 443 from     │  │
                    │  │   0.0.0.0/0                                 │  │
                    │  │              │                               │  │
                    │  │        [NAT Gateway]                        │  │
                    │  └──────────────┼───────────────────────────┘  │
                    │                 │ (outbound-only path down)     │
                    │  ┌──────────────▼───────────────────────────┐  │
                    │  │   PRIVATE SUBNET — 10.0.2.0/24              │  │
                    │  │                                              │  │
                    │  │   EC2: Order / Payment / Inventory modules  │  │
                    │  │   SG: inbound only from Gateway's SG        │  │
                    │  │              │                               │  │
                    │  │              ▼                               │  │
                    │  │   RDS: PostgreSQL                           │  │
                    │  │   SG: inbound port 5432, ONLY from the      │  │
                    │  │   application modules' SG — not the         │  │
                    │  │   Gateway, not the internet, nothing else   │  │
                    │  └──────────────────────────────────────────┘  │
                    └───────────────────────────────────────────────┘
```

**Definition of done:** a labeled diagram covering the VPC, both subnet types, both gateway types, and Security Group rules — all four present above, with the database's Security Group deliberately excluding even the Gateway, since the Gateway should only ever talk to the application modules, never the database directly.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** apply to 2 Tier B companies.

---

## Day 74 — Interview Questions

**Q1. What does `find(x)` return, conceptually, and how is "same group" determined?**
*A:* The root of `x`'s tree — the node that is its own parent. Two elements are in the same group exactly when `find` returns the same root for both.

**Q2. Why can an unoptimized Union-Find degrade to O(n) per operation?**
*A:* If unions always attach in a fixed direction regardless of tree size, repeated unions can build a long chain (effectively a linked list), making `find` walk the entire chain in the worst case.

**Q3. Prove why union by rank alone bounds height to O(log n).**
*A:* Rank only increases when merging two equal-rank trees, and each such merge at least doubles the resulting tree's minimum node count (by induction: a rank-`r` tree has ≥2^r nodes). So for `n` total elements, rank — and height — can never exceed log₂(n).

**Q4. Explain path compression's effect using a concrete example.**
*A:* Given a chain `parent[2]=1, parent[3]=2, parent[4]=3`, calling `find(4)` walks to the root (1) and then re-points `parent[2]`, `parent[3]`, and `parent[4]` all directly to 1 — so any later `find` on those nodes is a single O(1) lookup instead of a multi-hop walk.

**Q5. What does O(α(n)) mean in practice?**
*A:* α is the inverse Ackermann function, which grows so slowly that it's ≤4 for any realistic `n` — effectively constant time for all practical inputs, even though it's not literally O(1) in the strict mathematical sense.

**Q6. Name one thing DFS/BFS can do for connectivity questions that Union-Find genuinely cannot.**
*A:* Reconstruct the actual path between two connected nodes — Union-Find only answers whether they're connected, never how.

**Q7. In Redundant Connection, why does the first edge found to connect two already-same-root nodes have to be the correct answer, not just *an* answer?**
*A:* Edges are processed in their original input order, and the graph was a valid tree right up until this specific edge — nothing processed before it could have created a cycle, so this is necessarily the unique edge whose removal restores a tree, and it's automatically the last such edge in original order since nothing after it has been examined yet.

**Q8. Why does Number of Provinces's static-matrix framing make Union-Find and DFS roughly equivalent, when the Concept Card argued Union-Find is usually better for connectivity questions?**
*A:* Union-Find's advantage is specifically for repeated or incremental queries. A fixed matrix handed over all at once has no incremental structure to exploit — both approaches must read all n² entries regardless, so neither has a structural advantage here.

**Q9. What precisely makes a subnet "public" versus "private" in a VPC?**
*A:* Its route table — a public subnet routes `0.0.0.0/0` to an Internet Gateway; a private subnet has no such route, and only reaches the internet outbound (if at all) through a NAT Gateway.

**Q10. What does "stateful" mean for a Security Group, precisely?**
*A:* If an outbound request is allowed, the response to it is automatically allowed back in without a separate matching inbound rule — the Security Group tracks connection state rather than evaluating every packet independently in both directions.

---

## Daily Deliverable Check

- [ ] Redundant Connection and Number of Provinces solved, pushed to `dsa-java/union-find/` — Union-Find ladder now at 2/7 required.
- [ ] Can implement `UnionFind` (both optimizations) from memory, and explain why each optimization is needed, not just that it exists.
- [ ] AWS deployment diagram complete, covering VPC, both subnet types, both gateway types, and Security Group rules.
- [ ] Applied to 2 Tier B companies.

---

## What Tomorrow Assumes You Already Know Cold

Day 75 assumes the `UnionFind` class built today is solid enough to reuse directly, without re-deriving `find`/`union` from scratch — tomorrow's two problems both call it as a tool, not re-teach it. It also assumes the "connected or not, asked repeatedly" framing from today's trade-off discussion is internalized, since tomorrow's Most Stones Removed problem depends on recognizing a components-counting question that isn't phrased as one on the surface.

**Next:** [Day 75 Resource Book](Day75_Resource_Book.md) — Union-Find: Real-World Constraint Problems, and AWS IAM.
