# Week 10 (Revised): Backtracking Completes, Graphs Begin, and the Flagship Platform Grows Real Microservices Patterns

**What changed:** Backtracking closes at 12 problems (up from 10) — Combination Sum II joins as the natural duplicate-handling companion to Combination Sum. Graphs BFS/DFS begins its run toward 12 problems (up from 9). On the platform side, this is where Resilience4j, Feign, and the Gateway get built — and because Feign now calls a real Product module that already exists in the same platform instead of a mocked stand-in, it's a more honest exercise than the original plan's version of this same work.

---

## Day 64 — Backtracking: Combination Sum and Its Duplicate-Handling Twin, and Resilience4j

### DSA Block (2.5 hrs)
- Problem 5: Combination Sum — LeetCode #39 — Medium — Pattern: Backtracking
  - Hint: since elements can repeat, when you include one, recurse *without* advancing the index; only advance when you choose not to use the current element.
  - Complexity: Time O(2ⁿ) | Space O(n)
- Problem 6: Combination Sum II — LeetCode #40 — Medium — Pattern: Backtracking with Duplicate Handling **(new)**
  - Hint: sort first. Each number can only be used once this time, so skip a choice at the same recursion depth if it equals the previous choice and the previous one wasn't included in this branch — the exact duplicate-avoidance trick you'll reuse on Subsets II.
  - Complexity: Time O(2ⁿ) | Space O(n)

### Theory Block (2 hrs)
- Topic: Circuit Breakers with Resilience4j
- A system built from multiple services has a specific failure mode a single monolith never encounters: one slow or failing downstream service can exhaust the caller's own resources (threads, connections) while it waits, and that exhaustion then cascades to whatever was calling *the caller*. A Circuit Breaker watches the failure rate of calls to a dependency and, once it crosses a threshold, "opens" — it stops attempting the real call entirely and immediately returns a fallback (or fails fast), giving the struggling dependency room to recover instead of being hit with an ever-growing pile of waiting requests.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: create a standalone endpoint in the Order module that mimics a slow third-party API (`Thread.sleep`). Add `@CircuitBreaker` to a service method calling it, configured to open after 3 failures with a fallback response.
- Definition of done: after 3 timeouts, the circuit opens and subsequent calls immediately return the fallback.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: apply to 2 more Tier C companies.

### Daily Deliverable
- [ ] Combination Sum and Combination Sum II solved, pushed to `dsa-java/backtracking/`.
- [ ] Circuit breaker live, verified opening and falling back correctly.

---

## Day 65 — Backtracking: Phone Letters and Parentheses, and Feign Clients

### DSA Block (2.5 hrs)
- Problem 7: Letter Combinations of a Phone Number — LeetCode #17 — Medium — Pattern: Backtracking
  - Hint: map each digit to its letters; recurse one digit deeper each call, trying every letter for the current digit.
  - Complexity: Time O(4ⁿ) | Space O(n)
- Problem 8: Generate Parentheses — LeetCode #22 — Medium — Pattern: Backtracking
  - Hint: track open and close counts; add `(` whenever `open < n`, add `)` whenever `close < open`.
  - Complexity: Time O(4ⁿ/√n) | Space O(n)

### Theory Block (2 hrs)
- Topic: Feign Clients for Service-to-Service Communication
- Feign gives you declarative, interface-based HTTP calls between services — you write an interface describing what the call looks like, and a working HTTP client is generated for you underneath. A `@FeignClient` interface with method signatures and mapping annotations replaces the boilerplate of manually building an HTTP request every time one part of your system needs to talk to another.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: define a `ProductServiceClient` `@FeignClient` interface in the Order module, calling the real Product module — since both already exist in the same platform, this is a genuine working call, not a call to a placeholder URL. Add a fallback for when the Product module is unreachable.
- Definition of done: the Feign client compiles and correctly fetches real product data; the fallback returns a placeholder response when the target is unreachable (test this by temporarily stopping the Product module).

### Career Block (1 hr)
- LinkedIn: Post 14 — "Declarative HTTP clients: how Feign saves you from writing boilerplate REST calls."
- Networking: identify 5 Target companies (Tier C) for early interview practice.

### Daily Deliverable
- [ ] Letter Combinations of a Phone Number and Generate Parentheses solved, pushed.
- [ ] `ProductServiceClient` Feign interface with fallback pushed, calling a real module.

---

## Day 66 — Backtracking on Grids and Palindromes, and the API Gateway

