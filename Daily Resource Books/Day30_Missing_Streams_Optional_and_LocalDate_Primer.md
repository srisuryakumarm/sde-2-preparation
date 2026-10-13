# Day 30 — Missing Primer: Streams, `Optional`, and `LocalDate`

**Do this first on Day 30 (≈ 60 min, after the Lambdas primer), then continue with Day 30's regular plan.**
**Needed by:** Day 34 onward. Mapping entities to response objects (`tasks.stream().map(TaskResponse::from).toList()`), Spring Data's `findById` (returns `Optional`), Day 36's `dueDate` (a `LocalDate`), Day 46's "not-found" test path, and code on Days 64 and 128 all assume these.
**Why it is missing:** none of the three is taught anywhere in the 148 books (Streams appear only inside code on Days 64 and 128).
**Already in place:** records (Day 28), lambdas and method references (the Lambdas primer you just did), generics (Day 16), `Comparator` (Day 17).

---

## 1. Streams — a pipeline over a collection
A stream is **source → zero or more intermediate operations → one terminal operation**. It does **not** modify the source, and nothing runs until the terminal operation is called.

```java
record Task(long id, String title, Priority priority, TaskStatus status, LocalDate due) {}

List<String> highTitles = tasks.stream()
        .filter(t -> t.priority() == Priority.HIGH)        // keep matching
        .sorted(Comparator.comparing(Task::due))            // order (NPE if due is null!)
        .map(Task::title)                                   // transform each element
        .toList();                                          // terminal: collect (Java 16+; unmodifiable list)

boolean anyOverdue = tasks.stream().anyMatch(t -> t.due().isBefore(LocalDate.now()));
long doneCount     = tasks.stream().filter(t -> t.status() == TaskStatus.DONE).count();
Optional<Task> first = tasks.stream().filter(t -> t.priority() == Priority.LOW).findFirst();
Map<TaskStatus, Long> byStatus = tasks.stream()
        .collect(Collectors.groupingBy(Task::status, Collectors.counting()));
long maxId = tasks.stream().mapToLong(Task::id).max().orElse(0);  // primitive streams: sum / max / average
```

| Operation | Kind | Use |
|---|---|---|
| `filter(pred)` | intermediate | keep elements that match |
| `map(fn)` | intermediate | transform each element |
| `sorted(cmp)`, `distinct()`, `limit(n)` | intermediate | order, de-duplicate, truncate |
| `toList()`, `collect(...)` | terminal | produce a collection / map / grouped result |
| `count()`, `anyMatch/allMatch/noneMatch`, `findFirst()` | terminal | answer a question |
| `mapToInt(...).sum()/max()` | terminal | numeric reductions |

**Pitfalls**
- A stream can be consumed **once**; reusing it throws `IllegalStateException: stream has already been operated upon or closed`.
- Do not mutate outside state inside `map`/`filter` (that is what the "effectively final" rule is warning you about).
- `Collectors.toList()` gives a mutable list; `Stream.toList()` gives an unmodifiable one — know which you have.
- For LeetCode loops over arrays, plain `for` loops are usually clearer and faster. Use streams in application code (controllers, services), not in hot inner loops.

## 2. `Optional<T>` — "this may be absent", made explicit
Spring Data's `findById(id)` returns `Optional<Task>` instead of `null`, so the compiler forces you to decide what "absent" means.

```java
Task task = repo.findById(id)
        .orElseThrow(() -> new NotFoundException("task " + id));   // Supplier lambda; runs only if empty

TaskResponse dto = repo.findById(id).map(TaskResponse::from)        // transform if present
                       .orElseThrow(() -> new NotFoundException("task " + id));

repo.findById(id).ifPresent(t -> log.info("found {}", t.title()));
String title = maybeTitle.orElse("untitled");                       // fixed default
String title2 = maybeTitle.orElseGet(() -> computeDefault());       // lazy default — orElse(compute()) ALWAYS computes
```
**Rules:** return `Optional` from methods that may find nothing. Do **not** use it for fields, method parameters, or collections (return an empty list instead). Avoid `isPresent()` + `get()` — `orElseThrow`/`map` say the same thing without the trap of calling `get()` on an empty value.

## 3. `java.time` in five lines
| Type | Use for | Examples |
|---|---|---|
| `LocalDate` | a calendar date, no time, no zone (a due date) | `LocalDate.of(2026, 10, 8)`, `LocalDate.now()`, `d.plusDays(7)`, `d.isBefore(other)`, `ChronoUnit.DAYS.between(a, b)` |
| `Instant` | a point on the UTC timeline (`createdAt`) | `Instant.now()` |
| `LocalDateTime` | date + time, no zone (rarely right for servers) | `LocalDateTime.now()` |
| `Duration` | an amount of time | `Duration.ofMinutes(5)` |

Do not use `java.util.Date`, `Calendar` or `SimpleDateFormat` (mutable, error-prone; you will still see them in old tutorials). In Postgres, `LocalDate` maps to `date` and `Instant` to `timestamptz`.

## 4. Exercise (40 min, `java-fundamentals`)
Using `record Task(long id, String title, Priority priority, TaskStatus status, LocalDate due)` and a list of 8 tasks:
1. Titles of HIGH-priority tasks, sorted by due date.
2. How many tasks are overdue (due before today)?
3. `Map<TaskStatus, List<String>>` of titles per status (`groupingBy` + `mapping`).
4. `Optional<Task> findById(long id)` over the list; a caller that throws your own `NotFoundException extends RuntimeException` through `orElseThrow`.
5. Map `List<Task>` → `List<TaskResponse>` (another record) with a `TaskResponse::from` method reference.
6. Re-run (1) after a task has `due == null`; read the `NullPointerException`; decide how you would handle it (`Comparator.nullsLast`, or reject nulls earlier).

## Checklist
- [ ] Can write `filter → map → toList()` and `groupingBy` from memory.
- [ ] Can say why `orElse(compute())` and `orElseGet(() -> compute())` differ.
- [ ] Can name three places `Optional` should *not* be used.
- [ ] Can pick `LocalDate` vs `Instant` for a due date and a `createdAt`.
- [ ] Exercises 1–6 pushed to `java-fundamentals`. Continue with Day 30's regular plan.
