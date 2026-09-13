# Week 20 — Consolidated Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Covers:** Days 134–140 (`Week_20_Revised.md`)

---

Every interview question from every Day 134–140 Resource Book, consolidated here for review — 66 questions total. Organized by day, numbered continuously across the week. This is a review document: for the full explanation behind any answer, including worked traces and code, see that day's Resource Book directly.

**A note on this week's shape, worth stating up front:** unlike every DSA-phase week, this bank contains no LeetCode problems — `Week_20_Revised.md` itself has none, matching the pattern already set by the LLD and HLD phases (Weeks 16–19). Every question here is about real infrastructure, operations, and system-hardening work: Kubernetes, Helm, distributed tracing, chaos engineering, a service mesh, a hand-rolled concurrent data structure, CI/CD, and capacity estimation applied to a system actually built rather than only designed.

---

## Day 134 — Load Testing with k6, and the Platform's First Kubernetes Manifests

**1. Why can a service's average latency look completely healthy while a meaningful number of real users are having a bad experience?**

*Answer:* An average collapses the whole distribution into one number. In a worked example with 980 requests at 45ms and 20 at 3,000ms, the average (104ms), the median (45ms), and even p95 (45ms) all look fine — only p99 (3,000ms) reveals that 2% of requests, a real and often large volume of actual users, are waiting three full seconds.

**2. What does a Virtual User (VU) represent in a k6 test, and what does "50 VUs" actually claim?**

*Answer:* A VU is one simulated client executing the test script independently, in a loop, for the test's duration; VUs run concurrently. "50 VUs" is a statement about concurrency, not directly about throughput — actual requests-per-second depends on how long each iteration takes and how much `sleep()` think-time is inserted.

**3. Why include `sleep()` calls in a load test script instead of hitting the endpoint as fast as possible?**

*Answer:* `sleep()` models realistic user think-time between actions. Omitting it produces a valid but different measurement — a deliberate stress/breaking-point test — rather than a realistic "N concurrent users browsing normally" baseline, which is what today's task specifically asks for.

**4. What does a k6 `check()` do differently from a JUnit assertion, and why does that matter for a load test?**

*Answer:* `check()` records a pass/fail result without stopping execution, unlike a test-framework assertion that halts on failure. A load test needs to keep generating load even after some requests fail, or a handful of early errors would end the whole measurement prematurely.

**5. In a Kubernetes Deployment manifest, why must `spec.selector.matchLabels` match `spec.template.metadata.labels` exactly?**

*Answer:* The ReplicaSet controller uses the selector to determine which Pods belong to this Deployment. If the template's labels don't match, the created Pods aren't recognized as belonging to the Deployment that created them, breaking the reconcile loop's ability to track and manage them correctly.

**6. A Service's selector doesn't match any real Pod's labels. What actually happens, and how would you diagnose it?**

*Answer:* Pod creation succeeds independently, but the Service's Endpoints object ends up empty — a silent, non-error condition. Callers see a hang, timeout, or connection refusal with no message pointing at the real cause. Diagnose with `kubectl get endpoints <service>` (empty is the tell), `kubectl describe service`, and `kubectl get pods --show-labels` compared against the selector.

**7. Does a Kubernetes Service "point at" a Deployment directly?**

*Answer:* No. A Service selects Pods by label, entirely independently of whatever created those Pods. The apparent connection between a Deployment and "its" Service exists only because both manifests conventionally use the same label — there's no structural reference between them.

**8. What's the difference between a Service's `port` and `targetPort`?**

*Answer:* `port` is the port the Service itself listens on for other callers; `targetPort` is the port the actual container listens on. They can differ — a Service could expose `80` while the container listens on `8081`.

**9. What's the precise difference between exceeding a CPU limit and exceeding a memory limit in Kubernetes?**

*Answer:* Exceeding a memory limit gets the container killed (OOMKilled) — memory isn't compressible, so the kernel's cgroup OOM killer terminates the process. Exceeding a CPU limit throttles the container instead — CPU time is compressible, so the CFS bandwidth controller just caps how much CPU time it gets per period, making it slower rather than killing it.

**10. What does the scheduler actually use resource `requests` for, versus `limits`?**

