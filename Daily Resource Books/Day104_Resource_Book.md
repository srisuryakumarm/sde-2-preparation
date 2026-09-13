# Day 104 — SQL Practice, Part 2: Window Functions

**Series:** SDE-2 Interview Prep · Week 15, Day 104 (of 105 DSA-phase days)
**Curriculum Map:** [00_Curriculum_Map.md](./00_Curriculum_Map.md)
**◀ Previous:** [Day 103](./Day103_Resource_Book.md) · **Next ▶:** [Day 105](./Day105_Resource_Book.md)

**Companion to:** Day 6 of `Week_15_Revised.md`

---

## Recap

Yesterday built subqueries, correlated subqueries, `GROUP BY`/`HAVING`, and two self-join shapes on top of Week 6's SQL fundamentals — and established the logical query processing order (`FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`) that today extends by one more rule. Today is, deliberately, the single densest new-concept day of the SQL track: **window functions** — the one SQL topic genuinely likely to trip up someone whose SQL background is mostly transactional CRUD rather than analytical querying.

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Explain, precisely, how a window function differs from `GROUP BY` — specifically, why a window function does **not** collapse rows the way grouping does.
2. Distinguish `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` exactly, on a dataset containing ties, and explain why only one of the three is correct for "top N distinct values."
3. Explain why a window function's result cannot be referenced directly in that same query's `WHERE` clause, and what to do instead.
4. Use `LAG`/`LEAD` to compare a row against a neighboring row without a self-join.
5. Write a `CASE` expression and use it inside `SUM(...)` to perform conditional aggregation.
6. Use `PARTITION BY` to compute a ranking independently within each group, and explain how this generalizes what yesterday's correlated subquery could only do for a single group at a time.

## Concept Dependency Map for Today

```
Day 103 (SQL logical processing order,     Day 103 (GROUP BY collapses
correlated subquery for "max per group")   rows into one-per-group)
              │                                      │
              ▼                                      ▼
   Window Functions (NEW): OVER (...) —      Contrast: window functions
   compute a value relative to a set of      compute a value PER ROW,
   related rows, WITHOUT collapsing rows      rows are NOT collapsed
              │
    ┌─────────┼──────────────┬────────────────┐
    ▼         ▼               ▼                 ▼
ROW_NUMBER  RANK        DENSE_RANK           LAG / LEAD
  (NEW)     (NEW)          (NEW)            (NEW: look at a
    — three genuinely different                neighboring row,
      tie-handling behaviors                    no self-join needed)
    │         │               │                 │
    └─────────┴───────┬───────┴─────────────────┘
                       ▼
          PARTITION BY (NEW: restart the window
          function independently per group —
          generalizes Day 103's correlated-MAX
          "one group at a time" limitation)
                       │
                       ▼
        CASE expressions (NEW) + conditional
        aggregation: SUM(CASE WHEN ... THEN 1 ELSE 0 END)
                       │
                       ▼
      LC 177, LC 180, LC 262, LC 626, LC 185
```

---

## Window Functions — the foundation

**The core distinguishing fact, stated precisely:** `GROUP BY` **collapses** rows sharing a value into a single output row per group. A window function does **not** collapse anything — **every input row still appears in the output**, with an *additional* computed column added to it, reflecting some aggregate or ranking calculated over a "window" of rows related to the current one.

**Anatomy of `OVER (...)`:**

```sql
<window function>() OVER (
    PARTITION BY <column>    -- optional: restart the window independently per group
    ORDER BY <column>        -- defines the ordering the window function uses (e.g., for ranking, or for LAG/LEAD)
)
```

`PARTITION BY` divides rows into independent groups **for the purposes of the window function only** — unlike `GROUP BY`, it doesn't reduce row count at all; it just tells the window function "restart your calculation at the start of each new partition."

**One crucial, easy-to-miss rule, extending yesterday's logical processing order:** window functions are evaluated **after** `WHERE`/`GROUP BY`/`HAVING`, conceptually alongside the final `SELECT` list — which means **a window function's result cannot be referenced directly in that same query's `WHERE` clause** (its value doesn't exist yet at the point `WHERE` is evaluated). Filtering on a window function's output requires wrapping the windowed query in a **subquery** (or a CTE) and filtering in the **outer** query instead. This comes up directly in several of today's problems below.

