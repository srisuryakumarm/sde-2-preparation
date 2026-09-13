# Day 70 Resource Book — Multi-Source BFS, DAGs, and Week 10 Consolidation

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 69 Resource Book](Day69_Resource_Book.md)
**Next ▶:** Day 71 Resource Book (Week 11)
**Companion to:** Day 70 of `Week_10_Revised.md`

---

## Recap

Sunday, lighter hours by design. Before today's new material: the plan's own self-check — pick one Backtracking problem from this week (Combination Sum II or N-Queens are the two with the most independent reasoning to reconstruct) and solve it cold, no notes, before reading further. If either one needs the book open to get through, that's useful information now, not a problem to discover mid-interview later.

Today closes Graphs' opening week with two problems that each extend Day 68's Concept Card in a different direction: **multi-source BFS** (starting from many nodes at once, not one) and a **DAG-guaranteed DFS that deliberately skips the `visited` structure** Day 68 called mandatory — worth resolving precisely, not glossing over as a contradiction. **One extra practice problem** appears today (01 Matrix, LC 542), a direct multi-source-BFS sibling to Rotting Oranges, checked against `Week_11_Revised.md` and confirmed absent from next week's required list.

---

## Learning Objectives

By the end of today, without notes:

1. Solve Rotting Oranges, explaining multi-source BFS as a direct extension of Day 49's level-isolation trick, not a new mechanism.
2. Solve All Paths From Source to Target, and explain precisely why skipping the `visited` structure here does *not* contradict Day 68's "graphs need visited" rule.
3. (Extension) Recognize 01 Matrix as the same multi-source BFS shape as Rotting Oranges, computing distance instead of time.
4. Explain what Saga Choreography solves that a single distributed transaction can't, and why "choreography" (vs. "orchestration") means no central coordinator.
5. Reconstruct Week 10's full problem and concept inventory without checking the curriculum map first.

---

## Concept Dependency Map for Today

```
Day 49 — Tree BFS, queue.size() level-isolation trick (level = distance from ONE root)
Day 68 — Graph BFS (Concept Card), visited as mandatory
        │
        ▼
Today, Problem 1 — Rotting Oranges (LC 994)
  MULTI-source BFS — seed the queue with EVERY starting node at once;
  "minutes elapsed" IS Day 49's level counter, just from many roots

Day 68 — visited as mandatory (because graphs CAN cycle)
        │
        ▼
Today, Problem 2 — All Paths From Source to Target (LC 797)
  DFS on a GUARANTEED DAG — no visited needed, because Day 68's rule
  was conditioned on cycles being POSSIBLE, and a DAG rules them out

        │  (EXTENSION — not required)
        ▼
01 Matrix (LC 542) — Rotting Oranges' exact multi-source BFS shape,
  computing minimum distance instead of minimum time

Day 48 — Kafka Fundamentals      Day 69 — Kafka Schema Registry
        │                                       │
        ▼                                       ▼
Saga Choreography (NEW) — a multi-step business transaction spanning
  services, coordinated via events, with NO central coordinator
```

---

# Part 1 — Rotting Oranges (LeetCode 994, Medium) — Pattern: Multi-Source BFS

**Statement:** A grid contains `0` (empty), `1` (fresh orange), or `2` (rotten orange). Every minute, any fresh orange adjacent (4-directionally) to a rotten one becomes rotten. Return the minimum number of minutes until no fresh orange remains, or `-1` if that's impossible.

## The extension, stated precisely before writing any code

Day 49's tree BFS used `queue.size()` at the top of each `while` iteration to process one full "level" — all nodes at the current distance from the root — before moving to the next, and that level count directly *was* the distance from the root. Every BFS run in this series so far has started from exactly **one** node. Here, rot doesn't spread from one starting point — it spreads simultaneously from **every** rotten orange already on the grid at minute 0. The fix is almost embarrassingly small given how it sounds: **seed the queue with every initially-rotten cell before the first iteration, instead of just one start node** — everything else about the level-by-level mechanism is unchanged, and "minutes elapsed" falls directly out of it as the same level counter Day 49 already used, just now measuring simultaneous distance from *many* sources instead of one.

