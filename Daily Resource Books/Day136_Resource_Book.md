# Day 136 — Multi-Environment Configuration: Dev, Staging, Production

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 135 Resource Book](Day135_Resource_Book.md)
**Next ▶:** [Day 137 Resource Book](Day137_Resource_Book.md)
**Companion to:** Day 136 of `Week_20_Revised.md`

---

## Recap

Yesterday's Helm chart has exactly one `values.yaml`, providing one set of defaults for every environment. Today layers three environment-specific files on top of that same, unmodified chart — the templates themselves don't change at all today, only which values feed them. This is also the day today's new material turns out to already have a direct ancestor: **Spring Profiles** (Week 8, Day 51) established the exact same idea — `spring.profiles.active` selects which profile-specific file's values layer on top of the shared `application.yml`, one build artifact, different runtime behavior per environment — one layer *up* the stack from where today's work happens. That citation isn't incidental; it's the throughline the whole day is organized around.

---

## Learning Objectives

By the end of today, without notes:

1. Predict the final resolved value of a given key across multiple layered Helm values files, including the specific case where a nested map partially overrides a lower-precedence file and a list wholesale-replaces it instead of merging.
2. State precisely why a Kubernetes Secret is not, by itself, encryption — and prove it, not just assert it.
3. Explain the "build once, deploy many times" principle and argue concretely why baking environment-specific values into the image itself undermines the actual point of testing.
4. Explain why editing a ConfigMap does not, by itself, update already-running Pods, and name a real mechanism that closes that gap.

---

## Concept Dependency Map

```
Week 8, Day 51: Spring Profiles — spring.profiles.active selects which
                profile file's values layer on top of application.yml;
                ONE build artifact, environment-specific behavior at startup
Day 135: helm-chart/ + a single values.yaml
        │
        ▼
Day 136
        │
        ├── -f override precedence: values.yaml (base) < values-dev.yaml
        │     < values-staging.yaml < values-prod.yaml (later -f wins;
        │     maps deep-merge, lists replace wholesale)
        │
        ├── ConfigMap (non-sensitive) vs. Secret (sensitive —
        │     base64-encoded by default, proven NOT to be encryption)
        │
        ├── "Build once, deploy many times" — one image, externalized
        │     config, argued against the image-per-environment alternative
        │
        └── ConfigMap edits don't auto-restart Pods — env vars are read
              once at container startup, not re-read live
        │
        ▼
Day 137: tracing gets added to this now environment-aware, Helm-deployed platform
```

---

# Part 1 — Layering Values Files

```bash
helm install ecommerce-platform ./helm-chart -f values-dev.yaml
```

**The precedence rule, precisely:** `values.yaml` supplies the base default for *every* key the chart's templates reference. Each `-f <file>` supplied at install/upgrade time overrides specific keys on top of that base — and when multiple `-f` flags are given, files listed **later** win over files listed earlier on any key they both set. A key that a values file simply doesn't mention is left untouched, falling through to whatever a lower-precedence file (ultimately `values.yaml` itself) already set — this is a **merge**, not a **replacement**.

```yaml
# values-dev.yaml
replicaCount: 1
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
logging:
  level: DEBUG
database:
  host: dev-db.internal

# values-staging.yaml
replicaCount: 2
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
logging:
  level: INFO
database:
  host: staging-db.internal

# values-prod.yaml
replicaCount: 3
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "1000m"
    memory: "1Gi"
logging:
  level: WARN
database:
  host: prod-db.internal
```

**Worked trace — predicting a resolved value, not just asserting one:** suppose `values.yaml` (the base) sets `image.pullPolicy: IfNotPresent` and `replicaCount: 2`, and `values-prod.yaml` sets `replicaCount: 3` but never mentions `image.pullPolicy` at all.

```bash
helm install ecommerce-platform ./helm-chart -f values-prod.yaml
```

`replicaCount` resolves to **3** — `values-prod.yaml` explicitly overrides it. `image.pullPolicy` resolves to **`IfNotPresent`** — untouched by the prod file, so it falls through to the base `values.yaml`'s default. The prod install ends up with a genuine *mix* of prod-specific and base-default values, not a wholesale replacement of one file by another — assuming the opposite (that supplying `-f values-prod.yaml` means "use *only* what's in this file, ignore the base entirely") is a real, checkable misconception, and it's exactly the kind of assumption that produces a silently-wrong deployment rather than a loud error.

