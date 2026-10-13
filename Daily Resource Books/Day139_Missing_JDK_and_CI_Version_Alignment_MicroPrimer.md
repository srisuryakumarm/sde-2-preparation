# Day 139 — Missing Primer: Make CI Build with the Same JDK as the Code

**Do this first on Day 139 (≈ 15 min), then continue with Day 139's regular plan.**
**Needed by:** Day 139's GitHub Actions pipeline (build, tests, JaCoCo).
**Why it is missing:** Day 139's book sets up **JDK 17** in CI, while the code is built on **JDK 21** (Day 1; `<java.version>21</java.version>`; Day 28's exhaustive pattern `switch` needs 21). A JDK 17 runner fails with `release version 21 not supported`.
**Already in place:** the CI/CD and GitHub Actions micro-primer (Day 139), the Maven wrapper (Day 31), Testcontainers tests (Day 47).

## 1. The step
```yaml
- uses: actions/setup-java@v4            # use the latest major version shown on the action's Marketplace page
  with:
    distribution: temurin
    java-version: '21'                   # must equal <java.version> in the root pom
    cache: maven
- run: ./mvnw -B -ntp verify             # wrapper, batch mode, no download-progress noise
```

## 2. Rules
- **One source of truth:** `<java.version>` in the root POM. CI mirrors it; change both together.
- **The Boot version needs no CI setting** — it comes from the pom's `<parent>`.
- **Use `./mvnw`**, not `mvn`, so CI and your laptop run the same Maven.
- GitHub's `ubuntu-latest` runners have Docker, so Testcontainers tests (Day 47) run in CI as they do locally — do not replace them with mocks to "make CI pass".
- JaCoCo: bind the plugin's `prepare-agent` and `report` goals into the build so `verify` produces the report (Day 139's book walks through it).

## Checklist
- [ ] Workflow uses JDK 21 and `./mvnw -B -ntp verify`; the pipeline is green.
- [ ] `java.version` in the pom and `java-version` in the workflow match.
- [ ] Continue with Day 139's regular plan.
