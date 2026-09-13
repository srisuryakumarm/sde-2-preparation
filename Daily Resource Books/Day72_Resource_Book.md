# Day 72 — Multi-Source Traversal, Reversed: Reachability From a Border, and Networking Fundamentals II (HTTP, Load Balancers)

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 71 Resource Book](Day71_Resource_Book.md)
**Next ▶:** [Day 73 Resource Book](Day73_Resource_Book.md)
**Companion to:** Day 72 of `Week_11_Revised.md`

---

## Recap

Day 70 introduced **multi-source BFS** for a *distance* question: seed the queue with every source at once, and the level counter measures "minutes elapsed" from whichever source reached a cell first (Rotting Oranges, 01 Matrix). Today reuses the exact same "seed with every source simultaneously" idea, but for a genuinely different question: not *how far*, but *can this cell reach the border at all*. That's a **reachability** question, not a distance question — and the distinction matters, because it changes which traversal (BFS or DFS) is actually required, something today makes explicit rather than assuming BFS is always the answer just because the source is multi-seeded.

---

## Learning Objectives

By the end of today, without notes:

1. State the difference between multi-source BFS's *distance* role (Day 70) and today's *reachability* role, and explain why the reachability version doesn't actually need BFS specifically.
2. Solve Surrounded Regions by marking border-connected cells safe first, never flipping a cell speculatively.
3. Prove — not just state — why Pacific Atlantic Water Flow's traversal must be *reversed* (search outward from each ocean, not outward from each cell), and reproduce the exact comparison-operator flip that reversal requires.
4. Explain HTTP as a protocol layered on TCP, and HTTPS as HTTP plus TLS.
5. Contrast L4 and L7 load balancing precisely enough to justify, unprompted, which one the Gateway you already built actually is.

---

## Concept Dependency Map

```
Day 68: Graphs — "a grid is a graph in disguise,"
        visited requirement, BFS-vs-DFS signal
        │
Day 70: Multi-source BFS — seed the queue with
        EVERY source before iteration 1; distance role
        │
        ▼
TODAY: Multi-source traversal, REACHABILITY role (new)
├─ Surrounded Regions (LC 130) — multi-source from
│  every border 'O', mark safe, flip the rest
└─ Pacific Atlantic Water Flow (LC 417) — multi-source
   from EACH ocean's border, REVERSED comparison
   (expand to height >= current, not <=)
        │
        ▼
Graphs BFS/DFS: 10/12 → 12/12 required tomorrow

Independent theory track:
Day 71: TCP/UDP, DNS
        │
        ▼
TODAY: Networking Fundamentals II — HTTP/HTTPS,
       Load Balancers (L4 vs L7), health checks
```

---

# Part 1 — Multi-Source Traversal: The Reachability Role

**The one idea underneath both of today's problems:** it's far cheaper to search *outward from a small set of known starting points* than to search *outward from every individual cell, asking whether it can reach one of those points*. Both problems below are naturally phrased the second way ("can this cell reach the border") and both get solved by flipping the search direction to the first way ("what can the border reach"). This flip is the actual technique being taught today — not either individual problem.

**Prerequisites, confirmed:** adjacency-via-coordinates and the `visited` requirement (Day 68) ✅, multi-source seeding (Day 70) ✅.

---

## Problem 9: Surrounded Regions (LeetCode 130, Medium) — Pattern: Multi-Source DFS/BFS From the Border

**Statement:** given an `m x n` board of `'X'` and `'O'`, capture every region of `'O'`s that is **fully surrounded** by `'X'`s (flip it to `'X'`) — a region counts as surrounded only if none of its cells connect, through other `'O'`s, to the border.

### Brute force

For every `'O'` cell not already on the border, run its own DFS/BFS to check whether that connected component touches the border at all. Correct, but each of up to `m×n` cells can trigger a search that itself costs up to `O(m×n)` — **O((mn)²)** worst case, since a single large connected region gets re-traversed once per cell that happens to sit inside it.