### `ROW_NUMBER()` vs. `RANK()` vs. `DENSE_RANK()` — the classic trap, resolved precisely

All three assign a number to each row based on an `ORDER BY` within the window — the difference is entirely in **how they handle ties**. Concretely, for salaries `[100, 100, 90, 80]`, ordered descending:

| Salary | `ROW_NUMBER()` | `RANK()` | `DENSE_RANK()` |
|---|---|---|---|
| 100 | 1 | 1 | 1 |
| 100 | 2 | 1 | 1 |
| 90 | 3 | **3** | **2** |
| 80 | 4 | 4 | 3 |

- **`ROW_NUMBER()`** gives every row a distinct, sequential number, **arbitrarily** breaking ties (whichever row the engine happens to encounter first, unless a further `ORDER BY` tiebreaker is specified) — `1, 2, 3, 4`, always, regardless of ties.
- **`RANK()`** gives tied rows the *same* rank, but **leaves a gap** afterward equal to the number of tied rows — both `100`s get rank `1`, and the next distinct value (`90`) jumps to rank `3` (skipping `2`, since two rows already "used up" ranks 1 and 2).
- **`DENSE_RANK()`** also gives tied rows the same rank, but leaves **no gap** — `90` gets rank `2`, immediately following the tie, since it only cares about the count of *distinct* values seen so far, not the count of *rows*.

> 🔑 **Key Takeaway — which one to reach for:** `DENSE_RANK()` is the correct choice whenever the actual intent is "rank among **distinct values**" — which is precisely the case for "Nth highest salary" or "top N salaries," both of which are questions about distinct salary *values*, not about row position. Using `RANK()` for "top 3" can wrongly **exclude** a genuinely-3rd-distinct-value employee if a tie earlier consumed a rank number (a 2-way tie for 1st jumps straight to rank 3, meaning "rank ≤ 3" only actually captures the top **2 distinct values**, not 3). Using `ROW_NUMBER()` can wrongly **cut off mid-tie or over-include**, since it assigns unique numbers even to genuinely equal values. This distinction is worked through concretely in Problem 5 below, with a counter-example proving why `DENSE_RANK()` is the only one of the three that's actually correct there.

---

## Problem 1 — Nth Highest Salary (LeetCode #177, Medium)

**Table:** `Employee(id, salary)`. **Task:** return the Nth highest **distinct** salary (N given as a parameter), or `NULL` if it doesn't exist.

### Approach 1 — Parameterized `LIMIT`/`OFFSET` (extends yesterday directly)

LeetCode's expected format wraps this in a stored function:

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    SET N = N - 1;   -- MySQL historically needs this as an intermediate variable —
                      -- LIMIT/OFFSET clauses don't reliably accept an expression
                      -- computed directly from a function parameter in one step.
    RETURN (
        SELECT DISTINCT salary
        FROM Employee
        ORDER BY salary DESC
        LIMIT 1 OFFSET N
    );
