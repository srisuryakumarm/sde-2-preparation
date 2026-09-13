# Day 71 — Topological Sort, and Networking Fundamentals I (TCP/UDP/DNS)

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 70 Resource Book](Day70_Resource_Book.md)
**Next ▶:** [Day 72 Resource Book](Day72_Resource_Book.md)
**Companion to:** Day 71 of `Week_11_Revised.md`

---

## Recap

Week 10 closed with Graphs open at 6/12 required (+2 extra), and the Concept Card from Day 68 established three things that today leans on directly, without re-deriving any of them: **adjacency list vs. matrix** (list wins for the sparse graphs interviews favor), **`visited` is mandatory whenever cycles are possible** (a tree can't cycle, a graph can), and the **BFS-vs-DFS interview signal** (shortest/minimum/fewest → BFS, unconditionally). Day 69 went one step further and used **3-state coloring** (unvisited / visiting / safe) to detect cycles in a *directed* graph — Find Eventual Safe States. Today's new material is the direct escalation of exactly that idea: instead of just detecting whether a directed cycle exists, Topological Sort asks for a full linear ordering that's only possible when no cycle exists at all. Today closes the last 2 of Graphs' 12 required problems for real interview cycle-detection, and starts Networking — the first of two "how the internet actually works" theory days this week.

---

## Learning Objectives

By the end of today, without notes:

1. State the precise definition of a topological order, and explain why it's only defined for a DAG (Directed Acyclic Graph).
2. Implement Kahn's Algorithm from memory — in-degree array, seed queue, process-and-decrement loop — and explain why a leftover in-degree at the end means a cycle.
3. Explain the DFS-based (3-state coloring) alternative, and connect it explicitly to Day 69's Find Eventual Safe States rather than presenting it as new.
4. Solve Course Schedule and Course Schedule II, and explain precisely what changes between an *existence* check and a *construction* problem when the underlying algorithm is identical.
5. Explain the TCP three-way handshake, contrast it with UDP's guarantees (or lack of them), and walk through DNS resolution from a cold cache to a returned IP address.

---

## Concept Dependency Map

```
Day 4: Queue (ArrayDeque)          Day 68: Graphs — adjacency list,
        │                                  visited requirement, BFS/DFS signal
        │                                          │
        └──────────────┬───────────────────────────┘
                        ▼
              Day 69: 3-state coloring
              (unvisited / visiting / safe) —
              cycle detection in a DIRECTED graph
                        │
                        ▼
              TODAY: Topological Sort
              ├─ NEW: in-degree — Kahn's Algorithm (BFS-based)
              ├─ Alternative: DFS + 3-state coloring, reused
              │  from Day 69, reversed-finish-order
              ├─ Course Schedule (LC 207) — existence check
              └─ Course Schedule II (LC 210) — construction
                        │
                        ▼
              Graphs BFS/DFS: 8/12 → 10/12 required today
              (closes tomorrow and the day after)

Independent theory track:
Networking Fundamentals I — TCP/UDP, DNS
(no DSA prerequisite; first of a two-day arc, continues Day 72)
```

---

# Part 1 — Topological Sort

## Concept Card — Topological Sort

**Prerequisites, confirmed:** Graph BFS/adjacency-list construction (Day 68) ✅, `Queue` via `ArrayDeque` (Day 4) ✅, the 3-state coloring cycle-detection idea (Day 69) ✅.

**What it is:** a topological order is a linear ordering of a directed graph's vertices such that for every directed edge `u → v`, `u` appears **before** `v` in the ordering. It only exists for a **DAG** — a Directed Acyclic Graph. If the graph has a directed cycle, no valid ordering can exist, because somewhere in that cycle you'd need some node to come both before *and* after another node in the same cycle — a contradiction. This is why **topological sort and directed-cycle detection are the same question asked two ways**: "can I order this?" and "is this acyclic?" have identical answers.

**Why it matters:** any "what must happen before what" problem reduces to this — build systems (compile `B` before `A` if `A` depends on `B`), task scheduling with dependencies, and, today's example, course prerequisites.

**Interview signal:** "prerequisites," "dependencies," "build order," "can all tasks be completed" — and, less obviously, any problem that's secretly asking "does this directed graph have a cycle."

There are two genuinely distinct ways to compute a topological order. Both are covered in full, because the DFS version is not a footnote — it's the direct escalation of a technique you already built four days ago.

### Approach 1 — Kahn's Algorithm (BFS-based, in-degree tracking)

**The core idea:** a node is safe to place in the output the moment every one of its prerequisites has already been placed. "Every prerequisite already placed" is exactly what **in-degree zero** means, if in-degree is defined as "number of incoming edges not yet accounted for."

**Mechanism:**
1. Build the adjacency list, and alongside it, an `inDegree[]` array — for every directed edge `u → v`, increment `inDegree[v]` by 1 (v has one more prerequisite).
2. Seed a queue with every node whose `inDegree` is already 0 — these have no prerequisites and are immediately safe to place.
3. While the queue isn't empty: dequeue a node, append it to the result order. For each of its outgoing neighbors, decrement that neighbor's `inDegree` by 1 (one of its prerequisites has now been satisfied). If a neighbor's `inDegree` hits exactly 0, it just became safe — enqueue it.
4. When the queue empties, check the result order's length against the total node count. If they match, every node was placed — a valid order exists. If the result is **shorter**, some nodes never reached in-degree 0, which can only happen if they're stuck in a cycle (each waiting on another node in the same cycle, none of which can ever "go first").

```java
public static int[] topologicalSort(int numNodes, int[][] edges) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numNodes; i++) adj.add(new ArrayList<>());
    int[] inDegree = new int[numNodes];

    for (int[] edge : edges) {
        int u = edge[0], v = edge[1];       // u -> v
        adj.get(u).add(v);
        inDegree[v]++;
    }

    Queue<Integer> queue = new ArrayDeque<>();
    for (int i = 0; i < numNodes; i++) {
        if (inDegree[i] == 0) queue.offer(i);
    }

    int[] order = new int[numNodes];
    int index = 0;
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order[index++] = node;
        for (int neighbor : adj.get(node)) {
            inDegree[neighbor]--;
            if (inDegree[neighbor] == 0) queue.offer(neighbor);
        }
    }

    return index == numNodes ? order : new int[0];   // empty = cycle detected
}
```

**Why this can't skip a valid node:** every node with in-degree 0 gets enqueued exactly once, either at seed time or the instant its last prerequisite is processed. A node is never "passed over" — it either eventually reaches in-degree 0 (and gets queued) or it never does (because it's genuinely stuck in a cycle with something else). There's no third outcome.

