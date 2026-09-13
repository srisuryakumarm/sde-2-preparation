# Day 39 — Linked Lists Capstone, Stacks Begin, and ConcurrentHashMap

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 38 Resource Book](Day38_Resource_Book.md)
**Next ▶:** [Day 40 Resource Book](Day40_Resource_Book.md)
**Companion to:** Day 39 of `Week_06_Revised.md`

---

## ⚠️ Overlap Notice — Read This First

Today's required Stack problem has a full overlap with earlier material — flagged in `00_Curriculum_Map.md` since Week 1 and carried forward through every intervening weekly check, finally landing on the week it actually matters for.

| LC # | Problem | Status |
|---|---|---|
| 20 | Valid Parentheses | Already solved — Week 1, Day 4 (Required). **Recap only, below.** |
| 146 | LRU Cache | Genuinely new — closes Linked Lists. **Full depth below.** |

No substitute problem is added today. Per this series' own established precedent — Linked Lists on Day 34, Binary Search on Day 28, and every pattern's opening day before it — a freshly-opening pattern doesn't get bonus practice on the day it opens. Today is Stacks' opening day; the time freed by recapping Valid Parentheses goes toward LRU Cache and the doubly linked list prerequisite it needs, not toward an extra Stack problem.

---

## Recap

Today closes Linked Lists — 11 required problems, Easy through Medium, spanning Day 34 through today. Every technique it needed is already in hand: fast/slow pointers, reversal, dummy heads, HashMap-based node mapping. LRU Cache needs exactly one more building block first — a **doubly** linked list, since everything so far has used only a single `next` pointer, and LRU Cache's O(1) removal specifically depends on a node knowing its own predecessor without a traversal to find it.

Valid Parentheses, today's other required problem, was already solved back in Week 1, Day 4 — before "Stack" had been named as a formal pattern the way HashMap/HashSet or Two Pointers were. Today gives Stacks that same formal treatment, using the exact `ArrayDeque`-as-stack approach you've already been using since that day.

On the concurrency track, yesterday's `Condition` showed fine-grained *coordination*. Today's `ConcurrentHashMap` shows fine-grained *locking* — the same "don't make every thread contend for one shared resource" instinct, applied to a `Map` instead of a bounded buffer.

