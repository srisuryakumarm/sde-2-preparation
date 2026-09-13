# Day 44 — Stacks: The Hard Tier, and Docker Compose

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 43 Resource Book](Day43_Resource_Book.md)
**Next ▶:** [Day 45 Resource Book](Day45_Resource_Book.md)
**Companion to:** Day 44 of `Week_07_Revised.md`

---

## Recap

Yesterday used the stack as a simulation tool — deferred, correctable state. Today returns to the strictly *monotonic* discipline from Day 40–41 (increasing/decreasing invariant, each element pushed and popped at most once) and applies it to the hardest problem shape it supports: Largest Rectangle in Histogram, followed by Maximal Rectangle, which reduces a 2D problem to n applications of the first. On the theory side, yesterday's single-container Docker basics extend today to orchestrating several containers together with Docker Compose.

## Learning Objectives

By the end of today, without notes:

1. Explain why Largest Rectangle in Histogram's stack must hold *indices*, not heights, and state the invariant it maintains.
2. Trace the pop-and-compute-width step precisely enough to get the boundary arithmetic right under pressure.
3. Explain how Maximal Rectangle reduces a 2D matrix problem to n independent calls of yesterday's — today's — histogram algorithm, and why that reduction is valid.
4. Write a `docker-compose.yml` that runs `todo-api`, Postgres, and Redis together, and explain how the containers resolve each other by name.

## Concept Dependency Map

```
Day 40/41 — Monotonic Stack invariant (increasing/decreasing) +
            amortized "pushed/popped once" proof
        │
        ▼
LC 84 Largest Rectangle in Histogram — stack of INDICES, strictly
increasing heights; a shorter incoming bar pops and finalizes every
taller bar it disqualifies
        │
        ▼
LC 85 Maximal Rectangle — row-by-row: build a "how many 1s stack up
above this cell" histogram per row, run LC 84 on it, n times
        │
        ▼
Day 43 — Docker image/container, Dockerfile layer caching
        │
        ▼
NEW: Docker Compose — one docker-compose.yml orchestrates MULTIPLE
containers; same-network containers resolve each other by SERVICE
NAME, no manual IP configuration
        │
        ▼
Project: todo-api + Postgres + Redis, wired together via Compose
```

---

## Problem 10: Largest Rectangle in Histogram (LeetCode 84, Hard) — Pattern: Monotonic Stack

**Statement:** Given an array of non-negative integers representing bar heights of a histogram where each bar has width 1, find the area of the largest rectangle that can be formed within the histogram's outline.

This is the hardest problem the series has posed so far, and it's worth the full three-approach treatment.

### Approach 1 — Brute force: expand around every bar

```java
public static int largestRectangleAreaBruteForce(int[] heights) {
    int maxArea = 0;
    int n = heights.length;
    for (int i = 0; i < n; i++) {
        int height = heights[i];
        int left = i, right = i;
        while (left > 0 && heights[left - 1] >= height) left--;
        while (right < n - 1 && heights[right + 1] >= height) right++;
        maxArea = Math.max(maxArea, height * (right - left + 1));
    }
    return maxArea;
}
```

For every bar, treat it as the *shortest* bar in some candidate rectangle, and expand left and right as far as the surrounding bars stay at least that tall. Correct — every possible maximal rectangle has *some* bar as its limiting height, and that bar is checked directly — but each of the n bars can trigger an O(n) expansion, giving **O(n²)**.

### Approach 2 — Divide and conquer (a genuinely distinct middle ground)

```java
public static int largestRectangleAreaDivideConquer(int[] heights) {
    return helper(heights, 0, heights.length - 1);
}

private static int helper(int[] heights, int left, int right) {
    if (left > right) return 0;
    if (left == right) return heights[left];

    int minIndex = left;
    for (int i = left; i <= right; i++) {
        if (heights[i] < heights[minIndex]) minIndex = i;
    }

    int throughMin = heights[minIndex] * (right - left + 1);
    return Math.max(throughMin,
           Math.max(helper(heights, left, minIndex - 1),
                     helper(heights, minIndex + 1, right)));
}
```

