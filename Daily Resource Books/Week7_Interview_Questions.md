# Week 7 — Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Covers:** Days 43–49 — Stacks/Monotonic Stack (closing), Trees (opening) — plus Docker, Docker Compose, JUnit 5/AssertJ, Mockito, TestContainers, and Kafka Fundamentals.

Every question below is pulled directly from its day's Resource Book, in the same order it was taught. Use this as a standalone review pass once all seven days are done — if a question here doesn't come back immediately, that's a pointer to which day's book to reopen.

---

## Day 43 — Stack Simulation, and Docker Basics

**Q1. Why does Asteroid Collision reach for a stack instead of, say, two pointers?**
The problem requires comparing each new left-moving asteroid against the *most recently surviving* asteroid to its left, and a collision can cascade backward through several previous survivors. A stack's LIFO order gives you exactly "the nearest survivor" in O(1), and popping destroyed asteroids off the top naturally exposes the next-nearest one to check against.

**Q2. In Asteroid Collision, why is the collision-check a `while` loop and not an `if`?**
A single large left-mover can destroy multiple smaller right-movers in a row. An `if` would only resolve one collision per asteroid and incorrectly leave the rest of the destroyed chain on the stack; the `while` keeps resolving against the new top until the current asteroid is destroyed, survives outright, or the stack no longer opposes it.

**Q3. What's the one adjacency pattern that can ever produce a collision, and why does that fact justify the stack approach?**
Only a right-mover immediately followed (at some point) by a left-mover can collide — anything moving apart or in the same direction never meets. This is exactly what lets a single left-to-right pass work: by the time you reach a left-mover, everything that could possibly collide with it is already sitting on the stack.

**Q4. Walk through why Basic Calculator II's brute-force splicing approach is O(n²) while the stack approach is O(n).**
Splicing removes tokens from a `List`, and each removal shifts every subsequent element down by one — an O(n) operation — repeated for up to O(n) operators, giving O(n²) total. The stack approach makes one pass, doing O(1) amortized work per character (a push, or a pop-then-push), for O(n) total.

**Q5. What does `lastSign` actually track, and why is it updated *after* processing the current number rather than *when* the operator character is first seen?**
`lastSign` holds the operator that applies to the number currently being accumulated — which is the operator seen *before* this number started, not the one just encountered. The current character is only ever the operator for the *next* number, so it's stored into `lastSign` only after the just-finished number has been finalized against the *previous* value of `lastSign`.

**Q6. Why does a `*` or `/` in Basic Calculator II require popping from the stack, while `+` and `-` don't?**
A `+`/`-` term is final the instant it's pushed — nothing later in a left-to-right scan can change it. A `*`/`/` needs to correct the term that's already on top of the stack (multiply or divide it by the new number), so the stack has to expose — and let you replace — its most recently pushed value, which is exactly what pop-then-push gives you.

**Q7. What's the difference between a Docker image and a container?**
An image is a static, layered, read-only snapshot built once from a `Dockerfile`. A container is a running instance of that image, with a thin writable layer on top for runtime changes; that writable layer is discarded when the container is removed, and the underlying image is never modified by running (or removing) a container.

**Q8. If you edit one line in `TaskController.java` and rebuild, which Docker layers get rebuilt, assuming the cache-friendly `Dockerfile` ordering from today?**
Only the `COPY src ./src` layer and everything after it (the `mvn package` layer) — the earlier `COPY pom.xml .` and `RUN mvn dependency:go-offline` layers are unaffected by a source change and are reused straight from cache.

**Q9. Why does putting `COPY . .` before dependency resolution hurt build times, specifically?**
It bundles your source files into the same layer's cache key as your dependency list. Since source changes on nearly every commit, that layer's cache key changes on nearly every commit too — invalidating the (expensive, network-bound) dependency-download step on every single build, even though the dependencies themselves didn't actually change.

**Q10. What does a multi-stage build remove from the final image that a single-stage build would include, and why does that matter?**
It removes the JDK compiler, Maven itself, and the full source tree — only the compiled `.jar` crosses from the build stage into the minimal runtime stage. This produces a smaller image (faster to pull and start) and a smaller attack surface, since there's no build tooling present in the deployed container at all.

---

