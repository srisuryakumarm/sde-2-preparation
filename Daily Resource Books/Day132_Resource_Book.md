# Day 132 — HLD #14: Leaderboard System, and HLD #15: Search Autocomplete

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 131 Resource Book](Day131_Resource_Book.md)
**Next ▶:** [Day 133 Resource Book](Day133_Resource_Book.md)
**Companion to:** Day 132 of `Week_19_Revised.md`

---

## Recap

Sorted Sets have been used three times now without ever explaining *why* they behave the way they do: named as one of Redis's data structures on Day 123, used for geospatial indexing on Day 128, used again for delayed-job polling yesterday — every time treated as a black box with a proven O(log n) contract, the exact same relationship this series has had with `TreeMap`'s Red-Black tree since Week 1, Day 17. Today that box finally opens: the **Skip List** is the structure underneath, and it's built from scratch, mechanism first, today. The day's second system, Search Autocomplete, is lighter by comparison — it needs no new data structure at all, only a new way of *deploying* one already fully taught, Week 9's Trie.

---

## Learning Objectives

By the end of today, without notes:

1. Trace a Skip List search by hand and explain why expected search cost is O(log n), not merely assert it.
2. Explain precisely why random node height, not a deterministic promotion rule, is the engineering choice that avoids rebalancing entirely.
3. State why a Skip List beats a B-tree specifically for high-frequency rank updates, grounded in each structure's actual write-path cost, not a memorized preference.
4. Explain why a search-autocomplete Trie can be served from a periodically-rebuilt, read-only replica rather than updated live, and why that's a deliberate trade-off, not a shortcut.

---

## Concept Dependency Map

```
Day 17: TreeMap — used by its Red-Black-tree O(log n) contract, mechanism
        never taught (the established precedent today's approach follows)
Day 40: B-tree indexes — the write-cost baseline Skip Lists are contrasted against
Day 59–61: Tries (opened, closed, full depth) — Week 9
Day 123: Sorted Sets — named, not built
Day 128, 131: Sorted Sets — used operationally (Geo, delayed-job polling)
        │
        ▼
Today, System 1 — Leaderboard
  ├─ Skip List (NEW, full mechanism — the deferred payoff from all three uses above)
  ├─▶ Redis Sorted Sets, ZADD/ZRANGE/ZRANK (APPLIED — now with mechanism in hand)
  ├─ Redis Cluster (NEW, brief — contrasted precisely against Consistent Hashing, Day 60/123)
  └─ Tie-handling via composite score (NEW)

Today, System 2 — Search Autocomplete
  └─▶ Distributed, Asynchronously-Rebuilt Trie (NEW wrapper around Week 9's
        already-fully-taught Trie structure — no new data structure needed)
```

---

## Part 1 — Requirements (Step 1, Leaderboard)

A "top-selling products this week" leaderboard: scores (sales counts) update continuously and frequently; users query the current standings far more often than any single score changes, but score changes themselves are frequent enough — a bursty flash-sale event especially — that update cost genuinely matters, not just read cost.

---

## Part 2 — The Skip List

### Prerequisites (confirmed)
- Linked lists, and traversal cost (Week 1–2).
- B-tree indexes and their write-time rebalancing cost (Week 6, Day 40) — the direct point of contrast below.

### Definition

A Skip List is a probabilistic structure built from multiple **levels** of sorted linked lists. Level 0, the bottom, contains every element, fully sorted. Each level above contains a randomly-chosen subset of the level below it — roughly half, if each element is independently promoted with probability ½ — acting as **express lanes** that let a search skip over many elements at once before dropping down to a denser level for the final approach.

### Mechanism — search, traced by hand

```
Level 2:  [1] ─────────────────────────── [9] ──────────────── [17]
Level 1:  [1] ────────────── [5] ──────── [9] ──────── [13] ── [17] ─ [21]
Level 0:  [1]─[3]─[5]─[7]─[9]─[11]─[13]─[15]─[17]─[19]─[21]
```

**Searching for 13:**

| Level | Current | Next | Compare | Action |
|---|---|---|---|---|
| 2 | (head) | 1 | — | move to 1 |
| 2 | 1 | 9 | 9 ≤ 13 | move to 9 |
| 2 | 9 | 17 | 17 > 13 | drop to level 1 |
| 1 | 9 | 13 | 13 ≤ 13 | move to 13 |
| 1 | 13 | 17 | 17 > 13 | drop to level 0 |
| 0 | 13 | — | — | **found** |