**The insight:** the largest rectangle either spans the *entire* current range at the range's minimum height, or it lies entirely to the left of that minimum, or entirely to the right — it can never straddle the minimum without being limited by it. Find the minimum (O(range size) per call), then recurse on both sides. When the array is roughly balanced by each split, this gives **O(n log n)** — but a sorted (or reverse-sorted) input puts the minimum at one end every time, degrading to **O(n²)**, the same worst case as Approach 1. It's worth knowing this approach exists — it's a legitimate, sometimes-asked technique — but its worst case doesn't actually beat brute force, which is exactly why Approach 3 is the one to reach for by default.

### Approach 3 — Optimized: monotonic increasing stack

```java
public static int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>();   // stores INDICES; heights at those indices strictly increasing
    int maxArea = 0;
    int n = heights.length;

    for (int i = 0; i <= n; i++) {
        int currentHeight = (i == n) ? 0 : heights[i];   // sentinel 0 flushes the stack at the end
        while (!stack.isEmpty() && heights[stack.peek()] >= currentHeight) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
```

**Why the stack holds indices, not heights:** computing a rectangle's width needs to know *how far apart* two bars are, which means knowing their positions — a height alone can't tell you that.

**The invariant:** the stack always holds indices whose heights are strictly increasing, bottom to top. When a new bar is shorter than (or equal to) the bar on top of the stack, that top bar can never extend any further right than the current position — its rectangle is now fully determined, so it's finalized: popped, and its area computed using its own height, with a width that spans from *one past the new stack top* (its nearest taller-or-equal neighbor on the left) to *the current index* (its nearest shorter neighbor on the right, exclusive). This is the exact same "expand to the nearest shorter bar on each side" idea as Approach 1 — the stack is just what lets every bar discover its own boundaries in O(1) amortized time instead of a fresh O(n) scan.

The sentinel `i == n → currentHeight = 0` at the end is what forces every bar still on the stack to get finalized, rather than needing a separate cleanup loop after the main one.

**Trace:** `heights = [2, 1, 5, 6, 2, 3]`.

| i | currentHeight | pops (height, width, area) | maxArea | stack after |
|---|---|---|---|---|
| 0 | 2 | — | 0 | `[0]` |
| 1 | 1 | pop 0: h=2, w=1 (stack empty→w=i=1), area=2 | 2 | `[1]` |
| 2 | 5 | — | 2 | `[1, 2]` |
| 3 | 6 | — | 2 | `[1, 2, 3]` |
| 4 | 2 | pop 3: h=6, w=4−2−1=1, area=6 → pop 2: h=5, w=4−1−1=2, area=**10** | **10** | `[1, 4]` |
| 5 | 3 | — | 10 | `[1, 4, 5]` |
| 6 (sentinel) | 0 | pop 5: h=3,w=1,area=3 → pop 4: h=2,w=4,area=8 → pop 1: h=1,w=6,area=6 | 10 | `[6]` |

Final answer: **10** (the `5, 6` bars, width 2, from the `[5,6]` pair). Matches the known result for this classic example.

**Complexity: Time O(n) — Day 41's amortized argument again: every index is pushed once and popped at most once across the whole run. Space O(n)** worst case (a strictly increasing input never pops until the sentinel).

**Edge cases:** all bars the same height (the `>=` in the pop condition, not `>`, matters here — using strict `>` would leave equal-height bars un-merged and undercount the width); a single bar (`n=1`, sentinel loop still handles it correctly); strictly decreasing input (every bar pops on the very next iteration — the stack never holds more than one index at a time).

> 💡 **Interview Insight:** This is a problem where narrating the *invariant* before writing code is what separates a candidate who's memorized the solution from one who understands it. Say out loud: "the stack stays strictly increasing in height; when that breaks, the popped bar's right boundary is exactly here, and its left boundary is whatever's now exposed underneath it" — then the code is just executing that sentence.

---

## Problem 11: Maximal Rectangle (LeetCode 85, Hard) — Pattern: Monotonic Stack, applied row by row

**Statement:** Given a 2D binary matrix filled with `'0'` and `'1'`, find the area of the largest rectangle containing only `'1'`s.

### Approach 1 — Brute force: check every possible rectangle

