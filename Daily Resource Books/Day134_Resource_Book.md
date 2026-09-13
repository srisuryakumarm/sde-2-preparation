# Day 134 — Load Testing with k6, and the Platform's First Kubernetes Manifests

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 133 Resource Book](Day133_Resource_Book.md)
**Next ▶:** [Day 135 Resource Book](Day135_Resource_Book.md)
**Companion to:** Day 134 of `Week_20_Revised.md`

---

> ⚠️ **Flag, not a silent fix — a claim worth checking before it gets repeated to an interviewer.** This week's own plan opens by saying `todo-api` and "the original plan's `order-management-api`" both "got real Kubernetes deployments" earlier in the series. Checked directly against `00_Curriculum_Map.md`: `todo-api` is documented as Docker- and Docker-Compose-managed only (Week 7, Days 43–44), explicitly frozen and feature-complete from Day 62 onward, with no Kubernetes deployment logged anywhere in its history. `order-management-api` never existed as a separate repository at all — per `Week_21_Revised.md`'s own later note, it was absorbed directly into `scalable-ecommerce-platform` back in Week 9. Neither project's documented history supports "both got real Kubernetes deployments." This doesn't change anything about today's actual work — the platform's first manifests get written either way — so it's flagged here and in the curriculum map rather than silently repeated or silently corrected, the same treatment this map has given every prior instance of a plan's framing outrunning its own documented history.

---

## Recap

Yesterday (Day 133) closed the System Design phase entirely — sixteen HLD systems across Weeks 18–19, five mocks, every deferred thread resolved. Today opens something structurally different from every week since Week 16: not a new system to *design*, but the one system you've actually been *building* since Week 9 — `scalable-ecommerce-platform`, four modules (Product, Order, Payment, Notification) plus a Gateway — getting run the way it would actually run in production.

Three things get reused hard today, all already fully taught:

- **Docker** (Week 7, Day 43) and **Docker Compose** (Week 7, Day 44) — `todo-api`'s multi-stage `Dockerfile` and service-name-based container networking are the direct baseline. The platform's own modules are assumed to already build into Docker images the same way, since nothing between Week 9 and now changed that.
- **Kubernetes, from zero** (Week 15, Day 99) — Pod → ReplicaSet → Deployment → Service, and the **declarative reconcile loop**: you declare desired state, a controller continuously closes the gap to it, forever. That's a citation, not a re-teach. Today is the first day you actually *write* the YAML for a real, multi-module system instead of reasoning about the model in the abstract.
- **`scalable-ecommerce-platform`'s four-module skeleton** (Week 9, Day 62) — the parent/child Maven POM structure, and which module owns which responsibility. Today's manifests mirror that structure one-to-one: one Deployment and one Service per module.

---

## Learning Objectives

By the end of today, without notes:

1. Explain why average latency can look fine while real users suffer, and compute p50/p95/p99 from a raw list of latencies by hand.
2. Write and run a k6 load test script against a real HTTP endpoint, and correctly interpret every line of its summary output.
3. Write a valid Kubernetes Deployment and Service manifest from scratch for a Spring Boot module — correctly wiring labels and selectors — and explain, concretely, what breaks if they don't match.
4. State the precise difference between what a resource **request** does and what a resource **limit** does, including why exceeding one gets you throttled and exceeding the other gets you killed.
5. Explain exactly what `kubectl apply --dry-run=client` validates, and what it does *not* catch.

---

## Concept Dependency Map

```
Week 7, Days 43-44: Docker + Docker Compose (todo-api)
Week 9, Day 62: scalable-ecommerce-platform's 4-module skeleton
Week 15, Day 99: Kubernetes from zero — Pod → ReplicaSet → Deployment → Service,
                 the reconcile loop, HPA (averageUtilization vs. Pod CPU REQUEST)
Week 11, Days 76-77: Micrometer → /actuator/prometheus → Prometheus → Grafana, live
        │
        ▼
Day 134 — two independent new threads, same day
        │
        ├── Thread A: Load Testing Fundamentals
        │     ├─ VUs, throughput
        │     ├─ p50 / p95 / p99 — why the tail matters, proven on real numbers
        │     ├─ k6 script anatomy (options, default function, check, sleep)
        │     └─ reading a k6 summary report
        │
        └── Thread B: Real Kubernetes manifests for the platform
              ├─ Deployment: replicas, selector, template labels, containers,
              │   resources.requests / resources.limits
              ├─ Service: ClusterIP, port vs. targetPort, label-based discovery
              │   (a live instance of Day 99's reconcile loop, not a static lookup)
              └─ kubectl apply --dry-run=client — what it validates, and what it can't
        │
        ▼
Day 135: Helm templates exactly what gets hand-written today
Day 140: today's k6 numbers become the real data behind the at-scale analysis
```

