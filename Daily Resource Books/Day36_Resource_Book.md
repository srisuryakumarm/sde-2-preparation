# Day 36 — Linked List Cycles (Floyd's, Extended), and Spring Data JPA

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 35 Resource Book](Day35_Resource_Book.md)
**Next ▶:** [Day 37 Resource Book](Day37_Resource_Book.md)
**Companion to:** Day 36 of `Week_06_Revised.md`

---

## Recap

Day 34 gave you fast/slow pointers for exactly one job: finding the middle of a linked list (`slow` moves one step, `fast` moves two, and when `fast` runs out of list, `slow` is standing at the middle). Day 35 combined that mechanism with reversal (Palindrome Linked List) and introduced the dummy-head technique (Merge Two Sorted Lists), then used dummy-head again for a bounded reversal (LC 92) and built two genuinely different O(N log k) approaches to merging k lists (LC 23).

Today reuses the *exact same* fast/slow mechanism — same two pointers, same speeds — but points it at a completely different question. Instead of "where does the list end," today asks "does this list even *have* an end." The mechanism doesn't change; the stopping condition does. That's the whole story of Linked List Cycle, and it's worth holding onto that framing precisely, because interviewers will sometimes ask "how is this related to what you did with the middle-of-list problem?" and "same tool, different question" is the correct, complete answer.

On the theory side, today pivots to a fresh track: `todo-api` has been a live Spring Boot app with REST endpoints since Day 34, but it has never touched a database. Today it gets a real, persistent entity for the first time, using Spring Data JPA — built directly on Day 34's reflection concept, which resurfaces today doing genuinely new work.

---

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Prove — not just state — why the fast/slow mechanism is guaranteed to detect a cycle if one exists, using the closing-gap argument.
2. Solve Linked List Cycle and Linked List Cycle II, including deriving the phase-2 "reset to head" step's correctness from the underlying distance math, rather than memorizing it as a trick.
3. Recognize Floyd's Cycle Detection when it's disguised as a problem that never mentions a linked list at all.
4. Explain what an ORM does mechanically, define every JPA annotation used today, and get `todo-api` persisting a real entity to PostgreSQL.

---

## Concept Dependency Map

```
Day 34: fast/slow pointers (Middle of Linked List)
  — mechanism: slow +1 step, fast +2 steps, same loop, per iteration
        │
        ▼
Today, Phase 1 — SAME mechanism, NEW stopping condition:
  loop until fast runs off the end (no cycle) OR slow == fast (cycle found)
        │
        ▼
Today, Phase 2 (Cycle II only) — NEW step, derived from distance math:
  reset one pointer to head, advance both +1 step at a time
  → they meet exactly at the cycle's start node
        │
        ▼
Extension: the SAME two-phase mechanism, applied where there's no
linked list at all — only an implicit "next" function
  (Happy Number: next(n) = sum of squares of digits)
  (Find the Duplicate Number: next(i) = nums[i], array read as pointers)

─────────────────────────────────────────────────────────

Day 34: REST APIs, Spring Boot, reflection (a running program reading
its own classes'/methods'/annotations' metadata at runtime)
        │
        ▼
Today: Spring Data JPA
  ├─ @Entity/@Id/@Column — annotations Hibernate reads via reflection
  │  at startup to build a class ↔ table mapping
  └─ JpaRepository<T,ID> — an interface with ZERO code you write;
     Spring generates a working implementation at runtime, again via
     reflection — the same mechanism from Day 34, doing new work
```

---

## Part 1 — Linked Lists: Cycle Detection

### 🔗 Recap: Fast/Slow Pointers, the Mechanism (Day 34)

Two pointers start at `head`. Every iteration, `slow = slow.next` and `fast = fast.next.next`. Day 34 used this to find a middle: since `fast` covers ground twice as fast as `slow`, by the time `fast` reaches the end, `slow` has covered exactly half the distance. Nothing about the pointers themselves changes today — what changes is what you're watching for while they move.

---

### Problem 5: Linked List Cycle (LeetCode 141, Easy) — Pattern: Floyd's Cycle Detection

