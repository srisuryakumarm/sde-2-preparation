# Week 15 (Revised): Bit Manipulation and Tries Close, Segment Trees, Sorting From Scratch, SQL Bonus Track, and the Entire DSA Curriculum Completes

**What changed:** Bit Manipulation closes at 10 problems (up from 8), and this same day closes Tries at 7 (the "one more, arrives with Bit Manipulation" promise from Week 9, now delivered). Segment Trees stays deliberately light at 2 problems — genuine exposure to a rare topic, not exhaustive coverage, exactly as the original plan's own reasoning intended. This week also adds the SQL practice track you asked to include, and closes the entire DSA curriculum.

---

## Day 99 — Bit Manipulation Capstone, and Tries Fully Closes

### DSA Block (2.5 hrs)
- Problem 9: Reverse Bits — LeetCode #190 — Easy — Pattern: Bit Manipulation
  - Hint: build the result bit by bit — shift the result left by one, OR in the current lowest bit of the input, then shift the input right by one. Repeat 32 times.
  - Complexity: Time O(1) | Space O(1)
- Problem 10: Maximum XOR of Two Numbers in an Array — LeetCode #421 — Medium — Pattern: Bit Trie
  - Hint: insert every number's binary representation into a Trie, one bit at a time. To maximize XOR for each number, greedily try to walk the *opposite* bit at every level of the Trie — XOR is maximized when the bits differ.
  - Complexity: Time O(n) | Space O(n)

**This closes Bit Manipulation: 10 problems — up from 8 in the original plan. It also closes Tries entirely at 7 problems — the 6 core problems from Week 9, plus this one, exactly as promised back then.**

### Theory Block (2 hrs)
- Topic: Kubernetes Horizontal Pod Autoscaler (HPA)
- HPA watches a metric — typically CPU utilization, though custom metrics work too — and automatically scales the number of running replicas up or down within configured bounds, backed by the Metrics Server reading real usage data.
- Coding exercise: write an `hpa.yaml` targeting the platform's Order module deployment, scaling up past 70% CPU utilization.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: apply the HPA manifest to a local Minikube cluster; run a quick load test (Apache Bench or `k6`) to spike CPU artificially.
- Definition of done: `kubectl get hpa` shows the replica count increasing under load.

### Career Block (1 hr)
- LinkedIn: Post 19 — "I watched Kubernetes scale my service from 1 to 3 pods under load" (a screenshot of `kubectl get hpa` mid-scale is a strong visual).
- Networking: identify 2 more Tier B companies.

### Daily Deliverable
- [ ] Reverse Bits and Maximum XOR of Two Numbers in an Array solved — Bit Manipulation complete at 10, Tries fully closed at 7.
- [ ] HPA scaling verified under load. LinkedIn Post 19 published.

---

## Day 100 — Segment Trees Begin, and Creational Design Patterns

### DSA Block (2.5 hrs)

**Concept Card — Segment Trees**
- What: a binary tree where each node stores an aggregate (sum, min, max) over a range of the underlying array. Updating a single leaf takes O(log n) to propagate the change up to every affected ancestor, and querying any range also takes O(log n) instead of O(n).
- Why: whenever an array needs *both* frequent updates *and* frequent range queries, neither a plain prefix-sum array (fast queries, but O(n) per update) nor brute force (fast updates, but O(n) per query) is good enough for both at once. This is a genuinely advanced, comparatively rare interview topic — worth solid exposure, not exhaustive mastery, which is exactly why it stays at 2 problems here rather than being expanded like the genuinely thin patterns were.
- Interview signal: "range sum/min/max query," combined explicitly with "the array is also being updated."

- Problem 1: Range Sum Query - Mutable — LeetCode #307 — Medium — Pattern: Segment Tree
  - Hint: build a tree where each node stores the sum of its range. Both `update` and `sumRange` only ever touch O(log n) nodes on their way up or down the tree.
  - Complexity: Time O(log n) | Space O(n)

### Theory Block (2 hrs)
- Topic: GoF Design Patterns — Creational
- Singleton (exactly one instance of a class exists, globally accessible), Factory Method (object creation is delegated to a subclass or dedicated method, decoupling the caller from a specific concrete class), Builder (constructs a complex object step by step, useful when a constructor would otherwise need a long list of optional parameters). These are the patterns concerned with *how objects get created*, as opposed to what they do afterward — the natural first stop before next week's LLD systems, where these patterns get applied to real problems instead of toy examples.
- Coding exercise: write a thread-safe Singleton using Double-Checked Locking, then rewrite it using the Enum Singleton approach — the *Effective Java*-recommended way, and immune to the reflection and serialization attacks that the double-checked version isn't protected against.

### Project Block (1.5 hrs)
- Repository: `lld-java` (new — this repository holds every LLD system from next week onward).
- Task: initialize the repository; implement both Singleton variants from the exercise above in a `design-patterns` module.
- Definition of done: both compile, and a comment explains why Enum Singleton is generally the safer default.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: reply to any recruiter inbound messages.

### Daily Deliverable
- [ ] Range Sum Query - Mutable solved, pushed to `dsa-java/segment-trees/`.
- [ ] `lld-java` initialized with both Singleton implementations.

