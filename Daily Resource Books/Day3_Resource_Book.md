# Day 3 Resource Book — Big-O Notation and ArrayList

**Series:** Week 1 SDE-2 Prep · [← Curriculum Map](./00_Curriculum_Map.md) · [← Day 2](./Day2_Resource_Book.md) · Next: Day 4 →

**Companion to:** Day 3 of `Week_01_Revised.md`

---

## Recap: what today builds on, and what today pays off

Days 1-2 used the word "O(1)" informally, twice — array access, then a nested-loop forward reference — always with a promise that the real definition was coming. Today is that payoff. Big-O is arguably the single highest-leverage topic in this entire plan: it's the shared vocabulary every complexity discussion in every remaining interview problem this series covers will be conducted in, starting with the very next day.

## Learning Objectives

By the end of today, without notes, you should be able to:

1. State what Big-O actually measures, and explain why constants and non-dominant terms are dropped.
2. Classify the time complexity of an unfamiliar code snippet on sight, including distinguishing nested loops from sequential ones.
3. Explain best/worst/average case, and know which one interviews default to caring about.
4. Explain precisely why `ArrayList.add()` is "amortized O(1)" rather than simply "O(1)" — including the actual math, not just the label.
5. Explain why doubling capacity (rather than growing by a fixed amount) is what makes that amortized bound hold.
6. Use `ArrayList` correctly, including the two gotchas that trip up almost everyone the first time: the `remove(int)` vs. `remove(Object)` ambiguity, and `Integer` caching's effect on `==`.

## Concept Dependency Map for Today

```
Big-O notation (needs: loops — Day 1; arrays, for the concrete examples — Day 2)
        │
        ▼
Analyzing code snippets: nested vs. sequential loops, dropping constants/non-dominant terms
        │
        ▼
Best / worst / average case
        │
        ▼
Amortized analysis (needs: Big-O fully formal — this is a refinement of it, not a separate topic)
        │
        ▼
ArrayList: dynamic resizing explained THROUGH amortized analysis
        │
        ▼
Autoboxing / wrapper classes (needs: ArrayList — generics can't hold primitives directly)
        │
        ▼
Practice: redo Day 2's array exercises with ArrayList<Integer>
```

---

# Section 1 — Big-O Notation, Formally

### What Big-O actually measures

Every previous mention of "O(1)" or "O(n²)" in this series has been informal, leaning on intuition. Here's the precise version: **Big-O describes how the amount of work an algorithm does grows as the size of its input grows** — specifically, it describes the *shape* of that growth, not an exact operation count, and it deliberately ignores hardware speed, programming language, or any other constant-factor detail.

This distinction — shape of growth, not exact count — is the part worth sitting with, because it's the part that makes Big-O actually useful. Two algorithms, one taking exactly `3n + 7` steps and another taking exactly `n` steps, are **both O(n)** — they grow at the *same rate* as `n` increases, even though the first one is always doing more total work at any given `n`. Big-O is answering "if I double my input size, does my work roughly double, roughly quadruple, barely change, or explode?" — not "exactly how many operations will this take on my specific machine."

> 🔑 **Key Takeaway:** Big-O is about **trend at scale**, not a stopwatch measurement. An O(n²) algorithm can genuinely outperform an O(n) one for small inputs (constants and lower-order terms matter at small scale) — Big-O's guarantee is specifically about what happens as `n` grows large, which is exactly the regime real systems eventually hit.

### The complexity classes, from fastest-growing work to slowest

| Notation | Name | Intuition | Typical example |
|---|---|---|---|
| O(1) | Constant | Same work regardless of input size | Array indexing, arithmetic |
| O(log n) | Logarithmic | Work shrinks by a fraction (usually half) each step | Binary search (Week 2) |
| O(n) | Linear | One pass over the input | A single loop over an array |
| O(n log n) | Linearithmic | A linear pass combined with a logarithmic one, typically via sorting | Efficient sorting (mergesort) |
| O(n²) | Quadratic | Work for every *pair* of elements | Nested loop over the same input |
| O(2ⁿ) | Exponential | Work doubles with every additional input element | Naive recursive Fibonacci (no memoization) |
| O(n!) | Factorial | Work explodes across every possible ordering | Generating all permutations |

