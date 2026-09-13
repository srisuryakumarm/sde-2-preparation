# Week 11 — Consolidated Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Source days:** [Day 71](Day71_Resource_Book.md) · [Day 72](Day72_Resource_Book.md) · [Day 73](Day73_Resource_Book.md) · [Day 74](Day74_Resource_Book.md) · [Day 75](Day75_Resource_Book.md) · [Day 76](Day76_Resource_Book.md) · [Day 77](Day77_Resource_Book.md)

70 questions across 7 days, covering Graphs' close (Topological Sort, multi-source reachability, Word Ladder, Floyd-Warshall), Union-Find's full arc (naive → optimized → 7 applications), Minimum Spanning Trees, and this week's two theory tracks (general networking + AWS, and observability). Organized by day, in the order each concept was actually taught — use it as a straight read-through for full-week review, or jump to a specific day for targeted drilling.

---

## Day 71 — Topological Sort, and Networking Fundamentals I

**Q1. What is a topological order, precisely, and why is it only defined for a DAG?**
*A:* A linear ordering of a directed graph's vertices such that for every edge `u → v`, `u` appears before `v`. It requires acyclicity because a cycle would force some node to appear both before and after another node in the same cycle — a contradiction no ordering can satisfy.

**Q2. Walk through Kahn's Algorithm from memory.**
*A:* Build an adjacency list and an in-degree array. Seed a queue with every node at in-degree 0. Repeatedly dequeue a node, append it to the result, and decrement the in-degree of each of its neighbors — enqueue any neighbor that drops to 0. If the final result contains every node, a valid order exists; if it's shorter, the remaining nodes are stuck in a cycle.

**Q3. Why does an empty seed queue at the very start correctly signal a cycle?**
*A:* If nothing has in-degree 0, every node has at least one unmet prerequisite — which, since there's no external "root" outside the graph, can only happen if every node's prerequisites trace back to itself through some cycle.

**Q4. Explain the DFS-based alternative, and its connection to something already taught.**
*A:* 3-state coloring — white (unvisited), gray (currently on the active DFS path), black (fully explored) — identical in spirit to Day 69's Find Eventual Safe States. A gray neighbor means a back edge onto the current path, i.e., a cycle. The topological order is the reverse of DFS finish order.

**Q5. Why is topological order the *reverse* of DFS finish order, not the finish order itself?**
*A:* A node can't finish (turn black) until everything reachable from it — including anything it points to — has already finished. So for any edge `u → v`, `v` always finishes before `u`. Reversing finish order restores the "prerequisite before dependent" requirement.

**Q6. What's the one-sentence difference between Course Schedule and Course Schedule II?**
*A:* Identical algorithm; Course Schedule reports whether every node got placed (a boolean), Course Schedule II reports the actual placement order (the array itself).

**Q7. Why check the complement/prerequisite relationship correctly — which node does the edge point *from*?**
*A:* For a `[course, prerequisite]` pair, the edge points `prerequisite → course` — the prerequisite must be processed (in-degree-wise) before the course can become eligible, so the course's in-degree is what gets incremented, and the prerequisite is what appears in the adjacency list's source position.

