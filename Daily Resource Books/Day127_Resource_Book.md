# Day 127 — HLD #8: WhatsApp / Chat System at Scale

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 126 Resource Book](Day126_Resource_Book.md)
**Next ▶:** [Day 128 Resource Book](Day128_Resource_Book.md)
**Companion to:** Day 127 of `Week_19_Revised.md`

---

## Recap

Day 126 closed Week 18 by putting BookMyShow under real load — city-sharding, a Redis TTL+Lua seat-hold reusing Day 121's atomicity pattern, and proactive cache warming as a deliberate extension of Day 123's reactive Thundering-Herd mitigations — then consolidated the week: the HLD 5-step framework (Requirements → Estimation → HLD → Detailed Design → Bottlenecks) applied three times and defended live under follow-up in Mock #2, all four rate-limiting algorithms, Redis's complete picture, Consistent Hashing and Sharding each reused a second time, Snowflake ID generation, and quorum/Raft/split-brain with etcd's role under Kubernetes named explicitly.

Today opens Week 19, which closes the HLD phase entirely: nine more systems across seven days, three more mock interviews (Days 128, 131, 133) landing on top of Week 18's two, for five HLD mocks total — matching the LLD phase's five. Today's system, WhatsApp, is chosen to open the week because it's the first system in the whole HLD arc that is genuinely **stateful at the connection level** — every prior system (URL Shortener, the rate limiters, the notification system, the cache, ID generation, consensus, BookMyShow) could route any request to any server, because nothing about the request depended on *which specific server instance* was involved. A chat client's live connection breaks that assumption for the first time, and today's entire theory block is really one extended argument about what has to change once it does.

---

## Learning Objectives

By the end of today, without notes:

1. Explain why a WebSocket connection is stateful in a way a normal HTTP request is not, and name the two concrete system components that statefulness forces into existence.
2. Trace a message end-to-end through all three delivery cases — same server, different server, recipient offline — naming which component handles each branch and why.
3. Explain precisely why a wide-column store fits chat history better than a relational table, from the storage-engine mechanism up, not just "it scales."
4. Apply the HLD 5-step framework unprompted to a new system, including a real estimation pass with arithmetic shown, not asserted.

---

## Concept Dependency Map

```
Day 71–72: TCP fundamentals, HTTP/HTTPS request-response, L4 vs. L7 Load Balancers
Day 48, 50: Kafka Fundamentals, Kafka Consumers (partition-level ordering, consumer groups)
Day 40: SQL Fundamentals — B-tree indexes, the write-cost of maintaining them
Day 44, 123: Redis as infra; Redis's full data-structure picture
Day 59, 61: CAP Theorem; Replication Models (leaderless, quorum, R+W>N)
Day 78: Sharding Strategies — the "celebrity problem," named
Day 120: The HLD 5-Step Framework (Requirements → Estimation → HLD → Detailed Design → Bottlenecks)
        │
        ▼
Today — WhatsApp / Chat System
  ├─ Persistent WebSockets (NEW)
  │     needs: HTTP request-response as the thing it's contrasted against (Day 72)
  │
  ├─▶ Connection-Management Layer (NEW)
  │     needs: WebSockets (today, above) — a connection pinned to one server
  │     is what makes this component necessary at all
  │
  ├─▶ Message Routing via Kafka (APPLIED, not re-taught)
  │     needs: Kafka Fundamentals (Day 48), Consumer Groups (Day 50),
  │            Connection-Management Layer (today, above) — routing needs
  │            to know *where* to route first
  │
  ├─ Read Receipts: delivered vs. read (NEW, light)
  │     needs: the delivery paths above, as the events that trigger each status
  │
  └─▶ Offline Storage — Wide-Column Store (NEW)
        needs: SQL Fundamentals' B-tree write cost (Day 40, as the contrast),
               CAP Theorem (Day 59) and Replication Models (Day 61) for its
               consistency posture
```

---

## Part 1 — Requirements (Step 1 of the HLD Framework)

