# Day 100 — Segment Trees Open, and Creational Design Patterns Begin: Singleton

**Series:** SDE-2 Interview Prep · Week 15, Day 100 (of 105 DSA-phase days)
**Curriculum Map:** [00_Curriculum_Map.md](./00_Curriculum_Map.md)
**◀ Previous:** [Day 99](./Day99_Resource_Book.md) · **Next ▶:** [Day 101](./Day101_Resource_Book.md)
**Companion to:** Day 2 of `Week_15_Revised.md`

---

## Recap

Yesterday closed two patterns at once — Bit Manipulation (10/10) and Tries (7/7) — with Maximum XOR of Two Numbers in an Array fusing both into a Bit Trie. Kubernetes also opened from zero: Pod, ReplicaSet, Deployment, Service, and the declarative reconcile-loop model underneath all of them, closing with the Horizontal Pod Autoscaler as a controller running that same loop against a computed metric.

Today opens two genuinely new threads with nothing else scheduled around them: **Segment Trees**, this series' first tree structure built specifically for range aggregation, and **Design Patterns**, this series' first explicit LLD (Low-Level Design) content — starting with Singleton, the simplest of the three Creational patterns the Theory Block introduces today.

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Build a sum-Segment-Tree from scratch — `build`, `update`, `query` — and state precisely why each operation is O(log n).
2. Explain why neither brute force nor a prefix-sum array is sufficient once an array needs *both* frequent updates *and* frequent range queries.
3. Name all three Creational patterns from today's Theory Block and, in one sentence each, what problem each one solves.
4. Implement a correct thread-safe lazy Singleton using Double-Checked Locking, explain precisely why `volatile` is required for correctness (not just a defensive habit), and implement the Enum Singleton alternative.
5. Explain why Enum Singleton is immune to reflection-based and serialization-based attacks that a conventional Singleton class is vulnerable to.

## Concept Dependency Map for Today

```
Week 8 (Heaps: array-backed complete       Week 3 (Prefix Sum: O(1) query,
binary tree, children at 2i+1/2i+2)        O(n) update if array changes)
      │                                            │
      ▼                                            ▼
  Segment Tree (NEW: same index math          Brute force (O(1) update,
  as Heaps, but each node stores a            O(n) query) — the OTHER
  RANGE aggregate, not a priority)            end of the trade-off
      │                                            │
      └───────────────────┬────────────────────────┘
                           ▼
        Range Sum Query - Mutable (LC 307)
        — the exact gap neither prior structure closes:
          O(log n) update AND O(log n) query, together

Week 1 (OOP: classes, interfaces,          Week 5–6 (Threads, synchronized,
abstract classes, private fields,          ReentrantLock, race conditions —
SOLID, static)                             but NOT yet: volatile)
      │                                            │
      └───────────────────┬────────────────────────┘
                           ▼
              Singleton pattern (NEW):
        private constructor + static accessor
                           │
              ┌────────────┴─────────────┐
              ▼                          ▼
   Double-Checked Locking          Enum Singleton
   (needs NEW: volatile,           (needs Week 1's enum
   the reordering bug it fixes)    per-constant syntax,
                                   applied as a new idiom)
```

---

## Part 1 — Segment Tree (New Data Structure)

### The gap this closes

You've had two tools for "give me the sum of a range" for a while, and both have a real weakness:

| Approach | Update a single element | Query a range sum |
|---|---|---|
| Brute force (no structure) | O(1) — just overwrite | O(n) — walk the range every time |
| Prefix Sum array (Week 3) | O(n) — one change invalidates every prefix sum from that index onward | O(1) — subtract two prefix sums |
| **Segment Tree** | **O(log n)** | **O(log n)** |

Prefix Sum's O(1) query looks unbeatable — until the array is no longer static. Change `nums[3]`, and every prefix sum for index 3 and beyond is now wrong; fixing all of them is an O(n) walk. A Segment Tree gives up prefix sum's unbeatable O(1) query in exchange for **both operations landing at O(log n)** — the right trade the moment an array needs to support real, frequent mutation *and* real, frequent range queries at the same time, which is exactly the interview signal worth listening for: *"range sum/min/max query"* stated **together with** *"and the array is also updated."*

### What it is, structurally

