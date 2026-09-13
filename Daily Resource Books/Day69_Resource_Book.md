# Day 69 Resource Book — Graph Traversal Continues, and Kafka Schema Registry

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 68 Resource Book](Day68_Resource_Book.md)
**Next ▶:** [Day 70 Resource Book](Day70_Resource_Book.md)
**Companion to:** Day 69 of `Week_10_Revised.md`

---

## Recap

Yesterday's Concept Card established *why* graphs need an explicit `visited` structure, and both problems used it in its simplest form — permanently marking a cell so it's never reconsidered. Today's first problem, Clone Graph, is the first one this series has posed where a **cycle is not just theoretically possible but is exactly the thing the algorithm has to survive** — a graph with a cycle, traversed the naive way, would recurse forever. Today's second problem introduces **coloring** — a new use of `visited` that stores more than a boolean, and directly sets up an extra-practice problem that extends the same idea one step further.

Graphs is no longer opening fresh today, so — per this series' precedent (Monotonic Stack's extras landing Day 41 once past its own Day 40 opening; Heaps' extras landing Days 55–56 once past Day 54) — **one piece of extra practice appears today**, chosen and checked against `Week_11_Revised.md` before being added: **Find Eventual Safe States (LC 802)**, which does not appear anywhere in next week's required list. It's placed as clearly-marked extension material, since today's DSA block already carries two required Mediums plus a full Theory and Project block.

---

## Learning Objectives

By the end of today, without notes:

1. Solve Clone Graph, explaining precisely why a plain recursive DFS with no bookkeeping would infinite-loop on this exact input shape, and how a `HashMap<Node,Node>` fixes it.
2. Solve Is Graph Bipartite?, explaining 2-coloring as a direct extension of yesterday's `visited` idea — storing a color, not just a boolean.
3. (Extension) Explain how Find Eventual Safe States' 3-state coloring generalizes today's 2-coloring to detect cycles in a *directed* graph.
4. Explain why an unversioned event schema breaks silently across independently-evolving services, and what Schema Registry does about it.

---

## Concept Dependency Map for Today

```
Day 68 — visited set (boolean, permanent marking)
Day 4/5 — HashMap (key→value lookup)
        │
        ▼
Today, Problem 1 — Clone Graph (LC 133)
  visited becomes HashMap<Node,Node> — not just "seen," but "seen,
  AND here is its already-built clone" — survives a cycle by
  returning the in-progress clone instead of recursing again

Day 68 — visited set, generalized to carry information
        │
        ▼
Today, Problem 2 — Is Graph Bipartite? (LC 785)
  visited becomes a 2-value COLOR map — BFS/DFS assigns alternating
  colors; any edge connecting two same-colored nodes is a contradiction

        │  (EXTENSION — not required)
        ▼
Find Eventual Safe States (LC 802)
  Color generalizes from 2 states to 3 (unvisited/visiting/safe) —
  DIRECTED-graph cycle detection, not undirected 2-partitioning

Day 48 — Kafka Fundamentals (topics, partitions)
Day 69 (today) — Order entity, JPA
        │
        ▼
Kafka Schema Registry (NEW) — versioned message contracts between
  independently-evolving services
```

---

# Part 1 — Clone Graph (LeetCode 133, Medium) — Pattern: Graph DFS + HashMap

**Statement:** Given a reference to a node in a connected undirected graph, return a deep copy (clone) of the graph. Each node has a value and a list of neighbors.

## Why a plain DFS, with no bookkeeping, genuinely infinite-loops here

This is worth working through concretely, not just asserted, because it's the clearest possible demonstration of yesterday's Concept Card claim. Picture even the smallest possible cycle: node `1` connected to node `2`, and node `2` connected back to node `1`. A naive `clone(node)` that recurses into every neighbor unconditionally would clone `1`, then recurse to clone its neighbor `2`, which would recurse to clone *its* neighbor `1` — which recurses to clone `2` — forever, with no base case ever reached, because nothing is tracking "I've already started cloning this node." This is not a performance concern, the way an unnecessary `O(n²)` might be — it's a genuine non-terminating program, exactly the failure mode Day 68's Concept Card warned about in the abstract, now concrete.

