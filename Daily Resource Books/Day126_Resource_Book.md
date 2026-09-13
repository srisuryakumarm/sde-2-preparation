# Day 126 Resource Book — BookMyShow at Scale, and Week 18 Consolidation

**Series:** SDE-2 Interview Prep Resource Books · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 125](./Day125_Resource_Book.md) · **Next ▶:** Week 19, Day 127
**Companion to:** Day 126 of `Week_18_Revised.md`

---

> ⚠️ **Discrepancy Notice — Read This First.** `00_Curriculum_Map.md`'s own forward-looking notes (written before `Week_18_Revised.md` existed in its final form) refer to this self-check as **"Day 125's."** The actual `Week_18_Revised.md` places it here, on **Day 126**. This Resource Book follows the actual plan file — the authoritative source for what each day requires — and the self-check below is Day 126's. This is flagged explicitly, and flagged again in today's curriculum map update, exactly the way this map has flagged its own past day-labeling discrepancies (Food Delivery/Hotel Booking, the Week 16 day range) rather than silently resolving it either way.

## Recap: what today is

Six systems deep, this week has built a full distributed-systems toolkit one piece at a time: sharding and the framework itself (Day 120), distributed rate limiting (Day 121), fan-out and deduplication (Day 122), caching and Consistent Hashing (Day 123), globally unique IDs (Day 124), and consensus (Day 125). Today doesn't add a new mechanism. It takes a system you already fully designed at LLD scale — BookMyShow, Week 17, Days 116–117 — and asks what changes when "one theater, one process" becomes "every theater, every city, at once." Every answer below reuses something already built this week; nothing here is new machinery, only new combination.

## Learning Objectives

By the end of today, without notes:

1. Explain why BookMyShow's LLD version never needed sharding, and why the HLD version can't avoid it.
2. Explain the seat-hold mechanism precisely — its atomicity, and why it needs no separate cleanup logic.
3. Recognize a high-demand launch as a Thundering Herd scenario, and explain why *known* timing changes the best mitigation.
4. Solve a Backtracking problem cold, unaided, confirming the pattern is still genuinely reflexive.

## Concept Dependency Map

```
Everything below was fully taught earlier this week or before — today combines it:
  Sharding Strategies ....................... Week 12, Day 78 · applied Day 120
  Redis TTL + Lua atomicity .................. Day 123 · Day 121
  Consistent Hashing / Cache-Aside /
    Thundering Herd ........................... Day 123
  Snowflake ID generation ..................... Day 124
  Quorum / Raft / consensus-backed
    leader failover ............................ Day 125
  BookMyShow's original LLD locking
    (SELECT FOR UPDATE, optimistic CAS) ........ Week 17, Days 116-117
  Backtracking (Combination Sum family) ....... Week 10, Day 64
        │
        ▼
Applied, combined, nothing new: BookMyShow at Scale
```

---

## Self-Check (15 min) — Solve Cold, Without Hints, Before Reading Further

**Combination Sum II** (LeetCode 40, Medium) — Backtracking, originally taught Week 10, Day 64.

**Statement:** given a collection of candidate numbers `candidates` (which may contain duplicates) and a target integer `target`, find all unique combinations where the numbers sum to `target`. **Each number in `candidates` may only be used once per combination**, and the result must not contain duplicate combinations.

> Set a 15-minute timer. Solve it cold — no scrolling ahead. What follows is the check, not the lesson; the mechanism itself was fully taught Week 10, Day 64.

---

*(solution and check below — attempt the problem above first)*

---

**Recap of the mechanism (Week 10, Day 64 — not re-taught):** sort first, then backtrack with a `start` index that advances to `i + 1` on each recursive call (each number used at most once), pruning any branch once `candidates[i] > remaining` (sorted array — nothing further in the loop can possibly work either).

**What's genuinely different from Combination Sum (LC 39), specifically:**
1. **No reuse:** the recursive call advances to `i + 1`, not `i` — LC 39 allowed reusing the same index indefinitely; here, each element is consumed once.
2. **Duplicate-skip:** the input can contain duplicate values, and the output must not contain duplicate *combinations* — handled by skipping a candidate when `i > start && candidates[i] == candidates[i - 1]`, which only skips a repeated value **at the same recursion level**, not a repeated value used at different depths within the same branch.