---

# Part 1 — Load Testing Fundamentals

## Why "it's fast on average" is a claim that can hide real pain

**The core idea:** a single average collapses a whole distribution of user experiences into one number, and it's mathematically possible — common, even — for that number to look completely healthy while a meaningful slice of real requests are slow enough to matter.

**Worked example, not asserted:** suppose a service handles 1,000 requests. 980 of them (98%) complete in 45ms. The remaining 20 (2%) — hitting some slow path, a cold cache, a GC pause, a lock wait — take 3,000ms.

| Metric | Value | How it's computed |
|---|---|---|
| Average | 104.1ms | (980×45 + 20×3000) / 1000 = (44,100 + 60,000) / 1000 |
| p50 (median) | 45ms | The 500th value in the sorted list — comfortably inside the 980 fast requests |
| p95 | 45ms | The 950th value — *still* inside the fast group, since only the last 20 of 1,000 are slow |
| p99 | 3,000ms | The 990th value — now inside the slow group, since positions 981–1,000 are the slow 20 |

*(Percentile here uses the common "nearest-rank" convention: the Nth percentile of k sorted values is the value at position `ceil(N/100 × k)`.)*

**The point, stated precisely:** both the **average** (104ms) and **p50** (45ms) — the two numbers most likely to get glanced at on a dashboard — look fine. Even **p95** looks fine. Only **p99** reveals that a full 2% of requests — which, at real traffic volumes, is not a rounding error, it's thousands of actual users per hour — are waiting three full seconds. This is exactly the "p50 of 50ms can hide a p99 of 3 seconds" claim from today's plan, now proven on concrete numbers rather than taken on faith.

**🔑 Key Takeaway:** always ask which percentile a number is before trusting it. "Fast on average" and "fast for everyone" are different claims, and only the tail (p95, p99, sometimes p99.9 at real scale) distinguishes them.

**⚠️ Common Mistake:** treating throughput and latency as the same axis. Throughput (requests completed per second) measures *volume*; latency (time per request) measures *experience*. A system can raise throughput by adding parallelism while individual latency stays flat, gets worse, or even improves (up to the point resources saturate) — the two have to be reasoned about together, not collapsed into "performance."

## Virtual Users (VUs)

**Definition:** a Virtual User is one simulated client, executing the load test script's logic in a loop, independently of every other VU, for the test's configured duration. k6 runs VUs concurrently (as lightweight goroutines under the hood) — 50 VUs means 50 independent, simultaneous request streams, not one stream repeated 50 times sequentially.

**Why this matters for what a number like "50 VUs" actually claims:** it's a statement about *concurrency*, not directly about throughput. How many total requests 50 VUs generate in 30 seconds depends on how long each iteration actually takes (a fast endpoint means each VU completes more loop iterations in the same 30 seconds) and how much artificial `sleep()` think-time the script inserts between actions.

## k6, Concretely

**Install:** `brew install k6` (macOS) or via the official apt repository (Linux), or run it containerized (`docker run grafana/k6 run ...`) with no local install at all.

**Anatomy of a script:**

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 50,
  duration: '30s',
  thresholds: {
    http_req_duration: ['p(95)<500'],   // fail the test if p95 latency exceeds 500ms
    http_req_failed: ['rate<0.01'],      // fail the test if error rate exceeds 1%
  },
};

