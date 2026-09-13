# Week 18 (Revised): System Design Begins — Foundational Systems, and HLD Mocks Start Early

**What changed:** the 5-step System Design framework gets explicitly named and walked through step by step on Day 120, rather than assumed after one brief preview. HLD mock interviews start on Day 122 — after just the second system — instead of being saved entirely for the final week of the whole plan.

---

## Day 120 — HLD #1: URL Shortener, Applying the 5-Step Framework Explicitly

### Theory Block (2.5 hrs)
- Topic: URL Shortener — the Framework, Named at Every Step
- **Step 1, Requirements:** shorten a long URL, redirect on access, custom aliases (optional), analytics (optional) — clarify scope before designing anything. Say this step out loud to yourself even when practicing solo; naming it is what makes it a habit instead of something you skip under pressure.
- **Step 2, Estimation (do this math explicitly, every single time):** 100M new URLs/day → ~1,200 writes/sec average. Reads typically outnumber writes 10:1+ for this kind of system → ~12,000 reads/sec. Storage: 100M/day × 500 bytes/record × 365 days × 5 years ≈ 90TB. The exact number matters less than being visibly fluent doing this arithmetic live — interviewers watch *how* you estimate, not just whether the final figure is right.
- **Step 3, High-Level Design:** a Key Generation Service pre-generates unique short codes, avoiding collision-checking on the write path, using Base62 encoding (`[a-zA-Z0-9]`, 62^7 ≈ 3.5 trillion codes at 7 characters — comfortably enough). A cache sits in front of the database for hot redirects, since reads dominate.
- **Step 4, Detailed Design:** pick the piece that matters most here — the Key Generation Service — and go one level deeper: how does it avoid handing out the same code twice across multiple instances running concurrently?
- **Step 5, Bottlenecks:** the redirect path must be fast, since users feel that latency directly — cache aggressively. The database needs to handle write volume — consider sharding by short-code hash once single-node capacity is exceeded.
- Coding exercise: write a Base62 encoder/decoder in Java.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: the Base62 encoder/decoder above.
- Definition of done: encodes and decodes correctly round-trip for a range of test integers.

### Career Block (1 hr)
- Technical Blog: write and publish Blog Post 1 — "System Design: The 5-Step Framework, Applied to a URL Shortener."
- LinkedIn: share your blog post.

### Daily Deliverable
- [ ] Can walk through the URL Shortener design end to end, naming each of the 5 steps explicitly, including the estimation math, without notes.
- [ ] Base62 encoder/decoder pushed. Blog Post 1 published.

**Worth knowing as you start this phase:** Google's L4 loop specifically does not include a system design round — L4 engineers aren't expected to design systems there, so this isn't tested at that level. That doesn't make the next two weeks less important: Stripe, Rippling, Databricks, Atlassian, and Uber all test system design directly, and how you describe your own project's architecture (the platform) matters at every one of them, Google included, when discussing past work. Depth here isn't wasted on any target company — it's just not the thing Google's loop specifically probes.

---

## Day 121 — HLD #2: Rate Limiter, Formalized

### Theory Block (2.5 hrs)
- Topic: Rate Limiter — Beyond the Algorithm You Already Implemented
- You built a Token Bucket rate limiter for the platform's Gateway back in Week 10 — today formalizes the full landscape: **Token Bucket** (allows bursty traffic up to the bucket's capacity), **Leaking Bucket** (smooths output to a constant rate, no bursts allowed at all), **Fixed Window** (simple to implement, but allows up to 2x the intended limit right at a window boundary — a real, commonly-probed flaw), **Sliding Window Log** (accurate, but memory-heavy, since it stores every individual request timestamp).
- **Distributed rate limiting:** a single server's in-memory counter stops working the moment you have multiple instances, since the count needs to live somewhere every instance can see. Redis with a Lua script (an atomic check-and-increment in one round trip, avoiding a race between checking the count and incrementing it) is the standard answer.
- **Requirements to clarify:** rate limit per user, per IP, or per API key? What should happen on rejection — a hard 429 reject, or queuing the request instead?
- Coding exercise: none — you already have a working implementation; today is articulating the full trade-off space around it.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: upgrade the existing rate limiter to use Redis with a Lua script for atomic check-and-increment, so it works correctly across multiple Gateway instances instead of just one.
- Definition of done: running two Gateway instances behind a load balancer, the shared rate limit is still enforced correctly — not double the intended limit.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: research System Design architectures at your target companies specifically (e.g., how Stripe handles idempotency, how Uber handles surge pricing).

### Daily Deliverable
- [ ] Can compare all four rate limiting algorithms and their trade-offs from memory.
- [ ] Redis-backed distributed rate limiter live and verified across multiple instances.

---

## Day 122 — HLD #3: Notification System, and HLD Mock #1

