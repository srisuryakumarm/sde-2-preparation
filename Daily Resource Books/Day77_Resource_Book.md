# Day 77 (Sunday) — Union-Find Capstone, Minimum Spanning Trees, and Grafana

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 76 Resource Book](Day76_Resource_Book.md)
**Next ▶:** [Day 78 Resource Book](Day78_Resource_Book.md)
**Companion to:** Day 77 of `Week_11_Revised.md`

---

## Recap

Union-Find sits at 6/7 required after four straight days of it — a redundant edge, a component count, an equation contradiction, a removal formula, an index rearrangement, an email merge. Today's final required problem asks one more connectivity question, then the week pivots: `UnionFind` stops being *the thing being learned* and becomes *a tool reused inside something new* — Kruskal's Algorithm, for Minimum Spanning Trees. That shift — from "here's a structure" to "here's an algorithm that calls that structure as a subroutine" — is itself worth noticing, since it's exactly what Week 12 will do again with Dijkstra's and Heaps.

---

## Self-Check (15 min)

Before today's new material, pick **one** Graph problem from earlier this week — not Union-Find — and solve it cold, no hints, no looking back at this week's Resource Books. Course Schedule II or Pacific Atlantic Water Flow are good picks specifically because each has a non-obvious correctness argument (the DFS-finish-order reversal; the reversed-comparison proof) that's easy to *recognize* when reading but harder to *reproduce* from a blank page. If it doesn't come together within a reasonable stretch, that's useful information — better to find the gap now than in front of an interviewer — and worth a targeted reread of the relevant Day 71–73 section before moving on.

---

## Learning Objectives

By the end of today, without notes:

1. Solve Graph Valid Tree, and explain why checking edge count *and* acyclicity together — without a separate connectivity scan — is provably sufficient.
2. State the Cut Property and use it to justify, with a real exchange argument, why Kruskal's greedy edge selection is always safe.
3. Trace Kruskal's Algorithm by hand on a 5-node weighted graph, and explain how Prim's reaches the same total weight through a structurally different process.
4. Implement Kruskal's Algorithm reusing the `UnionFind` class exactly as a cycle-detection subroutine.
5. Explain what Grafana adds on top of Prometheus, and write a PromQL query for HTTP request rate.

---

## Concept Dependency Map

```
Day 74: UnionFind class
        │
        ▼
TODAY: Graph Valid Tree (LC 261) — Union-Find CLOSES at 7/7
        │
        ▼
Minimum Spanning Trees (NEW — theory, no dedicated LC problem today)
├─ Kruskal's — sorts edges, reuses UnionFind AS-IS for cycle detection
├─ Prim's — grows from one node, reuses PriorityQueue
│  (Days 17/26/54) for cheapest-crossing-edge selection
└─ Correctness: the Cut Property — an exchange argument,
   same style established in Week 4's Greedy Algorithms

Independent theory track:
Day 76: Prometheus, Micrometer, /actuator/prometheus
        │
        ▼
TODAY: Grafana — visualizes what Prometheus already scrapes
```

---

# Part 1 — Union-Find Capstone

**Prerequisites, confirmed:** `UnionFind` (Day 74) ✅, all six of this week's prior Union-Find applications ✅.

## Problem 7: Graph Valid Tree (LeetCode 261, Medium — Premium in some regions) — Pattern: Union-Find

**Statement:** given `n` nodes labeled `0` to `n-1` and a list of undirected edges, determine whether they form a valid tree.

**⚠️ Access note, preserved from the plan:** this problem sits behind LeetCode Premium in some regions. If inaccessible, the algorithm and proof below stand on their own and are worth internalizing regardless — this exact "edge count + acyclicity" check is a genuinely common interview whiteboard question even when the specific LeetCode problem isn't reachable.

**Approach:** a graph with `n` nodes is a tree exactly when it's connected **and** acyclic **and** has exactly `n-1` edges — but checking all three independently is redundant. The efficient check:

1. If `edges.length != n-1`, return `false` immediately — too few edges can't possibly connect everything; too many guarantees a cycle exists somewhere.
2. Process every edge through `union()`. If any edge connects two nodes that already share a root, that edge closes a cycle — return `false`.
3. If every edge processed without a cycle, **and** step 1 already confirmed exactly `n-1` edges, the graph is guaranteed connected — return `true`, with no separate connectivity scan needed.

