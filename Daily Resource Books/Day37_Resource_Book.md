# Day 37 — Linked Lists: Harder Pointer Manipulation, and Explicit Locks

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 36 Resource Book](Day36_Resource_Book.md)
**Next ▶:** [Day 38 Resource Book](Day38_Resource_Book.md)
**Companion to:** Day 37 of `Week_06_Revised.md`

---

## Recap

Yesterday closed out the cycle-detection sub-family of pointer tricks. Today's two problems are a different flavor entirely — both are still "two pointers on a linked list," but the mechanism is a *fixed offset* between them (Remove Nth From End) and a *synchronized parallel walk with carried state* (Add Two Numbers), rather than a speed difference. Both reuse the dummy-head technique from Day 35 directly.

On the theory side, today leaves linked lists behind and returns to Day 29's Threads and JVM Concurrency Model. That day demonstrated that thread execution order is non-deterministic — but it stopped short of showing an actual *wrong answer* coming out of a program. Today is where that happens: you'll watch a program produce a genuinely incorrect result because of unsynchronized shared state, and then fix it with a real locking mechanism.

---

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Derive the correct pointer gap for Remove Nth Node From End from a worked example, rather than recalling "n+1" as a memorized constant.
2. Explain precisely why `count++` is not atomic, at the level of what actually happens when it runs.
3. Reproduce a concrete thread interleaving that causes a lost update, and explain why more threads make the corruption *more* likely to appear, not less.
4. State exactly what `ReentrantLock` offers beyond `synchronized`, without overstating the difference (both are reentrant — that part is not new).

---

## Concept Dependency Map

```
Day 35: dummy head — a throwaway node to attach to before the real head exists
        │
        ├──▶ Today: Remove Nth Node From End (LC 19)
        │      fixed-gap two pointers, gap = n+1, using dummy head
        │      to avoid special-casing "remove the head itself"
        │
        └──▶ Today: Add Two Numbers (LC 2)
               parallel walk of two lists + a carried `carry` variable,
               dummy head to avoid special-casing an empty result list

Day 29: Threads, JVM concurrency model, six real Thread.State values,
        Runnable via anonymous inner class — demonstrated non-deterministic
        ORDERING only, not incorrect results
        │
        ▼
Today, NEW: `synchronized` — the JVM's built-in mutual-exclusion primitive
        (brief primer — needed as a baseline before contrasting it with...)
        │
        ▼
Today, NEW: `ReentrantLock` — explicit locking with more control
        (tryLock/timeout, fairness policy, NOT more "reentrant" than
        synchronized already was — that specific claim is a common
        misconception, corrected below)
        │
        ▼
Today: first actual DATA CORRUPTION demo in this series (`count++`
        under 100 concurrent threads) — fixed with ReentrantLock
```

---

## Part 1 — Linked Lists: Fixed-Offset Pointers, and Parallel Walks

### Problem 7: Remove Nth Node From End of List (LeetCode 19, Medium) — Pattern: Two Pointers (Fixed Offset)

**Statement:** Given `head` and an integer `n`, remove the `n`th node from the *end* of the list, and return the (possibly new) head. Do it in one pass.

#### Approach 1 — Two-pass

```java
public ListNode removeNthFromEndTwoPass(ListNode head, int n) {
    int length = 0;
    for (ListNode node = head; node != null; node = node.next) {
        length++;
    }
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode current = dummy;
    for (int i = 0; i < length - n; i++) {
        current = current.next;
    }
    current.next = current.next.next;
    return dummy.next;
}
```

First pass measures the list's length `L`. The node to remove, counted from the front (0-indexed), is at position `L - n`. A second pass walks `L - n` steps from `dummy` to land just before it. **Time O(L), Space O(1)** — but two full passes.

