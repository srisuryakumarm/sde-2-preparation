# Day 47 — Missing Primer: Run Kafka Locally (KRaft, no ZooKeeper) and Spring Boot 4's Kafka Starter

**Do this first on Day 47 (≈ 45 min), then continue with Day 47's regular plan.**
**Needed by:** Day 48 ("Run Kafka locally via Docker … confirm messages flow with a console producer and consumer before touching application code"), Day 50, Day 69 (Schema Registry), Day 79, Day 80, Day 93.
**Why it is missing:** Day 48's book says "run Kafka locally via Docker" without a command. The Infrastructure Buffer file mentions ZooKeeper — Kafka 4.x runs in KRaft mode only; **there is no ZooKeeper container to start.**
**Already in place:** Docker basics (Day 43), Docker Compose (Day 44).

## 1. Start a single-node Kafka
```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:4.0.0     # or the newest 4.x tag on Docker Hub (apache/kafka)
docker logs kafka | tail -5                                      # wait for "Kafka Server started"
```
It listens on `localhost:9092` for clients on your Mac. If the container is memory-hungry on a small machine, add `-e KAFKA_HEAP_OPTS="-Xmx512M -Xms256M"`.

## 2. Prove it works with the CLI (this is Day 48's "before touching application code")
```bash
docker exec kafka /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic task-events --partitions 3
docker exec kafka /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
docker exec -it kafka /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic task-events
#   type a few lines, Ctrl-C
docker exec -it kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic task-events --from-beginning
docker exec kafka /opt/kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-group
```
Say out loud what you saw: a topic with 3 partitions, messages with offsets, a consumer group's lag.

## 3. Spring Boot 4: the starter you need
Use **`org.springframework.boot:spring-boot-starter-kafka`** (no version). On Boot 4 adding only `org.springframework.kafka:spring-kafka` leaves `spring.kafka.*` properties **ignored** and `@KafkaListener` unconfigured. (Easiest: tick "Spring for Apache Kafka" in a throwaway Initializr project and copy it.) Minimal properties:
```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      auto-offset-reset: earliest
```

## 4. Mind the address
Keep the application on your Mac (`./mvnw spring-boot:run`) for Days 48–50, so `localhost:9092` is correct. If you later put the app and Kafka on the **same Docker/Compose network**, `localhost:9092` stops working for the app — Kafka's advertised listener must then be the service name. Make that change deliberately (Day 134+), not by accident.

## 5. Clean up
`docker stop kafka && docker rm kafka` — the data is ephemeral (no volume), which is fine for learning.

## Checklist
- [ ] `kafka` container running; topic created; console producer → consumer round trip works.
- [ ] Can say why there is no ZooKeeper and which starter Boot 4 needs.
- [ ] Continue with Day 47's regular plan (Testcontainers), then Day 48.