You'll work almost entirely in the first five for this plan's near-term scope; the last two are worth being able to *recognize and name* rather than something you'll be optimizing today.

```
Growth rate, visualized at increasing input size (illustrative, not to scale):

O(1)        ────────────────────────────────  (flat — never grows)
O(log n)    ──────╱───────────────────────    (grows, but barely, even at huge n)
O(n)        ────────────╱──────────────       (straight diagonal line)
O(n log n)  ──────────────╱────────           (a bit steeper than linear)
O(n²)       ────────────────────╱╱            (curves upward, steeply, as n grows)
O(2ⁿ)       ──────────────────────────╱│      (explodes — becomes unusable past small n)
```

### How to actually analyze a snippet of code

Four rules cover the overwhelming majority of what you'll classify:

**Rule 1 — A single loop over the input is O(n).**

```java
int sum = 0;
for (int i = 0; i < arr.length; i++) {
    sum += arr[i];
}
```
One pass, constant work per element → O(n).

**Rule 2 — A loop nested inside another loop, both dependent on the same input size, multiplies: O(n) × O(n) = O(n²).**

```java
for (int i = 0; i < arr.length; i++) {
    for (int j = 0; j < arr.length; j++) {
        System.out.println(arr[i] + ", " + arr[j]);
    }
}
```
For *every* one of the `n` outer iterations, the inner loop runs a full `n` iterations again — total work is `n × n = n²`.

**Rule 3 — Sequential (not nested) loops *add*, they don't multiply — and the dominant term wins.**

```java
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);          // O(n)
}
for (int j = 0; j < arr.length; j++) {
    System.out.println(arr[j] * 2);      // another, separate O(n)
}
```
This is `O(n) + O(n) = O(2n)`, which simplifies to **O(n)** — not O(n²). This is a genuinely common point of confusion: **two loops back-to-back is not automatically quadratic; only nested loops multiply.** The giveaway is structural: is the second loop *inside* the first (nested → multiply) or *after* the first, running independently (sequential → add)?

**Rule 4 — A loop that shrinks the problem by a constant fraction (typically halving) each iteration is O(log n).**

```java
int n = 1000000;
int steps = 0;
while (n > 1) {
    n = n / 2;
    steps++;
}
```
Each iteration cuts `n` roughly in half. The question "how many times can you halve `n` before reaching 1?" is precisely what a logarithm (base 2) answers — for `n = 1,000,000`, that's only about 20 iterations, not a million. This exact "halve the problem each step" shape is the core mechanic behind binary search, arriving Week 2 — worth filing away now, since recognizing this shape on sight is most of what's needed to spot an O(log n) opportunity later.

### Simplification rules — and why each one is actually justified, not just a convention

**Drop constants:** `O(2n)` becomes `O(n)`. A loop that runs through the array twice (two *separate*, sequential O(n) passes) is technically `2n` operations, but as `n` grows large, the difference between "grows like `n`" and "grows like `2n`" becomes about the *shape* of growth, not a meaningfully different category — both double when the input doubles, which is the property Big-O is actually trying to capture.

**Drop non-dominant (lower-order) terms:** `O(n² + n)` becomes `O(n²)`. As `n` grows large, the `n²` term completely dwarfs the `n` term — at `n = 1000`, `n²` is `1,000,000` while the extra `n` contributes a comparatively tiny `1,000`. The smaller term becomes negligible at scale, so it's dropped.

**Different inputs get different variables.** A function that checks whether *any* element of array `A` also appears in array `B` should be expressed in terms of **both** input sizes — commonly `O(m + n)` or `O(m × n)` depending on the approach — never collapsed into a single "O(n)" as if both arrays were the same size or even the same input. This isn't a stylistic nicety: conflating two genuinely different input sizes into one variable can hide real, meaningful differences in how an algorithm actually scales, and stating complexity precisely in terms of the actual inputs involved is exactly the kind of rigor a tier-1 interview is listening for.

