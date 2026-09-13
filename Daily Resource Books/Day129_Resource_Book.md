# Day 129 — HLD #10: Netflix / Video Streaming, and HLD #11: Instagram Feed

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 128 Resource Book](Day128_Resource_Book.md)
**Next ▶:** [Day 130 Resource Book](Day130_Resource_Book.md)
**Companion to:** Day 129 of `Week_19_Revised.md`

---

## Recap

Day 128 closed with the series' first write-dominated system and a mock defending already-known material. Today is two systems in one day, each lighter than yesterday's single deep dive, and both are really extensions of ideas already on the table rather than fully new territory: Netflix's CDN story is Day 123's Cache-Aside pattern relocated to a geographically distributed edge tier, and its Thundering Herd risk (Day 123) resurfaces at global scale, complete with the same proactive-warming fix Day 126 already used. Instagram's fan-out problem is the direct, fully-solved version of a shape this series has now touched **three times** without solving it: Day 78 named "the celebrity problem" for sharding, Day 122's per-channel notification fan-out was a much smaller-scale cousin, and Day 127's hot-conversation partition risk flagged the same shape again just yesterday-before-yesterday. Today it finally gets a real answer.

---

## Learning Objectives

By the end of today, without notes:

1. Explain adaptive bitrate streaming's actual mechanism — chunking and manifest-driven quality switching — not just "it adjusts to your internet speed."
2. Explain precisely why a CDN is Cache-Aside relocated to the network edge, and why that reframing predicts CDNs inherit Thundering Herd too.
3. State, with real numbers, exactly why pure push collapses for a celebrity account, and derive hybrid fan-out as the fix rather than recalling it as a memorized answer.
4. Apply the HLD framework's Estimation and Bottleneck steps to two systems in one sitting, keeping each appropriately concise for the time available.

---

## Concept Dependency Map

```
Day 78: Sharding — "the celebrity problem" (named)
Day 122: Notification System — per-channel fan-out (a small-scale cousin)
Day 123: Distributed Cache — Cache-Aside, Thundering Herd + mitigations
Day 126: BookMyShow at Scale — proactive cache warming
Day 127: WhatsApp — hot-conversation partitions (the celebrity problem, again)
Day 48: Kafka Fundamentals (async pipeline backbone, applied again today)
        │
        ▼
Today, System 1 — Netflix / Video Streaming
  ├─ Transcoding Pipeline (NEW)
  ├─▶ Adaptive Bitrate Streaming (NEW — needs: Transcoding, above)
  └─▶ CDN (NEW — reframes Day 123's Cache-Aside at the edge;
        inherits Day 123's Thundering Herd + Day 126's warming fix)

Today, System 2 — Instagram / Feed Generation
  ├─ Push / fan-out-on-write (NEW — the celebrity problem's actual solution begins here)
  ├─ Pull / fan-out-on-read (NEW)
  └─▶ Hybrid fan-out (NEW — closes the loop Days 78, 122, and 127 all opened)
```

---

## Part 1 — Netflix / Video Streaming

### Requirements (Step 1)

Upload once, serve at whatever quality a viewer's current network can sustain, to a global, geographically dispersed audience, with minimal startup delay and minimal mid-playback rebuffering.

### Transcoding Pipeline

**Definition:** converting an uploaded master file into multiple output renditions — different resolutions and bitrates — as an offline, asynchronous process, before any viewer ever requests the video.

**Mechanism:** upload completion publishes an event (the same Kafka-driven async pattern from Day 48 onward, applied here rather than re-derived); a fleet of transcoding workers consumes that event and produces the full rendition ladder (say, 1080p/high, 1080p/low, 720p, 480p, 240p), writing each as a separate file to blob storage.

**Why async, specifically:** encoding every rendition is CPU/GPU-intensive and can take minutes for a long file — doing this synchronously on upload would force the uploader to wait unacceptably. Decoupling it lets the pipeline scale independently and absorb bursty upload traffic without blocking anyone.

**Cost, concretely:** every additional rendition roughly multiplies storage for that title — a title stored at 6 renditions costs meaningfully more than the original alone, a real number worth naming rather than treating storage as free.

### Adaptive Bitrate Streaming (ABR)

**Mechanism:** each rendition is split into short chunks (2–10 seconds), all renditions' chunks aligned at the same timestamps; a manifest file lists every available rendition and its chunk URLs. The player downloads one chunk, measures how long that download actually took against the chunk's own playback duration to estimate current bandwidth, and picks the **next** chunk's quality level accordingly — up if bandwidth is generous, down if it's constrained. Because chunks are independently decodable and aligned across renditions, the player can switch quality **at a chunk boundary** with no visible glitch or restart.

**Why it works:** it never wastes bandwidth on quality the network can't sustain (avoiding rebuffering) and never serves needlessly low quality when bandwidth is generous — a continuous, self-correcting feedback loop rather than a single quality choice made once at the start.

**Trade-off:** it requires a discrete ladder of pre-encoded quality "rungs" rather than one continuously variable stream — simpler to produce and cache, at the cost of never matching bandwidth *exactly*, only to the nearest available rung.

### CDN