export default function () {
  const res = http.get('http://localhost:8081/api/products');
  check(res, {
    'status is 200': (r) => r.status === 200,
  });
  sleep(1);
}
```

- **`export const options`** — configures the load shape. `vus`/`duration` here describe a flat load (constant concurrency for a fixed time); k6 also supports `stages` (e.g., ramp from 0→50 VUs over 10s, hold, ramp down) for a more realistic traffic-growth shape instead of an instant step function — worth knowing this exists as the more realistic option, marked here as **extension material**: today's task asks for a flat 50-VU/30s run specifically, and a flat run is the right choice for a first baseline measurement, since it isolates "what does steady-state concurrency look like" from "how does the system behave while load is actively changing" — two genuinely different questions.
- **`export default function`** — the body every VU executes, once per iteration, in a loop, for the test's duration.
- **`check()`** — records a named, pass/fail assertion (visible in the summary's checks section) **without stopping execution** on failure, unlike a test-framework assertion (JUnit's `assertEquals`, say) that halts the current test. A load test needs to keep generating load even after individual requests fail, or a handful of early errors would silently end the whole measurement.
- **`sleep(1)`** — simulates think-time between a real user's actions. Omitting it doesn't break anything, but it changes what the test actually measures: no `sleep` means every VU hammers the endpoint back-to-back as fast as it can respond, which is a valid *stress test* shape (find the breaking point) but not a realistic "50 real concurrent users browsing a catalog" shape (today's actual goal) — which is which is a modeling decision worth being able to state out loud, not an accident of whether you remembered the line.
- **`thresholds`** — lets k6 itself declare pass/fail against a specific percentile target. This is the direct practical payoff of the percentile theory above: `p(95)<500` is a machine-checkable version of "the tail, not the average, is the thing we actually care about."

**Running it:** `k6 run load-test.js`

**Reading the summary — a representative excerpt, annotated:**

```
     ✓ status is 200

     checks.........................: 100.00% ✓ 1487      ✗ 0
     data_received..................: 2.1 MB  70 kB/s
     http_req_duration..............: avg=32.14ms min=8.02ms med=28.91ms max=412.6ms p(90)=48.3ms p(95)=61.7ms
     http_req_failed.................: 0.00%   ✓ 0         ✗ 1487
     http_reqs.......................: 1487    49.5/s
     iterations......................: 1487    49.5/s
     vus..............................: 50      min=50      max=50
```

- **`checks`** — pass/fail count for every `check()` call across every VU/iteration; `100.00%` here means every single response actually returned 200.
- **`http_req_duration`** — the latency distribution, with `avg`/`med` (p50) alongside `p(90)`/`p(95)` explicitly broken out — exactly the numbers Part 1's worked example just walked through by hand, now produced by a real tool.
- **`http_req_failed`** — the error rate, tracked completely separately from latency. A service can have excellent latency and a terrible error rate, or the reverse — they're independent failure axes, worth stating explicitly rather than conflating "it's slow" and "it's broken."
- **`http_reqs`** — total completed requests and the realized rate (req/s) — the actual measured throughput, which may differ from a naive `VUs / sleep-interval` estimate once real network and processing time are accounted for.
- **`vus`** — confirms the concurrency level actually sustained throughout the run.

## Exercise: `load-test.js` Against the Product Module's Read Endpoints

**The goal:** 50 VUs, constantly hitting Product's read endpoints, for 30 seconds — a baseline measurement of the platform as it exists *before* any of this week's chaos or scaling changes, so later comparisons (Day 138's chaos runs, Day 140's bottleneck analysis) have something concrete to compare against.

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 50,
  duration: '30s',
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    http_req_failed: ['rate<0.01'],
  },
};

const BASE_URL = 'http://localhost:8081'; // Product module, via the Gateway in later runs

export default function () {
  const listRes = http.get(`${BASE_URL}/api/products`);
  check(listRes, {
    'list status is 200': (r) => r.status === 200,
  });

  sleep(1);

  const detailRes = http.get(`${BASE_URL}/api/products/1`);
  check(detailRes, {
    'detail status is 200': (r) => r.status === 200,
  });

  sleep(1);
}
```

Two endpoints per iteration — a list call and a detail call — is a deliberately more realistic shape than hitting one single endpoint in a loop, since it mirrors an actual user browsing (see the list, then open one item) rather than an artificial single-URL hammer.

**⚠️ Common Mistakes, as a checklist:**

- **No `sleep()` at all**, when the goal is a "how does normal traffic feel" baseline rather than a deliberate stress test — produces a number, but not the number the task is actually asking for.
- **Running the test from the same machine the service runs on**, letting the load generator itself compete for CPU with the thing being measured — a real, common source of misleading local results; worth flagging even though today's Minikube-adjacent setup makes it hard to fully avoid.
- **Treating today's numbers as universally valid** once Day 136 introduces environment-specific sizing — a load test result is only meaningful *for the configuration it was run against*; a dev-sized deployment's numbers say nothing reliable about production capacity. This gets revisited directly on Day 140.
- **Instant flat concurrency (`vus: 50` from second zero)** when modeling gradual real-world traffic growth — not wrong for a baseline, but worth being able to name `stages`-based ramping as the more realistic alternative on request.