### Worked Trace — Kahn's Algorithm

`numNodes = 4`, edges (as `prereq → course`): `0→1, 0→2, 1→3, 2→3` (a diamond: courses 1 and 2 both need 0; course 3 needs both 1 and 2).

Initial in-degree: `[0:0, 1:1, 2:1, 3:2]`. Seed queue: `[0]` (only node with in-degree 0).

| Step | Dequeue | Order so far | Neighbors processed | In-degree updates | Newly enqueued |
|---|---|---|---|---|---|
| 1 | 0 | `[0]` | 1, 2 | `1: 1→0`, `2: 1→0` | 1, 2 |
| 2 | 1 | `[0,1]` | 3 | `3: 2→1` | — (not yet 0) |
| 3 | 2 | `[0,1,2]` | 3 | `3: 1→0` | 3 |
| 4 | 3 | `[0,1,2,3]` | (none) | — | — |

Final order `[0,1,2,3]`, length 4 = `numNodes`. Valid — no cycle. Verify against the edges: 0 before 1 ✓, 0 before 2 ✓, 1 before 3 ✓, 2 before 3 ✓. Every edge constraint holds.

**Now the cycle case**, the minimal version: `numNodes = 2`, edges `1→0, 0→1` (course 0 needs 1, course 1 needs 0 — a direct mutual dependency). In-degree: `[0:1, 1:1]`. Seed queue: **empty** — nothing has in-degree 0, since each node's sole prerequisite is the other. The `while` loop never executes. `index` stays `0`, which is `≠ 2`. Cycle correctly detected, with no work done beyond building the in-degree array — the emptiness of the seed queue itself is the signal.