**Definition:** a globally distributed network of edge servers caching content physically close to end users, so a request is served from a nearby node instead of round-tripping to one, possibly distant, origin.

**Mechanism — and the direct reframe worth stating explicitly:** a user's request routes to their nearest edge location; if that edge already has the content cached, it serves directly; if not, it fetches from origin exactly once, caches the result, and serves every subsequent nearby request from that cache from then on. This is **Day 123's Cache-Aside pattern**, relocated from an application-adjacent Redis instance to a geographically distributed edge tier — same shape (check cache, fall through on miss, populate, serve locally next time), different physical layer.

> 🔑 **Key Takeaway:** recognizing a CDN as "Cache-Aside at the edge" isn't just a tidy analogy — it correctly predicts that a CDN inherits Cache-Aside's known failure mode too.

**And it does:** a hugely anticipated new release drops, and **every** edge node worldwide experiences a cache-miss stampede to origin simultaneously — Day 123's Thundering Herd, now global instead of single-node. The mitigation is the one already established: **pre-warm** the content at edge locations ahead of the known release time, exactly Day 126's proactive-cache-warming extension, reapplied at this layer rather than invented fresh.

> ⚠️ **Common Mistake:** assuming a CDN removes origin load uniformly. It removes it for *popular, cached* content — video viewership is heavily skewed toward a small set of popular titles, which is precisely the distribution a CDN is economically built around. Unpopular, long-tail content still pays close to full origin-fetch latency on every access, since it's rarely cached anywhere.

### Estimation (Step 2)

Assume **200 million subscribers**, averaging **2 hours** of viewing per day, at a blended average bitrate around **3 Mbps** across the rendition mix actually being watched.

- **Aggregate sustained bandwidth:** 200,000,000 × 2 hrs × 3 Mbps, spread across the day, lands in the **multiple-terabits-per-second** range sustained.
- **Why this number is the whole argument for a CDN, not decoration:** serving that volume from one origin, repeatedly, to a globally dispersed audience, is not merely slower without edge caching — at this scale it's simply infeasible on any single origin's network capacity. The estimation step doesn't just describe scale here; it's the actual justification for the CDN design decision, the same causal role estimation played for Day 127's connection registry.

### Bottlenecks (Step 5)

