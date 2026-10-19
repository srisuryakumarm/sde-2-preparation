# Day 40 — Missing Primer: Transaction Isolation Levels, Anomalies, and MVCC

**Do this first on Day 40 (≈ 60 min, on SQL Fundamentals day), then continue with Day 40's regular plan.**
**Needed by:** Day 41 (`@Transactional`), Days 103–104 (SQL practice), Day 117 (BookMyShow locking), Day 130 (payment idempotency) — and any backend interview that asks "what is an isolation level?".
**Why it is missing:** the Day 36 ACID micro-primer defines the "I" in ACID in a line. The terms "isolation level", "MVCC", "phantom read" and `SELECT … FOR UPDATE` appear nowhere in the repo's books or primers (the curriculum map says the ACID primer covers them; it does not).
**Already in place:** ACID and SQL micro-primers (Day 36), a running Postgres and `psql` (Day 36), race conditions (Days 29, 37).

---

## 1. The question isolation answers
Two transactions run at the same time. **How much of each other's work may they see?** The answer is the *isolation level*. Stronger isolation = fewer surprises, more blocking or retries.

## 2. The anomalies
| Anomaly | What happens |
|---|---|
| **Dirty read** | you read another transaction's *uncommitted* change |
| **Non-repeatable read** | you read the same row twice and get different values (someone committed in between) |
| **Phantom read** | you run the same query twice and get a different *set of rows* |
| **Lost update** | two read-modify-write cycles overwrite each other (`balance = 100 + 10` twice → 110, not 120) |
| **Write skew** | two transactions read overlapping data, each writes a *different* row, and together they break a rule (two doctors both go off call, leaving none) |

## 3. SQL-standard levels (what each is *allowed* to show you)
| Level | Dirty | Non-repeatable | Phantom |
|---|---|---|---|
| Read Uncommitted | allowed | allowed | allowed |
| Read Committed | no | allowed | allowed |
| Repeatable Read | no | no | allowed (by the standard) |
| Serializable | no | no | no — the result equals *some* one-at-a-time order |

## 4. What PostgreSQL actually does
- **Default is Read Committed:** every *statement* sees a fresh snapshot of committed data.
- Read Uncommitted behaves as Read Committed — Postgres never shows dirty reads.
- **Repeatable Read = snapshot isolation:** one snapshot per *transaction*; no phantoms; but write skew is possible, and if you update a row someone else changed since your snapshot you get `could not serialize access due to concurrent update` (SQLSTATE `40001`) — **your code must retry**.
- **Serializable** (SSI) detects dangerous dependency cycles and aborts one transaction with `40001` — again, retry.
- (MySQL/InnoDB defaults to Repeatable Read and uses gap locks — know that it differs.)

## 5. MVCC — why readers don't block writers
Postgres keeps **several versions of a row**. Each version records the transaction that created it (`xmin`) and the one that deleted/replaced it (`xmax`). A transaction's snapshot decides which version it sees. An `UPDATE` writes a *new* version; old versions are removed later by **VACUUM**. Consequences: reads never wait for writes; long-running transactions keep old versions alive (table bloat); that is why you keep transactions short.

## 6. Locks, when MVCC is not enough
- `SELECT … FOR UPDATE` locks the matched rows until commit (others who try to lock/update them wait). `FOR SHARE` blocks writers only. `SKIP LOCKED` lets many workers pull *different* rows from a queue table. `NOWAIT` fails instead of waiting.
- **Deadlock:** A locks row 1 then wants row 2; B locks row 2 then wants row 1. Postgres detects it and aborts one with `40P01`. Prevent it with a **consistent lock order** and short transactions.
- **Optimistic locking:** add a `version` column (`@Version` in JPA) and update `WHERE id=? AND version=?`; zero rows updated = someone else won → retry (Day 117).

## 7. Choosing, in practice
Read Committed is right for most code. Prevent lost updates with an **atomic update** (`SET balance = balance + 10`), `FOR UPDATE`, or optimistic locking. Use Repeatable Read/Serializable only with a retry loop. **Never hold a database transaction open across a network call** (HTTP, Kafka) — it holds locks and old versions for the duration.
In Spring: `@Transactional(isolation = Isolation.REPEATABLE_READ)`; `DEFAULT` means "the database default". Serialization/lock failures surface as `CannotSerializeTransactionException`/`CannotAcquireLockException` — catch and retry.

## 8. Exercise (30 min, two `psql` windows on `tododb`)
```sql
CREATE TABLE accounts(id int PRIMARY KEY, balance int NOT NULL);
INSERT INTO accounts VALUES (1,100),(2,100);
```
1. **Non-repeatable read.** A: `BEGIN; SELECT balance FROM accounts WHERE id=1;` B: `UPDATE accounts SET balance=50 WHERE id=1;` (autocommit). A: select again → 50. Repeat with A starting `BEGIN ISOLATION LEVEL REPEATABLE READ;` → still 100.
2. **Lost update.** Both sessions `BEGIN; SELECT balance …;` (100), then each runs `UPDATE accounts SET balance=110 WHERE id=1; COMMIT;` → one increment lost. Fix it with `UPDATE … SET balance = balance + 10` and again with `SELECT … FOR UPDATE`.
3. **Deadlock.** A updates row 1 then row 2; B updates row 2 then row 1 (interleave by hand). Read the `deadlock detected` error.
4. *Stretch — write skew:* table `oncall(doctor text, on_duty bool)` with two `true` rows; each session checks "count ≥ 2" then sets itself `false`. Succeeds at Repeatable Read; at `SERIALIZABLE` one gets `40001`.

## Checklist
- [ ] Can define dirty read, non-repeatable read, phantom, lost update, write skew in one sentence each.
- [ ] Can state Postgres's default level and what Repeatable Read really is.
- [ ] Can explain MVCC and why long transactions hurt.
- [ ] Can name three ways to prevent a lost update.
- [ ] Exercises 1–3 done. Continue with Day 40's regular plan.
