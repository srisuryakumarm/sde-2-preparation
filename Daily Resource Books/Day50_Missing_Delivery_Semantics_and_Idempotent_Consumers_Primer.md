# Day 50 — Missing Primer: Delivery Semantics and Idempotent Consumers

**Do this first on Day 50 (≈ 50 min, before the Spring Kafka consumer work), then continue with Day 50's regular plan.**
**Needed by:** Day 69 (outbox — "at-least-once"), Day 70 and Day 79 (Saga steps must tolerate duplicates), Day 93 (retries and DLQ), Day 130 (idempotency keys), Day 131 (HLD answers).
**Why it is missing:** at-most-once / at-least-once / exactly-once are first taught as a heading only on Day 131, and idempotent consumers on Day 130 — yet Day 48 has no mention of delivery guarantees, and the Saga (Day 70) and retry/DLQ (Day 93) days depend on them.
**Already in place:** Kafka fundamentals and local setup (Days 47–48), producers/consumers and offsets (Days 48–50), the `Task` service on Postgres with `@Transactional` (Day 41's file).

---

## 1. Why duplicates (and losses) happen
- **Producer retry:** the broker stored the message but the acknowledgement was lost → the producer sends it again.
- **Consumer crash:** the handler finished its side effect but the process died before the **offset was committed** → the message is delivered again.
- **Rebalance, retry, DLQ replay, outbox relay (Day 69):** every one re-delivers.
- The opposite failure: commit the offset *first*, crash before handling → the message is **lost**.

## 2. The three guarantees
| Guarantee | How | Failure mode |
|---|---|---|
| **At-most-once** | commit the offset *before* processing | may **lose** messages |
| **At-least-once** | process, *then* commit (Spring Kafka's default: auto-commit off, commit after the listener returns) | may **duplicate** |
| **Exactly-once** | Kafka's *idempotent producer* (`enable.idempotence`, on by default in current clients) removes producer-retry duplicates within a partition; *transactions* (`transactional.id`, consumers with `isolation.level=read_committed`) make a consume-transform-produce step atomic | **only inside Kafka** |
Exactly-once does **not** cover side effects outside Kafka: a DB write, an email, a payment API call. For those, the practical goal is **effectively once = at-least-once delivery + idempotent processing**.

## 3. Idempotent consumers — making a duplicate harmless
An operation is **idempotent** if doing it twice has the same effect as doing it once.
1. **Naturally idempotent writes:** `SET status = 'PAID'`, an **upsert**, `INSERT … ON CONFLICT DO NOTHING` — not `balance = balance + 10`.
2. **Dedupe table:** each event carries a unique `eventId`. In **one transaction** insert it into `processed_events(event_id PRIMARY KEY)` *and* apply the business change; a duplicate hits the primary-key violation and is skipped.
3. **State-machine guard:** only apply `PAYMENT_COMPLETED` if the order is `PENDING` — a replay finds it already `PAID` and does nothing.
4. **Version/sequence check:** ignore an event older than the current version (also protects against reordering).
5. **Idempotency keys at the API** (Day 130): the client sends a unique key; the server stores key → response and returns the stored result on a repeat.

## 4. Ordering and retries
Kafka orders **within a partition** only: use the aggregate id (e.g. `orderId`) as the **message key** so all events of one order go to one partition. Retries and parallel consumers can still re-order or repeat, so handlers should tolerate **duplicates and out-of-order** events. Separate **retryable** failures (timeout) from **poison messages** (bad payload) that go to a **DLQ** (Day 93) — and remember a replay from the DLQ is another duplicate.

## 5. Exercise (≈ 25 min)
1. Paper table: four events (producer timeout, consumer crash before commit, crash after commit but before handling, outbox relay restart) × (at-most-once, at-least-once): lost, duplicated, or fine?
2. In `todo-api`'s Kafka listener (Day 50) add `processed_events(event_id PRIMARY KEY)` via Flyway; handle each event in one `@Transactional` method that inserts the id and applies the change. Publish the **same** event twice with the console producer; prove one effect and a skipped duplicate.
3. In three sentences: why does a Kafka transaction not make "send a confirmation email" exactly-once, and what would make it safe?

## Checklist
- [ ] Can define the three guarantees and say which one Spring Kafka gives by default.
- [ ] Can name three ways to make a consumer idempotent and write the dedupe-table pattern.
- [ ] Can explain why Kafka's exactly-once stops at Kafka's boundary.
- [ ] Can say why the message key matters for ordering. Continue with Day 50's regular plan.