**In scope, clarified before designing anything:** 1:1 messaging, group messaging, message delivery status (sent → delivered → read), online/offline presence. **Explicitly acknowledged but deliberately deferred:** media sharing (photos, video, voice notes) — the actual mechanics of storing and serving large binary objects get their own full treatment on **Day 133** (chunking, content-addressable dedup, multipart upload). Naming media sharing as in-scope today without designing it is itself the correct interview move: a candidate who tries to improvise object-storage design from scratch, mid-answer, on a system where it's a side concern, burns time better spent on the system's actual distinguishing problem. Today's diagram treats a media message as "delegate to the object-storage layer, get back a URL, send the URL as if it were text" — correct at this level of abstraction, not a cop-out.

**Non-functional requirements:** low latency (a message should feel instant, not "eventually arrives"); high availability (a chat app being briefly unreachable is a much worse user experience than, say, a slightly stale product recommendation); **eventual consistency is acceptable** for read receipts and presence status specifically — if your "online" indicator is wrong for two seconds, nothing breaks — but message *delivery itself* cannot silently drop a message. That asymmetry (loose consistency on metadata, strict delivery guarantees on content) is worth stating out loud in an interview, because it's the kind of precision that distinguishes "I know the CAP theorem" from "I know where to actually spend the consistency budget."

---

## Part 2 — Persistent WebSockets

### Prerequisites (confirmed)
- HTTP request-response, and what a request/response cycle actually costs (Week 10, Day 72).
- TCP as the transport HTTP itself rides on (Week 10, Day 71).

### The problem HTTP alone can't solve here

Every system built so far in this series is **client-initiated**: the client asks, the server answers, the connection's job is done. Chat needs the opposite direction too — the server (or really, another user, via the server) needs to be able to push a message to a client that never asked for anything. Plain HTTP has no mechanism for that at all; something has to change about the connection itself.

### Mechanism

A WebSocket connection starts as an ordinary HTTP request carrying an `Upgrade: websocket` header. If the server agrees, it responds `101 Switching Protocols` — and from that exact point on, the **same underlying TCP connection** stops speaking HTTP and starts speaking a much lighter, framed protocol where either side can write to the socket at any moment, unprompted. This is a **full-duplex, persistent** channel: "full-duplex" meaning both directions are open simultaneously (not politely taking turns the way request-response does), "persistent" meaning the TCP connection itself stays open across many messages instead of being torn down and re-established per interaction.

### Why it works

Because the TCP connection is genuinely still open, the server's process already has a live file descriptor pointed at that specific client. Writing a message into that socket at any time requires no new connection setup, no repeated headers, no new TCP handshake — the "instant" feeling of chat is a direct, mechanical consequence of the connection already existing before the message does.

### When to reach for it — the concrete signal

The server needs to push data to the client **without the client asking first**, and the latency needs to feel close to instant — chat, live notifications, collaborative editing, multiplayer state. If the client is always the one initiating ("give me my inbox") a normal request is simpler and should be preferred.

### Trade-offs against the nearest alternatives

| Approach | Direction | Latency | Cost per update |
|---|---|---|---|
| Short polling | Client-initiated, repeated | Up to the poll interval | A full HTTP request every interval, whether or not anything changed |
| Long polling | Client-initiated, server holds the request open | Near-instant, but a new request cycle starts immediately after every delivery | A full HTTP handshake + headers per delivered update |
| Server-Sent Events (SSE) | **Server → client only**, over plain HTTP | Near-instant | One open connection, no per-message overhead |
| WebSocket | **Both directions**, one connection | Near-instant | One open connection, no per-message overhead |

SSE is the closer competitor, not polling — it's genuinely persistent and just as cheap per message. It loses here for one specific reason: chat needs the client to *send* messages on the same low-latency channel it receives them on, and SSE is one-directional by design. A chat app running SSE would still need a separate mechanism (ordinary HTTP POSTs) for the client-to-server direction — two systems doing one job. WebSocket wins by being the smallest single mechanism that actually covers both directions.

### The real, concrete cost — not Big-O, but it drives every estimate below