A Segment Tree over an array of `n` elements is a binary tree where:
- Each **leaf** represents exactly one array element.
- Each **internal node** represents a contiguous *range* of the array, and stores the combined aggregate (sum, here) of everything in that range — computed from its two children's aggregates.
- The **root** represents the entire array.

It's built and indexed exactly like Week 8's heap: an implicit array-backed binary tree, node `i`'s children at `2*i+1` and `2*i+2`. The only real difference from a heap is *what* each node stores — a heap node holds one element in a priority-ordered structure; a Segment Tree node holds an aggregate over an entire subrange.

> 🔗 **Direct reuse from Week 8:** the child/parent index math is unchanged. If `2i+1`/`2i+2` for children and `(i-1)/2` for parent are already reflexive from Heaps, nothing new needs to be learned about *navigating* the tree — only about what's stored at each node and how it's combined.

A recursive array-backed Segment Tree is conventionally over-allocated to size `4n` (rather than the tighter `2×2^⌈log₂n⌉`) — a simple, safe bound that avoids `ArrayIndexOutOfBoundsException` for array sizes that aren't a clean power of two, at the cost of a small, constant-factor amount of wasted space. This is still O(n) space asymptotically; `4n` is just a practical implementation detail, not a complexity change.

### Build — O(n)

```java
class SegmentTree {
    private final int[] tree;
    private final int n;

    public SegmentTree(int[] nums) {
        n = nums.length;
        tree = new int[4 * n];
        if (n > 0) build(nums, 0, 0, n - 1);
    }

    // node: index into `tree`. [start, end]: the array range this node covers (inclusive).
    private void build(int[] nums, int node, int start, int end) {
        if (start == end) {
            tree[node] = nums[start];          // leaf
            return;
        }
        int mid = start + (end - start) / 2;
        build(nums, 2 * node + 1, start, mid);         // left child
        build(nums, 2 * node + 2, mid + 1, end);        // right child
        tree[node] = tree[2 * node + 1] + tree[2 * node + 2];   // combine
    }
```

Every node is visited and combined exactly once — roughly `2n − 1` total nodes (n leaves + n−1 internal, for a full binary tree of this shape) — giving O(n) total build work.

### Update (point update) — O(log n)

```java
    public void update(int index, int value) {
        update(0, 0, n - 1, index, value);
    }

    private void update(int node, int start, int end, int index, int value) {
        if (start == end) {
            tree[node] = value;                // reached the target leaf
            return;
        }
        int mid = start + (end - start) / 2;
        if (index <= mid) {
            update(2 * node + 1, start, mid, index, value);
        } else {
            update(2 * node + 2, mid + 1, end, index, value);
        }
        tree[node] = tree[2 * node + 1] + tree[2 * node + 2];   // recompute on the way back up
    }
```

**Why O(log n):** the recursion follows exactly one root-to-leaf path — at each level, the range is halved, and the tree has height O(log n). Only the ancestors of the changed leaf need their aggregate recomputed, and there are O(log n) of them.

### Range Query — O(log n)

```java
    public int query(int left, int right) {
        return query(0, 0, n - 1, left, right);
    }

    private int query(int node, int start, int end, int left, int right) {
        if (right < start || end < left) {
            return 0;                          // completely outside — identity element for sum
        }
        if (left <= start && end <= right) {
            return tree[node];                 // completely inside — no need to go deeper
        }
        int mid = start + (end - start) / 2;
        int leftSum = query(2 * node + 1, start, mid, left, right);
        int rightSum = query(2 * node + 2, mid + 1, end, left, right);
        return leftSum + rightSum;              // partial overlap — combine both halves
    }
}
```

**Why O(log n) — proven, not asserted:** at any single level of the recursion, the node ranges at that level partition the array into disjoint contiguous blocks. A node is either **completely outside** the query range (return immediately, O(1), no further recursion), **completely inside** it (return the precomputed aggregate immediately, O(1), no further recursion), or **straddles a boundary** of the query range (partial overlap, must recurse into both children). Because the query range `[left, right]` has exactly two boundaries, **at most two nodes per level can straddle a boundary** — one where the query's left edge falls, one where its right edge falls. Every other node at that level is cleanly resolved in O(1). With O(log n) levels and at most a small constant number of "still recursing" nodes per level, the total number of nodes visited across the entire query is O(log n).

