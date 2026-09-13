# Week 19 — Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Companion to:** `Week_19_Revised.md` (Days 127–133)

Every question from every day this week, pulled into one review document — 100 questions across seven days, closing the HLD phase at sixteen systems total across Weeks 18–19. Organized by day, in the same order the material was taught, so a review pass follows the same dependency order the teaching did. Use this for spaced review after the week's Resource Books themselves have already been worked through once — it's a review tool, not a substitute for the fuller explanations, worked traces, and mock-interview scripts in each day's own book.

---

## Day 127 — WhatsApp / Chat System

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

## Day 128 — Uber Driver Location Tracking

**Q1. Encode, roughly, why a shared geohash prefix implies proximity.** Each bit halves the active range in one dimension; two points sharing a prefix of length k survived the identical sequence of halvings, meaning both fell in the same shrinking cell at every step — only possible if they started close together.

**Q2. What's the geohash boundary problem, and how is it handled?** Two genuinely nearby points can straddle a cell edge and get different prefixes entirely. Correct nearby-search checks the query cell's 8 neighbors too, not just an exact-prefix match.

**Q3. Does a longer geohash always represent the same physical cell size everywhere on Earth?** No — longitude lines converge toward the poles, so the same string length represents a smaller physical width at high latitudes than near the equator.

**Q4. When does a quadtree earn its extra complexity over a geohash?** When density is genuinely skewed (dense cities vs. sparse rural areas) — a quadtree's subdivision tracks actual point density; a geohash grid is fixed-precision everywhere regardless of how points are actually distributed.

**Q5. What data structure actually backs Redis Geo, and what does that buy you?** A Sorted Set, keyed by a 52-bit interleaved geohash score — `GEOSEARCH` is a sorted-set range query wearing a geospatial API, inheriting Sorted Sets' O(log n) contract (named Day 123, mechanism taught Day 132) for free.

**Q6. Why is invoking CAP theorem for a single-machine cache wrong, precisely?** CAP describes a trade-off that only exists during a network partition between multiple nodes; a single machine has no other node to partition from, so there's no distributed system to reason about in the first place.

**Q7. What's the generalizable lesson behind the CAP-misapplication anecdote?** State *why* a concept applies before invoking it — vocabulary breadth isn't the bar; precision about preconditions is.