### Approach — `HashMap<Node, Node>`: original → its clone

```java
public static Node cloneGraph(Node node) {
    if (node == null) {
        return null;
    }
    return dfs(node, new HashMap<>());
}

private static Node dfs(Node node, Map<Node, Node> visited) {
    if (visited.containsKey(node)) {
        return visited.get(node);   // ALREADY being (or already fully) cloned — return it, don't recurse again
    }
    Node clone = new Node(node.val);
    visited.put(node, clone);       // record BEFORE recursing into neighbors — this is what breaks the cycle
    for (Node neighbor : node.neighbors) {
        clone.neighbors.add(dfs(neighbor, visited));
    }
    return clone;
}
```

**Why the map must be populated *before* recursing into neighbors, not after:** if `visited.put(node, clone)` happened *after* the neighbor loop instead of before it, a cycle would still infinite-loop — by the time a cyclic neighbor path leads back to `node`, the map still wouldn't contain it yet, since the original call hasn't returned to reach its own `put` statement. Populating the map immediately after creating the clone, *before* touching any neighbor, is what guarantees a cyclic path back to an in-progress node finds it already mapped and returns the existing (possibly still-incomplete) clone reference instead of recursing again.

**Why this correctly produces a fully-connected clone despite returning an "incomplete" clone mid-recursion:** the `clone` object returned on a cycle-hit is the *same object* that the original call is still in the process of populating `neighbors` for. Java objects are reference types (Day 2) — returning that reference and later mutating its `neighbors` list from the original, still-in-progress call is exactly the same "several references pointing at one shared, mutable heap object" mechanism from Day 9, applied here on purpose rather than being a bug to avoid.

### Trace: a 3-node cycle, `1—2—3—1` (a triangle)

```
dfs(1, {}):
  not in map → clone1 = Node(1); map={1:clone1}
  neighbor 2: dfs(2, map)
    not in map → clone2 = Node(2); map={1:clone1, 2:clone2}
    neighbor 1: dfs(1, map) → 1 IS in map → return clone1  (cycle survived — no further recursion)
    neighbor 3: dfs(3, map)
      not in map → clone3 = Node(3); map={1:clone1,2:clone2,3:clone3}
      neighbor 1: dfs(1,map) → return clone1
      neighbor 2: dfs(2,map) → return clone2
      clone3.neighbors = [clone1, clone2]; return clone3
    clone2.neighbors = [clone1, clone3]; return clone2
  neighbor 3: dfs(3, map) → 3 IS in map → return clone3
  clone1.neighbors = [clone2, clone3]; return clone1
```

**Confirmed by direct execution:** running exactly this algorithm on a 3-node triangle produces a clone whose adjacency (by value) exactly matches the original's — `{1:[2,3], 2:[1,3], 3:[1,2]}` on both sides — while every cloned node is a genuinely distinct object from its original, confirming both correctness (same structure) and depth (actual copies, not shared references) simultaneously.

### Complexity

**Time: O(V + E)** — every node is visited (and cloned) exactly once, courtesy of the map check; every edge is examined exactly once as each node's neighbor list is processed. **Space: O(V)** for the map (one entry per node) plus O(V) worst-case recursion depth.

### Common Mistakes and Edge Cases

- ⚠️ **Populating the map after the neighbor loop instead of before** — proven above to still infinite-loop on any cycle, since the whole point is catching an in-progress node, not just a fully-finished one.
- ⚠️ **Comparing nodes by value instead of by reference for the map key** — `HashMap<Node,Node>` here relies on default reference-based `equals()`/`hashCode()` (no custom override), which is correct and required, since two *different* original nodes could coincidentally share the same `val`; keying by value would incorrectly merge them.
- ⚠️ **Forgetting the `node == null` guard** — LeetCode's own specification calls for `null` input to return `null`, a trivial but easy-to-skip edge case.
- Edge case: a single node with no neighbors → clones just that one node, empty neighbor list, no recursion beyond the base call.
- Edge case: the whole graph is one large cycle → handled correctly by the same mechanism as the 3-node trace above, regardless of cycle length.