**Statement:** Given the `head` of a linked list, determine if the list contains a cycle — some node's `next` pointer eventually loops back to a node already visited, rather than terminating at `null`.

#### Approach 1 — HashSet of visited nodes

```java
public boolean hasCycleHashSet(ListNode head) {
    Set<ListNode> visited = new HashSet<>();
    ListNode current = head;
    while (current != null) {
        if (!visited.add(current)) {   // add() returns false if already present
            return true;
        }
        current = current.next;
    }
    return false;
}
```

Walk the list, recording every node reference visited. If you ever land on a node already in the set, you've looped back — a cycle exists. If you reach `null`, there's no cycle. **Time O(n), Space O(n).**

#### Approach 2 — Optimized: Floyd's Cycle Detection (fast/slow)

```java
public boolean hasCycle(ListNode head) {
    if (head == null) return false;
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            return true;
        }
    }
    return false;
}
```

**Why checking `fast != null && fast.next != null` is exactly right:** `fast.next.next` needs `fast.next` to already be non-null before it's safe to dereference `.next` a second time. Checking both conditions in the loop guard means the loop simply exits the moment either pointer *would* run off the end — which only happens when there is genuinely no cycle.

**Why this is guaranteed to find a cycle if one exists — the proof, not just the claim:** Once *both* pointers have entered the cycle, think of the gap between them, measured in steps along the cycle, going from `slow` forward to `fast`. Each iteration, `slow` advances 1 and `fast` advances 2, so `fast` gains exactly 1 step on `slow` — the gap shrinks by exactly 1 every single iteration. A cycle has some finite length `c`. The gap starts somewhere in the range `0` to `c-1` the moment both pointers are inside the cycle, and because it decreases by exactly 1 each step (never jumps by 2 or more), it cannot skip over 0 — it *must* hit exactly 0 within at most `c` further iterations. A gap of 0 means `slow == fast`. So meeting is not probabilistic or "usually true" — it's forced by the arithmetic.

**Worked trace.** Build a list where the cycle doesn't start at the head, since that's the more general (and more interesting) case:

```
H → A → B → C → D → E → F
        ↑___________________|
        (F.next = C — the cycle starts at C, length 4: C→D→E→F→C)
```

| t | slow | fast | note |
|---|---|---|---|
| 0 | H | H | start |
| 1 | A | B | fast: H→A→B |
| 2 | B | D | fast: B→C→D |
| 3 | C | F | fast: D→E→F |
| 4 | D | D | fast: F→C→D — **slow == fast** |

They meet at `D` after 4 iterations. Cycle correctly detected.

**Complexity: Time O(n) — each pointer visits a bounded number of nodes before either exiting or meeting. Space O(1) — exactly two pointer variables, regardless of list length.**

**Edge cases:**
- `head == null`: the initial guard returns `false` immediately — an empty list has no cycle.
- Single node, `node.next == null`: loop guard fails on the first check (`fast.next` is `null`) — correctly reports no cycle.
- Single node, `node.next == node` (self-loop): `fast` becomes `node.next.next = node`, `slow` becomes `node.next = node` — both land on `node` in one iteration — correctly reports a cycle.
- Two nodes, no cycle: `fast` reaches `null` cleanly after one iteration.

**💡 Interview Insight:** this algorithm has a widely-used nickname — "the tortoise and the hare" — worth knowing, since interviewers often refer to it that way without saying "Floyd's" explicitly. Also worth saying out loud unprompted: the HashSet approach is *correct* and worth mentioning first as your brute force, but the fast/slow approach is the one that demonstrates you actually understand the pointer arithmetic rather than reaching for extra memory as a default.

---

### Problem 6: Linked List Cycle II (LeetCode 142, Medium) — Pattern: Floyd's, Extended **(new)**

**Statement:** Given `head`, return the node where the cycle begins. If there is no cycle, return `null`. (Unlike Problem 5, this asks *where*, not just *whether*.)

#### Approach 1 — HashSet