*Answer:* `requests` is the bin-packing input the scheduler uses to decide whether a node has room for a Pod, compared against the node's allocatable capacity. `limits` is a runtime-enforced ceiling on actual usage once the Pod is running — two different mechanisms serving two different purposes.

**11. What does `kubectl apply --dry-run=client` actually validate, and what can it miss?**

*Answer:* It validates that the manifest is well-formed and matches the resource's client-side schema — catches typos and structural errors. It does not catch a label-selector mismatch, a non-existent container image, or anything requiring live cluster state or server-side admission control — `--dry-run=server` is needed for that level of validation.

**12. Walk through, end to end, what happens after `kubectl apply` on a Deployment manifest.**

*Answer:* The API server validates and persists the desired state to etcd; the Deployment controller notices via a watch and creates/updates a ReplicaSet; the ReplicaSet controller creates the specified number of Pods; `kube-scheduler` assigns each Pod to a node using its resource requests; the kubelet on that node pulls the image and starts the container; the Service's Endpoints object updates once the Pod is running and matches the selector. This is Day 99's reconcile loop, traced concretely.

---

## Day 135 — Helm Charts, and Deploying the Platform for Real

**13. State the concrete scaling argument for why Helm beats hand-maintained per-environment manifests.**

*Answer:* Raw manifests scale as O(modules × environments) — every new environment multiplies the entire file count, and every shared change has to be found and edited everywhere by hand. A Helm chart scales as O(modules) template files plus O(environments) small values files — one shared template set, parameterized once.

**14. What are the three core pieces of a Helm chart, and what does each one do?**

*Answer:* `Chart.yaml` holds the chart's own metadata (name, chart version, app version); `values.yaml` holds default configuration; `templates/` holds Go-template YAML files that get rendered by substituting `{{ .Values.x }}` placeholders with real values.

**15. What does `helm install` actually do, end to end?**

*Answer:* It reads the chart and values, renders every template into plain final Kubernetes YAML, submits that YAML to the API server via the same calls `kubectl apply` would make, and additionally records the result as a numbered release, enabling later `helm upgrade` and `helm rollback`.

**16. Why would `{{ .Values.resources }}` (bare interpolation) break a template that's inserting a nested YAML block?**

*Answer:* Bare interpolation stringifies the underlying Go map object directly rather than rendering it as YAML. The correct pattern is `{{- toYaml .Values.resources | nindent 12 }}`, which converts the value to proper YAML text and indents it to match the surrounding structure.

**17. What's the difference between `helm install` and `helm upgrade --install`, and why does automation almost always use the latter?**

*Answer:* `helm install` fails if a release with that name already exists. `helm upgrade --install` is idempotent — it installs if the release is absent and upgrades if it's already present — which is what makes it safe to run repeatedly from a CI/CD pipeline without first checking whether a prior deploy exists.

**18. Explain the difference between Kubernetes Ingress and Spring Cloud Gateway, and why the platform needs both.**

*Answer:* Ingress is a cluster-networking object, interpreted by an Ingress Controller, that routes external traffic by host/path to the right internal Service — no business logic. Spring Cloud Gateway is an actual application, doing JWT validation, rate limiting, and business-aware routing across the platform's own modules. Ingress gets traffic to the Gateway; the Gateway decides what happens to it once it's inside the application tier. Neither can cleanly replace the other.

**19. Does an Ingress object do anything by itself, once applied to the cluster?**

*Answer:* No — it does nothing without an Ingress Controller (e.g., NGINX Ingress Controller) already running in the cluster and watching for Ingress objects; the controller is a separate, cluster-level prerequisite.

**20. Why would a locally built Docker image fail to pull inside a Minikube-deployed Pod, even though `docker images` shows it exists?**

*Answer:* Minikube runs its own internal container runtime, separate from the host machine's Docker daemon. The image exists on the host but not inside Minikube's runtime, producing `ImagePullBackOff`. Fixed via `eval $(minikube docker-env)` before building, or `minikube image load` afterward.

**21. What happens if a Helm template references a `values.yaml` key that doesn't exist?**

