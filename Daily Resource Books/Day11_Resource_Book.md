# Day 11 — Two Pointers: Extending to Four, and String Internals

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 10 Resource Book](Day10_Resource_Book.md)
**Next ▶:** [Day 12 Resource Book](Day12_Resource_Book.md)
**Companion to:** Day 11 of `Week_02_Revised.md`

---

## Overlap Notice

Both of today's required problems are genuinely new — no overlap with `00_Curriculum_Map.md`'s inventory, and neither appears in `Week_03_Revised.md`. Full depth on both, as planned.

**One explicit judgment call, flagged per the generation prompt's overlap/extra-practice guidance:** no extra practice problem is added today for Boats to Save Most People's specific sub-variant (opposite-ends with a conditional "pair, or strand the extreme" decision rule). The natural-looking candidate — Assign Cookies (LC 455) — was considered and rejected: on inspection, its two pointers actually move in the *same* direction independently (structurally the "one-forward-pointer-each" variant from Week 1, the same shape as Is Subsequence), not opposite-ends convergence at all, despite both problems being "greedy" and both involving sorting. Slotting it in as a "second rep" of Boats' pattern would mislabel it. Rather than force a mismatched extra problem in to hit a quota, Boats gets full depth (including a real proof of its greedy step) and this sub-variant gets one strong rep instead of a padded, inaccurate second one — consistent with "don't pad a pattern that's genuinely narrow or a one-off." (This distinction — same surface vocabulary, different underlying pointer mechanics — is itself worth internalizing; it's built into today's problem sections below.)

---

## Recap

Yesterday extended 3Sum to 3Sum Closest — same skeleton, different objective. Today extends the *skeleton itself*: 4Sum adds a second outer loop around the same opposite-ends inner sweep, and Boats to Save Most People introduces a genuinely new decision rule within the opposite-ends family. Yesterday's overflow discussion stops being theoretical today — 4Sum's constraints make it a real correctness issue.

---

## Learning Objectives

1. Extend the outer-loop-plus-two-pointers skeleton from 3-Sum-shaped to 4-Sum-shaped, and correctly identify every level that needs duplicate-skipping.
2. Justify Boats to Save Most People's greedy pairing rule with a real exchange argument, not just "it works."
3. Explain, from the mechanism, why `String` concatenation in a loop is O(n²) — including why the compiler's own automatic `StringBuilder` optimization does *not* save you here.

---

## Concept Dependency Map

```
Day 10: 3Sum Closest — outer loop + opposite-ends inner sweep
        │
        ├──▶ Today: 4Sum — two outer loops + same inner sweep
        │            └─ needs: Day 10's overflow mechanics (long, not int)
        │
        └──▶ Today: Boats to Save Most People — opposite-ends,
                     new decision rule (pair-or-strand)

Week 1 Day 3: amortized analysis (ArrayList's doubling)
        │
        └──▶ Today: String Internals — StringBuilder's doubling,
                     same amortized argument reused
```

---

## Part 1 — Two Pointers: Extending to Four

### 4Sum (LC 18)

**Statement:** Given an array `nums` and target `target`, return all unique quadruplets `[a,b,c,d]` such that `a+b+c+d == target`.

### Approach 1 — Brute force

```java
public static List<List<Integer>> fourSumBruteForce(int[] nums, int target) {
    Arrays.sort(nums);                 // sorting first makes each stored quadruplet's order canonical
    Set<List<Integer>> resultSet = new HashSet<>();
    int n = nums.length;
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            for (int k = j + 1; k < n; k++) {
                for (int l = k + 1; l < n; l++) {
                    long sum = (long) nums[i] + nums[j] + nums[k] + nums[l];
                    if (sum == target) {
                        resultSet.add(Arrays.asList(nums[i], nums[j], nums[k], nums[l]));
                    }
                }
            }
        }
    }
    return new ArrayList<>(resultSet);
}
```

Four nested loops checking every combination, deduplicated via a `Set`. Time O(n⁴), Space O(n) for dedup bookkeeping.

**Intermediate (worth naming, not required to implement):** three nested loops fixing three of the four values, then a HashSet lookup for the fourth — analogous to how Two Sum's HashMap approach relates to Two Sum II's two-pointer approach. Time O(n³) average, but carries hashing overhead and doesn't reuse the sorted-array structure the way two pointers do — a real approach, but not the one this pattern-family is building toward.

### Approach 2 — Optimized: sort + two outer loops + opposite-ends

