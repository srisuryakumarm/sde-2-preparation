# Day 43 — Missing Primer: Database Indexing in Depth — Composite, Covering, and Reading `EXPLAIN`

**Do this first on Day 43 (≈ 50 min, before Docker Basics), then continue with Day 43's regular plan.**
**Needed by:** Day 49 (N+1 and fetching — a fix is often "the join column needs an index"), Day 78 (its pagination exercise tells you to run `EXPLAIN ANALYZE` and "add the composite index" without explaining either; sharding assumes an index on the shard key), Days 103–104 (SQL practice — "why is this query slow?"), Day 118 (SQL vs NoSQL), Day 127 (B-tree vs LSM), and the **"data model and indexes" step of every HLD answer on Days 120–133**. "How would you index this table?" is one of the most common backend questions at every target company.
**Why it is missing:** Day 40's *Indexes* section is two short paragraphs — a B-tree gives O(log n) lookups, and indexes slow writes. Composite indexes, the leftmost-prefix rule, covering indexes, clustered vs secondary indexes, why the planner sometimes ignores an index, and `EXPLAIN` appear in none of the 148 books. (Day 103–104's SQL practice never has you read a query plan, and Day 78's file uses `EXPLAIN ANALYZE` as an exercise step without teaching it.)
**Already in place:** tables, keys, joins and the basic index idea (Day 40), local Postgres (Day 36's file — native or Docker; use whichever you set up), Flyway migrations (Day 40's files), `todo-api`. Docker itself is taught later today — if you run Postgres natively you do not need it for this primer.

---

## 1. How an index actually works — the B+ tree
An index is a **separate, sorted structure** that maps column values to row locations. Almost every relational database uses a **B+ tree**: a balanced tree whose nodes are whole **disk pages** (8 KB in Postgres), each holding hundreds of keys.
- **Wide and shallow:** with ~hundreds of keys per page, three or four levels cover millions of rows → a point lookup is ~3–4 page reads, not 20+ as a binary tree would need. (You meet the binary search tree on Days 46–51; a B+ tree is the same sorted-and-balanced idea made *wide* instead of tall, because on disk the cost is "pages read", not comparisons.)
- **Leaves are linked in order**, so a range (`BETWEEN`, `ORDER BY … LIMIT`) is "find the start, then walk right" — no sort needed.
- **Postgres:** the table is a *heap* (unordered); every index, including the primary key's, is a separate B-tree pointing into the heap. **MySQL/InnoDB and SQL Server:** the primary key index is **clustered** — the table rows themselves are stored in key order, and every *secondary* index entry holds the primary key (so a secondary lookup is two tree walks). Interview line: *"a clustered index is the table; a secondary index is a pointer."*
- Primary-key and `UNIQUE` constraints create an index automatically. **A foreign-key column does not** (in Postgres) — an unindexed FK makes joins and parent deletes scan the child table.

## 2. Read a query plan before you guess
```sql
EXPLAIN SELECT …;                    -- the planner's estimate; does NOT run the query
EXPLAIN (ANALYZE, BUFFERS) SELECT …; -- RUNS it and shows real row counts, time and pages touched
```
**`EXPLAIN ANALYZE` really executes the statement.** For `UPDATE`/`DELETE` wrap it: `BEGIN; EXPLAIN ANALYZE DELETE …; ROLLBACK;` (verified: the rows are still there afterwards).

What to look for, in order:
| Plan node | Meaning |
|---|---|
| **Seq Scan** | read every row of the table; fine for tiny tables or when most rows match |
| **Index Scan** | walk the index, then fetch each row from the table |
| **Index Only Scan** | the index alone answered the query (a **covering** index) — `Heap Fetches: 0` is the goal |
| **Bitmap Index/Heap Scan** | collect matching row locations from the index, then read the table pages in order — the middle ground for "many rows" |
| `Rows Removed by Filter` | big number = you read rows only to throw them away → missing index |
| `Sort` node | the database had to sort; the right index removes it |
| estimated vs actual `rows` | far apart = stale statistics → run `ANALYZE table_name` |

## 3. Composite indexes and the leftmost-prefix rule
An index on `(user_id, status, created_at)` is sorted by `user_id`, **then** by `status` within a user, **then** by `created_at` within that. So it can serve:
| Query | Uses the index? |
|---|---|
| `WHERE user_id = ?` | ✅ prefix |
| `WHERE user_id = ? AND status = ?` | ✅ prefix |
| `WHERE user_id = ? AND status = ? ORDER BY created_at DESC LIMIT 20` | ✅ all three — **and no sort step**, the index already has the order |
| `WHERE status = ?` / `WHERE created_at > ?` (no `user_id`) | ❌ cannot seek — the rows for one status are scattered across every user |
**Column-order rule of thumb:** equality columns first, then the range or sort column last. Ask "which columns do my hottest queries filter by equality?" before ordering.

