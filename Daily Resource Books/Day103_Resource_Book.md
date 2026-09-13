# Day 103 — SQL Practice, Part 1: Subqueries, Aggregation, and Self-Joins

**Series:** SDE-2 Interview Prep · Week 15, Day 103 (of 105 DSA-phase days)
**Curriculum Map:** [00_Curriculum_Map.md](./00_Curriculum_Map.md)
**◀ Previous:** [Day 102](./Day102_Resource_Book.md) · **Next ▶:** [Day 104](./Day104_Resource_Book.md)

**Companion to:** Day 5 of `Week_15_Revised.md`

---

## Recap

Yesterday closed the algorithmic side of this series' DSA phase — merge sort and quicksort built from scratch, Quickselect applied to a familiar problem, and a full recall pass across every pattern covered since Week 1. Today opens a different kind of track entirely: **SQL**, using LeetCode's SQL problem set the same way this series has used its algorithmic problem set — as pattern-recognition practice, not just syntax practice.

Week 6, Day 40 already covered SQL's foundations: keys, normalization, JOIN types (and the LEFT-JOIN-fills-NULL vs. INNER-JOIN-excludes distinction), indexes and their O(log n)-vs-O(n) trade-off, and the `NOT IN`/NULL trap. None of that is re-taught. Today builds **subqueries, correlated subqueries, `GROUP BY`/`HAVING`, and self-joins** on top of it, for the first time.

> 💡 **A dialect note, worth stating once and assuming from here on:** every query in today's and tomorrow's material uses **MySQL syntax**, matching LeetCode's actual SQL execution environment. Some of what's taught (date arithmetic functions, exact `LIMIT`/`OFFSET` syntax) varies across database systems — PostgreSQL and SQL Server, in particular, phrase a few of these differently. Worth knowing this is a real portability consideration, not something to treat as universal.

## Learning Objectives

By the end of today, without notes, you should be able to:

1. State SQL's logical query processing order (`FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`) and explain why it determines what's legal where — specifically, why `HAVING` can filter on an aggregate and `WHERE` can't.
2. Write a subquery that correctly returns `NULL` (not an error, and not zero rows) when the value it's looking for doesn't exist.
3. Write and explain a correlated subquery — specifically, what "correlated" means mechanically, and why it's conceptually re-evaluated per outer row.
4. Distinguish `WHERE` from `HAVING` precisely, and explain why `GROUP BY email HAVING COUNT(*) > 1` is the standard shape for "find groups with more than one member."
5. Write two structurally different self-joins (a date-offset join and a hierarchical self-reference join), and explain exactly why an `INNER JOIN` naturally excludes rows with a `NULL` foreign key.

## Concept Dependency Map for Today

```
Week 6, Day 40 (JOINs, keys, indexes,
NOT IN / NULL trap — already solid)
                │
                ▼
   SQL Logical Query Processing Order (NEW):
   FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
   — the foundation everything below is built on
                │
   ┌────────────┼─────────────────┬───────────────────┐
   ▼            ▼                 ▼                    ▼
Subquery   Correlated        GROUP BY / HAVING     Self-Join
(NEW)      Subquery (NEW:    (NEW: aggregate        (NEW: one table,
           references the   THEN filter, vs         joined to itself
           OUTER query's     WHERE's filter-         via two aliases —
           row)              THEN-aggregate)          two distinct shapes:
   │            │                 │                    date-offset, and
   ▼            ▼                 ▼                    hierarchical)
LC 176      LC 184            LC 182              LC 197, LC 181
```

---

## SQL Logical Query Processing Order — the foundation for everything today

SQL reads like `SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY`, but that's the **written** order, not the **logical evaluation** order. The engine actually processes clauses roughly like this:

```
FROM     → which table(s), and how are they joined
WHERE    → filter individual ROWS, before any grouping happens
GROUP BY → collapse the remaining rows into groups
HAVING   → filter GROUPS, using aggregate results
SELECT   → compute the final output columns
ORDER BY → sort the result
LIMIT    → cap the number of returned rows
```

