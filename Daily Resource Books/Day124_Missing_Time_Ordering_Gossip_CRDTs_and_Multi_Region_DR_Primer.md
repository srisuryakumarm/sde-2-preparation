# Day 124 — Missing Primer: Time and Ordering, Gossip, CRDTs, and Multi-Region / Disaster Recovery

**Do this first on Day 124 (≈ 60 min, before Distributed ID Generation), then continue with Day 124's regular plan.**
**Needed by:** Day 124 (Snowflake IDs depend on clocks), Day 125 (consensus and leader election), Day 127 (chat message ordering), Day 133 (Google Drive: sync, conflicts, multi-region) — and SDE-2 distributed-systems questions.
**Why it is missing:** vector clocks, Lamport timestamps, gossip, CRDTs and RPO/RTO are named only in passing (Days 125, 127, 133) and never taught.
**Already in place:** CAP, replication, consistent hashing (Days 59–61), sharding (Day 78), Saga/2PC (Days 70, 79).

---

## 1. There is no global clock
Machine clocks drift and jump (NTP corrections); two events on different machines cannot be ordered by wall-clock time reliably. Distributed systems therefore use **logical time**.

## 2. Lamport timestamps — a consistent order
Each process keeps a counter. **Local event or send:** `counter++` (send it with the message). **Receive:** `counter = max(counter, received) + 1`. If A *happened-before* B then `L(A) < L(B)` — but the reverse is **not** true, so Lamport time can't tell you that two events were *concurrent*.

## 3. Vector clocks — detecting concurrency
Each process keeps a **vector** of counters (one slot per process); on receive, take the element-wise max, then bump your own slot. Compare two vectors: all ≤ with one < → *A happened before B*; neither dominates → the events are **concurrent** (a real conflict). Dynamo-style stores use this to detect conflicting writes and hand the choice to the application or a merge rule. (**Hybrid logical clocks** combine wall-clock and a counter — used by several databases.)

## 4. Quorums
With N replicas, write to W and read from R: if **R + W > N**, every read overlaps the latest write. (N=3, W=2, R=2.) Lower W or R for speed and accept staleness (Day 59's CAP trade-off).

## 5. Gossip and failure detection
**Gossip:** each node periodically shares what it knows with a few random peers; information spreads **epidemically** in O(log N) rounds, with no central coordinator (Cassandra, Consul). **Failure detection:** heartbeats and timeouts — too short → false alarms, too long → slow recovery; **phi-accrual** detectors output a *suspicion level* instead of a yes/no.

## 6. CRDTs and operational transformation (collaborative state)
Concurrent edits without a central lock, then convergence:
- **CRDT** (conflict-free replicated data type): merge is commutative, associative and idempotent, so replicas converge in any order. Examples: **G-Counter** (one counter per replica, merge = element-wise max, value = sum), **PN-Counter**, **LWW-Register** (last write wins by timestamp), **OR-Set**.
- **OT** (operational transformation): transform concurrent operations against each other (classic Google Docs); needs a central ordering server.
- CRDTs trade metadata size for no coordination; used in collaborative editors, offline-first apps, Redis Enterprise.

## 7. Multi-region and disaster recovery
- **RPO** (recovery point objective): how much data you can afford to lose (e.g. 5 min). **RTO** (recovery time objective): how long you can be down (e.g. 30 min). Backups and async replication bound RPO; failover automation bounds RTO.
- **Active-passive:** one region serves, another stands by (simpler, wastes capacity, failover takes time). **Active-active:** all regions serve (low latency, resilience) but need **conflict resolution** (LWW, CRDTs, partition users to a home region).
- Cross-region replication is **asynchronous** (round trips ~100 ms) — so failover can lose recent writes. Route with DNS/latency routing (Day 73's file); test failover regularly.

## 8. Exercise (≈ 30 min, paper + Java)
1. Three processes P1–P3 exchange six messages (invent them): assign **Lamport** timestamps, then **vector clocks**; find one pair of concurrent events.
2. Implement a **G-Counter** in ~20 lines (`increment(replicaId)`, `merge(other)`, `value()`); merge three replicas in different orders and show they converge.
3. A product-catalogue service must survive losing a region: choose RPO and RTO, active-passive or active-active, and say what happens to in-flight orders.
4. In two sentences: why can't Snowflake IDs (Day 124) rely on wall-clock time alone, and what do they add?

## Checklist
- [ ] Can apply the Lamport and vector-clock rules and say what each can/can't prove.
- [ ] Can state R + W > N and what it buys.
- [ ] Can explain gossip, a G-Counter, and CRDT vs OT.
- [ ] Can define RPO/RTO and contrast active-passive with active-active.
- [ ] Continue with Day 124's regular plan.
