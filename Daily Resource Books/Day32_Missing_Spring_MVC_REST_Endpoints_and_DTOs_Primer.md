# Day 32 — Missing Primer (Part B): Spring MVC — REST Endpoints, DTOs, Layering, Logging

**Do this first on Day 32 (≈ 2 h 15 min, after Part A), then continue with Day 32's regular plan.**
**Needed by:** Day 34 (`/health` controller), Day 35 (Task CRUD), Day 59 (`/tasks/autocomplete?prefix=` — v1 said Day 68), every controller from Day 62 on.
**Supersedes:** Part 5 of `Day34_Missing_Spring_and_Spring_Boot_Primer.md` (which bound `@RequestBody` straight to the `Task` entity).
**Already in place:** Spring core (Day 31), HTTP/JSON/`curl` (Part A), records (Day 28), `Optional`/Streams (Day 30), generics (`ResponseWrapper<T>`, Day 16).

---

## 1. Controllers and request binding
```java
@RestController
@RequestMapping("/api/notes")                          // base path shared by the methods below
public class NoteController {
    private final NoteService service;
    public NoteController(NoteService service) { this.service = service; }

    @PostMapping                                       // @RequestBody: the payload of POST/PUT/PATCH
    public ResponseEntity<NoteResponse> create(@RequestBody CreateNoteRequest req) {
        NoteResponse created = service.create(req);
        return ResponseEntity.created(URI.create("/api/notes/" + created.id())).body(created);   // 201 + Location
    }

    @GetMapping("/{id}")                               // @PathVariable: WHICH resource
    public NoteResponse get(@PathVariable long id) { return service.get(id); }

    @GetMapping                                        // @RequestParam: filter / search / paginate OVER a resource
    public List<NoteResponse> list(@RequestParam(required = false) String contains) { return service.list(contains); }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable long id) {
        service.delete(id);
        return ResponseEntity.noContent().build();     // 204
    }
}
```
- Return a plain object for "`200` + body"; return `ResponseEntity` when you must control the status or headers (`created`, `noContent`, `notFound`).
- **Idempotent** (repeating it leaves the same state): `GET`, `PUT`, `DELETE`. **Not**: `POST`. `PUT` replaces the whole resource; `PATCH` changes part of it. (Interview staple.)
- `@Controller` (not `@RestController`) returns *view names*, not JSON — a classic first bug.

## 2. DTOs, not entities
Request and response types are **separate records** from your domain/entity class:
```java
public record CreateNoteRequest(String title, String body) {}
public record NoteResponse(long id, String title, String body) {
    static NoteResponse from(Note n) { return new NoteResponse(n.id(), n.title(), n.body()); }   // Note is a record too
}
```
Three reasons: (1) you do not leak internal fields; (2) clients cannot set fields they must not (`id`, `status` — "over-posting"); (3) your API is not welded to your DB schema (Day 36 changes the entity, not the API).

## 3. Layering
**Controller** = HTTP only (binding, status codes). **Service** = business rules (and, from Day 41, transactions). **Repository/store** = data access. No `ResponseEntity` in a service; no business rule in a controller. It is Day 6's single-responsibility principle, applied.

## 4. Logging
```java
private static final Logger log = LoggerFactory.getLogger(NoteService.class);   // SLF4J ships with Boot
log.info("created note id={}", id);        // {} placeholders: the string is not built unless the level is enabled
```
Levels: `ERROR` · `WARN` · `INFO` (default) · `DEBUG` · `TRACE`. Never `System.out.println` in services. (Lombok's `@Slf4j` is only sugar for that one line — skip Lombok until you can write the boilerplate yourself.)

## 5. Exercise B — in-memory Notes API (75–90 min, `spring-sandbox`)
- `Note` (a record, so `n.id()` works), `CreateNoteRequest`, `NoteResponse`.
- `NoteStore` — a `@Repository` backed by `ConcurrentHashMap<Long, Note>` plus an `AtomicLong` id generator. Say out loud *why* it must be thread-safe (singleton + many request threads). Day 39 explains `ConcurrentHashMap` internals; for now it is "a `HashMap` that is safe under concurrency".
- `NoteService` (SLF4J logging) and `NoteController` with: `POST /api/notes` → `201` + `Location`; `GET /api/notes/{id}` → `200` (the `404` arrives in Day 33's primer); `GET /api/notes?contains=x` → filtered list through a stream.
- Run the `curl` commands from Part A §1. Then POST without `Content-Type` and confirm `415`. Then POST `{"title":"x"}` and note that a missing `body` arrives as `null` (validation arrives on Day 33).

## Checklist
- [ ] One example URL each for `@PathVariable`, `@RequestParam`, `@RequestBody`.
- [ ] Three reasons for DTOs instead of entities.
- [ ] Which status for create / delete / missing / invalid; which verbs are idempotent.
- [ ] Exercise B pushed; all `curl` calls behave as described.
- [ ] Continue with Day 32's regular plan.