This ordering is the precise, mechanical reason for a rule that otherwise looks arbitrary: **`WHERE` cannot reference an aggregate function, but `HAVING` can** — because `WHERE` is evaluated *before* `GROUP BY` even happens (there's no aggregate result to reference yet), while `HAVING` is evaluated *after* grouping and aggregation are already complete. This single ordering fact is the foundation for every problem below, and for tomorrow's window-function material as well.

---

## Problem 1 — Second Highest Salary (LeetCode #176, Medium)

**Table:** `Employee(id, salary)`. **Task:** return the second-highest **distinct** salary, or `NULL` if no second-highest exists.

### Approach 1 — Subquery (primary)

```sql
SELECT MAX(salary) AS SecondHighestSalary
FROM Employee
WHERE salary < (SELECT MAX(salary) FROM Employee);
```

**Mechanism:** the inner subquery `(SELECT MAX(salary) FROM Employee)` is evaluated once, independently of the outer query, producing a single value — the highest salary. The outer query then finds the max salary strictly *less than* that value, which is, by definition, the second-highest distinct salary.

**Why this correctly handles the "no second-highest exists" edge case, natively:** if every employee shares the same single salary, the `WHERE` clause matches **zero rows** (nothing is strictly less than the max). `MAX()` applied to an empty result set returns `NULL` in standard SQL — not an error, not zero rows returned. This is exactly the behavior LC176 requires, and it falls out of the query's own logic with no special-casing needed.

### Approach 2 — `LIMIT`/`OFFSET` (alternative, needs more care)

```sql
SELECT (
    SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1
) AS SecondHighestSalary;
```

Two details here are easy to get wrong, and worth being precise about:

> ⚠️ **Common Mistake 1 — forgetting `DISTINCT`.** Without it, if the highest salary appears **twice** (e.g., salaries `[100, 100, 90]`), `ORDER BY salary DESC LIMIT 1 OFFSET 1` returns the **second row**, which is still `100` — not the second-*highest-distinct-value*, `90`. `DISTINCT` collapses duplicate salary values before the `LIMIT`/`OFFSET` counts rows, which is what makes "offset 1" mean "the next distinct value" rather than "the next row."

> ⚠️ **Common Mistake 2 — the empty-result-set shape.** A bare `SELECT DISTINCT salary FROM Employee ORDER BY salary DESC LIMIT 1 OFFSET 1` returns **zero rows** if there's no second-highest value — not a row containing `NULL`, which is what LC176 actually expects (a single row, single column, value `NULL`). Wrapping the whole thing as a **scalar subquery inside an outer `SELECT`** (as shown above) forces exactly one output row regardless: if the inner query returns nothing, the scalar subquery itself evaluates to `NULL`, and the outer `SELECT` still returns one row containing it.

**Trade-off between the two approaches:** the `MAX`-subquery approach is self-contained and needs no `DISTINCT`/wrapping care for this specific case, but doesn't generalize past "second" without nesting another `MAX(... WHERE salary < ...)` layer. The `LIMIT`/`OFFSET` approach generalizes immediately to "Nth highest" by changing one number — directly relevant tomorrow.

**Complexity:** without an index on `salary`, either approach requires scanning and sorting/comparing across the full table — O(n log n) if the engine sorts, or O(n) for a single `MAX` scan. With an index on `salary`, both can be satisfied largely via the index itself, dramatically cheaper — a direct callback to Week 6's B-tree index material.

**Edge cases:** empty table (`NULL`, correctly, via both approaches); all salaries identical (`NULL`, as walked through above); exactly two distinct salaries (works correctly with either approach, no special-casing).

---

## Problem 2 — Department Highest Salary (LeetCode #184, Medium)

**Tables:** `Employee(id, name, salary, departmentId)`, `Department(id, name)`. **Task:** for every department, list every employee earning the maximum salary *within that department* (including ties).

### Approach — Correlated Subquery

```sql
SELECT d.name AS Department, e.name AS Employee, e.salary AS Salary
FROM Employee e
JOIN Department d ON e.departmentId = d.id
WHERE e.salary = (
    SELECT MAX(salary)
    FROM Employee e2
    WHERE e2.departmentId = e.departmentId    -- references the OUTER row — this is what makes it correlated
);
```

**What "correlated" actually means, mechanically:** Problem 1's subquery was **independent** — it could be evaluated once, up front, with no knowledge of the outer query at all. This subquery references `e.departmentId`, a column from the **outer** query's current row. Conceptually, it's re-evaluated **once per outer row**, each time computing "the max salary within *this specific employee's* department" — a genuinely different value for an employee in Engineering than for one in Sales. That per-row re-evaluation, driven by a reference to the outer row, is exactly what "correlated" means.

**Why this correctly handles ties, by construction:** using `=` against the correlated `MAX` naturally returns **every** employee whose salary equals their department's max — if two employees in the same department are tied for highest, both satisfy `e.salary = (that department's max)` independently, and both appear in the result. A naive `GROUP BY departmentId` reaching for `MAX(salary)` alone would collapse each department to a single row, losing exactly this "list every tied employee" requirement without extra machinery.

**Performance, precisely — not hand-waved:** naively, a correlated subquery looks like it re-scans the `Employee` table once per outer row — O(n × m) row-touches in the worst case, without help. In practice, a real query planner often rewrites a correlated subquery like this into an equivalent join internally, and — this is the concrete, citable point — **an index on `(departmentId, salary)` turns the inner lookup into a fast index range scan** rather than a full table scan per outer row, the same O(log n)-vs-O(n) trade-off Week 6's B-tree material already established, now applied inside a correlated subquery specifically.

> 🔗 **Forward reference:** tomorrow's window-function toolkit (`RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC)`) expresses this exact "per-group top value" shape more directly, and generalizes cleanly to "top N per group" in a way a correlated `MAX` subquery cannot without much more complexity. Not built today, since window functions haven't been taught yet — named here as motivation for what's coming.

**Common mistakes:** using `GROUP BY departmentId, MAX(salary)` and expecting ties to show up automatically (they won't — a plain `GROUP BY` collapses to one row per group, silently dropping tied employees unless the query is deliberately structured, as above, to keep every row and filter by comparison instead of collapsing); forgetting the correlation entirely (writing `WHERE e2.departmentId = e2.departmentId`, a self-referencing typo that trivially always matches every row — worth a careful eye on which alias is used where).

**Complexity:** O(n) for the outer scan, with the correlated inner lookup's cost depending entirely on indexing, as above — from O(n) per outer row (no index) down to O(log n) per outer row (with a `(departmentId, salary)` index), i.e., O(n²) worst case down to O(n log n) with proper indexing.

---

## Problem 3 — Duplicate Emails (LeetCode #182, Easy)

**Table:** `Person(id, email)`. **Task:** return every email that appears more than once.

```sql
SELECT email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

