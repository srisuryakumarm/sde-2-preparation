# Day 119 — Missing Primer: Back-of-the-Envelope Estimation for System Design

**Do this first on Day 119 (Sunday; ≈ 75 min), then continue with Day 119's regular plan.**
**Needed by:** Day 120 (HLD #1 — "perform back-of-envelope estimation live"), every HLD day through Day 133, and the HLD mocks.
**Why it is missing:** estimation is a step in the HLD framework (Days 118/120) but nothing teaches the **numbers** or the **method**; "back-of-the-envelope", "capacity estimation" and "QPS" appear only on Days 118, 126, 140.
**Already in place:** the 5-step HLD framework (Day 118), percentiles and availability (Day 76's file), caching and sharding ideas (Days 78, 123).

---

## 1. Why estimate
It tells you **which architecture you need** — one Postgres or sharding? one cache node or a cluster? — and shows the interviewer you reason with numbers. Aim for the right **order of magnitude**, state assumptions, and round aggressively.

## 2. Numbers to memorize
| Quantity | Value |
|---|---|
| seconds per day | 86,400 ≈ **10⁵** (use 10⁵ for mental math) |
| seconds per month / year | ≈ 2.5 × 10⁶ / 3 × 10⁷ |
| 2¹⁰ / 2²⁰ / 2³⁰ / 2⁴⁰ / 2⁵⁰ | 1 KB / 1 MB / 1 GB / 1 TB / 1 PB |
| characters, small ints | 1–4 bytes; a UUID 16 bytes; a typical row/JSON record 0.5–2 KB |
**Latency, order of magnitude (approximate):** memory reference ~100 ns · read 1 MB from memory ~0.1 ms · SSD random read ~0.1 ms · read 1 MB from SSD ~1 ms · round trip inside a data centre ~0.5 ms · HDD seek ~10 ms · cross-continent round trip ~100–150 ms. **Memory ≫ SSD ≫ disk ≫ network** — hence caches.
**Rough capacities (state them as assumptions):** an app server ~1k–10k simple requests/s · one relational primary ~1k–10k indexed queries/s · one Redis node ~100k ops/s · a 1 Gbps link ~125 MB/s.

## 3. The method
1. **Clarify scale:** users, daily active users (DAU), actions per user per day, read:write ratio.
2. **QPS** = DAU × actions per day ÷ 86,400. **Peak** = average × 2–5.
3. **Storage** = records per day × bytes per record × retention × replication factor.
4. **Bandwidth** = QPS × payload size (in and out).
5. **Cache** ≈ the hot 20 % of daily data (80/20 rule).
6. **Servers** = peak QPS ÷ capacity per server (+ headroom). Decide: single node, replicas, shards?
7. **Sanity-check** against the numbers in §2, and say what you ignored.

## 4. Worked examples
**URL shortener** — 100 M new URLs/month, reads 100× writes.
Writes ≈ 10⁸ ÷ 2.6 × 10⁶ ≈ **40/s**; reads ≈ **4,000/s**; peak ≈ **12,000/s** → one primary plus replicas and a cache; no sharding needed for throughput. Storage: 100 M × 12 months × 5 years = 6 × 10⁹ URLs × 500 B = **3 TB** → shard or use a scalable store later. Key space: 62⁶ ≈ 5.7 × 10¹⁰ > 6 × 10⁹, so 6–7 base62 characters. Cache: reads/day ≈ 4,000 × 10⁵ ≈ 3.5 × 10⁸; hot 20 % × 500 B ≈ **35 GB** → one large Redis or a small cluster.
**Chat** — 50 M DAU × 40 messages/day = 2 × 10⁹/day ≈ **23,000 msg/s** average, ≈ 70,000 at peak. Storage: 100 B × 2 × 10⁹ = **200 GB/day ≈ 73 TB/year** (× 3 with replication). Connections: assume 10 % online = 5 M WebSockets ÷ ~100k per server ≈ **50 gateway servers**.
**Platform orders** — 10 M DAU, 5 % order daily = 500k orders/day ≈ **6/s** (flash-sale peak ×20 ≈ 120/s). Storage: 500k × 2 KB = 1 GB/day ≈ **365 GB/year** — a single Postgres is fine. Events: 5 per order × 1 KB ≈ 2.5 GB/day through Kafka — trivial.

## 5. Practice (answers in parentheses — compute first!)
1. 300 M DAU feed: each posts 0.5/day and reads 100 items/day. Writes/s? Reads/s? (≈ 1.7k/s writes, ≈ 350k/s reads → fan-out and caching matter.)
2. A video site receives 500 hours of video per minute; assume 1 GB per hour raw. Storage per day? (≈ 720 TB/day.)
3. A rate limiter keeps one 16-byte counter per user for 100 M users. Memory? (≈ 1.6 GB — fits one Redis node.)
4. Your platform: 20 M DAU, 3 product views per session, 1 session/day, 100:1 view:order — requests/s, and is one Postgres enough for the *catalogue reads*? (≈ 700/s average, ≈ 2–3k at peak: a cache helps, sharding doesn't.)

## 6. Habits
Write assumptions on the board first; keep units in every line; round (86,400 → 10⁵); say "I'll assume…" instead of asking for everything; finish with *what the numbers imply for the design*.

## Checklist
- [ ] Can recall the constants in §2 without looking.
- [ ] Can compute QPS, peak, storage and bandwidth from DAU in 3 minutes.
- [ ] Can turn the numbers into a decision (cache? replicas? shards?).
- [ ] Practice 1–4 done. Continue with Day 119's regular plan.