#### Approach 2 — Optimized: one-pass, fixed-gap two pointers

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode fast = dummy;
    ListNode slow = dummy;

    for (int i = 0; i <= n; i++) {
        fast = fast.next;
    }

    while (fast != null) {
        fast = fast.next;
        slow = slow.next;
    }

    slow.next = slow.next.next;
    return dummy.next;
}
```

**Why the gap is `n + 1`, not `n`:** the goal is for `slow` to end up on the node *just before* the target, since removal is `slow.next = slow.next.next`. If `fast` only led by `n`, then when `fast` hit `null`, `slow` would land exactly *on* the target node, not before it. Leading by one extra step is what makes `slow` stop one node early — worth deriving this from the trace below rather than memorizing "+1" as an arbitrary constant.

**Why the dummy head is necessary here specifically:** if the node to remove is the head itself (`n` equals the list's length), there's no "previous node" to redirect — without a dummy, this needs a special case (`if (n == length) return head.next;`) before the main logic even runs. With `dummy.next = head`, `slow` can legitimately end up *at* `dummy`, and `dummy.next = dummy.next.next` correctly reassigns what "the new head" is, uniformly, with no branch.

**Worked trace, `[1,2,3,4,5]`, `n = 2`** (remove the 2nd-from-end, which is `4`):

`fast` advances `n+1 = 3` steps from `dummy`: `dummy → 1 → 2 → 3`. Now walk both together until `fast` is `null`:

| step | fast | slow |
|---|---|---|
| start | 3 | dummy |
| 1 | 4 | 1 |
| 2 | 5 | 2 |
| 3 | null | 3 |

`slow` stops at node `3`. `slow.next` (`4`) is the target: `3.next = 5`. Result: `[1,2,3,5]` — `4` correctly removed.

**Complexity: Time O(L), Space O(1)**, where `L` is the list's length — one pass, versus Approach 1's two.

**Edge cases:**
- `n` equals the list's length (remove the head): `fast` advances past the end entirely during the initial `n+1` steps, becoming `null` before the second loop even starts — `slow` never moves off `dummy`, and `dummy.next = dummy.next.next` correctly drops the original head.
- `n = 1` (remove the tail): the gap positions `slow` exactly one node before the last — verified by the same trace mechanics above with a different `n`.
- Single-node list, `n = 1`: `dummy.next` becomes `null` — an empty list, correctly returned.

**💡 Interview Insight:** stating "I'll use a dummy head so I don't need a separate branch for removing the head" *before* writing the code is a strong, specific signal — it shows the special case was anticipated, not stumbled into. The problem explicitly asks for one pass; leading with the two-pass approach and then explicitly proposing the fixed-gap optimization (rather than jumping straight to the optimal answer) demonstrates the same reasoning-out-loud habit this series has built since Day 5.

---

### Problem 8: Add Two Numbers (LeetCode 2, Medium) — Pattern: Math Simulation on Linked List

**Statement:** Two non-empty linked lists represent non-negative integers, with digits stored in **reverse order** (the least-significant digit is the head). Add the two numbers and return the sum, also as a linked list in reverse-digit order.

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    int carry = 0;

    while (l1 != null || l2 != null || carry != 0) {
        int val1 = (l1 != null) ? l1.val : 0;
        int val2 = (l2 != null) ? l2.val : 0;
        int sum = val1 + val2 + carry;
        carry = sum / 10;
        current.next = new ListNode(sum % 10);
        current = current.next;

        if (l1 != null) l1 = l1.next;
        if (l2 != null) l2 = l2.next;
    }

    return dummy.next;
}
```

**Why reverse order actually makes this easier, not harder:** addition, done by hand, starts from the least-significant digit — exactly the order these lists are already in. There's no need to reverse anything first; walk both lists head-to-tail, and you're naturally processing digits from least to most significant, exactly matching how carrying works.

**Why the loop condition includes `carry != 0`:** both lists can be fully exhausted while a final carry still needs to produce one more digit (e.g., `9 + 1`). Missing this condition would silently drop that final digit.

**Why check for `null` before dereferencing `.val`, per list, independently:** the two lists can have different lengths — once the shorter one is exhausted, it contributes `0` to every remaining sum, letting the longer list's remaining digits pass through (plus any carry) without special-casing "list 1 ran out but list 2 didn't."

**Worked trace, `l1 = [2,4,3]` (342), `l2 = [5,6,4]` (465):**

| step | val1 | val2 | carry (in) | sum | digit out | carry (out) |
|---|---|---|---|---|---|---|
| 1 | 2 | 5 | 0 | 7 | 7 | 0 |
| 2 | 4 | 6 | 0 | 10 | 0 | 1 |
| 3 | 3 | 4 | 1 | 8 | 8 | 0 |

Result: `[7,0,8]` = 807 = 342 + 465. ✓

**Worked trace for the carry-overflow edge case, `l1 = [9,9]` (99), `l2 = [1]` (1):**

| step | val1 | val2 | carry (in) | sum | digit out | carry (out) |
|---|---|---|---|---|---|---|
| 1 | 9 | 1 | 0 | 10 | 0 | 1 |
| 2 | 9 | 0 (l2 exhausted) | 1 | 10 | 0 | 1 |
| 3 | 0 (both exhausted) | 0 | 1 | 1 | 1 | 0 |