Five comparisons reached the target, and — this is the actual mechanism, not a side effect — nodes `3, 7, 11, 15, 19, 21` were **never inspected at all**. At this tiny scale (n=11) that's a modest saving over a level-0-only linear scan (which needs 7 comparisons: 1, 3, 5, 7, 9, 11, 13); the ratio understates the real win, which grows multiplicatively as n grows, since every additional level roughly doubles how much of the bottom list a single top-level hop represents.

### Why random height — the actual engineering insight

A node's height is decided by an independent coin flip at insertion time: start at level 0, and keep promoting one level higher for as long as the flip comes up heads (capped at some maximum). This is **not** the only conceivable design — a scheme that deterministically promotes exactly every 2nd element would also give a layered structure — but a deterministic scheme has to actively **maintain** that exact invariant as elements are inserted and deleted, which means rebalancing other, unrelated nodes on every write, the same structural cost a B-tree pays. Random height means inserting a new node is a purely **local** splice — walk down recording predecessors (the same traversal as search), flip coins to decide this one node's height, splice it into each level up to that height — with zero need to touch, inspect, or restructure any other existing node.

> 🔑 **Key Takeaway:** this is the entire reason Skip Lists are so much simpler to implement correctly than a balanced tree while achieving the same expected complexity — randomness replaces rebalancing.

### Complexity, proven

With promotion probability ½, the expected number of levels is O(log n) (each level holds roughly half the elements of the one below). At any single level, a rightward search step is expected to encounter about 2 nodes before hitting one that's also present in the level above — meaning, on average, only O(1) rightward hops happen before a search drops down a level. Total expected cost: O(log n) levels × O(1) hops per level = **O(log n) expected**. This is a *probabilistic* guarantee, not a worst-case one — an adversarially unlucky sequence of coin flips could in principle produce a degenerate structure, but this is vanishingly unlikely in practice and, critically, requires no active rebalancing to avoid, unlike a naive unbalanced BST's worst case (a sorted-order insertion sequence), which is both easy to trigger by accident and genuinely O(n).

> ⚠️ **Common Mistake:** stating Skip List operations as a deterministic O(log n) worst-case guarantee. They're expected/probabilistic — reliable enough in practice to be Redis's actual production choice, but not the same kind of guarantee a balanced tree's rotation invariant provides.

### Why this beats a B-tree specifically for real-time rank updates

A B-tree index (Day 40) is tuned for read-heavy workloads with comparatively rare writes — every insert or delete can trigger node splits, merges, or rebalancing to preserve the tree's structural invariants (uniform leaf depth, node fill-factor bounds), an expensive operation under a workload where scores are changing **constantly**. A Skip List's insert or update is the local splice described above — updating one element's position never requires touching or restructuring any *other* unrelated element. For a leaderboard where every scoring event is a write to an already-present member's position, that difference is the entire argument.

**The concrete payoff for rank queries specifically:** Redis's actual Sorted Set implementation augments each forward pointer with a **span** — how many level-0 elements that pointer skips over — letting `ZRANK` compute a member's exact rank by summing spans traversed during the same search already being done to locate it, in the same O(log n) pass. This is the specific, named reason Redis chose this structure for exactly this use case, not an incidental detail.

---

## Part 3 — Redis Sorted Sets, Applied (with the mechanism now in hand)

```
ZADD leaderboard:this-week 1500 "product:42"
ZADD leaderboard:this-week 2300 "product:88"
ZINCRBY leaderboard:this-week 50 "product:42"     -- a sale event, updating in place
ZREVRANGE leaderboard:this-week 0 9 WITHSCORES    -- top 10
ZRANK leaderboard:this-week "product:42"          -- exact current rank
```

Every one of these commands is now explainable at the mechanism level: `ZADD`/`ZINCRBY` are Skip List inserts/local-splice updates; `ZREVRANGE` is a level-0 traversal reached via the express lanes; `ZRANK` is a search that accumulates span along the way.

---

## Part 4 — Redis Cluster (brief — and precisely distinguished from Consistent Hashing)

**Mechanism:** Redis Cluster shards keys across nodes using a **fixed set of 16,384 hash slots** — each key is assigned to a slot via `CRC16(key) mod 16384`, and each cluster node owns some subset of these slots.