**`GROUP BY`, taught properly:** `GROUP BY email` collapses every row sharing the same `email` value into a single group — one output row per **distinct** email, rather than one per original row. This is what makes aggregate functions (`COUNT`, `SUM`, `AVG`, `MAX`, `MIN`) meaningful per-group instead of per-row: `COUNT(*)` here counts how many original rows fell into each email's group.

**`HAVING`, and precisely why it's not `WHERE`:** per the logical processing order above, `WHERE` filters individual rows **before** grouping happens — at that stage, there's no "count per group" yet to filter on. `HAVING` filters **groups**, evaluated **after** `GROUP BY` has already collapsed rows and computed aggregates — which is the only point at which "does this group have more than one member" is even a question that can be asked. This is the classic, canonical shape for "find groups exceeding some size": `GROUP BY <the thing to group on> HAVING COUNT(*) > <threshold>`.

> 💡 **Worth knowing:** `COUNT(*)` counts rows in the group regardless of `NULL`s; `COUNT(email)` specifically would skip rows where `email` is `NULL`. They're equivalent here only because `email` is guaranteed non-`NULL` by this table's constraints — worth being precise about the distinction rather than treating the two as always interchangeable.

**Complexity:** computing `GROUP BY` requires the engine to either **sort** by the grouping key (O(n log n)) or build an in-memory **hash** keyed by it (O(n) average) — which strategy is chosen is a query-planner decision, but either way, an index on `email` can make this considerably cheaper by supplying data already sorted, or by supporting the hash build with a cheaper scan.

