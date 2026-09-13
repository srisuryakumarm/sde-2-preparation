# Day 101 — Segment Trees Close, and Factory Method / Builder Implemented

**Series:** SDE-2 Interview Prep · Week 15, Day 101 (of 105 DSA-phase days)
**Curriculum Map:** [00_Curriculum_Map.md](./00_Curriculum_Map.md)
**◀ Previous:** [Day 100](./Day100_Resource_Book.md) · **Next ▶:** [Day 102](./Day102_Resource_Book.md)

**Companion to:** Day 3 of `Week_15_Revised.md`

---

## Recap

Yesterday opened Segment Trees from zero — build/update/query, all O(log n), closing the gap neither brute force nor Prefix Sum can close alone — and solved Range Sum Query - Mutable, a Segment Tree used in the most direct way possible: indexed by array *position*, aggregating sums. Yesterday also opened Design Patterns with Singleton, in full depth: the private-constructor-plus-static-accessor shape, the concurrency bug Double-Checked Locking has without `volatile`, and Enum Singleton as the safer modern default.

Today closes Segment Trees with a problem that uses the *same* structure in a meaningfully different way — indexed by **value**, not position — and moves the Theory Block on to the two remaining Creational patterns, implemented comprehensively.

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Solve Count of Smaller Numbers After Self using a Segment Tree indexed by value (with coordinate compression), and explain precisely why processing right-to-left is what makes the technique correct.
2. Explain, by name, how a Fenwick Tree (BIT) achieves the same complexity with a smaller constant factor, and connect its indexing directly to Week 14's `n & (-n)` identity.
3. Distinguish a "Simple Factory" from the true GoF Factory Method pattern, and explain why the distinction matters for the Open/Closed Principle.
4. Implement a Builder that produces an immutable object, and explain what specifically breaks (or degrades) if the built object's fields aren't `final`.

## Concept Dependency Map for Today

```
Day 100 (Segment Tree: build/update/query,      Week 2 (overflow, negative-number
indexed by ARRAY POSITION)                       handling) + Week 14 (n & (-n))
      │                                                  │
      ▼                                                  ▼
  NEW MENTAL MODEL: Segment Tree indexed          Fenwick Tree / BIT (EXTENSION):
  by VALUE instead — each leaf = "how many        1-indexed array, i & (-i) gives
  times has this value been inserted so far"      each index's "responsibility range"
      │                                                  │
      └─────────────────────┬────────────────────────────┘
                             ▼
        Count of Smaller Numbers After Self (LC 315)
        — process right→left, query "count smaller
          than me, among what's been inserted so far"

Week 1 (private constructor pattern,      Week 1 (SOLID: Open/Closed,
from Day 100's Singleton)                 Dependency Inversion)
      │                                          │
      └──────────────────┬───────────────────────┘
                          ▼
              Factory Method (NEW):
        abstract creator + concrete subclasses,
        vs. the "Simple Factory" almost-pattern

Week 1 (immutability via `final` fields,   Day 100 (private constructor,
constructors)                              static nested classes)
      │                                          │
      └──────────────────┬───────────────────────┘
                          ▼
                  Builder (NEW):
      fluent step-by-step construction, avoiding
      the "telescoping constructor" anti-pattern
```

---

## Part 1 — Count of Smaller Numbers After Self (LeetCode #315, Hard)

**Statement:** given an integer array `nums`, return `counts` where `counts[i]` = the number of elements to the **right** of index `i` that are strictly **smaller** than `nums[i]`.

### Approach 1 — Brute force

For each `i`, scan every `j > i`, count `nums[j] < nums[i]`.

```java
public int[] countSmaller(int[] nums) {
    int n = nums.length;
    int[] counts = new int[n];
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            if (nums[j] < nums[i]) counts[i]++;
        }
    }
    return counts;
}
```

O(n²) time, O(1) extra space. Correct, too slow for the constraints this problem is actually rated Hard against.

