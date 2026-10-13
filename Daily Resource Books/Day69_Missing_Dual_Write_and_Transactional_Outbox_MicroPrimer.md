# Day 69 — Missing Primer: The Dual-Write Problem and the Transactional Outbox

**Do this first on Day 69 (≈ 30 min), then continue with Day 69's regular plan.**
**Needed by:** Day 70 (Saga choreography starts with "save the order, then publish `OrderCreated`"), Day 79 (compensation), Day 130 (idempotency), and Day 48's `TaskCreatedEvent`. Day 130 mentions the outbox only as extension material.
**Already in place:** `@Transactional` and its limits (Day 41), Kafka (Days 48–50), Schema Registry/Avro (Day 69), CAP and replication (Days 59–61).

## 1. The problem
`order-service` must do two things when an order is placed: **write a row to its database** and **publish `OrderCreated` to Kafka**. They are two different systems; no single transaction covers both (`@Transactional` covers only the database — Day 41).
- DB commit succeeds, publish fails (crash, broker down) → the order exists but **nobody ever hears about it**; the Saga never starts.
- Publish succeeds, DB transaction then rolls back → an event describes an order that **does not exist**.
Reordering the two calls only swaps which failure you get.

## 2. The fix: Transactional Outbox
Write the business row **and** an "event to send" row in the **same database transaction**; a separate relay publishes outbox rows to Kafka afterwards.
```sql
CREATE TABLE outbox_event (
  id            UUID PRIMARY KEY,
  aggregate_id  VARCHAR(64)  NOT NULL,   -- e.g. the order id; also the Kafka message key (keeps per-order ordering)
  type          VARCHAR(64)  NOT NULL,   -- 'OrderCreated'
  payload       JSONB        NOT NULL,
  created_at    TIMESTAMPTZ  NOT NULL DEFAULT now(),
  published_at  TIMESTAMPTZ              -- NULL = not yet published
);
```
```java
@Transactional
public Order create(CreateOrderRequest req) {
    Order order = orderRepository.save(Order.from(req));
    outboxRepository.save(OutboxEvent.of("OrderCreated", order.getId(), toJson(order)));   // same transaction
    return order;                                      // one commit: both rows exist, or neither does
}

@Scheduled(fixedDelay = 1000)                          // the relay (@Scheduled is taught on Day 131)
public void publishPending() {
    for (OutboxEvent e : outboxRepository.findTop50ByPublishedAtIsNullOrderByCreatedAt()) {
        kafkaTemplate.send("order-events", e.getAggregateId(), e.getPayload()).get();   // wait for the broker's ack
        e.markPublished();                              // persisted afterwards
    }
}
```
If the relay crashes after sending but before marking, the event is sent again. So the guarantee is **at-least-once** — consumers must be **idempotent** (Day 130).

## 3. The alternatives (be able to name them)
| Option | Verdict |
|---|---|
| Outbox + polling relay (above) | simplest correct answer; adds a table and a poller |
| Outbox + CDC (e.g. Debezium reads the database log) | no polling, lower latency; more infrastructure |
| Kafka transactions | atomic across Kafka topics only — not the database |
| 2PC / XA across DB and broker | rejected on Day 79 (blocking, availability cost) |

## 4. Where this plan stands
Days 48 and 70 publish directly after the save — fine for learning, but write it down in the README as a **known limitation** ("dual write; outbox is the fix"). Optional: implement the polling outbox on the Day 70A buffer day (≈ 1.5 h). Either way you can now answer the standard interview question: *"What if the service crashes after saving but before publishing?"*

## Checklist
- [ ] Can describe the two failure windows without notes.
- [ ] Can explain the outbox in four sentences and why delivery becomes at-least-once.
- [ ] Can name CDC, Kafka transactions and 2PC and say why each is or is not the answer.
- [ ] Continue with Day 69's regular plan.