---

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Explain why a doubly linked list makes O(1) removal-of-a-known-node possible, when a singly linked list cannot do this without a traversal.
2. Design and implement LRU Cache from its two required capabilities (O(1) lookup, O(1) recency-tracking) rather than recalling the HashMap+DLL combination as a memorized shape.
3. State the LIFO property and recognize the "most recent unresolved thing" signal that means a problem wants a stack.
4. Explain, precisely, what `ConcurrentHashMap` locks (and doesn't) compared to `Collections.synchronizedMap()`, including what CAS is and why it avoids locking entirely for some operations.

---

## Concept Dependency Map

```
Day 34–38: singly linked lists — node has ONLY `next`
        │
        ▼
Today, NEW: doubly linked list — node ALSO has `prev`
  → removing a KNOWN node becomes O(1): no traversal needed
    to find its predecessor, since node.prev already IS it
        │
        ▼
Today, NEW: dummy head AND dummy tail (extends Day 35's single
  dummy head) — both ends of the real list are always adjacent
  to a permanent sentinel, so insert/remove-at-either-end never
  needs a null check for "is the list currently empty"
        │
        ▼
Today: LRU Cache (LC 146) = HashMap (O(1) lookup, since Week 1)
       + Doubly Linked List (O(1) reordering) — CLOSES LINKED LISTS

─────────────────────────────────────────────────────────

Week 1, Day 4: ArrayDeque used AS a stack (informally)
        │
        ▼
Today: Concept Card — Stacks (formal pattern, same ArrayDeque)
  🔗 Recap: Valid Parentheses (LC 20) — already solved, Week 1 Day 4

─────────────────────────────────────────────────────────

Week 2–3: HashMap internals (bucket array, hashCode/equals)
Day 37: synchronized, intrinsic locks
        │
        ▼
Today, NEW: ConcurrentHashMap — bucket-level locking + CAS,
  contrasted directly against Collections.synchronizedMap()'s
  single whole-map lock
```

---

## Part 1 — Doubly Linked Lists (New Prerequisite)

### Why a `prev` pointer changes what's possible

Every linked-list problem so far has been solvable with a single `next` pointer — but all of them shared one limitation: **if you have a direct reference to some node and want to remove it, you still need to find its predecessor**, because removal means redirecting `predecessor.next`, and a singly linked node has no way to ask "who points to me?" Finding that predecessor means a traversal from `head`, which is O(n) — even though you already had a reference to the node itself.

**Definition — doubly linked list:** a linked list where every node holds two pointers: `next` (as always) *and* `prev`, referencing its immediate predecessor.

```java
class Node {
    int key;
    int value;
    Node prev;
    Node next;

    Node(int key, int value) {
        this.key = key;
        this.value = value;
    }
}
```

With `prev` available, removing a *known* node becomes two pointer reassignments, no traversal at all:

```java
node.prev.next = node.next;
node.next.prev = node.prev;
```

### Dummy head *and* dummy tail

Day 35 introduced a single dummy head to avoid special-casing "the list is currently empty" when inserting at the front. A doubly linked list that needs O(1) insertion and removal at **both** ends benefits from the same trick applied twice — one permanent sentinel at each end:

```
dummyHead ⇄ [real nodes, zero or more] ⇄ dummyTail
```

Both sentinels exist for the entire lifetime of the list and are never treated as "real" data. The real list — however many or few elements it holds — always sits strictly between them. This means `dummyHead.next` and `dummyTail.prev` are **always** valid references to something (in the worst case, to each other, when the real list is empty) — insertion and removal at either end become exactly the same handful of pointer reassignments every time, with zero conditional logic for "is this the first/last real element" or "is the list currently empty."

**🔑 Key Takeaway:** this is the exact same motivation as Day 35's single dummy head, just applied at both ends simultaneously — a sentinel's entire purpose is converting an edge case (empty list, first element, last element) into the *general* case, so the same four lines of code handle every situation uniformly.

---

## Part 2 — LRU Cache

### Problem 11: LRU Cache (LeetCode 146, Medium) — Pattern: HashMap + Doubly Linked List

**Statement:** Design a Least Recently Used (LRU) cache with a fixed `capacity`. `get(key)` returns the value if present (else `-1`) and marks that key as most recently used. `put(key, value)` inserts or updates a key; if this pushes the cache over capacity, evict the **least** recently used entry first. Both operations must run in O(1) average time.

**Naming the two separate requirements before designing anything — this is the right way to open this problem out loud:** O(1) *lookup* by key, and O(1) *reordering by recency* on every access. No single data structure you already have does both — a HashMap alone gives O(1) lookup but no ordering; a plain linked list alone gives ordering but O(n) lookup. **The design is choosing one structure for each job, and connecting them.**

- HashMap`<Integer, Node>` → O(1) lookup, mapping each key directly to its node.
- Doubly linked list, ordered by recency (most-recently-used right after `dummyHead`; least-recently-used right before `dummyTail`) → O(1) move-to-front and O(1) remove-from-tail, since the map already hands you a direct node reference — no traversal ever needed.

```java
class LRUCache {
    private class Node {
        int key, value;
        Node prev, next;
        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    private final int capacity;
    private final Map<Integer, Node> map;
    private final Node dummyHead;
    private final Node dummyTail;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        this.dummyHead = new Node(0, 0);
        this.dummyTail = new Node(0, 0);
        dummyHead.next = dummyTail;
        dummyTail.prev = dummyHead;
    }

    public int get(int key) {
        if (!map.containsKey(key)) {
            return -1;
        }
        Node node = map.get(key);
        remove(node);
        addToFront(node);      // a read still counts as "use" — must refresh recency
        return node.value;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) {
            Node node = map.get(key);
            node.value = value;
            remove(node);
            addToFront(node);
        } else {
            if (map.size() == capacity) {
                Node lru = dummyTail.prev;   // the node just before the tail sentinel
                remove(lru);
                map.remove(lru.key);
            }
            Node newNode = new Node(key, value);
            map.put(key, newNode);
            addToFront(newNode);
        }
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void addToFront(Node node) {
        node.next = dummyHead.next;
        node.prev = dummyHead;
        dummyHead.next.prev = node;
        dummyHead.next = node;
    }
}
```

**Why `get()` still mutates state, despite "reading" a value:** "least recently *used*" counts reads as usage — skipping the move-to-front step on `get()` would make the eviction policy track insertion order instead of actual recency, which is a different (and wrong) policy.

**Why the capacity check happens *before* inserting the new node, not after:** checking `map.size() == capacity` while the new key is not yet present tells you correctly whether adding it is *about* to exceed capacity — evicting first, then inserting, keeps the final size at exactly `capacity`, never briefly at `capacity + 1`.

**Worked trace, `capacity = 2`:**

| operation | returns | cache, MRU → LRU, after |
|---|---|---|
| `put(1, 1)` | — | `[1]` |
| `put(2, 2)` | — | `[2, 1]` |
| `get(1)` | `1` | `[1, 2]` — 1 moved to front |
| `put(3, 3)` | — (evicts key `2`, the LRU) | `[3, 1]` |
| `get(2)` | `-1` | `[3, 1]` — unchanged, key absent |

At `put(3, 3)`, the map was already at capacity 2, so the node just before `dummyTail` — key `2`, since key `1` had just been refreshed to the front by `get(1)` — is evicted first, correctly leaving `1` (the more recently used of the original two) in the cache.

**Complexity: Time O(1) average for both `get()` and `put()`** — the map gives O(1) average lookup; every DLL operation (`remove`, `addToFront`) touches only pointers on nodes the map already handed you directly, with no traversal. **Space O(capacity)** — the map and the list both hold at most `capacity` real entries.

**Edge cases:**
- `capacity = 0`: every `put()` would need to evict immediately; worth explicitly asking an interviewer whether this input is guaranteed not to occur, since the problem's own constraints typically guarantee `capacity ≥ 1`.
- Updating an existing key via `put()`: must both update the value *and* refresh recency — updating only the value (skipping the move-to-front) is a common, easy-to-miss bug.
- Evicting: **must remove the entry from the map, not just the linked list.** Removing only from the DLL and forgetting `map.remove(lru.key)` leaves a stale map entry — a subsequent `get()` on the evicted key would incorrectly still find it and return a value that's no longer supposed to exist.

**💡 Interview Insight:** this is one of the most frequently asked *design* questions at the SDE-2 level specifically because it forces an explicit "which structure solves which requirement" argument — stating that decomposition out loud, before writing code, is the actual signal being evaluated. Two follow-ups worth having ready: (1) *"What if this needed to be thread-safe?"* — wrap the two mutating methods with a `ReentrantLock` (Day 37), exactly like `Counter`, since both `get()` and `put()` mutate shared state. (2) *"What about LFU (Least Frequently Used) instead of LRU?"* — a materially harder variant (LeetCode 460) needing a frequency count *and* a way to find the least-frequent group in O(1); worth knowing it exists and roughly why it's harder, without needing to solve it live.

**🔑 Linked Lists, closed.** With LRU Cache, the pattern reaches 11 required problems (4 from Week 5 + 7 from this week) plus 4 extra-practice problems (LC 92 and LC 23 from Week 5; LC 202 and LC 287 from Day 36 this week) — **15 distinct Linked List problems solved in total.** Every technique — fast/slow pointers, reversal, dummy heads, HashMap-based node mapping, and now doubly linked lists — should be reflexive going forward; later patterns will cite this week's work rather than re-derive it.

---

## Part 3 — Concept Card: Stacks

**What:** LIFO — Last In, First Out. The last element pushed is always the first one popped. You've used `ArrayDeque` as a stack (`push()`/`pop()`/`peek()`, all at the same end) since Week 1, Day 4 — today just gives that usage a name and a recognized place among the other pattern families (HashMap/HashSet, Two Pointers, Sliding Window, Greedy/Intervals, Binary Search, Linked Lists) this series has formally covered.

**Why `ArrayDeque`, not `java.util.Stack`:** worth knowing precisely, since it's a real, common gotcha. `java.util.Stack` is a legacy class (part of Java since version 1.0) that extends `Vector` — every method is `synchronized`, meaning it pays thread-safety overhead on *every single operation*, even in ordinary single-threaded use where that safety is never needed. `ArrayDeque` carries no such overhead and is what the standard library's own documentation recommends over `Stack` for typical use. This is exactly why this series has used `ArrayDeque` as a stack from the very beginning, rather than the class literally named `Stack`.

**Why:** any problem centered on "the most recent unresolved thing" — matching brackets, undoing an operation, evaluating an expression with nested structure — has a shape LIFO fits naturally, because the *most recently opened* thing is exactly the *first* thing that needs to be closed or resolved.

**Interview signal:** "matching," "balanced," "nested," "most recent," "evaluate an expression," "undo."

---

### 🔗 Recap: Valid Parentheses (LeetCode 20, Easy)

**Original coverage:** Week 1, Day 4, Required. Full solution, for reference:

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> pairs = Map.of(')', '(', ']', '[', '}', '{');

    for (char c : s.toCharArray()) {
        if (pairs.containsKey(c)) {                 // c is a closing bracket
            if (stack.isEmpty() || stack.pop() != pairs.get(c)) {
                return false;
            }
        } else {
            stack.push(c);                            // c is an opening bracket
        }
    }
    return stack.isEmpty();
}
```

Push every opening bracket. On a closing bracket, the top of the stack must be its exact matching opener — pop it if so, fail immediately if not (either the stack is empty, meaning there's nothing to match against, or the popped bracket is the wrong type). At the end, an empty stack means everything was matched; anything left over means an opener was never closed. **Time O(n), Space O(n)** worst case (all openers, e.g. `"((((("`).

**🔑 Key Takeaway (recap):** this is the canonical exemplar for the entire Stack pattern family, the same role Two Sum played for HashMap — nearly every stack problem from here on is some variation of "track the most recent unresolved thing, and resolve it against whatever arrives next."

---

## Part 4 — ConcurrentHashMap Internals

### Prerequisites (confirmed)

- HashMap internals — bucket array, `hashCode()`/`equals()` contract, collision handling — Weeks 2–3.
- `synchronized`, intrinsic locks — Day 37.

### The problem with one lock for an entire map

```java
Map<Integer, Integer> map = Collections.synchronizedMap(new HashMap<>());
```

This wraps every method of a plain `HashMap` with `synchronized` on a single shared lock object. **Every** `get()`, `put()`, and `remove()`, from **every** thread, contends for that **one** lock — regardless of which keys, and therefore which underlying buckets, they're actually touching. Two threads writing to completely unrelated keys still serialize behind each other, one waiting while the other holds the lock, even though nothing about their actual work overlaps.

### `ConcurrentHashMap` (Java 8+): locking at the bucket, not the map

The underlying structure is still the same bucket array from Weeks 2–3's HashMap internals — the difference is entirely in *what gets locked, and when*:

- **Inserting into an empty bucket** needs no lock at all — it's done via **CAS** directly on that bucket array slot.
- **Modifying a non-empty bucket** (appending to a collision chain, or during treeification) is `synchronized`, but specifically on **the first node of that one bucket** — not on the map as a whole. Threads touching *different* buckets never block each other at all; only threads genuinely contending for the *same* bucket do.
- **Reads (`get()`) are effectively lock-free** in the common case — they don't contend with writers at all, relying on careful memory-visibility guarantees (`volatile` reads) rather than acquiring anything.

**Definition — CAS (Compare-And-Swap):** an atomic, hardware-level CPU instruction: *read a memory location's current value; if it still equals an expected value, write a new value and report success — atomically, as a single indivisible step; otherwise, change nothing and report failure*, at which point the caller typically retries. Because the "check, then update" happens as one indivisible hardware operation, there's no possibility of another thread interleaving in the middle of it — there is no "middle" to interleave into — which is exactly why CAS-based operations need no explicit lock at all to stay correct.

**⚠️ Common Mistake — this is a performance comparison, not a correctness one.** Both `Collections.synchronizedMap()` and `ConcurrentHashMap` are equally *correct* under concurrent access — neither loses updates or corrupts state, unlike Day 37's plain unsynchronized `int`. The difference is purely how much genuine parallelism each allows: `synchronizedMap` permits **zero** — every operation, from every thread, fully serializes — while `ConcurrentHashMap` permits operations on different buckets to proceed **simultaneously**. Worth naming explicitly: a plain, unwrapped `HashMap` accessed by multiple threads concurrently is a third, *incorrect* option — concurrent structural modification can corrupt its internal bucket structure or silently lose entries, which is why it's never an acceptable choice here at all, safe or not.

**🔑 Key Takeaway:** `ConcurrentHashMap`'s real advantage isn't a cleverer algorithm for any single operation — it's *locking scope*. Shrinking what a lock protects, from "the entire map" down to "one bucket," is the entire mechanism, and it's a generally reusable idea worth being able to name in other contexts, not something specific to maps.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** `ConcurrentMapBenchmark` class — benchmark concurrent writes from 10 threads into a `Collections.synchronizedMap()` versus a `ConcurrentHashMap`.

```java
public class ConcurrentMapBenchmark {

