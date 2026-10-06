# Binary Search — Quick Revision

## 1. Core Idea

**Binary Search = repeatedly eliminate half of the search space.**

Instead of checking every possibility:

```text
n → n/2 → n/4 → n/8 → ...
```

Therefore:

- **Time:** `O(log n)`
- **Space:** `O(1)` for iterative implementation

---

## 2. When Can Binary Search Be Used?

Binary Search needs the search space to have some kind of **order or monotonicity** that lets us eliminate half.

### Direct Binary Search

Search inside an already sorted structure:

```text
[2, 5, 8, 12, 16, 23, 38]
```

Compare the middle with the target and discard the impossible half.

### Binary Search on the Answer

The input does not need to be sorted.

Instead, the **possible answers** have a monotonic condition:

```text
F F F F T T T T
        ↑
     boundary
```

Example:

```text
speed = 1 → impossible
speed = 2 → impossible
speed = 3 → impossible
speed = 4 → possible
speed = 5 → possible
...
```

Search for the minimum valid answer.

---

## 3. Monotonic

**Monotonic = keeps moving in one direction and never reverses.**

Examples:

```text
1 2 3 4 5 6        ✅ increasing
```

```text
10 8 7 5 3 1       ✅ decreasing
```

```text
1 2 2 3 3 5 8      ✅ increasing
```

Not monotonic:

```text
1 2 3 2 4           ❌
```

because the direction changed.

In Binary Search, a common monotonic pattern is:

```text
F F F F T T T T
```

or:

```text
T T T F F F
```

This gives us a **boundary** we can search for.

---

## 4. Search Space

Maintain:

```text
[left ........ right]
```

Meaning:

> The answer, if it exists, is still inside this range.

This is the key **invariant**.

### Binary Search invariant

> **Never eliminate the true answer from the search space.**

---

## 5. Standard Binary Search

```java
int left = 0;
int right = nums.length - 1;

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
```

### Why the updates?

If:

```text
nums[mid] < target
```

everything up to `mid` is too small:

```java
left = mid + 1;
```

If:

```text
nums[mid] > target
```

everything from `mid` onward is too large:

```java
right = mid - 1;
```

---

## 6. Safe Midpoint

Use:

```java
int mid = left + (right - left) / 2;
```

Instead of:

```java
int mid = (left + right) / 2;
```

### Why?

`left + right` can overflow Java `int`.

Example:

```text
left  = 2,000,000,000
right = 2,100,000,000
```

Their sum exceeds:

```text
Integer.MAX_VALUE = 2,147,483,647
```

Java `int` overflow wraps silently.

The safer formula calculates:

```text
right - left
```

first, then adds half of the range to `left`.

---

## 7. First Bad Version

Versions look like:

```text
Good Good Good Good Bad Bad Bad Bad
                    ↑
                first bad
```

This is a monotonic predicate:

```text
F F F F T T T T
```

Goal:

> Find the **first `True`**.

### Rules

If:

```java
isBadVersion(mid) == false
```

the first bad version must be after `mid`:

```java
left = mid + 1;
```

If:

```java
isBadVersion(mid) == true
```

`mid` itself could be the first bad version:

```java
right = mid;
```

So we **keep `mid`**.

### Implementation

```java
public int firstBadVersion(int n) {
    int left = 1;
    int right = n;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (isBadVersion(mid)) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }

    return left;
}
```

When:

```text
left == right
```

only one candidate remains.

That candidate is the first bad version.

---

## 8. Exact Search vs Boundary Search

### Exact Search

Goal:

```text
Find 23
```

When:

```text
nums[mid] == target
```

we can immediately return.

### Boundary Search

Goal:

```text
Find first valid / first bad / minimum feasible
```

When `mid` is valid, it may still be the answer.

So:

```text
First True:
    True  → keep mid → right = mid
    False → discard mid → left = mid + 1
```

Key pattern:

```text
F F F F T T T T
        ↑
    first true
```

---

## 9. Interview Recognition Signals

Think **Binary Search** when you see:

### Direct

> "The array is sorted."

### Answer-based

