# Day 79 — Dijkstra's: Constrained Variants, and 2PC vs. Saga

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 78 Resource Book](Day78_Resource_Book.md)
**Next ▶:** [Day 80 Resource Book](Day80_Resource_Book.md)
**Companion to:** Day 79 of `Week_12_Revised.md`

---

## Recap

Yesterday established Dijkstra's core mechanism — min-heap of `(distance, node)`, finalize-on-pop, relax neighbors, skip stale entries — and proved it correct via a domination argument that depends entirely on non-negative weights. Problem 2 then showed that mechanism is a *shape*, not a fixed recipe: flipping the objective (maximize instead of minimize) flips the heap direction and the combining operator, while the proof's skeleton survives unchanged.

Today pushes that same lesson further, in two different directions. Problem 3 adds a **constraint** (at most K stops) that breaks vanilla Dijkstra's core assumption — greedily finalizing a node the instant it's cheapest-so-far is no longer safe, because "cheapest" and "within budget" can pull in opposite directions. Problem 4 redefines what "distance" even *means* — the quantity being minimized becomes a path's *maximum* edge, not its *sum*.

---

## Learning Objectives

By the end of today, without notes:

1. Construct a concrete counterexample showing exactly why plain Dijkstra's — and even a naively-modified, "visited once" version of it — gives a wrong answer once a stops constraint is added.
2. State Bellman-Ford's mechanism and prove, by induction, why K+1 rounds of full-edge-list relaxation correctly bounds a path to at most K+1 edges.
3. Explain why a minimax relaxation rule ("this path's cost is its largest edge, not its total") still admits a Dijkstra-shaped greedy solution, and adapt the correctness proof to it.
4. State Two-Phase Commit's mechanism precisely, explain its specific blocking failure mode, and connect that cost directly to Day 64's cascading-failure argument.

---

## Concept Dependency Map

```
Yesterday: Dijkstra's core (min-heap, finalize-on-pop, relax, non-negative-weight proof)
        │
        ├──▶ constrained by a HOP COUNT — the "finalize permanently" assumption breaks
        │         │
        │         ▼
        │    Bellman-Ford (NEW) — no greedy finalization at all; K+1 full relaxation
        │    rounds against a FROZEN snapshot, each round = exactly one more allowed edge
        │         │
        │         ▼
        │    Problem 3: Cheapest Flights Within K Stops (LC 787)
        │         (alt: modified Dijkstra with (cost,node,stopsUsed) state — trap included)
        │
        └──▶ "distance" redefined as a path's MAXIMUM edge, not its sum — same greedy shape,
                  new relaxation rule
                  │
                  ▼
             Problem 4: Path With Minimum Effort (LC 1631)
                  (alt. approach cites: Binary Search on the Answer — Wk5 D32-33,
                   Union-Find — Wk11 D74-77)

Saga Choreography/Orchestration (RECAP — Wk10 D70, not re-taught) ─┐
Resilience4j / cascading-failure via held resources (Wk10 D64) ────┼──▶ Two-Phase Commit (NEW),
                                                                     │   contrasted against Saga
CAP's consistency/availability tension (Wk9 D59, cited briefly) ───┘
```

---

# Part 1 — Dijkstra's Constrained and Redefined

## Problem 3: Cheapest Flights Within K Stops (LeetCode 787, Medium) — Pattern: Bellman-Ford / Modified Dijkstra

**Statement:** Given directed flights `[from, to, price]`, a source, a destination, and `k`, find the cheapest price using **at most `k` stops** (i.e., at most `k+1` flights/edges). Return `-1` if impossible within that budget.

### Why plain Dijkstra's — even a "sensibly modified" version — genuinely fails here

Consider: `n=4`, flights `0→1 (100)`, `1→2 (100)`, `0→2 (500)`, `2→3 (100)`, `src=0`, `dst=3`, `k=1` (at most 1 stop, at most 2 flights).

There is exactly one *valid* path within budget: `0→2→3`, cost `500+100=600`, using node `2` as its one allowed stop. The path `0→1→2→3` is cheaper (`100+100+100=300`) but uses **two** stops — over budget, invalid.