## Day 44 — Stacks: The Hard Tier, and Docker Compose

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
Postgres is stateful — its data must survive the container being removed and recreated, which is exactly what a container's own writable layer does *not* guarantee. `app` is stateless (all its state lives in Postgres and Redis), so there's nothing on its own filesystem that needs to persist across restarts.

---

## Day 45 — Stacks Capstone, and JUnit 5/AssertJ

**Q1. Why does the naive "strip the parentheses and evaluate left to right" approach fail on `"2-(5-6)"`?**
A `-` sign immediately before a `(` needs to flip the sign of every term inside those parentheses, not just the first one. Stripping the parens and evaluating left to right only negates the first number after the `-`, then treats everything after it as if it were still at the outer nesting level — giving `2-5-6=-9` instead of the correct `2-(5-6)=3`.

**Q2. Precisely, what gets pushed onto the stack when a `(` is encountered, and why both values?**
The result accumulated so far, and the sign that was pending before the `(`. Both are needed on the way back out: the pending result is what the finished group gets added onto, and the pending sign is what the *entire* finished group must be multiplied by — which is exactly what correctly propagates a `-` across every term inside the group.

**Q3. What happens at a `)`, in order, and why that specific order?**
First, the last number inside the group is finalized into the group's local `result`. Then that local result is multiplied by the sign popped off the stack (the sign that applied to the whole group). Then the result popped off the stack (what had been built before the `(`) is added back on. Multiplying by the group's sign has to happen before adding back the outer context, or the outer sign would incorrectly apply to the outer context too.

**Q4. How does Basic Calculator's use of the stack differ from Basic Calculator II's, given both are "Stack — general purpose"?**
Basic Calculator II defers a single *value* — a term that a later `*` or `/` might still need to correct. Basic Calculator defers an entire *evaluation context* — a result-so-far and a sign — one full context per level of paren nesting, which is why its stack can grow proportionally to nesting depth rather than to term count.

**Q5. Sort Daily Temperatures and Evaluate Reverse Polish Notation into the two Stack families from this week's review, and justify each.**
Daily Temperatures is a true monotonic stack — it maintains a decreasing-height invariant to answer "how far to the next greater element," a positional-neighbor question. Evaluate RPN is general-purpose LIFO — the stack just holds pending operands with no ordering invariant on their values; it's simulating an ordered sequence of operations, not comparing neighbors.

**Q6. Why does `@BeforeEach` run before every single `@Test` method, rather than once per test class?**
Tests must be independent of each other and of execution order, which isn't guaranteed by JUnit. Running setup fresh before every test method guarantees no test can accidentally depend on state left behind by a previous one.

**Q7. What's the actual advantage of `@ParameterizedTest` over three separate `@Test` methods with the same body and different inputs?**
Beyond fewer lines, it means there's exactly one copy of the test logic to get right and to maintain — three separate near-identical methods risk a bug or an update being fixed in one copy and silently missed in the other two.

**Q8. What does `@DataJpaTest` actually configure, and why can't it be used to test `TaskController` directly?**
It spins up an embedded database and scans only `@Entity` classes and Spring Data JPA repositories — it does not load the web layer (controllers, request mapping) at all. Testing `TaskController`'s own behavior needs `@WebMvcTest` or `@SpringBootTest`; `@DataJpaTest` is the right tool for the persistence layer underneath the controller, not the controller itself.

**Q9. Give one concrete advantage of AssertJ's `assertThat(...).hasSize(3).first().matches(...)` chain over the equivalent separate JUnit assertions.**
When one link in the chain fails, AssertJ can report specifically which condition on which element failed, in language close to the assertion's own intent — separate `assertEquals`/`assertTrue` calls each report in isolation and don't carry that same connected context about what larger property was actually being checked.

---

## Day 46 — Trees Begin, and Mockito

**Q1. How does `TreeNode` differ structurally from the Linked List `Node` class, and what stays exactly the same?**
`TreeNode` has two self-references (`left`, `right`) instead of one (`next`), forming branches instead of a single chain. What stays the same is the core idea — a class holding a value plus one or more references to another instance of itself — and every recursive technique for handling `null`, defining a base case, and combining results carries over unchanged; only the number of children to combine changes.