> "Find the minimum/maximum value such that a condition is satisfied."

Examples:

```text
minimum speed
minimum capacity
minimum time
maximum possible value
first valid position
first bad version
```

Ask:

1. **What is my search space?**
2. **Can I test a candidate?**
3. **Does the result become monotonic?**
4. **Can I eliminate half after checking the middle?**

---

## 10. The One Mental Model

```text
BINARY SEARCH
     │
     ├── Search sorted input
     │      ↓
     │   compare target
     │      ↓
     │   discard half
     │
     └── Search possible answers
            ↓
       test candidate
            ↓
       discard half
```

**Same idea, different search space.**

---

## 11. Five Things to Remember

1. **Binary Search = eliminate half.**
2. **It requires order or monotonicity.**
3. **`[left, right]` is the current search space.**
4. **Invariant: never eliminate the true answer.**
5. **Safe midpoint:**

```java
left + (right - left) / 2
```

---

## Interview One-Liner

> **Binary Search works when the search space is ordered or monotonic, allowing us to eliminate half of the possibilities after each check, giving `O(log n)` time.**


# Part 2 — Records and Sealed Classes

## 1. `record`

### What is it?
A `record` is a **compact immutable data carrier**.

Use it when a class's main job is simply to **hold a fixed set of values**.

```java
public record Point(int x, int y) {}
```

### What Java gives automatically

```text
✓ Canonical constructor
✓ private final components
✓ equals()
✓ hashCode()
✓ toString()
✓ No setters
✓ Implicitly final
```

Example:

```java
Point p = new Point(3, 4);

p.x();       // 3
p.y();       // 4
p.getX();    // ❌
```

### Important distinction

```text
Normal JavaBean → getX()
Record          → x()
```

### `equals()` / `hashCode()`

Generated from **all record components**.

```java
new Point(3, 4).equals(new Point(3, 4)) // true
```

The generated pair correctly follows the `equals()` / `hashCode()` contract.

### Restrictions

A record:

```text
✓ can implement interfaces
✓ can have methods
✓ can have static fields
✓ can have additional constructors

✗ cannot extend another class
✗ cannot have extra instance fields
✗ cannot be extended
```

Every record already extends `java.lang.Record` and is implicitly `final`.

### Mental model

> **record = "I just need an immutable object to carry these values."**

---

# 2. `sealed`

### What is it?

A `sealed` class/interface **controls who can extend or implement it**.

```java
public sealed interface PaymentState
        permits Pending, Success, Failed {}
```

This means:

```text
PaymentState
 ├── Pending
 ├── Success
 └── Failed
```

Only those permitted types can directly implement `PaymentState`.

---

## 3. `permits`

```java
permits Pending, Success, Failed
```

means:

> These are the allowed direct implementations.

Something else cannot do:

```java
class Cancelled implements PaymentState {} // ❌
```

unless `Cancelled` is added to the permitted hierarchy.

---

# 4. What must a permitted subtype be?

Each direct permitted subtype must explicitly be:

```text
final
sealed
non-sealed
```

### `final`

Stops the hierarchy.

```java
final class Pending implements PaymentState {}
```

### `sealed`

Allows another restricted level.

```java
sealed class PaymentState
        permits Pending, Success {}
```

### `non-sealed`

Reopens unrestricted inheritance from that point.

```java
non-sealed class Pending implements PaymentState {}
```

### Important shortcut

Records are already `final`.

So:

```java
record Pending() implements PaymentState {}
```

automatically satisfies the requirement.

---

# 5. Records + Sealed Interfaces

They work especially well together.

```java
public sealed interface PaymentState
        permits Pending, Success, Failed {}

public record Pending() implements PaymentState {}

public record Success(String transactionId)
        implements PaymentState {}

public record Failed(String reason)
        implements PaymentState {}
```

Mental model:

```text
sealed → controls the allowed types
record → makes each type an immutable data carrier
```

---

# 6. Exhaustive `switch`

A switch is **exhaustive** when it handles **every possible type/value** that can reach it.