*Answer:* A missing nested key (dotting further into an already-nil value) fails immediately with a nil-pointer render error. A missing top-level key referenced directly often renders as the literal text `<no value>` inserted into the output instead of failing — worth guarding against with an explicit `default` fallback or `--strict` mode, since a silent placeholder string is arguably worse than a clean failure.

---

## Day 136 — Multi-Environment Configuration: Dev, Staging, Production

**22. If `values.yaml` sets `replicaCount: 2` and `values-prod.yaml` sets `replicaCount: 3` but says nothing about `image.pullPolicy`, what does a prod install actually resolve to for each key?**

*Answer:* `replicaCount` resolves to 3 (explicitly overridden by the prod file); `image.pullPolicy` resolves to whatever `values.yaml` set as the base default, since the prod file never mentions it and values files merge rather than wholesale-replace each other.

**23. Do Helm values files merge the same way for lists as they do for maps?**

*Answer:* No. Maps deep-merge — an unmentioned key falls through to a lower-precedence file. Lists wholesale-replace — a list set in a higher-precedence file completely replaces the corresponding list from a lower-precedence file rather than appending to or merging with it.

**24. Is a Kubernetes Secret encrypted?**

*Answer:* Not by default — its values are base64-encoded, which is trivially reversible with a single `base64 -d` command and no key required. Real protection needs etcd encryption-at-rest configured separately, or an external secret manager (Vault, AWS Secrets Manager, Sealed Secrets) that keeps plaintext out of both git and any base64-reversible object.

**25. What's the direct connection between today's ConfigMap/Secret work and Week 8's Spring Profiles?**

*Answer:* ConfigMaps and Secrets are how environment-specific values physically reach a running container, typically as environment variables. Spring Profiles is how the application itself then decides which of its own config layers to use once those values arrive — two layers of the same "one build artifact, externalized values determine behavior" idea, not two unrelated mechanisms.

**26. Why is baking environment-specific configuration into the image itself worse than externalizing it, beyond just extra build time?**

*Answer:* It breaks the core premise of testing — if staging and production run genuinely different image builds, the artifact validated in staging is not the same bytes shipped to production, so passing tests no longer guarantees anything about what's actually deployed.

**27. If you edit a ConfigMap that's injected into a Pod as an environment variable, does the running Pod pick up the change?**

*Answer:* No. Environment variables are read once, at container startup; a running container has no ongoing connection to the ConfigMap object. The Pod needs an explicit rollout restart (manual, scripted, or via a watcher tool like Reloader) to see the new value.

**28. Does mounting a ConfigMap as a volume instead of an environment variable fully solve the live-update problem?**

*Answer:* Only partially. The kubelet does periodically sync the mounted file's content without a Pod restart, but the application still has to actually notice and re-read that file at runtime — which Spring Boot doesn't do automatically without something like Spring Cloud Config's `@RefreshScope` explicitly wired up.

---

## Day 137 — Distributed Tracing, For Real This Time

**29. What question does distributed tracing answer that aggregate metrics cannot?**

*Answer:* Metrics show that something is slow in aggregate (e.g., p95 latency is 420ms) across a whole population of requests. Tracing shows, for one specific request, the exact path across every service it touched and exactly where the time went — a diagnostic for one instance, not a population-level number.

**30. What's the relationship between a Trace ID and a Span ID?**

*Answer:* A Trace ID is generated once, when a request first enters the system, and is shared across every downstream hop that request causes. A Span ID identifies one specific unit of work within that trace; spans form a parent-child tree, all sharing the same Trace ID.

**31. How does trace context get from one service to the next over a synchronous HTTP call, without application code manually passing an ID through every method?**

*Answer:* Micrometer Tracing instruments the HTTP client and server layers directly, automatically attaching tracing headers (B3 or W3C `traceparent`) to outgoing calls and reading them on incoming requests — instrumentation at the framework boundary, the same mechanism family as Spring AOP and Resilience4j's proxy-based approach.

**32. Does distributed tracing context propagate automatically across a Kafka message the same way it does over HTTP?**

*Answer:* No. Publishing to Kafka isn't an HTTP call, so nothing carries tracing headers along automatically. Propagating trace context across Kafka requires deliberately carrying it as message headers, via Spring Kafka's tracing instrumentation — a genuine, separate integration point, not a free consequence of having tracing set up elsewhere.