---

## Day 101 — Segment Trees Complete, and Factory/Builder Patterns

### DSA Block (2.5 hrs)
- Problem 2: Count of Smaller Numbers After Self — LeetCode #315 — Hard — Pattern: Segment Tree / Fenwick Tree
  - Hint: traverse the array right to left, inserting each element into a Segment or Fenwick tree indexed by value, querying the count of elements smaller than it that you've already seen.
  - Complexity: Time O(n log n) | Space O(n)

**This closes Segment Trees: 2 problems — genuine exposure to a rare-but-real interview topic, deliberately not expanded further, matching the original plan's own correct reasoning on this one.**

### Theory Block (2 hrs)
- Topic: Factory Method and Builder Patterns, Implemented
- Factory Method delegates object creation to a subclass or a dedicated method, decoupling the calling code from needing to know a concrete class name. Builder constructs a complex object step by step through a fluent chain of calls, which is especially useful when a constructor would otherwise need many optional parameters and you want to avoid an unreadable pile of positional arguments.
- Coding exercise: implement both patterns comprehensively, with unit tests for each.

### Project Block (1.5 hrs)
- Repository: `lld-java`.
- Task: the Factory Method and Builder implementations above, in the `design-patterns` module.
- Definition of done: both patterns implemented, tested, and pushed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: continue Tier B outreach and follow-up.

### Daily Deliverable
- [ ] Count of Smaller Numbers After Self solved — Segment Trees complete at 2 problems.
- [ ] Factory Method and Builder implementations pushed.

---

## Day 102 — Sorting Algorithms, From Scratch

### DSA Block (2.5 hrs)

**Concept Card — Sorting Algorithms, From Scratch**
- What: `Collections.sort()` and `Arrays.sort()` have been used throughout this plan without ever implementing the algorithm underneath either one — today closes that gap.
- Why: "implement merge sort" or "explain quicksort's worst case" is a real, if less frequently asked, interview question — and understanding *why* Java's `Arrays.sort()` uses different algorithms for primitives (dual-pivot quicksort) versus objects (a variant of merge sort called Timsort) requires actually knowing both algorithms, not just their names.
- Merge sort: divide-and-conquer, O(n log n) guaranteed in every case, stable (equal elements keep their relative order), needs O(n) extra space. Quicksort: also O(n log n) on average, but O(n²) in the worst case on adversarial input, typically faster in practice anyway due to better cache locality, and sorts in place.

- Exercise: implement merge sort and quicksort from scratch — no LeetCode wrapper needed. Write both, test them on a hand-built array, and verify the output matches `Arrays.sort()`'s.
- Problem: Kth Largest Element in an Array — LeetCode #215 — Medium — Pattern: Quickselect (Revision)
  - Hint: you already solved this with a heap back in Week 8 — today, solve it again using Quickselect instead (a quicksort partition step, but only recursing into whichever side actually contains the target index), averaging O(n) instead of O(n log k).
  - Complexity: Time O(n) average | Space O(1)

### Theory Block (1 hr)
- Topic: The Entire DSA Curriculum, Looking Back
- You've now covered every major interview pattern from HashMap through Sorting. Spend this hour differently than usual: list every pattern by name, from memory, in roughly the order you learned them. For each one, without looking anything up, state the one-sentence interview signal that tells you "this is that pattern." This is the single highest-value hour in the entire DSA phase — it's the difference between having solved 203 problems and being able to *recognize* which of those 203 solving-techniques a brand-new problem actually needs.

### Project Block (1 hr)
- Repository: `dsa-java`.
- Task: none new — use this slot to do a final pass making sure every pattern folder is complete, correctly named, and easy to navigate. This repository is about to become a resource you lean on constantly for interview review, not just a place solutions went.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: verify all Tier B and C applications from the last month are actually submitted, not just drafted.

### Daily Deliverable
- [ ] Merge sort and quicksort implemented from scratch and verified against `Arrays.sort()`.
- [ ] Kth Largest Element in an Array solved via Quickselect.
- [ ] Full pattern-and-signal recall exercise complete.

---

## Day 103 — SQL Practice, Part 1: Joins and Aggregations

### SQL Block (2.5 hrs)
- Many SDE-2 loops include a SQL screening round or SQL-flavored questions inside a broader round, separate from the LeetCode-style DSA rounds. You already covered SQL fundamentals (joins, keys, indexes) back in Week 6 — today and tomorrow turn that into the same kind of pattern-recognition practice you've been doing for algorithmic problems, using LeetCode's SQL problem set.
- Second Highest Salary — LeetCode #176 — Medium — Pattern: Subquery / LIMIT-OFFSET
  - Hint: a naive `MAX()` only gets you the highest; wrap it in a subquery that excludes the actual highest, or use `LIMIT 1 OFFSET 1` on a descending sort.
- Department Highest Salary — LeetCode #184 — Medium — Pattern: Correlated Subquery / Window Function
  - Hint: for each department, you need the max salary — a correlated subquery comparing each row's salary to the max within its own department (or a `RANK()` window function) both work.