An open WebSocket connection holds a file descriptor and a slice of memory on its server for as long as it's open, **whether or not any message is currently flowing through it**. This is the classic "C10K problem" (and, at WhatsApp's real scale, C10M) — the practical ceiling on how many *idle* connections one server process can hold open is a hard resource constraint, not a tuning afterthought. Every later estimate in today's book about "how many servers do we need" traces back to this one fact.

> ⚠️ **Common Mistake:** treating an open WebSocket as "free" because no messages are being sent through it right now. It costs memory and a file descriptor for its entire open lifetime — a million idle connections is a real capacity number, not a zero.

> ⚠️ **Common Mistake:** assuming a load balancer handles a WebSocket upgrade the same way it handles a normal request. An L7 (application-aware) load balancer terminates and inspects HTTP semantics — it needs explicit configuration to recognize and pass through an `Upgrade` handshake rather than timing it out as an unusually long-lived request. An L4 (connection-level) balancer, from Day 72, is actually less likely to interfere here, since it's routing raw TCP and was never inspecting HTTP semantics per-message in the first place.

> 💡 **Interview Insight:** reconnection handling isn't a footnote — mobile networks drop connections constantly. A reconnecting client needs to ask "what did I miss," and the honest answer is: that's *exactly* the same question a fully-offline user's client asks on reconnect. Section 5 below (Offline Storage) answers both with the same mechanism, not two separate ones — worth saying out loud unprompted, because unifying the two cases is the kind of design choice interviewers specifically listen for.

---

## Part 3 — The Connection-Management Layer

### Definition

A fast, shared lookup service mapping every currently-connected `userId` to the specific server instance (and connection) currently holding their WebSocket — updated on every connect and disconnect.

### Why it has to exist at all

A stateless HTTP system never needs this: any request can be served by any server, because nothing about the *previous* request is remembered anywhere. A WebSocket breaks that completely — this specific TCP connection lives inside this specific server process, for as long as it stays open. Once that's true, **something** in the system has to remember and actively route around it, because the load balancer's own routing decision was already made once, at connect time, and can't be revisited per message.

### Mechanism

A Redis hash (Redis as infra since Day 44, full picture Day 123) is the natural fit: `HSET conn:{userId} serverId, connectionId` on connect, cleared on clean disconnect. A short TTL with a periodic heartbeat renewal is the necessary safety net — without it, a server that crashes *without* a clean disconnect leaves a permanently stale, wrong entry, silently routing every future message for that user into a black hole.

> ⚠️ **Common Mistake:** skipping the heartbeat/TTL and relying only on explicit disconnect events to clean up the registry. Servers crash; "explicit disconnect" is not a guaranteed event, and a registry that's never proven to self-heal is a real, production-shaped bug, not a hypothetical one.

### Not the same problem as sticky sessions

A load balancer's sticky-session feature (routing *your own* future requests back to the server that already has your session, usually via a cookie or IP hash) solves a genuinely different problem: it answers "does my next request keep landing where my state already is." It does **not** answer "how does someone else's server find where *my* connection currently lives" — that second question is what the connection registry exists for. Conflating the two is a common, precise-sounding mistake worth being able to name and correct on the spot.

---

## Part 4 — Message Routing via Kafka (applying, not re-teaching, Days 48 and 50)

Kafka Fundamentals (Day 48) and Consumer Groups (Day 50) are fully reflexive by now — nothing about partitioning, ordering-within-a-partition, or consumer-group semantics is new today. What's new is the **application**: using Kafka as the transport between chat-server instances, not between microservices publishing domain events.

### Detailed design — tracing a message through every branch

Client A is connected to Chat Server 1. Client A sends "hey" to Client B.