**⚠️ Common Mistake, precise and worth stating exactly:** this merge behavior applies to **maps**, not to **lists**. If a lower-precedence file sets a list value (say, an array of allowed CORS origins) and a higher-precedence file sets the *same* key to a different list, the higher-precedence file's list **wholesale replaces** the lower one — it does not append to it or merge element-by-element. A values file assuming it's only "adding one more allowed origin" by listing just that one origin will actually end up with a list containing *only* that one origin, silently dropping every origin the base file had. Maps merge; lists replace — the two behave differently, and conflating them is a genuine, documented Helm surprise.

---

# Part 2 — ConfigMaps and Secrets

**ConfigMap** — non-sensitive configuration (a log level, a feature flag, a non-secret hostname), stored as plain key-value data, injected into a Pod either as environment variables or as mounted files.

**Secret** — the same shape, intended for sensitive values (database passwords, API keys). **Precisely, and worth proving rather than taking on faith: a Kubernetes Secret's value is base64-*encoded* by default — it is not encrypted.**

```bash
echo -n 'my-db-password' | base64
# bXktZGItcGFzc3dvcmQ=

echo 'bXktZGItcGFzc3dvcmQ=' | base64 -d
# my-db-password
```

Encoding and decoding are both trivial, reversible, single-command operations with no key or secret material involved at all — anyone with read access to the Secret object (via `kubectl get secret -o yaml`, or direct access to the underlying etcd store) can recover the plaintext instantly. This is worth being able to demonstrate, not just recite, because "Secrets are encrypted" is a genuinely common, genuinely wrong assumption.

**What actually provides real protection, named without building any of them today (extension material):** enabling encryption-at-rest for the Secret data inside etcd itself (a cluster-admin-level API server configuration, not something a Secret manifest controls on its own), or moving secret material out of plain Kubernetes Secrets entirely via an external secret manager — HashiCorp Vault, AWS Secrets Manager, or a Kubernetes-native tool like Sealed Secrets or the External Secrets Operator, which keep the real plaintext outside of both git and any base64-reversible object.

**🔗 Backward Reference (Week 8, Day 51):** ConfigMaps and Secrets are *how environment-specific values physically get into a running container* — typically as environment variables. Spring Profiles is how the *application itself* then decides which of its own config layers to actually use, once those values have arrived — commonly by reading a `SPRING_PROFILES_ACTIVE` environment variable that was itself sourced from exactly this kind of ConfigMap.

**🔑 Key Takeaway:** ConfigMap/Secret injection and Spring Profiles are two different layers of the identical underlying idea — "one build artifact, externalized values determine its behavior" — not two unrelated mechanisms that happen to share a theme.

---

# Part 3 — "Build Once, Deploy Many Times"

**The alternative this argues against:** baking environment-specific values directly into the image — a separate build (and separate Dockerfile, or separate build args) per environment, with the database connection string, log level, and so on hardcoded at build time rather than supplied at deploy time.

**Why this is worse, concretely, not just "against best practice":**

1. **It multiplies build and test cost.** Three environments means building and validating three separate images instead of one — real CI time, three times over, for work that produces functionally identical application code.
2. **It breaks the actual point of testing.** If staging and production are genuinely different image builds, the artifact that passed every test in staging is **not the same set of bytes** being shipped to production — some difference, however small, sits between "what was validated" and "what actually runs." The entire premise of testing before a release is that the tested artifact *is* the released artifact; baking environment differences into the image itself quietly breaks that premise.

**The proper approach, and why today's actual work is what makes it possible:** build exactly one image per release. Promote that *same* image reference unchanged across dev, staging, and production. Vary only the externalized configuration — today's Helm values files plus ConfigMaps/Secrets — around it. This is precisely what Day 139's CI/CD pipeline is going to formalize: one build produces one image, and separate `helm install ... -f values-<env>.yaml` invocations are what differ, not the artifact itself.