For every pair of corners (or every top-left corner and every width/height), verify the entire rectangle is all `'1'`s. This is at least **O((mn)²)** — there are O(mn) possible top-left corners and O(mn) possible bottom-right corners, and verifying each candidate rectangle costs additional work on top of that. Correct, hopelessly slow for anything but toy inputs.

### Approach 2 — Optimized: reduce to n calls of Problem 10

```java
public static int maximalRectangle(char[][] matrix) {
    if (matrix.length == 0 || matrix[0].length == 0) return 0;
    int cols = matrix[0].length;
    int[] heights = new int[cols];
    int maxArea = 0;

    for (char[] row : matrix) {
        for (int j = 0; j < cols; j++) {
            heights[j] = (row[j] == '1') ? heights[j] + 1 : 0;
        }
        maxArea = Math.max(maxArea, largestRectangleArea(heights));   // today's Problem 10, unchanged
    }
    return maxArea;
}
```

**The reduction:** treat each row as the *base* of a histogram, where `heights[j]` is "how many consecutive `'1'`s stack up directly above this cell, including this row." A `'0'` resets that column's height to 0; a `'1'` extends it by one from the row above. After processing row `i`, `heights` is exactly the histogram you'd see if you sliced the matrix horizontally at row `i` and looked straight up — and the largest all-`'1'`s rectangle *whose bottom edge is row `i`* is exactly the largest rectangle in *that* histogram, which is precisely the problem you already solved above. Running this once per row and tracking the global max covers every possible bottom edge, which covers every possible rectangle.

**Trace (bottom edge = row 2 of the standard 4×5 example):**

```
Row 0: 1 0 1 0 0        heights after row 0: [1, 0, 1, 0, 0]
Row 1: 1 0 1 1 1        heights after row 1: [2, 0, 2, 1, 1]
Row 2: 1 1 1 1 1        heights after row 2: [3, 1, 3, 2, 2]
Row 3: 1 0 0 1 0        heights after row 3: [4, 0, 0, 3, 0]
```

Running Problem 10's algorithm on `[3, 1, 3, 2, 2]` (heights after row 2) finds a rectangle of height 2 spanning columns 2–4 (heights `3,2,2`, limited by the smallest, `2`, over width 3) → area 6, which is the correct answer for this classic example.

**Complexity: Time O(m × n) — building each row's height array is O(n), and Problem 10's algorithm is itself O(n), run once per row (m rows). Space O(n)** — one reusable height array plus Problem 10's stack, neither of which scales with the number of rows.

**Edge cases:** an all-`'0'` matrix (every histogram is all-zero, area 0 throughout); a single row (equivalent to running Problem 10 once, directly); an all-`'1'` matrix (`heights` grows every row without resetting, and the answer converges to `m × n`).

> 💡 **Interview Insight:** The moment to earn credit here is naming the reduction *before* writing any code — "this becomes n calls to Largest Rectangle in Histogram, using each row as a base" is the entire idea, and a candidate who states that plainly, then reuses yesterday's — excuse — today's earlier function verbatim, reads as someone who recognizes when a new-looking problem is actually an old one in disguise, which is exactly the muscle this whole series is built around.