> 💡 **Interview Insight:** This problem is a strong vehicle for demonstrating you understand *why* the map-before-recursing ordering matters, not just that it's the "correct" order to copy from a template. Walking through what specifically breaks with the reversed order — unprompted — is a much stronger signal than silently producing working code.

---

# Part 2 — Is Graph Bipartite? (LeetCode 785, Medium) — Pattern: Graph 2-Coloring

**Statement:** Given an undirected graph, determine whether its nodes can be split into two groups such that every edge connects a node in one group to a node in the *other* group (equivalently: can every node be colored one of two colors such that no edge connects two same-colored nodes).

## `visited` generalizes from "have I seen this" to "what did I assign it"

Yesterday's `visited` answered one question: has this cell been claimed. Today's needs to answer a second question at the same time: **what color was this node assigned** — because checking bipartiteness means comparing a *new* node's would-be color against colors already assigned to its neighbors, not just checking whether it's new.

### Approach — BFS, assign alternating colors, detect a same-color edge

```java
public static boolean isBipartite(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n];           // 0 = uncolored, 1 = color A, -1 = color B
    for (int start = 0; start < n; start++) {
        if (color[start] != 0) {
            continue;   // already colored via an earlier component's BFS
        }
        color[start] = 1;
        Queue<Integer> queue = new ArrayDeque<>();
        queue.offer(start);
        while (!queue.isEmpty()) {
            int node = queue.poll();
            for (int neighbor : graph[node]) {
                if (color[neighbor] == 0) {
                    color[neighbor] = -color[node];   // opposite color from the node we came from
                    queue.offer(neighbor);
                } else if (color[neighbor] == color[node]) {
                    return false;   // contradiction: an edge connects two SAME-colored nodes
                }
            }
        }
    }
    return true;   // every node colored with no contradiction found anywhere
}
```

**Why `-color[node]` is a clean way to flip between exactly two colors:** using `1` and `-1` (rather than, say, `0`/`1`) makes "the opposite color" a single negation, and `0` remains free to unambiguously mean "not yet colored" — a small but genuinely useful encoding choice, not just a stylistic preference, since it collapses "check current color, assign the other one" into one expression.

**Why the outer `for` loop over every `start` node is necessary, not defensive extra code:** the graph isn't guaranteed to be connected — there can be multiple disconnected components, each needing its own independent BFS, since a single BFS from one starting node only ever reaches nodes in *that* node's connected component. This is the identical "the graph might not be one connected piece" consideration Day 68's Number of Islands handled with its own outer double-loop — here expressed over graph nodes instead of grid cells.

### Trace: proving both a bipartite and a non-bipartite case

**Bipartite:** `graph = [[1,3],[0,2],[1,3],[0,2]]` (a 4-cycle: 0-1-2-3-0).
`color[0]=1` → BFS: neighbor 1 uncolored → `color[1]=-1`; neighbor 3 uncolored → `color[3]=-1`. Process node 1: neighbor 0 already colored, `color[0]=1 ≠ color[1]=-1` → fine; neighbor 2 uncolored → `color[2]=1`. Process node 3: neighbor 0 already colored, `1≠-1` → fine; neighbor 2 already colored, `color[2]=1 = -color[3]=1`? — check: `color[2]=1`, `color[3]=-1`, is `color[2]==color[3]`? `1 == -1`? No → fine. Process node 2: neighbors 1 (`-1≠1`, fine) and 3 (`-1≠1`, fine). No contradiction anywhere → **returns `true`**, matching the verified correct result.

**Not bipartite:** `graph = [[1,2,3],[0,2],[0,1,3],[0,2]]` (adds an odd-length cycle: 0-1-2-0 is a *triangle*, length 3). `color[0]=1`. Process 0: neighbor 1 uncolored → `color[1]=-1`; neighbor 2 uncolored → `color[2]=-1`; neighbor 3 uncolored → `color[3]=-1`. Process 1: neighbor 0 (`1≠-1`, fine); neighbor 2 — **already colored `-1`**, and `color[1]` is also `-1` → `color[2]==color[1]` → **contradiction, returns `false`** — correctly, since a triangle (any odd-length cycle) can never be validly 2-colored: node 0 and node 2 are directly connected, and BFS/DFS forced them to the same color via node 1, but no consistent 2-coloring of an odd cycle exists at all. Confirmed by direct execution against exactly this input.

