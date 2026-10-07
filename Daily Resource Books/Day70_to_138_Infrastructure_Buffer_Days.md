# Gap Fill: Day 70 to 138 — Infrastructure Buffer Days

**Gap addressed (#6):** nearly every Project Block from Week 9 onward depends on `scalable-ecommerce-platform` compiling and running from wherever the previous day left it. Kafka, Zookeeper, Kubernetes, Helm, and Istio running locally are all notoriously fiddly — a genuinely common way a day gets lost isn't a gap in understanding, it's an afternoon burned on a broken Minikube cluster. There's no buffer built in anywhere for this, and it's a real single point of failure for the whole plan.

**How this slots in:** a de-risking philosophy (below) plus three standalone buffer days, inserted after the three highest-risk infrastructure clusters in the plan. Each is a genuinely optional insurance day — if the preceding stretch went cleanly, skip it or use it as free time. None of the existing Week_XX_Revised.md files are touched.

---

## De-risking philosophy (apply this throughout, not just on the three buffer days below)

**Time-box troubleshooting.** If a specific infra step isn't working after 45–60 minutes of genuine troubleshooting, stop. Log exactly where it broke, fall back to the last known-good state, and move on to something else that day rather than losing the whole day to one broken container. Coming back to it fresh — or on a dedicated buffer day — is almost always faster than pushing through frustration in the moment.

**Keep a fallback path alive.** As the platform grows into Kubernetes/Helm territory (Week 20), don't let the `docker-compose.yml` path from Week 7 quietly rot. Keep it working alongside the K8s path for as long as practical. If a K8s-specific day breaks, the Docker Compose version still runs, which means a broken K8s afternoon blocks *that day's specific new feature*, not the entire platform.

**Snapshot before risky changes.** Before installing a service mesh (Istio/Linkerd) or making any change likely to leave the cluster in a half-broken state, note the exact working commit hash and, if using Minikube, consider `minikube stop` + a filesystem-level backup of the `.minikube` directory, or at minimum make sure every manifest is committed to Git *before* trying something new — so "revert" is a real, fast option rather than a rebuild from memory.

**Distinguish "broken" from "still loading."** Especially on an M1 Air, a service that hasn't started yet often looks identical to one that's actually crashed. Check `docker logs` / `kubectl logs` / `kubectl describe pod` before assuming failure — a surprising amount of "why isn't this working" time is actually just impatience with legitimately slow cold starts on a resource-constrained laptop.

---

## Buffer Day 1 (Day 70A) — Post Week 10

**Placement:** after Day 70 (Week 10 close — Resilience4j, Feign, the Gateway, JWT, rate limiting, Kafka Schema Registry, and Saga choreography between two real modules all land in this one week), before Week 11 begins.

**Why this stretch specifically:** five distinct pieces of new infrastructure in five days, several of which depend on each other working correctly at the same time (the Saga flow specifically needs the Gateway, JWT, *and* the Kafka event chain all functioning together).

**Common failure modes to check for:**
- Kafka or Zookeeper container failing to start — usually a port conflict with something else already bound to 9092/2181, or Docker Desktop's memory allocation set too low for both containers plus the four Spring Boot modules to run simultaneously.
- Schema Registry unable to reach the Kafka broker — almost always a hostname mismatch between `application.yml`'s configured broker address and the actual service name inside `docker-compose.yml`.
- Feign client connection-refused errors between modules — check the target module is actually listening on the expected port *inside* the Docker network, not just reachable from the host machine.
- Circuit breaker not tripping when expected — usually a threshold or sliding-window-size misconfiguration in the `@CircuitBreaker` annotation's settings rather than a code bug.

**Use of the day:** first, catch up on anything from Days 64–70 that didn't fully work using the checklist above. If already caught up, this becomes free time, or an extra pass at the Theory Spaced Repetition rotation (`Day_42_to_140_Theory_Spaced_Repetition_Track.md`).

---

## Buffer Day 2 (Day 98A) — Post Week 14

**Placement:** after Day 98 (Week 14 close — Eureka service discovery, Kubernetes ConfigMaps/Secrets, and StatefulSets/DaemonSets concepts all land this week), before Week 15 begins.

**Common failure modes to check for:**
- Services not registering with Eureka — usually a wrong or missing `eureka.client.service-url.defaultZone` in one module's config, or a module starting before the Eureka server itself is ready (a startup-ordering issue, not a config-correctness one).
- Confusing base64 encoding with real encryption — a Secret that "looks wrong" when decoded is often working exactly as designed; base64 is an encoding, not encryption, and isn't meant to look secure on inspection.
- If StatefulSets are actually attempted locally (rather than just discussed, as the original Day 97 content frames it) — pod DNS naming for StatefulSets follows a different, more rigid pattern than Deployments; a "can't resolve hostname" error is often just this rather than a real networking failure.

**Use of the day:** same structure as Buffer Day 1 — catch up first, use remaining time for spaced-repetition review or LinkedIn/networking catch-up if nothing needs fixing.

---

## Buffer Day 3 (Day 138A) — Post Week 20's Chaos Day

**Placement:** after Day 138 (Chaos Engineering + Service Mesh — three separate chaos experiments, Istio/Linkerd sidecar injection, and a hand-rolled thread-safe bounded queue all land on a *single day*), before Day 139 (CI/CD Finalization).

**Why this is the single highest-risk day in the entire plan:** Day 138 combines the most resource-intensive tooling in the whole curriculum (a service mesh) with the most inherently destructive activity (deliberately killing pods and injecting network chaos) on a MacBook Air M1 that's also running the full four-module platform, Minikube, and whatever chaos-injection tooling (Toxiproxy/Pumba) is chosen. This is exactly the kind of day the gap this file addresses was written about.

**Common failure modes to check for:**
- Istio/Linkerd sidecar injection failing silently — almost always a missing namespace label (`istio-injection=enabled` or the Linkerd equivalent) rather than an actual mesh installation problem.
- Minikube resource exhaustion — a full platform plus a service mesh plus chaos tooling is genuinely heavy for an M1 Air's allocated resources; if pods are stuck `Pending` or getting OOM-killed, check `minikube config` resource limits before assuming a code or config bug.
- Toxiproxy/Pumba unable to find the target container — container names inside a Kubernetes cluster differ from their `docker-compose` equivalents; a script copied from Compose-era chaos testing needs updating, not just re-running.
- A chaos experiment "succeeding" in a way that's actually just everything crashing uninformatively — the point of the exercise is graceful degradation with a *documented, expected* fallback behavior, not merely "the system went down and I have a screenshot." If the failure mode doesn't match what Resilience4j was actually configured to do, that's a real finding worth writing into `docs/chaos-test-results.md`, not a result to paper over.

**Hardware-specific fallback, worth deciding in advance rather than mid-struggle:** if Istio proves too heavy for the M1 Air's available resources even after a genuine troubleshooting pass, Linkerd is meaningfully lighter and demonstrates the same core concept (sidecar injection, one traffic-split demo). If even that struggles, a stripped-down two-service mesh demo (rather than injecting sidecars across the full four-module platform) still satisfies the spirit of Day 138's "genuine exposure, not exhaustive mastery" framing — this is explicitly a differentiator-add, not a must-have at full scale, and a working small demo beats a broken full-scale attempt.

**Use of the day:** given this is the highest-risk day in the plan, treat this buffer as more likely to be genuinely needed than the other two. If Day 138 went cleanly, this becomes a good day for the professional mock-interview checkpoint 2 described in `Day_119_to_148_Professional_Mock_Calibration.md`, since it sits right at the HLD-to-Capstone boundary.

---

## Cumulative note

These three buffer days are insurance, not mandatory extra content — each one's own instructions say to skip straight to something else if nothing needs fixing. If all three end up genuinely unneeded, that's a good sign, not a wasted file.