**Q8. Why is this system write-dominated rather than read-dominated, and what does that change about the design instinct?** Every active driver pings every few seconds; riders search comparatively rarely. It changes the instinct away from "add a read cache" (this series' usual reflex) toward optimizing the write path itself — sharding and cheap in-place updates matter more than read caching here.

**Q9. Why does this system's storage stay roughly constant instead of growing over time, unlike Day 127's message store?** Location writes overwrite an existing member rather than appending a new one — one entry per active driver at any moment, not one entry per event ever sent.

**Q10. Why is geographic sharding preferred over a naive hash-based shard here?** Hash-based sharding scatters physically-adjacent drivers across unrelated shards at random, which defeats a spatial query entirely; geographic sharding is Day 78's range-based sharding, applied with "range" reinterpreted as physical region — matching the query's own locality.

**Q11. What happens to a driver whose app crashes without a clean disconnect?** Without mitigation, their last known position would sit in the index forever. A TTL on each entry, refreshed by every successful ping, lets a driver who stops reporting silently expire out of search results.

**Q12. In today's mock, what's the actual difference between "confident-sounding" and "genuinely convincing" under a 'why not simpler' push?** A genuinely convincing answer grounds the choice in a specific complexity or failure-mode argument (an O(...) claim, a named concrete failure); confident-sounding restates the choice more firmly without adding new information.

**Q13. Why does Uber's interview format specifically penalize over-engineering, and what does that look like in practice?** It's testing whether a candidate reaches for the mechanism the stated requirements actually justify, not the most sophisticated one available — proposing, say, a full quadtree implementation when the actual density is roughly uniform would be exactly this failure, penalized even if technically correct.

**Q14. Why is Redis Geo usually the pragmatic real answer instead of hand-rolling a quadtree?** It gives geohashing's simplicity and off-the-shelf tooling (an existing, battle-tested Sorted-Set-backed implementation) without requiring a bespoke tree structure to build and maintain — the right default unless density skew specifically justifies the extra complexity.

---

## Day 129 — Netflix / Video Streaming & Instagram Feed

**Q1. Why does adaptive bitrate streaming need chunks to be independently decodable and aligned across renditions?** Alignment lets the player switch which rendition it's pulling from exactly at a chunk boundary with no visible glitch; without independent decodability, switching mid-stream would require re-establishing decoder state, breaking playback.

**Q2. How does the player actually decide when to switch quality?** It measures how long each chunk download took relative to that chunk's own playback duration, estimating current throughput, and picks the next chunk's rendition accordingly — a continuous feedback loop, not a one-time choice.

**Q3. In what precise sense is a CDN "Cache-Aside at the edge"?** Both check a cache first, fall through to a slower source on a miss exactly once, populate the cache, and serve subsequent nearby requests from it — the same shape, relocated from an application-adjacent cache to a geographically distributed edge tier.

**Q4. Why does that reframing predict CDNs inherit Thundering Herd?** Because Thundering Herd is a property of the Cache-Aside shape itself (many concurrent misses on the same key racing to repopulate it) — recognizing the shape means recognizing the failure mode comes with it, at whatever layer the shape appears.

**Q5. What's the CDN-layer equivalent of Day 126's proactive cache warming?** Pre-populating edge caches with a hugely anticipated new release ahead of its scheduled drop time, so the release moment doesn't trigger a simultaneous global cache-miss stampede to origin.

**Q6. Why does a CDN not eliminate origin load uniformly?** It removes load for popular, frequently-cached content specifically; long-tail, rarely-requested content is seldom cached anywhere and still pays close to full origin latency.

**Q7. Contrast push and pull fan-out on both write cost and read cost.** Push: cheap, fast reads (precomputed), write cost proportional to follower count. Pull: trivial writes, read cost proportional to follow count, paid on every single feed open.

**Q8.** [Trace] **Using the 100M-follower example, why does even a generous 50,000 writes/sec fan-out pipeline still fail here?** 100,000,000 ÷ 50,000 ≈ 2,000 seconds — over half an hour to clear one post's fan-out, during which followers see a stale feed and the pipeline is saturated by a single post.

**Q9. Why is hybrid fan-out not just "a compromise," but the actually-correct answer?** It matches cost to where it's cheap: push for the many accounts where per-post cost is small, pull for the few accounts where push's cost would be enormous — rather than accepting one strategy's worst case everywhere.

**Q10. In hybrid fan-out, why does the pull-time merge stay cheap even as a user's total follow count grows?** The pull only ever spans the small number of celebrity accounts a user follows, not their entire follow list — bounded by how many mega-accounts realistically exist, not by the user's overall following count.

**Q11. Name the three prior places in this series this same underlying tension already appeared, before today's actual fix.** Day 78 (sharding's "celebrity problem," named); Day 122 (notification per-channel fan-out, small scale); Day 127 (WhatsApp's hot group-chat partition).

**Q12. Why must a transcoding pipeline run asynchronously rather than during upload?** Encoding every rendition is CPU/GPU-intensive and can take minutes; doing it synchronously would force the uploader to wait unacceptably, and would block the upload path from scaling independently of encoding capacity.

**Q13. Why does Netflix's bandwidth estimation matter as more than a scale statistic?** It's the direct justification for the CDN itself — at multiple-terabit sustained scale, serving every request from one origin isn't just slower, it's infeasible on any single origin's network capacity.

**Q14. What determines whether an account should be on the push path or the pull path in a hybrid system?** A follower-count threshold — accounts below it get pushed to normally; accounts above it are pulled in at read time instead, since their follower count is what makes push expensive in the first place.

---

## Day 130 — Payment System

**Q1. Why must the client, not the server, generate the idempotency key?** Only the client knows whether a given request is a fresh action or a retry of one already sent — a server-generated key on each incoming attempt would treat every retry as a brand-new intent, defeating the entire mechanism.

**Q2. What's the single most common real bug in idempotency-key implementations?** Generating a new key on every retry attempt instead of reusing the original key tied to the user's actual intent.

**Q3. Why does the mechanism need an explicit `in-progress` status, not just `completed` vs. `not found`?** Without it, two genuinely concurrent duplicate requests could both see "not found" and both proceed to charge — `in-progress` is what a second, racing request checks against to correctly back off instead.

**Q4.** [Trace] **Walk through the unsafe distributed-lock release bug end to end.** A holds a lock past its TTL due to a stall; the lock auto-expires; B acquires it legitimately; A wakes up and blindly deletes the key, removing B's active lock; C then acquires the same lock while B still believes it holds it — two processes now proceeding concurrently on what was meant to be exclusive.

**Q5. What's the specific fix for the unsafe-release bug?** Release must atomically check that the current lock value is still the releasing process's own token before deleting — via a single Lua script, since a separate `GET` then `DEL` leaves a race window.

**Q6. Why is a Redis `SETNX`-based lock preferred here over Day 124's `synchronized` block?** `synchronized` only protects against concurrent threads within a single JVM; a distributed lock is needed because the payment service runs as multiple separate instances, none of which share memory.

**Q7. What's the actual difference between what an idempotency key protects against and what a distributed lock protects against?** An idempotency key stops the *same* logical request from being processed twice, even long after the first completed. A distributed lock stops *different*, genuinely concurrent operations from touching the same resource at the same instant. Neither substitutes for the other.

**Q8. Why does the lock's TTL involve a genuine trade-off with no perfect setting?** Too short, and the lock can expire while the work is still legitimately in progress, letting someone else in prematurely (today's unsafe-release scenario). Too long, and a real crash leaves everyone else blocked longer than necessary. The right value is a judgment call grounded in expected work duration, not a fixed default.