**💡 Interview Insight:** "How would you load test a service?" and "what's the difference between p50, p95, and p99, and why would you care?" are both extremely common questions once a resume lists real production experience. The strongest answer to the first names a *specific tool and shape* (VUs, duration, realistic think-time) rather than staying abstract; the strongest answer to the second is exactly today's worked example — a concrete case where the average and the tail tell different stories — rather than a definition recited from memory. A natural, unprompted follow-up: "if p99 spikes but p50 doesn't move, where do you look first?" — the honest answer is a specific *subset* of requests is hitting a slow path (cache miss, GC pause, lock contention, a specific downstream dependency) that the majority of traffic never touches, which is exactly the kind of thing Day 137's distributed tracing exists to actually locate, rather than guess at.

---

# Part 2 — Kubernetes Deployment and Service Manifests, For Real

## Prerequisites Confirmed

- Pod → ReplicaSet → Deployment → Service and the reconcile loop (Week 15, Day 99) — theory fully covered; today is the first time actually **writing** manifest YAML for this platform, not new conceptual material.
- Each of the platform's four modules is assumed to already build into a Docker image, following the same multi-stage `Dockerfile` pattern taught for `todo-api` (Week 7, Day 43) — today's task is about deploying those images to Kubernetes, not building them from scratch again.

## Anatomy of a Deployment Manifest

```yaml
# k8s/product-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
  namespace: ecommerce
  labels:
    app: product-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: product-service
  template:
    metadata:
      labels:
        app: product-service
    spec:
      containers:
        - name: product-service
          image: product-service:latest
          ports:
            - containerPort: 8081
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

- **`apiVersion: apps/v1`, `kind: Deployment`** — Deployment has lived at the stable `apps/v1` API group since Kubernetes 1.9; there's no legacy alternate version to worry about confusing this with.
- **`metadata.namespace: ecommerce`** — an explicit namespace, not the implicit `default` every prior exercise (and most tutorials) quietly relies on. Namespaces are Kubernetes' own scoping boundary for names, RBAC, and resource quotas; naming one explicitly here is a small, real, correct practice that costs nothing today and avoids collisions later when more than one thing might live in the same cluster.
- **`spec.replicas: 2`** — a deliberate, modest starting point. Day 136 makes this exact value environment-specific (1 for dev, 2 for staging, 3+ for prod); today's raw manifest picks one reasonable number to get something running before that variation is introduced.
- **`spec.selector.matchLabels` and `spec.template.metadata.labels` must match, exactly.** This is not a stylistic convention — it's how the ReplicaSet controller (Day 99) knows which Pods belong to *this* Deployment. See the worked failure trace below for exactly what goes wrong if they don't.
- **`resources.requests` vs. `resources.limits`** — covered in full below; not the same mechanism, and conflating them is a real, common mistake.

The other three modules — Order, Payment, Notification — follow this identical shape: same structure, different `name`, `image`, and `containerPort` (Order on 8082, Payment on 8083, Notification on 8084, by convention for today).

## Anatomy of a Service Manifest

```yaml
# k8s/product-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: product-service
  namespace: ecommerce
spec:
  selector:
    app: product-service
  ports:
    - port: 8081
      targetPort: 8081
  type: ClusterIP