```java
public ListNode detectCycleHashSet(ListNode head) {
    Set<ListNode> visited = new HashSet<>();
    ListNode current = head;
    while (current != null) {
        if (!visited.add(current)) {
            return current;   // first node seen twice IS the cycle's start
        }
        current = current.next;
    }
    return null;
}
```

The first node encountered a *second* time is, by definition, the cycle's entry point — everything before it was visited exactly once. **Time O(n), Space O(n).**

#### Approach 2 — Optimized: Floyd's, Two Phases

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    // Phase 1 — identical to Problem 5: find a meeting point inside the cycle
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            // Phase 2 — reset one pointer to head, advance both one step at a time
            ListNode ptr1 = head;
            ListNode ptr2 = slow;
            while (ptr1 != ptr2) {
                ptr1 = ptr1.next;
                ptr2 = ptr2.next;
            }
            return ptr1;   // == ptr2 — the cycle's start node
        }
    }
    return null;   // fast reached the end cleanly — no cycle
}
```

**Phase 1 is a direct reuse of Problem 5 — nothing new there.** Phase 2 is the genuinely new idea, and it deserves a real proof, not just "trust the trick":

**Setup.** Let `a` = distance from `head` to the cycle's start. Let `b` = distance from the cycle's start to the meeting point found in Phase 1 (measured going forward around the cycle). Let `c` = the cycle's total length.

**The distance equations.** When `slow` and `fast` meet:
- `slow` has traveled `a + b` steps total (walked `a` steps to reach the cycle, then `b` more steps around it).
- `fast` has traveled `a + b + n·c` steps total, for some positive integer `n` — it entered the cycle at the same point, but being twice as fast, it lapped the cycle `n` extra times before `slow` caught up to being met.
- `fast` always travels exactly twice as far as `slow`, since it takes 2 steps for every 1 of `slow`'s: `a + b + n·c = 2(a + b)`.

**Solving for `a`:**
```
a + b + nc = 2a + 2b
nc = a + b
a  = nc − b
   = (n−1)c + (c − b)
```

`(c − b)` is exactly the *remaining* distance from the meeting point forward to the cycle's start (since `b` was the distance already covered from the start to the meeting point). So the equation says: **walking `a` steps from `head` lands you in the same place as walking `(c − b)` steps from the meeting point, plus `(n−1)` complete extra trips around the cycle** — and a complete trip around the cycle always returns you to exactly where you started. Those extra trips don't change *where* you end up, only how many steps it takes to get there.

That's why Phase 2 works: `ptr1` (from `head`) and `ptr2` (from the meeting point) move in lockstep, one step each. After `a` steps, `ptr1` is standing exactly on the cycle's start — and by the equation above, `ptr2` has *also* landed exactly on the cycle's start (having covered `c − b` steps plus zero or more complete loops). They arrive at the same node at the same time, and that node is provably the cycle's start — not merely "a node inside the cycle."

**Worked trace**, continuing the exact same list from Problem 5's trace (`H→A→B→C→D→E→F→C`, cycle start `C`, `a=3`, cycle length `c=4`):

Phase 1 already found the meeting point at `D` (from the trace above). Here, `b` = distance from `C` to `D` = 1. Check the equation: `slow` traveled 4 steps total = `a+b` = `3+1` = 4 ✓. `fast` traveled 8 steps total = `a+b+nc` = `3+1+4n` = `8` ⟹ `n=1` ✓ (fast looped the cycle exactly once extra).

Phase 2: `ptr1` starts at `H`, `ptr2` starts at `D`.

| step | ptr1 | ptr2 |
|---|---|---|
| 0 | H | D |
| 1 | A | E |
| 2 | B | F |
| 3 | C | C |

Both reach `C` after exactly 3 steps — matching `a = 3` exactly, and confirming `(c−b) = 4−1 = 3` with zero extra loops needed here (`n−1 = 0`). **`C` is correctly returned as the cycle's start.**

**Complexity: Time O(n) — Phase 1 is bounded exactly as in Problem 5; Phase 2 takes at most `a` steps, itself bounded by `n`. Space O(1).**

**Edge cases:**
- No cycle: Phase 1's loop exits via `fast` reaching `null`; the function returns `null` without ever entering Phase 2.
- Cycle starts at `head` (`a = 0`): Phase 2's `while (ptr1 != ptr2)` never executes even once, since `ptr1 = head` is already the meeting-point-derived answer — correctly returns `head` immediately.

**💡 Interview Insight:** "Why does resetting one pointer to `head` work?" is one of the single most commonly asked follow-ups in the entire linked-list interview repertoire. An answer that reproduces the `a = (n−1)c + (c−b)` derivation, even briefly and without a whiteboard, separates candidates who memorized a trick from candidates who understand it — and it's exactly the kind of thing worth saying *before* being asked "but why does that work," since volunteering it is a stronger signal than only producing it under direct questioning.

---

## Extension (optional, time-permitting) — Floyd's Beyond Linked Lists

Everything above proves the fast/slow mechanism works whenever there's a well-defined "next" step and a finite space of possible positions — nothing in the proof actually required an explicit `ListNode` object. These two problems make that generalization concrete. They're genuinely useful interview reps for this pattern, but they're not required for today's deliverable — treat them as skippable if today is already full.

### Extra Practice: Happy Number (LeetCode 202, Easy)

**Statement:** Define `next(n)` as the sum of the squares of `n`'s digits. Starting from a positive integer, repeatedly apply `next`. The number is "happy" if this process reaches `1`. Return `true` if `n` is happy, `false` if the process loops forever without reaching `1`.

**The reframe:** there is no linked list anywhere in this problem statement — but `next(n)` *is* a "next pointer." Every positive integer is a "node"; `next(n)` tells you which "node" comes after it. "Loops forever" is exactly "this implicit list has a cycle." "Reaches 1" is exactly "this implicit list terminates" (a cycle of length 1, `1 → 1`, is the only way this sequence's cycle can include 1 — either you land on `1` and stay there, or you enter some other cycle that never includes 1).

```java
public boolean isHappy(int n) {
    int slow = n;
    int fast = n;
    do {
        slow = nextNumber(slow);
        fast = nextNumber(nextNumber(fast));
    } while (slow != fast);
    return slow == 1;
}

