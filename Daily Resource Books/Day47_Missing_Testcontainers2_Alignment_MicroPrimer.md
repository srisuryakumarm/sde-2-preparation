# Day 47 — Missing Primer: Testcontainers 2.x Names on Boot 4.1

**Do this first on Day 47 (≈ 15 min), then continue with Day 47's regular plan.**
**Needed by:** Day 47's Project Block (replace H2 with PostgreSQL through Testcontainers), Days 62+ (integration tests per module).
**Why it is missing:** Testcontainers 2.x renamed its artifacts and the Postgres container class's package; the Day 47 book was written against the 1.x names.
**Already in place:** Docker (Day 43), `@DataJpaTest` on H2 (Day 45), a Postgres image you already run locally (Day 36).

## 1. What changed in Testcontainers 2.x (the version Boot 4 uses)
- Modules got a `testcontainers-` prefix: the Postgres module is **`org.testcontainers:testcontainers-postgresql`** (it was `org.testcontainers:postgresql`). The other modules follow the same pattern — if a coordinate in the book does not resolve, tick **Testcontainers** in a throwaway Initializr project and copy what it writes.
- The class moved: **`org.testcontainers.postgresql.PostgreSQLContainer`** (the old `org.testcontainers.containers.PostgreSQLContainer` is deprecated). Constructor/generics signatures differ slightly between 1.x and 2.x — let IntelliJ complete the call rather than copying a snippet.
- You also need `org.springframework.boot:spring-boot-testcontainers` (test scope).

## 2. Two ways to wire the container into Spring
| | Plan's way | Simpler way (Boot 3.1+) |
|---|---|---|
| How | `@DynamicPropertySource` method that does `registry.add("spring.datasource.url", postgres::getJdbcUrl)` (+ username, password) | put `@ServiceConnection` on the container field; Boot configures the datasource for you |
| Lambda/method ref | `postgres::getJdbcUrl` is a `Supplier` evaluated after the container starts (Day 30) | none |

Do it the plan's way first (it shows what is really happening), then switch to `@ServiceConnection` and delete the boilerplate.

## 3. Things that bite
- Keep `@DataJpaTest`'s default H2 replacement from winning: add `@AutoConfigureTestDatabase(replace = Replace.NONE)`.
- Docker must be running before `./mvnw test`; the first run pulls the image (slow).
- Pin the image (`postgres:16`) so tests use the same major version as your dev database.
- Flyway runs against the real container now, so the Day 45 "Flyway on H2" workaround can go.

## Checklist
- [ ] Repository tests pass against a Postgres container; H2 dependency/workaround removed.
- [ ] Can explain `@DynamicPropertySource` vs `@ServiceConnection` in two sentences.
- [ ] Continue with Day 47's regular plan.
