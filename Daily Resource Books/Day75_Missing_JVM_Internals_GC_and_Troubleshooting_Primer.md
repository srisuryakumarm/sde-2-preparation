# Day 75 — Missing Primer: JVM Internals — Memory, Garbage Collection, Class Loading, JIT, and Troubleshooting

**Do this first on Day 75 (≈ 90 min), then continue with Day 75's regular plan.**
**Needed by:** Day 76 (Prometheus/Micrometer JVM metrics: heap, GC pauses, threads), Day 134 (load tests: heap sizing), Day 138 (chaos/OOM), and Java SDE-2 interviews (GC, class loading, "how do you debug a memory leak?").
**Why it is missing:** Day 9 covers stack vs heap; garbage-collection algorithms, class loading, the JIT and JVM tooling are only mentioned in passing (Days 8, 9, 11, 18, 130, 134) and never taught.
**Already in place:** JVM memory, primitives and strings (Days 9–11), threads and locks (Days 29, 37–39), Docker/Kubernetes memory limits (Days 43, 95).

---

## 1. Where memory lives
| Area | Holds | Failure |
|---|---|---|
| **Heap** — young (eden + 2 survivor spaces) and old | objects | `OutOfMemoryError: Java heap space` |
| **Metaspace** (native memory) | class metadata | `OutOfMemoryError: Metaspace` (often a class-loader leak) |
| **Thread stacks** | frames per thread | `StackOverflowError`; `unable to create native thread` |
| Code cache, direct buffers | JIT code; NIO buffers | `Direct buffer memory` |
A container's memory limit must cover **all** of these, not just `-Xmx`.

## 2. Garbage collection
An object is garbage when no **GC root** (stack variable, static field, …) can reach it. The **generational hypothesis** — most objects die young — drives the design: new objects go to eden; a **minor GC** copies survivors; long-lived objects are promoted to old; collecting old is rarer and costlier. A **stop-the-world** pause freezes application threads.
| Collector | Character |
|---|---|
| Serial / Parallel | simple / max throughput, longer pauses |
| **G1** (default since Java 9) | region-based; targets a pause goal (`-XX:MaxGCPauseMillis`) |
| **ZGC** | concurrent, pauses well under a millisecond even for huge heaps; generational mode available in Java 21 (`-XX:+UseZGC -XX:+ZGenerational`) |
Trade-off triangle: **throughput vs pause latency vs memory footprint**. Containers: the JVM reads cgroup limits; prefer `-XX:MaxRAMPercentage=75` over a hard-coded `-Xmx`.

## 3. Class loading
Load → link (verify, prepare, resolve) → initialize. Loaders: **bootstrap → platform → application**, with **parent-first delegation** (ask the parent before loading yourself). A class is unloaded only when its loader is garbage — so frameworks that create loaders (redeploys, Spring Boot DevTools) can leak Metaspace.

## 4. JIT
The JVM starts by interpreting, then compiles hot methods: **tiered compilation** (C1 quick → C2 optimizing), with inlining and escape analysis. Consequence: **warm-up** — the first seconds are slower, which is why naive micro-benchmarks lie (use JMH; Day 39's benchmark carries this caveat).

## 5. The memory model in one rule set
**Happens-before** guarantees visibility: an unlock happens-before the next lock of the same monitor; a `volatile` write happens-before later reads of it; `Thread.start()` happens-before the thread's actions; a thread's actions happen-before `join()` returns; `final` fields are safely published after construction. Without a happens-before edge, another thread may see stale or reordered values (Day 37 primer, Part 5).

## 6. Troubleshooting toolkit
| Symptom | First move |
|---|---|
| high CPU | find the hot thread (`top -H` / Activity Monitor), then `jcmd <pid> Thread.print` and match the native id (hex) |
| hang / deadlock | `jcmd <pid> Thread.print` — look for **"Found one Java-level deadlock"** |
| memory growth | `jcmd <pid> GC.class_histogram`; heap dump `jcmd <pid> GC.heap_dump file.hprof`; analyze in VisualVM or **Eclipse MAT** (dominator tree) |
| OOM in production | start with `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp` |
| GC pauses | `-Xlog:gc*` (unified logging, Java 9+); watch pause times and frequency |
| production profiling | **Java Flight Recorder**: `jcmd <pid> JFR.start duration=60s filename=rec.jfr` |
Micrometer (Day 76) exposes `jvm.memory.used`, `jvm.gc.pause`, `jvm.threads.live` — the same story as dashboards.

## 7. Exercise (≈ 45 min, `todo-api`)
1. Run with `-Xmx256m -Xlog:gc*:file=gc.log`; add a temporary endpoint that allocates in a loop; read the GC log: minor GCs, promotion, pause times.
2. Add a deliberate leak (a `static List<byte[]>` that grows). Run with `-XX:+HeapDumpOnOutOfMemoryError`, let it die, open the dump in VisualVM or MAT and find the leaking list.
3. Take a thread dump with `jcmd`; find the main thread and a Tomcat worker. Create a two-lock deadlock and find it in the dump.
4. Write one paragraph: G1 vs ZGC — when would you choose each?

## Checklist
- [ ] Can draw heap (young/old), metaspace and stacks, and name an OOM for each.
- [ ] Can explain generational GC and the throughput/latency trade-off; can compare G1 and ZGC.
- [ ] Can describe parent-first class loading and how class-loader leaks happen.
- [ ] Can state three happens-before rules; can name the commands for a thread dump and a heap dump.
- [ ] Continue with Day 75's regular plan.
