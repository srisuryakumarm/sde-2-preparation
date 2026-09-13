# Day 9 — Two Pointers: Sorted-Array Variants, and the JVM Memory Model

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 8 Resource Book](Day8_Resource_Book.md)
**Next ▶:** [Day 10 Resource Book](Day10_Resource_Book.md)
**Companion to:** Day 9 of `Week_02_Revised.md`

---

## ⚠️ Overlap Notice — Read This First

Same situation as yesterday: both of today's plan-required problems were already solved in Week 1, Day 6, as extra practice.

| LC # | Problem | Originally solved |
|---|---|---|
| 977 | Squares of a Sorted Array | Week 1, Day 6 (Extra Practice) |
| 167 | Two Sum II (Input Array Is Sorted) | Week 1, Day 6 (Extra Practice) |

Both get a short recap; the freed DSA time goes to **Sum of Square Numbers (LC 633)** — a new opposite-ends rep with a genuinely different flavor worth seeing: the "array" being searched isn't given to you, it's an implicit numeric range.

---

## Recap

Yesterday closed the fast-slow variant with LC 80 and opened Recursion — the call stack, base/recursive cases, and the concrete cost of unbounded recursion. Today returns to **opposite-ends** convergence, then goes one level under Week 1's "pass-by-value" rule to explain *why* Java behaves that way, using exactly the stack-frame model from yesterday.

---

## Learning Objectives

1. Recognize opposite-ends two pointers even when the "array" is an implicit range rather than a literal input array (LC 633).
2. State precisely, and defend if pushed, why Java is pass-by-value even for objects — including why that phrase is not a contradiction of "you can mutate an object's fields through a method."
3. Explain what lives on the stack vs. the heap, and why that split is what *produces* pass-by-value semantics, rather than pass-by-value being an arbitrary rule.

---

## Concept Dependency Map

```
Week 1 Day 1: pass-by-value (primitives) — established as a rule
Week 1 Day 2: pass-by-value (objects/arrays) — extended, same rule
Day 8 (yesterday): stack frames, made explicit — parameters/locals live in a frame
        │
        ├──▶ Today: JVM Memory Model — WHY pass-by-value holds,
        │           at the mechanism level (stack frame vs. heap object)
        │
Week 1 Day 6: Two Pointers, opposite-ends (LC 167, LC 977)
        │
        └──▶ Today: LC 633 — opposite-ends over an implicit range
```

---

## Part 1 — Two Pointers: Recap and One New Rep

### 🔗 Recap: Squares of a Sorted Array (LC 977)

**Original coverage:** Week 1, Day 6, Extra Practice, labeled *Opposite Ends (From the Back)* — a deliberate hybrid label, since the pointers converge by comparing `|left|` and `|right|` (opposite-ends movement) while the result is filled starting from its last index (from-the-back construction).

```java
public static int[] sortedSquares(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    int left = 0, right = n - 1;
    for (int i = n - 1; i >= 0; i--) {          // fill the OUTPUT from the back
        if (Math.abs(nums[left]) > Math.abs(nums[right])) {
            result[i] = nums[left] * nums[left];
            left++;
        } else {
            result[i] = nums[right] * nums[right];
            right--;
        }
    }
    return result;
}
```

The largest square in a sorted array (which may contain negatives) always comes from one of the two extremes, never the middle — that's the whole insight. Time O(n), Space O(1) excluding the output array.

### 🔗 Recap: Two Sum II — Input Array Is Sorted (LC 167)

**Original coverage:** Week 1, Day 6, Extra Practice.

```java
public static int[] twoSum(int[] numbers, int target) {
    int left = 0, right = numbers.length - 1;
    while (left < right) {
        int sum = numbers[left] + numbers[right];
        if (sum == target) {
            return new int[]{left + 1, right + 1};   // LeetCode 167 wants 1-INDEXED positions
        } else if (sum < target) {
            left++;
        } else {
            right--;
        }
    }
    throw new IllegalArgumentException("No solution found");
}
```

`left` and `right` start at the two ends; if the sum is too small, `left++` (only increasing `left` can increase the sum, since the array is sorted); if too large, `right--`. Time O(n), Space O(1). This is the canonical shape of opposite-ends: sortedness is what guarantees moving a pointer inward moves the sum in a predictable direction, which is what makes the greedy pointer movement provably correct rather than just a heuristic.

**⚠️ Small, easy-to-drop detail:** LC 167 asks for **1-indexed** positions in the return value, unlike almost every other problem in this ladder — `left + 1, right + 1`, not the raw 0-indexed `left, right`. Easy to lose a point on for a reason that has nothing to do with the actual algorithm.

---

### New Problem: Sum of Square Numbers (LC 633)

**Statement:** Given a non-negative integer `c`, determine whether there exist two non-negative integers `a` and `b` such that `a² + b² = c`.