### Complexity

**Time: O(V + E), Space: O(V)** — identical shape and reasoning to any BFS traversal (Day 68's Concept Card): every node enqueued once, every edge examined once from each endpoint.

### Common Mistakes and Edge Cases

- ⚠️ **Forgetting the outer loop over disconnected components** — a graph that's actually two separate bipartite pieces, checked only from one starting node, would silently leave the second piece's nodes at `color=0`, never verified at all.
- ⚠️ **Checking `color[neighbor] != color[node]` as the ONLY condition for success**, without the `== 0` branch first — conflates "uncolored" (`0`) with "wrongly colored," since `0` would never legitimately equal `1` or `-1`, but the *reason* to assign a fresh color versus check an existing one are different code paths that must not be merged.
- Edge case: a graph with no edges at all → trivially bipartite (every node its own component, colored arbitrarily, no edge ever exists to create a contradiction).
- Edge case: any odd-length cycle anywhere in the graph → always non-bipartite; this is actually the precise mathematical characterization of bipartiteness (a graph is bipartite if and only if it contains no odd-length cycle) — worth stating if asked for the underlying theory, not just "run the coloring check."

> 💡 **Interview Insight:** Stating the odd-cycle characterization explicitly ("bipartite iff no odd cycle") — even though the coloring algorithm above doesn't need to check for cycles directly to get the right answer — signals a level of graph theory fluency beyond "I can run BFS with a color array."

---

## Extension (Optional) — Find Eventual Safe States (LeetCode 802, Medium)

**Not required by this week's plan.** Included as extra practice — checked against `Week_11_Revised.md` and `00_Curriculum_Map.md`'s problem table; it appears nowhere in either. Skippable under time pressure; the two required problems above fully satisfy today's deliverables on their own.

**Statement:** In a **directed** graph, a node is "safe" if every possible path starting from it eventually leads to a terminal node (a node with no outgoing edges) — equivalently, no path from it ever runs into a cycle. Return all safe nodes.

## Why this is today's coloring idea, generalized from 2 states to 3

Is Graph Bipartite tracked two colors on an *undirected* graph to catch a same-color contradiction. This problem tracks **three** states on a **directed** graph, to catch something different: a cycle. The third state is the genuinely new idea — distinguishing "currently being explored, on the active call stack" from "fully explored and confirmed fine."

```java
public static List<Integer> eventualSafeNodes(int[][] graph) {
    int n = graph.length;
    int[] state = new int[n];    // 0 = unvisited, 1 = visiting (on current DFS path), 2 = safe
    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < n; i++) {
        if (dfs(graph, i, state)) {
            result.add(i);
        }
    }
    return result;
}

private static boolean dfs(int[][] graph, int node, int[] state) {
    if (state[node] != 0) {
        return state[node] == 2;   // already resolved — either confirmed safe, or currently mid-path (unsafe)
    }
    state[node] = 1;   // mark as "currently on this DFS's active path"
    for (int neighbor : graph[node]) {
        if (!dfs(graph, neighbor, state)) {
            return false;   // hit an unsafe neighbor (or a cycle back to a "visiting" node) — this node is unsafe too
        }
    }
    state[node] = 2;   // every neighbor resolved safely — THIS node is safe
    return true;
}
```

**Why hitting a node in state `1` (not just state `0` or `2`) is exactly how a cycle gets caught:** if DFS reaches a node whose `state` is `1`, that node is *currently* on the active recursion path that led here — meaning this path has looped back on itself. `state[node] == 2`'s check on that node correctly evaluates to `false` (since it's `1`, not `2`), propagating "unsafe" back up through every node on the cycle. This is the same "am I about to revisit something already in progress" question Clone Graph's map-before-recursing answered above, generalized from "prevents infinite recursion" to "detects and correctly reports the cycle as the reason for unsafety."