**Q2. Give the precise, edge-based definitions of depth and height, and state each one's base case for a `null` node.**
Depth of a node is the number of edges on the path from the root down to it (root has depth 0). Height of a node is the number of edges on the longest downward path from it to a leaf (a leaf has height 0). Using the recursive formula `height = 1 + max(height(left), height(right))`, the base case for a `null` child must be `-1`, so a leaf — with two `null` children — correctly computes to `1 + max(-1,-1) = 0`.

**Q3. LeetCode's "Maximum Depth of Binary Tree" uses base case `null → 0`, not `-1`. What is it actually computing, and why does that base case make sense for it?**
It's computing the tree's height, expressed as a count of *nodes* on the longest path rather than *edges* — for a single-node tree it returns 1, not the graph-theoretic depth of 0. With base case `0`, a leaf computes to `1 + max(0,0) = 1`, correctly counting "one node," which is exactly LeetCode's own stated definition for this problem.

**Q4. What are the four named traversal orders, and which one shares Maximum Depth and Invert Binary Tree's underlying shape?**
Preorder (root, left, right), inorder (left, root, right), postorder (left, right, root), and level-order (breadth-first, by depth). Both problems are postorder-shaped: each needs both children's results before it can combine them into its own answer, even though neither literally performs a traversal that prints values.

**Q5. Precisely state the space complexity of a recursive DFS tree traversal — not just "O(n)."**
O(h), where h is the tree's height: O(log n) for a balanced tree, degrading to O(n) in the worst case of a completely skewed tree (every node with only one child). "O(n)" is a correct but loose upper bound that's really describing the worst case only.

**Q6. Why is Maximum Depth's iterative BFS alternative worth knowing, given the recursive version is shorter?**
It avoids the call-stack depth risk the recursive version carries on a pathologically skewed tree — recursion depth equals tree height, and a sufficiently deep, unbalanced tree risks a `StackOverflowError`. BFS's space bound is O(w), the tree's maximum width, a structurally different risk than O(h) stack depth.

**Q7. In Invert Binary Tree's recursive solution, why must `root.left` be saved to a temp variable before either assignment happens?**
The two assignments overwrite `root.left` and `root.right` in sequence; if `root.left` isn't saved first, the second assignment (`root.right = invertTree(temp)`) would need `temp` to still hold the *original* left child, but without saving it, that value would already have been overwritten by the first assignment.

**Q8. Why doesn't the order of traversal (stack vs. queue, DFS vs. BFS) matter for Invert Binary Tree, when it clearly does matter for Maximum Depth?**
Swapping one node's children is a fully local operation that doesn't depend on any other node's swap having already happened. Maximum Depth, by contrast, needs each child's *result* before it can compute its own — a genuine data dependency that traversal order must respect (children before parent), which Invert Binary Tree simply doesn't have.

**Q9. What does `@Mock` actually create, and what does `@InjectMocks` do with it?**
`@Mock` creates a fake implementation of a dependency (e.g., `TaskRepository`) that does nothing unless explicitly stubbed. `@InjectMocks` constructs the class under test (`TaskService`) and automatically wires any `@Mock` fields into it, without needing manual `new TaskService(taskRepository)` wiring.

**Q10. Why does a Mockito-based `TaskService` test run in milliseconds, while a `@DataJpaTest` does not?**
The Mockito test has no database and no Spring context at all — `TaskRepository` is a plain fake object, so calling it is just a regular method call with no I/O. `@DataJpaTest` spins up a real (if embedded) database and Spring context, which carries genuine startup and I/O cost even though it's not a full production database.

---

## Day 47 — Tree DFS Continues, and TestContainers

**Q1. Why must a tree serialization include an explicit marker for `null` children, not just omit them?**
Without an explicit marker, structurally different trees can produce identical serialized sequences — e.g., `[1, 2]` (left child only) and `[1, null, 2]` (right child only) both serialize to `[1, 2]` if `null`s are simply skipped, incorrectly making two differently-shaped trees compare as equal.

**Q2. Walk through why Symmetric Tree's `isMirror` compares `left.left` against `right.right`, rather than `left.left` against `right.left`.**
A mirror reflection swaps positions at every level, not just at the top — the left subtree's *left* child sits in the position that mirrors the right subtree's *right* child, and vice versa. Comparing same-side to same-side would check for two identical subtrees, not a mirrored pair.

