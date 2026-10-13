# Day 36 — Missing Primer: A Running PostgreSQL, the JPA Dependencies, and Three Entity Rules

**Do this first on Day 36 (≈ 60 min), then continue with Day 36's regular plan.**
**Needed by:** Day 36's Project Block (entity, repository, datasource), Day 40 (Flyway), Day 41 (service → repository), Day 47 (Testcontainers).
**Why it is missing:** Day 36 needs a running Postgres, but Docker is taught on Day 43 (the book offers a one-line `docker run` without setting Docker up), and Day 34's JPA/PostgreSQL dependencies were removed (see Day 34's file).
**Already in place:** SQL, YAML and ACID micro-primers (all labelled Day 36 — read them first, ~15 min total), Task CRUD in memory (Day 35), `Optional` and `LocalDate` (Day 30).

---

## 1. Get a Postgres running (pick one)
Day 36's `application.yml` assumes: user `postgres`, password `postgres`, database `tododb`, port 5432.

**A — Docker (the book's one-liner).** Needs Docker Desktop installed and running (you need it from Day 43 anyway):
```bash
docker run --name todo-postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=tododb \
           -p 5432:5432 -d postgres:17          # pin a major version so a future "latest" cannot surprise you
docker exec -it todo-postgres psql -U postgres -d tododb      # open a SQL shell
```
**B — native (lighter than Docker Desktop on a small-RAM Mac):**
```bash
brew install postgresql@17
brew services start postgresql@17
export PATH="/opt/homebrew/opt/postgresql@17/bin:$PATH"       # add to ~/.zshrc too
createuser -s postgres                                         # Homebrew's default superuser is YOUR macOS user, not "postgres"
psql -d postgres -c "ALTER USER postgres PASSWORD 'postgres';"
createdb -O postgres tododb
psql -h localhost -U postgres -d tododb                        # open a SQL shell
```
`psql` survival kit: `\l` (databases) · `\c tododb` (connect) · `\dt` (tables) · `\d tasks` (describe) · `\q` (quit).

## 2. Add the dependencies you removed on Day 34
In `pom.xml`: `spring-boot-starter-data-jpa`, `org.postgresql:postgresql` (runtime scope) — and, for Day 45's `@DataJpaTest`, the matching test starter (`spring-boot-starter-data-jpa-test` on Boot 4). Use the version-proof trick (throwaway Initializr project, copy coordinates). Reload Maven. **Do not start the app until the datasource below exists.**

## 3. Datasource configuration
Rename `application.properties` to `application.yml` (after the YAML micro-primer) and use the block from Day 36's book:
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/tododb
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: update        # Day 40 replaces this with Flyway + validate
    show-sql: true
```

## 4. Three entity rules the book assumes you know
1. **An `@Entity` is a normal class, not a record:** it needs a no-argument constructor (`protected` is fine) and must not be `final`. Imports are `jakarta.persistence.*`.
2. **Enums:** `@Enumerated(EnumType.STRING)` on `status` and `priority`. The default (`ORDINAL`) stores 0/1/2 and silently corrupts data the day someone reorders the enum.
3. **Ids and dates:** `@Id @GeneratedValue(strategy = GenerationType.IDENTITY)` on `Long id`; `LocalDate dueDate` maps to a Postgres `date` column.

## 5. First-run errors
| You see | Cause | Fix |
|---|---|---|
| `Connection refused` | Postgres not running | start the container / `brew services start postgresql@17` |
| `password authentication failed for user "postgres"` | role/password mismatch | recreate per §1 |
| `database "tododb" does not exist` | not created | `createdb -O postgres tododb` |
| `Failed to configure a DataSource` | yml missing or mis-indented | re-check §3 and the YAML primer |

Proof it worked: start the app, then `\dt` and `\d tasks` in `psql` show the table Hibernate created.

## Checklist
- [ ] `psql` opens `tododb`; the table appears after the first run.
- [ ] JPA + driver + JPA test starter in the pom; no secrets other than local dev defaults in git.
- [ ] Can state the three entity rules.
- [ ] Continue with Day 36's regular plan (the JPA lesson and entity/repository).
