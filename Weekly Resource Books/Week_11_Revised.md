# Week 11 (Revised): Graphs Completes (12 Problems), Union-Find Begins, Networking and AWS Fundamentals

**What changed:** Graphs BFS/DFS closes at 12 problems (up from 9). Union-Find begins its run toward 7 problems (up from 3–4) — this was one of the thinnest patterns identified in the original audit.

---

## Day 71 — Graph Traversal Continues, and Topological Sort

### DSA Block (2.5 hrs)

**Concept Card — Topological Sort**
- What: an ordering of a directed acyclic graph's nodes such that every edge points from an earlier node to a later one.
- Why: any "what must happen before what" problem — build systems, task scheduling with dependencies, course prerequisites.
- Interview signal: "prerequisites," "dependencies," "build order" (and doubles as cycle detection — no valid ordering means a cycle exists).

- Problem 7: Course Schedule — LeetCode #207 — Medium — Pattern: Topological Sort / Cycle Detection
  - Hint: build the prerequisite graph; if Kahn's algorithm can't process every node, a cycle exists and completion is impossible.
  - Complexity: Time O(V+E) | Space O(V+E)
- Problem 8: Course Schedule II — LeetCode #210 — Medium — Pattern: Topological Sort
  - Hint: identical to Course Schedule, but record and return the actual processing order.
  - Complexity: Time O(V+E) | Space O(V+E)

### Theory Block (2 hrs)
- Topic: Networking Fundamentals I — TCP/UDP and DNS
- TCP is connection-oriented: before any data moves, both sides complete a handshake, and every byte sent is acknowledged, retransmitted if lost, and reassembled in order. UDP skips all of that — it's connectionless and fire-and-forget, which makes it faster but tolerant only of applications that can handle occasional loss (video streaming, for example). DNS resolves a hostname to an IP address through a hierarchy: your resolver asks a root server, which points to a top-level-domain server, which points to the authoritative server for that specific domain — and the answer gets cached aggressively at every layer along the way, since re-doing this lookup for every single request would be far too slow.
- Coding exercise: none — write out, step by step, what happens between typing a URL into a browser and that browser having an actual IP address to connect to.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: none new — use this slot to review the Gateway, JWT, and rate-limiting work from last week and make sure it's clean and well-tested before new features land on top of it.

### Career Block (1 hr)
- LinkedIn: Post 15 — Topological sort explained via "which courses can you actually take."
- Networking: identify 3 Target Tier B companies known for platform/infra engineering.

### Daily Deliverable
- [ ] Course Schedule and Course Schedule II solved, pushed to `dsa-java/graphs/`.
- [ ] Can explain TCP vs. UDP and DNS resolution without notes.

---

## Day 72 — Multi-Source BFS (Reversed), and Networking Fundamentals II

### DSA Block (2.5 hrs)
- Problem 9: Surrounded Regions — LeetCode #130 — Medium — Pattern: Multi-source DFS/BFS from the border
  - Hint: any `'O'` connected to the border can never be surrounded — mark those first (DFS/BFS from every border `'O'`), then flip everything else.
  - Complexity: Time O(m×n) | Space O(m×n)
- Problem 10: Pacific Atlantic Water Flow — LeetCode #417 — Medium — Pattern: Multi-source BFS/DFS (reversed)
  - Hint: start from the ocean borders and traverse *uphill* — cells reachable from both border-searches flow to both oceans.
  - Complexity: Time O(m×n) | Space O(m×n)

