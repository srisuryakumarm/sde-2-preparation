# Week 8 (Revised): Trees Completes (15 Problems, Including the Median Fix), Heaps Begins

**What changed:** Trees closes at 15 problems (up from 11) — Lowest Common Ancestor of a general Binary Tree and Serialize/Deserialize Binary Tree are genuinely valuable additions, and Median of Two Sorted Arrays finally lands here, fulfilling what the original plan promised on Day 21 and then never delivered. Heaps begins its run toward 10 problems (up from 6) — this was the thinnest pattern in the original plan, so it gets real investment here.

---

## Day 50 — Binary Search Trees Begin, and Kafka Consumers

### DSA Block (2.5 hrs)

**Concept Card — Binary Search Trees**
- What: a binary tree with an ordering invariant — left subtree smaller, right subtree larger.
- Why: O(log n) search/insert/delete on a balanced tree, and inorder traversal visits nodes in sorted order for free.
- Interview signal: "BST," or a tree explicitly described as sorted in this structural sense.

- Problem 9: Validate Binary Search Tree — LeetCode #98 — Medium — Pattern: DFS with Boundaries
  - Hint: pass down a valid `(min, max)` range; the left child must respect `< root.val` as the new max, the right `> root.val` as the new min.
  - Complexity: Time O(n) | Space O(n)
- Problem 10: Kth Smallest Element in a BST — LeetCode #230 — Medium — Pattern: Inorder Traversal
  - Hint: an inorder traversal of a BST visits nodes in ascending order — count as you go and stop at `k`.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: Kafka Consumers and `@KafkaListener`
- Consumer groups split partition reads for parallelism; each partition is read by only one consumer within a group at a time. Offset committing (auto vs. manual) controls what happens on a crash — auto-commit can silently drop messages if you crash after committing but before finishing processing.
- Coding exercise: write a `@KafkaListener` method logging receipt of yesterday's `TaskCreatedEvent`.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: implement the `@KafkaListener` from the exercise above.
- Definition of done: creating a task logs both the producer send and the listener's receipt.

### Career Block (1 hr)
- LinkedIn: Post 12 — Kafka partitions and consumer groups, with a simple diagram.
- Networking: identify 3 Target Tier B companies.

### Daily Deliverable
- [ ] Validate Binary Search Tree and Kth Smallest Element in a BST solved, pushed to `dsa-java/trees/`.
- [ ] `@KafkaListener` live, verified end to end. LinkedIn Post 12 published.

---

## Day 51 — BST and Construction, and Spring Cloud Config

### DSA Block (2.5 hrs)
- Problem 11: Lowest Common Ancestor of a BST — LeetCode #235 — Medium — Pattern: BST Traversal
  - Hint: if both targets are less than the current node, go left; if both greater, go right; otherwise the current node is the LCA.
  - Complexity: Time O(n) | Space O(n)
- Problem 12: Construct Binary Tree from Preorder and Inorder Traversal — LeetCode #105 — Medium — Pattern: Divide and Conquer
  - Hint: the first element of preorder is always the root; find it in inorder to know exactly how many nodes belong to the left vs. right subtree.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: Spring Cloud Config and Externalized Configuration
- Hardcoding config (DB URLs, API keys, feature flags) means rebuilding to change anything — externalized config lets the same build artifact behave differently per environment, which is exactly the property you need once you have separate Dev, Staging, and Production environments to support.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: move hardcoded config into `application.yml` profiles (`dev`, `prod`), selected via `spring.profiles.active`.
- Definition of done: `-Dspring.profiles.active=dev` vs `prod` visibly changes behavior.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: send connection requests to the 3 Tier B targets from yesterday.

### Daily Deliverable
- [ ] Lowest Common Ancestor of a BST and Construct Binary Tree from Preorder/Inorder solved, pushed.
- [ ] `dev`/`prod` profiles working in `todo-api`.

---

## Day 52 — Trees Beyond BST: General LCA and Serialization, and WireMock

### DSA Block (2.5 hrs)
- Problem 13: Lowest Common Ancestor of a Binary Tree (general) — LeetCode #236 — Medium — Pattern: DFS, post-order **(new)**
  - Hint: this is genuinely different from Day 51's BST version — there's no ordering to exploit. Recurse into both children; if both return non-null, the current node is the LCA. If only one does, propagate it upward.
  - Complexity: Time O(n) | Space O(n)
- Problem 14: Serialize and Deserialize Binary Tree — LeetCode #297 — Hard — Pattern: DFS (preorder) with null markers **(new)**
  - Hint: preorder traversal, writing an explicit sentinel (e.g., `"#"`) for every null child. Deserializing just replays the same preorder logic, consuming tokens one at a time — the sentinels tell you exactly where each subtree ends.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: WireMock for API Mocking