Result: `[0,0,1]` = 100 = 99 + 1. ✓ — the third iteration only happens *because* the loop condition checks `carry != 0`, not just `l1 != null || l2 != null`.

**Complexity: Time O(max(m, n)), Space O(max(m, n))** — one pass across the longer list; the output list's length is bounded by the longer input's length plus at most one extra digit for a final carry.

**⚠️ Common Mistake:** forgetting the extra `carry != 0` condition and losing the final digit on inputs like `[9,9] + [1]`. This edge case is genuinely popular as an interview follow-up specifically because it's easy to miss on a first pass.

**💡 Interview Insight:** worth stating unprompted: if the digits were stored in *forward* order instead (most-significant first — the more "natural" way humans usually write numbers), this problem gets meaningfully harder, since you'd need to process from the tail backward, which a singly linked list can't do directly — that's exactly what LC 445 (Add Two Numbers II) asks, and it needs a stack to reverse the effective processing order. That problem is deferred until stacks are covered, later this week — worth a forward pointer now, full treatment later.

---

## Part 2 — Explicit Locks

### Prerequisites (confirmed)

- Threads, the JVM's stack-per-thread/shared-heap model, `Thread.State`'s six real values — Day 29.
- `Runnable` via anonymous inner class syntax — Day 29 (reused throughout today's code).

### Recap: what Day 29 showed, and what it deliberately didn't

Day 29's demo had multiple threads print messages, and showed the *order* those prints appeared in was different every run — genuinely non-deterministic, but never *wrong*: every message printed exactly once, nothing was lost or duplicated. Today's demo is different in kind: it produces an outright incorrect final number, reliably, because of what unsynchronized *shared, mutable* state actually does under concurrent modification.

### Why `count++` is not one operation

```java
count++;
```

This single line of Java source compiles down to three distinct steps at the bytecode level:

1. **Read** the current value of `count` from memory.
2. **Increment** that value in a local register.
3. **Write** the incremented value back to `count`.

Nothing about the JVM guarantees these three steps happen *together*, uninterrupted, when multiple threads are involved. Another thread can run its own read/increment/write in between any of these three steps of the first thread's.

### The lost-update trace — proof, not assertion

Two threads, A and B, both execute `count++` starting from `count = 0`. The *correct* final result, if these ran one at a time, would be `2`.

| time | Thread A | Thread B | `count` in memory |
|---|---|---|---|
| t1 | reads `count` → local = 0 | | 0 |
| t2 | | reads `count` → local = 0 | 0 |
| t3 | computes local = 0 + 1 = 1 | | 0 |
| t4 | | computes local = 0 + 1 = 1 | 0 |
| t5 | writes local (1) → `count` | | **1** |
| t6 | | writes local (1) → `count` | **1** |

Final `count = 1`. Two increments happened; only one is reflected. **Thread B's write silently overwrote Thread A's** — not because anything crashed or threw an exception, but because both threads read the *same* starting value before either had written back. This is a **lost update**, and it's exactly the failure mode Day 29's demo never produced.

**Why more threads make this worse, not better:** with only 2 threads, this exact interleaving might not happen on every run — the JVM's thread scheduling is non-deterministic, so sometimes A fully finishes before B starts, and the result comes out correct by luck. With 100 threads each incrementing 1000 times (100,000 total increments), the sheer number of possible interleavings makes *some* lost update virtually certain on every run — and the final count will typically be different across separate runs of the identical program, which is itself a strong signal that something is wrong.

```java
public class RaceConditionDemo {
    private static int unsafeCount = 0;

    public static void main(String[] args) throws InterruptedException {
        final int numThreads = 100;
        final int incrementsPerThread = 1000;
        Thread[] threads = new Thread[numThreads];

        for (int i = 0; i < numThreads; i++) {
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < incrementsPerThread; j++) {
                        unsafeCount++;   // NOT atomic
                    }
                }
            });
        }

        for (Thread t : threads) t.start();
        for (Thread t : threads) t.join();

        System.out.println("Expected: " + (numThreads * incrementsPerThread));
        System.out.println("Actual:   " + unsafeCount);   // reliably LESS — and different every run
    }
}
```

### The classic fix: `synchronized`

Every Java object carries an **intrinsic lock** (also called a *monitor*) — a piece of built-in JVM machinery, one per object, that at most one thread can "hold" at a time.

```java
public class SynchronizedCounter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```

A `synchronized` instance method acquires the intrinsic lock on `this` before it runs, and releases it automatically when the method returns — **whether it returns normally or via an exception.** That automatic release, guaranteed by the JVM regardless of how the method exits, is `synchronized`'s biggest practical convenience. A `synchronized` *block* (`synchronized (someObject) { ... }`) does the same thing but lets you lock on any chosen object, and only around part of a method rather than the whole thing.

**⚠️ Common Misconception, worth correcting directly:** `synchronized` locks are **already reentrant** — a thread that already holds an object's intrinsic lock can call another `synchronized` method on that same object (including recursively) without deadlocking itself; the JVM tracks a hold count internally and only truly releases the lock once it drops back to zero. `ReentrantLock`'s name does *not* mean it invented reentrancy — it's naming a property `synchronized` already had, while adding genuinely new capabilities elsewhere.

### `ReentrantLock` — what's actually new

```java
public class Counter {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }

    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
```

`ReentrantLock` (from `java.util.concurrent.locks`) implements the `Lock` interface, requiring explicit `lock()`/`unlock()` calls rather than a keyword. What it actually adds over `synchronized`:

| Capability | `synchronized` | `ReentrantLock` |
|---|---|---|
| Reentrant (same thread re-acquires without self-deadlock) | Yes | Yes — **not a new capability**, despite the name |
| Automatic release, even on exception | Yes, guaranteed by the JVM | **No** — a missed `unlock()` (e.g., no `finally`) holds the lock forever |
| Attempt without blocking (`tryLock()`), optionally with a timeout | No | Yes |
| Configurable fairness (roughly FIFO among waiting threads) | No — JVM picks any waiting thread | Yes — `new ReentrantLock(true)` |
| Interruptible while waiting (`lockInterruptibly()`) | No | Yes |
| Multiple independent wait-sets on one lock (`Condition` objects) | No — one implicit wait-set per object | **Yes** — needed for tomorrow's Producer-Consumer |

**⚠️ Common Mistake — this is the real cost of the added flexibility:** because release is no longer automatic, `lock.lock()` **must** be followed by a `try { ... } finally { lock.unlock(); }`. If an exception is thrown between `lock()` and a bare `unlock()` call placed after the risky code (rather than in a `finally`), the lock is never released — every other thread calling `lock()` on that same lock blocks *forever*. This isn't a style preference; it's a correctness requirement `synchronized` simply doesn't need, because the JVM enforces release for you.

```java
public class SafeCounterDemo {
    public static void main(String[] args) throws InterruptedException {
        final int numThreads = 100;
        final int incrementsPerThread = 1000;
        Counter counter = new Counter();
        Thread[] threads = new Thread[numThreads];

        for (int i = 0; i < numThreads; i++) {
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < incrementsPerThread; j++) {
                        counter.increment();
                    }
                }
            });
        }

        for (Thread t : threads) t.start();
        for (Thread t : threads) t.join();

        System.out.println("Expected: " + (numThreads * incrementsPerThread));
        System.out.println("Actual:   " + counter.getCount());   // ALWAYS exactly matches, every run
    }
}
```

**Why this is now correct:** `lock.lock()` ensures only one thread at a time can be between the `lock()` and `unlock()` calls — the three-step read/increment/write sequence can no longer be interrupted by another thread's own read/increment/write, because no other thread can even *enter* `increment()`'s critical section until the current holder calls `unlock()`. The lost-update interleaving from the trace above is now structurally impossible.

**🔑 Key Takeaway:** locking doesn't make threads run faster — it makes a specific *section of code* run one-thread-at-a-time, trading some parallelism for correctness. That trade-off is inherent to mutual exclusion itself, not a flaw specific to `ReentrantLock` — it's worth naming explicitly if asked "does adding a lock have a cost," since the honest answer is yes, and the cost is reduced concurrency on the locked section specifically.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** write a `Counter` class using `ReentrantLock` (as above); confirm correctness under 100 concurrent threads.

**Practical steps:** implement `Counter` exactly as shown. Write a small driver (`SafeCounterDemo` above, or your own equivalent) that spins up 100 threads, each incrementing 1000 times, joins them all, and asserts the final count equals exactly 100,000. Run it several times in a row — the point of the exercise is confirming the result is **identical every single run**, not just correct once. As a side-by-side sanity check worth doing once (not part of the graded deliverable), run the unsafe version too, and observe that its final count varies between runs and is reliably less than 100,000.

**Definition of done:** pushed, verified correct under concurrent load — meaning you've actually run it multiple times and confirmed the count is exactly right every time, not just once.

## Career Block Guide (1 hr)

**LinkedIn (20 min):** engagement — comment on 3–5 posts.

**Networking:** send 3 connection requests to engineers whose blog posts you've actually read. The note should reference something specific from the post — what you found useful, disagreed with, or wanted to ask about — the same "shows you actually looked" standard from Day 10 applies here.

---

## Day 37 — Interview Questions

**Q1. Why does Remove Nth Node From End need a gap of `n + 1`, not `n`?** `slow` needs to stop one node *before* the target so `slow.next = slow.next.next` can perform the removal; leading `fast` by exactly `n` would land `slow` directly on the target instead of just before it.

**Q2. Why is a dummy head necessary for Remove Nth Node From End specifically?** If the node to remove is the head itself, there's no preceding node to redirect without one — the dummy gives every case, including removing the head, a uniform predecessor to operate on.

**Q3. In Add Two Numbers, why does the loop need to check `carry != 0` in addition to both lists being non-null?** Both lists can be fully exhausted while a final carry still needs to produce one more output digit (e.g., 99 + 1) — omitting this check would silently drop that digit.

**Q4. Why is reverse-digit order actually convenient for Add Two Numbers, rather than an obstacle?** Addition by hand starts from the least-significant digit, which is exactly where these lists already begin — no reversal is needed before summing.

**Q5. Why isn't `count++` atomic?** It compiles to three separate steps — read the current value, increment it in a local register, write it back — and nothing prevents another thread's own read/increment/write from interleaving between any of those three steps.

**Q6. Walk through a concrete interleaving that causes a lost update on a shared counter.** Two threads both read `count = 0` before either writes back; both compute `1`; both write `1` — one of the two increments is silently lost, and the final value is `1` instead of the correct `2`.

**Q7. Is `ReentrantLock` reentrant in a way `synchronized` is not?** No — this is a common misconception. `synchronized`'s intrinsic locks have always been reentrant; a thread can already re-acquire its own held lock without self-deadlocking. `ReentrantLock`'s real advantages are elsewhere: `tryLock()`/timeouts, configurable fairness, interruptible acquisition, and multiple `Condition` objects per lock.

**Q8. What's the single biggest correctness risk `ReentrantLock` introduces that `synchronized` doesn't have?** Release is no longer automatic — a missed `unlock()` call (e.g., an exception thrown before an `unlock()` that isn't in a `finally` block) holds the lock forever, blocking every other thread waiting on it. `synchronized` guarantees release on any method exit, including via exception.

**Q9. Does locking make a program run faster?** No — it makes a specific critical section execute one thread at a time, which is a deliberate trade of some parallelism for correctness, not a performance optimization.

**Q10. Why does running the unsafe counter demo multiple times produce a different final count each time, but the locked version doesn't?** Thread scheduling is non-deterministic, so which specific interleavings of read/increment/write actually occur — and how many updates get lost — varies run to run. The locked version structurally prevents any interleaving inside the critical section, so its result is deterministic every time.

---

## Daily Deliverable Check

- [ ] Remove Nth Node From End (LC 19) and Add Two Numbers (LC 2) solved, both traced through by hand, pushed to `dsa-java/linked-lists/`.
- [ ] Can reproduce the lost-update interleaving trace from memory, and explain why `count++` isn't atomic in terms of the three underlying steps.
- [ ] Can state precisely what `ReentrantLock` adds over `synchronized` — without claiming reentrancy itself is new.
- [ ] `Counter` (using `ReentrantLock`) pushed to `java-fundamentals`, run multiple times under 100 concurrent threads with a consistently correct final count.
- [ ] LinkedIn engagement done. 3 connection requests sent, each referencing a specific post.

---

## What Tomorrow Assumes You Already Know Cold

Day 38 assumes `ReentrantLock`'s `lock()`/`unlock()`/`try`-`finally` pattern is fully reflexive, since tomorrow's `Condition` objects (`lock.newCondition()`) are created *from* a `ReentrantLock` — today's mechanism is a direct, non-optional prerequisite, not just related material. It also assumes today's two pointer-manipulation problems didn't need any new mechanism beyond what Days 34–36 already established — tomorrow's Reorder List explicitly combines three techniques you now have in full (finding a middle, reversing, and merging), with no new pointer mechanic introduced to do it.
