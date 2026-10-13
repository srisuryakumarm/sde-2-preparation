# Day 127 — Missing Primer: Redis Sorted Sets and Geo Commands

**Do this first on Day 127 (≈ 30 min), then continue with Day 127's regular plan.**
**Needed by:** Day 128 (a `LocationService` using Redis Geo), Day 131 (scheduler polling a sorted set), Day 132 (leaderboards). Day 132's own book admits sorted sets "have been used three times now without ever explaining why they behave the way they do" — this moves the explanation to where the first use is.
**Already in place:** the Redis basics micro-primer (Day 44), `@Cacheable` and the Redis cache (Day 123), the Lua rate limiter (Day 121). The skip-list internals micro-primer (Day 132) explains *why* they are fast.

## 1. Sorted set = unique members, each with a numeric score, kept in score order
Add, update and range queries are O(log n). Run these in `redis-cli` (`docker exec -it <redis> redis-cli`):
```
ZADD leaderboard 1500 alice 1200 bob 1800 carol      # score first, then member
ZINCRBY leaderboard 50 bob                           # bob → 1250
ZRANGE leaderboard 0 2 REV WITHSCORES                # top 3, highest first (Redis 6.2+; older: ZREVRANGE)
ZREVRANK leaderboard alice                           # 0-based rank from the top
ZSCORE leaderboard carol
ZREM leaderboard bob
```
Patterns: **leaderboard** (score = points); **delayed job queue** (score = the time to run — `ZADD jobs <epoch> job:1`, then `ZRANGE jobs -inf <now> BYSCORE LIMIT 0 10` fetches what is due; Day 131); **sliding-window counters**.

## 2. Geo commands are sorted sets in disguise
Redis encodes each (longitude, latitude) as a geohash and stores it as a sorted-set score.
```
GEOADD drivers 77.5946 12.9716 driver1 77.6101 12.9352 driver2      # LONGITUDE first, then latitude
GEOSEARCH drivers FROMLONLAT 77.5946 12.9716 BYRADIUS 5 km ASC WITHDIST COUNT 5
GEODIST drivers driver1 driver2 km
```
Traps: **longitude comes first**; valid latitudes are about ±85.05°; `GEORADIUS` is deprecated — use `GEOSEARCH`. Because a geo set *is* a sorted set, `ZREM` removes a member and `ZRANGE` lists them.

## 3. From Spring
Through `RedisTemplate`: `opsForZSet().add(key, member, score)`, `opsForGeo().add(key, new Point(lon, lat), member)` (`Point(x = longitude, y = latitude)`), and `opsForGeo().search(...)`. Keep the CLI as your ground truth: if the Java result surprises you, run the same query in `redis-cli`.

## 4. Exercise (≈ 20 min, `redis-cli`)
1. A five-player leaderboard: add, bump two scores, list the top 3, find one player's rank.
2. A delayed-job set: add three jobs with timestamps (past, now, future); fetch only the due ones.
3. Four drivers around a point; find those within 3 km; compute two distances.

## Checklist
- [ ] Can explain a sorted set in two sentences and name three uses.
- [ ] Can say why longitude-first is a classic bug.
- [ ] Can say why geo data can be removed with `ZREM`.
- [ ] Continue with Day 127's regular plan.