1. **Chat Server 1 looks up B** in the connection registry.
2. **Case a — B is on Chat Server 1 too:** deliver directly, in-process, over B's already-open WebSocket. No Kafka hop needed — the cheapest path, and worth explicitly calling out as the fast path rather than routing everything through Kafka unconditionally.
3. **Case b — B is on Chat Server 2:** Chat Server 1 publishes the message to Kafka, tagged with Server 2's ID as part of the message envelope. Chat Server 2's own consumer — every chat-server instance runs a consumer subscribed to this inter-server topic, filtering for messages tagged with its own ID — picks it up and delivers over B's live WebSocket.
4. **Case c — B is offline** (no registry entry, or one that's expired): the message is written directly to the offline message store (Part 5) as pending. When B reconnects, their newly-assigned chat server queries the store for everything addressed to B and pushes it down the new connection — **the identical mechanism Part 2's reconnection case needs**, unified rather than duplicated.
5. **Regardless of path**, the message is *also* durably persisted to the wide-column store asynchronously (Part 5) — every message, not only the offline ones — since chat history and multi-device sync both need it. This has to be async: blocking the real-time delivery path on a synchronous database write would reintroduce exactly the latency the whole WebSocket design exists to avoid.

### Why Kafka specifically, not a simpler pub/sub

Redis Pub/Sub is fire-and-forget: if the subscribing server instance happens to be mid-restart when a message publishes, that message is gone. Kafka's topic is a durable, ordered log — a briefly-unavailable consumer catches up on reconnect rather than silently losing anything. For a chat delivery path, that durability is the whole point of picking Kafka over a lighter alternative, and it's worth being able to state that trade-off unprompted rather than defaulting to Kafka out of habit.

### Read Receipts — delivered vs. read

Two genuinely distinct events, triggered by different things:

- **Delivered:** fires automatically the instant the message successfully hands off in case (a), (b), or the reconnect-delivery branch of case (c) above — it means "reached the recipient's device," nothing about the recipient's attention.
- **Read:** fires only when the recipient's client actually renders/opens the message, which emits its own lightweight event back through the exact same routing mechanism, reversed (B's server → registry lookup for A → same three-case routing) to update A's UI.

"Read" can only ever happen after "delivered" — the ordering is structural, not a business rule bolted on separately, since a message that was never delivered was never available to be read.

---

## Part 5 — Offline Storage: The Wide-Column Store