> ⚠️ **Common Mistake:** Reflexively calling anything with "two loops" O(n²) without checking whether they're nested or sequential. Always ask: does the *total* work multiply (nested — each outer step re-triggers the full inner loop) or add (sequential — each loop runs once, independently)?

### Best case, worst case, average case

- **Worst case** — the input that makes the algorithm take the *longest*. This is what "Big-O" almost always refers to by default in interview contexts, and unless explicitly told otherwise, **assume worst case is what's being asked for.**
- **Best case** — the input that makes it *fastest*. Rarely the headline metric, but worth being able to identify (e.g., a linear search whose target happens to be the very first element checked is O(1) best case, even though the same algorithm is O(n) worst case).
- **Average case** — expected performance across a "typical" distribution of inputs. Harder to compute rigorously, and cited specifically for certain well-known algorithms — quicksort, for instance, has an O(n log n) average case but an O(n²) worst case, and that specific gap is a genuinely common interview talking point once sorting algorithms are covered.

> 💡 **Interview Insight:** If you state a complexity without qualifying which case you mean, a strong interviewer will ask. Get ahead of it — state "worst case" explicitly when you give a Big-O answer, unless you have a specific reason to highlight best or average case instead. This single habit reads as meaningfully more rigorous than leaving it ambiguous.

### Practice: classify these five snippets yourself before reading the answer

```java
// Snippet A
int x = arr[0] + arr[arr.length - 1];

// Snippet B
for (int i = 0; i < arr.length; i++) {
    for (int j = 0; j < arr.length; j++) {
        if (arr[i] == arr[j]) count++;
    }
}

// Snippet C
for (int i = 0; i < arr.length; i++) { total += arr[i]; }
for (int j = 0; j < arr.length; j++) { product *= arr[j]; }

// Snippet D
int n = arr.length;
while (n > 1) { n = n / 2; steps++; }

// Snippet E — the array is sorted first, then scanned once
Arrays.sort(arr);                              // sorting: O(n log n)
for (int i = 0; i < arr.length; i++) { ... }    // then a single scan: O(n)
```

**Answers:**
- **A — O(1).** A fixed, small number of operations, completely independent of `arr.length`.
- **B — O(n²).** Nested loop, both bounds tied to the same input size (Rule 2).
- **C — O(n).** Two *sequential* O(n) loops → O(2n) → simplifies to O(n) (Rule 3 — not nested, so they add, not multiply).
- **D — O(log n).** Halves each iteration (Rule 4).
- **E — O(n log n).** Sequential steps add: `O(n log n) + O(n)`. The `n log n` term dominates as `n` grows (it grows faster than plain `n`), so the lower-order `O(n)` term drops, leaving `O(n log n)` — a direct, concrete instance of the "sequential steps add, dominant term wins" rule, and the single most common real-world way O(n log n) actually shows up: sort, then do a linear-time pass over the sorted result.


---

# Section 2 — Amortized Analysis

### Why a new kind of analysis is needed at all

Every complexity rule in Section 1 assumes a single operation's cost is roughly consistent every time it runs. Some operations don't behave that way — they're cheap almost every time, but occasionally, predictably, expensive. Judging such an operation by its worst single call in isolation would be *technically* true but practically misleading. **Amortized analysis** asks a different, more useful question: *if I perform this operation `n` times in a row, what's the average cost per operation, spread across that whole sequence?*

This isn't the same thing as "average case" from Section 1. Average case is about averaging over *different possible inputs* (what if the data is random?). Amortized analysis is about averaging over a *sequence of operations on one growing structure*, and — critically — it's a guarantee, not a probability: it holds for *every* sequence, not just "typical" ones.

> My Definition: "As the data structure grows, it occasionally performs an expensive maintenance operation (like resizing). Amortized analysis spreads the cost of those rare expensive operations across all the cheap operations that came before and after."

### Why this matters immediately: ArrayList's resizing problem

