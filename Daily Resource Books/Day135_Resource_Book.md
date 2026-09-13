# Day 135 — Helm Charts, and Deploying the Platform for Real

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 134 Resource Book](Day134_Resource_Book.md)
**Next ▶:** [Day 136 Resource Book](Day136_Resource_Book.md)
**Companion to:** Day 135 of `Week_20_Revised.md`

---

## Recap

Yesterday you hand-wrote a Deployment and a Service manifest for each of the platform's four modules — eight files, every value hardcoded directly into the YAML. That was deliberate: today's entire lesson is "the same eight manifests, now parameterized," and the difference only means something if yesterday's raw version is fresh. Today also reuses Spring Cloud Gateway (Week 10, Day 66) as the thing being contrasted against a genuinely different, cluster-level concept — Kubernetes Ingress — so the two "gateway-shaped" ideas don't get silently merged into one.

---

## Learning Objectives

By the end of today, without notes:

1. State, with real numbers, exactly why hand-maintained per-environment manifests stop scaling — not as received wisdom, but as an argument you can reproduce.
2. Write a Helm chart (`Chart.yaml`, `values.yaml`, `templates/`) that renders to valid Kubernetes manifests, and trace precisely what `helm install` does, step by step.
3. Explain the difference between Kubernetes Ingress and Spring Cloud Gateway, place each correctly in the request path, and explain why the platform needs both, not one instead of the other.
4. Deploy the platform to a local Minikube cluster via Helm and verify every module is running and healthy.

---

## Concept Dependency Map

```
Day 134: raw Deployment + Service manifests, one pair per module, values hardcoded
Week 10, Day 66: Spring Cloud Gateway — application-level entry point,
                 routing + auth + rate limiting, its own Deployment
        │
        ▼
Day 135
        │
        ├── The templating problem, stated as a scaling argument (not asserted)
        │
        ├── Helm's mechanism
        │     ├─ Chart.yaml (metadata) / values.yaml (defaults) / templates/
        │     ├─ {{ .Values.x }} placeholders, _helpers.tpl named templates
        │     └─ helm install, traced end to end — same API calls as kubectl,
        │         automated, plus a stored "release" for upgrade/rollback
        │
        ├── Ingress vs. Spring Cloud Gateway — two different layers,
        │     explicitly not the same thing despite both being "a gateway"
        │
        └── Minikube — a local single-node cluster to deploy the chart into
        │
        ▼
Day 136: values-dev.yaml / values-staging.yaml / values-prod.yaml,
         layered on top of today's exact same chart
```

---

# Part 1 — The Templating Problem, As a Real Scaling Argument

**The concrete cost of yesterday's approach, made explicit:** four modules, two manifests each (Deployment + Service) — 8 files. Every one of those files is currently hand-edited directly. Tomorrow adds three environments (dev, staging, prod), each needing its own replica count, resource limits, and config. Extended naively, that's:

| | File count |
|---|---|
| Today (1 "environment," hardcoded) | 4 modules × 2 manifests = **8 files** |
| Naive per-environment copies (3 environments) | 4 modules × 2 manifests × 3 environments = **24 files** |
| With a Helm chart | 1 chart (≈4 template files) + 3 small values files = **≈7 files, total** |

The naive approach scales as **O(modules × environments)** — every new environment multiplies the *entire* file count, and every shared change (bumping a base resource limit, adding a new label convention) has to be found and edited in every one of those files, correctly, by hand, everywhere it appears. The templated approach scales as **O(modules) + O(environments)** — one small template set, plus one small values file per environment. This is the same shape of argument as any other "why the optimized approach is better" case in this series: not a vague appeal to "best practice," but a concrete claim about how badly the alternative degrades as a real variable (environment count) grows.

**⚠️ Common Mistake:** treating Helm as adding complexity for its own sake. The template *indirection* is a real, small cost (a new syntax to learn, one more layer between "what I wrote" and "what actually got applied") — but it's a cost paid once, in exchange for eliminating a cost that would otherwise be paid on every single environment added from here forward.

**🔑 Key Takeaway:** templating doesn't reduce how much *configuration* exists — it reduces how many *places* that configuration has to be hand-edited when something shared changes. That's the entire value proposition, stated precisely rather than as a vague appeal to best practice.