```java
public List<List<Integer>> combinationSum2(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(candidates);
    backtrack(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] candidates, int remaining, int start,
                        List<Integer> current, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(current));
        return;
    }
    for (int i = start; i < candidates.length; i++) {
        if (candidates[i] > remaining) break;                          // sorted — prune the rest of this level
        if (i > start && candidates[i] == candidates[i - 1]) continue; // skip a same-level duplicate
        current.add(candidates[i]);
        backtrack(candidates, remaining - candidates[i], i + 1, current, result); // i+1: used once
        current.remove(current.size() - 1);
    }
}
```

**Why `i > start`, specifically, and not `i > 0`:** `i > 0` would also skip a duplicate value that's the *first* choice at a **new** recursion level — which is wrong, since that value hasn't been tried yet at this level. `i > start` correctly means "skip this value only if it's identical to the previous value **already tried at this exact level**," while still allowing that same value to be chosen as the first pick when the recursion moves one level deeper.

**Trace — `candidates = [1,1,2,5,6,7,10]` (sorted), `target = 8`, focused on the duplicate-skip:** at the top level (`start = 0`), `i = 0` picks the first `1` — fully explored, eventually contributing `[1,1,6]`, `[1,2,5]`, `[1,7]`. Back at the top level, `i = 1` is also a `1` — but `i (1) > start (0)` and `candidates[1] == candidates[0]`, so it's **skipped**: exploring it would only reproduce combinations already found via `i = 0`. The full result: `[[1,1,6],[1,2,5],[1,7],[2,6]]`.

**Complexity:** time `O(2ⁿ)` worst case (each element either included or excluded), meaningfully reduced in practice by both the sort-enabled early-`break` pruning and the duplicate-skip; space `O(n)` for recursion depth plus the current combination (output storage aside).

**Edge cases:** every candidate exceeding `target` (no combinations — the `break` fires immediately at `i = start` on the very first call); an array that's entirely one repeated value (duplicate-skip must not also block using that value across *different* depths in one branch — it doesn't, since the skip check only fires at `i > start`, never blocking `i == start`); an empty result (a perfectly valid, correct output, not a bug to chase).

**Interview framing:** say out loud, before coding, that this is "Combination Sum's shape, but each number used once, plus duplicate values in the input needing a skip rule so the output has no duplicate combinations" — naming both differences unprompted is a stronger opening than diving straight into code. Likely follow-up: "what if you only needed the *count* of combinations, not the combinations themselves?" — worth having ready that a counting-only version can sometimes be reframed as a DP problem over `(index, remaining target)` state, though producing the actual combinations, as required here, needs the backtracking approach regardless.

**Self-check result:** ✅ solved cold, or ❌ needed a peek — either way, this is exactly the signal spaced repetition exists to surface; a ❌ here is useful information, not a setback, and is worth a second cold attempt in a few days rather than being drilled further right now.

---

## Part 1 — Why BookMyShow Needs Sharding Now

The LLD version (Week 17, Days 116–117) modeled one theater's seat map inside one process, with one database. **A single city, let alone a national platform, has thousands of theaters and tens of thousands of shows a day — no single database instance holds that comfortably, and no single process should be a single point of failure for every city's bookings simultaneously.** This is Sharding Strategies (Week 12, Day 78), applied here for the second time this week — **shard the booking database by city.** Every booking, seat query, and hold for a given city routes to that city's shard, exactly the same reasoning as Day 120's short-code-hash sharding, just with a different, business-meaningful shard key.

**Why city, specifically, rather than a hash of the booking ID:** almost every query this system runs is naturally scoped to one city already — a user browsing showtimes is looking at *their* city's theaters, never querying across cities at once. Sharding by city means the overwhelming majority of queries hit exactly one shard, with no cross-shard fan-out required for the common case — a genuinely better fit here than Day 120's hash-based choice, precisely because the two systems' actual query patterns differ (Day 120 never has a meaningful range or grouping key; BookMyShow's queries are naturally grouped by city already).