- Duplicate Emails — LeetCode #182 — Easy — Pattern: GROUP BY / HAVING
  - Hint: `GROUP BY email HAVING COUNT(*) > 1` — the classic "find groups with more than one member" shape.
- Rising Temperature — LeetCode #197 — Easy — Pattern: Self-Join on Consecutive Dates
  - Hint: join the table to itself where one side's date is exactly one day after the other's, then compare temperatures across that join.
- Employees Earning More Than Their Managers — LeetCode #181 — Easy — Pattern: Self-Join
  - Hint: join the table to itself, matching each employee's `managerId` to their manager's `id`, then compare salaries directly.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: no specific outreach today.

### Daily Deliverable
- [ ] All 5 SQL problems solved, pushed to `dsa-java/sql/`.

---

## Day 104 — SQL Practice, Part 2: Window Functions

### SQL Block (2.5 hrs)
- Window functions are the one SQL topic most likely to genuinely trip up someone whose SQL experience is mostly transactional CRUD queries rather than analytical ones — worth deliberate practice.
- Nth Highest Salary — LeetCode #177 — Medium — Pattern: Window Function (DENSE_RANK) or parameterized LIMIT-OFFSET
  - Hint: `DENSE_RANK() OVER (ORDER BY salary DESC)` handles ties correctly in a way that plain `LIMIT`/`OFFSET` doesn't — worth knowing both approaches and when each is appropriate.
- Consecutive Numbers — LeetCode #180 — Medium — Pattern: Window Function (LAG/LEAD) or Self-Join
  - Hint: compare each row to the two rows before it using `LAG()` twice, or perform the equivalent with two self-joins on consecutive IDs.
- Trips and Users — LeetCode #262 — Hard — Pattern: Multi-table Join + Conditional Aggregation
  - Hint: join trips to users twice (once for the client, once for the driver) to filter out banned users, then use conditional aggregation (`SUM(CASE WHEN ... THEN 1 ELSE 0 END)`) to compute the cancellation rate per day.
- Exchange Seats — LeetCode #626 — Medium — Pattern: CASE + Parity Logic
  - Hint: swap adjacent odd/even-numbered seats using `CASE WHEN id % 2 = 1 AND id != (SELECT MAX(id) ...) THEN id + 1 WHEN id % 2 = 0 THEN id - 1 ELSE id END`.
- Department Top Three Salaries — LeetCode #185 — Hard — Pattern: Window Function (DENSE_RANK, partitioned)
  - Hint: `DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC)`, then filter to rank ≤ 3 — the `PARTITION BY` is what makes this "top three *per department*" instead of "top three overall."

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: no specific outreach today.

### Daily Deliverable
- [ ] All 5 SQL problems solved, pushed to `dsa-java/sql/`. SQL practice track complete at 10 problems.

---

## Day 105 (Sunday) — The Entire DSA Curriculum Is Complete

### Self-Check (20 min)
- [ ] Without notes: name every DSA pattern covered since Day 5, in order. For three patterns chosen at random, explain the interview signal that identifies each one and solve one problem from that pattern cold.

### DSA Block — none new today; this day is entirely consolidation

### Career Block (2 hrs)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard, and a real milestone: Day 105, the entire DSA curriculum is closed.** 203 DSA problems across every pattern from HashMap/HashSet through Sorting, plus a 10-problem SQL practice track — genuinely comprehensive, not padded. Both of the two pattern families that didn't exist anywhere in the original plan (Greedy/Intervals, Prefix Sum/Kadane's) are fully closed. Both of the thinnest gaps from the original audit (Heaps, Union-Find, Dijkstra's, Tries) are fixed. The one broken promise (Median of Two Sorted Arrays) is fixed. `todo-api` is complete and frozen; `scalable-ecommerce-platform` has grown into a genuine microservices platform with AOP, Resilience4j, Feign, a Gateway with JWT and rate limiting, Kafka with Schema Registry and a DLQ, Saga choreography, Eureka service discovery, Secrets-based config, and HPA — everything `order-management-api` was originally going to teach, built directly into what's now your one flagship project. `lld-java` is initialized with the first three Creational patterns. LLD systems begin next week.
- **Honest accounting on timeline:** this closes 8 days later than the last schedule update projected (Day 105 instead of Day 97) — Dynamic Programming and the sorting/consolidation work at the end both took slightly more real time than the earlier estimate assumed. Rather than keep adjusting projections as new weeks get written, the next message will give you the actual final total based on what's really been built, not another forward estimate.

### Daily Deliverable
- [ ] Full-pattern recall self-check complete.
- [ ] Weekly ritual and scorecard complete — **the DSA phase, in full depth, is done.**

---

### A note on what changes starting Week 16

Everything above — every pattern, every problem, every theory block — was built identically regardless of which company might eventually ask it. That doesn't change now. What changes from Week 16 onward: the LLD and HLD systems, the mock interviews, and the behavioral prep get a layer of target-company-specific rehearsal on top of the same underlying depth, aimed at the researched company list (Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech), and applications begin. See `Target_Company_Research_and_Interview_Guide.md` for the full research behind what follows.
