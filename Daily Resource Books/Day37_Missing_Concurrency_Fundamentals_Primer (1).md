# Concurrency Fundamentals Primer: The Missing Piece Before Day 37

**Where this fits:** insert this as its own theory block, right before Day 37 (Explicit Locks / `ReentrantLock`). Day 29 teaches thread creation and the thread lifecycle, but stops one step short of the actual problem concurrency control exists to solve — it never shows a race condition happening, and never explains `synchronized`. Day 37 then opens with *"`ReentrantLock` does what `synchronized` does with more control"* — a sentence that only makes sense if you already know what `synchronized` does. This primer is that missing step.

**Why this exists:** every concurrency day from here through the end of the plan (37, 38, 39, 100, 111, 117, 138) either uses a tool that replaces something more basic you were never taught, or asks you to write genuinely correct concurrent code and prove it under load. Without seeing a race condition actually happen once, in front of you, all of this is memorized syntax instead of an understood problem.

**What this primer deliberately does *not* cover:** the formal Java Memory Model specification, `ExecutorService` tuning/scheduling trivia beyond the basics, and reactive/asynchronous programming (which never appears anywhere in this plan). `wait()`/`notify()`/`Condition` usage patterns stay Day 38's job, and `ConcurrentHashMap`'s internal bucket structure stays Day 39's — this primer's job is narrower: give you the actual problem (race conditions), the actual primitive (`synchronized`), and the two ideas (atomicity, visibility) that every one of those later days quietly assumes you already have.

---

## Part 1: The Problem — What a Race Condition Actually Is

Before any tool for fixing concurrency bugs makes sense, you need to watch one happen.

```java
public class UnsafeCounter {
    private int count = 0;

    public void increment() {
        count++; // this looks like one step. It is not.
    }

    public int getCount() {
        return count;
    }
}
```

```java
public class RaceConditionDemo {
    public static void main(String[] args) throws InterruptedException {
        UnsafeCounter counter = new UnsafeCounter();
        int numThreads = 100;
        int incrementsPerThread = 1000;

        Thread[] threads = new Thread[numThreads];
        for (int i = 0; i < numThreads; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < incrementsPerThread; j++) {
                    counter.increment();
                }
            });
            threads[i].start();
        }
        for (Thread t : threads) {
            t.join(); // wait for every thread to finish before checking the result
        }

        System.out.println("Expected: " + (numThreads * incrementsPerThread));
        System.out.println("Actual:   " + counter.getCount());
    }
}
```

Run this, and the actual count comes out *lower* than expected — reliably, not as some rare fluke. That gap is a **race condition** made visible.

**Why `count++` isn't one step.** At the machine level, it's actually three separate operations:
1. Read the current value of `count` from memory.
2. Add 1 to that value.
3. Write the new value back to `count`.

If Thread A reads `count = 5`, and before A finishes writing `6` back, Thread B *also* reads `count = 5` — both threads compute `6` and both write `6`. Two increments happened, but the count only went up by one. An update was silently lost. This is exactly the "corrupt shared state" that Day 29 named but never showed you.

A **critical section** is the piece of code where this kind of interleaving can cause damage — here, the three-step read-modify-write of `count`. A **race condition** is any bug where the correctness of the result depends on the unpredictable timing of two or more threads.

---

## Part 2: The Fix — Mutual Exclusion and the `synchronized` Keyword

The fix is **mutual exclusion**: make sure only one thread can be inside a critical section at a time. In Java, the original, built-in tool for this is the `synchronized` keyword.

**Every object in Java has an associated intrinsic lock (its "monitor"), whether you ever use it or not.** A `synchronized` method or block acquires the lock on a specific object before running, and releases it automatically when execution leaves that method or block — including if an exception is thrown. While one thread holds an object's lock, any other thread trying to enter a `synchronized` method or block on *that same object* simply waits until the lock is free.

```java
public class SafeCounter {
    private int count = 0;

    public synchronized void increment() { // acquires the lock on "this" before running
        count++;
    } // lock released automatically here, even on an exception

    public synchronized int getCount() {
        return count;
    }
}
```