### Approach 2 — DFS with 3-State Coloring (the direct escalation of Day 69)

**This is not new material — it's Day 69's Find Eventual Safe States technique, generalized.** That problem used 3 states — unvisited, visiting, safe — to detect a cycle in a directed graph. Topological sort needs exactly the same three states, plus one addition: recording the finish order.

**Mechanism:** DFS from every unvisited node. Mark a node **GRAY** ("visiting") the instant you enter it — this means "currently on the active recursion path." Recurse into each neighbor:
- Neighbor is **GRAY** → you've looped back onto your own current path. That's a **back edge**, meaning a directed cycle. Report failure immediately.
- Neighbor is **WHITE** (unvisited) → recurse into it.
- Neighbor is **BLACK** ("done") → already fully explored from every angle; skip it, nothing new to learn.

After every neighbor of the current node has been fully processed, mark the current node **BLACK** and **prepend** it to the result (or push onto a stack and reverse at the very end).

**Why the order is reversed relative to finish time — proven, not asserted:** a node is marked BLACK only after *all* of its dependents-via-outgoing-edges have already been fully explored and are themselves BLACK. That means whichever node finishes DFS *last* among a `u → v` pair must be `u` — `v` (the thing `u` depends on) always finishes first, since DFS can't finish exploring `u` until it's finished exploring everything reachable from `u`, `v` included. So DFS finish order is the **reverse** of a valid topological order; prepending (or reversing a finish stack) undoes that flip.

```java
public static int[] topologicalSortDFS(int numNodes, int[][] edges) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numNodes; i++) adj.add(new ArrayList<>());
    for (int[] edge : edges) adj.get(edge[0]).add(edge[1]);

    int[] state = new int[numNodes];   // 0 = white, 1 = gray, 2 = black
    Deque<Integer> finishStack = new ArrayDeque<>();
    boolean[] hasCycle = {false};

    for (int i = 0; i < numNodes; i++) {
        if (state[i] == 0) {
            dfs(i, adj, state, finishStack, hasCycle);
        }
    }

    if (hasCycle[0]) return new int[0];

    int[] order = new int[numNodes];
    for (int i = 0; i < numNodes; i++) order[i] = finishStack.pop();
    return order;
}

private static void dfs(int node, List<List<Integer>> adj, int[] state,
                         Deque<Integer> finishStack, boolean[] hasCycle) {
    state[node] = 1;   // gray — visiting
    for (int neighbor : adj.get(node)) {
        if (state[neighbor] == 1) {
            hasCycle[0] = true;   // back edge — cycle
            return;
        }
        if (state[neighbor] == 0) {
            dfs(neighbor, adj, state, finishStack, hasCycle);
        }
        // black neighbor: already fully explored, nothing to do
    }
    state[node] = 2;   // black — done
    finishStack.push(node);
}
```

**⚠️ Common Mistake:** using a plain boolean `visited[]` (2 states) instead of 3. With only "visited / not visited," a node still on the *current path* looks identical to a node that was fully explored on a *different, already-finished* branch — both read as "visited." That collapses the exact distinction (currently-in-progress vs. genuinely-done) that makes cycle detection possible at all. This is precisely why Day 69 introduced the third state in the first place — Is Graph Bipartite's 2-coloring couldn't detect a *directed* cycle, only an undirected structural property.