> ⚠️ **Worth being precise about, not conflating:** this is a **different** engineering approach from Consistent Hashing (Day 60, recapped Day 123), even though both solve "distribute keys across nodes, handle membership changes gracefully." Consistent Hashing minimizes remapping when a node joins or leaves by using a hash *ring*'s geometric properties. Redis Cluster instead handles membership changes by explicitly **migrating specific slots** between nodes — an administrative/automatic rebalancing operation, not a ring-based property. Two different solutions to a similar-sounding problem; naming them as if they're the same mechanism would be exactly the kind of imprecise claim worth avoiding.

**Why it matters here, and an honest limitation:** Redis Cluster lets a leaderboard scale past one node's throughput ceiling — but sharding a Sorted Set breaks the single-node `ZRANK`'s clean global-rank property, since a member's rank *within its own shard* is no longer its true rank across the whole dataset. Real large-scale leaderboards typically sidestep this rather than solve it exactly: most use cases genuinely only need "top N" plus "my own rank," not a precisely correct global rank for every single member, so systems maintain a smaller, separately-aggregated top-N view rather than a perfectly accurate sharded rank for everyone.

---

## Part 5 — Tie-Handling

Redis breaks same-score ties by comparing member name lexicographically by default — rarely the semantically correct choice for a leaderboard, where the desired rule is usually **earliest achievement wins the higher rank**.

### The technique: a composite score

Encode the tiebreaker directly into the numeric score: `combinedScore = actualScore × BIG_CONSTANT − timestamp`, where `BIG_CONSTANT` is chosen large enough that a difference of even 1 in `actualScore` always outweighs any possible difference in `timestamp`.

**Worked, verified example** (small toy numbers, `BIG_CONSTANT = 1,000,000`):

- Player A: score 1000, achieved at timestamp 5 (earlier). `combinedScore = 1000×1,000,000 − 5 = 999,999,995`.
- Player B: score 1000, achieved at timestamp 8 (later). `combinedScore = 1000×1,000,000 − 8 = 999,999,992`.
- 999,999,995 > 999,999,992 → **A ranks higher**, correctly rewarding the earlier achievement.

**Confirming the constant doesn't let a tiebreaker override a genuine score difference:**

- Player C: score 1001 (just one point more), achieved very late, timestamp 999,999. `combinedScore = 1001×1,000,000 − 999,999 = 1,000,000,001`.
- Compare to A's 999,999,995 → C (1,000,000,001) still ranks above A, because a single point of `actualScore` (worth 1,000,000 in the combined score) vastly outweighs any realistic timestamp difference. The constant is doing exactly its job: dominate the primary comparison, and only ever break exact ties.

> ⚠️ **Common Mistake:** picking `BIG_CONSTANT` without checking it actually dominates the full realistic range of the tiebreaker term. Using real Unix timestamps (~10⁹–10¹⁰ magnitude) requires a correspondingly larger constant than this toy example's `1,000,000` — the arithmetic must be re-verified against real magnitudes, not assumed to transfer automatically.

---

## Part 6 — Search Autocomplete: A Distributed, Asynchronously-Rebuilt Trie

### Prerequisites (confirmed)
Tries, opened and closed in full — structure, traversal, O(L) cost independent of how many total words are stored (Week 9, Days 59–61).

### What's actually new today

Not the Trie itself — nothing about its structure or traversal needs re-teaching. What's new is **how it's deployed**: as a read-only, replicated, periodically-rebuilt serving structure, deliberately decoupled from the live search traffic that eventually feeds it.

### Mechanism

Search logs are aggregated continuously but processed **offline**, in batch — a scheduled job counts query frequency and rebuilds the Trie, precomputing and caching each node's **top-K most frequent completions in its own subtree** directly on that node, during the build. The rebuilt Trie deploys on a schedule (hourly, say), replicated across many read-serving instances that each answer queries independently, with no coordination needed between them at query time — a read-only structure has no write-path to reason about mid-serving at all.

**Why precomputing top-K per node matters:** without it, answering "top completions for this prefix" costs O(L) to reach the prefix's node plus a full traversal of that node's entire subtree to find the best completions — expensive for a popular short prefix with a huge subtree. With top-K cached at build time, the same query costs O(L) to reach the node plus O(1) to read the already-computed list.

### Why asynchronous rebuild is a deliberate trade-off, not a shortcut

> ⚠️ **Explicit anti-pattern, named directly by the plan's own framing:** updating the live Trie in real time, per search event, is unnecessary complexity here — autocomplete suggestions genuinely don't need per-keystroke real-time exactness. A trending query might take up to the rebuild interval to appear as a suggestion, which is an entirely acceptable trade for a dramatically simpler, cheaper, easier-to-scale serving path.

