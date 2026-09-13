# Day 8 — Two Pointers Continues, and Recursion Deep Dive

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 7 Resource Book](Day7_Resource_Book.md) (Week 1)
**Next ▶:** [Day 9 Resource Book](Day9_Resource_Book.md)
**Companion to:** Day 8 of `Week_02_Revised.md`

---

## ⚠️ Overlap Notice — Read This First

Both of today's plan-required DSA problems were already solved in Week 1, as extra practice added on top of that week's official ladder:

| LC # | Problem | Originally solved |
|---|---|---|
| 26 | Remove Duplicates from Sorted Array | Week 1, Day 6 (Extra Practice) |
| 283 | Move Zeroes | Week 1, Day 7 (Extra Practice) |

Per the overlap rule in the generation prompt, both get a short recap instead of a full re-teach, and the DSA time this frees up goes to one genuinely new problem in the same variant — **Remove Duplicates from Sorted Array II (LC 80)** — so today still delivers real new reps, not just review. This is exactly the situation `00_Curriculum_Map.md` flagged in its "Known Overlap" section; today resolves it.

---

## Recap

Week 1 closed with the Two Pointers pattern family fully introduced (opposite-ends, from-the-back, fast-slow, one-forward-pointer-each, opposite-ends-with-one-allowed-skip) across 12 problems, and you left Day 7 able to name which variant an unfamiliar problem calls for before writing code. Today re-enters the **same-direction (fast/slow)** variant specifically — the one where both pointers move rightward together, one scanning ahead (`fast`) and one marking where the next "keep" element goes (`slow`).

Today also opens an entirely new thread: **Recursion**, built from nothing but what Week 1 already gave you about methods and the call stack.

---

## Learning Objectives