### Which approach to reach for

Both are O(V+E) time and O(V+E) space, and both are worth knowing cold. Kahn's is generally preferred for **iterative** style (no recursion, no stack-overflow risk on deep graphs) and gives you cycle detection "for free" as an emptiness check. The DFS version is preferred when you're already deep in DFS-shaped thinking for the rest of a problem, or when you specifically need the 3-state coloring machinery anyway (as Day 69 already did). In an interview, stating both exist and picking one — with a one-sentence reason — is a stronger opening than jumping straight to code.

**🔗 Direct citation:** the state machine here — white/gray/black — is *identical in spirit* to Day 69's unvisited/visiting/safe. That problem asked "is this specific node eventually safe" (no cycle reachable from it); today asks "is there a cycle anywhere, and if not, what's a valid order" — same mechanism, broader question.

---

## Problem 7: Course Schedule (LeetCode 207, Medium) — Pattern: Topological Sort / Cycle Detection

**Statement:** `numCourses` courses, labeled `0` to `numCourses-1`. `prerequisites[i] = [a, b]` means you must take course `b` before course `a`. Return `true` if you can finish all courses, `false` otherwise.

**Approach — direct application of Kahn's Algorithm.** Build the graph with edges `b → a` for each `[a, b]` pair (note the direction: `b` must come first, so the edge points *from* the prerequisite *to* the course that needs it). Run Kahn's exactly as above. The answer is simply: did every node reach the queue?

```java
public static boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] inDegree = new int[numCourses];

    for (int[] p : prerequisites) {
        int course = p[0], prereq = p[1];
        adj.get(prereq).add(course);
        inDegree[course]++;
    }

    Queue<Integer> queue = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++) if (inDegree[i] == 0) queue.offer(i);

    int processed = 0;
    while (!queue.isEmpty()) {
        int course = queue.poll();
        processed++;
        for (int next : adj.get(course)) {
            if (--inDegree[next] == 0) queue.offer(next);
        }
    }

    return processed == numCourses;
}
```

**Brute force, for contrast:** for every course, DFS to see if it can reach itself (a cycle involving it) — O(V×(V+E)) in the worst case, since a fresh traversal runs per node. Kahn's does the whole job in one O(V+E) pass by tracking readiness incrementally instead of re-deriving reachability from scratch per node.

**Complexity:** Time O(V+E) where V = `numCourses`, E = `prerequisites.length`. Space O(V+E) for the adjacency list plus O(V) for in-degree and the queue.

**Edge cases:**
- `prerequisites` is empty → every in-degree is 0, every course enqueues immediately, trivially `true`.
- A course lists itself as its own prerequisite (`[a, a]`) → in-degree of `a` becomes ≥1 from itself but `a` can never be the thing that decrements it before being placed — it's a self-loop, a cycle of length 1, correctly caught.
- Duplicate prerequisite pairs → in-degree just gets incremented more than "necessary," which only delays (never prevents) that node reaching 0 once its *distinct* prerequisites are satisfied — doesn't break correctness, only means `inDegree[node]` no longer equals "number of distinct prerequisites," which is fine since the algorithm never assumed that.

**💡 Interview Insight:** say out loud, before coding: "this is topological sort in disguise — the real question is whether the prerequisite graph has a cycle." Naming that reframe first is the single highest-value sentence in this problem; it signals you recognized the pattern rather than pattern-matched a random graph algorithm.

---

## Problem 8: Course Schedule II (LeetCode 210, Medium) — Pattern: Topological Sort

**Statement:** Same setup as Course Schedule, but return an *actual valid ordering* of all courses, or an empty array if impossible.

**Approach:** identical to Course Schedule, except instead of only counting how many nodes got processed, record them in an array as they're dequeued — that array *is* the topological order.

