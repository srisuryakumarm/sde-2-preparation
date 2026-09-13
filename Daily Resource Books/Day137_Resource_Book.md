# Day 137 — Distributed Tracing, For Real This Time

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 136 Resource Book](Day136_Resource_Book.md)
**Next ▶:** [Day 138 Resource Book](Day138_Resource_Book.md)
**Companion to:** Day 137 of `Week_20_Revised.md`

---

## Recap

The platform has had a working **metrics** pipeline since Week 11 — Micrometer instrumentation through `/actuator/prometheus`, scraped by Prometheus, visualized in Grafana (Days 76–77). Micrometer was established there specifically as **a facade** over a metrics backend. Today reuses that exact framing for a *second*, genuinely different Micrometer module — **Micrometer Tracing** — which is a facade over a *tracing* backend the same way the metrics module is a facade over Prometheus, but answers a different question entirely (Part 1 below is explicit about which). Today also reaches back to Spring Cloud Gateway (Week 10, Day 66) as where a trace begins, Kafka's pub/sub mechanics and Saga Choreography (Week 7, Day 48; Week 10, Day 70) for tracing's one genuine blind spot, and Idempotency Keys (Week 19, Day 130) for the afternoon's API design work.

---

## Learning Objectives

By the end of today, without notes:

1. Name the three pillars of observability and state, precisely, the different question each one answers.
2. Explain Trace ID and Span ID, how context propagates automatically across a synchronous HTTP call chain, and trace one request across all four modules by hand.
3. Explain exactly why trace context does **not** automatically survive a hop through Kafka the way it does an HTTP call, and what has to be done deliberately to fix that.
4. Defend, with a concrete argument for each, an API versioning strategy, a pagination approach, idempotency-key handling, and a structured error contract for a real endpoint.

---

## Concept Dependency Map

```
Week 11, Days 76-77: Micrometer (a facade) -> /actuator/prometheus ->
                      Prometheus -> Grafana — METRICS, already live
Week 10, Day 66: Spring Cloud Gateway — the platform's single external entry point
Week 7, Day 48: Kafka pub/sub, in-partition ordering guarantee
Week 10, Day 70: Saga Choreography, built directly on Kafka pub/sub
Week 19, Day 130: Idempotency Keys — client-generated key, check-before-act
        │
        ▼
Day 137
        │
        ├── Part A: Distributed Tracing
        │     ├─ Three pillars of observability — metrics vs. logs vs. traces
        │     ├─ Trace ID / Span ID / parent-child spans
        │     ├─ Automatic HTTP context propagation, traced end to end
        │     ├─ Micrometer Tracing + Zipkin (storage + waterfall UI)
        │     ├─ MDC log correlation — trace ID auto-tagging log lines
        │     ├─ Sampling rate trade-off
        │     └─ ⚠️ Context does NOT auto-propagate across Kafka —
        │           Saga's one genuine tracing blind spot
        │
        └── Part B: API Design Practice (Stripe-specific add-on)
              ├─ Versioning: URI-based vs. header-based
              ├─ Pagination: offset vs. cursor — a concurrent-insert
              │     failure traced concretely, not asserted
              ├─ Idempotency-key handling, applied directly from Day 130
              └─ A structured error contract vs. status-code-only
        │
        ▼
Day 138: chaos experiments get partly diagnosed through today's tracing setup
```

---

# Part A — Distributed Tracing

## The Three Pillars, and the Question Each One Actually Answers

| Pillar | Answers | Already live on this platform? |
|---|---|---|
| **Metrics** | "In aggregate, how is the system behaving?" (p95 latency, error rate, request volume — numbers, over time) | Yes — Week 11, Days 76–77 |
| **Logs** | "What did this one process actually do or say, at this moment?" | Implicitly, but not yet *correlated* across services |
| **Traces** | "For this *one specific request*, what was the exact path across every service it touched, and where did the time actually go?" | **No — today's new content** |

**Why metrics alone aren't enough, concretely:** Grafana can already show "p95 latency is 420ms." It cannot answer "which *specific* request took 3 seconds, and which of the four modules it passed through actually caused that" — that's an aggregate number describing a whole population of requests, not a diagnostic tool for one of them. Distributed tracing exists specifically to answer the second question, turning "which module caused this slow request" from a guessing game into a single lookup.

## Trace ID, Span ID, and How Context Propagates

**Trace ID** — one ID, generated once, when a request first enters the system (at the Gateway, for external traffic). Every downstream hop that request causes carries the *same* Trace ID.