```java
public static List<List<Integer>> fourSum(int[] nums, int target) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    int n = nums.length;

    for (int i = 0; i < n - 3; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;              // skip duplicate i

        for (int j = i + 1; j < n - 2; j++) {
            if (j > i + 1 && nums[j] == nums[j - 1]) continue;      // skip duplicate j

            int left = j + 1, right = n - 1;
            long remaining = (long) target - nums[i] - nums[j];     // long — see overflow note below

            while (left < right) {
                long sum = (long) nums[left] + nums[right];
                if (sum == remaining) {
                    result.add(Arrays.asList(nums[i], nums[j], nums[left], nums[right]));
                    while (left < right && nums[left] == nums[left + 1]) left++;    // skip duplicate left
                    while (left < right && nums[right] == nums[right - 1]) right--; // skip duplicate right
                    left++;
                    right--;
                } else if (sum < remaining) {
                    left++;
                } else {
                    right--;
                }
            }
        }
    }
    return result;
}
```

Sort `nums`; outer loop `i` fixes the first element, outer loop `j` fixes the second; the inner opposite-ends sweep — literally 3Sum Closest's inner loop with the target adjusted, but hunting for an *exact* match again (back to 3Sum's objective, not Closest's) — handles the last two. **Skip duplicates at every level:** `i`, `j`, `left`, and `right` all need a "if this value equals the previous one at this same level, skip" guard, or the same quadruplet gets emitted more than once. This is the direct generalization of 3Sum's duplicate-skipping rule, applied one level deeper at each of the four positions rather than just the outer one.

**Trace, confirming both the logic and the dedup guards:** `nums = [1,0,-1,0,-2,2]`, `target = 0` → sorted `[-2,-1,0,0,1,2]`. Walking the outer loops: `i=0(-2), j=1(-1)` → inner sweep finds `left=4(1), right=5(2)` summing to the needed `3` → `[-2,-1,1,2]`. `i=0(-2), j=2(0)` → inner sweep finds `left=3(0), right=5(2)` summing to the needed `2` → `[-2,0,0,2]`. `i=0(-2), j=3` is skipped (`nums[3]==nums[2]`, both `0`). `i=1(-1), j=2(0)` → inner sweep finds `left=3(0), right=4(1)` summing to the needed `1` → `[-1,0,0,1]`. Remaining `(i,j)` combinations either get skipped by a duplicate guard or produce no match. **Final: `[[-2,-1,1,2], [-2,0,0,2], [-1,0,0,1]]`** — matches the well-known expected output for this exact input.

