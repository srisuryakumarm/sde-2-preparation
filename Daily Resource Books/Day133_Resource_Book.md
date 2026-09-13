# Day 133 (Sunday) — HLD #16: Google Drive / Object Storage, HLD Mock #5, and HLD Phase Complete

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 132 Resource Book](Day132_Resource_Book.md)
**Next ▶:** [Day 134 Resource Book](Day134_Resource_Book.md)
**Companion to:** Day 133 of `Week_19_Revised.md`

---

## Recap

Sixteen systems, five mocks, two weeks — today closes the HLD phase entirely with the system this whole arc has been quietly building toward since Day 127's deferred media-sharing note: Google Drive. Object storage is also the natural place to land the phase's last new data-integrity idea, content-addressable deduplication, and its last new consistency question, sync conflicts — a fitting close, since "how much inconsistency can this specific system actually tolerate" has been the thread running underneath Days 127 through 132 without ever being named as the single question it actually is.

---

## Learning Objectives

By the end of today, without notes:

1. Recall all sixteen HLD systems from these two weeks from memory, and name each one's single most distinctive concept.
2. Explain, with a worked example, why fixed-size chunking silently breaks deduplication after a small edit, and why content-defined chunking doesn't.
3. Justify choosing conflict-copy over Last-Write-Wins for this specific system, not as a default preference but as a reasoned trade-off.
4. Defend any of the sixteen systems from this phase under open-ended, self-selected pressure — including one you did not personally feel most confident about.

---

## Concept Dependency Map

```
Day 127: media sharing named as in-scope, deliberately deferred to "the object-
         storage layer" — today is that deferral's payoff
Day 40: hashing / HashMap fundamentals (Week 1–2) — the lookup underneath
        content-addressable dedup's chunk index
Day 61: synchronous vs. asynchronous replication trade-offs — the same kind of
        question sync-conflict handling asks, one layer up
        │
        ▼
Today — Google Drive / Object Storage
  ├─ Chunking: fixed-size vs. content-defined (NEW)
  ├─▶ Content-Addressable Deduplication (NEW — needs: chunking, above,
  │     and hashing fundamentals, Week 1–2)
  ├─ Sync Conflicts: LWW vs. conflict-copy (NEW)
  └─ Multipart Upload to S3 (NEW — a transport-level idea, distinguished
        precisely from chunking's storage-level purpose)

  Separately, today: the two-week Self-Check (all 16 systems); HLD Mock #5,
  candidate's choice; and the full Week 19 Consolidation.
```

---

## Self-Check (15 min) — All Sixteen Systems, From Memory First

*Before reading the table below, actually attempt the list cold. The table is the answer key, not the exercise.*

| # | System | Day | Its single most distinctive concept |
|---|---|---|---|
| 1 | URL Shortener | 120 | The 5-step HLD framework itself, taught live for the first time |
| 2 | Rate Limiter | 121 | Four algorithms (Token Bucket, Leaky Bucket, Fixed/Sliding Window) + Redis/Lua atomicity |
| 3 | Notification System | 122 | Per-channel fan-out + best-effort dedup, deliberately short of full idempotency |
| 4 | Distributed Cache | 123 | Redis's full data-structure picture + Thundering Herd |
| 5 | ID Generation | 124 | Snowflake ID — timestamp + worker ID + sequence, bit-packed |
| 6 | Consensus | 125 | Quorum / Raft / split-brain |
| 7 | BookMyShow at Scale | 126 | City-sharding + Redis TTL/Lua seat-hold + proactive cache warming |
| 8 | WhatsApp / Chat | 127 | Persistent WebSockets + the connection-management routing layer |
| 9 | Uber Location Tracking | 128 | Geospatial indexing — geohashing, quadtrees, Redis Geo |
| 10 | Netflix / Video Streaming | 129 | Adaptive bitrate streaming + CDN (Cache-Aside, relocated to the edge) |
| 11 | Instagram Feed | 129 | Hybrid fan-out — the celebrity problem, finally solved |
| 12 | Payment System | 130 | Idempotency keys + distributed locks + the double-entry ledger |
| 13 | Distributed Job Scheduler | 131 | At-least-once execution + SQS Visibility Timeout |
| 14 | Leaderboard | 132 | The Skip List, finally built from scratch |
| 15 | Search Autocomplete | 132 | A Trie redeployed as read-only, replicated, and asynchronously rebuilt |
| 16 | Google Drive / Object Storage | 133 | Content-addressable deduplication |

---

## Part 1 — Chunking: Fixed-Size vs. Content-Defined

### Fixed-size chunking, and its hidden flaw — proven, not asserted