**Edge cases:** no duplicates at all (empty result, correctly — no group has `COUNT(*) > 1`); every row identical (single group, correctly returned once, not once per duplicate).

**Interview framing:** state the `WHERE`-vs-`HAVING` distinction, grounded in the logical processing order, before writing the query — it's a small query, so the reasoning behind *why* `HAVING` (not `WHERE`) is required is the actual signal being tested.

---

## Problem 4 — Rising Temperature (LeetCode #197, Easy)

**Table:** `Weather(id, recordDate, temperature)`. **Task:** find every date where the temperature was strictly higher than the **previous calendar day's** temperature.

```sql
SELECT w2.id
FROM Weather w1
JOIN Weather w2 ON DATEDIFF(w2.recordDate, w1.recordDate) = 1
WHERE w2.temperature > w1.temperature;
```

**Self-join, taught properly — the mechanism:** a self-join treats one table as though it were two, by referencing it twice under two different **aliases** (`w1`, `w2`) in the same query. This lets rows of the table be compared **against other rows of the same table** — here, `w1` plays the role of "yesterday" and `w2` plays the role of "today," linked by the join condition rather than by being genuinely different tables.

**Why `DATEDIFF`, specifically, rather than assuming consecutive rows means consecutive dates:**

> ⚠️ **Common Mistake:** joining on row adjacency (e.g., matching `w2.id = w1.id + 1`) instead of genuine date arithmetic. If the `Weather` table has any gaps in its recorded dates — a day with no reading at all — "the next row" and "the next calendar day" silently stop being the same thing, and an `id`-adjacency join would compare temperatures across a gap, which the problem doesn't ask for and shouldn't be counted. `DATEDIFF(w2.recordDate, w1.recordDate) = 1` checks the actual calendar relationship directly, which is correct regardless of whether `id` happens to be gapless.

**Worked trace:** `Weather` = `{(1, '2024-01-01', 10), (2, '2024-01-02', 25), (3, '2024-01-03', 20)}`. Self-joining on `DATEDIFF = 1` produces candidate pairs `(w1=Jan 1, w2=Jan 2)` and `(w1=Jan 2, w2=Jan 3)`. Filtering `w2.temperature > w1.temperature`: `25 > 10` → true, Jan 2 (id 2) qualifies. `20 > 25` → false, Jan 3 doesn't. Result: `{2}`.

**Complexity:** a self-join's cost depends heavily on whether `recordDate` is indexed — with an index, each row's date-offset partner is found via an efficient lookup; without one, the join conceptually compares every row against every other row, O(n²), relying on the query planner to do better only if it can.

---

## Problem 5 — Employees Earning More Than Their Managers (LeetCode #181, Easy)

**Table:** `Employee(id, name, salary, managerId)`. **Task:** return employees who earn more than their own manager.

```sql
SELECT e1.name AS Employee
FROM Employee e1
JOIN Employee e2 ON e1.managerId = e2.id
WHERE e1.salary > e2.salary;
```

**A second, structurally different self-join shape:** Problem 4's self-join matched rows by a **date offset** (an arithmetic relationship between two columns). This one matches rows by a **hierarchical self-reference** — `e1.managerId = e2.id` links each employee (`e1`) directly to the row representing their own manager (`e2`), which is itself just another row in the very same table. Different join condition, same underlying self-join mechanism.

**Why `INNER JOIN` specifically — and why it's correct, not just convenient:** employees with no manager have `managerId IS NULL`. An `INNER JOIN` naturally **excludes** these — and it's worth being precise about *why*, since this connects directly back to Week 6's NULL-handling material:

> 🔗 **Direct connection to Week 6's `NOT IN`/NULL trap:** in SQL's three-valued logic, **any** comparison involving `NULL` — including `NULL = e2.id` — evaluates to `UNKNOWN`, never `TRUE`. A `JOIN` condition (like a `WHERE` condition) only keeps rows where the condition evaluates to `TRUE`; `UNKNOWN` doesn't qualify. So a manager-less employee's row simply never finds a match on `e1.managerId = e2.id`, and `INNER JOIN` drops it silently and correctly — which is exactly the right behavior here, since "does this employee earn more than a manager they don't have" isn't a meaningful question to begin with.

**Common mistakes:** using `LEFT JOIN` here out of a reflexive "always use LEFT JOIN to be safe" habit — it would preserve manager-less employees as rows with `NULL` in every `e2` column, which then fails the `WHERE e1.salary > e2.salary` comparison anyway (again, `NULL` comparisons are `UNKNOWN`, not `TRUE`) and get filtered out regardless — so `LEFT JOIN` happens to produce the same *final* result here, but for the wrong conceptual reason, and would behave differently if the query's shape changed slightly (e.g., if it needed to *report* on manager-less employees rather than just exclude them from a comparison). `INNER JOIN` is the version that's correct by direct design, not by a filtering step cleaning up after a looser join.

**Complexity:** identical shape to Problem 4's — a self-join whose cost depends on whether the join column (`managerId`/`id`, here effectively the primary key) is indexed; `id` being a primary key means it's indexed by default in virtually every real schema, making this join efficient in practice without any extra work.

---

## Career Block Guide (1 hr)

- **LinkedIn:** engagement — 20 minutes commenting substantively on 3–5 posts.
- **Networking:** no specific outreach task today.

