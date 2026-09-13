# Day 139 — CI/CD Finalization, Health Checks, and Documentation

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 138 Resource Book](Day138_Resource_Book.md)
**Next ▶:** [Day 140 Resource Book](Day140_Resource_Book.md)
**Companion to:** Day 139 of `Week_20_Revised.md`

---

> ⚠️ **The same flag as Day 134, resurfacing.** Today's plan again says readiness/liveness probes were "already configured back in the DSA phase for `todo-api`." As established Day 134: `00_Curriculum_Map.md` shows no Kubernetes deployment for `todo-api` anywhere in its history — Docker and Docker Compose only, frozen since Day 62 — so there's nothing to corroborate a prior probe configuration either. Not re-litigated at length a second time; today's probes get built as genuinely new material regardless, exactly as Day 134 already decided to treat the underlying claim.

---

## Recap

Today closes a loop opened across three separate weeks. **TestContainers vs. WireMock** (Week 7, Day 47; Week 8, Day 52) — a real dependency for realism versus a fake one for control — is the direct ancestor of today's "coverage isn't the same as verified correctness" theme: both were originally taught precisely because *what* a test actually exercises matters more than whether a test merely runs. **Actuator and Micrometer** (Week 11, Days 76–77) already expose `/actuator/health` and `/actuator/prometheus`; today extends that exact family with two new, more specific sub-paths. **Helm** (Day 135) gets validated today the same way Day 134's raw manifests were — a `--dry-run`-style check, now applied to a rendered chart instead of a hand-written manifest.

---

## Learning Objectives

By the end of today, without notes:

1. Explain, with a concrete counterexample, why a high code-coverage percentage does not imply a test suite actually verifies correctness — and what mutation testing checks instead.
2. Configure JaCoCo so a coverage drop below a set floor genuinely fails a build, not merely appears in a report nobody reads.
3. State the precise mechanical difference between a liveness probe's failure response and a readiness probe's failure response, and explain why checking a downstream dependency in a liveness probe is a well-known anti-pattern.
4. Build a complete CI/CD pipeline — test, coverage gate, image build and push, Helm chart validation — and trace exactly what happens, step by step, when a commit breaks a test or drops coverage.

---

## Concept Dependency Map

```
Week 7, Day 47: TestContainers — a real dependency, for realism
Week 8, Day 52: WireMock — a fake dependency, for control (contrasted directly)
Week 11, Days 76-77: Actuator + Micrometer -> /actuator/prometheus, live
Day 134: kubectl apply --dry-run=client; Service endpoints (Day 134's
         "empty endpoints" mechanism, reused today for a different purpose)
Day 135: helm install / helm template
        │
        ▼
Day 139
        │
        ├── Coverage vs. correctness — a real counterexample, then
        │     mutation testing named as the honest (if costlier) alternative
        │
        ├── JaCoCo wired as an enforced build-failing gate, not just a report
        │
        ├── GitHub Actions: checkout -> JDK 17 -> mvn test (coverage gate
        │     runs inside this) -> Docker build+push -> helm lint / template
        │     validation -- traced end to end, including the failure path
        │
        ├── Liveness vs. Readiness probes — different failure response,
        │     different Endpoints consequence, one genuine anti-pattern
        │
        └── README finalized — the platform's single source of truth
        │
        ▼
Day 140: this pipeline and these probes are simply "how the platform
         behaves" now — cited, not re-explained, in the final analysis
```

---

# Part 1 — Why Coverage Percentage Isn't the Same Claim as "Well Tested"

**A real counterexample, not an assertion:**

```java
public int divide(int a, int b) {
    return a / b;   // no zero-check at all
}

@Test
void testDivide() {
    divide(10, 2);   // the line executes -- "covered" -- but nothing is asserted
}
```

This test gives `divide()` **100% line coverage**. It verifies **nothing** — not that the result is actually `5`, and certainly not the missing divide-by-zero behavior. Coverage counts *lines executed*, not *correctness confirmed* — a distinction easy to state and easy to forget while staring at a green coverage badge.

**Mutation testing — the honest, if more expensive, alternative:** deliberately introduce a small "mutation" into the production code (flip `a / b` to `a * b`, or a `<` to `<=`) and re-run the existing test suite. If the mutated code still passes every test — the mutant **survives** — that's direct proof the suite isn't actually verifying that specific piece of logic, regardless of what the coverage report claims. **PIT (pitest)** is the standard real tool for this in the Java ecosystem, named here for conceptual completeness — today names the *concept*, per the plan's own scope, without wiring it into the pipeline.