**Q8. What's the TCP three-way handshake, and why three steps rather than two?**
*A:* SYN (client proposes a starting sequence number), SYN-ACK (server acknowledges and proposes its own), ACK (client acknowledges the server's). Three steps are needed because a two-step handshake would only confirm one direction of communication works, not both.

**Q9. Why does UDP exist at all, given it drops TCP's reliability guarantees?**
*A:* For applications where TCP's retransmission/reordering machinery introduces more latency cost than the reliability is worth — live streaming, gaming, and DNS itself — a late-but-correct packet is worse than a dropped one.

**Q10. Trace DNS resolution from a cold cache to a returned IP.**
*A:* Resolver asks a root server (which points to the relevant TLD server) → TLD server (which points to the domain's authoritative nameserver) → authoritative nameserver returns the actual IP → result cached at every layer along the way per its TTL.

---

## Day 72 — Multi-Source Traversal, Reversed, and Networking Fundamentals II

**Q1. What's the difference between Day 70's multi-source BFS and Day 72's technique, even though both seed multiple starting points at once?**
*A:* Day 70 answers a *distance* question (minutes elapsed, using BFS's level-by-level guarantee specifically). Day 72 answers a *reachability* question (can this cell reach the border at all) — which doesn't need BFS's level structure, so DFS works equally well and is often simpler code.

**Q2. Why does Surrounded Regions mark border-connected `'O'`s safe *before* flipping anything, rather than flipping in a single pass?**
*A:* Flipping while scanning risks acting on a cell before the algorithm has confirmed whether it's actually border-connected. The two-phase split — mark safe first, sweep second — guarantees every decision is made with complete information.

**Q3. Why is Surrounded Regions' brute force O((mn)²), and what specifically fixes it?**
*A:* Checking every interior `'O'` with its own search re-traverses shared connected regions once per cell inside them. Searching once, outward from the border, computes the same reachability information in a single O(mn) pass instead.

**Q4. State Pacific Atlantic Water Flow's real flow rule, and the exact comparison used when the search is reversed.**
*A:* Real flow requires `height(neighbor) ≤ height(current)`. The reversed search, starting from the ocean, requires `height(neighbor) ≥ height(current)` — the same edge, read backward.

**Q5. Prove the reversal is correct, don't just state it.**
*A:* `height(B) ≤ height(A)` and `height(A) ≥ height(B)` are the same inequality. A reversed path found by expanding on `≥` from the ocean outward, read backward, is therefore a genuine forward flow path satisfying the original `≤` rule — the reversal isn't an approximation, it's a logically identical restatement of the same edge condition.

**Q6. Given a cell whose only neighbors are all strictly higher, what can you conclude?**
*A:* It cannot flow anywhere except an ocean it's already directly bordering — it's a local low point with no valid outgoing step, since flow requires non-increasing height.

**Q7. What's the shared root technique connecting Surrounded Regions and Pacific Atlantic Water Flow?**
*A:* Both flip "search from every interior point toward a small target set" into "search from the small target set outward" — turning an O((mn)²) worst case into O(mn) by eliminating redundant re-traversal of shared regions.

**Q8. Is the Gateway already built (Day 66) an L4 or L7 load balancer, and how do you know?**
*A:* L7 — it routes based on the HTTP path and validates a JWT from the request header, both of which require reading the actual HTTP request content, something an L4 balancer (IP/port only) never sees.

**Q9. What's the mechanical difference between a health check and a circuit breaker, given both stop traffic to a failing dependency?**
*A:* A health check is the load balancer actively polling a dedicated endpoint on a schedule, external to the request path. A circuit breaker (Day 64) watches the outcomes of real calls as they happen and trips based on an observed failure rate — no separate polling endpoint involved.

**Q10. Why does HTTPS not change how HTTP itself works?**
*A:* TLS is layered underneath HTTP, encrypting the connection before any HTTP request/response data is sent — the request/response model, methods, and status codes are unchanged; only the transport is now encrypted.

---

## Day 73 — Graphs Capstone: Word Ladder and Floyd-Warshall

**Q1. What makes Word Ladder's graph "implicit," and why generate neighbors on the fly instead of precomputing them?**
*A:* No adjacency list exists ahead of time — nodes are words, edges are one-letter differences, generated per word via character substitution. Generating on demand only does work proportional to what BFS actually explores, rather than paying to build every edge up front.

**Q2. Is Word Ladder multi-source or single-source, and why does that distinction matter here?**
*A:* Single-source — there's exactly one `beginWord`. It matters because it correctly separates this problem from Days 70 and 72's multi-source techniques, which solve a structurally different kind of question (multiple simultaneous starting points, not one).

**Q3. Why is BFS specifically required for Word Ladder, not just convenient?**
*A:* The problem asks for the *shortest* transformation sequence — a true minimum-steps question. BFS's level-by-level exploration guarantees the first time the target is dequeued, that's the minimum step count; DFS gives no such guarantee.

**Q4. State the Floyd-Warshall recurrence and explain each term.**
*A:* `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])` — either the current best path from `i` to `j` stays unchanged, or a path routing through intermediate node `k` is shorter, checked incrementally as `k` ranges over every node.

**Q5. Why must `k` be the outermost loop in Floyd-Warshall?**
*A:* Correctness requires every pair `(i,j)` to be fully finalized for intermediate set `{0,...,k-1}` before any pair starts using `k` as a bridge. Only making `k` outermost guarantees that full-layer-before-next-layer ordering; `i` and `j`'s relative order doesn't matter.

**Q6. Why is it safe to update the Floyd-Warshall distance matrix in place, using a single 2D array rather than a fresh copy per `k`?**
*A:* `dist[i][k]` and `dist[k][j]` can't be improved by allowing `k` itself as an intermediate — that would require revisiting `k`, which never shortens a shortest path. So reading those values mid-pass is always safe, even if they were "touched" earlier in that same pass.

**Q7. Why is Floyd-Warshall not classified as a graph traversal, despite operating on a graph?**
*A:* There's no queue, stack, or `visited` array — it builds an answer table from smaller already-solved subproblems and reuses them, the same DP-flavored shape previewed back in Week 3's Kadane's Algorithm, not a BFS/DFS descendant.

**Q8. When would you prefer Floyd-Warshall over running a single-source shortest-path algorithm multiple times?**
*A:* When you need shortest paths between every pair of nodes and the node count is small — Floyd-Warshall computes the full all-pairs table in one `O(V³)` pass, versus `V` separate single-source runs.

**Q9. In Find the City With the Smallest Number of Neighbors at a Threshold Distance, why does the tie-break comparison use `<=` rather than `<`?**
*A:* The rule requires returning the largest-indexed city among ties. Iterating indices in increasing order and using `<=` means a later equal count always overwrites the stored answer, correctly landing on the largest index.

**Q10. Classify this week's five DSA-block Graphs problems by what each one actually needed — not just "graph."**
*A:* Course Schedule/II: ordering and cycle-detection. Surrounded Regions and Pacific Atlantic: pure reachability (BFS or DFS interchangeably). Word Ladder: BFS specifically, for a true shortest-path question. Find the City: neither — a distinct, non-traversal algorithm family (Floyd-Warshall).

---

## Day 74 — Union-Find Begins, and AWS Networking Fundamentals

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

## Day 75 — Union-Find: Real-World Constraint Problems, and AWS IAM

**Q1. Why must every `==` equation be processed before any `!=` equation is checked, in Satisfiability of Equality Equations?**
*A:* Equality is transitive, and a `!=` violation can come from a transitive chain that isn't visible in any single equation. Checking against a partially built union structure risks missing exactly that kind of chained contradiction.

**Q2. In Most Stones Removed with Same Row or Column, why union rows and columns as nodes instead of comparing stone pairs directly?**
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

**Q10. What's the shared structural idea connecting Satisfiability of Equality Equations and Most Stones Removed, despite looking unrelated on the surface?**
*A:* Both reduce to "count connected components, then derive the answer from that count" — Satisfiability via a direct same-root contradiction check, Most Stones Removed via a size-minus-components removal formula proven by a spanning-tree argument.

---

## Day 76 — Union-Find: String Grouping, and Observability with Prometheus

**Q1. Prove that a connected component of swappable indices admits any permutation, not just state it.**
*A:* For any two positions in the component, a path of allowed swaps connects them; performing that path's swaps in sequence moves a character all the way from one end to the other. Repeating this, one character at a time, realizes any target arrangement — connectivity through allowed swaps is what guarantees full rearrangement.

**Q2. Why does Smallest String With Swaps sort characters within each component independently, rather than sorting the whole string at once?**
*A:* Only characters within the same connected component can ever reach each other's positions — characters in different components are never interchangeable, so each component's achievable arrangements are entirely independent of every other component's.

**Q3. Why does Accounts Merge union emails, not account entries?**
*A:* Two accounts might share no email directly but both share one with a third account, meaning all three belong to the same person transitively. Unioning emails as the fundamental unit captures that chain automatically; unioning accounts pairwise would require checking every pair's overlap explicitly.

**Q4. Why is name never used to determine which accounts merge?**
*A:* The problem defines identity purely by shared email — two different people can share a name, and merging on name would incorrectly combine them. Name is only used to label the final merged group, never to decide connectivity.

**Q5. What does "pull-based" mean for Prometheus, precisely?**
*A:* Prometheus itself initiates an HTTP request against each monitored service's metrics endpoint on a fixed schedule, rather than services pushing their own metric updates outward to a central collector.

**Q6. Name a genuine advantage of push-based metrics collection over pull, and Prometheus's own answer to that gap.**
*A:* A short-lived batch job may not exist long enough to be caught by a periodic scrape. Prometheus's Pushgateway lets such jobs push their final metrics to an intermediary, which Prometheus then scrapes from normally.

**Q7. Is Micrometer a monitoring system? If not, what is it?**
*A:* No — it's a vendor-neutral instrumentation API. It doesn't store, query, or visualize metrics itself; it lets application code emit counters/gauges/timers once and plug in different backend registries without changing that instrumentation code.

**Q8. What format does `/actuator/prometheus` expose metrics in, and why does that matter?**
*A:* Prometheus's plain-text exposition format — one line per metric series with labels and a value. It matters because it's exactly what Prometheus's scraper expects; no translation layer is needed between the application and the metrics database.

**Q9. Why tag metrics with the application/module name in a multi-module platform?**
*A:* Once multiple modules all expose `/actuator/prometheus` and get scraped into the same Prometheus instance, the tag is what distinguishes which module a given metric series came from — without it, identical metric names from different modules would be indistinguishable.

**Q10. What's the shared reasoning connecting Smallest String With Swaps and Accounts Merge?**
*A:* Both union an abstraction chosen to make connectivity visible — index positions (not raw characters) in one, emails (not account entries) in the other — and both then group by resulting root to derive the final answer.

---

## Day 77 — Union-Find Capstone, Minimum Spanning Trees, and Grafana

**Q1. Why does Graph Valid Tree's algorithm skip a separate connectivity check?**
*A:* For `n` vertices, any two of {connected, acyclic, exactly `n-1` edges} force the third. The algorithm already confirms `n-1` edges (step 1) and acyclicity (step 2, no cycle found), which together algebraically force connectivity — a forest with `c` components has `n-c` edges, and `n-c=n-1` forces `c=1`.

**Q2. State the Cut Property.**
*A:* For any partition of a graph's vertices into two non-empty sets, the minimum-weight edge crossing that partition belongs to some minimum spanning tree.

**Q3. Prove the Cut Property using an exchange argument.**
*A:* If the minimum crossing edge `e` isn't in some MST `T`, adding `e` to `T` creates a cycle that must cross the cut an even number of times — so at least one other edge `e'` in that cycle also crosses it, with `weight(e) ≤ weight(e')` by `e`'s minimality. Swapping `e'` for `e` yields a spanning tree no heavier than `T`, containing `e` — so some MST contains `e`.

**Q4. How does Kruskal's algorithm reuse Graph Valid Tree's exact logic?**
*A:* "Would this edge connect two nodes already in the same Union-Find set" is the identical cycle-check Graph Valid Tree performs — Kruskal's calls it live during construction as a filter, rather than as a final pass/fail check.

**Q5. When would you prefer Prim's over Kruskal's?**
*A:* On dense graphs, where an O(V²) array-scan (no heap needed) beats Kruskal's O(E log E) sort — or when the algorithm needs to grow outward from a specific starting node for other structural reasons.

**Q6. Why does Prim's need to check whether a popped node is already in the MST, rather than trusting every popped heap entry?**
*A:* The same node can be pushed multiple times at different candidate weights as the MST grows; once it's actually added, any remaining stale, higher-weight entries for it in the heap must be discarded when popped, or they'd be incorrectly reprocessed.

**Q7. What does Grafana add that Prometheus alone doesn't provide?**
*A:* Prometheus stores and can be queried for metrics, but doesn't visualize them — Grafana queries a data source like Prometheus (via PromQL) and renders dashboards/panels on top of it; Grafana itself stores no metrics.

**Q8. Why does `rate(http_server_requests_seconds_count[5m])` give a requests-per-second figure rather than a raw count?**
*A:* The underlying metric is a cumulative counter; `rate()` computes the per-second average rate of increase over the trailing window specified (5 minutes here), converting an ever-growing total into a meaningful current throughput figure.

**Q9. What closed this week, and what's still open going into Week 12?**
*A:* Graphs (12/12, closed Day 73) and Union-Find (7/7, closed Day 77) are both fully closed — the two thinnest gaps the original plan audit identified. Dijkstra's and Dynamic Programming open next week, both previously unstarted.

**Q10. Why does the plan's own "150" total-problems figure not match the curriculum map's "151," and why does the map use 151?**
*A:* The gap traces back to a one-problem undercount first introduced in Week 9's own Day 63 scorecard, which appears to have been carried forward uncorrected rather than recomputed — confirmed by Week 12's own header being exactly `150+13`, a stable but uncorrected increment. The map uses its own independently-verified arithmetic as the authoritative figure, consistent with how it already handled the same situation at Week 9.

---

## Quick-Reference Index by Topic

- **Topological Sort / cycle detection:** Day 71, Q1–Q7
- **Multi-source traversal (reachability vs. distance):** Day 72, Q1–Q7
- **BFS on implicit graphs / Floyd-Warshall:** Day 73, all
- **Union-Find mechanics and complexity proofs:** Day 74, Q1–Q8
- **Union-Find applied — equations, components, redundant resources:** Day 75, all
- **Union-Find applied — string/index rearrangement, identity merging:** Day 76, Q1–Q4, Q10
- **Minimum Spanning Trees (Kruskal's, Prim's, Cut Property):** Day 77, Q1–Q6
- **Networking (TCP/UDP/DNS/HTTP/Load Balancing):** Day 71 Q8–Q10, Day 72 Q8–Q10
- **AWS (VPC/IAM/S3/EBS):** Day 74 Q9–Q10, Day 75 Q6–Q9
- **Observability (Prometheus/Micrometer/Grafana):** Day 76 Q5–Q9, Day 77 Q7–Q8
- **Week-level accounting / self-audit:** Day 77 Q9–Q10
