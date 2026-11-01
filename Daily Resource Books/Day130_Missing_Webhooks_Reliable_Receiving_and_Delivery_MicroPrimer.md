# Day 130 — Missing Primer: Webhooks — Receiving and Delivering Them Reliably

**Do this first on Day 130 (≈ 30 min, before the Payment System design), then continue with Day 130's regular plan.**
**Needed by:** Day 130 (the payment design — see "why it is missing"), Day 122 (a notification system also *sends* to customer endpoints), Day 131 (the reconciliation job that backstops lost webhooks), practical coding rounds (Day 108's primer lists the round shapes; an "integrate this payments API and handle its callbacks" task is the Stripe-style variant), and **Stripe-style loops generally, where webhooks are the core integration surface** (your Day 130 follow-ups mention Stripe by name).
**Why it is missing:** the word "webhook" appears in two books in passing (a Kafka DLQ project hint on Day 93 and Kubernetes admission webhooks on Day 138 — a different thing). **Day 130's HLD treats the external payment gateway as a synchronous call** ("Client → Payment Service → external Payment Gateway → async reconciliation") and never explains how a real provider reports the **final** result — by calling *you* back — nor what to do when that callback is duplicated, late, out of order, forged, or never arrives.
**Already in place:** idempotency keys and at-least-once delivery (Day 50's primer, Day 130's Part 2), retries with backoff and jitter (Day 64's file), HMAC vs RSA signing at a conceptual level (Day 67's JWT primer), the transactional outbox (Day 69's primer), HTTP status codes and idempotent methods (Day 72's file).

---

## 1. What a webhook is
A **webhook is an HTTP callback**: instead of you *polling* "has my payment finished yet?", the provider **POSTs a JSON event to a URL you registered** the moment something happens (`payment.succeeded`, `refund.created`). The URL belongs to **you** (receiving) or to **your customers** (delivering — you become the provider). In real systems you use **both**: webhooks for speed, and a **reconciliation poller** as the safety net for lost ones (Day 131).

## 2. Receiving webhooks — five rules
1. **Verify authenticity.** Anyone on the internet can POST to your URL. The provider signs each request: **HMAC-SHA256** over `"<timestamp>.<raw request body>"` with a shared secret, sent in a header. You recompute it and compare **in constant time**. Include the **timestamp** in the signed text and reject requests older than ~5 minutes (**replay protection**).
```java
static boolean verify(String timestamp, byte[] rawBody, String signatureHex, byte[] secret, long toleranceSeconds) throws Exception {
    long ts = Long.parseLong(timestamp);
    if (Math.abs(Instant.now().getEpochSecond() - ts) > toleranceSeconds) return false;     // replay window
    Mac mac = Mac.getInstance("HmacSHA256");
    mac.init(new SecretKeySpec(secret, "HmacSHA256"));
    mac.update((timestamp + ".").getBytes(StandardCharsets.UTF_8));
    byte[] expected = mac.doFinal(rawBody);                                                  // HMAC over "<ts>.<raw body>"
    byte[] actual = HexFormat.of().parseHex(signatureHex);
    return MessageDigest.isEqual(expected, actual);                                          // constant-time compare
}
```
(Compiled and tested: a valid signature passes; a changed body, or a one-hour-old timestamp, fails. The "timestamp + dot + body" shape is how Stripe's documented scheme works; each provider's header names and encoding differ — always read theirs.)
**Verify the *raw bytes*, before parsing.** In Spring take the body as `byte[]` / `String` (`@RequestBody byte[] raw`), verify, **then** parse the JSON — a framework that parses and re-serialises changes whitespace and key order, and the signature no longer matches.
2. **Acknowledge fast.** Providers wait only a few seconds, then treat a timeout or non-`2xx` as failure and **retry**. So: verify → **durably store the event** (insert into a `webhook_events` table or publish to a queue) → return `200` immediately → process **asynchronously**. Never do the business logic inside the HTTP handler.
3. **Be idempotent.** Delivery is **at-least-once**: expect duplicates. Keep a unique constraint on the provider's **event id**; a repeat is a no-op (Day 50's idempotent-consumer pattern).
4. **Do not rely on order.** `payment.failed` can arrive after `payment.succeeded`, or before your own "create payment" call has even returned. Make state changes **monotonic** (a state machine: `PENDING → PAID` allowed; `PAID → PENDING` ignored), **upsert by the provider's reference id**, and for anything that moves money **re-fetch the object from the provider's API** instead of trusting the payload.
5. **Secure the endpoint:** HTTPS only, secret kept out of the repo, rotate secrets, no sensitive data in logs.

## 3. Delivering webhooks (when *you* are the platform — Day 122, Day 131)
- Write the event in the **same transaction** as the business change (outbox, Day 69's primer); a **delivery worker** sends it — at-least-once.
- **Retry** failed deliveries with **exponential backoff and jitter** for hours or days (Day 64's file); stop after N attempts and show the customer a **"failed deliveries" page with manual replay**.
- **Sign every request** (§2); give each endpoint its own secret; support secret rotation (accept two secrets briefly).
- **Isolate customers:** a slow or dead customer endpoint must not block others — one queue/partition per endpoint or per tenant, a **timeout** (5–10 s) and a **circuit breaker** per endpoint (disable after repeated failure, notify the owner), and a rate limit so you cannot flood them.
- **SSRF protection:** customers choose the URL — **refuse to call private and internal addresses** (`127.0.0.1`, `10.x`, `169.254.169.254` cloud metadata), re-check after DNS resolution, don't follow redirects blindly. This is the classic security question about webhook systems.
- Version the event schema; include a unique `id`, `type`, `created` timestamp and the object; document ordering as "not guaranteed".

## 4. How it completes Day 130's payment design
`POST /payments` carries an **Idempotency-Key** → your service creates a payment row `PENDING` and calls the provider (also with an idempotency key) → returns `202 Accepted` / `PENDING` to the client → the customer completes 3-D Secure or the bank transfer → the provider **webhooks** `payment.succeeded` → your idempotent handler moves `PENDING → PAID`, writes the **ledger entries** (Day 130 Part 4) and emits an outbox event → a **reconciliation job** polls the provider for any payment still `PENDING` after N minutes (the nightly `@Scheduled` task of Days 130–131). **The subtle race:** the webhook can arrive *before* your create-call returns, so the handler must **upsert by the provider's payment id**, not assume your row already exists.

## 5. Exercise (≈ 15 min, repo `java-fundamentals`)
1. Type `verify` above plus a `sign(...)` helper that produces the hex signature. Write four tests: valid passes; tampered body fails; timestamp one hour old fails; wrong secret fails.
2. Simulate two deliveries of the same event id (use a `Set<String>` standing in for a unique index): the second must not change state. Then deliver `payment.failed` *after* `payment.succeeded` and prove the state machine ignores it.
3. On paper: add the webhook receiver and a reconciliation job to Day 130's HLD arrow ("Client → Payment Service → Payment Gateway → async reconciliation") and say which failure each addition covers (lost webhook, duplicate, forged, out-of-order).

## Checklist
- [ ] Can say webhook vs polling and why you need both.
- [ ] Can list the five receiving rules (verify raw body + timestamp, ack fast, idempotent, order-tolerant, secure).
- [ ] Can write HMAC verification with a constant-time compare and a replay window.
- [ ] Can design delivery: outbox, backoff + jitter, per-endpoint isolation, replay UI, SSRF protection.
- [ ] Can describe the webhook-before-response race and the reconciliation backstop. Continue with Day 130's regular plan.
