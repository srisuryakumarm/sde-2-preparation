# Day 41 — Stacks: Circular Variants, Min Stack, and Monotonic Stack Practice

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 40 Resource Book](Day40_Resource_Book.md)
**Next ▶:** [Day 42 Resource Book](Day42_Resource_Book.md)
**Companion to:** Day 41 of `Week_06_Revised.md`

---

## A Note on Today's Structure

`Week_06_Revised.md` gives today only a DSA Block and a Career Block — no Theory or Project block, unlike every other day this week. That's carried through faithfully here, not an omission. It leaves genuine room in the day, and per Day 40's note, that room is used for the Monotonic Stack extra practice that was deliberately deferred rather than added on Day 40 itself (Monotonic Stack's opening day). Today is the natural place for it: the pattern is now established, not opening, and both of today's required problems reuse it directly.

---

## Recap

Both required problems today reuse mechanisms from the last two days directly — Next Greater Element II is yesterday's exact monotonic-stack logic with one new wrapping technique layered on top; Min Stack is a new application of the two-stack idea from Day 40's recap (two stacks, playing different roles, working together). Neither introduces a new *kind* of stack usage, which is exactly why today has room for extra reps instead of new theory.

---

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Explain why simulating a circular array with `2n` iterations (using `i % n`) correctly reuses yesterday's monotonic stack logic unchanged, and why indices — not values — must be stored on the stack this time.
2. Prove, with a concrete counter-trace, why Min Stack's secondary stack must push on `<=` rather than strict `<`.
3. Recognize a *stateful, streaming* monotonic stack (one that persists across separate calls, not a single array pass) as the same underlying mechanism in a different shape.
4. Explain why Remove K Digits needs an *increasing* monotonic stack, the mirror image of Next Greater Element's decreasing one, and connect that flip to what each problem is actually optimizing for.

---

## Concept Dependency Map

```
Day 40: Monotonic Stack — decreasing invariant, amortized O(n) proof
(pushed once / popped at most once)
        │
        ├──▶ Today: Next Greater Element II (LC 503)
        │      SAME logic, +2 new pieces:
        │      · circular wrap → iterate 2n times, i % n
        │      · store INDICES (not values) — LC 503 allows duplicates
        │
        ├──▶ Today, extra: Online Stock Span (LC 901)
        │      SAME logic, persisted ACROSS separate method calls
        │      (a stream over time, not one pass over a fixed array)
        │
        └──▶ Today, extra: Remove K Digits (LC 402)
               MIRROR of the invariant: INCREASING stack, greedy —
               pop a larger digit when a smaller one follows it

Day 40 recap: two stacks, opposite roles (Implement Queue using Stacks)
        │
        └──▶ Today: Min Stack (LC 155) — two stacks again, different
               roles: one holds all values, one tracks the running min
```

---

## Part 1 — Required Problems

### Problem 4: Next Greater Element II (LeetCode 503, Medium) — Pattern: Monotonic Stack, Circular **(new)**

**Statement:** `nums` is now a single, **circular** array — after the last element, "next" wraps back to index `0`. Find the next greater element for *every* index, `-1` if none exists even after a full wrap. (Unlike yesterday's LC 496, `nums` here may contain duplicate values.)

**What's actually new here — stated up front, since the underlying logic is not:** two things, layered on yesterday's unchanged mechanism. First, simulating circularity by iterating `2n` times instead of `n`, using `i % n` as the real index — this lets every element effectively "look" a full lap ahead without physically duplicating the array. Second, storing **indices** on the stack rather than values, since duplicate values would make a value-only stack unable to tell two equal entries apart.

```java
public int[] nextGreaterElements(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();   // holds INDICES

    for (int i = 0; i < 2 * n; i++) {
        int actualIndex = i % n;
        while (!stack.isEmpty() && nums[stack.peek()] < nums[actualIndex]) {
            result[stack.pop()] = nums[actualIndex];
        }
        if (i < n) {
            stack.push(actualIndex);      // only push during the FIRST lap
        }
    }
    return result;
}
```

**Why pushing only happens when `i < n`:** the second lap (`i` from `n` to `2n-1`) exists purely to give *already-stacked, still-unresolved* indices from the first lap one more chance to find their answer as the "virtual" wraparound plays out — it doesn't need to introduce new entries for positions already represented on the stack. Pushing again during the second lap would create duplicate stack entries for the same index, corrupting the invariant.

**Worked trace, `nums = [1, 2, 1]`** (`n = 3`):

| i | actualIndex | nums[actualIndex] | stack before | action | stack after |
|---|---|---|---|---|---|
| 0 | 0 | 1 | `[]` | push (i<n) | `[0]` |
| 1 | 1 | 2 | `[0]` | `nums[0]=1<2` → pop 0, result[0]=2; push 1 | `[1]` |
| 2 | 2 | 1 | `[1]` | `nums[1]=2<1`? No; push 2 | `[1,2]` |
| 3 | 0 | 1 | `[1,2]` | `nums[2]=1<1`? No (equal) | `[1,2]` |
| 4 | 1 | 2 | `[1,2]` | `nums[2]=1<2` → pop 2, result[2]=2; `nums[1]=2<2`? No | `[1]` |
| 5 | 2 | 1 | `[1]` | `nums[1]=2<1`? No | `[1]` |

Final `result = [2, -1, 2]`. **Verify directly:** index 0 (`1`) → next element `2` is greater ✓. Index 1 (`2`) → wrapping all the way around, nothing exceeds `2` ✓ (`-1`). Index 2 (`1`) → wraps to index 0 (`1`, not greater), then index 1 (`2`, greater) ✓.

**Complexity: Time O(n)** — `2n` iterations, and the same "pushed at most once (first lap only), popped at most once overall" amortized argument from yesterday applies unchanged; the constant factor of 2 doesn't change the big-O. **Space O(n).**

**Edge cases:**
- All elements equal: no `<` comparison ever succeeds — every result stays `-1`.
- Single element: `2n = 2` iterations; the element compares against itself on the second lap, finds nothing greater, correctly returns `[-1]`.

**💡 Interview Insight:** stating "this is yesterday's exact algorithm, plus a `2n`/`i % n` wrap and index-based storage instead of value-based" is a stronger opening than re-deriving the monotonic invariant from scratch — it signals the pattern is genuinely internalized, not re-learned per problem.

---

### Problem 5: Min Stack (LeetCode 155, Medium) — Pattern: Two Stacks

**Statement:** Design a stack supporting `push`, `pop`, `top`, and `getMin` — **all in O(1).**

**Why a single stack with an "also track the min" field doesn't work:** the minimum can change on *any* `pop()`, not just the current top's own value — if the current minimum gets popped, whatever the *next* minimum is has to already be known instantly, not recomputed by scanning what remains.

```java
class MinStack {
    private Deque<Integer> stack = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>();

    public void push(int val) {
        stack.push(val);
        if (minStack.isEmpty() || val <= minStack.peek()) {
            minStack.push(val);
        }
    }

    public void pop() {
        int val = stack.pop();
        if (val == minStack.peek()) {
            minStack.pop();
        }
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return minStack.peek();
    }
}
```

A second stack tracks the running minimum *as of each point in the push history* — `minStack.peek()` is always the correct current minimum, in O(1), with no scan.

**Why `push` uses `val <= minStack.peek()`, not strict `<` — proof by counter-example, not assertion.** Trace `push(2), push(0), push(3), push(0), pop(), getMin()` **with the buggy strict-`<` version**:

| call | `stack` | `val < minStack.peek()`? | `minStack` |
|---|---|---|---|
| push(2) | `[2]` | (empty → push) | `[2]` |
| push(0) | `[2,0]` | `0<2` → push | `[2,0]` |
| push(3) | `[2,0,3]` | `3<0`? No | `[2,0]` |
| push(0) | `[2,0,3,0]` | `0<0`? **No (strict fails on equal)** | `[2,0]` — **the second `0` is silently NOT recorded** |
| pop() | `[2,0,3]` | popped val `0` `==` `minStack.peek()` `0` → **pops it** | `[2]` — **now wrong!** |
| getMin() | | | returns `2` |

**`getMin()` now incorrectly returns `2`**, when the actual minimum of the remaining stack `[2,0,3]` is `0`. The bug: the second `push(0)` was skipped (since `0 < 0` is false), so `minStack` never recorded that there were *two* `0`s — the subsequent `pop()` then wrongly treated the single `minStack` entry as fully consumed.

**Now the correct `<=` version, same sequence:**

| call | `stack` | `val <= minStack.peek()`? | `minStack` |
|---|---|---|---|
| push(2) | `[2]` | (empty → push) | `[2]` |
| push(0) | `[2,0]` | `0<=2` → push | `[2,0]` |
| push(3) | `[2,0,3]` | `3<=0`? No | `[2,0]` |
| push(0) | `[2,0,3,0]` | `0<=0`? **Yes** → push | `[2,0,0]` |
| pop() | `[2,0,3]` | popped val `0` `==` `minStack.peek()` `0` → pop | `[2,0]` |
| getMin() | | | returns `0` — **correct** |

Recording the duplicate minimum a second time is exactly what lets `minStack` still correctly reflect `0` as the minimum after one of the two `0`s is popped.

**Complexity: Time O(1) for every operation — `push`, `pop`, `top`, `getMin` each touch only the top of one or both stacks. Space O(n)** worst case (a non-increasing sequence of pushes doubles up on both stacks every time).

**Edge cases:** popping down to empty leaves both stacks empty correctly; `getMin()`/`top()` are only ever called on a non-empty stack per the problem's own constraints — worth stating that assumption out loud if asked.

**💡 Interview Insight:** the `<=` fix is small enough to type without thinking, but *explaining* why strict `<` fails — with a concrete duplicate-value trace, not just "trust me" — is what actually distinguishes a correct-by-luck submission from a genuinely understood one.

---

## Part 2 — Monotonic Stack Extra Practice

The two problems below aren't from the plan — they're here because Monotonic Stack is this week's highest-value pattern, and today's schedule has the room. Both use the exact invariant from Day 40, applied to shapes that look nothing alike on the surface.

### Extra Practice: Online Stock Span (LeetCode 901, Medium)

**Statement:** Design `StockSpanner`. Each call to `next(price)` gives *today's* price and returns the **span** — the number of consecutive days, ending today and counting backward, where the price has been `≤` today's price.

**What's genuinely new here versus Days 40–41's array-based problems:** the stack must **persist across separate method calls** over time — this isn't one pass over a fixed array, it's a stream, and the "array" is effectively being built one element at a time by the caller. Recognizing that the same monotonic mechanism still applies, just wrapped in a class holding state between calls, is the actual skill this problem is building.

```java
class StockSpanner {
    private Deque<int[]> stack = new ArrayDeque<>();   // [price, span]

    public int next(int price) {
        int span = 1;
        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            span += stack.pop()[1];
        }
        stack.push(new int[]{price, span});
        return span;
    }
}
```

**Why popped spans get *accumulated*, not discarded:** if a previous day's price was `≤` today's, every day *that* day's own span already covered is also `≤` today's price — those days don't need to be re-counted individually; their entire span folds into today's in one step, which is exactly what keeps this O(1) amortized per call instead of re-scanning history.

**Worked trace, calls in order `100, 80, 60, 70, 60, 75, 85`:**

| call | stack before (price,span) | pops (accumulated) | span returned | stack after |
|---|---|---|---|---|
| 100 | `[]` | none | 1 | `[(100,1)]` |
| 80 | `[(100,1)]` | none (`100≤80`? No) | 1 | `[(100,1),(80,1)]` |
| 60 | `...,(80,1)` | none | 1 | `...,(80,1),(60,1)` |
| 70 | `...,(60,1)` | pop `(60,1)` → span=2 | 2 | `...,(80,1),(70,2)` |
| 60 | `...,(70,2)` | none (`70≤60`? No) | 1 | `...,(70,2),(60,1)` |
| 75 | `...,(60,1)` | pop `(60,1)`→2, pop `(70,2)`→4 | 4 | `(100,1),(80,1),(75,4)` |
| 85 | `...,(75,4)` | pop `(75,4)`→5, pop `(80,1)`→6 | 6 | `(100,1),(85,6)` |

Results `[1,1,1,2,1,4,6]` — each one directly verifiable by counting consecutive `≤`-today days backward from that point.

**Complexity: each call is amortized O(1)** — the identical "pushed once, popped at most once" argument, now spread across the object's *entire lifetime* of calls rather than a single array pass. **Space O(N)** for N total calls.

**💡 Interview Insight:** naming this explicitly as "the same monotonic stack, but the state persists between calls instead of resetting each time" is the generalization worth stating unprompted — it's the difference between having memorized "monotonic stack solves next-greater-style array problems" and actually understanding what makes the mechanism work in the first place.

---

### Extra Practice: Remove K Digits (LeetCode 402, Medium)

**Statement:** Given a non-negative integer as a string `num` and an integer `k`, remove exactly `k` digits so the remaining digits (kept in their original order) form the **smallest possible** number. No leading zeros in the result, except the result `"0"` itself.

**The greedy insight, stated first:** a smaller digit *earlier* always beats a larger digit earlier, regardless of what follows — so whenever a digit is immediately followed by a strictly smaller one, removing the larger, earlier digit can only help. That's an **increasing** monotonic stack — the mirror image of Days 40–41's decreasing one, because this problem wants to eliminate exactly the pattern (a bigger digit sitting before a smaller one) that a decreasing stack was built to detect and act on for a *different* reason.

```java
public String removeKdigits(String num, int k) {
    Deque<Character> stack = new ArrayDeque<>();

    for (char digit : num.toCharArray()) {
        while (!stack.isEmpty() && k > 0 && stack.peek() > digit) {
            stack.pop();
            k--;
        }
        stack.push(digit);
    }

    while (k > 0 && !stack.isEmpty()) {   // number was already non-decreasing — trim from the end
        stack.pop();
        k--;
    }

    StringBuilder sb = new StringBuilder();
    while (!stack.isEmpty()) {
        sb.append(stack.pop());
    }
    sb.reverse();

    int start = 0;
    while (start < sb.length() - 1 && sb.charAt(start) == '0') {
        start++;                           // strip leading zeros
    }
    String result = sb.substring(start);

    return result.isEmpty() ? "0" : result;
}
```

**Worked trace, `num = "1432219"`, `k = 3`:**

| digit | stack before | action | stack after | k |
|---|---|---|---|---|
| 1 | `[]` | push | `[1]` | 3 |
| 4 | `[1]` | `1>4`? No; push | `[1,4]` | 3 |
| 3 | `[1,4]` | `4>3` → pop, k=2; `1>3`? No; push | `[1,3]` | 2 |
| 2 | `[1,3]` | `3>2` → pop, k=1; `1>2`? No; push | `[1,2]` | 1 |
| 2 | `[1,2]` | `2>2`? No; push | `[1,2,2]` | 1 |
| 1 | `[1,2,2]` | `2>1` → pop, k=0; push | `[1,2,1]` | 0 |
| 9 | `[1,2,1]` | `k=0`, no more pops; push | `[1,2,1,9]` | 0 |

`k` reaches `0` before the end — the "trim from the end" loop doesn't run. Building the result (popping reverses order back to left-to-right): `"1219"`. **Matches the known correct answer exactly.**

**Why the "trim from the end" fallback is needed at all:** if `num`'s digits are already non-decreasing throughout (e.g., `"12345"`), the `while (stack.peek() > digit)` condition never once fires — every digit just gets pushed. In that case, removing digits to minimize the number means removing from the **end** (the most significant *remaining* digits always matter more, so keep the earliest ones) — this is exactly what the fallback loop does.

**⚠️ Common Mistake — leading zeros after removal.** `num = "10200"`, `k = 1`: the greedy pass produces `"0200"` before stripping — the leading zero must be removed, giving `"200"`, *not* `"0200"`. The `start < sb.length() - 1` bound is deliberate: it stops short of stripping the *last* remaining character even if it's `"0"`, so a genuinely all-zero result correctly stays as the single character `"0"` rather than being stripped down to an empty string.

**Complexity: Time O(n)** — every digit is pushed once and popped at most once, across both the main loop and the end-trimming fallback combined (bounded by `min(k, n)` total pops either way); building the result string is a further O(n). **Space O(n).**

**Edge cases:** `k ≥ num.length()`: everything can be removed — the final `result.isEmpty() ? "0" : result` check handles this, returning `"0"` rather than an empty string. All-zero result after stripping (e.g., `num="10"`, `k=2`): correctly returns `"0"`.

**💡 Interview Insight:** stating the increasing-vs-decreasing distinction explicitly — "yesterday's problems wanted to know what's bigger and coming later; this one wants to eliminate a bigger digit sitting before a smaller one" — is a strong, concise way to show the pattern's shape is understood, not just its code.

---

## Career Block Guide (1 hr)

**LinkedIn (20 min):** engagement — comment on 3–5 posts.

**Networking:** review 3 engineering manager profiles — look for shared connections, recent posts, or team focus areas that would make a future outreach message genuinely specific rather than generic; no outreach yet, just the groundwork.

---

## Day 41 — Interview Questions

**Q1. What two things does Next Greater Element II add on top of yesterday's monotonic stack logic?** Iterating `2n` times with `i % n` to simulate circular wraparound, and storing indices instead of values, since this version of the problem allows duplicate values that a value-only stack couldn't distinguish.

**Q2. Why does Next Greater Element II only push during the first `n` iterations?** The second lap exists solely to give already-stacked, still-unresolved indices a chance to resolve against the simulated wraparound — pushing again during the second lap would create duplicate entries for positions already represented on the stack.

**Q3. Why must Min Stack's secondary stack push using `<=` rather than strict `<`?** Using strict `<` skips recording a value tying the current minimum — when that minimum is later popped from the main stack, the secondary stack incorrectly believes the minimum has changed, when in fact an equal value is still present in what remains.

**Q4. What makes Online Stock Span's use of a monotonic stack different from an array-based next-greater problem?** The stack persists across separate method calls over time rather than resetting for one fixed-array pass — it's the identical amortized mechanism, applied to a stream instead of a static array.

**Q5. Why does Online Stock Span accumulate a popped entry's span instead of discarding it?** Every day covered by a popped entry's span was already `≤` that entry's price, and that price is itself `≤` today's — so all of those days are transitively `≤` today's price too, and folding their count directly into today's span avoids re-counting them individually.

**Q6. Why does Remove K Digits use an increasing stack instead of a decreasing one?** The goal is to eliminate a larger digit sitting before a smaller one, since a smaller leading digit always produces a smaller number regardless of what follows — the exact mirror of what a decreasing stack tracks.

**Q7. When does Remove K Digits need to trim from the end rather than during the main scan?** When the input digits are already non-decreasing throughout, so the main loop's pop condition never fires — removing digits to minimize the number then means dropping the least significant (rightmost) remaining digits instead.

---

## Daily Deliverable Check

- [ ] Next Greater Element II (LC 503) and Min Stack (LC 155) solved, both traced by hand, pushed to `dsa-java/stacks/`.
- [ ] Online Stock Span (LC 901) and Remove K Digits (LC 402) solved as extra Monotonic Stack practice, pushed alongside.
- [ ] Can reproduce the Min Stack `<=`-vs-`<` counter-example from memory, not just recall the fix.
- [ ] Can state, in one sentence each, what's new in the circular variant and what's new in the streaming variant of Monotonic Stack.
- [ ] LinkedIn engagement done. 3 engineering manager profiles reviewed.

---

## What Tomorrow Assumes You Already Know Cold

Tomorrow closes the week with Daily Temperatures and Evaluate Reverse Polish Notation, plus a cold self-check on a Linked List problem from earlier this week. Daily Temperatures reuses today's exact index-based Monotonic Stack storage (from Next Greater Element II) directly, with no re-explanation. The self-check assumes Linked List fluency hasn't faded now that four full days of Stacks have gone by since — if it has, that's worth surfacing honestly tomorrow rather than papering over.