**⚠️ Common Mistake — integer overflow, no longer hypothetical.** LeetCode's actual constraints for this problem allow `nums[i]` and `target` up to `±10⁹` in magnitude. Summing four such values can approach `±4×10⁹` — comfortably outside `int`'s range of roughly `±2.1×10⁹` (yesterday's exact table: `int` tops out at `2,147,483,647`). Computing `nums[i] + nums[j] + nums[left] + nums[right]` as `int` arithmetic can silently wrap (yesterday's two's-complement mechanism, not a new phenomenon) and produce a wrong comparison against `target` with no exception, no warning, no crash — just a wrong answer. **Use `long` for the running sum and all intermediate comparisons.** This is one of the most common real causes of an unexplained wrong answer on this specific problem, and it's a direct, concrete payoff of yesterday's theory block rather than an unrelated new fact.

**Complexity:** Time O(n³) — two nested O(n) outer loops × an O(n) inner two-pointer sweep. Space O(1) extra, excluding sort and output.

**Optimization worth naming (extension, not required for the day's deliverable):** early pruning per outer-loop level — if the four smallest remaining values already sum above `target`, or the four largest already sum below it, break out of that loop level immediately. Doesn't change worst-case complexity but meaningfully cuts real runtime on typical inputs.

**Edge cases:**
- Array length < 4: no valid quadruplets possible; return empty immediately (the outer loop bounds `i` up to `n-4` naturally produce zero iterations, so this needs no special-casing if the bounds are written correctly — but it's worth checking explicitly rather than trusting it silently).
- All identical elements: duplicate-skipping at every level should collapse this to at most one quadruplet in the output (or zero, if that repeated value ×4 doesn't hit the target) — a good self-check case to hand-trace.

**💡 Interview Insight:** if asked "how far does this generalize — could you do KSum?" — yes, and saying so explicitly is worth it: K-2 nested outer loops, wrapping the same opposite-ends inner sweep, with duplicate-skipping at every level. The technique doesn't fundamentally change past 4; only the constant-factor nesting depth does, and this is generally taught as a recursive generalization once you're comfortable with the fixed 3-and-4 cases.

---

### Boats to Save Most People (LC 881)

**Statement:** `people[i]` is the weight of the i-th person. Each boat carries at most 2 people, with combined weight at most `limit`. Return the minimum number of boats needed to carry everyone.

**Brute force:** try all ways of pairing people into boats, minimize the count — factorial-scale, not viable, and not worth coding to make the point.

### Optimized — greedy insight, then two pointers

```java
public static int numRescueBoats(int[] people, int limit) {
    Arrays.sort(people);
    int left = 0, right = people.length - 1;
    int boats = 0;

    while (left <= right) {
        if (people[left] + people[right] <= limit) {
            left++;          // lightest remaining fits with the heaviest — both go
        }
        right--;             // heaviest remaining ALWAYS goes this round, paired or alone
        boats++;
    }
    return boats;
}
```

Sort `people` ascending. `left` tracks the lightest unassigned person, `right` the heaviest. Each iteration launches exactly one boat carrying `people[right]` — paired with `people[left]` if they fit together (`left++` too), alone otherwise. `right--` and `boats++` happen unconditionally every iteration, which is what makes this compact: the heaviest remaining person is *always* accounted for in the boat this iteration represents, whether or not they got a companion.

**Why this is correct — a real exchange argument, not an assertion:** the heaviest remaining person must go on *some* boat, alone or paired. If they can be paired with anyone at all, pairing them with the *lightest* remaining person is never worse than any other valid pairing choice: suppose instead they were paired with some heavier person X, and the lightest person was paired with someone else (or went alone). Since the lightest person's weight is the smallest available, they fit with strictly more potential partners than any other unpaired person does — meaning whatever boat *they* end up on could just as easily have held the heaviest person instead, without exceeding the limit (the heaviest+lightest pairing was already shown to fit). Swapping the pairing this way changes nothing about total boats used but never makes things worse, which is the standard shape of an exchange argument: any optimal solution can be transformed, step by step, into the greedy solution's structure without increasing the boat count — so the greedy choice is at least as good as any alternative. And if the heaviest person genuinely can't fit with even the lightest remaining person, no pairing at all is possible for them (every other remaining person is heavier still), so "alone" isn't just greedy, it's forced.

**Worked trace:** `people = [3,2,2,1]`, `limit = 3`. Sorted: `[1,2,2,3]`.

| left | right | people[left]+people[right] | vs limit | action | boats |
|---|---|---|---|---|---|
| 0(1) | 3(3) | 4 | >3, can't pair | heaviest alone, right-- | 1 |
| 0(1) | 2(2) | 3 | ≤3, pair | both go, left++, right-- | 2 |
| 1(2) | 1(2) | — | single person left | goes alone | 3 |

Final: **3 boats.** Manual verification: only pair among all of `{1,2,2,3}` that fits within limit 3 is `(1,2)` — `2+2=4>3` and `2+3=5>3` both fail — so at most one pair is possible, meaning the other two people each need their own boat: `1 pair + 2 singles = 3 boats` total, confirming 3 is optimal, not just what the algorithm happens to produce.

**Complexity:** Time O(n log n) — dominated by the sort; the two-pointer sweep itself is O(n). Space O(1) extra (O(log n) to O(n) for the sort itself, depending on the sorting algorithm's implementation, which is standard and not something this problem introduces).

**Edge cases:**
- Single person: `left == right` on the first check — trivially their own boat, `boats = 1`.
- Everyone fits together in pairs: the "pair" branch fires every iteration, `boats = n/2` (or `(n+1)/2` with an odd remainder going alone at the very end).
- No one can ever pair (every weight already close to `limit`): the "alone" branch fires every iteration, `boats = n` — degrades gracefully to "everyone alone," which is correct when that's genuinely the only option.

**⚠️ Common Mistake:** trying to pair greedily from the *lightest* end without checking against the heaviest first (e.g., pairing adjacent lightweights together) — this is a different, incorrect greedy rule. The correctness argument above specifically relies on always testing the *current heaviest remaining* person against the *current lightest remaining* person; pairing two arbitrary lights together can waste capacity that the heaviest person actually needed.

**💡 Interview Insight — the "looks similar, isn't" trap named above is worth surfacing yourself.** If you've also solved Assign Cookies (LC 455) — sort both children's greed and cookies' sizes, greedily satisfy the least-greedy child with the smallest sufficient cookie — it's tempting to file it as "the same pattern" as Boats, since both are sorted, greedy, and O(n log n). They're not the same pointer shape: Assign Cookies advances one pointer unconditionally every step and the other only on a match (Week 1's "one-forward-pointer-each," the same shape as Is Subsequence), while Boats converges two pointers from opposite ends with a genuine branch on whether to pair or strand. Two problems can share "sort, then greedy, then linear scan" as a high-level strategy while using different two-pointer mechanics underneath — noticing which one you're actually looking at, rather than pattern-matching on vocabulary, is exactly the skill this whole series is built around.

---

## Part 2 — String Internals

### Prerequisites (confirmed)

- Strings introduced as objects, `==` vs. `.equals()` for Strings (Week 1, Day 2).
- Stack vs. heap (Day 9) — Strings live on the heap like any other object; today adds the specific mechanics of *how* they're managed there.
- Amortized analysis, specifically justified for `ArrayList`'s doubling strategy (Week 1, Day 3) — today reuses that exact argument for `StringBuilder`.

### Immutability, Precisely

**Definition:** once a `String` object is created, its character content can never change — there is no method on `String` that mutates it in place.

Every method that looks like a modification (`concat()`, `replace()`, `substring()`, `toUpperCase()`, string `+`) returns a **brand-new** `String` object; the original is untouched and still exists, referenced or not, until garbage collected.

```java
String s = "hello";
s.concat(" world"); // creates a new String — and immediately discards it, since the result isn't assigned
System.out.println(s); // still "hello" — s itself was never touched
```

### The String Pool

**Definition:** the string pool (also "intern pool") is a special reused-object cache, living on the heap, that stores string **literals** (text written directly in source code, like `"hello"`).

When the compiler encounters a string literal, it checks the pool first: if an equal value is already there, the existing reference is reused; otherwise, a new entry is added.

```java
String a = "hello";
String b = "hello";
System.out.println(a == b); // true — both point at the SAME pooled object

String c = new String("hello");
System.out.println(a == c); // false — new String() always allocates fresh, bypassing the pool
System.out.println(a.equals(c)); // true — value equality is unaffected either way
```

**🔗 This is the exact same shape as yesterday's Integer cache trap**, one abstraction layer over: a reuse mechanism makes `==` *coincidentally* report `true` for some cases (pooled literals) while remaining, underneath, a pure reference comparison — `.equals()` is the only reliable check for value equality, for the same underlying reason in both cases.

### Why `+=` in a Loop Is Quietly O(n²)

Because `String` is immutable, `s += x` inside a loop doesn't append to `s` — it builds an entirely new `String`, which requires allocating a new backing array and **copying every existing character into it**, plus the new content. Do this `n` times, and the total copying work is `1 + 2 + 3 + ... + n`, an arithmetic series summing to `O(n²)`.

```java
String result = "";
for (int i = 0; i < n; i++) {
    result += data[i]; // full copy of everything accumulated so far, every single iteration
}
```

**💡 Interview Insight — the nuance most people miss entirely.** Modern `javac` *does* automatically rewrite a single `+`-concatenation statement into `StringBuilder` calls under the hood (`a + b + c` roughly compiles to `new StringBuilder().append(a).append(b).append(c).toString()`). It's tempting to conclude this fixes the loop case too — it does not. The compiler's rewrite happens **per statement**, not across loop iterations: each pass through the loop body creates a **fresh** `StringBuilder`, appends the accumulated `result` (a full copy of everything so far) plus the new piece, and immediately calls `toString()` — throwing that `StringBuilder` away at the end of the very same iteration. The compiler cannot merge separate iterations of a loop into a single accumulating `StringBuilder`, because it has no way to know in advance how many iterations will run or that `result` is being used purely as an accumulator. So the O(n²) cost survives the "hidden" compiler optimization completely intact — this is worth stating precisely if it comes up, since "doesn't the compiler already use StringBuilder?" is exactly the kind of half-remembered fact that leads people to wrongly wave off a real performance bug.

### `StringBuilder` — the Actual Fix

`StringBuilder` wraps a single **mutable** internal character buffer (backed by a resizable array), over-allocated and **doubled** in capacity when it fills up — the identical doubling strategy `ArrayList` uses, justified by the identical amortized argument from Week 1, Day 3: most `append()` calls are cheap O(1) copies into existing free space; the occasional resize-and-copy is expensive, but averaged (amortized) over all `n` calls, the cost per call is still O(1). Total cost across `n` appends: O(n), not O(n²) — because unlike the `+=` case, there is exactly **one** buffer being incrementally grown, not a brand-new full-content copy on every iteration.

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) {
    sb.append(data[i]); // O(1) amortized — no full-content copy most iterations
}
String result = sb.toString(); // one final conversion
```

### Common Mistakes

- **⚠️ Believing the compiler's per-statement `StringBuilder` rewrite protects loop concatenation.** Covered above — it doesn't, and this is worth stating exactly if asked, not glossed over.
- **⚠️ Calling `.toString()` inside the loop instead of once at the end**, if manually managing a `StringBuilder` in an unusual way — defeats the mutable-buffer advantage entirely by forcing a fresh `String` allocation every iteration anyway.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** `StringPerformance` benchmark class, explanation in the class Javadoc.

Practical guidance: concatenate a string 10,000 times with `+=`, time it (`System.nanoTime()` before/after); do the same with `StringBuilder.append()`; print both durations side by side. Expect a dramatic, easily visible gap at n=10,000 — if the gap looks small, the JIT may be optimizing more aggressively than expected at that scale; try n=50,000 or 100,000 to make the O(n²) vs O(n) gap unmistakable. The Javadoc should state, in your own words, why the gap exists — referencing the full-copy-per-iteration mechanism, not just "StringBuilder is faster."

## Career Block Guide (1 hr)

- **LinkedIn engagement (20 min):** 3–5 substantive comments, same standard as prior days.
- **Networking:** reach out to 1 peer or former coworker, purely to stay warm on the relationship — no ask attached. A short, specific message (referencing something real — a shared project, a recent update in their career, an article relevant to both of you) reads as genuine; a message that opens with an ask reads as transactional, and this block is explicitly about the former.

---

## Day 11 — Interview Questions

**Q1. Walk through 4Sum's approach.** Sort, two nested outer loops fixing two elements, inner opposite-ends two-pointer sweep on the remainder for an exact-sum pair, duplicate-skipping at all four levels (both outer indices and both inner pointers) to avoid emitting the same quadruplet twice.

**Q2. Why does 4Sum specifically need `long` for its running sums, when 3Sum didn't need to worry about this?** LeetCode's constraints allow individual values up to `±10⁹`; summing four of them can approach `±4×10⁹`, which exceeds `int`'s roughly `±2.1×10⁹` range and can silently wrap via two's complement, producing a wrong comparison with no error thrown.

**Q3. Justify Boats to Save Most People's greedy pairing rule — why pair lightest with heaviest specifically?** An exchange argument: the heaviest remaining person must go on some boat; if any pairing works for them, pairing with the lightest remaining person is never worse than any alternative, since the lightest person could have shared with strictly more potential partners than anyone else — any other valid pairing can be reshuffled into this one without increasing the total boat count.

**Q4. Are Boats to Save Most People and Assign Cookies the same two-pointer pattern?** No, despite both being sorted-and-greedy: Boats converges two pointers from opposite ends with a pair-or-strand branch; Assign Cookies advances two pointers independently in the same direction, only one of them conditionally — structurally the "one-forward-pointer-each" shape, the same as Is Subsequence.

**Q5. Why is naive `String` concatenation in a loop O(n²)?** Strings are immutable, so each `+=` builds an entirely new object and copies every previously accumulated character into it; summed over `n` iterations, that's an arithmetic series, `O(n²)` total.

**Q6. Doesn't the compiler already rewrite `+` into `StringBuilder` calls — so why does the loop case still cost O(n²)?** The compiler's rewrite is per-statement: each loop iteration gets its own fresh, throwaway `StringBuilder` that still has to copy in everything accumulated so far as its starting point. The optimization never spans multiple iterations, so the full-copy-per-iteration cost survives intact.

**Q7. Why is `StringBuilder.append()` in a loop O(n) instead of O(n²)?** One single mutable buffer is grown in place (doubling when full, the same amortized-O(1)-per-operation argument as `ArrayList`), rather than a fresh full-content copy being made on every append.

**Q8. What determines whether `==` returns `true` for two `String` variables holding the same text?** Whether both references point at the same object — which happens automatically for pooled literals, but not for `new String(...)`, regardless of whether the underlying text is identical; `.equals()` is the only reliable value check.

---

## Daily Deliverable Check

- [ ] 4Sum solved, pushed to `dsa-java/two-pointers/`, using `long` for running sums.
- [ ] Boats to Save Most People solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain why naive String concatenation in a loop is O(n²) — including why the compiler's automatic `StringBuilder` rewrite doesn't fix it.
- [ ] `StringPerformance` benchmark pushed, with a Javadoc explanation in your own words.

---

## What Tomorrow Assumes You Already Know Cold

Day 12 assumes today's overflow-awareness (long vs. int for accumulating sums) and the opposite-ends skeleton are both fully reflexive — Container With Most Water's recap tomorrow builds directly on today's exchange-argument style of reasoning, and Sort Colors' three-pointer partitioning is introduced as a structure that *doesn't* cleanly fit either "opposite-ends" or "fast-slow," which only lands if those two labels are already solid, not shaky.