### Approach 2 — Optimized: Segment Tree, indexed by *value*

**The reframe — a genuinely new mental model, not just yesterday's tree reused verbatim:** yesterday's tree answered "what's the sum over array positions `[l, r]`?" Today's tree answers a completely different question: **"how many of the values I've inserted so far are less than some target value?"** The tree's leaves no longer correspond to array *positions* at all — they correspond to possible *values*, and each leaf stores a **count**: how many times that value has been inserted into the structure so far.

**The right-to-left invariant — why this direction, precisely:** process `nums` from the **last** index to the **first**. At the moment index `i` is processed, only elements at indices `> i` (i.e., genuinely to the *right* of `i`) have been inserted into the tree so far — nothing at or left of `i` has been touched yet. So "how many values already in the tree are smaller than `nums[i]`" is *exactly* "how many elements to the right of `i` are smaller than `nums[i]`" — the question the problem is actually asking, answered directly by construction, not by coincidence.

**Coordinate compression — the general, robust technique:** `nums` can contain negative numbers and values far apart from each other, so indexing a tree directly by raw value would need a tree sized to the full value range, wasteful (or outright infeasible) if that range is huge or sparse. Coordinate compression fixes this generally, for *any* value range: sort the array, remove duplicates, and map each distinct value to its **rank** (its position in that sorted, deduplicated list) — collapsing an arbitrary value range down to a dense `0..k-1` index space, `k` = number of distinct values, always `≤ n`.

```java
public int[] countSmaller(int[] nums) {
    int n = nums.length;
    int[] sorted = nums.clone();
    Arrays.sort(sorted);
    int k = 0;
    int[] unique = new int[n];
    for (int val : sorted) {
        if (k == 0 || unique[k - 1] != val) {
            unique[k++] = val;
        }
    }
    // rank(val) = index of val in unique[0..k-1], found via binary search
    int[] tree = new int[4 * k];         // sum-Segment-Tree over k possible ranks, all counts start at 0
    int[] counts = new int[n];
    for (int i = n - 1; i >= 0; i--) {
        int rank = lowerBound(unique, k, nums[i]);       // this value's compressed index
        counts[i] = query(tree, 0, 0, k - 1, 0, rank - 1); // how many SMALLER ranks inserted so far
        update(tree, 0, 0, k - 1, rank, 1);                // insert this value (increment its count)
    }
    return counts;
}
```

(`query`/`update` here are the same shape as yesterday's Segment Tree — the tree stores counts and combines with `+`, not raw array values — and `lowerBound` is a standard binary-search-for-first-index-not-less-than, reusing Week 4/5's binary search template directly.)

### Worked trace — proving correctness

`nums = [5, 2, 6, 1]` (expected output `[2, 1, 1, 0]`).

**Compression:** sorted-unique values `[1, 2, 5, 6]` → ranks: `1→0, 2→1, 5→2, 6→3`.

Processing right to left:

| i | nums[i] | rank | Query: count of ranks < rank, among inserted so far | counts[i] | Insert |
|---|---|---|---|---|---|
| 3 | 1 | 0 | {} (nothing inserted yet) → 0 | **0** | insert rank 0 |
| 2 | 6 | 3 | {rank 0} → 1 rank is < 3 → 1 | **1** | insert rank 3 |
| 1 | 2 | 1 | {rank 0, rank 3} → only rank 0 is < 1 → 1 | **1** | insert rank 1 |
| 0 | 5 | 2 | {rank 0, rank 3, rank 1} → ranks 0 and 1 are < 2 → 2 | **2** | insert rank 2 |

Final `counts` in original left-to-right order: `[2, 1, 1, 0]` — matches the expected output exactly.

**Why the query range is `[0, rank-1]` and not `[0, rank]`:** the problem wants **strictly** smaller elements, not smaller-or-equal. Ranks are assigned to *distinct* values (post-deduplication), so "count of ranks < rank" is exactly "count of inserted values strictly less than this value" — an off-by-one here (using `rank` instead of `rank-1` as the upper bound) would incorrectly count equal values as "smaller."

