# Day 34 — Missing Primer: Initializr Setup, and the Spring Boot Version Decision

**Do this first on Day 34 (≈ 20 min), then continue with Day 34's regular plan.**
**Needed by:** Day 34's Project Block (`todo-api` is created today) — and every project day after it, because the version you pick today is the version the whole plan runs on.
**Supersedes:** the dependency list in Day 34's weekly plan ("Web, JPA, PostgreSQL, and Validation") and the version-less Initializr step in Day 34's book.
**Already in place:** Day 30–33 primers (Java bits, Maven, Spring core, MVC, validation); `ResponseWrapper<T>` from Day 16.

---

## 1. The decision
| Item | Choice |
|---|---|
| **Spring Boot** | **4.1.x — the latest 4.1 patch that start.spring.io offers on the day you generate the project** (4.1.1 shipped Aug 20, 2026; do not take 4.1.0). Pin it in the pom's `<parent>` and never change it mid-plan. |
| Java | 21 (LTS; already installed since Day 1; Boot 4.1 supports Java 17 through 26) |
| Build | Maven, through the wrapper (`./mvnw`) |
| Spring Cloud (Day 62+) | the `2025.1.x` ("Oakwood") train — copy the exact value Initializr writes (see Day 63's file) |

## 2. Why 4.1.x — the answer you can give an interviewer
Support status (as of Oct 2026): Boot 4.1.0 went GA on **June 10, 2026**, with open-source support running to about **July 2027**. Boot 4.0 ends OSS support on **Dec 31, 2026**. Boot 3.5 (and Spring Framework 6.2) ended OSS support on **June 30, 2026**. Spring Boot has no LTS line, so "stay on the old one" means "stay on an unsupported one".

1. **It is the only line that stays supported through your whole job search and first months on the job.** 3.x is already out of OSS support; 4.0 ends in under three months.
2. **It is what new Spring projects start on today:** Spring Framework 7, Jakarta EE 11, Java 17+ baseline, Jackson 3 by default, modular starters.
3. **Every library this plan uses has a Boot-4-compatible release** (checked Oct 2026): Spring Cloud 2025.1.x (Gateway, OpenFeign, Eureka; 2025.1.2 added 4.1.0 compatibility), Resilience4j 2.4+ (`resilience4j-spring-boot4`), springdoc-openapi 3.0.x, Spring for Apache Kafka 4.x (`spring-boot-starter-kafka`), Testcontainers 2.x, Flyway's own starter. No dependency forces a downgrade.
4. **The honest cost is "tutorial lag":** many tutorials and the plan's older snippets are Boot 3-era. That cost is bounded — about ten renames, listed in §5 and handled in the per-day alignment files — and handling them is itself evidence you understand what changed.

**Likely follow-ups**
- *"Why not 3.5?"* — It is the last 3.x line and its free support ended June 30, 2026; Spring has no LTS, so I chose the supported line.
- *"Why not 4.0?"* — 4.0 ends OSS support Dec 31, 2026; 4.1 is a minor upgrade on the same APIs, with more patch releases ahead of it.
- *"Why not 4.1.0?"* — I pin the latest patch (4.1.1 alone shipped 98 bug fixes over 4.1.0); patch bumps within 4.1 are routine, minor bumps are a deliberate decision.
- *"Why Java 21, not 25?"* — 21 is a widely deployed LTS and the plan's Java 21 features (sealed types, pattern-matching `switch`) are built on it; Boot 4.1 supports newer Javas, so moving later is a one-line change.

## 3. Initializr settings (start.spring.io, in the browser)
Project **Maven** · Language **Java** · Spring Boot **4.1.x** (latest, not SNAPSHOT/RC/M) · Group `com.yourname` · Artifact `todo-api` · Packaging **Jar** · Java **21**.
**Dependencies: Spring Web and Validation — and nothing else today.** Download, unzip, open the folder in IntelliJ.

## 4. Why not JPA and PostgreSQL today
Day 34's weekly plan ticks Web, JPA, PostgreSQL and Validation, and its definition of done is "the app starts". With JPA and a JDBC driver on the classpath but no `spring.datasource.url`, Boot fails at startup:
> `Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.`

The datasource is not configured until Day 36 (and Postgres is not installed until then). So: **add JPA and PostgreSQL on Day 36**, when Postgres exists (`Day36_Missing_Local_Postgres_and_JPA_Prereqs_MicroPrimer.md`). If you already generated the project with them: delete the `spring-boot-starter-data-jpa` and `postgresql` dependencies from `pom.xml` and reload Maven.

## 5. Boot 3 → Boot 4 names you will meet (use the right column)
| Plan / older tutorial says (Boot 3 era) | On Boot 4.x | First hit |
|---|---|---|
| `spring-boot-starter-web` | `spring-boot-starter-webmvc` | Day 34 |
| one `spring-boot-starter-test` for everything | per-technology test starters (e.g. `spring-boot-starter-webmvc-test`, `spring-boot-starter-data-jpa-test`); test-annotation import paths moved — let IntelliJ import | Day 36, 45 |
| `flyway-core` | `spring-boot-starter-flyway` + `flyway-database-postgresql` | Day 40 |
| `@MockBean`, `@SpyBean` | `@MockitoBean`, `@MockitoSpyBean` | Day 45–47 |
| `org.testcontainers:postgresql`, `org.testcontainers.containers.PostgreSQLContainer` | `org.testcontainers:testcontainers-postgresql`, `org.testcontainers.postgresql.PostgreSQLContainer` | Day 47 |
| `org.springframework.kafka:spring-kafka` alone | `spring-boot-starter-kafka` (without it `spring.kafka.*` is ignored) | Day 48 |
| `spring-boot-starter-aop` (Day 62 pins `3.3.0`) | `spring-boot-starter-aspectj`; never pin Boot starters | Day 62, 64 |
| `resilience4j-spring-boot3` | `resilience4j-spring-boot4` (≥ 2.4.0, give it an explicit version) | Day 64 |
| `spring-cloud-starter-gateway`, `spring.cloud.gateway.routes` | `spring-cloud-starter-gateway-server-webflux`, `spring.cloud.gateway.server.webflux.routes` | Day 66 |
| `Jwts.parserBuilder()` (JJWT 0.11) | `Jwts.parser().verifyWith(key)…parseSignedClaims(token)` (JJWT 0.12+) | Day 67 |
| springdoc `2.8.x` | springdoc `3.0.x` | Day 42, 139 |
| Jackson 2 imports `com.fasterxml.jackson.databind` | Jackson 3 `tools.jackson.databind`; plain DTO records unaffected | custom serializers only |

**Version-proof habit:** whenever a snippet's dependency does not resolve, generate a throwaway project at start.spring.io with the same Boot version, tick the dependency, and copy what it writes into `pom.xml`.

## 6. Record the decision (2 min)
Put this in `todo-api/README.md` (and, later, the platform's `docs/`):
```markdown
## ADR-001 — Spring Boot 4.1.x on Java 21
Decision: Spring Boot 4.1.<patch> (parent in pom.xml), Java 21, Maven wrapper.
Why: 4.1 is the current GA line, supported until ~July 2027; 3.5 and Spring Framework 6.2 left OSS support on 2026-06-30
and 4.0 ends 2026-12-31. All libraries used here (Spring Cloud 2025.1.x, Resilience4j 2.4+, springdoc 3.x,
Spring Kafka 4.x, Testcontainers 2.x) support Boot 4.
Cost: older tutorials use Boot 3 names (webmvc, @MockitoBean, …); mapped in the Day 34 alignment table.
```

## 7. Then do Day 34's project block
Copy `ResponseWrapper<T>` into `todo-api`; add a `HealthController` returning `ResponseWrapper<String>` at `GET /health`; `./mvnw spring-boot:run`; `curl -i localhost:8080/health` → `200`. Commit `mvnw`, `.mvn/`, `pom.xml` — never `target/`.

## Checklist
- [ ] `todo-api` generated with Boot 4.1.x (latest patch), Java 21, **Web + Validation only**.
- [ ] `/health` returns `200`; startup log has no DataSource error.
- [ ] ADR-001 written in the README.
- [ ] Can answer "why this Boot version?" in 30 seconds without notes.
- [ ] Continue with Day 34's regular plan.