**33. Why does this matter specifically for a platform whose Saga is built on Kafka?**

*Answer:* Saga Choreography chains its steps via Kafka events, not synchronous calls. Without deliberate header propagation, a trace started at the Gateway would silently end the moment the flow crosses into an asynchronous Saga step, breaking the single end-to-end trace exactly where the architecture becomes genuinely event-driven.

**34. How does log correlation across services actually work here?**

*Answer:* Micrometer Tracing injects the current Trace ID and Span ID into SLF4J's MDC, a thread-local store a properly configured log pattern includes automatically in every log line — so filtering logs by one Trace ID reconstructs that request's full log output across every service it touched.

**35. What's the actual trade-off behind a tracing sampling rate?**

*Answer:* Tracing every request has a real, volume-proportional overhead and storage cost. Sampling reduces that cost but risks missing the one specific slow or failing request you actually needed visibility into — a real completeness-versus-cost trade-off, not a free setting.

**36. Trace through, with concrete positions, why offset-based pagination can duplicate or skip an item under concurrent inserts.**

*Answer:* If results are sorted newest-first and a new item is inserted while a client is paging, every existing item's rank shifts down by one. A client's second page request (by numeric offset) now lands on ranks that partially overlap the first page (a duplicate) and misses the rank that got pushed past the requested window entirely (a skip) — both silently, with no error raised.

**37. Why is cursor-based pagination immune to that specific failure, and what does it give up in exchange?**

*Answer:* A cursor anchors to a specific item's own sort position rather than a numeric offset, so a new insertion elsewhere doesn't change what "the next items after this one" means. In exchange, it can't support jumping directly to an arbitrary page number the way offset-based pagination can.

**38. Why isn't an HTTP status code alone enough for a client to handle an API error correctly?**

*Answer:* A single status code (e.g., 400) can't distinguish genuinely different situations — malformed JSON, a specific field failing validation, or a well-formed request that violates a business rule — that a calling program needs to branch on differently. A structured error body with a stable machine-checkable code solves this.

---

## Day 138 — Chaos Engineering, Deepened; a Light Service Mesh; a Thread-Safe Queue Built by Hand

**39. What does a chaos experiment actually verify that a unit test of the same resilience mechanism doesn't?**

*Answer:* A unit test verifies the mechanism behaves correctly under artificial, controlled conditions (a mocked failure). A chaos experiment verifies it behaves the same way under a real, induced failure in the actual running system — closing the gap between "configured to work" and "observed to work."

**40. Why doesn't `kubectl delete pod` alone produce a sustained failure for a chaos test?**

*Answer:* The ReplicaSet controller's reconcile loop notices the missing Pod within seconds and creates a replacement automatically. A sustained failure requires removing the desired state itself — scaling the Deployment to zero replicas — not deleting a single Pod instance.

**41. What does a connection pool's `maximum-pool-size` actually bound, and what happens when a request needs a connection and none is free?**

*Answer:* It bounds how many concurrent database operations the module can sustain at once, independent of how many concurrent HTTP requests it might otherwise accept. A request needing a connection when the pool is fully borrowed waits up to `connectionTimeout`; if none frees up in time, it throws a pool-exhausted exception.

**42. Why can connection pool saturation cascade into total module unavailability, even for requests that never touch the database?**

*Answer:* Threads blocked waiting for a database connection are still occupying the module's own request-handling thread pool. If enough pile up, that thread pool itself gets exhausted, leaving no threads available to handle any request — including ones with no database dependency at all.

**43. What's the architectural difference between Resilience4j's circuit breaker and what a service mesh sidecar provides?**

*Answer:* Resilience4j is a library living inside the application's own process, configured in application code. A service mesh sidecar implements similar cross-cutting behavior (retries, timeouts, routing) entirely outside application code, at the network layer, uniformly across services regardless of language or framework.

**44. Why can't a plain Kubernetes Service do a 90/10 weighted traffic split on its own?**

*Answer:* A plain Service load-balances roughly evenly across every Pod matching its selector, with no concept of differential weighting between subsets. Weighted splitting requires a service mesh's own routing resources on top of the base Service/Pod model.

