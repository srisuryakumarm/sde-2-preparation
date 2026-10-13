# Day 33 — Missing Primer: Bean Validation, Global Error Handling, and the Day-34 Readiness Gate

**Do this first on Day 33 (≈ 2 h), then continue with Day 33's regular plan.**
**Needed by:** Day 34 (the Validation starter is ticked in Initializr), Day 35 (Task CRUD with `400`/`404` responses), Day 46 (the "not-found path" you test), every controller afterwards.
**Why it is missing:** Day 34 adds the Validation starter, but Bean Validation, `@ControllerAdvice`/`@ExceptionHandler` and `ResponseEntity` appear nowhere in the 148 books.
**Already in place:** Parts A and B of Day 32 and the Notes API in `spring-sandbox`.

---

## 1. Bean Validation — constraints on the request record
```java
public record CreateNoteRequest(
        @NotBlank @Size(max = 100) String title,
        @Size(max = 1000) String body) {}

@PostMapping
public ResponseEntity<NoteResponse> create(@Valid @RequestBody CreateNoteRequest req) { ... }
```
- Imports are `jakarta.validation.constraints.*` (not `javax`).
- Common constraints: `@NotBlank` (strings), `@NotNull`, `@Size`, `@Min`/`@Max`, `@Email`, `@FutureOrPresent` (dates).
- `@Valid` is what triggers them. A failing `@RequestBody` raises `MethodArgumentNotValidException` → Spring answers `400` with a generic body.
- Constraints placed directly on `@PathVariable`/`@RequestParam` parameters (e.g. `@RequestParam @Min(1) int page`) are validated automatically on Spring 6.1+ and answer `400` (`HandlerMethodValidationException`). Do **not** add `@Validated` to the controller class — that switches to an older AOP route with a different exception.
- Rule of thumb: validate the *shape* at the boundary (the DTO); enforce true invariants again where they matter (service/DB constraint).

## 2. One place for errors
```java
@RestControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    public ProblemDetail notFound(NotFoundException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail invalid(MethodArgumentNotValidException ex) {
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, "Validation failed");
        Map<String, String> errors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(fe -> errors.put(fe.getField(), fe.getDefaultMessage()));      // lambda from Day 30
        pd.setProperty("errors", errors);
        return pd;
    }
}
```
- `NotFoundException` is your own `extends RuntimeException`; services throw it, only this class knows about HTTP.
- `ProblemDetail` is Spring's built-in error body (RFC 9457: `type`, `title`, `status`, `detail`, plus your own properties).
- **Design note:** Day 34 wraps `/health` in `ResponseWrapper<T>`. For CRUD, either return resources directly with correct status codes and `ProblemDetail` for errors (the common REST style — an envelope just repeats what the status already says), or wrap everything consistently. Pick one and be ready to defend it; Day 137's API-design work asks exactly this.

## 3. Exercise C — finish the sandbox (30 min)
Add validation to `CreateNoteRequest`; add `NotFoundException` and `ApiExceptionHandler`; add `PUT /{id}` and keep `DELETE`. Prove with `curl`:
`201` + `Location` · `GET` of a missing id → `404` with a `ProblemDetail` body · blank title → `400` with an `errors` map · `DELETE` → `204`; `DELETE` again → `404`.

## 4. Day-34 readiness gate (≈ 45 min)
**Cold-build drill (25 min max, do not open the sandbox):** new Initializr project (Spring Web + Validation only) → copy `ResponseWrapper<T>` from Day 16 → a `HealthController` returning `ResponseWrapper<String>` → `./mvnw spring-boot:run` → `curl -i localhost:8080/health` shows `200` and a JSON body.

**Explain out loud, no notes (60–90 s each):**
- [ ] IoC vs DI; why constructor injection; bean, `ApplicationContext`, singleton and its thread-safety consequence.
- [ ] What `@SpringBootApplication` expands to; what auto-configuration is and how it "backs off".
- [ ] The six-step path of a request from `curl` to JSON.
- [ ] `@PathVariable` vs `@RequestParam` vs `@RequestBody`, one URL each; why DTOs, not entities.
- [ ] Where the Boot version lives in a pom; why starters have no version; the five `./mvnw` commands.
- [ ] The Day-34 `DataSource` failure and why Day 34 will not include JPA/PostgreSQL.
- [ ] Cold-build drill finished in ≤ 25 min; sandbox (Days 31–33) pushed.

**Fail 3 or more →** spend the first hour of Day 34 on those items before Linked Lists. **Time-box rule:** if any primer ran over, stop at the box, copy unfinished items under "Carry-over" at the bottom of this file, and move on — the gate above is the only hard stop.

## Carry-over
- [ ] …