**Q9. What is Redlock, and why would a system reach for it over single-instance `SETNX`?** Acquiring the lock across a majority of several independent Redis instances (Quorum, Day 125, reapplied) before considering it held — needed when a single Redis instance's own availability is itself an unacceptable single point of failure for the lock's guarantee.

**Q10. Why is an account's balance derived rather than stored directly in a double-entry ledger?** The append-only entry log is the actual source of truth; a stored balance column, where it exists for read performance, is explicitly a cache of what the entries already prove, not an independent fact.

**Q11. In what precise sense is a double-entry ledger "self-checking"?** Every entry has an equal-and-opposite twin, so the sum across the entire ledger must always equal exactly zero — any deviation is an immediate, provable signal of a bug, not something that needs to be independently discovered.

**Q12. Why is an append-only ledger safer under concurrency than an incrementable balance field?** Two concurrent debits against an append-only log simply become two independent new rows, both preserved correctly with no race; two concurrent increments against one mutable field risk a lost update without careful locking discipline.

**Q13. Of the four earlier appearances of "check-then-act" in this series (Days 121, 122, 124, 126), what's genuinely new about today's treatment rather than a repeat?** Today generalizes from "does this resource already exist" (the earlier four) to "has this exact logical intent already been fully processed, and can two truly concurrent copies of it be prevented from both succeeding" — combined, for the first time, with the ledger's own independent correctness guarantee.

**Q14. Why does a real payment system need both idempotency keys and distributed locks rather than just one?** They guard against different failure shapes — a retried request needs idempotency; two unrelated concurrent operations on the same order need mutual exclusion. A system with only one of the two is still vulnerable to the failure the other one prevents.

