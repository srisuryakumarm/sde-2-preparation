# Day 80 — Dijkstra's Capstone, and Kafka Streams

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 79 Resource Book](Day79_Resource_Book.md)
**Next ▶:** [Day 81 Resource Book](Day81_Resource_Book.md)
**Companion to:** Day 80 of `Week_12_Revised.md`

---

## Recap

Yesterday's Path With Minimum Effort redefined "distance" as a path's *maximum edge*, weighting the connection between two adjacent cells. Today's capstone problem reuses that exact minimax relaxation rule, with one precise difference worth stating up front rather than discovering mid-problem: today's "cost" attaches to **nodes** (each cell's own elevation), not **edges** (the difference between two cells) — a small but real shift in what gets compared during relaxation. The mechanism and its correctness proof carry over unchanged; only the quantity being compared moves from an edge property to a node property.

This closes Dijkstra's Algorithm for the series. On the theory side, today closes the loop on the Kafka arc that's been running since Week 7: topics and partitions (Day 48), consumer groups (Day 50), and schema-enforced contracts (Day 69) all feed directly into today's Kafka Streams, which processes events *inside* the application itself rather than only producing or consuming raw messages.

---

## Learning Objectives

By the end of today, without notes:

1. Adapt yesterday's minimax Dijkstra to a node-weighted variant, and state precisely what changed and what didn't.
2. Give the full accounting for why Dijkstra's Algorithm closes at 5 problems this week rather than receiving further extra practice.
3. Define `KStream` and `KTable` precisely, including the stream-table duality that explains *why* a `KTable` is really just a compacted view of a stream, not a different kind of thing entirely.
4. Sketch a Kafka Streams topology connecting two topics through a transformation step.

---

## Concept Dependency Map

```
Yesterday: minimax Dijkstra (EDGE-weighted: candidate = max(dist[curr], |heightDiff|))
        │
        └──▶ same shape, NODE-weighted instead: candidate = max(dist[curr], grid[neighbor])
                  │
                  ▼
             Problem 5: Swim in Rising Water (LC 778, Hard) — Dijkstra's CLOSES (5/5)
                  (alt. approach cites: Binary Search on the Answer — Wk5, Union-Find — Wk11
                   — the SAME pairing used yesterday, now on a node-weighted grid)

Kafka Fundamentals — topics as logs, partitions (Day 48) ─┐
Kafka Consumers — consumer groups (Day 50) ────────────────┼──▶ Kafka Streams (NEW)
Kafka Schema Registry — Avro contracts (Day 69) ───────────┘        KStream (event-by-event)
                                                                      KTable (latest-value-per-key)
                                                                      stream-table duality
```

---

# Part 1 — Dijkstra's Closes

## Problem 5: Swim in Rising Water (LeetCode 778, Hard) — Pattern: Dijkstra, Node-Weighted Minimax

**Statement:** An `n×n` grid where `grid[i][j]` is that cell's elevation. Starting at `(0,0)` at time `0`, water rises over time; at time `t` you can occupy or move between any cells with elevation `≤ t`. Find the minimum time at which a path exists from `(0,0)` to `(n-1,n-1)`.

**Reframing this as a minimax path problem:** the answer is the smallest possible value of "the highest elevation you're forced to stand on," minimized over every path from start to end — including the start and end cells themselves. This is minimax again, but the quantity being compared at each step is a **node's own value** (`grid[neighbor]`), not an edge property.

### Approach 1 — Brute force: DFS over every path, tracking the running max

Exponential — explores every path, no pruning, tracking `max(elevations visited)` along each and keeping the minimum across all paths.

### Approach 2 — Binary Search on the Answer + Union-Find

The same pairing used yesterday for Path With Minimum Effort, now applied to node values instead of edge differences: binary search over candidate time `T` (from `0` to `n²-1`); for each `T`, union every pair of adjacent cells where **both** have elevation `≤ T`; check whether `(0,0)` and `(n-1,n-1)` land in the same component (and that both individually have elevation `≤ T`, or neither could be stood on at all). The smallest feasible `T` is the answer.

### Approach 3 — Optimized: Dijkstra's, node-weighted minimax

```java
public int swimInWater(int[][] grid) {
    int n = grid.length;
    int[][] dist = new int[n][n];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
    dist[0][0] = grid[0][0]; // standing at the start already requires t >= grid[0][0]

    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]); // {maxElevationSoFar, row, col}
    pq.offer(new int[]{grid[0][0], 0, 0});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int t = curr[0], r = curr[1], c = curr[2];
        if (r == n - 1 && c == n - 1) return t; // finalized destination
        if (t > dist[r][c]) continue; // stale

        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= n || nc < 0 || nc >= n) continue;
            int candidate = Math.max(t, grid[nr][nc]); // NODE value, not an edge difference
            if (candidate < dist[nr][nc]) {
                dist[nr][nc] = candidate;
                pq.offer(new int[]{candidate, nr, nc});
            }
        }
    }
    return -1; // unreachable — cannot occur on a fully-connected grid, kept for completeness
}
```

**What changed from yesterday, precisely, and what didn't:** the relaxation line is `Math.max(t, grid[nr][nc])` instead of `Math.max(e, Math.abs(heights[r][c] - heights[nr][nc]))` — the compared quantity is the *destination cell's own elevation* rather than a *computed difference between two cells*. The seed value also changes: `dist[0][0] = grid[0][0]` (not `0`), because even standing still at the start already requires the water to be at least that high. Everything else — the min-heap, the finalize-on-pop discipline, the stale-entry check, and the correctness proof itself (extending a path can only hold-or-increase its running maximum, since `max(a,b) ≥ a` regardless of what's being compared) — is unchanged.

**Worked trace:** `grid = [[0,2],[1,3]]`.

`dist[0][0] = 0`. Pop `(0,0,0)`: relax right `(0,1)`: `max(0,2)=2` → push `(2,0,1)`; relax down `(1,0)`: `max(0,1)=1` → push `(1,1,0)`.
Pop `(1,1,0)`: relax up (no improvement, `dist[0][0]=0` already smaller); relax right to `(1,1)` — **the destination**: `max(1,3)=3` → push `(3,1,1)`.
Pop `(2,0,1)`: relax down to `(1,1)`: `max(2,3)=3`, current `dist[1][1]=3` — not strictly smaller, no update, no push.
Pop `(3,1,1)`: `(r,c)=(1,1)` is the destination → **return `3`.**

Both routes through the grid are forced to eventually touch elevation `3` (the destination cell's own value) — the algorithm correctly finds `3` as the unavoidable minimum, regardless of which route gets there.

**Complexity:** Time O(n² log n) — `V = n²` cells, `E = O(n²)` edges (≤4 per cell), so `O(E log V) = O(n² log(n²)) = O(n² · 2log n) = O(n² log n)`. Space O(n²).

**Edge cases:** `n=1` (start equals destination — `grid[0][0]` is both the seed and the immediate answer, returned on the first pop); a grid where every cell shares the same elevation (`dist` propagates that constant value everywhere, all paths tie).

**⚠️ Common Mistake:** seeding `dist[0][0] = 0` out of habit from earlier Dijkstra problems, instead of `grid[0][0]` — the starting cell's own elevation is part of the answer here, unlike Problems 1 and 3 where a path's cost genuinely starts at zero before any edge is traversed.

**💡 Interview Insight:** naming the precise distinction between yesterday's edge-weighted minimax and today's node-weighted minimax, unprompted, is a strong signal — it shows the mental model is "Dijkstra's is a shape with a pluggable relaxation rule," not "I memorized five separate algorithms this week."

---

### This closes Dijkstra's Algorithm: 5 problems — up from 3 in the original plan

Network Delay Time, Path with Maximum Probability, Cheapest Flights Within K Stops, Path With Minimum Effort, Swim in Rising Water — 5 required, 0 extra. Combined with last week's Union-Find fix, both of the thinnest gaps identified in the original audit are now closed.

**Why no extra practice was added to this pattern, stated explicitly rather than left implicit:** per this series' own precedent, a pattern's true opening day (Day 78) correctly receives none — Trees (Day 46), Heaps (Day 54), Tries (Day 59), Backtracking (Day 61), Graphs (Day 68), and Union-Find (Day 74) all received the same treatment. What's different about Dijkstra's is that the *rest* of the pattern doesn't receive any either, unlike Heaps (extras landed Days 55–56) or Union-Find (its one extra landed Day 75). Two reasons, together: first, this pattern's required ladder was *already* deliberately expanded by this week's own plan revision, specifically to close a gap an earlier audit flagged — padding further on top of a revision whose entire purpose was "stop being thin" would work against that revision's own intent, not reinforce it. Second, the five required problems already span every distinct relaxation shape this pattern actually tests in interviews: a textbook sum-minimizing shortest path (Problem 1), a multiplicative/max-heap flip of the same shape (Problem 2), a hop-constrained variant needing a different algorithm entirely (Problem 3, Bellman-Ford), and two minimax variants — one edge-weighted, one node-weighted (Problems 4 and 5) — each also paired with a genuinely distinct alternative approach (Binary Search on the Answer + Union-Find, cited twice). A sixth problem would be reinforcing an already-well-covered shape, not closing a gap, on a day whose time budget is already carrying a Hard problem and a full new theory thread.

---

# Part 2 — Kafka Streams

### Prerequisites (confirmed)

- Kafka Fundamentals — topics as append-only logs, partitions for parallelism, ordering guaranteed only *within* a partition (Week 7, Day 48).
- Kafka Consumers — consumer groups, each partition read by exactly one consumer within a group (Week 8, Day 50).
- Kafka Schema Registry — versioned Avro contracts, enforced at publish time (Week 10, Day 69).

**What it is:** Kafka Streams is a library for processing events **inside your own application**, directly against Kafka topics, rather than only producing and consuming raw messages and hand-rolling the processing logic around them (which is everything the platform has done through Day 69). It provides two core abstractions:

- **`KStream`** — an unbounded, record-by-record view of a topic: every event, in the order it was published (within a partition), processed one at a time as it arrives. Modeling "every individual order placed" is a `KStream`.
- **`KTable`** — a **changelog**: the *latest known value per key*, updated as new records for that key arrive. A new record for an existing key doesn't add a new independent entry — it **replaces** that key's current value, the same way an `UPDATE` (or `UPSERT`) would in a conventional table. Modeling "the current status of order #482, whatever it most recently changed to" is a `KTable`.

**Why a `KTable` works — the stream-table duality, precisely:** a `KTable` is not a fundamentally different kind of storage from a `KStream` — it's a **compacted view** of one. Internally, Kafka Streams backs a `KTable` with a changelog topic using Kafka's log-compaction feature (which retains only the most recent record per key, discarding older ones for that same key over time) plus, for fast local point-lookups, a materialized state store (commonly RocksDB) that the application queries directly instead of re-scanning the topic. Every `KTable` can be viewed as the stream of updates that built it (the changelog itself is a `KStream`), and any `KStream` can be turned into a `KTable` by aggregating it (e.g., "count of orders per user, updated live" is a `KTable` built by aggregating a `KStream` of individual order events). This duality is the actual mechanism, not just a naming convention — it's why the same underlying Kafka infrastructure (topics, partitions, replication) can serve both an event-by-event view and a latest-value view of the same data.

**When to reach for which:** `KStream` when every individual event matters on its own (audit logging every order, alerting on every failed payment). `KTable` when only the *current* state per key matters (a running total, a latest status) and the full history of how it got there is irrelevant to the consumer.

**Coding exercise (per plan):** a `KStream` topology reading `raw-order-events`, normalizing each value, writing to `processed-order-events`.

```java
StreamsBuilder builder = new StreamsBuilder();

KStream<String, RawOrderEvent> rawOrders = builder.stream("raw-order-events");

KStream<String, ProcessedOrderEvent> processedOrders = rawOrders
        .mapValues(raw -> normalize(raw)); // pure transformation, one record in, one record out

processedOrders.to("processed-order-events");

KafkaStreams streams = new KafkaStreams(builder.build(), streamsConfig);
streams.start();
```

`mapValues` (not `map`) is the right choice here specifically because only the *value* changes — the key (presumably an order or user identifier) stays the same, and `mapValues` preserves partitioning by key without triggering an unnecessary repartition, unlike `map`, which Kafka Streams must assume *might* have changed the key.

**Common mistakes:**
- **⚠️ Treating a `KTable` like a remotely-queried database table.** It's a continuously-updating materialized view, updated in real time as new stream records arrive — not something fetched fresh from a remote store on each query.
- **⚠️ Assuming `KStream` processing preserves order across the whole topic.** Day 48's guarantee still applies unchanged: ordering is guaranteed only *within* a single partition, never across an entire topic.
- **⚠️ Forgetting the changelog topic backing a `KTable` needs the same durability/replication consideration as any other Kafka topic.** It's not a free, purely in-memory structure — updates to a `KTable` still produce real, replicated writes.

**Edge cases:** a `KStream` receiving a record with a `null` value (Kafka Streams treats this as a **tombstone** for `KTable` semantics — it signals "delete this key" rather than "set this key's value to null," which matters if the same topology also builds a `KTable` from the same stream).

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`. **Task:** implement the `KStream` topology above as a standalone demo module.

**Definition of done:** pushed, running, verified via console consumer — publish a raw event to `raw-order-events`, confirm the normalized version appears on `processed-order-events`.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** second mock-interview swap with your accountability partner — include one Graph or Union-Find problem, since those are the freshest patterns right now, closed just last week and the week before.

---

## Day 80 — Interview Questions

**Q1. What changed between Path With Minimum Effort's relaxation rule and Swim in Rising Water's, and what stayed the same?** The compared quantity shifted from an edge property (the height difference between two adjacent cells) to a node property (the destination cell's own elevation) — the min-heap, finalize-on-pop discipline, and correctness proof are all unchanged, since both rely only on "extending a path can't decrease its running maximum."

**Q2. Why does Swim in Rising Water seed `dist[0][0] = grid[0][0]` instead of `0`?** The starting cell's own elevation is already part of the answer — you need the water at least that high just to stand at the start, unlike a sum-based Dijkstra problem where a zero-length path genuinely costs zero.

**Q3. Name the alternative approach to Swim in Rising Water, and what it costs relative to Dijkstra's.** Binary search on the candidate time `T`, combined with Union-Find unioning every adjacent pair with both elevations `≤ T`, checking start/end connectivity — costs an extra `log(maxElevation)` factor and repeated Union-Find construction versus a single Dijkstra pass.

**Q4. Give the full accounting for why Dijkstra's Algorithm received zero extra practice this week.** The required ladder was already deliberately expanded from 3 to 5 problems by this week's own plan revision specifically to close a known gap, and the five problems already span every distinct relaxation shape the pattern tests — sum-minimizing, multiplicative/max-heap, hop-constrained (a different algorithm), and two minimax variants each paired with a distinct alternative approach; a sixth would reinforce, not close, a gap.

**Q5. Define `KStream` and `KTable` precisely, in one sentence each.** `KStream` is an unbounded, event-by-event view of a topic, processed in arrival order within a partition. `KTable` is a changelog — the latest value per key, updated (not appended) as new records for that key arrive.

**Q6. Explain the stream-table duality — why is a `KTable` not a fundamentally different structure from a `KStream`?** A `KTable` is a compacted view of a stream: its changelog topic (retaining only the latest record per key via log compaction) is itself a `KStream` of updates, and any `KStream` can be aggregated into a `KTable` — the same underlying infrastructure serves both views.

**Q7. What actually backs a `KTable` internally?** A compacted changelog topic plus, typically, a local materialized state store (e.g., RocksDB) that Kafka Streams queries directly for fast point lookups, instead of re-scanning the topic on every query.

**Q8. Why does the `mapValues` operator preserve partitioning while `map` might not?** `mapValues` only transforms the value, leaving the key — and therefore the partition a record belongs to — untouched; `map` can change the key, which forces Kafka Streams to assume a repartition might be needed.

**Q9. What does a `null` value mean in a `KStream` feeding a `KTable`, and why does it matter?** A tombstone — an explicit signal to delete that key from the `KTable`'s materialized view, not to set the key's value to null; this only matters when the same topology is also building table semantics from the stream.

**Q10. Does Kafka Streams change Kafka's partition-ordering guarantee from Day 48?** No — ordering is still guaranteed only within a single partition, never across an entire topic; `KStream` processing inherits this unchanged.

---

## Daily Deliverable Check

- [ ] Swim in Rising Water (LC 778) solved — Dijkstra's ladder complete at 5 problems, zero extra, with the full reasoning for that decision explainable from memory.
- [ ] Can state the stream-table duality precisely and explain what actually backs a `KTable`.
- [ ] KStream demo module pushed, running, verified via console consumer.
- [ ] Second mock-interview swap completed, including a Graph or Union-Find problem.

---

## What Tomorrow Assumes You Already Know Cold

Day 81 opens Dynamic Programming — the largest pattern in the entire plan — and needs two things fully solid before it starts: recursion, the call stack, and base/recursive cases (Week 2, Day 8), since tomorrow's very first exercise re-traces `fib(5)`'s call tree from that day to *prove* the redundant-computation problem DP exists to solve; and `HashMap` (Week 1, Day 4), since top-down memoization is literally "recursion plus a HashMap cache," not a new storage mechanism. Nothing from this week's Dijkstra's or Kafka Streams work carries forward directly — Dynamic Programming is an unrelated pattern pair, picked up fresh tomorrow.
