# Day 78 — Dijkstra's Algorithm Begins, and Sharding Strategies

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 77 Resource Book](Day77_Resource_Book.md)
**Next ▶:** [Day 79 Resource Book](Day79_Resource_Book.md)
**Companion to:** Day 78 of `Week_12_Revised.md`

---

## Recap

Day 77 closed Union-Find at 7/7 required (8 distinct with Day 75's extra) and immediately reused the `UnionFind` class, unmodified, as Kruskal's Algorithm's cycle-detection subroutine — the same day the pattern closed. Prim's Algorithm was taught alongside it, reusing the `PriorityQueue` machinery from Days 17, 26, and 54, and both algorithms' correctness rested on the **Cut Property**, proven via an exchange argument in the same style Week 4's Greedy Algorithms established.

Today opens Dijkstra's Algorithm — and it is not a new mechanism built from nothing. It's the single-source shortest-path sibling of yesterday's Prim's: both repeatedly pull the frontier's cheapest candidate from a min-heap and use it to improve its neighbors. The difference is only *what* "cheapest" measures (total path length from a fixed source, vs. the weight of one edge extending a growing tree) and *what* gets proven (a different property than the Cut Property, but the same "the greedy choice can be shown correct because any competitor is already worse" proof shape). That parallel is drawn explicitly below, not left implicit.

On the theory side, today opens System Design's data-scaling track in a new direction: Sharding, which splits *different* data across multiple machines — the complementary technique to Replication (Week 9, Day 61), which copies the *same* data across multiple machines.

---

## Learning Objectives

By the end of today, without notes:

1. State Dijkstra's Algorithm's mechanism precisely, and prove — not assert — why the greedy choice (finalize whichever frontier node has the smallest tentative distance) is always correct, given non-negative edge weights.
2. Explain exactly why Java's `PriorityQueue` (no `decrease-key` operation) forces the "insert a new entry, skip stale ones on pop" implementation pattern, and how that choice affects the complexity bound.
3. Recognize when a shortest-path problem needs Dijkstra's *shape* but a different relaxation rule — today's Path with Maximum Probability flips both the heap direction and the combining operator.
4. Explain hash-based vs. range-based sharding, name the "celebrity problem," and connect naive modulo-based sharding directly back to why Consistent Hashing (Week 9, Day 60) exists.

---

## Concept Dependency Map

```
Heaps, full mechanism (Day 54) ─────────────┐
PriorityQueue in practice (Days 17, 26, 54) ─┤
Graph adjacency list (Day 68) ───────────────┼──▶ Prim's Algorithm, MST (Day 77)
Weighted edges, matrix form (Day 73, F-W) ───┘         same "pop cheapest frontier
                                                         candidate, relax neighbors"
                                                         shape, min spanning TREE weight
                                                                │
                                                                ▼
                                              Dijkstra's Algorithm (TODAY) — same shape,
                                              minimizes cumulative SOURCE-TO-NODE distance
                                                                │
                              ┌─────────────────────────────────┴─────────────────────────────┐
                              ▼                                                                 ▼
        Problem 1: Network Delay Time (LC 743)                    Problem 2: Path w/ Max Probability (LC 1514)
        min-heap · sum edge weights · textbook shape              max-heap · multiply edge weights · flipped shape

Replication Models (Day 61) ──▶ contrasted with ──▶ Sharding Strategies (NEW, today)
Consistent Hashing (Day 60) ──▶ direct fix for  ──▶ naive hash-mod-N sharding's remap problem
```

---

# Part 1 — Dijkstra's Algorithm

### Concept Card — Dijkstra's Algorithm (Single-Source Shortest Path, Non-Negative Weights)

**Prerequisites, confirmed:**
- Heaps / `PriorityQueue` mechanism, full array-based binary heap — Day 54, exercised again Days 55–58.
- Graph representation and traversal (adjacency list, `visited` discipline) — Day 68, closed Day 73.
- A weighted-edge graph representation already exists in this series — Day 73's Floyd-Warshall used a weight *matrix* for all-pairs distances. Today reuses the same idea of a weighted edge but stores it as a weighted adjacency **list** instead, because today's problems ask for shortest paths from *one* source, not *every* pair — building a full V×V matrix to answer a single-source query would waste O(V²) space for no benefit.
- Prim's Algorithm (Day 77) — mechanically, today's algorithm is a five-line diff from Prim's. Both hold a min-heap of `(candidate value, node)` pairs, both pop the smallest, both relax neighbors, both skip a node that's already finalized. Prim's candidate value is "weight of the edge that would connect this node to the growing tree"; Dijkstra's is "total distance from the source, along the path used to reach this node so far." Everything else — the loop shape, the heap discipline, the finalize-on-pop pattern — carries over unchanged.

**What it is:** Dijkstra's Algorithm finds the shortest path from a single source node to every other reachable node in a weighted graph, provided **every edge weight is non-negative.** It maintains a running "best distance found so far" for every node, and repeatedly *finalizes* whichever not-yet-finalized node currently has the smallest tentative distance — on the claim that once a node is finalized, its tentative distance is already the true shortest distance and will never improve again.

**Mechanism, in depth:**

1. Build a weighted adjacency list: `Map<Integer, List<int[]>>`, where each entry `{neighbor, weight}` describes one directed edge.
2. Initialize `dist[source] = 0`, every other `dist[node] = ∞` (in Java, `Integer.MAX_VALUE` or a sufficiently large sentinel — watch for overflow if you add a weight to `MAX_VALUE` directly; guard the addition or use `long`).
3. Push `(0, source)` onto a min-heap ordered by distance.
4. While the heap isn't empty: pop the `(d, node)` pair with smallest `d`.
   - **If `d > dist[node]`, this entry is stale — skip it.** (Explained below — this check is not optional bookkeeping, it's what keeps the algorithm correct *and* efficient under Java's `PriorityQueue`.)
   - Otherwise, `node` is now finalized. For each `(neighbor, weight)` in `node`'s adjacency list: compute `candidate = dist[node] + weight`. If `candidate < dist[neighbor]`, this is a *relaxation* — update `dist[neighbor] = candidate` and push `(candidate, neighbor)`.
5. When the heap empties, every reachable node's `dist[]` entry holds the true shortest distance from the source.

**Why the stale-entry check exists — a Java-specific implementation detail, not a cosmetic one:** the textbook description of Dijkstra's assumes a heap supporting `decrease-key` — when a node's distance improves, you *update its existing heap entry in place*. Java's `PriorityQueue` doesn't expose that operation (doing it "properly" would need a custom indexed heap). The standard, correct workaround is **lazy deletion**: instead of updating an entry, push a brand-new `(candidate, neighbor)` pair every time a relaxation succeeds, leaving the old, now-stale entry sitting in the heap. When a stale entry is eventually popped, `dist[node]` will already hold something smaller (set by whichever *later* push actually finalized that node first) — the `d > dist[node]` check catches this and skips it. Without this check, the algorithm would still terminate and *usually* still be correct by luck, but it can waste time re-relaxing a node's neighbors using an already-superseded, larger distance, which is wasteful, not incorrect (relaxation can never make a distance *worse* — a stale, larger `d` simply fails every `candidate < dist[neighbor]` test it attempts). The check turns "wasteful" into "efficient": it caps total heap operations at the number of pushes, which is bounded by the number of edges.

**Why it works — the greedy correctness proof (an exchange argument, same shape as the Cut Property, different property):**

Claim: when Dijkstra's pops and finalizes node `u` with tentative distance `d[u]`, that value is already the true shortest distance `δ(u)`.

Proof, by induction on finalization order. The source finalizes first with `d=0=δ(source)`, trivially true (no negative weight can produce a smaller distance to itself). Assume every node finalized *before* `u` has a correct, true-shortest `d[]` value. Suppose, for contradiction, that `δ(u) < d[u]` — some shorter path `P` to `u` exists that the algorithm hasn't found yet. Since `u` isn't finalized until now, `P` must leave the already-finalized set at some point; let `x` be the *first* not-yet-finalized node on `P`, and `y` its predecessor on `P` (either `y` is already finalized, or `y` is the source). By the inductive hypothesis, `d[y] = δ(y)`. Because `P` is a shortest path, its prefix up to `x` is also a shortest path (a shortest path's sub-path is always itself shortest — cutting a detour into the middle of an already-shortest path can only add length), so `δ(x) = δ(y) + weight(y,x)`. The moment `y` was finalized, the algorithm relaxed edge `(y,x)`, which means `d[x] ≤ δ(y) + weight(y,x) = δ(x)`. **This is exactly where non-negative weights become essential:** since every edge weight from `x` onward to `u` along `P` is `≥ 0`, the prefix distance can't exceed the whole path's distance, so `δ(x) ≤ δ(u)`. Chaining these: `d[x] ≤ δ(x) ≤ δ(u) < d[u]`. But `x` is *not yet finalized*, and the algorithm claims to have popped `u` specifically because it had the **smallest** tentative distance among all not-yet-finalized nodes — `d[x] < d[u]` directly contradicts that. No such `x` can exist, so no shorter path `P` can exist either: `δ(u) = d[u]`. ∎

**The one-sentence version, for stating out loud in an interview:** any path that could beat the greedily-chosen node's distance would have to pass through some other unfinished, farther-or-equal node first — and since weights can't be negative, passing through that node first can only add distance, never recover it. That's the domination argument (same family as Week 4's frontier-domination proof for Jump Game) applied to a shortest-path frontier instead of a reachability frontier.

**Why (vs. BFS):** BFS (Week 10, Day 68 onward) finds shortest paths by *edge count* — every edge implicitly "costs" 1. The moment edges carry different weights, "fewest edges" and "lowest total weight" stop being the same question; a 1-edge path costing 100 is worse than a 3-edge path costing 6. BFS's level-order guarantee has nothing to say about weight.

**When to reach for it:** "minimum cost/time/distance" phrasing over a weighted graph; "cheapest," "fastest route," any problem where edges are explicitly weighted and all weights are non-negative.

**Trade-offs against the nearest alternatives:**
- **vs. Floyd-Warshall (Day 73):** Floyd-Warshall computes *all-pairs* shortest paths in O(V³), useful when you need distances between every pair. Dijkstra's computes *single-source* shortest paths in O(E log V) — far cheaper when you only need distances from one starting point, which is every problem today.
- **vs. Bellman-Ford (previewed today, formally taught tomorrow):** Bellman-Ford handles negative weights (and detects negative cycles) but costs O(V×E), strictly worse than Dijkstra's when weights are guaranteed non-negative.

**Complexity, derived:** with the lazy-deletion approach above, the heap holds at most one entry per successful relaxation. Each edge can trigger at most one relaxation from any given finalized node (each node is finalized exactly once, and processes each of its outgoing edges exactly once when it finalizes) — so there are at most O(E) pushes, plus O(V) for the initial handful of pushes. A binary heap holding up to O(E) entries has push/pop cost O(log E); since a simple graph has `E ≤ V²`, `log E ≤ 2 log V = O(log V)`. Total: O(E) pushes and pops, each O(log V) → **Time O(E log V), Space O(V + E)** (V+E for the adjacency list, up to O(E) for the heap).

**Common mistakes:**
- **⚠️ Using Dijkstra's with a negative edge weight.** The entire correctness proof above hinges on "passing through a farther node first can only add distance" — a negative edge breaks that outright: a longer-hop path could undercut an already-finalized "shorter" one *after* it's already been declared final. Dijkstra's has no mechanism to revisit a finalized node, so it produces a silently wrong answer, not a crash. This is exactly why Bellman-Ford exists as a slower, negative-weight-safe alternative.
- **⚠️ Forgetting the stale-entry check.** Not incorrect (see above), but throws away the entire complexity argument — worst case degrades toward repeatedly reprocessing nodes.
- **⚠️ Confusing "popped" with "finalized" before checking staleness.** Every popped entry needs the `d > dist[node]` guard *before* being treated as authoritative.
- **⚠️ Integer overflow when relaxing.** `dist[node] + weight` can overflow `int` if the sentinel for "unreached" is `Integer.MAX_VALUE` and you add to it without checking — Day 10/11's overflow habit applies directly. Guard with `if (dist[node] == INF) continue;` before adding, or use `long`.

**Edge cases:** an unreachable node (its `dist[]` entry never leaves the sentinel — report accordingly, e.g., `-1`); the source itself (`dist[source] = 0`, must be seeded before the loop starts); a graph with only one node.

---

## Problem 1: Network Delay Time (LeetCode 743, Medium) — Pattern: Dijkstra's Algorithm

**Statement:** Given a list of directed, weighted edges `times[i] = (u, v, w)`, a number of nodes `n` (labeled `1` to `n`), and a starting node `k`, return the minimum time for a signal starting at `k` to reach *every* node — or `-1` if some node is unreachable. The answer is the time the *last* node receives the signal, i.e., the maximum of all finalized shortest distances.

### Approach 1 — Brute force: BFS ignoring weights (deliberately wrong, worth seeing why)

A tempting first instinct is to reuse Week 10's BFS machinery directly, treating this as "shortest number of hops."

```java
// WRONG for this problem — included to show exactly why weight-blind BFS fails.
public int networkDelayTimeWrongBFS(int[][] times, int n, int k) {
    Map<Integer, List<int[]>> graph = buildAdjacency(times);
    int[] dist = new int[n + 1];
    Arrays.fill(dist, -1);
    dist[k] = 0;
    Queue<Integer> queue = new ArrayDeque<>();
    queue.offer(k);
    while (!queue.isEmpty()) {
        int node = queue.poll();
        for (int[] edge : graph.getOrDefault(node, List.of())) {
            int neighbor = edge[0], weight = edge[1];
            if (dist[neighbor] == -1) { // first time reached, by HOP COUNT
                dist[neighbor] = dist[node] + weight;
                queue.offer(neighbor);
            }
        }
    }
    // ... this can report the WRONG distance whenever a longer-hop, lower-weight
    // path exists to some node that a shorter-hop, higher-weight path reaches first.
    return -1; // placeholder — this approach is not completed, deliberately
}
```

**Why this is wrong, precisely:** BFS's `dist[neighbor] == -1` guard locks in a node's distance the *first* time it's reached — correct when every edge costs the same (1 hop), because the first arrival is guaranteed to be via the fewest hops, which BFS's level-order guarantee ensures. With weighted edges, the first arrival by hop count is **not** guaranteed to be the cheapest by total weight. A 1-hop path costing 100 would lock in `dist[x]=100` before a 2-hop path costing `5+5=10` is even explored, and the guard would then refuse to update it. This is precisely why BFS is confined to unweighted graphs (or graphs where every edge has identical weight) in this series.

### Approach 2 — Optimized: Dijkstra's Algorithm

```java
public int networkDelayTime(int[][] times, int n, int k) {
    Map<Integer, List<int[]>> graph = new HashMap<>();
    for (int[] t : times) {
        graph.computeIfAbsent(t[0], x -> new ArrayList<>()).add(new int[]{t[1], t[2]});
    }

    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]); // {distance, node}
    pq.offer(new int[]{0, k});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int d = curr[0], node = curr[1];
        if (d > dist[node]) continue; // stale entry — skip

        for (int[] edge : graph.getOrDefault(node, List.of())) {
            int neighbor = edge[0], weight = edge[1];
            int candidate = d + weight;
            if (candidate < dist[neighbor]) {
                dist[neighbor] = candidate;
                pq.offer(new int[]{candidate, neighbor});
            }
        }
    }

    int maxDist = 0;
    for (int node = 1; node <= n; node++) {
        if (dist[node] == Integer.MAX_VALUE) return -1; // some node unreachable
        maxDist = Math.max(maxDist, dist[node]);
    }
    return maxDist;
}
```

This is the Concept Card's mechanism applied directly, with one problem-specific detail: the answer isn't `dist[]` itself, it's `max(dist[])` — "when does the *last* node hear the signal" is exactly "what's the largest finalized distance across every node."

**Worked trace:** `times = [[2,1,1],[2,3,1],[3,4,1]]`, `n=4`, `k=2`.

Adjacency: `2 → [(1,1), (3,1)]`, `3 → [(4,1)]`. `dist = [_, ∞, 0, ∞, ∞]` (index 0 unused).

| Pop | d | node | stale? | relax | new dist[] |
|---|---|---|---|---|---|
| 1 | 0 | 2 | no | 1: 0+1=1 (<∞, push); 3: 0+1=1 (<∞, push) | dist[1]=1, dist[3]=1 |
| 2 | 1 | 1 (or 3, tie) | no | 1 has no outgoing edges | unchanged |
| 3 | 1 | 3 | no | 4: 1+1=2 (<∞, push) | dist[4]=2 |
| 4 | 2 | 4 | no | no outgoing edges | unchanged |

Final `dist = [_, 1, 0, 1, 2]`. Max over nodes 1–4: `max(1,0,1,2) = 2`. **Answer: 2.**

**Complexity:** Time O(E log V), Space O(V+E) — as derived in the Concept Card above. `n ≤ 100`, `times.length ≤ 6000` in LeetCode's actual constraints, so this comfortably fits.

**Edge cases:**
- A node with no outgoing edges (never relaxes anything, but can still be *reached* — handled naturally, no special case).
- `k` itself never receiving any relaxation (correct — `dist[k]=0` is seeded directly, not derived).
- Any node still at `Integer.MAX_VALUE` after the loop → unreachable → return `-1` immediately, don't let it pollute the `max()`.
- **⚠️ Nodes are 1-indexed in this specific problem** (`1` to `n`) — size arrays `n+1` and iterate from `1`, not `0`. A LeetCode-specific off-by-one, not a general Dijkstra's fact.

**💡 Interview Insight:** state the brute-force-BFS temptation and *why it's wrong* out loud before writing Dijkstra's — this is one of the few problems where naming the wrong-but-tempting approach and explaining precisely why it fails (not just "BFS doesn't handle weights," but the specific "first arrival ≠ cheapest arrival" mechanism) demonstrates more understanding than jumping straight to the correct code. The likely follow-up: "what if weights could be negative?" — answer: Dijkstra's breaks (see the Concept Card's common-mistakes section), and Bellman-Ford (tomorrow) is the fix.