**Complexity:** sorting for compression is O(n log n); each of `n` insert/query pairs costs O(log k) ≤ O(log n) against the Segment Tree. **Total: O(n log n) time, O(n) space** — a large improvement over brute force's O(n²) for big inputs.

**Common mistakes / edge cases:**
- Forgetting to deduplicate before assigning ranks — without it, ranks aren't dense/consistent, and the whole compression breaks.
- Processing left-to-right instead of right-to-left — this answers a *different* question (count of smaller elements to the *left*), not the one asked.
- Using `<= rank` instead of `< rank` (i.e., forgetting the `-1`) — silently counts equal elements as smaller, wrong for any input with duplicate values.
- Empty array or single-element array — both handled correctly with no special-casing (a single element trivially has zero elements to its right).

> 🔗 **Forward reference:** a classic alternative for this exact problem is a **merge-sort-based counting approach**, which counts cross-inversions during the merge step itself. It's genuinely elegant, but it depends on merge sort's mechanism — which tomorrow builds from scratch for the first time. It's covered there, once the prerequisite actually exists, rather than referenced today before it's been taught.

> 💡 **Interview Insight:** state the brute-force O(n²) first, then the reframe — "process right to left, and this becomes: how many *smaller values* have I already seen?" — before naming a data structure. The Segment Tree (or Fenwick Tree, below) is the *implementation* of that reframe, not the insight itself; leading with the insight is what actually signals understanding.

---

### `[EXTENSION]` — Fenwick Tree (Binary Indexed Tree), a smaller-constant-factor alternative

The plan tags this problem's pattern as "Segment Tree / Fenwick Tree" for good reason — a Fenwick Tree solves the exact same problem, at the exact same O(n log n) complexity, with meaningfully less code and a much smaller constant factor (a single flat array and an iterative loop, no recursion, no tree-node objects). It's presented here as extension material — genuinely worth knowing by name and mechanism, not required for today's core deliverable.

**The structure:** a 1-indexed array `bit[1..k]`. `bit[i]` doesn't store "just index i" — it stores the aggregate of a *range* of indices ending at `i`, where the range's length is determined by `i`'s **lowest set bit**.

> 🔗 **Direct reuse of Week 14's `n & (-n)`:** that identity — isolating the lowest set bit — is *exactly* what determines each Fenwick index's "span of responsibility." This is the same bit trick from Day 98, now powering a completely different data structure.

```java
class FenwickTree {
    private final int[] bit;   // 1-indexed
    private final int size;

    public FenwickTree(int size) {
        this.size = size;
        this.bit = new int[size + 1];
    }

    public void update(int i, int delta) {          // i is 1-indexed
        for (; i <= size; i += i & (-i)) {
            bit[i] += delta;
        }
    }

    public int prefixSum(int i) {                    // sum of indices [1, i]
        int sum = 0;
        for (; i > 0; i -= i & (-i)) {
            sum += bit[i];
        }
        return sum;
    }

    public int rangeSum(int left, int right) {        // both 1-indexed, inclusive
        return prefixSum(right) - (left > 1 ? prefixSum(left - 1) : 0);
    }
}
```

For this problem: `update(rank + 1, 1)` inserts a value (the `+1` converts a 0-indexed rank into the structure's 1-indexed scheme), and `counts[i] = prefixSum(rank)` (the count of all *smaller* ranks, 1-indexed up to but not including the current rank's own 1-indexed position) reproduces the exact same right-to-left algorithm above.