private int nextNumber(int n) {
    int sum = 0;
    while (n > 0) {
        int digit = n % 10;
        sum += digit * digit;
        n /= 10;
    }
    return sum;
}
```

This is Problem 5's exact structure — a `do/while` instead of `while` only because both pointers legitimately start at the same value `n` and need to take at least one step before the equality check means anything (unlike a linked list, where `head` and a fast-started `head` genuinely differ in position after one real iteration).

**Worked trace, `n = 19` (happy):** `19 → 1²+9²=82 → 8²+2²=68 → 6²+8²=100 → 1²+0²+0²=1`. Reaches `1` — happy.

**Worked trace, `n = 2` (not happy):** `2 → 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4` — `4` reappears, confirming a cycle (`4→16→37→58→89→145→42→20→4→...`) that never includes `1`.

**Complexity:** bounding this precisely requires knowing that for any number with `d` digits, `next(n) ≤ 81d` — so `next` rapidly shrinks any large number down into a small bounded range (at most 3 digits within a couple of applications), after which the state space is small and finite. Both approaches (HashSet or Floyd's) are effectively O(log n) amortized in practice, since the shrinking happens fast; the important claim to make out loud is simply that the sequence is *eventually* bounded, which is what guarantees a cycle must exist if `1` is never reached — an unbounded sequence would make "must cycle" unprovable. **Space:** O(1) for Floyd's vs. O(k) for a HashSet, where `k` is the number of distinct values visited before repeating.

**💡 Interview Insight:** the moment worth naming out loud is the reframe itself — recognizing that "a process that repeats forever without reaching a target value" is structurally identical to "a linked list with a cycle," despite the problem never mentioning pointers or nodes. That recognition, stated before writing any code, is worth more here than the code itself.

---

### Extra Practice: Find the Duplicate Number (LeetCode 287, Medium)

**Statement:** Given an array `nums` of `n + 1` integers, where every value is in the range `[1, n]`, exactly one value is duplicated (it may appear more than twice). Find it — **without modifying the array, and using only O(1) extra space.**

**Why this constraint rules out the obvious approaches:** a HashSet is O(n) space, not O(1). Sorting either mutates the input or costs O(n) space for a copy. Those constraints are the tell that something structural is expected instead.

**The reframe:** treat each index `i` as a "node," and treat the *value stored at* `nums[i]` as that node's "next pointer" — it tells you which index to go to next. Because every value is in `[1, n]` while indices run `0` to `n`, index `0` is never *pointed to* by any value (values start at 1) — so starting a traversal at index `0` is always safe, exactly like a `head` node that sits outside any cycle. Because there are `n+1` values crammed into only `n` possible "destinations" (`1` through `n`), the pigeonhole principle guarantees at least one value is hit twice — which means at least two different indices point to the same place, which is exactly what creates a cycle in this implicit structure. **The node where that cycle begins is provably the duplicated value itself** — the first place two different paths converge.

```java
public int findDuplicate(int[] nums) {
    int slow = nums[0];
    int fast = nums[0];

    // Phase 1 — find a meeting point inside the cycle
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);

    // Phase 2 — identical to Linked List Cycle II
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
```

**Worked trace, `nums = [1, 3, 4, 2, 2]`** (`n = 4`, values in `[1,4]`, duplicate is `2`):

Phase 1:
| step | slow | fast |
|---|---|---|
| init | `nums[0]=1` | `nums[0]=1` |
| 1 | `nums[1]=3` | `nums[nums[1]]=nums[3]=2` |
| 2 | `nums[3]=2` | `nums[nums[2]]=nums[4]=2` |

`slow == fast == 2` — meeting point found.

Phase 2 (`slow` reset to `nums[0]=1`, `fast` stays at `2`):
| step | slow | fast |
|---|---|---|
| init | 1 | 2 |
| 1 | `nums[1]=3` | `nums[2]=4` |
| 2 | `nums[3]=2` | `nums[4]=2` |

Both land on `2` — **correctly returns the duplicate, `2`.**

**Complexity: Time O(n), Space O(1)** — meeting Phase 1's constraint exactly, unlike the HashSet alternative.

**⚠️ Common Mistake:** reaching for `Arrays.sort()` or a `HashSet<Integer>` out of habit, without registering that the problem statement is explicitly ruling both out. Naming the constraint and *then* explaining why it forces this specific technique is a much stronger interview answer than silently arriving at the optimal solution — it shows the constraint was read, not just the problem title.

**💡 Interview Insight:** this problem is a favorite specifically *because* it looks nothing like a linked-list problem on first read. "This is Cycle II, applied to an array where the values themselves are the pointers" is the single sentence that unlocks it — worth saying immediately once you spot it, since arriving at that sentence is the actual difficulty of this problem, not the two-phase mechanics that follow from it.

---

## Part 2 — Spring Data JPA: `todo-api` Gets Real Persistence

### Prerequisites (confirmed)

- REST APIs, Spring Boot, HTTP verbs/status codes — Day 34.
- **Reflection** — a running program inspecting its own classes, methods, and annotations *at runtime* — Day 34. Today reuses this mechanism directly, for a new purpose.

### What an ORM actually does, and why one exists

**The problem:** your `Task` objects live in Java as instances with fields and methods. A relational database stores *rows* in *tables*, related through keys. These are fundamentally different shapes — this mismatch has a name, **the object-relational impedance mismatch** — and without help, every save/load operation means hand-writing `INSERT`/`SELECT`/`UPDATE` SQL and manually copying values between `ResultSet` columns and object fields.

**Definition — ORM (Object-Relational Mapper):** a library that lets you describe your data as plain objects, and translates between those objects and database rows automatically, in both directions.

**Where today's three names fit together:**
- **JPA (Jakarta Persistence API)** — a *specification*: a set of interfaces and annotations (`@Entity`, `@Id`, `EntityManager`, …) describing what an ORM for Java should look like. JPA itself has no implementation.
- **Hibernate** — the most widely used *implementation* of that specification. It's the library actually doing the work: reading your annotated classes, generating SQL, executing it, and mapping results back to objects.
- **Spring Data JPA** — a layer *on top of* Hibernate/JPA that removes even more boilerplate, most visibly by letting you declare a repository as a bare interface with no method bodies at all, which Spring then implements for you at runtime.

### `@Entity`, `@Id`, `@Column` — what Hibernate reads via reflection

```java
@Entity
@Table(name = "tasks")
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(length = 2000)
    private String description;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private TaskStatus status;

    @Enumerated(EnumType.STRING)
    private Priority priority;

    private LocalDate dueDate;

    // JPA requires a no-args constructor. Hibernate uses reflection to
    // instantiate entities directly and populate fields, bypassing any
    // other constructor entirely — this one exists purely for Hibernate.
    protected Task() {
    }

    public Task(String title, String description, TaskStatus status, Priority priority, LocalDate dueDate) {
        this.title = title;
        this.description = description;
        this.status = status;
        this.priority = priority;
        this.dueDate = dueDate;
    }

    public Long getId() { return id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    public TaskStatus getStatus() { return status; }
    public void setStatus(TaskStatus status) { this.status = status; }
    public Priority getPriority() { return priority; }
    public void setPriority(Priority priority) { this.priority = priority; }
    public LocalDate getDueDate() { return dueDate; }
    public void setDueDate(LocalDate dueDate) { this.dueDate = dueDate; }
}

