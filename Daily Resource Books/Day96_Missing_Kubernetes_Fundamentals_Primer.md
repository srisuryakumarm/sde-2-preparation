# Kubernetes Fundamentals Primer: The Missing Piece Before Day 96

**Where this fits:** insert this before Day 96 (Kubernetes ConfigMaps and Secrets). No day before this ever explains what a Pod, Deployment, Service, Node, or cluster actually is — Day 96 jumps straight to "both can be injected into a pod as environment variables" as if that vocabulary were already settled. It gets used, in growing depth, across Days 95, 96, 97, 99, 125, and then extensively through 134–140 — this primer is the foundation all of those rest on.

**Why this exists:** you already know Docker (Day 43) and Docker Compose (Day 44) — running one container, then wiring several together on one machine. Kubernetes is the next step up, and everything that makes it different from Compose needs to click once, properly, before the vocabulary starts piling up.

**What this primer deliberately does *not* cover:** ConfigMaps/Secrets (Day 96, the very next day), StatefulSets/DaemonSets (Day 97), the Horizontal Pod Autoscaler (Day 99), Helm (Day 135), multi-environment values files (Day 136), distributed tracing in a cluster (Day 137), service mesh and chaos engineering (Day 138), or readiness/liveness probes and CI/CD (Day 139). Each of those gets its own real theory block exactly where it's needed. This primer's only job is the mental model and vocabulary underneath all of them: what a Pod is, what a Deployment does, what a Service is for, and how you actually talk to a cluster.

---

## Part 1: The Problem — Why an Orchestrator, Beyond Docker Compose

`docker-compose` assumes one machine. It brings up a fixed set of containers, on the host you run it on, and if that host goes down, everything on it goes down with it. Nothing restarts a crashed container automatically across a fleet of machines, nothing spreads load across multiple copies of a service, and nothing reschedules work if a machine dies.

**Kubernetes (K8s) is a container orchestrator**: you describe a *desired state* — "I want 3 healthy copies of this container running, always" — and Kubernetes continuously works to make the *actual state* match it, across a whole fleet of machines. If a container crashes, it gets restarted. If a machine dies, its work gets rescheduled elsewhere. If you want more copies to handle more load, you change one number.

The core mental shift from Compose: Kubernetes is **declarative**, not imperative. You don't script the steps to get to a working state — you describe the end state you want, in a file, and the system continuously reconciles reality toward it.

---

## Part 2: The Building Blocks

- **Node** — a single machine (physical or virtual) that's part of the cluster and capable of running containers.
- **Cluster** — the full set of Nodes, managed together as one unit by Kubernetes' own control-plane software.
- **Pod** — the smallest deployable unit in Kubernetes. **Not the same thing as a container.** A Pod wraps one or more containers that need to be scheduled and run together, sharing network and storage. In this plan, each module (Product, Order, Payment, Notification) will be one container inside one Pod — but you never run a container directly in Kubernetes; you always run a Pod.
- **Deployment** — describes the desired state for a set of identical Pods: "keep 3 replicas of this Pod running, built from this image, always." A Deployment continuously ensures that many Pods actually exist, replacing any that crash or get deleted. This is exactly the "Deployment" Day 97 later contrasts against StatefulSet ("Deployments treat pods as interchangeable...").
- **ReplicaSet** — the thing a Deployment actually creates and manages underneath to keep the right number of Pod copies alive. You'll rarely touch one directly — a Deployment manages it for you — but you'll see it in `kubectl get` output, so it's worth knowing it exists.
- **Service** — Pods are disposable: they're created and destroyed constantly (crashes, redeploys, scaling), and each one gets a fresh internal IP every time. A **Service** gives a stable address that automatically routes to whichever Pods are currently healthy, so nothing else in the system needs to track individual Pod IPs. *(Worth being explicit: this is a different meaning of "Service" from an application service like Order or Payment — same word, unrelated concept, easy to conflate.)*

---

## Part 3: A Manifest, Line by Line

