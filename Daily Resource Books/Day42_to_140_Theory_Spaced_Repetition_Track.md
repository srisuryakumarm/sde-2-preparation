# Gap Fill: Day 42 to 140 — Theory Spaced Repetition Track

**Gap addressed (#5):** DSA patterns get real spaced repetition throughout the plan — weekly "solve cold" self-checks, and revision problems threaded through the LLD/HLD weeks. Java, concurrency, Spring, and systems *theory* doesn't get this treatment anywhere. Each topic is taught once, in a single 2-hour Theory Block, and never scheduled to come back. By Week 21, an interviewer asking about a Week 3 topic cold is a real risk this specifically closes.

**How this slots in:** a recurring 15–20 minute "Theory Cold-Recall" addition to the existing Sunday Career Block, starting Day 42 (once enough backlog exists to make rotation meaningful) and running through Day 140 (Capstone close) — Week 21 already has its own dedicated final-review self-checks (Day 146, Day 148) that pick up where this leaves off. **This adds ~15–20 minutes to 14 already-full Sundays; that trade-off is flagged explicitly below, not assumed.**

Scope: this is specifically the Java/Spring/systems-theory backlog. DSA pattern recall is already handled elsewhere in the plan and isn't duplicated here.

---

## The mechanism

Each Sunday, before or alongside the existing Weekly Industry Awareness Ritual:

1. **Blind recall (5 min):** write down just the topic name(s) for that Sunday, nothing else. Then, from memory, explain each one out loud for up to 90 seconds — no notes, no looking anything up.
2. **Check against source (5 min):** open the relevant Week_XX_Revised.md file and compare the recalled explanation against what was actually taught. Note anything missed or gotten wrong.
3. **Log the gap, if any (5–10 min):** if a topic came back shaky, that's the signal to spend a few extra minutes right then re-reading the original explanation — not to panic, just to patch it before it compounds into a real blind spot by Week 21.

This is deliberately lightweight. The point isn't deep re-study every week; it's making sure nothing goes more than roughly 5–6 weeks without at least a 90-second gut-check.

## Design choice, flagged explicitly

Two ways to run this were considered: strict Anki-style increasing intervals per topic (mathematically optimal, but genuinely impractical to hand-schedule across 90+ days by hand), versus a **recency-biased round-robin** — each Sunday reviews that week's own newest theory plus a rotating callback to older material, cycling through the full backlog over time. The table below uses the second approach. It's less precise than true spaced repetition but far more usable as a plain weekly checklist, which matters more here than mathematical optimality.

An alternative, if 15–20 extra minutes on Sunday isn't workable: swap this into one non-Sunday day's "LinkedIn engagement" slot instead of adding it on top — that slot already exists most weekdays and this is arguably higher-leverage than an extra 20 minutes of scrolling.

---

## The rotation table

| Sunday | Callback (older material) | This week's own material |
|---|---|---|
| **Day 42** | `==` vs. `.equals()` (Day 2); the four OOP pillars (Day 5) | ConcurrentHashMap internals (Day 39) |
| **Day 49** | SOLID principles (Day 6); JVM Memory Model — stack vs. heap, pass-by-value (Day 9) | Docker layer caching (Day 43); Kafka fundamentals (Day 48) |
| **Day 56** | Integer cache trap (Day 10); Collections internals — ArrayList/LinkedList/ArrayDeque (Day 12) | Kafka consumer groups & offset committing (Day 50); Spring Cloud Config (Day 51) |
| **Day 63** | HashMap internals — buckets, treeification, equals/hashCode contract (Day 15); Generics & type erasure (Day 16) | CAP theorem (Day 59); Consistent hashing (Day 60) |
| **Day 70** | Comparable vs. Comparator (Day 17); Checked vs. unchecked exceptions (Day 18) | Circuit breakers / Resilience4j concept (Day 64); JWT vs. OAuth2 (Day 67) |
| **Day 77** | Records & sealed classes (Day 28); Thread lifecycle, Runnable vs. Thread (Day 29) | TCP vs. UDP, DNS resolution (Day 71); AWS VPC / subnets / IAM basics (Day 74) |
| **Day 84** | REST principles & HTTP verbs/status codes (Day 34); Spring Data JPA / ORM concept (Day 36) | Sharding strategies — range vs. hash, the celebrity problem (Day 78); 2PC vs. Saga (Day 79) |
| **Day 98** *(Day 91 skipped — leave week 2's own close)* | ReentrantLock / ReadWriteLock, wait-notify/Condition (Days 37–38); SQL joins & indexes (Day 40) | Service discovery / Eureka concept (Day 95); K8s ConfigMaps vs. Secrets — encoding vs. encryption (Day 96) |
| **Day 105** | Why mock vs. real dependency in a test — Mockito/TestContainers philosophy (Days 46–47); Docker Compose service networking (Day 44) | Kubernetes HPA (Day 99); GoF Creational patterns — Singleton, Factory, Builder (Day 100) |
| **Day 112** | Spring AOP (Day 62); Replication models — single-leader/multi-leader/leaderless (Day 61) | Structural + Behavioral GoF patterns (Day 106–107); the Law of Demeter (Day 108) |
| **Day 119** | Rate limiting — Token Bucket (Day 68); Saga choreography vs. orchestration (Day 79, second pass) | Chain of Responsibility (Day 113); Pessimistic vs. optimistic locking (Day 117) |
| **Day 126** | Consistent hashing (Day 60, second pass); AWS IAM — roles vs. users, least privilege (Day 75) | Distributed ID generation — Snowflake's bit layout (Day 124); Distributed consensus & quorum (Day 125) |
| **Day 133** | Two-Phase Commit vs. Saga (Day 79, second pass) | Fan-out strategies — push vs. pull vs. hybrid (Day 129); Idempotency keys (Day 130) |
| **Day 140** | — | **Integration pass:** from the full list above, pick the 3 topics that felt shakiest across every prior Sunday and explain each cold, unprompted, for a full 2 minutes each — not a new topic, a genuine gap-check before Week 21's own final reviews take over. |

---

## Why the early weeks get priority

The specific risk named in the gap this file addresses is a Week 3 topic going cold by Week 21. The table above deliberately front-loads the Week 1–6 backlog into the *first six rotation slots* (Days 42, 49, 56, 63, 70, 77) rather than spreading it evenly across the whole plan — by Day 77, every theory topic from Weeks 1 through 6 has had at least one explicit callback, well before the risk described in the gap has time to materialize.

## Daily Deliverable (applies to each Sunday in the table)
- [ ] Blind recall attempted for every listed topic before checking the source file.
- [ ] Any shaky recall patched with a short re-read the same day, not deferred.
- [ ] By Day 140: every topic in the full table has been explicitly recalled at least once, and the 3 personally-weakest have had a second pass.
