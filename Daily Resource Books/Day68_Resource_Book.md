# Day 68 Resource Book — Graphs Begin, and Rate Limiting

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 67 Resource Book](Day67_Resource_Book.md)
**Next ▶:** [Day 69 Resource Book](Day69_Resource_Book.md)
**Companion to:** Day 68 of `Week_10_Revised.md`

---

## Recap

Backtracking closed yesterday at a full 12/12. Today opens **Graphs (BFS/DFS)** — genuinely new anywhere in this series, not a variant of anything already built. Every tree traversed since Day 46 (Trees), Day 50 (BSTs), and Day 59 (Tries) shared one quiet, load-bearing property: **a tree has exactly one path between any two nodes, and no cycles.** That's *why* none of those traversals ever needed to track which nodes had already been visited — it was structurally impossible to revisit one. A graph drops that guarantee. Today makes that difference explicit, and everything that follows for the next three days builds on it.

**Per this series' own established precedent** (Trees Day 46, Heaps Day 54, Tries Day 59, Backtracking Day 61 — every pattern's genuine opening day, correctly withholding extras): **no extra practice is added today.** Any extra Graph reps land on a later day this week, once the base mechanism is no longer opening fresh.

---

## Learning Objectives

By the end of today, without notes:

1. State precisely why every tree traversal so far never needed a `visited` set, and why a graph does.
2. Choose between an adjacency list and an adjacency matrix for a given graph, justifying the choice by density and the operations actually needed.
3. Recognize, from a problem statement, when it calls for BFS and when it calls for DFS.
4. Solve Number of Islands and Max Area of Island, explaining precisely why grid traversal here is *not* backtracking, despite using the same recursive DFS shape.
5. Explain the Token Bucket algorithm's two independent guarantees — bounded burst, bounded long-run rate — and implement it.

---

## Concept Dependency Map for Today

```
Day 8  — Recursion (needed for DFS, unchanged mechanism)
Day 4  — Queue via ArrayDeque (needed for BFS, unchanged mechanism)
Day 46 — TreeNode, universal recursive template
Day 49 — Tree BFS, queue.size() level-isolation trick
        │
        ▼
Concept Card — Graphs (BFS/DFS) [NEW]
  ├─ A tree IS a graph (connected, acyclic, exactly one path between
  │  any two nodes) — DFS/BFS mechanics transfer UNCHANGED
  ├─ NEW requirement: explicit `visited` — a graph CAN cycle, a tree
  │  structurally cannot
  ├─ Representation: adjacency list vs. adjacency matrix
  └─ "A grid is a graph in disguise" — cells are nodes, adjacency is edges
        │
        ▼
Today, Problem 1 — Number of Islands (LC 200)
  Matrix DFS/BFS — permanent marking, NOT backtracking (no un-choose)
        │
        ▼
Today, Problem 2 — Max Area of Island (LC 695)
  Identical scaffold, DFS return value aggregated instead of just counted

Day 4 — Queue/counter mechanics
        │
        ▼
Token Bucket Rate Limiting (NEW) — bounded burst via a capacity cap,
  bounded average rate via a fixed refill rate — two independent knobs
```

---

# Concept Card — Graphs (BFS/DFS)

## What a graph is, and why a tree already was one

A **graph** is a set of nodes (vertices) connected by edges. A **tree**, precisely, is a special case: a **connected, acyclic** graph with exactly one path between any two nodes. Every recursive template built for trees since Day 46 — base case, combine results from neighbors, return upward — is *already* a graph traversal; it simply never had to handle the general case, because a tree's structure ruled out the complications a general graph introduces.

