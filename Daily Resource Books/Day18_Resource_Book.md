# SDE-2 Resource Book Series
## Day 18 — Sliding Window: At-Most-K-Distinct, and Exception Handling

**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**← Previous:** [Day 17](Day17_Resource_Book.md) &nbsp;|&nbsp; **Next →:** [Day 19](Day19_Resource_Book.md)
**Companion to:** Day 18 of `Week_03_Revised.md`

---

### Recap

Both of today's problems are instances of Day 15's shrink-until-valid (longest-window) template — Fruit Into Baskets introduces a new *kind* of validity check (distinct-key count, via a HashMap, rather than a violation counter), and Longest Subarray of 1's After Deleting One Element reuses Day 15's exact zero-counting check from Max Consecutive Ones III with one extra twist at the end. Exception Handling, today's theory, is a fresh thread — its one real dependency is Day 8/9's fact that `StackOverflowError` is an `Error`, not a `RuntimeException`, which you'll now see fits into a bigger hierarchy.

---

### Learning Objectives

By the end of today, without notes, you should be able to:
1. Solve Fruit Into Baskets by recognizing it as "longest window with at most 2 distinct values" and generalize the shrink condition to a HashMap's size.
2. Solve Longest Subarray of 1's After Deleting One Element, and explain precisely why the answer requires subtracting 1 — including in the all-ones edge case.
3. Draw the `Throwable` hierarchy from memory and place `RuntimeException`, checked exceptions, and `Error` correctly within it.
4. Explain what `try-with-resources` does mechanically, and why it fixes a real bug that manual `try/finally` has.

---

### Concept Dependency Map

```
Day 15 — LC 1004 (shrink-until-valid,          Day 15 — LC 1004's exact zero-counting check
   violation counter: zeroCount > k)                        │
        │                                                    ▼
        ▼                                          Day 18: LC 1493 — same check, PLUS
Day 18: LC 904 — same shape,                          mandatory-deletion adjustment (−1)
   NEW check: HashMap.size() > 2
        │
        ▼
  LC 1838 (Extra) — sort + window with a "budget" constraint (new sub-flavor)

Day 8/9 — StackOverflowError IS-AN Error, not a RuntimeException
        │
        ▼
Day 18: Exception Handling — full Throwable hierarchy,
   checked vs. unchecked, try-with-resources / AutoCloseable
        │
        ▼
  CacheMissException + try-with-resources demo (Project)
```

---

## Problem 9: Fruit Into Baskets

**LeetCode #904 — Medium — Pattern: Sliding Window (at-most-K-distinct, K=2)**

**Statement:** given an array of tree types, pick a contiguous subarray using at most 2 distinct types, maximizing its length.

**Brute force:** for every subarray, count distinct types. O(n²) or O(n³).

**Optimal — this is "longest window with at most 2 distinct values" wearing a costume:**