### Optimized: multi-source from the border, reversed

Instead of asking, for every interior `'O'`, "can you reach the border" — ask once, from the border, "what can *you* reach." Run DFS or BFS starting from **every** `'O'` that's actually on the border simultaneously, marking everything reachable through connected `'O'`s as safe (a temporary sentinel, `'#'`, works well — no separate `visited` array needed, since the sentinel itself records "already handled"). Afterward, sweep the whole board once: anything still `'O'` was never reached from the border, so flip it to `'X'`; anything marked `'#'` restore to `'O'`.

```java
public static void solve(char[][] board) {
    if (board == null || board.length == 0) return;
    int rows = board.length, cols = board[0].length;

    // Phase 1 — mark every border-connected 'O' as safe ('#')
    for (int r = 0; r < rows; r++) {
        markSafe(board, r, 0, rows, cols);
        markSafe(board, r, cols - 1, rows, cols);
    }
    for (int c = 0; c < cols; c++) {
        markSafe(board, 0, c, rows, cols);
        markSafe(board, rows - 1, c, rows, cols);
    }

    // Phase 2 — flip untouched 'O's, restore safe ones
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (board[r][c] == 'O') board[r][c] = 'X';       // never reached from border
            else if (board[r][c] == '#') board[r][c] = 'O';  // restore
        }
    }
}

private static void markSafe(char[][] board, int r, int c, int rows, int cols) {
    if (r < 0 || r >= rows || c < 0 || c >= cols || board[r][c] != 'O') return;
    board[r][c] = '#';
    markSafe(board, r + 1, c, rows, cols);
    markSafe(board, r - 1, c, rows, cols);
    markSafe(board, r, c + 1, rows, cols);
    markSafe(board, r, c - 1, rows, cols);
}
```

**Why this correctly captures exactly the surrounded regions — by definition, not by observation:** a region is surrounded *precisely when* it has no path, through adjacent `'O'`s, back to the border. `markSafe` computes the exact complement of that: everything reachable from a border `'O'` through a chain of `'O'`s. Whatever's left untouched after Phase 1 is, by that same definition, surrounded — there's no gap between what the algorithm computes and what the problem asks for.

**⚠️ Common Mistake:** flipping `'O'`s to `'X'`s *while scanning*, without a separate "mark safe first" pass. Interior `'O'`s that are genuinely border-connected can get processed before the algorithm has confirmed that connection if flipping and searching happen in the same pass — the two-phase separation (mark safe, *then* sweep) is what guarantees correctness regardless of scan order.

**⚠️ Common Mistake:** treating only the four corners, or only rows, as "the border." All of row 0, all of the last row, all of column 0, and all of the last column count.

**Worked trace** — the standard 4×4 case:

```
X X X X
X O O X
X X O X
X O X X
```

Border scan: the only border cell that's `'O'` is `(3,1)`. `markSafe(3,1)`: its neighbors are `(2,1)='X'`, `(3,0)='X'`, `(3,2)='X'` — none are `'O'`, so the search stops immediately; `(3,1)` becomes `'#'` and nothing else. No other border cell is `'O'`, so Phase 1 ends there.

Phase 2 sweep: the connected group `{(1,1), (1,2), (2,2)}` is still `'O'` (never touched in Phase 1, since it has no path to `(3,1)` or any other border cell) → flipped to `'X'`. `(3,1)` is `'#'` → restored to `'O'`.

```
X X X X
X X X X
X X X X
X O X X
```

Verify by definition: `(3,1)` sits ON the border, so it can't be "surrounded" — correctly preserved. The group `{(1,1),(1,2),(2,2)}` has no adjacent path to any border cell — correctly captured.