**45. Trace the exact race in a `wait()`/`notify()` bounded queue that uses `if` instead of `while` to guard the wait.**

*Answer:* With capacity 1 and the queue full, two producers can both see the full condition and both call `wait()`. A single `notifyAll()` after one item is consumed wakes both; since the guard was `if`, neither re-checks the condition before adding — both proceed, and the queue ends up with 2 items in a capacity-1 queue.

**46. Why does this bounded queue implementation use two separate `Condition` objects instead of one?**

*Answer:* A blocked `put()` only needs to know "is there space now," and a blocked `take()` only needs to know "is there an item now." A single shared condition would wake every waiter — producers and consumers alike — on every change regardless of relevance; two conditions let each `signal()` target only the waiters that specific change could actually unblock.

**47. Is `signal()` safe to use here instead of `signalAll()`, and why?**

*Answer:* Yes — each successful `put()` creates exactly one new "an item is available" fact, so waking exactly one `notEmpty` waiter is sufficient and correct (the symmetric argument holds for `take()`/`notFull`). `signalAll()` would also be correct, just less efficient, since the `while` guard makes an unnecessarily-woken thread simply re-block.

**48. Why must `lock.unlock()` be called inside a `finally` block for a `ReentrantLock`, when `synchronized` doesn't need the equivalent?**

*Answer:* `synchronized` releases its monitor automatically on any exit path, including an exception. `ReentrantLock` does not — if an exception is thrown inside the critical section and `unlock()` isn't in a `finally`, the lock is held forever, deadlocking every future caller.

**49. In the concurrent test proving the queue's correctness, why gate every thread behind a shared `CountDownLatch` instead of just starting them normally?**

*Answer:* Threads started normally begin sequentially, with real contention left to chance. A shared latch releases every thread simultaneously, forcing genuine overlapping contention for the same lock — exactly the condition under which a race like the broken `if`-based version would actually manifest; without it, a flawed implementation could pass purely by luck.

**50. What does "Big-O of `put()`" mean when the call might block, and why is that a slightly different kind of question than usual?**

*Answer:* The non-blocking fast path is O(1). While blocked, the relevant properties are safety (the invariant is never violated) and liveness (the thread eventually proceeds once the system makes progress) — properties classical Big-O, built for single-threaded algorithms, doesn't directly capture. Naming that distinction is the correct answer, not forcing a number onto a question that isn't really asking for one.

---

## Day 139 — CI/CD Finalization, Health Checks, and Documentation

**51. Why can a test suite have 100% line coverage on a method and still verify nothing meaningful about it?**

*Answer:* Coverage counts lines executed, not assertions made. A test that calls a method without asserting anything about its result achieves full coverage while verifying nothing — including missing a divide-by-zero bug entirely.

**52. What does mutation testing actually measure, and why isn't it the default instead of line coverage?**

*Answer:* It deliberately introduces small bugs into the code and checks whether the existing test suite catches them — a surviving mutant proves the suite doesn't actually verify that logic. It's not the default because it requires re-running the full suite once per mutant, which is expensive and slow at real scale compared to cheap, fast line/branch coverage.

**53. What's the actual difference between generating a JaCoCo report and enforcing a coverage floor?**

*Answer:* The `report` goal only produces a human-readable report — nothing prevents coverage from silently dropping over time. The `check` goal, bound to `verify`, compares actual coverage against a configured minimum and fails the build with a non-zero exit code if it isn't met — only `check` actually gates anything.

**54. Trace exactly what happens in the pipeline when a commit breaks a test.**

*Answer:* `mvn verify` fails with a non-zero exit code; GitHub Actions marks that step and the job as failed; every subsequent step (Docker build, push, Helm validation) is skipped by default, so nothing broken ever reaches the registry or gets deployed.

**55. Why tag the CI-built image with the commit SHA instead of `:latest`?**

*Answer:* `:latest` is non-deterministic — the same tag can point at different actual image contents over time, making it ambiguous which build is actually running. A SHA-based tag is immutable, tying one tag to exactly one set of bytes, which is what "build once, deploy many times" actually requires.

**56. What does a liveness probe failure cause Kubernetes to do, versus a readiness probe failure?**