---

## Problem 2: Path with Maximum Probability (LeetCode 1514, Medium) — Pattern: Dijkstra, Multiplicative

**Statement:** Given an undirected weighted graph where each edge's weight is a *success probability* (0 to 1), and a `start`/`end` node pair, return the maximum probability of any path from `start` to `end` — the product of that path's edge probabilities. Return `0` if no path exists.

### Approach — Optimized: Dijkstra's, flipped

Two things flip simultaneously, and both flips are consequences of the *same* underlying fact: **probabilities shrink as a path gets longer** (multiplying by a number ≤1 never increases the product), whereas distances grow as a path gets longer. Dijkstra's greedy proof relied on "the frontier's smallest tentative value is final, because nothing can make it smaller by detouring through something currently worse" — here, "worse" now means "lower probability," so the greedy pick must be the frontier's **largest** tentative value, popped from a **max-heap**.

```java
public double maxProbability(int n, int[][] edges, double[] succProb, int start, int end) {
    Map<Integer, List<double[]>> graph = new HashMap<>();
    for (int i = 0; i < edges.length; i++) {
        int u = edges[i][0], v = edges[i][1];
        double p = succProb[i];
        graph.computeIfAbsent(u, x -> new ArrayList<>()).add(new double[]{v, p});
        graph.computeIfAbsent(v, x -> new ArrayList<>()).add(new double[]{u, p}); // undirected
    }

    double[] prob = new double[n];
    prob[start] = 1.0; // probability of "being at the source, having gone nowhere" is certainty

    PriorityQueue<double[]> pq = new PriorityQueue<>((a, b) -> Double.compare(b[0], a[0])); // MAX-heap
    pq.offer(new double[]{1.0, start});

    while (!pq.isEmpty()) {
        double[] curr = pq.poll();
        double p = curr[0];
        int node = (int) curr[1];
        if (node == end) return p; // popped from a max-heap: first time END is finalized, it's optimal
        if (p < prob[node]) continue; // stale entry — skip

        for (double[] edge : graph.getOrDefault(node, List.of())) {
            int neighbor = (int) edge[0];
            double candidate = p * edge[1]; // MULTIPLY, not add
            if (candidate > prob[neighbor]) { // GREATER, not smaller
                prob[neighbor] = candidate;
                pq.offer(new double[]{candidate, neighbor});
            }
        }
    }
    return 0.0; // end never finalized — unreachable
}
```