```java
static String describe(PaymentState state) {
    return switch (state) {
        case Pending p ->
            "Payment is pending";

        case Success s ->
            "Payment succeeded: " + s.transactionId();

        case Failed f ->
            "Payment failed: " + f.reason();
    };
}
```

`describe()` is just a normal method that receives a `PaymentState`.

It is **not part of the interface**.

---

## Why does this switch not need `default`?

Because `PaymentState` is sealed.

Java knows:

```text
Possible PaymentState types:

Pending
Success
Failed
```

The switch handles all three:

```text
Pending  ✓
Success  ✓
Failed   ✓
```

Therefore:

```text
Exhaustive ✓
```

---

# 7. What happens when a new state is added?

Suppose you add:

```java
public record Cancelled(String reason)
        implements PaymentState {}
```

and update:

```java
permits Pending, Success, Failed, Cancelled
```

Now the compiler knows:

```text
PaymentState
 ├── Pending
 ├── Success
 ├── Failed
 └── Cancelled
```

But the old switch still has only:

```text
Pending  ✓
Success  ✓
Failed   ✓
Cancelled ✗
```

So the switch becomes:

```text
❌ Not exhaustive
```

The compiler forces you to add:

```java
case Cancelled c ->
    "Payment cancelled: " + c.reason();
```

### Why is this useful?

It prevents this kind of bug:

> "I added a new state but forgot to update some code that handles payment states."

The compiler points out the missing cases.

---

# 8. Why `sealed` makes exhaustive checking possible

With a normal interface:

```java
interface PaymentState {}
```

someone could create another implementation somewhere else.

So the compiler cannot know the complete set of possible types.

With:

```java
sealed interface PaymentState
        permits Pending, Success, Failed
```

the compiler knows the complete hierarchy.

Therefore it can verify:

> **Did you handle every possible case?**

---

# 9. Why not just use `default`?

You can:

```java
default -> "Unknown";
```

But then if `Cancelled` is added, the compiler won't force you to explicitly handle it.

The purpose of exhaustive switching is to make new cases **visible at compile time**.

---

# 10. Record Patterns

Java 21 also allows:

```java
case Success(String txId) ->
    "Payment succeeded: " + txId;
```

instead of:

```java
case Success s ->
    "Payment succeeded: " + s.transactionId();
```

This directly deconstructs the record.

Not essential for the core idea, but recognize the syntax.

---

# 11. Java Version

```text
Records                    → Java 16+
Sealed classes/interfaces  → Java 17+
Pattern matching for switch → Java 21+
Record patterns             → Java 21+
```

For the exact exhaustive type-pattern switch shown in these notes:

> **Think Java 21+.**

---

# 12. Common Mistakes

```text
❌ record.x → expecting getX()
   → Record uses x()

❌ Forgetting final/sealed/non-sealed
   → Required for direct permitted subtypes
   → Records are already final

❌ Assuming sealed + pattern switch works as standard Java 17
   → Exhaustive type-pattern switch is standard in Java 21+

❌ Thinking describe() belongs to PaymentState
   → It is simply code that consumes a PaymentState

❌ Adding a new permitted type but not updating every exhaustive switch
   → Compiler catches the missing case
```

---

# Final Mental Model

```text
record
↓
"How should this object store data?"
↓
Immutable data carrier


sealed
↓
"Which types are allowed in this hierarchy?"
↓
Closed / controlled hierarchy


exhaustive switch
↓
"Have I handled every allowed type?"
↓
Compiler checks it
```

### One example tying everything together

```java
sealed interface PaymentState
        permits Pending, Success, Failed {}

record Pending() implements PaymentState {}

record Success(String transactionId)
        implements PaymentState {}

record Failed(String reason)
        implements PaymentState {}

static String describe(PaymentState state) {
    return switch (state) {
        case Pending p -> "Pending";
        case Success s -> "Success: " + s.transactionId();
        case Failed f -> "Failed: " + f.reason();
    };
}
```

**Remember:**

> `record` = immutable data  
> `sealed` = restricted hierarchy  
> `permits` = allowed types  
> exhaustive `switch` = every allowed type must be handled