public enum TaskStatus { TODO, IN_PROGRESS, DONE }
public enum Priority { LOW, MEDIUM, HIGH }
```

- **`@Entity`** marks a class as one Hibernate should manage — at startup, Hibernate scans for every `@Entity`-annotated class using reflection (Day 34's mechanism, now doing real work) and builds a mapping from that class to a table.
- **`@Id`** marks the primary-key field. **`@GeneratedValue(strategy = GenerationType.IDENTITY)`** delegates ID generation to the database itself (an auto-incrementing column) rather than Java choosing IDs.
- **`@Column`** customizes the mapped column — `nullable = false` adds a `NOT NULL` constraint; `length` bounds a `VARCHAR`'s size. A field with no `@Column` at all still gets mapped — using the field name and Hibernate's default type inference — `@Column` only needed when you want to *override* a default.
- **`@Enumerated(EnumType.STRING)`** is worth flagging as a real, sharp gotcha: **the default, if you omit this annotation entirely, is `EnumType.ORDINAL`** — it stores the enum constant's *integer position* (`TODO=0`, `IN_PROGRESS=1`, `DONE=2`). That's fragile: reordering the enum's constants later — or inserting a new one in the middle — silently changes what every previously stored integer *means*, with no error at read time. `EnumType.STRING` stores the constant's *name* instead, which is unaffected by reordering. Defaulting to `STRING` deliberately, rather than relying on the ordinal default, is a genuinely good habit worth stating out loud if asked about JPA pitfalls.
- **`@Table(name = "tasks")`** overrides the default table name (which would otherwise be derived from the class name). Not strictly required here — Spring Boot's default naming strategy would already produce `tasks` from `Task` — but stating it explicitly documents the mapping rather than leaving it implicit.

### `JpaRepository<T, ID>` — an interface with no implementation

```java
public interface TaskRepository extends JpaRepository<Task, Long> {
}
```

That's the entire file. No class implements it anywhere in your source tree — and yet `taskRepository.save(task)`, `.findById(id)`, `.findAll()`, `.deleteById(id)` all work. **How:** at startup, Spring scans for interfaces extending `JpaRepository`, and — using reflection once again — generates a working implementation *at runtime* as a dynamic proxy, backing every inherited method with real JPA/Hibernate calls. This is precisely the same mechanism Day 34 introduced for `@RestController`/`@GetMapping` routing, now applied to a different kind of interface. `JpaRepository<Task, Long>` itself is generic (Day 16) in the simplest way you've used generics so far: `Task` is the entity type, `Long` is the type of its `@Id` field — Spring uses both type parameters to know what to generate.

**Worth knowing for depth:** Spring Data also supports *derived query methods* — declaring a method like `List<Task> findByStatus(TaskStatus status)` on the interface, with no body, and Spring parses the method *name itself* to build the corresponding query. Not needed for today's task, but a very common interview question about "how does a repository with no implementation actually work" expects exactly this answer.

### `application.yml` — connecting to PostgreSQL

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/tododb
    username: postgres
    password: postgres
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```