END
```

This is exactly yesterday's Second-Highest `LIMIT 1 OFFSET 1` approach, generalized — `OFFSET 1` becomes `OFFSET (N-1)`, and wrapping the result as a scalar (the function's `RETURN`) gives the same "empty result becomes `NULL`, not zero rows" behavior discussed yesterday, now for any N.

### Approach 2 — `DENSE_RANK()` window function

```sql
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM Employee
) ranked
WHERE rnk = N;
```

**Why the subquery wrapper is required here, precisely:** per the rule stated above, `rnk` (a window function's output) doesn't exist yet at the point a `WHERE` clause in the *same* query would try to reference it — `WHERE DENSE_RANK() OVER (...) = N` directly is not legal. Computing it in an inner query first, then filtering on the now-materialized `rnk` column in an outer query, is the standard, necessary workaround.

**Why `DENSE_RANK()` specifically, connecting directly back to yesterday's `DISTINCT`-based approach:** `DENSE_RANK()` treats a run of duplicate salary *values* as occupying a single rank — which is exactly, precisely what `SELECT DISTINCT salary ... LIMIT/OFFSET` was already doing in Approach 1, via a completely different mechanism. **`DENSE_RANK()` is, semantically, "rank among distinct values" — the same intent as yesterday's `DISTINCT` keyword, expressed as a ranking function instead of a deduplication step.** Seeing these as the same underlying idea, arrived at two different ways, is more valuable than memorizing either one in isolation.

**Complexity:** the `LIMIT`/`OFFSET` approach needs the engine to sort (or use an index) and can stop early once it reaches the Nth row — good when N is small relative to table size. The window-function approach computes a rank for **every** row before filtering, which is more consistent work regardless of N, but composes more naturally if the query needs the ranks for anything else in the same pass.

**Edge cases:** N larger than the number of distinct salaries (`NULL`, correctly, via both approaches — the `LIMIT`/`OFFSET` version's `OFFSET` simply exceeds the available rows, and the window-function version's `WHERE rnk = N` matches nothing); N = 1 (reduces to "the highest salary," works correctly with no special-casing in either approach).

---

## Problem 2 — Consecutive Numbers (LeetCode #180, Medium)

**Table:** `Logs(id, num)`. **Task:** find every `num` that appears at least three times **consecutively** (three rows in a row, ordered by `id`, all sharing the same `num`).

### Approach 1 — `LAG()` twice

```sql
SELECT DISTINCT num AS ConsecutiveNums
FROM (
    SELECT
        num,
        LAG(num, 1) OVER (ORDER BY id) AS prev1,
        LAG(num, 2) OVER (ORDER BY id) AS prev2
    FROM Logs
) windowed
WHERE num = prev1 AND num = prev2;
```

**`LAG`, precisely:** `LAG(column, offset) OVER (ORDER BY ...)` returns the value of `column` from the row `offset` positions **before** the current row, within the specified ordering — returning `NULL` if no such prior row exists (e.g., at the very start). `LEAD` is the symmetric opposite, looking forward instead of back.

**Why this requires the same subquery wrapper as Problem 1:** `prev1`/`prev2` are window-function outputs, so — exactly as established above — they can't be referenced in a `WHERE` clause in the same query that computes them. Computing them in an inner query, then filtering `num = prev1 AND num = prev2` in the outer query, is required, not optional.

**Correctness — why this precise condition captures "three consecutive":** at any given row, `prev1` is the value one row back and `prev2` is the value two rows back. If the current row's `num`, `prev1`, and `prev2` are all equal, that means the current row and the two immediately preceding it (in `id` order) all share the same value — exactly three consecutive matching rows, with the current row being the *last* of the three. `DISTINCT` in the outer `SELECT` collapses the case where a run is longer than three (e.g., four or five in a row would otherwise report the same `num` multiple times, once per row past the third).

**Worked trace:** `Logs = {(1,1), (2,1), (3,1), (4,2), (5,1), (6,2), (7,2)}`.

| id | num | prev1 (LAG 1) | prev2 (LAG 2) | num=prev1=prev2? |
|---|---|---|---|---|
| 1 | 1 | NULL | NULL | no |
| 2 | 1 | 1 | NULL | no |
| 3 | 1 | 1 | 1 | **yes** |
| 4 | 2 | 1 | 1 | no |
| 5 | 1 | 2 | 1 | no |
| 6 | 2 | 1 | 2 | no |
| 7 | 2 | 2 | 1 | no |

Only `id=3` qualifies → `num = 1` is the (deduplicated) answer. Matches the expected result for this classic version of the dataset.

### Approach 2 — Self-join (the "old way," worth naming for contrast)

Before window functions, this same question required a **three-way self-join** — joining `Logs` to itself twice more, matching on consecutive `id`s, then comparing all three `num` values directly. It works, but it's noticeably more verbose and, for larger "at least K consecutive" variants, requires one additional self-join per additional row of consecutiveness required. `LAG` scales to "K consecutive" by just adding more `LAG(num, i)` columns, without adding more joins — a concrete, citable reason window functions are often the cleaner tool for this shape once K grows past 2 or 3.

**Complexity:** the `LAG`-based approach is a single pass with a window computation, generally O(n log n) (dominated by the `ORDER BY id` the window relies on) — versus the self-join approach's join cost, which without careful indexing can approach O(n²).

**Common mistakes:** forgetting that `LAG` returns `NULL` for the first row(s) of the ordering, and that `NULL = NULL` is `UNKNOWN`, not `TRUE` — so a run of `NULL`s at the very start of the table correctly never satisfies `num = prev1 AND num = prev2` on its own, with no special-casing needed (another quiet callback to SQL's three-valued logic); forgetting `DISTINCT` in the outer query, which would report the same qualifying `num` once per row past the third in a longer run, rather than once overall.

---

## Problem 3 — Trips and Users (LeetCode #262, Hard)

**Tables:** `Trips(id, client_id, driver_id, city_id, status, request_at)`, `Users(users_id, banned, role)`. **Task:** for each day in a given date range, compute the cancellation rate — `(cancelled trips) / (total trips)` — **excluding** any trip where either the client or the driver is banned.

```sql
SELECT
    t.request_at AS Day,
    ROUND(
        SUM(CASE WHEN t.status != 'completed' THEN 1 ELSE 0 END) / COUNT(*),
        2
    ) AS "Cancellation Rate"
