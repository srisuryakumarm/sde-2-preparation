# Day 42 — Missing Primer: Thread Pools in Depth, `CompletableFuture`, Virtual Threads, and `ThreadLocal`

**Do this first on Day 42 (Sunday; ≈ 90 min), then continue with Day 42's regular plan.**
**Needed by:** Day 64 (Resilience4j time limiter and bulkhead), Day 131 (`@Scheduled` jobs), Day 138 (bounded blocking queue), Day 137 (log/trace context) — and Java SDE-2 interviews, which ask all four.
**Why it is missing:** the Day 37 Concurrency primer (Part 6) shows a fixed thread pool and `Future` only. `CompletableFuture`, virtual threads, pool tuning and `ThreadLocal`/MDC appear nowhere in the repo.
**Already in place:** threads, locks, `ConcurrentHashMap` (Days 29, 37–39), lambdas (Day 30), the Day 37 primer's `ExecutorService` basics.

---

## 1. `ThreadPoolExecutor` — the real constructor
```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
        2, 4,                                   // corePoolSize, maximumPoolSize
        30, TimeUnit.SECONDS,                   // keep-alive for threads above core
        new ArrayBlockingQueue<>(2),            // BOUNDED work queue
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.CallerRunsPolicy());   // what to do when full
```
**Submission order:** (1) fewer than *core* threads → start a thread; (2) else put the task in the **queue**; (3) queue full and fewer than *max* → start an extra thread; (4) else **reject**.
**Rejection policies:** `AbortPolicy` (default: throws `RejectedExecutionException`), `CallerRunsPolicy` (the submitting thread runs it — natural back-pressure), `DiscardPolicy`, `DiscardOldestPolicy`.
**Traps:** `Executors.newFixedThreadPool(n)` uses an **unbounded** queue (memory blow-up under load); `newCachedThreadPool()` allows an **unbounded** number of threads. Prefer an explicit bounded pool.
**Sizing:** CPU-bound ≈ number of cores (+1). I/O-bound ≈ cores × (1 + wait time / compute time) — then *measure*. Use separate pools per dependency so one slow service cannot starve the rest (that is the **bulkhead** idea, Day 64).

## 2. `CompletableFuture` — composing async work
`Future.get()` blocks and cannot be chained. `CompletableFuture` can:
```java
CompletableFuture<Product> p = CompletableFuture.supplyAsync(() -> productClient.get(id), ioPool);
CompletableFuture<Stock>   s = CompletableFuture.supplyAsync(() -> stockClient.get(id),   ioPool);

View view = p.thenCombine(s, View::new)                       // both done → combine
             .orTimeout(500, TimeUnit.MILLISECONDS)           // fail with TimeoutException if too slow (Java 9+)
             .exceptionally(ex -> View.degraded(id))          // fallback value on failure
             .join();                                         // wait; throws unchecked CompletionException
```
| Method | Meaning |
|---|---|
| `supplyAsync` / `runAsync` | start work (**pass your own executor**) |
| `thenApply` | transform the result (like `map`) |
| `thenCompose` | chain a step that itself returns a `CompletableFuture` (like `flatMap`) |
| `thenCombine` | join two independent futures |
| `allOf` / `anyOf` | wait for all / the first |
| `exceptionally` / `handle` / `whenComplete` | error handling / observe the outcome |

Without an executor argument, work runs on `ForkJoinPool.commonPool()` — **do not block it with I/O**. `join()` vs `get()`: same wait, but `join()` throws unchecked, `get()` throws checked exceptions.
Parallel streams (`list.parallelStream()`) use the same common pool: avoid them for blocking calls or tiny workloads.

## 3. Virtual threads (Java 21, JEP 444)
A **virtual thread** is a cheap thread scheduled by the JVM onto a small pool of *carrier* (platform) threads. When it blocks on I/O it is **parked** and the carrier runs another one — so you can write simple blocking code and still serve tens of thousands of concurrent waits.
```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {       // one new virtual thread per task — never pool them
    for (int i = 0; i < 10_000; i++) exec.submit(() -> { Thread.sleep(1000); return null; });
}                                                                     // close() waits for all tasks
```
- Best for **I/O-bound** thread-per-request work; no benefit for CPU-bound work.
- **Pinning:** on Java 21, blocking *inside* a `synchronized` block pins the carrier thread (use `ReentrantLock` instead); Java 24 removed the `synchronized` limitation.
- Spring Boot: `spring.threads.virtual.enabled=true` runs request handling on virtual threads.
- vs. reactive (Day 65): virtual threads give scalability **without** changing to the `Mono`/`Flux` style. The Gateway stays reactive; ordinary services may not need to be.

## 4. `ThreadLocal` and MDC
A `ThreadLocal` gives each thread its own copy of a value; Spring uses it for the security context and transactions, and SLF4J's **MDC** uses it so every log line carries a `requestId`.
```java
MDC.put("requestId", id);          // pattern: %X{requestId}
try { ... } finally { MDC.clear(); }
```
**Pitfalls:** in a *pool*, the thread outlives the request — always `remove()`/`clear()` or the value leaks into the next task. A value is **not** propagated to other threads you spawn (copy it explicitly, or use a `TaskDecorator` in Spring). Day 137's tracing relies on exactly this context propagation.

## 5. Exercises (45 min, `java-fundamentals`)
1. Build the pool in §1 (2 core, 4 max, queue 2). Submit 10 tasks that sleep 500 ms and print the thread name; see which are rejected. Switch to `CallerRunsPolicy` and watch the main thread run tasks.
2. Three simulated 300 ms calls: run sequentially, then with `supplyAsync` + `thenCombine`; compare total time. Add `orTimeout(200 ms)` and an `exceptionally` fallback.
3. Start 10,000 tasks that sleep 1 s on `newFixedThreadPool(200)` and on `newVirtualThreadPerTaskExecutor()`; compare wall-clock time.
4. A `ThreadLocal<String>` set in a pooled task but never removed: show it leaking into a later task, then fix it.

## 6. Spring's `@Async`
`@EnableAsync` on a configuration class and `@Async` on a bean method runs that method on a **Spring-managed executor** and returns `void` or a `CompletableFuture`. Define your own bounded `ThreadPoolTaskExecutor` (core/max/queue) instead of the default; remember the **self-invocation** rule (Day 41) — calling an `@Async` method from the same class runs it synchronously — and that the security/MDC context is **not** copied to the new thread unless you add a `TaskDecorator`. (`@Scheduled` jobs, Day 131, use a scheduler thread pool the same way.)

## Checklist
- [ ] Can recite the four-step submission order and name two rejection policies.
- [ ] Can say why `newFixedThreadPool` can run out of memory.
- [ ] Can chain, combine and time-limit a `CompletableFuture`, and say which pool it runs on.
- [ ] Can say when virtual threads help, what pinning is, and how they differ from reactive.
- [ ] Can explain MDC and the `ThreadLocal` leak. Continue with Day 42's regular plan.