**`ddl-auto: update`** tells Hibernate to inspect your `@Entity` classes at startup and automatically create or alter tables to match — genuinely convenient for early development, since you never hand-write a `CREATE TABLE`. **⚠️ Common Mistake, flagged now for later:** this same convenience is a real liability once real data exists — an auto-generated `ALTER TABLE` can silently drop a column Hibernate no longer sees a mapping for, with no confirmation step. Day 40 replaces this with Flyway-managed, version-controlled migrations and switches this setting to `validate` (schema must already match — Hibernate stops trusting itself to change it). `show-sql`/`format_sql` are purely diagnostic — they print the generated SQL to the console so you can see exactly what Hibernate is doing under the hood, which is worth leaving on while you're still building intuition for what an ORM generates.

**⚠️ Common Mistake:** forgetting the no-args constructor. Hibernate instantiates entities via reflection *before* populating fields — if the only constructor requires arguments, instantiation fails at the reflection step, before your field-population logic is even relevant. It's conventional to make this constructor `protected` (not `public`) — visible to Hibernate and subclasses, but not invitingly available to your own application code, which should go through the real constructor instead.

---

## Project Block Guide (1.5 hrs)

**Repository:** `todo-api`. **Task:** define `Task`, `TaskStatus`, `Priority`, and `TaskRepository` as above; configure PostgreSQL in `application.yml`.