Thundering Herd at the edge on a hot new release (mitigated by pre-warming, above); long-tail content cache-miss rate (an accepted, economically reasonable cost, not a bug to eliminate); transcoding pipeline backlog during a burst of simultaneous uploads (mitigated the same way any queue-backed pipeline is — scale worker count with backlog depth, Day 48's own consumer-group scaling logic).

---

## Part 2 — Instagram / Feed Generation

### Requirements (Step 1)

A user's feed shows recent posts from everyone they follow, roughly time-ordered (ranking algorithms are explicitly out of scope for today — the fan-out mechanism underneath is the actual system-design question here).

### Push (fan-out-on-write)

**Definition:** the instant a user posts, that post is written directly into every one of their followers' precomputed feed lists.

**Mechanism:** post creation publishes an event; an async fan-out job enumerates the poster's follower list and inserts a reference to the new post into each follower's own feed cache (typically a capped, recent-N structure per user).

**Trade-off:** reads become trivial — a feed open is just "fetch my already-assembled list," O(1)-ish regardless of how many people the user follows. Writes cost is proportional to **follower count** — fine for a typical user, catastrophic for one with millions.

### Pull (fan-out-on-read)

**Definition:** nothing is precomputed; a feed is assembled on demand, at read time, by querying everyone the user follows and merging results.

**Mechanism:** for each of the N followed accounts, fetch their K most recent posts, then merge the N result sets by timestamp — structurally a K-way merge, the same shape underlying Merge K Sorted Lists if that pattern's worth a cold gut-check right now.

**Trade-off:** writes are trivial (nothing precomputed at post time). Every single feed **read** pays a cost proportional to how many accounts the user follows — fine occasionally, expensive at real scale since, unlike push, this cost is paid on *every* feed open, not concentrated at post time.

### Hybrid Fan-out — closing the loop

**Definition:** push for the overwhelming majority of accounts with ordinary follower counts; pull (or a deliberately capped push) specifically for celebrity accounts whose follower count would make full push prohibitively expensive.

**Mechanism:** a user's assembled feed = their precomputed pushed feed, **merged at read time** with a live pull of just the handful of celebrity accounts they follow — the pull portion only ever spans a small, bounded number of accounts (the celebrities), never the user's entire follow list, keeping that read-time cost small regardless of how many ordinary accounts they follow.

**Why this specific split is correct, not arbitrary:** it matches cost to where it's actually cheap. Most posts come from accounts with modest followings, where push's per-post cost is genuinely small and the read-speed payoff is large. A celebrity's post is rare relative to their follower count; paying a bounded pull-time cost for just that one account, merged in alongside an already-fast precomputed feed, is dramatically cheaper than fanning out to millions of followers on every single post.

> 🔗 **This is the resolution** of the exact shape Day 78 named ("the celebrity problem"), Day 122 touched at small scale (per-channel fan-out for one event to a few channels — nothing like this volume), and Day 127 flagged again just two days ago (a massive group chat's hot partition). Three separate appearances of the same underlying tension, one real fix.

### Coding Exercise — ~150 words, spoken-answer length

*Prompt: why does a pure push strategy specifically break down for celebrity accounts? Use real numbers.*

> Pure push writes a new post into every follower's precomputed feed the instant it's created. For a typical user with a few hundred followers, that's a few hundred cheap writes — fine. For a celebrity with 100 million followers, one single post triggers 100 million fan-out writes, essentially instantly. Even at a generous sustained 50,000 writes/second on the fan-out pipeline, clearing that one post's backlog takes roughly 2,000 seconds — over half an hour — during which followers see a stale feed and the write infrastructure is saturated by one person's post. Worse, celebrities post relatively often and unpredictably, so this isn't a rare edge case worth shrugging off — it's a recurring, self-inflicted overload event. The fix isn't making push faster; it's not paying that cost at write time for these specific accounts at all, and instead merging their posts in at read time, where cost stays proportional to the reader's own feed open, not the celebrity's follower count.

### Estimation (Step 2)

Assume **500 million DAU**, averaging **~300** followed accounts each, with the top ~1,000 celebrity accounts each carrying **50–100 million** followers (reusing the exercise's own 100M figure for consistency). A naive pure-push system sizes its worst case by celebrity follower count, not typical follower count — exactly why the estimation step here has to explicitly consider the tail, not just the average user, or the resulting design misses the system's actual bottleneck entirely.

### Bottlenecks (Step 5)

Celebrity fan-out cost under pure push (solved above); pull-time merge cost growing with how many celebrities a single user follows (bounded in practice — nobody follows thousands of mega-celebrities, unlike ordinary follow counts which can genuinely reach into the hundreds); feed cache staleness for pushed feeds if the async fan-out job falls behind under load (mitigated the same way any Kafka-consumer backlog is, per Day 48).

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`. **Task:** sketch — design only, no implementation — a hybrid fan-out approach for a hypothetical "product recommendations feed" feature, applying today's theory to the platform's own domain.

**Design doc skeleton (`docs/product-feed-fanout.md`):**

```markdown
# Product Recommendations Feed — Hybrid Fan-out Design

## Requirement
Users follow specific brands/sellers. When a followed brand publishes a
new product or promotion, it should appear in the user's personalized
feed shortly after — the exact same shape as Instagram's post-to-feed
problem, applied to sellers instead of individual users.

## Why hybrid, not pure push or pure pull
Most sellers have a modest number of followers — push is cheap and keeps
reads fast for the common case. A small number of large brand partners
carry follower counts large enough that a full push, on every promotion,
would be exactly Instagram's celebrity problem — reproduced here at the
platform's own scale.

## Design
- Push: on a normal seller's new post, fan out into each follower's
  precomputed feed cache (Redis, capped recent-N per user).
- Pull: large-brand-partner posts are NOT pushed. A user's feed read
  merges their precomputed pushed feed with a live pull of just the
  handful of large-brand-partners they follow — bounded, since no user
  follows more than a small number of these.
- Threshold: any seller crossing a defined follower count (e.g., 500K)
  moves from the push path to the pull path going forward.
```

**Definition of done:** the short doc above, adapted with the platform's own actual module boundaries and Redis usage, pushed to `docs/`.

---

## Career Block Guide (1 hr)

**LinkedIn Post 25 — a draft to adapt, not copy verbatim:**

> **Push vs. pull: why your feed algorithm has to change for a celebrity account.**
>
> Pure "push on write" — instantly copying a new post into every follower's feed — works great until one account has 100 million followers. One post from that account triggers 100 million writes, essentially at once. Even at a generous 50K writes/second, that's over half an hour just to clear one post's backlog, with followers staring at a stale feed the whole time.
>
> The fix isn't a faster pipeline — it's recognizing that a small number of accounts need a completely different strategy: pull their posts in live, at read time, instead of pushing them everywhere in advance. Everyone else still gets the fast, precomputed path.
>
> A good reminder that "the algorithm" isn't one algorithm — it's whichever one matches the actual shape of the cost you're trying to avoid.

**Networking:** engage heavily today — comment on 5 posts, more than the usual 3–5 baseline, reflecting today's specific plan emphasis.

---

## Day 129 — Interview Questions

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

## Daily Deliverable Check

- [ ] Can explain adaptive bitrate streaming and CDN caching without notes.
- [ ] Can explain hybrid fan-out and why pure push fails for celebrity accounts, without notes, including the arithmetic.
- [ ] Hybrid fan-out design doc for the product recommendations feed pushed to `docs/`.
- [ ] LinkedIn Post 25 published.

---

## What Tomorrow Assumes You Already Know Cold

Day 130 assumes the "check-then-act" shape — recognized informally four separate times already across Days 121, 122, 124, and 126 — is fully available to be named and generalized tomorrow, since Day 130 is where it finally gets its formal, general-purpose treatment. It also assumes today's habit of reusing an existing number across a design's exercise, estimation, and career post (the 100M-follower figure, kept consistent throughout) continues — internal consistency of the numbers you actually say out loud is itself something interviewers notice.