**Why the imperfect proxy (JaCoCo) is still what teams actually gate CI on, stated honestly rather than glossed over:** mutation testing re-runs the entire suite once per mutant — potentially hundreds of runs for a single logic-heavy class — genuinely expensive at real scale, and slower to set up and interpret. Line/branch coverage is cheap, fast, and trivial to gate a build on. JaCoCo remains the practical default specifically because of that cost asymmetry, not because coverage is actually the better signal — worth being able to state this trade-off precisely rather than presenting coverage as sufficient on its own.

**🔑 Key Takeaway:** "well tested" and "highly covered" are different claims. A coverage percentage is a proxy for the second, cheap to measure; it says nothing direct about the first, which is what actually matters and what mutation testing actually checks.

**🔗 Backward Reference (Week 7, Day 47; Week 8, Day 52):** this is the same underlying concern that motivated choosing TestContainers (a real dependency, for realism) over WireMock (a fake one, for control) — *what* a test actually exercises matters more than whether it merely runs and passes.

---

# Part 2 — JaCoCo, Wired as an Enforced Gate, Not Just a Report

**The distinction that matters, stated precisely:** *generating* a coverage report and *failing the build* when coverage drops are two different things, and it's entirely possible to have the first without the second — a report sitting in `target/site/jacoco/index.html` that nobody opens regularly is exactly how coverage silently erodes over months.

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <id>prepare-agent</id>
            <goals><goal>prepare-agent</goal></goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
        </execution>
        <execution>
            <id>check</id>
            <goals><goal>check</goal></goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

- **`prepare-agent`** — instruments bytecode at test-run time so line/branch execution actually gets recorded.
- **`report`** — generates the human-readable HTML/XML coverage report.
- **`check`** — the piece that actually enforces anything: bound by default to Maven's `verify` phase, it compares the recorded coverage against the configured `<minimum>` and **fails the build with a non-zero exit code** if it's not met. This single goal is the entire difference between "we measure coverage" and "we enforce a coverage floor" — omitting it means everything above is purely observational.

---

# Part 3 — The GitHub Actions Pipeline, Traced End to End

```yaml
# .github/workflows/ci.yml
name: CI
on: [push]

jobs:
  build-test-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Run tests with coverage gate
        run: mvn verify   # runs tests, then JaCoCo's `check` goal — fails here if coverage < 80%

      - name: Log in to registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}/order-service:${{ github.sha }}

      - name: Validate Helm chart
        run: |
          helm lint ./helm-chart
          helm template ./helm-chart | kubectl apply --dry-run=client -f -
```

**The image tag, worth calling back to explicitly:** `${{ github.sha }}` — the commit's own SHA — replaces the `:latest` tag Day 134 used for local convenience. This is precisely the fix Day 134 flagged as a mistake worth revisiting: `:latest` is non-deterministic (the same tag can point at different actual bytes over time) and defeats "build once, deploy many times" (Day 136) by making it ambiguous *which* build a running Pod is actually running. A SHA-based tag is immutable — one tag, one exact set of bytes, forever.

**Helm chart validation, combining two already-taught mechanisms into one line:** `helm template` renders the chart to plain Kubernetes YAML (Day 135); piping that straight into `kubectl apply --dry-run=client -f -` validates the *rendered output* against the real Kubernetes schema (Day 134) — proving not just that the chart's own templates are syntactically valid Go templates, but that what they actually *produce* is valid Kubernetes YAML, in one combined step.

**Traced precisely — what happens when a commit breaks a test or drops coverage below 80%:**

1. `mvn verify` runs the test suite; JaCoCo's `check` goal evaluates coverage against the 80% floor.
2. Either the tests themselves fail, or `check` fails due to insufficient coverage — either way, the `mvn verify` command exits with a **non-zero status code**.
3. GitHub Actions marks this step (and therefore the entire job) as **failed**.
4. Every subsequent step — Docker build, registry push, Helm validation — is **skipped by default**, since a failed step halts the job unless a later step explicitly opts in with `if: always()` or similar.
5. **Nothing broken ever reaches the container registry, let alone gets deployed.** This is the actual mechanical payoff of a "quality gate" — not a policy statement, but a specific, traceable sequence of skipped steps that makes shipping a coverage-dropping or test-breaking change to production structurally impossible through this pipeline.

---

