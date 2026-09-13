# Week 6 (Revised): Linked Lists Completes, Stacks Begin

**What changed:** Linked Lists closes at 11 problems (up from 9) — Linked List Cycle II joins as a natural extension of Cycle Detection. Stacks/Monotonic Stack grows to 12 problems (up from 10).

---

## Day 36 — Linked Lists: Cycles, and Spring Data JPA

### DSA Block (2.5 hrs)
- Problem 5: Linked List Cycle — LeetCode #141 — Easy — Pattern: Floyd's Cycle Detection
  - Hint: if a cycle exists, `fast` eventually "laps" `slow` and they become equal.
  - Complexity: Time O(n) | Space O(1)
- Problem 6: Linked List Cycle II — LeetCode #142 — Medium — Pattern: Floyd's, extended **(new)**
  - Hint: once `fast` and `slow` meet inside the cycle, reset one pointer to the head and advance both one step at a time — they meet again exactly at the cycle's start. The math behind why this works is worth understanding, not just memorizing.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Spring Data JPA
- `@Entity`, `@Id`, `@Column` map a class to a table. `JpaRepository<Entity, IdType>` generates CRUD from an interface — no implementation needed. This is the core idea behind an ORM (Object-Relational Mapper): you describe your data as plain Java objects and let a framework translate between those objects and SQL rows, instead of hand-writing `INSERT`/`SELECT`/`UPDATE` statements for every operation.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: define a `Task` entity (`id`, `title`, `description`, `status`, `priority`, `dueDate`) and `TaskRepository`. Configure PostgreSQL in `application.yml`.
- Definition of done: the app connects to local Postgres and Hibernate creates the `tasks` table.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: follow up on Day 34's outreach.

### Daily Deliverable
- [ ] Linked List Cycle and Linked List Cycle II solved, pushed to `dsa-java/linked-lists/`.
- [ ] `Task` entity and repository committed, table auto-generated.

---

## Day 37 — Linked Lists: Harder Pointer Manipulation, and Explicit Locks

### DSA Block (2.5 hrs)
- Problem 7: Remove Nth Node From End of List — LeetCode #19 — Medium — Pattern: Two Pointers (Fixed Offset)
  - Hint: dummy head; advance `fast` n+1 steps ahead, then move both until `fast` is null — `slow.next` is the node to remove.
  - Complexity: Time O(n) | Space O(1)
- Problem 8: Add Two Numbers — LeetCode #2 — Medium — Pattern: Math Simulation on Linked List
  - Hint: traverse both lists together, tracking a `carry`; `sum = val1 + val2 + carry`.
  - Complexity: Time O(max(m,n)) | Space O(max(m,n))

### Theory Block (2 hrs)
- Topic: Explicit Locks
- `ReentrantLock` does what `synchronized` does with more control — timeout-based acquisition, fairness policies. `ReadWriteLock` allows multiple concurrent readers but exclusive writers.
- Coding exercise: write a `Counter` class using `ReentrantLock`; confirm it's correct under 100 concurrent threads incrementing it.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: the `Counter` exercise above.
- Definition of done: pushed, verified correct under concurrent load.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: send 3 connection requests to engineers whose blog posts you've read.

### Daily Deliverable
- [ ] Remove Nth Node From End and Add Two Numbers solved, pushed.
- [ ] `ReentrantLock` `Counter` pushed and verified.

---

## Day 38 — Linked Lists: Advanced Combinations, and Wait/Notify

### DSA Block (2.5 hrs)
- Problem 9: Reorder List — LeetCode #143 — Medium — Pattern: Fast/Slow + Reversal + Merge
  - Hint: find the middle, reverse the second half, then merge both halves alternately — three techniques you already have, combined.
  - Complexity: Time O(n) | Space O(1)
- Problem 10: Copy List with Random Pointer — LeetCode #138 — Medium — Pattern: HashMap for Node Mapping
  - Hint: pass 1 maps old nodes to new nodes in a `HashMap<Node, Node>`; pass 2 wires `next`/`random` using the map.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: Wait/Notify and `Condition`
- `wait()`/`notify()`/`notifyAll()` let threads coordinate around a shared condition instead of busy-spinning. `Condition` (paired with `ReentrantLock`) is the modern equivalent with clearer semantics.
- Coding exercise: implement Producer-Consumer using `ReentrantLock` and two `Condition`s (`notFull`, `notEmpty`).

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: the Producer-Consumer exercise above.
- Definition of done: pushed, runs without deadlocking or losing items.

