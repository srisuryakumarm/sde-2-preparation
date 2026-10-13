# Day 68 — Missing Primer: Money in Java (`BigDecimal`), and Two Fixes to Day 69's Avro Schema

**Do this first on Day 68 (≈ 30 min), then continue with Day 68's regular plan.**
**Needed by:** Day 69 (the `Order` entity and its Avro event), Day 115 (Splitwise), Day 130 (payment idempotency).
**Why it is missing:** `BigDecimal` is never taught (Day 10 covers primitives only), yet Day 115 warns that using `double` for money "at all is a mistake". Day 69's `Order.avsc` then stores `totalPrice` as a `double`, and puts `"logicalType": "timestamp-millis"` at the wrong nesting level (Avro ignores it there, so `createdAt` is a plain `long`).
**Already in place:** primitives and floating point (Day 10), Task CRUD on Postgres (Days 35–41), `LocalDate`/`Instant` (Day 30).

## 1. Why `double` fails
`0.1 + 0.2` is `0.30000000000000004`. Binary floating point cannot represent most decimal fractions exactly, and the error accumulates across line items, taxes and refunds.

## 2. Two correct representations
| Where | Use |
|---|---|
| inside a service and its database | `BigDecimal` with a fixed scale; Postgres `NUMERIC(12,2)` |
| in messages and APIs between services | an integer in **minor units** (`long` paise/cents) |

## 3. `BigDecimal` rules
```java
BigDecimal price = new BigDecimal("199.99");                 // from a String — never new BigDecimal(0.1)
BigDecimal total = price.multiply(BigDecimal.valueOf(3))
                        .setScale(2, RoundingMode.HALF_UP);   // choose the rounding rule on purpose
price.compareTo(other) == 0                                   // compare VALUES; equals() also compares scale (2.0 != 2.00)
price.divide(divisor)                                         // throws ArithmeticException if the result does not terminate
price.divide(divisor, 2, RoundingMode.HALF_UP)                // always give a scale and rounding mode
```
JPA: `@Column(nullable = false, precision = 12, scale = 2) private BigDecimal totalPrice;` → `NUMERIC(12,2)`. Converting at the edge: `long minor = amount.movePointRight(2).longValueExact();` and `BigDecimal.valueOf(minor, 2)` back.

## 4. Fix Day 69's Avro schema
```json
{ "name": "totalPriceMinor", "type": "long" },                                   // was: "totalPrice" as double
{ "name": "createdAt", "type": { "type": "long", "logicalType": "timestamp-millis" } }   // logicalType goes INSIDE the type
```
(An Avro `decimal` logical type on `bytes` also works but needs conversion code; minor units are the simplest correct choice for the event contract.) Keep `BigDecimal` inside `order-service`; convert when building the event.

## 5. Exercise (≈ 20 min)
1. A 3-line invoice (e.g. 199.99 × 3, 49.50 × 2, 5.25 × 7) plus 18 % GST: compute the total with `double` and with `BigDecimal`; print both. Add a case where they differ.
2. Write `toMinor(BigDecimal)` and `fromMinor(long)` with tests, including a value like `0.015` (what should rounding do?).
3. Sketch the `Order` entity fields with `BigDecimal` and the `OrderCreated` Avro record with `long ... Minor` fields.

## Checklist
- [ ] Can explain why `double` is wrong for money and name the two correct representations.
- [ ] Can state three `BigDecimal` rules (String constructor, `compareTo`, explicit scale and rounding).
- [ ] `Order.avsc` corrected (minor units; nested `logicalType`).
- [ ] Continue with Day 68's regular plan.
