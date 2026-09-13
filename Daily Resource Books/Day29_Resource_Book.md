# Day 29 — Binary Search Continues: Insertion Points, Peak Finding, and the JVM Concurrency Model

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 28 Resource Book](Day28_Resource_Book.md)
**Next ▶:** [Day 30 Resource Book](Day30_Resource_Book.md)
**Companion to:** Day 29 of `Week_05_Revised.md`

---

## Recap

Day 28 gave you binary search twice: LC 704 (exact match, "on the input") and LC 278 (boundary search on a monotonic condition, "on the answer") — plus the overflow-safe midpoint, `left + (right - left) / 2`, which itself leans on Day 10's overflow mechanics (`(left + right) / 2` can silently overflow `int` when both are large; subtracting first avoids ever forming that oversized intermediate sum). Today stays on the "on the input" side of that fork and stretches the template two new ways: reading an answer off *where the loop naturally ends* rather than off a `found` flag, and applying binary search to an array that isn't sorted at all — only locally well-behaved.

The theory switches tracks entirely. Threads have no dependency on anything from Week 4; they build directly on Day 9's JVM Memory Model (stack vs. heap) and Day 2's single-inheritance rule, both of which get reused today, not re-explained.

---

## Learning Objectives

By the end of today, without notes:

1. Apply the exact-match binary search template to a problem where the answer is *where the search converges*, not a value found mid-search — and prove why that convergence point is correct.
2. State, and prove, the invariant that lets binary search run on an array with no global ordering at all — only a local "which direction is a peak guaranteed" signal.
3. Explain, from the stack/heap mechanism, exactly why two threads can corrupt shared object state but not each other's local primitives.
4. Name all six of the JVM's actual `Thread.State` values (not just the five-stage conceptual model) and say which of the plan's "Blocked/Waiting" bucket maps to which.
5. Build a minimal multi-threaded Java program using `Runnable` and explain, from the scheduler's perspective, why its output order isn't fixed across runs.

---

## Concept Dependency Map

```
Day 28: LC 704 exact-match template + overflow-safe midpoint (left + (right-left)/2)
Day 28: LC 278 boundary search, "on the answer" (first formal appearance of that framing)
Day 10: int overflow mechanics (why (left+right)/2 is unsafe)
        │
        ├──▶ Today, Problem 3: Search Insert Position (LC 35)
        │        exact-match template, reused verbatim — new twist is reading the
        │        answer off WHERE left and right cross, not off a found index
        │
        └──▶ Today, Problem 4: Find Peak Element (LC 162)
                 binary search on an array with NO global order — NEW invariant:
                 "the slope at mid, alone, guarantees a peak on one side"

Day 1: process/program model, JVM basics
Day 2: interfaces; single inheritance (`extends` one class, `implements` many)
Day 9: JVM Memory Model — one stack per call, ONE shared heap
Day 18: checked vs. unchecked exceptions (InterruptedException is checked)
        │
        └──▶ Today: Threads and the JVM Concurrency Model
                 ├─ one stack PER THREAD, still one shared heap per process
                 │  (Day 9's picture, extended from 1 thread to N)
                 ├─ Thread vs. Runnable (needs: single inheritance, Day 2)
                 ├─ NEW SYNTAX: anonymous inner classes (needed for Runnable,
                 │  reused later this week for Comparator)
                 └─ Thread lifecycle; observing real interleaving/non-determinism
```

---

# Part 1 — Binary Search Continues

### Prerequisites (confirmed)

- Exact-match binary search template, overflow-safe midpoint — Day 28.
- Array indexing, `int` overflow mechanics — Day 2, Day 10.

The single idea underneath both of today's problems: binary search doesn't actually require "the array is sorted." It requires "at every step, a piece of local information lets me safely discard half the remaining search space, without missing the answer." A sorted array is the most common way to get that guarantee — but as Problem 4 below shows, it isn't the only way.

---

## Problem 3: Search Insert Position (LeetCode 35, Easy) — Pattern: Binary Search, Exact Match (Insertion Point)

**Statement:** Given a sorted array of distinct integers `nums` and an integer `target`, return the index of `target` if it exists. If it doesn't, return the index where it would be inserted to keep the array sorted.