Dijkstra's pops in cost order. The cheapest way to *reach node 2 at all*, ignoring stops, is via `0→1→2` at cost `200` — cheaper than `0→2` direct at `500`. A naive "modify Dijkstra's by also tracking stops, but still mark a node visited/finalized the first time it's popped" implementation pops `(cost=200, node=2, stops=2)` first. This state is over budget (`stops=2 > k=1`) and gets correctly rejected — **but if rejecting it also marks node `2` as "done, never revisit,"** the algorithm will never later consider the valid state `(cost=500, node=2, stops=1)`, sitting further back in the heap, and will incorrectly report `-1` — even though `600` is clearly achievable.

**The fix:** a node's "have I dealt with this already" status can't be a single boolean. It has to depend on *how many stops were used to get there*, because a costlier-but-fewer-stops arrival can unlock a cheaper *completion* that a cheaper-but-stop-exhausted arrival cannot. State here is really `(node, stopsUsed)`, not `node` alone — collapsing that second dimension away is the entire bug.

### Approach 1 — Brute force: exhaustive DFS

```java
// Explore every path within the stop budget, track the minimum cost found.
private int cheapestBruteForce(Map<Integer, List<int[]>> graph, int node, int dst,
                                 int stopsLeft, int costSoFar, int best) {
    if (node == dst) return Math.min(best, costSoFar);
    if (stopsLeft < 0) return best;
    for (int[] edge : graph.getOrDefault(node, List.of())) {
        best = cheapestBruteForce(graph, edge[0], dst, stopsLeft - 1, costSoFar + edge[1], best);
    }
    return best;
}
```

Explores every path up to `k+1` edges deep — worst case branches by out-degree at every level, exponential in `k`. **Time O(E^k)** roughly (bounded branching per level), **Space O(k)** for the recursion depth.

### Approach 2 — Modified Dijkstra, correctly handling the `(node, stopsUsed)` state

```java
public int findCheapestPriceDijkstra(int n, int[][] flights, int src, int dst, int k) {
    Map<Integer, List<int[]>> graph = new HashMap<>();
    for (int[] f : flights) {
        graph.computeIfAbsent(f[0], x -> new ArrayList<>()).add(new int[]{f[1], f[2]});
    }

    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]); // {cost, node, stopsUsed}
    pq.offer(new int[]{0, src, 0});

    int[] bestStopsSeen = new int[n]; // lowest stopsUsed we've actually PROCESSED from, per node
    Arrays.fill(bestStopsSeen, Integer.MAX_VALUE);

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int cost = curr[0], node = curr[1], stops = curr[2];
        if (node == dst) return cost; // cheapest valid completion — min-heap guarantees this is optimal

        // Reject over-budget states WITHOUT marking the node as exhausted —
        // a rejection here must not block a later, lower-stops arrival at the same node.
        if (stops > k) continue;
        if (stops >= bestStopsSeen[node]) continue; // truly no better than something already processed
        bestStopsSeen[node] = stops;

        for (int[] edge : graph.getOrDefault(node, List.of())) {
            pq.offer(new int[]{cost + edge[1], edge[0], stops + 1});
        }
    }
    return -1;
}
```

**⚠️ The exact bug this fixes:** `bestStopsSeen[node]` is only ever written *after* a state passes the `stops > k` check. An over-budget pop is discarded without touching `bestStopsSeen`, which is precisely what leaves room for a later, valid, higher-cost/lower-stops arrival at the same node to still be processed.

**Trace, on the counterexample above (`k=1`):**

| Pop (cost, node, stops) | dst? | stops>k? | stops≥bestStopsSeen[node]? | action |
|---|---|---|---|---|
| (0, 0, 0) | no | no (0≤1) | no (∞) | process; `bestStopsSeen[0]=0`; push (100,1,1), (500,2,1) |
| (100, 1, 1) | no | no | no (∞) | process; `bestStopsSeen[1]=1`; push (200,2,2) |
| (200, 2, 2) | no | **yes (2>1)** | — | **rejected — `bestStopsSeen[2]` untouched** |
| (500, 2, 1) | no | no | no (∞, unset) | process; `bestStopsSeen[2]=1`; push (600,3,2) |
| (600, 3, 2) | **yes** | — | — | **return 600** |