You met arrays' core limitation on Day 2: fixed size, can't grow. `ArrayList` is Java's answer — a class that *looks* like it grows freely, `add()`-ing elements with no size limit. Under the hood, it's still backed by a real, fixed-size array; resizing is simply handled *for you*, automatically, whenever it's needed:

1. Most calls to `add()` are cheap: there's already an open slot in the backing array, so the new element just gets placed there directly. **O(1).**
2. Occasionally, the backing array is completely full. When that happens, `add()` must: allocate a brand-new, larger array (Java's `ArrayList` **doubles** the capacity), copy *every single existing element* into it, and only then place the new element. **That copy is O(n).**

Judged call-by-call, `add()` is sometimes O(1) and occasionally O(n) — so what's the honest, useful thing to say about its complexity overall?

### The actual math — worth walking through once, carefully

Suppose you call `add()` a total of `n` times on an initially empty `ArrayList`. Because capacity **doubles** each time a resize is triggered, resizes happen when the list's size passes 1, 2, 4, 8, 16, ... — each resize copying a number of elements equal to the list's size *at that moment*.

The total copying work across *every* resize, for `n` total appends, is bounded by:

```
1 + 2 + 4 + 8 + ... + n/2 + n
```

This is a geometric series, and it has a well-known, clean closed form: this sum is always **less than `2n`**, no matter how large `n` gets (specifically, the sum of powers of 2 up to `n` equals `2n - 1`). So across `n` total `add()` calls, the *total* work — every cheap O(1) placement plus every expensive O(n) resize-and-copy, all added together — is bounded by `n` (the cheap placements) `+ 2n` (all the resize copying, combined) `= 3n`, which is **O(n)**.

Spread that total O(n) work evenly across the `n` calls that produced it, and the **average cost per call is O(n) / n = O(1)**. That average — not any single call's actual cost — is what "amortized O(1)" means. Some individual calls really do cost O(n); the guarantee is about the total, divided across the sequence that produced it, and that guarantee holds for *every* possible sequence of `n` appends, not just typical ones — which is what makes it a stronger, more useful statement than an "average case."

> 🔑 **Key Takeaway:** "Amortized O(1)" is not "usually fast, occasionally slow, so let's just call it fast." It's a precise mathematical claim: *the total cost of `n` operations is O(n)*, which means the average cost per operation, over any sequence of them, is O(1) — a genuinely different and stronger statement than an informal "usually."

### Why doubling, specifically — and not, say, growing by a fixed +10 each time

This design choice is worth being able to justify, not just recite, because it's exactly the kind of "why this and not that" question a tier-1 interview might actually ask. Suppose the backing array instead grew by a **fixed** amount — say, always +10 slots — every time it filled up.

- Resizes would then happen every 10 appends, **always**, regardless of how large the list has already grown — unlike doubling, where resizes become exponentially *rarer* as the list grows (the gap between resize N and resize N+1 keeps getting bigger).
- Each resize still copies **every existing element** — so the *later* resizes, on a much larger list, copy far more data than the early ones, while still happening at the same fixed frequency.
- Total copying work across `n` appends becomes proportional to `10 + 20 + 30 + ... ≈ n²/10` — which is **O(n²)**, not O(n).
- That makes the amortized cost per append **O(n)**, not O(1) — a fixed-growth strategy is genuinely, asymptotically worse, not just a minor tuning difference.

Doubling is specifically what keeps the *frequency* of expensive resizes shrinking fast enough, relative to the growing *cost* of each one, to keep the total bounded by O(n) rather than O(n²). This is a real, deliberate trade-off in `ArrayList`'s design, not an arbitrary implementation detail — and articulating *why* the growth factor matters, unprompted, is a strong signal in exactly the way this series keeps building toward.

---

# Section 3 — ArrayList

### Declaring, adding, accessing

```java
import java.util.ArrayList;

ArrayList<Integer> list = new ArrayList<>();
list.add(10);
list.add(20);
list.add(30);

int first = list.get(0);        // 10 — indexed access, still O(1) (see below)
list.set(0, 99);                 // overwrite the value AT index 0
int size = list.size();          // 3 — a METHOD (parentheses), the THIRD sibling to know apart:
                                  //   array.length (field), string.length() (method), list.size() (method)
boolean has = list.contains(20); // true — a linear O(n) scan under the hood; no shortcut without more structure
```

**Is `list.get(i)` still O(1)?** Yes — `ArrayList` is backed internally by a real array, so `.get(i)` simply delegates straight to that array's indexed access, using exactly the same base-address-plus-offset calculation from Day 2, Section 1. The dynamic-resizing machinery only activates on `add()` (and `remove()`); reading an existing index is untouched by any of it.

### The generics restriction, and why wrapper classes exist

`ArrayList<int>` **does not compile.** Java's generics (the `<...>` type parameter) only work with **object** types, never primitives — this is a hard language restriction, not a missing feature you're overlooking. To hold `int` values in an `ArrayList`, Java provides **wrapper classes**: `Integer` wraps `int`, `Double` wraps `double`, `Boolean` wraps `boolean`, `Character` wraps `char` — each a genuine object with the primitive value stored inside it.

### Autoboxing and unboxing

```java
ArrayList<Integer> list = new ArrayList<>();
list.add(5);            // AUTOBOXING: the primitive int 5 is automatically wrapped into an Integer object
int x = list.get(0);    // AUTO-UNBOXING: the Integer object is automatically unwrapped back to a raw int
```

This conversion happens automatically and invisibly — you write plain `int`-looking code, and the compiler inserts the wrapping/unwrapping for you. It is not, however, free: each boxed value is a real object requiring actual heap allocation, unlike a raw primitive. At the scale of a handful of values this is invisible; inside a performance-sensitive loop processing millions of elements, autoboxing overhead is a real, measurable, occasionally interview-relevant cost.

### Gotcha #1 — `remove(int)` vs. `remove(Object)`

```java
ArrayList<Integer> list = new ArrayList<>(List.of(10, 20, 30));

list.remove(1);                    // removes the element AT INDEX 1 → removes 20 → list is now [10, 30]
list.remove(Integer.valueOf(10));  // removes the VALUE 10, wherever it is → list is now [30]
```

`ArrayList` has **two overloaded `remove()` methods**: `remove(int index)` and `remove(Object o)`. When you write `list.remove(1)`, the compiler sees a literal `int` and matches it to the `remove(int index)` overload — meaning it removes **by position**, not by value, which is genuinely surprising the first time and a very well-known interview trivia gotcha. To remove a *value* that happens to be an `int`, you have to force the compiler toward the `Object` overload — either by boxing it explicitly with `Integer.valueOf(...)`, or by declaring the variable you're removing as an `Integer` in the first place (an already-boxed `Integer` variable matches the `Object` overload directly, with no ambiguity).

> ⚠️ **Common Mistake:** Assuming `list.remove(someInt)` removes the *value* `someInt`. On an `ArrayList<Integer>`, it does not — it removes whatever sits at that *index*, and will throw an `IndexOutOfBoundsException` if that index doesn't exist, which is often the first sign something's wrong.

### Gotcha #2 — `Integer` caching and `==`

This is a direct, surprising extension of Day 2's String `==` lesson — the exact same *shape* of bug shows up here for a completely different underlying reason:

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b);   // true

