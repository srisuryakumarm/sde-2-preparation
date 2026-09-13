# Week 19 (Revised): HLD Systems Complete — Nine More Systems, Three More Mock Interviews

**What changed:** three more HLD mocks land across this week (Days 128, 131, 133), closing the phase at five total — matching LLD's five, and a genuine multiple of the original plan's single HLD mock.

---

## Day 127 — HLD #8: WhatsApp / Chat System

### Theory Block (2.5 hrs)
- Topic: WhatsApp / Chat System
- **Requirements:** 1:1 chat, group chat, read receipts, media sharing, online presence — clarify which are in scope before designing.
- **Persistent WebSockets:** unlike HTTP's request/response, a chat client holds an open connection so the server can push messages instantly, without the client needing to poll. A connection-management layer tracks which server holds which user's active connection, since a given user's WebSocket connects to exactly one server instance at a time.
- **Message routing:** if the sender and receiver are connected to different servers, the message routes through a message broker (Kafka fits naturally here) between them.
- **Read receipts:** an acknowledgment flow — delivered (reached the recipient's device) and read (the recipient actually opened it) are genuinely different events, triggered by different things.
- **Offline storage:** messages for a currently-offline user queue in a database until they reconnect — a wide-column store (Cassandra-style) fits this write-heavy, simple-query pattern well.
- Coding exercise: sketch the full HLD — WebSocket servers, the routing/broker layer, and the message store — labeling exactly where a message goes if the recipient is offline versus online on a different server.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: none new today — use this time to review and clean up the module skeleton, ensuring Product, Order, Payment, and Notification modules each have a clear, documented responsibility boundary before more logic gets added on top.
- Definition of done: a `docs/architecture.md` draft explaining the module boundaries.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: check in on applications submitted Day 118 — respond promptly to any recruiter outreach, since response speed itself is sometimes read as a signal.

### Daily Deliverable
- [ ] WhatsApp HLD diagram complete, correctly routing both online and offline message cases.
- [ ] `docs/architecture.md` drafted.

---

## Day 128 — HLD #9: Uber Driver Location Tracking, and HLD Mock #3

### Theory Block (2 hrs)
- Topic: Uber Driver Location Tracking
- **The core challenge:** millions of drivers updating their location every few seconds is an extreme write-heavy workload, and "find drivers near me" is a geospatial query, not a simple key lookup.
- **Geohashing:** encodes a lat/long pair into a string where nearby locations share a common prefix, turning "find nearby" into a prefix-range query a normal index can handle directly.
- **Quadtrees:** recursively subdivide space into four quadrants, going deeper only where drivers are actually dense — efficient for highly uneven distribution (dense in cities, sparse elsewhere), which geohashing's fixed-precision grid handles less gracefully.
- **Redis Geo:** `GEOADD` and `GEORADIUS`/`GEOSEARCH` give you this out of the box, backed by a sorted set underneath — often the pragmatic real answer instead of hand-rolling a quadtree from scratch.
- Coding exercise: spin up Redis locally; use `GEOADD` to add several coordinates and `GEOSEARCH` to find points within a radius.
- **A real, specific lesson from an actual Uber interview:** a candidate designing this exact kind of system brought up CAP theorem while discussing a single-machine in-memory cache — CAP only applies once a system is genuinely distributed across a network partition, so raising it there was a real ding, not a neutral aside. The lesson generalizes: Uber's bar isn't "do you know the vocabulary," it's "do you know precisely when a concept applies and when it doesn't." Say why a concept is relevant before you invoke it, every time, not just here.

### HLD Mock #3 — Uber Format (1 hr)
- 45-minute HLD mock, using **Distributed Cache** as the subject, but run today's mock the way Uber's real loop runs: think out loud constantly, justify every design decision as you make it rather than presenting a finished answer, and expect to be pushed on precisely *why* each choice applies here specifically. Uber's real feedback pattern rewards ruthless clarity about time complexity and penalizes over-engineering — your accountability partner should interrupt with "why not simpler" at least twice.

### Project Block (1 hr)
- Repository: `scalable-ecommerce-platform`.
- Task: add a `LocationService` stub to the Order module using Redis Geo commands, simulating "find delivery partners within 3km" for a future delivery-tracking feature.
- Definition of done: querying a hardcoded set of partner locations returns the correct nearby subset.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: send connection requests to 3 engineers at location-heavy platforms (ride-share, delivery).

### Daily Deliverable
- [ ] Can explain geohashing vs. quadtrees and when each fits better, from memory.
- [ ] Redis Geo `LocationService` stub working correctly.
- [ ] HLD Mock #3 completed and debriefed.

---

## Day 129 — HLD #10: Netflix / Video Streaming, and HLD #11: Instagram Feed

### Theory Block (2.5 hrs)
- Topic: Netflix / Video Streaming
- **Transcoding pipeline:** an uploaded video gets processed into multiple resolutions and bitrates — adaptive bitrate streaming means the player switches quality based on the viewer's current bandwidth, mid-playback, without the viewer doing anything. **CDNs:** video content is cached at edge locations geographically close to viewers, since re-fetching a popular video from a single origin server for every viewer worldwide doesn't scale and adds unacceptable latency.

- Topic: Instagram / News Feed Generation
- **Push (fan-out on write):** when you post, your content is immediately pushed into every follower's feed cache — fast reads, but expensive writes for a very popular account, since one post can trigger millions of fan-out writes. **Pull (fan-out on read):** a feed is assembled on demand by querying everyone you follow at read time — cheap writes, but slow reads at real scale. **Hybrid fan-out:** push for normal users, pull (or a capped push) for celebrity accounts with huge follower counts — this specific hybrid is the answer interviewers are usually looking for, not a pure strategy either way.
- Coding exercise: none — write 150 words on why a pure push strategy specifically breaks down for celebrity accounts, using real numbers to make the point (a celebrity with 100M followers posting once triggers 100M writes instantly).

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: sketch (design only, no implementation needed) a hybrid fan-out approach for a hypothetical "product recommendations feed" feature, applying today's Instagram theory to the platform's own domain.
- Definition of done: a short design doc in `docs/`.

### Career Block (1 hr)
- LinkedIn: Post 25 — "Push vs. pull: why your feed algorithm has to change for a celebrity account" (a concrete, well-known example makes a strong hook).
- Networking: engage heavily — comment on 5 posts today.

### Daily Deliverable
- [ ] Can explain adaptive bitrate streaming and CDN caching without notes.
- [ ] Can explain hybrid fan-out and why pure push fails for celebrity accounts, without notes.
- [ ] Hybrid fan-out design doc pushed. LinkedIn Post 25 published.

---

## Day 130 — HLD #12: Payment System

### Theory Block (2.5 hrs)
- Topic: Payment Gateway Design
- **Idempotency keys:** a client-generated unique key attached to a payment request, so a retried request — a network timeout, a client-side retry — doesn't charge the customer twice. The server checks "have I already processed this exact key" before doing anything at all.
- **Distributed locks:** prevent two concurrent requests for the same logical operation (the same order, say) from both proceeding — a Redis `SETNX`-based lock is the common lightweight answer.
- **Double-entry ledger:** every transaction records two balanced entries, a debit and a credit, which makes the entire system auditable and mathematically self-checking — if debits and credits ever don't balance, you've found a real bug immediately, not a discrepancy someone has to hunt for blind.
- Coding exercise: write idempotency-wrapper pseudocode using a Redis `SETNX` lock to prevent double-charging on a retried request.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: implement idempotency-key checking in the Payment module — a repeated request with the same key returns the original result instead of processing it again.
- Definition of done: firing the same payment request twice with the same idempotency key results in exactly one charge, verified with a test.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: follow up specifically on any Stripe or Rippling applications — both companies move relatively fast once started, so a week of silence is worth a polite nudge to the recruiter.

### Daily Deliverable
- [ ] Can explain idempotency keys and the double-entry ledger without notes.
- [ ] Idempotency-key checking live and verified in the Payment module.

---

## Day 131 — HLD #13: Distributed Job Scheduler, and HLD Mock #4

### Theory Block (2 hrs)
- Topic: Distributed Job Scheduler
- **Delayed message queues:** a job scheduled for the future isn't just "put on a queue now" — SQS Visibility Timeout or a Redis Sorted Set (score = execution timestamp) both let you efficiently poll for "what's ready to run right now."
- **Quartz Scheduler** concepts: cron-like scheduling, and misfire handling — what happens if the scheduler itself was down when a job should have fired.
- **At-least-once vs. exactly-once execution:** distributed schedulers commonly guarantee at-least-once — a job might run twice if a worker crashes mid-execution after claiming the job but before marking it done — which means job handlers generally need to be idempotent, tying directly back to yesterday's Payment System theory.
- Coding exercise: write a Redis Lua script snippet that polls a Sorted Set for delayed jobs where `score (timestamp) < now`, atomically claiming them.

### HLD Mock #4 (1 hr)
- 45-minute HLD mock, using **Payment System** as the subject — this one tends to have the most follow-up depth available (idempotency, locking, the ledger), so use the full time pushing into "what if two of these requests race" scenarios rather than stopping at the first correct answer.

### Project Block (1 hr)
- Repository: `scalable-ecommerce-platform`.
- Task: add a `@Scheduled` background task simulating a nightly reconciliation job — verifying every Order maps to a successful Payment, flagging mismatches.
- Definition of done: the scheduled task runs successfully, correctly flags a deliberately-introduced mismatch in test data.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: review your application tracker across all 7 target companies — status, next steps, any prep gaps a specific upcoming round exposes.
- **Worth a specific revision pass given Uber's own advice:** "practice DSU (Union-Find), DP, and classic CS problems, be ruthless about time complexity" — if either pattern feels rusty, this week is the time to redo one problem from each cold, not the week of the actual Uber interview.

### Daily Deliverable
- [ ] Can explain at-least-once execution and why it demands idempotent job handlers, without notes.
- [ ] Nightly reconciliation `@Scheduled` task live and correctly flagging mismatches.
- [ ] HLD Mock #4 completed and debriefed.

---

## Day 132 — HLD #14: Leaderboard System, and HLD #15: Search Autocomplete

### Theory Block (2.5 hrs)
- Topic: Leaderboard System
- **Redis Sorted Sets**, underneath, are backed by a Skip List — a probabilistic structure giving O(log n) insert/update/rank-query, which is exactly why Redis handles real-time rank updates so much more gracefully than a standard RDBMS B-tree index would under constant score changes. **Redis Cluster** for scale, and explicit tie-handling (same score, different rank — usually resolved by earliest achievement time).

- Topic: Search Autocomplete
- Directly building on the Tries work from Week 9: a **distributed Trie** serves hot queries from cache, with the underlying Trie itself updated asynchronously — search logs get aggregated, an offline batch job processes them, and a rebuilt Trie deploys on a schedule, not live per-keystroke — since real-time exactness genuinely isn't needed for autocomplete suggestions.
- Coding exercise: practice explaining, out loud, why a standard B-tree index struggles with real-time leaderboard rank updates compared to a Skip List — this exact comparison question comes up often enough to be worth having crisp.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: implement a simple leaderboard feature (e.g., "top-selling products this week") using Redis Sorted Sets.
- Definition of done: `ZADD`/`ZRANGE`/`ZRANK` correctly maintain and query the leaderboard as sales data changes.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: if a warm connection exists at any of the 7 target companies but no referral ask has happened yet, this is a reasonable point to make it — directly, briefly, and only once the relationship has had at least one real conversation behind it.

### Daily Deliverable
- [ ] Can explain why Skip Lists beat B-trees for this specific workload, without notes.
- [ ] Redis Sorted Set leaderboard live and correctly ranked on the platform.

---

## Day 133 (Sunday) — HLD #16: Google Drive / Object Storage, HLD Mock #5, and HLD Complete

### Self-Check (15 min)
- [ ] List every HLD system covered these two weeks from memory, and for each, name the one concept that's most distinctly *its* problem (e.g., Uber → geospatial indexing, Payment → idempotency).

### Theory Block (2.5 hrs)
- Topic: Google Drive / Object Storage
- **Chunking:** large files split into fixed-size blocks before upload — enables resumable uploads (only re-send the chunk that failed, not the whole file) and deduplication.
- **Deduplication:** hashing each chunk (content-addressable storage) means two users uploading the identical file only store it once — the client can even check "does this hash already exist server-side" *before* uploading, saving bandwidth entirely.
- **Sync conflicts:** two devices editing the same file offline, both reconnecting later — needs either last-write-wins (simple, can silently lose data) or a conflict-copy strategy (both versions kept, user resolves manually) — Drive-style products use the latter for anything non-trivial, since silent data loss is a much worse failure mode than an extra file to clean up.
- **Multipart upload to S3:** the real-world implementation of chunking, letting large files upload in parallel, resumable pieces.
- Coding exercise: write pseudocode for a file-chunking-and-hashing algorithm a client would run before uploading, to check for deduplication server-side first.

### HLD Mock #5 (1 hr)
- 45-minute HLD mock, your choice of subject from anything covered across these two weeks — same rule as the final LLD mock: pick whichever system you feel least confident defending, not the one that would go smoothest.

### Project Block (1 hr)
- Repository: `scalable-ecommerce-platform`.
- Task: none new — use this slot to review and update `docs/architecture.md` with everything designed across these two System Design weeks, since the capstone's remaining weeks build the platform this document describes.
- Definition of done: `docs/architecture.md` reflects the real, current intended architecture, not just Day 127's draft.

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard, and HLD is complete: Day 133.** Sixteen systems across two weeks — URL Shortener, Rate Limiter, Notification, Distributed Cache, ID Generation, Consensus, BookMyShow-at-scale, WhatsApp, Uber, Netflix, Instagram, Payment, Job Scheduler, Leaderboard, Search Autocomplete, Google Drive — on top of the fully-closed DSA curriculum and ten LLD systems. Five HLD mocks done and debriefed across the phase, matching LLD's five, for ten total design mocks before the capstone hardening and behavioral weeks even begin — several of them run in the specific format a target company actually uses (Uber's think-aloud style, the Atlassian product-reframe). Applications have now been live for two weeks across Rippling, Google, Databricks, Stripe, Uber, Atlassian, and Walmart Global Tech — this is a realistic point for first-round interviews to actually be scheduled. The remaining weeks harden the capstone platform itself and shift fully into interview execution.

### Daily Deliverable
- [ ] Google Drive HLD complete, including the chunking/dedup pseudocode.
- [ ] `docs/architecture.md` fully updated.
- [ ] HLD Mock #5 completed and debriefed. Application tracker current across all 7 companies. Weekly ritual and scorecard complete — **HLD phase closed.**