**⚠️ Common Mistake:** treating separate per-environment images as the "safer" choice, on the theory that isolation feels careful. It's the opposite of safe in the specific, checkable sense above — it removes any guarantee that what was tested is what ships.

---

# Part 4 — Why a ConfigMap Edit Doesn't Update a Running Pod

**The mechanism, precisely:** when a ConfigMap's values are injected as **environment variables**, those variables are read exactly once, at container startup — a running container has no ongoing connection back to the ConfigMap object at all. Editing the ConfigMap changes the stored object; it does not push anything into an already-running process.

**Traced concretely:**

```bash
kubectl edit configmap product-config -n ecommerce   # change LOG_LEVEL: INFO -> DEBUG
kubectl logs -f deployment/product-service            # still logging at INFO — nothing changed
```

Nothing visibly fails — which is exactly what makes this a dangerous gap rather than an obvious one. The application keeps behaving as if the old value were still current until something forces its Pods to restart.

**The two real fixes, named precisely:**

1. **Manual or scripted rollout restart** after any config change — `kubectl rollout restart deployment/product-service` — simple and correct, but requires actual discipline (or automation) to remember every time, since nothing enforces it.
2. **A watcher tool** (Stakater's **Reloader** is the commonly cited real-world example) that watches ConfigMaps and Secrets and automatically triggers a rolling restart of any Deployment referencing one, the moment it changes — the more mature fix, named here as **extension material**, not built today.

**A precise nuance worth having, since it's easy to get slightly wrong:** mounting a ConfigMap as a **volume** (a file inside the container) instead of an environment variable behaves differently at the kubelet level — the kubelet *does* periodically sync the mounted file's content to reflect ConfigMap changes, without a Pod restart. But this only solves half the problem: **the application still has to actually notice and re-read that file at runtime** to see the update. Spring Boot doesn't do this automatically on its own; live config reload needs something like Spring Cloud Config's `@RefreshScope` explicitly wired up — and Day 51's own material named Spring Cloud Config as a real pattern that exists, while explicitly **not** building it for this platform. So even the volume-mount path doesn't close this gap by itself here, without that additional piece.

---

# Exercise: The Three Environment Values Files

**The goal:** the three files shown in Part 1, wired end to end — a demonstrably different deployment from the exact same chart.

```bash
helm install ecommerce-platform-dev ./helm-chart -f values-dev.yaml -n ecommerce-dev --create-namespace
helm install ecommerce-platform-prod ./helm-chart -f values-prod.yaml -n ecommerce-prod --create-namespace

kubectl get deployments -n ecommerce-dev
kubectl get deployments -n ecommerce-prod
```

**Definition of done:** the dev and prod installs show visibly different replica counts and resource configurations in `kubectl get deployment -o yaml`, despite both having been rendered from the identical `helm-chart/` directory.

**⚠️ Common Mistakes, full checklist for today:**

- Committing a values file containing real, plaintext-equivalent (base64) secrets to git — base64 is encoding, not protection, and git history is effectively permanent.
- Assuming a values file listing one additional item in a list field merges with the base file's list, rather than replacing it wholesale.
- Assuming a `kubectl edit configmap` takes effect on already-running Pods without a restart.
- Building separate images per environment "to be careful," which actually removes the guarantee that the tested artifact is the shipped one.

**💡 Interview Insight:** "How do you manage configuration and secrets across environments?" is a near-guaranteed question once Kubernetes and multi-environment deployment are on a resume. The strongest answer names the specific layering mechanism (a base chart plus environment-specific values files), is explicit that Secrets are base64-encoded and names what *actually* provides protection instead, and states the "build once, deploy many times" principle as the reason config is externalized in the first place rather than baked into the image — three concrete, checkable claims instead of one vague "we use environment variables."

---

# Project Block Guide (3.5 hrs)

**Repository:** `scalable-ecommerce-platform`.

**Task:** create `values-dev.yaml` (1 replica, relaxed limits, verbose/`DEBUG` logging), `values-staging.yaml` (2 replicas, moderate limits, staging database host), and `values-prod.yaml` (3+ replicas, tighter limits, production-grade config, minimal/`WARN` logging), exactly as shown above. Wire the corresponding ConfigMaps and Secrets per environment.

**Definition of done:** `helm install ... -f values-dev.yaml` and `helm install ... -f values-prod.yaml` produce visibly different deployments — different replica counts, different config — from the exact same underlying chart.

---

# Career Block Guide (1 hr)

**LinkedIn:** 20 minutes of genuine engagement — commenting on 3–5 posts.

**Networking:** research the hiring manager for a role already applied to, and send a direct, specific message — referencing something real from their profile or recent activity, not a generic connection request.

---

# Day 136 — Interview Questions

**Q1. If `values.yaml` sets `replicaCount: 2` and `values-prod.yaml` sets `replicaCount: 3` but says nothing about `image.pullPolicy`, what does a prod install actually resolve to for each key?**
*Answer:* `replicaCount` resolves to 3 (explicitly overridden by the prod file); `image.pullPolicy` resolves to whatever `values.yaml` set as the base default, since the prod file never mentions it and values files merge rather than wholesale-replace each other.

**Q2. Do Helm values files merge the same way for lists as they do for maps?**
*Answer:* No. Maps deep-merge — an unmentioned key falls through to a lower-precedence file. Lists wholesale-replace — a list set in a higher-precedence file completely replaces the corresponding list from a lower-precedence file rather than appending to or merging with it.

**Q3. Is a Kubernetes Secret encrypted?**
*Answer:* Not by default — its values are base64-encoded, which is trivially reversible with a single `base64 -d` command and no key required. Real protection needs etcd encryption-at-rest configured separately, or an external secret manager (Vault, AWS Secrets Manager, Sealed Secrets) that keeps plaintext out of both git and any base64-reversible object.

**Q4. What's the direct connection between today's ConfigMap/Secret work and Week 8's Spring Profiles?**
*Answer:* ConfigMaps and Secrets are how environment-specific values physically reach a running container, typically as environment variables. Spring Profiles is how the application itself then decides which of its own config layers to use once those values arrive — two layers of the same "one build artifact, externalized values determine behavior" idea, not two unrelated mechanisms.

**Q5. Why is baking environment-specific configuration into the image itself worse than externalizing it, beyond just extra build time?**
*Answer:* It breaks the core premise of testing — if staging and production run genuinely different image builds, the artifact validated in staging is not the same bytes shipped to production, so passing tests no longer guarantees anything about what's actually deployed.

**Q6. If you edit a ConfigMap that's injected into a Pod as an environment variable, does the running Pod pick up the change?**
*Answer:* No. Environment variables are read once, at container startup; a running container has no ongoing connection to the ConfigMap object. The Pod needs an explicit rollout restart (manual, scripted, or via a watcher tool like Reloader) to see the new value.

**Q7. Does mounting a ConfigMap as a volume instead of an environment variable fully solve the live-update problem?**
*Answer:* Only partially. The kubelet does periodically sync the mounted file's content without a Pod restart, but the application still has to actually notice and re-read that file at runtime — which Spring Boot doesn't do automatically without something like Spring Cloud Config's `@RefreshScope` explicitly wired up.

---

## Daily Deliverable Check

- [ ] `values-dev.yaml`, `values-staging.yaml`, `values-prod.yaml` created, each producing a visibly different deployment from the same chart.
- [ ] Corresponding ConfigMaps and Secrets wired per environment.
- [ ] Can trace, by hand, the resolved value of a key across multiple layered values files, including the maps-merge-but-lists-replace nuance.
- [ ] Can prove — not just state — that a Kubernetes Secret is base64-encoded rather than encrypted.
- [ ] Can explain why a ConfigMap edit doesn't reach an already-running Pod, and name at least one real fix.

---

## What Tomorrow Assumes You Already Know Cold

Day 137 deploys distributed tracing across this same environment-aware, Helm-managed platform — it assumes today's values-file layering is solid enough that adding a new configuration value (tracing endpoint, sampling rate) to all three environment files is a completely mechanical extension of a pattern already understood, not new territory. It also assumes the ConfigMap/Secret vs. Spring Profiles distinction from today, since tomorrow's tracing configuration is itself exactly this kind of externalized, per-environment value.