### Approach 1 — Brute force

```java
public static int searchInsertBruteForce(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] >= target) {
            return i;
        }
    }
    return nums.length;   // target is larger than every element — insert at the end
}
```

Scan left to right; the first index whose value is `>= target` is the answer (an exact match satisfies `>=` too, so this one loop handles both cases). Time O(n), Space O(1).

### Approach 2 — Optimized: binary search, same template as LC 704

```java
public static int searchInsert(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return left;   // insertion point
}
```

This is Day 28's LC 704 template with exactly one line changed: instead of `return -1` on no match, `return left`.

**Why `left` is guaranteed correct — the invariant:** at every point in the loop, everything at an index `< left` is strictly less than `target`, and everything at an index `> right` is `>= target`. This holds at the start trivially (there's nothing outside `[left, right]` yet) and is preserved by both branches: moving `left = mid + 1` only happens after confirming `nums[mid] < target`, so everything up to and including the old `mid` is now correctly below `left`; moving `right = mid - 1` only happens after confirming `nums[mid] > target` (the `==` case already returned), so everything from the old `mid` onward is correctly at or above `right + 1`. The loop exits exactly when `left > right` — and since the loop condition guarantees `right == left - 1` at that moment, the invariant says: everything before `left` is `< target`, and everything from `left` onward (which is now the *entire* remaining array) is `>= target`. That makes `left` exactly the first index at or above `target` — the correct insertion point, by definition.

**Worked trace:** `nums = [1, 3, 5, 6]`, `target = 2`.

| left | right | mid | nums[mid] | comparison | action |
|---|---|---|---|---|---|
| 0 | 3 | 1 | 3 | 3 > 2 | `right = 0` |
| 0 | 0 | 0 | 1 | 1 < 2 | `left = 1` |
| 1 | 0 | — | — | `left > right` | loop ends |

Returns `left = 1`. Check: inserting `2` between `nums[0]=1` and `nums[1]=3` keeps the array sorted — `[1, 2, 3, 5, 6]`. Correct.

One more, at the boundary: `target = 7` (larger than every element). `left=0,right=3 → mid=1(3<7) → left=2`; `left=2,right=3 → mid=2(5<7) → left=3`; `left=3,right=3 → mid=3(6<7) → left=4`; now `left=4 > right=3`, loop ends, returns `4` — one past the last index, i.e., append at the end. Correct.

**Complexity:** Time O(log n), Space O(1).

**Edge cases:**
- `target` smaller than every element → converges to `left = 0`.
- `target` larger than every element → converges to `left = nums.length`, one past the last valid index (this is legal *as a return value*, since it means "insert after everything"; it would be out of bounds as an *access*, but nothing here accesses `nums[left]` after the loop).
- `target` equal to an existing element → caught by the `==` branch directly, never falls through to the insertion-point logic.
- Single-element array → loop runs at most once, both branches still correct.

**⚠️ Common Mistake:** returning `right` instead of `left`, or writing `right + 1`. Since the loop always exits with `right == left - 1` (proven above), `right + 1` and `left` are numerically identical — but `left` is the version worth internalizing, because it's the one that generalizes cleanly to boundary-search problems later this week (Day 31), where the two pointers *don't* always end exactly one apart in the same way.

**💡 Interview Insight:** open by naming this as "the same shape as Binary Search, LC 704, with the failure case redefined" — recognizing a template rather than re-deriving from scratch is exactly what's being evaluated. A near-certain follow-up: *"what if duplicates were allowed?"* The honest answer is that this specific template still returns *a* valid insertion point, but which one (leftmost vs. rightmost among equal values) becomes ambiguous — that ambiguity is precisely what Day 31's boundary-search problem (LC 34) exists to resolve on purpose, so it's fine to name the connection now and defer the mechanism.

---

## Problem 4: Find Peak Element (LeetCode 162, Medium) — Pattern: Binary Search on an Unsorted Array (NEW)

**Statement:** `nums` is an array where no two adjacent elements are equal. A peak is an element strictly greater than its neighbors; treat `nums[-1]` and `nums[n]` as `-infinity`. Return the index of *any* peak — the array may have several, and any correct one is accepted. Required in O(log n).

### Approach 1 — Brute force

```java
public static int findPeakBruteForce(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        boolean leftOk = (i == 0) || nums[i - 1] < nums[i];
        boolean rightOk = (i == nums.length - 1) || nums[i + 1] < nums[i];
        if (leftOk && rightOk) {
            return i;
        }
    }
    return -1;   // unreachable given the problem's guarantees, but keeps the method total
}
```

Check every index directly against its neighbors, using the boundary convention. Time O(n), Space O(1). Correct, but doesn't use the fact that a *much* faster answer exists — the boundary condition alone guarantees a peak exists somewhere, without needing to inspect every element.

### Approach 2 — Optimized: binary search on the local slope

```java
public static int findPeakElement(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < nums[mid + 1]) {
            left = mid + 1;    // ascending at mid — a peak is guaranteed to the right
        } else {
            right = mid;       // descending at mid — a peak is guaranteed at mid, or to the left
        }
    }
    return left;   // left == right here, and it's guaranteed to be a peak
}
```

**🔑 Key Takeaway — what actually makes this binary search, when the array isn't sorted:** binary search only ever needs one thing at each step: a rule that safely discards half the remaining space without discarding the answer. A globally sorted array is one way to get that rule (Day 28, Day 29 Problem 3). Here the rule comes from a purely *local* comparison instead — `nums[mid]` vs. `nums[mid + 1]` — and that's enough, because of the boundary convention.

**Why this works — the correctness proof:** claim — at every iteration, the current range `[left, right]` is guaranteed to contain at least one peak (using the `-infinity` boundary convention at the true array edges). This holds initially for the whole array: walking from the left boundary (`-infinity`) across the array to the right boundary (`-infinity`), the sequence cannot be monotonically increasing forever, because it's bounded above by nothing and below by `-infinity` on both true ends — somewhere it must turn from non-decreasing to decreasing, and that turn is a peak by definition.

Now show each branch preserves the claim on a strictly smaller range:
- If `nums[mid] < nums[mid + 1]` (ascending at `mid`): walk rightward from `mid + 1` toward the right boundary. Either the sequence keeps rising all the way to the true right edge, in which case the last real element is greater than everything to its left *and* greater than the virtual `-infinity` to its right — a peak — or at some point it turns downward, and that turning point is a peak by the same argument as the base case. Either way, `[mid + 1, right]` contains a peak, and that's exactly the new range (`left = mid + 1`).
- If `nums[mid] >= nums[mid + 1]` (which, since no two adjacent elements are equal, really means `nums[mid] > nums[mid + 1]`): if `mid` itself isn't already a peak, it's only because `nums[mid - 1] >= nums[mid]` — but then apply the identical argument leftward. Either the walk left from `mid` finds a turning point (a peak) before reaching the true left edge, or it rises the whole way to the true left edge, making that edge element a peak against its `-infinity` left boundary. Either way, `[left, mid]` contains a peak — exactly the new range (`right = mid`).

Each step strictly shrinks the range (`mid + 1 > mid`, and `mid < right` whenever `left < right`, so `right = mid` is strictly smaller too), so the loop terminates. When `left == right`, the claim says that single-element range contains a peak — so that element *is* one.

**Worked trace:** `nums = [1, 2, 1, 3, 5, 6, 4]`.

| left | right | mid | nums[mid] | nums[mid+1] | comparison | action |
|---|---|---|---|---|---|---|
| 0 | 6 | 3 | 3 | 5 | 3 < 5, ascending | `left = 4` |
| 4 | 6 | 5 | 6 | 4 | 6 > 4, descending | `right = 5` |
| 4 | 5 | 4 | 5 | 6 | 5 < 6, ascending | `left = 5` |
| 5 | 5 | — | — | — | `left == right` | loop ends |

Returns `5`. Check: `nums[5] = 6`; neighbors are `nums[4] = 5` and `nums[6] = 4`, both smaller. Valid peak. (Note `nums[3] = 3` is *not* a peak — `nums[2] = 1 < 3` but `nums[4] = 5 > 3` — the algorithm correctly walked past it without ever needing to check it against both neighbors directly.)

**Complexity:** Time O(log n) — the range strictly halves (or better) each iteration. Space O(1).

**Edge cases:**
- Single element (`n = 1`) — `left == right == 0` immediately, loop body never runs, returns `0`. Correct: a lone element beats `-infinity` on both sides trivially.
- Strictly increasing array (e.g., `[1, 2, 3, 4]`) — every comparison is ascending, `left` walks all the way to `n - 1`, which is correctly the peak (beats its real left neighbor and the virtual right boundary).
- Strictly decreasing array — mirror image, converges to index `0`.
- Multiple valid peaks — the algorithm returns whichever one it converges toward; the problem statement accepts any of them, so this is never a correctness issue.

**⚠️ Common Mistake:** assuming "binary search" means "the array must be sorted." The actual requirement is the weaker one proven above — a rule that discards half the space without discarding the answer. Conflating "sorted" with "binary-searchable" is exactly the gap this problem is designed to close.

**💡 Interview Insight:** LeetCode 852, *Peak Index in a Mountain Array*, is the same technique applied to an array *guaranteed* to be strictly increasing and then strictly decreasing — a true single mountain. That guarantee removes the need to reason about multiple peaks or virtual boundaries at all; if it comes up, it's a strictly easier version of what you just proved here, not a new problem to re-derive.

---

# Part 2 — Threads and the JVM's Concurrency Model

### Prerequisites (confirmed)

- Program model: source → compiler → bytecode → JVM (Day 1).
- JVM Memory Model: one call stack (holding local variables, primitives, and object *references*) vs. one heap (holding every object) — Day 9.
- Interfaces and single inheritance: a class `extends` exactly one class but can `implements` many interfaces — Day 2.
- Checked vs. unchecked exceptions — Day 18 (needed below: `InterruptedException` is checked).

## What a thread actually is

A **process** is one running instance of a program, with its own memory space, managed by the OS. A **thread** is an independent path of execution *inside* a process. A single Java program has at minimum one thread (the one running `main`) — everything from Day 1 through yesterday has been implicitly single-threaded.

**This is a direct extension of Day 9, not a new memory model.** Day 9 established that every method call gets its own stack frame (holding that call's local variables and primitives), while every object, no matter which code created it, lives in one shared heap. Day 9 only ever discussed one thread, so "the stack" sounded like a single, fixed thing. The complete picture: it's **one stack per thread** — every thread gets its own entirely separate sequence of frames and local variables — but still **one heap, shared by every thread in the process**.

```
Thread A's stack        Thread B's stack        (one per thread — private)
┌─────────────┐         ┌─────────────┐
│ frame: run() │         │ frame: run() │
│  int i = 3   │         │  int i = 9   │        ← different memory, same variable name,
└─────────────┘         └─────────────┘            zero interaction, zero risk

                    Heap (ONE, shared by every thread in the process)
              ┌───────────────────────────┐
              │  Counter object: count=17  │   ← BOTH threads can hold a reference
              └───────────────────────────┘      to this SAME object and modify it
```

**🔑 Key Takeaway — this is exactly why concurrent code corrupts shared state:** a local primitive (like `int i` above) is always safe across threads — it lives on that thread's own stack, so two threads with a same-named local variable are touching two completely distinct memory locations. An object on the heap is different: if two threads each hold a reference to the *same* heap object and both modify its fields without coordination, the modifications can interleave unpredictably — a **race condition**. Today's exercise below only demonstrates the *scheduling* side of this (non-deterministic order); actual data corruption from a shared mutable object is Week 6, Day 37's `Counter` exercise, once `synchronized`/`ReentrantLock` are on the table as the fix. Flagging that connection now, since it's the direct payoff of today's mental model.

## Creating a thread: `extends Thread` vs. `implements Runnable`

```java
// Option A — extends Thread
class GreeterThread extends Thread {
    @Override
    public void run() {
        System.out.println("Hello from GreeterThread");
    }
}
// usage: new GreeterThread().start();

// Option B — implements Runnable (preferred)
class GreeterTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Hello from GreeterTask");
    }
}
// usage: new Thread(new GreeterTask()).start();
```

**Why `Runnable` is preferred — this is Day 2's single-inheritance rule, applied directly:** a class can `extends` only one thing. If a class extends `Thread`, that one `extends` slot is spent — it can never extend anything else, ever, even if its own natural type hierarchy would call for it (imagine a class that's fundamentally a `DataProcessor` but also needs to run on its own thread — it can't be both `extends DataProcessor` and `extends Thread` at once). Implementing `Runnable` costs nothing — a class keeps its one `extends` slot free and simply *also* declares itself runnable, exactly the way it could `implements` any other interface alongside its real superclass. `Runnable` also cleanly separates *the work to be done* (a `Runnable`) from *the mechanism that runs it* (a `Thread`) — a separation that pays off directly later, once higher-level tools like `ExecutorService` take `Runnable` tasks and decide *how and when* to actually run them on a managed pool of threads, rather than you managing raw `Thread` objects yourself. That's coming later in the plan; today is only the raw mechanism underneath it.

**⚠️ Common Mistake:** calling `.run()` directly instead of `.start()`. `new GreeterThread().run()` just calls `run()` as an ordinary method, on the *current* thread — no new thread is created at all, and the "concurrent" behavior silently never happens. `.start()` is the only call that actually asks the JVM to spin up a new OS-backed thread and have *it* execute `run()`.

### New Syntax: Anonymous Inner Classes

Both examples above needed a full named class just to provide one throwaway implementation of `run()`. Java has a lighter-weight tool for exactly this situation: an **anonymous inner class** — a class declared and instantiated in a single expression, with no name of its own.

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello from an anonymous class");
    }
};
new Thread(task).start();
```

**Mechanically:** this still compiles to a real class file (the compiler generates a name like `SomeOuterClass$1.class` behind the scenes) — the only difference from `GreeterTask` above is that *you* never named it, because you only ever needed exactly one instance, used right where it's defined. This is the same underlying mechanism as Day 5's enum-constant bodies (`SAVINGS { @Override public double calculateInterest(...) {...} }`) — that syntax also generates an unnamed subclass per constant; today's syntax is the general-purpose version of the same idea, usable anywhere an interface or superclass is expected.

**When to reach for it vs. a named class:** a named class when the implementation is reused in multiple places, or complex enough to deserve its own file and a real name. An anonymous class when you need exactly one, short, throwaway instance, right where it's used — like a one-off `Runnable`, or (reused later this week) a one-off `Comparator` passed straight into a `PriorityQueue` constructor.

## Thread Lifecycle

The conceptual model: **New → Runnable → Running → Blocked/Waiting → Terminated.**

- **New:** a `Thread` object exists (`new Thread(...)` has run), but `.start()` hasn't been called — no actual OS thread exists yet, just the Java object describing one.
- **Runnable:** `.start()` has been called; the thread is eligible to run, waiting for the OS scheduler to actually grant it CPU time.
- **Running:** the scheduler has granted CPU time; `run()` is actively executing.
- **Blocked/Waiting:** alive, but not executing — waiting for a lock another thread holds, or waiting on some coordination signal.
- **Terminated:** `run()` has returned, normally or via an uncaught exception. A terminated thread cannot be restarted — calling `.start()` again throws `IllegalThreadStateException`.

**Technical accuracy check — the JVM's actual `Thread.State` enum has six values, not five**, and it's worth being exact about the mapping rather than treating the five-stage model as literal: `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, `TERMINATED`. Two corrections to the conceptual model above:

1. Java's `RUNNABLE` state covers **both** "eligible, waiting for the scheduler" and "actually executing right now" — the JVM doesn't expose that OS-level distinction as two separate states. So the conceptual model's "Runnable" and "Running" boxes are one and the same `Thread.State` value in practice; they're kept as two conceptual stages here only because *why* a thread isn't running yet (scheduler hasn't picked it) is a useful mental distinction, even though the JVM itself doesn't track it separately.
2. The conceptual model's single "Blocked/Waiting" bucket is actually **three** distinct JVM states: `BLOCKED` specifically means waiting to acquire a lock held by another thread (a `synchronized` block, coming Week 6); `WAITING` means waiting indefinitely on a coordination call with no timeout (`Object.wait()`, `Thread.join()`); `TIMED_WAITING` is the same but with a timeout (`Thread.sleep(ms)` — used below — `Object.wait(ms)`). This distinction matters concretely the first time a real thread dump needs reading, since `jstack` output reports these three separately, not as one bucket.

## Coding Exercise: Observing Real Interleaving

```java
class PrintTask implements Runnable {
    @Override
    public void run() {
        String name = Thread.currentThread().getName();
        for (int i = 1; i <= 5; i++) {
            System.out.println(name + ": " + i);
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();   // preserve the interrupt, don't swallow it
            }
        }
    }
}

public class ThreadInterleavingDemo {
    public static void main(String[] args) {
        Thread threadA = new Thread(new PrintTask(), "Thread-A");
        Thread threadB = new Thread(new PrintTask(), "Thread-B");
        threadA.start();
        threadB.start();
    }
}
```