Split a file into equal-sized pieces — say, 4 bytes for a small worked example. Original content `ABCDEFGHIJKL` chunks as `[ABCD][EFGH][IJKL]`.

Now insert a single character `X` at the very start: `XABCDEFGHIJKL`. Re-chunking at fixed 4-byte boundaries gives `[XABC][DEFG][HIJK][L]`.

Compare: `[ABCD]` vs. `[XABC]` — different, correctly, since content genuinely shifted. But `[EFGH]` vs. `[DEFG]` — **also different**, even though the actual bytes `E`, `F`, `G`, `H` are all still present in the file, completely unchanged. They've simply been re-grouped across a different boundary. **Every chunk from the insertion point onward fails to match its old hash**, despite almost none of the real content having changed — a single-byte edit at the front of a large file silently destroys deduplication for the entire rest of it.

### Content-defined chunking — why it avoids this

Instead of fixed positions, boundaries are chosen using a rolling hash over a small sliding window of bytes, declaring a boundary whenever the window's hash satisfies some local condition (e.g., its lowest bits are all zero). Because the condition depends only on a **local** window of content, an edit far from a given position doesn't change whether that position still qualifies as a boundary — once the edited region has scrolled out of the rolling window, boundary decisions downstream return to exactly where they were before the edit, and the chunks in that region hash identically to before, correctly deduplicating despite the earlier edit.

> 🔑 **Key Takeaway:** the difference isn't "smarter chunking" in the abstract — it's specifically that content-defined boundaries are a *local* function of content, while fixed-size boundaries are a *global* function of position, and a global function is exactly what a local edit disrupts everywhere downstream of it.

---

## Part 2 — Content-Addressable Deduplication

**Definition:** identify and store data by a hash of its own content, so identical content anywhere in the system — across files, users, or versions of the same document — is recognized and stored exactly once.

**Mechanism:** hash each chunk (SHA-256 or similar); check a chunk index for that hash; if present, add a reference to the existing chunk from this file's manifest — no new storage write at all; if absent, write the chunk and add it to the index.

**Why it matters economically, not just conceptually:** many users store the same popular file (a common installer, a widely-shared PDF); a single user's successive edited versions of one document share most of their content; unrelated files often share common byte patterns (shared boilerplate, common headers). Content-addressable storage captures all three cases automatically, with no special-casing for "this looks like a duplicate."

> 💡 **Interview Insight, worth stating unprompted:** content addressing gives integrity verification for free. Since a chunk's identifier *is* derived from its content, re-hashing a retrieved chunk and comparing it to its claimed hash detects corruption or tampering — a genuine bonus property, not a separate mechanism bolted on.

> ⚠️ **The hash-collision question, answered precisely, not dismissed or over-engineered:** two genuinely different chunks producing the same SHA-256 hash is a real theoretical possibility. Its probability at any realistic storage scale is small enough to be an accepted, standard trade-off in production systems — the correct interview answer names the risk honestly and states why it's accepted, rather than either waving it away as impossible or proposing elaborate collision-handling machinery the actual probability doesn't justify.

---

## Part 3 — Sync Conflicts

Two devices edit the same file while disconnected from each other; both come back online and try to sync. Something has to resolve the resulting divergence.

**Last-Write-Wins (LWW):** compare timestamps, keep the later edit, discard the earlier one entirely.
- Trade-off: simple, but can silently **destroy real user work** — if Device A edits at 2:00pm and Device B edits at 2:01pm, both offline from each other, LWW keeps B and A's edit vanishes with no warning to the user that anything was lost.
- A sharper failure mode worth naming: LWW's correctness depends on genuinely comparable timestamps across devices. A device with a forward-skewed clock can make a genuinely *older* edit appear later, incorrectly winning and discarding real, newer work — a clock problem masquerading as a data problem.

**Conflict-copy:** detect that two versions diverged from a common ancestor; instead of silently choosing, keep **both** — the "winning" version stays at the original path, the other is saved separately (`"document (conflicted copy from <device>, <date>).ext"`) for the user to manually reconcile.
- Trade-off: never silently loses data, at the cost of pushing reconciliation work onto the user — frequent conflicts can leave a confusing pile of conflicted-copy files.

**The actual design decision for this system, stated directly rather than left open:** Google Drive-style file sync uses conflict-copy, because silently discarding a user's document edit is a much worse failure than asking them to occasionally resolve a conflict by hand. This is the right call *for this system specifically* — a setting-sync feature with low-stakes, frequently-overwritten values might reasonably prefer LWW's simplicity instead. Stating which trade-off fits *this* system, and why, is the actual interview answer — not simply listing both options.