FROM Trips t
JOIN Users c ON t.client_id = c.users_id AND c.banned = 'No'
JOIN Users d ON t.driver_id = d.users_id AND d.banned = 'No'
WHERE t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY t.request_at;
```

**Two joins against `Users`, under two different aliases, for two different purposes — worth distinguishing from a true self-join:** this isn't quite the same shape as yesterday's self-joins (which compared two rows of the *same conceptual role* against each other). Here, `Users` is joined **twice**, once to check the *client's* banned status and once to check the *driver's* — two genuinely different roles, both happening to be looked up in the same underlying table. The mechanism (alias the same table more than once) is identical to a self-join; the *purpose* — filtering, not comparing — is different, which is worth being precise about if asked.

**`CASE` expressions, taught properly:**

```sql
CASE WHEN <condition> THEN <value1> ELSE <value2> END
```

is a conditional **expression** — usable anywhere a value is expected, including inside an aggregate function. `CASE WHEN t.status != 'completed' THEN 1 ELSE 0 END` evaluates to `1` for a cancelled trip and `0` for a completed one, **per row**.

**Conditional aggregation — why `SUM(CASE WHEN ... THEN 1 ELSE 0 END)` correctly counts matching rows:** `SUM()` adds up whatever the `CASE` expression evaluates to, across every row in the group. Since that's `1` for exactly the rows matching the condition and `0` for every other row, the total is precisely a **count of matching rows within that group** — a widely-used, genuinely important idiom, not a magic incantation. (`t.status != 'completed'` correctly captures "cancelled" here, since the status column's possible cancellation values — cancelled by driver, cancelled by client — are both simply "not completed.")

**Complexity:** two joins plus a `GROUP BY` — cost dominated by whichever join/filter is least selective; indexing `banned` and `request_at` (and the join columns) meaningfully helps in practice, worth a brief mention even without building out a full execution-plan analysis.

**Common mistakes:** filtering banned users with a `WHERE` clause instead of folding the condition into the `JOIN`'s `ON` clause — functionally similar here, but worth being deliberate about which one is intended, since a banned-status condition in `WHERE` after a `LEFT JOIN` (rather than `INNER JOIN`) would behave differently than the same condition inside `ON`; getting the cancellation definition backwards (counting completed trips instead of cancelled ones — easy to invert by mistake under time pressure).

---

## Problem 4 — Exchange Seats (LeetCode #626, Medium)

**Table:** `Seat(id, student)`. **Task:** swap every pair of adjacent seats (1↔2, 3↔4, ...); if the total number of seats is odd, the last (unpaired) seat stays in place.

```sql
SELECT
    CASE
        WHEN id % 2 = 1 AND id != (SELECT MAX(id) FROM Seat) THEN id + 1
        WHEN id % 2 = 0 THEN id - 1
        ELSE id
    END AS id,
    student
