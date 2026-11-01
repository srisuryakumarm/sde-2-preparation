# Day 66 — Missing Primer: Kafka Internals — Replication, ISR, `acks`, Retention and Compaction, Rebalancing

**Do this on Day 66 (≈ 45 min; Day 66 has no other primer), then continue with Day 66's regular plan.**
**Needed by:** Day 69 (Schema Registry and "at-least-once" assume you know what a *committed* message is), Day 70 (saga events must survive a broker failure), Day 80 (Kafka Streams state is backed by **compacted** topics), Day 93 (DLQ topics and retention), Day 122 (notification fan-out), Days 127 and 131 (chat and job scheduler lean on Kafka), and every HLD follow-up of the form *"what happens when a broker dies?"*, *"how do you guarantee no message loss?"*, *"how many partitions?"*, *"how long do you keep data?"*
**Why it is missing:** Day 48 teaches topics, partitions, producers, consumer groups and offsets; Day 50's primer teaches delivery semantics and idempotent consumers. **Broker-side durability appears nowhere:** the words *ISR*, *`acks=all`*, *`min.insync.replicas`*, *retention* and *compaction* occur in none of the 148 books, and rebalancing is only touched in passing. Without them "Kafka is durable" is a slogan, not an answer.
**Already in place:** partitions, consumer groups, offsets (Days 48–50), delivery semantics and the idempotent producer (Day 50's primer), replication basics — leader/follower, sync vs async, failover (Day 58's primer), CAP (Day 59), a single-node Kafka to experiment on (Day 47's file: container `kafka`, image `apache/kafka:4.0.0`).

---

## 1. The broker view of a topic
A Kafka **cluster** is several **brokers** (servers). Each partition of a topic is a **log** — an append-only sequence of messages, stored as files — and is **replicated** on several brokers.
- **Replication factor (RF):** how many copies of each partition exist. Production default: **3**.
- Each partition has one **leader** and RF−1 **followers**. Producers write to the leader and consumers read from the leader (by default); followers **pull** from the leader to stay in sync — the leader/follower model from Day 58's primer, applied per partition. Leaders are spread across brokers so every broker is leader for *some* partitions.
- A **controller** keeps cluster metadata and elects leaders. Kafka 4.x runs the controller with its own Raft-based quorum (**KRaft**) — **no ZooKeeper** (Day 47's file). Raft itself is Day 125.

## 2. ISR and the high watermark — what "committed" means
- **ISR (in-sync replicas):** the leader plus every follower that has kept up recently (`replica.lag.time.max.ms`, 30 s by default in recent versions). A follower that falls behind is **dropped** from the ISR and re-added when it catches up.
- **High watermark:** the highest offset replicated to **all ISR members**. A message is **committed** only below it, and **consumers can only read committed messages** — so a leader crash can never "un-deliver" something a consumer already saw.

## 3. Producer `acks` — the durability dial
| `acks` | The producer is told "success" when… | Risk |
|---|---|---|
| `0` | it has sent the bytes (no reply) | lost on any failure — only for metrics you can drop |
| `1` | the **leader** has written it | leader dies before followers copy it → **acknowledged message lost** |
| `all` (`-1`) | **all current ISR members** have it | safest; slower by one replication round-trip |

`min.insync.replicas` (broker/topic setting; default 1) is the **floor**: with `acks=all`, if the ISR is smaller than this the write **fails** (`NotEnoughReplicas`) instead of "succeeding" with fewer copies.
**The durable recipe — memorise it:** **RF = 3, `min.insync.replicas = 2`, `acks = all`** (and the idempotent producer, Day 50's primer). Modern Kafka clients already default to `acks=all` and idempotence on (since Kafka 3.0; older clients defaulted to `acks=1`). It tolerates **one** broker failure with no loss of acknowledged data, and when two are gone it **refuses writes rather than lie** — a CP choice (Day 59).

## 4. What happens when a broker dies (RF = 3, min ISR = 2, `acks=all`)
| Scenario | Writes | Reads | Data |
|---|---|---|---|
| a **follower's** broker dies | ✅ (ISR = 2 ≥ 2) | ✅ | none lost |
| the **leader's** broker dies | brief pause (seconds) while the controller elects a new leader **from the ISR**; producers retry | ✅ after election | none lost — the new leader has every committed message |
| **two** brokers die | ❌ rejected (ISR = 1 < 2) | ✅ committed data is still readable | none lost; **availability sacrificed** |
| same cluster but producers use `acks=1` and the leader dies right after acknowledging | ✅ | ✅ | **the acknowledged message can be gone** |
`unclean.leader.election.enable` (default **false**): if *every* ISR member is dead, the partition stays **unavailable** until one returns. Setting it `true` lets an out-of-sync replica become leader — the partition is available again, but messages it never received are **lost** (AP over CP).

## 5. Ordering, keys and how many partitions
- **Order is guaranteed only within one partition.** A keyed message goes to `hash(key) % partitionCount` (murmur2), so all events for one key (one order, one user) stay ordered. Messages with no key are spread across partitions in batches, with no ordering.
- **Adding partitions later changes the key→partition mapping** — existing keys move and per-key ordering across the change breaks. Over-provision up front.
- **Parallelism ceiling:** a consumer group can use at most as many consumers as there are partitions (extra consumers sit idle). More partitions = more throughput and parallelism, but more files, more replication traffic and slower failover. Rule of thumb: partitions ≥ the consumers you will need at peak, then multiply for growth headroom.

## 6. Retention vs compaction — "how long do we keep it?"
A partition log is split into **segment files** (1 GiB by default). Retention deletes **whole old segments**, never single messages.
- **`cleanup.policy=delete` (default):** keep for `retention.ms` (default **7 days**) and/or `retention.bytes` (default unlimited). Kafka is **not** a queue that forgets after delivery — consumers track their own offsets, and the same data can be **re-read** (replay) until retention removes it. A consumer that is down longer than the retention period finds its offset gone (`auto.offset.reset` decides what happens next).
- **`cleanup.policy=compact`:** keep **at least the latest value per key**, discarding older values for the same key in the background (not instantly; the active segment is never compacted). A message with a key and a **null value** is a *tombstone* that deletes the key. Used for **changelogs and state** — the `__consumer_offsets` topic itself, and Kafka Streams state backups (Day 80).

## 7. Consumer groups: rebalance and its consequences
A **rebalance** reassigns partitions among the group's consumers when a consumer joins, leaves, **stops heart-beating** (`session.timeout.ms`) or takes too long between polls (`max.poll.interval.ms`, 5 min). During an eager rebalance consumption pauses; newer assignors (cooperative sticky) move only the partitions that have to move. **Consequence:** offsets that were processed but not yet committed are re-delivered after a rebalance → **duplicates** → idempotent consumers (Day 50's primer). Consumer **lag** (latest offset − committed offset) is the number to alert on (Day 76–77).

## 8. Why it is fast (extends Day 48's "sequential I/O")
Sequential appends + the **OS page cache** (reads usually come from memory) + **batching and compression** on the producer (`linger.ms`, `batch.size`, `compression.type`) + **zero-copy** sends from page cache to socket. Throughput is mostly limited by network and disk, not the broker's CPU.

## 9. Exercise (≈ 20 min; Day 47's single-node container must be running: `docker start kafka` or Day 47's `docker run`)
1. **Describe a topic:** `docker exec kafka /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic task-events` — find `PartitionCount`, `ReplicationFactor`, and per partition **`Leader`, `Replicas`, `Isr`** (all the same single broker id here).
2. **Try the impossible:** create a topic with `--replication-factor 3` — it must **fail** because only one broker exists. Explain why in one sentence (a replica must live on a *different* broker).
3. **Compaction config:** `kafka-topics.sh … --create --topic latest-state --partitions 1 --config cleanup.policy=compact`, then `/opt/kafka/bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name latest-state --describe`.
4. **Paper (10 min):** for `RF=3, min.insync=2` fill the table in §4 *from memory*, then re-answer it for `acks=1`, and for `min.insync.replicas=1`.
5. Write two sentences you would say in an HLD interview when asked *"how does your Kafka topic avoid losing orders?"*

## Checklist
- [ ] Can explain leader/follower per partition, ISR, high watermark and what "committed" means.
- [ ] Can recite the durable recipe (RF 3, min ISR 2, `acks=all`) and say what happens with 1 and with 2 brokers down.
- [ ] Can explain unclean leader election as a CP-vs-AP switch.
- [ ] Can say why order is per partition, why adding partitions breaks key ordering, and the consumer-count ceiling.
- [ ] Can contrast `delete` retention with `compact` and name a use for each; can say why a rebalance causes duplicates. Continue with Day 66's regular plan.