### Prerequisites (confirmed)
- SQL Fundamentals — specifically, that a B-tree index trades write speed for read speed (Week 6, Day 40).
- CAP Theorem (Week 9, Day 59) and Replication Models — leaderless, quorum-based, R+W>N (Week 9, Day 61).
- Sequential disk appends as the mechanical reason for throughput (Week 7, Day 48, applied there to Kafka's log).

### Definition

A wide-column store organizes data by a **partition key** (which physical node owns this data) and, within each partition, a **clustering key** that fixes the on-disk *order* of rows sharing that partition — a fundamentally different shape from a relational table's fixed-schema rows, and optimized for a specific access pattern: "give me everything for key X, in order," not arbitrary joins or ad-hoc queries.

### Mechanism — why writes are cheap here specifically

A wide-column store's storage engine is typically an **LSM-tree** (Log-Structured Merge tree): a write goes first to an in-memory table plus a write-ahead log, and is only periodically flushed to disk as an immutable, sorted file, with background compaction merging older files later. This is the *same* underlying principle Day 48 named for Kafka's own throughput — sequential writes dramatically beat random-access writes — applied here to a database's storage engine rather than a message log. Contrast directly with Day 40's B-tree: a B-tree index pays a real, immediate cost on every write (potentially rebalancing nodes to keep itself ordered for fast reads); an LSM-tree defers that cost entirely to background compaction, trading some read amplification (a read might need to check several files) for dramatically better write throughput. That trade is exactly right for chat: write volume is enormous and constant, reads are narrow and predictable (one conversation's recent history).

### Applying it here

**Partition key:** `conversationId` — every message in one conversation lands on one partition, which is precisely what makes "give me the last 50 messages of this chat" a cheap, single-partition range scan instead of a scatter-gather across the whole cluster. **Clustering key:** `timestamp` (or a monotonic message ID) — fixes chronological order within that partition for free, as a property of storage layout rather than a runtime sort.

### Why this, not Postgres

Chat's actual access pattern — extremely high write volume, always queried by "everything for this one key, in order," essentially never joined against anything else — is exactly the pattern a wide-column store is built for and a relational table is not specialized for. This isn't "NoSQL is faster" as a blanket claim (a claim like that should be a red flag in an interview answer, not a selling point) — it's a specific match between this system's specific access pattern and this storage engine's specific design.

### Trade-offs against the relational alternative

No joins, and no cheap arbitrary secondary-query flexibility without maintaining a separate index table by hand. Consistency is typically **tunable, leaning eventual** across replicas — Cassandra itself was already named on Day 59 as an AP-leaning *default*, not a fixed classification, and it achieves that through the same leaderless, quorum-based replication (R+W>N) taught on Day 61. That's an acceptable trade *here* specifically because message history briefly lagging on one replica read doesn't affect the real-time delivery path at all — delivery already happened over the WebSocket route in Part 4, independent of this async persistence step.

### Common mistake and a real edge case

> ⚠️ **Common Mistake:** picking a low-cardinality or skewed partition key. A viral group chat with an enormous, constantly-active membership turns its own `conversationId` into a **hot partition** — mechanically the identical shape Day 78 named "the celebrity problem" for sharding, and one that resurfaces again, at a different layer entirely, in **two days** on Day 129's fan-out discussion. Worth flagging that connection explicitly rather than treating each occurrence as a new problem.

**Edge case worth naming:** an unbounded conversation (one that's been active for years) grows one partition indefinitely. Real systems bucket the partition key by time window too (e.g., `conversationId + month`) specifically to keep individual partitions bounded — a refinement worth mentioning even though today's stub implementation won't need it at toy scale.

---

## Part 6 — Estimation (Step 2 of the HLD Framework)

Assume, for a tractable back-of-envelope pass (the arithmetic process is what's being tested here, not memorized real-world figures): **500M daily active users**, each sending an average of **40 messages/day**.

- **Messages/day:** 500M × 40 = 20 billion messages/day.
- **Average throughput:** 20B ÷ 86,400s ≈ **≈230K messages/sec**, average.
- **Peak throughput:** real chat traffic isn't flat — assume a 3–4x peak multiplier ≈ **700K–1M messages/sec** at peak. Every capacity number below should be sized against peak, not average, or the system falls over exactly when it matters most.
- **Storage:** at ~100 bytes/message average (mostly short text), 20B × 100B ≈ **2TB/day** of raw message data, before replication overhead — landing squarely on Part 5's wide-column store, sized for sustained high-volume append.
- **Concurrent connections — the number that actually drives the architecture:** even if only ~10–15% of DAU are concurrently connected at any given moment, that's **50–75 million simultaneous open WebSocket connections**. Given Part 2's C10K/C10M ceiling — call it roughly 500K–1M sustainable idle connections per well-tuned server instance — that alone implies **on the order of 50–150 chat-server instances**, purely for connection capacity, before a single message is even routed. This is the concrete number that makes Part 3's connection registry and Part 4's cross-server routing **structurally unavoidable**, not an optional refinement — worth stating that causal link out loud in an interview rather than presenting the registry as a design preference.

---

## Part 7 — HLD Diagram (today's coding exercise)

```
                         ┌────────────────────┐
        Client A ───WS───▶  Chat Server 1     │
                         └─────────┬──────────┘
                                   │  1. lookup B in registry
                                   ▼
                         ┌────────────────────┐
                         │ Connection Registry │◀── heartbeat/TTL from
                         │   (Redis Hash)      │    every chat server
                         │ userId → server,conn│
                         └─────────┬──────────┘
                    ┌──────────────┼───────────────────┐
             same server      diff. server           no entry (offline)
                    │              │                     │
                    ▼              ▼                     ▼
           deliver in-process  publish to Kafka    write to Offline Store
                    │         (tagged: Server 2)     (Wide-Column Store,
                    │              │                  partition=conversationId,
                    │              ▼                  cluster=timestamp)
                    │      Chat Server 2 consumes            │
                    │      tagged messages, delivers          │
                    │      to Client B over its WS            │
                    │              │                          │
                    └──────┬───────┘                          │
                           ▼                                  │
                     Client B receives ◀── on reconnect, new server
                     "delivered" fires      queries offline store,
                           │                delivers backlog (same path)
                           ▼
                  Client B opens message
                           │
                           ▼
                  "read" event, routed back
                  to A via the same 3-case logic
                           │
                           ▼
                     Client A sees "read"

  (async, off the hot path, from every message regardless of branch)
  Chat Server ──▶ Kafka ──▶ persistence consumer ──▶ Wide-Column Store
```

---

## Part 8 — Bottlenecks (Step 5 of the HLD Framework)

- **Connection capacity per server** — the Part 6 arithmetic's direct consequence; mitigated by horizontal scaling of chat-server instances plus the registry that makes horizontal scaling actually work for a stateful connection.
- **Hot conversations / massive group chats** — the same "celebrity problem" shape named on Day 78, resurfacing at the partition level here and, in two days, at the feed level on Day 129. A single huge group's send fans out to every member's delivery path at once; mitigation echoes what Day 129 will formalize.
- **The registry itself** — a single unsharded Redis instance holding tens of millions of connection mappings is itself a scaling and availability concern; mitigated the same way any Redis-backed component is (Day 123): sharding the registry and accepting brief staleness on failover, since a slightly-stale routing entry just causes one retried hop, not data loss.
- **Kafka partition sizing for the inter-server topic** — too few partitions caps cross-server delivery throughput regardless of how many chat servers exist; directly reapplies Day 48's partition-parallelism fact rather than introducing anything new.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`. **Task:** no new feature today — the slot is deliberately spent on documentation and boundary hygiene: review Product, Order, Payment, and Notification, and make sure each module's responsibility is unambiguous and actually written down before more logic lands on top of any of them.

**A `docs/architecture.md` starting skeleton, to adapt rather than copy verbatim:**

```markdown
# scalable-ecommerce-platform — Architecture

## Module Boundaries

### Product
Owns: catalog data, pricing, inventory counts.
Does not own: order state, payment state.

### Order
Owns: order lifecycle (created → paid → fulfilled → ...), the Saga
choreography coordinating Product/Payment/Notification (Week 10, Day 70).
Does not own: how a payment is actually processed.

### Payment
Owns: charge processing, idempotency-key checking (Day 130), the
double-entry ledger (Day 130).
Does not own: order lifecycle state — Payment reports success/failure;
Order decides what that means for the order.

### Notification
Owns: per-channel delivery (Week 18, Day 122), templating.
Does not own: *when* a notification is warranted — that decision belongs
to whichever module's event triggered it.

## Cross-Cutting
- Gateway: routing, rate limiting (Week 18, Day 121).
- Kafka: the event backbone connecting all four modules (Day 48 onward).
```

**Definition of done:** the draft above, adapted to what the codebase's actual module boundaries currently are (not what they're aspirationally supposed to be) — this file gets revisited and brought fully current on Day 133, so an honest snapshot today is more useful than a polished but inaccurate one.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement only today — 20 minutes commenting on 3–5 posts, no new post to draft.

**Networking:** check in on the applications submitted Day 118. Respond promptly to any recruiter outreach that's come in — response speed itself is sometimes read as a signal about how seriously a candidate is taking the process, independent of the content of the response.

---

## Day 127 — Interview Questions

**Q1. Why can't a chat application just use ordinary HTTP request/response?** HTTP is client-initiated by design — the server can never push data the client didn't ask for. Chat fundamentally needs the server (on another user's behalf) to deliver a message the recipient never requested at that moment.

**Q2. Walk through the WebSocket handshake mechanically.** The client sends a normal HTTP request with an `Upgrade: websocket` header; the server responds `101 Switching Protocols`; from that point on, the same TCP connection carries a lightweight framed protocol instead of HTTP, and either side can write at any time.

**Q3. WebSocket vs. long-polling vs. Server-Sent Events — when would you pick each?** Long-polling if you need something quick and don't control the infrastructure well enough for persistent connections. SSE if you only need server-to-client push. WebSocket when you need genuine low-latency traffic in *both* directions on one connection — chat's requirement exactly.

**Q4. Why is an idle, open WebSocket connection a real cost, not a free one?** It holds a file descriptor and memory on its server for its entire open lifetime regardless of message activity — the C10K/C10M constraint that directly drives how many server instances the system needs.

**Q5. Why does cross-server message delivery need something like Kafka rather than a load balancer alone?** A load balancer's routing decision is made once, at connect time. It has no mechanism to revisit "which server currently holds User B's connection" on every subsequent message from a different user — that's a lookup-and-route problem the connection registry and Kafka solve, not something a stateless LB can.

**Q6.** [Trace] **A is on Chat Server 1, B is on Chat Server 2. Walk the full path of a message from A to B.** Server 1 looks up B in the registry, finds Server 2, publishes to Kafka tagged for Server 2; Server 2's consumer picks it up and delivers over B's live WebSocket; "delivered" fires on that successful handoff.

**Q7. What happens if B is fully offline when A sends?** The message is written directly to the wide-column offline store instead of being routed. On reconnect, B's newly-assigned server queries the store for everything pending and delivers it down the new connection — the same mechanism that serves a briefly-reconnecting client, not a separate one.

**Q8. What's the precise difference between "delivered" and "read," and can "read" ever fire before "delivered"?** Delivered fires on successful handoff to the recipient's device; read fires only when the client actually renders the message. Read can never precede delivered — a message can't be read before it existed on the recipient's device.

**Q9. Why is a wide-column store a better fit here than a relational table, mechanically — not just "it scales"?** Its LSM-tree storage engine defers write cost to background compaction rather than paying it immediately the way a B-tree index does (Day 40) — matching chat's real access pattern of enormous write volume against simple "everything for this key, in order" reads, with no joins needed.

**Q10. What are the partition key and clustering key here, and why those specifically?** Partition key `conversationId`, so one conversation's messages live on one partition; clustering key `timestamp`, fixing chronological order as a property of storage layout rather than a runtime sort.

**Q11. Why is eventual consistency acceptable for this store but not for the delivery path itself?** Delivery already happened over the real-time WebSocket route independent of this store; the store's job is durable history and multi-device sync, where a few hundred milliseconds of replica lag has no user-visible consequence.

**Q12. What's the hot-partition risk in this design, and where else has this exact shape appeared in the series?** A massive, constantly-active group chat turns its own `conversationId` into a hot partition — the identical shape Day 78 named the "celebrity problem" for sharding, resurfacing again at the feed level on Day 129.

**Q13. Sticky sessions and the connection registry sound similar — what's the actual difference?** Sticky sessions answer "does *my own* next request land back where my state already is." The registry answers "how does *someone else's* server find where *my* connection currently lives" — a different question a sticky-session config can't answer at all.

**Q14. Given 500M DAU and 40 messages/user/day, what's the average messages/sec, and why does peak matter more than average for sizing?** ≈230K/sec average; real traffic isn't flat, so sizing against a 3–4x peak (≈700K–1M/sec) is what keeps the system up during the actual moments it needs to hold, not just on a typical afternoon.

**Q15. Media sharing was named as in-scope during requirements but not designed today — why, and is that a gap?** It's a deliberate deferral, not an oversight: the actual object-storage mechanics (chunking, dedup, multipart upload) get their own full system on Day 133, and building that from scratch mid-answer here would burn time better spent on chat's actual distinguishing problem — stateful connection routing.

---

## Daily Deliverable Check

- [ ] WhatsApp HLD diagram complete, correctly routing all three cases: same-server, cross-server, and offline delivery.
- [ ] Can explain, from memory, why a WebSocket connection is stateful in a way an HTTP request isn't, and name both components that statefulness forces into existence.
- [ ] Can trace a message end-to-end through all three delivery branches without notes.
- [ ] `docs/architecture.md` drafted with all four current module boundaries.
- [ ] Applications from Day 118 checked; any recruiter outreach responded to.

---

## What Tomorrow Assumes You Already Know Cold

Day 128 assumes today's Redis fluency (the connection registry, built as a Redis hash) is solid enough that tomorrow's `GEOADD`/`GEOSEARCH` work needs no re-introduction to Redis itself — only to the geospatial commands specifically. It also assumes the HLD 5-step framework's **Estimation** step, walked through with real arithmetic today, is now something you reach for unprompted on a new system rather than something that has to be recalled as a checklist item — tomorrow's mock (HLD Mock #3) will not remind you to do it.