**Practical steps:**
1. Add `spring-boot-starter-data-jpa` and the `postgresql` driver to your `pom.xml` (or `build.gradle`) if they aren't already there from Day 34's scaffolding.
2. Have a local PostgreSQL instance reachable at the URL in `application.yml` — a quick way, if you don't already have Postgres installed: `docker run --name todo-postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=tododb -p 5432:5432 -d postgres` (Docker itself is Day 43's topic — using it as a one-line convenience here doesn't require understanding it yet).
3. Add the entity, enums, and repository files exactly as above.
4. Start the app and watch the console (thanks to `show-sql`) — you should see a generated `CREATE TABLE` statement (or an `ALTER TABLE`, on a second run) execute against your database.
5. Verify directly: `psql -U postgres -d tododb -c '\d tasks'` should show all six columns with the types Hibernate inferred.

**Definition of done:** the app connects to local Postgres, and Hibernate creates the `tasks` table with all six columns correctly typed.

## Career Block Guide (1 hr)

**LinkedIn (20 min):** engagement only today — comment thoughtfully on 3–5 posts in your feed related to backend engineering, Java, or interview prep. A genuinely useful comment (a specific observation or a real follow-up question) is worth more to your visibility than volume.

**Networking:** follow up on Day 34's outreach. Concretely, that means checking whether any of those contacts responded, and if so, replying promptly and specifically — referencing what they said, not sending a generic thank-you. If someone hasn't responded, this is not yet the day to send a second message — give it more time before following up again.

---

## Day 36 — Interview Questions

**Q1. Explain the fast/slow pointer mechanism, and prove it must detect a cycle if one exists.** Two pointers start at `head`; each iteration, `slow` advances one node, `fast` advances two. If a cycle exists, once both pointers are inside it, the gap between them (measured along the cycle) shrinks by exactly 1 every iteration — since it can never skip past 0, it must hit exactly 0 within at most the cycle's length in further steps, forcing a meeting.

**Q2. Why does Cycle II reset a pointer to `head` instead of continuing from the meeting point?** Because the distance from `head` to the cycle's start (`a`) is provably equal to the distance from the meeting point forward to the cycle's start (`c − b`), modulo whole trips around the cycle — derived from `a + b + nc = 2(a+b)`. Advancing both pointers one step at a time from those two starting points therefore lands them on the same node — the cycle's start — at the same time.