**Q15. What's a concrete real bug that idempotency-key mismatch detection specifically catches?** A client bug that reuses the same idempotency key for two requests with genuinely different payloads (say, two different charge amounts) — without payload comparison, the server would silently return the *first* request's cached result for a request that was actually asking for something different.

---

## Day 131 — Distributed Job Scheduler

**Q1. What is SQS Visibility Timeout, mechanically?** When a consumer receives a message, it becomes invisible to other consumers for a set window; if not explicitly deleted within that window, it becomes visible again for reclaiming — a lease with an expiry.

**Q2. What earlier concept in this series is Visibility Timeout structurally identical to?** Day 130's TTL-based distributed lock — same lease-with-expiry shape, applied to a queue message instead of a payment resource.

**Q3. Why does deleting a message on receive (rather than on completion) break the at-least-once guarantee?** A consumer that crashes mid-processing after the message was already deleted would silently lose it forever — at-least-once specifically requires the message to survive until completion is *confirmed*, not merely attempted.

**Q4. What's a concrete real bug caused by setting the visibility timeout too short?** A job that genuinely takes longer than the timeout gets reclaimed and run again by a second worker while the first is still legitimately processing it — duplicate execution caused purely by misconfiguration, not a flaw in the mechanism.

**Q5. Why does a hand-rolled Redis Sorted Set queue need extra work that SQS gives for free?** SQS's visibility timeout is a built-in queue feature; a Sorted Set poller has to build that same leasing discipline itself, typically using the exact SETNX-with-TTL pattern from Day 130.

**Q6. What's a misfire, and name two different correct policies for handling one.** A scheduled fire time passing without the job running (usually because the scheduler was down). "Fire now, once" suits jobs where late-but-once is fine; "ignore and wait for the next scheduled time" suits jobs tied to a specific moment's meaning that a late run can't actually represent.

**Q7. Why is Quartz's cron format different from plain Unix cron, precisely?** Quartz uses six fields with seconds first, and requires a `?` wildcard in exactly one of day-of-month or day-of-week since specifying both is contradictory; plain Unix cron has five fields and no `?`.

**Q8. Why do distributed schedulers commonly guarantee at-least-once rather than true exactly-once?** True exactly-once would require distributed consensus over "did this complete" and "will nothing else attempt it" — expensive and complex. It's cheaper to accept occasional duplicate delivery and require the job handler to be idempotent instead.

**Q9. What does "effectively-once" mean, and how does it differ from true exactly-once?** At-least-once delivery combined with idempotent processing, producing an outcome indistinguishable from exactly-once *from the outside*, even though the underlying mechanism genuinely may retry.

**Q10. Why must the polling Lua script combine the range-read and the removal into one atomic operation?** Two separate commands leave a window where a second poller could read the same due jobs before the first removes them, causing both to claim and process the same job.

**Q11. This is the fifth appearance of the same underlying shape in this series — name at least three of the other four.** Day 121 (rate limiting), Day 122 (notification dedup), Day 126 (seat-hold), Day 130 (idempotency claim and safe lock release).

**Q12. In today's mock, what's the real gap in a naive idempotency design that push 2 exposed?** A charge that succeeds at the gateway but crashes before the local `completed` record is written leaves the system's own store out of sync with what actually happened externally — closed by also passing the idempotency key to the gateway itself as a second source of truth.

**Q13. Why does `@Scheduled(cron = ...)` in Spring not give you Quartz's actual misfire-policy configuration?** `@Scheduled` uses Spring's own cron parser for scheduling syntax only; Quartz's configurable `MISFIRE_INSTRUCTION_*` policies require wiring actual Quartz beans, not just the annotation.

**Q14. Why is LC 1143 (Longest Common Subsequence) a reasonable single problem to represent "is my DP still sharp," rather than an arbitrary pick?** It's the foundational 2D-string-DP recurrence that Edit Distance directly generalizes — a clean, minimal test of the exact recognition skill a harder, already-revised problem builds on.

