# Day 122 — Missing Primer: Redis in Operation — TTL, Eviction, Persistence, Replication, Sentinel, and Cluster

**Do this first on Day 122 (≈ 45 min), then continue with Day 122's regular plan.**
**Needed by:** Day 123 (HLD #4: Distributed Cache), Day 121's rate limiter in production, Days 128 and 132 (Geo, leaderboards), and any "what happens when Redis is full / restarts / fails?" question.
**Why it is missing:** the Day 44 Redis micro-primer has no TTL, eviction, persistence or high-availability content; those appear only in passing in later books.
**Already in place:** Redis basics (Day 44), sorted sets and Geo (Day 127's file — do that primer too before Day 128), cache-aside and cache invalidation ideas (Day 123's theory), replication concepts (Day 61).

---

## 1. TTL and expiry
```
SET session:42 abc EX 60        # expire in 60 seconds
TTL session:42                  # seconds left (-1 = no expiry, -2 = gone)
EXPIRE key 300   |   PERSIST key
```
Redis expires keys **lazily** (when touched) and with a periodic **active sampling** of keys that have a TTL — so memory is freed *approximately* on time, not to the millisecond. Add random **jitter** to TTLs so a million keys don't expire in the same second (cache stampede, Day 123).

## 2. Eviction — what happens when memory is full
Set `maxmemory`, then `maxmemory-policy` decides:
| Policy | Evicts |
|---|---|
| `noeviction` (**default**) | nothing — writes fail with an error |
| `allkeys-lru` | least-recently-used among **all** keys (the usual cache choice) |
| `allkeys-lfu` | least-frequently-used |
| `volatile-lru` / `volatile-lfu` / `volatile-ttl` | only keys that have a TTL |
| `allkeys-random`, `volatile-random` | random |
LRU/LFU are **approximated by sampling** a few keys (`maxmemory-samples`) — cheaper than a true LRU list.

## 3. Persistence — surviving a restart
- **RDB:** periodic point-in-time **snapshots** (a forked child writes a file). Compact, fast restart; you can lose everything since the last snapshot.
- **AOF:** an **append-only log** of every write; `appendfsync everysec` (default; lose ≤ ~1 s), `always` (safest, slowest). Rewritten in the background to stay small.
- Often **both**; or **none** when Redis is a pure cache (rebuild from the database).

## 4. Replication, Sentinel, Cluster
- **Replication:** replicas copy the primary **asynchronously** — reads can be stale, and a failover can lose the last writes.
- **Sentinel:** monitors a primary and its replicas, runs **automatic failover**, and tells clients where the new primary is. High availability, **no sharding**.
- **Redis Cluster:** shards the key space into **16,384 hash slots** across nodes (each with replicas). Multi-key operations must hit one slot — force that with **hash tags**: `{user:42}:cart` and `{user:42}:profile` share a slot.

## 5. Operations that bite
Commands run on **one thread** — a slow command (`KEYS *`, huge `SMEMBERS`, a big Lua script) blocks everyone: use `SCAN` instead of `KEYS`. **Big keys** (a 100 MB list) cause latency spikes and uneven shards. Redis is memory-bound: size it for peak, and watch `used_memory` and `evicted_keys`.

## 6. Exercise (≈ 20 min, `redis-cli`)
1. `SET a 1 EX 10`, poll `TTL a` until it disappears; set TTLs with and without jitter on 5 keys.
2. `CONFIG SET maxmemory 2mb` and `CONFIG SET maxmemory-policy allkeys-lru`; load keys in a loop (`redis-cli --pipe`, or a shell loop of `SET key$i value`) until `INFO stats` shows `evicted_keys > 0`. Switch to `noeviction` and read the write error.
3. Check `CONFIG GET save` and `CONFIG GET appendonly`; say which durability you have and what a crash would cost.
4. Answer: "Redis is the cache for product pages — which policy, which persistence, and what if the primary dies?"

## Checklist
- [ ] Can set a TTL, explain lazy + active expiry, and why TTLs need jitter.
- [ ] Can name three eviction policies and the default.
- [ ] Can compare RDB and AOF, and Sentinel and Cluster.
- [ ] Can explain hash tags and why `KEYS *` is dangerous. Continue with Day 122's regular plan.
