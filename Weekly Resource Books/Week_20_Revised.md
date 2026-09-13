# Week 20 (Revised): Capstone Hardening — Kubernetes, Deeper Chaos Engineering, and the Platform's Own At-Scale Exercise

**What changed, and why this week matters most out of the whole audit:** the original capstone stayed on `docker-compose` the entire time, even though two much earlier, smaller practice projects (`todo-api`, and the original plan's `order-management-api`) both got real Kubernetes deployments. That was backwards — your actual production experience is specifically Kubernetes, Docker, and multi-environment deployment, and the project meant to showcase your strongest work was the one place that depth never showed up. This week fixes that directly, deepens the single manual chaos test into something more real, and gives the platform the same formal "estimate this at scale" treatment the 16 HLD systems already got — because defending your own project's architecture under exactly that kind of questioning is an explicit goal by the end of this week.

---

## Day 134 — Load Testing with k6, and the Platform's First Kubernetes Manifests

### Theory Block (1.5 hrs)
- Topic: Load Testing Fundamentals
- Virtual Users (VUs) simulate concurrent real users; a load test script defines what each VU does and how many run simultaneously. Key metrics: request rate (throughput), p50/p95/p99 latency — the tail matters more than the average, since a p50 of 50ms can hide a p99 of 3 seconds that real users actually experience — and error rate under load.
- Coding exercise: none — the project below is the exercise.

### Project Block (3.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task, part 1: install `k6`. Write a `load-test.js` simulating 50 VUs constantly hitting the Product module's read endpoints for 30 seconds.
- Task, part 2: write the platform's first real Kubernetes manifests — `deployment.yaml` and `service.yaml` for each module (Product, Order, Payment, Notification), including resource requests/limits. This is the fix: the platform stops being docker-compose-only from today.
- Definition of done: the k6 script runs and outputs a summary showing request rate and p95 latency. Manifests are valid (`kubectl apply --dry-run=client -f .` passes) and pushed to a `k8s/` folder.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: continue tracking responses across all 7 target companies; this is a reasonable point in the timeline for first technical screens to be getting scheduled.

### Daily Deliverable
- [ ] Basic k6 load test script running and producing a summary report.
- [ ] First Kubernetes manifests for every module, validated and pushed.

---

## Day 135 — Helm Charts, and Deploying the Platform for Real

### Theory Block (1.5 hrs)
- Topic: Helm — Templating Kubernetes Manifests
- Writing separate, near-identical YAML files for every environment is exactly the kind of repetition Helm exists to eliminate. A Helm chart templates your Kubernetes manifests with placeholders, and a `values.yaml` file fills those placeholders in — one chart, many possible configurations, instead of copy-pasted YAML per environment.
- Coding exercise: none — the project below is the exercise.

### Project Block (3.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: convert yesterday's raw manifests into a proper Helm chart — templated Deployments and Services for all four modules, plus an Ingress routing to the Gateway.
- Definition of done: `helm install ecommerce-platform ./helm-chart` successfully deploys the entire platform to a local Minikube cluster; `kubectl get pods` shows every module running and healthy.

### Career Block (1 hr)
- LinkedIn: Post 26 — "I finally moved my capstone off docker-compose and onto real Kubernetes, with Helm" (a genuinely honest, differentiated post — most candidates' portfolio projects never make this jump either).
- Networking: it's been over two weeks since applications went out Day 118 — if any of the 7 companies has gone fully silent with no rejection or next step, a polite, brief follow-up to the recruiter is reasonable at this point.

### Daily Deliverable
- [ ] Helm chart deploys the full platform successfully to Minikube.
- [ ] Every module confirmed running and healthy via `kubectl get pods`.

---

## Day 136 — Multi-Environment Configuration: Dev, Staging, Production

### Theory Block (1.5 hrs)
- Topic: Multi-Environment Deployment, Properly
- The same Helm chart should deploy differently depending on environment — different replica counts, different resource limits, different database connection strings, different feature flags — without duplicating the chart itself. Separate `values-dev.yaml`, `values-staging.yaml`, and `values-prod.yaml` files, layered on top of the base chart, are the standard way to express this: this is literally the environment-management problem you already handle at your actual job, now formalized in your own project.
- Coding exercise: none — the project below is the exercise.

### Project Block (3.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: create `values-dev.yaml` (1 replica per module, relaxed resource limits, verbose logging), `values-staging.yaml` (2 replicas, moderate limits, staging database), and `values-prod.yaml` (3+ replicas, tighter limits, production-grade config, minimal logging). Wire ConfigMaps and Secrets per environment.
- Definition of done: `helm install ... -f values-dev.yaml` and `helm install ... -f values-prod.yaml` produce visibly different deployments — different replica counts, different config — from the exact same underlying chart.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: research the hiring manager for a role you've applied to and send a direct, specific message.

### Daily Deliverable
- [ ] Three environment-specific values files live, each producing a visibly different deployment from the same chart.

---

## Day 137 — Distributed Tracing, For Real This Time

### Theory Block (1.5 hrs)
- Topic: Distributed Tracing, Implemented
- Micrometer Tracing generates a Trace ID when a request enters the Gateway and propagates it automatically through every downstream call, tagging log lines with it — turning "which module actually caused this slow request" from a guessing game into a single Zipkin UI lookup, across a system that's now actually running in Kubernetes instead of on a laptop.
- Coding exercise: none — the project below is the exercise.

### Project Block (3.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: integrate Micrometer Tracing and Zipkin across all four modules, deployed via the Helm chart.
- Definition of done: hitting the Gateway (now running in Kubernetes) generates a Trace ID that propagates through every module it touches, viewable as one complete trace in the Zipkin UI.
- **Add-on, 30 min — Stripe-specific API design practice:** pick one existing platform endpoint (e.g., Order creation) and redesign it with explicit rigor: versioning strategy (`/v1/orders` vs. header-based), pagination approach for the list endpoint, idempotency key handling (you already built this for Payment — apply the same thinking here), and a real error-contract (structured error codes, not just HTTP status). Write the redesigned API spec in `docs/api-design-notes.md`. This is close to exactly what Stripe's API design round actually evaluates.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: research the hiring manager for a role you've applied to and send a direct, specific message.

### Daily Deliverable
- [ ] A single request's full trace visible across all four modules in Zipkin, running in Kubernetes.
- [ ] API design notes for one redesigned endpoint pushed.

---

## Day 138 — Chaos Engineering, Deepened, and a Light Service Mesh

### Theory Block (1.5 hrs)
- Topic: Chaos Engineering, Beyond One Manual Test
- Deliberately injecting failure into a running system — killing an instance, adding artificial network latency, saturating a dependency — verifies your system degrades gracefully instead of catastrophically, tested under conditions you actually chose rather than discovered during a real incident. A single manual container kill is a reasonable first exposure, but three distinct, semi-automated failure modes tell a much stronger story about what you actually understand.
- Also today: a light service mesh introduction. A service mesh (Istio or Linkerd) adds a sidecar proxy alongside each pod, handling traffic routing, retries, and observability at the network layer instead of inside application code — even a minimal exposure (sidecar injection plus one traffic-splitting demo) is a differentiated addition most candidates' portfolios don't have.
- **Databricks-specific add-on:** their concurrency round is repeatedly described by real candidates as the hardest part of their loop — real implementation (a thread-safe logger, a bounded queue with backpressure), not a LeetCode pattern. Today's chaos experiments already exercise your understanding of concurrent failure modes under load — spend an extra 30 minutes implementing one small, genuinely thread-safe component from scratch (a bounded blocking queue using `ReentrantLock`/`Condition`, no `java.util.concurrent` shortcuts) and write a test proving it's correct under concurrent access. This is closer to Databricks' actual bar than the Producer-Consumer exercise from Week 6 alone.
- Coding exercise: none — the project below is the exercise.

### Project Block (3.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task, part 1: while a k6 load test runs against the Kubernetes-deployed platform, run three distinct chaos experiments — (a) kill the Payment pod directly and observe the Order module's circuit breaker, (b) use Toxiproxy or Pumba to inject artificial latency into the Product module and observe Gateway-level timeout behavior, (c) saturate the Order module's connection pool and observe whether it fails gracefully or cascades.
- Task, part 2: install a lightweight service mesh (Istio or Linkerd) on the Minikube cluster; inject sidecars into the platform's pods; demonstrate one traffic-split (e.g., 90/10 between two versions of one module).
- Task, part 3: the hand-rolled thread-safe bounded blocking queue from the theory block, in `java-fundamentals`, with a concurrent-access test proving correctness.
- Definition of done: documented findings in `docs/chaos-test-results.md` for all three experiments — what failed, what didn't, whether the fallback behavior matched what Resilience4j was actually configured to do. Service mesh traffic split demonstrated and screenshotted. Thread-safe queue implementation pushed and verified.

### Career Block (1 hr)
- LinkedIn: Post 27 — "I ran three chaos experiments against my own Kubernetes deployment — here's what broke and what didn't" (real chaos-testing results against a real K8s deployment is a genuinely strong, differentiated post).
- Networking: continue application push.

### Daily Deliverable
- [ ] Three chaos experiments executed and documented with real evidence, not just claims.
- [ ] Service mesh sidecar injection and traffic split demonstrated. LinkedIn Post 27 published.
- [ ] Hand-rolled thread-safe bounded blocking queue implemented and tested under concurrent access.

---

## Day 139 — CI/CD Finalization, Health Checks, and Documentation

### Theory Block (1 hr)
- Topic: CI/CD Quality Gates, and Reliable Startup Sequencing
- High code coverage doesn't mean good tests — mutation testing (deliberately introducing small bugs and checking whether your tests actually catch them) is the honest measure, though JaCoCo's line/branch coverage is the practical, widely-used proxy. A CI pipeline that only runs tests but doesn't *enforce* a coverage floor lets quality quietly erode over time. Separately: a service starting before its dependencies are ready crashes on boot — Kubernetes readiness/liveness probes (which you already configured back in the DSA phase for `todo-api`) are what prevent this in a real cluster, rather than a fixed startup delay and hoping for the best.
- Coding exercise: none — the project below is the exercise.

### Project Block (4 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task, part 1: finalize the GitHub Actions pipeline — checkout, JDK 17 setup, `mvn test`, JaCoCo coverage check (fail below 80%), Docker image build, push to a container registry, and a final step that packages and validates the Helm chart itself.
- Task, part 2: add liveness and readiness probes to every module's Kubernetes deployment; finalize the top-level `README.md` — how to run the full stack locally (`docker-compose` for quick local dev) and in Kubernetes (`helm install`), how to access Swagger, links to `docs/architecture.md`, `docs/chaos-test-results.md`, and load test results.
- Definition of done: a push triggers the full pipeline, including Helm chart validation; a deliberately-broken test or coverage drop fails the build; `kubectl get pods` shows every module passing its readiness probe before receiving traffic; README is genuinely portfolio-ready.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: verify every application across all 7 target companies is actually submitted and tracked, not just drafted — and confirm resume/portfolio links in each application still point to the current, Kubernetes-deployed state of the platform.

### Daily Deliverable
- [ ] Full CI/CD pipeline live, including Helm chart validation, correctly failing on broken tests, low coverage, or an invalid chart.
- [ ] Readiness/liveness probes live on every module. Top-level README finalized and portfolio-ready.

---

## Day 140 (Sunday) — Bottleneck Analysis, the Platform's Own At-Scale Exercise, and Capstone Complete

### Self-Check (15 min)
- [ ] Without looking, explain why the platform uses Saga instead of 2PC, why the Gateway does rate limiting instead of each module doing it independently, and why it now runs on Kubernetes with Helm instead of docker-compose — these are exactly the "why this, not that" questions a project deep-dive interview asks.

### Project Block (4 hrs) — today has two distinct halves

**Half 1: Bottleneck analysis from real data.**
- Analyze the Day 134-138 load test and chaos test results in full — did database CPU max out first, or did the Gateway throttle first, or something else entirely? Write the analysis into `docs/load-test-results.md`: methodology, baseline results, the three chaos experiments' findings, and a clear bottleneck hypothesis backed by evidence, not guesswork.

**Half 2: The formal at-scale exercise — apply the HLD framework to your own platform.**
- Every one of the 16 HLD systems from Weeks 18-19 got a real estimation pass: current QPS, storage, and an explicit "what breaks first at 10x, what breaks at 100x" analysis. The platform itself never got that treatment until now. Do it for real: estimate the platform's current capacity based on your actual load test numbers, then work through, in writing, exactly what would break first at 10x current load (likely candidates: the single Postgres instance, the in-process rate limiter if any module still has one, the Gateway's connection pool) and what would break at 100x (likely candidates: needing to actually shard the database, needing a proper service mesh instead of today's light exposure, needing multi-region deployment). This becomes the single most valuable page in your entire portfolio for a project deep-dive round, because it's evidence you can reason about your own system at scale, not just that you built it.

- Definition of done: `docs/load-test-results.md` complete with real numbers (p95/p99 latency, error rate under load, throughput). `docs/at-scale-analysis.md` complete with the 10x/100x breakdown, each bottleneck backed by a specific reason grounded in the actual architecture, not a generic answer.

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard, and the capstone is genuinely done: Day 140.** `scalable-ecommerce-platform` now runs on real Kubernetes via Helm, with three separate environment configurations, distributed tracing across all four modules, three documented chaos experiments (not one), a light service mesh demonstration, a full CI/CD pipeline that validates the Helm chart itself, a hand-rolled thread-safe queue matching Databricks' actual concurrency bar, real API design notes matching Stripe's actual round, and — new this week — a formal at-scale analysis of the platform's own architecture, the same rigor every HLD system already got. This is a materially different portfolio piece than "it runs and has some tests, and stays on docker-compose forever" — it's a system genuinely built the way your actual production experience says systems should be built, defensible under real questioning, with evidence, at more than one target company's specific bar.

### Daily Deliverable
- [ ] `docs/load-test-results.md` and `docs/at-scale-analysis.md` both complete with real data and defensible reasoning.
- [ ] Weekly ritual and scorecard complete — **capstone phase closed, and this closes the single biggest structural gap from the original audit.**