## 4. Covering indexes and index-only scans
If every column the query needs is *in the index*, the table is never touched.
```sql
CREATE INDEX idx_cover ON tasks_demo (user_id, status, created_at DESC) INCLUDE (title);
```
`INCLUDE` columns ride along in the leaf pages without being part of the sort key. Cost: a bigger index. (Postgres needs a recently vacuumed table for `Heap Fetches: 0`.)

## 5. Why the planner ignores your index (the classic six)
1. **A function on the column:** `WHERE lower(title) = 'x'` cannot use an index on `title` → create an **expression index**: `CREATE INDEX … ON t (lower(title));`.
2. **Leading wildcard:** `LIKE '%777'` cannot use a B-tree (no known start). `LIKE 'abc%'` can only if the database uses the `C` collation or the index is created with `text_pattern_ops` — the official `postgres` Docker image is **not** `C`, so use `text_pattern_ops` for prefix search.
3. **Implicit type conversion** (comparing a `varchar` column with a number, or `int` with `bigint` in some engines).
4. **Most rows match:** if a filter matches a large share of the table, reading the index and then the table is slower than one sequential pass — verified: `WHERE status IN ('OPEN','DONE','BLOCKED')` is a Seq Scan even with an index on `status`.
5. **`OR` across different columns** and **leading column missing** (§3).
6. **Stale statistics** or a tiny table — `ANALYZE` first, test on realistic data volumes.

## 6. The price of an index
Every index must be updated on every insert/update/delete of an indexed column, and takes disk and memory. In the exercise below, the 13 MB table ends up with **~31 MB of indexes**. Rules: index for your real queries, drop unused ones (`pg_stat_user_indexes` shows scan counts), and prefer **partial indexes** (`CREATE INDEX … WHERE status = 'OPEN'` — 472 kB instead of megabytes) when you only ever query a small slice. Other index types worth naming: **hash** (equality only), **GIN** (arrays, JSONB, full-text), **BRIN** (huge append-only, time-ordered tables), **unique**.
**In production:** `CREATE INDEX CONCURRENTLY` builds without blocking writes, but **cannot run inside a transaction block** (verified error). A Flyway migration normally runs in a transaction, so that one needs Flyway's per-migration `executeInTransaction=false` setting (check the Flyway docs); a plain `CREATE INDEX` in a migration is fine for this plan.

## 7. Exercise (≈ 25 min, any local Postgres — scratch database, not `todo-api`)
```sql
CREATE TABLE tasks_demo (id bigserial PRIMARY KEY, user_id int, status text, title text, created_at timestamptz);
INSERT INTO tasks_demo (user_id, status, title, created_at)
SELECT g % 1000, (ARRAY['OPEN','DONE','BLOCKED'])[1 + g % 3], 'task ' || g, now() - (g % 365) * interval '1 day'
FROM generate_series(1, 200000) AS g;
ANALYZE tasks_demo;

EXPLAIN (ANALYZE, BUFFERS)
SELECT id, title FROM tasks_demo WHERE user_id = 42 AND status = 'OPEN' ORDER BY created_at DESC LIMIT 20;
```
1. **Before:** you should see a (Parallel) **Seq Scan**, a `Sort`, and `Rows Removed by Filter` ≈ 99,966 per worker. (On the verification run: ~16 ms. Yours will differ — the *shape* matters.)
2. `CREATE INDEX idx_user_status_created ON tasks_demo (user_id, status, created_at DESC); ANALYZE tasks_demo;` and re-run: **Index Scan, no Sort node, ~0.06 ms** — about 250× faster.
3. Run `SELECT id FROM tasks_demo WHERE status = 'OPEN' AND created_at > now() - interval '2 days';` — confirm it does **not** use the index (leftmost-prefix). Say why in one sentence.
4. Drop the index and create the **covering** version from §4; re-run the query selecting only `title` — look for **Index Only Scan** and `Heap Fetches: 0` (run `VACUUM ANALYZE tasks_demo;` first).
5. Compare `WHERE title = 'task 777'` with `WHERE lower(title) = 'task 777'` after `CREATE INDEX … (title)`; fix the second with an expression index.
6. Write the Flyway migration file name and body you would add to `todo-api` for the composite index (`V3__add_task_indexes.sql` — adjust the number to your next version).

## Checklist
- [ ] Can explain a B+ tree in three sentences (wide, shallow, linked leaves) and why disks care.
- [ ] Can say clustered vs secondary index, and that Postgres FK columns are not indexed automatically.
- [ ] Can read a plan: Seq Scan vs Index (Only) Scan vs Bitmap; `Rows Removed by Filter`; estimated vs actual rows.
- [ ] Can design a composite index for a given query (equality first, then range/sort) and state the leftmost-prefix rule.
- [ ] Can name the six reasons an index is ignored, and the write-cost trade-off. Continue with Day 43's regular plan.