Swap `UnsafeCounter` for `SafeCounter` in the demo above, and `Actual` always equals `Expected`, every run.

You can also lock on a specific object explicitly, rather than the whole method:

```java
private final Object lock = new Object();
private int count = 0;

public void increment() {
    synchronized (lock) {
        count++;
    }
}
```

**Now Day 37 will make sense.** `ReentrantLock` does exactly this same job — mutual exclusion around a critical section — but as an explicit object you call `.lock()`/`.unlock()` on, instead of an implicit keyword. That gets you things `synchronized` can't offer: `tryLock()` (attempt to acquire without blocking forever), timeouts, and fairness policies (first-come-first-served ordering among waiting threads):

```java
private final ReentrantLock lock = new ReentrantLock();
private int count = 0;

public void increment() {
    lock.lock();
    try {
        count++;
    } finally {
        lock.unlock(); // MUST be in finally — unlike synchronized, a Lock does not
                        // release itself automatically. An exception between lock()
                        // and unlock() without this would leave the lock held forever.
    }
}
```

That `try`/`finally` isn't a style preference — it's the one real, common bug `ReentrantLock` introduces that `synchronized` structurally can't.

---

## Part 3: Atomicity, and Why `count++` Isn't Atomic

An **atomic operation** is one that appears instantaneous to every other thread — from the outside, it either hasn't happened yet, or it's fully done. There's no in-between state another thread can ever observe.

Simple field reads and writes of most types (`int`, `boolean`, object references) are atomic in Java on their own. `count++` is *not* atomic, because — as Part 1 showed — it's actually three separate operations, and another thread can observe (and interfere with) the state in between them. *(A genuinely obscure but real fact worth knowing: `long` and `double` reads/writes aren't even guaranteed atomic on their own without `volatile`, since a 64-bit value can be written as two separate 32-bit operations on some JVMs.)*

`java.util.concurrent.atomic` provides types that make compound operations like increment genuinely atomic, without needing an explicit lock at all:

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounter {
    private final AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet(); // one atomic operation — no lock needed
    }

    public int getCount() {
        return count.get();
    }
}
```

This also fixes the demo, with no `synchronized` or `Lock` anywhere. How it manages that without ever blocking a thread is Part 4.

---

## Part 4: Compare-And-Swap (CAS) — Atomicity Without Locking

`incrementAndGet()` achieves atomicity through a mechanism called **Compare-And-Swap (CAS)**, a single instruction most modern CPUs support directly in hardware. Conceptually, it works like this:

1. Read the current value.
2. Compute the new value you want to write (current + 1).
3. In one atomic hardware step: *"if the value in memory is still exactly what I read in step 1, swap it for the new value. If something else already changed it, don't swap — report failure instead."*

If the swap fails (another thread got there first), the operation just **retries** from step 1 — it doesn't block, and no thread ever sits waiting for a lock to free up. Under contention, this is often faster than locking, precisely because nobody's ever just sitting idle.

**This is the sentence in Day 39 that now makes sense:** *"Java 8+ locks individual bucket nodes with CAS operations and fine-grained `synchronized`, far less contention than locking the whole map."* `ConcurrentHashMap` uses CAS for simple, low-conflict operations (like checking whether a bucket is empty before inserting), and only falls back to a real per-bucket `synchronized` lock when there's an actual conflict to resolve — which is exactly why it beats wrapping a plain `HashMap` in one giant lock.

---

## Part 5: Visibility — the Other Half of the Problem, and `volatile`

Atomicity (Parts 3–4) solves the *interleaving* problem. There's a second, genuinely different problem: **visibility**.

Modern CPUs cache values per-core for speed. A write by Thread A can sit in that core's cache without being flushed to main memory — meaning Thread B might simply never observe Thread A's update, for an unpredictable amount of time, even with no interleaving bug at all. This isn't about *timing* of an operation racing another; it's about one thread's change never becoming *visible* to another.

`volatile` on a field tells the JVM: never cache this field per-thread — always read and write it directly, and guarantee that any write is immediately visible to every thread that reads it afterward.

```java
private volatile boolean running = true; // a classic use case: a flag one thread
                                          // sets to stop a loop running on another thread