**Confirmed against the classic example:** `graph = [[1,2],[2,3],[5],[0],[5],[],[]]` → **`[2,4,5,6]`**, matching the known correct output exactly.

> 🔗 **Forward reference:** this exact 3-state coloring pattern (unvisited / in-progress / resolved) is the same shape behind cycle detection in directed graphs generally — including the topological-sort-via-DFS technique Week 11's Course Schedule problems build on. Recognizing it here first, as a variant of today's 2-coloring, is a head start on that connection next week.

---

# Part 3 — Kafka Schema Registry

## Prerequisites, confirmed

Day 48's Kafka fundamentals (topics, partitions, producers, consumers) and Day 36's JPA entities — today's project connects both directly, giving the same `Order` concept two distinct representations for two distinct purposes.

## The failure mode this exists to prevent

Day 48 established topics and partitions as the *pipe* messages flow through between independently-deployed services. It didn't address a separate, equally real question: what governs the **shape** of what flows through that pipe? Without an answer, here's the concrete failure: the Order module publishes an `OrderCreated` event as raw JSON. Some time later, a developer working on Order renames a field (`orderId` → `id`) or changes a type (a `String` price becomes a `BigDecimal`) — a change that compiles fine and passes Order's own tests, because Order's own tests don't know or care what any consumer expects. The Notification module, deployed independently and expecting the old shape, either throws a deserialization exception on every subsequent message, or — worse, and worse specifically *because* it's silent — successfully deserializes into subtly wrong data (a missing field silently defaulting to `null`, say), with no error at all, no alert, just wrong behavior discovered later, downstream, disconnected from its actual cause.

**This is a distributed-systems-specific version of a problem compile-time type checking solves for free inside one process:** two methods in the same codebase with a mismatched signature simply fail to compile. Two independently-deployed services communicating via events have no such shared compiler — nothing stops one from changing its message shape out from under the other, unless something is deliberately introduced to play that role.

## The mechanism: a central, versioned contract, checked before publishing

**Schema Registry** is a separate service storing every version of every message schema used across the system. Producers register their schema (or reference an already-registered version) before publishing; the registry enforces a configured **compatibility mode** — most commonly **backward compatibility**, meaning a *new* schema version must still be readable by code written against the *previous* version. A field rename or a type change that would break existing consumers is **rejected at publish time**, not discovered later as a runtime failure in a completely different service.

```json
// Order.avsc — the Avro schema, the actual CONTRACT other services rely on
{
  "type": "record",
  "name": "OrderCreated",
  "namespace": "com.platform.order.events",
  "fields": [
    { "name": "orderId", "type": "string" },
    { "name": "productId", "type": "string" },
    { "name": "quantity", "type": "int" },
    { "name": "totalPrice", "type": "double" },
    { "name": "createdAt", "type": "long", "logicalType": "timestamp-millis" }
  ]
}
```

**Why this is Avro (a binary, schema-defined format) rather than plain JSON, specifically:** JSON is self-describing and human-readable, but carries no enforced contract at all — any producer can emit any shape, and any consumer discovers a mismatch only by actually trying (and failing) to read a field that isn't there. Avro requires a schema to exist and be registered *before* a message can be serialized against it, which is what makes registry-enforced compatibility checking possible in the first place — there's a concrete schema for the registry to compare a new version against. As a secondary, genuinely real benefit: Avro's binary encoding is materially more compact on the wire than JSON's text representation, at real scale.

## Why the JPA entity and the Avro schema are deliberately two separate things

**This distinction is worth stating precisely, since collapsing it is a natural but real mistake:** Day 36's `Order` JPA entity governs how order data is *persisted* inside the Order module's own database — its shape is Order's own internal business, free to include implementation details no other service should ever depend on (internal audit fields, soft-delete flags, whatever Order's own persistence needs). The Avro schema governs the **event contract** other services depend on — deliberately a *subset*, and not necessarily even structurally identical, since a published event's shape is a promise made to every consumer, while the entity's shape is a private implementation detail of one module. Publishing the raw JPA entity directly as the event schema — tempting, since it's already right there — would accidentally leak internal persistence details into a cross-service contract, and would force every future internal refactor of the entity to consider whether it's also silently breaking every external consumer.

