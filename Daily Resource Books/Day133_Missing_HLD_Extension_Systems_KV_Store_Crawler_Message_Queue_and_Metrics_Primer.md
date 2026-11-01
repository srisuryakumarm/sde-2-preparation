# Day 133 — Missing Primer (STRETCH): HLD Extension Systems — Key-Value Store, Web Crawler, Message Queue, Metrics / Top-K, and Migration Patterns

**Do this on Day 133 (Sunday, ≈ 90 min) or spread it across the Sundays of Days 133 / 140 / 147 — after Day 133's Mock #5 and before the capstone hardening block. STRETCH: skip it if you are behind** (see the triage rules in `Day1_to_148_Realistic_Timeline_and_Buffer_Strategy.md` — the 16 designed systems are the MUST-KEEP set). Do it if a target loop (Databricks, Google, Uber) is close, or in the final two weeks as extra mock material.
**Needed by:** Day 133's Mock #5 ("candidate's choice" — these four are fair game), the Day 146 system-design mock, and the loops at Databricks / Google / Uber, where "design a key-value store / web crawler / message queue / metrics system" are standard prompts.
**Why it is missing:** the plan designs 16 systems and every HLD mock re-uses one of them. **Four classic prompts are never designed — a distributed key-value store (Dynamo-style), a web crawler, a distributed message queue, and a metrics/monitoring and ad-click/top-K aggregation system.** **Merkle trees, hinted handoff, read repair and linearizability appear in no book.** Migration strategy (strangler fig) — the standard "senior" follow-up — appears nowhere either.
**Already in place:** the 5-step HLD framework and estimation (Days 119–120), consistent hashing (Day 60), replication and CAP (Days 58–61), quorums, time ordering, gossip, vector clocks and CRDTs (Day 124's file), Raft (Day 125), Bloom filters, Count-Min sketch and LSM trees (Day 123's file), Redis sorted sets (Day 127's file), Kafka and its internals (Days 48–50, 66's file), rate limiting, caching, search, object storage (Days 121–133), stream vs batch and OLTP vs OLAP (Day 80's file), the outbox (Day 69's file).

---

## 0. How to use this file
For each system below, the skeleton is the framework you already know: **requirements → estimation → API → data and components → deep dive and trade-offs**. The point is to **reuse the building blocks you have already learned**, so each section lists them. After reading, pick one and do a cold 25-minute oral mock.