**Why the printed order changes between runs:** once both threads are started, *which* thread actually gets the CPU at any given moment is decided by the OS scheduler — a decision that depends on system load, other running processes, and scheduling policy details entirely outside this program's control. `Thread.sleep(100)` only guarantees that *this* thread won't run for at least 100ms; it says nothing about which thread the scheduler picks next once that time is up. Running the exact same program three times can — and typically will — produce three different interleavings of `Thread-A: 1`, `Thread-B: 1`, `Thread-A: 2`, and so on. This is **non-determinism**: identical code and input, but a different (each individually valid) observable output across runs.

**⚠️ Common Mistake, and an important scope note:** this exercise demonstrates *scheduling* non-determinism, not *data corruption* — there's no shared mutable state here for the two threads to corrupt, since each thread only ever touches its own local `i` (safe, per-thread stack) and calls `System.out.println()` (whose individual calls are internally synchronized, so one call's text won't get character-interleaved with another's). Actual corruption requires a shared mutable object both threads write to without coordination — that's Week 6, Day 37's `Counter` exercise, once locking is available as the fix. Don't conflate "output order is unpredictable" with "the count came out wrong" — they're different failure modes with different fixes.

**⚠️ Common Mistake:** forgetting `Thread.sleep()` throws a **checked** `InterruptedException` (Day 18) — it must be caught or declared. And when caught but not usefully handled, the idiomatic fix is `Thread.currentThread().interrupt()` (shown above), which re-flags the thread as interrupted for any *outer* code that might be checking — silently swallowing the exception instead breaks cooperative cancellation for any code built on top of this one later.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** `ThreadInterleavingDemo`, exactly as built above.