---

## Part 4 — Multipart Upload to S3

**Mechanism:** initiate an upload session (receive an Upload ID); upload each part independently, tagged by part number, potentially in parallel and out of order, each returning an ETag; call "Complete Multipart Upload" with the full list of parts, and the service assembles them into the final object.

**Why it matters:** for a multi-gigabyte file, a single-shot upload failing at 95% would otherwise mean restarting from byte zero. Multipart upload bounds retry cost to a single part's size and enables genuine parallel throughput.

> ⚠️ **Precisely distinguished from chunking, not the same idea wearing two names:** multipart upload is a **transport-level** concept — getting bytes to the server reliably and in parallel. Content-defined chunking (Part 1) is a **storage-level** concept — enabling deduplication. They're related in spirit (both split a blob into pieces) and often used together in the same real pipeline, but they solve different problems and can use entirely different boundary sizes and strategies. Conflating them would blur two genuinely separate design decisions into one.

---

## Part 5 — Estimation, HLD, Bottlenecks (Steps 2, 3, 5)

**Estimation:** assume 1 billion users storing an average of 5GB each — 5 exabytes of raw content before any deduplication. This is precisely the number that makes deduplication an economic necessity rather than a nice-to-have: even a modest dedup ratio across shared and repeated content at this scale represents an enormous, direct storage-cost reduction — the estimation step is, again, the actual justification for the design choice, not decoration on top of it.

**HLD:** client chunks the file locally (content-defined) before upload → checks each chunk's hash against the Chunk Index → uploads only genuinely new chunks via multipart upload to blob storage → saves the file's manifest (its ordered list of chunk hashes) to a manifest store, keyed by file ID.

**Bottlenecks:** the Chunk Index itself, at extreme scale, needs to be a fast, horizontally-scalable hash lookup — shardable by hash prefix, conceptually the same locality-preserving instinct behind consistent hashing and range-based sharding already established (Days 60, 78), applied here to a lookup index rather than a data store. Sync-conflict handling itself doesn't have a pure engineering fix — conflict-copy is a product decision about acceptable user friction, not a bottleneck with a purely technical resolution.

---

## Coding Exercise — Chunking-and-Hashing Dedup Check

```
function uploadFile(fileId, fileBytes):
    chunks = contentDefinedChunk(fileBytes)     // rolling-hash boundaries, not fixed-size
    manifest = []

    for chunk in chunks:
        hash = SHA256(chunk)
        if chunkIndex.exists(hash):
            manifest.append(hash)               // reference existing chunk — no new write
        else:
            blobStore.write(hash, chunk)         // via multipart upload if chunk is itself large
            chunkIndex.add(hash)
            manifest.append(hash)

    manifestStore.save(fileId, manifest)         // ordered chunk-hash list reconstructs the file
```

---

## Part 6 — HLD Mock #5: Candidate's Choice

**Format:** the phase's fifth and final mock, subject chosen by you — the same "candidate's choice" precedent the LLD phase's own fifth and final mock set on Day 119.

**How to actually choose, honestly:** not the system that would go smoothest — the one you feel *least* confident defending. A genuine self-check, not a rhetorical one: which system's Bottleneck step did you answer least convincingly the first time you wrote it? If an interviewer picked a system for you at random from all sixteen, which pick would make your stomach drop a little? That system is the correct choice for today, precisely because it's the one still worth the pressure.

**Worth remembering: the choice spans all sixteen systems across both weeks, not just today's or this week's.** An illustrative example, deliberately drawn from **Week 18** to make that scope concrete:

> **[Interviewer]:** "Let's do Consensus. Walk me through why a system needs it at all."
>
> **[You]:** *(recap Day 125 — Quorum, Raft, split-brain, from memory)*
>
> **[Interviewer, pushing]:** "Your quorum-based approach assumes a majority of nodes can always be reached. What actually happens during a genuine network partition where neither side has a majority?"
>
> **[You]:** "Neither side can commit new writes — a minority partition, by definition, can't reach quorum, so it correctly refuses to accept writes rather than risk two sides diverging with conflicting data. That's the deliberate trade-off: availability is sacrificed on the minority side specifically to preserve consistency, rather than allowing both sides to keep accepting writes and reconciling later."

