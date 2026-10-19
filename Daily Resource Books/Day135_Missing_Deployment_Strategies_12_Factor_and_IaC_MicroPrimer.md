# Day 135 — Missing Primer: Deployment Strategies, Feature Flags, the Twelve-Factor App, and Infrastructure as Code

**Do this first on Day 135 (≈ 45 min, alongside Helm), then continue with Day 135's regular plan.**
**Needed by:** Day 136 (multi-environment configuration), Day 138 (chaos and rollback), Day 139 (CI/CD), and SDE-2 "how do you ship safely?" questions.
**Why it is missing:** blue-green, canary, feature flags, the twelve-factor principles and Infrastructure as Code are mentioned in passing (Days 34, 96, 99, 136) but never taught.
**Already in place:** Kubernetes Deployments, probes and ConfigMaps/Secrets (Days 96–99), Helm (Day 135's theory), Flyway migrations (Days 36–40), the CI/CD primer (Day 139).

---

## 1. Deployment strategies
| Strategy | How | Trade-off |
|---|---|---|
| **Recreate** | stop old, start new | simplest; downtime |
| **Rolling update** (Kubernetes default) | replace pods gradually | no downtime; both versions run briefly, so they must be compatible |
| **Blue-green** | run a full new environment beside the old; switch traffic at once; keep the old for instant rollback | fast rollback; double the capacity |
| **Canary** | send a small share (1 % → 10 % → 100 %) to the new version and watch the SLIs (Day 76's file) | limits blast radius; needs traffic splitting (Ingress/mesh) and good metrics |
| **Shadow** | mirror live traffic to the new version, ignore its responses | tests under real load without user impact |
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }     # add one new pod before removing an old one
```
Readiness probes gate the rollout: a new pod receives traffic only when `/actuator/health/readiness` is up. Commands: `kubectl rollout status deployment/order-service`, `kubectl rollout undo deployment/order-service`.

## 2. Compatible releases: database migrations and feature flags
Because old and new code run **at the same time**, schema changes must be backward-compatible — **expand → migrate → contract**: (1) add the new nullable column; (2) deploy code that writes both; (3) backfill; (4) switch reads; (5) later drop the old column. Never rename or drop in one step (Flyway, Day 40).
**Feature flags** separate **deploy** from **release**: ship the code dark, turn it on for 1 % of users, kill it instantly without a rollback. Costs: flag debt (delete old flags), more test paths.

## 3. The Twelve-Factor App (the ones that matter)
**Config in the environment** (Days 96, 136) · **stateless processes**, state in backing services (Redis/DB) · **backing services as attached resources** (swap by URL) · **build / release / run** separated (one image, many environments) · **logs as event streams** to stdout · **dev/prod parity** (same Postgres major version — Day 36/44/47) · **port binding** · **disposability** (fast start, graceful shutdown) · **concurrency by scaling out processes**.

## 4. Infrastructure as Code (IaC)
Describe infrastructure in files, review it like code, apply it repeatably. **Terraform** — declarative, multi-cloud; `plan` shows the diff, `apply` makes it, a **state** file records reality (guard it; remote state with locking). **CloudFormation** — AWS-native. **Pulumi** — real programming languages. **Helm** (today) packages Kubernetes manifests; Terraform provisions the cluster and cloud resources underneath — different layers.

## 5. Exercise (≈ 20 min, Minikube)
1. Deploy a sample app with 3 replicas. Update the image with a **deliberately failing readiness probe**: watch the rollout stall with old pods still serving; `kubectl rollout undo`.
2. Write the **expand/contract** plan for renaming `tasks.due_date` to `tasks.deadline` as five numbered steps.
3. Decide, with one reason each: rolling, blue-green or canary for (a) a payment-service change, (b) a typo fix, (c) a new recommendation algorithm.

## Checklist
- [ ] Can contrast the five strategies and pick one for a given risk.
- [ ] Can explain expand/contract and why old and new code overlap.
- [ ] Can state five twelve-factor principles; can say what Terraform adds beyond Helm.
- [ ] Continue with Day 135's regular plan.
