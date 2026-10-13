# Day 41 — Missing Primer: Wire `TaskService` to the Database (and `@Transactional`, stated precisely)

**Do this first on Day 41 (≈ 2 h 30 min — Day 41 has no theory or project block), then continue with Day 41's regular plan.**
**Needed by:** Day 46 (Mockito test of `TaskService` mocking `TaskRepository`), Day 47 (repository tests on real Postgres), Day 48 (Kafka producer on task creation), Day 59 and Day 61.
**Why it is missing:** Day 36 defines the entity and repository but never connects them to the service/controller; Day 46 then mocks a `TaskRepository` that `TaskService` is assumed to use.
**Already in place:** Task CRUD in memory (Day 35), entity + repository + Postgres (Day 36), Flyway + `validate` (Day 40), `Optional` (Day 30).

---

## 1. The change
- In `TaskService`, replace `TaskStore` with `TaskRepository` (constructor-injected). Delete `TaskStore`.
- Map between the `Task` entity and the DTOs in **one** place (e.g. `TaskResponse.from(task)` and an `apply(UpdateTaskRequest)` method on the service) — controllers keep returning DTOs only.
- `findById(id)` returns `Optional<Task>` → `orElseThrow(() -> new NotFoundException(...))`. `existsById`, `deleteById`, `findAll` exist already.
- `save(task)` both inserts (id `null`) and updates (id present) — read the SQL in the log to see which one ran.
- Add one derived query to the repository: `List<Task> findByStatus(TaskStatus status)`; Spring Data builds the query from the method name. Use it for `GET /tasks?status=`. (`Pageable` arrives on Day 78.)

## 2. `@Transactional`, stated precisely
Runs a method inside **one database transaction** (one `DataSource`): its writes commit together or not at all.
- Put it on **service** methods that write (`create`, `update`, `delete`); use `@Transactional(readOnly = true)` on pure reads.
- It rolls back on **unchecked** exceptions (`RuntimeException`, `Error`) by default — **not** on checked exceptions unless you set `rollbackFor`.
- It works through a **proxy**: calling a `@Transactional` method from another method **in the same class** bypasses it (no transaction).
- It covers **the database only**. A Kafka send inside the method is not rolled back, and the DB commit does not wait for Kafka — the dual-write problem (Day 69's primer). Remember this on Day 48.

## 3. Verify like an engineer
1. With `show-sql: true`, read the SQL for one `POST`, one `GET /{id}`, one `GET ?status=`, one `DELETE`.
2. Run Day 35's `docs/requests.http` again — **unchanged** — against Postgres. Every call must return the same status as before (that is the payoff of the controller/DTO/handler layering).
3. Restart the app: the data must survive. Look at the rows in `psql`.
4. Break it on purpose: stop Postgres and call the API; read the exception; start it again.

## Definition of done
- [ ] `TaskService` depends on `TaskRepository`; `TaskStore` is gone; controller and DTOs untouched.
- [ ] Writes are `@Transactional`, reads `readOnly = true`; you can state the three limits in §2.
- [ ] `docs/requests.http` passes unchanged; data survives a restart; pushed.
- [ ] Continue with Day 41's regular plan.