    public static void main(String[] args) throws InterruptedException {
        long syncTime = benchmark(Collections.synchronizedMap(new HashMap<Integer, Integer>()));
        long chmTime = benchmark(new ConcurrentHashMap<Integer, Integer>());

        System.out.println("synchronizedMap:    " + syncTime + " ms");
        System.out.println("ConcurrentHashMap:  " + chmTime + " ms");
    }

    private static long benchmark(final Map<Integer, Integer> map) throws InterruptedException {
        final int numThreads = 10;
        final int writesPerThread = 100_000;
        Thread[] threads = new Thread[numThreads];

        long start = System.currentTimeMillis();
        for (int i = 0; i < numThreads; i++) {
            final int threadId = i;
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < writesPerThread; j++) {
                        map.put(threadId * writesPerThread + j, j);   // distinct keys per thread
                    }
                }
            });
        }
        for (Thread t : threads) t.start();
        for (Thread t : threads) t.join();
        return System.currentTimeMillis() - start;
    }
}
```

**Why keys are deliberately distinct per thread (`threadId * writesPerThread + j`):** this isolates the benchmark to pure *write contention* — every thread is doing genuinely independent work, with no shared keys to update. Under `synchronizedMap`, that independence doesn't matter at all — the single lock still fully serializes every thread. Under `ConcurrentHashMap`, independent keys landing in different buckets should proceed largely in parallel, which is exactly the gap this benchmark is designed to expose.

**Definition of done:** pushed, with the measured timing difference recorded and a one-line explanation of *why* — pointing at bucket-level locking versus a single whole-map lock, not just "ConcurrentHashMap is faster."

## Career Block Guide (1 hr)

**LinkedIn (20 min):** engagement — comment on 3–5 posts.

**Networking:** send connection requests to the 5 Tier B targets identified yesterday.

---

## Day 39 — Interview Questions

**Q1. Why does removing a known node take O(1) in a doubly linked list but not a singly linked one?** A doubly linked node holds `prev` directly, so `node.prev.next = node.next` needs no search. A singly linked node has no way to find its predecessor except a traversal from `head`, which is O(n) even with a direct reference to the node itself.

**Q2. Why does LRU Cache need two different data structures rather than one?** No single structure gives both O(1) lookup by key and O(1) reordering by recency — a HashMap alone has no ordering; a linked list alone has O(n) lookup. Combining them, with the map holding direct node references into the list, gives O(1) for both.

**Q3. Why must `get()` in LRU Cache modify the data structure, even though it's conceptually a "read"?** The eviction policy is based on *recency of use*, and a read counts as use — skipping the move-to-front step would make the cache track insertion order instead, which is a different, incorrect policy.

**Q4. What goes wrong if eviction removes a node from the linked list but forgets to remove it from the map?** The map retains a stale entry pointing to a node that's no longer part of the tracked list — a subsequent `get()` on the evicted key incorrectly succeeds and returns a value that should no longer be considered present.

**Q5. Why use `ArrayDeque` rather than `java.util.Stack`?** `Stack` extends `Vector` and is fully `synchronized` — every operation pays thread-safety overhead even in ordinary single-threaded use. `ArrayDeque` has none of that overhead and is the standard library's own recommended choice.

**Q6. What's the core difference between `Collections.synchronizedMap()` and `ConcurrentHashMap`, given both are thread-safe?** Both are equally correct under concurrent access. `synchronizedMap` uses one lock for the entire map, so every operation from every thread fully serializes. `ConcurrentHashMap` locks at the level of individual buckets (and uses CAS to avoid locking entirely for some operations), so operations on different buckets can proceed in parallel.

**Q7. What is CAS, and why does it avoid needing a lock?** Compare-And-Swap is an atomic hardware instruction: read a value, and if it still matches an expected value, write a new one — as one indivisible step. Because there's no partial, interruptible "middle" to that operation, no other thread can interleave into it, so correctness doesn't require excluding other threads with a lock.

**Q8. Is it ever safe to use a plain, unwrapped `HashMap` from multiple threads concurrently?** No — concurrent structural modification can corrupt its internal bucket structure or silently lose entries; it's not merely slower than the safe alternatives, it's incorrect.

---

## Daily Deliverable Check

- [ ] Doubly linked list mechanics (why `prev` enables O(1) removal, dummy-head-and-tail) explained from memory, no notes.
- [ ] LRU Cache (LC 146) solved and traced by hand, pushed to `dsa-java/linked-lists/` — **Linked Lists complete at 11 required + 4 extra = 15 distinct.**
- [ ] Valid Parentheses (LC 20) confirmed solid from Week 1 (recap only, no re-solve needed), pushed to `dsa-java/stacks/`.
- [ ] Can state the LIFO property and the "most recent unresolved thing" interview signal for Stacks, unprompted.
- [ ] Can explain what `ConcurrentHashMap` locks versus `synchronizedMap`, including a correct definition of CAS.
- [ ] `ConcurrentMapBenchmark` pushed to `java-fundamentals`, with measured timing and a one-line mechanism explanation.
- [ ] LinkedIn engagement done. Connection requests sent to the 5 Tier B targets.

---

## What Tomorrow Assumes You Already Know Cold

Tomorrow assumes Stacks' formal identity — LIFO, `ArrayDeque`, the "most recent unresolved thing" signal — is now solid, since Day 40 opens an entirely new sub-pattern (Monotonic Stack) directly on top of it without re-explaining what a stack is. It also assumes today's `ConcurrentHashMap` mechanism (bucket-level locking, CAS) is understood well enough to recognize as a specific instance of a general idea — "shrink what a lock protects" — since that same idea, applied differently, resurfaces in later system-design-adjacent contexts. Tomorrow's SQL theory is unrelated to today's concurrency track entirely; it connects instead to Day 36's JPA entities, picking up exactly where the ORM abstraction left off.