### DSA Block (2.5 hrs)
- Problem 9: Word Search — LeetCode #79 — Medium — Pattern: Matrix Backtracking
  - Hint: standard DFS from each cell; mark a cell visited by temporarily overwriting its character, then restore it when backtracking out.
  - Complexity: Time O(m×n×4ˡ) | Space O(l)
- Problem 10: Palindrome Partitioning — LeetCode #131 — Medium — Pattern: Backtracking
  - Hint: at each position, try every prefix that's a palindrome, recurse on the remainder, backtrack if no valid partition follows.
  - Complexity: Time O(n×2ⁿ) | Space O(n)

### Theory Block (2 hrs)
- Topic: Spring Cloud Gateway
- A gateway is the single entry point every external request passes through before reaching any internal service. Centralizing routing, and often authentication and rate limiting, here means that logic exists once instead of being duplicated inside every individual service.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: set up Spring Cloud Gateway in front of the whole platform; configure routes forwarding `/products/**`, `/orders/**`, and `/payments/**` to their respective modules.
- Definition of done: hitting the Gateway correctly routes to each underlying module.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: follow up on Tier C applications.

### Daily Deliverable
- [ ] Word Search and Palindrome Partitioning solved, pushed.
- [ ] Gateway routing correctly to all three modules.

---

## Day 67 — Backtracking Capstone, and JWT at the Gateway

### DSA Block (2.5 hrs)
- Problem 11: Subsets II — LeetCode #90 — Medium — Pattern: Backtracking with Duplicate Handling
  - Hint: sort first; skip an element at the same recursion depth if it equals the previous one *and* the previous one wasn't included — the standard duplicate-avoidance trick, same one Combination Sum II used.
  - Complexity: Time O(n×2ⁿ) | Space O(n)
- Problem 12: N-Queens — LeetCode #51 — Hard — Pattern: Backtracking
  - Hint: place queens row by row; maintain HashSets for used columns and the two diagonal directions (`row+col`, `row-col`) to check validity in O(1).
  - Complexity: Time O(n!) | Space O(n)

**This closes Backtracking: 12 problems, warm-up through Hard — up from 10 in the original plan.**

### Theory Block (2 hrs)
- Topic: JWT Authentication at the Gateway
- Users authenticate once and receive a signed JWT; the Gateway validates the signature and expiry on every request *before* routing downstream, so individual modules don't each need to re-implement auth checks. This is custom JWT issuance, not OAuth 2.0 — OAuth 2.0 is a separate authorization *framework* that can happen to issue JWTs; they're complementary, not the same thing, and interviewers regularly probe this exact distinction.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: implement JWT validation at the Gateway level.
- Definition of done: requests without a valid Bearer token are rejected with 401 before reaching any module.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: research the hiring manager for a role you've applied to and send a direct message.

### Daily Deliverable
- [ ] Subsets II and N-Queens solved — Backtracking ladder complete at 12 problems.
- [ ] JWT validation live at the Gateway, verified rejecting unauthenticated requests.

---

## Day 68 — Graphs Begin, and Rate Limiting

### DSA Block (2.5 hrs)

**Concept Card — Graphs (BFS/DFS)**
- What: nodes connected by edges — trees are graphs with no cycles and exactly one path between any two nodes, so tree DFS/BFS transfers directly, plus you now need a `visited` set since graphs can cycle.
- Why: BFS explores level by level (shortest path in an *unweighted* graph); DFS explores as deep as possible (exhaustive search, cycle detection, connectivity).
- Where: adjacency list (`Map<Node, List<Node>>` — efficient for sparse graphs, most real ones) vs. adjacency matrix (2D array, O(V²) space). A grid is a graph in disguise.
- Interview signal: "connected components," "shortest path unweighted" → BFS. "Does a path exist," "explore fully" → DFS.

- Problem 1: Number of Islands — LeetCode #200 — Medium — Pattern: Matrix DFS/BFS
  - Hint: scan the grid; on an unvisited `1`, increment your island count and DFS/BFS to mark every connected `1` as visited.
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 2: Max Area of Island — LeetCode #695 — Medium — Pattern: Matrix DFS
  - Hint: same scaffold as Number of Islands, but the DFS returns the count of cells visited — track the max.
  - Complexity: Time O(m×n) | Space O(m×n)