---

## Part 2 — Seat-Hold Expiry

**The new problem, distinct from what Week 17 already solved:** the LLD version's locking (`SELECT ... FOR UPDATE`, optimistic version-CAS) answers "how do I stop two people from booking the identical seat at the identical instant." It never answered a different question: **how does a seat a user has selected but not yet paid for get held for them temporarily — and automatically released if they abandon the flow?**

**The mechanism — reusing Day 121's Lua-atomicity pattern directly, not inventing a new one:**

```lua
-- KEYS[1] = hold key, e.g. "hold:show:501:seat:A12"
-- ARGV[1] = userId requesting the hold
-- ARGV[2] = hold duration in seconds (e.g. 600 = 10 minutes)
local existing = redis.call('GET', KEYS[1])
if existing then
    return 0   -- already held (by this user or someone else — caller distinguishes)
end
redis.call('SET', KEYS[1], ARGV[1], 'EX', ARGV[2])
return 1       -- hold acquired
```

**Why this needs the same Lua wrapper Day 121 needed, for the identical reason:** a plain `GET`-then-`SET`, as two separate round trips, has exactly the check-then-act race already proven dangerous four times this week — two users could both `GET` and see no existing hold before either `SET`s theirs, both believing they've claimed the seat. Wrapping both calls in one Lua script closes that gap the same way it closed Day 121's rate-limit race: single-threaded execution guarantees nothing else runs between the `GET` and the `SET`.

**Why no separate cleanup logic is needed at all — a genuinely elegant consequence of choosing TTL specifically:** the `EX` argument means Redis itself deletes the key once the hold window elapses. A user who never completes payment simply has their hold vanish on its own; no background job, no scheduled cleanup task, no extra code path for "release an abandoned hold" — the mechanism that grants the hold is the identical mechanism that releases it.

---

## Part 3 — High-Demand Launches: Thundering Herd, Recognized and Pre-Empted

**The scenario, stated precisely:** a hugely anticipated film's booking page opens at a publicly announced time. Thousands of users, refreshing in anticipation, hit that page within the same handful of seconds — and since it's a *brand-new* show, **nothing has been cached for it yet.** Every one of those near-simultaneous requests misses the cache identically and queries the seat-availability data directly — this is Day 123's Thundering Herd, textbook, not a variant.

**The one thing that's different from Day 123's general framing, worth naming explicitly as a genuine extra insight, not a repeat:** Day 123's mitigations (a repopulation mutex, stale-while-revalidate) are both **reactive** — they respond gracefully once a herd has already started. Here, the on-sale moment is **known in advance**, publicly announced. That changes the best answer entirely: **proactively warm the cache** — populate the seat-map cache with the show's data *before* the announced on-sale instant, so the very first wave of requests hits a warm cache rather than triggering a herd at all. Reactive mitigation is what you reach for when you *can't* predict the spike; proactive warming is strictly better whenever you *can* — which a scheduled on-sale time is a clean, concrete example of.

> 💡 **Interview Insight:** naming this distinction — "the mitigation changes because the timing is knowable here" — rather than reflexively repeating Day 123's answer unchanged, is exactly the kind of judgment that separates reciting a fact from actually reasoning about a specific system's specific circumstances.

---

## Part 4 — Estimation, Worked

**Given:** 1,000 shows/day, 200 seats/show, a 2-hour peak window. The plan deliberately withholds a peak-concentration percentage — stating assumptions explicitly, exactly as Day 120 modeled, is part of the exercise itself, not a gap to silently paper over.

