# Day 40 — Stacks Continue, and SQL Fundamentals

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 39 Resource Book](Day39_Resource_Book.md)
**Next ▶:** [Day 41 Resource Book](Day41_Resource_Book.md)
**Companion to:** Day 40 of `Week_06_Revised.md`

---

## ⚠️ Overlap Notice — Read This First

One of today's two required problems is a repeat — flagged in `00_Curriculum_Map.md` since Week 1.

| LC # | Problem | Status |
|---|---|---|
| 232 | Implement Queue using Stacks | Already solved — Week 1, Day 4 (Extra Practice). **Recap only.** |
| 496 | Next Greater Element I | Genuinely new — opens Monotonic Stack. **Full depth below.** |

No substitute problem is added today, and this is a deliberate choice, not an oversight: today is Monotonic Stack's opening day (LC 496 is its first problem), and per this series' own precedent, a freshly-opening pattern doesn't get bonus practice on the day it opens — the same treatment Stacks itself received yesterday. The time freed by recapping LC 232 is reinvested in giving Next Greater Element I full, unhurried depth, since it's introducing a genuinely new way of thinking about a stack, not just another application of LIFO matching. If you want more Monotonic Stack reps, tomorrow's schedule has real room for them — see Day 41.

---

## Recap

Yesterday formalized Stacks as LIFO and closed Linked Lists. Today's recap (Implement Queue using Stacks) is a different *use* of the same LIFO container — building a FIFO structure *out of* two LIFO ones — worth seeing again briefly since it's a genuinely clever composition, even though it's not new material. Today's real new content, Next Greater Element I, uses a stack in a way that's a step beyond Valid Parentheses' matching: instead of checking whether things match, it's about maintaining a specific *invariant* — a monotonic order — among whatever's currently on the stack.

Theory pivots entirely away from concurrency today: `todo-api` has had a working `Task` entity and JPA repository since Day 36, but nothing so far has looked at the actual SQL being generated underneath. Today does.

---

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Explain why the two-stack queue's amortized cost is O(1), even though a single `pop()` call can occasionally cost O(n).
2. State, and prove, why a stack maintained during a single left-to-right scan of an array is guaranteed to stay monotonic — not just observe that it happens to.
3. Prove Next Greater Element I is O(n) overall despite a `while` loop nested inside a `for` loop, using an amortized "each element pushed and popped at most once" argument.
4. State the precise difference between what a `LEFT JOIN` and an `INNER JOIN` do with a non-matching row, and explain why `NOT IN` can silently return zero rows in a way `NOT EXISTS` never does.

---

## Concept Dependency Map

```
Yesterday: Stacks, LIFO, ArrayDeque, Valid Parentheses (matching)
        │
        ├──▶ 🔗 Recap: Implement Queue using Stacks (LC 232)
        │      two stacks, opposite roles, amortized O(1)
        │
        └──▶ Today, NEW sub-pattern: Monotonic Stack
               a stack maintained to stay strictly ordered —
               not for matching, but for tracking "candidates
               still waiting for something bigger/smaller"
                    │
                    ▼
             Next Greater Element I (LC 496) — first Monotonic
             Stack problem; amortized O(n) despite nested loops

─────────────────────────────────────────────────────────

Day 36: @Entity/@Id/@Column, JpaRepository, ddl-auto: update
        │
        ▼
Today, NEW: SQL Fundamentals — keys, normalization, JOINs, indexes
  (what JPA has been generating underneath, made explicit)
        │
        ▼
Project: Flyway — versioned migration replaces ddl-auto: update,
        writing the exact CREATE TABLE JPA was inferring automatically
```

---

## Part 1 — Stacks Continue

### 🔗 Recap: Implement Queue using Stacks (LeetCode 232, Easy)

**Original coverage:** Week 1, Day 4, Extra Practice. Full solution, for reference:

```java
class MyQueue {
    private Deque<Integer> inStack = new ArrayDeque<>();
    private Deque<Integer> outStack = new ArrayDeque<>();

    public void push(int x) {
        inStack.push(x);
    }

    public int pop() {
        transferIfNeeded();
        return outStack.pop();
    }

    public int peek() {
        transferIfNeeded();
        return outStack.peek();
    }

    public boolean empty() {
        return inStack.isEmpty() && outStack.isEmpty();
    }

    private void transferIfNeeded() {
        if (outStack.isEmpty()) {
            while (!inStack.isEmpty()) {
                outStack.push(inStack.pop());
            }
        }
    }
}
```

`push()` always goes to `inStack`. `pop()`/`peek()` operate on `outStack`, refilling it from `inStack` — one element at a time, popped and pushed — only when `outStack` is empty. That refill reverses the order, which is exactly what turns LIFO into FIFO: the *oldest* element in `inStack` was pushed first, so it's at the *bottom* of `inStack` — and pushing everything onto `outStack` one at a time puts that oldest element on *top* of `outStack`, ready to be the next one out.

**🔑 Key Takeaway (recap): why this is amortized O(1), not O(n), despite a transfer that's clearly O(n) in isolation.** Track any single element's total lifetime cost: it's pushed onto `inStack` once, and — at some later point — popped off `inStack` and pushed onto `outStack` once, then eventually popped off `outStack` once. That's a fixed, constant number of operations per element, no matter how many times `pop()`/`peek()` are called in between (a `peek()` that doesn't need to refill costs O(1) directly). Spread over `n` total elements, total work is O(n) for any sequence of `n` operations — averaging to O(1) *amortized* per operation, even though any single `pop()` call can occasionally trigger the full O(n) transfer.

---

### Problem 3: Next Greater Element I (LeetCode 496, Easy) — Pattern: Monotonic Stack + HashMap

**Statement:** `nums1` is a subset of `nums2` (`nums2` has distinct values). For every element of `nums1`, find its *next greater element* in `nums2` — the first element to its right in `nums2` that's strictly larger — or `-1` if none exists. Return the answers, in `nums1`'s order.

#### Approach 1 — Brute force

```java
public int[] nextGreaterElementBruteForce(int[] nums1, int[] nums2) {
    int[] result = new int[nums1.length];
    for (int i = 0; i < nums1.length; i++) {
        int target = nums1[i];
        int j = indexOf(nums2, target);
        int answer = -1;
        for (int k = j + 1; k < nums2.length; k++) {
            if (nums2[k] > target) {
                answer = nums2[k];
                break;
            }
        }
        result[i] = answer;
    }
    return result;
}
```

For each query, scan right in `nums2` until something bigger turns up. **Time O(n·m)** (`n` = `nums2` length, `m` = `nums1` length, ignoring the index lookup) — a strictly decreasing `nums2` forces every query to scan almost the entire remaining array.

#### Approach 2 — Optimized: Monotonic Stack, built in one pass over `nums2`

**The idea, stated before the code:** scan `nums2` left to right, keeping a stack of values seen so far that are *still waiting* for something bigger to their right — meaning nothing larger has appeared yet. When a new value arrives, it might be the answer for *several* waiting values at once — pop every stack value smaller than it (recording the new value as each one's answer), then push the new value itself, since it's now waiting for its own answer.

```java
public int[] nextGreaterElement(int[] nums1, int[] nums2) {
    Map<Integer, Integer> nextGreater = new HashMap<>();
    Deque<Integer> stack = new ArrayDeque<>();   // values still waiting for a "next greater"

    for (int num : nums2) {
        while (!stack.isEmpty() && stack.peek() < num) {
            nextGreater.put(stack.pop(), num);
        }
        stack.push(num);
    }
    // anything left on the stack when the scan ends has no next greater

    int[] result = new int[nums1.length];
    for (int i = 0; i < nums1.length; i++) {
        result[i] = nextGreater.getOrDefault(nums1[i], -1);
    }
    return result;
}
```

**Why the stack is guaranteed to stay monotonically decreasing, bottom to top — proof, not observation:** every value still on the stack is there specifically *because* nothing bigger than it has appeared yet — that's the exact condition for staying on the stack rather than being popped. Consider any two values on the stack, with `x` below `y`: `x` was pushed *before* `y`, and if `y` had been ≥ `x`, then when `y` arrived, the `while` loop would have popped `x` right then (recording `y` as `x`'s answer) — `x` could only still be sitting below `y` if `y` was smaller than `x` when it was pushed. So every element is smaller than everything beneath it, by construction, at every point during the scan — the stack cannot ever *become* non-monotonic, because the loop actively enforces it on every single push.

**Worked trace, `nums2 = [1, 3, 4, 2]`, `nums1 = [4, 1, 2]`:**

| num | stack before | action | stack after | map updates |
|---|---|---|---|---|
| 1 | `[]` | push | `[1]` | — |
| 3 | `[1]` | `1<3` → pop 1, map[1]=3 | `[3]` | `{1:3}` |
| 4 | `[3]` | `3<4` → pop 3, map[3]=4 | `[4]` | `{1:3, 3:4}` |
| 2 | `[4]` | `4<2`? No → stop | `[4, 2]` | `{1:3, 3:4}` |

Scan ends. `[4, 2]` remains on the stack — decreasing bottom to top, exactly as proven above — and neither gets a map entry (no next-greater exists for either).

Lookups for `nums1 = [4, 1, 2]`: `4 → -1` (never popped — confirmed correct: nothing after `4` in `[1,3,4,2]` is bigger than it), `1 → 3`, `2 → -1` (last element, nothing to its right). **Result: `[-1, 3, -1]`.**

**Complexity: Time O(n + m)** — `n` = `nums2`'s length for the stack scan, `m` = `nums1`'s length for the final lookups. **The stack scan itself is O(n), not O(n²), despite the nested `while` inside the `for`** — here's the amortized argument, worth being able to reproduce exactly: every element of `nums2` is pushed onto the stack **exactly once**, and popped **at most once**, over the *entire* run of the algorithm (once popped, it's gone for good — it's never pushed again). Total push operations = n. Total pop operations ≤ n. Total work across the whole scan is therefore bounded by `2n` — linear, regardless of how the pops happen to be distributed across iterations. **Space: O(n)** for the stack and map.

**A worthwhile observation on the two extremes:** a strictly *decreasing* `nums2` (e.g., `[4,3,2,1]`) triggers **zero** pops — every element just gets pushed and stays — while a strictly *increasing* `nums2` (e.g., `[1,2,3,4]`) triggers a pop on almost every iteration, keeping the stack tiny. Both are still O(n) total work; the amortized argument holds regardless of which extreme (or anything in between) actually occurs.

**Edge cases:**
- Query value with no next greater: correctly defaults to `-1` via `getOrDefault`.
- `nums2` strictly increasing: every element's immediate successor is its answer; the stack never holds more than one element at a time once the scan gets going.
- `nums2` strictly decreasing: nothing is ever popped; every query on a value from this array correctly returns `-1`.

**💡 Interview Insight:** the proof that the stack stays monotonic — not just the code that happens to produce one — is exactly what separates "I've seen this pattern before" from "I understand why this pattern works," and it's worth stating unprompted, before being asked "why is this correct." The amortized O(n) argument (pushed once, popped at most once) is the second thing worth volunteering, since a nested loop reflexively reads as O(n²) to anyone not looking closely.

---

## Part 2 — SQL Fundamentals

### Prerequisites (confirmed)

- `@Entity`, `@Id`, `@Column`, `JpaRepository`, `ddl-auto: update` — Day 36. Today makes explicit what JPA has been generating underneath since then.

### Keys

**Primary key:** the column (or set of columns) that uniquely identifies each row in a table. Cannot be `NULL`. A table has at most one.

**Foreign key:** a column in one table that references another table's primary key, enforcing **referential integrity** — the database refuses an `INSERT` whose foreign-key value doesn't actually exist in the referenced table, and (depending on the constraint's configured behavior — `ON DELETE RESTRICT`, `CASCADE`, etc.) governs what happens if the referenced row is later deleted. `orders.user_id` referencing `users.id` is the running example below.

### Normalization

**Definition:** organizing tables to minimize redundancy and avoid update/insert/delete anomalies — situations where the same fact is stored in more than one place and can drift out of sync.

- **1NF:** every column holds a single, atomic value — no comma-separated lists crammed into one field.
- **2NF:** 1NF, plus every non-key column depends on the *entire* primary key (relevant specifically when the key is composite — a column depending on only part of a multi-column key violates this).
- **3NF:** 2NF, plus no *transitive* dependencies — a non-key column shouldn't depend on another non-key column. Storing a user's email directly on every one of their `orders` rows is a transitive dependency (the email depends on *the user*, not on *the order*) — better to store it once, on `users`, and look it up via a join when needed.

**Why this matters concretely, not just as a rule to recite:** if a user's email were duplicated across every order row, updating it means updating every one of those rows — miss even one, and the data is now silently, invisibly inconsistent, with no error raised anywhere.

### JOINs — the distinction that actually matters

Two tables, for a concrete running example:

**`users`:** `(1, 'Alice')`, `(2, 'Bob')`, `(3, 'Carol')`
**`orders`:** `(101, user_id=1, amount=50)`, `(102, user_id=1, amount=30)`, `(103, user_id=2, amount=20)`

Carol (id 3) has placed no orders.

```sql
SELECT u.name, o.id, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

**Result:** `Alice/101/50`, `Alice/102/30`, `Bob/103/20`. **Carol doesn't appear at all** — `INNER JOIN` only returns rows with a match on *both* sides.

```sql
SELECT u.name, o.id, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

**Result:** the same three rows, **plus** `Carol/NULL/NULL`. `LEFT JOIN` keeps every row from the left table regardless of whether a match exists, filling in `NULL` for the right side's columns when it doesn't.

**🔑 Key Takeaway — the exact distinction to hold onto:** `INNER JOIN` **excludes** a row entirely when there's no match. `LEFT JOIN` **includes** it, with `NULL`s standing in for "no matching row exists." These are two different outcomes, not two strengths of the same thing — and mixing them up is a common, real source of silently wrong query results (a report meant to include Carol, using `INNER JOIN`, would simply and silently omit her).

`RIGHT JOIN` is `LEFT JOIN`'s mirror (keeps every row from the *right* table instead). `FULL OUTER JOIN` keeps every row from *both* sides, `NULL`-filling on whichever side lacks a match.

### Required Exercise: Users Who Have Never Placed an Order

This is exactly the `LEFT JOIN`-produces-`NULL` behavior above, put to direct use.

#### Approach A — `LEFT JOIN` + `IS NULL`

```sql
SELECT u.name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

The `LEFT JOIN` keeps Carol with `o.id = NULL`; the `WHERE` clause isolates exactly the rows where that happened — precisely the users with no matching order.

#### Approach B — `NOT EXISTS`

```sql
SELECT u.name
FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

A correlated subquery: for each user, check whether *any* order references them; include the user if none do.

#### Approach C — `NOT IN`, and the trap worth knowing cold

```sql
SELECT u.name
FROM users u
WHERE u.id NOT IN (SELECT user_id FROM orders);
```

**⚠️ Common Mistake — this one is sharp and genuinely worth memorizing:** if `orders.user_id` can ever be `NULL` for any row (say, a guest-checkout order not tied to any account), this query **silently returns zero rows, for every user, including ones that clearly have no orders.** Why: `x NOT IN (a, b, NULL)` is evaluated as `x != a AND x != b AND x != NULL`. Any comparison against `NULL` evaluates to `UNKNOWN` — not `true`, not `false` — and `UNKNOWN` inside an `AND` chain makes the *entire* expression `UNKNOWN` for every row, which `WHERE` treats the same as `false`. One stray `NULL` anywhere in the subquery's results silently breaks the query for every row, with no error. **`NOT EXISTS` has no such trap** — it only ever asks "does a matching row exist," never comparing directly against a `NULL` value — which is exactly why it (or the `LEFT JOIN` version) is the generally safer default over `NOT IN` whenever the subquery's column isn't guaranteed `NOT NULL`.

### Indexes

**Definition:** a separate data structure — typically a **B-tree** — that lets the database locate rows matching a condition (`WHERE id = 5`) in O(log n), instead of scanning every row (O(n)). A primary key is automatically indexed.

**The trade-off, precisely:** an index is *extra* structure that has to be kept in sync with the table — every `INSERT`, `UPDATE`, or `DELETE` must also update every index defined on that table, not just the underlying row itself. More indexes speed up reads that can use them, at the direct cost of slower writes, since each write is now doing strictly more work.

---

## Project Block Guide (1.5 hrs)

**Repository:** `todo-api`. **Task:** add Flyway; move schema ownership from `ddl-auto: update` to a versioned `V1__create_tasks_table.sql`; set `ddl-auto: validate`.

**`src/main/resources/db/migration/V1__create_tasks_table.sql`** — matching Day 36's `Task` entity field-for-field:

```sql
CREATE TABLE tasks (
    id BIGSERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description VARCHAR(2000),
    status VARCHAR(20) NOT NULL,
    priority VARCHAR(20),
    due_date DATE
);
```

**Why `due_date`, not `dueDate`:** Spring Boot's default Hibernate naming strategy automatically converts a camelCase Java field name to a snake_case column name — `dueDate` becomes `due_date` with no `@Column(name = ...)` override needed. Worth confirming this matches what Hibernate had already been auto-generating under `ddl-auto: update`, by checking the column name `show-sql` printed on Day 36.

**Updated `application.yml`:**

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/tododb
    username: postgres
    password: postgres
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true
  flyway:
    enabled: true
    locations: classpath:db/migration
```

**What changes, mechanically:** Flyway's naming convention is `V{version}__{description}.sql` (double underscore). On startup, Flyway checks a table it manages itself, `flyway_schema_history`, comparing which migrations have already been applied against which migration files currently exist — it applies any new ones, in version order, and refuses to start if a *previously applied* file's contents no longer match their recorded checksum (migrations are meant to be immutable once applied — fixing a mistake means writing a new `V2__...` migration, not editing `V1`). `ddl-auto: validate` then tells Hibernate to check the actual schema against what your `@Entity` classes expect, and fail loudly on any mismatch — it no longer has permission to silently change anything itself, closing exactly the risk flagged on Day 36.

**Definition of done:** the app boots via the Flyway-managed schema; `flyway_schema_history` shows `V1` applied; a fresh database with no prior schema comes up correctly from the migration alone.

## Career Block Guide (1 hr)

**LinkedIn (20 min):** engagement — comment on 3–5 posts.

**Networking:** begin tailoring your resume with the Spring Boot + concurrency work from these two weeks — the first of several scheduled refresh points, deliberately placed earlier than the original plan had it, so the resume stays current with what you've actually built rather than accumulating a backlog of updates.

---

## Day 40 — Interview Questions

**Q1. Why is the two-stack queue's `pop()` amortized O(1) despite occasionally costing O(n)?** Every element is pushed onto `inStack` once and, over its total lifetime, moved to `outStack` once and popped from it once — a bounded, constant amount of work per element regardless of how the transfers happen to be distributed across individual calls.

**Q2. Prove that the Monotonic Stack in Next Greater Element I is guaranteed to stay decreasing from bottom to top.** Any element still on the stack has, by definition, not yet had something bigger appear after it — the `while` loop pops anything smaller the instant something bigger arrives. For any two elements with one below the other, the lower one could only remain below the upper one if the upper one was smaller when it was pushed — otherwise it would have popped the lower one immediately. The invariant is actively enforced on every push, not incidental.

**Q3. Why is Next Greater Element I's stack scan O(n) overall, given the nested `while` inside the `for`?** Every element is pushed exactly once and popped at most once across the entire run — total operations are bounded by `2n`, linear, regardless of how many iterations any single element happens to sit on the stack for.

**Q4. What's the exact difference between what `INNER JOIN` and `LEFT JOIN` do with a non-matching row?** `INNER JOIN` excludes the row entirely. `LEFT JOIN` includes it, with `NULL` filled in for every column from the side that had no match.

**Q5. Why can `NOT IN` silently return zero rows when `NOT EXISTS` wouldn't?** If the `NOT IN` subquery's result set contains even one `NULL`, every comparison against it evaluates to `UNKNOWN`, which propagates through the `AND` chain and makes the whole `WHERE` condition `UNKNOWN` — treated as `false` — for every row. `NOT EXISTS` only ever checks whether a matching row exists and never directly compares against a `NULL` value, so it has no equivalent failure mode.

**Q6. What's the actual trade-off an index introduces?** Faster lookups (O(log n) via a B-tree, instead of an O(n) full scan) at the cost of slower writes — every `INSERT`/`UPDATE`/`DELETE` must also update every index on the table, not just the row itself.

**Q7. What does 3NF specifically forbid, with a concrete example?** Transitive dependencies — a non-key column depending on another non-key column rather than on the primary key directly. Storing a user's email on every order row is a violation, since the email depends on the user, not the order; duplicating it risks the copies drifting out of sync.

**Q8. What does `ddl-auto: validate` actually check, and what does it refuse to do?** It compares the actual database schema against what the `@Entity` classes expect and fails startup loudly on any mismatch — unlike `update`, it never modifies the schema itself, closing off the silent-alteration risk that convenience carries.

**Q9. Why does Flyway refuse to start if an already-applied migration file's contents have changed?** Migrations are meant to be immutable, append-only history — Flyway tracks a checksum per applied migration specifically to detect this, and treats a mismatch as a sign the historical record of what was actually run against the database can no longer be trusted; the fix is a new migration file, not an edit to an old one.

---

## Daily Deliverable Check

- [ ] Implement Queue using Stacks (LC 232) confirmed solid from Week 1 (recap only, no re-solve needed).
- [ ] Next Greater Element I (LC 496) solved, the monotonic-invariant proof and the amortized O(n) argument both reproducible from memory, pushed to `dsa-java/stacks/`.
- [ ] Can state the `LEFT JOIN` vs. `INNER JOIN` distinction precisely, and explain the `NOT IN`/`NULL` trap without notes.
- [ ] Flyway migration (`V1__create_tasks_table.sql`) live in `todo-api`; `ddl-auto: validate` in place; `flyway_schema_history` confirmed showing `V1` applied.
- [ ] LinkedIn engagement done. Resume tailoring begun with Spring Boot + concurrency work.

---

## What Tomorrow Assumes You Already Know Cold

Tomorrow assumes the Monotonic Stack invariant and its amortized-O(n) proof are both fully reflexive — Next Greater Element II reuses the identical stack logic unchanged, adding only a circular-array wrapping technique on top, and the resource book won't re-derive the monotonic argument from scratch. It also assumes SQL's `LEFT JOIN`/`NULL` behavior is solid, since nothing further in this week's plan revisits it directly — today was SQL's only dedicated day. Today's project deliberately used the freed recap time for depth rather than an extra problem; tomorrow's schedule has genuine room (no Theory or Project block in the original plan) for the Monotonic Stack practice that was deliberately deferred from today.