public void stop() {
    running = false;
}

public void run() {
    while (running) { // without volatile, this loop is not guaranteed to ever see
                       // the write from stop() — it could spin forever
        // do work
    }
}
```

**The distinction that matters, precisely:** `volatile` guarantees *visibility*, not *atomicity*. `volatile int count; count++;` is **still broken** — it's still three separate operations, just now each individual read and write is guaranteed visible. `volatile` is the right tool for a single flag like `running` above; it is not a substitute for `synchronized`/`Lock`/`Atomic*` on any compound, multi-step operation.

**This is also what Day 100's Double-Checked Locking exercise actually hinges on:**

```java
public class Singleton {
    private static volatile Singleton instance; // volatile matters here, specifically

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {                  // first check, no lock — fast path
            synchronized (Singleton.class) {
                if (instance == null) {           // second check, now holding the lock
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

`new Singleton()` isn't really one atomic step at the bytecode level — it's (1) allocate memory, (2) run the constructor, (3) assign the reference to `instance`. Without `volatile`, the JVM is allowed to reorder steps 2 and 3, meaning another thread could see a *non-null* `instance` that actually points to a half-constructed object — memory allocated, constructor not yet finished — and use it. `volatile` prevents exactly this reordering. It's a famous, genuinely subtle Java gotcha, and precisely why Day 100 asks you to compare Double-Checked Locking against the simpler, safer Enum Singleton pattern.

---

## Part 6: Higher-Level Tools — `ExecutorService`

Day 29 said: *"you'll get to Java's higher-level concurrency tools (`ExecutorService`...) later."* This is later.

Creating a raw `Thread` for every single task doesn't scale — thread creation has real OS-level overhead, and with no cap on how many you spin up, a burst of work could create thousands of threads and exhaust memory or choke on context-switching. `ExecutorService` fixes this with a **pool of reusable worker threads**: you submit tasks, the pool hands each one to an available worker, queues the rest, and reuses threads instead of constantly creating and destroying them.

```java
import java.util.concurrent.*;

ExecutorService executor = Executors.newFixedThreadPool(4); // 4 reusable worker threads

for (int i = 0; i < 20; i++) {
    int taskId = i;
    executor.submit(() -> {
        System.out.println("Task " + taskId + " running on " + Thread.currentThread().getName());
    });
}

executor.shutdown();                              // stop accepting new tasks
executor.awaitTermination(10, TimeUnit.SECONDS);   // wait up to 10s for a clean finish
```

Run this and watch the thread names — only 4 distinct threads handle all 20 tasks, reused as they free up.

When you need a *result* back, not just fire-and-forget, `submit()` a `Callable` instead of a `Runnable`, and you get a `Future`:

```java
Callable<Integer> task = () -> {
    Thread.sleep(100);
    return 42;
};

Future<Integer> future = executor.submit(task);
Integer result = future.get(); // blocks until the task finishes, then returns its result
```

Nothing later in this plan builds on `ExecutorService` by name, but it's the real-world tool that replaces almost every raw `Thread` you'd otherwise hand-write, and it's a fair, common SDE-2 interview question in its own right.

---

## Part 7: Connecting This Forward

- **Day 37 (`ReentrantLock`, `ReadWriteLock`):** a direct extension of Part 2 — an explicit lock object instead of `synchronized`'s implicit one, buying `tryLock()`, timeouts, and fairness.
- **Day 38 (Wait/Notify, `Condition`):** builds on Part 2's monitor-lock concept specifically — `wait()`/`notify()`/`notifyAll()` are methods on `Object` itself precisely because every object already has the intrinsic lock Part 2 described; `Condition` is `ReentrantLock`'s equivalent of the same idea.
- **Day 39 (`ConcurrentHashMap` internals):** directly explained by Part 4's CAS walkthrough.
- **Day 100 (Double-Checked Locking):** directly explained by Part 5's `volatile` section.
- **Day 111 (Parking Lot thread safety) and Day 117 (BookMyShow locking):** both ask you to spawn many threads and *prove* no double-booking happens — that's Part 1's race condition, deliberately induced and then shown fixed, using exactly the tools from Parts 2–5.
- **Day 138 (Databricks-specific bounded blocking queue):** explicitly requires `ReentrantLock`/`Condition` with "no `java.util.concurrent` shortcuts" — meaning you hand-build, from Parts 2 and 6's underlying primitives, something `ExecutorService` would normally hand you for free. Understanding *why* it normally works is what makes building it by hand possible.

---

## Coding Exercise

1. **See it break.** Build `UnsafeCounter` and `RaceConditionDemo` exactly as in Part 1. Run it 3 times. Confirm the actual count comes in below expected each time — this is the bug, made visible on purpose.
2. **Fix it two ways.** Rewrite `increment()` as (a) a `synchronized` method, and (b) using an explicit `ReentrantLock` with the `try`/`finally` pattern. Confirm both versions produce the exact expected count, every run, at least 5 runs each — the same "don't trust a single passing run" standard the plan already uses for its own concurrency tests (Day 111, Day 117).
3. **See visibility, separately from atomicity.** Build the `volatile boolean running` stop-flag example from Part 5. In a comment, explain why removing `volatile` wouldn't reliably break it on every single run — visibility bugs are often non-deterministic, which is itself worth knowing: a concurrency bug can pass by luck far more easily than a normal logic bug.
4. **`ExecutorService` in practice.** Submit 20 numbered tasks to a 4-thread fixed pool, print which thread handles which task, and confirm threads get reused rather than created fresh each time.

**Definition of done:** you can explain, out loud, without notes: why `count++` isn't atomic, what `synchronized`/`ReentrantLock` actually do (mutual exclusion via a lock), what CAS is and how it gets atomicity without ever blocking a thread, what `volatile` guarantees — and, just as importantly, what it does *not* guarantee — and why `ExecutorService` is preferred over creating raw `Thread`s by hand.

**Repository:** `java-fundamentals`, matching Day 37's own project block — push all four parts of the exercise there under a `concurrency` package, ahead of Day 37's `Counter`/`ReentrantLock` task.

---

## Quick Reference

| Term | One-line definition |
|---|---|
| Race condition | A bug where correctness depends on the unpredictable timing/interleaving of threads |
| Critical section | Code where unsynchronized concurrent access can corrupt shared state |
| Mutual exclusion | Ensuring only one thread executes a critical section at a time |
| Intrinsic lock (monitor) | The built-in lock every Java object has, used by `synchronized` |
| `synchronized` | Acquires an object's intrinsic lock for a method/block, releases it automatically on exit |
| Atomic operation | An operation that appears instantaneous — no other thread ever observes a partial result |
| CAS (Compare-And-Swap) | An atomic hardware instruction: swap a value only if it hasn't changed since you read it; retry if it has |
| Visibility | Whether one thread's write is guaranteed to be seen by another thread's later read |
| `volatile` | Guarantees visibility of a field's reads/writes across threads — does **not** guarantee atomicity |
| `ExecutorService` | A pool of reusable worker threads that tasks are submitted to, instead of hand-creating raw `Thread`s |
| `Future` | A handle to a still-running (or completed) task's eventual result |

### Daily Deliverable
- [ ] Can explain a race condition and why `count++` isn't atomic, without notes.
- [ ] Can explain what `synchronized` actually does (mutual exclusion via an object's intrinsic lock).
- [ ] Can explain CAS and why it achieves atomicity without blocking.
- [ ] Can state precisely what `volatile` guarantees and what it doesn't.
- [ ] Can explain why `ExecutorService` is preferred over raw `Thread` creation.
- [ ] All four coding exercise parts complete and pushed to `java-fundamentals`, verified correct across multiple runs.
- [ ] Ready for Day 37 with the actual problem understood, not just the next tool's name.