### Theory Block (2 hrs)
- Topic: Notification System
- **Requirements:** channels in scope (Push, Email, SMS)? User preferences per channel? Priority levels?
- **High-Level Design:** a Notification Service receiving requests, a per-channel worker/queue (Push, Email, SMS each have different providers and different failure modes, so separate queues make sense), a Template service so notification content isn't hardcoded per call site.
- **Deduplication:** using Redis to track "was this exact notification already sent in the last N minutes" — prevents a retry storm from spamming a user.
- **Retry queues:** a failed send goes to a DLQ (the exact pattern already built on the platform's own Kafka listener) rather than blocking the queue or silently dropping the message.
- Coding exercise: design the database schema tracking user notification preferences and per-message delivery status.
- **Company-flavored variant, worth 10 minutes:** Atlassian's system design round asks you to design a piece of one of their actual products. Reframe today's system as "design Confluence's page-watch notification system" instead of a generic notifier — same architecture, but now you're fielding "what if a page has 10,000 watchers and gets edited every minute" instead of an abstract prompt. Practicing the reframe matters more than the specific answer.

### HLD Mock #1 (1 hr)
- 45-minute HLD mock with your accountability partner, using **URL Shortener** (Day 120) as the subject. The specific thing to get feedback on: did you narrate the 5 steps out loud in order, or did they blur together under time pressure? That's the exact failure mode this early mock is meant to catch, while it's cheap to fix.

### Project Block (1 hr)
- Repository: `scalable-ecommerce-platform`.
- Task: design the notification preferences/delivery-status schema above as an actual migration, even without full implementation yet.
- Definition of done: schema committed, matches the design from the theory block.

### Career Block (1 hr)
- LinkedIn: Post 23 — WhatsApp-style notification architecture breakdown.
- Networking: target SDE-2s specifically at Rippling, Google, Databricks, Stripe, Uber, Atlassian, and Walmart Global Tech on LinkedIn; send 5 connection requests.
- Check application status on the ones submitted Day 118 — this is roughly when a first response (screen invite or rejection) tends to arrive.

### Daily Deliverable
- [ ] Notification System design complete, including the DB schema.
- [ ] HLD Mock #1 completed and debriefed. LinkedIn Post 23 published.

---

## Day 123 — HLD #4: Distributed Cache

### Theory Block (2.5 hrs)
- Topic: Distributed Cache
- **Memcached vs. Redis:** Memcached is purely an in-memory key-value cache — simpler, multi-threaded. Redis adds richer data structures (sorted sets, lists, geospatial commands — several of which show up later this week), persistence options, and pub/sub, which is usually why it's the more versatile default choice today.
- **Consistent hashing in the caching layer:** directly reusing the `ConsistentHashingDemo` built back in Week 9 — as cache nodes are added or removed, only keys near the changed position on the ring get redistributed, not the whole keyspace.
- **Thundering Herd:** when a popular cache key expires, many concurrent requests can simultaneously miss the cache and all hammer the database at once. Mitigation: a mutex/lock so only one request repopulates the cache while others briefly wait, or serving slightly-stale data while one request refreshes it in the background.
- Coding exercise: none — verbally walk through the architecture of a global distributed cache in 15 minutes, timing yourself, out loud.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: add `@Cacheable`/`@CacheEvict` against Redis to the Product module (Cache-Aside pattern).
- Definition of done: fetching a product twice logs a real DB query only on the first fetch.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: reach out to a peer for a casual virtual coffee chat.

### Daily Deliverable
- [ ] Can explain Thundering Herd and its mitigation without notes.
- [ ] Cache-Aside live on the platform's Product module.

---

## Day 124 — HLD #5: Distributed ID Generation (A Real Gap in Most Self-Study Plans)

### Theory Block (2.5 hrs)
- Topic: Distributed Unique ID Generation
- A single auto-increment column stops working once you have multiple database shards or services generating IDs independently — you need globally unique IDs generated *without* central coordination on every single request. This is one of the most commonly asked "small" System Design questions, and it's genuinely absent from a lot of otherwise-thorough self-study material.
- **UUID:** simple, needs no coordination at all, but 128 bits (large) and not sortable by creation time — a real cost for database index locality, since inserts land randomly across the index instead of appending at the end.
- **Twitter Snowflake:** a 64-bit ID composed of a timestamp (sortable by creation order), a machine/worker ID (avoids collisions across different generators), and a sequence number (handles multiple IDs generated in the same millisecond on the same machine) — the standard answer at most tier-1 companies, worth knowing the exact bit layout, not just the name.
- **Database ID range allocation:** each service instance reserves a range of IDs (say, 1000 at a time) from a central counter, then hands them out locally — fewer round trips than requesting one ID per request, at the cost of some IDs being "wasted" if an instance restarts mid-range.
- Coding exercise: implement a simplified Snowflake ID generator in Java (timestamp + machine ID + sequence, packed into a `long`).

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: the Snowflake generator above, as a shared utility usable across every module on the platform.
- Definition of done: generates monotonically increasing, collision-free IDs under a quick concurrent-generation test.

### Career Block (1 hr)
- LinkedIn: Post 24 — "The ID generation problem you didn't know you'd have at scale" (Snowflake explained — genuinely differentiated content, since most candidates haven't studied this).
- Networking: follow up with any of the 7 target companies where a connection has gone warm but no conversation has happened yet.

### Daily Deliverable
- [ ] Can explain Snowflake's bit layout and why UUIDs alone aren't always sufficient, from memory.
- [ ] Snowflake generator implemented and pushed. LinkedIn Post 24 published.

---

## Day 125 — HLD #6: Distributed Consensus, and HLD Mock #2

### Theory Block (2 hrs)
- Topic: Distributed Consensus — Raft and Paxos, Conceptually
- Replication assumes you already know who the leader is. Consensus algorithms solve the harder problem underneath: how do a set of nodes *agree* on a single value (including "who is the leader") even when some nodes fail or messages are delayed — without ending up in a split-brain state where two nodes both believe they're the leader.
- **Quorum:** the core idea both algorithms share — a decision only becomes final once a *majority* of nodes acknowledge it, which guarantees any two quorums necessarily overlap in at least one node, and that overlap is exactly what prevents two conflicting decisions from both being finalized.
- **Paxos** is the original, and famously difficult to reason about correctly. **Raft** was explicitly designed to be more understandable, splitting the problem into leader election, log replication, and safety — and is what most production systems actually implement today (etcd, which sits underneath the Kubernetes Control Plane, uses Raft).
- You are not expected to implement Raft — the goal is being able to explain *why* consensus is needed and what a quorum guarantees, cleanly, if it comes up.
- Coding exercise: none — write 150 words explaining, in your own words, why single-leader replication still needs a consensus mechanism to safely handle leader failure and election, rather than just picking any surviving node.

### HLD Mock #2 (1 hr)
- 45-minute HLD mock, using **Rate Limiter** as the subject — specifically practice defending the choice between the four algorithms under a "why not just use X instead" style follow-up, which is the most common way this question gets pushed on in a real interview.

### Project Block (1 hr)
- Repository: `scalable-ecommerce-platform`.
- Task: a short `docs/consensus-notes.md` connecting today's theory back to etcd's role in Kubernetes, which you've already relied on for weeks without necessarily naming the mechanism underneath it.
- Definition of done: pushed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: follow up on any recruiter responses.

### Daily Deliverable
- [ ] Can explain quorum and why it prevents split-brain, without notes.
- [ ] Consensus notes pushed. HLD Mock #2 completed and debriefed.

---

## Day 126 (Sunday) — HLD #7: BookMyShow at Scale

### Self-Check (15 min)
- [ ] Solve one Backtracking or Trie problem cold, without hints — the DSA muscle needs occasional upkeep even during the LLD/HLD-heavy stretch.

### Theory Block (2.5 hrs)
- Topic: BookMyShow — From LLD to HLD
- You already solved the *object design* and *concurrency* for a single booking a couple of weeks ago. Today scales it up: **multiple theaters, multiple cities** — sharding by city/region makes sense, since a booking never spans cities. **Seat-hold expiry** — a Redis TTL-based hold (5-10 minutes) so an abandoned booking flow doesn't lock a seat forever, releasing it automatically. **High-demand launches** (a blockbuster's opening night) — this is where Distributed Cache and Thundering Herd knowledge applies directly, since everyone hits the same "seat map" data at once.
- **Estimation:** 1000 shows/day across a city, 200 seats/show average, peak booking window concentrated in a 2-hour launch window — work through the peak QPS math explicitly.
- Coding exercise: draw the full HLD diagram — Gateway, Booking Service, Redis (locks + cache), Payment Service, Database (sharded by city) — labeling where each of this week's concepts (rate limiting, distributed cache, ID generation, consensus-backed leader election if relevant to your locking choice) actually shows up.

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 126, one week into System Design. Seven foundational systems complete (URL Shortener, Rate Limiter, Notification, Distributed Cache, Distributed ID Generation, Distributed Consensus, BookMyShow-at-scale), and two HLD mocks already done and debriefed — the original plan's entire HLD mock allocation was exactly one, positioned right before real interviews began. Applications have been live for a week across all 7 target companies; track responses as they come in, and if any recruiter screen gets scheduled, treat it as real signal to accelerate the corresponding mock rehearsal for that specific company's format.

### Daily Deliverable
- [ ] BookMyShow-at-scale HLD diagram complete, explicitly connecting this week's concepts to the design.
- [ ] Weekly ritual and scorecard complete. Application tracker updated with current status across all 7 companies.