### Career Block (1 hr)
- LinkedIn: Post 9 — "I made a HashMap lose track of an object on purpose" (callback to Week 3's hashCode demo).
- Networking: identify 5 Target Tier B companies.

### Daily Deliverable
- [ ] Reorder List and Copy List with Random Pointer solved, pushed.
- [ ] Producer-Consumer demo pushed. LinkedIn Post 9 published.

---

## Day 39 — Linked Lists Capstone, Stacks Begin, and ConcurrentHashMap

### DSA Block (2.5 hrs)
- Problem 11: LRU Cache — LeetCode #146 — Medium — Pattern: HashMap + Doubly Linked List
  - Hint: a HashMap gives O(1) lookup; a hand-rolled doubly linked list gives O(1) removal/reinsertion at the ends. Store DLL node references as the map's values.
  - Complexity: Time O(1) all ops | Space O(capacity)

**This closes Linked Lists: 11 problems, Easy through Medium — up from 9 in the original plan.**

**Concept Card — Stacks**
- What: LIFO — the last thing pushed is the first thing popped. You've used `ArrayDeque` as a stack since Week 1; the pattern itself starts appearing in problems now.
- Why: any "most recent unresolved thing" problem — matching brackets, undo operations, expression parsing.
- Interview signal: "matching/balanced," "nested," "most recent," "evaluate an expression."

- Problem 1: Valid Parentheses — LeetCode #20 — Easy — Pattern: Stack
  - Hint: push opening brackets; on a closing bracket, check it matches the top of the stack, then pop.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: `ConcurrentHashMap` Internals
- Java 8+ locks individual bucket nodes with CAS operations and fine-grained `synchronized`, far less contention than locking the whole map. `Collections.synchronizedMap()` wraps a plain HashMap with one global lock, serializing every access.
- Coding exercise: benchmark concurrent writes from 10 threads into a `Collections.synchronizedMap()` vs. a `ConcurrentHashMap`.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `ConcurrentMapBenchmark` class from the exercise above.
- Definition of done: pushed, with the timing difference and a one-line explanation.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: send connection requests to the 5 Tier B targets from yesterday.

### Daily Deliverable
- [ ] LRU Cache and Valid Parentheses solved — Linked Lists complete at 11, Stacks begun. Pushed to `dsa-java/linked-lists/` and `dsa-java/stacks/` respectively.
- [ ] `ConcurrentMapBenchmark` pushed with measured timing.

---

## Day 40 — Stacks Continue, and SQL Fundamentals

### DSA Block (2.5 hrs)
- Problem 2: Implement Queue using Stacks — LeetCode #232 — Easy — Pattern: Two Stacks
  - Hint: an "in" stack for pushes, an "out" stack for pops — transfer everything only when "out" is empty.
  - Complexity: Time O(1) amortized | Space O(n)
- Problem 3: Next Greater Element I — LeetCode #496 — Easy — Pattern: Monotonic Stack + HashMap
  - Hint: process with a decreasing monotonic stack; whenever you pop an element because the current one is bigger, you've found that popped element's "next greater."
  - Complexity: Time O(n+m) | Space O(n)

### Theory Block (2 hrs)
- Topic: SQL Fundamentals
- Primary/foreign keys, normalization. `INNER`, `LEFT`, `RIGHT`, `FULL OUTER` joins — and the difference between "no matching row" (LEFT JOIN gives NULLs) and "row excluded" (INNER JOIN). Indexes (B-trees) speed lookups at the cost of slower writes, since every insert now also has to update the index structure, not just the table itself.
- Coding exercise: write a query joining `users` and `orders` to find every user who has never placed an order.

### Project Block (1.5 hrs)
- Repository: `todo-api`.
- Task: add Flyway. Move schema ownership from `ddl-auto: update` to a versioned `V1__create_tasks_table.sql`; set `ddl-auto: validate`.
- Definition of done: the app boots via Flyway-managed schema; `flyway_schema_history` shows `V1` applied.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: begin tailoring your resume with the Spring Boot + concurrency work from these weeks — the first of several scheduled refresh points (this one's earlier than the original plan had it, deliberately, so it stays current).

### Daily Deliverable
- [ ] Implement Queue using Stacks and Next Greater Element I solved, pushed.
- [ ] Flyway migration live in `todo-api`.

---

## Day 41 — Stacks: Circular Variants, and Min Stack

### DSA Block (2.5 hrs)
- Problem 4: Next Greater Element II — LeetCode #503 — Medium — Pattern: Monotonic Stack (circular array) **(new)**
  - Hint: simulate wrapping around by iterating `2n` times using `i % n` — the monotonic-decreasing-stack logic from yesterday is otherwise unchanged.
  - Complexity: Time O(n) | Space O(n)
- Problem 5: Min Stack — LeetCode #155 — Medium — Pattern: Two Stacks
  - Hint: a secondary stack tracks the running minimum, pushing a new minimum only when the incoming value is `<=` the current one.
  - Complexity: Time O(1) all ops | Space O(n)

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: review 3 engineering manager profiles.

### Daily Deliverable
- [ ] Next Greater Element II and Min Stack solved, pushed.

---

## Day 42 (Sunday) — Consolidation, and Monotonic Stack Begins

### Self-Check (15 min)
- [ ] Solve one Linked List problem from earlier this week cold, without hints.

### DSA Block (2 hrs)
- Problem 6: Daily Temperatures — LeetCode #739 — Medium — Pattern: Monotonic Decreasing Stack
  - Hint: store indices, not temperatures. When today's temperature beats the one at the top of the stack, pop it and record the day difference.
  - Complexity: Time O(n) | Space O(n)
- Problem 7: Evaluate Reverse Polish Notation — LeetCode #150 — Medium — Pattern: Stack
  - Hint: push numbers; on an operator, pop the top two, apply it, push the result back.
  - Complexity: Time O(n) | Space O(n)

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 42, six weeks in, **84 total DSA problems solved.** Linked Lists fully closed at 11 (up from 9). Stacks/Monotonic Stack underway, 7 of its expanded 12-problem set done. `todo-api` now has Task CRUD, Flyway-managed schema, and threading fundamentals (ReentrantLock, Condition, ConcurrentHashMap) covered in `java-fundamentals`.

### Daily Deliverable
- [ ] Daily Temperatures and Evaluate Reverse Polish Notation solved, pushed.
- [ ] Weekly ritual and scorecard complete.