## 1. Distributed key-value store (Dynamo / Cassandra style)
**Requirements:** `put(key, value)` / `get(key)`; always writable (high availability); scale to hundreds of nodes; tunable consistency; survive node and datacenter failures.
**Design (each box is a block you know):**
- **Partitioning:** consistent hashing with **virtual nodes** (Day 60) → keys spread evenly; a node joining or leaving moves only ~1/N of the keys.
- **Replication:** each key is stored on the **N next distinct nodes** clockwise on the ring (the *preference list*). Any node can be the **coordinator** for a request.
- **Quorums:** **R + W > N** guarantees the read set overlaps the latest write set. Typical N=3: `W=2, R=2` balanced; `W=1, R=3` fast writes; `W=3, R=1` fast reads; `W=1, R=1` eventual and fastest (Day 124's file).
- **Sloppy quorum + hinted handoff:** if a replica is down, write to the next healthy node on the ring with a **hint** ("this belongs to node X"); when X returns, the stand-in hands the data back. Keeps writes available during failures.
- **Versioning and conflicts:** concurrent writes to different replicas → **vector clocks** (Day 124) let the system detect conflicts; resolve by last-write-wins (simple, can lose a write when clocks skew) or by returning siblings for the application / a CRDT to merge.
- **Repair:** **read repair** (a read that finds a stale replica writes the newest value back to it) and **anti-entropy** (below).
- **Failure detection and membership:** **gossip** (Day 124).
- **Per-node storage:** an **LSM tree** — write-ahead log + in-memory memtable + immutable sorted SSTables + background compaction + Bloom filters to skip files (Day 123's file).
**Merkle trees (anti-entropy):** to find which keys differ between two replicas **without comparing every key**, each replica builds a **hash tree over its key ranges**: a leaf = hash of one range's data, a parent = hash of its two children. Compare the **root hashes**; if equal, the replicas match (one comparison). If different, descend only into the children whose hashes differ — the work is proportional to *the number of differences*, not the data size. Used by Cassandra's repair and Dynamo; the same idea underlies Git's object trees and file-sync diffing (Day 133's Drive design).
**Consistency ladder (what "consistent" can mean):**
| Model | Promise | Cost / example |
|---|---|---|
| **Linearizable** | behaves as if there is **one copy**; every operation takes effect atomically at one instant, respecting real-time order | needs consensus/quorum on every operation → latency, unavailable in a minority partition (**CP**): etcd, ZooKeeper, Spanner |
| **Sequential** | everyone sees the same order, not necessarily real-time | rarely named in interviews |
| **Causal** | if A caused B, nobody sees B before A; concurrent operations may appear in different orders | available under partitions; vector clocks |
| **Eventual** | if writes stop, replicas converge | **AP**: Dynamo, Cassandra at low R/W |
Session guarantees (read-your-writes, monotonic reads — Day 58's file) are the practical layer on top of eventual. **Say:** "the *C* in CAP is **linearizability**."
**Trade-offs to volunteer:** AP + eventual = fast and always writable but the application must tolerate stale reads and conflicts; CP (Raft-based, Day 125) = simpler semantics, lower availability.

## 2. Web crawler
**Requirements:** crawl ~1 billion pages/month; be **polite** (never hammer a host); robust (malformed HTML, slow servers, traps); extensible (new content types); de-duplicate.
**Estimation:** 1 B pages / (30 × 86,400 s) ≈ **400 pages/s**; at ~100 KB per page ≈ **40 MB/s** of download and **~100 TB/month** of raw storage.
**Pipeline:** seed URLs → **URL frontier** → **fetcher** (checks the cached `robots.txt`, uses a **DNS cache** — resolution is a classic bottleneck, sets timeouts) → **content store** (object storage, Day 133; metadata in a database) → **parser / link extractor** → **dedup** → new URLs back into the frontier.
**The frontier (the deep dive):** the Mercator design has **front queues by priority** (page importance, freshness) feeding **back queues, one per host**, each with a "not before" time so a host is hit at most once every few seconds — *politeness*. A worker handles whole hosts, so **partition URLs by `hash(host)`** (consistent hashing) and politeness stays local.
**De-duplication:** *URL-seen* test = a **Bloom filter** in memory in front of a persistent set (a false positive just skips a new URL — acceptable); *content-seen* = exact hash of the page, or **simhash** for near-duplicates. **Normalise URLs** first (lower-case host, drop `#fragment`, resolve relative links, sort query parameters).
**Traps and edge cases:** infinite URL spaces (calendars, session ids) → depth and URL-length limits and per-host caps; **recrawl scheduling** by observed change rate; JavaScript-rendered pages need a headless browser (10–100× the cost); checkpoint the frontier (a log such as Kafka) so a crash resumes.

## 3. Distributed message queue (Kafka / SQS)
**Requirements:** produce and consume at high throughput, durability, ordering where needed, at-least-once delivery, scale-out.
**Design:** a **topic** is split into **partitions**; each partition is an **append-only log** of segment files (sequential I/O, page cache — Day 66's file); partitions are **replicated** with a leader and ISR; producers **batch**; consumers **pull** within **consumer groups** and track **offsets**; a **controller/metadata quorum** (Raft, Day 125) elects leaders; retention or compaction bounds storage.
**Two families — know the contrast:**
| | **Log-based (Kafka)** | **Queue-based (SQS, RabbitMQ)** |
|---|---|---|
| Message after consumption | stays (until retention) — **replayable** | removed on **ack** |
| Ordering | per partition | none (standard SQS) / per message group (FIFO) |
| Consumers | pull + offsets; parallelism ≤ partitions | competing consumers; **visibility timeout** hides a message while one worker processes it; unacked → reappears |
| Poison messages | handled by the app (skip/DLQ topic) | built-in **DLQ** after N receives |
**Deep dives to offer:** delivery semantics (Day 50's primer), hot partitions (choose keys well), back-pressure (consumers pull), rebalancing and duplicates, multi-region replication (Day 124).

## 4. Metrics / monitoring system, and ad-click / top-K aggregation
**Metrics system.** Agents/exporters on every host → **collectors** → a **queue** (Kafka, to absorb bursts) → a **time-series database** (TSDB) → query service → dashboards; a separate **rule evaluator** → **alert manager** → notifications (Day 122).
- **Pull vs push:** pull (Prometheus) = the server scrapes `/metrics` — easy health detection and service discovery, awkward for short-lived jobs and firewalls; push (StatsD / OpenTelemetry) = hosts send — works for batch jobs, but you can be flooded.
- **TSDB specifics:** write-heavy append of `(series, timestamp, value)`; excellent compression (delta-of-delta timestamps, XOR-encoded floats); **downsampling / roll-ups** (1 s data for 2 days, 1 min for 30 days, 1 h for a year); retention tiers.
- **The killer:** **label cardinality** — a label with unbounded values (`user_id`, request id) multiplies the number of series and kills the TSDB. Alerts: SLO **burn-rate** rules (Day 76's file); the alert manager **groups, de-duplicates and silences**.
**Ad-click aggregation / Top-K ("top 10 ads per minute").**
- Click events → **Kafka partitioned by `ad_id`** → **stream processor** with **tumbling windows** (Day 80's file) → aggregates to an **OLAP store** (Day 80's file); a nightly **batch** job recomputes exact numbers and reconciles (**lambda**: batch + stream; **kappa**: stream only, replay from Kafka).
- **Correctness:** de-duplicate by `click_id`; write **idempotently** keyed by `(ad_id, window)`; **late events** → a *watermark* with allowed lateness; fraud filtering.
- **Hot key** (a viral ad): add a random salt to the key, aggregate partially in stage 1, merge in stage 2.
- **Top-K:** exact = Redis sorted set per window (Day 127's file); approximate at massive scale = **Count-Min sketch + a min-heap of size K** per partition, then merge (Day 123's file).
- **Batch mental model (MapReduce → Spark):** *map* (transform each record) → **shuffle** (group by key across the network — **the expensive step**) → *reduce* (aggregate). **Skew** makes one reducer the bottleneck. MapReduce writes intermediate data to disk between stages; **Spark** builds a lazy **DAG** of transformations and keeps data in memory, with *narrow* dependencies (no shuffle) vs *wide* ones (shuffle).

## 5. Migration patterns (the senior follow-up: "how would you move the monolith?")
- **Strangler fig:** put a **routing layer (your Gateway, Day 66)** in front; move one **capability** at a time to a new service and route its traffic there; the old code shrinks until it can be deleted. Never a big-bang rewrite.
- **Anti-corruption layer:** a translator between the old model and the new one so legacy concepts do not leak into the new service (Day 112's DDD vocabulary).
- **Parallel run / shadow traffic:** send real requests to both, **compare** responses, serve only the old one, until the new matches.
- **Data migration:** **expand → backfill → dual-read and verify → switch → contract** (Day 135's file teaches it in detail; here you only need the names); keep data in sync with the **outbox / change-data-capture** (Day 69's file), **not** with application-level dual writes (they diverge on failure).
- **Always:** a tested **rollback**, feature flags (Day 135's file), and a metric that defines "done".

## 6. Practice (≈ 25 min each)
1. Pick one of §1–§4 and give a **cold oral answer** (requirements → estimation → components → one deep dive → trade-offs). Record it (Voice Memos), re-listen for hand-waving.
2. **Drill table:** for each of the four systems write the **three biggest bottlenecks / failure modes** and the block from your earlier days that answers each.
3. Answer in 60 s each: *"What is a Merkle tree for?"*, *"R + W > N — why?"*, *"What is the C in CAP, precisely?"*, *"How does a crawler stay polite?"*, *"Log-based vs queue-based broker?"*, *"How do you migrate a monolith safely?"*

## Checklist
- [ ] Can design a Dynamo-style store: ring + vnodes, N/R/W, hinted handoff, read repair, Merkle anti-entropy, LSM per node.
- [ ] Can state the consistency ladder and say which systems sit where.
- [ ] Can design a crawler's frontier (priority + per-host politeness) and its dedup (Bloom filter + content hash).
- [ ] Can contrast log-based and queue-based brokers.
- [ ] Can design a metrics pipeline (pull vs push, TSDB, cardinality) and a stream-based top-K with late events and hot keys.
- [ ] Can describe a strangler-fig migration with shadow traffic and expand/contract. Continue with Day 133's regular plan (or skip this file if you are behind).