---

## Day 132 — Leaderboard System & Search Autocomplete

**Q1.** [Trace] **Search a Skip List for a value using the diagram in Part 2 and count the comparisons.** Starting at the top level, moving right while the next node's value doesn't exceed target, dropping a level whenever it does, until level 0 confirms the value — reaching 13 in the example took 5 comparisons while skipping 6 of the structure's 11 nodes entirely.

**Q2. Why is expected Skip List search O(log n), argued from the mechanism, not just stated?** O(log n) expected levels (each level holds roughly half the elements of the one below), and O(1) expected rightward hops per level before dropping down — multiplying gives O(log n) expected total.

**Q3. Why is node height chosen randomly rather than by a fixed promotion rule?** A deterministic promotion scheme must actively rebalance other nodes to preserve its exact invariant on every insert; random height makes insertion a purely local splice, touching no other existing node.

**Q4. Is Skip List's O(log n) a worst-case guarantee?** No — it's expected/probabilistic. An adversarially unlucky sequence of random heights could in principle degrade it, but this is vanishingly unlikely in practice and requires no active rebalancing to avoid, unlike an unbalanced BST's easily-triggered worst case.

**Q5. Why does a B-tree specifically struggle with a leaderboard's real-time rank updates?** Every write can trigger a node split or rebalance to preserve structural invariants, which is expensive precisely when writes (score changes) are constant rather than occasional — the leaderboard's actual access pattern.

**Q6. How does `ZRANK` compute an exact rank in O(log n)?** Each forward pointer is augmented with a span (how many level-0 elements it skips); summing spans traversed during the same search that locates the member yields its exact rank in one pass.

**Q7. Is Redis Cluster's sharding scheme the same thing as Consistent Hashing?** No — Redis Cluster uses a fixed 16,384 hash-slot scheme and handles membership changes by explicitly migrating slots; Consistent Hashing minimizes remapping via a hash ring's geometric properties. Different mechanisms solving a similar-sounding problem.

**Q8. What breaks when a Sorted-Set-backed leaderboard is sharded across a Redis Cluster?** A single `ZRANK` only reflects rank within that member's own shard, not the true global rank across the whole sharded dataset.

**Q9. Why is a composite score the right way to implement "earliest achievement wins" tie-handling?** Redis's default tiebreak (lexicographic by member name) carries no achievement-time meaning; encoding time directly into the score, scaled so it can never override a genuine score difference, gives semantically correct ordering using the structure's native comparison.

**Q10.** [Trace] **Given scores of 1000 at timestamps 5 and 8 with `BIG_CONSTANT = 1,000,000`, which combined score is higher, and does that match the intended rule?** 999,999,995 vs. 999,999,992 — the earlier timestamp (5) produces the higher combined score, correctly ranking the earlier achievement above the later one at an identical raw score.

**Q11. What has to be true of `BIG_CONSTANT` for the tiebreaker to never distort genuine score differences?** It must exceed the full realistic range of the timestamp term, so that even a 1-point difference in actual score outweighs any possible timestamp-driven difference.

**Q12. What's genuinely new about today's Trie usage, given Tries were fully taught back in Week 9?** Not the structure — the deployment pattern: a read-only, replicated structure built offline in batch and served without any live write path, rather than a Trie maintained under continuous live updates.

**Q13. Why is precomputing top-K completions per Trie node worth the extra build-time cost?** Without it, a query costs a full subtree traversal from the prefix node to find the best completions; with it, the same query costs O(1) beyond the O(L) traversal to reach the node.

**Q14. Why is asynchronous, scheduled rebuild the correct choice for autocomplete specifically, and not a universal rule?** Autocomplete tolerates suggestions being up to one rebuild-interval stale, which buys much simpler, cheaper serving infrastructure — a trade-off that is explicitly wrong for a system like Day 130's payments, which cannot tolerate equivalent staleness.