**Complexity:** Time O(m×n) — every cell is visited a constant number of times total across the unified search (as opposed to the brute force's per-cell re-search). Space O(m×n) worst case, for the recursion stack on a board that's entirely `'O'`.

**Edge cases:** a board with no `'O'`s at all (nothing to do); a board entirely `'O'` (every cell reachable from the border, since the whole board touches it — nothing gets flipped); a single-row or single-column board (every cell is border by definition, nothing is ever surrounded).

**💡 Interview Insight:** naming the brute force's O((mn)²) cost *and* the specific reason the reversal fixes it ("stop asking every interior cell to search for the border; ask the border to search for everyone") is a stronger opening than jumping straight to the multi-source code — it's the same sentence that will also justify the next problem.

---

## Problem 10: Pacific Atlantic Water Flow (LeetCode 417, Medium) — Pattern: Multi-Source BFS/DFS, Reversed

**Statement:** given an `m x n` grid of heights, the Pacific touches the top row and left column, the Atlantic touches the bottom row and right column. Water flows from a cell to an orthogonal neighbor only if the neighbor's height is **less than or equal to** the current cell's height (downhill or flat). Return every cell from which water can reach **both** oceans.

### Brute force

For each of the `m×n` cells, run a DFS/BFS checking whether a valid non-increasing path exists to *each* ocean's border. **O((mn)²)** worst case — the same redundant-re-search cost as Surrounded Regions, for the identical structural reason.

### Optimized: multi-source from each ocean, with the comparison **reversed**

**The idea, precisely:** instead of asking, from each cell, "can I reach the ocean," start at the ocean and ask, "what can reach *me*." But reachability by real water flow requires height to be **non-increasing** along the path *toward* the ocean — so walking the search **backward**, from the ocean outward, requires the **opposite** condition: a neighbor is added to the reachable set only if its height is **greater than or equal to** the current cell's height.

**Why this reversal is correct — proven, not asserted:** real water flow allows a step from cell `A` to neighbor `B` exactly when `height(B) ≤ height(A)`. Reverse that single edge: walking from `B` back to `A` is the same edge, just traversed backward, and it's governed by the identical underlying rule — `height(B) ≤ height(A)` is the same statement as `height(A) ≥ height(B)`. So a reversed search starting *at* the ocean and stepping to a neighbor `N` is valid exactly when `height(N) ≥ height(current)`. Every reversed path found this way, read backward, is a real, valid forward flow path to the ocean — the reversal doesn't approximate the original condition, it's logically identical to it, just walked in the opposite direction.

Run this reversed search twice — once seeded with every Pacific-border cell, once with every Atlantic-border cell — producing two boolean "reachable" grids. The answer is every cell marked `true` in **both**.

```java
public static List<List<Integer>> pacificAtlantic(int[][] heights) {
    int rows = heights.length, cols = heights[0].length;
    boolean[][] pacific = new boolean[rows][cols];
    boolean[][] atlantic = new boolean[rows][cols];

    for (int r = 0; r < rows; r++) {
        dfs(heights, pacific, r, 0, rows, cols);          // left column
        dfs(heights, atlantic, r, cols - 1, rows, cols);  // right column
    }
    for (int c = 0; c < cols; c++) {
        dfs(heights, pacific, 0, c, rows, cols);          // top row
        dfs(heights, atlantic, rows - 1, c, rows, cols);  // bottom row
    }

    List<List<Integer>> result = new ArrayList<>();
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (pacific[r][c] && atlantic[r][c]) {
                result.add(Arrays.asList(r, c));
            }
        }
    }
    return result;
}

private static void dfs(int[][] heights, boolean[][] reachable, int r, int c, int rows, int cols) {
    reachable[r][c] = true;
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    for (int[] d : dirs) {
        int nr = r + d[0], nc = c + d[1];
        if (nr < 0 || nr >= rows || nc < 0 || nc >= cols || reachable[nr][nc]) continue;
        if (heights[nr][nc] >= heights[r][c]) {   // REVERSED condition — the entire trick
            dfs(heights, reachable, nr, nc, rows, cols);
        }
    }
}
```

**⚠️ Common Mistake, and the single highest-stakes line in this problem:** using `<=` (the *forward* flow condition) instead of `>=` when expanding from the ocean. This is not a minor slip — it silently inverts the entire algorithm's correctness while still compiling and running without error, just producing a wrong answer on any grid where height actually varies.

### Worked Trace

```
      col0 col1 col2
row0:  1    2    3
row1:  8    9    4
row2:  7    6    5
```

Pacific border = row 0 + column 0: `{(0,0),(0,1),(0,2),(1,0),(2,0)}`. Atlantic border = row 2 + column 2: `{(2,0),(2,1),(2,2),(0,2),(1,2)}`.

**Reversed search from Pacific**, expanding to `height(neighbor) ≥ height(current)`: seeding all 5 Pacific-border cells, `(0,1)=2 → (1,1)=9` (9≥2, valid), `(0,2)=3 → (1,2)=4` (4≥3, valid) `→ (2,2)=5` (5≥4, valid) `→ (2,1)=6` (6≥5, valid). Every one of the 9 cells ends up Pacific-reachable — verified concretely for `(2,1)=6`: reversed path `(0,2)→(1,2)→(2,2)→(2,1)` found; reading it **backward** as a real flow path gives `(2,1)=6 → (2,2)=5 → (1,2)=4 → (0,2)=3`, and `6≥5≥4≥3` — a genuinely valid non-increasing forward flow all the way to a Pacific-border cell. The reversal produced a real, checkable path, not just a claim.

**Reversed search from Atlantic**, same rule, seeded from the Atlantic border: this reaches `(1,0)=8` (from `(2,0)=7`, since `8≥7`), then `(1,1)=9` (from `(1,0)=8` or `(2,1)=6`), and everything else the Atlantic border touches — **except** `(0,0)=1` and `(0,1)=2`. Check `(0,1)=2` directly: its only neighbors are `(0,0)=1` (lower — a valid step, but a dead end that's already Pacific-only) and `(0,2)=3` and `(1,1)=9` (both *higher* than 2, so real forward flow from `(0,1)` can never step to either — flow requires non-increasing height). `(0,1)` is stuck; it cannot reach the Atlantic.

**Result:** every cell except `(0,0)` and `(0,1)` reaches both oceans — 7 of the 9 cells. This matches direct inspection: `(0,0)=1` is already sitting at the Pacific border with nowhere lower to flow, and `(0,1)=2`'s only reachable neighbor is `(0,0)`, a dead end.

**Complexity:** Time O(m×n) — two separate multi-source searches, each visiting every cell at most once, so `O(2×m×n) = O(m×n)`. Space O(m×n) for the two boolean grids plus recursion stack.

**Edge cases:** a corner cell (e.g., `(0,0)` in a grid where Pacific = top+left, Atlantic = bottom+right) sits on exactly one ocean's border unless the grid is `1×1`, in which case it touches both simultaneously and is trivially in the answer set; a completely flat grid (every height equal) — every cell can flow to every neighbor, so every cell reaches both oceans.

**💡 Interview Insight:** if asked "why not just do the straightforward forward search from every cell" — the answer isn't "it's slower," it's the specific *shape* of the slowdown: the forward version repeats work proportional to how large connected flat-or-descending regions are, exactly like Surrounded Regions' brute force repeats work proportional to component size. Naming that shared root cause across two superficially different problems is exactly the kind of pattern-transfer an interviewer is listening for.

---

**This closes Graphs at 12/12 required — up from 9 in the original plan.** One day remains before the pattern's full review (Day 73), but the required ladder itself is complete as of today.

**Note on extra practice:** none added today, for the same reasoning as Day 71 — Graphs is closing this week at a comprehensive 14 distinct problems (12 required + 2 extra from Week 10), and both of today's problems are themselves fresh reps of the reachability role, not a repeat of anything already covered.

---

# Part 2 — Networking Fundamentals II: HTTP, HTTPS, and Load Balancers

### Prerequisites (confirmed)

TCP (Day 71) ✅ — HTTP is built directly on top of it, not an independent protocol.

## HTTP: Request/Response, Layered on TCP

**What it is:** HTTP is a request/response protocol. A client opens a TCP connection (Day 71's three-way handshake happens first, underneath, invisibly to the application), then sends a request — a method (`GET`, `POST`, `PUT`, `DELETE`, ...), a path, headers, and an optional body. The server responds with a status code, headers, and an optional body, over that same TCP connection.

**HTTPS = HTTP + TLS.** Before any HTTP request/response data flows, a TLS handshake negotiates an encrypted channel — the server presents a certificate, both sides agree on encryption keys, and everything sent afterward (including the HTTP request itself, method and path included) is encrypted in transit. HTTP itself doesn't change; TLS is a layer added underneath it, the same way HTTP itself sits on top of TCP.

## Load Balancers: Distributing Traffic Across Multiple Servers

**What it is:** a load balancer sits in front of a pool of backend server instances and decides, for each incoming request, which instance handles it. This is what lets a system scale horizontally (add more instances) instead of only vertically (make one instance bigger).

### Layer 4 (L4) Load Balancing

Operates at the **transport layer** — it sees IP addresses and ports, nothing about the actual content of the request. It's fast, because there's nothing to parse beyond a packet header, but it's genuinely blind: it cannot route based on a URL path, a header, or a cookie, because it never looks that far into the data.

### Layer 7 (L7) Load Balancing

Operates at the **application layer** — it actually terminates and reads the HTTP request (method, path, headers, cookies) before deciding where to send it. This costs more overhead (parsing a full request instead of a packet header), but buys real flexibility: routing `/api/*` to one service and `/static/*` to another, sticky sessions via a cookie, or — directly relevant to what's already been built — validating a JWT before a request is ever allowed through at all.

**🔗 Direct connection to what's already built:** the `scalable-ecommerce-platform`'s Spring Cloud Gateway (Day 66) reads the request path to decide which downstream module handles it, and validates a JWT (Day 67) before forwarding anything — both of those are **L7 behaviors by definition**. An L4 load balancer, seeing only IP and port, could not perform either — it has no visibility into a path string or an `Authorization` header at all. The Gateway isn't "like" an L7 load balancer; for the purposes of this platform, it *is* one, with additional application logic layered on top.

| | L4 | L7 |
|---|---|---|
| Operates on | IP + port | Full HTTP request (path, headers, cookies, body) |
| Speed | Faster — minimal parsing | Slower — must terminate and parse the request |
| Routing granularity | Coarse (which server) | Fine (which service, based on path/header/cookie) |
| Can validate a JWT? | No — never sees it | Yes |
| Typical use | Raw TCP/UDP traffic, extreme low-latency needs | HTTP APIs, microservice routing (this platform) |

### Health Checks: How a Load Balancer Stops Sending Traffic to a Dead Instance

A load balancer periodically checks each backend instance — commonly by hitting a dedicated `/health` endpoint on a schedule. An instance that stops responding correctly (times out, or returns an error status) gets marked unhealthy and is removed from rotation automatically, with no human paging anyone first. **🔗 This is the same "stop cascading failure automatically" instinct behind Day 64's Resilience4j Circuit Breaker** — a different mechanism (external health polling vs. an internal proxy watching call outcomes), but the same underlying goal: detect a failing dependency and stop sending it work before it drags everything else down too.

**⚠️ Common Mistake:** assuming a load balancer is *either* L4 *or* L7 as some kind of permanent architectural label for "a load balancer" in general. Real infrastructure frequently layers both — an L4 balancer distributing raw connections across a set of L7 balancers/gateways, each of which does the finer-grained routing. The Gateway sitting in front of this platform's modules is the L7 layer of exactly that kind of stack.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`. No new feature work — today's task is the writing exercise itself.

**Task:** write roughly 150 words comparing when you'd choose an L4 load balancer over an L7 one, specifically for the Gateway already built. A strong answer names what the Gateway currently does that requires L7 (path-based routing, JWT validation) and states plainly that none of that is possible at L4 — then names the actual scenario where L4 would be the right call regardless (e.g., a raw low-latency TCP service with no HTTP semantics at all, or an L4 balancer placed *in front of* a fleet of L7 gateways rather than replacing one).

**Definition of done:** the ~150-word comparison written and saved alongside the platform's docs.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement only today — 20 minutes commenting thoughtfully on 3–5 posts in your network. No new post; not every day needs one, and forcing one out on a lighter theory day tends to produce weaker content than letting the next genuinely interesting result (tomorrow's Graphs closing, or Union-Find opening in two days) carry a post instead.

**Networking:** send the connection requests to the 3 infra-focused target companies identified yesterday. Keep each note short and specific — one concrete detail from their profile or a recent post beats a generic "I'd love to connect" by a wide margin in response rate.

---

## Day 72 — Interview Questions

**Q1. What's the difference between Day 70's multi-source BFS and today's technique, even though both seed multiple starting points at once?**
*A:* Day 70 answers a *distance* question (minutes elapsed, using BFS's level-by-level guarantee specifically). Today answers a *reachability* question (can this cell reach the border at all) — which doesn't need BFS's level structure, so DFS works equally well and is often simpler code.

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

**Q7. What's the shared root technique connecting today's two problems?**
*A:* Both flip "search from every interior point toward a small target set" into "search from the small target set outward" — turning an O((mn)²) worst case into O(mn) by eliminating redundant re-traversal of shared regions.

**Q8. Is the Gateway already built an L4 or L7 load balancer, and how do you know?**
*A:* L7 — it routes based on the HTTP path and validates a JWT from the request header, both of which require reading the actual HTTP request content, something an L4 balancer (IP/port only) never sees.

**Q9. What's the mechanical difference between a health check and a circuit breaker, given both stop traffic to a failing dependency?**
*A:* A health check is the load balancer actively polling a dedicated endpoint on a schedule, external to the request path. A circuit breaker (Day 64) watches the outcomes of real calls as they happen and trips based on an observed failure rate — no separate polling endpoint involved.

**Q10. Why does HTTPS not change how HTTP itself works?**
*A:* TLS is layered underneath HTTP, encrypting the connection before any HTTP request/response data is sent — the request/response model, methods, and status codes are unchanged; only the transport is now encrypted.

---

## Daily Deliverable Check

- [ ] Surrounded Regions and Pacific Atlantic Water Flow solved, pushed — Graphs ladder now closes at 12/12 required.
- [ ] Can explain, from memory, why the reachability role doesn't require BFS specifically, unlike Day 70's distance role.
- [ ] Can reproduce the exact reversed comparison (`≥`, not `≤`) for Pacific Atlantic and explain why the reversal is correct, not just faster.
- [ ] L4 vs. L7 load balancer comparison written (~150 words), naming which one the Gateway already is.
- [ ] Connection requests sent to the 3 infra-focused targets identified yesterday.

---

## What Tomorrow Assumes You Already Know Cold

Day 73 closes Graphs with two problems that sit in genuinely different families from anything traversal-shaped: single-source BFS shortest path on an *implicit* graph (Word Ladder), and an entirely new algorithm family, Floyd-Warshall, that isn't a traversal at all. It assumes today's careful distinction between BFS's distance role and DFS/BFS's reachability role is solid, since tomorrow draws yet another distinction (single-source vs. multi-source, and traversal vs. non-traversal) on top of it — the habit of asking "what exactly is this search computing" before picking an algorithm, not just "is this a graph problem," is what today was actually building.

**Next:** [Day 73 Resource Book](Day73_Resource_Book.md) — Graphs Capstone: Word Ladder and Floyd-Warshall.