*(No extra practice added today — Stacks/Monotonic Stack is closing, not opening, and already picked up two extra reps back in Week 6 [Day 41: LC 901, 402]; today's own two required problems are both Hard and dense enough on their own.)*

---

# Part 2 — Docker Compose

### Prerequisites (confirmed)

Docker image vs. container, and `Dockerfile` mechanics — Day 43, directly. Today doesn't introduce a new execution model, just a way to coordinate several containers that already work individually.

### The Problem Compose Solves

`todo-api` needs Postgres and (starting today) Redis to actually function. Running each with separate `docker run` commands means manually creating a shared network, remembering every flag on every restart, and manually wiring connection strings to whatever IP address each container happens to get. **Docker Compose** replaces all of that with one declarative file:

```yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/tododb
      SPRING_REDIS_HOST: cache
    depends_on:
      - db
      - cache

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: tododb
      POSTGRES_PASSWORD: devpassword
    volumes:
      - pgdata:/var/lib/postgresql/data

  cache:
    image: redis:7-alpine

volumes:
  pgdata:
```

### Service-Name Resolution

`docker-compose up` creates one shared network for every service in the file, and gives each service **DNS resolution by its service name** automatically — inside the `app` container, the hostname `db` resolves to the Postgres container's address, and `cache` resolves to Redis's, with no manual IP lookup or configuration anywhere. This is exactly why the JDBC URL above says `jdbc:postgresql://db:5432/...` — `db` is the *service name* from this file, not a literal hostname that exists anywhere else. Compare this to `localhost`-based connection strings you'd use running everything unContainerized on one machine (as `todo-api` has, until today) — inside Compose's network, `localhost` from the `app` container's perspective refers to the `app` container itself, not to `db` or `cache`, which is a very common source of "connection refused" confusion the first time someone containerizes a previously-`localhost`-wired app.

### `volumes` and Data Persistence

A container's writable layer (Day 43) is discarded when the container is removed — for a database, that would mean losing all data on every `docker-compose down`. The `volumes:` entry mounts a Docker-managed volume at Postgres's data directory, which persists independently of the container's own lifecycle — removing and recreating the `db` container leaves the volume, and the data in it, untouched.

### `depends_on` — What It Actually Guarantees

`depends_on` controls **start order**, not **readiness**. `db` will be *started* before `app`, but Postgres can take a moment after its process starts before it's actually ready to accept connections — `app` can still fail to connect on a cold start if it tries before Postgres has finished initializing.

> ⚠️ **Common Mistake:** Treating `depends_on` as "wait until the dependency is ready." It only guarantees container *start* order, not service *readiness* — a real production Compose file typically adds a `healthcheck:` block to `db` and a `condition: service_healthy` under `app`'s `depends_on` to actually wait for readiness. Worth knowing the gap exists even if today's project doesn't need to close it for local development.

### Common Mistakes

- ⚠️ **Wiring `localhost` instead of the service name** — inside Compose's network, each service is a separate host; `localhost` refers to the container making the request, not to another service.
- ⚠️ **Forgetting `depends_on`'s readiness gap** (above) — intermittent startup failures that "fix themselves" on a manual retry are the classic symptom.
- ⚠️ **No named volume for stateful services** — data silently vanishes on the next `docker-compose down`, which is easy to not notice until it matters.

---

# Project Block Guide (1.5 hrs)

**Repository:** `todo-api`. Write the `docker-compose.yml` above (adjusted to your actual `application.yml` property names from Day 34/36), wiring `app`, `db` (Postgres), and `cache` (Redis).

**Definition of done:** `docker-compose up -d` brings up all three services; `todo-api` successfully connects to both Postgres and Redis using their *service names*, not `localhost` or a hardcoded IP; `docker-compose down` and `docker-compose up -d` again preserves your Postgres data (confirm by creating a task, tearing down, bringing back up, and checking it's still there).

---

# Career Block Guide (1 hr)

### LinkedIn

Engagement day — spend 20 minutes commenting thoughtfully on 3–5 posts in your network (not just "Great post!" — reference something specific from the content, or add a related detail from your own experience).

### Networking

Identify 2 more Tier C companies for early practice applications, adding to yesterday's 2.

---

# Day 44 — Interview Questions

**Q1. Why must Largest Rectangle in Histogram's stack hold indices rather than heights directly?**
Computing a rectangle's area needs its width, which depends on the *positions* of its left and right boundaries — a bare height value carries no positional information, so the stack has to store indices to let width be computed as a difference between positions.

**Q2. State the monotonic stack's invariant in this problem, precisely.**
The stack holds indices whose corresponding heights are strictly increasing from bottom to top. Whenever an incoming bar's height is less than or equal to the height at the current top, that top index is popped and finalized, because the incoming bar proves the popped bar can never extend any further to the right.

**Q3. When a bar is popped, how are its left and right boundaries determined?**
Its right boundary is the current index `i` (the first bar that disqualifies it). Its left boundary is one past whatever index is newly exposed on top of the stack after the pop — the nearest remaining bar that's still at least as tall. Width is the difference between those two positions, minus one.

**Q4. Why does the algorithm append a sentinel height of 0 at the end instead of just stopping after the input array?**
The sentinel forces every bar still sitting on the stack at the end to be finalized through the same pop-and-compute logic already used throughout the pass, instead of requiring a separate cleanup loop with duplicated logic.

**Q5. Why does the pop condition use `>=` rather than strict `>`?**
Using `>=` ensures equal-height bars get merged into a single continuous span rather than stopping short — with strict `>`, a run of equal-height bars would understate their combined width, since the algorithm would never pop (and thus never account for) a bar of exactly the same height as the one about to be pushed.

**Q6. Describe the divide-and-conquer alternative for Largest Rectangle in Histogram, and explain why it isn't the default choice despite being O(n log n) on average.**
Find the minimum height in the current range, take the max of the rectangle spanning the whole range at that minimum height, the best rectangle entirely left of the minimum, and the best rectangle entirely right of it — recursing on both sides. It's O(n log n) when the splits are balanced, but a sorted or reverse-sorted input puts the minimum at one end every time, degrading it to the same O(n²) worst case as the brute-force approach, which is exactly what the stack approach avoids unconditionally.

**Q7. Explain, precisely, how Maximal Rectangle reduces to Largest Rectangle in Histogram.**
Track, per column, how many consecutive `'1'`s stack up directly above the current row (resetting to 0 on a `'0'`). After processing each row, that array is exactly the histogram you'd see slicing the matrix at that row and looking upward — and the largest all-`'1'`s rectangle with its bottom edge on that row is exactly the largest rectangle in that histogram. Running this once per row and tracking a global max covers every possible bottom edge, and therefore every possible rectangle.

**Q8. What's the time complexity of Maximal Rectangle, and why?**
O(m × n): building each row's height array is O(n), and running the histogram algorithm on it is also O(n) (Day 41/44's amortized argument), repeated once per row (m rows) — O(n) work, m times, is O(m × n).

**Q9. In Docker Compose, how does the `app` container reach the `db` container, and why won't `localhost` work for that?**
By the service name `db`, which Compose resolves via DNS automatically within the shared network it creates for all services in the file. `localhost` from inside the `app` container refers to the `app` container itself, not to any other service — it's a different network namespace entirely, which is why a `localhost`-wired connection string that worked when everything ran unContainerized on one machine breaks once the app moves into Compose.

**Q10. What does `depends_on` actually guarantee, and what's the practical risk of assuming it guarantees more?**
It only guarantees container *start order* — that `db` starts before `app` attempts to start. It does not wait for Postgres to actually finish initializing and accept connections, so `app` can still fail to connect on a cold start even though `depends_on` was respected; production setups typically add a `healthcheck` and a `condition: service_healthy` to close this gap.

**Q11. Why does the Postgres service need a named volume, but the `app` service doesn't?**
Postgres is stateful — its data must survive the container being removed and recreated, which is exactly what a container's own writable layer does *not* guarantee (Day 43). `app` is stateless (all its state lives in Postgres and Redis), so there's nothing on its own filesystem that needs to persist across restarts.

---

# Daily Deliverable Check

- [ ] Largest Rectangle in Histogram (LC 84) and Maximal Rectangle (LC 85) solved — brute force, and the reduction between them, explained out loud, not just coded — pushed.
- [ ] `docker-compose.yml` written for `app` + `db` (Postgres) + `cache` (Redis).
- [ ] `docker-compose up -d` brings up the full stack, wired together by service name.
- [ ] Postgres data confirmed to survive a `down` / `up -d` cycle via the named volume.
- [ ] 20 minutes of LinkedIn engagement completed.
- [ ] 2 more Tier C companies identified.

---

## What Tomorrow Assumes You Already Know Cold

Day 45 assumes the full monotonic-stack lineage — Day 40–41's invariant and amortized proof, today's index-based boundary arithmetic — is solid enough to extend one more time to Basic Calculator, which adds parentheses on top of Day 43's Basic Calculator II and needs the stack to save and restore state across nested scopes, not just track a running computation. It also assumes today's Compose-based `todo-api` stack is up and stable, since tomorrow's JUnit 5/AssertJ theory starts a fresh, independent thread rather than building further on Docker.