By the end of today, without notes, you should be able to:
1. State the fast-slow invariant for Remove Duplicates and Move Zeroes from memory, and extend it correctly to "at most 2 occurrences" (LC 80).
2. Explain what a stack frame is, what it contains, and why unbounded recursion throws `StackOverflowError` — and why that's an `Error`, not an `Exception`.
3. Write a correct base case and recursive case for a new problem, and justify correctness by induction, not just "it worked on my test case."
4. Trace a small recursive call tree by hand (you'll do this for `fib(5)`) and explain, concretely, why naive recursive Fibonacci recomputes work.

---

## Concept Dependency Map

```
Week 1 Day 1: Methods, parameters, pass-by-value (primitives)
Week 1 Day 6-7: Two Pointers — fast-slow variant (LC 26, LC 283)
        │
        ├──▶ Today: LC 80 — fast-slow, generalized to "allow K=2"
        │
        └──▶ Today: Recursion
                 ├─ built on: method calls (Week 1 Day 1)
                 ├─ new: the call stack, made explicit
                 ├─ new: base case / recursive case as a formal contract
                 └─ previews: memoization (Dynamic Programming, later week)
```

---

## Part 1 — Two Pointers: Recap and One New Rep

### 🔗 Recap: Remove Duplicates from Sorted Array (LC 26)

**Original coverage:** Week 1, Day 6, Extra Practice. Full treatment there — this is the optimized solution only, for reference; open Week 1's Day 6 book for the brute-force derivation if it doesn't come back immediately.

```java
public static int removeDuplicates(int[] nums) {
    int slow = 1;                              // first element is trivially unique
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow - 1]) {     // genuinely new value
            nums[slow] = nums[fast];
            slow++;
        }
    }
    return slow;
}
```

`slow` marks the boundary of the deduplicated prefix built so far; `fast` scans every element once. Whenever `nums[fast] != nums[slow-1]`, that's a genuinely new value — write it at `nums[slow]`, then advance both. Time O(n), Space O(1). If you can't reproduce this from memory in under a minute, that's the signal to open Week 1's Day 6 book before moving on — everything below assumes it's solid.

### 🔗 Recap: Move Zeroes (LC 283)

**Original coverage:** Week 1, Day 7, Extra Practice. Optimized solution only, for reference.

```java
public static void moveZeroes(int[] nums) {
    int slow = 0;                          // next slot for a non-zero value
    for (int fast = 0; fast < nums.length; fast++) {
        if (nums[fast] != 0) {
            int temp = nums[slow];
            nums[slow] = nums[fast];
            nums[fast] = temp;              // SWAP, not overwrite — see below for why
            slow++;
        }
    }
}
```

Same fast-slow skeleton as LC 26, different write rule: `slow` marks the next slot for a non-zero element. When `fast` finds a non-zero, **swap** `nums[slow]` and `nums[fast]` (not overwrite — swapping is what correctly pushes the zeros to the back instead of just duplicating values), then advance both `slow` and `fast`. When `fast` finds a zero, only `fast` advances. Time O(n), Space O(1).

**🔑 Key Takeaway (recap):** LC 26 *overwrites* (source array has no zeros to preserve elsewhere); LC 283 *swaps* (has to relocate the zeros somewhere, not discard them). Same skeleton, different write rule, because the two problems have different constraints on what happens to the "discarded" elements. This is precisely the kind of distinction a tier-1 interviewer expects you to articulate unprompted.

---

### New Problem: Remove Duplicates from Sorted Array II (LC 80)

**Statement:** Given an array `nums` sorted in non-decreasing order, remove duplicates in place such that each unique element appears **at most twice**. Return the new length `k`. The first `k` elements of `nums` should hold the result (order matters, extra elements beyond `k` don't).

This is the standard, real interview follow-up to LC 26 — "now allow up to two copies" is one of the most commonly asked extensions of this exact problem, which is exactly why it earns the freed-up slot today.

### Approach 1 — Brute force

```java
public static int removeDuplicatesBruteForce(int[] nums) {
    if (nums.length == 0) return 0;
    List<Integer> result = new ArrayList<>();
    result.add(nums[0]);
    int runLength = 1;                          // how many times the CURRENT value has repeated so far
    for (int i = 1; i < nums.length; i++) {
        runLength = (nums[i] == nums[i - 1]) ? runLength + 1 : 1;
        if (runLength <= 2) {
            result.add(nums[i]);
        }
    }
    for (int i = 0; i < result.size(); i++) {
        nums[i] = result.get(i);                // copy back into the original array
    }
    return result.size();
}
```

Build a new list, tracking how many times the current value has repeated consecutively; append only while that run length is ≤ 2, then copy back into `nums`. Correct, but uses O(n) extra space and does more bookkeeping than necessary. Time O(n), Space O(n).

### Approach 2 — Optimized: fast-slow, generalized

```java
public static int removeDuplicates(int[] nums) {
    if (nums.length <= 2) return nums.length;   // fewer than 3 elements can never violate "at most 2" — see below
    int slow = 2;
    for (int fast = 2; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow - 2]) {     // safe to keep — can't create a 3rd consecutive copy
            nums[slow] = nums[fast];
            slow++;
        }
    }
    return slow;
}
```

`slow = 2`, `fast = 2` — the first two elements are always valid, since any array of length ≤ 2 trivially satisfies "at most 2 occurrences." For each `fast` from 2 to n-1: if `nums[fast] != nums[slow - 2]`, it's safe to keep — write `nums[slow] = nums[fast]`, then `slow++`. `fast` advances every iteration regardless. Return `slow`.

**⚠️ Common Mistake worth catching here specifically — this is a real bug, not a hypothetical one.** It's tempting to assume the `nums.length <= 2` guard is unnecessary, reasoning "the loop starts at `fast = 2`, so for a length-0 or length-1 array the loop body just never runs, and returning `slow` (initialized to `2`) is fine." **It is not fine** — for `nums = []` this would incorrectly return `2` instead of `0`; for `nums = [5]` it would incorrectly return `2` instead of `1`, both because `slow` was initialized to `2` regardless of whether the array actually *has* two elements. The guard isn't defensive boilerplate here — without it, this specific function returns a wrong length for the two shortest possible inputs. (It happens to work out, correctly, only for length exactly `2` — which is exactly the kind of boundary that's easy to test once, see it pass, and wrongly conclude the general case is fine.)

**Why `slow - 2` and not `slow - 1`?** You're asking: "would keeping this value create a *third* consecutive occurrence in the output built so far?" The last two elements written to the output are at `slow-2` and `slow-1`. If the incoming value differs from what's at `slow-2`, it's safe — either the two most recent output values were already different (meaning `slow-2` held some earlier, unrelated value), or they were equal (two copies of some value already banked), and a value different from *that* is definitely a new value with a clean slate. Comparing against `slow-1` would incorrectly block a legitimate *second* copy of the current run.

**⚠️ Common Mistake:** Comparing `nums[fast]` against the *original* array's earlier elements (e.g., keeping a separate "last seen" tracker read from unmodified positions) instead of against the **already-written output** at `nums[slow-2]`. The elegant part of this technique is that because `slow <= fast` always holds, positions `< slow` have already been finalized by your own writes — you can safely reuse the array itself as your own scratch space. Missing this is what makes people reach for an unnecessary counter variable or a HashMap.

**Worked trace:** `nums = [1,1,1,2,2,3]`, expecting `[1,1,2,2,3]`, k=5.

| fast | nums[fast] | nums[slow-2] | Action | slow after |
|---|---|---|---|---|
| 2 | 1 | nums[0]=1 | equal → skip | 2 |
| 3 | 2 | nums[0]=1 | differ → write nums[2]=2 | 3 |
| 4 | 2 | nums[1]=1 | differ → write nums[3]=2 | 4 |
| 5 | 3 | nums[2]=2 *(the value just written, not the original 1!)* | differ → write nums[4]=3 | 5 |

Final array (first 5 positions): `[1,1,2,2,3]`. `k=5`. Matches. Notice row 4: `nums[slow-2]` at that point reads a value **your own algorithm wrote earlier in this same pass** — this is the crux of the technique, and worth re-deriving by hand if it doesn't click immediately.

**Complexity:** Time O(n) — single pass, `fast` visits each index once. Space O(1) — in-place, no auxiliary structure.

**Edge cases:**
- Array length ≤ 2: handled by the explicit early-return guard above — *not* automatically correct from the loop bounds alone, as just shown.
- All elements identical: only the first two get kept, rest all skip — verify by hand with `[2,2,2,2]` → `[2,2]`, k=2.
- No duplicates at all: every comparison differs, every element gets written, `slow` ends up equal to `n` — behaves identically to a no-op copy.

**💡 Interview Insight:** If asked to generalize further ("at most K occurrences"), the pattern extends cleanly: compare `nums[fast]` against `nums[slow - K]`. Saying this out loud — recognizing the K=2 case as an instance of a K-generalizable idea, not a one-off trick — is exactly the kind of unprompted generalization that separates "solved this specific problem" from "understands the pattern."

---

## Part 2 — Recursion, Properly

### Prerequisites (confirmed)

- Method declaration, parameters, return values (Week 1, Day 1).
- Pass-by-value for primitives (Week 1, Day 1) — you'll need this exact fact again below, since it's what guarantees recursive calls can't accidentally corrupt a caller's local variables through a parameter.

### What Recursion Actually Is — The Mechanism

**Definition:** a recursive method is one that calls itself, directly or indirectly, on a smaller or simpler version of its own input. That's the label. Here's the mechanism underneath it.

Every method call — recursive or not — causes the JVM to push a **stack frame** (also called an *activation record*) onto the current thread's **call stack**.

**Definition:** a stack frame is a block of memory, pushed on every method invocation, holding:
- The method's parameters and local variables, as of that specific call.
- A return address — where execution resumes in the *caller* once this call finishes.
- Enough bookkeeping to restore the caller's own state.

When a method returns, its frame is **popped** — its memory is reclaimed immediately, no garbage collection needed (this is one of the reasons stack allocation is fast: reclaiming it is just moving a pointer back, not tracing reachability).

Here's what makes recursion different from a chain of distinct method calls: `factorial(5)` calling `factorial(4)` calling `factorial(3)`... is still just "one method calling another" from the JVM's point of view. It doesn't know or care that they're the "same" method. Each call — even a call to yourself — gets its own fresh frame, with its own independent copy of the parameters and locals. `factorial(3)`'s local variables are completely invisible to and independent from `factorial(4)`'s, even though it's "the same code." This is the single most important mental model to hold onto: **recursion is just ordinary method calls, arranged so that a method happens to call itself** — nothing magic, no shared state between the layers unless you explicitly pass it through parameters or return values.

```
factorial(5) called
┌─────────────────────┐
│ frame: factorial(5)  │  n=5, waiting on factorial(4)
├─────────────────────┤
│ frame: factorial(4)  │  n=4, waiting on factorial(3)
├─────────────────────┤
│ frame: factorial(3)  │  n=3, waiting on factorial(2)
├─────────────────────┤
│ frame: factorial(2)  │  n=2, waiting on factorial(1)
├─────────────────────┤
│ frame: factorial(1)  │  n=1 → base case, returns 1
└─────────────────────┘
        ▲ stack grows this direction as calls deepen,
          unwinds (pops) as each call returns
```

That diagram is tracing this exact method:

```java
public static long factorial(int n) {
    if (n <= 1) {                    // base case
        return 1;
    }
    return n * factorial(n - 1);     // recursive case
}
```

(`long`, not `int`, as the return type — factorial grows fast enough to overflow `int` past `n = 12` or so; Day 10 covers exactly why that overflow would be silent rather than an error, but there's no reason to walk into it here when the fix costs nothing.)

### Base Case and Recursive Case — A Formal Contract, Not a Vague Rule

Every correct recursive method needs exactly two ingredients:

1. **Base case:** at least one condition where the method returns a value directly, without calling itself. This is what stops the recursion.
2. **Recursive case:** the method calls itself on an input that is *strictly closer* to some base case — smaller, shorter, simpler, whatever "closer" means for this problem.

**Why it works (the actual justification, not just the rule):** this is a direct application of mathematical induction. If you can show (a) the base case is correct, and (b) *assuming* the recursive call on the smaller input returns the correct answer for that smaller input, the current call's logic correctly builds the right answer from it — then by induction, the method is correct for every input that eventually reaches the base case. You don't need to trace every layer by hand to trust a recursive method; you need to verify the base case and verify the inductive step, exactly once each. This is a fundamentally different (and, once internalized, faster) way to reason about correctness than mentally simulating the whole call stack — and it's also exactly how you should *justify* a recursive solution out loud in an interview.

**⚠️ Common Mistake:** a base case that's never actually reachable. `n <= 0` as your base case is useless if your recursive case calls `f(n + 1)` — the input is moving *away* from the base case, not toward it. This compiles fine and fails at runtime, often not until a much later, harder-to-diagnose input size.

### What Happens Without a Reachable Base Case

Each call keeps pushing a new frame, and no call ever returns to start popping them. The call stack — a **fixed-size** region of memory allocated per thread (its size is configurable via the JVM's `-Xss` flag, but it is never unbounded) — eventually fills up. At that point, the JVM throws `StackOverflowError`.

**⚠️ Precision matters here:** `StackOverflowError` is an `Error`, not a `RuntimeException`. Its full ancestry is `StackOverflowError → VirtualMachineError → Error → Throwable`. This is a genuinely common thing to get backwards — plenty of people assume it's a kind of `RuntimeException` because it happens at runtime and is unchecked (technically true — `Error` is unchecked too — but for a different reason, and part of a different branch of the exception hierarchy entirely). The distinction matters in practice: `Error` signals "something is wrong at a level the program generally shouldn't try to recover from," which is exactly the philosophy behind treating unbounded recursion as a bug to fix, not a condition to catch and handle.

### Trade-offs vs. Iteration

- **Recursion** tends to read more naturally when the *problem itself* is defined recursively — trees, divide-and-conquer, backtracking (all later in this plan). The code often mirrors the mathematical definition directly.
- **Cost:** every layer of recursion costs a stack frame — meaning O(depth) extra space beyond whatever the algorithm's own data structures need, which is easy to forget when tallying up an algorithm's space complexity. Deep recursion (proportional to a large `n`) risks `StackOverflowError` in a way an equivalent loop never would.
- **⚠️ Java does not perform tail-call optimization.** Some languages automatically rewrite a recursive call that's the very last operation in a function into a loop, eliminating the extra stack frames. Java's JVM does not do this, ever — a "tail-recursive-looking" Java method still consumes one stack frame per call, full stop. Don't assume otherwise, and don't repeat the "tail recursion is free in Java" claim to an interviewer — it isn't.

### Worked Trace: `fib(5)` and Why It Recomputes Work

Naive recursive Fibonacci — two base cases this time, which is completely normal (nothing requires exactly one):

```java
public static long fib(int n) {
    if (n == 0) return 0;              // base case 1
    if (n == 1) return 1;              // base case 2
    return fib(n - 1) + fib(n - 2);    // recursive case
}
```

Full call tree for `fib(5)`:

```
fib(5)
├─ fib(4)
│  ├─ fib(3)
│  │  ├─ fib(2)          ← call A of fib(2)
│  │  │  ├─ fib(1)  [base]
│  │  │  └─ fib(0)  [base]
│  │  └─ fib(1)  [base]
│  └─ fib(2)              ← call B of fib(2)
│     ├─ fib(1)  [base]
│     └─ fib(0)  [base]
└─ fib(3)
   ├─ fib(2)               ← call C of fib(2)
   │  ├─ fib(1)  [base]
   │  └─ fib(0)  [base]
   └─ fib(1)  [base]
```

Count every call: `fib(5)`×1, `fib(4)`×1, `fib(3)`×2, `fib(2)`×**3**, `fib(1)`×5, `fib(0)`×3. Total = 15 calls to compute a single `fib(5)`.

**`fib(2)` alone is recomputed 3 separate times** — each one redoing the exact same work, with no memory of having done it before. This is the concrete, hands-on evidence for why naive recursive Fibonacci is expensive: the number of calls grows roughly with the golden ratio to the power of n (tightly, `Θ(φⁿ)` where `φ ≈ 1.618`; loosely and more commonly stated, `O(2ⁿ)` — a valid but not tight upper bound, worth knowing which one you're actually citing).

**🔑 Key Takeaway — space is *not* exponential, even though time is.** At any instant, only one root-to-leaf path is actually sitting on the call stack — every sibling branch's frames are fully popped before the next branch is explored (this is depth-first execution). So while `fib(5)` makes 15 *calls* over its lifetime, the call stack never holds more than 5 frames *at once*. Space is O(n) (proportional to depth), time is O(φⁿ) (proportional to total calls). Conflating these two is a very common mistake — don't.

**🔗 Forward reference:** this exact redundancy — recomputing `fib(2)` from scratch every time it's needed — is precisely the problem memoization solves: cache each result the first time it's computed, and every later call becomes an O(1) lookup instead of a re-exploration of the whole subtree. Dynamic Programming, later in this plan, formalizes this. Nothing about memoization needs to be understood in depth yet — just recognize the shape of the problem it will solve, because you just watched it happen by hand.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals`. **Task:** `RecursionPractice` class — `factorial(int n)`, `fibonacci(int n)`, and a method that recursively sums a number's digits.

Practical guidance for each:
- **`factorial`:** base case `n <= 1 return 1`; recursive case `return n * factorial(n - 1)`. Comment which line is which, explicitly — the "definition of done" requires this named, not just implied by the code's shape.
- **`fibonacci`:** base cases for `n == 0` and `n == 1` (two base cases is completely normal — nothing requires exactly one). Recursive case `return fibonacci(n-1) + fibonacci(n-2)`.
- **Digit sum:** base case is the single-digit input (`n < 10 → return n`); recursive case peels off one digit at a time: `return n % 10 + digitSum(n / 10)`. Trace `digitSum(129)` by hand before running it — `129 → 9 + digitSum(12) → 9 + (2 + digitSum(1)) → 9 + 2 + 1 = 12`.

Definition of done (from the plan): each method's comment names its base case and recursive case explicitly, then push.

## Career Block Guide (1 hr)

- **LinkedIn engagement (20 min):** comment on 3–5 posts. A comment that adds a specific technical opinion or a genuine question outperforms generic agreement — "this matches what I've seen with X" reads as a real practitioner; "Great post!" doesn't.
- **Networking:** follow up on Day 6's connection requests. For anyone who accepted but hasn't replied to an opener, one short, low-pressure follow-up is appropriate (e.g., referencing something specific from their profile or a recent post); for anyone who hasn't accepted yet, leave it — a second request reads as pressure, not persistence.

---

## Day 8 — Interview Questions

**Q1. Why does `ArrayDeque` never come up as an alternative here, the way it might for a stack-based problem?** Fast-slow two pointers operate on a single array with direct indexed access — there's no need for push/pop semantics at either end; the pattern's entire value is that it needs no auxiliary structure at all.

**Q2. In LC 80, why compare against `nums[slow-2]` instead of maintaining a separate count variable?** Because `slow <= fast` always holds, the output array itself already encodes everything you need — `nums[slow-2]` and `nums[slow-1]` are, respectively, the last two values your own algorithm committed to the output. A counter is redundant state tracking something the array already tells you.

**Q3. What is a stack frame, precisely?** A block of memory pushed onto the call stack on every method invocation, holding that call's parameters, local variables, and a return address; popped when the call returns.

**Q4. Why is `StackOverflowError` an `Error` and not a `RuntimeException`?** Different branch of the `Throwable` hierarchy entirely (`Error → Throwable`, `RuntimeException → Exception → Throwable`) — `Error` signals a condition the program generally shouldn't try to catch and recover from, which matches the philosophy that unbounded recursion is a bug, not a runtime condition to handle gracefully.

**Q5. Give an example of a base case that's technically present but still causes infinite recursion.** A base case that's syntactically there but never reachable — e.g., base case `n == 0` while the recursive case calls `f(n + 1)` instead of `f(n - 1)`, moving away from the base case instead of toward it.

**Q6. Why is naive recursive Fibonacci's time complexity exponential but its space complexity only linear?** Time is proportional to the *total number of calls* across the whole tree (exponential — roughly `Θ(φⁿ)`), but space is proportional to the *maximum stack depth at any single instant*, which is just `n`, since depth-first execution pops each branch before exploring the next.

**Q7. Does Java optimize tail-recursive methods into loops?** No. Unlike some functional languages, the JVM performs no tail-call optimization; every recursive call, tail position or not, consumes a stack frame.

**Q8. What's the actual argument for why a recursive method is correct, beyond "I tested it"?** Induction: show the base case is correct, then show that *assuming* the recursive call correctly solves the smaller subproblem, the current layer correctly builds the right answer from that. Both parts verified once implies correctness for every input that reaches the base case.

**Q9. In Move Zeroes, why swap instead of overwrite?** The zeros that get displaced still need to end up somewhere in the array (at the back) — overwriting would silently destroy information that Remove Duplicates doesn't have to preserve, since Remove Duplicates' "extra" elements past the new length are explicitly allowed to hold anything.

---

## Daily Deliverable Check

- [ ] Remove Duplicates (recap) and Move Zeroes (recap) confirmed solid from Week 1.
- [ ] Remove Duplicates from Sorted Array II (LC 80) solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain, out loud and unprompted, why unbounded recursion crashes and what a stack frame is.
- [ ] Can trace `fib(5)`'s call tree by hand and state how many times `fib(2)` gets recomputed.
- [ ] `RecursionPractice` pushed, with explicit base-case/recursive-case comments on all three methods.

---

## What Tomorrow Assumes You Already Know Cold

Day 9 formalizes the JVM's stack/heap split and revisits pass-by-value at the *mechanism* level — it assumes today's stack frame model (parameters and locals live in the frame, frames are pushed/popped per call) is completely solid, since that's precisely the "stack" half of tomorrow's "stack vs. heap" story. It also assumes the fast-slow two-pointer skeleton is now reflexive enough that tomorrow's opposite-ends recap (LC 977, LC 167) needs no re-derivation, just re-recognition.
