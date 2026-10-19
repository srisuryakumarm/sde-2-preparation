# Day 78 — Missing Primer: Pagination — Offset vs Keyset (Cursor)

**Do this first on Day 78 (≈ 40 min), then continue with Day 78's regular plan.**
**Needed by:** Day 78's `todo-api` pagination task, every list endpoint afterwards (orders, products), HLD answers (feeds, search, leaderboards), and SDE-2 API-design questions ("how would you paginate 100 million rows?").
**Why it is missing:** the books use Spring's `Pageable` but never contrast offset and keyset pagination; "keyset"/"cursor pagination" appear nowhere in the repo.
**Already in place:** Task CRUD on Postgres (Days 35–41), indexes and `EXPLAIN`-style thinking (Day 40), DTOs (Day 32), query-cost intuition (Day 40).

---

## 1. Offset / limit (page + size)
```sql
SELECT * FROM tasks ORDER BY created_at DESC LIMIT 20 OFFSET 4000;
```
- **Pros:** trivial; supports "jump to page 200" and "page 3 of 57".
- **Cons:** the database must produce and **discard** the first 4,000 rows — cost grows with page depth. And it is **unstable**: if rows are inserted or deleted between requests, items shift, so users see **duplicates or skipped rows**. A total count needs an extra `COUNT(*)`.

## 2. Keyset (seek / cursor) pagination
Remember where you stopped and continue *after* it:
```sql
SELECT * FROM tasks
WHERE (created_at, id) < (:lastCreatedAt, :lastId)        -- row-value comparison
ORDER BY created_at DESC, id DESC
LIMIT 21;                                                  -- ask for size + 1: the extra row tells you "has next"
```
- **Constant cost** per page (an index seek on `(created_at DESC, id DESC)`), and **stable** under concurrent inserts.
- **Needs** a deterministic, unique sort key (add `id` as the tiebreaker) and a matching composite index.
- **Cons:** no random "jump to page N", no cheap total count, sort order is fixed by the key.
- The client gets an opaque **cursor** (the last row's key, e.g. Base64-encoded JSON) in the response: `{ "items": [...], "nextCursor": "eyJ…" }`.

## 3. In Spring Data
- `Pageable` / `PageRequest.of(page, size, Sort)`: `Page<T>` runs the data query **plus a `COUNT`**; `Slice<T>` skips the count and fetches `size + 1` — use `Slice` when you don't show totals.
- Always include a unique tiebreaker in the `Sort`, or pages can overlap.
- For keyset, Spring Data's **Scroll API** (`Window<T>`, `ScrollPosition`) supports it for derived queries — check the reference docs for your version — or write the `WHERE (…) <` query with `@Query`.
- Cap `size` (e.g. max 100) so a client can't request a million rows.

## 4. Choosing
| Need | Pick |
|---|---|
| admin table, small data, "go to page N" | offset |
| infinite scroll, feeds, large/changing tables, APIs for machines | **keyset** |
| show a total count | offset + `COUNT` (or an estimate) |

## 5. Exercise (≈ 25 min, `todo-api`)
1. Seed 100,000 tasks: `INSERT INTO tasks (title, status, priority) SELECT 'task ' || g, 'TODO', 'LOW' FROM generate_series(1, 100000) g;` (adjust columns to your schema).
2. `EXPLAIN ANALYZE` offset 90,000 vs a keyset query for the same position; compare time and rows scanned. Add the composite index and re-run.
3. Implement `GET /tasks?size=&cursor=` returning `{items, nextCursor}`; insert a row while paging and show that offset would repeat an item but your cursor does not.

## Checklist
- [ ] Can explain why deep offsets are slow and why they produce duplicates/skips.
- [ ] Can write a keyset query and name the index it needs.
- [ ] Can say `Page` vs `Slice` and when each is right.
- [ ] Continue with Day 78's regular plan.