Integer c = 200;
Integer d = 200;
System.out.println(c == d);   // false
```

Java's `Integer` autoboxing **caches** (reuses) wrapper objects for a small range of values — by default, **-128 to 127** — because these small values are disproportionately common in real code, and reusing cached objects for them avoids constant, wasteful object creation. `Integer a = 100` and `Integer b = 100` both fall inside that cached range, so autoboxing hands back the *same* cached object for both — `==` (comparing references, exactly as with String) reports `true`. `200` falls **outside** the cached range, so each autoboxing creates a genuinely new object — `==` correctly reports `false`.

> 🔑 **Key Takeaway:** just as with String, **never use `==` to compare `Integer` (or any wrapper type) values — always use `.equals()`.** The fact that it "happens to work" for small numbers during casual testing, and then silently breaks the moment a value exceeds 127, is precisely what makes this bug dangerous rather than obviously wrong — it's the same underlying lesson as Day 2's String gotcha, wearing a different costume.

---

# Section 4 — Practice: Redoing Day 2's Array Exercises with ArrayList

## Find the Maximum, with `ArrayList<Integer>`

```java
public static int findMaxArrayList(ArrayList<Integer> list) {
    if (list == null || list.isEmpty()) {
        throw new IllegalArgumentException("List must not be null or empty");
    }
    int max = list.get(0);
    for (int i = 1; i < list.size(); i++) {
        if (list.get(i) > max) {
            max = list.get(i);
        }
    }
    return max;
}
```

**What changed, and what stayed the same:** `.length` → `.size()`; `arr[i]` → `list.get(i)`; `arr.length == 0` → the more convenient `.isEmpty()` (a method Java's arrays never had, since arrays don't carry any convenience methods at all — they're a language-level construct, not a class). `list.get(i) > max` still compiles and behaves correctly even though `list.get(i)` returns an `Integer` object, not a raw `int` — the `>` comparison operator triggers automatic unboxing behind the scenes. **The algorithm itself — the loop structure, the "seed with the first real element" logic, the O(n) complexity — is completely unchanged.** Only the API surface used to talk to the underlying data changed.

## Reverse In Place, with `ArrayList<Integer>`

```java
public static void reverseInPlaceArrayList(ArrayList<Integer> list) {
    int left = 0;
    int right = list.size() - 1;
    while (left < right) {
        int temp = list.get(left);
        list.set(left, list.get(right));
        list.set(right, temp);
        left++;
        right--;
    }
}
```

**What changed:** there's no direct `arr[i] = x` assignment syntax for `ArrayList` — writing to an index requires the explicit `.set(index, value)` method call. **What stayed the same:** the exact same two-pointer swap-from-both-ends logic from Day 2, at O(n) time and O(1) extra space — the *pattern* is identical; only the syntax for reading and writing an index changed from `[]` to `.get()`/`.set()`.

> 💡 **Interview Insight:** Being able to translate an algorithm you already understand between `int[]` and `ArrayList<Integer>` fluently — recognizing which parts are the *algorithm* (unchanged) and which parts are just *API surface* (`[]` vs. `.get()`/`.set()`, `.length` vs. `.size()`) — is a small but real signal of genuine understanding versus memorized syntax. Interviewers do sometimes deliberately switch which structure a problem is phrased in, specifically to see whether a candidate's understanding is portable or brittle.


### One more piece of the API: for-each iteration

```java
ArrayList<Integer> list = new ArrayList<>(List.of(10, 20, 30));
for (int value : list) {
    System.out.println(value);
}
```

Exactly the same for-each syntax from Day 2's arrays, and exactly the same trade-off: reach for it when you only need the values, and fall back to an index-based loop (`for (int i = 0; i < list.size(); i++)`) the moment you need the position — for instance, the two-pointer swap in the reversal exercise above genuinely needs indices and couldn't be written as a for-each.

---

# Section 5 — Project Block

In your `java-fundamentals` repository, add a `collections` package (you'll keep building it out across the rest of this week) containing `ArrayListPractice.java` with today's two rewritten exercises.

**Definition of done:** pushed, with a code comment written **in your own words** explaining why `ArrayList` can grow and a raw array can't. Writing this explanation yourself, rather than copying language from here, is the actual point — if you can't produce it without looking, that's the honest signal to re-read Sections 2-3 before moving on, since today's amortized-analysis reasoning is exactly the kind of thing that's easy to nod along to and harder to reproduce cold.

---

# Section 6 — Career Block

### LinkedIn Post 2

Topic: one thing that genuinely surprised you about how Java actually runs. You now have real material for this — the bytecode/JVM pipeline (Day 1), the String pool (Day 2), or today's amortized-doubling behavior are all genuinely surprising to most people meeting them for the first time, and a post built around a real "wait, that's how it works?" moment reads as far more credible and engaging than a generic progress update.

### Networking

Reach out to 2 college alumni at companies on your radar — shared educational background is a strong, natural opener that doesn't feel like a cold ask, and is worth using deliberately as you build out your outreach list.

---

# Day 3 — Interview Questions

---

**1. What does Big-O actually measure?**

*Answer:* How the amount of work an algorithm does grows as input size grows — the *shape* of that growth — deliberately ignoring constant factors, hardware, and programming language. It answers "does work roughly double, quadruple, or barely change as input doubles?", not "exactly how many operations will this take."

---

**2. Why does Big-O drop constants — e.g., why is O(2n) just written as O(n)?**

*Answer:* Big-O captures growth trend at scale, not exact operation counts. Both `n` and `2n` double when the input doubles — they belong to the same growth category — so the constant factor doesn't change which "shape" of growth the algorithm belongs to.

---

**3. Why does Big-O drop non-dominant (lower-order) terms — e.g., why does O(n² + n) become O(n²)?**

*Answer:* As `n` grows large, the `n²` term completely dwarfs the `n` term (at n=1000, n² = 1,000,000 vs. n = 1,000) — the smaller term becomes negligible at the scale Big-O is concerned with.

---

**4. Why should two differently-sized inputs (e.g., two separate arrays) get different variables in a complexity expression, rather than both being called "n"?**

*Answer:* Collapsing genuinely different input sizes into one variable can hide real differences in how an algorithm scales — e.g., checking membership across two arrays of sizes `m` and `n` should be expressed as `O(m + n)` or `O(m × n)`, not folded into a single misleading "O(n)."

---

**5. Two loops that run one after another (not nested) — what's the complexity, and how is that different from two nested loops?**

*Answer:* Sequential loops *add*: O(n) + O(n) = O(2n) = O(n). Nested loops *multiply*: an inner O(n) loop running fully for every outer iteration gives O(n) × O(n) = O(n²). The structural giveaway is whether the second loop is inside the first (multiply) or after it, independently (add).

---

**6. What's the complexity of a loop that halves its working value every iteration, and why?**

*Answer:* O(log n). The question "how many times can you halve n before reaching 1?" is exactly what a base-2 logarithm answers — for n = 1,000,000, only about 20 iterations, not a million.

---

**7. Define best, worst, and average case. Which does an interview default to caring about?**

*Answer:* Worst case: the input maximizing runtime. Best case: the input minimizing it. Average case: expected performance over a typical input distribution. Interviews default to worst case unless stated otherwise — state "worst case" explicitly when giving a Big-O answer rather than leaving it ambiguous.

---

**8. Why is `ArrayList.add()` described as "amortized O(1)" instead of simply "O(1)"?**

*Answer:* Individual calls aren't uniformly O(1) — most are (an open slot exists), but occasionally a call triggers a full resize-and-copy, which is O(n). "Amortized O(1)" is the precise claim that the *total* cost across any sequence of n appends is O(n), making the *average* cost per call O(1) — a guarantee about the whole sequence, not about any single call.

---

**9. Walk through why the total copying work across n appends is O(n), not more.**

*Answer:* Because capacity doubles, resizes happen at sizes 1, 2, 4, 8, ..., n — each copying that many elements. The total copying work is the geometric series 1+2+4+...+n, which sums to less than 2n regardless of how large n gets. Total work (cheap placements + all copying) is bounded by O(n), so the amortized cost per call is O(n)/n = O(1).

---

**10. Why does `ArrayList` double its capacity instead of growing by a fixed amount each time? What would happen if it grew by, say, +10 every time it filled up?**

*Answer:* Fixed growth means resizes happen at a constant frequency (every 10 appends) regardless of how large the list already is, while each resize still copies every existing element — so later resizes on a large list copy far more data at the same frequency as early ones. Total copying work becomes O(n²), giving an amortized cost of O(n) per append, not O(1). Doubling makes resizes exponentially rarer as the list grows, which is specifically what keeps total copying bounded by O(n).

---

**11. What is autoboxing, and why does `ArrayList<Integer>` need it?**

*Answer:* Java generics only accept object types, not primitives — `ArrayList<int>` doesn't compile. Wrapper classes (`Integer` for `int`, etc.) are real objects that hold a primitive value inside them. Autoboxing is Java automatically converting a primitive to its wrapper object (and auto-unboxing, the reverse) wherever needed, invisibly.

---

**12. Is there a performance cost to autoboxing?**

*Answer:* Yes — each boxed value requires genuine heap allocation as an object, unlike a raw primitive. Negligible for small-scale use, but a real, measurable cost inside performance-sensitive loops handling large volumes of data.

---

**13. What's the difference between `list.remove(1)` and `list.remove(Integer.valueOf(1))` on an `ArrayList<Integer>`?**

*Answer:* `remove(1)` matches the `remove(int index)` overload — it removes whatever element sits **at index 1**. `remove(Integer.valueOf(1))` matches the `remove(Object o)` overload — it removes the first element **equal to the value 1**, wherever it is. Java has both overloads, and a literal `int` argument always resolves to the index-based one.

---

**14. Why is `Integer a = 100; Integer b = 100; a == b` `true`, while `Integer c = 200; Integer d = 200; c == d` is `false`?**

*Answer:* Java's `Integer` autoboxing caches wrapper objects for the range -128 to 127. Values in that range reuse the same cached object, so `==` (comparing references) reports `true`. Values outside that range are freshly allocated on each autoboxing, so `==` reports `false` even though the content is identical — the same underlying reference-vs-content pitfall as String `==`, for a different reason. Always use `.equals()` for wrapper types.

---

**15. Is `ArrayList.get(i)` still O(1)?**

*Answer:* Yes — `ArrayList` is backed internally by a real array, so `.get(i)` delegates directly to that array's indexed access. Only `add()` (and `remove()`) interact with the resizing machinery; reading an existing index is unaffected by it.

---

**16. [Code reading]** What's the time complexity, and why?

```java
for (int i = 0; i < arr.length; i++) {
    for (int j = 0; j < arr.length; j++) {
        sum += arr[i] * arr[j];
    }
}
for (int k = 0; k < arr.length; k++) {
    total += arr[k];
}
```

*Answer:* O(n²). The nested loop alone is O(n²) (Rule 2). The trailing single loop is a separate, sequential O(n), which *adds* rather than multiplies: O(n²) + O(n) = O(n²) once the non-dominant term is dropped.

---

**17. [Debug]** What's wrong here, assuming the intent is to remove the *value* `3` from the list?

```java
ArrayList<Integer> list = new ArrayList<>(List.of(1, 2, 3, 4, 5));
list.remove(3);
```

*Answer:* This removes the element **at index 3** (the value `4`), not the value `3` — because the literal `int` argument resolves to the `remove(int index)` overload. The fix: `list.remove(Integer.valueOf(3));`, which forces the `remove(Object)` overload and removes by value instead.

---

## Daily Deliverable Check

- [ ] Can classify a short code snippet's time complexity across the five common classes
- [ ] Comfortable with `ArrayList`'s core methods: `.add()`, `.get()`, `.set()`, `.remove()`, `.size()`, `.contains()`, for-each
- [ ] `ArrayListPractice.java` pushed to `java-fundamentals/collections`, with your own explanation of why `ArrayList` can grow and a raw array can't
- [ ] LinkedIn Post 2 published

---

## What Tomorrow Assumes You Already Know Cold

Day 4 builds directly on: Big-O (especially recognizing and stating O(1) average-case operations, which is exactly how HashSet/HashMap will be introduced) and `.equals()` from Day 2 (HashMap's correctness depends on it directly — you'll see exactly how tomorrow). If classifying a nested vs. sequential loop still takes conscious effort rather than being immediate, that's worth another pass through Section 1 before continuing — Day 4 assumes Big-O is a fluent vocabulary now, not something being actively reasoned through.

**Next:** [Day 4 Resource Book](./Day4_Resource_Book.md) — HashSet, HashMap, Stack, and Queue.
