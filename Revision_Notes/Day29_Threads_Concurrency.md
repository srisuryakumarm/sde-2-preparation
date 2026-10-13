# Threads & Concurrency Model — Quick Revision

## 1. Process vs Thread

**Process**
- A running instance of a program.
- Has its own memory space.

**Thread**
- An independent path of execution inside a process.
- A Java program starts with at least one thread: the thread running `main()`.

```text
Process
 ├── Thread A
 ├── Thread B
 └── Thread C
```

---

## 2. Thread Memory Model

The key mental model:

```text
Each Thread → Own Stack
All Threads → Shared Heap
```

### Stack
Contains:
- Method stack frames
- Local variables
- Primitive values
- Object references

Each thread has its **own stack**, so:

```java
Thread A → int i = 3;
Thread B → int i = 9;
```

These are completely separate variables.

### Heap
- Objects live on the heap.
- The heap is shared by all threads in the process.

Therefore:

```text
Thread A ──┐
           ├──→ Same heap object
Thread B ──┘
```

If multiple threads modify the same mutable object without coordination → **possible race condition**.

> Important: Unpredictable execution order is not the same as data corruption.

---

## 3. Creating a Thread

### `extends Thread`

```java
class MyThread extends Thread {
    @Override
    public void run() {
        // task
    }
}
```

### `implements Runnable` — preferred

```java
class MyTask implements Runnable {
    @Override
    public void run() {
        // task
    }
}

new Thread(new MyTask()).start();
```

### Why prefer `Runnable`?

Java supports **single inheritance**.

```text
extends → only one class
implements → multiple interfaces
```

`Runnable` also separates:

```text
Runnable → Work / Task
Thread   → Mechanism that executes the task
```

This separation becomes important later with `ExecutorService` and thread pools.

---

## 4. `run()` vs `start()`

### `run()`

```java
thread.run();
```

Just a normal method call.

```text
Current thread
     ↓
  run()
```

**No new thread is created.**

### `start()`

```java
thread.start();
```

Starts a new thread, which then executes `run()`.

```text
start()
   ↓
New thread
   ↓
run()
```

### Remember

> `run()` = execute method  
> `start()` = start thread

---

## 5. Anonymous Inner Class

Useful for a one-off `Runnable`:

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

new Thread(task).start();
```

Use:
- **Named class** → reusable/complex implementation.
- **Anonymous class** → small, one-off implementation.

---

## 6. Thread Lifecycle

### Conceptual model

```text
NEW
 ↓
RUNNABLE
 ↓
executing
 ↓
WAITING / BLOCKED / TIMED_WAITING
 ↓
RUNNABLE
 ↓
TERMINATED
```

### Actual Java `Thread.State`

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

### Meaning

| State | Meaning |
|---|---|
| `NEW` | Thread created but `start()` not called |
| `RUNNABLE` | Eligible to run / executing |
| `BLOCKED` | Waiting to acquire a lock |
| `WAITING` | Waiting indefinitely |
| `TIMED_WAITING` | Waiting for a specified time |
| `TERMINATED` | `run()` has finished |

### Important

Java does **not** have a separate `RUNNING` state.

`RUNNABLE` covers both:
- waiting for CPU
- actually executing

---

## 7. `sleep()`

```java
Thread.sleep(100);
```

Means:

> This thread should not execute for the specified period.

It does **not** mean:

> This thread will definitely execute immediately after 100 ms.

After sleeping, the scheduler decides when it gets CPU time again.

`sleep()` causes:

```text
TIMED_WAITING
```

---

## 8. Thread Scheduling & Non-Determinism

When multiple threads are running:

```java
threadA.start();
threadB.start();
threadC.start();
```

The scheduler determines which runnable thread gets CPU time.

Therefore output order can vary:

```text
A1
B1
C1
A2
C2
B2
```

or:

```text
C1
A1
B1
B2
C2
A2
```

Both can be valid.

### Best comment for the code

```java
// The scheduler determines which runnable thread gets CPU time,
// so the execution order and resulting output can vary between runs.
```

### Key term

**Non-determinism** = the same program can produce different valid execution/output orders across runs.

---

## 9. Scheduling Non-Determinism ≠ Race Condition

### Scheduling non-determinism

```text
Thread A prints
Thread B prints
Thread C prints
```

The order can change.

This is what the `PrintTask` exercise demonstrates.

### Race condition

Multiple threads access and modify the **same shared mutable state** without proper coordination.

Example:

```java
counter.count++;
```

Potential result:

```text
Incorrect final value
```

So remember:

```text
Unpredictable order
        ≠
Data corruption
```

---

## 10. `InterruptedException`

`Thread.sleep()` can throw the checked exception:

```java
InterruptedException
```

Therefore it must be caught or declared.

Recommended handling:

```java
try {
    Thread.sleep(100);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

### Why?

Calling:

```java
Thread.currentThread().interrupt();
```

preserves the thread's interrupted status instead of silently swallowing the interruption.

---

# SDE-2 Mental Model

Remember these five statements:

1. **A process contains threads.**
2. **Each thread has its own stack.**
3. **Threads in the same process share the heap.**
4. **`start()` creates concurrent execution; `run()` is just a method call.**
5. **The scheduler controls execution order, so concurrent execution can be non-deterministic.**

### One-line summary

> **Threads execute independently with separate stacks, but can access shared heap objects, and the scheduler can interleave their execution in a non-deterministic order.**