If Consensus (or any other system) is genuinely the one that would make your stomach drop, this is the shape of the pressure to expect — and the shape of the honest, mechanism-grounded answer to give back.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform`. **Task:** update `docs/architecture.md` — the consolidation pass promised back on Day 127, bringing the module-boundary draft fully current with everything actually built since.

**Definition of done:** every module boundary documented on Day 127 reflects the codebase's *actual* current state — Payment now documents idempotency-key checking and the ledger (Day 130); a note added about the reconciliation job (Day 131) and its relationship to Order and Payment; the LocationService stub (Day 128) and LeaderboardService (Day 132) both documented under their owning modules. An honest, current snapshot, not a historical record of what Day 127 originally guessed.

---

## Career Block Guide

**Weekly Industry Awareness Ritual:** the usual scan of industry news relevant to the 7 target companies — Rippling, Google, Databricks, Stripe, Uber, Atlassian, and Walmart Global Tech — anything that changed this week worth knowing walking into a conversation.

**Weekly Scorecard — HLD Phase Complete:** sixteen HLD systems named and defensible, five HLD mock interviews run and debriefed — a genuine multiple of the original plan's single HLD mock, matching the LLD phase's own five exactly. Ten design mocks total across both phases. Applications remain live across all seven target companies after two weeks of steady, consistent effort — the systems-design half of interview preparation is now genuinely complete, with execution (mocks, applications, behavioral readiness) the remaining work ahead.

---

## Day 133 — Interview Questions

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

## Daily Deliverable Check

- [ ] All sixteen HLD systems named from memory, each with its one distinctive concept, before checking the answer table.
- [ ] Can explain, with the worked example, why fixed-size chunking breaks dedup and content-defined chunking doesn't.
- [ ] `docs/architecture.md` fully current with every module built since Day 127.
- [ ] HLD Mock #5 completed and debriefed — genuinely on the system you were least confident about, not the most comfortable one.
- [ ] Weekly Industry Awareness Ritual and Weekly Scorecard completed.

---

## Week 19 Consolidation

### What actually got built

Nine HLD systems (WhatsApp, Uber, Netflix, Instagram, Payment, Distributed Job Scheduler, Leaderboard, Search Autocomplete, Google Drive), completing the HLD phase at sixteen systems total across Weeks 18–19. Three HLD mocks (#3, #4, #5 — Days 128, 131, 133), bringing the phase to five mocks total, matching the LLD phase's five exactly and quadrupling the original plan's single HLD mock. One deferred thread fully closed: idempotency keys and SETNX-based distributed locks, formalized in full on Day 130 after four earlier informal appearances. One genuinely new data structure built from scratch: the Skip List (Day 132), after three uses of Sorted Sets as a black box (Days 123, 128, 131). Two cold-revision problems solved outside the required plan (LC 721, LC 1143), continuing the spaced-repetition practice Weeks 16–18 established.

### Planned vs. actual

This week has no LeetCode-problem plan to measure against — the relevant unit is systems and mocks, not problems. Planned: 9 systems, 3 mocks. Delivered: 9 systems, 3 mocks. No system was deferred or compressed; the plan's own day-by-day pacing held without adjustment, unlike some earlier DSA weeks that required reordering to keep prerequisites clean.

### Diagnostic — if any of these aren't solid, revisit before Week 20

- Can you derive Skip List's O(log n) from the mechanism (levels + expected hops per level), not recite it as a memorized fact?
- Can you trace the unsafe distributed-lock release bug (Day 130) step by step, unprompted, including *why* the token check has to be atomic with the delete?
- Can you do the celebrity-problem arithmetic (Day 129) from scratch — not recall the conclusion, but redo the division that shows why 50,000 writes/sec still isn't fast enough?
- Can you explain why a wide-column store's LSM-tree write path is cheaper than a B-tree's, in terms of *when* each pays its rebalancing cost, not just "NoSQL scales better"?

### What Week 20 assumes

Week 20 shifts the platform from designed to *deployed* — Kubernetes, Helm, distributed tracing, chaos engineering, and a service mesh, closing with a full bottleneck-and-scale analysis applied back across all sixteen HLD systems from these two weeks. That closing exercise assumes every one of the sixteen systems genuinely received a real estimation pass and a real bottleneck analysis, not merely a design sketch — which is precisely why this week's resource books treated Estimation and Bottlenecks as mandatory steps on every system, even the ones covered lightly on a two-system day, rather than optional depth to skip under time pressure.

---

## What Next Week Assumes You Already Know Cold

Day 134 assumes the full HLD phase — all sixteen systems, defensible cold, not just recognizable — is genuinely closed, since Week 20's own capstone-hardening work will refer back to specific systems by name (which one needed a distributed lock, which one needed sharding) rather than re-describing them. It also assumes the habit built across these two weeks — estimation before design, bottlenecks stated honestly rather than glossed over, trade-offs named against a specific alternative rather than asserted in the abstract — now travels with you into infrastructure work that isn't about designing a system from scratch, but about operating one that already exists.