```

- **`spec.selector` matches the Deployment's `template.metadata.labels` — not the Deployment's own name.** This is worth stating precisely because it's easy to assume a Service "points at" a Deployment directly; it doesn't. A Service selects **Pods**, by label, completely independently of whatever created those Pods. The apparent link between a Deployment and "its" Service is entirely a product of both manifests using the same label — convention, not a structural reference.
- **`port` vs. `targetPort`** — `port` is the port the *Service itself* listens on (what other Pods use when calling this Service by name); `targetPort` is the port the *container* actually listens on. They can differ (a Service could expose `80` while the container listens on `8081`); today's manifests keep them equal for simplicity, but the distinction is worth being able to state — it's a one-line difference a real system uses constantly (a Service fronting a container on a non-standard port with a conventional external port).
- **`type: ClusterIP`** (the default if `type` is omitted) — internal-only, reachable from inside the cluster by other Pods, not from outside it. `NodePort` and `LoadBalancer` expose a Service externally; neither is needed yet for internal modules, and the Gateway's own external exposure is Day 135's Ingress, not something handled at the Service level today.

**How a Service actually finds its Pods — the reconcile loop, again, in a new instance:** a Service does not hold a static list of Pod IPs. The control plane continuously watches which currently-Running, currently-Ready Pods match the Service's selector, and keeps an Endpoints (or EndpointSlice) object up to date with that live set; `kube-proxy` on every node uses that continuously-updated object to route traffic. This is exactly Day 99's "declare desired state, a controller continuously closes the gap to it, forever" pattern, applied here to service discovery specifically rather than to replica count — worth naming explicitly as the same mechanism rather than a new one.

### Worked Failure Trace: A Label/Selector Mismatch

Suppose the Deployment's Pod template is labeled `app: product-service` (as written above), but the Service was written with a typo — `selector: { app: product }`, missing `-service`.

**What actually happens, step by step:**

1. The Deployment still creates its Pods successfully — Pod creation doesn't consult the Service at all, so nothing here fails loudly.
2. The Service's selector (`app: product`) matches **zero** currently-running Pods, since every real Pod is labeled `app: product-service`.
3. The Service's Endpoints object is empty. There is no error state for this — an empty-selector-match is a completely valid, silent condition from the API server's point of view.
4. Any request routed to the Service hangs, times out, or gets connection-refused, depending on exactly what's in front of it — and critically, **the error message doesn't point at the actual cause.** A caller sees a timeout or a refused connection; nothing says "your selector doesn't match anything."

**How you'd actually debug this, concretely:**

```bash
kubectl get endpoints product-service -n ecommerce   # shows an empty ENDPOINTS column — the tell
kubectl describe service product-service -n ecommerce # confirms the selector as written
kubectl get pods -n ecommerce --show-labels           # compare actual Pod labels against the selector above
```

An empty `ENDPOINTS` column on an otherwise-healthy-looking Service is one of the single most common real Kubernetes debugging signatures — worth having this exact three-command sequence ready, not just the abstract fact that "labels have to match."

## Resource Requests and Limits, Precisely

**`requests`** is what the **scheduler** uses to decide whether a node has room for this Pod — it's a bin-packing input, compared against each node's allocatable capacity, not a ceiling on what the container can actually use once running.

**`limits`** is an enforced **ceiling**, and the two resource types are enforced by genuinely different mechanisms with genuinely different consequences:

- **Exceed the memory limit → the container is killed.** Memory is not a compressible resource — there's no way to "slow down" memory usage the way you can throttle CPU, so the kernel's cgroup OOM killer terminates the process outright. `kubectl get pods` shows the Pod's status as `OOMKilled`, and it's restarted per the Deployment's restart policy.
- **Exceed the CPU limit → the container is throttled, not killed.** CPU time is compressible — the kernel's CFS bandwidth controller simply caps how much CPU time the container gets within each scheduling period, making it run slower, not terminating it.

**⚠️ Common Mistake:** treating "hit a limit" as one uniform failure mode. It isn't — CPU and memory limits fail in structurally different ways, and confusing the two ("it got OOMKilled because it used too much CPU," or expecting a memory overage to just "run slower") is a real, checkable misconception, not a rounding error in terminology.

**🔗 Forward Reference (Day 99):** this is exactly the mechanism Day 99's HPA note already flagged — `averageUtilization` is measured against a Pod's CPU **request**, not against node capacity. The `requests.cpu: "250m"` value written today is precisely the number a future HPA (if one were added to this platform) would compute utilization against.

**⚠️ Common Mistake, the other direction:** setting requests unrealistically high "to be safe." This doesn't make the Pod safer — it makes bin-packing worse (fewer Pods fit per node, since the scheduler reserves the full requested amount whether or not it's actually used), and taken far enough, a request that exceeds *any single node's* allocatable capacity makes the Pod permanently unschedulable. Setting requests too low has the opposite failure mode: the scheduler over-packs a node believing there's room, and real contention shows up under load — which is precisely the kind of artifact Day 140's bottleneck analysis has to be able to distinguish from a genuine application-level bottleneck.

## `kubectl apply --dry-run=client`, Precisely

**What it validates:** the manifest is well-formed YAML that matches the resource's client-side OpenAPI schema — catches a misspelled field name, wrong indentation, or a value of the wrong type.

**What it does *not* validate:**
- Whether the label selector actually matches anything real (that's a semantic property of live cluster state, not a schema property).
- Whether the referenced container image actually exists or will successfully pull.
- Anything that depends on server-side admission controllers or existing objects in the cluster.

`--dry-run=server` goes further — it sends the request to the real API server for full validation (including admission control) without persisting the object — worth knowing exists as the more thorough option, though today's task only asks for the client-side check.

## Common Mistakes — Full Checklist for Today

- Selector/template label mismatch (traced fully above).
- Confusing a Service's `port` (the Service's own listening port) with `targetPort` (the container's actual port).
- Treating `limits` and `requests` as interchangeable when reasoning about scheduling versus runtime enforcement — they're two different mechanisms, not two names for one idea.
- Forgetting that `imagePullPolicy` defaults to `Always` specifically when the image tag is `latest` or omitted — for any other explicit tag, the default is `IfNotPresent`. The platform's manifests use `:latest` today (revisited with real, immutable tags once Day 139's CI/CD pipeline exists), so every `kubectl apply` right now will always attempt a fresh pull, whether or not the underlying image actually changed.
- Believing `--dry-run=client` passing means the manifest will actually work once applied for real.

**💡 Interview Insight:** "Walk me through what happens when you run `kubectl apply` on a Deployment" is a standard real question once Kubernetes is on a resume, and the strongest answer traces the entire reconcile loop end to end using today's own artifacts: the API server validates and persists the desired state to etcd → the Deployment controller notices via a watch → it creates or updates a ReplicaSet → the ReplicaSet controller creates the specified number of Pod objects → `kube-scheduler` assigns each Pod to a node using its resource **requests** → the kubelet on that node pulls the image and starts the container → the Service's Endpoints object updates to include the new Pod. That's Day 99's reconcile loop, narrated concretely against real manifests instead of described abstractly — exactly the kind of unprompted synthesis that distinguishes "I know the definition" from "I've actually operated this."

---

# Project Block Guide (3.5 hrs)

**Repository:** `scalable-ecommerce-platform`.

**Part 1 — `load-test.js`:** install k6; write the script above (or your own equivalent following the same shape) against the Product module's read endpoints; run it locally for a genuine 30-second, 50-VU baseline before any of this week's later changes land, so Day 140 has a true "before" number to compare against.

**Part 2 — `k8s/` manifests:** for each of the four modules, write `k8s/<module>-deployment.yaml` and `k8s/<module>-service.yaml` following today's annotated templates — same shape, different name/image/port per module. Validate every file:

```bash
kubectl apply --dry-run=client -f k8s/
```

**Definition of done:** the k6 script runs to completion and produces a summary showing request rate and p95 latency (save this output — it's Day 140's baseline data). Every manifest in `k8s/` passes `--dry-run=client` validation and is pushed.

---

# Career Block Guide (1 hr)

**LinkedIn:** 20 minutes of genuine engagement — commenting on 3–5 posts, not just scrolling past them.

**Networking:** keep tracking responses across all 7 target companies (Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech) — applications have been live since Day 118, so first technical screens are a reasonable thing to expect to start seeing scheduled around now.

---

# Day 134 — Interview Questions

**Q1. Why can a service's average latency look completely healthy while a meaningful number of real users are having a bad experience?**
*Answer:* An average collapses the whole distribution into one number. In a worked example with 980 requests at 45ms and 20 at 3,000ms, the average (104ms), the median (45ms), and even p95 (45ms) all look fine — only p99 (3,000ms) reveals that 2% of requests, a real and often large volume of actual users, are waiting three full seconds.

**Q2. What does a Virtual User (VU) represent in a k6 test, and what does "50 VUs" actually claim?**
*Answer:* A VU is one simulated client executing the test script independently, in a loop, for the test's duration; VUs run concurrently. "50 VUs" is a statement about concurrency, not directly about throughput — actual requests-per-second depends on how long each iteration takes and how much `sleep()` think-time is inserted.

**Q3. Why include `sleep()` calls in a load test script instead of hitting the endpoint as fast as possible?**
*Answer:* `sleep()` models realistic user think-time between actions. Omitting it produces a valid but different measurement — a deliberate stress/breaking-point test — rather than a realistic "N concurrent users browsing normally" baseline, which is what today's task specifically asks for.

**Q4. What does a k6 `check()` do differently from a JUnit assertion, and why does that matter for a load test?**
*Answer:* `check()` records a pass/fail result without stopping execution, unlike a test-framework assertion that halts on failure. A load test needs to keep generating load even after some requests fail, or a handful of early errors would end the whole measurement prematurely.

**Q5. In a Kubernetes Deployment manifest, why must `spec.selector.matchLabels` match `spec.template.metadata.labels` exactly?**
*Answer:* The ReplicaSet controller uses the selector to determine which Pods belong to this Deployment. If the template's labels don't match, the created Pods aren't recognized as belonging to the Deployment that created them, breaking the reconcile loop's ability to track and manage them correctly.

**Q6. A Service's selector doesn't match any real Pod's labels. What actually happens, and how would you diagnose it?**
*Answer:* Pod creation succeeds independently, but the Service's Endpoints object ends up empty — a silent, non-error condition. Callers see a hang, timeout, or connection refusal with no message pointing at the real cause. Diagnose with `kubectl get endpoints <service>` (empty is the tell), `kubectl describe service`, and `kubectl get pods --show-labels` compared against the selector.

**Q7. Does a Kubernetes Service "point at" a Deployment directly?**
*Answer:* No. A Service selects Pods by label, entirely independently of whatever created those Pods. The apparent connection between a Deployment and "its" Service exists only because both manifests conventionally use the same label — there's no structural reference between them.

**Q8. What's the difference between a Service's `port` and `targetPort`?**
*Answer:* `port` is the port the Service itself listens on for other callers; `targetPort` is the port the actual container listens on. They can differ — a Service could expose `80` while the container listens on `8081`.

**Q9. What's the precise difference between exceeding a CPU limit and exceeding a memory limit in Kubernetes?**
*Answer:* Exceeding a memory limit gets the container killed (OOMKilled) — memory isn't compressible, so the kernel's cgroup OOM killer terminates the process. Exceeding a CPU limit throttles the container instead — CPU time is compressible, so the CFS bandwidth controller just caps how much CPU time it gets per period, making it slower rather than killing it.

**Q10. What does the scheduler actually use resource `requests` for, versus `limits`?**
*Answer:* `requests` is the bin-packing input the scheduler uses to decide whether a node has room for a Pod, compared against the node's allocatable capacity. `limits` is a runtime-enforced ceiling on actual usage once the Pod is running — two different mechanisms serving two different purposes.

**Q11. What does `kubectl apply --dry-run=client` actually validate, and what can it miss?**
*Answer:* It validates that the manifest is well-formed and matches the resource's client-side schema — catches typos and structural errors. It does not catch a label-selector mismatch, a non-existent container image, or anything requiring live cluster state or server-side admission control — `--dry-run=server` is needed for that level of validation.

**Q12. Walk through, end to end, what happens after `kubectl apply` on a Deployment manifest.**
*Answer:* The API server validates and persists the desired state to etcd; the Deployment controller notices via a watch and creates/updates a ReplicaSet; the ReplicaSet controller creates the specified number of Pods; `kube-scheduler` assigns each Pod to a node using its resource requests; the kubelet on that node pulls the image and starts the container; the Service's Endpoints object updates once the Pod is running and matches the selector. This is Day 99's reconcile loop, traced concretely.

---

## Daily Deliverable Check

- [ ] `load-test.js` written and run: 50 VUs, 30 seconds, against the Product module's read endpoints, producing a summary with request rate and p95 latency — saved as Day 140's baseline.
- [ ] Can compute p50/p95/p99 by hand from a raw list of latencies, and explain why the average alone can mislead.
- [ ] `k8s/<module>-deployment.yaml` and `k8s/<module>-service.yaml` written for all four modules (Product, Order, Payment, Notification).
- [ ] Every manifest passes `kubectl apply --dry-run=client -f k8s/` and is pushed.
- [ ] Can explain, from memory, exactly what breaks (and what doesn't) when a Service's selector doesn't match a Deployment's Pod labels.
- [ ] Can state the CPU-throttle-vs-memory-OOM-kill distinction precisely, unprompted.

---

## What Tomorrow Assumes You Already Know Cold

Day 135 takes today's raw Deployment and Service manifests as a **given, stable artifact** and immediately starts templating them into a Helm chart — it does not re-explain what a Deployment's `selector`/`template.labels` pairing does, what `resources.requests`/`limits` mean, or how a Service finds Pods. If any of those three mechanisms feel shaky, that's worth fixing before tomorrow rather than during it, since tomorrow's entire content is "the same manifests, now parameterized" — the underlying Kubernetes objects themselves are assumed fully reflexive.