**Q3. What's the actual advantage of the direct recursive comparison in Same Tree / Symmetric Tree over serializing both structures first and comparing the results?**
It avoids allocating extra data structures (full serialized lists, or in Symmetric Tree's brute force, an entire copied-and-inverted subtree) purely to hold an intermediate representation — the direct comparison only needs O(h) call-stack space, versus O(n) for the serialized approach.

**Q4. In Subtree of Another Tree, why is checking whether `subRoot`'s values merely *appear* somewhere in `root` not sufficient?**
Value membership ignores structure — a node in `root` can contain every value from `subRoot` while having additional children `subRoot` doesn't have, which is not a true subtree match. The check needs full structural equality (matching shape and values together, via `isSameTree`), not just value presence.

**Q5. State the time complexity of Subtree of Another Tree and justify it.**
O(n × m), where n is the size of `root` and m is the size of `subRoot`: in the worst case, `isSameTree` is attempted at every one of `root`'s n nodes, and each attempt costs up to O(m) to potentially walk all of `subRoot`.

**Q6. Why doesn't H2 fully substitute for Postgres in a persistence-layer test, even though both are relational databases?**
H2 doesn't enforce Postgres-specific SQL dialect behavior, constraint timing, or Postgres-only features (like native JSON column types) — a test suite fully passing against H2 can still fail against real Postgres in production, because H2 never actually exercised the database engine the application is really deployed against.

**Q7. Why does TestContainers need `@DynamicPropertySource` specifically, rather than a fixed connection string in `application.yml`?**
The container's Postgres instance is assigned a port dynamically by Docker at startup, to avoid clashing with anything else running locally — the actual connection URL isn't known until the container has already started, so it has to be read and injected at test-run time rather than hardcoded in advance.

**Q8. What does `@AutoConfigureTestDatabase(replace = Replace.NONE)` do, and why is it necessary alongside TestContainers?**
It tells `@DataJpaTest` not to fall back to its own default embedded-database substitution — without it, `@DataJpaTest`'s default behavior would silently override the real TestContainers Postgres connection with its own embedded database, defeating the entire point of using TestContainers.

---

## Day 48 — Tree DFS with Global State, and Kafka Fundamentals

**Q1. What specifically does Balanced Binary Tree's brute-force approach recompute redundantly, and what's its complexity?**
It calls `height()` independently at every node, and each call re-walks that node's entire subtree from scratch — the same subtree's height ends up computed many times over. This is O(n log n) for a balanced tree and degrades to O(n²) for a skewed one.

**Q2. What does the `-1` sentinel mean in the optimized Balanced Binary Tree solution, and why does every caller need to check for it before proceeding?**
`-1` means "an imbalance was already found somewhere below this point" — as opposed to a non-negative return, which is a genuine height with no imbalance found yet. Every caller checks for `-1` first specifically so the check can short-circuit: once imbalance is found anywhere, the failure just propagates straight up without computing any further heights on the rest of the tree.

**Q3. In Diameter of Binary Tree, what does the `height` helper return, versus what does it track as a side effect — and why are those two different things?**
It returns the node's height upward, because that's what the parent's own height computation needs. It separately tracks `maxDiameter` — the best `leftHeight + rightHeight` seen at any node — as a side effect, because no parent ever needs that value returned to it; it only ever needs comparing against a single running best.

**Q4. Why can't a plain local variable hold Diameter's running max in Java, and what are the two standard fixes?**
A local variable declared in the outer method isn't reachable for a separate recursive helper method to update — Java doesn't give a nested method write access to a caller's local variable. The two fixes are an instance field on the class (simplest, most common), or explicitly threading a mutable holder — typically a single-element array — through the recursive calls.

**Q5. Why does the diameter-defining path not necessarily pass through the root, and what does that imply about where the check has to happen?**
The longest path can lie entirely within one subtree, never touching the root at all. That's why `leftHeight + rightHeight` must be checked and compared against the running max at *every* node during the traversal, not just once at the root.

**Q6. Walk through exactly why `1 + min(minDepth(left), minDepth(right))` gives the wrong answer for a node with only one child.**
A `null` child contributes `0` to the `min` comparison, and since `0` is the smallest possible value, it wins the `min` regardless of what the real (non-null) side computes — incorrectly reporting depth `1`, as if the current node itself were a leaf. But a node with one child is explicitly not a leaf by the problem's own definition, so its minimum depth must continue down its one existing side instead.

**Q7. Why does the naive minimum-depth formula happen to give the correct answer on a perfectly balanced tree, even though it's wrong in general?**
A perfectly balanced tree has no nodes with exactly one child — every internal node has either zero or two children — so the specific case the naive formula mishandles (a single-child node) never actually occurs, and the bug never surfaces on that shape of input.

**Q8. What is a Kafka partition for, and what guarantee does Kafka make — and not make — about ordering?**
A partition is an independent, ordered log that a topic is split into, enabling parallel reads and writes across multiple consumers/producers. Kafka guarantees ordering only *within* a single partition — there's no guaranteed relative order between records in different partitions of the same topic.

**Q9. Why are sequential disk appends the mechanical reason Kafka achieves high throughput?**
Sequential writes (appending to the end of an existing file) are dramatically faster than random-access writes on both mechanical and SSD-backed disks. Kafka's append-only log design means every write is sequential by construction, which is what lets it sustain high throughput while still durably persisting every message to disk.

**Q10. What does a Kafka consumer group actually split among its members, and what happens if two separate consumer groups read the same topic?**
A consumer group splits a topic's *partitions* among its member consumers, so each partition is read by only one consumer within that group at a time. Two independent consumer groups each receive every message on the topic independently — one group's consumption has no effect on what another group sees.

---

## Day 49 — Tree BFS, and Week 7 Consolidation

**Q1. Why does the BFS level-order scaffold capture `queue.size()` into a variable before the inner loop, instead of just checking `queue.isEmpty()` inside it?**
At the start of each outer iteration, the queue holds exactly the current level's nodes. Capturing that count freezes it, so the inner loop processes precisely those nodes — even though it's simultaneously adding the *next* level's children to the same queue. Checking `isEmpty()` (or using the queue's live, changing size) instead would sweep newly-added children into the same pass as their own parents, collapsing every level into one.