```java
public static boolean validTree(int n, int[][] edges) {
    if (edges.length != n - 1) return false;

    UnionFind uf = new UnionFind(n);
    for (int[] edge : edges) {
        if (!uf.union(edge[0], edge[1])) {
            return false;   // already connected — this edge closes a cycle
        }
    }
    return true;
}
```

### Why Step 3 Needs No Separate Connectivity Check — Proven, Not Asserted

This rests on a graph theory fact worth stating precisely: for a graph with `n` vertices, any **two** of the following three properties force the third — **(a)** connected, **(b)** acyclic, **(c)** exactly `n-1` edges.

Proof of the specific direction used here — **(b) + (c) ⟹ (a)**: an acyclic graph is a forest — a disjoint union of trees. If that forest has `c` components, and component `i` has `kᵢ` vertices, each component (being a tree itself) has exactly `kᵢ - 1` edges, so the whole forest has `Σ(kᵢ - 1) = n - c` edges total. Step 1 already fixed the edge count at `n - 1`, and step 2 already confirmed acyclicity (no cycle was ever found) — so `n - c = n - 1`, which forces `c = 1`: **exactly one component**, i.e., connected. The algorithm never needs to check connectivity directly because acyclicity plus the exact edge count *algebraically forces it*.

**Worked check:** `n=5, edges=[[0,1],[0,2],[0,3],[1,4]]` — `edges.length=4=n-1` ✓; processing all four edges finds no repeated root at any step → `true` (a valid, star-shaped tree). Contrast: `n=5, edges=[[0,1],[1,2],[2,3],[1,3],[1,4]]` — `edges.length=5 ≠ n-1=4` → `false` immediately, no union work even needed.

**Complexity:** Time O(n × α(n)). Space O(n).

**💡 Interview Insight:** stating the three-property relationship up front — "I only need to check two of the three conditions, because together they force the third" — is a stronger opening than diving into code, and it's exactly the kind of graph theory fact a tier-1 interviewer may push on directly ("why don't you also check connectivity?").

---

**This closes Union-Find: 7 required — up from 3–4 in the original plan, matching Graphs' closure last week as the second of the two thinnest gaps the original audit identified. Combined with yesterday's one extra (LC 1319), Union-Find closes at 8 distinct problems.**

---

# Part 2 — Minimum Spanning Trees: Kruskal's and Prim's

**Prerequisites, confirmed:** `UnionFind` (Day 74) ✅ — reused directly, unmodified, as Kruskal's cycle-detection subroutine. `PriorityQueue`/min-heap mechanics (Days 17, 26, 54) ✅ — reused directly by Prim's. Sorting (Day 3) ✅.

## Concept Card — Minimum Spanning Tree

**What it is:** given a connected, undirected, weighted graph, a **spanning tree** is a subset of its edges connecting every vertex with no cycles — which, tying directly back to today's first problem, means exactly `n-1` edges. A **minimum** spanning tree is the spanning tree whose edges sum to the smallest possible total weight.

**Interview signal:** "connect everything at minimum cost," "minimum cost to link all \[cities/computers/points\]," any problem describing a fully-connected cost structure where you need the cheapest way to make everything reachable.

### Kruskal's Algorithm

**Mechanism:** sort every edge by weight, ascending. Process edges in that order; for each, check whether its two endpoints are already in the same Union-Find set. If not, union them and add the edge to the MST. If they already share a root, skip it — adding it would close a cycle. Stop once `n-1` edges have been selected.

**This is a direct reuse, not a variation, of today's first problem's exact check** — "would this edge connect two nodes already in the same set" is the identical question Graph Valid Tree just asked, called here as a live filter during construction instead of a final validation.

