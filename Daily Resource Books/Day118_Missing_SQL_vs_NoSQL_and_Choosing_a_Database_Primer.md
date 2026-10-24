# Day 118 — Missing Primer: SQL vs NoSQL — The Database Families and How to Choose

**Do this first on Day 118 (≈ 60 min, before the HLD framework), then continue with Day 118's regular plan.**
**Needed by:** Days 120–133 (every HLD answer has a "which database and why?" step), Day 127 (chat: wide-column), Day 130 (payments: relational), Day 132–133 (leaderboards, drive).
**Why it is missing:** key-value, document, wide-column and graph stores, and the SQL-vs-NoSQL decision, are never taught as a topic — they are named only in passing in the CAP day (59) and the chat design (127). Day 73's AWS file lists DynamoDB but not how to model for it.
**Already in place:** SQL, indexes and ACID (Days 36–40), isolation (Day 40's file), Redis (Day 44), CAP and replication (Days 58–61), sharding (Day 78), quorums (Day 124's file), OLAP (Day 80's file), search (Day 131's file).

---

## 1. The families
| Family | Data model | Examples | Strengths | Weak at |
|---|---|---|---|---|
| **Relational** | tables, rows, foreign keys, joins | PostgreSQL, MySQL | ACID, flexible queries, constraints | scaling writes across machines |
| **Key-value** | `key → value` | Redis, DynamoDB (basic use) | extreme speed/scale, simple | no joins, query only by key |
| **Document** | JSON-like documents with nesting | MongoDB, Firestore | flexible schema, one read for a whole aggregate | cross-document joins/transactions are weaker |
| **Wide-column** | partition key + clustering columns, sparse rows | Cassandra, HBase, Bigtable | huge write throughput, linear scale, multi-region | ad-hoc queries; you model *per query* |
| **Graph** | nodes and edges | Neo4j | relationship traversals (friends of friends) | bulk analytics |
| **Time-series** | append-heavy by timestamp | Timescale, InfluxDB, Prometheus | ingest, downsampling | general OLTP |
| Search / OLAP | inverted index / columns | Elasticsearch / warehouses | full-text; scans (Days 131, 80) | being a source of truth |

## 2. Modelling is different
- **Relational:** model the *entities*, **normalize**, and let queries be flexible (joins, indexes).
- **NoSQL:** model around the **access patterns** — list the queries first, then design keys and **denormalize** (duplicate data) so each query hits one partition.
- **Documents:** *embed* data that is read together and owned together (an order with its items — your Day 49 aggregate); *reference* when data is shared or grows unbounded.
- **Wide-column / DynamoDB:** the **partition key** decides distribution — pick a high-cardinality key to avoid a **hot partition**; the **sort/clustering key** orders rows inside a partition. Example: chat messages with `partition = conversationId`, `clustering = timestamp DESC` — "latest 50 messages" is one sequential read.

## 3. Consistency and scaling
Relational databases scale **up** and with read replicas, and shard by hand (Day 78). Many NoSQL systems partition and replicate **by design** and let you **tune consistency** per request (R + W > N, Day 124's file) — at the price of weaker cross-record transactions (some now offer limited transactions; know the limits). "NoSQL = no transactions" is outdated; "NoSQL = you design around the limits" is accurate.

## 4. How to choose (say the requirement, then the store)
| If the main need is… | Reach for… |
|---|---|
| money, orders, inventory — **transactions and constraints** | relational |
| sessions, counters, rate limits, leaderboards — **speed on simple access** | Redis / key-value |
| flexible, nested catalogue data read as a unit | document store (or relational + JSON columns) |
| **massive write throughput**, time-ordered data, multi-region (chat, events, feeds) | wide-column |
| "friends of friends", recommendations over relationships | graph |
| full-text, faceting | search engine (derived index) |
| analytics over history | warehouse / columnar |
**Default:** *start with PostgreSQL*; move a specific workload to another store only when a measured requirement (scale, latency, access pattern) demands it. Every extra store costs sync logic (CDC/outbox, Day 69's file), operations, and expertise ("polyglot persistence").

## 5. Typical interview mapping
URL shortener → key-value or relational (simple lookups, huge read volume) · chat messages → wide-column · news-feed timelines → Redis lists/sorted sets + a durable store · payments → relational · product catalogue → relational or document, plus a search index · analytics → warehouse.

## 6. Exercise (≈ 25 min, paper)
1. Choose a store, with one-sentence justification, for: user accounts · order ledger · shopping-cart session · product search · chat history · "trending now" counters · clickstream analytics.
2. Model *orders* twice: relational (Day 49's tables) and DynamoDB-style — list the access patterns (by id, by customer + date) and the partition/sort key for each table or index.
3. Model chat messages for a wide-column store; say what the hot-partition risk is and how to bucket (e.g. add a day bucket to the key).

## Checklist
- [ ] Can name the five main families with an example and a weakness each.
- [ ] Can explain "model around access patterns" and what a partition key does.
- [ ] Can say when you would still choose PostgreSQL.
- [ ] Can map six common HLD designs to a store. Continue with Day 118's regular plan.