**Why this one, specifically:** every opposite-ends problem so far has searched over a literal input array. This one searches over an *implicit* range — `a` from `0` to `⌊√c⌋`, `b` the same — with no array in sight. Recognizing that opposite-ends two pointers work over any monotonic, ordered domain (not just "an array someone handed you") is a genuine generalization, and exactly the kind of connection an interviewer is listening for when they ask "have you seen anything like this before?"

### Approach 1 — Brute force

```java
public static boolean judgeSquareSumBruteForce(int c) {
    for (long a = 0; a * a <= c; a++) {
        double b = Math.sqrt(c - a * a);
        if (b == Math.floor(b)) {          // is b a whole number?
            return true;
        }
    }
    return false;
}
```

For each `a` from `0` to `⌊√c⌋`, compute `b = √(c - a²)` and check whether it's a non-negative integer. Time O(√c) — already not bad, but there's a cleaner two-pointer framing that avoids a square root and a floating-point equality check inside the loop.

### Approach 2 — Optimized: opposite-ends over `[0, ⌊√c⌋]`

```java
public static boolean judgeSquareSum(int c) {
    long a = 0;
    long b = (long) Math.sqrt(c);
    while (a <= b) {
        long sum = a * a + b * b;
        if (sum == c) {
            return true;
        } else if (sum < c) {
            a++;               // increasing a can only increase a², moving the sum up
        } else {
            b--;               // decreasing b can only decrease b², moving the sum down
        }
    }
    return false;
}
```