**Q3. Does the two-phase approach ever fail to find the true cycle start?** No — the derivation doesn't depend on which specific meeting point Phase 1 happens to land on; any valid `b` satisfies the same equation, so Phase 2 is correct regardless of exactly where inside the cycle the pointers first meet.

**Q4. How would you recognize that Happy Number is a cycle-detection problem, given that it never mentions a linked list?** Any deterministic process with a well-defined "next state" and a bounded set of possible states either terminates or eventually revisits a state — which is structurally a linked list with a cycle, whether or not any object with a `.next` field exists.

**Q5. Why does Find the Duplicate Number specifically forbid sorting or extra data structures, and how does that shape the solution?** Those constraints (no modification, O(1) space) rule out both an O(n) HashSet and an in-place sort's implicit assumptions, forcing a technique that uses no extra memory — which is exactly what Floyd's, reused via treating array values as implicit pointers, provides.

**Q6. What does an ORM actually solve?** The object-relational impedance mismatch — Java objects and relational rows are structurally different shapes; an ORM translates between them automatically, so code works with plain objects instead of hand-written SQL and manual `ResultSet` mapping.

**Q7. What's the relationship between JPA, Hibernate, and Spring Data JPA?** JPA is a specification (interfaces/annotations, no implementation). Hibernate is the most common implementation of that specification, doing the real mapping/SQL work. Spring Data JPA sits on top of both, removing further boilerplate — most visibly, generating a working repository implementation from a bare interface.

**Q8. Mechanically, how does `JpaRepository<Task, Long>` work with no implementation class anywhere?** At startup, Spring scans for interfaces extending `JpaRepository`, and — using reflection — generates a dynamic proxy implementation at runtime, backing every method with real JPA calls. The two generic type parameters tell Spring the entity type and its ID type.

**Q9. What's the risk in leaving `ddl-auto: update` on permanently?** Hibernate can alter or drop columns automatically based on what it currently sees mapped, with no review step — safe for early development, but risky once real data exists, since an unintended schema change (or a dropped column no longer mapped) happens silently.

**Q10. What does `@Enumerated(EnumType.STRING)` protect against, and what happens if you omit it?** The default without this annotation is `EnumType.ORDINAL`, which stores an enum constant's integer position — reordering or inserting a new constant later silently changes what previously stored values mean. `EnumType.STRING` stores the constant's name instead, which is stable across reordering.

**Q11. Why does the `Task` entity need a no-args constructor?** Hibernate instantiates entities via reflection, populating fields directly, before any of your own constructor logic would run — a constructor requiring arguments would make reflective instantiation fail at that step.

---

## Daily Deliverable Check

- [ ] Linked List Cycle (LC 141) and Linked List Cycle II (LC 142) solved, both approaches understood, pushed to `dsa-java/linked-lists/`.
- [ ] Can reproduce the Cycle II distance-math proof from memory, not just the two-phase steps.
- [ ] *(Optional, time-permitting)* Happy Number (LC 202) and Find the Duplicate Number (LC 287) solved as extension practice.
- [ ] `Task` entity, `TaskStatus`/`Priority` enums, and `TaskRepository` committed; `tasks` table auto-generated and verified against a running Postgres instance.
- [ ] LinkedIn engagement done. Day 34 outreach followed up on.

---

## What Tomorrow Assumes You Already Know Cold

Day 37 assumes the fast/slow mechanism and dummy-head technique are both fully reflexive — Remove Nth Node From End reuses a fixed-offset variant of two pointers without re-deriving the base idea, and both of tomorrow's problems build directly on Day 34–36's pointer-manipulation fluency. It also assumes `todo-api` is now genuinely persisting `Task` entities, since nothing about tomorrow's theory (concurrency) touches the database further — today's project work stands on its own. Tomorrow's theory pivots to a different track entirely: Day 29's Threads/JVM concurrency model needs to be solid, since Day 37 builds real locking directly on top of it — including the first demonstration in this series of actual data *corruption* from a race condition, not just non-deterministic ordering.
