# Day 6 Resource Book — Two Pointers Begins, and SOLID Principles

**Series:** Week 1 SDE-2 Prep · [← Curriculum Map](./00_Curriculum_Map.md) · [← Day 5](./Day5_Resource_Book.md) · Next: Day 7 →

**Companion to:** Day 6 of `Week_01_Revised.md`

---

## Recap: what today formalizes

You've already implemented the core Two Pointers mechanic **twice**, on Day 2 — array reversal and the palindrome check — each time with an explicit note that it would be formalized as a named pattern later. Today is that payoff. Unlike Day 5's HashMap/HashSet family (five genuinely distinct sub-patterns), Two Pointers is really one core idea — two indices doing the work of nested loops — expressed through a small number of variants depending on how the two indices move relative to each other.

## Learning Objectives

By the end of today, without notes:

1. Identify all three Two Pointers variants covered today — opposite-ends convergence, from-the-back merging, same-direction fast/slow — and recognize which a new problem calls for.
2. Explain precisely why sortedness (or a similar structural property) is what makes Two Pointers valid in each case — this isn't a trick that works everywhere.
3. State all five SOLID principles with your own one-line example for each.
4. Explain "dependency injection" and "constructor injection" correctly, and identify a DIP violation on sight.

## Concept Dependency Map for Today

```
Arrays + iteration (Day 2) + the two informal two-pointer instances you already wrote (Day 2)
        │
        ▼
Two Pointers, formalized — three variants:
  1. Opposite ends, converging inward     (Valid Palindrome, Reverse String)
  2. From the back                          (Merge Sorted Array)
  3. Same direction, different speeds       (Remove Duplicates — NEW today)
        │
        ▼
SOLID (needs: encapsulation, inheritance, interfaces, polymorphism — Day 2 & 5)
        │
        ▼
Practice: NotificationService (DIP) · TaxCalculator refactor of Day 5's Account hierarchy
```

---

# Part 1 — Two Pointers, Formalized

### The core idea

**Two Pointers** means tracking two indices into a structure and moving them according to some rule — instead of a nested loop checking every pair — to solve a problem in a single pass. **The critical thing to understand precisely: this only works because of some structural property of the input (most commonly, sortedness) that guarantees moving a pointer in a particular direction can never skip past a valid answer.** Two Pointers is not a trick that applies to arbitrary unsorted data — it's a technique that *exploits* a property the input already has.

**Where you already built this, without the name:** Day 2's array-reversal exercise (`left`/`right` swapping inward) and Day 2's palindrome check (`left`/`right` comparing inward) are both, precisely, the "opposite ends, converging inward" variant below — you derived the mechanic from first principles before it had a label.

> 💡 **Interview Insight:** "Sorted array" combined with "find a pair," "find a triplet," "reverse in place," or "partition without extra space" is a direct, strong signal to reach for Two Pointers over a HashMap-based approach — precisely because sortedness is available to exploit, and Two Pointers achieves O(1) extra space where a hash-based approach would cost O(n).

---

## Problem 1: Valid Palindrome (LeetCode 125, Easy) — Variant: Opposite Ends

**Statement:** Given a string, determine if it's a palindrome, considering **only alphanumeric characters** and **ignoring case**.

This directly extends Day 2's palindrome check with the two complications your plan's hint calls out.

### Approach 1 — Build a cleaned copy first

```java
public static boolean isPalindromeCleanCopy(String s) {
    StringBuilder cleaned = new StringBuilder();
    for (char c : s.toCharArray()) {
        if (Character.isLetterOrDigit(c)) {
            cleaned.append(Character.toLowerCase(c));
        }
    }
    int left = 0, right = cleaned.length() - 1;
    while (left < right) {
        if (cleaned.charAt(left) != cleaned.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}
```

Correct, and genuinely readable — filter and lowercase into a new string, then run Day 2's exact two-pointer check. The cost: **O(n) extra space** for the cleaned copy.