# Part 4 — Readiness vs. Liveness Probes

Both extend the same Actuator/Micrometer `/actuator/health` family already live since Week 11 — Spring Boot exposes `/actuator/health/liveness` and `/actuator/health/readiness` once `management.endpoint.health.probes.enabled=true` is set (and largely auto-detects when running inside Kubernetes to enable these probe-specific groups on its own).

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8081
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8081
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 3
```

- **`initialDelaySeconds`** — a grace period before the *first* check, avoiding a false failure while the app is still starting up.
- **`periodSeconds`** — how often the probe re-checks.
- **`failureThreshold`** — consecutive failures required before Kubernetes actually acts — a single transient blip doesn't trigger anything.

**The mechanical difference that actually matters, stated as a clean contrast:**

| | **Liveness failure** | **Readiness failure** |
|---|---|---|
| Kubernetes response | **Kills and restarts** the container | **Removes the Pod from the Service's Endpoints** (Day 134's exact mechanism) — traffic stops routing to it |
| Container survives? | No | Yes — untouched, just temporarily out of rotation |
| Implicit assumption | A restart will fix whatever's wrong | The problem is transient and will resolve on its own |

**⚠️ Common Mistake — a well-known, genuinely damaging anti-pattern:** configuring the **liveness** probe to check a downstream dependency (e.g., "can I reach the database"). If that dependency has a brief outage, **every** Pod's liveness probe fails **simultaneously** — kubelet restarts all of them at once, which does nothing to fix the dependency and actively makes recovery worse (a synchronized herd of restarts, every cache and connection pool cold at exactly the moment the dependency comes back). The correct rule: **liveness should only ask "is my own process internally stuck" (can my own request-handling loop still respond at all) — never "are my dependencies healthy."** Dependency health belongs in the **readiness** probe specifically, because *removing from rotation* (not restarting) is the correct response to a dependency being temporarily unavailable — exactly the distinction the table above draws.

**The startup-sequencing failure this specifically prevents:** without a readiness probe, a Pod is added to its Service's Endpoints as soon as the container process starts — if its own database or a dependency isn't reachable yet, it starts receiving real traffic immediately and fails every early request. A readiness probe (checking the dependency, correctly, since this is readiness not liveness) keeps the Pod out of the Endpoints list until a genuine check passes, closing exactly the gap the plan describes: "a service starting before its dependencies are ready crashes on boot."

---

# Part 5 — README Finalization

The top-level `README.md` becomes the platform's single source of truth — for both an interviewer skimming the repo and future-you returning to it cold:

- **Local dev:** `docker-compose up` — explicitly still the right tool for fast local iteration; today's Kubernetes/Helm path is the *real deployment* story, not a wholesale replacement for quick local development. Both have a genuine place, and saying so explicitly avoids implying docker-compose is now obsolete.
- **Kubernetes:** `helm install ecommerce-platform ./helm-chart -f values-<env>.yaml`.
- **API docs:** Swagger UI location and path.
- **Links:** `docs/architecture.md`, `docs/chaos-test-results.md`, `docs/api-design-notes.md`, and Day 134's load test results.

**💡 Interview Insight:** "What's the difference between a liveness probe and a readiness probe, and what happens if you check dependency health in the wrong one?" is close to a guaranteed question once Kubernetes production experience is on a resume. The strongest answer is the exact contrast above, stated as a mechanical consequence (restart vs. Endpoints removal), followed unprompted by the dependency-in-liveness anti-pattern — naming a mistake you've deliberately avoided is a stronger signal than a clean textbook definition on its own.

---

# Project Block Guide (4 hrs)

**Repository:** `scalable-ecommerce-platform`.

**Task 1:** finalize the GitHub Actions pipeline exactly as above — checkout, JDK 17, `mvn verify` (JaCoCo gate at 80%), Docker build/push with a SHA-based tag, Helm chart validation.

**Task 2:** add liveness and readiness probes to every module's Kubernetes Deployment; finalize the top-level `README.md`.

**Definition of done:** a push triggers the full pipeline, including Helm chart validation; a deliberately-broken test or an injected coverage drop **fails the build** (verify this actually happens — don't just assume the configuration is correct); `kubectl get pods` shows every module passing its readiness probe before receiving traffic; the README is genuinely portfolio-ready.

---

# Career Block Guide (1 hr)

**LinkedIn:** 20 minutes of genuine engagement — commenting on 3–5 posts.

**Networking:** verify every application across all 7 target companies is actually submitted and tracked, not just drafted — and confirm every resume/portfolio link points at the current, Kubernetes-deployed state of the platform, not an earlier docker-compose-only snapshot.

---

# Day 139 — Interview Questions

**Q1. Why can a test suite have 100% line coverage on a method and still verify nothing meaningful about it?**
*Answer:* Coverage counts lines executed, not assertions made. A test that calls a method without asserting anything about its result achieves full coverage while verifying nothing — including missing a divide-by-zero bug entirely.

**Q2. What does mutation testing actually measure, and why isn't it the default instead of line coverage?**
*Answer:* It deliberately introduces small bugs into the code and checks whether the existing test suite catches them — a surviving mutant proves the suite doesn't actually verify that logic. It's not the default because it requires re-running the full suite once per mutant, which is expensive and slow at real scale compared to cheap, fast line/branch coverage.

**Q3. What's the actual difference between generating a JaCoCo report and enforcing a coverage floor?**
*Answer:* The `report` goal only produces a human-readable report — nothing prevents coverage from silently dropping over time. The `check` goal, bound to `verify`, compares actual coverage against a configured minimum and fails the build with a non-zero exit code if it isn't met — only `check` actually gates anything.

**Q4. Trace exactly what happens in the pipeline when a commit breaks a test.**
*Answer:* `mvn verify` fails with a non-zero exit code; GitHub Actions marks that step and the job as failed; every subsequent step (Docker build, push, Helm validation) is skipped by default, so nothing broken ever reaches the registry or gets deployed.

**Q5. Why tag the CI-built image with the commit SHA instead of `:latest`?**
*Answer:* `:latest` is non-deterministic — the same tag can point at different actual image contents over time, making it ambiguous which build is actually running. A SHA-based tag is immutable, tying one tag to exactly one set of bytes, which is what "build once, deploy many times" actually requires.

**Q6. What does a liveness probe failure cause Kubernetes to do, versus a readiness probe failure?**
*Answer:* A liveness failure gets the container killed and restarted. A readiness failure removes the Pod from its Service's Endpoints — traffic stops routing to it — without touching the container itself.

**Q7. Why is checking a downstream dependency in a liveness probe a well-known anti-pattern?**
*Answer:* If the dependency has a brief outage, every Pod's liveness probe fails simultaneously, causing kubelet to restart all of them at once — which doesn't fix the dependency and makes recovery worse via a synchronized restart storm. Dependency health belongs in the readiness probe, since removing from rotation (not restarting) is the correct response to a temporarily unavailable dependency.

**Q8. How do readiness probes specifically prevent the "service starts before its dependencies are ready" crash?**
*Answer:* Without one, a Pod is added to its Service's Endpoints the moment the container starts, receiving traffic before it's actually able to serve it. A readiness probe keeps the Pod out of the Endpoints list until a real check passes, so it never receives traffic until it's genuinely ready.

**Q9. What does piping `helm template` into `kubectl apply --dry-run=client -f -` actually validate that `helm lint` alone doesn't?**
*Answer:* `helm lint` checks the chart's own template syntax and structure. Piping the *rendered* output into `kubectl`'s dry-run validates that what the templates actually produce is valid Kubernetes YAML matching the real resource schema — validating the output, not just the templates that generate it.

---

## Daily Deliverable Check

- [ ] Full CI/CD pipeline live: checkout, JDK 17, `mvn verify` with an enforced 80% JaCoCo gate, SHA-tagged Docker build/push, Helm chart validation.
- [ ] Verified (not assumed) that a deliberately broken test or coverage drop actually fails the pipeline.
- [ ] Liveness and readiness probes live on every module, with dependency checks correctly placed in readiness, not liveness.
- [ ] Can state the restart-vs-Endpoints-removal distinction and the dependency-in-liveness anti-pattern from memory.
- [ ] Top-level `README.md` finalized and genuinely portfolio-ready.

---

## What Tomorrow Assumes You Already Know Cold

Day 140 treats this pipeline and these probes as simply "how the platform behaves now" — the self-check explicitly expects an unprompted "why Kubernetes with Helm instead of docker-compose" answer, and the bottleneck analysis assumes readiness probes (not liveness) are where dependency health is being checked, since a bottleneck that manifests as failed readiness checks is a different diagnostic story than one that manifests as CPU throttling or connection-pool saturation (Day 138). None of this week's individual mechanisms get re-explained tomorrow — only synthesized.
