# Day 80 — Missing Primer: OLTP vs OLAP, Columnar Storage, and Batch vs Stream Processing

**Do this first on Day 80 (≈ 45 min, before Kafka Streams), then continue with Day 80's regular plan.**
**Needed by:** Day 80 (Kafka Streams — windows, state, event time), Day 131–132 (job scheduler, leaderboards), HLD answers about analytics, metrics and reporting pipelines.
**Why it is missing:** OLAP, data warehouses, columnar formats and the batch-vs-stream distinction appear nowhere in the repo; Kafka Streams is introduced without the stream-processing vocabulary.
**Already in place:** Postgres/SQL (Days 36–40), Kafka (Days 48–50, 69), event-driven design and CQRS (Day 79's file), sharding and replication (Days 61, 78).

---

## 1. OLTP vs OLAP
| | **OLTP** (transactions) | **OLAP** (analytics) |
|---|---|---|
| Workload | many small reads/writes ("place an order") | few huge scans and aggregations ("revenue by region last quarter") |
| Layout | **row-oriented**, normalized | **column-oriented**, denormalized (star schema: fact + dimension tables) |
| Example | PostgreSQL, MySQL | Snowflake, BigQuery, Redshift, ClickHouse |
| Priority | latency, consistency | throughput over huge data |
Running heavy analytics on your OLTP database slows customers down — so data is **copied out** to an analytical store.

## 2. Why columnar wins for analytics
A query like `SUM(amount) GROUP BY day` needs only two columns. A **column store** reads just those, compresses each column well (values of one type repeat: dictionary and run-length encoding) and processes them in vectorized batches. Open file formats: **Parquet**, **ORC**. Rows are *worse* at point lookups and single-row updates.

## 3. Warehouse, lake, lakehouse
**Warehouse:** curated, schema-on-write, SQL. **Data lake:** raw files (often on S3) of any shape, schema-on-read. **Lakehouse:** table formats (Delta Lake, Iceberg, Hudi) that add transactions and schema to files on object storage.

## 4. Moving the data
**ETL** (transform before loading) vs **ELT** (load raw, transform in the warehouse). **CDC** — capture changes from the database log (e.g. Debezium) into **Kafka**, then sink to the warehouse; this is also how a search index or cache is kept in sync.

## 5. Batch vs stream
| | Batch | Stream |
|---|---|---|
| Processes | a bounded dataset, periodically | an unbounded flow, continuously |
| Latency | minutes–hours | milliseconds–seconds |
| Engines | Spark | **Flink**, **Kafka Streams** (a library inside your service), Spark Structured Streaming |
**Lambda architecture:** a batch layer (accurate, slow) plus a speed layer (approximate, fast). **Kappa:** only streaming, replaying the log for reprocessing.

## 6. The ideas Kafka Streams assumes (Day 80)
- **Event time** (when it happened) vs **processing time** (when you saw it). Networks reorder and delay, so they differ.
- **Windows:** *tumbling* (fixed, non-overlapping), *sliding* (overlapping), *session* (gap-separated bursts).
- **Watermarks / grace period:** how long to wait for late events before closing a window.
- **State:** counts and joins need state, kept in local stores backed by a changelog topic for recovery.
- A `KTable` is a **changelog turned into a table** — the read model from Day 79.

## 7. Exercise (≈ 20 min, paper)
1. For "orders per product per minute on a live dashboard" and "monthly revenue report for finance", choose store, engine and freshness; say what is batch and what is stream.
2. Draw the path: `order-service` DB → outbox/CDC → Kafka → (a) Kafka Streams per-minute counts, (b) warehouse via a sink. Mark where event time vs processing time matters.
3. A 1-minute tumbling window closes at 12:01:00; an event stamped 12:00:55 arrives at 12:01:10 — what decides whether it is counted?

## Checklist
- [ ] Can contrast OLTP and OLAP, and row vs column storage, with a reason.
- [ ] Can explain ETL vs ELT and where CDC fits.
- [ ] Can define event time, window types and watermarks.
- [ ] Continue with Day 80's regular plan.
