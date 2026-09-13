# Week 9 (Revised): Heaps Completes, Tries Completes, Backtracking Begins, and the Flagship Platform Starts

**What changed:** Heaps closes at 10 problems (up from 6) — genuinely the biggest single fix from the audit's "thin pattern" list. Tries closes its core 6 problems here (7th, Maximum XOR of Two Numbers, still deferred to pair with Bit Manipulation, exactly as the original plan did). And this week, `order-management-api` never gets created — instead, `scalable-ecommerce-platform` initializes now, five weeks earlier than the original plan had it, and absorbs everything `order-management-api` was going to teach directly into itself.

---

## Day 57 — Heaps: Scheduling Patterns

### DSA Block (2.5 hrs)
- Problem 7: Reorganize String — LeetCode #767 — Medium — Pattern: Max-Heap by frequency **(new)**
  - Hint: same shape as Task Scheduler tomorrow — always place the most frequent remaining character, provided it isn't the same as the last one placed.
  - Complexity: Time O(n log k) | Space O(k)
- Problem 8: Task Scheduler — LeetCode #621 — Medium — Pattern: Max-Heap + Queue
  - Hint: a max-heap ordered by remaining task frequency, with a cooldown queue tracking when each type becomes available again.
  - Complexity: Time O(n) | Space O(1)

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: apply to 2 Tier B companies.

### Daily Deliverable
- [ ] Reorganize String and Task Scheduler solved, pushed to `dsa-java/heaps/`.

---

## Day 58 — Heaps Capstone: Two-Heap and Merge Patterns

### DSA Block (2.5 hrs)
- Problem 9: Find Median from Data Stream — LeetCode #295 — Hard — Pattern: Two Heaps
  - Hint: a max-heap for the lower half, a min-heap for the upper half, rebalanced so their sizes never differ by more than 1.
  - Complexity: Time O(log n) insert | Space O(n)
- Problem 10: Merge k Sorted Lists — LeetCode #23 — Hard — Pattern: PriorityQueue with Custom Comparator
  - Hint: insert the head of each list into a min-heap; repeatedly poll the smallest, append it, push its `next` back in.
  - Complexity: Time O(n log k) | Space O(k)

**This closes Heaps: 10 problems, Easy through Hard — up from 6 in the original plan, the single biggest gap fix from the audit.**

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: no specific outreach today.

### Daily Deliverable
- [ ] Find Median from Data Stream and Merge k Sorted Lists solved — Heaps ladder complete at 10 problems.

---

## Day 59 — Tries Begin, and the CAP Theorem

### DSA Block (2.5 hrs)

**Concept Card — Tries (Prefix Trees)**
- What: a tree where each edge represents one character, and root-to-node paths spell out shared prefixes.
- Why: O(m) lookup/insert for a word of length m, regardless of dictionary size — prefix queries (autocomplete) fall out naturally.
- Where: autocomplete, spell-checkers, IP routing tables, and the Search Autocomplete system design question later in this plan.
- Java shape: each `TrieNode` holds `TrieNode[26] children` and `boolean isEndOfWord`.

- Problem 1: Implement Trie (Prefix Tree) — LeetCode #208 — Medium — Pattern: Trie
  - Hint: `insert` walks/creates nodes character by character; `search` walks and checks `isEndOfWord`; `startsWith` walks without checking that flag.
  - Complexity: Time O(m) per op | Space O(n×m)
- Problem 2: Map Sum Pairs — LeetCode #677 — Medium — Pattern: Trie storing cumulative values **(new)**
  - Hint: each node can store a running sum contribution; on insert, walk the path and adjust each node's stored sum by the delta between the new and any previous value for that exact key.
  - Complexity: Time O(m) | Space O(n×m)

### Theory Block (2 hrs)
- Topic: The CAP Theorem
- During a network partition, you must choose between Consistency (every read sees the latest write) and Availability (every request gets a response) — you can't have all three of Consistency, Availability, and Partition tolerance at once during a partition. Outside of a partition, you can have both; the common "CP vs AP" framing is really "C or A *during* a partition." MongoDB defaults CP; Cassandra defaults AP.
- Coding exercise: none — write 150 words on why MongoDB and Cassandra made different defaults, and what application type would prefer each.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: implement Trie-based autocomplete for task titles.
- Definition of done: `/tasks/autocomplete?prefix=X` returns matching titles.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: begin researching System Design architectures at target companies.