### When to reach for Schema Registry vs. plain JSON messages

| | Plain JSON, no registry | Avro + Schema Registry |
|---|---|---|
| Contract enforced before publishing? | No — any shape can be sent | Yes — incompatible changes rejected at publish time |
| Where does a breaking change surface? | At runtime, in some consumer, disconnected from its cause | At publish time, in the producer that caused it |
| Message size at scale | Larger (text) | Smaller (binary) |
| Best fit when... | A single team owns both producer and consumer, low risk of drift | Independently-deployed services, evolving on separate schedules — exactly this platform's shape |

### Common Mistakes

- ⚠️ **Publishing the JPA entity directly as the event payload** — conflates persistence shape with contract shape, discussed above; a purpose-built event schema is the correct separation.
- ⚠️ **Assuming JSON messages are "fine" because they're readable** — human-readability says nothing about whether a consumer's *code* correctly handles a shape it wasn't written to expect; readability and contract-safety are unrelated properties.
- ⚠️ **Treating any schema change as automatically safe as long as the registry doesn't complain** — the registry enforces the *configured* compatibility mode, not "every possible notion of safety"; a change that's backward-compatible by the registry's definition can still be a meaningful business-logic change worth its own review.

> 💡 **Interview Insight:** "What stops one service's schema change from silently breaking another service listening on the same topic?" is a natural systems-design follow-up once event-driven architecture comes up at all. Naming Schema Registry specifically, and the backward-compatibility-enforced-at-publish-time mechanism, is a stronger answer than a vaguer "you'd version your messages" — precisely because *how* that versioning gets enforced is the actual engineering content.

---

## Project Block

**Repository:** `scalable-ecommerce-platform`.

**Task:** define an `Order` JPA entity (if not already present from earlier platform work) and a separate `OrderCreated.avsc` Avro schema for the event the Order module publishes on order creation. Register the schema and publish a real event through it.

**Practical guidance:** deliberately make the Avro schema's field set a genuine subset of the entity's — this is the point being exercised today, not an accident to avoid. Confirm the registry actually rejects an incompatible change: after the first successful publish, try changing a field's type in the `.avsc` file and attempt to register the new version, observing the registry's rejection directly rather than assuming the compatibility check works.

**Definition of done:** an `OrderCreated` event, schema-registered and Avro-serialized, is successfully published and consumed — and a deliberately incompatible schema change is observed being rejected by the registry, confirmed firsthand rather than taken on faith.

---

## Career Block

**LinkedIn Post 15** — "Why we needed Schema Registry: what happens when microservices' message contracts silently drift." Concrete, specific, and tied to something genuinely built today — the same posting guidance established since Day 10.

**Networking:** 20 minutes reviewing System Design fundamentals relevant to upcoming Tier A interview loops — connecting today's Gateway/resilience/messaging work to the broader HLD narrative this platform is building toward.

---

# Day 69 — Interview Questions

---

**1. Concretely, why does a plain recursive `clone(node)` with no bookkeeping infinite-loop on a graph containing a cycle?**

*Answer:* With nothing tracking which nodes are already being cloned, a cycle causes the recursion to keep rediscovering nodes it's already visited and recursing into them again — for example, cloning node 1 recurses into neighbor 2, which recurses back into neighbor 1, which recurses into 2 again, with no base case ever reached. It's a genuine non-terminating program, not just an inefficient one.

---

**2. Why must `visited.put(node, clone)` happen before the neighbor loop, not after, in Clone Graph?**

*Answer:* If the map were populated only after processing all neighbors, a cyclic path leading back to a node still mid-recursion wouldn't find it in the map yet (since that original call hasn't reached its own `put` statement), and the cycle would still cause infinite recursion. Populating immediately after creating the clone — before touching any neighbor — is what guarantees a cyclic path finds the in-progress node already mapped.

---

**3. Why is `HashMap<Node,Node>` keyed by reference, not by value, correct here?**