Matches the only valid path's cost exactly. **Complexity:** worst case still O(E log E) heap operations similar in shape to plain Dijkstra's, but with a larger effective state space (`node × stopsUsed` instead of just `node`) — in the worst case, a node can legitimately be processed once per distinct stop count up to `k`, so pushes are bounded by O(k×E) rather than O(E).

### Approach 3 — Optimized: Bellman-Ford, K+1 rounds (cleanest correctness argument)

**Concept Card — Bellman-Ford (Bounded-Hop / Negative-Weight-Safe Shortest Path)**

**Prerequisites, confirmed:** arrays (Day 2), graph edge-list representation (Day 68 onward, same idea as adjacency list, here used as a flat list since Bellman-Ford relaxes by *edge*, not by *node's neighbor list*).

**What it is:** instead of greedily finalizing nodes (Dijkstra's approach, which breaks under negative weights or hop constraints), Bellman-Ford relaxes **every edge in the graph**, **`k+1` times**, where each full pass is guaranteed to extend the "at most this many edges used" frontier by exactly one.

**Mechanism:**

```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    for (int round = 0; round <= k; round++) {      // k+1 rounds = at most k+1 edges
        int[] snapshot = dist.clone();                // freeze last round's values
        for (int[] flight : flights) {
            int u = flight[0], v = flight[1], w = flight[2];
            if (snapshot[u] != Integer.MAX_VALUE && snapshot[u] + w < dist[v]) {
                dist[v] = snapshot[u] + w;
            }
        }
    }
    return dist[dst] == Integer.MAX_VALUE ? -1 : dist[dst];
}
```

**Why it works — proof by induction on round number `r`:** claim — after `r` rounds, `dist[v]` equals the true minimum cost to reach `v` using **at most `r`** edges (or remains "unreached" if no such path exists). Base case `r=0`: `dist[src]=0` (reachable with zero edges, trivially minimal), everything else unreached — correct by definition. Inductive step: assume `dist[]` is correct for "at most `r`" after round `r`. In round `r+1`, every candidate `snapshot[u]+w` is tried against every edge `(u,v,w)`, where `snapshot[u]` is round `r`'s (already-correct, by hypothesis) "at most `r` edges" value. Any path reaching `v` in **at most `r+1`** edges is either a path using at most `r` edges (already reflected in `dist[v]`, which is never allowed to *increase*, so it's carried forward) or a path whose *final* edge is some `(u,v,w)`, with everything before that final edge reaching `u` in at most `r` edges — exactly what `snapshot[u]` captures. Trying every edge covers every such "exactly `r+1`-edge-ending" path, and taking the minimum against the carried-forward value covers both cases. So after round `r+1`, `dist[v]` is correct for "at most `r+1`" edges. ∎

**⚠️ Common Mistake — the single most important implementation detail:** relaxing against `dist[]` directly, in place, **without** the `snapshot` copy, is wrong. If round `r+1` updates `dist[u]` first and then immediately uses that *freshly updated, same-round* value to relax `dist[v]` via edge `(u,v)`, the algorithm has effectively chained two edges together within a single round — silently allowing more hops per round than intended, and breaking the "round `r` = at most `r` edges" invariant the whole proof depends on. The frozen snapshot is not a performance nicety; it's what makes the induction proof's premise actually hold.

**Complexity:** Time O(k×E) — `k+1` rounds, each an O(E) full scan of every edge. Space O(V) — `dist[]` plus the per-round `snapshot`.

**Why this is the better answer to lead with, even though the modified-Dijkstra approach also works:** Bellman-Ford's correctness proof needs no special-casing, no "don't mark this node exhausted" subtlety, and no larger state space — the round-bounded relaxation directly encodes the hop limit by construction. It's also strictly more general: it works unmodified even if some flight prices were negative (a route credit, say), which the Dijkstra-shaped approach fundamentally cannot handle, per yesterday's Concept Card.

**Edge cases:** `src == dst` (`dist[src]=0` already, correct with zero rounds needed — though the loop still runs harmlessly); no path exists within `k+1` edges at all → `dist[dst]` stays at the sentinel → `-1`; `k` larger than any path could need (extra rounds simply find nothing new to relax, cost O(E) each but harmless).

**💡 Interview Insight:** state the counterexample (the `0→1→2→3` vs. `0→2→3` scenario) unprompted before writing any code — this is the single strongest signal that you understand *why* the constraint breaks the naive approach, not just that a modification is needed. If pushed on which approach to implement live, Bellman-Ford's cleaner proof makes it the safer choice to code under pressure; naming the modified-Dijkstra alternative and its exact trap afterward, without implementing it, still demonstrates the full picture.

---

## Problem 4: Path With Minimum Effort (LeetCode 1631, Medium) — Pattern: Dijkstra, Minimax Variant

**Statement:** Given a grid of heights, find a path from the top-left to the bottom-right that minimizes the **maximum** absolute height difference between any two *consecutive* cells on the path (not the sum of differences — the single worst step).

### Approach 1 — Brute force: DFS over every path

Explore every path top-left to bottom-right, tracking the maximum step-difference along each, keep the minimum such maximum across all paths. Exponential — up to 4 directions at every cell, no pruning.

### Approach 2 — Binary Search on the Answer + Union-Find (a genuinely distinct approach)

Recall Binary Search on the Answer (Week 5, Days 32–33): "minimum X such that Y is achievable" is the tell, and Y here is "can you reach the bottom-right cell using only steps whose height-difference is ≤ some threshold `T`?" Binary search over candidate `T` values; for each candidate, union every pair of adjacent cells whose height difference is `≤ T` (Union-Find, Week 11), then check whether the start and end cells land in the same component. The smallest `T` for which they connect is the answer. This is a real, interview-valid alternative — it composes two already-mastered tools (Binary Search on the Answer, Union-Find) instead of introducing a new relaxation rule, at the cost of an extra `log(maxHeightDiff)` factor and rebuilding the Union-Find structure on every candidate `T`.

### Approach 3 — Optimized: Dijkstra's, minimax relaxation

```java
public int minimumEffortPath(int[][] heights) {
    int rows = heights.length, cols = heights[0].length;
    int[][] effort = new int[rows][cols];
    for (int[] row : effort) Arrays.fill(row, Integer.MAX_VALUE);
    effort[0][0] = 0;

    int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]); // {effort, row, col}
    pq.offer(new int[]{0, 0, 0});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int e = curr[0], r = curr[1], c = curr[2];
        if (r == rows - 1 && c == cols - 1) return e; // finalized destination — done
        if (e > effort[r][c]) continue; // stale

        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
            int stepDiff = Math.abs(heights[r][c] - heights[nr][nc]);
            int candidate = Math.max(e, stepDiff); // the FLIP: max, not sum
            if (candidate < effort[nr][nc]) {
                effort[nr][nc] = candidate;
                pq.offer(new int[]{candidate, nr, nc});
            }
        }
    }
    return 0; // unreachable is not possible on a fully-connected grid, but kept for completeness
}
```

**Why this is a Dijkstra shape, not a new algorithm:** "the grid is a graph in disguise" (Day 68's exact framing) — each cell is a node, each adjacent-cell pair is an edge weighted by their height difference. Yesterday's relaxation rule was `candidate = dist[node] + weight` (a running sum); today's is `candidate = max(effort[node], weight)` (a running maximum). **Why the greedy proof still holds:** the argument was "no detour through a currently-worse frontier node can help" — here "worse" means a larger running maximum, and since `max(a,b) ≥ a` always, adding one more step to a path can never *decrease* its running-maximum effort, only hold it steady or increase it. That's the exact non-negative-weight property Dijkstra's proof needed, just restated for a `max` combiner instead of a `+` combiner — the domination argument transfers without modification.

**Worked trace:** `heights = [[1,2,2],[3,8,2],[5,3,5]]`. Known answer: `2`, via the path staying along the bottom/right edge where every step differs by at most `2`.

- Pop `(0,0,0)`: relax right to `(0,1)`: `|1-2|=1`, `max(0,1)=1 < ∞` → push `(1,0,1)`. Relax down to `(1,0)`: `|1-3|=2`, `max(0,2)=2` → push `(2,1,0)`.
- Pop `(1,0,1)`: relax right to `(0,2)`: `|2-2|=0`, `max(1,0)=1` → push `(1,0,2)`. Relax down to `(1,1)`: `|2-8|=6`, `max(1,6)=6` → push `(6,1,1)`.
- Pop `(1,0,2)`: relax down to `(1,2)`: `|2-2|=0`, `max(1,0)=1` → push `(1,1,2)`.
- Pop `(1,1,2)`: relax down to `(2,2)` — **the destination**: `|2-5|=3`, `max(1,3)=3` → push `(3,2,2)`.
- Pop `(2,1,0)` (effort `2`, tied for next-smallest): relax right to `(2,1)`: `|5-3|=2`, `max(2,2)=2` → push `(2,2,1)`. Relax down: out of bounds.
- Pop `(2,2,1)`: relax right to `(2,2)` — the destination again: `|3-5|=2`, `max(2,2)=2 < 3` (current `effort[2][2]`) → **improves it**, push `(2,2,2)`.
- Pop `(2,2,2)`: `(r,c)=(2,2)` is the destination → **return `2`.**

Matches the known answer.

**Complexity:** Time O(rows×cols × log(rows×cols)) — O(E log V) with `V=rows×cols`, `E=O(V)` since each cell has ≤4 edges. Space O(rows×cols).

**Edge cases:** a 1×1 grid (start equals destination, answer `0`, returned on the very first pop); a grid where every cell has the same height (`effort=0` everywhere, first path found wins, all ties equally valid).

**⚠️ Common Mistake:** using `+` instead of `max` in the relaxation line out of pure habit from yesterday — this is the single most likely slip, precisely because the surrounding code is otherwise identical to Problem 1's shape.

**💡 Interview Insight:** naming *both* valid approaches (Dijkstra-minimax and Binary-Search+Union-Find) and their trade-off — Dijkstra's is a single pass with no extra log factor; Binary-Search+Union-Find is conceptually simpler to explain live but costs an extra `log(maxHeightDiff)` factor and repeated Union-Find rebuilding — is exactly the kind of unprompted trade-off discussion this series has been building toward.

---

# Part 2 — Two-Phase Commit vs. the Saga Pattern

**Note on today's theory:** Choreography and Orchestration were already fully defined in Week 10, Day 70 — recapped in one paragraph below, not re-taught. What's genuinely new today is Two-Phase Commit's full mechanism.

### 🔗 Recap: Saga — Choreography vs. Orchestration (Week 10, Day 70)

A multi-service business transaction, broken into a sequence of independent **local** transactions, chained by events — necessary because no single database transaction can span independently-deployed services. **Choreography:** no central coordinator; each service reacts to events it's subscribed to (this is exactly what the platform's Order→Payment flow already is, built on Day 48's Kafka pub/sub). **Orchestration:** one coordinator explicitly directs every step, trading a central dependency for full visibility of the flow in one place. A **compensating transaction** is not a database rollback — the earlier local transaction already *committed*; undoing its effect requires a new, explicit, forward-moving operation (e.g., "release the inventory reservation" as its own real operation, not an undo of the reservation's commit).

### Two-Phase Commit (NEW)

**Prerequisites, confirmed:** distributed transactions and why a single DB transaction can't span services (Week 10, Day 70); Resilience4j and the cascading-failure/thread-exhaustion argument (Week 10, Day 64).

**What it is:** a protocol for committing a transaction **atomically** across multiple independent participants (e.g., multiple services, each with their own database), coordinated by one designated **coordinator**.

**Mechanism, precisely:**

- **Phase 1 — Prepare (vote):** the coordinator asks every participant, "can you commit this?" Each participant does whatever is needed to *guarantee* it can follow through if told to — acquiring locks, writing an intent to durable storage — then replies `yes` or `no`. Critically, a participant that votes `yes` **cannot go back** on that promise; it must hold its locks and wait.
- **Phase 2 — Commit or Abort:** if *every* participant voted `yes`, the coordinator broadcasts "commit" and everyone finalizes. If *any* participant voted `no` (or didn't respond), the coordinator broadcasts "abort" and everyone rolls back.

**Why it blocks — the specific failure mode:** a participant that has voted `yes` is now in an in-between state: it has made a promise it can't unilaterally break, but hasn't yet been told the final outcome. **If the coordinator crashes during this window**, every participant that voted `yes` is stuck — holding its locks, unable to commit (it doesn't know if everyone else voted yes) and unable to abort (it already promised it could commit) — until the coordinator recovers and tells it what happened. This is *specifically* why 2PC doesn't scale well to microservices: it introduces a real, if narrow, availability cost tied directly to the coordinator's own uptime.

**🔗 Direct connection to Day 64:** this is the exact same *shape* of failure Resilience4j's cascading-failure argument described — a resource (there: a thread in a bounded pool; here: a lock, held by a participant) gets tied up waiting on something that isn't responding, and every caller stacked up behind that resource eventually stalls too. Day 64's argument was at single-call/thread-pool scope; 2PC's blocking cost is the identical mechanism at transaction/lock scope.

**2PC vs. Saga, contrasted directly:**

| | Two-Phase Commit | Saga |
|---|---|---|
| Consistency | Strong — atomic, all-or-nothing across every participant | Eventual — intermediate states are visible mid-sequence |
| Availability | Can block indefinitely if the coordinator fails mid-protocol | Each local transaction commits immediately, independently |
| Undo mechanism | True rollback (nothing committed until phase 2) | Compensating transactions (already-committed work reversed explicitly) |
| Where it fits | Rare in microservices; more common inside a single, tightly-coupled system | The default choice for cross-service business transactions in this platform |

**Common mistakes:**
- **⚠️ Believing 2PC "rolls back" the same way a database transaction does.** It's a genuinely different mechanism — coordinated *voting* before anyone commits anything, not undoing already-committed work.
- **⚠️ Conflating Saga's eventual consistency with "eventually consistent replication" (Day 61).** Related in spirit (both accept a temporary window of inconsistency for better availability) but at a different layer entirely — one's about replica convergence, the other's about a multi-step business transaction's intermediate visibility.

**Coding exercise (per plan):** 150 words on what the platform's Saga-based order flow does if the payment step fails *after* inventory has already been reserved.

> If `PaymentFailedEvent` fires after `InventoryReservedEvent` has already committed, the Order service — subscribed to `PaymentFailedEvent` via choreography — triggers the compensating action: publish an `InventoryReleaseEvent` that the Inventory service consumes to explicitly decrement the reserved count back down, restoring the stock as available. This is not a rollback; the original reservation *did* commit, and the release is a brand-new, independent local transaction whose entire purpose is to reverse that commit's real-world effect. The Order's own state transitions to a terminal `PAYMENT_FAILED` status rather than being deleted, preserving an audit trail of what was attempted. No 2PC-style lock is ever held across this window — every intermediate state (`RESERVED`, then briefly `RESERVED + PAYMENT_FAILED`, then `RELEASED`) is a fully committed, independently-visible state, which is exactly the availability trade Saga makes in exchange for giving up 2PC's atomicity guarantee.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`. **Task:** implement the compensating action described above.

**Practical guidance:** the Order module should already be listening for domain events (Day 48/70's choreography setup) — add a listener for `PaymentFailedEvent` that publishes a new `InventoryReleaseEvent`, consumed by the Inventory module to decrement its reserved count. Keep the compensating action itself idempotent (Day 50's manual-commit idempotency lesson applies directly here too — a redelivered `PaymentFailedEvent` shouldn't double-release inventory).

**Definition of done:** simulating a payment failure correctly triggers the compensating action, verified by checking the Order's final state and the Inventory's restored count.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** apply to 1 Tier B target company.

---

## Day 79 — Interview Questions

**Q1. Construct a concrete example where plain Dijkstra's gives the wrong answer once a stops constraint is added.** A cheap path reaching some intermediate node uses too many stops already; a pricier path to that same node uses fewer stops and has budget left to complete cheaply — Dijkstra's, which finalizes by cost alone, locks onto the cheap-but-stop-exhausted arrival and never explores the valid alternative.

**Q2. Why isn't "mark a node visited the first time it's popped" safe here?** Because the relevant state is `(node, stopsUsed)`, not `node` alone — a rejection for exceeding the stop budget must not block a later, valid arrival at the same node with fewer stops used.

**Q3. State Bellman-Ford's core loop and what each round guarantees.** Relax every edge in the graph once per round, against a frozen snapshot of the previous round's distances; after `r` rounds, every `dist[]` value is correct for "reachable using at most `r` edges."

**Q4. Why must Bellman-Ford relax against a snapshot rather than updating in place?** Updating in place lets one round's later edge relaxations use that same round's already-improved values, silently chaining multiple edges into a single round and breaking the "round `r` = at most `r` edges" guarantee the correctness proof depends on.

**Q5. Why does Bellman-Ford handle negative weights when Dijkstra's can't?** It never permanently finalizes a node mid-algorithm — every value stays open to improvement in a later round — so a negative edge that would undercut an already-"finalized" Dijkstra distance simply gets picked up in a subsequent relaxation round instead of being silently missed.

**Q6. Why does Path With Minimum Effort still admit a Dijkstra-shaped solution despite not summing weights?** The greedy proof only needs "extending a path by one more step can never decrease its cost" — true for both a running sum of non-negative weights and a running maximum, since `max(a,b) ≥ a` always holds.

**Q7. Name the alternative approach to Path With Minimum Effort that doesn't use Dijkstra's at all, and its cost.** Binary search on the answer (candidate effort threshold `T`) combined with Union-Find, unioning every adjacent-cell pair with a height difference `≤ T` and checking start/end connectivity — costs an extra `log(maxHeightDiff)` factor versus the single-pass Dijkstra approach.

**Q8. State Two-Phase Commit's two phases.** Prepare: the coordinator asks every participant to vote yes/no, and a `yes` vote is a binding promise. Commit/Abort: if every vote was yes, the coordinator tells everyone to commit; if any vote was no, everyone aborts.

**Q9. Precisely, when does 2PC block, and why?** When the coordinator crashes after a participant has voted yes but before that participant receives the final commit/abort instruction — the participant can't unilaterally proceed either way, since it's already promised to commit if told to, and has to wait for the coordinator to recover.

**Q10. Connect 2PC's blocking cost to a concept already covered.** The same mechanism as Day 64's cascading-failure argument — a resource (a lock here, a thread there) held while waiting on something unresponsive, stalling everything queued behind it — just at transaction scope instead of thread-pool scope.

**Q11. Why is a Saga's compensating transaction not the same thing as a database rollback?** The earlier local transaction already committed for real; there's nothing left to "roll back" — a compensating transaction is a new, independent, forward-moving operation that reverses the committed effect explicitly.

**Q12. Contrast what 2PC and Saga each trade away.** 2PC trades availability (blocking risk tied to the coordinator) for strong, atomic consistency; Saga trades strong consistency (a visible window of partial completion) for availability — every local step commits independently and immediately.

---

## Daily Deliverable Check

- [ ] Cheapest Flights Within K Stops (LC 787) and Path With Minimum Effort (LC 1631) solved, pushed — including the modified-Dijkstra trap explainable from memory, and the Bellman-Ford correctness proof.
- [ ] Can state Two-Phase Commit's mechanism and its blocking failure mode precisely, and can recap Saga (Choreography vs. Orchestration) from Week 10, Day 70 without re-deriving it.
- [ ] Saga compensating-action write-up complete (150 words, above) and implemented on the platform — `PaymentFailedEvent` correctly triggers `InventoryReleaseEvent`, idempotently.
- [ ] Applied to 1 Tier B target company.

---

## What Tomorrow Assumes You Already Know Cold

Day 80 needs today's minimax relaxation rule (`candidate = max(current, stepWeight)`) fully solid — Swim in Rising Water reuses the identical mechanism unchanged, just on a differently-shaped grid, and tomorrow's book won't re-derive why the greedy proof still holds. It also assumes the "grid is a graph in disguise" framing (Day 68, reinforced today) is fully reflexive, since tomorrow's problem never states "this is a graph problem" outright — recognizing that framing unprompted is the actual skill being tested.
