# Week 7 (Revised): Stacks Completes (12 Problems), Trees Begins, Docker and Testing

**What changed:** Stacks/Monotonic Stack closes at 12 problems (up from 10) — Maximal Rectangle joins as the natural Hard-tier extension of Largest Rectangle in Histogram. Trees begins its expanded run toward 15 problems (up from 11), including two genuinely valuable additions (Balanced Binary Tree, and later Lowest Common Ancestor of a general Binary Tree and Serialize/Deserialize) and — a few days from now — the Median of Two Sorted Arrays fix.

---

## Day 43 — Stack Simulation Problems, and Docker Basics

### DSA Block (2.5 hrs)
- Problem 8: Asteroid Collision — LeetCode #735 — Medium — Pattern: Stack Simulation
  - Hint: push right-moving asteroids; when a left-moving one arrives, resolve collisions against the stack top until one side is destroyed or they're compatible.
  - Complexity: Time O(n) | Space O(n)
- Problem 9: Basic Calculator II — LeetCode #227 — Medium — Pattern: Stack
  - Hint: track a `lastSign`; on `+`/`-`, push the signed number; on `*`/`/`, pop, compute, push back.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: Docker Basics
- An image is a static, layered snapshot; a container is a running instance of it. `Dockerfile` defines the build, layer by layer, with caching benefits when ordered least-to-most frequently changing — Docker only rebuilds a layer (and everything after it) if something in that layer actually changed, which is why dependency-installation steps belong near the top of a `Dockerfile`, before you copy in source code that changes on every commit.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: write a multi-stage `Dockerfile` (Stage 1: Maven build, Stage 2: slim JRE runtime). Build and run it.
- Definition of done: `docker run -p 8080:8080 todo-api` serves requests correctly.

### Career Block (1 hr)
- LinkedIn: Post 10 — "Docker multi-stage builds cut my image size by X%" (measure it).
- Networking: apply to 2 Tier C ("practice") companies.

### Daily Deliverable
- [ ] Asteroid Collision and Basic Calculator II solved, pushed to `dsa-java/stacks/`.
- [ ] `todo-api` running correctly inside Docker. LinkedIn Post 10 published.

---

## Day 44 — Stacks: The Hard Tier, and Docker Compose

### DSA Block (2.5 hrs)
- Problem 10: Largest Rectangle in Histogram — LeetCode #84 — Hard — Pattern: Monotonic Stack
  - Hint: maintain a stack of strictly increasing heights. A shorter bar means you pop and compute area for the popped height — you've found its right boundary; the new top is its left boundary.
  - Complexity: Time O(n) | Space O(n)
- Problem 11: Maximal Rectangle — LeetCode #85 — Hard — Pattern: Monotonic Stack, applied row by row **(new)**
  - Hint: for each row of a binary matrix, compute a "histogram height" array (how many consecutive 1s stack up above this cell) and run yesterday's Largest Rectangle in Histogram on it. The 2D problem collapses into the 1D one, once per row.
  - Complexity: Time O(m×n) | Space O(n)

### Theory Block (2 hrs)
- Topic: Docker Compose
- One `docker-compose.yml` can orchestrate multiple services (your app, Postgres, Redis) with networking handled automatically by service name — containers on the same Compose network can reach each other using the service name as a hostname, with no manual IP configuration needed.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: `docker-compose.yml` running your app, PostgreSQL, and Redis together.
- Definition of done: `docker-compose up -d` brings up the full stack, wired together.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 2 more Tier C companies for early practice applications.

### Daily Deliverable
- [ ] Largest Rectangle in Histogram and Maximal Rectangle solved, pushed.
- [ ] `docker-compose up -d` brings up the full stack.

---

## Day 45 — Stacks Capstone, and JUnit 5/AssertJ

### DSA Block (2 hrs)
- Problem 12: Basic Calculator — LeetCode #224 — Hard — Pattern: Stack
  - Hint: use a stack to hold the running sum and sign whenever you hit `(`, so you can restore context after the matching `)`.
  - Complexity: Time O(n) | Space O(n)

**This closes Stacks/Monotonic Stack: 12 problems, Easy through Hard — up from 10 in the original plan.**

### Theory Block (2 hrs)
- Topic: JUnit 5 and AssertJ
- `@Test`, `@BeforeEach`/`@AfterEach`, `@ParameterizedTest`. AssertJ's fluent assertions (`assertThat(x).isEqualTo(y)`) chain naturally for multi-condition checks, and read closer to a plain-English description of what's being verified than JUnit's built-in assertions do.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: JUnit 5 tests for `TaskController` using `@DataJpaTest` against embedded H2, with AssertJ assertions.
- Definition of done: `mvn test` passes.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 5 Target Tier C companies for early interview practice.

### Daily Deliverable
- [ ] Basic Calculator solved — Stacks/Monotonic Stack complete at 12 problems.
- [ ] `@DataJpaTest` suite passing.

---

## Day 46 — Trees Begin, and Mockito

### DSA Block (2.5 hrs)

**Concept Card — Binary Trees**
- What: a node with at most two children, `left` and `right`. Nearly every tree problem is recursion applied to this shape — you already have recursion fundamentals from Week 2.
- Why: hierarchical data (file systems, decision structures), and where recursive thinking gets real, unavoidable practice.
- Interview signal: anything phrased as a tree or "hierarchical structure." Universal first move: define what a helper returns for a `null` node (base case), then combine `left`/`right` results.
- Prerequisites: recursion ✅, self-referential classes ✅ (linked lists already used this shape).