Everything above gets described in YAML manifest files. Here's a minimal, complete pair — a Deployment and the Service in front of it:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
spec:
  replicas: 3                     # keep 3 identical Pods running
  selector:
    matchLabels:
      app: product-service        # how this Deployment finds "its" Pods
  template:                       # the actual Pod spec, stamped out 3 times
    metadata:
      labels:
        app: product-service      # must match selector.matchLabels above
    spec:
      containers:
        - name: product-service
          image: product-service:latest   # which Docker image to run
          ports:
            - containerPort: 8080         # port the container listens on
---
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  selector:
    app: product-service          # routes to any Pod carrying this label
  ports:
    - port: 80                    # port other things connect to on the Service
      targetPort: 8080            # port forwarded to on the actual container
  type: ClusterIP                 # only reachable from inside the cluster (the default)
```

A few things worth being precise about:
- `apiVersion` and `kind` tell Kubernetes which resource type this block describes.
- The Deployment's `selector` and the Pod `template`'s `labels` must match — that's the mechanism connecting a Deployment to the Pods it manages.
- A Service's `port` (what clients connect to) and `targetPort` (what the container actually listens on) can differ — they don't have to be the same number, which trips people up the first time.
- `type: ClusterIP` means internal-only. `NodePort` and `LoadBalancer` exist for external access — worth knowing they exist, not needed in depth here.

---

## Part 4: `kubectl` — Talking to the Cluster

`kubectl` is the command-line tool you use to interact with a cluster.

- `kubectl apply -f file.yaml` — "make the cluster's actual state match what's described in this file." Declarative, like the manifests themselves — not "run this once," but "converge to this."
- `kubectl get pods` / `kubectl get deployments` / `kubectl get services` — list current resources and their status.
- `kubectl describe pod <name>` — detailed info on one specific resource; your first stop when something isn't working.
- `kubectl logs <pod-name>` — a Pod's container output, the same idea as `docker logs`.

---

## Coding Exercise

1. Install Minikube (a single-node local Kubernetes cluster) and confirm `kubectl get nodes` shows one Node, `Ready`.
2. Write a trivial Deployment + Service manifest for any simple image (`nginx` is fine) — a smaller version of Part 3's example.
3. `kubectl apply -f` it. Confirm with `kubectl get pods` that 3 Pods come up (or however many replicas you set).
4. `kubectl describe pod <one-of-them>` and read through the output — note what information is there.
5. Delete one Pod directly with `kubectl delete pod <name>` and immediately run `kubectl get pods` again — watch Kubernetes replace it automatically, unprompted. This single moment is the entire point of Part 1's "desired state" idea, made visible.

**Definition of done:** you can explain, out loud, without notes: the difference between a Pod and a container, what a Deployment actually guarantees, why a Service exists at all, and what you just watched happen when you deleted a Pod by hand.

---

## Quick Reference

| Term | One-line definition |
|---|---|
| Node | A single machine that's part of the cluster |
| Cluster | The full set of Nodes, managed as one unit |
| Pod | The smallest deployable unit — one or more containers scheduled together |
| Deployment | Describes and maintains a desired number of identical Pod replicas |
| ReplicaSet | What a Deployment creates/manages underneath to keep replica count correct |
| Service | A stable address that routes to whichever matching Pods are currently healthy |
| `kubectl apply -f` | Converge the cluster's actual state to match a manifest file |
| `kubectl get` | List current resources and their status |
| `kubectl describe` | Detailed info on one specific resource |
| `kubectl logs` | View a Pod's container output |

### Daily Deliverable
- [ ] Can explain why Kubernetes exists, beyond what Docker Compose already does.
- [ ] Can explain Pod vs. container, and what a Deployment guarantees.
- [ ] Can explain why a Service exists, separate from a Pod's own IP.
- [ ] Minikube exercise complete — watched a deleted Pod get automatically replaced.
- [ ] Ready for Day 96 with the vocabulary already in place, not being met for the first time.