```java
public static int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] inDegree = new int[numCourses];

    for (int[] p : prerequisites) {
        adj.get(p[1]).add(p[0]);
        inDegree[p[0]]++;
    }

    Queue<Integer> queue = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++) if (inDegree[i] == 0) queue.offer(i);

    int[] order = new int[numCourses];
    int index = 0;
    while (!queue.isEmpty()) {
        int course = queue.poll();
        order[index++] = course;
        for (int next : adj.get(course)) {
            if (--inDegree[next] == 0) queue.offer(next);
        }
    }

    return index == numCourses ? order : new int[0];
}
```

**🔑 Key Takeaway:** Course Schedule and Course Schedule II are the *same algorithm*, and the only difference is what you extract from the run — a boolean ("did every node get placed") vs. the actual placement list. This is worth stating explicitly if asked to solve II right after I: don't re-derive anything, just widen what you're recording.

**Worked trace:** reuse the diamond graph from the Concept Card's trace above (`0→1, 0→2, 1→3, 2→3`) — same mechanical walk, and the returned array is exactly the `order` column's final row: `[0,1,2,3]`.

**Complexity:** identical to Course Schedule — Time O(V+E), Space O(V+E).

**Edge cases:**
- Multiple valid orderings can exist (e.g., if two courses have no dependency relationship between them, either can legally come first) — the problem accepts *any* valid order, and Kahn's naturally returns one consistent, deterministic order based on queue insertion order, which is a valid answer even if it's not the *only* valid answer.
- A cycle anywhere → return the empty array, exactly mirroring Course Schedule's `false`.

**⚠️ Common Mistake:** trying to solve II by first calling I to check feasibility, then re-running a *second*, separate pass to build the order. This works but does double the graph-construction and traversal work for no benefit — the boolean and the order fall out of the *same single pass*, so there's never a reason to run Kahn's twice.

**💡 Interview Insight:** if asked "what if I need the order that also breaks ties by, say, course number" — that's a signal to swap the plain `Queue` for a `PriorityQueue<Integer>` (Day 17/26/54's structure), seeding and draining it the same way; every node still enters and leaves exactly once, so the complexity becomes O(E + V log V) instead of O(V+E), only because of the heap's log-factor operations.

---

**This closes Graphs at 10/12 required.** Two more required problems remain (Days 72–73), which will bring the pattern to its full 12/12 close.

**Note on extra practice:** no extra practice added for Topological Sort specifically today. Reasoning, stated explicitly per this series' own convention: Topological Sort is a sub-technique inside the broader Graphs pattern, which is already closing this week at a comprehensive 12 required + 2 extra = 14 distinct problems (Weeks 10–11 combined) — padding a single day's narrow sub-technique risks trading pacing realism for repetition with no corresponding gap, the same reasoning Day 73's Floyd-Warshall gets below and Day 63's Combinations got in Week 9. Course Schedule and Course Schedule II already cover the sub-technique's two genuine flavors — existence check and construction — so there's no thin spot left for a third rep to fill.

---

# Part 2 — Networking Fundamentals I: TCP, UDP, and DNS

### Prerequisites (confirmed)

None from the DSA track — this is an independent theory thread, the same kind of fresh-start topic Day 34's REST APIs or Day 43's Docker were. It stands alone and will be built on directly tomorrow (HTTP, Load Balancers) and again in three days (AWS Networking, Day 74) — today's vocabulary is the foundation all of that sits on.

## TCP: Connection-Oriented, Reliable, Ordered

**What it is:** TCP (Transmission Control Protocol) guarantees that data arrives, arrives *in order*, and arrives *exactly once* — no drops, no duplicates, no reordering, from the application's point of view. It does this at the cost of real overhead: a connection has to be established before any data moves, and every byte sent needs to be acknowledged.

**The Three-Way Handshake — mechanism, not just the name:**