*(No separate Project Block today — the SQL Block above is itself today's full hands-on deliverable, pushed directly to the repository as noted below.)*

---

## Day 103 — Interview Questions

**Q1. State SQL's logical query processing order, and explain what it's actually used to determine.**

*Answer:* `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`. It determines what's legal to reference where — most importantly, why `WHERE` cannot filter on an aggregate result (it runs before grouping/aggregation exist) while `HAVING` can (it runs after).

---

**Q2. Why does `SELECT MAX(salary) FROM Employee WHERE salary < (SELECT MAX(salary) FROM Employee)` correctly return `NULL`, rather than erroring, when every salary is identical?**

*Answer:* If every salary is identical, no row satisfies `salary < (the max)`, so the `WHERE` clause matches zero rows. `MAX()` applied to an empty result set returns `NULL` in standard SQL rather than erroring or returning no rows — which is exactly the output LC176 expects, with no special-casing required.

---

**Q3. Why is `DISTINCT` required in the `LIMIT 1 OFFSET 1` approach to Second Highest Salary?**

*Answer:* Without it, a duplicated highest salary occupies two rows before any lower value appears — `ORDER BY salary DESC LIMIT 1 OFFSET 1` would return the *second row*, which could still be the same (highest) value, not the second-highest *distinct* value. `DISTINCT` collapses duplicate values first, so "offset 1" correctly means "the next distinct value."

---

**Q4. What does it mean for a subquery to be "correlated," mechanically?**

*Answer:* A correlated subquery references a column from the outer query's current row (e.g., `e2.departmentId = e.departmentId`, where `e` is the outer alias). Conceptually, it's re-evaluated once per outer row, each time producing a value specific to that row's context — unlike an independent subquery, which can be evaluated once regardless of the outer query.

---

**Q5. Why does the correlated-subquery approach to Department Highest Salary correctly return tied employees, while a plain `GROUP BY departmentId` with `MAX(salary)` would not?**

*Answer:* `GROUP BY departmentId` collapses each department to a single output row, which can't represent multiple tied top earners at once without extra machinery. Comparing each individual employee's salary against their department's correlated max (`WHERE e.salary = (correlated MAX)`) keeps every row and independently checks each one — so every employee tied for their department's maximum satisfies the condition and appears in the result.

---

**Q6. How can indexing change a correlated subquery's actual performance, concretely?**

*Answer:* Without an index, the inner correlated lookup can effectively become a full scan of the inner table for every outer row — O(n) per outer row, O(n²) overall in the worst case. An index on the correlated column (here, `(departmentId, salary)`) turns that inner lookup into a fast index range scan instead of a full scan, the same O(log n)-vs-O(n) distinction established for indexes back in Week 6.

---

**Q7. What's the precise difference between `WHERE` and `HAVING`, and why can't they be swapped?**

*Answer:* `WHERE` filters individual rows before any grouping occurs; `HAVING` filters groups after `GROUP BY` has collapsed rows and computed aggregates. `WHERE` can't reference an aggregate because none exists yet at that stage of processing; `HAVING` can, because it runs strictly after aggregation.

---

**Q8. Why does Rising Temperature's self-join use `DATEDIFF(...) = 1` instead of comparing adjacent row IDs?**

*Answer:* If the table has any gaps in recorded dates, "the next row" and "the next calendar day" are no longer guaranteed to be the same thing — an ID-adjacency join would silently compare across a gap. `DATEDIFF` checks the actual calendar relationship directly, which stays correct regardless of whether the data happens to be gapless.

---

**Q9. In Employees Earning More Than Their Managers, why does `INNER JOIN` correctly exclude employees with no manager, without any extra `WHERE` clause needed?**

*Answer:* A manager-less employee has `managerId IS NULL`. Any comparison involving `NULL`, including `NULL = e2.id`, evaluates to `UNKNOWN` under SQL's three-valued logic — never `TRUE`. A `JOIN` condition only keeps rows where it evaluates to `TRUE`, so that employee's row simply never matches and is dropped, correctly and by direct design — the same NULL-comparison principle already established in Week 6's `NOT IN` trap.

---

**Q10. A self-join is written using two aliases of the same table. What does each alias conceptually represent, and why is aliasing necessary at all?**

*Answer:* Each alias lets the same physical table be referenced as if it were two separate tables playing two different roles in the query — e.g., "yesterday" vs. "today," or "employee" vs. "their manager." Aliasing is necessary because SQL needs some way to distinguish which occurrence of a column (e.g., which table's `recordDate`) is being referred to in the `JOIN`/`WHERE` clauses; without distinct aliases, the query can't disambiguate the two roles.

---

## Daily Deliverable Check

- [ ] All 5 SQL problems solved and pushed to `dsa-java/sql/`.
- [ ] SQL logical processing order stated correctly, with the `WHERE`-vs-`HAVING` distinction traced back to it.
- [ ] Second Highest Salary's `NULL`-on-empty-result behavior explained precisely for both the `MAX`-subquery and `LIMIT`/`OFFSET` approaches.
- [ ] Correlated subquery mechanism explained — what makes it "correlated," and why it correctly handles ties.
- [ ] `GROUP BY`/`HAVING` shape (`GROUP BY x HAVING COUNT(*) > n`) reproducible without notes.
- [ ] Both self-join shapes (date-offset, hierarchical) written correctly, with the `NULL`/three-valued-logic reasoning for `INNER JOIN` on the hierarchical one explainable.
- [ ] LinkedIn engagement completed.

## What Tomorrow Assumes You Already Know Cold

Tomorrow (Day 104) assumes today's logical processing order, subqueries, correlated subqueries, and both self-join shapes are solid — tomorrow's window functions are introduced as, in part, a cleaner alternative to exactly the correlated-subquery pattern Problem 2 used today, and that comparison only lands if today's version is fresh. Tomorrow also extends the logical processing order established today with one new rule specific to window functions: they're evaluated after `WHERE`/`GROUP BY`/`HAVING`, which is precisely why a window function's result can't be filtered directly in the same query's `WHERE` clause — a rule that will look arbitrary without today's ordering foundation already in place.

**Next ▶:** [Day 104](./Day104_Resource_Book.md)