### Daily Deliverable
- [ ] Implement Trie and Map Sum Pairs solved, pushed to `dsa-java/tries/`.
- [ ] CAP theorem explanation written. Autocomplete live in `todo-api`.

---

## Day 60 — Tries Continue, and Consistent Hashing

### DSA Block (2.5 hrs)
- Problem 3: Longest Word in Dictionary — LeetCode #720 — Medium — Pattern: Trie + DFS **(new)**
  - Hint: a word only counts if every one of its prefixes is also a complete word in the dictionary — walk the Trie and only descend into children marked `isEndOfWord`.
  - Complexity: Time O(n×m) | Space O(n×m)
- Problem 4: Design Add and Search Words Data Structure — LeetCode #211 — Medium — Pattern: Trie + DFS
  - Hint: `.` matches any character — try all 26 possible children recursively instead of following a single path.
  - Complexity: Time O(m) search | Space O(n×m)

### Theory Block (2 hrs)
- Topic: Consistent Hashing
- Plain hash-based sharding (`hash(key) % N`) remaps almost every key when `N` changes. Consistent hashing places servers and keys on a ring; a key belongs to the next server clockwise, so adding/removing one server only reshuffles nearby keys.
- Coding exercise: implement basic consistent hashing using a `TreeMap<Integer, String>` as the ring; map a key using `treeMap.ceilingKey()`.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `ConsistentHashingDemo` class. Add/remove a server and print how many keys actually moved.
- Definition of done: pushed with the before/after key-movement count.

### Career Block (1 hr)
- LinkedIn: Post 13 — "Why adding a server shouldn't reshuffle your whole cache."
- Networking: identify 3 more Target Tier B companies.

### Daily Deliverable
- [ ] Longest Word in Dictionary and Design Add and Search Words solved, pushed.
- [ ] `ConsistentHashingDemo` pushed with measured key movement. LinkedIn Post 13 published.

---

## Day 61 — Tries Core Complete, Backtracking Begins, and Replication Models

### DSA Block (2.5 hrs)
- Problem 5: Replace Words — LeetCode #648 — Medium — Pattern: Trie
  - Hint: build a Trie from the dictionary roots; walk it for each sentence word until `isEndOfWord`, then replace.
  - Complexity: Time O(n×m) | Space O(n×m)
- Problem 6: Word Search II — LeetCode #212 — Hard — Pattern: Trie + Matrix Backtracking
  - Hint: build a Trie of every word you're searching for, then DFS from every cell — the Trie lets you abandon a path the instant no remaining word shares that prefix.
  - Complexity: Time O(m×n×4ˡ) | Space O(k×l)

**This closes Tries' core set at 6 problems (the 7th, Maximum XOR of Two Numbers, arrives with Bit Manipulation, same as the original plan).**

**Concept Card — Backtracking**
- What: explore a decision tree by making a choice, recursing into it, and undoing the choice when you return.
- Why: many "generate all X" or "find any valid X" problems have no shortcut faster than exploring efficiently with pruning.
- Interview signal: "generate all," "find all valid," "every possible arrangement/combination/permutation."
- Prerequisites: recursion ✅. There's no canonical "Easy" LeetCode backtracking problem — before the real problems, generate every binary string of length 3 by hand on paper: at each position you have two choices, draw the decision tree, 3 levels deep, 8 leaves. That *is* the backtracking shape.

- Problem 1: Subsets — LeetCode #78 — Medium — Pattern: Backtracking
  - Hint: at each element, branch into two paths — include it, or don't — and recurse on both.
  - Complexity: Time O(n×2ⁿ) | Space O(n)

