# Day 79 — Missing Primer: CQRS and Event Sourcing

**Do this first on Day 79 (≈ 45 min, alongside 2PC vs Saga), then continue with Day 79's regular plan.**
**Needed by:** Day 80 (Kafka Streams builds read models from event streams), Day 130 (payments as an event ledger), microservice and HLD interviews (both terms are standard vocabulary).
**Why it is missing:** "CQRS" and "event sourcing" appear nowhere in the repo, though Saga (Day 70, 79), the outbox (Day 69's file) and Kafka (Days 48–80) lead straight to them.
**Already in place:** Saga and 2PC (Days 70, 79), Kafka and events (Days 48–50, 69), sealed types and `switch` patterns (Day 28), the outbox (Day 69's file).

---

## 1. CQRS — Command Query Responsibility Segregation
Use **different models** for changing data (**commands**: `PlaceOrder`, validated against business rules) and reading it (**queries**: `OrderSummary`, shaped for the screen). Often the read model is a **denormalized view**, updated by events from the write side, in its own store.
- **Why:** reads and writes have different shapes and scale (reads usually dominate); a read model avoids joins and N+1 (Day 49's file); each side scales independently.
- **Cost:** the read side is **eventually consistent** with the write side; more moving parts. Plain CRUD (your `todo-api`) does **not** need CQRS.

## 2. Event sourcing
Instead of storing the *current state*, store the **sequence of events that led to it** (`OrderPlaced`, `PaymentCompleted`, `OrderShipped`) in an append-only **event store**. Current state = **fold** the events. A **snapshot** every N events avoids replaying from the beginning.
- **Gains:** complete audit trail, time travel ("what did it look like last Tuesday?"), easy to build new read models by replaying, natural fit for event-driven systems.
- **Costs:** events are forever, so **schema evolution** needs care (Avro/Schema Registry, Day 69); every query needs a projection; "delete my data" (GDPR) is hard (crypto-shredding); a steeper learning curve.
- Event sourcing and CQRS are **independent** — they are often combined because the event stream feeds the read models.

## 3. How it fits what you already built
| Piece | Role |
|---|---|
| Aggregate (Order) | validates a command, emits events |
| Outbox + Kafka | delivers the events reliably |
| Consumers / Kafka Streams `KTable` (Day 80) | build **read models** (projections) |
| Saga (Day 70, 79) | coordinates several aggregates by reacting to events |
| Idempotent consumers (Day 130) | make replays and redelivery safe |

## 4. Exercise (≈ 20 min, `java-fundamentals`)
```java
sealed interface OrderEvent permits OrderPlaced, PaymentCompleted, OrderShipped, OrderCancelled {}
record OrderPlaced(String orderId, long totalMinor) implements OrderEvent {}
record PaymentCompleted(String orderId) implements OrderEvent {}
record OrderShipped(String orderId) implements OrderEvent {}
record OrderCancelled(String orderId, String reason) implements OrderEvent {}

enum Status { PLACED, PAID, SHIPPED, CANCELLED }
record OrderState(String orderId, long totalMinor, Status status) {
    static OrderState apply(OrderState s, OrderEvent e) {
        return switch (e) {                                   // exhaustive over the sealed type
            case OrderPlaced p      -> new OrderState(p.orderId(), p.totalMinor(), Status.PLACED);
            case PaymentCompleted c -> new OrderState(s.orderId(), s.totalMinor(), Status.PAID);
            case OrderShipped sh    -> new OrderState(s.orderId(), s.totalMinor(), Status.SHIPPED);
            case OrderCancelled x   -> new OrderState(s.orderId(), s.totalMinor(), Status.CANCELLED);
        };
    }
}
```
1. Fold a list of events into the current state (`reduce` starting from `null`).
2. Build a **read model**: count orders per `Status` from many events (a projection).
3. Add a new event type and let the compiler show you every `switch` that must change.
4. In two sentences: what would you need to add before you could trust this in production (hint: idempotency, ordering per order id, snapshots, schema versions)?

## 5. Interview answers
*"Is Kafka an event store?"* — It can hold the log, but event sourcing also needs per-aggregate ordering, optimistic concurrency on append, and efficient load-by-aggregate — usually a dedicated event store or a carefully partitioned topic.
*"When would you not use event sourcing?"* — simple CRUD, no audit need, small team.

## Checklist
- [ ] Can define CQRS and event sourcing separately and say why they are often paired.
- [ ] Can list two benefits and two costs of event sourcing.
- [ ] Can say where the outbox, Saga and Kafka Streams fit.
- [ ] Exercise folded and projected. Continue with Day 79's regular plan.