**Why `long`, given the problem says `int c`:** `c` can be as large as `2³¹ - 1` (Day 10's exact `int` range, coming up next), which puts `⌊√c⌋` around `46,340`. Both `a` and `b` can independently get that large *during the search* — and `46340² + 46340²` is a little over `4.29 × 10⁹`, comfortably past `int`'s roughly `2.1 × 10⁹` ceiling. Declaring `a` and `b` as `long` here isn't stylistic caution, it's load-bearing: with `int` arithmetic, `sum` can silently wrap (Day 10's overflow mechanism, arriving one day early in a real example) and produce a wrong comparison against `c` with no error at all.

**Why this is correct — the same argument as LC 167, transplanted:** the function `a² + b²` is monotonic in each variable independently over non-negative integers — increasing `a` alone strictly increases the sum, decreasing `b` alone strictly decreases it. That monotonicity is *exactly* the property that made "move the pointer that pushes the sum in the needed direction" provably correct in LC 167 (there, sortedness of the array supplied the monotonicity; here, the algebra of squares supplies it directly). Same proof skeleton, different source of the monotonic guarantee.

**Worked trace:** `c = 20`. `⌊√20⌋ = 4`. `a=0, b=4`.

| a | b | a²+b² | vs. c=20 | action |
|---|---|---|---|---|
| 0 | 4 | 0+16=16 | < 20 | a++ |
| 1 | 4 | 1+16=17 | < 20 | a++ |
| 2 | 4 | 4+16=20 | == 20 | **found**: 2²+4²=20 → true |

Matches: `2² + 4² = 4 + 16 = 20`. ✓

**Complexity:** Time O(√c) — `a` and `b` together traverse a range of size `O(√c)` at most once. Space O(1).

**Edge cases:**
- `c = 0`: `a=0, b=0` immediately satisfies `0+0=0` → true.
- `c` itself a perfect square (e.g., `c=4`): `a=0, b=2` gives `0+4=4` → true immediately — don't assume both `a` and `b` need to be positive; zero is explicitly allowed.
- No valid pair exists (e.g., `c=3`): loop runs to completion (`a` and `b` cross) without a match → false. Trust the loop condition (`a <= b`) to terminate correctly rather than adding an artificial iteration cap.

**⚠️ Common Mistake:** computing `Math.sqrt(c)` inside the loop on every iteration (recomputing a square root you already have a cheaper way to track), or comparing a computed square root to an integer using `==` on a `double` — floating-point comparison for exact equality is unreliable in general. The two-pointer version sidesteps both problems entirely by staying in integer arithmetic throughout.

**💡 Interview Insight:** if asked "does this generalize?" — yes: any problem where you're searching for two values satisfying a monotonic relationship over an ordered domain is an opposite-ends candidate, whether that domain is a given array (LC 167) or an implicit numeric range you construct yourself (LC 633).

---

## Part 2 — The JVM Memory Model: Stack vs. Heap

### Prerequisites (confirmed)

- Stack frames, pushed/popped per method call (Day 8, yesterday).
- Pass-by-value for primitives (Week 1, Day 1) and the extension to objects/arrays (Week 1, Day 2) — today explains the *mechanism* behind both, rather than introducing new rules.

### The Two Regions, and What Lives Where

The JVM splits memory (per thread, broadly) into two regions with very different lifetimes:

- **The stack** — **Definition:** a fixed-size, per-thread region holding one *stack frame* per active method call (Day 8's mechanism, named again here). Each frame holds that call's local variables and parameters. For a **primitive** local variable (`int`, `double`, `boolean`, ...), the actual value sits directly in the frame. For an **object-typed** local variable, what sits in the frame is not the object itself — it's a **reference** (conceptually, an address pointing at where the object actually lives).
- **The heap** — **Definition:** the shared memory region where every object instance actually lives — every array, every `new SomeClass()`, every `String` (with one nuance covered in Day 11's String Internals). Objects on the heap are **not** tied to any single method call's lifetime; they persist until nothing anywhere references them anymore, at which point the Garbage Collector reclaims them. (Full GC mechanics are out of scope for today — extension material, not required for anything on this week's deliverables.)

```
STACK (per thread)                    HEAP (shared)
┌─────────────────────┐
│ frame: caller()       │
│  int x = 5            │──(value 5 lives right here)
│  Foo f = ──────────────┼───────────▶  ┌───────────────┐
│                        │              │ Foo instance    │
└─────────────────────┘              │  field: 10       │
                                       └───────────────┘
```

### Why This Split *Produces* Pass-By-Value — Not a Separate Rule

Java passes arguments by value, full stop — no exceptions, no special case for objects. What gets copied into the new frame when you call a method is always: for a primitive, its value; for an object-typed parameter, the **reference** (the address), copied by value.

This single fact explains both halves of the behavior Week 1 already established:

- **A method can mutate an object's fields through a passed reference**, because the copy of the reference inside the new frame still points at the *same* heap object as the caller's reference. Following that reference and changing a field changes the one object both references point at — no contradiction with "pass by value," because it's the *reference's value* (the address) that got copied, and that copy still points where the original did.
- **A method cannot make the caller's variable point at a different object**, because reassigning the *parameter* (`param = new Foo();`) only changes what the local copy of the reference points at, inside that method's own frame. The caller's original variable, in the caller's own frame, was never touched — it still holds its original reference. There is no mechanism in the language for a callee to reach back into a caller's frame and overwrite one of its local variables. This is worth stating precisely rather than assuming otherwise: **Java offers no way for a method to reassign a variable that lives in its caller's frame.** Full stop, no workaround, by design.

**🔑 Key Takeaway:** "pass-by-value" and "you can mutate object state through a method" are not in tension. What's copied is always a value; for object types, that value just happens to be an address rather than the data itself. Once you see it as "the stack always holds either data or an address, and only the value at that address is ever copied," the whole story (primitives don't visibly change, object fields visibly do, reassignment never propagates back) falls out as one consistent mechanism instead of three memorized special cases.

### The Three Cases, Side by Side, Runnable

```java
class Box {
    int value;
}

public class PassByValueDemo {

    static void modifyPrimitive(int x) {
        x = 100;                 // only this method's own local copy changes
    }

    static void mutateField(Box b) {
        b.value = 100;           // follows the shared reference — caller sees this
    }

    static void reassignReference(Box b) {
        b = new Box();           // only repoints THIS method's local copy of the reference
        b.value = 999;           // mutates the NEW object, which the caller has no reference to
    }

    public static void main(String[] args) {
        int a = 5;
        modifyPrimitive(a);
        System.out.println(a);                 // 5  — untouched

        Box original = new Box();
        original.value = 1;
        mutateField(original);
        System.out.println(original.value);    // 100 — the shared object WAS mutated

        reassignReference(original);
        System.out.println(original.value);    // still 100 — caller's reference still points at the ORIGINAL Box
    }
}
```

Three calls, three outcomes, one mechanism: `modifyPrimitive` gets its own copy of the `int` value — nothing to propagate back. `mutateField` gets its own copy of the *reference*, but that copy still points at the one real `Box` on the heap — following it and changing `.value` is visible everywhere. `reassignReference` also gets its own copy of the reference, but this time repoints *that copy* at a brand-new `Box` — the caller's `original` variable, sitting in `main`'s own frame, was never touched and still points at the first `Box`.

### Trade-offs / Design Consequences

- **Stack allocation is fast and automatic:** a frame's memory is reclaimed the instant its method returns — just moving a pointer, no scanning, no GC involvement.
- **Heap allocation is more expensive but has flexible lifetime:** an object can outlive the method that created it (e.g., returned to a caller, stored in a field, stored in a collection) precisely *because* it isn't tied to any one frame's lifetime.
- This is also why arrays (which are objects in Java, even arrays of primitives) exhibit "pass-by-value-of-a-reference" behavior identical to any other object — a method can mutate an array's contents through a passed reference, but reassigning the local array parameter to point at a brand-new array never affects the caller's array variable.

### Common Mistakes

- **⚠️ Saying "Java is pass-by-reference for objects."** It is not. The reference itself is passed by value. This is one of the most common imprecise claims in the entire language — precise enough to be worth stating exactly, since getting it slightly wrong out loud in an interview is a real signal.
- **⚠️ Assuming Strings behave like other mutable objects.** Because `String` is immutable (full mechanism in Day 11), there's no field-mutation case to even worry about for Strings specifically — every apparent "modification" produces a brand-new object, so a method can never make the caller "see" a changed String through a reference the way it could with, say, a mutable `Point` object's field.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** `PassByValueDemo` class — one method that takes an `int` and tries to modify it (verify: no effect on the caller), one method that takes an object and mutates a field (verify: caller *does* see the change), placed side by side.

Practical guidance: write a third method too, even though the plan doesn't explicitly require it — one that takes an object parameter and **reassigns** it to a brand-new instance inside the method body, then verify the caller's original object is untouched. This third case is the one people most often get wrong when reasoning about it *without* running it, so actually seeing it side-by-side with the mutation case is worth the extra five minutes. Inline comments should say explicitly, at each relevant line, whether that variable/value is on the stack or the heap.

Definition of done (from the plan): pushed, and you can explain it without notes — meaning out loud, from the mechanism (stack/heap, reference-copied-by-value), not from memorized outcomes.

## Career Block Guide (1 hr)

- **LinkedIn engagement (20 min):** same as yesterday — 3–5 substantive comments.
- **Networking:** identify 3 Engineering Managers at target companies. Practically: search target companies on LinkedIn filtered to "Engineering Manager" / "Engineering Lead" titles in relevant orgs, prioritizing anyone who has posted in the last month (they're more likely to see and respond to outreach) or who shares a mutual connection. Save names now — Day 10's career block sends the actual connection requests, so today is purely reconnaissance.

---

## Day 9 — Interview Questions

**Q1. Is Java pass-by-reference for objects?** No. Java is pass-by-value everywhere. For object-typed parameters, the value being copied is a reference (an address) — never the object itself.

**Q2. If Java is pass-by-value, how can a method change an object's state that the caller sees afterward?** The copied reference still points at the same heap object as the caller's reference. Mutating a field through that reference mutates the one shared object — the *value* that changed hands was an address, and following that address reaches the real, shared data.

**Q3. Can a Java method reassign a variable that belongs to its caller?** No — never, by design. Reassigning a parameter only changes what that parameter's own copy (in the callee's frame) points to; the caller's original variable, in the caller's own frame, is untouched.

**Q4. Where does an `int` local variable live? Where does a `new int[100]` live?** The `int` value itself sits directly in the current stack frame. The array — being an object — lives on the heap; the local variable holding it is a reference in the stack frame pointing at that heap location.

**Q5. Why does an object outlive the method that created it, but a local `int` doesn't?** The object lives on the heap, whose lifetime is governed by reachability, not by any one method call. The `int` lives in a stack frame, which is reclaimed the instant its method returns — nothing else can reference that specific frame's memory afterward.

**Q6. In LC 633, what plays the role that "the array is sorted" played in LC 167?** The algebraic monotonicity of `a² + b²` in each variable independently — increasing `a` strictly increases the sum, decreasing `b` strictly decreases it — which is exactly the property that makes greedily moving one pointer or the other provably correct, the same way sortedness did for LC 167.

**Q7. Why avoid `Math.sqrt()` inside LC 633's loop?** It recomputes a value cheaply tracked incrementally instead, and inexact floating-point results risk unreliable equality comparisons against an integer target — the two-pointer version stays entirely in integer arithmetic.

---

## Daily Deliverable Check

- [ ] Squares of a Sorted Array and Two Sum II confirmed solid from Week 1 (recap only, no re-solve needed).
- [ ] Sum of Square Numbers (LC 633) solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain pass-by-value semantics for primitives and objects, precisely — including why "pass by reference" is the wrong phrase.
- [ ] `PassByValueDemo` pushed, including the reassignment case, with stack/heap comments at each relevant line.
- [ ] 3 target-company Engineering Managers identified (for tomorrow's outreach — no requests sent yet today).

---

## What Tomorrow Assumes You Already Know Cold

Day 10 assumes today's stack/heap split is completely solid, because tomorrow's Integer cache discussion depends on knowing precisely that `Integer` objects live on the heap while the `int` primitives that get autoboxed into them do not — without today's mechanism, the cache trap would just be a memorized gotcha instead of a consequence you can derive. Two Pointers itself needs nothing further reinforced — 3Sum tomorrow reuses the opposite-ends convergence argument from today and Week 1 directly, just with an added outer loop.