- **Ceiling:** $1{,}000 \times 200 = 200{,}000$ seats/day, if every seat sold — a deliberate upper bound for capacity planning.
- **Assume 30% of daily bookings concentrate in the 2-hour peak** (a stated, explicit assumption, not a given fact): $200{,}000 \times 0.30 = 60{,}000$ bookings in the peak window.
- **Peak write rate:** $60{,}000 \div 7{,}200\text{s} \approx 8.3$ bookings/sec sustained.
- **Read amplification is different here than Day 120's, and worth reasoning about freshly rather than reusing 10:1 by reflex:** a booking flow involves many seat-map views per one actual booking — assume roughly 15 reads per booking, a browsing-heavy ratio appropriate to *this* system specifically: $8.3 \times 15 \approx 125$ reads/sec sustained during peak.
- **The Thundering-Herd-relevant number, separately — a single blockbuster's on-sale burst:** if 50,000 people attempt to book within the first 60 seconds a specific show goes live, that's $50{,}000 \div 60 \approx 833$ reads/sec, **for that one show alone**, layered on top of the platform's ~125/sec steady peak baseline — the concrete number behind why proactive cache warming (Part 3) matters specifically at this instant, not as a general platform-wide concern.

---

## Coding Exercise: the Full HLD Diagram

```
Client ──▶ Load Balancer (W10 D72) ──▶ API Gateway + distributed Rate Limiter (Day 121)
                                              │
                          ┌───────────────────┴───────────────────┐
                          ▼                                       ▼
                 Booking Service                          Payment Service
                          │
        ┌─────────────────┼──────────────────────┐
        ▼                 ▼                       ▼
 Seat-Map Cache     Seat Hold (Redis TTL      Booking ID
 (Day 123: Cache-    + Lua atomicity,          (Day 124:
 Aside; Thundering   Day 121's pattern,        Snowflake)
 Herd-aware —        Part 2 above)
 proactively
 warmed, Part 3)
        │                 │
        └────────┬────────┘
                  ▼
        Sharded Booking DB — sharded by CITY (Week 12, Day 78; Part 1 above)
         ┌───────────────┐   ┌───────────────┐
         │ Shard: Mumbai  │   │ Shard: Chennai │  ...
         │ leader+followers│   │ leader+followers│
         └───────┬────────┘   └───────┬────────┘
                 │                     │
     each shard's leader election is Raft/quorum-backed,
     for safe failover if a shard's leader dies (Day 125)
                 │
     WITHIN one shard, the actual seat-commit step still uses
     BookMyShow's original LLD locking — SELECT FOR UPDATE /
     optimistic version-CAS (Week 17, Days 116-117) — that
     mechanism never needed to change; only what surrounds it did
```

**Labeling the callbacks explicitly, since that's the point of this exercise:** the Load Balancer and Gateway are Week 10; the rate limiter riding on the Gateway is Day 121; the cache and its Thundering-Herd awareness are Day 123; the booking ID is Day 124's Snowflake generator; the DB sharding is Week 12's strategy, applied here for the second time this week; each shard's safe leader failover is Day 125's consensus; and — the detail most worth stating out loud, since it's easy to assume everything changed — the actual seat-commit logic *inside* a single city's shard is completely unchanged from Week 17's LLD version. Scaling out didn't rewrite that mechanism; it just added everything *around* it.

---

## Career Block

**Weekly Industry Awareness Ritual:** a scan of recent engineering-blog or industry news from this week's target companies (Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech) — closing the loop on the networking research started Day 121, with anything genuinely relevant worth a line in the application tracker or a talking point for an upcoming conversation.

**Weekly Scorecard — Week 18:**