Run it three times in a row and actually look at the output — don't just assume it varies. Add a one-line comment directly above `main()` explaining, in your own words, what "non-deterministic" means *in this specific context* (which thread the scheduler picks next isn't controlled by your code). Definition of done: pushed, with that comment in place.

---

## Career Block Guide (1 hr)

**LinkedIn (20 min):** comment meaningfully on 3–5 posts in your feed — a specific reaction to something concrete in the post beats a generic "Great post!" by a wide margin, and is the difference between actually being noticed and being invisible engagement.

**Networking:** identify 3 companies with strong engineering blogs (tier-1 or strong tier-2 Indian product companies, or remote-friendly companies with public engineering writing — this is groundwork for later outreach, not outreach itself yet).

---

## Day 29 — Interview Questions

**Q1. Search Insert Position reuses LC 704's exact-match template almost unchanged. What's the one thing that actually differs?** Only the no-match return value — LC 704 returns `-1`; this problem returns `left`, the index where the search loop naturally converges once `left` crosses `right`.

**Q2. Prove why `left` is the correct insertion point when the loop ends.** The loop maintains the invariant that everything before `left` is `< target` and everything after `right` is `>= target`. Since the loop only exits when `right == left - 1`, at that moment the entire array is partitioned exactly at `left` — everything before is smaller, everything from `left` onward is `>=` — which is the definition of the correct insertion point.