**Trade-off against the nearest alternative (Fenwick Tree / Binary Indexed Tree):** a Fenwick Tree achieves the same O(log n) update/query for sums with a much smaller constant factor (a single flat array, an iterative loop, no recursion or node objects) — genuinely worth knowing by name and covered as extension material tomorrow. Its limitation: Fenwick Trees fundamentally exploit *invertibility* (range = prefix(r) − prefix(l−1), which needs subtraction), so they naturally support sum and XOR but do **not** directly generalize to min/max, which have no inverse operation. A Segment Tree handles **any** associative combine function uniformly — sum, min, max, GCD, anything — at the cost of a larger constant factor and (for this recursive form) more code.

**Complexity summary:** Build O(n) time, O(n) space (via the `4n` array). Update O(log n). Query O(log n).

**Common mistakes:**
- Sizing the array as `n` instead of `4n` — works for some inputs, throws `ArrayIndexOutOfBoundsException` for others depending on how the recursion happens to split, which makes it a nasty intermittent bug rather than a clean, reproducible one.
- Checking "completely inside" before "completely outside," or in the wrong order relative to the recursive split — the two base-case checks must both run *before* computing `mid` and recursing, or the pruning that makes this O(log n) instead of O(n) doesn't actually happen.
- Using `0` as the "no overlap" identity value for a **min** or **max** tree instead of `+∞`/`−∞` — `0` is only correct as sum's identity. This problem's tree is sum-based, so `0` is right here, but it's worth stating precisely *why* it's right, not just copying it into a different aggregate later.

> 💡 **Interview Insight:** a very standard follow-up is *"what if updates were range updates instead of point updates — add `x` to every element in `[l, r]`?"* The answer is **lazy propagation** — deferring a pending update at a node until a query actually needs to descend past it. Naming this by name, and that it's how range updates stay O(log n) instead of degrading to O(n) per update, is the expected depth; building it is `[EXTENSION]`, out of scope for today's deliberately-light two-problem exposure.

---

## Part 2 — Range Sum Query - Mutable (LeetCode #307, Medium)

**Statement:** design a class `NumArray` supporting `update(index, val)` (set `nums[index] = val`) and `sumRange(left, right)` (return the sum of `nums[left..right]` inclusive), with both operations called repeatedly and in any order.

### Approach 1 — Brute force
Store the array directly; `update` is O(1); `sumRange` walks the range, O(n). Fine if updates dominate and ranges are rarely queried — wrong shape for this problem, which calls both repeatedly.

### Approach 2 — Prefix Sum (why it fails here)
O(1) `sumRange` via two prefix-sum lookups — but `update` forces an O(n) rebuild of every prefix sum from that index onward. This is *precisely* the gap named in Part 1: fast query, slow update, and this problem needs both fast.

### Approach 3 — Optimized: Segment Tree
Wrap the `SegmentTree` class built above behind the exact API LeetCode expects:

```java
class NumArray {
    private final SegmentTree tree;

    public NumArray(int[] nums) {
        tree = new SegmentTree(nums);
    }

    public void update(int index, int val) {
        tree.update(index, val);
    }

    public int sumRange(int left, int right) {
        return tree.query(left, right);
    }
}
```

### Worked trace — proving correctness on a concrete example

`nums = [1, 3, 5]`. Building: `build(node=0, start=0, end=2)` → `mid = 1` → recurse left `build(node=1, start=0, end=1)` and right `build(node=2, start=2, end=2)`.
- `build(node=1, 0, 1)`: `mid = 0` → `build(node=3, 0, 0)` → leaf, `tree[3] = 1`. `build(node=4, 1, 1)` → leaf, `tree[4] = 3`. `tree[1] = tree[3] + tree[4] = 4`.
- `build(node=2, 2, 2)`: leaf, `tree[2] = 5`.
- `tree[0] = tree[1] + tree[2] = 4 + 5 = 9`. ✓ (matches `1+3+5=9` directly)

`sumRange(0, 2)`: query range `[0,2]` exactly matches root's range `[0,2]` → completely inside → return `tree[0] = 9` immediately, no descent needed. ✓