**Q15. Contrast the read/write balance of today's two systems.** The leaderboard is read-dominated with a real, non-trivial write load (frequent score updates); autocomplete is overwhelmingly read-dominated with near-zero live write volume, since all writing happens offline in batch.

---

## Day 133 — Google Drive / Object Storage

**Q1. Why does fixed-size chunking break deduplication after a small edit, concretely?** An edit shifts every subsequent byte's position, so every chunk boundary after the edit point lands somewhere different, changing the content — and therefore the hash — of every downstream chunk, even though almost none of the actual content changed.

**Q2. Why doesn't content-defined chunking have this problem?** Its boundaries are chosen by a local condition on a small sliding window of bytes, not by absolute position — once an edit has scrolled out of that window, boundary decisions downstream return to exactly where they were, and those chunks dedupe correctly again.

**Q3. What does content-addressable storage give you "for free" beyond deduplication?** Integrity verification — since a chunk's identifier is derived from its content, re-hashing a retrieved chunk and comparing it to its claimed hash detects corruption or tampering.

**Q4. What's the correct, precise answer to "what if two different chunks hash to the same value"?** The probability with a strong cryptographic hash at realistic storage scale is small enough to be an accepted, standard trade-off in production — worth naming honestly, not dismissed as impossible or over-engineered against.

**Q5. Why is conflict-copy the right choice for a file-sync system specifically, rather than a universal best practice?** Silently discarding a user's real edit (LWW's failure mode) is a much worse outcome here than asking the user to occasionally resolve a conflict by hand — the right trade-off is a property of what's being synced, not a fixed rule.

**Q6. What's a failure mode of Last-Write-Wins beyond "it can lose data"?** Clock skew across devices — a device with a forward-skewed clock can make a genuinely older edit appear to have happened later, incorrectly winning and discarding real, newer work.

**Q7. Is multipart upload the same idea as content-defined chunking, since both split a file into pieces?** No — multipart upload is a transport-level mechanism for reliable, parallel delivery of bytes to storage; content-defined chunking is a storage-level mechanism for deduplication. Related in spirit, solving different problems, often combined but not interchangeable.

**Q8. Why is the estimation step (5 exabytes before dedup) more than a scale statistic here?** It's the direct economic justification for building deduplication at all — even a modest dedup ratio at that scale is a substantial, concrete storage-cost reduction, not an abstract nicety.

**Q9. How would a Chunk Index itself be scaled at extreme volume?** Sharded by hash prefix — the same locality-preserving instinct behind consistent hashing and range-based sharding, applied here to a lookup index rather than a primary data store.

**Q10. Recall, without looking: which system introduced the double-entry ledger, and which introduced Skip Lists?** Payment System (Day 130) — idempotency keys, distributed locks, double-entry ledger. Leaderboard (Day 132) — Skip Lists, built fully from scratch for the first time.

**Q11. Why was media sharing named as a requirement on Day 127 but not designed until today?** It's a deliberate deferral — the actual mechanics (chunking, dedup, multipart upload) belong to object storage as their own system, and Day 127's diagram correctly treated media as delegated to that layer rather than improvised from scratch mid-answer on a system where it wasn't the distinguishing problem.

**Q12. What's the actual criterion for choosing an HLD Mock #5 subject, and why that criterion specifically?** The system you feel least confident defending, not the one that would go smoothest — because the mock's value is proportional to genuine pressure-testing, and a comfortable choice tests nothing that wasn't already solid.

**Q13. Across all sixteen systems, name two that were explicitly framed as opposite ends of the same spectrum.** Uber's location tracking (write-dominated, read-light) and most other systems in the series including the Leaderboard (read-dominated) — an explicit, deliberately named contrast rather than treating every system as needing the same "add a cache" instinct.

---
