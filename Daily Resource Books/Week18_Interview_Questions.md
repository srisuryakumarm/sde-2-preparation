# Week 18 Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books · [Curriculum Map](./00_Curriculum_Map.md)
**Consolidates:** [Day 120](./Day120_Resource_Book.md) · [Day 121](./Day121_Resource_Book.md) · [Day 122](./Day122_Resource_Book.md) · [Day 123](./Day123_Resource_Book.md) · [Day 124](./Day124_Resource_Book.md) · [Day 125](./Day125_Resource_Book.md) · [Day 126](./Day126_Resource_Book.md)

Every question from every day this week, in one place, for spaced review. 100 questions total. Organized by day, in the order each topic was taught — review in order the first pass, then jump around on later passes once the material is solid.

---

## Day 120 — System Design Begins: URL Shortener

---

**1. Name the 5 steps of the System Design framework, in order, and state in one sentence what each is for.**

*Answer:* Requirements (what to build and what's out of scope), Estimation (traffic/storage math, done live), High-Level Design (the major components and how a request flows through them), Detailed Design (go one level deeper on the one piece that matters most), Bottlenecks & Scale (where does this break first, and what fixes it).

---

**2. How does this framework differ from the LLD 5-step framework from Day 106, beyond both happening to have 5 steps?**

*Answer:* The LLD framework answers "how do I model this as classes inside one process" and produces working code. The System Design framework answers "how does this system run correctly across multiple machines at a given scale" and produces an architecture — code only for whichever single piece gets the Detailed Design treatment.

---

**3. What is a System Design interviewer actually evaluating during the Estimation step — the final number, or something else?**

*Answer:* The process — fluently converting a daily figure into a per-second rate, narrating the arithmetic live. Landing within roughly 20% of a defensible number is fine; freezing or asserting a number without showing the derivation is the real failure mode.

---

**4. Walk through the writes/sec calculation for 100M new URLs/day.**

*Answer:* 100,000,000 ÷ 86,400 seconds/day ≈ 1,157 writes/sec, rounded to ~1,200 for planning purposes. 86,400 = 24 × 60 × 60.

---

**5. Why do reads outnumber writes for a URL shortener, and roughly by how much?**

*Answer:* Each URL is created once but clicked many times afterward — commonly modeled at a 10:1 read:write ratio or higher, giving roughly 12,000 reads/sec against ~1,200 writes/sec here. That ratio is why the design leans so heavily on caching the redirect path specifically.

---

**6. What is Base62 encoding, and why 62 symbols rather than 10 or 16?**

*Answer:* A positional numeral system using 62 symbols (0–9, A–Z, a–z) instead of decimal's 10 or hex's 16. More symbols per position means shorter strings represent the same range of values, and the specific 62-symbol alphabet is URL-safe and case-sensitive, both properties a short link wants.

---

**7. [Trace] Hand-encode the integer 125 in Base62 and show each step.**

*Answer:* 125 % 62 = 1, append '1', n becomes 125/62 = 2. Then 2 % 62 = 2, append '2', n becomes 0, loop ends. Built string "12", reversed to "21". Decoding "21" gives 2×62+1 = 125, confirming the round trip.

---

**8. Why does the reversal step in `encode()` matter — what breaks without it?**

*Answer:* The div/mod loop naturally produces digits least-significant-first (the same reason manual decimal-to-binary conversion reads backward). Skipping the reversal doesn't crash — it silently produces a different, wrong-but-valid-looking Base62 string, which is a worse kind of bug than one that throws.

---

**9. Why shouldn't the Key Generation Service just Base62-encode a simple auto-incrementing counter?**

*Answer:* Sequential codes are predictable — anyone can enumerate every shortened URL in the system by requesting code, code+1, code+2, and so on, a real information-leak and scraping risk. The KGS pre-generates random codes specifically to avoid this.

---

**10. How does `SELECT ... FOR UPDATE SKIP LOCKED` prevent two KGS instances from handing out the same code, and how does it differ from the plain `FOR UPDATE` used for BookMyShow (Week 17, Day 117)?**

*Answer:* It locks and returns one currently-unlocked row, skipping any row a concurrent transaction has already locked, rather than blocking on it. BookMyShow's `FOR UPDATE` needed two transactions contending for one specific seat to genuinely wait on each other; here, any unclaimed code works equally well for either instance, so `SKIP LOCKED` lets both proceed in parallel against different rows instead of one waiting unnecessarily.

---

**11. What's the trade-off between the DB-atomic-claim approach and statically partitioning the code pool by instance?**

*Answer:* Static partitioning avoids touching a shared database row per claim, which is faster per-operation, but rebalancing the unclaimed pool when an instance is added or removed is a genuinely harder coordination problem. DB-atomic-claim scales instance count up or down with zero reconfiguration, at the cost of a shared-row operation per claim.

---

**12. Why does a cache belong in front of the URL Mapping DB specifically for this system?**

*Answer:* Step 2 established reads outnumber writes roughly 10:1, and Step 1 established the redirect path is the latency users feel directly — caching hot redirects keeps the vast majority of read traffic off the database entirely.

---

**13. Why hash-based sharding rather than range-based for the URL Mapping table specifically?**

*Answer:* Range-based sharding (Week 12, Day 78) earns its cost only when range queries need to stay cheap — this system never runs a range query, only exact-match lookups by short code, so hash-based sharding's even-load property is a pure win. In production this is virtually always consistent-hash-based (Week 9, Day 60), specifically to avoid a near-total remap when the shard count changes.

---

**14. A URL shortener wants to offer click analytics. Should redirects use HTTP 301 or 302, and why?**

*Answer:* 302 — a 301 gets cached by the browser after the first click, so every subsequent click bypasses the server entirely and analytics silently stop counting them. 302 forces every click to round-trip through the server, required for accurate counts even at higher server load.

---

**15. At 100M new URLs/day for 5 years, roughly what fraction of the 7-character Base62 keyspace gets consumed?**

*Answer:* Total URLs over 5 years ≈ 100M × 365 × 5 = 182.5 billion. The keyspace is 62⁷ ≈ 3.52 trillion. 182.5B ÷ 3.52T ≈ 5.2% — comfortably within capacity, the concrete arithmetic behind choosing 7 characters.

---

## Day 121 — Rate Limiter, Formalized

---

**16. Name all four rate limiting algorithms and, for each, the one-sentence core trade-off.**

*Answer:* Token Bucket — allows bursts up to capacity, O(1) memory. Leaking Bucket — smooths to a strictly constant output rate, no bursts survive it. Fixed Window — cheapest, but allows up to 2x the limit at a window boundary. Sliding Window Log — exact rolling accuracy, at O(limit) memory cost.

---

**17. [Worked example] Demonstrate Fixed Window's boundary flaw with concrete numbers.**

*Answer:* Limit 100/minute. 100 requests at 11:59:59 fill that window's counter to exactly 100. At 12:00:00 the counter resets. 100 more requests at 12:00:01 fill the new window to 100 as well. Total: 200 requests allowed within about 2 real seconds, against a stated limit of 100/minute.

---

**18. Why does Sliding Window Log not suffer from that same flaw?**

*Answer:* It anchors the window to "now," not to a fixed clock boundary, so there's no boundary to straddle — at any instant, it counts exactly the requests within the trailing window, giving a true rolling limit.

---

**19. What's the key behavioral difference between Token Bucket and Leaking Bucket?**

*Answer:* Token Bucket controls admission and allows bursts up to the bucket's capacity when idle time has accumulated tokens. Leaking Bucket controls the processing rate and enforces a strictly constant output rate regardless of how the input arrived.

---

**20. Why does a single in-memory rate-limit counter break once a service runs as multiple instances behind a load balancer?**

*Answer:* Each instance holds its own independent counter/bucket. A request's outcome depends on which instance the load balancer routes it to, so the effective combined limit across N instances approaches N times the intended limit, silently.

---

**21. Why is Redis the standard fix, rather than a shared relational database?**

*Answer:* The rate-limit check runs on every single request, needing sub-millisecond round trips — an in-memory store like Redis fits that hot-path latency requirement in a way a relational database's connection/transaction overhead doesn't.

---

**22. What race condition does an un-atomic "check count, then increment" have, and what established pattern is it the same shape as?**

*Answer:* Two instances can both read a count below the limit, both decide to allow, and both increment — letting the true count exceed the limit. Identical shape to the check-then-act race already seen in Parking Lot (Week 16, Day 111), the ATM (Week 17, Day 113), and BookMyShow (Week 17, Days 116–117).

---

**23. Why does wrapping the check-and-increment in a Lua script make it atomic in Redis specifically?**

*Answer:* Redis executes commands on a single thread, and a submitted Lua script runs as one indivisible unit — no other client's command can execute partway through it.

---

**24. [Trace] In the Token Bucket Lua script, a bucket with capacity=10, refill_rate=1/sec was last touched 15 seconds ago. What happens?**

*Answer:* elapsed = 15, so tokens + 15 far exceeds capacity; `math.min(capacity, ...)` caps the refill at 10 — the bucket is treated as fully refilled, correctly, since it sat idle longer than it takes to fill from empty.

---

**25. Why shouldn't a Redis Lua script call `os.time()` directly to get the current time?**

*Answer:* Redis deliberately disables non-deterministic Lua functions like `os.time()` inside scripts, since script behavior must be reproducible for replication. Time should come from an argument passed in by the caller, or from `redis.call('TIME')`.

---

**26. When would you rate-limit per-IP instead of per-user, and what's the downside?**

*Answer:* Per-IP catches abuse from a source regardless of whether it's authenticated. The downside: many real users can share one IP (a corporate NAT, a campus network), so per-IP limiting can unfairly throttle all of them for one bad actor's traffic.

---

**27. What's the trade-off between hard-rejecting with a 429 and queuing an over-limit request instead?**

*Answer:* A hard reject is cheap and immediate but drops the request entirely. Queuing smooths bursts at the cost of added latency, and the queue itself needs a bound — an unbounded queue just relocates the overload problem.

---

**28. What does HTTP 429 specifically mean, and what's a well-behaved companion to send with it?**

*Answer:* "Too Many Requests" — the client has exceeded the configured rate limit. A `Retry-After` header telling the client when it's safe to retry is standard practice alongside it.

---

**29. [Scenario] Two Gateway instances sit behind a load balancer, each still running Week 10's original in-memory Token Bucket. A client sends 150 requests against a limit of 100/minute, split evenly. What happens, and why?**

*Answer:* Roughly 75 requests hit each instance. Each instance's independent bucket (capacity 100) has plenty of room for 75, so all 150 requests are likely allowed — 1.5x the intended limit — because neither instance has visibility into the other's count.

---

**30. Why would a distributed Sliding Window Log naturally reach for a Redis Sorted Set rather than a plain list?**

*Answer:* A Sorted Set keyed by timestamp supports efficiently removing everything below a score and counting what remains, both without a full scan — exactly the two operations Sliding Window Log needs on every check. Full depth on Sorted Sets: Week 19, Day 132.

---

## Day 122 — Notification System, and HLD Mock #1

---

**31. Why does each notification channel get its own queue instead of one shared queue across Push, Email, and SMS?**

*Answer:* To avoid head-of-line blocking — if one channel's provider degrades, a shared queue would stall every other channel's messages behind it. Separate queues give each channel its own independent failure domain.

---

**32. What does the Template Service architecturally solve, beyond "keeping code DRY"?**

*Answer:* It centralizes notification copy in one place, so a wording change ships without a code deploy, and every call site producing the same notification type is guaranteed consistent.

---

**33. Walk through the deduplication check, and name its precise limitation.**

*Answer:* Check whether a Redis key for this (user, type, content) combination exists; if not, set it with a TTL and send; if it exists, skip. The check and the set are two separate round trips, so a small race window exists.

---

**34. Why is that race window acceptable for notifications but would not be acceptable for a payment request?**

*Answer:* An occasional duplicate notification costs mild user annoyance. The identical race in a payment context could double-charge a customer, a genuinely unacceptable failure, which is why payments need a fully atomic check-and-set instead (Week 19, Day 130).

---

**35. What is a Dead Letter Queue, and why does a failed notification send go there rather than being retried forever or dropped silently?**

*Answer:* A DLQ is a separate queue a message routes to after exhausting a bounded number of retries, so it can be inspected and reprocessed manually rather than blocking the main queue or vanishing. First established Week 14, Day 93.

---

**36. Why is `(user_id, notification_type, channel)` the right composite primary key for the preferences table, rather than a single surrogate key?**

*Answer:* "Enabled" is a property of that exact combination, not of any one column alone — the composite key structurally prevents two contradictory rows for the same combination from existing.

---

**37. Why model `status` as a small set of named states rather than a single boolean `sent` flag?**

*Answer:* A boolean can only distinguish sent from not-sent — it can't represent "currently retrying" versus "exhausted retries and dead-lettered," operationally different states that need to be queryable separately.

---

**38. A user has no row at all in `notification_preferences` for a given (type, channel) pair. What should the system do?**

*Answer:* The application layer needs a defined default — typically opt-in for critical/transactional types and opt-out by default for promotional ones — since an absent row is not the same as an explicit `enabled = false`.

---

**39. In the Confluence reframe, what specifically makes "10,000 watchers, edited every minute" a harder version of the notification problem?**

*Answer:* It's a fan-out problem — one edit event now has to become up to 10,000 individual notification sends, so cost scales with the number of interested readers, not the write itself. Full treatment: Week 19, Day 129.

---

**40. What specific thing is HLD Mock #1 designed to test, distinct from whether the design itself is correct?**

*Answer:* Whether the 5-step framework stays explicitly narrated, in order, under real time pressure — a candidate can have a correct design in mind and still lose points by never actually stating which step they're on.

---

**41. Name one common failure pattern HLD Mock #1 specifically watches for.**

*Answer:* Requirements bleeding directly into High-Level Design with Estimation skipped or reduced to an unexplained number.

---

**42. Why might Push, Email, and SMS need genuinely different retry/backoff behavior, not just separate queues?**

*Answer:* Each provider has different failure characteristics — a push provider's transient failure might resolve in seconds, while an email bounce is often permanent and retrying it is pointless.

---

**43. Why does Detailed Design in a mock need to go deeper than restating the High-Level Design in different words?**

*Answer:* Detailed Design's purpose is proving depth on the one piece that matters most — restating the box-and-arrow diagram verbally isn't a level deeper; a real answer picks one specific mechanism and explains it at implementation depth.

---

## Day 123 — Distributed Cache

---

**44. What's the core difference between Memcached and Redis, and why is Redis usually the more versatile default today?**

*Answer:* Memcached is a pure, multi-threaded key-value cache with no persistence and no richer data types. Redis adds richer data structures (sorted sets, lists, geospatial indexes), optional persistence, and pub/sub.

---

**45. Precisely, why does Thundering Herd happen?**

*Answer:* When a popular cache key expires or the cache is cold, many concurrent requests can independently see a cache miss within the same brief window and all query the database for the identical data at once.

---

**46. Why doesn't a shorter TTL fix Thundering Herd?**

*Answer:* TTL controls when an entry expires, not how many concurrent requests discover that expiration simultaneously — a popular key's expiration moment is exactly when the herd condition is most likely.

---

**47. Describe the stale-while-revalidate mitigation and why it doesn't need a distributed lock.**

*Answer:* Store a logical expiry earlier than the actual Redis TTL. Once logically stale but still physically present, a request gets the stale value back immediately while triggering a background refresh — no lock needed since no request races to hit the database.

---

**48. What's the trade-off between a repopulation mutex and stale-while-revalidate?**

*Answer:* A mutex guarantees exactly one database read per expiration, at the cost of building and safely operating a distributed lock. Stale-while-revalidate avoids the lock but accepts serving briefly stale data.

---

**49. How does Consistent Hashing (Week 9, Day 60) apply to a distributed cache specifically?**

*Answer:* It determines which cache node owns which keys via the ring and `ceilingKey()`, so when a node is added or removed, only the keys in the adjacent ring segment get remapped — most of the cache stays warm.

---

**50. Describe Cache-Aside's read path and write path separately.**

*Answer:* Read: check cache first; on a hit return it; on a miss read from the database, populate the cache, then return. Write: update the database, then evict the corresponding cache entry.

---

**51. Why evict on write instead of updating the cache with the new value directly?**

*Answer:* Updating the cache on every write means keeping two copies of the truth in sync forever, and any missed step leaves the cache silently wrong — a worse failure than a cache miss. Evicting lets the next read repopulate correctly from the source of truth.

---

**52. What mechanism does `@Cacheable` use to intercept a method call, and what prior material is this the same mechanism as?**

*Answer:* A Spring proxy wraps the bean at startup and intercepts external calls — the identical proxy-based AOP mechanism as Spring AOP (Week 9, Day 62) and Resilience4j's Circuit Breaker (Week 10, Day 64).

---

**53. Why does calling a `@Cacheable` method from inside the same class silently skip caching?**

*Answer:* Self-invocation calls the real object directly, not through the proxy that implements the caching behavior — the same self-invocation limitation already seen with Spring AOP and Resilience4j.

---

**54. How would you actually verify `@Cacheable` is working, rather than just assuming it from the code?**

*Answer:* Add a spy or logging on the repository call it wraps, call the cached method twice with the same argument, and assert the underlying repository was only hit once.

---

**55. Name two of Redis's richer data structures beyond plain key-value strings, and one problem each is naturally suited to.**

*Answer:* Sorted sets — ranked data like a leaderboard (Week 19, Day 132). Geospatial indexes — "find things near this location" queries (Week 19, Day 128, Uber's driver tracking).

---

**56. Why might a pure cache deliberately run with Redis persistence turned off?**

*Answer:* A cache miss just falls through to the source of truth — a cheap, self-healing failure mode by design — so there's little value paying persistence's overhead to protect trivially reconstructable data.

---

**57. What's the complexity of a Cache-Aside hit, a miss, and an eviction?**

*Answer:* A hit is O(1) — a Redis GET. A miss is the underlying query's cost plus one Redis SET. An eviction is O(1) — a Redis DEL.

---

## Day 124 — Distributed ID Generation

---

**58. Why does sharding break simple database auto-increment IDs?**

*Answer:* Each shard is a separate database with its own independent sequence, so two different shards can each hand out the same ID value to different rows, with no visibility into each other.

---

**59. What is a UUID, and why does UUIDv4 need no coordination between generators?**

*Answer:* A 128-bit value, with UUIDv4 deriving 122 of those bits from randomness — any two independently-generated UUIDv4s collide at odds low enough to treat as impossible.

---

**60. Precisely, why does UUID's lack of sortability hurt database performance?**

*Answer:* Random UUID values inserted as a primary key scatter across every part of a B-Tree index rather than appending at the end, causing more page splits and worse cache locality than a monotonically increasing key.

---

**61. State Twitter Snowflake's 64-bit layout in order, with each field's size.**

*Answer:* 1 unused bit (keeps the value non-negative), 41 bits of timestamp (ms since a custom epoch), 10 bits of machine ID, 12 bits of per-millisecond sequence.

---

**62. Why 41 bits for the timestamp specifically, and why a custom epoch rather than Unix time?**

*Answer:* 2^41 ms ≈ 69.7 years of range. A custom epoch (the platform's own start date) avoids wasting decades of that range on dates before the system existed, the way starting from 1970 would.

---

**63. [Trace] Pack a Snowflake ID for a 1ms timestamp offset, machine ID 3, sequence 0, and show it decodes back correctly.**

*Answer:* (1 << 22) | (3 << 12) | 0 = 4,194,304 + 12,288 = 4,206,592. Decoding: id & 0xFFF = 0 (sequence), (id >> 12) & 0x3FF = 3 (machine ID), id >> 22 = 1 (timestamp offset) — all correct.

---

**64. How many IDs can a single machine generate in one millisecond, and where does that number come from?**

*Answer:* 4,096 — directly from the 12-bit sequence field, since 2^12 = 4,096 distinct values before it must wrap.

---

**65. What happens when the sequence counter is exhausted within the same millisecond?**

*Answer:* The generator spin-waits until the system clock genuinely advances to the next millisecond, rather than reusing a sequence value and risking a duplicate ID.

---

**66. Why does the generator throw an exception if the system clock moves backward, instead of just proceeding?**

*Answer:* A backward clock jump could reproduce a timestamp already used, silently creating a duplicate ID. Throwing is the safer failure mode — a visible, retriable error rather than silent data corruption discovered later.

---

**67. Why is `nextId()` marked `synchronized`, and what established pattern is this the same shape as?**

*Answer:* Two threads on the same machine could otherwise both read the same lastTimestamp/sequence and hand out the identical ID — the same check-then-act race already seen in Parking Lot, the ATM, BookMyShow, and Day 121's Gateway rate limiter.

---

**68. Compare Snowflake and database ID range allocation on their core trade-off.**

*Answer:* Snowflake needs no network call per ID and gives rough global time-ordering, at the cost of depending on a well-behaved system clock. Range allocation needs no clock at all, at the cost of a periodic network round trip to the coordinator.

---

**69. How does database ID range allocation relate conceptually to Day 120's Key Generation Service?**

*Answer:* Both pre-claim a batch of something so an expensive coordination step happens once per batch rather than on every individual request — the same "don't pay the coordination cost on the hot path" instinct applied to two different payloads.

---

**70. Why is Snowflake generally preferred over having every instance hit one shared, centrally-coordinated counter directly?**

*Answer:* A shared counter needs a network round trip and contention on a single resource for every ID request. Snowflake generates IDs purely locally, with zero network calls in the common case.

---

**71. Why does the storage cost difference between a 128-bit UUID and a 64-bit Snowflake ID matter at real scale?**

*Answer:* 16 bytes versus 8 bytes per ID doubles the storage cost not just for the ID column but for every foreign key referencing it — at real volumes, that compounds into significant infrastructure cost.

---

## Day 125 — Distributed Consensus, and HLD Mock #2

---

**72. Define quorum precisely.**

*Answer:* In a cluster of N nodes, a quorum is any subset of more than N/2 nodes — a strict majority.

---

**73. [Prove] In a 5-node cluster with quorum size 3, show that two different quorums must overlap.**

*Answer:* Two groups of 3 from only 5 total cannot be disjoint — 3+3=6 exceeds 5, so by the pigeonhole principle they share at least 3+3-5=1 node. That shared node can't honestly have voted for two contradictory outcomes at once.

---

**74. What is split-brain, and how does quorum-based election prevent it?**

*Answer:* Two nodes each independently believing they're the leader and both accepting writes, with no way to reconcile the divergence. Quorum prevents it because only a majority-holding group can ever elect a leader.

---

**75. Connect today's quorum requirement directly to the CAP theorem (Week 9, Day 59).**

*Answer:* A minority partition becoming unavailable rather than electing its own leader is CAP's trade-off made concrete — a consensus system explicitly chooses consistency over availability during a partition.

---

**76. How does today's quorum relate to Replication's R+W>N model (Week 9, Day 61)?**

*Answer:* Same underlying mathematical tool — majority overlap — applied to a different question. R+W>N guarantees a read overlaps with the latest write; leader-election quorum guarantees two conflicting elections can't both succeed.

---

**77. Why is Paxos historically considered difficult to implement correctly?**

*Answer:* Its base protocol is built around an abstract two-phase voting process with no built-in notion of a stable leader, making its real-world behavior notoriously subtle to reason about correctly.

---

**78. Name Raft's three decomposed sub-problems.**

*Answer:* Leader election, log replication, and safety.

---

**79. What does a Raft "term" prevent, specifically?**

*Answer:* A stale former leader from being mistaken for the current one — any message carrying an old term number is ignored by nodes already on a newer term.

---

**80. How does Raft's log replication close a specific gap left open by Replication Models (Day 61)?**

*Answer:* Day 61 described how writes flow from a leader to followers, but never specified a safe procedure for handing the leader role to a new node. Raft's log replication is exactly that missing procedure, made rigorous.

---

**81. What safety rule stops a newly-elected Raft leader from silently losing committed data?**

*Answer:* A node is disqualified from winning an election unless its own log is at least as up-to-date as a majority of the cluster.

---

**82. What does etcd use internally, and what have you been relying on it for since Week 15 without naming it?**

*Answer:* etcd uses Raft internally. Kubernetes' Control Plane (Day 99) stores all of its state in etcd, so every reconcile-loop decision has depended on etcd's Raft-based consensus the entire time.

---

**83. Why is an odd-sized cluster (3, 5, 7 nodes) the conventional choice for something like etcd?**

*Answer:* An even-sized cluster buys no extra fault tolerance over the next-smaller odd size — a 4-node cluster's majority is 3, tolerating only 1 failure, identical to a 3-node cluster, for the cost of an extra machine.

---

**84. Give the core argument for why single-leader replication needs consensus for safe leader failure handling.**

*Answer:* Without it, an uncoordinated election risks split-brain, or picking a node that wasn't fully caught up, silently losing committed writes. Quorum-based voting makes split-brain structurally impossible, and a log-completeness safety rule prevents the second.

---

**85. In HLD Mock #2's "why not X" format, what makes a defense structurally strong rather than weak?**

*Answer:* Naming the alternative's genuine advantage first, then giving a specific, concrete reason the original choice still wins for this system's actual requirements — rather than caving immediately or dismissing the alternative without engaging with it.

---

**86. Why did Raft, not Paxos, end up implemented in widely-used real infrastructure like etcd?**

*Answer:* Raft's explicit leader and clean decomposition into three separately-understandable pieces made it dramatically easier to implement correctly — and a subtly-wrong consensus implementation is more dangerous than one that's simply less famous.

---

## Day 126 — BookMyShow at Scale, and Week 18 Consolidation

---

**87. Why did BookMyShow's LLD version never need sharding, and why can't the HLD version avoid it?**

*Answer:* The LLD version modeled one theater in one process with one database. At national scale, thousands of theaters and tens of thousands of daily shows exceed what one instance can hold safely, requiring the database itself to be split.

---

**88. Why shard by city rather than by a hash of the booking ID?**

*Answer:* Almost every real query is already scoped to one city — a user browsing showtimes only looks at their own city's theaters — so city-based sharding routes the overwhelming majority of queries to exactly one shard.

---

**89. What new problem does the seat-hold mechanism solve that Week 17's original locking didn't?**

*Answer:* The original locking prevents two people from booking the same seat at the same instant. Seat-hold answers how a seat a user has selected but not yet paid for stays reserved temporarily, and is automatically released if abandoned.

---

**90. Why does the seat-hold check-and-set need to be wrapped in Lua, and what established pattern is this reusing?**

*Answer:* A plain GET-then-SET is two round trips with a race window. Wrapping both in a Lua script makes it atomic via Redis's single-threaded execution, the identical pattern Day 121 used for the rate limiter.

---

**91. Why does an abandoned seat-hold need no separate cleanup code?**

*Answer:* The hold is set with a Redis TTL — Redis itself deletes the key once the window elapses, so the mechanism that grants the hold is the one that releases it.

---

**92. Why is a blockbuster's midnight on-sale close to a textbook Thundering Herd scenario?**

*Answer:* Thousands of users refresh within the same few seconds the page goes live, and since it's a brand-new show, nothing has been cached yet — every request misses the cache simultaneously.

---

**93. Why does a known, announced on-sale time change the best mitigation compared to Day 123's general framing?**

*Answer:* Day 123's mitigations are reactive, for unpredictable spikes. A publicly announced on-sale time is knowable in advance, so proactively warming the cache before that instant is strictly better — the herd never gets the chance to start.

---

**94. Walk through the peak booking-rate estimation.**

*Answer:* 1,000 shows × 200 seats = 200,000 seats/day ceiling. Assuming 30% concentrate in the 2-hour peak: 60,000 bookings ÷ 7,200 seconds ≈ 8.3 writes/sec. At an assumed 15 reads per booking: ≈125 reads/sec sustained.

---

**95. Where does Distributed Consensus (Day 125) show up in this design, even though it isn't drawn as its own labeled component?**

*Answer:* Underneath each city shard — safe failover if a shard's leader dies depends on quorum-based, Raft-style consensus, the same mechanism underneath etcd and Kubernetes' Control Plane.

---

**96. In the full HLD diagram, what specifically did not change from BookMyShow's LLD version?**

*Answer:* The actual seat-commit logic within a single city's shard — SELECT FOR UPDATE and optimistic version-CAS from Week 17 — is unchanged. Scaling out added everything around that mechanism, not a rewrite of it.

---

**97. [Self-Check] How does Combination Sum II (LC 40) differ from Combination Sum (LC 39), specifically?**

*Answer:* Each number can only be used once (the recursive call advances to i+1, not i), and the input can contain duplicate values, requiring a same-level duplicate-skip so the output has no duplicate combinations.

---

**98. [Self-Check] Why must the array be sorted first, and what two separate purposes does that one sort serve?**

*Answer:* Sorting enables early termination (break once candidates[i] > remaining) and makes the duplicate-skip check meaningful, since equal values are guaranteed adjacent only once sorted.

---

**99. [Self-Check] Why is the duplicate-skip condition `i > start` and not `i > 0`?**

*Answer:* `i > 0` would also incorrectly skip a value that's the first choice at a brand-new recursion level. `i > start` only skips a value repeating one already tried at the same level.

---

**100. [Self-Check] What's the time and space complexity of the Combination Sum II backtracking solution?**

*Answer:* O(2ⁿ) time worst case, substantially reduced in practice by sort-enabled pruning and duplicate-skipping; O(n) space for recursion depth plus the in-progress combination.

---

*100 questions. Cross-reference: [00_Curriculum_Map.md](./00_Curriculum_Map.md) for the full cumulative problem table and concept index across all 18 weeks.*