### Approach 2 — Optimized: filter inline during the same pass

```java
public static boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) {
            left++;
        }
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) {
            right--;
        }
        if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right))) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

Same result, without the extra copy — the filtering happens *as* the pointers move, via the two inner `while` loops that skip non-alphanumeric characters from each side before every comparison.

**Why this is still O(n) despite the nested-looking `while`-inside-`while` structure — a direct callback to yesterday's Longest Consecutive Sequence reasoning:** the inner skip-loops don't each independently cost O(n) on top of the outer loop. Every character in the string is examined by *some* skip step or comparison exactly once, total, across the entire run — the total work across every inner-loop execution, summed together, is bounded by n. This is the same "looks nested, isn't multiplicative" shape from Day 5, showing up again in a new context.

> ⚠️ **Common Mistake:** Omitting the `left < right` guard *inside* the inner skip-loops — without it, an all-punctuation string could walk `left` and `right` past each other, reading out of bounds. Also common: comparing without lowercasing first, which would incorrectly reject valid mixed-case palindromes.

> 🔑 A small confirmation worth noticing: `Character.toLowerCase(char)` returns a **primitive** `char`, not a `Character` object — so the direct `!=` comparison here is correct and doesn't risk the wrapper-caching pitfall from Day 3. Comparing primitives with `!=`/`==` is always safe; it's only wrapper *objects* (`Integer`, `Character`, etc.) where reference-vs-content matters.

**Complexity: Time O(n), Space O(1)** for the optimized version (vs. O(n) extra space for the clean-copy version) — a real, worth-stating trade-off between readability and space efficiency.

**Edge cases:** empty string (trivially valid); all-punctuation string (correctly resolves to trivially valid — worth tracing by hand once to confirm the guards behave correctly when `left` and `right` converge without ever finding a comparable character pair); mixed case (`"A man, a plan, a canal: Panama"` → valid, exactly the classic example).

---

## Problem 2: Reverse String (LeetCode 344, Easy) — Variant: Opposite Ends

**Statement:** Reverse a string, given as a character array, in place, using O(1) extra space.

```java
public static void reverseString(char[] s) {
    int left = 0, right = s.length - 1;
    while (left < right) {
        char temp = s[left];
        s[left] = s[right];
        s[right] = temp;
        left++;
        right--;
    }
}
```

This is Day 2's array-reversal exercise, unchanged in every structural respect — different element type (`char` instead of `int`), identical algorithm. Worth pausing on **why LeetCode phrases this problem on a `char[]` rather than a `String` directly**: Strings are immutable (Day 2) — there is no way to reverse a `String` "in place" at all, ever, since a `String` object can never be mutated after creation. The problem is deliberately defined on a mutable `char[]` specifically to make "in place" a constraint that's actually achievable.

**Complexity: Time O(n), Space O(1)** — identical to Day 2's analysis, since this *is* Day 2's analysis.

---
## Problem 3: Merge Sorted Array (LeetCode 88, Easy) — Variant: From the Back

**Statement:** `nums1` has length `m + n`, with its first `m` elements holding real, sorted data and the remaining `n` slots empty (padded with placeholder zeros). `nums2` has `n` sorted elements. Merge `nums2` into `nums1` in place, keeping the result sorted.

### Approach 1 — Merge into a new array from the front

```java
public static void mergeNewArray(int[] nums1, int m, int[] nums2, int n) {
    int[] merged = new int[m + n];
    int i = 0, j = 0, k = 0;
    while (i < m && j < n) {
        merged[k++] = (nums1[i] <= nums2[j]) ? nums1[i++] : nums2[j++];
    }
    while (i < m) merged[k++] = nums1[i++];
    while (j < n) merged[k++] = nums2[j++];
    System.arraycopy(merged, 0, nums1, 0, m + n);
}
```

The standard two-sorted-lists merge step, from the front — correct, but needs an **O(m+n) extra** temporary array.

### Approach 2 — Optimized: merge from the *back*, in place

```java
public static void merge(int[] nums1, int m, int[] nums2, int n) {
    int i = m - 1;          // last REAL element in nums1
    int j = n - 1;          // last element in nums2
    int k = m + n - 1;      // last position in nums1 overall (including the empty padding)

    while (j >= 0) {
        if (i >= 0 && nums1[i] > nums2[j]) {
            nums1[k--] = nums1[i--];
        } else {
            nums1[k--] = nums2[j--];
        }
    }
}
```

**Why starting from the back is essential, not stylistic:** merging from the *front* directly into `nums1` would require overwriting its early elements with merged values before those elements have necessarily been read for every comparison they're still needed for — corrupting data still in use. Starting from the *back* and writing into the largest available empty slots first means every position written to has already been fully read and is no longer needed — the empty trailing padding is exactly what provides safe room to write into.

**Why the loop condition is `while (j >= 0)` alone, not `while (i >= 0 && j >= 0)`:** if `nums2` (`j`) is exhausted first while `nums1` (`i`) still has unprocessed elements, those remaining elements are *already* correctly positioned at the front — placing values from the back inward means they were never touched, and need no further action. But if `nums1`'s real data (`i`) is exhausted first while `nums2` (`j`) still has elements, those remaining elements **must** be explicitly copied in — stopping early (as `i >= 0 && j >= 0` would) leaves them uncopied, a genuine, easy-to-write bug. `while (j >= 0)` alone, with the `i >= 0` check folded into the inner condition, correctly handles both cases.

**Trace worth doing by hand once:** `nums1 = [1,2,3,0,0,0]` (m=3), `nums2 = [2,5,6]` (n=3) → expected `[1,2,2,3,5,6]`. Walking `i=2,j=2,k=5` forward: `3 vs 6` → take 6 (k=5); `3 vs 5` → take 5 (k=4); `3 vs 2` → take 3 (k=3, i decrements to 1); `2 vs 2` → take 2 from nums2 (k=2, j=-1) → loop stops (`j < 0`). Final: `[1,2,2,3,5,6]` — `nums1[0]` and `nums1[1]` were never touched, and are already correct.

**Complexity: Time O(m+n), Space O(1)** — versus the front-merge approach's same O(m+n) time but O(m+n) *extra* space.

**Edge cases:** `nums2` is empty (`n=0` — loop never runs, `nums1` is already correct); `nums1`'s real portion is empty (`m=0` — every element comes from `nums2`, correctly copied in full).

---

# Part 2 — Extra Practice: Three More Reps

## Extra Practice 1: Two Sum II — Input Array Is Sorted (LeetCode 167, Easy) — Variant: Opposite Ends

**Statement:** Given a **sorted** array and a target, return the 1-indexed positions of two numbers summing to the target.

```java
public static int[] twoSumSorted(int[] numbers, int target) {
    int left = 0, right = numbers.length - 1;
    while (left < right) {
        int sum = numbers[left] + numbers[right];
        if (sum == target) {
            return new int[]{left + 1, right + 1};   // 1-indexed, per this problem's specific requirement
        } else if (sum < target) {
            left++;    // sum too small — only increasing the left value can help, since the array is sorted
        } else {
            right--;   // sum too big — only decreasing the right value can help
        }
    }
    throw new IllegalArgumentException("No solution found");
}
```

**Why this works — the directional logic that makes opposite-ends convergence valid here:** because the array is sorted, if the current sum is too *small*, the only way to increase it is to move `left` rightward (moving `right` leftward would only select a smaller-or-equal value, making the sum *smaller* — the wrong direction). Symmetrically, if the sum is too *big*, only moving `right` leftward can help. Each step is guaranteed to eliminate at least one impossible candidate pair — this provable elimination, at every step, is what makes the technique correct rather than a lucky heuristic.

**Worth stating explicitly as a comparison to Day 5's Two Sum:** same underlying problem (find two numbers summing to target), but the *sorted* constraint here enables an O(1)-space solution, versus Day 5's O(n)-space HashMap approach for the unsorted version. If this array weren't sorted, you'd be back to a HashMap — or paying O(n log n) to sort first, a trade-off worth naming if it comes up (sorting first only pays off if you don't already have the O(n) space to spare, since the HashMap approach is faster in raw time).

**Complexity: Time O(n), Space O(1). Edge cases:** duplicate values (handled correctly — the algorithm only cares about sums, not uniqueness); negative numbers (sortedness holds regardless of sign, so the logic is unaffected).


## Extra Practice 2: Squares of a Sorted Array (LeetCode 977, Easy) — Variant: Opposite Ends (From the Back)

**Statement:** Given a sorted array (which may include negative numbers), return an array of the squares of each number, also sorted.

### Approach 1 — Square everything, then sort

```java
public static int[] sortedSquaresBruteForce(int[] nums) {
    int[] result = new int[nums.length];
    for (int i = 0; i < nums.length; i++) {
        result[i] = nums[i] * nums[i];
    }
    Arrays.sort(result);
    return result;
}
```

Correct, but **O(n log n)** — the sort dominates, and it ignores the sortedness already present in the input entirely.

### Approach 2 — Optimized: two pointers, filling the result from the back

```java
public static int[] sortedSquares(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    int left = 0, right = n - 1;
    int resultIndex = n - 1;

    while (left <= right) {
        int leftSquare = nums[left] * nums[left];
        int rightSquare = nums[right] * nums[right];
        if (leftSquare > rightSquare) {
            result[resultIndex] = leftSquare;
            left++;
        } else {
            result[resultIndex] = rightSquare;
            right--;
        }
        resultIndex--;
    }
    return result;
}
```

**The insight:** because the *original* array is sorted but may contain negatives, the **largest** squares must come from one of the two *ends* (the most negative or the most positive value — whichever has the larger magnitude) — never from the middle, where values are closest to zero and therefore have the smallest squares. Comparing the two end-candidates at each step and taking the larger square identifies the results in largest-to-smallest order — which is exactly why filling the result array **from the back forward** is the natural fit, avoiding any need to reverse afterward. This is a second instance of the "from the back" idea, reinforcing yesterday's Merge Sorted Array insight in a new context.

**Complexity: Time O(n), Space O(n)** for the required output (O(1) *extra* beyond it) — versus O(n log n) for the sort-based approach.

**Edge cases:** all non-negative input (the right pointer's squares always "win," gracefully degenerating to a simple in-order copy with no special-casing needed); all non-positive input (symmetric, left pointer always wins); an array containing zero (handled correctly regardless of which side it falls on).

---

## Extra Practice 3: Remove Duplicates from Sorted Array (LeetCode 26, Easy) — Variant: Same Direction, Different Speeds

**Statement:** Given a sorted array, remove duplicates in place so each unique element appears once, and return the new length. Must use O(1) extra space.

This is the **third distinct Two Pointers variant**, genuinely different in shape from the previous two — worth naming explicitly as such rather than treating it as "more of the same."

```java
public static int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;

    int slow = 0;   // marks the last position of the "confirmed unique" section
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow]) {
            slow++;
            nums[slow] = nums[fast];
        }
    }
    return slow + 1;
}
```

**Why this is a different flavor from opposite-ends convergence:** both pointers start at (or near) the *same* end and move in the *same* direction, but at different effective speeds. `fast` advances unconditionally every iteration, scanning ahead. `slow` only advances when a genuinely new, not-yet-recorded value is found — it marks the boundary of the "processed, unique-so-far" region and only moves forward to *write* a new value there. This "slow marks a write boundary, fast explores ahead" shape — often called **fast/slow** or **read/write pointers** — is extremely common and worth recognizing on sight as its own variant, distinct from convergence.

**Why this works *specifically because the array is sorted*:** in a sorted array, duplicates are guaranteed to be *adjacent*. Comparing `nums[fast]` only against `nums[slow]` (the single most recently confirmed unique value) is therefore sufficient to catch every duplicate — no auxiliary tracking structure needed at all. This is a genuinely useful contrast with Days 4-5's instinct: "duplicates" doesn't automatically mean "reach for a HashSet" — the *sorted* precondition here changes the calculus entirely, enabling an O(1)-space solution where a HashSet would cost O(n) unnecessarily. Matching the tool to the actual, specific constraints of the problem — not defaulting to the same structure reflexively — is exactly the judgment this series has been building toward.

**Complexity: Time O(n) — `fast` visits every element once; Space O(1).**

**Edge cases:** empty array (`0`, handled by the early check); all-identical elements (`slow` never advances past `0`, correctly returns length `1`); no duplicates at all (`slow` advances in lockstep with `fast`, returning the full original length). **Worth flagging explicitly, since it's a common point of confusion when first reading this problem:** the array's contents *beyond* the returned length are allowed to be anything — the problem only cares that the first `returnedLength` positions hold the correct unique values in order.


---

# Part 3 — SOLID Principles

SOLID is five design principles for object-oriented code, each addressing a specific way a design can become fragile or hard to change. All five build directly on Day 2's and Day 5's OOP foundations — encapsulation, inheritance, interfaces, and polymorphism are the raw tools; SOLID is guidance on how to *use* those tools well.

## S — Single Responsibility Principle (SRP)

**Definition:** a class should have exactly one reason to change.

**A violation, concretely:** a class that both calculates account interest *and* formats/prints account statements has two unrelated jobs bundled together. A change to interest calculation rules and a change to statement formatting are two entirely unrelated reasons this one class might need modification — tangled together for no structural reason.

**The fix:** split into `InterestCalculator` and `StatementFormatter`, each with exactly one job, and exactly one reason to ever change.

## O — Open/Closed Principle (OCP)

**Definition:** software entities should be **open for extension, but closed for modification** — you should be able to add new behavior without changing existing, already-tested code.

**A violation, concretely:** a method with a big `if/else` chain checking account *type* to decide how to calculate interest (`if (type == SAVINGS) {...} else if (type == CHECKING) {...}`) must be edited every time a new account type is introduced.

**Where you already built the fix, without the name attached:** yesterday's `Account` abstract class with `calculateInterest()` as an abstract method. Adding a new account type today requires only a *new subclass* — the existing `Account` class, and any code operating against the `Account` contract, never needs to change. Yesterday's design wasn't just a tidy OOP exercise — it was already OCP-compliant.

## L — Liskov Substitution Principle (LSP)

**Definition:** any object of a subclass should be replaceable with an object of its superclass without affecting the correctness, expected behavior, or observable outcomes of the program.

**The classic illustration, worth knowing precisely:** `Square extends Rectangle`, where `setWidth()` is overridden to also change the height (to preserve "squareness"). Code written to work correctly with *any* `Rectangle` — "set width to 5, set height to 10, expect area 50" — silently breaks if handed a `Square`: setting width to 5 then height to 10 leaves a square with side 10 (the second call overwrote the first), giving area 100, not the expected 50. The subclass is a `Rectangle` in a purely structural, "is-a" sense, but it violates the *behavioral contract* the superclass established — an LSP violation, even though the inheritance relationship looks perfectly reasonable on paper.

## I — Interface Segregation Principle (ISP)

**Definition:** clients shouldn't be forced to depend on methods they don't actually use — prefer several small, specific interfaces over one large, general-purpose one.

**A violation, concretely:** an interface `Worker` declaring both `work()` and `eat()` forces a `RobotWorker` class — which doesn't eat — to implement a meaningless, empty `eat()` method purely to satisfy the contract.

**The fix:** split into separate `Workable` and `Eatable` interfaces. `RobotWorker` implements only `Workable`; a `HumanWorker` can implement both.

## D — Dependency Inversion Principle (DIP)

**Definition:** high-level modules shouldn't depend directly on low-level modules — both should depend on a shared abstraction instead.

This principle gets the deepest treatment today, since it's your plan's own coding exercise.

### The violation

```java
public class EmailSender {
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

public class NotificationService {
    private EmailSender emailSender = new EmailSender();   // hard-coded dependency on a CONCRETE class