| Metric | Result |
|---|---|
| HLD systems designed | 7 / 7 planned (URL Shortener, Rate Limiter, Notification, Distributed Cache, ID Generation, Consensus, BookMyShow at Scale) |
| HLD mocks completed | 2 / 2 (Mock #1 — URL Shortener, 5-step narration; Mock #2 — Rate Limiter, defending under "why not X") |
| DSA revision problems | 1 / 1 (LC 40, Combination Sum II — see Discrepancy Notice above) |
| Blog posts published | 1 (Day 120) |
| LinkedIn posts published | 2 (Days 122, 124) |
| Applications live | 7 companies, tracked since Day 118 |

## Daily Deliverable Check

- [ ] Self-check completed and honestly scored (✅/❌), with a plan to retry in a few days if ❌.
- [ ] Can explain why BookMyShow's HLD version needs city-sharding while the LLD version never did.
- [ ] Can walk through the seat-hold mechanism's atomicity and why no separate cleanup logic exists.
- [ ] Can explain why a known on-sale time changes the best Thundering Herd mitigation from Day 123's general case.
- [ ] Full HLD diagram drawn, with every box correctly labeled back to the day (or prior week) that taught it.
- [ ] Weekly scorecard completed; application tracker updated.

---

## Day 126 — Interview Questions

---

**1. Why did BookMyShow's LLD version never need sharding, and why can't the HLD version avoid it?**

*Answer:* The LLD version modeled one theater in one process with one database — small enough for a single instance. At national scale, thousands of theaters and tens of thousands of daily shows exceed what one database instance can hold or serve safely, requiring the database itself to be split — Sharding Strategies, Week 12, Day 78.

---

**2. Why shard by city rather than by a hash of the booking ID?**

*Answer:* Almost every real query is already scoped to one city — a user browsing showtimes only ever looks at their own city's theaters — so city-based sharding routes the overwhelming majority of queries to exactly one shard, with no cross-shard fan-out needed for the common case.

---

**3. What new problem does the seat-hold mechanism solve that Week 17's original locking didn't?**

*Answer:* The original locking (`SELECT FOR UPDATE`, optimistic CAS) prevents two people from booking the same seat at the same instant. Seat-hold answers a different question: how a seat a user has selected but not yet paid for stays reserved temporarily, and is automatically released if they abandon the flow.

---

**4. Why does the seat-hold check-and-set need to be wrapped in Lua, and what established pattern is this reusing?**

*Answer:* A plain GET-then-SET is two round trips with a race window — two users could both see no existing hold before either sets theirs. Wrapping both in a Lua script makes it atomic via Redis's single-threaded execution, the identical pattern Day 121 used for the rate limiter's check-and-increment.

---

**5. Why does an abandoned seat-hold need no separate cleanup code?**

*Answer:* The hold is set with a Redis TTL (`EX`) — Redis itself deletes the key once the window elapses, so the same mechanism that grants the hold is the one that releases it; no background job or scheduled task is required.

---

**6. Why is a blockbuster's midnight on-sale close to a textbook Thundering Herd scenario?**

*Answer:* Thousands of users refresh within the same few seconds the page goes live, and since it's a brand-new show, nothing has been cached yet — every request misses the cache simultaneously and queries seat availability directly at once, exactly Day 123's Thundering Herd pattern.

---

**7. Why does a known, announced on-sale time change the best mitigation compared to Day 123's general framing?**

*Answer:* Day 123's mitigations (a repopulation mutex, stale-while-revalidate) are reactive, for when a spike's timing can't be predicted. A publicly announced on-sale time is knowable in advance, so proactively warming the cache before that instant is strictly better — the herd never gets the chance to start.

---

**8. Walk through the peak booking-rate estimation.**

*Answer:* 1,000 shows × 200 seats = 200,000 seats/day ceiling. Assuming 30% concentrate in the 2-hour peak: 60,000 bookings ÷ 7,200 seconds ≈ 8.3 writes/sec. At an assumed 15 reads per booking (a browsing-heavy ratio, reasoned about fresh rather than reused from Day 120): ≈125 reads/sec sustained.

---

**9. Where does Distributed Consensus (Day 125) show up in this design, even though it isn't drawn as its own labeled component?**

*Answer:* Underneath each city shard — a shard typically runs as a leader with followers, and safe failover if that leader dies depends on quorum-based, Raft-style consensus, the same mechanism underneath etcd and Kubernetes' Control Plane.

---

**10. In the full HLD diagram, what specifically did *not* change from BookMyShow's LLD version?**

*Answer:* The actual seat-commit logic within a single city's shard — `SELECT FOR UPDATE` and optimistic version-CAS from Week 17, Days 116–117 — is unchanged. Scaling out added sharding, caching, holds, IDs, and consensus around that mechanism; it never needed to rewrite the mechanism itself.

---

**11. [Self-Check] How does Combination Sum II (LC 40) differ from Combination Sum (LC 39), specifically?**

*Answer:* Each number can only be used once here (the recursive call advances to `i + 1`, not `i`), and the input can contain duplicate values, requiring a same-level duplicate-skip so the output has no duplicate combinations — neither constraint applies to LC 39.

---

**12. [Self-Check] Why must the array be sorted first, and what two separate purposes does that one sort serve?**

*Answer:* Sorting enables early termination (`break` once `candidates[i] > remaining`, since nothing later in a sorted array can work either) and makes the duplicate-skip check (`candidates[i] == candidates[i-1]`) meaningful, since equal values are guaranteed adjacent only once sorted — one O(n log n) sort buys both.

---

**13. [Self-Check] Why is the duplicate-skip condition `i > start` and not `i > 0`?**

*Answer:* `i > 0` would also incorrectly skip a value that's the first choice at a brand-new recursion level. `i > start` only skips a value that repeats one already tried at the *same* level, while still allowing that same value as the first pick one level deeper.

---

**14. [Self-Check] What's the time and space complexity of the Combination Sum II backtracking solution?**

*Answer:* O(2ⁿ) time worst case (include-or-exclude each element), substantially reduced in practice by sort-enabled pruning and duplicate-skipping; O(n) space for recursion depth plus the in-progress combination.

---

## Week 18 Consolidation

**What actually got built:** all seven planned HLD systems (URL Shortener, Rate Limiter formalized, Notification System, Distributed Cache, Distributed ID Generation, Distributed Consensus, BookMyShow at Scale), both planned HLD mocks, and one DSA revision problem. Planned vs. actual: **7/7 systems, 2/2 mocks, 1/1 revision problem** — no scope was dropped, and consistent with Weeks 16–17's own established precedent for non-DSA phases, **zero extra DSA practice problems were added** this week, since Week 18 isn't built around a DSA pattern needing reinforcement reps.

**Two deliberate sequencing choices made this week, flagged here since both involved reordering material relative to a literal reading of the plan:**
1. Redis's full picture (data structures, Memcached contrast, persistence, pub/sub) was **deferred from Day 121 to Day 123** — Day 121 borrowed only the specific commands its one problem needed, narrowly, the same way Day 120 borrowed `SELECT FOR UPDATE SKIP LOCKED` without a full SQL course first.
2. The formal **"SETNX-based distributed lock"** pattern and **idempotency keys**, both named as Week 19, Day 130 content, were deliberately **not** taught this week even though Day 122's deduplication and Day 126's seat-hold both needed *some* check-and-set mechanism — both were solved instead by reusing Day 121's already-taught Lua-atomicity pattern, keeping Week 19's own new content genuinely new when it arrives.

**Diagnostic — worth a second look before Week 19:**
- The distinction between a *conceptual* mitigation (Day 123's repopulation mutex, deliberately not built end-to-end) and a *built* one (stale-while-revalidate) — make sure this stays a clear distinction, not a blurred memory of "there were two mitigations."
- Raft's three-part decomposition (election, log replication, safety) — check it comes back without prompting, not just recognized when named.
- The "why not X" defense structure from Mock #2 (name the alternative's real merit, then the specific reason the choice still wins) — this is a recurring interview format, not a one-time check; worth deliberately re-using the structure in any future mock, this week's or otherwise.

**What Week 19 assumes, going in:** every mechanism built this week — sharding, distributed rate limiting, deduplication, caching and Consistent Hashing, unique ID generation, and consensus-backed safety — is now a reflexive building block to be **combined on demand**, not recalled one at a time. Week 19 continues directly: fan-out strategies for Instagram's feed (Day 129, the exact harder version of Day 122's Confluence reframe), geospatial indexing for Uber's driver tracking (Day 128, one of the richer Redis data structures named but not built this week), and — closing the loop this week deliberately left open — idempotency keys and SETNX-based distributed locks in full (Day 130), the formal version of the mechanism this week solved twice with a narrower, already-taught substitute.

---

**Next:** Week 19 begins with Day 127. See `00_Curriculum_Map.md` for the updated cumulative index and problem table.
