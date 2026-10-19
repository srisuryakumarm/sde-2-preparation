# Day 112 — Missing Primer (STRETCH): Domain-Driven Design Vocabulary and Hexagonal Architecture

**Do this first on Day 112 (Sunday; ≈ 45 min), then continue with Day 112's regular plan.** Skip if you are behind.
**Needed by:** LLD and microservice-design answers ("where is the service boundary?", "what is an aggregate?"), the platform's package structure (Day 62+), SDE-2 design discussions.
**Why it is missing:** "bounded context", "aggregate root", "value object" and "hexagonal/clean architecture" appear nowhere in the repo, though Days 61–70 draw service boundaries and Days 106–119 design domain models.
**Already in place:** SOLID and patterns (Days 6, 100–107), microservice orientation (Day 61's file), JPA relationships and aggregates in practice (Day 49's file), events (Day 79's file).

---

## 1. Strategic DDD (where the boundaries go)
- **Ubiquitous language:** developers and domain experts use the *same words* in conversation and code (`Order`, `Shipment`, `Refund`).
- **Bounded context:** a boundary inside which one model and one meaning of each word hold. "Product" means a catalogue entry in the catalogue context and a line item with a price snapshot in the order context. **A microservice boundary should align with a bounded context** — that is the rule of thumb behind Day 61's service map.
- **Context map:** how contexts relate (customer/supplier, shared kernel, **anti-corruption layer** — a translator that stops another model leaking in).

## 2. Tactical DDD (how to model inside one context)
| Concept | Meaning | Example |
|---|---|---|
| **Entity** | has an identity that persists as attributes change | `Order` (id) |
| **Value object** | no identity; equal if attributes are equal; immutable | `Money(amount, currency)`, `Address` — a Java **record** |
| **Aggregate** | a cluster of objects changed together and kept consistent as one unit | `Order` + its `OrderItem`s |
| **Aggregate root** | the only entry point; outsiders hold a reference to the root, never to inner parts | `Order` — callers use `order.addItem(...)`, not `item.setQuantity(...)` |
| **Repository** | loads and saves a whole aggregate | `OrderRepository` |
| **Domain service** | business logic that fits no single entity | `PricingService` |
| **Domain event** | a fact that happened | `OrderPlaced` (→ outbox/Kafka, Day 69/79) |
**Rules of thumb:** one transaction changes **one aggregate**; reference other aggregates **by id**, not by object (exactly why `OrderItem` holds a `productId`, Day 49); keep invariants ("an order has at least one item") inside the root.

## 3. Hexagonal architecture (ports and adapters)
Keep the **domain core** free of frameworks; talk to the world through **ports** (interfaces) implemented by **adapters**.
- **Inbound** adapters call into the core: REST controller, Kafka listener, scheduled job → they call *use-case* methods.
- **Outbound** ports are interfaces the core needs: `OrderRepository`, `PaymentGateway`, `EventPublisher`; adapters implement them with JPA, Feign, Kafka.
- The dependency arrow points **inward**: infrastructure depends on the domain, never the reverse (Day 6's Dependency Inversion, at architecture scale).
A pragmatic Spring layout for `order-service`: `api` (controllers, DTOs) · `application` (use cases, `@Transactional`) · `domain` (entities, value objects, ports) · `infrastructure` (JPA, Kafka, HTTP adapters). It is the layered design of Day 32 with the dependency rule made explicit; *clean architecture* is the same idea with more rings.
**Payoff:** domain logic unit-testable without Spring or a database; infrastructure is swappable. **Cost:** more types — don't force it on `todo-api`.

## 4. Exercise (≈ 20 min)
1. For the platform, list each bounded context and the words that mean different things in two of them.
2. Model `Order`: value objects (`Money`, `Address`), the aggregate root and its invariants; write `Order.addItem` that rejects a non-positive quantity.
3. Sketch the package structure for `order-service` and draw which package may import which.

## Checklist
- [ ] Can define bounded context, aggregate, value object and aggregate root with an example from the platform.
- [ ] Can explain why aggregates reference each other by id.
- [ ] Can describe ports and adapters and the inward dependency rule.
- [ ] Continue with Day 112's regular plan.