- Problem 1: Maximum Depth of Binary Tree — LeetCode #104 — Easy — Pattern: DFS
  - Hint: base case — `null` returns 0. Recursive case — `1 + max(depth(left), depth(right))`.
  - Complexity: Time O(n) | Space O(n)
- Problem 2: Invert Binary Tree — LeetCode #226 — Easy — Pattern: DFS
  - Hint: swap `left` and `right`, then recursively invert each subtree.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: Mockito
- `@Mock` creates a fake dependency implementation; `@InjectMocks` wires mocks into the class under test — unit-test `TaskService`'s logic without touching a real database, and without the test's runtime depending on a database being up and reachable.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: unit test for `TaskService`, mocking `TaskRepository`.
- Definition of done: the test passes in milliseconds — proof the database layer is fully mocked.

### Career Block (1 hr)
- LinkedIn: Post 11 — BFS vs DFS on trees, with a small diagram.
- Networking: send 5 connection requests to Senior Engineers at Tier A companies.

### Daily Deliverable
- [ ] Maximum Depth of Binary Tree and Invert Binary Tree solved, pushed to `dsa-java/trees/`.
- [ ] Mockito-based `TaskService` test passing. LinkedIn Post 11 published.

---

## Day 47 — Tree DFS Continues, and TestContainers

### DSA Block (2.5 hrs)
- Problem 3: Same Tree — LeetCode #100 — Easy — Pattern: DFS (two-tree comparison)
  - Hint: two trees are the same if both roots are null, or both non-null with equal values and recursively identical subtrees.
  - Complexity: Time O(n) | Space O(n)
- Problem 4: Symmetric Tree — LeetCode #101 — Easy — Pattern: DFS (mirrored comparison)
  - Hint: a tree is symmetric if its left subtree mirrors its right — compare `left.left` against `right.right` and `left.right` against `right.left`.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: TestContainers
- H2 is fast but doesn't support Postgres-specific features — a passing H2 test doesn't guarantee correctness against real Postgres. TestContainers spins up real Docker containers for integration tests.
- Coding exercise: write a JUnit test annotated `@Testcontainers` that spins up a generic PostgreSQL container and asserts it's running.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: replace H2 with TestContainers for PostgreSQL in your `@DataJpaTest` suite, using `@DynamicPropertySource`.
- Definition of done: `mvn test` spins up a real Postgres container and runs tests against it.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: review 3 engineering manager profiles for outreach tomorrow.

### Daily Deliverable
- [ ] Same Tree and Symmetric Tree solved, pushed.
- [ ] `todo-api` tests running against real TestContainers Postgres.

---

## Day 48 — Tree DFS with Global State, and Kafka Fundamentals

### DSA Block (2.5 hrs)
- Problem 5: Balanced Binary Tree — LeetCode #110 — Easy — Pattern: DFS, bottom-up height check **(new)**
  - Hint: have the recursive helper return `-1` as a sentinel the instant an imbalance is found anywhere below, short-circuiting the rest of the tree instead of wastefully continuing to compute heights.
  - Complexity: Time O(n) | Space O(n)
- Problem 6: Diameter of Binary Tree — LeetCode #543 — Easy — Pattern: DFS with a global max
  - Hint: the diameter *through* a node is `leftDepth + rightDepth`; track a running max separately from the depth value returned upward.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: Kafka Fundamentals
- Topics are append-only logs split into partitions for parallelism; producers write, consumers read, consumer groups split the work across instances. Sequential disk appends drive the high throughput.
- Coding exercise: run Kafka via Docker; use the CLI to create a topic, start a console producer, start a console consumer, watch messages arrive.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: add Spring Kafka. Configure a producer publishing a `TaskCreatedEvent` when a task is created.
- Definition of done: creating a task via the API produces a message visible in the console consumer.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: send connection requests to the 3 EMs reviewed yesterday.

### Daily Deliverable
- [ ] Balanced Binary Tree and Diameter of Binary Tree solved, pushed.
- [ ] Kafka producer live in `todo-api`.

---

## Day 49 (Sunday) — Consolidation, and Tree BFS

### Self-Check (15 min)
- [ ] Solve one Stack problem from earlier this week cold, without hints.

### DSA Block (2 hrs)
- Problem 7: Binary Tree Level Order Traversal — LeetCode #102 — Medium — Pattern: BFS
  - Hint: use a Queue; capture `queue.size()` before the inner loop to process exactly one level at a time.
  - Complexity: Time O(n) | Space O(n)
- Problem 8: Binary Tree Right Side View — LeetCode #199 — Medium — Pattern: BFS
  - Hint: same level-order scaffold, but only keep the last node visited at each level.
  - Complexity: Time O(n) | Space O(n)

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 49, seven weeks in, **97 total DSA problems solved.** Stacks/Monotonic Stack fully closed at 12 (up from 10). Trees underway with 8 of its expanded 15-problem set done, including BFS now covered. Docker, Docker Compose, JUnit/AssertJ/Mockito/TestContainers, and Kafka basics all live in `todo-api`.

### Daily Deliverable
- [ ] Level Order Traversal and Right Side View solved, pushed.
- [ ] Weekly ritual and scorecard complete.
