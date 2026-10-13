# Day 95 — Missing Primer: Install and Start a Local Kubernetes (kubectl, Minikube, Helm)

**Do this first on Day 95 (≈ 45 min), then continue with Day 95's regular plan.**
**Needed by:** Day 96 (ConfigMaps and Secrets applied to a cluster), Day 97, Day 99 (HPA — "assumes `kubectl`, Minikube, and a running Deployment already in place"), Days 134–138 (platform deployment, Helm, chaos).
**Why it is missing:** the Kubernetes primer teaches concepts and `kubectl` usage but never says how to install the tools or start a cluster; Day 99's book flags the gap but does not fix it. Helm is first needed on Day 135 and never installed anywhere (`brew install` appears only for k6).
**Already in place:** Docker (Days 43–44), the Kubernetes Fundamentals primer (labelled Day 96).

## 1. Install once
```bash
brew install kubectl minikube helm
kubectl version --client && minikube version && helm version
```

## 2. Start a cluster sized for a small machine
Minikube's Docker driver runs the whole cluster inside Docker Desktop's VM, so **Docker Desktop's memory limit is the ceiling** (Docker Desktop → Settings → Resources). On an 8 GB Mac give Docker about 4–5 GB, close memory-hungry apps, and start small:
```bash
minikube start --driver=docker --cpus=2 --memory=3072
kubectl get nodes                 # expect one node, STATUS Ready
kubectl config current-context    # expect: minikube
```
`minikube stop` pauses it (state kept); `minikube delete` wipes it. Run the cluster only on days that need it — Kafka, Postgres, Redis and four services will not all fit at once on this machine.

## 3. Add-ons you will need later
```bash
minikube addons enable metrics-server     # Day 99 (HPA reads CPU from it)
minikube addons enable ingress            # Day 135 (if you route through an Ingress)
```

## 4. Getting your own images into the cluster (Days 99, 134)
A local image is not on Docker Hub, so the cluster cannot pull it. Use either:
```bash
minikube image load todo-api:latest               # copy an image you built with Docker into Minikube
minikube image build -t todo-api:latest .         # or build directly inside Minikube
```
and set `imagePullPolicy: IfNotPresent` in the manifest (otherwise `ImagePullBackOff`).

## 5. First-day troubleshooting
| You see | Check |
|---|---|
| pod stuck `Pending` | `kubectl describe pod <name>` → usually not enough CPU/memory; raise Docker Desktop's limit, then recreate Minikube |
| `ImagePullBackOff` | §4 |
| `Unable to connect to the server` | `minikube status`; `minikube start` |

## 6. How this fits the books
Do the Kubernetes Fundamentals primer on Day 96 as planned. Day 99's book re-teaches "from containers to Pods" because it was written before that primer existed — treat that section as review.

## Checklist
- [ ] `kubectl`, `minikube`, `helm` installed; `kubectl get nodes` shows `Ready`.
- [ ] Docker Desktop memory limit set deliberately; Minikube started with explicit `--cpus/--memory`.
- [ ] Can load a local image into Minikube.
- [ ] Continue with Day 95's regular plan.