**Why the correctness proof still holds, with signs flipped:** the Concept Card's proof used "no detour through a farther node can help, since weights are non-negative" — here the analogous fact is "no detour through a lower-probability node can help, since every probability is `≤ 1`, so multiplying by an additional edge can only shrink (or hold, at `p=1`) a running probability, never grow it." The proof structure — contradiction via a not-yet-finalized node that would have to already be in the frontier with a better value — carries over by swapping every "≤" for "≥" and every "add" for "multiply."

**Worked trace:** `n=3`, edges `[[0,1],[1,2],[0,2]]`, probs `[0.5, 0.5, 0.2]`, `start=0`, `end=2`.

Two paths exist: direct `0→2` at `0.2`, or `0→1→2` at `0.5×0.5=0.25`. The longer path wins because its per-edge probabilities are both high.

`prob = [1.0, 0, 0]`. Pop `(1.0, 0)`: relax `1`: `1.0×0.5=0.5 > 0` → push `(0.5,1)`; relax `2`: `1.0×0.2=0.2 > 0` → push `(0.2,2)`. Pop `(0.5, 1)` (max-heap picks the larger of `0.5` and `0.2`): relax `2`: `0.5×0.5=0.25 > 0.2` (current `prob[2]`) → update, push `(0.25, 2)`. Pop `(0.25, 2)`: `node == end` → **return `0.25` immediately.** The stale `(0.2, 2)` entry is never even reached — the algorithm short-circuits the instant `end` is popped, since a max-heap guarantees nothing later in the heap can beat what's already been popped.

