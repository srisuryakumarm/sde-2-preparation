# Day 123 Resource Book — Distributed Cache

**Series:** SDE-2 Interview Prep Resource Books · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 122](./Day122_Resource_Book.md) · **Next ▶:** [Day 124](./Day124_Resource_Book.md)
**Companion to:** Day 123 of `Week_18_Revised.md`

---

## Recap: what today is

Redis has been running in `todo-api`'s Docker Compose stack since Week 6 (Day 44) as pure infrastructure, and Days 121–122 borrowed a handful of its commands narrowly — a counter here, an existence check there — for one job each, without ever stopping to look at what Redis actually *is*. Today closes that gap: the full picture of what a distributed cache is for, what Redis specifically offers beyond a plain key-value store, and the one failure mode (Thundering Herd) that catching-in-front-of-a-database can introduce if you're not deliberate about it. Today also reaches back further than Days 121–122 did — Consistent Hashing, taught in full back in Week 9, Day 60, gets reused here exactly as it was built, not re-derived.

## Learning Objectives

By the end of today, without notes:

1. Compare Memcached and Redis, and justify Redis as the more versatile default for most real systems.
2. Explain, precisely, why Thundering Herd happens and defend at least two distinct mitigations.
3. Explain the Cache-Aside pattern's read and write paths, and justify why it evicts on write rather than updating the cache directly.
4. Explain how Consistent Hashing (Week 9, Day 60) applies unmodified to a cache cluster's node-membership changes.

## Concept Dependency Map

```
Prerequisites, confirmed already covered:
  Consistent Hashing (full mechanism) .... Week 9, Day 60
  Redis, borrowed narrowly ............... Day 121 (INCR/EXPIRE/HMGET) · Day 122 (EXISTS/SET+TTL)
  Spring AOP / proxy-based interception .. Week 9, Day 62 · Resilience4j, Week 10, Day 64
  Mockito / testing doubles .............. Week 7, Day 46
        │
        ▼
NEW today:
  Redis, in full: data structures, persistence, pub/sub — the survey Days 121–122 deferred
        │
        ├─▶ Thundering Herd: the failure mode caching introduces
        │        ├─ Mitigation 1: mutex on repopulation (mechanism deferred to Week 19, Day 130 —
        │        │                 distributed locks get their own full treatment there)
        │        └─ Mitigation 2: stale-while-revalidate (built in full today, no lock needed)
        │
        └─▶ Cache-Aside pattern, via Spring's @Cacheable / @CacheEvict
        │
        ▼
Consistent Hashing (recap, Week 9 Day 60) applied to cache-node membership changes
```

---

## Part 1 — Redis, in Full

**Memcached vs. Redis:**

