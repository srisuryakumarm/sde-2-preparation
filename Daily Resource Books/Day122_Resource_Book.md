# Day 122 Resource Book — Notification System, and HLD Mock #1

**Series:** SDE-2 Interview Prep Resource Books · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 121](./Day121_Resource_Book.md) · **Next ▶:** [Day 123](./Day123_Resource_Book.md)
**Companion to:** Day 122 of `Week_18_Revised.md`

---

## Recap: what today is

Two systems and two days of the 5-step framework are now behind you, applied to URL Shortener (Day 120) and formalized further for the rate limiter (Day 121). Today applies the same framework to a third, genuinely different kind of system — one whose hard problem isn't storage or throughput, but **fan-out to unreliable third parties** (push, email, and SMS providers you don't control, each with its own failure modes) — and then, for the first time this phase, that design gets pressure-tested out loud, against another person, on a clock. HLD Mock #1 isn't new theory; it's the first real check on whether Day 120's framework survives contact with actual interview pressure.

## Learning Objectives

By the end of today, without notes:

1. Design a Notification System's high-level architecture, and justify per-channel queues as a deliberate choice, not a default.
2. Explain the deduplication check, including precisely why its race window is acceptable here when the identical race would not be acceptable for a payment.
3. Design a database schema covering per-channel user preferences and per-message delivery status.
4. Run a full 5-step framework walkthrough out loud, under a 45-minute clock, narrating every step explicitly rather than letting them blur together.

## Concept Dependency Map

```
Prerequisites, confirmed already covered:
  The 5-step HLD framework .............. Day 120 (applied a second time, Day 121)
  DLQ pattern ............................ Week 14, Day 93
  Kafka (producer/consumer/listener) ..... Week 7-8 · Week 10
  Redis, borrowed narrowly ............... Day 121 (EXPIRE, key existence checks)
  SQL / schema design .................... Week 6, Day 40
        │
        ▼
NEW today:
  Notification Service architecture — per-channel queues, a Template Service
        │
        ├─▶ Deduplication (Redis, best-effort — contrasted against payment
        │    idempotency, Week 19, Day 130, deliberately NOT taught yet)
        │
        └─▶ Retry → DLQ, applying Day 93's pattern to a new domain (not re-taught)
        │
        ▼
Applied: Notification System, then reframed as Confluence's page-watch feature
        │
        ▼
HLD Mock #1 (NEW format for this phase): the 5-step framework under real time pressure
```

---

## Part 1 — Requirements

- **Channels in scope:** Push, Email, SMS — each backed by a different third-party provider (FCM/APNs, SendGrid/SES, Twilio, respectively), each with its own latency, rate limits, and failure characteristics. That heterogeneity is the single fact that shapes most of today's design.
- **User preferences per channel:** a user might want push for "order shipped" but no SMS for anything promotional — preferences need to be tracked per `(user, notification type, channel)` triple, not as one global on/off switch.
- **Priority levels:** a security alert and a marketing blast don't deserve the same retry aggressiveness or the same queue treatment — worth naming as a requirement even if today's design doesn't build a full priority-queue system around it.

---

## Part 2 — High-Level Design

```
                 ┌───────────────────┐
Other services   │  Notification      │
(Order, Payment) │     Service        │
   ───request───▶│  (entry point)     │
                 └─────────┬──────────┘
                            │ looks up preferences, renders via Template Service
                            ▼
              ┌─────────────┬─────────────┬─────────────┐
              │ Push Queue  │ Email Queue │  SMS Queue   │   ◀── separate, deliberately
              └──────┬──────┴──────┬──────┴──────┬──────┘
                     ▼             ▼              ▼
              ┌───────────┐ ┌───────────┐  ┌───────────┐
              │ Push      │ │ Email     │  │ SMS       │
              │ Worker    │ │ Worker    │  │ Worker    │
              │ (FCM/APNs)│ │(SendGrid) │  │ (Twilio)  │
              └─────┬─────┘ └─────┬─────┘  └─────┬─────┘
                    │  failure, after retries      │
                    ▼             ▼                ▼
              ┌─────────────────────────────────────────┐
              │        Per-channel DLQ (Day 93)           │
              └─────────────────────────────────────────┘
```