### Theory Block (2 hrs)
- Topic: Replication Models
- Single-leader (simple, but the leader is a bottleneck/single point of failure), multi-leader (needs conflict resolution), leaderless (quorum-based, Dynamo-style). Synchronous replication guarantees durability but adds latency; asynchronous is fast but can lose recent writes on leader failure.
- Coding exercise: none — write 150 words on which replication model fits a banking ledger vs. a social media "like" counter.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: a design-note comment block in `TaskService` on where you'd route reads vs. writes with a leader/replica Postgres setup, and the "read your own writes" risk it introduces.
- Definition of done: pushed as a design note. This is `todo-api`'s last new task — from tomorrow, active project work moves to the flagship platform.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: apply to 1 Tier B target company.

### Daily Deliverable
- [ ] Replace Words and Word Search II solved — Tries core complete at 6.
- [ ] Subsets solved — Backtracking begun.
- [ ] Replication model comparison written.

---

## Day 62 — Backtracking Continues, and the Flagship Platform Initializes

### DSA Block (2.5 hrs)
- Problem 2: Permutations — LeetCode #46 — Medium — Pattern: Backtracking
  - Hint: swap elements into position one at a time, recurse on the remainder, swap back before trying the next option.
  - Complexity: Time O(n×n!) | Space O(n)

### Theory Block (2 hrs)
- Topic: Spring AOP (Aspect-Oriented Programming)
- Aspect-Oriented Programming lets you inject behavior (logging, security, transactions) around existing methods without modifying them — an Aspect defines *what* to inject, a Pointcut defines *where*, Advice defines *when* (before/after/around). Spring uses this internally for `@Transactional`.
- Coding exercise: create a custom `@LogExecutionTime` annotation and an Aspect that measures and logs the execution time of any method annotated with it.

### Project Block (1.5 hrs)
- Repository: **`scalable-ecommerce-platform` (new — this is now your one flagship project, not a second throwaway one).**
- Task: initialize a multi-module Maven project (Product, Order, Payment, Notification modules) from the start, rather than as a single-service repo that gets restructured later. Apply your `@LogExecutionTime` aspect to a dummy endpoint in the Order module.
- Definition of done: all four modules compile independently; the app starts; curling the annotated endpoint prints the execution time via the AOP interceptor.
- **Note on `todo-api`:** it stays exactly as it is — a complete, working Spring Boot practice project (health endpoint, Task CRUD, Flyway, Docker/Compose, JUnit/Mockito/TestContainers, Kafka, Trie autocomplete, config profiles, WireMock). It's not getting Kubernetes treatment or further feature work; that infrastructure depth now goes into the platform you'll actually be defending in interviews.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: apply to 2 more Tier C companies.

### Daily Deliverable
- [ ] Permutations solved, pushed.
- [ ] `scalable-ecommerce-platform` initialized with its full module skeleton and working AOP logging.

---

## Day 63 (Sunday) — Consolidation, and Backtracking Continues

### Self-Check (15 min)
- [ ] Solve one Heap or Trie problem from earlier this week cold, without hints.

### DSA Block (2 hrs)
- Problem 3: Combinations — LeetCode #77 — Medium — Pattern: Backtracking
  - Hint: order doesn't matter here — only recurse forward from the current index to avoid duplicate combinations.
  - Complexity: Time O(C(n,k)) | Space O(k)
- Problem 4: Permutations II — LeetCode #47 — Medium — Pattern: Backtracking with Duplicate Handling **(new)**
  - Hint: sort first; skip a choice at the same recursion depth if it equals the previous choice *and* the previous one hasn't been used yet in this branch — the standard duplicate-avoidance trick, same one you'll reuse on Subsets II.
  - Complexity: Time O(n×n!) | Space O(n)

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 63, nine weeks in, **123 total DSA problems solved.** Heaps fully closed at 10 (up from 6) and Tries' core fully closed at 6, both genuinely fixed gaps from the original plan. Backtracking is 4 problems into its run. `scalable-ecommerce-platform` is live with a real multi-module skeleton and AOP — five weeks earlier than the original plan's equivalent project, and it's the only backend project you'll be building from here forward.

### Daily Deliverable
- [ ] Combinations and Permutations II solved, pushed.
- [ ] Weekly ritual and scorecard complete.
