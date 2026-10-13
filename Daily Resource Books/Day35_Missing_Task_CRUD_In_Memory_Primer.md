# Day 35 — Missing Primer: Task CRUD in `todo-api` (in memory first)

**Do this first on Day 35 (Sunday; ≈ 2 h 30 min), then continue with Day 35's regular plan.**
**Needed by:** Day 36 (the entity and repository replace what you build here), Day 41 (service wired to the database), Day 46 (Mockito test of `TaskService`), Day 48 (Kafka producer attached to task creation), Day 59 (`/tasks/autocomplete`), Day 61 (design note in `TaskService`).
**Why it is missing:** Days 46, 48, 59, 61 and 62 treat `TaskController` and `TaskService` as already existing ("live since Day 34", Day 48's book), but Day 34 builds only `/health` and Day 36 only the entity and repository. No day builds the CRUD.
**Already in place:** every Day 30–33 primer, the sandbox Notes API (same pattern, different resource), `todo-api` with `/health` (Day 34).

---

## 1. What to build (in `todo-api`, no database yet)
| Class | Job |
|---|---|
| `Task` | domain class: `id` (Long), `title`, `description`, `status` (`TaskStatus {TODO, IN_PROGRESS, DONE}`), `priority` (`Priority {LOW, MEDIUM, HIGH}`), `dueDate` (`LocalDate`). A normal class, **not** a record — Day 36 turns it into a JPA entity, and entities cannot be records. |
| `CreateTaskRequest`, `UpdateTaskRequest`, `TaskResponse` | DTO records (never expose `Task` directly) |
| `TaskStore` | in-memory, thread-safe (`ConcurrentHashMap<Long, Task>` + `AtomicLong`) — **Day 41 deletes it** |
| `TaskService` | business rules; throws `NotFoundException`; logs with SLF4J |
| `TaskController` | HTTP only |
| `ApiExceptionHandler`, `NotFoundException` | as in Day 33's primer |

## 2. Endpoints and expected status codes (base path `/tasks`, matching Day 59's `/tasks/autocomplete`)
| Call | Success | Errors |
|---|---|---|
| `POST /tasks` | `201` + `Location` + `TaskResponse` | `400` with an `errors` map |
| `GET /tasks/{id}` | `200` | `404` |
| `GET /tasks?status=&priority=` | `200` + list (filters optional; use a stream) | `400` for an unknown enum value |
| `PUT /tasks/{id}` | `200` (replaces all editable fields) | `400`, `404` |
| `PATCH /tasks/{id}/status` body `{"status":"DONE"}` | `200` | `400`, `404` |
| `DELETE /tasks/{id}` | `204` | `404` |

**Validation (on the request records):** `title` `@NotBlank @Size(max = 100)`; `description` `@Size(max = 1000)`; `priority` `@NotNull`; `dueDate` optional, `@FutureOrPresent`.
**Stretch rule (exercises `409`):** a task already `DONE` cannot be edited or re-statused → `409 Conflict` with a `ProblemDetail`.

## 3. Build order (so it never gets stuck)
1. Enums + `Task` + DTOs → compile.
2. `TaskStore` + `TaskService` + a temporary `CommandLineRunner` that creates two tasks and prints them (proves the wiring before HTTP).
3. `TaskController`: `POST` and `GET /{id}`; test with `curl`.
4. Add validation + `ApiExceptionHandler`; prove `400` and `404`.
5. Add list filters, `PUT`, `PATCH`, `DELETE`; remove the temporary runner.
6. Write `docs/requests.http` (or `requests.sh`): ≥ 10 calls covering every row above, including each error path.

## 4. Definition of done
- [ ] Every call in `docs/requests.http` returns the documented status.
- [ ] Controller contains no business rules; service contains no `ResponseEntity`.
- [ ] `./mvnw test` is still green (the generated `contextLoads` test).
- [ ] Pushed to `todo-api`. Say out loud what changes on Day 36 (only `Task` and `TaskStore`) and what does **not** (controller, DTOs, handler).
- [ ] Continue with Day 35's regular plan.