**Complexity:** Time O(E log V), Space O(V+E) — identical shape to Problem 1; only the comparator and combining operator changed.

**Edge cases:**
- `start == end`: `prob[start]` is already `1.0`, and the very first pop returns it immediately — correct, since "the probability of a zero-length path" is certainty.
- No path exists between `start` and `end`: the heap empties without ever popping `end` → falls through to `return 0.0`.
- A probability of exactly `0` on some edge: never worth taking (multiplying by `0` can never beat *any* positive running probability), naturally deprioritized by the max-heap without special-casing.

**⚠️ Common Mistake:** forgetting that returning the instant `end` is popped is an optimization that specifically depends on this being a **max**-heap — with a min-heap (Problem 1's shape), popping the source-side "smallest distance" first doesn't let you stop early the same way, because relaxations can still find a *cheaper* path through nodes visited later. Here, once `end` is popped from a max-heap, nothing remaining in the heap or graph can produce something larger — the same domination argument as the Concept Card's proof, just pointed the other direction.

**💡 Interview Insight:** if asked how to avoid floating-point underflow on a very long path (repeated multiplication of many sub-1 values can drift toward `0` and lose precision), the standard fix — worth naming even without implementing it — is to take the `log` of every edge weight and **add** logs instead of multiplying raw probabilities, turning the problem back into a plain (additive, min-heap) Dijkstra's on `-log(p)` values, then exponentiate the final answer. Flagging this as extension material: not needed for LeetCode's test cases, but a real technique worth knowing exists.

---

# Part 2 — Sharding Strategies

### Prerequisites (confirmed)

- Replication Models (Week 9, Day 61) — single-leader, multi-leader, leaderless/quorum (`R+W>N`).
- Consistent Hashing (Week 9, Day 60) — TreeMap-backed hash ring, `ceilingKey()` + wraparound, built specifically to fix naive `%N` hashing's near-total-remap problem.

**What it is:** as a single database instance reaches its storage or throughput limits, **sharding** splits *different* rows of data across multiple independent database instances (shards), so no single machine has to hold or serve the entire dataset. This is the write/storage-scaling complement to Replication, which instead copies the *same* data onto multiple machines for redundancy and read throughput — the two techniques solve different problems and are commonly used together, not as alternatives to each other.

**Two common sharding strategies:**

- **Range-based sharding:** assign contiguous key ranges to each shard (e.g., user IDs 1–1000 → shard A, 1001–2000 → shard B). Simple to reason about, and range queries (`WHERE id BETWEEN 500 AND 700`) stay cheap, since they typically touch only one or two shards. The failure mode: **hot spots** — if access is skewed toward one range (recently-created users are far more active than old ones, for instance), the shard holding that range absorbs disproportionate load while others sit idle.
- **Hash-based sharding:** compute `hash(key) % N` to pick a shard, spreading load evenly regardless of access skew by key range. The cost: range queries become expensive, since consecutive keys land on effectively random shards, and a range scan now has to fan out to *every* shard and merge results.

**🔗 Direct connection to Consistent Hashing (Week 9, Day 60):** naive hash-based sharding's `hash(key) % N` is *exactly* the scheme Consistent Hashing was built to replace. Recall Day 60's motivating problem: with plain `%N`, adding or removing a single shard changes `N`, which changes the modulus for essentially *every* key — a near-total remap, even though only one shard's worth of data logically needs to move. Consistent Hashing (the hash-ring, `ceilingKey()`, virtual nodes) fixes this by remapping only a proportional slice of keys when the shard count changes. In practice, "hash-based sharding" in a production system almost always means *consistent*-hash-based sharding, specifically to avoid this remap cost — plain modulo is the naive version worth understanding first, but not what you'd actually reach for.

**The celebrity problem:** neither range-based nor hash-based sharding protects against **one single key** receiving disproportionate traffic — a viral post, a bestselling product, a celebrity's account. Whichever shard that one key happens to land on absorbs a load spike no partitioning scheme prevents, because sharding strategies partition *keys*, and this failure mode is about one key being popular, not about which partition it's in. (Mitigations exist — read replicas for hot keys, application-level caching in front of the shard, splitting a single hot key's data further — but the sharding strategy itself, by construction, cannot solve this.)

**Trade-offs, side by side:**

| | Range-based | Hash-based |
|---|---|---|
| Range queries | Cheap (touch few shards) | Expensive (fan out to all) |
| Load distribution | Skews toward hot ranges | Even, by construction |
| Resharding cost | Split one range in two | Proportional remap only if consistent hashing is used |
| Celebrity problem | Not solved | Not solved |

**Common mistakes:**
- **⚠️ Treating Sharding and Replication as interchangeable.** They solve different problems (write/storage scale vs. read scale/redundancy) and are typically layered together (each shard is itself often replicated).
- **⚠️ Assuming hash-based sharding "solves" hot spots entirely.** It solves *range* skew (all traffic hitting one contiguous key range); it does nothing for the celebrity problem (one specific key being overloaded).

**Coding exercise (per plan):** sketch how the platform's `orders` table would be sharded by `userId` using hash-based sharding.

```
shardIndex = consistentHash(userId) % numShards   // conceptually; in practice, ring lookup, not raw %

Shard assignment sketch:
  userId=482 → hash → ring position → nearest shard clockwise → e.g. Shard 2
  userId=91  → hash → ring position → nearest shard clockwise → e.g. Shard 0
```

**What happens if one user creates a disproportionate number of orders:** hash-based sharding by `userId` means *all* of that one user's orders land on the *same* shard (since they all hash to the same key) — this is exactly the celebrity problem, applied to `orders`. A single unusually active user (a bulk-ordering business account, say) could overload one shard while the others stay idle. The fix isn't a different sharding key alone — it's usually a composite/secondary key (e.g., shard by `(userId, orderId)` or add a synthetic suffix) specifically for accounts identified as outliers, plus monitoring to catch this before it becomes an incident.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`. **Task:** add pagination to the platform's GET endpoints using Spring Data's `Pageable`/`PageRequest`.

**Practical guidance:**
- Repository layer: if `OrderRepository` already extends `JpaRepository<Order, Long>`, it already inherits `findAll(Pageable pageable)` — no new method needed, just a new *way of calling* what's already there.
- Controller layer: accept page/size as query parameters and construct a `Pageable`:
  ```java
  @GetMapping("/orders")
  public Page<Order> getOrders(
          @RequestParam(defaultValue = "0") int page,
          @RequestParam(defaultValue = "20") int size) {
      Pageable pageable = PageRequest.of(page, size);
      return orderRepository.findAll(pageable);
  }
  ```
- `Page<T>` (not `List<T>`) is the return type Spring Data gives back — it carries `content` (the actual items), `totalElements`, `totalPages`, and `number` (current page index) automatically serialized into the JSON response; no manual metadata wiring needed.
- `page` is 0-indexed by convention — `page=0` is the first page, matching the definition of done below.

**Definition of done:** `GET /orders?page=0&size=5` returns exactly 5 items plus page metadata (`totalElements`, `totalPages`, etc.) in the response body.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts from your network (no dedicated post today, per the plan).

**Networking:** follow up on any pending recruiter or alumni messages — a short, specific note referencing the actual thread, not a generic bump.

---

## Day 78 — Interview Questions

**Q1. State Dijkstra's Algorithm's core loop from memory.** Maintain a min-heap of `(distance, node)`, seeded with `(0, source)`. Repeatedly pop the smallest; if its distance is stale (greater than the current best known), skip it; otherwise finalize the node and relax every outgoing edge, pushing any improved neighbor distance as a new heap entry.

**Q2. Prove why the greedily-finalized node's distance is guaranteed correct.** Any path that could beat it would have to pass through some other not-yet-finalized node first; since all weights are non-negative, that node's own distance-so-far can't exceed the full path's length, meaning it would already sit in the frontier with a smaller tentative distance than the node just popped — contradicting that the popped node had the smallest tentative distance of anything in the frontier.

**Q3. Why does the proof specifically require non-negative weights?** The step "the prefix of a shortest path can't be longer than the whole path" only holds when every edge from that prefix onward adds a non-negative amount — a negative edge could make a longer-hop path cheaper than an already-finalized "shortest" one, and Dijkstra's has no mechanism to revisit a finalized node.

**Q4. Why does the implementation push a new heap entry on every relaxation instead of updating an existing one?** Java's `PriorityQueue` doesn't support `decrease-key`; pushing a new entry and later skipping stale ones (lazy deletion) achieves the same effect without a custom indexed heap.

**Q5. What does skipping a stale heap entry actually cost if you forget to do it?** Not correctness — relaxation only ever improves a distance, so a stale (too-large) entry simply fails its relax checks — but it costs the complexity bound, since the heap can be reprocessed with outdated values instead of being bounded by genuine relaxations.

**Q6. Derive Dijkstra's time complexity.** At most one heap push per successful relaxation, bounded by O(E) edges, plus O(V) initial pushes; each push/pop on a heap of size O(E) costs O(log E) = O(log V) since E ≤ V²; total O(E log V).

**Q7. Why does Path with Maximum Probability use a max-heap and multiplication instead of a min-heap and addition?** Probabilities are all ≤1, so extending a path by another edge can only shrink (or hold) the running product — the "detouring can't help" argument flips from "can't make it smaller" to "can't make it larger," so the greedy pick becomes the frontier's largest tentative value.

**Q8. In Path with Maximum Probability, why is it safe to return the instant `end` is popped?** Because it's popped from a max-heap — nothing else remaining in the heap or reachable through further relaxation can ever produce a larger probability than the current maximum.

**Q9. Why does plain BFS give the wrong answer on Network Delay Time?** BFS locks in a node's distance the first time it's reached, which is only guaranteed cheapest when every edge costs the same; with different weights, a longer-hop path can be cheaper than a shorter-hop one BFS would have already finalized.

**Q10. What's the LeetCode-specific gotcha in Network Delay Time's array sizing?** Nodes are labeled 1 to n, not 0-indexed — the `dist[]` array needs size `n+1`, and the final scan runs from node 1 through node n, not from 0.

**Q11. Distinguish hash-based sharding from range-based sharding, including their opposite failure modes.** Range-based keeps range queries cheap but is vulnerable to hot spots when access skews toward one range; hash-based spreads load evenly regardless of range skew but makes range queries expensive, since consecutive keys scatter across shards.

**Q12. What is the "celebrity problem," and why doesn't any sharding strategy solve it?** One specific key receiving disproportionate traffic — whichever shard it happens to land on gets overloaded regardless of the partitioning scheme, because sharding strategies balance load across the *key space*, not against one key's own popularity.

**Q13. How does naive hash-based sharding relate to Consistent Hashing?** Naive hash-based sharding uses `hash(key) % N`, which remaps nearly every key when N changes; Consistent Hashing is the fix — a hash ring where changing the shard count only remaps a proportional slice of keys, which is why production hash-based sharding is almost always consistent-hash-based in practice.

---

## Daily Deliverable Check

- [ ] Network Delay Time (LC 743) and Path with Maximum Probability (LC 1514) solved, with the greedy-correctness proof explainable from memory, pushed to `dsa-java/dijkstra/`.
- [ ] Can state precisely why the stale-entry check matters (correctness of the *bound*, not of the *answer*) and why negative weights break Dijkstra's outright.
- [ ] Sharding sketch complete (hash-based sharding of `orders` by `userId`, celebrity-problem risk identified) and connected explicitly to Consistent Hashing (Week 9, Day 60).
- [ ] Pagination live on the platform — `/orders?page=0&size=5` returns exactly 5 items plus page metadata.
- [ ] Pending recruiter/alumni messages followed up on.

---

## What Tomorrow Assumes You Already Know Cold

Day 79 needs today's Dijkstra's mechanism — the min-heap loop, the finalize-on-pop discipline, the stale-entry check — fully reflexive, since tomorrow doesn't re-derive any of it; it shows the mechanism *breaking* under a new constraint (a bounded number of stops) and being *modified* under a redefined notion of "distance" (a path's maximum edge, not its sum). Today's Problem 2 flip (max-heap, multiplicative relaxation) is the direct proof that Dijkstra's isn't one fixed algorithm but a *shape* that adapts to what's being optimized — tomorrow leans on that same instinct twice more. Today's Sharding discussion also sets up tomorrow's System Design topic indirectly: both are about what happens when a single-node assumption (one database, one participant in a transaction) stops holding at scale.
