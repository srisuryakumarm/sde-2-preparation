# Day 61 — Missing Primer: What a Microservice Is, and Why the Next Ten Days Exist

**Do this first on Day 61 (≈ 75 min), then continue with Day 61's regular plan.**
**Needed by:** Day 62 (you create four modules and the plan calls them "services"), Days 64–70 (circuit breaker, Feign, Gateway, JWT, rate limiting, Schema Registry, Saga), Day 79 (2PC vs Saga), Day 95 (Eureka), Day 137 (tracing).
**Why it is missing:** pieces appear — failure cascades on Day 64, "independent databases" on Day 70 — but no day explains what a microservice is, where the boundaries go, or why each later pattern exists. Without that, Days 64–70 look like seven unrelated tools.
**Already in place:** REST and Task CRUD (Days 34–41), Kafka basics (Days 48–50), CAP, consistent hashing and replication (Days 59–61).

---

## 1. Monolith vs microservices
| | One monolith | Several microservices |
|---|---|---|
| Deployable | one app | one app **per service**, each with its own `main`, port, and release |
| Data | one database | **each service owns its own data store** |
| Calls | method calls (fast, cannot half-fail) | network calls (slow, can fail, time out, or arrive twice) |
| Transactions | one DB transaction | none across services — Saga (Day 70, 79) |
| Debugging | one stack trace | one request spans many processes — tracing (Day 137) |
| Operations | one thing to run | many things to run, discover, configure, secure |

## 2. The three ideas that explain the rest
1. **A service owns its data.** No other service reads its tables or holds a foreign key into them. Order stores a `productId` and a **price snapshot** — never a join into Product's database. Others get data by asking (API) or by listening (events).
2. **Services talk over a network, in one of two ways.** *Synchronous* (REST/Feign): "I need the answer now to continue." *Asynchronous* (Kafka event): "something happened; whoever cares can react later."
3. **The network is unreliable.** Every remote call can be slow, fail, or be delivered twice. Every pattern in the next ten days is a response to one of those facts.

## 3. Your platform on one page
| Service (Maven module) | Owns | Calls (sync) | Emits / consumes (async) |
|---|---|---|---|
| `product-service` :8081 | products, stock | — | — |
| `order-service` :8082 | orders | Product (price/stock check) | emits `OrderCreated`; consumes `PaymentCompleted/Failed` |
| `payment-service` :8083 | payments, idempotency keys | — | consumes `OrderCreated`; emits `PaymentCompleted/Failed` |
| `notification-service` :8084 | notification preferences/status | — | consumes payment/order events |
| `api-gateway` :8080 (Day 66) | no data | routes to the services | — |
Local-machine rule: **one Postgres container with four databases** (one per service), not four Postgres containers.

## 4. Why each pattern exists (problem → pattern)
| Problem | Pattern | Day |
|---|---|---|
| a slow/failed dependency drags the caller down | circuit breaker + fallback | 64 |
| calling another service by hand-written HTTP code | declarative client (Feign) | 65 |
| clients should not know every service address | API gateway | 66 |
| who is calling, and are they allowed? | JWT at the gateway | 67 |
| one client can flood the system | rate limiting | 68 |
| services need a shared, evolvable message format | Avro + Schema Registry | 69 |
| no distributed transaction across services | Saga with compensation | 70, 79 |
| service addresses change | service discovery (Eureka) | 95 |
| the same message can arrive twice | idempotency keys | 130 |
| "where did this request go?" | distributed tracing | 137 |

## 5. Maven module ≠ running service
Day 62's multi-module project is a **build-time** grouping (one repo, one parent POM). At **run time** every module is its own Spring Boot process with its own port and database. "Running the platform" means several JVMs plus Postgres, Kafka and Redis — on a small machine, start only what the day needs.

## 6. The honest part (say this in interviews)
Microservices cost latency, partial failure, eventual consistency, harder debugging and a lot of local infrastructure. At the scale of a portfolio project a **modular monolith** would be simpler and faster. A strong answer to "why microservices here?": *"To practice the patterns real systems need — resilience, messaging, sagas, discovery, observability — and I can tell you what I would simplify at small scale and what would force the split."*

## 7. Exercise (≈ 30 min, on paper or in `docs/`)
1. Redraw §3 from memory as a diagram: boxes, sync arrows, event arrows.
2. For six interactions — Order→Product price check; Order→Payment; Payment→Notification; client→Order; "show product details fast"; "user clicks Pay twice" — write: sync or async? what happens if the other side is down? which pattern from §4 protects it?
3. Write the five-sentence answer to "why microservices for this project?" without notes.

## Checklist
- [ ] Can explain "a service owns its data" and what Order stores instead of a join.
- [ ] Can say sync vs async in one sentence each, with a platform example.
- [ ] Can map each of Days 64–70 to the problem it solves.
- [ ] Can say Maven module vs running service.
- [ ] Continue with Day 61's regular plan.