### Approach — Seed all rotten cells, BFS level-by-level, count fresh oranges remaining

```java
public static int orangesRotting(int[][] grid) {
    int rows = grid.length, cols = grid[0].length;
    Queue<int[]> queue = new ArrayDeque<>();
    int freshCount = 0;

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (grid[r][c] == 2) {
                queue.offer(new int[]{r, c});   // seed EVERY rotten orange — the multi-source part
            } else if (grid[r][c] == 1) {
                freshCount++;
            }
        }
    }

    if (freshCount == 0) return 0;   // nothing fresh to begin with — 0 minutes needed

    int minutes = 0;
    int[][] directions = {{1,0},{-1,0},{0,1},{0,-1}};

    while (!queue.isEmpty() && freshCount > 0) {
        int levelSize = queue.size();   // Day 49's exact level-isolation trick
        for (int i = 0; i < levelSize; i++) {
            int[] cell = queue.poll();
            for (int[] dir : directions) {
                int nr = cell[0] + dir[0], nc = cell[1] + dir[1];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;       // this fresh orange just rotted
                    freshCount--;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }
        minutes++;   // one full level processed = one minute elapsed
    }

    return freshCount == 0 ? minutes : -1;   // -1 if some fresh orange was never reachable at all
}
```

**Why `minutes++` sits outside the inner `for` loop, at the same indentation Day 49's `depth++` did:** the inner loop processes an entire level (everything currently in the queue at the top of this `while` iteration) before `minutes` advances — identical structurally to Day 49 processing one full tree level before incrementing depth, just seeded from many roots instead of one.

**Why `grid[nr][nc] = 2` doubles as both the "mark rotten" business logic and the visited marker:** this is the same overwrite-based marking Day 68 used, serving double duty — it's simultaneously the correct simulation of the orange actually rotting *and* what prevents that cell from being re-enqueued by a different rotten neighbor later in the same level (or a future level), which would otherwise both corrupt the count and risk reprocessing.

### Trace: a small grid with two separate rot sources

```
[[2,1,1],
 [1,1,0],
 [0,1,1]]
```

Seed: `(0,0)=2` → queue=[(0,0)]; `freshCount` = 6 (all the `1`s). `minutes=0`.

**Level 1** (`levelSize=1`): process `(0,0)`. Neighbors: `(1,0)=1`→rot, `freshCount=5`, enqueue; `(0,1)=1`→rot, `freshCount=4`, enqueue. `minutes` becomes `1`.

**Level 2** (`levelSize=2`, the two just enqueued): process `(1,0)`: neighbors `(2,0)=0`(skip, empty), `(1,1)=1`→rot, `freshCount=3`, enqueue. process `(0,1)`: neighbors `(0,2)=1`→rot, `freshCount=2`, enqueue; `(1,1)` already `2` now, skip. `minutes` becomes `2`.

**Level 3** (`levelSize=2`): process `(1,1)`: neighbor `(2,1)=1`→rot, `freshCount=1`, enqueue. process `(0,2)`: neighbor `(1,2)=0`(empty, skip). `minutes` becomes `3`.

**Level 4** (`levelSize=1`): process `(2,1)`: neighbor `(2,2)=1`→rot, `freshCount=0`, enqueue. `minutes` becomes `4`.

Loop condition `freshCount > 0` now false → exits. **Result: 4 minutes**, `freshCount==0` → return `4`, not `-1`. (`(2,0)` stays `0`/empty throughout — it was never fresh, correctly excluded from the count entirely.)

### Complexity

**Time: O(m × n)** — every cell is enqueued and processed at most once, identical amortized reasoning to Day 68. **Space: O(m × n)** worst case for the queue, if most of the grid starts rotten or fresh.

### Common Mistakes and Edge Cases