*Answer:* A liveness failure gets the container killed and restarted. A readiness failure removes the Pod from its Service's Endpoints — traffic stops routing to it — without touching the container itself.

**57. Why is checking a downstream dependency in a liveness probe a well-known anti-pattern?**

*Answer:* If the dependency has a brief outage, every Pod's liveness probe fails simultaneously, causing kubelet to restart all of them at once — which doesn't fix the dependency and makes recovery worse via a synchronized restart storm. Dependency health belongs in the readiness probe, since removing from rotation (not restarting) is the correct response to a temporarily unavailable dependency.

**58. How do readiness probes specifically prevent the "service starts before its dependencies are ready" crash?**

*Answer:* Without one, a Pod is added to its Service's Endpoints the moment the container starts, receiving traffic before it's actually able to serve it. A readiness probe keeps the Pod out of the Endpoints list until a real check passes, so it never receives traffic until it's genuinely ready.

**59. What does piping `helm template` into `kubectl apply --dry-run=client -f -` actually validate that `helm lint` alone doesn't?**

*Answer:* `helm lint` checks the chart's own template syntax and structure. Piping the *rendered* output into `kubectl`'s dry-run validates that what the templates actually produce is valid Kubernetes YAML matching the real resource schema — validating the output, not just the templates that generate it.

---

## Day 140 — Bottleneck Analysis, the Platform's Own At-Scale Exercise, and Capstone Complete

**60. Why is Two-Phase Commit a poor fit for a checkout flow spanning independently-owned Order, Payment, and Product services?**

*Answer:* 2PC's coordinator can leave participants blocked holding locks indefinitely if it crashes between the prepare and commit phases, and it assumes tight coupling to a shared distributed-transaction coordinator that doesn't fit naturally across independently-owned services and databases. Saga avoids both by using local transactions with compensating undo logic instead of cross-service locks.

**61. Why would an in-process rate limiter become actively wrong, not just slower, once a module runs multiple replicas?**

*Answer:* Each replica would enforce the configured limit independently, so the effective global limit becomes (replica count × per-instance limit) rather than the intended single global limit — a silent correctness failure, not a performance degradation, which is exactly why the limit needs to live in shared state (Redis+Lua) rather than in each instance's own memory.

**62. What specifically can Kubernetes and Helm do that docker-compose structurally cannot?**

*Answer:* Kubernetes' reconcile loop provides automatic self-healing and can schedule across multiple nodes rather than one host; Helm's templating eliminates hand-maintained, drift-prone per-environment configuration. Docker-compose was never designed to solve any of the three.

**63. In the platform's own load-test data, what distinguishes a dependency being genuinely down from a dependency merely being saturated?**

*Answer:* A genuinely down dependency produces an immediate, sharp error-rate jump with the circuit breaker's OPEN state visible right away. Saturation produces a slower, quieter latency climb first, with errors only appearing once queuing has been happening for some time — the same distinction Day 138 named between a clear failure and a cascading resource exhaustion.

**64. Why is the single Postgres instance the most likely bottleneck at 10x load specifically, rather than any of the application modules?**

*Answer:* Every application module scales horizontally via Kubernetes replicas, but the data layer was never made to. The database is the one tier in the whole architecture that hasn't been given a path to scale alongside everything above it.

**65. What's the difference between what would break at 10x versus what would break at 100x for this platform?**

*Answer:* At 10x, the likely bottlenecks are things that can be fixed by adding capacity to an existing design — the single database instance, a stray in-process limiter, the Gateway's connection pool. At 100x, the likely bottlenecks require actually changing the architecture — sharding the database, hardening the service mesh from a demo into something load-bearing, and distributing across multiple regions.

**66. Why is a real at-scale analysis of your own built system a stronger portfolio artifact than the same analysis applied to a system only ever designed on a whiteboard?**

*Answer:* It's backed by real, measured load-test data and a real, built architecture rather than a hypothetical one — an interviewer probing "what breaks first" against real evidence is asking a fundamentally harder question to fake an answer to than the same question about a system that only ever existed as a design.

---

---

**Total: 66 questions across 7 days.** Combined with Weeks 1–19's banks (`Week1_Interview_Questions.md` through `Week19_Interview_Questions.md`), the full archive now stands at 20 consolidated weekly interview banks.