FROM Seat
ORDER BY id;
```

**Reading the three `CASE` branches precisely:**
1. An **odd** `id` that is **not** the very last row → pair it with the *next* seat (`id + 1`).
2. An **even** `id` → pair it with the *previous* seat (`id - 1`).
3. **Fallback** (`ELSE id`) — reached only when neither branch above matched, which happens precisely for a **trailing, unpaired odd `id`** (the last seat, when the total count is odd) → stays in place.

**Worked trace, 5 students (odd total — the interesting case):**

| Original id | Branch taken | New id |
|---|---|---|
| 1 | odd, not max → `id+1` | 2 |
| 2 | even → `id-1` | 1 |
| 3 | odd, not max → `id+1` | 4 |
| 4 | even → `id-1` | 3 |
| 5 | odd, **is** max → fallback | 5 |

Seats 1↔2 swap, 3↔4 swap, seat 5 (the odd one out) stays — exactly the specified behavior, and the `id != (SELECT MAX(id) FROM Seat)` check is precisely what prevents seat 5 from incorrectly trying to pair with a nonexistent seat 6.

**Where ordering matters, and where it doesn't:** the `CASE` computation itself is entirely row-independent — each row's new `id` is computed from that row alone (plus the one scalar subquery for `MAX(id)`), with no dependency on row order. `ORDER BY id` at the end matters only for **how the final result is displayed**, not for the correctness of the swap logic itself — worth being precise about which part of the query each piece is actually doing.

**Complexity:** O(n) — one pass, one `CASE` evaluation per row, plus one scalar subquery for `MAX(id)` (typically O(1) with a primary-key index, or a cheap single-pass scan otherwise) evaluated once (or once per row if the query planner doesn't hoist it — worth knowing this is an optimization the engine may or may not apply automatically, not something to assume blindly).

---

## Problem 5 — Department Top Three Salaries (LeetCode #185, Hard)

**Tables:** `Employee(id, name, salary, departmentId)`, `Department(id, name)`. **Task:** for each department, return every employee among the top **three unique** salaries in that department.

```sql
SELECT d.name AS Department, ranked.name AS Employee, ranked.salary AS Salary
FROM (
    SELECT
        name,
        salary,
        departmentId,
        DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS rnk
    FROM Employee
) ranked
JOIN Department d ON ranked.departmentId = d.id
WHERE ranked.rnk <= 3;
```

**`PARTITION BY`, given its full treatment — and directly synthesizing the whole two-day SQL arc:** yesterday's Department Highest Salary (Problem 2, Day 103) solved "find the max **per department**" with a correlated subquery — a technique that works, but only because it was solving for a *single* value per group (just the max). `PARTITION BY` generalizes this cleanly to **any per-group ranking**, including "top 3," which a correlated `MAX` subquery has no natural way to express at all without stacking increasingly awkward extra logic. `PARTITION BY departmentId` tells `DENSE_RANK()` to **restart its ranking independently within each department** — department A's ranking has no effect on department B's, exactly analogous to how yesterday's correlated subquery recomputed its `MAX` independently per outer row, but now expressed as one clean windowed pass instead of a per-row subquery re-execution.

**Why `DENSE_RANK()` — and not `RANK()` or `ROW_NUMBER()` — is the only correct choice here, proven with a counter-example:**

Take a department with salaries `[90000, 90000, 85000, 80000, 75000]` (a tie for the top spot). "Top 3 unique salaries" should include everyone earning `90000`, `85000`, or `80000` — four employees total (two tied at `90000`, plus the `85000` and `80000` earners), since there are only **three distinct salary values** among them.

| Salary | `ROW_NUMBER()` | Included at ≤3? | `RANK()` | Included at ≤3? | `DENSE_RANK()` | Included at ≤3? |
|---|---|---|---|---|---|---|
| 90000 | 1 | ✅ | 1 | ✅ | 1 | ✅ |
| 90000 | 2 | ✅ | 1 | ✅ | 1 | ✅ |
| 85000 | 3 | ✅ | **3** | ✅ | **2** | ✅ |
| 80000 | 4 | ❌ *(wrong — should be included)* | **4** | ❌ *(wrong — should be included)* | **3** | ✅ *(correct)* |
| 75000 | 5 | ❌ | 5 | ❌ | 4 | ❌ |

`ROW_NUMBER()` incorrectly excludes the `80000` earner — because it burned two separate numbers (1 and 2) on the tied pair, pushing every subsequent distinct value one position later than it should be. `RANK()` makes the exact same mistake for a different stated reason — it also gives the tied pair rank 1 each, then jumps to rank 3 for `85000` (correct, coincidentally) but rank 4 for `80000`, past the cutoff. **Only `DENSE_RANK()` correctly reflects "this is the 3rd distinct value," regardless of how many rows were tied at ranks above it** — which is precisely why it's the only one of the three that correctly implements "top N unique values" in general, not just in this particular example.

**Complexity:** O(n log n) for the windowed sort-and-rank (per partition, effectively), plus the join to `Department`, generally efficient with a `(departmentId, salary)` index supporting both the partitioning and the ordering directly.

**Interview framing:** this problem is the natural place to *prove* the `DENSE_RANK()` choice with a concrete tie-containing counter-example (as above) rather than asserting it — being asked "why not `RANK()`" and having an immediate, precise, numeric answer is exactly the kind of depth a tier-1 interviewer is listening for on a Hard-rated SQL problem.

---

## SQL Practice Track — CLOSED at 10/10 required (0 extra)

Today's five problems bring the SQL practice track to exactly 10 — Second Highest Salary, Department Highest Salary, Duplicate Emails, Rising Temperature, Employees Earning More Than Their Managers (Day 103), plus Nth Highest Salary, Consecutive Numbers, Trips and Users, Exchange Seats, Department Top Three Salaries (today). **No extra practice is being added**, matching the plan's own explicit framing of this track as complete at 10, exactly as Segment Trees was explicitly defined as complete at 2 back on Day 101 — both are treated here as deliberately-scoped, fixed-size tracks rather than open-ended patterns to keep expanding. Across these 10 problems, every core SQL technique with real SDE-2 interview weight got genuine coverage: subqueries and correlated subqueries, `GROUP BY`/`HAVING`, two structurally distinct self-join shapes, and the full window-function toolkit (`ROW_NUMBER`/`RANK`/`DENSE_RANK` precisely distinguished, `LAG`/`LEAD`, `PARTITION BY`, `CASE` expressions, and conditional aggregation) — a real foundation, not a token gesture at the topic.

## Career Block Guide (1 hr)

- **LinkedIn:** engagement — 20 minutes commenting substantively on 3–5 posts.
- **Networking:** no specific outreach task today.

*(No separate Project Block today, matching yesterday's structure — the SQL Block above is today's full hands-on deliverable.)*

---

## Day 104 — Interview Questions

**Q1. What's the fundamental difference between a window function and `GROUP BY`?**

*Answer:* `GROUP BY` collapses rows sharing a value into one output row per group. A window function does not collapse anything — every input row still appears in the output, with an additional computed column reflecting some aggregate or ranking calculated over a related set of rows (the "window").

---

**Q2. On a dataset with a tie, walk through exactly how `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` would each number the rows.**

*Answer:* For descending salaries `[100, 100, 90, 80]`: `ROW_NUMBER()` gives `1, 2, 3, 4` — always unique, breaking the tie arbitrarily. `RANK()` gives `1, 1, 3, 4` — tied rows share a rank, but the next distinct value's rank skips ahead by the number of tied rows. `DENSE_RANK()` gives `1, 1, 2, 3` — tied rows share a rank, and the next distinct value's rank increases by exactly 1, with no gap.

---

**Q3. Why can't a window function's output be referenced directly in that same query's `WHERE` clause?**

*Answer:* Window functions are evaluated after `WHERE`/`GROUP BY`/`HAVING` in SQL's logical processing order — conceptually alongside the final `SELECT` list. At the point `WHERE` runs, the window function's result doesn't exist yet. Filtering on it requires wrapping the windowed query in a subquery (or CTE) and filtering in the outer query instead.

---

**Q4. Why is `DENSE_RANK()` specifically the right choice for "Nth highest salary," rather than `RANK()` or `ROW_NUMBER()`?**

*Answer:* `DENSE_RANK()` numbers distinct *values*, not rows — which matches "Nth highest salary"'s actual intent (a question about distinct salary values). It's the same underlying idea as yesterday's `SELECT DISTINCT salary ... LIMIT/OFFSET` approach, expressed through ranking instead of deduplication, rather than a different technique entirely.

---

**Q5. What does `LAG(num, 2) OVER (ORDER BY id)` return for the first two rows of the ordering, and why?**

*Answer:* `NULL` for both — `LAG` looks a fixed number of positions backward within the ordering, and there simply isn't a row two positions before the 1st or 2nd row. This falls out of the mechanism directly, with no special-casing required, and further downstream comparisons against a `NULL` correctly evaluate to `UNKNOWN`/not-true under SQL's three-valued logic.

---

**Q6. In Trips and Users, why is `Users` joined twice instead of once?**

*Answer:* The query needs to check the banned status of two different people playing two different roles in the same trip — the client and the driver — both of whom are looked up in the same `Users` table. Joining it twice, under two different aliases, lets each role's banned status be checked independently within the same query.

---

**Q7. Explain exactly why `SUM(CASE WHEN status != 'completed' THEN 1 ELSE 0 END)` correctly counts cancelled trips within a group.**

*Answer:* The `CASE` expression evaluates to `1` for each row where the trip isn't completed (i.e., was cancelled) and `0` otherwise — per row. `SUM()` then adds up those per-row values across the group; since only matching rows contribute a nonzero amount (exactly `1` each), the total equals the count of matching rows in that group.

---

**Q8. In Exchange Seats, why is the `id != (SELECT MAX(id) FROM Seat)` check necessary in the first `CASE` branch?**

*Answer:* Without it, the last seat — if its id is odd (i.e., the total seat count is odd) — would still try to swap with a nonexistent next seat (`id + 1`), which doesn't exist. That check specifically detects "this is a trailing, unpaired odd id" and routes it to the fallback branch (stay in place) instead of attempting an invalid swap.

---

**Q9. Prove, with a concrete counter-example, why `RANK()` would give a wrong answer for Department Top Three Salaries.**

*Answer:* Take salaries `[90000, 90000, 85000, 80000]` in one department, with a tie for first. `RANK()` gives both `90000`s rank 1, then jumps to rank 3 for `85000` (correct so far) but rank 4 for `80000` — pushing it past a `rnk <= 3` filter and wrongly excluding it, even though `80000` is genuinely the 3rd-highest *distinct* salary in the department. `DENSE_RANK()` gives `1, 1, 2, 3` instead, correctly keeping `80000` at rank 3.

---

**Q10. How does `PARTITION BY` relate to what yesterday's correlated subquery (Department Highest Salary) was doing?**

*Answer:* Yesterday's correlated subquery recomputed `MAX(salary)` independently for each outer row's department — effectively "restarting" the calculation per group, but only capable of expressing a single aggregate value (the max) per group. `PARTITION BY departmentId` generalizes that same "restart independently per group" idea into the window-function framework, but supports full rankings (not just a single max), which is exactly what "top 3 per department" needs and a correlated `MAX` subquery structurally cannot provide without much more complexity.

---

## Daily Deliverable Check

- [ ] All 5 SQL problems solved and pushed to `dsa-java/sql/`.
- [ ] SQL practice track confirmed complete at 10/10 problems, 0 extra — able to name all 10 and their core technique cold.
- [ ] `ROW_NUMBER()` vs. `RANK()` vs. `DENSE_RANK()` reproducible on a tie-containing example without notes, including which one is correct for "top N distinct" and why.
- [ ] The "window functions can't be filtered in the same query's `WHERE`" rule explainable, tied back to the logical processing order.
- [ ] `LAG`/`LEAD` mechanism explained; Consecutive Numbers solved both via `LAG` and (at least conceptually) via the older self-join alternative.
- [ ] `CASE` expressions and conditional aggregation (`SUM(CASE WHEN ... THEN 1 ELSE 0 END)`) explainable mechanically, not just as a memorized idiom.
- [ ] `PARTITION BY` explained as a generalization of yesterday's correlated-subquery "per group" idea.
- [ ] LinkedIn engagement completed.

## What Tomorrow Assumes You Already Know Cold

Tomorrow (Day 105) is a consolidation day — no new SQL or DSA content. It assumes the SQL track (all 10 problems, both days) is complete and reviewable on demand, since tomorrow's Weekly Scorecard treats the SQL track as one of the two major deliverables this week produced (alongside the DSA curriculum's own closure). It also assumes today's `DENSE_RANK()` reasoning specifically — the "rank among distinct values, correctly handling ties" argument — is solid, since it's one of the cleanest examples in the whole series of proving a specific technical choice with a counter-example rather than asserting it, worth being able to reproduce cold in an actual interview setting.

**Next ▶:** [Day 105](./Day105_Resource_Book.md)