- ⚠️ **Seeding the queue with only the first rotten cell found**, instead of all of them — silently turns this back into single-source BFS, computing an answer as if rot only spread from one point, understating how fast rot actually spreads when multiple sources exist.
- ⚠️ **Incrementing `minutes` inside the inner loop** instead of after it — would count once per *cell* processed rather than once per *level*, producing a wildly inflated answer.
- ⚠️ **Forgetting the `-1` case entirely** — a fresh orange with no path to any rotten source (isolated by empty cells) must be detected via `freshCount` remaining `> 0` after the queue empties, not assumed away.
- Edge case: no fresh oranges at all → correctly `0`, checked explicitly before the main loop even starts.
- Edge case: a fresh orange completely walled off by empty cells → correctly `-1`, since it's simply never reached by any BFS level.

> 🔗 **Backward reference:** if Day 49's level-isolation trick (`queue.size()` at the top of the loop) needed re-explanation to follow the trace above, that's worth revisiting directly — today's entire "new" mechanism is that trick, applied to a queue seeded from many roots instead of one.

---

# Part 2 — All Paths From Source to Target (LeetCode 797, Medium) — Pattern: DFS on a DAG

**Statement:** Given a **directed acyclic graph** of `n` nodes labeled `0` to `n-1`, find every possible path from node `0` to node `n-1`, and return them (in any order).

## Resolving the apparent contradiction with Day 68, precisely

Day 68's Concept Card stated, correctly, that graphs need an explicit `visited` structure. This problem's standard, correct solution uses **no `visited` structure at all** — and both facts are true simultaneously, because Day 68's rule was never "always use visited," it was **"a graph can contain a cycle, and cycles are what make `visited` necessary."** This problem's statement *guarantees* the input is a **DAG** — acyclic, by definition, no exceptions. Without a cycle possible anywhere in the input, the specific failure mode `visited` exists to prevent (infinite re-exploration of an already-seen node) cannot occur here, regardless of whether `visited` is present or not. The general rule ("graphs need visited") was always conditioned on cycles being possible — this problem is the first case this series has posed where that condition is explicitly false, and the rule correctly doesn't apply.

