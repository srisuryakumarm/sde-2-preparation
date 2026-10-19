# Day 123 — Missing Primer: Bloom Filters, HyperLogLog, and Count-Min Sketch

**Do this first on Day 123 (≈ 45 min, before the Distributed Cache theory), then continue with Day 123's regular plan.**
**Needed by:** Day 123 (cache penetration), Day 127 (LSM trees use Bloom filters), Day 133 (dedup in object storage), and the web-crawler/"have I seen this?" family of HLD questions.
**Why it is missing:** Bloom filters, HyperLogLog and count-min sketches are named only in passing (Days 36, 127, 132) and never taught.
**Already in place:** hashing and `HashMap` internals (Days 15, 23), bit manipulation (Day 95), Redis basics and TTL (Days 44, 122's file).

---

## 1. Why approximate?
At billions of items an exact set or counter costs gigabytes. These structures trade a **small, controlled error** for **tiny, fixed memory** and O(1) operations.

## 2. Bloom filter — "definitely not here" or "probably here"
A bit array of **m** bits and **k** hash functions. **Add:** set the k bits. **Query:** if *any* of the k bits is 0 → **definitely absent**; if all 1 → **probably present** (false positives possible, **never false negatives**). You cannot delete (a *counting* Bloom filter can).
**Sizing** for **n** items at false-positive rate **p**: `m = −n·ln(p) / (ln 2)²`, `k = (m/n)·ln 2`. Example: 1 M items at 1 % → m ≈ 9.6 M bits ≈ **1.2 MB**, k ≈ 7.
**Uses:** **cache penetration** (ask the Bloom filter before querying the DB for a key that probably doesn't exist), crawlers ("seen this URL?"), username availability, and **LSM-tree databases** (check the filter before reading an SSTable from disk, Day 127).
```java
class BloomFilter {
    private final BitSet bits; private final int m, k;
    BloomFilter(int expected, double fpp) {
        m = (int) Math.ceil(-expected * Math.log(fpp) / (Math.log(2) * Math.log(2)));
        k = Math.max(1, (int) Math.round((double) m / expected * Math.log(2)));
        bits = new BitSet(m);
    }
    private int idx(int h1, int h2, int i) { return Math.floorMod(h1 + i * h2, m); }   // double hashing simulates k hashes
    void add(String s) { int h1 = s.hashCode(), h2 = (h1 >>> 16) | 1; for (int i = 0; i < k; i++) bits.set(idx(h1, h2, i)); }
    boolean mightContain(String s) {
        int h1 = s.hashCode(), h2 = (h1 >>> 16) | 1;
        for (int i = 0; i < k; i++) if (!bits.get(idx(h1, h2, i))) return false;
        return true;
    }
}
```
(`String.hashCode()` is a weak hash for this — real code uses Murmur3, e.g. Guava's `BloomFilter`.)

## 3. HyperLogLog — "how many *distinct* items?"
Estimates the **cardinality** of a huge stream using ~12 KB with ≈ **0.8 % error**, by watching the longest run of leading zeros in hashed values. Redis has it built in: `PFADD visitors u1 u2`, `PFCOUNT visitors`, `PFMERGE`. Uses: unique visitors per day, distinct search queries.

## 4. Count-Min Sketch — "how often did X occur?"
A small 2-D array of counters with one hash per row; **add** increments one counter per row; **estimate** = the **minimum** across rows (it only ever over-estimates). Uses: heavy hitters / top-K trending items in a stream, rate limiting by key.

## 5. Choosing
| Question | Structure | Error |
|---|---|---|
| is X in the set? | Bloom filter | false positives only |
| how many distinct? | HyperLogLog | ± ~1 % |
| how often did X appear? | Count-Min Sketch | over-estimates |
| exact answer needed | a real set/map (and pay the memory) | none |

## 6. Exercise (≈ 20 min)
1. Run the `BloomFilter` above with 100,000 inserted keys at `fpp = 0.01`; test 100,000 *unseen* keys and measure the actual false-positive rate. Try `fpp = 0.001` and compare memory.
2. In `redis-cli`: `PFADD` 10,000 ids (a loop), `PFCOUNT` — how far off? Then `PFMERGE` two sets.
3. Explain in two sentences how a Bloom filter stops a cache-penetration attack on the product-detail endpoint.

## 7. Where Bloom filters live: LSM trees (and how they differ from B-trees)
- **B-tree** engines (PostgreSQL, MySQL/InnoDB) update pages **in place**: fast point reads and range scans, random-I/O writes (plus the write-ahead log).
- **LSM-tree** engines (Cassandra, RocksDB, HBase) write to a log and an in-memory **memtable**; when full it is flushed as an immutable sorted file (**SSTable**), and background **compaction** merges files. A read checks the memtable, then SSTables newest → oldest — **Bloom filters let it skip every SSTable that cannot contain the key**.
- Trade-off: LSM is **write-optimized** (sequential I/O) but pays **read, write and space amplification** from compaction. Rule of thumb: ingestion/time-series/write-heavy → LSM; read-heavy transactional with range queries → B-tree.

## Checklist
- [ ] Can state the Bloom filter's guarantee (no false negatives) and compute its size.
- [ ] Can name two uses each for Bloom, HyperLogLog and Count-Min.
- [ ] Can say why a Bloom filter cannot delete. Continue with Day 123's regular plan.