| | Memcached | Redis |
|---|---|---|
| Data model | Plain key-value strings only | Strings, hashes, lists, sets, sorted sets, geospatial indexes |
| Threading | Multi-threaded | Historically single-threaded for command execution (exactly what made Day 121's Lua atomicity guarantee true) |
| Persistence | None — pure in-memory cache | Optional — can snapshot to disk (RDB) or log every write (AOF), so a restart doesn't mean total data loss |
| Pub/sub messaging | No | Yes |
| Typical role today | A pure cache, nothing else | Cache **and** a lightweight data store / message broker, depending on how it's configured |

**Why Redis is usually the more versatile default, stated as a real trade-off rather than an assertion:** Memcached's simplicity is a genuine advantage when the job really is *only* "cache some key-value pairs, as fast as possible, no other requirements" — it has less machinery to reason about and, being multi-threaded, can use multiple cores for command execution in a way Redis's classic single-threaded core doesn't. Redis earns the "more versatile" label because the same running instance can serve as the cache (today), the rate limiter's shared counter (Day 121), a deduplication store (Day 122), and — with its richer data structures — the backing store for problems that aren't simple caching at all, which is exactly what shows up in the coming weeks: a sorted set is the natural fit for a leaderboard's ranked scores, and a geospatial index is the natural fit for "find things near this location" — both full topics of their own once this phase reaches HLD systems built specifically around them, in Week 19.

**Persistence options, briefly:** RDB (point-in-time snapshots to disk) and AOF (an append-only log of every write, replayable to reconstruct state) are Redis's two mechanisms for surviving a restart without losing everything — worth knowing they exist and that a *pure* cache often deliberately runs with persistence off entirely, since a cache miss just means "go read the source of truth," a cheap, self-healing failure mode a cache is specifically designed to tolerate.

**Pub/sub, briefly:** Redis supports publish/subscribe messaging directly — a publisher sends a message to a named channel, and every currently-subscribed client receives it immediately. Not used anywhere in today's design; worth knowing it exists as one more reason Redis often ends up doing more than "just caching" in a real system.

---

## Part 2 — Thundering Herd

**The problem, precisely:** a cache entry has a TTL. When it expires — or when the cache is cold-started and holds nothing yet — the *next* request for that key misses the cache and falls through to the database. That's fine for one request. The problem is scale: if the key is genuinely popular (a trending product, a viral post), dozens or hundreds of requests can arrive **within the same few milliseconds** the key was valid one moment and gone the next — every one of them independently sees a cache miss, and every one of them independently queries the database for the *identical* data, all at once. The cache — whose entire job is protecting the database from exactly this kind of load — momentarily does the opposite: it creates a synchronized spike of duplicate, redundant database load, concentrated at precisely the worst possible instant.

**Why TTL alone can't prevent this — worth being explicit about, since it's a natural question:** TTL controls *when* an entry expires, not *how many concurrent requests* discover that expiration at once. A popular key is, by definition, hit by many concurrent requests — so its expiration moment is exactly when the herd condition is most likely, not an edge case that only shows up on obscure keys.

### Mitigation 1 — a Repopulation Mutex (Conceptual Today)

The idea: when a cache miss occurs, one request acquires a short-lived lock specific to that key and becomes the *only* one that actually queries the database and repopulates the cache. Every other concurrent request for that same key either waits briefly and then re-checks the now-repopulated cache, or falls back to a slightly stale value if one's available, instead of every request independently hammering the database.

**Deliberately not built end-to-end today:** correctly implementing this lock — including handling what happens if the process holding it crashes before releasing it — is Week 19, Day 130's dedicated topic, where distributed locks get their own full treatment. Today's job is recognizing *when* this shape of solution is the right one, not yet building the lock mechanism itself; building a lighter version of that exact mechanism here would mean re-deriving it twice, once shallow and once properly, which the series' own "don't re-teach, cite it" discipline argues against.

### Mitigation 2 — Stale-While-Revalidate (Built in Full Today)

A different idea that needs no lock at all: store a **logical expiry** timestamp alongside the cached value — deliberately earlier than the actual Redis TTL.

```java
public class CachedValue<T> {
    private final T value;
    private final Instant logicalExpiry;   // "should be refreshed after this point"
    // getters omitted
}

public <T> T getWithStaleWhileRevalidate(String key, Supplier<T> loadFromDb,
                                          Duration freshFor, Duration staleGraceperiod) {
    CachedValue<T> cached = redisGet(key);

    if (cached == null) {
        // truly cold — no choice but a synchronous load, the rare unlucky case
        T fresh = loadFromDb.get();
        redisSet(key, new CachedValue<>(fresh, Instant.now().plus(freshFor)),
                  freshFor.plus(staleGraceperiod));   // Redis TTL is longer than the logical expiry
        return fresh;
    }

    if (Instant.now().isBefore(cached.logicalExpiry())) {
        return cached.value();   // genuinely fresh — the common case, fast path
    }

    // stale but present: return it immediately, don't make this request wait,
    // and kick off a background refresh for the NEXT reader
    triggerAsyncRefresh(key, loadFromDb, freshFor, staleGraceperiod);
    return cached.value();
}
```

**Why this works without a lock:** the Redis TTL is set *longer* than the logical expiry on purpose, so the value stays physically present in Redis for a grace period after it's logically considered stale. During that grace period, every request gets an immediate, correct-enough answer from the stale-but-present value — no request blocks on the database — while exactly one background refresh (not literally guaranteed exactly-one without additional coordination, but overwhelmingly likely to be a small handful rather than hundreds, since only requests that happen to trigger the async refresh path do so) repopulates the cache for the next reader. The genuine thundering-herd scenario — many *simultaneous synchronous* database reads — only remains possible in the narrow window after the Redis TTL has *also* expired (the key is entirely gone, not just logically stale), which the grace period is specifically sized to make rare.

**Trade-off between the two mitigations, stated directly:** the mutex approach guarantees exactly one database read per expiration event, at the cost of the added complexity (and failure modes) of a distributed lock. Stale-while-revalidate needs no lock and degrades gracefully, at the cost of occasionally serving data that's a few seconds logically stale — acceptable for a product listing or a social feed, considerably less acceptable for, say, an account balance.

> 💡 **Interview Insight:** naming that trade-off explicitly — "stale-while-revalidate accepts brief staleness to avoid needing a lock at all; a mutex guarantees freshness at the cost of lock complexity" — and then picking one *based on what the specific data tolerates* is a stronger answer than confidently naming just one mitigation as if it were the only correct choice.

---

## Part 3 — Consistent Hashing, Recapped and Applied (Week 9, Day 60)

**The mechanism, in one paragraph, exactly as originally built — not re-derived:** servers and keys are both hashed onto one logical ring. A key belongs to whichever server is the next one clockwise from it, found via `TreeMap.ceilingKey()` in `O(log S)` time (`S` = number of servers), with wraparound handled by `firstKey()` when a key hashes past the last server on the ring. The entire payoff, worth restating because it's exactly what today's cache cluster needs: when a server is added or removed, **only the keys in the ring segment adjacent to that change** get remapped — not the whole keyspace, the way naive `hash(key) % N` would remap almost everything the moment `N` changes.

**Applied here, directly, with nothing new to derive:** as cache nodes are added (scaling up for more capacity) or removed (a node failing), Consistent Hashing is exactly what decides which node now owns which keys, and it's exactly why that decision touches only a small, localized fraction of the cache's total keyspace — the majority of cached entries stay correctly assigned to the same node they were already on, meaning most of the cache's warmth survives a membership change instead of the entire cache going cold at once.

---

## Part 4 — The Cache-Aside Pattern

**Mechanism — the read path:** on a read, check the cache first. On a hit, return the cached value directly. On a miss, read from the database, write the result into the cache, then return it — so the *next* read for that same key is a hit.

**Mechanism — the write path:** on a write, update the database, then **evict** (delete) the corresponding cache entry — rather than updating the cache entry in place with the new value.

**Why evict rather than update-in-place on write — the real design question here:** updating the cache directly on every write means keeping two copies of the truth (the database row and the cached value) in sync on every single write path, forever — and any write path that forgets this step, or any concurrent-write ordering issue, leaves the cache silently wrong, which is a much worse failure mode than a cache miss (a wrong answer served confidently, vs. a correct answer that's merely slightly slower to fetch). Evicting instead means the *next read* naturally repopulates the cache from the database — the actual source of truth — guaranteeing the cache can never drift from the database for longer than one read cycle, at the cost of that next read paying a cache-miss penalty. Simplicity and a bounded staleness window win out over an update path that has to be gotten right on every single write, forever.

### Implementing It: `@Cacheable` and `@CacheEvict`

```java
@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10));
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}

@Service
public class ProductService {

    private final ProductRepository productRepository;
    // constructor omitted

    @Cacheable(value = "products", key = "#productId")
    public Product getProduct(Long productId) {
        // this method body only executes on a cache MISS
        return productRepository.findById(productId)
            .orElseThrow(() -> new ProductNotFoundException(productId));
    }

    @CacheEvict(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }
}
```

**The mechanism underneath, connected directly to what you already know:** `@Cacheable` and `@CacheEvict` are **proxy-based**, the exact same interception mechanism as Spring AOP itself (Week 9, Day 62) and Resilience4j's Circuit Breaker (Week 10, Day 64). Spring wraps `ProductService` in a proxy at startup; a call to `getProduct()` from *outside* the class goes through that proxy first, which checks the cache before ever letting the real method body run. The same **self-invocation limitation** you've already seen twice applies identically here: if a method *inside* `ProductService` called `this.getProduct(id)` directly, that call never passes through the proxy — no caching happens at all, silently. Recognizing this as the same mechanism showing up a third time, rather than a new fact to memorize, is exactly the kind of connection worth making explicit out loud in an interview.

**How to actually verify caching is working — the project's own definition of done:** add logging (or a Mockito spy, Week 7, Day 46) on `ProductRepository.findById()`, call `getProduct()` twice with the same ID, and confirm the repository — and therefore the database — is only hit once. A passing test here is a genuine correctness check, not just a manual "it looks faster" observation.

**Complexity:** cache hit — `O(1)`, a Redis `GET`. Cache miss — the underlying database query's own cost, plus one Redis `SET`. Eviction — `O(1)`, a Redis `DEL`.

**Edge cases:** a `@CacheEvict` call on an entity that was never cached (a no-op — `DEL` on a missing key is safe and cheap, not an error); a read racing a concurrent write's eviction (the read either sees the old cached value or correctly misses and reloads — the bounded-staleness argument above already covers this as an accepted trade-off, not a bug); the TTL configured in `CacheConfig` expiring naturally even with no write at all, which is a second, independent reason a given key eventually gets refreshed from the database besides an explicit eviction.

---

## Project Block

**Repository:** `scalable-ecommerce-platform`. **Task:** the `@Cacheable`/`@CacheEvict` wiring above against Redis for the Product module. **Definition of done:** fetching the same product twice logs a real database query only on the first fetch — verified with a repository spy or logging, not just observed manually.

## Career Block

**LinkedIn engagement:** 20 minutes commenting on 3–5 posts.

**Networking:** reach out to a peer for a casual virtual coffee chat — a lower-stakes networking action than a cold outreach, worth mixing in.

## Daily Deliverable Check

- [ ] Can explain Thundering Herd and defend at least two mitigations, including which data characteristics favor each, without notes.
- [ ] Cache-Aside live on the platform's Product module, with a verified test proving the database is only hit on a cache miss.
- [ ] Can explain why Cache-Aside evicts on write rather than updating the cache in place.
- [ ] Can explain how Consistent Hashing (Week 9, Day 60) applies unmodified to cache-node membership changes.

---

## Day 123 — Interview Questions

---

**1. What's the core difference between Memcached and Redis, and why is Redis usually the more versatile default today?**

*Answer:* Memcached is a pure, multi-threaded key-value cache with no persistence and no richer data types. Redis adds richer data structures (sorted sets, lists, geospatial indexes), optional persistence, and pub/sub — letting one running instance serve as a cache, a shared counter store, and more, rather than being limited to caching alone.

---

**2. Precisely, why does Thundering Herd happen?**

*Answer:* When a popular cache key expires (or the cache is cold), many concurrent requests can independently see a cache miss within the same brief window and all query the database for the identical data at once — turning the cache's protective role into a synchronized spike of redundant database load right at the moment of expiration.

---

**3. Why doesn't a shorter TTL fix Thundering Herd?**

*Answer:* TTL controls when an entry expires, not how many concurrent requests discover that expiration simultaneously — a popular key is, by definition, hit by many concurrent requests, so its expiration moment is exactly when the herd condition is most likely regardless of how the TTL is tuned.

---

**4. Describe the stale-while-revalidate mitigation and why it doesn't need a distributed lock.**

*Answer:* Store a logical expiry earlier than the actual Redis TTL. Once logically stale but still physically present, a request gets the stale value back immediately (no blocking) while triggering a background refresh for the next reader. No lock is needed because serving the stale value directly avoids every request racing to hit the database at once.

---

**5. What's the trade-off between a repopulation mutex and stale-while-revalidate?**

*Answer:* A mutex guarantees exactly one database read per expiration, at the cost of building and safely operating a distributed lock. Stale-while-revalidate avoids the lock entirely but accepts serving briefly stale data — acceptable for something like a product listing, less acceptable for an account balance.

---

**6. How does Consistent Hashing (Week 9, Day 60) apply to a distributed cache specifically?**

*Answer:* It determines which cache node owns which keys via the ring and `ceilingKey()`, so when a node is added or removed, only the keys in the adjacent ring segment get remapped — most of the cache stays warm on the same node instead of the whole cache going cold on any membership change.

---

**7. Describe Cache-Aside's read path and write path separately.**

*Answer:* Read: check cache first; on a hit return it; on a miss read from the database, populate the cache, then return. Write: update the database, then evict the corresponding cache entry rather than updating it in place.

---

**8. Why evict on write instead of updating the cache with the new value directly?**

*Answer:* Updating the cache on every write means keeping two copies of the truth in sync forever, and any missed or misordered update leaves the cache silently wrong — a worse failure than a cache miss. Evicting lets the next read repopulate correctly from the actual source of truth, bounding staleness to one read cycle.

---

**9. What mechanism does `@Cacheable` use to intercept a method call, and what prior material is this the same mechanism as?**

*Answer:* A Spring proxy wraps the bean at startup and intercepts external calls before the real method body runs — the identical proxy-based AOP mechanism as Spring AOP itself (Week 9, Day 62) and Resilience4j's Circuit Breaker (Week 10, Day 64).

---

**10. Why does calling a `@Cacheable` method from inside the same class silently skip caching?**

*Answer:* Self-invocation calls the real object directly, not through the proxy that implements the caching behavior — the exact same self-invocation limitation already seen with Spring AOP and Resilience4j.

---

**11. How would you actually verify `@Cacheable` is working, rather than just assuming it from the code?**

*Answer:* Add a spy or logging on the repository call it wraps, call the cached method twice with the same argument, and assert the underlying repository — and therefore the database — was only hit once.

---

**12. Name two of Redis's richer data structures beyond plain key-value strings, and one problem each is naturally suited to.**

*Answer:* Sorted sets — naturally suited to ranked data like a leaderboard (full treatment Week 19, Day 132). Geospatial indexes — naturally suited to "find things near this location" queries (full treatment Week 19, Day 128, Uber's driver tracking).

---

**13. Why might a pure cache deliberately run with Redis persistence turned off?**

*Answer:* A cache miss just falls through to the source of truth — a cheap, self-healing failure mode by design — so there's little value in paying persistence's overhead to protect data that's disposable and trivially reconstructable on a restart.

---

**14. What's the complexity of a Cache-Aside hit, a miss, and an eviction?**

*Answer:* A hit is O(1) — a single Redis `GET`. A miss is the underlying query's own cost plus one Redis `SET`. An eviction is O(1) — a single Redis `DEL`.

---

## What Tomorrow Assumes You Already Know Cold

Day 124 assumes today's Redis fluency — TTLs, the idea of a value with metadata alongside it — carries forward directly, since a Snowflake ID's machine-ID assignment is a genuinely different problem, but the general comfort with "state that needs to be globally visible can't just live in one process's memory" (first established distributed-rate-limiting, Day 121, then reinforced today) is exactly the instinct tomorrow's ID generation problem needs again, applied to yet another shape.

**Next:** [Day 124 Resource Book](./Day124_Resource_Book.md) — Distributed ID Generation.