**🔗 Backward Reference (Day 134):** every field in the templates below is the exact same field from yesterday's raw manifests — `replicas`, `image`, `containerPort`, `resources` — now pulled from `.Values` instead of hardcoded. Nothing about the underlying Kubernetes objects changes today, only where their values come from.

---

# Part 2 — Helm's Mechanism

A Helm **chart** is a directory with a specific structure:

```
helm-chart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── _helpers.tpl
    ├── product-deployment.yaml
    ├── product-service.yaml
    ├── order-deployment.yaml
    ├── order-service.yaml
    ├── payment-deployment.yaml
    ├── payment-service.yaml
    ├── notification-deployment.yaml
    ├── notification-service.yaml
    └── ingress.yaml
```

**`Chart.yaml`** — the chart's own metadata:

```yaml
apiVersion: v2
name: ecommerce-platform
description: Helm chart for the scalable-ecommerce-platform capstone
version: 0.1.0
appVersion: "1.0.0"
```

`apiVersion: v2` is the current Helm 3 chart format (Helm 2's `v1` format is legacy). `version` is the chart's own version (bump this when the chart's templates change); `appVersion` is informational metadata about the application version being deployed — the two are independent numbers tracking two different things, worth not conflating.

**`values.yaml`** — the chart's default configuration:

```yaml
replicaCount: 2

image:
  tag: latest
  pullPolicy: IfNotPresent

modules:
  product:
    port: 8081
  order:
    port: 8082
  payment:
    port: 8083
  notification:
    port: 8084

resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"

ingress:
  enabled: true
  host: ecommerce.local
```

**`templates/product-deployment.yaml`** — Day 134's raw manifest, now parameterized:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
  namespace: ecommerce
  labels:
    {{- include "ecommerce.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
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
          image: "product-service:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.modules.product.port }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

**`templates/_helpers.tpl`** — a named template for the one piece of boilerplate every manifest in the chart repeats:

```yaml
{{- define "ecommerce.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end -}}
```

- **`{{ .Values.x }}`** pulls a value straight from `values.yaml` (or from whichever `-f` file overrides it — tomorrow's mechanism).
- **`{{ .Chart.Name }}` / `{{ .Release.Name }}`** are Helm's own built-in objects, not user-defined values — `Release.Name` is the name given at `helm install <name> ...` (`ecommerce-platform`, per the plan), automatically available in every template.
- **`{{- toYaml .Values.resources | nindent 12 }}`** is the correct pattern for inserting a *nested* YAML block (like the whole `resources:` map) from values into a template, indented to match the surrounding structure. Using bare `{{ .Values.resources }}` instead would **not** produce valid YAML — Go's template engine would stringify the underlying map object directly (its Go `map[string]interface{}` representation), not render it as YAML — this is a genuinely common, easy-to-hit Helm mistake, not a hypothetical one.
- **`{{- ... -}}`** (the hyphens) strip surrounding whitespace/newlines that would otherwise leave stray blank lines in the rendered output — a small but real detail, since Helm-rendered YAML still has to be valid YAML, and YAML is whitespace-sensitive.

**`templates/ingress.yaml`** — routes external traffic in:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  namespace: ecommerce
spec:
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: gateway-service
                port:
                  number: 8080
```

The other three modules' Deployment/Service templates follow the identical shape shown for Product — same structure, `.Values.modules.order.port` in place of `.Values.modules.product.port`, and so on. **Extension material, worth naming but not required today:** at real scale, these four near-identical template pairs could themselves be collapsed into a single templated file using a `{{ range $name, $module := .Values.modules }}` loop over the `modules` map — the same "don't repeat structurally identical content" instinct behind Helm itself, applied one level deeper. Today keeps four explicit files instead, since the direct one-to-one mapping from yesterday's raw manifests makes the *transformation itself* easy to verify by eye — the loop-based version is a genuine next step, not today's task.

## `helm install`, Traced End to End

```bash
helm install ecommerce-platform ./helm-chart
```

1. Helm reads `Chart.yaml`, `values.yaml`, and every file under `templates/`.
2. It renders every template — substituting `{{ .Values.x }}` and the built-in objects (`.Release`, `.Chart`) with real values — producing plain, final Kubernetes YAML in memory. (`helm template ./helm-chart` runs *only* this step, printing the rendered YAML without applying anything — genuinely useful for previewing output before committing to it, the Helm-level analog of `kubectl apply --dry-run`.)
3. Helm submits that rendered YAML to the Kubernetes API server — **the exact same API calls a manual `kubectl apply` would make**; nothing about how the cluster processes the request is different once it arrives.
4. From there, it's Day 134's reconcile loop, unchanged: the API server persists desired state, the Deployment controller creates ReplicaSets, ReplicaSets create Pods, the scheduler places them, kubelets start containers.
5. Helm additionally records this as a numbered **release** — the exact rendered manifest, stored, tied to a release name and revision number. This is what makes `helm upgrade` (apply a new version of the chart/values against an *existing* release) and `helm rollback ecommerce-platform 1` (revert to a previously stored revision) possible — a capability plain `kubectl apply` doesn't give you on its own, since `kubectl` has no built-in concept of "the previous state before this."

**⚠️ Common Mistake:** running `helm install` again against a release name that already exists — it fails outright, since `install` assumes the release is new. `helm upgrade --install <name> ./chart` is the idempotent form (installs if absent, upgrades if already present) almost always used in real automation, including Day 139's own CI/CD pipeline.

---

# Part 3 — Ingress vs. Spring Cloud Gateway: Two Different Layers

These sound similar and are easy to conflate — worth a direct, explicit disambiguation rather than letting the names blur together.

| | **Kubernetes Ingress** | **Spring Cloud Gateway** (Week 10, Day 66) |
|---|---|---|
| What it is | A Kubernetes *object*, interpreted by a separately-installed **Ingress Controller** (e.g., NGINX Ingress Controller) | An ordinary Spring Boot application, running as its own Deployment/Pod |
| Layer | Cluster networking layer | Application layer |
| What it does | Routes external HTTP(S) traffic by **host/path** to the right internal Service | Centralizes routing, authentication, and rate limiting **across the platform's own modules**, with real application logic |
| Business awareness | None — pure L7 host/path matching | Full — this is where JWT validation and Token Bucket rate limiting (Day 68) actually live |

**Where each sits in one real request, traced concretely:**

```
External client
      │
      ▼
Ingress  (matches host "ecommerce.local", routes to the Gateway's Service)
      │
      ▼
gateway-service  (a Kubernetes Service, ClusterIP or otherwise)
      │
      ▼
Spring Cloud Gateway Pod  (JWT validation, rate limiting, routes to the right module)
      │
      ▼
product-service / order-service / ... (internal Service → internal Pod)
```

**Why both are needed — neither replaces the other:** Ingress solves "how does traffic from outside the cluster even reach the right Service" — a networking problem, solved once, at the cluster edge, commonly also handling TLS termination. Spring Cloud Gateway solves a completely different problem — "once a request is inside my application tier, how do I apply authentication and rate limiting consistently across every module, in one place, in real application code" — Day 66's own reasoning for why it exists at all (routing, auth, and rate limiting would otherwise be independently implemented and drift across modules). Trying to make Ingress alone do the Gateway's job means pushing business logic into Ingress Controller annotations — fragile and limited compared to real Spring code. Trying to make the Gateway alone do Ingress's job means losing cluster-level host/path routing and typically losing the conventional place TLS termination happens.

**⚠️ Common Mistake:** assuming an Ingress object does something by itself. It does nothing without an Ingress Controller actually running in the cluster and watching for Ingress objects — installing that controller is a cluster-level prerequisite, separate from writing the Ingress YAML itself.

---

# Part 4 — Minikube

**What it is:** a single-node Kubernetes cluster that runs locally (inside a VM or container on your own machine), for development and testing — not a scaled-down "toy" API, a genuinely real Kubernetes control plane and kubelet, just with everything on one node instead of many.

`minikube start` launches it; `kubectl` then targets it exactly like any other cluster, since Minikube is a real conformant Kubernetes distribution, not a simulation.

**⚠️ Common Mistake, genuinely common in practice:** a locally-built Docker image is invisible to Minikube by default, because Minikube runs its own internal container runtime, separate from your host machine's Docker daemon. Building an image normally and then referencing it in a manifest produces `ImagePullBackOff` — Minikube tries to *pull* an image that only exists in your host's local Docker, not in any registry it can reach. The fix is either `eval $(minikube docker-env)` before building (so the build target *is* Minikube's own daemon) or `minikube image load <image>` after building normally — worth knowing this exists as a checklist item the first time a Pod gets stuck in this exact state.

---

# Exercise: Converting Day 134's Manifests Into a Helm Chart

**The goal:** everything Day 134 hand-wrote, rendered by one parameterized chart instead.

```bash
helm install ecommerce-platform ./helm-chart
kubectl get pods -n ecommerce
```

**Definition of done, verified concretely:** `helm install` completes without error; `kubectl get pods -n ecommerce` shows all four modules' Pods in `Running` state with their readiness column satisfied (today's manifests don't yet configure explicit readiness probes — Day 139's own topic — so "healthy" here means the container process is up and the Service has live endpoints, not yet a fully probe-verified readiness signal).

**⚠️ Common Mistakes, full checklist for today:**

- Referencing a `values.yaml` key that doesn't exist. A missing **nested** key (dotting further into something that's already nil) fails immediately at render time with a "nil pointer evaluating" error. A missing **top-level** key referenced directly often renders as the literal text `<no value>` inserted straight into the output YAML instead of failing cleanly — arguably worse than an error, since it can produce technically-parseable-but-wrong YAML that only fails later, deep inside Kubernetes API validation, far from the actual cause. `{{ .Values.foo | default "bar" }}` (an explicit fallback) or installing with `--strict` (which turns any nil access into an immediate failure) are the two real fixes — prefer failing loudly and immediately over a silent placeholder string.
- Inserting a nested YAML block with bare interpolation instead of `toYaml ... | nindent N` (detailed above) — produces a Go map's string form, not valid YAML.
- Running `helm install` twice against the same release name instead of `helm upgrade --install`.
- Forgetting Minikube needs the image loaded into its own runtime, not just built locally.

**💡 Interview Insight:** "Why Helm instead of raw manifests?" has a strong answer and a weak one. The weak answer is "it's best practice." The strong answer is today's Part 1 scaling argument, stated with real numbers: N environments × M modules of hand-maintained YAML, versus one chart plus N small values files — and being able to name the specific Helm mechanisms (templating, values layering, release tracking enabling rollback) rather than gesturing at "convenience." A natural, likely follow-up: "how would you roll back a bad deploy?" — `helm rollback <release> <revision>`, made possible specifically because Helm stores the full rendered manifest for every past release, a capability plain `kubectl apply` doesn't have on its own.

---

# Project Block Guide (3.5 hrs)

**Repository:** `scalable-ecommerce-platform`.

**Task:** build out `helm-chart/` exactly as templated above — `Chart.yaml`, `values.yaml`, `templates/_helpers.tpl`, and a Deployment + Service template pair for all four modules, plus `templates/ingress.yaml`.

**Definition of done:** `helm install ecommerce-platform ./helm-chart` successfully deploys the entire platform to a local Minikube cluster; `kubectl get pods -n ecommerce` shows every module running.

---

# Career Block Guide (1 hr)

**LinkedIn — Post 26:** "I finally moved my capstone off docker-compose and onto real Kubernetes, with Helm." A genuinely honest, differentiated post specifically because most portfolio projects never make this jump at all — worth saying plainly rather than performatively, since it's true.

**Networking:** applications went out Day 118 — it's now been over two weeks. If any of the 7 target companies has gone fully silent with no rejection and no next step, a brief, polite follow-up to the recruiter is reasonable at this point, not premature.

---

# Day 135 — Interview Questions

**Q1. State the concrete scaling argument for why Helm beats hand-maintained per-environment manifests.**
*Answer:* Raw manifests scale as O(modules × environments) — every new environment multiplies the entire file count, and every shared change has to be found and edited everywhere by hand. A Helm chart scales as O(modules) template files plus O(environments) small values files — one shared template set, parameterized once.

**Q2. What are the three core pieces of a Helm chart, and what does each one do?**
*Answer:* `Chart.yaml` holds the chart's own metadata (name, chart version, app version); `values.yaml` holds default configuration; `templates/` holds Go-template YAML files that get rendered by substituting `{{ .Values.x }}` placeholders with real values.

**Q3. What does `helm install` actually do, end to end?**
*Answer:* It reads the chart and values, renders every template into plain final Kubernetes YAML, submits that YAML to the API server via the same calls `kubectl apply` would make, and additionally records the result as a numbered release, enabling later `helm upgrade` and `helm rollback`.

**Q4. Why would `{{ .Values.resources }}` (bare interpolation) break a template that's inserting a nested YAML block?**
*Answer:* Bare interpolation stringifies the underlying Go map object directly rather than rendering it as YAML. The correct pattern is `{{- toYaml .Values.resources | nindent 12 }}`, which converts the value to proper YAML text and indents it to match the surrounding structure.

**Q5. What's the difference between `helm install` and `helm upgrade --install`, and why does automation almost always use the latter?**
*Answer:* `helm install` fails if a release with that name already exists. `helm upgrade --install` is idempotent — it installs if the release is absent and upgrades if it's already present — which is what makes it safe to run repeatedly from a CI/CD pipeline without first checking whether a prior deploy exists.

**Q6. Explain the difference between Kubernetes Ingress and Spring Cloud Gateway, and why the platform needs both.**
*Answer:* Ingress is a cluster-networking object, interpreted by an Ingress Controller, that routes external traffic by host/path to the right internal Service — no business logic. Spring Cloud Gateway is an actual application, doing JWT validation, rate limiting, and business-aware routing across the platform's own modules. Ingress gets traffic to the Gateway; the Gateway decides what happens to it once it's inside the application tier. Neither can cleanly replace the other.

**Q7. Does an Ingress object do anything by itself, once applied to the cluster?**
*Answer:* No — it does nothing without an Ingress Controller (e.g., NGINX Ingress Controller) already running in the cluster and watching for Ingress objects; the controller is a separate, cluster-level prerequisite.

**Q8. Why would a locally built Docker image fail to pull inside a Minikube-deployed Pod, even though `docker images` shows it exists?**
*Answer:* Minikube runs its own internal container runtime, separate from the host machine's Docker daemon. The image exists on the host but not inside Minikube's runtime, producing `ImagePullBackOff`. Fixed via `eval $(minikube docker-env)` before building, or `minikube image load` afterward.

**Q9. What happens if a Helm template references a `values.yaml` key that doesn't exist?**
*Answer:* A missing nested key (dotting further into an already-nil value) fails immediately with a nil-pointer render error. A missing top-level key referenced directly often renders as the literal text `<no value>` inserted into the output instead of failing — worth guarding against with an explicit `default` fallback or `--strict` mode, since a silent placeholder string is arguably worse than a clean failure.

---

## Daily Deliverable Check

- [ ] `helm-chart/` built: `Chart.yaml`, `values.yaml`, `templates/_helpers.tpl`, Deployment + Service templates for all four modules, `templates/ingress.yaml`.
- [ ] `helm install ecommerce-platform ./helm-chart` successfully deploys the full platform to Minikube.
- [ ] `kubectl get pods -n ecommerce` confirms every module running.
- [ ] Can state the O(modules × environments) vs. O(modules) + O(environments) scaling argument from memory, with real numbers.
- [ ] Can explain, unprompted, why Ingress and Spring Cloud Gateway are not the same thing and are not substitutes for each other.
- [ ] LinkedIn Post 26 published.

---

## What Tomorrow Assumes You Already Know Cold

Day 136 takes today's chart as a fixed, working artifact and layers three environment-specific `values-*.yaml` files on top of it — it does not re-explain `Chart.yaml`, `values.yaml`, or the templating mechanism itself. The one thing tomorrow does extend directly is `.Values` — today's chart reads defaults from a single `values.yaml`; tomorrow's whole lesson is what happens when a *second* values file is layered on top via `-f`, so the override-precedence rule needs today's basic templating mechanism to already be solid, not something to re-derive from scratch mid-lesson.