**Directed vs. undirected:** an edge either points one way (directed — `A → B` doesn't imply `B → A`) or both ways implicitly (undirected — a connection is mutual by definition). **Weighted vs. unweighted:** an edge either carries a cost/distance, or every edge is treated as equally "1 step." Today, and for the rest of this week, every graph is **unweighted** — weighted shortest-path algorithms (Dijkstra, and the light Floyd-Warshall exposure) are a deliberately later destination in this series, not needed for anything this week.

## Why a `visited` set is now mandatory, not optional

This is the single most important new fact today, worth being able to state precisely: **a tree cannot cycle, by definition — there is exactly one path from the root to any node, so a traversal can never revisit a node it's already seen.** A graph carries no such guarantee. Two nodes can be mutually reachable through more than one path, and a cycle (a path that returns to a node already on it) is entirely legal. Without explicitly tracking which nodes have already been visited, a DFS or BFS on a graph containing a cycle can revisit the same nodes indefinitely — at minimum wasting enormous redundant work, at worst never terminating at all. Every graph traversal from today forward carries an explicit `visited` structure (a `Set`, or — on a grid specifically — marking cells directly, as today's two problems do) for exactly this reason.

## Representation: adjacency list vs. adjacency matrix

**Adjacency list** — `Map<Node, List<Node>>` (or, for LeetCode's common integer-labeled-node convention, `List<List<Integer>>` indexed by node number): for each node, store only the neighbors it's actually connected to. Space is O(V + E) — proportional to what's actually there. This is the default choice for **sparse** graphs (relatively few edges compared to the maximum possible), which describes most real-world and most interview graphs.

**Adjacency matrix** — a 2D array, `matrix[i][j] = true` (or a weight) if an edge connects `i` and `j`. Space is **O(V²)**, regardless of how many edges actually exist — even a graph with almost no edges pays the full quadratic cost. Its advantage: checking whether a *specific* edge exists is O(1), where an adjacency list requires scanning that node's neighbor list.

| | Adjacency List | Adjacency Matrix |
|---|---|---|
| Space | O(V + E) | O(V²) always |
| "Does edge (i,j) exist?" | O(degree of i) | O(1) |
| "All neighbors of i?" | O(degree of i) — exactly what's stored | O(V) — must scan the whole row |
| Best fit when... | Sparse graph (most real/interview graphs), need to enumerate neighbors | Dense graph, or frequent specific-edge-existence checks matter more than space |

**"A grid is a graph in disguise," stated precisely:** every cell is a node; every pair of adjacent cells (up/down/left/right, by this week's convention) is an implicit edge — never materialized as an actual adjacency list, since the neighbor relationship is computable directly from coordinates (`(r±1, c)`, `(r, c±1)`) rather than stored. This is why today's two problems need no separate graph-construction step at all; the grid itself already *is* the graph.

## BFS vs. DFS: the actual interview signal, not just two traversal orders

**BFS** (queue-based, level by level — Day 49's exact mechanism) explores every node at distance `k` before any node at distance `k+1`. This ordering is *why* BFS is the correct choice whenever a problem asks for **shortest path in an unweighted graph** — the first time BFS reaches a target node is guaranteed to be via the fewest possible edges, precisely because everything closer was necessarily explored first.

**DFS** (recursive or explicit-stack, as deep as possible before backtracking — Day 46's exact mechanism) is the right default for **exhaustive exploration**: does *any* path exist, is the graph fully connected, does a cycle exist. DFS doesn't inherently find the *shortest* anything — it finds *a* path, however long, first.

> 🔑 **Key Takeaway — the interview tell:** "shortest path," "minimum steps," "fewest moves" → **BFS**, unconditionally, on an unweighted graph. "Does a path exist," "explore every reachable node," "is this connected" → **DFS** (or BFS — either correctly answers pure reachability, though DFS is often the simpler code for it). Getting this backwards — using DFS where the problem actually needs *shortest*, or vice versa — is a common, avoidable mistake worth checking for explicitly before writing code.

---

# Part 1 — Number of Islands (LeetCode 200, Medium) — Pattern: Matrix DFS/BFS

**Statement:** Given an `m × n` grid of `'1'`s (land) and `'0'`s (water), return the number of islands — a maximal group of horizontally/vertically connected `'1'`s.

## Why this is graph traversal and *not* backtracking, precisely

This distinction is worth making explicit and defending, not glossed over — it's exactly the kind of "these look similar, are they the same thing" question a tier-1 interview probes. Day 61 defined backtracking specifically by its **un-choose** step, necessary because backtracking mutates *one shared structure across an entire decision tree* that must be restored for sibling branches to explore correctly. Today's traversal marks a cell visited **permanently** — once a land cell is confirmed part of an island, there is no scenario where "un-marking" it and revisiting is ever needed or correct; every cell belongs to exactly one island, forever, for the rest of the algorithm's execution. There is no un-choose step here **because there is nothing to restore** — this is plain graph DFS/BFS (flood fill), not backtracking, even though the code shape (recursive, explores neighbors) looks superficially similar to Day 66's Word Search.

### Approach — DFS flood-fill from every unvisited land cell

```java
public static int numIslands(char[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    int count = 0;
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == '1') {
                count++;              // found a NEW island's first cell
                dfs(grid, r, c);      // sink the ENTIRE island — mark every connected cell
            }
        }
    }
    return count;
}

private static void dfs(char[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length || grid[r][c] != '1') {
        return;   // out of bounds, or already water/already visited
    }
    grid[r][c] = '0';   // mark visited — PERMANENTLY; never restored
    dfs(grid, r + 1, c);
    dfs(grid, r - 1, c);
    dfs(grid, r, c + 1);
    dfs(grid, r, c - 1);
}
```

**Why `count++` happens at the outer scan, not inside `dfs`:** the outer double loop only ever calls `dfs` on a cell that's still `'1'` — meaning it hasn't been claimed by any previously-discovered island. Each such call is, by construction, the *first* cell of a brand-new island; everything that `dfs` goes on to mark during that one call belongs to that same island and must not be double-counted.

**A `visited[][]` boolean array instead of mutating the input directly** would also be correct, and is the better choice when the input shouldn't be mutated (a real, common constraint worth naming rather than assuming mutation is always fine) — this is the exact same "is mutating the caller's input an acceptable side effect" judgment call Day 5's Contains Duplicate raised for sorting a copy versus sorting in place.

### Trace: confirming against a 2-island grid

```
["1","1","0","0","0"]
["1","1","0","0","0"]
["0","0","1","0","0"]
["0","0","0","1","1"]
```

Outer scan hits `(0,0)='1'` first → `count=1`, DFS sinks the entire top-left 2×2 block (`(0,0),(0,1),(1,0),(1,1)`), all four turn to `'0'`. Scan continues, skipping already-`'0'` cells, until `(2,2)='1'` → `count=2`, DFS sinks just that single cell (no connected neighbors). Scan continues until `(3,3)='1'` → `count=3`, DFS sinks `(3,3)` and `(3,4)` together. Final: **3 islands** — confirmed by direct execution against this exact grid.

### Complexity

**Time: O(m × n)** — every cell is visited by the outer scan exactly once, and marked (thus never re-entering `dfs`'s body past the guard) at most once total across the whole algorithm — the same "total work bounded by total cells, not multiplied per cell" amortized shape as Day 5's Longest Consecutive Sequence. **Space: O(m × n)** worst case — a grid that's entirely land causes the DFS recursion to go as deep as the total cell count before unwinding (the call stack, not any explicit data structure, is what costs this).

### Common Mistakes and Edge Cases

- ⚠️ **Forgetting to mark a cell visited *before* recursing into its neighbors** (marking after, or not at all) — causes infinite mutual re-visiting between adjacent cells, since each would keep finding the other still marked `'1'`.
- ⚠️ **Checking bounds after accessing `grid[r][c]`** instead of before — throws `ArrayIndexOutOfBoundsException`, the identical ordering mistake flagged for Day 66's Word Search.
- ⚠️ **Using diagonal neighbors** — this problem's connectivity is explicitly 4-directional (up/down/left/right) only; diagonal cells don't count as connected.
- Edge case: entirely water grid → `0` islands, outer loop never triggers a `dfs` call.
- Edge case: entirely land grid → `1` island, one single `dfs` call sinks everything.
- Edge case: `1×1` grid → `0` or `1` depending on that single cell's value, no recursion needed either way.

> 💡 **Interview Insight:** Explicitly naming *why* this isn't backtracking — no un-choose step needed, because marking is permanent, not path-scoped — is a stronger signal than silently writing correct code that happens to look like Day 66's Word Search. It demonstrates the concept (what distinguishes backtracking from plain graph traversal) is understood as a real distinction, not a surface pattern.

---

# Part 2 — Max Area of Island (LeetCode 695, Medium) — Pattern: Matrix DFS

**Statement:** Given an `m × n` grid of `0`s and `1`s, return the area of the largest island (largest connected group of `1`s, 4-directionally). Return `0` if there is no island.

## The one real change from Problem 1: DFS now returns a value

Problem 1's `dfs` returned `void` — its only job was marking cells and letting the outer loop count islands. Here, `dfs` needs to report **how many cells** it just sank, so the outer loop can track the maximum across all islands.

### Approach — Same scaffold, DFS returns its own subtree's cell count

```java
public static int maxAreaOfIsland(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    int maxArea = 0;
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == 1) {
                maxArea = Math.max(maxArea, dfs(grid, r, c));
            }
        }
    }
    return maxArea;
}

private static int dfs(int[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length || grid[r][c] != 1) {
        return 0;   // out of bounds or not land — contributes nothing to area
    }
    grid[r][c] = 0;   // mark visited, permanently — same reasoning as Problem 1
    return 1 + dfs(grid, r + 1, c) + dfs(grid, r - 1, c) + dfs(grid, r, c + 1) + dfs(grid, r, c - 1);
}
```

**Why this is the exact same "combine results from neighbors" shape Day 46 established for trees, worth citing directly:** `1 + dfs(...) + dfs(...) + dfs(...) + dfs(...)` is structurally identical to Day 46's Maximum Depth of Binary Tree (`1 + max(depth(left), depth(right))`) — a base case contributing a fixed value, combined with recursive results from every neighbor. The only real differences: a graph node can have up to 4 neighbors instead of a tree's fixed 2, and the combination here is a **sum** (total area) rather than a **max** (depth) — the recursive *shape*, not the combination *operation*, is what transferred unchanged from trees to graphs.

### Trace: a small L-shaped island

```
[[1,1,0],
 [0,1,0],
 [0,1,1]]
```

`dfs(0,0)`: mark `(0,0)=0`, return `1 + dfs(1,0) + dfs(-1,0) + dfs(0,1) + dfs(0,-1)`.
- `dfs(1,0)`: `grid[1][0]==0` (water) → returns `0`.
- `dfs(-1,0)`: out of bounds → returns `0`.
- `dfs(0,1)`: `grid[0][1]==1` → mark `(0,1)=0`, return `1 + dfs(1,1) + dfs(-1,1) + dfs(0,2) + dfs(0,0)`.
  - `dfs(1,1)`: `grid[1][1]==1` → mark, return `1 + dfs(2,1) + dfs(0,1) + dfs(1,2) + dfs(1,0)`.
    - `dfs(2,1)`: `grid[2][1]==1` → mark, return `1 + dfs(3,1)[OOB=0] + dfs(1,1)[now 0]=0 + dfs(2,2) + dfs(2,0)[water]=0`.
      - `dfs(2,2)`: `grid[2][2]==1` → mark, return `1 + 0+0+0+0 = 1` (all its neighbors already marked or OOB).
    - so `dfs(2,1) = 1 + 0 + 0 + 1 + 0 = 2`.
    - `dfs(0,1)` (already marked `0` by now) → returns `0`. `dfs(1,2)`: water → `0`. `dfs(1,0)`: water → `0`.
  - so `dfs(1,1) = 1 + 2 + 0 + 0 + 0 = 3`.
  - `dfs(0,2)`: water → `0`. `dfs(0,0)` (already marked) → `0`.
  - so `dfs(0,1) = 1 + 3 + 0 + 0 + 0 = 4`.
- `dfs(0,-1)`: out of bounds → `0`.

`dfs(0,0) = 1 + 0 + 0 + 4 + 0 = 5` — **matches the grid's actual 5 land cells exactly**, confirming the sum-of-neighbor-returns correctly accumulates the whole connected region's size.

### Complexity

**Time: O(m × n), Space: O(m × n)** worst case — identical bounds and identical reasoning to Problem 1; only the per-call return value changed, not the traversal's shape or cost.

### Common Mistakes and Edge Cases

- ⚠️ **Returning `1` from the base-case-failure branch instead of `0`** — the out-of-bounds/water case contributes *nothing* to area; returning `1` there would silently inflate every island's counted size.
- ⚠️ **Forgetting `Math.max` at the outer loop** and instead summing every `dfs` call's result — that would compute *total* land area across the whole grid, not the *largest single island*, a different (and wrong, for this problem) quantity.
- Edge case: no land at all → `0`, matching the problem's own explicit specification for this case.
- Edge case: multiple islands of different sizes → only the single largest area is returned, smaller islands correctly ignored by the `Math.max` comparison.

> 🔗 **Forward reference:** this "DFS returns a value to be aggregated by the caller" shape — rather than DFS just marking and the caller separately counting — is exactly the shape Day 69's Clone Graph reuses, just aggregating a cloned reference instead of a count.

---

# Part 3 — Rate Limiting: Token Bucket

## Prerequisites, confirmed

No new data structure is strictly required — a counter and a timestamp are enough; an `ArrayDeque` (Day 4) is a natural implementation choice but not conceptually necessary.

## The mechanism: two independent, simultaneously-enforced guarantees

A **token bucket** holds a maximum **capacity** of tokens and **refills** at a fixed rate over time. Each incoming request consumes exactly one token; a request arriving when the bucket is empty is rejected (or queued, depending on the design). This single mechanism enforces **two different guarantees at once**, worth separating explicitly rather than treating "rate limiting" as one undifferentiated idea:

1. **Bounded burst:** the bucket's capacity caps how many requests can be served *instantaneously*, back to back, before it's empty — allowing legitimate short bursts of traffic without immediately rejecting them.
2. **Bounded long-run average rate:** the refill rate caps how quickly the bucket recovers, which caps the *sustained* request rate over any long window, regardless of burst patterns.

**Why both matter, and why a simpler "N requests per minute, reset every minute" counter doesn't capture the same thing:** a naive fixed-window counter allows a burst of `2N` requests clustered right at a window boundary (N at the very end of one window, N at the very start of the next, arbitrarily close together in real time) — a real gap the token bucket's continuous refill avoids, since tokens accumulate continuously rather than resetting in discrete jumps.

### Approach — Track tokens and a last-refill timestamp

```java
public class TokenBucket {
    private final long capacity;
    private final double refillRatePerMillis;   // tokens added per millisecond
    private double currentTokens;
    private long lastRefillTimestamp;

    public TokenBucket(long capacity, double refillRatePerSecond) {
        this.capacity = capacity;
        this.refillRatePerMillis = refillRatePerSecond / 1000.0;
        this.currentTokens = capacity;              // start full
        this.lastRefillTimestamp = System.currentTimeMillis();
    }

    public synchronized boolean tryConsume() {
        refill();
        if (currentTokens >= 1) {
            currentTokens -= 1;
            return true;    // request allowed
        }
        return false;       // bucket empty — reject
    }

    private void refill() {
        long now = System.currentTimeMillis();
        long elapsed = now - lastRefillTimestamp;
        double tokensToAdd = elapsed * refillRatePerMillis;
        currentTokens = Math.min(capacity, currentTokens + tokensToAdd);   // never exceed capacity
        lastRefillTimestamp = now;
    }
}
```

**Why `synchronized` here, directly citing Day 37's data-corruption demonstration:** under concurrent requests, `tryConsume()` reads and writes `currentTokens` — the exact unguarded-shared-mutable-state shape Day 37's `Counter` exercise showed corrupts silently under real concurrent access. Without a lock here, two simultaneous requests could both read the same `currentTokens` value before either writes back the decremented result, allowing more requests through than the bucket's capacity should ever permit — a real correctness bug, not just a performance concern, applying Week 6's lesson to a genuinely new context.

**Why `Math.min(capacity, ...)` on refill matters:** without capping at `capacity`, tokens would accumulate without bound during any sufficiently long idle period, and a client that's been quiet for a while could then burst *far* beyond the intended capacity the moment it resumes — defeating the entire bounded-burst guarantee.

### Common Mistakes

- ⚠️ **Refilling in fixed discrete jumps** (e.g., "add N tokens every minute, on a timer") instead of continuously based on elapsed time — reintroduces the same boundary-clustering burst problem a token bucket exists to avoid, discussed above.
- ⚠️ **Forgetting to cap tokens at `capacity` on refill** — allows unbounded token accumulation during idle periods, breaking the burst guarantee.
- ⚠️ **Skipping synchronization under the assumption "it's just a counter"** — proven above to be a genuine correctness bug under concurrent access, not a hypothetical one.

> 💡 **Interview Insight:** Being able to name *both* guarantees a token bucket provides — not just "it limits requests" — and explain concretely why a naive fixed-window counter fails to provide the burst guarantee, is what distinguishes real understanding of this algorithm from having seen the name before.

---

## Project Block

**Repository:** `scalable-ecommerce-platform`.

**Task:** apply a request rate limiter filter at the Gateway using the Token Bucket logic above — in-memory for now (this gets upgraded to a distributed, Redis-backed version once the HLD phase covers why a single in-memory bucket per Gateway instance breaks down the moment there's more than one Gateway instance running).

**Practical guidance:** apply the filter at the same Gateway layer as yesterday's JWT validation, and think explicitly about ordering — should rate limiting run before or after auth? (A defensible answer: rate limiting first, cheaply rejecting excess traffic before spending any cycles on token parsing/validation — though reasonable systems make different choices depending on whether unauthenticated traffic is itself a bigger concern.)

**Definition of done:** the Gateway returns HTTP `429` once the configured limit is exceeded — verified by sending requests faster than the refill rate and observing the `429` response, not just reading the configuration.

---

## Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** track application responses from earlier in the week; reach out to internal recruiters for roles applied to last week.

---

# Day 68 — Interview Questions

---

**1. Why did no tree traversal in this series, from Day 46 through Day 61's Word Search II, ever need an explicit `visited` structure?**

*Answer:* A tree is, by definition, connected and acyclic, with exactly one path between any two nodes — it's structurally impossible for a tree traversal to revisit a node it's already seen. A general graph carries no such guarantee and can contain cycles, which is exactly why an explicit `visited` structure becomes mandatory starting today.

---

**2. When is an adjacency matrix actually the better choice over an adjacency list?**

*Answer:* When the graph is dense (a large fraction of all possible edges actually exist) or when checking whether one *specific* edge exists is a frequent operation — the matrix answers that in O(1) where a list requires scanning a neighbor list. For sparse graphs, the list's O(V+E) space is strictly better than the matrix's unconditional O(V²).

---

**3. Given a problem statement, what's the specific phrasing that signals BFS over DFS?**

*Answer:* "Shortest path," "minimum steps," or "fewest moves" on an unweighted graph — BFS's level-by-level exploration guarantees the first time it reaches a target is via the fewest possible edges. DFS doesn't inherently find the shortest anything; it's the right default for "does a path exist" or "explore everything reachable."

---

**4. Why is Number of Islands' DFS not backtracking, despite the recursive, explore-neighbors code shape?**

*Answer:* Backtracking requires an un-choose step specifically because it mutates one shared structure across a whole decision tree that must be restored for sibling branches. Number of Islands marks a cell visited permanently — once part of a confirmed island, there's never a scenario requiring that mark to be undone. With nothing to restore, there's no un-choose step, which is exactly what makes this plain graph traversal rather than backtracking.

---

**5. In Max Area of Island, why does the out-of-bounds/water base case return `0` and not `1`?**

*Answer:* That branch represents a cell contributing nothing to the island's area — it's either off the grid or water, neither of which is land. Returning `1` there would incorrectly count a non-land cell as part of the area, inflating every island's computed size.

---

**6. What's the direct structural connection between Max Area of Island's DFS and Day 46's Maximum Depth of Binary Tree?**

*Answer:* Both are "base case contributes a fixed value, combine recursive results from every neighbor" — `1 + max(depth(left), depth(right))` for a tree's two children versus `1 + dfs(...) + dfs(...) + dfs(...) + dfs(...)` summed over a grid cell's up to four neighbors. The recursive shape is identical; only the number of neighbors (2 fixed vs. up to 4) and the combining operation (max vs. sum) differ.

---

**7. Name the two independent guarantees a token bucket provides, and why a naive "N requests per fixed minute window" counter fails to provide one of them.**

*Answer:* Bounded burst (via capacity) and bounded long-run average rate (via refill rate). A fixed-window counter allows up to 2N requests clustered arbitrarily close together across a window boundary — N at the very end of one window and N at the very start of the next — which a continuously-refilling token bucket doesn't permit, since tokens accumulate smoothly over time rather than resetting in discrete jumps.

---

**8. Why is `synchronized` genuinely required on `tryConsume()`, not just good practice?**

*Answer:* Under concurrent requests, two threads could both read the same `currentTokens` value before either writes back its decrement, allowing more requests through than the bucket's capacity should permit — a real correctness bug (more tokens consumed than actually existed), the same category of unguarded-shared-state corruption Day 37 demonstrated directly.

---

**9. What breaks if token refill doesn't cap at `capacity`?**

*Answer:* Tokens would accumulate without bound during any sufficiently long idle period, letting a client that's been quiet burst far beyond the intended capacity the moment it resumes — defeating the bounded-burst guarantee the capacity is supposed to enforce.

---

**10. Why does a grid need no explicit adjacency list built before running DFS/BFS on it?**

*Answer:* A grid's neighbor relationship (up/down/left/right) is directly computable from a cell's own coordinates — `(r±1, c)` and `(r, c±1)` — rather than needing to be looked up from a stored structure. The grid itself already encodes the graph; there's nothing separate to construct.

---

## Daily Deliverable Check

- [ ] Number of Islands (LC 200) solved — the "not backtracking" distinction stated clearly.
- [ ] Max Area of Island (LC 695) solved, pushed to `dsa-java/graphs/`.
- [ ] Rate limiter returning 429 at the Gateway once tripped — verified with real requests exceeding the refill rate.

---

## What Tomorrow Assumes You Already Know Cold

Day 69 assumes today's Concept Card is fully solid — the adjacency-list-vs-matrix trade-off, the BFS/DFS interview signal, and specifically *why* a `visited` structure is now mandatory — since tomorrow's Clone Graph is the first problem this series has posed where a graph genuinely can contain a cycle that would break a naive traversal without one. It assumes today's permanent-marking-is-not-backtracking distinction is settled, since Is Graph Bipartite tomorrow uses a similar-looking "mark and check neighbors" shape for a structurally different purpose (2-coloring, not flood-fill) that's worth not conflating with today's.

**Next:** [Day 69 Resource Book](./Day69_Resource_Book.md) — Graph Traversal Continues, and Kafka Schema Registry.