```java
public static void kruskal(int n, int[][] edges) {
    // edges[i] = {u, v, weight}
    Arrays.sort(edges, (a, b) -> a[2] - b[2]);
    UnionFind uf = new UnionFind(n);

    List<int[]> mstEdges = new ArrayList<>();
    int totalWeight = 0, edgesUsed = 0;

    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], w = edge[2];
        if (uf.union(u, v)) {          // true only if this didn't close a cycle
            mstEdges.add(edge);
            totalWeight += w;
            edgesUsed++;
            if (edgesUsed == n - 1) break;   // MST complete — no need to check remaining edges
        }
    }

    System.out.println("MST edges:");
    for (int[] e : mstEdges) {
        System.out.println(e[0] + " - " + e[1] + " (weight " + e[2] + ")");
    }
    System.out.println("Total weight: " + totalWeight);
}
```

### Prim's Algorithm

**Mechanism:** start from any single node, treating it as the current (trivial) MST. Maintain a min-heap of candidate edges connecting the current MST to nodes outside it. Repeatedly pop the cheapest candidate; if its far endpoint is already inside the MST, it's a stale entry — discard and pop again. Otherwise, add that edge and its new endpoint to the MST, and push all of the new node's outgoing edges as fresh candidates. Repeat until every node is included.

```java
public static int prim(int n, List<List<int[]>> adj) {   // adj.get(u) = list of {neighbor, weight}
    boolean[] inMST = new boolean[n];
    PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> a[1] - b[1]);  // {node, weight}
    minHeap.offer(new int[]{0, 0});   // start arbitrarily at node 0

    int totalWeight = 0, nodesAdded = 0;
    while (nodesAdded < n) {
        int[] current = minHeap.poll();
        int node = current[0], weight = current[1];
        if (inMST[node]) continue;   // stale entry

        inMST[node] = true;
        totalWeight += weight;
        nodesAdded++;

        for (int[] edge : adj.get(node)) {
            if (!inMST[edge[0]]) {
                minHeap.offer(new int[]{edge[0], edge[1]});
            }
        }
    }
    return totalWeight;
}
```

### Correctness: The Cut Property — an Exchange Argument

**Statement of the property:** for any partition of the graph's vertices into two non-empty sets (a "cut"), the minimum-weight edge crossing that cut belongs to **some** minimum spanning tree.

**Proof, in the same exchange-argument style established back in Week 4's Greedy Algorithms:** let `e` be the minimum-weight edge crossing a cut `(S, V-S)`, and suppose some MST `T` doesn't contain `e`. Adding `e` to `T` creates exactly one cycle (since `T` is already a spanning tree, any extra edge closes exactly one). That cycle starts and ends on the same side of the cut, so it must cross the cut an **even** number of times — and since it crosses at least once (via `e` itself), it crosses **at least twice**, meaning some *other* edge `e'` in that cycle also crosses the cut. Since `e` was chosen as the *minimum*-weight crossing edge, `weight(e) ≤ weight(e')`. Swap `e'` out of `T` for `e`: the result is still a spanning tree (removing one cycle edge, adding another edge that reconnects the same two pieces), with total weight no greater than `T`'s original weight. So a spanning tree at least as good as `T`, and containing `e`, always exists — proving some MST contains `e`.

**Why this justifies both algorithms:** Kruskal's, at each step, is implicitly choosing the minimum-weight edge crossing the cut between "the two components this edge would merge" and everything else — the Cut Property guarantees that choice is always safe. Prim's, at each step, chooses the minimum-weight edge crossing the cut between "the current tree" and "everything not yet included" — the identical property, applied to a different, evolving cut.

**Complexity:** Kruskal's — Time O(E log E) (sorting dominates; `E log E = E log V` since `E ≤ V²`) + O(E × α(V)) for the union operations ≈ **O(E log E)**. Space O(V+E). Prim's — Time **O(E log V)** with a binary heap (each edge pushed at most once, each push/pop costs O(log E)). Space O(V+E).

**Trade-off — which to reach for:** Kruskal's is the natural choice **today specifically**, because `UnionFind` already exists and this reuses it directly with zero new machinery — and it fits sparse, edge-list-shaped graphs well. Prim's is often preferred on dense graphs (with an adjacency matrix and a simple O(V²) array-scan instead of a heap, no `log` factor at all) or when the algorithm needs to grow outward from a specific starting node for other reasons. If asked "what's the alternative to what you just built" — naming Prim's, and the dense-vs-sparse trade-off, is the expected follow-up.

### Worked Trace — Kruskal's, by Hand