### Theory Block (2 hrs)
- Topic: Networking Fundamentals II — HTTP and Load Balancers
- HTTP is a request/response protocol built on top of TCP; HTTPS adds TLS, negotiating an encrypted channel before any actual HTTP data flows. A load balancer sits in front of multiple servers and distributes incoming traffic across them — Layer 4 balancers work at the IP/port level (fast, but blind to what's actually in the request), while Layer 7 balancers read the HTTP content itself and can route based on it (more overhead, more flexibility). Health checks are what let a load balancer stop sending traffic to an instance that's stopped responding correctly, without a human having to intervene.
- Coding exercise: none — write 150 words comparing when you'd choose an L4 load balancer over an L7 one for the Gateway you've already built.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: none new — the writing exercise above is today's task.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: send connection requests to the 3 infra-focused targets from yesterday.

### Daily Deliverable
- [ ] Surrounded Regions and Pacific Atlantic Water Flow solved, pushed.
- [ ] L4 vs. L7 load balancer comparison written.

---

## Day 73 — Graphs Capstone, and Shortest Path Foundations

### DSA Block (2.5 hrs)
- Problem 11: Word Ladder — LeetCode #127 — Hard — Pattern: BFS for Shortest Path (Unweighted)
  - Hint: shortest path in an unweighted graph is always BFS; generate every one-letter mutation of the current word and check membership in the word set.
  - Complexity: Time O(m²×n) | Space O(m²×n)
- Problem 12: Find the City With the Smallest Number of Neighbors at a Threshold Distance — LeetCode #1334 — Medium — Pattern: Floyd-Warshall (all-pairs shortest path) **(new)**
  - Hint: with a small enough number of cities, computing shortest paths between *every* pair at once (`dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j])` for every intermediate `k`) is simpler than running a single-source algorithm once per city. This is your one deliberate, light exposure to Floyd-Warshall — not a pattern to over-invest in, but worth recognizing by name.
  - Complexity: Time O(V³) | Space O(V²)

**This closes Graphs BFS/DFS: 12 problems — up from 9 in the original plan.**

### Theory Block (1 hr)
- Topic: Graphs, Reviewed — When BFS Beats DFS, and When Neither Is Enough
- Write down, in one sentence each: which of this week's problems needed BFS specifically because they asked for a *shortest* path or *minimum steps*, and which just needed *any* valid path or *full* exploration (DFS territory). Then note that today's Floyd-Warshall problem needed neither on its own — it's a different algorithm family entirely, for a different question ("shortest path between every pair," not "shortest path from one source").

### Project Block (1 hr)
- Repository: none new — make sure all 12 Graph solutions are pushed and organized in `dsa-java/graphs/`.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: review 3 more engineering manager profiles.

### Daily Deliverable
- [ ] Word Ladder and Find the City With the Smallest Number of Neighbors at a Threshold Distance solved — Graphs ladder complete at 12 problems.
- [ ] BFS vs. DFS reflection written.

---

## Day 74 — Union-Find Begins, and AWS Networking Fundamentals

### DSA Block (2.5 hrs)

**Concept Card — Union-Find (Disjoint Set Union)**
- What: tracks disjoint sets, supporting `find` (which set does this element belong to) and `union` (merge two sets), both nearly O(1) with two optimizations: path compression (flatten the structure during `find`, so future lookups are faster) and union by rank/size (always attach the smaller tree under the larger, keeping trees shallow).
- Why: whenever a problem asks "are these two things connected, directly or transitively," Union-Find answers it faster and more simply than re-running BFS/DFS from scratch every time a new connection is added.
- Interview signal: "are these connected," "how many groups/provinces," "does adding this edge create a cycle."
- Java shape: a `parent[]` array (each index initially its own parent), ideally with a `rank[]` array too.

- Problem 1: Redundant Connection — LeetCode #684 — Medium — Pattern: Union-Find
  - Hint: process edges in order; the first edge whose endpoints are already in the same set is the redundant one.
  - Complexity: Time O(n×α(n)) | Space O(n)
- Problem 2: Number of Provinces — LeetCode #547 — Medium — Pattern: Union-Find / DFS
  - Hint: start with every city in its own set; union connected cities per the matrix; the answer is however many distinct sets remain.
  - Complexity: Time O(n²) | Space O(n)

### Theory Block (2 hrs)
- Topic: AWS Fundamentals — Networking
- A VPC (Virtual Private Cloud) is your own isolated network within a cloud provider's infrastructure. Public subnets route out to the internet through an Internet Gateway; private subnets can only reach the internet outbound, through a NAT Gateway, and can't be reached directly from outside. Security Groups act as a stateful firewall attached to individual instances — "stateful" meaning a response to an allowed outbound request is automatically allowed back in, without needing a separate inbound rule for it.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: diagram an AWS deployment for the platform — EC2 instances in a public subnet behind the Gateway, RDS in a private subnet, Security Groups restricting the database to only the application instances.
- Definition of done: a labeled diagram covering the VPC, both subnet types, both gateway types, and Security Group rules.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: apply to 2 Tier B companies.

### Daily Deliverable
- [ ] Redundant Connection and Number of Provinces solved, pushed to `dsa-java/union-find/`.
- [ ] AWS deployment diagram complete.

---

## Day 75 — Union-Find: Real-World Constraint Problems, and AWS IAM

### DSA Block (2.5 hrs)
- Problem 3: Satisfiability of Equality Equations — LeetCode #990 — Medium — Pattern: Union-Find **(new)**
  - Hint: process every `"=="` equation first, unioning both sides. Then check every `"!="` equation — if both sides end up in the same set after the unions, the constraints are unsatisfiable.
  - Complexity: Time O(n×α(n)) | Space O(n)
- Problem 4: Most Stones Removed with Same Row or Column — LeetCode #947 — Medium — Pattern: Union-Find **(new)**
  - Hint: union stones sharing a row or column (using a combined index space, e.g. row indices and column indices offset so they don't collide). The answer is `total stones - number of connected components`, since you can always remove all but one stone per component.
  - Complexity: Time O(n×α(n)) | Space O(n)

### Theory Block (2 hrs)
- Topic: AWS IAM, and S3 vs. EBS
- IAM Roles are assumed temporarily by a resource (like an EC2 instance) and don't have permanent credentials; IAM Users are persistent identities with their own long-lived credentials. The principle of least privilege means granting only the specific permissions something actually needs, nothing more. S3 is object storage — flat, accessed over HTTP, effectively unlimited in scale, good for files, backups, and static assets. EBS is block storage attached to exactly one EC2 instance at a time — a virtual hard drive, the kind of thing a database's actual data files would live on.
- Coding exercise: write a minimal IAM policy JSON granting read-only access to a specific S3 bucket.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: the IAM policy JSON above, saved as `iam-policy.json` with a comment explaining each permission.
- Definition of done: pushed.

### Career Block (1 hr)
- LinkedIn: Post 16 — "Why BFS gives the wrong answer on a weighted graph" (a preview of tomorrow's Dijkstra territory, written a day ahead).
- Networking: apply to 2 Tier B companies.

### Daily Deliverable
- [ ] Satisfiability of Equality Equations and Most Stones Removed with Same Row or Column solved, pushed.
- [ ] `iam-policy.json` pushed.

---

## Day 76 — Union-Find: String Grouping, and Observability with Prometheus

### DSA Block (2.5 hrs)
- Problem 5: Smallest String With Swaps — LeetCode #1202 — Medium — Pattern: Union-Find **(new)**
  - Hint: union every index pair that's allowed to swap. Within each resulting connected component, you can rearrange characters into any order — so sort the characters belonging to each component and place them back at the component's sorted index positions.
  - Complexity: Time O(n log n × α(n)) | Space O(n)
- Problem 6: Accounts Merge — LeetCode #721 — Medium — Pattern: Union-Find + HashMap
  - Hint: map every email to an integer ID; union IDs sharing an account, then group all emails by their set's root ID.
  - Complexity: Time O(n×k×α) | Space O(n×k)

### Theory Block (2 hrs)
- Topic: Observability — Prometheus and Micrometer
- Prometheus scrapes metrics from your application on a schedule — it's pull-based, meaning Prometheus itself reaches out and asks each service for its current metrics, rather than services pushing metrics out. Micrometer is the library that exposes those metrics from a Java application in a format Prometheus understands, via Spring Boot Actuator's `/actuator/prometheus` endpoint.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: add Actuator and the Micrometer Prometheus registry; expose `/actuator/prometheus` across the platform's modules.
- Definition of done: the endpoint returns real metrics in Prometheus's text format for each module.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: no specific outreach today.

### Daily Deliverable
- [ ] Smallest String With Swaps and Accounts Merge solved, pushed.
- [ ] `/actuator/prometheus` live with real metrics.

---

## Day 77 (Sunday) — Union-Find Capstone, Minimum Spanning Trees, and Grafana

### Self-Check (15 min)
- [ ] Solve one Graph problem from earlier this week cold, without hints.

### DSA Block (2 hrs)
- Problem 7: Graph Valid Tree — LeetCode #261 — Medium — Pattern: Union-Find **(new — LeetCode Premium in some regions)**
  - Hint: a graph is a valid tree if and only if it has exactly `n-1` edges and forms exactly one connected component — union every edge, and check both conditions (edge count, and that every node ends up in the same set).
  - Complexity: Time O(n×α(n)) | Space O(n)

**This closes Union-Find: 7 problems — up from 3–4 in the original plan, one of the thinnest gaps identified in the audit.**

- Topic: Minimum Spanning Trees — Kruskal's and Prim's
- A Minimum Spanning Tree connects every node in a graph with the minimum possible total edge weight, with no cycles. Kruskal's algorithm sorts every edge by weight and greedily adds each one, skipping any edge that would create a cycle — checking "would this create a cycle" is exactly what your `UnionFind` class from Day 74 answers in near-O(1). Prim's algorithm instead grows a single tree outward from one starting node, always adding the cheapest edge that connects the current tree to a new node, using a min-heap to find that cheapest edge efficiently.
- Exercise: trace Kruskal's by hand on a small 5-node weighted graph you sketch yourself, then implement it using your existing `UnionFind` class to check each candidate edge.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: implement Kruskal's using your `UnionFind` class on a small hardcoded weighted graph. Separately, run Prometheus and Grafana locally via Docker Compose pointed at the platform, and build one dashboard panel showing HTTP request rate.
- Definition of done: Kruskal's prints the selected MST edges and total weight; Grafana dashboard is live locally.

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 77, eleven weeks in, **150 total DSA problems solved.** Graphs fully closed at 12 (up from 9). Union-Find fully closed at 7 — up from 3–4, one of the two thinnest gaps the original audit found (Dijkstra is the other, starting next). AWS networking/IAM and Prometheus/Grafana are both live on the platform.

### Daily Deliverable
- [ ] Graph Valid Tree solved — Union-Find ladder complete at 7 problems.
- [ ] Kruskal's implementation pushed. Grafana dashboard live.
- [ ] Weekly ritual and scorecard complete.