- Stubs an HTTP endpoint's response (status, body, latency) so you can test how your code handles a third-party API's success, failure, or timeout, without that API actually needing to be up.
- Coding exercise: stub `http://localhost:8089/payment` returning 200 OK with fixed JSON.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: the WireMock stub above, as a JUnit test fixture (you'll use this for real once Resilience4j arrives in a few weeks).
- Definition of done: pushed, test passes against the stubbed endpoint.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: review 3 engineering manager profiles.

### Daily Deliverable
- [ ] Lowest Common Ancestor of a Binary Tree (general) and Serialize/Deserialize Binary Tree solved, pushed.
- [ ] WireMock stub test pushed.

---

## Day 53 — Trees Capstone: The Median Fix, and Divide-and-Conquer Reviewed

### DSA Block (2 hrs)
- Problem 15: Median of Two Sorted Arrays — LeetCode #4 — Hard — Pattern: Binary Search / Divide and Conquer
  - Hint: this is a Binary Search problem in disguise, not a merge problem — binary search on the *partition point* in the smaller array, such that everything to its left (combined across both arrays) is ≤ everything to its right. You now have the recursive divide-and-conquer intuition from Construct Binary Tree (Day 51) to draw on, which is exactly why this sat here rather than back in Week 4.
  - Complexity: Time O(log(min(m,n))) | Space O(1)

**This closes Trees: 15 problems, spanning DFS, BFS, BST traversal, construction, and general-tree LCA — up from 11 in the original plan. It also closes the loop on Median of Two Sorted Arrays, which the original plan promised on Day 21 and then quietly never delivered anywhere in the remaining 99 days.**

*(Tree DP — House Robber III, Binary Tree Maximum Path Sum — still arrives with Dynamic Programming, once DP thinking is in place to layer on top, exactly as the original plan intended.)*

### Theory Block (1 hr)
- Topic: Trees, Reviewed — BST Traversal vs. General Tree DFS
- Write down, in one sentence each: why does BST's Lowest Common Ancestor (Day 51) get to skip checking both subtrees, while general Binary Tree LCA (Day 52) can't? The ordering invariant is the entire difference, and being able to state that crisply is worth more than having solved both problems.

### Project Block (1 hr)
- Repository: none new — make sure all 15 Tree solutions are pushed and organized in `dsa-java/trees/`.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: casual check-in with your accountability partner.

### Daily Deliverable
- [ ] Median of Two Sorted Arrays solved — Trees ladder fully complete at 15 problems, dropped-thread fixed.
- [ ] BST vs. general-tree LCA reflection written.

---

## Day 54 — Heaps Begin, and Frequency Patterns

### DSA Block (2.5 hrs)

**Concept Card — Heaps / Priority Queue**
- What: a binary tree stored in an array where every parent is smaller (min-heap) or larger (max-heap) than its children — O(log n) insert/remove-extreme, O(1) peek.
- Why: whenever you repeatedly need "the current smallest/largest," a heap beats re-sorting or a linear scan every time.
- Where: task schedulers, Dijkstra's algorithm (coming with Graphs), event simulation, top-K queries.
- Interview signal: "k largest/smallest," "kth largest," "merge k sorted things," "median of a stream."
- Prerequisites: Comparable/Comparator ✅ (Week 3).
- Java API: `PriorityQueue<T>` — `.offer()`, `.poll()`, `.peek()`; pass a `Comparator` for max-heap behavior.

- Problem 1: Kth Largest Element in a Stream — LeetCode #703 — Easy — Pattern: Fixed-size Min-Heap **(new)**
  - Hint: maintain a min-heap of exactly size `k` across calls — every new value gets added, and if the heap exceeds `k`, poll the smallest. The heap's top is always the current kth largest.
  - Complexity: Time O(log k) per call | Space O(k)
- Problem 2: Last Stone Weight — LeetCode #1046 — Easy — Pattern: Max-Heap
  - Hint: repeatedly pop the two heaviest stones, push back the difference if nonzero.
  - Complexity: Time O(n log n) | Space O(n)

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 3 Target Tier B companies.

### Daily Deliverable
- [ ] Kth Largest Element in a Stream and Last Stone Weight solved, pushed to `dsa-java/heaps/`.

---

## Day 55 — Heaps Continue, and Spring Cloud Config Wrap-Up

### DSA Block (2.5 hrs)
- Problem 3: Kth Largest Element in an Array — LeetCode #215 — Medium — Pattern: Min-Heap of size k
  - Hint: keep a min-heap of size `k`; whenever it exceeds `k`, poll the smallest. The heap's top is the answer.
  - Complexity: Time O(n log k) | Space O(k)
- Problem 4: K Closest Points to Origin — LeetCode #973 — Medium — Pattern: Max-Heap of size k **(new)**
  - Hint: same shape as Kth Largest, inverted — keep a *max*-heap of size `k` by distance; whenever it exceeds `k`, poll the farthest point out.
  - Complexity: Time O(n log k) | Space O(k)

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: apply to 2 Tier B companies.

### Daily Deliverable
- [ ] Kth Largest Element in an Array and K Closest Points to Origin solved, pushed.

---

## Day 56 (Sunday) — Consolidation, and Heaps: Frequency and Scheduling

### Self-Check (15 min)
- [ ] Solve one Tree problem from earlier this week cold, without hints.

### DSA Block (2 hrs)
- Problem 5: Top K Frequent Elements — LeetCode #347 — Medium — Pattern: HashMap + Heap
  - Hint: count frequencies with a HashMap, keep a min-heap of size `k` ordered by frequency.
  - Complexity: Time O(n log k) | Space O(n)
- Problem 6: Ugly Number II — LeetCode #264 — Medium — Pattern: Min-Heap generating candidates in order **(new)**
  - Hint: start a min-heap with `1`. Repeatedly pop the smallest, and push its products with 2, 3, and 5 back in — skipping duplicates. The nth pop is the answer.
  - Complexity: Time O(n log n) | Space O(n)

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 56, eight weeks in, **110 total DSA problems solved.** Trees are fully closed at 15 problems (up from 11), and — genuinely worth pausing on — the Median of Two Sorted Arrays gap from the original plan's Day 21 is now actually closed instead of just promised. Heaps is 6 problems into its expanded 10, already meaningfully past where the original plan's thin 6-problem treatment stopped entirely.

### Daily Deliverable
- [ ] Top K Frequent Elements and Ugly Number II solved, pushed.
- [ ] Weekly ritual and scorecard complete.