A 5-node graph, weights arbitrary but chosen to include a tie (to show tie-breaking doesn't threaten correctness):

```
Edges: (0,2)=1  (1,2)=2  (3,4)=2  (0,1)=4  (1,3)=5  (2,3)=8  (2,4)=10
```

Sorted ascending: `(0,2,1), (1,2,2), (3,4,2), (0,1,4), (1,3,5), (2,3,8), (2,4,10)`.

| Edge processed | Roots before | Same root? | Action | MST weight so far | Components |
|---|---|---|---|---|---|
| (0,2,1) | 0, 2 | No | Add | 1 | {0,2}, {1}, {3}, {4} |
| (1,2,2) | 1, root of {0,2} | No | Add | 3 | {0,1,2}, {3}, {4} |
| (3,4,2) | 3, 4 | No | Add | 5 | {0,1,2}, {3,4} |
| (0,1,4) | both root of {0,1,2} | **Yes** | Skip — cycle | 5 | (unchanged) |
| (1,3,5) | root of {0,1,2}, root of {3,4} | No | Add | 10 | {0,1,2,3,4} — done |

4 edges selected (`n-1=4`), loop exits early — **MST total weight: 10**, edges `{(0,2,1), (1,2,2), (3,4,2), (1,3,5)}`.

**Verifying this is genuinely minimum, not just *a* spanning tree:** the three cheapest edges (1, 2, 2) already connect `{0,1,2}` and `{3,4}` as two separate pieces at cost 5. Exactly one more edge is needed to bridge them, and every edge that actually crosses between the two pieces is `(1,3,5)`, `(2,3,8)`, or `(2,4,10)` — the cheapest of those three is `5`, exactly what Kruskal's selected. No cheaper valid tree exists.

**Prim's on the same graph, starting from node 0, reaches the identical total:** `0→2` (1) → `2→1` (2) → `1→3` (5) → `3→4` (2), total `1+2+5+2=10` — the same weight, discovered through a different process (growing outward from one node rather than globally sorting all edges first), landing on the same edge set here since this graph's MST happens to be unique.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals` (Kruskal's implementation) and local Docker Compose (Prometheus + Grafana).

**Task 1 — Kruskal's:** implement Kruskal's using the existing `UnionFind` class on a small hardcoded weighted graph (the 5-node example above works directly), printing the selected MST edges and total weight.

**Task 2 — Prometheus + Grafana locally, via Docker Compose:**

```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
    volumes: ["./prometheus.yml:/etc/prometheus/prometheus.yml"]
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    depends_on: [prometheus]
```

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'ecommerce-platform'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['gateway:8080', 'order-service:8081', 'payment-service:8082', 'inventory-service:8083']
```

In Grafana: add Prometheus as a data source (`http://prometheus:9090`), then build one dashboard panel using a PromQL query for HTTP request rate, reading Micrometer's default Spring Boot metric name:

```promql
rate(http_server_requests_seconds_count[5m])
```

This computes requests-per-second over a trailing 5-minute window, per label combination (method, URI, status) — directly visualizing the `application`-tagged metrics Day 76's project exposed.

**Definition of done:** Kruskal's prints the selected MST edges and total weight; the Grafana dashboard is live locally, showing real HTTP request rate data scraped from the platform's modules.

---

## Career Block Guide (1 hr)

**Weekly Industry Awareness Ritual (20 min):** clear the TLDR Newsletter backlog; read one engineering blog post.

**Weekly Scorecard:**

The plan's own Day 77 header states **"150 total DSA problems solved."** Cross-checking that figure against this map's own cumulative, required-ladder-only count (verified independently through Week 8's confirmed 110, plus Week 9's confirmed 14, plus Week 10's confirmed 14, plus this week's directly-counted 13) gives **151**, not 150 — a one-problem gap. This isn't a new error: it's the same drift this map already caught at Week 9's Day 63 (which stated "123," one below the verified 124), evidently carried forward uncorrected through both Week 10's and Week 11's own sequentially-incremented internal tallies rather than recomputed from scratch — and it stays perfectly self-consistent going forward, too: Week 12's own Day 84 header states "163," which is exactly `150 + 13` (Week 12's own required count), confirming the plan's tallies keep incrementing consistently from the same uncorrected baseline rather than the error resurfacing randomly. **This map uses 151 as the authoritative figure**, per its own established convention of using verified arithmetic over a manually-carried total, while reporting the plan's own "150" here transparently, flagged, exactly as this map handled the Week 9 case.

**Actual: Graphs fully closed at 12 (up from 9) — 14 distinct with Week 10's 2 extras. Union-Find fully closed at 7 (up from 3–4) — 8 distinct with this week's 1 extra. Both of the original audit's two thinnest gaps are now closed.** AWS networking/IAM and Prometheus/Grafana are both live on the platform.

---

## Day 77 — Interview Questions

**Q1. Why does Graph Valid Tree's algorithm skip a separate connectivity check?**
*A:* For `n` vertices, any two of {connected, acyclic, exactly `n-1` edges} force the third. The algorithm already confirms `n-1` edges (step 1) and acyclicity (step 2, no cycle found), which together algebraically force connectivity — a forest with `c` components has `n-c` edges, and `n-c=n-1` forces `c=1`.

**Q2. State the Cut Property.**
*A:* For any partition of a graph's vertices into two non-empty sets, the minimum-weight edge crossing that partition belongs to some minimum spanning tree.

**Q3. Prove the Cut Property using an exchange argument.**
*A:* If the minimum crossing edge `e` isn't in some MST `T`, adding `e` to `T` creates a cycle that must cross the cut an even number of times — so at least one other edge `e'` in that cycle also crosses it, with `weight(e) ≤ weight(e')` by `e`'s minimality. Swapping `e'` for `e` yields a spanning tree no heavier than `T`, containing `e` — so some MST contains `e`.

**Q4. How does Kruskal's algorithm reuse today's first problem's exact logic?**
*A:* "Would this edge connect two nodes already in the same Union-Find set" is the identical cycle-check Graph Valid Tree performs — Kruskal's calls it live during construction as a filter, rather than as a final pass/fail check.

**Q5. When would you prefer Prim's over Kruskal's?**
*A:* On dense graphs, where an O(V²) array-scan (no heap needed) beats Kruskal's O(E log E) sort — or when the algorithm needs to grow outward from a specific starting node for other structural reasons.

**Q6. Why does Prim's need to check `inMST[node]` after popping from the heap, rather than trusting every popped entry?**
*A:* The same node can be pushed multiple times at different candidate weights as the MST grows; once it's actually added, any remaining stale, higher-weight entries for it in the heap must be discarded when popped, or they'd be incorrectly reprocessed.

**Q7. What does Grafana add that Prometheus alone doesn't provide?**
*A:* Prometheus stores and can be queried for metrics, but doesn't visualize them — Grafana queries a data source like Prometheus (via PromQL) and renders dashboards/panels on top of it; Grafana itself stores no metrics.

**Q8. Why does `rate(http_server_requests_seconds_count[5m])` give a requests-per-second figure rather than a raw count?**
*A:* The underlying metric is a cumulative counter; `rate()` computes the per-second average rate of increase over the trailing window specified (5 minutes here), converting an ever-growing total into a meaningful current throughput figure.

**Q9. What closed this week, and what's still open going into Week 12?**
*A:* Graphs (12/12, closed Day 73) and Union-Find (7/7, closed today) are both fully closed — the two thinnest gaps the original plan audit identified. Dijkstra's and Dynamic Programming open next week, both previously unstarted.

**Q10. Why does the plan's own "150" figure not match this map's "151," and why is 151 used here instead?**
*A:* The gap traces back to a one-problem undercount first introduced in Week 9's own Day 63 scorecard, which appears to have been carried forward uncorrected rather than recomputed — confirmed by Week 12's own header being exactly `150+13`, a stable but uncorrected increment. This map uses its own independently-verified arithmetic as the authoritative figure, consistent with how it already handled the same situation at Week 9.

---

## Daily Deliverable Check

- [ ] Graph Valid Tree solved (or the algorithm fully internalized, if Premium-restricted) — Union-Find ladder complete at 7/7 required.
- [ ] Can state and prove the Cut Property, and use it to justify both Kruskal's and Prim's correctness.
- [ ] Kruskal's traced by hand on a self-sketched 5-node graph, and implemented using the existing `UnionFind` class — MST edges and total weight printed.
- [ ] Grafana dashboard live locally, showing real HTTP request rate from the platform.
- [ ] Weekly ritual complete. Scorecard reviewed, including the flagged 150-vs-151 discrepancy.

---

## Week 11 Consolidation

### What Actually Got Built

- **Graphs BFS/DFS closed at 12/12 required** (up from 9 in the original plan) — Topological Sort (Kahn's + DFS 3-state coloring), multi-source reachability (forward and reversed), single-source BFS on an implicit graph (Word Ladder), and Floyd-Warshall's all-pairs shortest path as a deliberate, bounded exposure to a non-traversal algorithm family. Combined with Week 10's 2 extras: **14 distinct Graphs problems total.**
- **Union-Find opened and closed within the week, at 7/7 required** (up from 3–4) — a brand-new data structure taught fully from scratch (naive → union-by-rank → path compression → combined O(α(n))), applied to cycle detection, component counting, equation satisfiability, a removal-ordering proof, index rearrangement, and email merging. Plus 1 extra (LC 1319, deliberately picked up from a Week 10 deferral note). **8 distinct Union-Find problems total.**
- **Minimum Spanning Trees** taught as a direct payoff of Union-Find — Kruskal's (reusing `UnionFind` as-is) and Prim's (reusing the `PriorityQueue` machinery from Days 17/26/54), both justified by a full Cut Property exchange argument, not asserted.
- **Two full theory arcs**: general networking (TCP/UDP, DNS, HTTP, Load Balancers) and AWS-specific networking/IAM/storage (VPC, subnets, IGW/NAT, Security Groups, IAM Roles vs. Users, S3 vs. EBS) — the second explicitly built on the first's vocabulary.
- **Observability, end to end, live on the platform**: Micrometer instrumentation → `/actuator/prometheus` → Prometheus scraping → Grafana visualization, a complete working pipeline, not just individual pieces.
- Gateway, JWT validation, and rate limiting (Week 10's work) reviewed and confirmed clean.

### Planned vs. Actual

**13/13 required problems solved (100%)**, plus 1 extra practice problem (Number of Operations to Make Network Connected) — **14 new distinct problems this week**, zero dropped or deferred. Both patterns this week's plan revision targeted (Graphs, Union-Find) closed exactly as planned, at their expanded counts.

### Diagnostic — Worth Checking Before Week 12

- Can the Union-Find complexity progression be stated **and proven** at each step (naive O(n) → union-by-rank-alone O(log n), via the doubling argument → combined with path compression O(α(n))) — not just the final number recalled?
- Can the reason `k` must be Floyd-Warshall's outermost loop be explained via the layer-completion argument, rather than recalled as an arbitrary rule?
- Can the Cut Property be stated and used to justify Kruskal's safety, from a blank page?
- Is the distinction between multi-source *reachability* (Day 72) and multi-source *distance* (Day 70) still sharp, or has it started to blur back into "BFS from multiple sources," undifferentiated?
- If any of these feel shaky rather than automatic, that's worth ten minutes against the relevant day above before Week 12 begins — everything in Week 12 assumes these are settled tools, not concepts still being actively reconstructed.

### What Week 12 Assumes

Week 12 opens **Dijkstra's Algorithm** — a genuinely new single-source weighted-shortest-path algorithm, but one that reuses two things wholesale: the `PriorityQueue` mechanics from Days 17/26/54 (Dijkstra's min-heap holds `(distance, node)` pairs, structurally similar to Prim's `(node, weight)` heap from today), and the specific insight Day 75's LinkedIn Post 16 previewed a week early — that BFS's "fewest edges" guarantee silently stops meaning "cheapest path" the moment edge weights differ. It also opens **Dynamic Programming**, the largest single pattern in the entire plan, building on the informal DP-flavored reasoning already previewed twice without the formal name — Week 3's Kadane's Algorithm, and this week's own Floyd-Warshall (Day 73). Neither topic requires anything from Union-Find or MST specifically; both requires this week's Graphs and general algorithmic-proof fluency to be solid, unremarkable background.

---

**Next:** [Day 78 Resource Book](Day78_Resource_Book.md) — Dijkstra's Algorithm Begins, and Sharding Strategies.