### Theory Block (2 hrs)
- Topic: Rate Limiting — Token Bucket
- A token bucket refills at a fixed rate and holds a maximum capacity; each request consumes a token, and requests are rejected (or queued) once the bucket is empty — allowing controlled bursts up to the bucket size while enforcing a long-run average rate.
- Coding exercise: implement a basic Token Bucket algorithm locally in Java.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: apply a request rate limiter filter at the Gateway using the Token Bucket logic above (in-memory for now — this gets upgraded to a distributed, Redis-backed version once the HLD phase covers why in-memory isn't enough with multiple Gateway instances).
- Definition of done: the Gateway returns HTTP 429 once the limit is exceeded.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: track application responses; reach out to internal recruiters for roles applied to last week.

### Daily Deliverable
- [ ] Number of Islands and Max Area of Island solved, pushed to `dsa-java/graphs/`.
- [ ] Rate limiter returning 429 at the Gateway once tripped.

---

## Day 69 — Graph Traversal Continues, and Kafka Schema Registry

### DSA Block (2.5 hrs)
- Problem 3: Clone Graph — LeetCode #133 — Medium — Pattern: Graph DFS + HashMap
  - Hint: `HashMap<Node, Node>` maps originals to clones; if a node's already mapped when visited, return the existing clone — this handles cycles correctly.
  - Complexity: Time O(V+E) | Space O(V)
- Problem 4: Is Graph Bipartite? — LeetCode #785 — Medium — Pattern: Graph 2-Coloring **(new)**
  - Hint: try to color the graph with 2 colors via BFS or DFS, alternating colors between neighbors. If any edge connects two same-colored nodes, it isn't bipartite.
  - Complexity: Time O(V+E) | Space O(V)

### Theory Block (2 hrs)
- Topic: Kafka Schema Registry
- As modules evolve independently, an unversioned message format breaks silently the moment one module changes its shape. Schema Registry enforces a versioned contract (Avro is the common serialization format) — producers and consumers agree on a schema, and compatibility rules (backward/forward) prevent breaking changes from shipping unnoticed.
- Coding exercise: define a simple `.avsc` Avro schema for an `OrderEvent`.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: define an `Order` entity and `OrderRepository`; add the Confluent Kafka Schema Registry dependency and configure Avro serializers.
- Definition of done: `Order` entity compiles and is tested with `@DataJpaTest`; the `.avsc` schema is committed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: follow up on any recruiter responses from this week.

### Daily Deliverable
- [ ] Clone Graph and Is Graph Bipartite? solved, pushed.
- [ ] `Order` entity/repository and Avro schema committed.

---

## Day 70 (Sunday) — Consolidation, Multi-Source BFS, and Saga Choreography

### Self-Check (15 min)
- [ ] Solve one Backtracking problem from earlier this week cold, without hints.

### DSA Block (2 hrs)
- Problem 5: Rotting Oranges — LeetCode #994 — Medium — Pattern: Multi-source BFS
  - Hint: start BFS from *every* rotten orange simultaneously, tracking elapsed minutes level by level.
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 6: All Paths From Source to Target — LeetCode #797 — Medium — Pattern: DFS on a DAG **(new)**
  - Hint: no visited set needed — it's a DAG, so no cycles exist. DFS from node 0, adding the current path to your result whenever you reach the last node.
  - Complexity: Time O(2ⁿ×n) | Space O(n)

### Theory Block (1 hr)
- Topic: Saga Choreography
- When an Order is created, emit `OrderCreatedEvent`; a Payment listener reacts, and on success emits `PaymentSucceededEvent`; the Order module listens to that and updates status to `COMPLETED` — a distributed transaction completed asynchronously via events, with no 2PC anywhere. Because the Payment module is real (not a dummy listener in a throwaway repo), this Saga flow is genuinely end-to-end.
- Coding exercise: none — the project below is the exercise.

### Project Block (1 hr)
- Repository: `scalable-ecommerce-platform`.
- Task: implement the Saga choreography flow above, using the real Payment module.
- Definition of done: creating an order eventually flips its status to `COMPLETED` asynchronously via the Kafka event chain, between two real modules.

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 70, ten weeks in, **137 total DSA problems solved.** Backtracking fully closed at 12 (up from 10). Graphs underway with 6 of its expanded 12-problem set done. The platform now has Resilience4j, real (not mocked) Feign calls, a Gateway with JWT and rate limiting, Kafka with Schema Registry, and a genuine Saga flow between two real modules — everything `order-management-api` was going to teach, now built directly into what will actually be your capstone.

### Daily Deliverable
- [ ] Rotting Oranges and All Paths From Source to Target solved, pushed.
- [ ] Saga choreography flow completing orders asynchronously between real modules.
- [ ] Weekly ritual and scorecard complete.