`update(1, 2)` (set `nums[1] = 2`): descend from node 0 `[0,2]`, `mid=1`, target index `1 <= mid` → recurse left into node 1 `[0,1]`; there, `mid=0`, target index `1 > mid` → recurse right into node 4 `[1,1]` → leaf, set `tree[4] = 2`. Unwinding: `tree[1] = tree[3] + tree[4] = 1 + 2 = 3`; `tree[0] = tree[1] + tree[2] = 3 + 5 = 8`. ✓ (matches new true sum `1+2+5=8`)

**Complexity for this problem specifically:** O(n) one-time build, O(log n) per `update`, O(log n) per `sumRange`, O(n) space.

**Edge cases:** single-element array (root is also a leaf — `build`'s base case fires immediately, no recursion needed); `left == right` (a single-element range query, still correct — the query still finds a node that's "completely inside" and returns without descending unnecessarily far); negative numbers (sum's identity, `0`, is correct regardless of sign — no special-casing needed, unlike a min/max tree would require).

**Interview framing:** open by naming the brute-force/prefix-sum trade-off table before writing any code — that comparison *is* the justification for reaching for a Segment Tree at all, and stating it unprompted is a strong signal. The standard follow-up is the lazy-propagation question named above.

---

## Part 3 — Theory Block: Creational Design Patterns, and Singleton in Full Depth

### The Creational family, in one pass

The three patterns named in today's Theory Block all answer some version of the question *"how does an object get created?"* — as opposed to what it does once it exists (that's a different family, covered next week). One sentence each:

- **Singleton** — ensure exactly one instance of a class exists, globally reachable. Today's full-depth pattern.
- **Factory Method** — delegate the decision of *which concrete class to instantiate* to a subclass or dedicated method, so the calling code depends only on an abstraction, never a concrete constructor. Tomorrow's full-depth pattern.
- **Builder** — construct a complex object step by step through a fluent chain of calls, for the case where a plain constructor would otherwise need a long, error-prone list of optional parameters. Also tomorrow's full-depth pattern.

All three exist because directly calling `new ConcreteClass(...)` scattered throughout a codebase creates tight coupling to specific classes and specific constructor shapes — exactly the kind of dependency Week 1's SOLID principles (specifically the Dependency Inversion and Open/Closed principles) already named as worth avoiding. These three patterns are that avoidance, made concrete.

### Singleton — the motivating problem

Consider an application-wide `ConfigurationManager`: it reads configuration (say, from a file or environment) once, and every part of the application needs to read the *same* values afterward. Two problems if this were just an ordinary class instantiated wherever it's needed:
1. **Wasted/duplicated work** — every `new ConfigurationManager()` call re-reads and re-parses the config from scratch.
2. **Inconsistent state** — if config were ever mutable, or if two reads raced against a concurrent file change, different parts of the application could end up looking at *different* `ConfigurationManager` instances with subtly different data.

Singleton's job: guarantee **exactly one instance**, constructed once, shared everywhere.

### The mechanism

Two ingredients, both small individually:
1. A **private constructor** — prevents any code outside the class from calling `new` on it directly. (This is a small, natural extension of `private` — already known since Week 1 — applied to a constructor instead of a field.)
2. A **private static field** holding the one instance, exposed through a **public static accessor method**.

```java
public class ConfigurationManager {
    private static ConfigurationManager instance;
    private final Map<String, String> settings;

    private ConfigurationManager() {
        settings = loadSettingsFromDisk();      // expensive — exactly why we want this to happen once
    }

    public static ConfigurationManager getInstance() {
        if (instance == null) {
            instance = new ConfigurationManager();
        }
        return instance;
    }
}
```

This version works — under a single thread. It is **not thread-safe**: two threads can both evaluate `instance == null` as `true` before either finishes constructing an instance, and both proceed to construct and assign, producing two distinct instances — the exact race-condition shape Week 6, Day 37's `Counter` example already demonstrated, now applied to object construction instead of an integer increment.

### Fixing it — the spectrum, and where each stop actually lands