**Q3. What's the core insight that lets Find Peak Element binary search an array with no global sort order?** Binary search only needs a rule that safely discards half the search space at each step without discarding the answer — a sorted array is one way to get that, but a purely local comparison (`nums[mid]` vs. `nums[mid+1]`) works too, as long as it's proven to always leave a peak on the kept side.

**Q4. Walk through why an ascending slope at `mid` guarantees a peak strictly to the right.** Walking right from `mid+1`, the sequence either keeps rising until the (virtual `-infinity`) right boundary, making the last real element a peak, or it turns downward somewhere, and that turning point is a peak. Either way a peak exists in `[mid+1, right]`.

**Q5. Why doesn't the array need to have exactly one peak for this to work?** The problem only asks for *any* valid peak, and the proof only ever claims "a peak exists somewhere in this range" — it never claims uniqueness, so multiple peaks don't break the argument or the algorithm.

**Q6. Precisely, what's the difference between a thread and a process?** A process is an independently running program instance with its own memory space, managed by the OS. A thread is one path of execution inside a process; a process can contain multiple threads, and every thread in that process shares the process's memory.

**Q7. Why can two threads corrupt a shared object's fields but never corrupt each other's local `int` variables?** Every thread gets its own private call stack, which is where local primitives live — so two threads' local variables are physically different memory, zero interaction possible. All objects, regardless of which thread created them, live in one heap shared by every thread in the process — so two threads holding references to the same heap object can both modify it, with no guarantee about how those modifications interleave.