> 🔗 **Worth contrasting directly:** this is the opposite staleness tolerance from Day 130's payment correctness, which could not accept any equivalent slack. Recognizing *which* systems can tolerate this trade and which can't — not applying it uniformly — is the actual skill.

### Estimation, both systems (Step 2, kept compact — a two-system day)

**Leaderboard:** assume a flash-sale peak of ~50,000 score updates/sec against ~500,000 leaderboard reads/sec — reads dominate, the **opposite asymmetry from Day 128's Uber system** (write-dominated), worth naming as an explicit, deliberate contrast rather than treating every system as needing the same instinct.

**Autocomplete:** assume 500M DAU, ~5 searches/day each, ~8 keystrokes per search triggering a suggestion request: 500M × 5 × 8 = 20 billion requests/day ≈ **~230K requests/sec average** — enormous read volume against near-zero live write volume (writes only ever happen in the periodic batch rebuild, never per-request).

### Bottlenecks (Step 5, both systems)

Leaderboard: a hot single leaderboard key at extreme write volume (mitigated by Redis Cluster, with the ranking caveat above honestly noted). Autocomplete: batch-rebuild job duration growing with log volume (mitigated the same way any batch pipeline scales — parallelize the aggregation, same principle as every Kafka-consumer-group scaling argument already established).

---

## Coding Exercise — Explaining Skip List vs. B-tree Out Loud

*A model answer, sized to actually say in under a minute — practice delivering this from memory, not reading it:*

> "A B-tree index is built for read-heavy workloads with occasional writes — every insert can trigger a node split or rebalance to keep the tree's structural invariants intact, which is expensive when writes are constant, not occasional. A leaderboard's real-time rank updates are exactly that constant-write case — every scoring event moves an existing member's position. A Skip List handles that differently: updating a member's position is a local splice at each level the node participates in, using randomly-assigned heights instead of a rigid structural invariant, so no other node ever needs to be touched or rebalanced as a side effect. That's the entire reason Redis backs Sorted Sets with a Skip List rather than a B-tree — the write pattern here specifically favors it."

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`. **Task:** implement a "top-selling products this week" leaderboard using Redis Sorted Sets.

```java
@Service
public class LeaderboardService {

    private final RedisTemplate<String, String> redis;
    private static final String KEY = "leaderboard:products:this-week";

    public LeaderboardService(RedisTemplate<String, String> redis) {
        this.redis = redis;
    }

    public void recordSale(String productId, double units) {
        redis.opsForZSet().incrementScore(KEY, productId, units);
    }

    public Set<ZSetOperations.TypedTuple<String>> topN(int n) {
        return redis.opsForZSet().reverseRangeWithScores(KEY, 0, n - 1);
    }

    public Long rankOf(String productId) {
        return redis.opsForZSet().reverseRank(KEY, productId);
    }
}
```

**Definition of done:** `ZADD`/`ZRANGE`/`ZRANK` (via `incrementScore`/`reverseRangeWithScores`/`reverseRank` here) correctly maintain and query the leaderboard as sales data changes — verified by recording sales for several products and confirming both the top-N order and an individual product's rank update correctly after a new sale.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** if a warm connection exists at any of the 7 target companies but no referral ask has happened yet, this is a reasonable point to make it — directly, briefly, and only once the relationship has had at least one real conversation behind it, not as a first message.

---

## Day 132 — Interview Questions

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

## Daily Deliverable Check

- [ ] Can explain why Skip Lists beat B-trees for this specific workload, without notes, including the mechanism-level reason, not just the conclusion.
- [ ] Redis Sorted Set leaderboard live and correctly ranked on the platform, including a verified rank change after a new sale.
- [ ] Can deliver the Skip List vs. B-tree spoken answer from memory, in under a minute.

---

## What Tomorrow Assumes You Already Know Cold

Day 133 assumes the entire arc of this week — nine systems, most of them extensions or payoffs of concepts already seeded earlier rather than fully isolated topics — is available to be recalled as a connected whole, since tomorrow's self-check opens by asking for exactly that list from memory. It also assumes today's staleness-tolerance framing (autocomplete's acceptable lag vs. payment's zero tolerance) is reflexive, since tomorrow's object-storage sync-conflict theory asks the same kind of question — how much inconsistency a system can actually tolerate — one more time, in a new setting.
