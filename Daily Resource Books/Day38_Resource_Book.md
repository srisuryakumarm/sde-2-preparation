# Day 38 — Linked Lists: Advanced Combinations, and Wait/Notify

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 37 Resource Book](Day37_Resource_Book.md)
**Next ▶:** [Day 39 Resource Book](Day39_Resource_Book.md)
**Companion to:** Day 38 of `Week_06_Revised.md`

---

## Recap

Today's first problem, Reorder List, doesn't introduce a single new pointer mechanic — it combines three you already have in full: finding a middle (Day 34), reversing a list (Day 34), and merging (Day 35's dummy-head merge, adapted here to an alternating interleave rather than a sorted merge). The second problem, Copy List with Random Pointer, reuses HashMap (long since second nature) against a linked-list node shape you haven't seen yet — a second pointer field, `random`, that can point anywhere in the list rather than always forward.

On the theory side, yesterday's `ReentrantLock` solved *mutual exclusion* — making sure only one thread touches shared state at a time. Today solves a different problem: what does a thread do when it's *allowed* to touch the shared state, but the state isn't in the right shape yet — a producer with a full buffer, a consumer with an empty one? Spinning in a loop, repeatedly re-checking, wastes CPU. Today's `wait()`/`notify()` and `Condition` let a thread step aside and sleep until something worth re-checking has actually happened, then resumes exactly the safe, single-threaded-at-a-time execution yesterday's lock already guarantees.

---

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Solve Reorder List by explicitly naming which of the three prior techniques each step reuses, with no new mechanism introduced.
2. Explain why the HashMap approach to Copy List with Random Pointer correctly handles a `random` pointer that is `null`, without an explicit `null` check in the wiring step.
3. State exactly why `wait()` must be called inside a `while` loop re-checking its condition, never an `if`, with two independent reasons.
4. Explain what a `Condition` object gives you that a single `synchronized` object's built-in wait-set does not, and why that matters specifically for a producer/consumer with two different reasons to wait.

---

## Concept Dependency Map

```
Day 34: find middle (fast/slow)   Day 34: reverse a list   Day 35: dummy-head merge
        │                                  │                        │
        └──────────────┬───────────────────┴────────────┬───────────┘
                        ▼                                ▼
              Today: Reorder List (LC 143) — same three techniques,
              chained: split at middle → reverse second half → merge
              by ALTERNATING (not by sorted order)

Day 4-15: HashMap                Today, NEW node shape:
        │                        a `random` pointer alongside `next`,
        │                        pointing anywhere in the list (or null)
        └──────────────┬─────────────────┘
                        ▼
              Today: Copy List with Random Pointer (LC 138)
              old-node → new-node map, built in one pass,
              wired together in a second pass

─────────────────────────────────────────────────────────

Day 37: ReentrantLock — mutual exclusion (one thread at a time)
        │
        ▼
Today, NEW: wait()/notify()/notifyAll() — a thread can step aside
        and sleep until conditions change, instead of spinning
        │
        ▼
Today, NEW: Condition (lock.newCondition()) — the ReentrantLock-based
        equivalent, with a KEY upgrade: multiple independent wait-sets
        from ONE lock
        │
        ▼
Project: Producer-Consumer, using TWO Conditions (notFull, notEmpty)
        — needs yesterday's ReentrantLock directly, not optionally
```

---

## Part 1 — Linked Lists: Combining Techniques

### Problem 9: Reorder List (LeetCode 143, Medium) — Pattern: Fast/Slow + Reversal + Merge

**Statement:** Given a list `L0 → L1 → … → Ln`, reorder it in place to `L0 → Ln → L1 → Ln-1 → L2 → Ln-2 → …`. No new nodes; only pointer rewiring.

**The plan, stated before any code — this is the answer to "how would you approach this":** split the list at its middle, reverse the second half, then merge the two halves by alternating one node from each — three techniques you already have, applied back to back.

```java
public void reorderList(ListNode head) {
    if (head == null || head.next == null) return;

    // Step 1 — find the middle (Day 34's fast/slow)
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // Step 2 — cut the list in two, reverse the second half (Day 34's reversal)
    ListNode secondHalfStart = slow.next;
    slow.next = null;
    ListNode prev = null, curr = secondHalfStart;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    ListNode secondHalfReversed = prev;

    // Step 3 — merge by ALTERNATING (not by sorted value, unlike Day 35's merge)
    ListNode first = head, second = secondHalfReversed;
    while (second != null) {
        ListNode firstNext = first.next;
        ListNode secondNext = second.next;
        first.next = second;
        second.next = firstNext;
        first = firstNext;
        second = secondNext;
    }
}
```

**Why the middle-finding boundary (`fast.next != null && fast.next.next != null`) gives the *first* half the extra node on odd-length lists:** for `n` nodes, this condition stops `slow` at position `⌈n/2⌉` rather than `⌊n/2⌋` — worth confirming by trace, not just trusting, since an off-by-one here silently produces a wrong split.

**Worked trace, `[1,2,3,4,5]`** (odd length — the more interesting case):

*Find middle:* `slow` ends at node `3`. Split: first half `[1,2,3]`, second half (pre-reversal) `[4,5]`.

*Reverse second half:* `[5,4]`.

*Merge, alternating:*

| step | first (before) | second (before) | action | first (after) | second (after) |
|---|---|---|---|---|---|
| 1 | 1 | 5 | `1.next=5`, `5.next=2` | 2 | 4 |
| 2 | 2 | 4 | `2.next=4`, `4.next=3` | 3 | null |

Loop ends (`second == null`). Final chain: `1 → 5 → 2 → 4 → 3 → null`.

**Verify against the target pattern** `L0, Ln, L1, Ln-1, L2` for `n=5` (0-indexed, `L0..L4`): `L0=1, L4=5, L1=2, L3=4, L2=3` → `1, 5, 2, 4, 3`. **Matches exactly** — and node `3` (the unpaired middle element on this odd-length list) correctly ends up last, with nothing appended after it, because `second` ran out one iteration before `first` did.

**Complexity: Time O(n) — three sequential linear passes, not nested. Space O(1)** — every step rewires existing nodes; nothing new is allocated.

**Edge cases:**
- 0 or 1 nodes: the initial guard returns immediately — already trivially "reordered."
- 2 nodes: the middle-finding loop never executes even once (`fast.next.next` is `null` on the very first check), so `slow` stays at the head, and the two-node list is correctly left as-is (there's only one way to interleave two elements).

**💡 Interview Insight:** naming this as "three known techniques, chained" *before* writing any code is the single strongest thing to say here — it signals you recognized the composition rather than needing to invent a fourth, novel idea. If asked to justify the O(1) space claim specifically, the answer is that every step operates by reassigning `.next` pointers on nodes that already exist — nothing is copied or duplicated at any stage.

---

### Problem 10: Copy List with Random Pointer (LeetCode 138, Medium) — Pattern: HashMap for Node Mapping

**Statement:** Each node has a `next` pointer (as usual) *and* a `random` pointer, which can point to **any** node in the list, or to `null`. Return a fully independent deep copy — new node objects throughout, with `next` and `random` correctly mirroring the original's structure.

#### Approach 1 — Brute force: locate by index

```java
public Node copyRandomListBruteForce(Node head) {
    if (head == null) return null;

    Node dummy = new Node(0);
    Node newCurrent = dummy, oldCurrent = head;
    while (oldCurrent != null) {                 // clone the `next` chain only, first
        newCurrent.next = new Node(oldCurrent.val);
        newCurrent = newCurrent.next;
        oldCurrent = oldCurrent.next;
    }

    oldCurrent = head;
    newCurrent = dummy.next;
    while (oldCurrent != null) {
        if (oldCurrent.random != null) {
            int index = 0;
            for (Node scan = head; scan != oldCurrent.random; scan = scan.next) {
                index++;                          // find the ORIGINAL index of the random target
            }
            Node target = dummy.next;
            for (int i = 0; i < index; i++) target = target.next;  // walk that far in the NEW list
            newCurrent.random = target;
        }
        oldCurrent = oldCurrent.next;
        newCurrent = newCurrent.next;
    }
    return dummy.next;
}
```

Clone the `next` chain first, ignoring `random` entirely. Then, for every node, find its `random` target's *position* in the original list by linear scan, and walk that many steps into the new list to find the matching copy. **Time O(n²)** — for each of `n` nodes, up to O(n) to locate the target's index plus O(n) to walk to it. **Space O(n)** for the new list itself.

#### Approach 2 — Optimized: HashMap, two passes

```java
public Node copyRandomList(Node head) {
    if (head == null) return null;

    Map<Node, Node> oldToNew = new HashMap<>();

    Node current = head;
    while (current != null) {                    // Pass 1 — create every new node, map old → new
        oldToNew.put(current, new Node(current.val));
        current = current.next;
    }

    current = head;
    while (current != null) {                     // Pass 2 — wire next and random using the map
        Node newNode = oldToNew.get(current);
        newNode.next = oldToNew.get(current.next);
        newNode.random = oldToNew.get(current.random);
        current = current.next;
    }

    return oldToNew.get(head);
}
```

**The trick that removes the O(n²) scan entirely:** instead of *searching* for where a `random` target lives, look it up directly — the map built in Pass 1 already knows, for any old node, exactly which new node corresponds to it, in O(1).

**Why `newNode.next = oldToNew.get(current.next)` needs no explicit `null` check:** when `current.next` is `null` (the last node), `oldToNew.get(null)` is called. `HashMap` permits a `null` key, and since `put(null, ...)` was never called, `get(null)` correctly returns `null` — `newNode.next` is set to `null` exactly when it should be, with no branch needed. The identical reasoning covers `current.random` being `null`.

**Worked trace**, a 3-node list `A(val=7) → B(val=13) → C(val=11)`, with `B.random = A` and `A.random = C.random = null`:

*Pass 1:* `oldToNew = {A: A′, B: B′, C: C′}` (primes denote the new copies; fields not yet wired).

*Pass 2:*

| current | `newNode.next = map.get(current.next)` | `newNode.random = map.get(current.random)` |
|---|---|---|
| A | `map.get(B) = B′` | `map.get(null) = null` |
| B | `map.get(C) = C′` | `map.get(A) = A′` |
| C | `map.get(null) = null` | `map.get(null) = null` |

Result: `A′ → B′ → C′ → null`, with `B′.random = A′` — the structure is mirrored exactly, using entirely new node objects.

**Complexity: Time O(n) — two linear passes. Space O(n)** — the map holds one entry per node.

**Edge cases:**
- `head == null`: returns `null` immediately.
- A node's `random` points to *itself*: handled automatically — `oldToNew.get(current)` (looking up the node currently being processed) is already in the map from Pass 1, correctly producing a self-referencing copy.
- Every `random` is `null`: reduces to a plain list clone — no special-casing needed, since the `null`-key lookup already handles it.

**💡 Interview Insight:** worth mentioning as a follow-up, even though it's beyond today's required depth: an O(1)-extra-space version exists, by *interleaving* copied nodes directly into the original list (`A → A′ → B → B′ → …`), which makes every copy's `random` reachable as `original.random.next` without any map at all — then a final pass un-interleaves the two lists. It's a genuinely elegant trick, but the two-pass HashMap version above is the expected default answer; volunteering that the O(1) version exists (without necessarily implementing it live) is a strong signal of depth beyond what was asked.

---

## Part 2 — Wait/Notify, and `Condition`

### Prerequisites (confirmed)

- `ReentrantLock`, `lock()`/`unlock()`, the `try`/`finally` pattern — Day 37, needed directly (not just conceptually) for `Condition` below.
- `synchronized`, intrinsic locks — Day 37.

### The problem mutual exclusion alone doesn't solve

A lock guarantees only one thread touches shared state at a time — but it says nothing about *when* a thread should act. Consider a bounded buffer: a producer that finds the buffer full has nothing useful to do until a consumer removes something. One bad option is **busy-waiting** — a loop that just keeps re-checking `while (buffer.isFull()) { }` — which burns CPU cycles continuously for no benefit, and is even more wasteful the longer the wait. `wait()`/`notify()` exist to let a thread genuinely sleep — consuming no CPU at all — until there's an actual reason to re-check.

### `wait()`, `notify()`, `notifyAll()`

Every object's intrinsic lock (Day 37) comes with an associated **wait set** — a place threads can register themselves as "waiting on this object." `wait()`, `notify()`, and `notifyAll()` are methods on `Object` itself (every object has them), and they **must** be called from within a `synchronized` block or method on that same object, or the JVM throws `IllegalMonitorStateException`.

- **`wait()`** releases the object's lock and suspends the calling thread, adding it to the wait set. When later woken, the thread **re-acquires the lock** before `wait()` returns and execution continues.
- **`notify()`** wakes exactly one thread from the wait set — which one is unspecified.
- **`notifyAll()`** wakes every thread in the wait set; they don't all resume at once, though — each has to reacquire the (single) lock in turn, one at a time, before actually continuing.

```java
public class SimpleBoundedBuffer {
    private final Queue<Integer> buffer = new LinkedList<>();
    private final int capacity;

    public SimpleBoundedBuffer(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void put(int value) throws InterruptedException {
        while (buffer.size() == capacity) {
            wait();
        }
        buffer.add(value);
        notifyAll();
    }

    public synchronized int take() throws InterruptedException {
        while (buffer.isEmpty()) {
            wait();
        }
        int value = buffer.poll();
        notifyAll();
        return value;
    }
}
```

**⚠️ Common Mistake — the single most important rule in this section: always call `wait()` inside a `while` loop re-checking the actual condition, never an `if`.** Two independent reasons this matters, either one sufficient on its own:

1. **Spurious wakeups are real.** The JVM specification explicitly permits a waiting thread to wake up *without* any `notify()`/`notifyAll()` call at all — rare, but a documented possibility, not a hypothetical. Code that assumes "if I woke up, my condition must be true" is wrong on that assumption alone.
2. **`notifyAll()` wakes threads for whom the change is irrelevant.** In `SimpleBoundedBuffer` above, a single object's wait set mixes producers *and* consumers together — there's no way to wake "just the consumers." When `put()` calls `notifyAll()`, every waiting thread wakes, including other producers who have no reason to proceed. Each one **must** re-check its own condition (`while (buffer.size() == capacity)`) upon waking, find it's still true if it's an unrelated producer, and call `wait()` again — an `if` would incorrectly barrel ahead based on stale information.

### `Condition` — multiple wait-sets from one lock

`SimpleBoundedBuffer`'s real inefficiency: producers and consumers share *one* wait set, so `notifyAll()` necessarily wakes threads that have no reason to proceed, just so the ones that *do* have a reason are guaranteed to be among them. `Condition` (from `java.util.concurrent.locks`, created via `lock.newCondition()`) fixes this by letting **one `Lock` have multiple, independent wait-sets** — `await()`/`signal()`/`signalAll()` are the direct equivalents of `wait()`/`notify()`/`notifyAll()`, but scoped to only the threads waiting on that *specific* `Condition` object.

```java
public class BoundedBuffer {
    private final Queue<Integer> buffer = new LinkedList<>();
    private final int capacity;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public BoundedBuffer(int capacity) {
        this.capacity = capacity;
    }

    public void put(int value) throws InterruptedException {
        lock.lock();
        try {
            while (buffer.size() == capacity) {
                notFull.await();          // ONLY producers ever wait here
            }
            buffer.add(value);
            notEmpty.signal();            // wakes ONLY a consumer, never another producer
        } finally {
            lock.unlock();
        }
    }

    public int take() throws InterruptedException {
        lock.lock();
        try {
            while (buffer.isEmpty()) {
                notEmpty.await();         // ONLY consumers ever wait here
            }
            int value = buffer.poll();
            notFull.signal();             // wakes ONLY a producer, never another consumer
            return value;
        } finally {
            lock.unlock();
        }
    }
}
```

**Why `signal()` (waking just one) is safe here, when plain `notify()` on a shared wait-set is generally discouraged:** every thread waiting on `notEmpty` is, by construction, a consumer — `notFull.await()` is never called by a consumer, and `notEmpty.await()` is never called by a producer. Since exactly one item became available, waking exactly one consumer is both safe and sufficient — there's no risk of waking an unrelated thread that has nothing useful to do, which is precisely the risk `notify()` on a single, mixed-purpose wait set carries.

**🔑 Key Takeaway:** the `while`-not-`if` rule from above still applies identically here — `await()` must still be re-checked in a loop, for the same two reasons (spurious wakeups remain possible, and even a same-purpose `signalAll()` can wake more threads than the number of newly-available slots).

**💡 Interview Insight, worth having ready:** "how can two locks deadlock" is a very standard follow-up in this territory, even though today's code only uses one lock. The classic answer: Thread A holds Lock 1 and waits for Lock 2, while Thread B holds Lock 2 and waits for Lock 1 — neither can proceed, and neither will ever release what it holds. The general fix is to always acquire multiple locks in a single, globally consistent order across every thread.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** implement Producer-Consumer using `ReentrantLock` and two `Condition`s (`notFull`, `notEmpty`), as above.

```java
public class ProducerConsumerDemo {
    public static void main(String[] args) throws InterruptedException {
        final BoundedBuffer buffer = new BoundedBuffer(5);

        Thread producer = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    for (int i = 1; i <= 20; i++) {
                        buffer.put(i);
                        System.out.println("Produced: " + i);
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();   // restore the interrupt status
                }
            }
        });

        Thread consumer = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    for (int i = 1; i <= 20; i++) {
                        int value = buffer.take();
                        System.out.println("Consumed: " + value);
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

**Why this can't deadlock:** a producer only ever waits on `notFull` (buffer full), a consumer only ever waits on `notEmpty` (buffer empty) — since the capacity is greater than zero, the buffer can never be simultaneously full *and* empty, so it's never possible for both threads to be stuck waiting at once. Every `put()` that succeeds signals `notEmpty`; every `take()` that succeeds signals `notFull` — progress always eventually unblocks the other side. **Why no item is lost or duplicated:** every access to `buffer` happens while holding `lock` — between a `put()`'s `buffer.add(value)` and the matching `take()`'s `buffer.poll()`, no other thread can touch `buffer` at all, so exactly the items added are the items later removed, each exactly once.

**Practical steps:** implement `BoundedBuffer` exactly as above, capacity 5. Run the driver and confirm all 20 items are produced *and* consumed, with the console interleaving looking genuinely concurrent (produced/consumed lines interleaving, not strictly alternating or strictly batched) across multiple runs.

**Definition of done:** pushed, runs without deadlocking or losing items — confirmed by actually running it more than once, the same discipline as yesterday's concurrency check.

## Career Block Guide (1 hr)

**LinkedIn:** **Post 9** — "I made a HashMap lose track of an object on purpose," a callback to Week 3's `hashCode`/`equals` contract demo. Draft to adapt in your own voice:

> Revisited an old demo today: a `HashMap` genuinely losing track of an object because its `hashCode()` and `equals()` disagreed with each other — mutate the fields `equals()`/`hashCode()` depend on *after* insertion, and the map can no longer find the entry, even though the object is still sitting right there in the bucket array. Coming back to it after weeks of linked lists and concurrency was a good reminder that the early fundamentals are the ones that keep resurfacing.

Adapt the specifics to whatever your Week 3 demo actually showed — the point is a genuine callback, not this exact wording.

**Networking:** identify 5 Target Tier B companies. Use whatever criteria you've been applying to Tier A so far (company size, tech stack fit, compensation band), scaled down one notch — Tier B should still be a genuine stretch, not a fallback.

---

## Day 38 — Interview Questions

**Q1. Walk through Reorder List's approach and name which prior technique each step reuses.** Find the middle via fast/slow (Day 34); split and reverse the second half (Day 34's reversal); merge by alternating one node from each half (Day 35's dummy-head merge, adapted from sorted-order merging to alternation).

**Q2. Why does an odd-length list's unpaired middle node correctly end up last in Reorder List, with no special case?** The middle-finding split gives the first half one extra node on odd lengths; during the alternating merge, the second half (the reversed tail) runs out one iteration before the first half does, so the first half's final extra node is simply appended last, with the loop ending as soon as `second` is `null`.

**Q3. In Copy List with Random Pointer, why does `oldToNew.get(current.next)` work correctly even when `current.next` is `null`?** `HashMap` allows a `null` key; since `put(null, ...)` was never called, `get(null)` returns `null` — exactly the desired result when a node has no next node, with no explicit branch needed.

**Q4. What makes the brute-force approach to Copy List with Random Pointer O(n²) specifically?** For each of `n` nodes, finding its `random` target's position requires an O(n) scan of the original list, and then walking that far into the new list costs another O(n) — O(n) work per node, O(n²) total.

**Q5. Why must `wait()` be called inside a `while` loop, never an `if`?** Two independent reasons: spurious wakeups are a documented JVM-level possibility with no `notify()` involved at all, and `notifyAll()` can wake threads whose specific condition still doesn't hold (e.g., another producer, when only consumers should proceed) — both require re-checking the actual condition after waking, not assuming it's now true.

**Q6. What does `Condition` provide that `wait()`/`notify()` on a single object does not?** Multiple independent wait-sets from one lock — `lock.newCondition()` can be called more than once, letting semantically different groups of waiting threads (producers vs. consumers) be signaled separately, rather than every waiter sharing one wait-set and having to individually re-check whether a given wakeup was even relevant to them.

**Q7. Why is `notFull.signal()` safe to use instead of `signalAll()` in the `BoundedBuffer` implementation?** Every thread waiting on `notFull` is, by construction, a producer — there's no risk of waking an unrelated thread, since only producers ever call `notFull.await()`. Exactly one slot became available, so waking exactly one producer is sufficient.

**Q8. Must `wait()`/`notify()` and `Condition.await()`/`signal()` be called while holding the associated lock?** Yes — for `wait()`/`notify()`, the calling thread must hold the object's intrinsic lock (i.e., be inside a `synchronized` block/method on that object) or the JVM throws `IllegalMonitorStateException`; `Condition`'s methods carry the identical requirement relative to the `Lock` that created them.

**Q9. Describe a scenario where two locks deadlock.** Thread A holds Lock 1 and is waiting to acquire Lock 2; Thread B holds Lock 2 and is waiting to acquire Lock 1. Neither can make progress, and neither releases what it already holds — resolved in general by always acquiring multiple locks in one consistent order across every thread.

---

## Daily Deliverable Check

- [ ] Reorder List (LC 143) and Copy List with Random Pointer (LC 138) solved, pushed to `dsa-java/linked-lists/`.
- [ ] Can explain Reorder List as three known techniques chained, with no new mechanism, from memory.
- [ ] Can state both reasons `wait()`/`await()` must be re-checked in a `while` loop, not an `if`.
- [ ] Producer-Consumer (`BoundedBuffer`, `ReentrantLock` + two `Condition`s) pushed to `java-fundamentals`, run multiple times with all 20 items produced and consumed, no deadlock.
- [ ] LinkedIn Post 9 published. 5 Target Tier B companies identified.

---

## What Tomorrow Assumes You Already Know Cold

Tomorrow's LRU Cache needs today's HashMap-for-O(1)-lookup fluency directly, plus a genuinely new structure — a **doubly** linked list, which tomorrow introduces from scratch before using it (today and every prior linked-list day used only a single `next` pointer). Linked Lists close out tomorrow at 11 problems; today's combination-of-techniques mindset (Reorder List) is exactly the mindset LRU Cache needs one more time, at slightly higher stakes. On the concurrency side, tomorrow's `ConcurrentHashMap` theory assumes `synchronized` and `ReentrantLock` are both solid, since the comparison between a single coarse-grained lock and fine-grained internal locking only makes sense once "one lock guarding everything" is a concrete, already-understood baseline — which Day 37's `Counter` and today's `BoundedBuffer` have now both demonstrated directly.