**Span ID** — one ID per unit of work within that trace — a span for the Gateway's own handling, a child span for its call into Order, a further child span for Order's call into Payment, and so on. Spans form a parent-child tree, all sharing one Trace ID.

**How this actually gets from one service to the next, over HTTP:** Micrometer Tracing instruments the HTTP client and server layers Spring already uses (the same instrumentation-at-the-framework-boundary idea Spring AOP's proxies rely on, Week 9, Day 62) so that an outgoing `RestTemplate`/`WebClient` call automatically adds tracing headers (commonly the B3 format — `X-B3-TraceId`, `X-B3-SpanId`, `X-B3-ParentSpanId` — or the newer W3C `traceparent` header), and an incoming request automatically reads those same headers to continue the existing trace instead of starting a new one. **This happens without application code manually threading an ID through every method call** — it's instrumentation at the framework boundary, exactly like every other proxy-based mechanism this series has already established (Spring AOP, Resilience4j, Redis's `@Cacheable`).

**Traced end to end, for one real request:**

```
Client → Gateway           Trace: T1, Span: S1 (root)
Gateway → Order-Service     Trace: T1, Span: S2 (child of S1)
Order-Service → Payment     Trace: T1, Span: S3 (child of S2)
Order-Service → Product     Trace: T1, Span: S4 (child of S2)
```

One Trace ID (`T1`), four spans, a clear parent-child tree — viewable as a single waterfall in Zipkin (below), showing exactly how much of the total request time each span consumed, and in what order they actually happened (S3 and S4 could be sequential or concurrent, depending on how Order-Service is written — the waterfall view makes that visible directly, rather than something to infer from logs).

## Zipkin

**What it is:** a tracing backend — spans get reported to it (batched, asynchronously, so tracing itself doesn't add meaningful latency to the request it's instrumenting), and its UI renders a full trace as a waterfall: one bar per span, positioned and sized by its actual start time and duration, nested to show the parent-child structure.

**Why this specific view matters:** a waterfall makes "where did the time go" a literal reading exercise — the widest bar is where the request spent the most time, and its position in the nesting shows *which* service was responsible, directly, without cross-referencing separate log files from four different services by hand.

## Log Correlation, via MDC

Micrometer Tracing also injects the current Trace ID and Span ID into SLF4J's **MDC** (Mapped Diagnostic Context) — a thread-local key-value store that a properly configured log pattern includes in every log line automatically. The practical effect: every log line emitted while handling a given request is automatically tagged with that request's Trace ID, so filtering logs across all four modules by one Trace ID reconstructs the exact same request's full log output, in order — logs and traces, correlated, without either one having been built with the other explicitly in mind at the call-site level.

## Sampling — The Real Trade-off

Tracing every single request in a high-traffic production system has a real cost — data volume and processing overhead scale directly with request volume. **Sampling** — tracing only some percentage of requests — trades completeness for cost: a low sampling rate keeps overhead small but risks *missing the one specific slow or broken request you actually needed to see*. A common refinement, worth naming as **extension material**: sample everything that errors or exceeds a latency threshold, and sample only a small fraction of otherwise-normal traffic — biasing sampling toward the requests most worth actually looking at, rather than a flat uniform rate.

## ⚠️ The One Genuine Blind Spot: Kafka

**The claim, stated precisely:** trace context propagates automatically across a synchronous HTTP call, because Micrometer Tracing instruments the HTTP client/server layer directly. It does **not** propagate automatically across an asynchronous Kafka message — a `KafkaTemplate.send()` call is not an HTTP call, and nothing about publishing a message onto a topic carries tracing headers along for free the way an outgoing REST call does.

**Why this matters specifically for this platform:** Saga Choreography (Day 70) is built directly on Kafka pub/sub — a business transaction's steps are chained by *events*, not synchronous calls. Without deliberate propagation, a trace started at the Gateway would end cleanly the moment the flow crosses into an event-driven Saga step — the Trace ID simply wouldn't be there on the consumer side, breaking the single end-to-end waterfall exactly at the point the architecture becomes genuinely asynchronous.

**🔗 Backward Reference (Week 10, Day 70):** the Saga Choreography flow this affects runs entirely on Kafka events, not synchronous calls — the exact transport this blind spot describes.

**The fix, precisely:** trace context has to be carried explicitly as **Kafka message headers** — Micrometer Tracing's Kafka instrumentation (via Spring Kafka) *can* do this, injecting the current Trace ID/Span ID into the outgoing record's headers on publish, and continuing the trace from those headers on consume — but it is a deliberate integration point, not something that happens automatically just because Micrometer Tracing is present elsewhere in the application, the way the HTTP case does.

**🔑 Key Takeaway:** "distributed tracing is set up" is not one uniform fact about a system — it can be true for every synchronous HTTP hop and silently false the moment a request's flow crosses an asynchronous boundary, unless that boundary was configured for it specifically. Naming this distinction unprompted, in the context of a system that (like this one) genuinely mixes synchronous calls and Kafka-based choreography, is a strong, specific signal — much stronger than a general "we use distributed tracing" claim.

---

# Exercise: Wiring Tracing Across the Helm-Deployed Platform

**The goal:** integrate Micrometer Tracing and Zipkin across all four modules, deployed via yesterday's Helm chart, and confirm one full trace.

```yaml
# application.yml, added to every module
management:
  tracing:
    sampling:
      probability: 1.0   # trace everything, for this exercise; production would sample lower
  zipkin:
    tracing:
      endpoint: http://zipkin.ecommerce.svc.cluster.local:9411/api/v2/spans
```

`zipkin.ecommerce.svc.cluster.local` is a Kubernetes Service's in-cluster DNS name (Day 134's Service-discovery mechanism, now used for a new purpose) — every module reaches Zipkin the same way it would reach any other internal Service, no new networking concept required.

**Definition of done:** hitting the Gateway (now running in Kubernetes, behind yesterday's Ingress) generates a Trace ID that propagates through every module it touches, viewable as one complete, correctly-nested trace in the Zipkin UI.

**⚠️ Common Mistakes, checklist:**

- Assuming a trace initiated at the Gateway automatically continues once a Saga step publishes to Kafka — it doesn't, without the explicit header-propagation fix above.
- Setting `sampling.probability: 1.0` in production without realizing this is a genuine overhead/volume trade-off, not a free "more visibility is always better" setting.
- Forgetting that Zipkin reporting is asynchronous/batched — a trace not appearing *immediately* in the UI right after a request completes isn't necessarily a broken pipeline.

**💡 Interview Insight:** "How would you debug a slow request in a microservices system?" is close to a guaranteed question with distributed-systems experience on a resume. The strongest answer distinguishes the three pillars explicitly (metrics tell you *that* something's slow in aggregate; a trace tells you *which specific hop* is responsible for *this specific* request) rather than reaching for "we have logging" as a catch-all. A strong, likely follow-up: "does that work the same way across an async, event-driven step?" — the honest answer is today's Kafka blind spot, stated precisely rather than glossed over.

---

# Part B — API Design Practice (Stripe-Specific Add-On)

**The exercise:** redesign the platform's Order creation endpoint with real rigor, in `docs/api-design-notes.md`.

## Versioning: URI-Based vs. Header-Based

| | URI-based (`/v1/orders`) | Header-based (`Accept: application/vnd.ecommerce.v1+json`) |
|---|---|---|
| Discoverability | High — visible in the URL itself, testable in a browser or with a bare `curl` | Lower — invisible without inspecting request headers |
| Caching | Simple — different versions are literally different URLs, cacheable independently by any generic HTTP cache | Harder — a cache needs to vary on the header, not just the URL |
| Conceptual cleanliness | The resource's "identity" arguably shouldn't include its representation version | Cleaner in that specific sense — same URL, same resource, different negotiated representation |

**The choice, defended rather than just stated:** URI-based versioning for this platform's public API, specifically because discoverability and cache-simplicity matter more here than the conceptual-purity argument for header-based versioning — a partner integrating against this API benefits directly from being able to see and test the version in the URL, which is the concrete, practical thing an API-design round is actually listening for: a reasoned trade-off, not a rule recited from memory.

## Pagination: Offset vs. Cursor — a Concurrent-Insert Failure, Traced

**Offset-based:** `GET /v1/orders?offset=10&limit=10`, returning items ranked 11–20 by `created_at DESC` (newest first).

**The concrete failure, worked through with real positions, not asserted:** a client fetches page 1 (`offset=0`), getting the current ranks 1–10. Before it fetches page 2, a **new** order is created — since results are sorted newest-first, this new order becomes the new rank 1, and every existing order's rank shifts down by one (old rank *N* becomes new rank *N*+1).

The client now fetches page 2 (`offset=10`), expecting "the next 10 after what I already saw." It actually receives **new** ranks 11–20 — which are **old** ranks 10–19 (since new rank = old rank + 1). The result: **old rank 10** — the very last item already shown on page 1 — appears **again** on page 2 (a duplicate), and **old rank 20**, which should have appeared, has been pushed to new rank 21 and is silently **skipped** entirely (it now belongs to page 3). Neither failure raises an error — both are silent correctness bugs, visible only by comparing what the client actually received against what a static snapshot would have returned.

**Cursor-based:** `GET /v1/orders?after=2024-01-15T10:30:00Z_ord_00457` — the cursor encodes the last-seen item's own sort position (a timestamp plus an ID tiebreaker for items with identical timestamps), and the server returns "the next N items after this exact point," anchored to a specific item rather than a numeric position that shifts when the underlying set changes. A new insertion elsewhere in the set doesn't change what "after this specific order" means, so the duplicate/skip failure above cannot occur.

**The trade-off, stated honestly rather than declaring one side an unconditional winner:** cursor-based pagination is stable under concurrent modification but doesn't support "jump directly to page 7" the way offset-based pagination trivially does, since a cursor only knows how to say "the next page after here," not "page 7 specifically." **The choice for Order creation's list endpoint:** cursor-based, since order lists are actively written to in real time (new orders arriving constantly) — precisely the condition that makes offset-based pagination's failure mode a live risk rather than a theoretical one, and "jump to page 7" isn't a real use case for a list that's fundamentally consumed newest-first.

## Idempotency-Key Handling — Applied Directly From Day 130

**🔗 Backward Reference (Week 19, Day 130):** a client-generated key represents *one logical intent*, not one HTTP attempt. The server checks whether that key has already been completed **before** acting; on a genuine duplicate (a client retry after a timeout, say), it returns the *stored result* of the original attempt rather than re-executing the operation.

**Applied here, concretely:** `POST /v1/orders` requires an `Idempotency-Key` header. On receipt, Order-Service checks a store (keyed by that value) for a prior completed result for this exact key before creating anything; if found, it returns the original response directly. This is the exact mechanism already fully taught for the abstract Payment System HLD exercise (Day 130), now applied as real API-contract design for this platform's own Order creation endpoint — a citation and an application, not new material.

## A Structured Error Contract, Not Just a Status Code

**The problem with status codes alone:** a single `400 Bad Request` doesn't let a client's code distinguish "the JSON itself was malformed," "a specific field failed validation," and "the request was well-formed but violated a business rule" (say, insufficient inventory) — three genuinely different situations a calling program needs to handle differently, collapsed into one number.

**The fix — a structured error body alongside the status code:**

```json
{
  "error": {
    "code": "INSUFFICIENT_INVENTORY",
    "message": "Requested quantity exceeds available stock for SKU-4471.",
    "request_id": "a1b2c3d4-..."
  }
}
```

`code` is a stable, machine-checkable string a client can branch on programmatically, independent of the human-readable `message` (which can change wording without breaking any integration that depends on it). `request_id` ties the error directly back to a specific Trace ID — today's tracing work and today's API design work meeting directly: a client reporting "my request with this ID failed" gives you an immediate Zipkin lookup, not a search through logs by approximate timestamp.

**💡 Interview Insight:** an API-design round (explicitly, at Stripe) is evaluating whether you treat these as genuinely separate design decisions, each with a real trade-off, rather than defaults absorbed without examination. The strongest structure for answering any one of these questions live is exactly the shape used above: name the alternative, name the concrete failure or cost it has, then state the choice and why it fits *this specific* endpoint — not a universal rule.

---

# Project Block Guide (3.5 hrs + 30 min add-on)

**Repository:** `scalable-ecommerce-platform`.

**Task:** integrate Micrometer Tracing and Zipkin across all four modules, deployed via the Helm chart, as shown above.

**Definition of done:** hitting the Gateway (in Kubernetes) generates a Trace ID that propagates through every module it touches, viewable as one complete trace in Zipkin.

**Add-on (30 min):** write `docs/api-design-notes.md` redesigning the Order creation endpoint — versioning strategy, pagination approach for the list endpoint, idempotency-key handling, and a structured error contract, following the reasoning above.

---

# Career Block Guide (1 hr)

**LinkedIn:** 20 minutes of genuine engagement — commenting on 3–5 posts.

**Networking:** research the hiring manager for a role already applied to, and send a direct, specific message.

---

# Day 137 — Interview Questions

**Q1. What question does distributed tracing answer that aggregate metrics cannot?**
*Answer:* Metrics show that something is slow in aggregate (e.g., p95 latency is 420ms) across a whole population of requests. Tracing shows, for one specific request, the exact path across every service it touched and exactly where the time went — a diagnostic for one instance, not a population-level number.

**Q2. What's the relationship between a Trace ID and a Span ID?**
*Answer:* A Trace ID is generated once, when a request first enters the system, and is shared across every downstream hop that request causes. A Span ID identifies one specific unit of work within that trace; spans form a parent-child tree, all sharing the same Trace ID.

**Q3. How does trace context get from one service to the next over a synchronous HTTP call, without application code manually passing an ID through every method?**
*Answer:* Micrometer Tracing instruments the HTTP client and server layers directly, automatically attaching tracing headers (B3 or W3C `traceparent`) to outgoing calls and reading them on incoming requests — instrumentation at the framework boundary, the same mechanism family as Spring AOP and Resilience4j's proxy-based approach.

**Q4. Does distributed tracing context propagate automatically across a Kafka message the same way it does over HTTP?**
*Answer:* No. Publishing to Kafka isn't an HTTP call, so nothing carries tracing headers along automatically. Propagating trace context across Kafka requires deliberately carrying it as message headers, via Spring Kafka's tracing instrumentation — a genuine, separate integration point, not a free consequence of having tracing set up elsewhere.

**Q5. Why does this matter specifically for a platform whose Saga is built on Kafka?**
*Answer:* Saga Choreography chains its steps via Kafka events, not synchronous calls. Without deliberate header propagation, a trace started at the Gateway would silently end the moment the flow crosses into an asynchronous Saga step, breaking the single end-to-end trace exactly where the architecture becomes genuinely event-driven.

**Q6. How does log correlation across services actually work here?**
*Answer:* Micrometer Tracing injects the current Trace ID and Span ID into SLF4J's MDC, a thread-local store a properly configured log pattern includes automatically in every log line — so filtering logs by one Trace ID reconstructs that request's full log output across every service it touched.

**Q7. What's the actual trade-off behind a tracing sampling rate?**
*Answer:* Tracing every request has a real, volume-proportional overhead and storage cost. Sampling reduces that cost but risks missing the one specific slow or failing request you actually needed visibility into — a real completeness-versus-cost trade-off, not a free setting.

**Q8. Trace through, with concrete positions, why offset-based pagination can duplicate or skip an item under concurrent inserts.**
*Answer:* If results are sorted newest-first and a new item is inserted while a client is paging, every existing item's rank shifts down by one. A client's second page request (by numeric offset) now lands on ranks that partially overlap the first page (a duplicate) and misses the rank that got pushed past the requested window entirely (a skip) — both silently, with no error raised.

**Q9. Why is cursor-based pagination immune to that specific failure, and what does it give up in exchange?**
*Answer:* A cursor anchors to a specific item's own sort position rather than a numeric offset, so a new insertion elsewhere doesn't change what "the next items after this one" means. In exchange, it can't support jumping directly to an arbitrary page number the way offset-based pagination can.

**Q10. Why isn't an HTTP status code alone enough for a client to handle an API error correctly?**
*Answer:* A single status code (e.g., 400) can't distinguish genuinely different situations — malformed JSON, a specific field failing validation, or a well-formed request that violates a business rule — that a calling program needs to branch on differently. A structured error body with a stable machine-checkable code solves this.

---

## Daily Deliverable Check

- [ ] Micrometer Tracing and Zipkin integrated across all four modules, deployed via Helm.
- [ ] A single request's full trace visible across all four modules in Zipkin, running in Kubernetes.
- [ ] Can explain, unprompted, why tracing doesn't automatically survive a Kafka hop and what fixes it.
- [ ] `docs/api-design-notes.md` written: versioning, pagination, idempotency-key handling, and error contract for the redesigned Order creation endpoint.
- [ ] Can trace, with concrete numbers, exactly how offset-based pagination duplicates or skips an item under concurrent inserts.

---

## What Tomorrow Assumes You Already Know Cold

Day 138 runs real chaos experiments against the platform and expects to **observe** their effects partly through today's tracing and the Resilience4j Circuit Breaker mechanism (Week 10, Day 64) — it does not re-teach how to read a Zipkin trace or re-explain the circuit breaker's state machine. It also assumes today's Kafka-tracing blind spot is solid, since one of tomorrow's chaos experiments touches Gateway-level timeout behavior, a synchronous path where tracing *does* work cleanly — the contrast between the two is only meaningful if today's distinction is already firm.