```java
public int totalFruit(int[] fruits) {
    Map<Integer, Integer> basket = new HashMap<>();
    int left = 0, best = 0;
    for (int right = 0; right < fruits.length; right++) {
        basket.merge(fruits[right], 1, Integer::sum);
        while (basket.size() > 2) {
            int leftType = fruits[left];
            basket.put(leftType, basket.get(leftType) - 1);
            if (basket.get(leftType) == 0) basket.remove(leftType);
            left++;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Why it works:** identical shrink-until-valid shape to Day 15's Max Consecutive Ones III — only the validity check changes, from a violation counter to "how many distinct keys does the window's frequency map currently have." `basket.size() > 2` is the new violation condition; removing a key entirely (not just decrementing) once its count hits zero is what keeps `size()` accurate.

**Trace:** `fruits = [1,2,1,2,3]`.

The window grows cleanly through indices 0–3 (`basket = {1:2, 2:2}`, size 2, `best=4`). At `right=4` (type `3`), `basket.size()` becomes 3 → shrink: remove `fruits[0]=1` (count 2→1, key survives), `fruits[1]=2` (count 2→1, key survives), `fruits[2]=1` (count 1→0, key **removed**) — three single-step shrinks before `size()` drops back to 2 (`{2:1, 3:1}`). Final window `[3,4]`, length 2. `best` stays **4**.

**Complexity:** Time O(n) — total-movement argument, as always. Space O(1) — the map holds at most 3 keys at any moment (2 valid + 1 that triggers the shrink).

**Edge cases & mistakes:**
- ⚠️ Decrementing a key's count without checking for zero and removing it — `basket.size()` would then never shrink, since a `0`-count key still counts as "present" in the map.
- ⚠️ All-one-type input: `basket.size()` never exceeds 1; the whole array is the answer, no special case needed.

**💡 Interview framing:** the single most valuable sentence you can say here: "this is 'longest window with at most K distinct values' with K hardcoded to 2 — I'd write it as a general `atMostKDistinct(nums, k)` helper if K were a parameter." 🔗 That's exactly what tomorrow's Longest Substring with At Most K Distinct Characters asks you to actually do.

---

## Problem 10: Longest Subarray of 1's After Deleting One Element

**LeetCode #1493 — Medium — Pattern: Sliding Window (variable)**

**Statement:** given a binary array, delete exactly one element, then return the length of the longest contiguous run of 1s remaining.

**Brute force:** try deleting each index, recompute the longest run of 1s in the resulting array each time. O(n²).

**Optimal — "longest window with at most one zero," then subtract one for the mandatory deletion:**

```java
public int longestSubarray(int[] nums) {
    int left = 0, zeroCount = 0, maxWindow = 0;
    for (int right = 0; right < nums.length; right++) {
        if (nums[right] == 0) zeroCount++;
        while (zeroCount > 1) {
            if (nums[left] == 0) zeroCount--;
            left++;
        }
        maxWindow = Math.max(maxWindow, right - left + 1);
    }
    return maxWindow - 1;
}
```

**Why the `−1` is required even when the best window has zero zeros in it — this is the part worth proving, not assuming:** the deletion in this problem is **mandatory**, regardless of whether there's a zero available to delete. If the longest "at most one zero" window happens to contain a zero, you delete that zero, and the remaining run of 1s has length `windowLength − 1`. If the longest such window happens to contain *zero* zeros (only possible if the array segment is all 1s), you still must delete *something* — so you delete one of the 1s, and the remaining run is again `windowLength − 1`. Either way, the answer is `maxWindow − 1`, unconditionally — there is no case where you keep the full window length.

**Trace — the edge case that makes this concrete:** `nums = [1,1,1,1]` (all ones, zero zeros anywhere). `zeroCount` never exceeds 0, so the shrink loop never fires, and `maxWindow` grows to `4` (the whole array). Return `4 − 1 = 3`. Manually: delete any single `1`, and the two remaining runs of 1s on either side become adjacent (since deleting the middle element of an array closes the gap) — length 3. Confirmed.

A second trace on `nums = [0,1,1,1,0,1,1,0,1]` gives `maxWindow = 6` (the window `[1,6]`, one zero at index 4), so the answer is `6 − 1 = 5`.

**Complexity:** Time O(n), Space O(1) — identical to Max Consecutive Ones III, since the mechanism is the same check.

**Edge cases & mistakes:**
- ⚠️ **The single most common bug on this exact problem:** forgetting the `−1` in the all-ones case, because it "feels" like you shouldn't be penalized for having no zero to delete. The problem statement mandates exactly one deletion regardless — the arithmetic above shows why the penalty applies uniformly.
- ⚠️ Single-element array `[0]` or `[1]`: after the mandatory deletion, nothing remains — the formula still gives `maxWindow − 1 = 1 − 1 = 0`, which is correct (an empty subarray has length 0).

**💡 Interview framing:** "I'll solve 'longest window with at most one zero' with Day 15's exact template, then subtract one — because the deletion is mandatory even in the no-zero case." Stating the mandatory-deletion reasoning *before* being asked heads off the most likely follow-up entirely.

---

## Extra Practice: Frequency of the Most Frequent Element

**LeetCode #1838 — Medium — Pattern: Sliding Window (variable) over a sorted array, budget-constrained**

✅ **Overlap check:** absent from `00_Curriculum_Map.md`'s inventory and from `Week_04_Revised.md`. Added because this is a genuinely distinct sliding-window sub-flavor — sort first, then window with a numeric "budget" — that none of today's or the week's other problems cover, and it's a very real, frequently-tested combination.

**Statement:** given `nums` and an operation budget `k` (each operation increments one element by 1, any number of operations on the same element allowed), return the maximum possible frequency of any single value after spending at most `k` operations.

**Brute force:** for each candidate target value, compute the cost to raise every element ≤ it up to it, within budget. Without sorting first, this is expensive to evaluate per-candidate; O(n²) or worse.

**Optimal — sort ascending, then a variable window with a cost check:**

```java
public int maxFrequency(int[] nums, int k) {
    Arrays.sort(nums);
    long windowSum = 0;
    int left = 0, best = 1;
    for (int right = 0; right < nums.length; right++) {
        windowSum += nums[right];
        while ((long) nums[right] * (right - left + 1) - windowSum > k) {
            windowSum -= nums[left];
            left++;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Why sorting first, and why target the window's own rightmost element:** you can only *increment* elements (never decrement), so for a fixed window, the cheapest value to make everything equal to is the window's own **maximum** — targeting anything higher would cost strictly more (you'd still need to raise every smaller element that far, plus raise the current maximum too), and you're not allowed to lower the maximum down to meet a smaller target. Once sorted, a window's maximum is always its rightmost element, `nums[right]`. The cost to raise every element in `[left, right]` up to `nums[right]` is `nums[right] * (windowSize) − windowSum` (target value repeated windowSize times, minus what's actually there) — if that cost exceeds `k`, shrink from the left, which both drops the most expensive-to-raise element and reduces `windowSum` accordingly.

**Trace:** `nums = [1,2,4]`, `k = 5` (already sorted).

| right | windowSum | cost = nums[right]·(size) − windowSum | shrink? | best |
|---|---|---|---|---|
| 0 | 1 | 1·1−1=0 | no | 1 |
| 1 | 3 | 2·2−3=1 | no | 2 |
| 2 | 7 | 4·3−7=5 | no (5≤5) | 3 |

Final answer: **3** (raise `1→4` and `2→4`, costing `3+2=5`, exactly the budget — all three elements become `4`).

**Complexity:** Time O(n log n) — dominated by the initial sort; the window pass itself is O(n) by the usual total-movement argument. Space O(1) extra (in-place sort aside).

**Edge cases & mistakes:**
- ⚠️ **Overflow:** `nums[right] * (right - left + 1)` can exceed `int` range for large inputs (values up to ~10^5, window sizes up to ~10^5 → product up to ~10^10) — cast to `long` before multiplying, as the code above does. This is exactly the overflow-wraparound danger from Day 10's Primitives lesson, resurfacing in a new context.
- ⚠️ Trying to target a value *other* than the window's current maximum "to save operations" — the proof above shows this can never be cheaper, so there's no case worth checking.

**💡 Interview framing:** "sorting turns this into 'for a window, what's the cheapest value to unify everyone to' — and once sorted, that's always the window's own rightmost element, since I can only push values up, never down." That single sentence is the entire justification an interviewer is listening for.

---

## Theory: Exception Handling

### The `Throwable` hierarchy

```
Throwable
├── Error                         (serious JVM-level problems — not meant to be caught/recovered)
│   └── ... e.g. StackOverflowError, OutOfMemoryError
│       🔗 Day 8/9: StackOverflowError IS-AN Error, not a RuntimeException — this is why
└── Exception
    ├── (checked exceptions)      any Exception subclass that ISN'T a RuntimeException
    │   └── e.g. IOException, SQLException
    └── RuntimeException          (unchecked)
        └── e.g. NullPointerException, ArrayIndexOutOfBoundsException, IllegalArgumentException
```

**Checked exceptions** must be declared (`throws`) or caught — the compiler enforces this at compile time. They represent conditions a caller might reasonably need to react to (a file that might not exist, a network call that might fail).

**Unchecked exceptions** (`RuntimeException` and its subclasses) have no compiler enforcement — they typically represent programmer errors (a null dereference, an out-of-bounds index) rather than external, recoverable conditions.

### `try-with-resources`

Any class implementing `AutoCloseable` (single method, `close()`) can be declared inside a `try (...)` header — its `close()` is called automatically at the end of the block, **even if an exception was thrown**, and (with multiple resources) in reverse order of declaration.

```java
class SimulatedFileReader implements AutoCloseable {
    private final String filename;

    SimulatedFileReader(String filename) {
        this.filename = filename;
        System.out.println("Opening " + filename);
    }

    String read() {
        if (filename.equals("missing.txt")) {
            throw new CacheMissException("Cache miss for: " + filename);
        }
        return "contents of " + filename;
    }

    @Override
    public void close() {
        System.out.println("Closing " + filename);
    }
}
```

🔑 **Key Takeaway — the actual bug this fixes:** with a manual `try { ... } finally { resource.close(); }`, if **both** the try block *and* `close()` throw, the `finally` block's exception silently **replaces** the original one — the real cause of the failure is lost. `try-with-resources` fixes this precisely: if both throw, the try block's exception is the one that propagates, and `close()`'s exception is attached to it as a *suppressed* exception (retrievable via `getSuppressed()`) instead of overwriting it.

⚠️ **Common Mistake:** assuming `try-with-resources` only matters for files/streams. Anything implementing `AutoCloseable` qualifies — database connections, custom resources like the simulated reader above, locks, and more.

### The checked-exceptions debate

Java is unusual among mainstream languages (C++, C#, Python, Kotlin, and others don't have compiler-enforced checked exceptions) in having this feature at all, and it's a genuinely long-running internal debate:

- **For:** forces callers of an API to explicitly handle or acknowledge conditions the API's author knows can fail — useful when the caller genuinely needs to make a decision (e.g., "retry" vs. "fail" on a network error).
- **Against:** in practice, checked exceptions push toward boilerplate — empty `catch` blocks, blanket `throws Exception` just to satisfy the compiler, and swallowed errors — and they interact badly with functional-style code (a lambda passed to a `Stream` operation can't throw a checked exception without extra wrapping). Much of modern Java practice (including large frameworks like Spring) leans toward unchecked exceptions for most application-level errors specifically because of this friction.

💡 Having a real, defensible opinion here — not just "checked exceptions are bad" — is genuinely worth being able to articulate; it's a common conversational interview topic precisely because there's no single correct answer.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals` · **Task:** `CacheMissException` and the try-with-resources demo

```java
class CacheMissException extends RuntimeException {
    public CacheMissException(String message) {
        super(message);
    }
}

public class CacheMissDemo {
    public static void main(String[] args) {
        try (SimulatedFileReader reader = new SimulatedFileReader("missing.txt")) {
            System.out.println(reader.read());
        } catch (CacheMissException e) {
            System.out.println("Logged cleanly: " + e.getMessage());
        }
    }
}
```

Running this prints, in order: `"Opening missing.txt"`, then `"Closing missing.txt"`, then `"Logged cleanly: Cache miss for: missing.txt"` — the close-before-catch ordering is your proof that `close()` ran even though `read()` threw.

**Definition of done:** pushed, with a comment stating when you'd choose checked vs. unchecked for your own custom exception types (and why `CacheMissException` above is unchecked — it's a "cache didn't have it" signal for the *calling code* to react to programmatically, arguably close to a borderline case worth defending either way).

---

## Career Block Guide (1 hr)

- **LinkedIn engagement (20 min):** 3–5 posts, substantive comments.
- **Networking — 2 college alumni at target companies:** alumni outreach converts well specifically because the shared affiliation is a built-in, non-generic reason to reach out — lead your message with it explicitly rather than burying it.

---

## Day 18 — Interview Questions

**Q1. Fruit Into Baskets is secretly which more general pattern?**
A: "Longest window with at most K distinct values," with K fixed at 2.

**Q2. In Fruit Into Baskets, why remove a key from the map entirely once its count hits zero, rather than leaving it at 0?**
A: `basket.size()` is what drives the validity check — a lingering zero-count key would still count toward `size()`, making the window look invalid when it's actually fine.

**Q3. Why must you subtract 1 from the "at most one zero" window length in Longest Subarray of 1's After Deleting One Element, even for an all-ones array?**
A: The deletion is mandatory regardless of whether a zero is available — if there's a zero, delete it (−1 from the window); if there isn't, you still must delete something, one of the 1s (also −1). Either way the answer is `window length − 1`.

**Q4. Draw (or describe) where `StackOverflowError` sits in the `Throwable` hierarchy, and why that placement matters.**
A: `Throwable → Error → StackOverflowError`. It's an `Error`, not a `RuntimeException`, meaning the JVM doesn't expect it to be routinely caught and handled — it signals a serious runtime condition (stack exhaustion), not an ordinary programmer error.

**Q5. What's the actual bug `try-with-resources` fixes, versus manual `try/finally`?**
A: If both the try block and the cleanup code throw, manual `finally` lets the cleanup's exception silently replace the original, losing the real cause; `try-with-resources` instead propagates the original exception and attaches the cleanup's exception as suppressed, preserving both.

**Q6. Give one argument for checked exceptions and one against.**
A: For: they force callers to explicitly acknowledge conditions the API author knows can fail. Against: in practice they push toward boilerplate (empty catches, blanket `throws Exception`) and interact poorly with functional-style/lambda code.

**Q7. In Frequency of the Most Frequent Element, why sort the array first?**
A: Sorting means a window's maximum is always its rightmost element, and since you can only increment (never decrement) values, the cheapest target for a window is always that window's own maximum — sorting is what makes "the rightmost element" and "the maximum" the same thing.

---

## Daily Deliverable Check

- [ ] Fruit Into Baskets (LC 904) and Longest Subarray of 1's After Deleting One Element (LC 1493) solved, pushed.
- [ ] **Extra:** Frequency of the Most Frequent Element (LC 1838) solved, same folder.
- [ ] Can explain checked vs. unchecked exceptions and when try-with-resources helps, without notes.
- [ ] `CacheMissException` and demo pushed, with the checked-vs-unchecked reasoning comment.

---

### What Tomorrow Assumes You Already Know Cold

Day 19 generalizes today's Fruit Into Baskets from "at most 2 distinct" to "at most K distinct" — the shrink condition (`map.size() > K`) will be described as a direct parameterization of today's, not re-derived. No new theory tomorrow; the full DSA block is reserved for two Hard-tier problems, so today's fluency with HashMap-based window validity checks is assumed to be fully automatic going in.