**Why this is a genuinely distinct structure, not just "Segment Tree with different variable names":** a Fenwick Tree fundamentally relies on **invertibility** — `rangeSum(l, r) = prefixSum(r) − prefixSum(l−1)` needs subtraction to work, which sum supports but min/max do not (there's no way to "subtract out" a minimum the way you can subtract out a sum). This is exactly the trade-off named in yesterday's Segment Tree material: Fenwick Trees are the leaner choice specifically for invertible aggregates like sum and XOR, while Segment Trees remain necessary for non-invertible ones like min/max.

**Complexity:** O(log n) update, O(log n) prefix-sum query — identical asymptotic complexity to the Segment Tree approach, with a smaller constant factor in practice due to the simpler, non-recursive, array-only implementation.

---

## Segment Trees — CLOSED at 2/2 required (0 extra)

Range Sum Query - Mutable (Day 100) and Count of Smaller Numbers After Self (today) close Segment Trees at exactly the 2 problems the plan called for. **No extra practice is being added**, and this is a deliberate call stated explicitly: the plan's own reasoning for this pattern — "genuine exposure to a rare-but-real interview topic, not exhaustive coverage" — is sound on its own terms, matches this series' established treatment of genuinely narrow patterns, and two problems already cover both major usage shapes a Segment Tree is asked about at the SDE-2 level (aggregation indexed by position, and aggregation indexed by value). A third problem would reinforce an already-demonstrated shape rather than add a new one.

---

## Part 2 — Theory Block: Factory Method and Builder, Implemented

### Factory Method

**The motivating problem:** the `scalable-ecommerce-platform`'s Notification module needs to produce different notification types — email, SMS, push — based on a user's preference, without every call site needing to know (and directly construct) each concrete class.

**Two things that get called "factory," only one of which is the actual GoF pattern — worth distinguishing precisely:**

**(a) "Simple Factory"** — not formally one of the GoF 23 patterns, but an extremely common idiom: one static method with a branch (`switch`/`if-else`) picking which concrete class to `new` up.

```java
class SimpleNotificationFactory {
    public static Notification create(String type) {
        switch (type) {
            case "EMAIL": return new EmailNotification();
            case "SMS":   return new SmsNotification();
            case "PUSH":  return new PushNotification();
            default: throw new IllegalArgumentException("Unknown type: " + type);
        }
    }
}
```

Easy to read, easy to write — but adding a new notification type means **editing this method's existing branch logic**, which is precisely what the Open/Closed Principle (Week 1, Day 6) says to avoid: code should be open for extension, closed for modification.

**(b) True Factory Method (GoF)** — an abstract creator declares an abstract method that each concrete subclass overrides to produce *its own* product:

```java
abstract class NotificationCreator {
    // the "factory method" itself
    public abstract Notification createNotification();

    // other logic that USES the product, written once, shared by every subclass
    public void send(String message) {
        Notification n = createNotification();
        n.dispatch(message);
    }
}

class EmailNotificationCreator extends NotificationCreator {
    @Override
    public Notification createNotification() {
        return new EmailNotification();
    }
}

class SmsNotificationCreator extends NotificationCreator {
    @Override
    public Notification createNotification() {
        return new SmsNotification();
    }
}
```

Adding a new notification type now means **adding a new subclass** — zero existing code is touched. This *does* respect Open/Closed, which is the real, substantive difference from the Simple Factory version above — not just "more classes," but a structural change in what has to be edited when the system grows.

> ⚠️ **Common Mistake:** calling *any* method that returns an object a "factory method," or treating Simple Factory and true Factory Method as the same pattern. They solve overlapping problems but make a genuinely different trade-off — worth being able to name the difference precisely if pushed on it, since conflating them is a real, common gap.

**Trade-off, stated honestly:** Simple Factory is less boilerplate and perfectly reasonable for a small, stable set of types that rarely change. True Factory Method scales better as new types are added *frequently* (each addition is purely additive, no existing code touched) at the cost of one extra class per product type — worth it when extension is a realistic, ongoing need, overkill when it isn't.

**Unit tests** (JUnit, matching this series' established testing conventions):

```java
class NotificationCreatorTest {
    @Test
    void emailCreatorProducesEmailNotification() {
        NotificationCreator creator = new EmailNotificationCreator();
        assertInstanceOf(EmailNotification.class, creator.createNotification());
    }

    @Test
    void smsCreatorProducesSmsNotification() {
        NotificationCreator creator = new SmsNotificationCreator();
        assertInstanceOf(SmsNotification.class, creator.createNotification());
    }

    @Test
    void sendDispatchesThroughTheCreatedProduct() {
        NotificationCreator creator = new EmailNotificationCreator();
        // a test double / spy on dispatch() would confirm send() actually
        // routes through whatever createNotification() returns, not a
        // hardcoded type — the real behavior this pattern is protecting.
    }
}
```

---

### Builder

**The anti-pattern this replaces — the telescoping constructor:** consider an `Order` needing an id, customer, several optional fields (discount code, gift-wrap flag, delivery instructions, priority flag...). A "solution" of overloaded constructors, one per combination of optional fields actually needed, quickly becomes unreadable and error-prone — especially once two parameters share a type (a `String` discount code next to a `String` delivery instruction, distinguished only by *position*, silently swappable by mistake with no compiler error).

```java
// The anti-pattern — don't write this:
Order o = new Order("ord-1", "cust-42", null, true, "Leave at door", false);
// Which boolean is gift-wrap and which is priority? Not obvious at the call site.
```

**Builder's mechanism:**

```java
class Order {
    private final String id;
    private final String customerId;
    private final String discountCode;      // optional
    private final boolean giftWrap;          // optional
    private final String deliveryInstructions; // optional

    private Order(Builder builder) {           // private — only Builder can construct
        this.id = builder.id;
        this.customerId = builder.customerId;
        this.discountCode = builder.discountCode;
        this.giftWrap = builder.giftWrap;
        this.deliveryInstructions = builder.deliveryInstructions;
    }

    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        private String id;
        private String customerId;
        private String discountCode;
        private boolean giftWrap = false;
        private String deliveryInstructions;

        public Builder id(String id) { this.id = id; return this; }
        public Builder customerId(String customerId) { this.customerId = customerId; return this; }
        public Builder discountCode(String code) { this.discountCode = code; return this; }
        public Builder giftWrap(boolean giftWrap) { this.giftWrap = giftWrap; return this; }
        public Builder deliveryInstructions(String instr) { this.deliveryInstructions = instr; return this; }

        public Order build() {
            if (id == null || customerId == null) {
                throw new IllegalStateException("id and customerId are required");
            }
            return new Order(this);
        }
    }
}

// usage — every field is self-documenting at the call site:
Order o = Order.builder()
    .id("ord-1")
    .customerId("cust-42")
    .giftWrap(true)
    .deliveryInstructions("Leave at door")
    .build();
```

**Why this is better than the telescoping constructor, and better than plain JavaBean-style setters, on multiple distinct axes:**
- **Readable, unambiguous call sites** — `.giftWrap(true)` can't be mistaken for `.deliveryInstructions(...)` the way two adjacent positional booleans/strings can.
- **Validated, atomic construction** — `build()` is the single point where "is this object actually valid" gets checked, and the target object doesn't exist at all until that check passes. Plain setters on a mutable bean allow an object to exist in a partially-configured, potentially-invalid intermediate state at any point after construction.
- **Immutability** — `Order`'s fields are `final`, set exactly once, inside its own private constructor, called only from `build()`. This gives thread-safety for free (an immutable object can be freely shared across threads with no synchronization needed at all) — a real, concrete benefit, not just a style preference.

> ⚠️ **Common Mistake:** implementing a "Builder" whose target class still has non-`final`, mutable fields with public setters. That's just a fluent API bolted onto a regular mutable bean — it keeps the readability benefit but **loses** the immutability and atomic-validation benefits, which are often the more important half of why Builder is reached for in the first place.

> ⚠️ **Common Mistake:** a setter method that returns `void` instead of `this` — breaks the fluent chain immediately, forcing the caller back to one statement per field.

**Unit tests:**

```java
class OrderBuilderTest {
    @Test
    void buildsWithAllFieldsSetCorrectly() {
        Order o = Order.builder()
            .id("ord-1")
            .customerId("cust-42")
            .giftWrap(true)
            .build();
        assertEquals("ord-1", o.getId());
        assertTrue(o.isGiftWrap());
    }

    @Test
    void missingRequiredFieldThrowsOnBuild() {
        assertThrows(IllegalStateException.class, () ->
            Order.builder().giftWrap(true).build()    // no id/customerId set
        );
    }

    @Test
    void optionalFieldsDefaultSensibly() {
        Order o = Order.builder().id("ord-2").customerId("cust-7").build();
        assertFalse(o.isGiftWrap());          // default false, not left uninitialized/null-unsafe
    }
}
```

**Complexity:** both patterns are O(1) — this is a structural/organizational pattern, not an algorithmic one; there's no asymptotic cost to discuss beyond "one extra object allocation per Builder use," which is negligible.

**Interview framing:** for Factory Method, lead with the Simple-Factory-vs-true-Factory-Method distinction unprompted — it's the detail that separates "has heard of factories" from "understands the actual pattern." For Builder, lead with the telescoping-constructor problem it solves before writing any code, and mention immutability as a benefit explicitly, since it's easy to build a "Builder" that accidentally forfeits it.

---

## Project Block Guide (1.5 hrs)

**Repository:** `lld-java`, `design-patterns` module.

1. Implement the true Factory Method version above (abstract creator + concrete subclasses) for at least two notification types, plus the Simple Factory version alongside it for direct comparison.
2. Implement the `Order` Builder above (or an equivalent domain object with at least one required and several optional fields), with validation in `build()` and `final` fields on the built object.
3. Write the unit tests sketched above for both patterns — confirming correct product types (Factory Method) and both the happy path and the missing-required-field failure path (Builder).
4. **Definition of done:** both patterns implemented, tested, and pushed to `lld-java`.

## Career Block Guide (1 hr)

- **LinkedIn:** engagement — 20 minutes commenting substantively on 3–5 posts.
- **Networking:** continue Tier B outreach and follow-up on prior conversations.

---

## Day 101 — Interview Questions

**Q1. What's different about how today's Segment Tree is indexed, compared to yesterday's?**

*Answer:* Yesterday's tree was indexed by array *position* — leaf `i` held `nums[i]`. Today's tree is indexed by *value* (after coordinate compression) — each leaf represents a possible value, and holds a count of how many times that value has been inserted so far. Same structure, genuinely different mental model of what a leaf means.

---

**Q2. Why does Count of Smaller Numbers After Self process the array right to left?**

*Answer:* At the moment index `i` is processed, only elements at indices greater than `i` should have been inserted into the tree — those are exactly the elements "to the right of `i`." Processing right to left guarantees that invariant by construction: nothing at or left of `i` has been touched when `i` is queried.

---

**Q3. What is coordinate compression, and why is it needed here rather than indexing directly by raw value?**

*Answer:* Sort the array, deduplicate, and map each distinct value to its rank (its position in the sorted-unique list) — collapsing an arbitrary, possibly huge or sparse value range into a dense `0..k-1` index space. It's needed because the tree needs to be sized by *number of distinct values*, not by the raw magnitude of those values, which could be enormous or negative.

---

**Q4. Why does the query use range `[0, rank-1]` instead of `[0, rank]`?**

*Answer:* The problem wants strictly smaller elements. Since ranks are assigned to distinct (deduplicated) values, `[0, rank-1]` counts only values strictly less than the current one; including `rank` itself would incorrectly count values equal to the current one as "smaller."

---

**Q5. What's the overall time complexity, and what dominates it?**

*Answer:* O(n log n) — sorting for coordinate compression is O(n log n), and each of the n insert/query pairs against the Segment Tree costs O(log k) where k ≤ n. Both terms are O(n log n); neither dominates the other asymptotically.

---

**Q6. How does a Fenwick Tree's indexing connect to material from Week 14?**

*Answer:* A Fenwick Tree's `update`/`prefixSum` loops both step by `i & (-i)` — the exact "isolate the lowest set bit" identity derived in Week 14, Day 98. That value determines each index's "span of responsibility" within the structure.

---

**Q7. Why can't a Fenwick Tree be used for a range-minimum-query version of this kind of problem?**

*Answer:* Fenwick Trees compute a range aggregate as `prefixSum(r) − prefixSum(l−1)`, which requires the aggregate to be invertible (undoable via subtraction). Sum and XOR support this; min and max do not — there's no way to "subtract out" a minimum from a combined result. Min/max range queries need a Segment Tree instead.

---

**Q8. What's the actual difference between a "Simple Factory" and the true GoF Factory Method pattern?**

*Answer:* A Simple Factory is one static method with conditional branching (switch/if-else) choosing which concrete class to instantiate — adding a new type means editing that method. True Factory Method is an abstract creator with an abstract method that each concrete subclass overrides to produce its own product — adding a new type means adding a new subclass, with zero existing code touched. The second one respects the Open/Closed Principle; the first doesn't.

---

**Q9. What problem does Builder solve that a constructor with many optional parameters doesn't?**

*Answer:* It avoids the "telescoping constructor" problem — an unreadable, error-prone pile of positional arguments, especially risky once two parameters share a type and can be silently swapped with no compiler error. Builder's fluent, named method calls make every field self-documenting at the call site.

---

**Q10. What specifically is lost if a "Builder" is implemented over a target class that still has mutable, publicly-settable fields?**

*Answer:* The immutability and atomic-validation benefits — arguably the more important half of the pattern. A mutable target can exist in a partially-configured, invalid intermediate state at any point, and there's no single moment where "is this object valid" gets enforced. Only the readability of the fluent call syntax survives; the correctness guarantees Builder is usually reached for do not.

---

**Q11. Why does making the built object's fields `final` matter beyond just "good practice"?**

*Answer:* It gives thread-safety for free — a fully immutable object (all fields final, set once in a private constructor, no mutation possible afterward) can be freely shared and read across multiple threads with zero synchronization needed, since there's no mutable state for concurrent access to corrupt.

---

## Daily Deliverable Check

- [ ] Count of Smaller Numbers After Self (LC 315) solved via a value-indexed Segment Tree with coordinate compression; right-to-left invariant explainable precisely.
- [ ] Segment Trees confirmed closed at 2/2 required, with the deliberate zero-extra reasoning understood, not just accepted.
- [ ] Fenwick Tree mechanism understood as extension material, including its connection to Week 14's `n & (-n)`.
- [ ] Simple Factory vs. true Factory Method distinguished precisely, with the Open/Closed Principle connection stated.
- [ ] Builder implemented over an immutable target with required-field validation in `build()`.
- [ ] Unit tests written and passing for both Factory Method and Builder.
- [ ] Both patterns pushed to `lld-java/design-patterns`.
- [ ] LinkedIn engagement completed. Tier B outreach/follow-up continued.

## What Tomorrow Assumes You Already Know Cold

Tomorrow (Day 102) assumes today's Segment Tree work is complete, closed background — the structure itself won't be revisited unless a future pattern specifically calls for range-query-with-updates again. It also assumes Factory Method and Builder are solid, since Week 16 applies both directly to real LLD systems rather than toy examples. Tomorrow otherwise pivots to a largely independent thread — Sorting algorithms from scratch — whose real prerequisites reach further back: Week 2's overflow/negative-number handling, Week 2 Day 12's Dutch National Flag partitioning (Sort Colors), and Week 8's Quickselect preview from the Kth Largest Element problem. None of today's two new topics are required background for tomorrow; today closes cleanly before tomorrow opens something new.

**Next ▶:** [Day 102](./Day102_Resource_Book.md)