1. **SYN** — client → server: "I'd like to connect. My starting sequence number is `X`."
2. **SYN-ACK** — server → client: "Acknowledged, expecting `X+1` next. My own starting sequence number is `Y`."
3. **ACK** — client → server: "Acknowledged, expecting `Y+1` next."

After step 3, both sides have confirmed they can both send *and* receive from each other — that's why it takes three steps, not two: a two-step handshake would only confirm one direction actually works.

**Why every byte gets acknowledged:** once connected, the sender tracks which sequence numbers the receiver has confirmed. If an acknowledgment doesn't arrive within a timeout, TCP assumes the data (or the ACK itself) was lost and **retransmits**. Out-of-order packets get buffered and reassembled using their sequence numbers before being handed to the application — the application never sees partial or scrambled data, only a clean, ordered byte stream.

**🔑 Key Takeaway:** all of TCP's guarantees — no loss, no duplication, correct order — come from exactly one mechanism: sequence numbers plus acknowledgment plus retransmission-on-timeout. There's no separate "ordering system" and "reliability system"; it's one bookkeeping scheme doing both jobs at once.

## UDP: Connectionless, Fire-and-Forget

**What it is:** UDP (User Datagram Protocol) skips the handshake entirely and skips every guarantee TCP makes. A UDP packet (datagram) is sent once; if it's lost, nobody retransmits it; if two packets arrive out of order, nobody reorders them for you. This sounds strictly worse until you notice what it buys back: **far lower overhead and lower latency**, since there's no connection setup, no acknowledgment bookkeeping, and no waiting for retransmission.

**When this trade actually wins:** applications that can tolerate occasional loss but can't tolerate the *latency* TCP's reliability machinery introduces — live video/audio streaming (a dropped frame is fine; a frame arriving late and out of order to be "corrected" is not), online gaming (stale position data is worse than missing data), and DNS queries themselves (small, latency-sensitive, and cheap to just retry from the application layer if one packet goes missing).

**⚠️ Common Mistake:** describing UDP as "TCP without the good parts." It's not that UDP is TCP-minus-features — it's a genuinely different trade-off, deliberately built for a category of application where TCP's guarantees cost more (in latency) than they're worth.

| | TCP | UDP |
|---|---|---|
| Connection | Handshake required | None — fire and forget |
| Reliability | Guaranteed delivery, retransmits on loss | No guarantee — lost packets stay lost |
| Ordering | Guaranteed | Not guaranteed |
| Overhead | Higher (headers, ACKs, handshake) | Lower |
| Typical use | HTTP, file transfer, anything needing correctness | Video/audio streaming, gaming, DNS queries |

## DNS: Resolving a Hostname to an IP Address

**The problem it solves:** computers route by IP address; humans use names. DNS is the hierarchy that maps one to the other.

**The resolution chain, step by step, assuming a cold cache** (nothing cached anywhere yet):