| Version | Thread-safe? | Lazy? | Cost after first call |
|---|---|---|---|
| Naive (above) | ❌ No | ✅ Yes | Cheap, but wrong |
| `synchronized` on the whole method | ✅ Yes | ✅ Yes | **Every** call pays lock overhead, forever |
| Double-Checked Locking, no `volatile` | ❌ Still broken (subtly) | ✅ Yes | Cheap after first call — but wrong |
| Double-Checked Locking, **with** `volatile` | ✅ Yes | ✅ Yes | Cheap after first call |
| Enum Singleton | ✅ Yes | ✅ Yes (class-load-time) | Cheap, and simplest to get right |

The fully-`synchronized`-method version is correct but pays the lock's overhead on **every single call**, forever — even long after `instance` is already set and there's nothing left to protect. Double-Checked Locking exists specifically to avoid that permanent tax.

### Double-Checked Locking — and why "double-checked" specifically

```java
public class ConfigurationManager {
    private static volatile ConfigurationManager instance;   // volatile — see below
    private final Map<String, String> settings;

    private ConfigurationManager() {
        settings = loadSettingsFromDisk();
    }

    public static ConfigurationManager getInstance() {
        if (instance == null) {                         // first check — no lock, fast path
            synchronized (ConfigurationManager.class) {
                if (instance == null) {                  // second check — under the lock
                    instance = new ConfigurationManager();
                }
            }
        }
        return instance;
    }
}
```

The **first check** (no lock) is what makes this fast after the instance already exists — the overwhelming majority of calls hit `instance == null` as `false` immediately and return, never touching the `synchronized` block at all. The **second check**, *inside* the lock, exists because multiple threads could have all passed the first check simultaneously (before any of them constructed anything) and are now queued at the lock — only the first one through should actually construct the instance; every thread behind it needs to re-check and discover it's already been done, or they'd each construct their own.

### Why `volatile` is required — proven, not just asserted

This is the one place a "looks correct" version is still wrong, and it's worth being precise about *why*.

`instance = new ConfigurationManager();` looks like one atomic step. It is not — conceptually it's three: **(1)** allocate memory for the new object, **(2)** run the constructor, initializing its fields, **(3)** assign the resulting reference to `instance`. Without `volatile` (or other synchronization on this specific field), the Java Memory Model permits the compiler/CPU to **reorder steps (2) and (3)** — from the *single thread* doing the construction, this reordering is invisible and harmless, since nothing about that thread's own subsequent behavior depends on the order. But it is not invisible to a **second** thread.

Suppose Thread A is inside the `synchronized` block, constructing the instance, and step (3) (publishing the reference) has been reordered ahead of step (2) (finishing the constructor). At that exact moment, Thread B calls `getInstance()`, evaluates the **first check** (`instance == null`) — no lock involved here — and sees a **non-null** reference, because it's already been published. Thread B returns that reference immediately, believing it has a fully-built `ConfigurationManager`. It doesn't — the constructor hasn't finished running on that object yet, from Thread B's point of view. Thread B may observe default field values (`null`, `0`, `false`) instead of the real, fully-initialized ones. This is a genuine, if rare and hard-to-reproduce, correctness bug — not a hypothetical.