**Q8. Why prefer `implements Runnable` over `extends Thread`?** Java allows a class to `extend` only one class. Extending `Thread` permanently spends that slot; implementing `Runnable` costs nothing and leaves the class free to extend whatever its actual type hierarchy calls for, while still being runnable on a thread.

**Q9. What actually happens if you call `.run()` instead of `.start()`?** `run()` executes as an ordinary method call on the *current* thread — no new thread is created, so nothing runs concurrently; the code that looks "multi-threaded" silently behaves as if it were sequential.

**Q10. The JVM's `Thread.State` enum has six values. Name them, and say which of the five-stage conceptual model each maps to.** `NEW` → New. `RUNNABLE` → covers both "Runnable" and "Running" (the JVM doesn't distinguish "eligible" from "actually executing"). `BLOCKED` → waiting specifically for a lock. `WAITING` and `TIMED_WAITING` → waiting on a coordination call, without and with a timeout respectively (both fall under the conceptual model's "Blocked/Waiting"). `TERMINATED` → Terminated.

**Q11. Why does running `ThreadInterleavingDemo` three times produce three different outputs, even though the code never changed?** Which thread the OS scheduler grants CPU time to, at any given moment, depends on factors entirely outside the program's control (system load, scheduling policy, timing). `Thread.sleep()` only guarantees a minimum wait, not what happens immediately after — so the exact interleaving of both threads' print statements isn't fixed by the code.

---

## Daily Deliverable Check

- [ ] Search Insert Position and Find Peak Element solved, pushed to `dsa-java/binary-search/`.
- [ ] Can explain, from the `nums[mid]` vs. `nums[mid+1]` comparison, why Find Peak Element doesn't require a sorted array.
- [ ] Can explain the thread lifecycle without notes, including the six real `Thread.State` values, not just the five-stage conceptual model.
- [ ] `ThreadInterleavingDemo` pushed, run 3 times, with a comment explaining non-determinism in your own words.
- [ ] LinkedIn engagement done; 3 target companies with strong engineering blogs identified.

---

## What Tomorrow Assumes You Already Know Cold

Day 30 assumes the exact-match binary search template — sorted-array assumption and all — is fully automatic, since tomorrow's problems (rotated sorted arrays) start from that template and modify exactly one thing about it: *which* half is guaranteed sorted, rather than *whether* the whole array is. Today's Find Peak Element proof style (showing an invariant is preserved and eventually forces a single correct answer) is the same style tomorrow's rotated-array proofs use — if today's proof still feels like a trick rather than a repeatable argument shape, it's worth re-reading before moving on. The threading material doesn't feed directly into tomorrow (no theory block on Day 30) — it resurfaces starting Week 6, Day 36, so nothing here needs to stay "hot," just retained.