    public void notify(String message) {
        emailSender.send(message);
    }
}
```

`NotificationService` — a "high-level" module representing the general business capability "send a notification" — directly depends on `EmailSender`, a "low-level" module representing one specific, concrete way notifications currently happen to be sent. Two concrete problems fall out of this: wanting to notify via SMS or push instead of (or alongside) email requires *modifying* `NotificationService` itself (an OCP violation too — these principles reinforce each other in practice), and `NotificationService` is nearly impossible to unit test in isolation, since it's permanently welded to a real `EmailSender`.

### The refactor: depend on an abstraction, inject the concrete implementation

```java
public interface MessageSender {
    void send(String message);
}

public class EmailSender implements MessageSender {
    @Override
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

public class SmsSender implements MessageSender {
    @Override
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

public class NotificationService {
    private final MessageSender messageSender;

    public NotificationService(MessageSender messageSender) {   // CONSTRUCTOR INJECTION
        this.messageSender = messageSender;
    }

    public void notify(String message) {
        messageSender.send(message);
    }
}
```

```java
NotificationService emailNotifier = new NotificationService(new EmailSender());
NotificationService smsNotifier = new NotificationService(new SmsSender());
```

**Why this is genuinely "inversion," not just "using an interface":** in the original version, `NotificationService` depends *downward* directly on `EmailSender`'s concrete details. In the refactored version, `NotificationService` depends only on the `MessageSender` **abstraction** — and `EmailSender` *also* depends on (implements) that same abstraction. Both the high-level and low-level modules now depend on a shared abstraction sitting between them, rather than one depending directly on the other — the traditional dependency direction is inverted.

**Vocabulary worth having exactly right:** passing a dependency *in* from outside, rather than a class constructing it internally, is called **dependency injection**. Doing this specifically via the constructor — as above — is **constructor injection**, generally preferred over alternatives like setter injection (which can leave an object partially, invalidly initialized between construction and the setter call). This vocabulary comes up constantly in real engineering conversations — it's the foundational idea behind dependency-injection frameworks (e.g., Spring) you'll very likely encounter later in backend-focused interview prep.

**The concrete payoff, worth stating explicitly:** `NotificationService` is now trivially unit-testable (pass in a fake/mock `MessageSender`, no real email ever sent during a test run) and extensible (a new `MessageSender` implementation requires zero changes to `NotificationService` — OCP compliance, achieved *through* DIP).


---

# Section — Project Block: Applying SOLID to Yesterday's `Account` Hierarchy

**Task:** extract a `TaxCalculator` interface so new tax rules can be added without modifying `Account` at all.

```java
public interface TaxCalculator {
    double calculateTax(double interest);
}

public class StandardTaxCalculator implements TaxCalculator {
    private static final double TAX_RATE = 0.20;

