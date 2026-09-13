# Micro-Primer: Redis Basics — Needed Before Day 44

**First needed:** Day 44, where Redis is added to `docker-compose.yml` alongside PostgreSQL with zero explanation. Properly and fully explained 79 days later, on Day 123 (Distributed Cache).

**Redis** is an in-memory data store. Unlike PostgreSQL, which persists data to disk as its source of truth, Redis keeps data in RAM — making reads and writes extremely fast, at the cost of being less durable by default (though it can optionally persist to disk too).

The simplest way to think about it: a **key-value store** — you set a value under a key, and get it back by that key. Conceptually similar to a `HashMap`, except it lives outside your application process, is reachable over the network, and can be shared across multiple instances of your app at once.

Why Day 44 wires it into the compose file *now*, before anything actually uses it: it's standing up the infrastructure ahead of when it's needed — for now, "bring the container up and confirm it's part of the running stack" is the entire task. Nothing reads or writes to it yet.

**Deferred:** what Redis is actually good for beyond this (caching, rate limiting, session storage), how it compares to Memcached, and its richer data structures (Sorted Sets, Geo commands) — all Day 123 and the HLD weeks that follow it (Days 128, 130, 132).

### Checklist
- [ ] Can explain what "in-memory" means and why it makes Redis fast.
- [ ] Understands Day 44's task is purely infrastructural — nothing to build on top of it yet.
- [ ] Ready for Day 44.
