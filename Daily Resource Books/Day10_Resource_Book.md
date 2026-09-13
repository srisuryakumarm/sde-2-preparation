# Day 10 — Two Pointers on Triplets, and the Integer Cache Trap

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 9 Resource Book](Day9_Resource_Book.md)
**Next ▶:** [Day 11 Resource Book](Day11_Resource_Book.md)
**Companion to:** Day 10 of `Week_02_Revised.md`

---

## ⚠️ Overlap Notice — Read This First

Only *one* of today's two required problems is a repeat this time — a partial overlap, unlike Days 8–9's full overlap.

| LC # | Problem | Status |
|---|---|---|
| 15 | 3Sum | Already solved — Week 1, Day 7 (Extra Practice). **Recap only.** |
| 16 | 3Sum Closest | Genuinely new. **Full depth below.** |

No substitute problem is needed today — with only one overlap instead of two, there's no freed block large enough to justify displacing 3Sum Closest's full treatment, and 3Sum Closest is squarely required, non-overlapping, real new material.

---

## Recap

Yesterday's JVM Memory Model explained *why* pass-by-value holds, at the mechanism level. Today's theory does the same move for a different Week 1 fact: the `==` vs. `.equals()` trap, which Week 1 Day 2 established as "the single most-referenced gotcha across the week." Today shows *why* it gets sneakier for `Integer` specifically, using the stack/heap split from yesterday directly.

On the DSA side: 3Sum, recapped from Week 1, is the direct ancestor of today's new problem — same skeleton, different objective function.

---

## Learning Objectives