`volatile` on `instance` fixes this by forbidding exactly this reordering for its own writes and reads: a write to a `volatile` field cannot be reordered ahead of any write that happened before it in program order (so the constructor's field writes are guaranteed to happen-before the write to `instance`), and a read of a `volatile` field establishes a happens-before relationship with whichever write it observes — meaning any thread that sees a non-null `instance` is also guaranteed to see everything that happened-before that write, including the fully-completed construction.

> 🔑 **Key Takeaway:** `volatile` here is not "extra safety" or a defensive habit — it is the specific, necessary fix for a specific, provable reordering bug. Without it, Double-Checked Locking is broken in a way that will not show up in ordinary testing (it depends on a genuine race between construction and a concurrent read, on real multi-core hardware, under real reordering) — the kind of bug that passes every test and then appears in production. This connects directly back to Week 6, Day 37's `synchronized`/`ReentrantLock` material and Week 5, Day 29's shared-heap-across-threads model — `volatile` is a *third*, distinct concurrency tool alongside those two, solving a visibility/ordering problem rather than a mutual-exclusion problem.

> ⚠️ **Common Mistake:** treating `volatile` as interchangeable with `synchronized`. `volatile` guarantees visibility and ordering for reads/writes of *that one field* — it does **not** provide mutual exclusion (it can't make a compound operation like `count++` atomic, for instance). The `synchronized` block in Double-Checked Locking is still doing real work — preventing two threads from both constructing an instance — and `volatile` is doing separate, complementary work — making sure a completed construction is *visible* correctly across threads. Neither replaces the other here; both are required.

### Enum Singleton — the modern default

```java
public enum ConfigurationManagerEnum {
    INSTANCE;

    private final Map<String, String> settings;

    ConfigurationManagerEnum() {
        settings = loadSettingsFromDisk();
    }

    public String get(String key) {
        return settings.get(key);
    }
}
// usage: ConfigurationManagerEnum.INSTANCE.get("db.url")
```

This reuses Week 1's enum-per-constant syntax (a constructor, fields, methods on an enum) in a specific new idiom: an enum with **exactly one constant**. It gets thread-safety for free, with no `volatile` and no explicit lock, because of a guarantee built into the JVM itself:

> 🔑 **Key Takeaway — why no lock is needed here:** class initialization in the JVM is specified to be thread-safe — the classloader effectively holds an implicit lock during a class's initialization, and a class is guaranteed to be initialized **exactly once**, no matter how many threads reference it concurrently for the first time. `INSTANCE` is created during `ConfigurationManagerEnum`'s class initialization, so this guarantee covers it directly — no additional locking code needs to be written or reasoned about.

**Immune to two real attacks a conventional Singleton isn't automatically protected against:**
- **Reflection:** ordinarily, a private constructor can still be invoked via reflection (`constructor.setAccessible(true)` followed by `constructor.newInstance()`), silently creating a second instance and defeating the entire pattern. The JDK explicitly forbids this specifically for enum types — `Constructor.newInstance()` checks whether the target class is an enum and throws `IllegalArgumentException: Cannot reflectively create enum objects` if so. This is a deliberate, hard-coded JDK safeguard, not an incidental side effect.
- **Serialization:** deserializing an ordinary Singleton with Java's default mechanism calls no constructor at all — it reconstructs the object's fields directly, producing a **second**, distinct instance, unless the class explicitly implements `readResolve()` to redirect deserialization back to the existing instance. Enum deserialization is handled specially by the JVM: only the constant's **name** is serialized, and deserialization resolves it via `Enum.valueOf()`, which looks up the *existing* constant rather than constructing anything new — structurally incapable of producing a duplicate.

**Trade-off against Double-Checked Locking:** Enum Singleton is simpler and safer by construction, but an enum implicitly extends `java.lang.Enum`, so it **cannot extend any other class** (Java's single-inheritance rule) — a real constraint if the Singleton needs to inherit from some existing base class. It can still implement interfaces freely. It also looks unusual to developers unfamiliar with the idiom, which is a readability cost worth weighing, not a correctness one.

> ⚠️ **Common Mistake — a balanced note, not just praise:** Singleton is a genuinely useful pattern for the "exactly one, globally shared, expensive-ish resource" shape — but it is also frequently criticized as a disguised global variable. A class that reaches for `ConfigurationManager.getInstance()` internally has a hidden dependency that's hard to swap out in a unit test (you can't easily substitute a mock instance the way you could with constructor-injected dependencies). Worth knowing the criticism exists and being able to discuss it, not just defend the pattern uncritically.

**Complexity:** O(1) access after the instance exists, for both versions. Double-Checked Locking's *first* call pays one `synchronized` block; every call after pays only the cheap unsynchronized check. Enum Singleton's cost is paid once, at class-load time.

**Interview framing:** state the naive version's race condition first (prove it, the way Day 37's `Counter` did), then Double-Checked Locking with the `volatile` reasoning, then Enum as the modern preferred default. A very likely follow-up: *"why not just synchronize the whole method?"* — answer with the permanent-overhead argument from the table above, not just "it's slower."

---

## Project Block Guide (1.5 hrs)

**Repository:** `lld-java` (new — every LLD system from Week 16 onward lives here).

1. Initialize the repository with a standard Maven/Gradle skeleton (matching the structure already established for `scalable-ecommerce-platform`) and a `design-patterns` module.
2. Inside it, implement **both** Singleton variants from Part 3 above — the Double-Checked Locking version (with `volatile`) and the Enum version — as separate, clearly-named classes.
3. Write a short comment (or a small `main` demo) explaining *why* Enum Singleton is the generally-safer default — reflection immunity, serialization immunity, no `volatile`/lock reasoning required — while noting the single-inheritance limitation as the one real reason to reach for Double-Checked Locking instead.
4. **Definition of done:** both classes compile cleanly, and the comment/demo makes the trade-off explicit rather than just implementing both silently.

## Career Block Guide (1 hr)

- **LinkedIn:** engagement — spend 20 minutes leaving substantive comments on 3–5 posts in your network (not just reactions).
- **Networking:** reply to any recruiter inbound messages received this week.

---

## Day 100 — Interview Questions

**Q1. Why isn't a plain Prefix Sum array good enough for Range Sum Query - Mutable?**

*Answer:* Prefix Sum gives O(1) queries, but a single `update` invalidates every prefix sum from that index onward, forcing an O(n) rebuild. This problem calls both `update` and `sumRange` repeatedly, so O(n) updates are unacceptable — exactly the gap a Segment Tree closes by landing both operations at O(log n).

---

**Q2. Walk through why a Segment Tree's range query is O(log n) — don't just state it.**

*Answer:* At each level of the recursion, node ranges partition the array into disjoint blocks. A node is either fully outside the query (return in O(1)), fully inside it (return the precomputed aggregate in O(1)), or straddles one of the query's two boundaries (must recurse into both children). At most two nodes per level can straddle a boundary — one per edge of the query range — so each level contributes only a small constant number of "still recursing" nodes. With O(log n) levels total, the whole query visits O(log n) nodes.

---

**Q3. What's the index math for navigating the tree, and where has it been seen before in this series?**

*Answer:* Node `i`'s children are at `2i+1` and `2i+2`, parent at `(i-1)/2` — identical to Week 8's array-backed heap indexing. The structures store different things (a heap node holds one prioritized element; a Segment Tree node holds a range aggregate), but the navigation math is unchanged.

---

**Q4. Segment Tree vs. Fenwick Tree (BIT) — what's the actual trade-off?**

*Answer:* Fenwick Trees get the same O(log n) update/query for sums with a much smaller constant factor (a flat array, iterative, no node objects) — but they rely on invertibility (range = prefix(r) − prefix(l−1)), so they naturally support sum/XOR but not min/max, which have no inverse. Segment Trees handle any associative combine function uniformly, at the cost of more code and a larger constant factor.

---

**Q5. If this problem needed range updates (add x to every element in `[l, r]`) instead of point updates, how would that change the approach?**

*Answer:* Lazy propagation — deferring a pending update at a node until a query actually needs to descend past it, so a range update doesn't have to eagerly touch every individual leaf in the range. Named here as the correct answer; not built today, since today's exposure is deliberately kept to point updates.

---

**Q6. Name all three Creational patterns from today and, in one line each, what each one solves.**

*Answer:* Singleton — exactly one instance of a class, globally reachable. Factory Method — delegates *which* concrete class gets instantiated to a subclass or dedicated method, decoupling the caller from concrete constructors. Builder — constructs a complex object step by step, avoiding a constructor with a long list of optional parameters.

---

**Q7. Why is the naive lazy Singleton (`if (instance == null) instance = new X();`, no locking) broken under concurrency?**

*Answer:* Two threads can both evaluate `instance == null` as true before either finishes constructing an instance — both then proceed to construct and assign, producing two distinct instances. The same race-condition shape as Week 6, Day 37's `Counter` example, applied to object construction instead of an integer increment.

---

**Q8. Why does Double-Checked Locking check `instance == null` twice instead of once?**

*Answer:* The first check (unsynchronized) is a fast path — once the instance exists, the overwhelming majority of calls return immediately without ever touching the lock. The second check, inside the `synchronized` block, exists because multiple threads may have all passed the first check and be queued at the lock; only the first one through should construct the instance, and every thread behind it must re-check to discover it's already done, or each would construct its own.

---

**Q9. Why does Double-Checked Locking specifically need `volatile` on the instance field — what actually breaks without it?**

*Answer:* `instance = new X()` is really three steps — allocate, run the constructor, assign the reference. Without `volatile`, the JVM/CPU may legally reorder the constructor's field writes and the reference assignment, since that reordering is invisible to the single thread doing the construction. A second thread could then see a non-null `instance` (published early) before the constructor has actually finished running, and could observe a partially-initialized object — default field values instead of real ones. `volatile` forbids exactly this reordering for its own reads/writes and establishes a happens-before relationship, guaranteeing any thread that observes a non-null `instance` also sees the fully-completed construction.

---

**Q10. Is `volatile` a substitute for `synchronized` here?**

*Answer:* No — they solve different problems. `synchronized` provides mutual exclusion (only one thread constructs the instance). `volatile` provides visibility/ordering (a thread that sees the reference also sees the fully-constructed object behind it). Double-Checked Locking needs both; neither alone is sufficient.

---

**Q11. Why is Enum Singleton thread-safe with no explicit lock or `volatile` at all?**

*Answer:* JVM class initialization is specified to be thread-safe — the classloader effectively holds an implicit lock during initialization, and a class is guaranteed to initialize exactly once regardless of how many threads reference it concurrently for the first time. The enum constant is created during that initialization, so it inherits this guarantee directly.

---

**Q12. How is Enum Singleton immune to reflection-based attacks that a conventional Singleton isn't?**

*Answer:* A conventional Singleton's private constructor can still be invoked via reflection (`setAccessible(true)` + `newInstance()`), silently creating a second instance. The JDK explicitly guards against this for enums: `Constructor.newInstance()` checks whether the declaring class is an enum and throws `IllegalArgumentException("Cannot reflectively create enum objects")` if so — a deliberate, hard-coded safeguard specific to enum types.

---

**Q13. How is Enum Singleton immune to serialization-based attacks?**

*Answer:* Deserializing an ordinary Singleton with Java's default mechanism reconstructs the object's fields directly, without calling any constructor — producing a second, distinct instance, unless `readResolve()` is explicitly implemented to redirect back to the existing one. Enums are serialized specially: only the constant's name is written, and deserialization resolves it via `Enum.valueOf()`, which looks up the existing constant rather than constructing anything — structurally incapable of producing a duplicate.

---

**Q14. What's the real trade-off of choosing Enum Singleton over Double-Checked Locking?**

*Answer:* Enum Singleton is simpler and safer by construction — but an enum implicitly extends `java.lang.Enum`, so it cannot extend any other superclass (it can still implement interfaces). Double-Checked Locking allows normal class inheritance at the cost of needing the `volatile`/locking reasoning to be exactly right.

---

## Daily Deliverable Check

- [ ] Segment Tree built from scratch (`build`/`update`/`query`), with the O(log n) query bound provable, not just quotable.
- [ ] Range Sum Query - Mutable (LC 307) solved via the Segment Tree wrapper; pushed to `dsa-java/segment-trees/`.
- [ ] Brute force vs. Prefix Sum vs. Segment Tree trade-off table reproducible without notes.
- [ ] All three Creational patterns named with a correct one-line description each.
- [ ] Thread-safe Double-Checked Locking Singleton implemented, with the `volatile`-reordering bug explainable precisely — what breaks without it, not just "it's needed."
- [ ] Enum Singleton implemented, with both the reflection-immunity and serialization-immunity mechanisms explainable.
- [ ] `lld-java` repository initialized; both Singleton variants in a `design-patterns` module; comment/demo explains the Enum-as-default reasoning.
- [ ] LinkedIn engagement (3–5 substantive comments) completed. Recruiter inbound replied to.

## What Tomorrow Assumes You Already Know Cold

Tomorrow (Day 101) assumes today's Segment Tree mechanism — `build`/`update`/`query`, the index math, the O(log n) argument — is fully reflexive, since tomorrow's Count of Smaller Numbers After Self reuses the exact same structure but applies it to a genuinely different mental model (a tree indexed by *value*, tracking counts, rather than by array position). It also assumes today's Singleton material — the private-constructor-plus-static-accessor shape, and specifically the reasoning that motivated `volatile` — is solid background, since tomorrow's Factory Method and Builder patterns are built on the same "control how an object gets constructed" foundation, one level up in complexity.

**Next ▶:** [Day 101](./Day101_Resource_Book.md)