**Why per-channel queues, specifically — the plan states this, worth actually proving:** put all three channels on one shared queue, and imagine the SMS provider (Twilio) starts timing out under its own load. Every worker pulling from that shared queue is now stuck waiting on slow SMS sends — including the ones that should be delivering push notifications in milliseconds. This is **head-of-line blocking**: one channel's degradation stalls every other channel behind it in the same line, even though push and SMS share nothing about *why* SMS is slow. Separate queues give each channel an independent failure domain — Twilio having a bad day has zero effect on push or email throughput.

**The Template Service:** notification content (`"Your order {orderId} has shipped!"`) lives in one place, parameterized, rather than hardcoded at every call site that wants to send a notification. Two concrete benefits worth stating, not just asserting: a marketing copy change ships without a code deploy, and every call site producing the same notification type stays consistent by construction — there's no way for two different services to accidentally send subtly different wording for the same event.

---

## Part 3 — Deduplication

**The problem:** a retry, a duplicate event from an at-least-once message queue (Week 8, Day 50's exact failure mode), or a bug upstream can trigger the same notification twice. Sending a user the same "your order shipped" push twice in five minutes is a bad experience worth actively preventing — a **retry storm**.

**The mechanism:**

```java
String dedupKey = "dedup:" + userId + ":" + notificationType + ":" + contentHash;
boolean alreadySent = redis.exists(dedupKey);
if (!alreadySent) {
    redis.set(dedupKey, "1", Duration.ofMinutes(N));   // TTL: the dedup window
    sendNotification(...);
} else {
    // skip — already sent recently
}
```

**A precise, deliberate limitation, worth stating out loud rather than glossing over:** the `exists` check and the `set` call above are **two separate round trips**, not one atomic operation — there's a small race window where two near-simultaneous sends could both see `alreadySent = false` before either has written the key. **This is acceptable here, specifically:** the cost of an occasional duplicate notification slipping through is mild annoyance, not a correctness failure — nobody's money or data integrity is at stake. Contrast this directly with a **payment** request, where that exact same race is not tolerable, and a genuinely atomic check-and-set is required — that's Week 19, Day 130's job, covered there in full rather than here, since the correctness bar for a payment is categorically stricter than for a notification, and building the heavier, fully-atomic mechanism here would be solving a problem this system doesn't actually have.

> 💡 **Interview Insight:** naming *why* a lighter-weight, best-effort mechanism is the right call here — not just defaulting to "add a lock because concurrency" — is a genuine signal of judgment. The strongest version of this answer explicitly states the cost of the rare failure case and argues it's acceptable, rather than reflexively reaching for the heaviest correct-looking tool.

---

## Part 4 — Retry Queues and the DLQ

**Recap, not a re-teach:** the Dead Letter Queue pattern (Week 14, Day 93) — a message that fails processing after a bounded number of retries gets routed to a separate queue/topic for inspection, rather than blocking the main queue indefinitely or being silently dropped. Today applies the identical mechanism to a new domain: a push/email/SMS send that keeps failing (provider outage, invalid device token, bounced email address) retries a bounded number of times, then lands in that channel's own DLQ rather than stalling every other message behind it in the same worker's queue — the same head-of-line-blocking reasoning from Part 2, now applied to failure handling specifically instead of cross-channel contention.

---

## Part 5 — Database Schema (Coding Exercise)

**Statement:** design a schema tracking (a) which channels a user has enabled for which notification types, and (b) the delivery status of every individual notification sent.

```sql
CREATE TABLE notification_preferences (
    user_id             BIGINT       NOT NULL,
    notification_type   VARCHAR(50)  NOT NULL,   -- e.g. 'ORDER_SHIPPED', 'PROMOTIONAL'
    channel             VARCHAR(20)  NOT NULL,   -- 'PUSH' | 'EMAIL' | 'SMS'
    enabled             BOOLEAN      NOT NULL DEFAULT TRUE,
    PRIMARY KEY (user_id, notification_type, channel)
);

CREATE TABLE notification_log (
    id                  BIGINT       PRIMARY KEY,
    user_id             BIGINT       NOT NULL,
    notification_type   VARCHAR(50)  NOT NULL,
    channel             VARCHAR(20)  NOT NULL,
    status              VARCHAR(20)  NOT NULL,   -- 'PENDING' | 'SENT' | 'FAILED' | 'DEAD_LETTERED'
    retry_count         INT          NOT NULL DEFAULT 0,
    created_at          TIMESTAMP    NOT NULL,
    sent_at             TIMESTAMP    NULL
);
```

**Why the three-column composite primary key on `notification_preferences`, specifically:** a preference is meaningless without all three dimensions together — "enabled" isn't a property of a user alone, or a notification type alone; it's a property of the *combination*. The composite key also makes an upsert (`ON CONFLICT ... DO UPDATE`, Week 6's SQL material) the natural way to change a preference, and structurally prevents two contradictory rows for the same `(user, type, channel)` triple from ever existing.

**Why `status` is a small fixed set of states rather than a single boolean `sent` flag:** a boolean can only distinguish "sent" from "not sent" — it can't represent "currently retrying" versus "gave up and dead-lettered," which are operationally very different states a support engineer needs to be able to query for separately (e.g., "show me everything stuck in DEAD_LETTERED for the SMS channel in the last hour"). This is the same instinct that made Vending Machine's State pattern (Week 16, Day 109) the right model for a small, named set of distinct states — here expressed as a column's value set rather than a class hierarchy, since this is data being queried, not behavior being dispatched.

**Edge cases:** a user with no row at all in `notification_preferences` for a given `(type, channel)` — the application layer needs a defined default (commonly: opt-in for critical/transactional types like order updates, opt-out by default for promotional ones), since an *absent* row is not the same as an explicit `enabled = false`; a notification that exhausts retries mid-send (correctly lands at `DEAD_LETTERED`, not stuck at `FAILED` forever); `sent_at` staying `NULL` for anything not yet `SENT`, which is what makes "how long has this been pending" a simple, direct query.

---

## Part 6 — Company-Flavored Variant: Confluence's Page-Watch Notifications (10 min)

Atlassian's system design round often asks for a design of one of their real products, not a generic prompt. Reframe today's system: **"design Confluence's page-watch notification system."** Same underlying architecture — a service, per-channel workers, a template layer — but the follow-up question sharpens it: **"what if a page has 10,000 watchers and gets edited every minute?"**

This is worth recognizing precisely for what it's exposing, without solving it fully today: one edit event now needs to become up to 10,000 individual notification sends — a **fan-out** problem, where the cost of a single write scales with the number of interested readers, not with the write itself. That's a genuinely different shape from anything built so far this week, and it gets its full, dedicated treatment — push-on-write vs. pull-on-read vs. a hybrid — in Week 19, Day 129 (Instagram's feed has the identical shape, at a much larger scale). Today's job is just recognizing *that* this is the harder version of the problem the moment "10,000 watchers, every minute" is on the table — practicing the reframe itself is what this ten minutes is actually for.

---

## HLD Mock #1

**Format:** 45 minutes, with your accountability partner, using **URL Shortener** (Day 120) as the subject — a system you already fully designed, deliberately, so today's mock isolates *how* you communicate the design under a clock from *whether* you can produce the design at all.

**The one thing being evaluated today, stated precisely, per the plan:** do the 5 steps stay narrated and distinct out loud, in order, or do they blur together once the clock is actually running? This is a different skill from having the design in your head — plenty of candidates who can produce a correct URL shortener design silently still lose real interview points by never actually *saying* "Step 1, requirements" before diving in.

### A Calibration Run-Through

What staying on the rails sounds like, condensed:

> "I'll walk through this in five steps: requirements, estimation, high-level design, detailed design, and bottlenecks. **Step 1, requirements** — functionally, I need to shorten a URL and redirect on access... [Day 120's actual requirements]. Moving to **Step 2, estimation** — at 100 million new URLs a day, that's..." *(does the actual division out loud)* "...roughly 1,200 writes a second. **Step 3, high-level design** — a Key Generation Service, a cache in front of the mapping database, since reads will outnumber writes here about 10 to 1..." — and so on, each transition **named explicitly**, not implied.

### Common Failure Patterns — What Your Partner Should Listen For

- **⚠️ Requirements bleeding directly into High-Level Design**, with Estimation skipped or reduced to a single unexplained number — the single most common way this specific failure shows up under time pressure.
- **⚠️ Detailed Design that never actually goes deeper than High-Level Design already did** — restating the same box-and-arrow description in more words isn't Step 4; Step 4 needs one specific mechanism (today's calibration example: the KGS's `SKIP LOCKED` claim) explained at implementation depth.
- **⚠️ Bottlenecks discussed with no connection back to Step 2's numbers** — "add a cache" without reconnecting to "because reads are ~12,000/sec and writes are ~1,200/sec" is a much weaker answer than one that visibly closes the loop back to the estimation step.

### Debrief Checklist

- [ ] Did every one of the 5 steps get named out loud, explicitly, before being discussed?
- [ ] Was the estimation math actually spoken, not just a final number stated?
- [ ] Did Detailed Design go meaningfully deeper than High-Level Design, on one specific piece?
- [ ] Did Bottlenecks connect back to a specific number from Step 2?
- [ ] Partner's overall read: did the steps blur together anywhere under time pressure?

---

## Project Block

**Repository:** `scalable-ecommerce-platform`. **Task:** commit the schema from Part 5 above as an actual database migration (Flyway, Week 6, Day 40). **Definition of done:** migration committed, matching the design above exactly — column names, types, and the composite primary key.

## Career Block

**LinkedIn Post 23 — WhatsApp-style notification architecture breakdown.** A structure that reuses today's own material directly: open with the per-channel-queue decision and the head-of-line-blocking reasoning behind it (a concrete "why," not just an architecture diagram description) — posts that explain a *reason*, not just a shape, tend to land better than a diagram dump.

**Networking:** target SDE-2s specifically at Rippling, Google, Databricks, Stripe, Uber, Atlassian, and Walmart Global Tech on LinkedIn; send 5 connection requests. Check application status on anything submitted Day 118 — this is roughly the window a first response tends to arrive in; respond promptly if anything comes back.

## Daily Deliverable Check

- [ ] Notification System design complete, including the schema, with per-channel queues justified (not just described) and deduplication's race-window trade-off explicitly stated.
- [ ] Schema migration committed, matching the design.
- [ ] HLD Mock #1 completed and debriefed against the checklist above.
- [ ] LinkedIn Post 23 published. 5 connection requests sent. Day 118 applications checked.

---

## Day 122 — Interview Questions

---

**1. Why does each notification channel get its own queue instead of one shared queue across Push, Email, and SMS?**

*Answer:* To avoid head-of-line blocking — if one channel's provider degrades or slows down, a shared queue would stall every other channel's messages behind it, even though the other channels have nothing to do with that provider's problem. Separate queues give each channel its own independent failure domain.

---

**2. What does the Template Service architecturally solve, beyond "keeping code DRY"?**

*Answer:* It centralizes notification copy in one place, so a wording change ships without a code deploy, and every call site producing the same notification type is guaranteed consistent — there's no path for two services to independently drift on the wording for the same event.

---

**3. Walk through the deduplication check, and name its precise limitation.**

*Answer:* Check whether a Redis key for this `(user, type, content)` combination exists; if not, set it with a TTL and send; if it exists, skip. The check and the set are two separate round trips, so a small race window exists where two near-simultaneous sends could both pass the check before either writes the key.

---

**4. Why is that race window acceptable for notifications but would not be acceptable for a payment request?**

*Answer:* An occasional duplicate notification costs mild user annoyance — no money or data integrity is at risk. The identical race in a payment context could double-charge a customer, a genuinely unacceptable failure, which is why payments need a fully atomic check-and-set instead (Week 19, Day 130).

---

**5. What is a Dead Letter Queue, and why does a failed notification send go there rather than being retried forever or dropped silently?**

*Answer:* A DLQ is a separate queue that a message routes to after exhausting a bounded number of retries, so it can be inspected and reprocessed manually rather than blocking the main queue indefinitely or vanishing without a trace. First established Week 14, Day 93; applied here to notification sends specifically.

---

**6. Why is `(user_id, notification_type, channel)` the right composite primary key for the preferences table, rather than a single surrogate key?**

*Answer:* "Enabled" is a property of that exact combination, not of any one column alone — the composite key structurally prevents two contradictory rows for the same combination from existing, and makes an upsert the natural way to change a preference.

---

**7. Why model `status` as a small set of named states rather than a single boolean `sent` flag?**

*Answer:* A boolean can only distinguish sent from not-sent — it can't represent "currently retrying" versus "exhausted retries and was dead-lettered," which are operationally different states that need to be queryable separately.

---

**8. A user has no row at all in `notification_preferences` for a given (type, channel) pair. What should the system do, and why can't the schema alone answer this?**

*Answer:* The application layer needs a defined default — typically opt-in for critical/transactional notification types and opt-out by default for promotional ones — because an absent row is not the same as an explicit `enabled = false`, and the schema itself has no way to express "what should happen when nothing is recorded."

---

**9. In the Confluence reframe, what specifically makes "10,000 watchers, edited every minute" a harder version of the notification problem?**

*Answer:* It's a fan-out problem — one edit event now has to become up to 10,000 individual notification sends, so the cost of a single write scales with the number of interested readers rather than with the write itself. Full treatment (push vs. pull vs. hybrid fan-out): Week 19, Day 129.

---

**10. What specific thing is HLD Mock #1 designed to test, distinct from whether the design itself is correct?**

*Answer:* Whether the 5-step framework stays explicitly narrated, in order, under real time pressure — a candidate can have a fully correct design in mind and still lose interview points by never actually stating which step they're on.

---

**11. Name one common failure pattern this mock is specifically watching for.**

*Answer:* Requirements bleeding directly into High-Level Design with Estimation skipped or reduced to an unexplained number — the most common way narration breaks down under time pressure, per today's debrief checklist.

---

**12. Why might Push, Email, and SMS need genuinely different retry/backoff behavior, not just separate queues?**

*Answer:* Each provider has different failure characteristics — a push provider's transient failure might resolve in seconds, while an email bounce is often permanent and retrying it is pointless. Treating all three with identical retry logic ignores information the failure type itself is providing.

---

**13. Why does Detailed Design in this mock need to go deeper than restating the High-Level Design in different words?**

*Answer:* Detailed Design's whole purpose is proving depth on the one piece that matters most — restating the box-and-arrow diagram verbally isn't a level deeper, it's the same level again; a real answer picks one specific mechanism (e.g., the KGS's atomic claim) and explains it at implementation depth.

---

## What Tomorrow Assumes You Already Know Cold

Day 123 assumes Redis's narrow role today — a key with a TTL, an existence check — is comfortable enough that tomorrow's much fuller Redis picture (data structures, Memcached contrast, persistence, pub/sub) lands as a natural expansion rather than a cold start. It also assumes Consistent Hashing (Week 9, Day 60) is genuinely reflexive, not just recognized by name, since tomorrow reuses it directly rather than re-deriving it.

**Next:** [Day 123 Resource Book](./Day123_Resource_Book.md) — Distributed Cache.