1. Recognize when a two-pointer sweep needs an outer loop wrapped around it (3Sum's shape), and adapt the inner objective (exact match → closest match) without changing the pointer-movement logic.
3. State the exact byte range of every Java integer type from memory, and explain — not just state — why overflow wraps silently instead of throwing.
4. Explain the Integer cache mechanism precisely enough to predict the output of `Integer.valueOf(x) == Integer.valueOf(y)` for any `x, y`, and connect it explicitly back to Week 1's `==` vs. `.equals()` rule.

---

## Concept Dependency Map

```
Week 1 Day 7: 3Sum (LC 15) — outer fixed element + inner opposite-ends
        │
        └──▶ Today: 3Sum Closest (LC 16) — same skeleton, track closest, not exact

Week 1 Day 2: == vs .equals(), primitives, autoboxing intro (Week 1 Day 3: ArrayList)
Day 9 (yesterday): stack (values) vs. heap (objects) — the mechanism
        │
        └──▶ Today: Primitives, Precisely
                 ├─ exact sizes/ranges, overflow mechanics
                 └─ Integer cache: WHY == sometimes "works" for small values
```

---

## Part 1 — Two Pointers: Recap and 3Sum Closest

### 🔗 Recap: 3Sum (LC 15)

**Original coverage:** Week 1, Day 7, Extra Practice. Optimized solution only, for reference — this is also literally the skeleton 3Sum Closest below is built from, so it's worth having the exact code fresh before reading the diff between the two.

```java
public static List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();

    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;   // skip duplicate FIXED element

        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left + 1]) left++;   // skip duplicate LEFT
                while (left < right && nums[right] == nums[right - 1]) right--; // skip duplicate RIGHT
                left++;
                right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    return result;
}
```

Sort the array; fix one element with an outer loop; run opposite-ends two pointers on the remainder, searching for a pair summing to the fixed element's negation; skip duplicate fixed elements (and duplicate pointer values on a match) to avoid emitting the same triplet twice. Time O(n²) (outer O(n) × inner two-pointer scan O(n)), Space O(1) excluding output and sort overhead.

**🔑 Key Takeaway (recap):** duplicate-skipping in 3Sum is not optional polish — it's required for correctness against the problem's own "no duplicate triplets" constraint. Hold onto that distinction; it flips for today's problem.

---

### New Problem: 3Sum Closest (LC 16)

**Statement:** Given an array `nums` and an integer `target`, find three numbers in `nums` whose sum is closest to `target`. Return that sum. Assume exactly one solution exists.

### Approach 1 — Brute force

```java
public static int threeSumClosestBruteForce(int[] nums, int target) {
    int closest = nums[0] + nums[1] + nums[2];
    for (int i = 0; i < nums.length - 2; i++) {
        for (int j = i + 1; j < nums.length - 1; j++) {
            for (int k = j + 1; k < nums.length; k++) {
                int sum = nums[i] + nums[j] + nums[k];
                if (Math.abs(sum - target) < Math.abs(closest - target)) {
                    closest = sum;
                }
            }
        }
    }
    return closest;
}
```

Check all `C(n,3)` triplets, track the one with minimum `|sum - target|`. Time O(n³), Space O(1).

### Approach 2 — Optimized: sort + two pointers (identical skeleton to 3Sum, different objective)

```java
public static int threeSumClosest(int[] nums, int target) {
    Arrays.sort(nums);
    int closest = nums[0] + nums[1] + nums[2];

    for (int i = 0; i < nums.length - 2; i++) {
        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (Math.abs(sum - target) < Math.abs(closest - target)) {
                closest = sum;
            }
            if (sum == target) {
                return sum;             // can't get any closer than an exact match
            } else if (sum < target) {
                left++;                 // need a larger sum
            } else {
                right--;                // need a smaller sum
            }
        }
    }
    return closest;
}
```

Same skeleton as the 3Sum recap above — sort, fix `i`, sweep `left`/`right` on the remainder — with two changes: no duplicate-skipping (see below), and the sweep tracks the *closest* sum seen rather than collecting exact matches.

**Why this reaches the true closest sum, not just *a* candidate:** at every `(i, left, right)` triple actually visited, the algorithm records the sum if it's the best seen so far — so correctness reduces to showing the sweep visits every triple that *could* be optimal. For a fixed `i`, the inner opposite-ends sweep is exhaustive over all `(left, right)` pairs in the sense that matters: at any point, moving `left` up or `right` down is the only way to change the sum in the needed direction (monotonicity of the pointer movement, same argument as every opposite-ends problem so far), so no reachable pair is skipped — the sweep never "walks past" a pair that could have beaten the current best, because it only ever moves toward larger or smaller sums as actually required.

**⚠️ Duplicate-skipping is *not* required here — this is the flip from 3Sum.** 3Sum needs to skip duplicate values to avoid *emitting the same triplet twice* in a result list. 3Sum Closest only needs to track a single best number, not a list of distinct triplets — visiting the same value combination more than once costs a little redundant work but produces no incorrect *output*, since duplicate visits simply re-derive a `sum` you may have already seen. Skipping duplicates here is a legitimate minor optimization (fewer redundant comparisons), never a correctness requirement. Stating this distinction unprompted is a strong signal in an interview: it shows you understand *why* a technique applies in one problem, not just that it applies.

**Worked trace:** `nums = [-1, 2, 1, -4]`, `target = 1`. Sorted: `[-4, -1, 1, 2]`.

| i | nums[i] | left | right | sum | \|sum-target\| | closest so far | action |
|---|---|---|---|---|---|---|---|
| 0 | -4 | 1(-1) | 3(2) | -4-1+2=-3 | 4 | -3 | -3<1 → left++ |
| 0 | -4 | 2(1) | 3(2) | -4+1+2=-1 | 2 | -1 | -1<1 → left++ (now left=right, inner loop ends) |
| 1 | -1 | 2(1) | 3(2) | -1+1+2=2 | 1 | 2 | 2>1 → right-- (now left=right, inner loop ends) |

The outer loop stops here — `i < nums.length - 2` (`4 - 2 = 2`) means `i` only ever takes the values `0` and `1`; `i=2` would leave `left=3, right=3` with no pair to sweep at all, so the bound skips that guaranteed-empty iteration rather than entering and immediately exiting it.

Final `closest = 2`. Verify by brute force over all four triplets of `[-4,-1,1,2]`: sums are `-4-1+1=-4`, `-4-1+2=-3`, `-4+1+2=-1`, `-1+1+2=2` — differences from target 1 are `5, 4, 2, 1` respectively. Minimum difference is 1, achieved by sum `2`. Matches.

**Complexity:** Time O(n²) — identical shape to 3Sum: O(n) outer × O(n) inner sweep. Space O(1) extra, excluding sort.

**Edge cases:**
- Exactly 3 elements: outer loop runs once, inner sweep runs once — no special-casing needed, the general loop bounds handle it.
- Multiple triplets tie for closest: the problem guarantees a well-defined answer exists; the algorithm returns whichever tied sum it encounters first, which is a valid answer since the problem only asks for the sum, not a specific triplet.
- All negative or all positive `nums`: no issue — the algorithm never assumes a mix of signs, only that the array is sorted.

**💡 Interview Insight:** if asked "how would you extend this to 4Sum Closest?" — same answer as generalizing 3Sum to 4Sum (tomorrow's problem): add another outer loop, same inner two-pointer sweep, same "track closest instead of exact" adjustment layered on top. Naming this connection out loud, before being asked to actually implement it, is exactly the kind of unprompted generalization interviewers are listening for.

---

## Part 2 — Primitives, Precisely

### Prerequisites (confirmed)

- Primitive types introduced at a basic level (Week 1, Day 1).
- `==` vs. `.equals()` (Week 1, Day 2) — today explains why this gotcha resurfaces in a sneakier form for `Integer` specifically.
- Stack (values) vs. heap (objects) mechanism (Day 9, yesterday) — today's Integer cache explanation depends on this directly.

### Sizes and Ranges — Exact, Not Approximate

| Type | Size | Range |
|---|---|---|
| `byte` | 8 bits | -128 to 127 |
| `short` | 16 bits | -32,768 to 32,767 |
| `int` | 32 bits | -2,147,483,648 to 2,147,483,647 (-2³¹ to 2³¹-1) |
| `long` | 64 bits | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 (-2⁶³ to 2⁶³-1) |
| `float` | 32 bits | IEEE 754 single precision — ~6-7 significant decimal digits |
| `double` | 64 bits | IEEE 754 double precision — ~15-17 significant decimal digits |

All integral types are **signed** in Java (there is no `unsigned int`, unlike C/C++) — one bit is always reserved for sign, which is why the positive range is one less than the negative range's magnitude (e.g., `int` goes up to `2³¹-1`, not `2³¹`, since `0` itself consumes one of the positive-side slots and the sign bit's presence shifts everything by one).

### Overflow: Wraps Silently, No Exception

**Definition:** integer overflow occurs when an arithmetic result falls outside the range a type can represent; in Java, integral overflow does not throw — it wraps around via two's complement representation.

**Why, mechanically:** `Integer.MAX_VALUE` (`2147483647`) in binary is `0111...1` — a `0` sign bit followed by 31 `1` bits. Adding `1` in binary flips every trailing `1` to `0` and carries into the sign bit: the result is `1000...0`, which in two's complement is exactly `Integer.MIN_VALUE` (`-2147483648`). The bit pattern didn't do anything mysterious — it just kept following ordinary binary addition rules, and the *sign bit* happened to be part of what got flipped.

```java
int x = Integer.MAX_VALUE;
System.out.println(x + 1); // -2147483648 — no exception, no warning
```

**⚠️ Common Mistake:** assuming Java throws on overflow, the way some languages (or Java's own `Math.addExact()`, `Math.multiplyExact()` — worth knowing these exist as the opt-in checked alternative) do. Plain `+`, `-`, `*` on `int`/`long` never throw on overflow; they wrap, silently, every time. This matters concretely for tomorrow's 4Sum, where summing values near the constraint boundary can overflow `int` — flagged explicitly when we get there.

### The Integer Cache

**Definition:** `Integer.valueOf(int)` — which is what autoboxing calls under the hood whenever an `int` is implicitly converted to an `Integer` — maintains a cache of pre-built `Integer` objects for values **-128 through 127, inclusive**.

When `valueOf` is called with a value in that range, it returns a reference to the existing cached object instead of allocating a new one. Outside that range, it allocates a fresh `Integer` object on the heap every single call.

```java
Integer a = Integer.valueOf(100);
Integer b = Integer.valueOf(100);
System.out.println(a == b); // true — both refer to the SAME cached object

Integer c = Integer.valueOf(200);
Integer d = Integer.valueOf(200);
System.out.println(c == d); // false — two DIFFERENT objects, same value
```

**Why this is exactly Day 9's mechanism showing up again:** `a == b` compares **references** — whether both variables hold the same heap address — because `Integer` is an object type, and `==` on object types is always reference comparison (never value comparison) in Java. For values in the cached range, both calls to `valueOf` return the identical cached object, so the references are equal and `==` happens to report `true`. Outside the cache, each call allocates a distinct object, so even though the *values* are equal, the *references* aren't — `==` correctly (from its own perspective) reports `false`.

**🔗 Direct connection to Week 1's headline gotcha:** this *is* the `==` vs. `.equals()` trap, wearing a disguise. `.equals()` on `Integer` always compares values correctly, everywhere, regardless of caching — `Integer.valueOf(200).equals(Integer.valueOf(200))` is `true`, always. `==` only ever compares references, and it happens to *look* like it's comparing values for small numbers purely because of the cache — which is precisely what makes this trap sneakier than the ordinary `==`-vs-`.equals()` case: code that "accidentally" works during testing with small values can silently break the moment real data pushes a value past 127.

**Worth knowing, precisely:** the range is guaranteed to include at least -128 to 127 by the Java Language Specification's autoboxing rules; some JVMs allow extending the *upper* bound via `-XX:AutoBoxCacheMax`, but the default upper bound is 127, and the lower bound is fixed at -128 regardless. Also worth knowing: `new Integer(100) == new Integer(100)` is **always** `false`, for *any* value, cached range or not — `new` always allocates fresh, bypassing the cache entirely, which is one of the reasons the `Integer(int)` constructor is deprecated as of Java 9 in favor of `valueOf()`.

### Common Mistakes

- **⚠️ Testing `==` on autoboxed `Integer`s with small values ("it works fine!") and generalizing.** It "works" by coincidence of the cache range, not because `==` does anything different for `Integer` than for any other object.
- **⚠️ Forgetting this applies to autoboxing, not just explicit `Integer.valueOf()` calls.** `Integer x = 100;` autoboxes via `Integer.valueOf(100)` under the hood — the cache trap is present anywhere autoboxing happens, including method calls, collection insertions (`List<Integer>`), and plain assignment, not only explicit `valueOf` calls.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** `TypesAndCache` class demonstrating overflow and the cache trap, heavily commented on *why* each happens (not just *that* it happens).

Practical guidance: include at minimum — the `Integer.MAX_VALUE + 1` overflow demo; `Integer.valueOf(100) == Integer.valueOf(100)` (true) directly next to `Integer.valueOf(200) == Integer.valueOf(200)` (false) so the contrast is visible in one place; and one line using `.equals()` on the same 200/200 case to show it correctly returns `true` regardless. Definition of done: pushed.

## Career Block Guide (1 hr)

**LinkedIn Post 4 — the Integer cache trap.** A draft to adapt, not copy verbatim — make it sound like you, and swap in your own phrasing:

> Spent part of today's SDE-2 prep on something I'd genuinely never had to think about carefully before: `Integer.valueOf(100) == Integer.valueOf(100)` is `true`. `Integer.valueOf(200) == Integer.valueOf(200)` is `false`.
>
> Same code shape, same kind of value, opposite result — because Java caches boxed `Integer` objects for -128 to 127. Below that range, `==` compares two different objects that happen to hold equal values and reports `false`; inside it, `==` happens to "work" purely because both variables point at the same cached object.
>
> The real lesson isn't the cache itself — it's that code tested only with small values can pass every test and still be wrong. `.equals()` doesn't care about any of this and just works. Cheap reminder to actually understand a rule instead of pattern-matching from a handful of examples.

Post it, then move to networking: send the connection requests to the 3 EMs identified yesterday. Keep the note short and specific — reference one concrete thing from their profile or a recent post; a generic "I'd love to connect" gets ignored at a far higher rate than a note that shows you actually looked.

---

## Day 10 — Interview Questions

**Q1. Walk through 3Sum Closest's approach and justify why the two-pointer sweep doesn't miss the true closest sum.** Sort, fix an outer element, run opposite-ends two pointers on the rest, tracking the sum closest to target seen so far; the sweep is exhaustive over reachable pairs because pointer movement is monotonic in the needed direction at every step, the same argument as every opposite-ends problem.

**Q2. Does 3Sum Closest need duplicate-skipping the way 3Sum does?** No — 3Sum needs it to avoid emitting the same triplet twice in a results list; 3Sum Closest only tracks a single best number, so revisiting an equivalent combination costs redundant work, not incorrect output.

**Q3. Why does `int` overflow wrap instead of throwing?** Two's complement arithmetic just keeps following ordinary binary addition rules past the range boundary; the sign bit is not special-cased, so incrementing past the maximum flips it, producing the minimum value with no runtime check involved.

**Q4. What's the exact range of the Integer cache, and where does it come from?** -128 to 127 inclusive, guaranteed by the JLS's autoboxing rules; `Integer.valueOf()` returns a shared cached instance in that range and allocates fresh otherwise.

**Q5. Why does `Integer.valueOf(50) == Integer.valueOf(50)` return `true` but `new Integer(50) == new Integer(50)` return `false`?** `valueOf` checks the cache first for values in range and returns the same shared object; `new` always allocates a fresh object, bypassing the cache regardless of value.

**Q6. Is the Integer cache trap a new rule, or a specific case of something already established?** A specific, sneakier case of `==` vs. `.equals()` — `==` always compares references for object types; the cache just makes references *coincidentally* equal for small values, which is exactly why it's easy to miss.

**Q7. Give a concrete way this trap causes a real bug.** Code compares two boxed `Integer`s with `==` instead of `.equals()`, passes every test written with small sample values, then silently misbehaves the first time production data includes a value outside -128..127.

---

## Daily Deliverable Check

- [ ] 3Sum confirmed solid from Week 1 (recap only, no re-solve needed).
- [ ] 3Sum Closest solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain the Integer cache trap from memory, including the exact range and why it's a disguised form of the `==`/`.equals()` gotcha.
- [ ] `TypesAndCache` pushed, with both the overflow and cache demonstrations, heavily commented.
- [ ] LinkedIn Post 4 published. Connection requests sent to the 3 EMs identified yesterday.

---

## What Tomorrow Assumes You Already Know Cold

Day 11 assumes today's overflow mechanics are solid without re-explanation — 4Sum's constraints put you close enough to `int`'s boundary that using `long` for running sums is a real, not theoretical, correctness requirement, and tomorrow's book will reference today's wraparound explanation rather than re-deriving it. The outer-loop-plus-opposite-ends skeleton (3Sum → 3Sum Closest) also needs to be fully reflexive, since 4Sum extends it one layer deeper without re-explaining the base mechanism.