1. Your machine asks a **recursive resolver** (typically your ISP's, or a public one like `8.8.8.8`) to resolve, say, `www.example.com`.
2. The resolver asks a **root server**: "who handles `.com`?" The root server doesn't know the answer to the *original* question — it only knows which server to point to next.
3. The root server returns the address of a **TLD (top-level domain) server** for `.com`.
4. The resolver asks that TLD server: "who's authoritative for `example.com`?" It responds with the **authoritative nameserver** for that specific domain.
5. The resolver asks the authoritative nameserver directly for `www.example.com`'s address, and gets back the actual IP.
6. The resolver returns that IP to your machine, which can now open a TCP connection (typically, for HTTP/HTTPS) to it.

**🔑 Key Takeaway — caching happens at every layer, aggressively, and this is load-bearing:** the browser caches the result, the OS caches it, the resolver caches it — each respecting a **TTL** (time-to-live) the authoritative server specifies. Redoing this entire five-hop chain for every single request to every website would be far too slow; caching is what makes DNS's latency invisible in ordinary browsing.

**⚠️ Common Mistake:** describing DNS resolution as "the browser directly asks the target website's server for its own IP." That's backwards — the target server is exactly what's *unknown* at the start; the whole point of the hierarchy is that no single server needs to know every domain's IP, each layer only needs to know who to delegate to next.

### Practice — walk it through yourself

The plan's exercise here isn't code — it's writing out, in your own words, every step between typing a URL and the browser having a real IP to connect to. Do this as a numbered list from scratch (browser cache check → OS cache check → resolver → root → TLD → authoritative → answer cached at every layer → TCP handshake against the resolved IP → HTTP request over that connection, previewed for tomorrow). Doing this cold, without looking at the five-step list above, is the actual test of whether the mechanism is solid — not whether you can recognize it when shown.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`. No new feature work today — this slot is deliberately a review pass, not new construction.

**Task:** re-read the Gateway routing (Day 66), JWT validation (Day 67), and Token Bucket rate limiting (Day 68) code you built last week. Confirm all three are still passing their existing tests, and specifically verify: the Gateway correctly rejects a request with a missing or invalid JWT *before* it reaches any downstream module, and the rate limiter's `synchronized` guard (Day 68, citing directly back to Day 37's Counter-corruption demonstration) is still in place — this is a correctness requirement, not a nice-to-have, and it's exactly the kind of thing that's easy to accidentally remove while refactoring.

**Definition of done:** all three pieces confirmed clean and passing under their existing tests; any drift found is fixed before moving on, not just noted.

---

## Career Block Guide (1 hr)

**LinkedIn Post 15 — Topological Sort, via "which courses can you actually take."** A draft to adapt, not copy verbatim:

> Today's DSA topic doubled as a genuinely satisfying "oh, that's what this is for" moment: topological sort.
>
> The setup is exactly course prerequisites — some courses need others done first, and you want a valid order to take all of them in. Turns out the algorithm for finding that order and the algorithm for detecting "wait, these prerequisites are actually impossible to satisfy" (a circular dependency) are the same algorithm. You're not writing two solutions — you're asking one question two ways.
>
> The mechanism itself is clean: track how many unmet prerequisites each course has, start with whatever has zero, and each time you "take" a course, cross it off everyone else's list. If you ever run out of zero-prerequisite courses before every course is taken, something was circular.
>
> Same idea underneath build systems, task schedulers, and dependency resolvers generally — not just a LeetCode pattern.

Post it, then move to networking: identify 3 Tier B target companies known specifically for platform/infrastructure engineering roles — the Gateway/rate-limiting/observability work this series has been building fits that profile directly, worth having in mind while researching.

---

## Day 71 — Interview Questions

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

## Daily Deliverable Check

- [ ] Course Schedule and Course Schedule II solved, pushed to `dsa-java/graphs/` — Graphs ladder now at 10/12 required.
- [ ] Both Kahn's Algorithm and the DFS 3-state-coloring alternative can be explained from memory, including why the DFS order needs reversing.
- [ ] Can explain TCP vs. UDP and full DNS resolution without notes.
- [ ] Gateway, JWT, and rate-limiting code reviewed, confirmed clean and passing.
- [ ] LinkedIn Post 15 published. 3 Tier B infra-focused target companies identified.

---

## What Tomorrow Assumes You Already Know Cold

Day 72 continues Graphs with a new role for multi-source traversal — **reachability from a border**, not shortest distance — and it assumes today's DFS/BFS mechanics and the `visited`-is-conditional rule (Day 68/70) are fully reflexive, since neither of tomorrow's two problems re-derives basic traversal from scratch. It also assumes the Networking vocabulary from today (TCP, the client/server request model) is solid, since tomorrow's Load Balancer discussion builds HTTP directly on top of TCP without re-explaining the transport layer underneath it.

**Next:** [Day 72 Resource Book](Day72_Resource_Book.md) — Multi-Source BFS (Reversed), and Networking Fundamentals II.