**A second, easy-to-miss reason a *global* visited set would be actively wrong here, not just unnecessary:** the goal is enumerating **every** path from `0` to `n-1` — and a node can legitimately appear on *multiple different valid paths*. A global `visited` set (marking a node as "used" the first time any path reaches it, the way Day 68's Number of Islands did) would incorrectly prevent a second, equally valid path from ever revisiting that node — silently dropping correct answers, not just doing unnecessary defensive work.

### Approach — Plain DFS, build path forward, record on reaching the target

```java
public static List<List<Integer>> allPathsSourceTarget(int[][] graph) {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    path.add(0);
    dfs(graph, 0, path, result);
    return result;
}

private static void dfs(int[][] graph, int node, List<Integer> path, List<List<Integer>> result) {
    int target = graph.length - 1;
    if (node == target) {
        result.add(new ArrayList<>(path));   // reached the target — record this complete path
        return;
    }
    for (int neighbor : graph[node]) {
        path.add(neighbor);                    // choose
        dfs(graph, neighbor, path, result);     // explore
        path.remove(path.size() - 1);            // un-choose
    }
}
```

**This is worth naming as genuinely backtracking again, not plain graph DFS the way Day 68's flood-fill was:** the `path` here is one shared, mutable structure across the whole exploration, and the un-choose step (`path.remove`) is both present and necessary — a node explored as part of one path must be removed from `path` before a *sibling* path (a different neighbor at the same recursion depth) is explored, or that sibling path would incorrectly appear to include it. This is Day 61's exact choose/explore/un-choose shape, applied to graph edges instead of array elements — worth explicitly distinguishing from Day 68's Number of Islands, which shared the *recursive DFS* shape but was proven *not* to be backtracking for the opposite reason (permanent marking, nothing to restore). Two graph problems, two visually similar recursive traversals, two different correct classifications — precisely the kind of distinction worth being able to draw on demand.

### Trace: `graph = [[1,2],[3],[3],[]]`

```
dfs(node=0, path=[0])
  neighbor 1: path=[0,1] → dfs(node=1, path=[0,1])
                neighbor 3: path=[0,1,3] → dfs(node=3, path=[0,1,3])
                              node==target(3) → ADD [0,1,3] ✓
                path restored to [0,1], then to [0]
  neighbor 2: path=[0,2] → dfs(node=2, path=[0,2])
                neighbor 3: path=[0,2,3] → dfs(node=3, path=[0,2,3])
                              node==target(3) → ADD [0,2,3] ✓
                path restored to [0,2], then to [0]
```

**Result: `[[0,1,3],[0,2,3]]`** — matches the known correct output for this exact classic input.

### Complexity

**Time: O(2ⱽ × V)** where `V = n` — in the densest possible DAG (every lower-numbered node connected to every higher-numbered one), the number of distinct paths from `0` to `n-1` can itself be exponential (each intermediate node is independently either included in a given path or skipped over via a more direct edge), and each discovered path costs up to O(V) to copy into the result. **Space: O(V)** for recursion depth and the shared `path`, excluding the output list itself.

### Common Mistakes and Edge Cases

- ⚠️ **Adding a `visited` set here defensively, "just in case"** — proven above to be actively incorrect, not just extra: it would block legitimate multi-path revisits of the same node and silently under-report the answer.
- ⚠️ **Forgetting the un-choose step**, reasoning "this isn't backtracking, it's just DFS" by over-generalizing from Day 68's flood-fill — this problem's shared mutable `path` genuinely does need it, unlike Day 68's permanent-marking case; the two graph problems require opposite judgments about backtracking-vs-not, and conflating them in either direction produces a bug.
- Edge case: `node 0` connects directly to `n-1` — a single-edge path, found and recorded immediately without deeper recursion.
- Edge case: no path exists from `0` to `n-1` at all — `result` stays empty; the base case is simply never reached along any branch (the problem's own constraints guarantee at least one path exists, but the algorithm degrades correctly regardless).

> ⚠️ **Common Mistake:** treating "graph problem, recursive DFS" as one undifferentiated bucket that always needs `visited` and never needs un-choosing (or vice versa). Today's two problems and Day 68's two problems are four separate, individually-justified answers to "does this specific traversal need `visited`, and does it need an un-choose step" — not one blanket rule. State the reasoning for *this* problem, not a memorized default.

> 💡 **Interview Insight:** Proactively explaining why `visited` is correctly *absent* here — rather than an interviewer having to ask "wait, don't you need a visited array?" — pre-empts the single most likely pushback this problem invites, and demonstrates the "conditioned on cycles being possible" version of the rule is genuinely understood, not just memorized as "always add visited to graph problems."

---

## Extension (Optional) — 01 Matrix (LeetCode 542, Medium)

**Not required by this week's plan.** Extra practice, checked against `Week_11_Revised.md` and confirmed absent — direct multi-source-BFS sibling to Rotting Oranges above. Skippable under time pressure.

**Statement:** Given an `m × n` binary matrix, return the distance of the nearest `0` for each cell.

## The exact same mechanism as Problem 1, computing a different quantity

Seed BFS from **every** `0` cell simultaneously (multi-source, identical to Rotting Oranges' every-rotten-cell seeding), and use the level count as the answer for each cell — except this time, rather than tracking a single global `minutes` counter, each cell records *its own* level number as its distance, since every cell (not just the survivors of a stopping condition) needs an answer.

```java
public static int[][] updateMatrix(int[][] mat) {
    int rows = mat.length, cols = mat[0].length;
    int[][] dist = new int[rows][cols];
    Queue<int[]> queue = new ArrayDeque<>();

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (mat[r][c] == 0) {
                queue.offer(new int[]{r, c});   // multi-source seed, identical to Rotting Oranges
            } else {
                dist[r][c] = -1;   // "not yet reached" sentinel, distinct from a real distance of 0
            }
        }
    }

    int[][] directions = {{1,0},{-1,0},{0,1},{0,-1}};
    while (!queue.isEmpty()) {
        int[] cell = queue.poll();
        for (int[] dir : directions) {
            int nr = cell[0] + dir[0], nc = cell[1] + dir[1];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && dist[nr][nc] == -1) {
                dist[nr][nc] = dist[cell[0]][cell[1]] + 1;   // one step farther than the cell that reached it
                queue.offer(new int[]{nr, nc});
            }
        }
    }
    return dist;
}
```

**Confirmed by direct execution against the classic example:** `[[0,0,0],[0,1,0],[1,1,1]]` → `[[0,0,0],[0,1,0],[1,2,1]]`, matching the known correct output.

**Why this doesn't need a `levelSize`/explicit-level-counting loop the way Rotting Oranges did:** Rotting Oranges needed one shared `minutes` value, incremented once per *level*. Here, every cell needs its *own* distance value, and `dist[nr][nc] = dist[cell[0]][cell[1]] + 1` computes that directly from whichever cell first reached it — the level-by-level BFS *ordering* still guarantees correctness (a cell is first reached via the shortest possible path, from whichever source is nearest), but there's no need to explicitly track which level the algorithm is currently on to get each individual cell's correct answer.

> 🔗 **Forward reference:** recognizing this as "Rotting Oranges' mechanism, minus the shared counter, plus a per-cell answer" — rather than a new problem to solve from first principles — is the entire point of doing it as extension practice today rather than skipping it.

---

# Part 3 — Saga Choreography

## Prerequisites, confirmed

Kafka pub/sub fundamentals (Day 48), yesterday's Schema Registry (the event contracts a saga's steps communicate through), and Day 64's cascading-failure reasoning, which resurfaces directly below.

## The problem: a business transaction that spans services with no shared database

Creating an order, on this platform, genuinely spans multiple independently-deployed services with independent databases: Order records the order, Payment charges the customer, Notification confirms it. Inside a single monolith with one database, this would be one ACID transaction — succeed completely or roll back completely, no in-between state ever visible. Across independent services, **there is no single database transaction that can span all of them** — each service can only transactionally guarantee consistency within its *own* data.

**Why the obvious-sounding fix — a distributed transaction (two-phase commit) — is the wrong tool here, tying directly back to Day 64:** 2PC requires every participating service to hold its resources locked and stay available for the *entire* duration of the coordinated commit. A single slow or unavailable participant blocks the whole transaction — which is exactly the resource-exhaustion cascading-failure shape Day 64's circuit breaker exists to prevent, now happening at the level of an entire multi-step business transaction instead of one HTTP call. Distributed transactions trade away the *availability* that was the whole reason for choosing independent, separately-deployed services in the first place.

## The mechanism: local transactions, chained by events, undone by compensation

A **Saga** breaks the multi-service transaction into a sequence of independent **local transactions**, one per service, each committing (or failing) entirely within that one service's own database — no cross-service locking, ever. Coordination between steps happens via events, exactly the publish/subscribe mechanism already built this week.

```
Order module:    receives request → creates order (local transaction)
                 → publishes OrderCreated  (Day 69's exact schema-registered event)

Payment module:  listens for OrderCreated → attempts charge (local transaction)
                 → publishes PaymentCompleted, OR publishes PaymentFailed

Notification module: listens for PaymentCompleted → sends confirmation (local transaction)

Order module:    ALSO listens for PaymentFailed → executes a COMPENSATING
                 transaction: marks the order Cancelled (a new, explicit,
                 forward-moving local transaction — NOT a database rollback)
```

**Why "compensating transaction" and "rollback" are genuinely different ideas, worth not conflating:** by the time `PaymentFailed` is published, the Order module's local transaction has already **committed** — the order genuinely exists in Order's database, fully and correctly, as far as Order's own database is concerned. There is nothing left to roll back in the database sense. Undoing the *business* effect requires a **new**, explicit, forward-moving operation (marking the order cancelled) — an intentional compensating action, not a database mechanism reversing something that technically never finished.

## Choreography vs. Orchestration — the actual distinction, not just two names

**Orchestration** uses one central coordinator service that explicitly commands each participant what to do next (a synchronous or command-driven "call Payment, then call Notification" sequence, centrally directed). **Choreography** — what's built above — has **no central coordinator at all**: every service independently listens for the events relevant to it and reacts by publishing its own event when done, with the overall sequence emerging from each service's local, independent reactions rather than being centrally directed. This is precisely why choreography is the natural fit for what's already been built this week: it's the *same* Kafka pub/sub mechanism from Day 48, applied to business-transaction coordination instead of simple point-to-point messaging — no new coordination infrastructure is needed, only new event types and new listeners on top of what already exists.

### When to reach for Choreography vs. Orchestration

| | Choreography | Orchestration |
|---|---|---|
| Central coordinator? | No — each service reacts independently | Yes — one service directs the whole sequence |
| Adding a new step | New listener on an existing event, no change to other services | Central coordinator's logic must be updated |
| Understanding the FULL flow | Requires tracing events across every participating service | Visible in one place — the coordinator |
| Best fit when... | Steps are naturally reactive, participants already event-driven (this platform) | The sequence has complex branching logic worth centralizing and observing in one place |

### Common Mistakes

- ⚠️ **Treating a compensating transaction as "just roll back the database"** — proven above to be a category error once the original local transaction has already committed; it requires a new, explicit business operation instead.
- ⚠️ **Assuming choreography needs no coordination logic anywhere** — it needs no *central* coordinator, but every participant still needs correctly-written listeners for the events relevant to it, including failure events; skipping the failure-path listener (e.g., Order never listening for `PaymentFailed`) leaves the saga with no compensation at all, silently.
- ⚠️ **Reaching for a distributed transaction "to keep things simple"** — proven above to reintroduce exactly the availability cost (locked resources, blocked on the slowest participant) this week's resilience work has been specifically designed to avoid.

> 💡 **Interview Insight:** Naming the specific trade-off choreography makes — no central coordinator, but the full flow's logic is now scattered across every participant rather than visible in one place — is a stronger answer than presenting choreography as strictly superior to orchestration. Both are legitimate, and the honest trade-off is exactly what a tier-1 systems conversation is listening for.

---

## Project Block

**Repository:** `scalable-ecommerce-platform`.

**Task:** implement the Order → Payment → Notification saga above via Kafka events, including the compensating path (`PaymentFailed` → Order cancels).

**Practical guidance:** deliberately trigger the failure path (a Payment failure) in addition to the happy path — a saga's compensation logic is exactly the part most likely to be under-tested if only the successful flow is ever exercised. Confirm the order actually transitions to a `Cancelled` state end-to-end through real events, not through directly calling a cancellation method.

**Definition of done:** both the full success path (order created → payment completes → notification sent) and the compensation path (order created → payment fails → order cancelled) are demonstrated working end-to-end via real published and consumed events.

---

## Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts. Week wrap-up: a short reflection post on this week's platform work (Resilience4j through Saga) tends to read well, tying a full week's concrete progress into one narrative.

**Networking:** review the week — applications sent, responses received, any interviews scheduled for the coming week.

---

# Day 70 — Interview Questions

---

**1. What's the one-sentence change that turns single-source BFS into multi-source BFS?**

*Answer:* Seed the queue with every starting node before the first iteration, instead of just one — the level-by-level processing mechanism (Day 49's `queue.size()` trick) is otherwise completely unchanged.

---

**2. In Rotting Oranges, why does `minutes++` sit outside the inner per-cell loop rather than inside it?**

*Answer:* `minutes` should increment once per fully-processed BFS level, not once per cell — incrementing inside the inner loop would count once per cell processed within a level, producing a far larger number than the actual elapsed time.

---

**3. Does All Paths From Source to Target's lack of a `visited` structure contradict Day 68's rule that graphs need one?**

*Answer:* No — Day 68's rule was conditioned on cycles being possible, not an unconditional mandate. This problem's input is explicitly guaranteed to be a DAG, ruling out cycles entirely, so the specific failure mode `visited` exists to prevent simply cannot occur here regardless of whether it's present.

---

**4. Beyond being unnecessary, why would adding a global `visited` set to All Paths From Source to Target actually produce a wrong answer?**

*Answer:* The goal is enumerating every distinct path to the target, and a single node can legitimately appear on multiple different valid paths. A global visited set would mark a node "used" the first time any path reached it, incorrectly blocking a different, equally valid path from revisiting that same node later — silently dropping correct answers.

---

**5. Is All Paths From Source to Target backtracking, or plain graph DFS like Day 68's Number of Islands? Justify it.**

*Answer:* It's backtracking — it maintains one shared, mutable `path` structure across the whole exploration and genuinely needs the un-choose step (`path.remove`), since a node added while exploring one neighbor must be removed before a sibling neighbor's exploration begins, or the sibling's path would incorrectly include it. This is the opposite classification from Number of Islands, which permanently marks cells with nothing ever needing restoration — two visually similar recursive graph traversals, two different correct answers to "is this backtracking."

---

**6. (Extension) What's the one structural difference between Rotting Oranges and 01 Matrix's BFS, given they share the same multi-source seeding?**

*Answer:* Rotting Oranges tracks one shared `minutes` counter incremented per level, since it only needs a single global answer. 01 Matrix computes each cell's own distance directly as `dist[source] + 1` when first reached, since every cell individually needs its own answer — no shared level counter is needed for that.

---

**7. Why can't a single database transaction span Order, Payment, and Notification directly?**

*Answer:* Each service is independently deployed with its own separate database; a database transaction can only guarantee atomicity within one database. There is no single transaction mechanism that spans multiple independent databases the way a monolith's one database could.

---

**8. Why is a distributed transaction (two-phase commit) the wrong fix here, connecting back to Day 64?**

*Answer:* Two-phase commit requires every participating service to hold resources locked and remain available for the entire commit duration — a single slow or unavailable participant blocks the whole transaction, the same resource-exhaustion cascading-failure shape Day 64's circuit breaker exists to prevent, now at the scale of an entire business transaction instead of one call.

---

**9. Why is "compensating transaction" not the same idea as "database rollback"?**

*Answer:* By the time a later step fails and triggers compensation, the earlier local transaction has already committed successfully in its own database — there's nothing left to roll back. Undoing the business effect requires a new, explicit, forward-moving operation (like marking an order cancelled), not a database mechanism reversing an unfinished transaction.

---

**10. What's the actual trade-off choreography makes compared to orchestration — not just "which is better"?**

*Answer:* Choreography needs no central coordinator, and reuses the pub/sub mechanism already in place — but the full transaction's logic ends up scattered across every participating service's own listeners rather than visible in one place, making the complete flow harder to see and reason about all at once. Orchestration centralizes that visibility at the cost of a coordinator every participant now depends on.

---

## Daily Deliverable Check

- [ ] Backtracking self-check completed cold, before today's new material.
- [ ] Rotting Oranges (LC 994) solved — multi-source seeding explained as the one real change from single-source BFS.
- [ ] All Paths From Source to Target (LC 797) solved — the "why no visited, and why that doesn't contradict Day 68" reasoning stated explicitly.
- [ ] (Extension, optional) 01 Matrix (LC 542) — attempted if time allows.
- [ ] All attempted problems pushed to `dsa-java/graphs/`.
- [ ] Saga (success path AND compensation path) demonstrated working end-to-end via real Kafka events in `scalable-ecommerce-platform`.

---

## Week 10 Consolidation

### What actually got built

**DSA:** Backtracking **closed** — 8 problems this week (Combination Sum, Combination Sum II, Letter Combinations of a Phone Number, Generate Parentheses, Word Search, Palindrome Partitioning, Subsets II, N-Queens), bringing the pattern to a final **12/12 required, 0 extra**, spanning six genuinely distinct choice models (full classification table: Day 67). Graphs BFS/DFS **opened** — 6 required problems (Number of Islands, Max Area of Island, Clone Graph, Is Graph Bipartite?, Rotting Oranges, All Paths From Source to Target) plus 2 flagged extras (Find Eventual Safe States, 01 Matrix), with 6 more required problems arriving next week to close it at 12.

**Platform (`scalable-ecommerce-platform`):** four straight days of real microservices resilience and gateway infrastructure — Resilience4j circuit breakers, Feign declarative HTTP clients, Spring Cloud Gateway with centralized routing, JWT authentication at the Gateway, Token Bucket rate limiting, Kafka Schema Registry with Avro-contracted events, and Saga choreography with a real compensating-transaction path. Every one of these is live and independently verified (breaker tripping observed, fallback firing observed, routing tested with real requests, registry rejection tested directly, saga's failure path exercised, not just its happy path).

### Planned vs. actual problem count

| | Planned | Actual |
|---|---|---|
| Required problems | 14 (2/day × 7 days) | 14 — all solved, zero found already in the curriculum map |
| Extra practice added | — | 2 (Find Eventual Safe States, 01 Matrix — both Graphs, both checked clean against Week 11) |
| Extra practice explicitly withheld, with reasoning given | — | Backtracking (Days 64–67): dense, comprehensive 12-problem required ladder, no thin spots to fill (Day 67) |
| **Total distinct problems this week** | | **16** |
| **Cumulative distinct problems, Weeks 1–10** | | **175** (159 through Week 9 + 16 this week) |

### Diagnostic — check these cold before moving on

- Can you write Combination Sum II's `i > start` skip condition from memory, and explain in one sentence why `i > 0` is wrong?
- Can you name all six distinct backtracking choice models this series has covered, without looking at Day 67's table?
- Can you explain, out loud, why Number of Islands is *not* backtracking, and why All Paths From Source to Target *is* — despite both being recursive graph DFS?
- Can you state precisely when a graph needs an explicit `visited` structure, and why All Paths From Source to Target correctly has none?
- Can you explain what specifically breaks without a circuit breaker, in terms of thread-pool exhaustion — not just "it prevents failures"?

If any of these need the book open, that's this week's actual diagnostic signal — worth a focused half-hour before Week 11 builds further on top of it, rather than a silent gap carried forward.

### What next week assumes

Week 11 continues Graphs to its own close (Course Schedule, Course Schedule II, Surrounded Regions, Pacific Atlantic Water Flow, Word Ladder, and a threshold-distance shortest-path problem) — directly building on this week's BFS/DFS mechanics, and specifically extending today's Find Eventual Safe States' 3-state coloring into full topological-sort cycle detection for Course Schedule. It then opens Union-Find as a new structure entirely. On the platform side, Week 11 assumes the Gateway, resilience, and event infrastructure built this week are stable foundations the next layer of distributed-systems work builds directly on top of, not fixtures to be revisited from scratch.

---

## What Tomorrow Assumes You Already Know Cold

Day 71 (Week 11) assumes today's two extensions to Day 68's Graph fundamentals — multi-source BFS, and the "visited is conditional on cycles being possible, not unconditional" precision — are both solid, since Course Schedule's cycle detection is a direct escalation of exactly that precision (a directed graph where a cycle means the input itself is invalid, not just complicating traversal). It also assumes this week's full Backtracking closure (Day 67) remains reflexive going forward, since Week 11 introduces no new backtracking, but earlier patterns continuing to resurface as reinforcement — cited briefly, not re-taught — is this series' standing convention from here on.

**Next:** Day 71 Resource Book (Week 11) — Graphs Continue: Topological Sort and Course Scheduling.