    @Override
    public double calculateTax(double interest) {
        return interest * TAX_RATE;
    }
}
```

```java
public double getTaxOnInterest(Account account, TaxCalculator taxCalculator) {
    return taxCalculator.calculateTax(account.calculateInterest());
}
```

**This one small refactor demonstrates three SOLID principles simultaneously, worth naming explicitly:** **SRP** — `Account` now manages only account state and interest math; tax rules live entirely in `TaxCalculator`, two genuinely separate reasons to change, now separated. **OCP** — a new tax rule (a progressive bracket system, a regional variation) means a new `TaxCalculator` implementation, with zero changes to `Account`. **DIP** — `getTaxOnInterest` depends on the `TaxCalculator` *abstraction*, not a concrete tax implementation baked directly into `Account`.

**Definition of done:** account state and tax logic are cleanly separated (verify: does `Account` reference tax rates or tax logic anywhere at all? It shouldn't); pushed.

---

# Section — Career Block

- **LinkedIn Post 3:** one genuine insight from the Four Pillars / SOLID work — a real "here's a design decision I understood differently after building it" observation reads far better than a generic summary of definitions.
- **Networking:** send 5 connection requests, each with a short, genuine, specific note (same bar as Day 1 and Day 2 — no generic templates).

---

# Day 6 — Interview Questions

---

**1. What structural property makes Two Pointers valid, and why doesn't it work on arbitrary unsorted data?**

*Answer:* Two Pointers relies on some structural guarantee (most commonly sortedness) that proves moving a pointer in a given direction can never skip past a valid answer. Without such a guarantee, there's no basis for deciding which direction to move a pointer, and the technique isn't applicable.

---

**2. Name the three Two Pointers variants covered today.**

*Answer:* Opposite ends, converging inward (Valid Palindrome, Reverse String, Two Sum II, Squares of a Sorted Array); from the back (Merge Sorted Array); same direction, different speeds — fast/slow (Remove Duplicates from Sorted Array).

---

**3. Why is Valid Palindrome's nested while-inside-while structure still O(n)?**

*Answer:* Every character is examined by some skip-step or comparison exactly once, total, across the whole run — the total work across all inner-loop executions combined is bounded by n, the same "looks nested, isn't multiplicative" shape as Day 5's Longest Consecutive Sequence.

---

**4. Why does LeetCode define Reverse String on `char[]` rather than `String`?**

*Answer:* Strings are immutable — there's no way to reverse a String "in place" at all. The problem uses a mutable `char[]` specifically so "in place, O(1) extra space" is an achievable constraint.

---

**5. Why must Merge Sorted Array merge from the back rather than the front?**

*Answer:* Merging from the front into `nums1` directly would overwrite early elements before they've necessarily been read for comparison, corrupting data still needed. Merging from the back writes only into positions already fully read, using the empty trailing padding as safe space.

---

**6. Why is the loop condition `while (j >= 0)` alone, not `while (i >= 0 && j >= 0)`, in Merge Sorted Array?**

*Answer:* If `nums1`'s real elements are exhausted first while `nums2` still has elements, those remaining `nums2` elements must be explicitly copied — stopping early would leave them uncopied. If `nums2` finishes first, `nums1`'s remaining elements are already correctly positioned and need no action, so continuing to drive on `j` alone handles both cases correctly.

---

**7. Two Sum II (sorted) vs. Two Sum (Day 5, unsorted) — same problem shape, different approach. What's the trade-off?**

*Answer:* Sortedness in Two Sum II enables O(1)-space two pointers; the unsorted Day 5 version needs a HashMap at O(n) space. If the array weren't sorted, you'd either use the HashMap approach or pay O(n log n) to sort first — worth it only if the O(n) space of the HashMap isn't available.

---

**8. In Two Sum II, why does moving `left` forward help when the current sum is too small?**

*Answer:* The array is sorted ascending, so `left` holds the smallest remaining candidate. Moving `left` forward can only increase the value it points to (and thus the sum); moving `right` instead would only decrease the sum further — the wrong direction.

---

**9. Why does Squares of a Sorted Array fill its result array from the back?**

*Answer:* Because the largest squares come from one of the two ends (largest magnitude, positive or negative), comparing both ends at each step identifies the largest remaining square first — naturally producing results in largest-to-smallest order, which fits filling the output array from its last position backward.

---

**10. What's different about Remove Duplicates' two-pointer shape compared to opposite-ends convergence?**

*Answer:* Both pointers start near the same end and move in the same direction, at different speeds — `fast` scans every element unconditionally, `slow` only advances to write when a genuinely new unique value is found. This "read/write, fast/slow" shape is structurally distinct from two pointers converging from opposite ends.

---

**11. Why does Remove Duplicates work correctly comparing only against the single most recent unique value, with no HashSet?**

*Answer:* In a sorted array, duplicates are guaranteed to be adjacent — so comparing each new element only to the last confirmed-unique value is sufficient to catch every duplicate, with no need to remember anything beyond that single value.

---

**12. State the Single Responsibility Principle with an example.**

*Answer:* A class should have exactly one reason to change. Example: a class that both calculates interest and formats statements has two unrelated reasons to change tangled together; splitting into `InterestCalculator` and `StatementFormatter` gives each exactly one.

---

**13. State the Open/Closed Principle. How does the `Account` abstract class from Day 5 already satisfy it?**

*Answer:* Open for extension, closed for modification — new behavior should be addable without changing existing, tested code. `Account`'s abstract `calculateInterest()` means a new account type is a new subclass; the existing `Account` class and any code using it never needs to change.

---

**14. Explain the classic Square/Rectangle LSP violation precisely.**

*Answer:* `Square extends Rectangle`, overriding `setWidth()` to also change height (preserving squareness). Code written against any `Rectangle` — set width 5, set height 10, expect area 50 — breaks when given a `Square`, since the second call silently overwrites the first, yielding a 10×10 square (area 100). The subclass is structurally an "is-a" Rectangle but violates the superclass's behavioral contract.

---

**15. State the Interface Segregation Principle with an example.**

*Answer:* Clients shouldn't be forced to depend on methods they don't use. Example: a `Worker` interface with both `work()` and `eat()` forces a `RobotWorker` to implement a meaningless `eat()`; splitting into `Workable` and `Eatable` lets `RobotWorker` implement only what it needs.

---

**16. State the Dependency Inversion Principle. Why does the original `NotificationService` (with `new EmailSender()` inside it) violate it?**

*Answer:* High-level modules shouldn't depend directly on low-level modules — both should depend on a shared abstraction. The original `NotificationService` (high-level) directly instantiates and depends on the concrete `EmailSender` (low-level), making it impossible to swap notification channels or unit-test in isolation without a real `EmailSender`.

---

**17. What is dependency injection, and what specifically is "constructor injection"?**

*Answer:* Dependency injection is supplying a dependency to a class from outside, rather than the class constructing it internally. Constructor injection specifically passes it in via the constructor — generally preferred over setter injection, since it guarantees the object is never in a partially-initialized state.

---

**18. Why is the refactored `NotificationService` easier to unit test than the original?**

*Answer:* It depends only on the `MessageSender` interface, so a test can inject a fake/mock implementation instead of a real `EmailSender` — no real email is ever sent during a test run, and behavior can be verified in isolation.

---

## Daily Deliverable Check

- [ ] Valid Palindrome, Reverse String, and Merge Sorted Array (+ the three extra reps) solved and pushed to `dsa-java/two-pointers/`
- [ ] Can explain all five SOLID principles with a one-line example each, without notes
- [ ] `TaxCalculator` refactor pushed, with account state and tax logic cleanly separated

---

## What Tomorrow Assumes You Already Know Cold

Day 7 finishes Two Pointers for the week with two more variants, and opens with a self-check requiring one-sentence justifications for `ArrayList`/`HashSet`/`HashMap`/`ArrayDeque` — if that self-check would currently require you to flip back through this series rather than answer immediately, that's worth closing before continuing. One of Day 7's extra problems (3Sum) directly combines today's Two Pointers with Day 5's HashMap-era thinking — both need to be genuinely automatic, not just individually understood.

**Next:** [Day 7 Resource Book](./Day7_Resource_Book.md) — Two Pointers Continues, and Week 1 Consolidation.