**Q2. Describe the DFS-based alternative to Level Order Traversal, and explain what makes it correct.**
Pass the current depth down through a standard DFS, and the first time a given depth is reached, append a new list to the result at that index. It's correct because level-order grouping only requires knowing each node's depth — which a DFS can track just as well via a parameter — it isn't inherently tied to breadth-first traversal at all.

**Q3. In Right Side View, why does keeping "the last node processed in the inner loop" correctly give the rightmost node at each level?**
Children are always offered left child before right child, so nodes are dequeued in left-to-right order at every level. The last node processed in a given level's inner loop is therefore always the rightmost one at that depth, which is exactly the node visible from the right side.

**Q4. Which is cheaper on a completely skewed (linked-list-shaped) tree — BFS or DFS-based level order — and why does that reverse the usual intuition?**
BFS is cheaper here: its queue never holds more than one node at a time on a skewed tree (width 1), so its extra space drops close to O(1) beyond the output. DFS's call stack, by contrast, still costs O(n) — equal to the tree's height. This reverses the usual pattern, where BFS normally has the worse worst-case space (up to O(n) width on a balanced tree) and DFS is the safer one (O(log n) on a balanced tree).

**Q5. Why is `sum` declared as a `long` in Average of Levels rather than an `int`?**
To guard against silent overflow — a level with many nodes, each holding a value near `Integer.MAX_VALUE`, could overflow a 32-bit accumulator before the average is even computed, corrupting the result without throwing any error.

**Q6. What would happen in Average of Levels if the division `sum / levelSize` were computed without the `(double)` cast?**
Java would perform integer division, truncating any fractional part and silently flooring every level's average — e.g., a true average of `14.5` would compute as `14`. The cast to `double` on at least one operand is what forces floating-point division instead.

**Q7. State this week's DSA total two ways, and explain why there are two different correct numbers.**
The plan's own required-ladder-only count is 97, matching its own Day 49 claim exactly. Including this series' added extra practice, the fuller distinct-problem count is 127. The gap (30) is exactly the extra practice problems added on top of the required ladder across all seven weeks so far, including this week's three (Subtree of Another Tree, Minimum Depth, Average of Levels).

---

*65 questions total across 7 days. Curriculum map updated to reflect Week 7 in `00_Curriculum_Map.md`.*
