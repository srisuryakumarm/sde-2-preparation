# Day 58 — Missing Primer: Replication Basics (the prerequisite for CAP)

**Do this first on Day 58 (≈ 30 min), then continue with Day 58's regular plan.**
**Needed by:** Day 59 (CAP theorem — it is about replicated data under a network partition), Day 61 (Replication Models, which assumes this vocabulary), Days 78, 120–133 (HLD).
**Why it is missing:** CAP is taught on Day 59 and replication formally on Day 61; Day 59 uses "replicas" without defining them. This 30-minute note gives the vocabulary first.
**Already in place:** databases and indexes (Days 36–40), Postgres (Day 36), Kafka partitions (Day 48).

---

## 1. Why copy data to several machines
**Availability** (survive a node failure), **read scaling** (many replicas serve reads), **locality** (a copy near the user). Replication = *copies of the same data*; sharding (Day 78) = *splitting different data*. Real systems do both.

## 2. Leader–follower (primary–replica)
Writes go to the **leader**, which records them in a log; **followers** apply the log and serve reads. This is how PostgreSQL streaming replication, MySQL, MongoDB replica sets and Redis replication work.
| Mode | Leader waits for… | Trade-off |
|---|---|---|
| **Synchronous** | the follower's acknowledgement | no data loss on failover; slower writes; a slow follower blocks writes |
| **Asynchronous** | nothing | fast writes; followers lag; the last writes can be **lost** if the leader dies |
| Semi-synchronous | one follower | the common compromise |

## 3. The consequences you must be able to name
- **Replication lag → stale reads:** you write, then read from a follower that hasn't caught up. Guarantees people ask for: **read-your-writes** (you see your own update), **monotonic reads** (time doesn't go backwards).
- **Failover:** detect a dead leader, **promote** a follower. Dangers: losing unreplicated writes, and **split brain** (two nodes both think they are leader).
- **Other topologies:** multi-leader (writes in several regions; conflicts to resolve) and leaderless (Dynamo-style: write to W nodes, read from R nodes, R + W > N — Day 124's file).

## 4. How it leads to CAP
When the network **partitions**, replicas cannot talk to each other. A system must choose: refuse some requests to stay **consistent** (every read sees the latest write), or answer from whatever replica it can reach and stay **available** but possibly stale. That choice is Day 59.

## 5. Exercise (≈ 10 min, paper)
1. Draw one leader and two async followers. Write `x=1`, then `x=2`; show a read from a lagging follower returning `1`.
2. The leader crashes before replicating `x=2`; a follower is promoted — what is `x`, and what happened to the client's acknowledged write?
3. For *orders* and for *product page views*, choose sync or async replication and justify in one line each.

## Checklist
- [ ] Can contrast replication with sharding.
- [ ] Can explain sync vs async replication and what each risks.
- [ ] Can define replication lag, read-your-writes and split brain.
- [ ] Continue with Day 58's regular plan, then Day 59's CAP theory.
