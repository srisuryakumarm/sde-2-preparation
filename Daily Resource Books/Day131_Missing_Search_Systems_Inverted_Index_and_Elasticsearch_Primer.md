# Day 131 — Missing Primer: Search Systems — Inverted Index, Relevance, and Elasticsearch

**Do this first on Day 131 (≈ 60 min), then continue with Day 131's regular plan.**
**Needed by:** Day 132 (HLD #15: Search Autocomplete — typeahead vs full search), any "design search for products/messages/files" question, and the product-search feature of the e-commerce platform.
**Why it is missing:** inverted indexes, relevance scoring and Elasticsearch/Lucene appear nowhere in the repo; Day 59's Trie covers prefix lookup only.
**Already in place:** Trie (Day 59), SQL indexes (Day 40), CDC/outbox and Kafka (Days 69, 80), sharding and scatter-gather ideas (Day 78), pagination (Day 78's file).

---

## 1. Why not `WHERE name LIKE '%phone%'`
A leading wildcard cannot use a B-tree index (full table scan), there is **no ranking** ("best match first"), and no language handling (case, plurals, typos). Search engines solve all three.

## 2. The inverted index
A normal ("forward") index maps document → words. An **inverted index** maps **term → list of documents** containing it (a *posting list*, often with term frequencies and positions).
```
doc1: "red running shoes"      red   → [1, 3]
doc2: "blue running jacket"    running → [1, 2, 3]
doc3: "red leather jacket"     shoes → [1]    jacket → [2, 3]    blue → [2]    leather → [3]
```
- **Indexing:** an **analyzer** tokenizes text, lowercases, removes stop words, and *stems* (`running` → `run`). The **same analysis runs on the query**.
- **Querying:** look up each term's posting list; **AND** = intersect the sorted lists, **OR** = union; then **score** and return the top K.

## 3. Relevance
**TF-IDF** intuition: a term matters more if it is frequent *in this document* (TF) and rare *across all documents* (IDF). **BM25** — the default in Lucene/Elasticsearch — refines it with saturating term frequency and document-length normalization. You can then boost fields (title ≫ description), popularity, recency.

## 4. Lucene and Elasticsearch in five facts
1. **Lucene** is the library; **Elasticsearch** (and the Apache-licensed fork **OpenSearch**) is the distributed server around it.
2. An **index** is split into **shards** (each a Lucene index) with **replicas** for availability and read throughput.
3. Lucene writes immutable **segments**; new documents become searchable on a **refresh** (default 1 s → *near-real-time*); background **merges** combine segments; deletes are tombstones.
4. **Mappings** matter: `text` fields are analyzed (full-text); `keyword` fields are exact (filters, sorting, aggregations).
5. **Queries vs filters:** a `match` query scores; a filter (`term`, `range`) only includes/excludes and is cacheable. Aggregations give facets ("brand: 120, price buckets").
A search request is **scatter-gather**: sent to every shard, each returns its top K, the coordinator merges. Deep `from + size` pagination is expensive — use `search_after` (the keyset idea from Day 78's file).

## 5. Search is a *secondary* index
The database stays the **source of truth**; the search index is a derived copy. Keep it in sync with **CDC or the outbox → Kafka → indexer** (Days 69, 80) — **not** a dual write from the request (the dual-write problem). Expect **eventual consistency** (a product edit appears seconds later). Rebuild from the source with a **zero-downtime reindex**: build a new index, switch an **alias**.

## 6. Typeahead (Day 132) vs search
| | Typeahead / autocomplete | Full search |
|---|---|---|
| Matches | **prefixes** of queries or titles | terms anywhere, ranked |
| Structures | Trie (in memory, Day 59), edge n-grams or a completion suggester (FST) in Elasticsearch | inverted index |
| Ranking | by query popularity / recency | BM25 + boosts |
| Latency target | < 50–100 ms; cache the top prefixes; client-side debounce | few hundred ms |

## 7. Exercise (≈ 30 min, `java-fundamentals` + paper)
1. Build an inverted index for five short documents: `Map<String, SortedSet<Integer>>`, lowercase + split on non-letters. Implement `and(a, b)` (intersect) and `or(a, b)`.
2. Add a crude stemmer (strip a trailing `s`/`ing`) and show a query for "shoe" finding "shoes".
3. Paper: design "search products" for the platform — source of truth, indexer pipeline, index fields (`text` vs `keyword`), query path, and how a price change reaches the index.
(You do not need to run Elasticsearch locally — it competes for memory with Kafka and Postgres.)

## Checklist
- [ ] Can explain an inverted index and how AND/OR queries use it.
- [ ] Can explain TF-IDF/BM25 in two sentences and why analyzers run on queries too.
- [ ] Can say why search is a derived index and how it stays in sync.
- [ ] Can contrast typeahead with full search. Continue with Day 131's regular plan.