*Answer:* Two genuinely different original nodes could coincidentally share the same value; keying by value would incorrectly treat them as the same node and merge their clones. Reference-based keys correctly distinguish every distinct node object regardless of value collisions.

---

**4. In Is Graph Bipartite, why must every disconnected component be checked independently, via an outer loop?**

*Answer:* A single BFS/DFS from one starting node only reaches nodes within that node's own connected component. A graph with multiple disconnected pieces would leave every node outside the first component's reach completely unchecked without an outer loop iterating over every node as a potential unstarted component.

---

**5. State the precise mathematical characterization of bipartiteness that the coloring algorithm is actually checking for.**

*Answer:* A graph is bipartite if and only if it contains no cycle of odd length. The 2-coloring algorithm doesn't explicitly search for cycles — it discovers the same fact indirectly, since an odd cycle is exactly what forces two directly-connected nodes into the same color during BFS/DFS coloring.

---

**6. (Extension) How does Find Eventual Safe States' third state differ in purpose from Is Graph Bipartite's two colors?**

*Answer:* Bipartite's two colors both represent "resolved, assigned a definite category" — the check is whether two adjacent resolved nodes contradict each other. Safe States' three states include one — "currently visiting, on the active DFS path" — that is deliberately *not yet resolved*; encountering a node in that specific state is what signals a cycle has been found, a distinction two states alone can't express.

---

**7. What's the concrete failure this platform would hit without Schema Registry, and where would it actually surface?**

*Answer:* If the Order module changed its published event's shape (a renamed field, a changed type) with no registry enforcing compatibility, the Notification module — deployed and evolving independently — would either throw deserialization errors or silently misread the new shape, discovered as a runtime failure inside Notification, disconnected from its actual cause inside Order.

---

**8. Why does this specifically need Avro rather than plain JSON messages?**

*Answer:* Avro requires a schema to exist and be registered before a message can be serialized against it, which is what makes registry-enforced compatibility checking possible at all — there's a concrete schema to compare a new version against. Plain JSON carries no such enforced contract; any shape can be published, and a mismatch is only discovered when a consumer actually fails to read an expected field.

---

**9. Why keep the JPA entity and the Avro event schema as two separate things, rather than publishing the entity directly?**

*Answer:* The JPA entity is a private implementation detail of the Order module's own persistence, free to include internal fields no other service should depend on. The Avro schema is a public contract other services rely on. Publishing the entity directly would leak internal persistence details into that contract and would force every future internal refactor to also consider whether it silently breaks external consumers.

---

**10. Does the registry rejecting an incompatible schema change guarantee the change is "safe" in every sense?**

*Answer:* No — it guarantees the change satisfies the specifically configured compatibility mode (commonly backward compatibility). A change can pass that check and still represent a meaningful business-logic change worth its own review; schema compatibility and business correctness are related but distinct concerns.

---

## Daily Deliverable Check

- [ ] Clone Graph (LC 133) solved — the map-before-recursing ordering justified, not just applied.
- [ ] Is Graph Bipartite? (LC 785) solved — both a bipartite and a non-bipartite case traced.
- [ ] (Extension, optional) Find Eventual Safe States (LC 802) — attempted if time allows; not required for today's core deliverables.
- [ ] All attempted problems pushed to `dsa-java/graphs/`.
- [ ] `OrderCreated.avsc` Avro schema registered and a real event published and consumed; an incompatible schema change deliberately attempted and observed being rejected.
- [ ] LinkedIn Post 15 published.

---

## What Tomorrow Assumes You Already Know Cold

Day 70 assumes BFS's queue-based mechanism (Day 68's Concept Card, reused directly today) is fully solid, since Rotting Oranges extends it into **multi-source BFS** — starting from several nodes simultaneously rather than one — building on today's traversal mechanics without re-deriving them. It also assumes today's Kafka Schema Registry work is stable, since tomorrow's Saga Choreography theory is built directly on top of the same event-publishing infrastructure exercised today.

**Next:** [Day 70 Resource Book](./Day70_Resource_Book.md) — Multi-Source BFS, DAGs, and Week 10 Consolidation.
