# Day 32 — Missing Primer (Part A): HTTP, JSON, `curl`, and Running a Spring Boot App

**Do this first on Day 32 (≈ 75 min, before Part B), then continue with Day 32's regular plan.**
**Needed by:** Day 34 (`curl localhost:8080/health`), Day 35 (CRUD you test with `curl`), Day 51 (profiles), Days 96 and 136 (Kubernetes config overrides your properties through environment variables).
**Supersedes:** Parts 5, 7 and 8 of `Day34_Missing_Spring_and_Spring_Boot_Primer.md` (and adds what it left out: JSON, `curl`, reading the startup log, configuration precedence).
**Already in place:** the `spring-sandbox` project and Spring core (Days 31).

---

## 1. HTTP in practice
- **Request** = method + path (+ `?query=string`) + headers + optional body. **Response** = status code + headers + optional body.
- **Status codes you will use:** `200` OK · `201` Created (+ a `Location` header) · `204` No Content · `400` bad/invalid request · `404` no such resource · `409` conflict with current state · `500` server bug. (Day 34's book adds `422`.)
- **Content-Type matters:** a POST body sent without `Content-Type: application/json` gets `415 Unsupported Media Type`.

```bash
curl -i localhost:8080/api/notes/1                          # -i = show status line and headers
curl -i -X POST localhost:8080/api/notes \
     -H 'Content-Type: application/json' \
     -d '{"title":"first","body":"hello"}'                  # single quotes around the JSON body
```

## 2. JSON ↔ Java
| JSON | Java |
|---|---|
| object `{...}` | a class / record |
| array `[...]` | `List<T>` |
| string / number / true,false / null | `String` / `int,long,double` / `boolean` / `null` |

Spring uses **Jackson** to convert both ways. Records work out of the box. A plain class returned from a controller needs getters, otherwise Jackson throws `No serializer found for class … no properties discovered`.

## 3. What Boot actually does
- **Auto-configuration is conditional.** Boot looks at the classpath and at beans you already defined, and configures what is missing — *backing off* wherever you define your own. See exactly what it decided: `java -jar app.jar --debug` prints a conditions report ("Positive matches" / "Negative matches").
- **Embedded Tomcat** serves requests; each request runs on its own worker thread (so a singleton bean is used by many threads — Day 31).
- **The path of one request (six steps):** (1) Tomcat receives it → (2) `DispatcherServlet` is the front door → (3) a handler mapping picks the controller method by (verb, path) → (4) arguments are bound (`@PathVariable`, `@RequestParam`, `@RequestBody` via Jackson) → (5) your method runs → (6) the return value is written as JSON by an `HttpMessageConverter`. An exception on the way is routed to an `@ExceptionHandler` (Day 33).

## 4. Configuration
- Initializr creates `src/main/resources/application.properties`. You will switch to `application.yml` on Day 36 after the YAML micro-primer; both mean the same thing (`server.port=8081` ≡ `server:` / `  port: 8081`).
- **Precedence (highest first):** command-line args → environment variables → `application-{profile}.yml` → `application.yml`.
- **Environment variables map automatically:** `SERVER_PORT` ↔ `server.port`, `SPRING_DATASOURCE_URL` ↔ `spring.datasource.url`, `SPRING_PROFILES_ACTIVE` ↔ `spring.profiles.active`. This is exactly how Kubernetes ConfigMaps/Secrets (Days 96, 136) override your file.
- **Logging:** `logging.level.root=INFO` is the default; `logging.level.com.yourname=DEBUG` turns on your own package.

## 5. The Day-34 trap (read once, now)
Boot configures a `DataSource` as soon as it sees Spring Data JPA **and** a JDBC driver on the classpath. With no `spring.datasource.url` and no embedded database, startup fails fast:
> `Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.`

Day 34's weekly plan adds JPA and PostgreSQL at Initializr time — which triggers exactly this. See `Day34_Missing_Initializr_Setup_and_Boot_Version_Decision_MicroPrimer.md`.

## 6. Exercise (≈ 30 min, `spring-sandbox`)
1. Add a `@RestController` with `@GetMapping("/hello")` returning a `Map.of("message", "hi")`; start it; `curl -i localhost:8080/hello` — note status, `Content-Type`, body.
2. Change the port three ways and prove it with `curl`: `--server.port=8081` on the command line, `SERVER_PORT=8082 ./mvnw spring-boot:run`, and `server.port=8083` in `application.properties`. Which one wins when you combine them?
3. Run the jar with `--debug`. Find the "Positive matches" entry that mentions `DispatcherServlet`, and one entry under "Negative matches" with the reason it did not apply.
4. Start the app twice to trigger `Port 8080 was already in use`; find the culprit with `lsof -i :8080`.

## Checklist
- [ ] Trace one request from `curl` to JSON in six steps, out loud.
- [ ] Name the status codes in §1 and when each is right.
- [ ] State the configuration precedence and the env-var mapping rule.
- [ ] Describe the Day-34 `DataSource` failure and its cause.
- [ ] Exercise done. Continue with Part B (Spring MVC primer), then Day 32's regular plan.
