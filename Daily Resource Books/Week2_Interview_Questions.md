# Week 2 Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Covers:** Day 8 → Day 14 (`Week_02_Revised.md`)
**Companion:** `Week1_Interview_Questions.md` (Week 1's equivalent bank)

Every Q&A pair from every day's Resource Book, consolidated for spaced review. Organized by day; within each day, DSA questions come before theory questions, matching the order each topic was introduced. If any answer here doesn't fully make sense on its own, the corresponding day's Resource Book has the full derivation — this bank is for review and self-testing, not first-pass learning.

---

## Day 8 — Two Pointers (Recap + LC 80), Recursion

**Q1. Why does `ArrayDeque` never come up as an alternative here, the way it might for a stack-based problem?**
Fast-slow two pointers operate on a single array with direct indexed access — there's no need for push/pop semantics at either end; the pattern's entire value is that it needs no auxiliary structure at all.

**Q2. In LC 80, why compare against `nums[slow-2]` instead of maintaining a separate count variable?**
Because `slow <= fast` always holds, the output array itself already encodes everything you need — `nums[slow-2]` and `nums[slow-1]` are, respectively, the last two values your own algorithm committed to the output. A counter is redundant state tracking something the array already tells you.

**Q3. What is a stack frame, precisely?**
A block of memory pushed onto the call stack on every method invocation, holding that call's parameters, local variables, and a return address; popped when the call returns.

**Q4. Why is `StackOverflowError` an `Error` and not a `RuntimeException`?**
Different branch of the `Throwable` hierarchy entirely (`Error → Throwable`, `RuntimeException → Exception → Throwable`) — `Error` signals a condition the program generally shouldn't try to catch and recover from, which matches the philosophy that unbounded recursion is a bug, not a runtime condition to handle gracefully.

**Q5. Give an example of a base case that's technically present but still causes infinite recursion.**
A base case that's syntactically there but never reachable — e.g., base case `n == 0` while the recursive case calls `f(n + 1)` instead of `f(n - 1)`, moving away from the base case instead of toward it.

**Q6. Why is naive recursive Fibonacci's time complexity exponential but its space complexity only linear?**
Time is proportional to the total number of calls across the whole tree (exponential — roughly `Θ(φⁿ)`), but space is proportional to the maximum stack depth at any single instant, which is just `n`, since depth-first execution pops each branch before exploring the next.

**Q7. Does Java optimize tail-recursive methods into loops?**
No. Unlike some functional languages, the JVM performs no tail-call optimization; every recursive call, tail position or not, consumes a stack frame.

**Q8. What's the actual argument for why a recursive method is correct, beyond "I tested it"?**
Induction: show the base case is correct, then show that assuming the recursive call correctly solves the smaller subproblem, the current layer correctly builds the right answer from that. Both parts verified once implies correctness for every input that reaches the base case.

**Q9. In Move Zeroes, why swap instead of overwrite?**
The zeros that get displaced still need to end up somewhere in the array (at the back) — overwriting would silently destroy information that Remove Duplicates doesn't have to preserve, since Remove Duplicates' "extra" elements past the new length are explicitly allowed to hold anything.

---

## Day 9 — Two Pointers (Recap + LC 633), JVM Memory Model

**Q1. Is Java pass-by-reference for objects?**
No. Java is pass-by-value everywhere. For object-typed parameters, the value being copied is a reference (an address) — never the object itself.

**Q2. If Java is pass-by-value, how can a method change an object's state that the caller sees afterward?**
The copied reference still points at the same heap object as the caller's reference. Mutating a field through that reference mutates the one shared object — the value that changed hands was an address, and following that address reaches the real, shared data.

**Q3. Can a Java method reassign a variable that belongs to its caller?**
No — never, by design. Reassigning a parameter only changes what that parameter's own copy (in the callee's frame) points to; the caller's original variable, in the caller's own frame, is untouched.

**Q4. Where does an `int` local variable live? Where does a `new int[100]` live?**
The `int` value itself sits directly in the current stack frame. The array — being an object — lives on the heap; the local variable holding it is a reference in the stack frame pointing at that heap location.

**Q5. Why does an object outlive the method that created it, but a local `int` doesn't?**
The object lives on the heap, whose lifetime is governed by reachability, not by any one method call. The `int` lives in a stack frame, which is reclaimed the instant its method returns.

**Q6. In LC 633, what plays the role that "the array is sorted" played in LC 167?**
The algebraic monotonicity of `a² + b²` in each variable independently — increasing `a` strictly increases the sum, decreasing `b` strictly decreases it — the same property that makes greedily moving one pointer or the other provably correct.

**Q7. Why avoid `Math.sqrt()` inside LC 633's loop?**
It recomputes a value cheaply tracked incrementally instead, and inexact floating-point results risk unreliable equality comparisons against an integer target.

---

## Day 10 — Two Pointers (Recap + LC 16), Primitives & Integer Cache

**Q1. Walk through 3Sum Closest's approach and justify why the two-pointer sweep doesn't miss the true closest sum.**
Sort, fix an outer element, run opposite-ends two pointers on the rest, tracking the sum closest to target seen so far; the sweep is exhaustive over reachable pairs because pointer movement is monotonic in the needed direction at every step.

**Q2. Does 3Sum Closest need duplicate-skipping the way 3Sum does?**
No — 3Sum needs it to avoid emitting the same triplet twice in a results list; 3Sum Closest only tracks a single best number, so revisiting an equivalent combination costs redundant work, not incorrect output.

**Q3. Why does `int` overflow wrap instead of throwing?**
Two's complement arithmetic just keeps following ordinary binary addition rules past the range boundary; incrementing past the maximum flips the sign bit, producing the minimum value with no runtime check involved.

**Q4. What's the exact range of the Integer cache, and where does it come from?**
-128 to 127 inclusive, guaranteed by the JLS's autoboxing rules; `Integer.valueOf()` returns a shared cached instance in that range and allocates fresh otherwise.

**Q5. Why does `Integer.valueOf(50) == Integer.valueOf(50)` return `true` but `new Integer(50) == new Integer(50)` return `false`?**
`valueOf` checks the cache first for values in range and returns the same shared object; `new` always allocates a fresh object, bypassing the cache regardless of value.

**Q6. Is the Integer cache trap a new rule, or a specific case of something already established?**
A specific, sneakier case of `==` vs. `.equals()` — `==` always compares references for object types; the cache just makes references coincidentally equal for small values.

**Q7. Give a concrete way this trap causes a real bug.**
Code compares two boxed `Integer`s with `==` instead of `.equals()`, passes every test written with small sample values, then silently misbehaves the first time production data includes a value outside -128..127.

---

## Day 11 — Two Pointers (LC 18, LC 881), String Internals

**Q1. Walk through 4Sum's approach.**
Sort, two nested outer loops fixing two elements, inner opposite-ends two-pointer sweep on the remainder for an exact-sum pair, duplicate-skipping at all four levels to avoid emitting the same quadruplet twice.

**Q2. Why does 4Sum specifically need `long` for its running sums, when 3Sum didn't need to worry about this?**
LeetCode's constraints allow individual values up to `±10⁹`; summing four of them can approach `±4×10⁹`, which exceeds `int`'s roughly `±2.1×10⁹` range and can silently wrap, producing a wrong comparison with no error thrown.

**Q3. Justify Boats to Save Most People's greedy pairing rule.**
An exchange argument: the heaviest remaining person must go on some boat; if any pairing works for them, pairing with the lightest remaining person is never worse than any alternative, since the lightest person could have shared with strictly more potential partners than anyone else — any other valid pairing can be reshuffled into this one without increasing the total boat count.

**Q4. Are Boats to Save Most People and Assign Cookies the same two-pointer pattern?**
No, despite both being sorted-and-greedy: Boats converges two pointers from opposite ends with a pair-or-strand branch; Assign Cookies advances two pointers independently in the same direction, only one of them conditionally — structurally the "one-forward-pointer-each" shape.

**Q5. Why is naive `String` concatenation in a loop O(n²)?**
Strings are immutable, so each `+=` builds an entirely new object and copies every previously accumulated character into it; summed over n iterations, that's an arithmetic series, O(n²) total.

**Q6. Doesn't the compiler already rewrite `+` into `StringBuilder` calls — so why does the loop case still cost O(n²)?**
The compiler's rewrite is per-statement: each loop iteration gets its own fresh, throwaway `StringBuilder` that still has to copy in everything accumulated so far as its starting point. The optimization never spans multiple iterations.

**Q7. Why is `StringBuilder.append()` in a loop O(n) instead of O(n²)?**
One single mutable buffer is grown in place (doubling when full, the same amortized argument as `ArrayList`), rather than a fresh full-content copy being made on every append.

**Q8. What determines whether `==` returns `true` for two `String` variables holding the same text?**
Whether both references point at the same object — automatic for pooled literals, but not for `new String(...)`, regardless of whether the underlying text is identical; `.equals()` is the only reliable value check.

---

## Day 12 — Two Pointers (Recap + LC 75), Collections Internals

**Q1. Justify Container With Most Water's greedy pointer movement.**
Area is bounded by the shorter of the two current lines; keeping the shorter one fixed and moving the taller one inward can never produce a larger area than moving the shorter one would have, since the bottleneck is unchanged or worse either way.

**Q2. State Sort Colors' three-region invariant.**
`[0, low)` holds only 0s, `[low, mid)` holds only 1s, `[mid, high]` is unprocessed, `(high, n-1]` holds only 2s — maintained at every step of the single pass.

**Q3. Why is it safe to advance both `low` and `mid` together in the `0` branch, but not `mid` alone in the `2` branch?**
In the `0` branch, the value displaced from `low` is always either unprocessed or a known `1` — the invariant already accounts for it. In the `2` branch, the value swapped in from `high` is completely unexamined and must be re-checked.

**Q4. Is Sort Colors an opposite-ends or a fast-slow two-pointer problem?**
Neither, cleanly — it's a three-pointer partition (Dutch National Flag) that borrows elements of both, worth naming as its own category.

**Q5. Why is `get(i)` O(1) on `ArrayList` but O(n) on `LinkedList`?**
`ArrayList` computes a direct memory address from the index; `LinkedList` has no such formula — it must walk node-to-node from whichever end is closer.

**Q6. Why can `LinkedList` be slower in practice than `ArrayList`, even when Big-O bounds are equal or favor `LinkedList`?**
Cache locality — `ArrayList`'s contiguous backing array lets the CPU prefetch multiple elements per cache line; `LinkedList`'s nodes are scattered across the heap, causing a cache miss on nearly every node.

**Q7. How does `ArrayDeque` achieve O(1) insertion at the front without shifting elements?**
It's a circular array: the `head` index simply decrements (wrapping around the array's boundary via modular/bitmask arithmetic) rather than requiring every existing element to move over.

---

## Day 13 — Trapping Rain Water, Two Pointers Full Consolidation

**Q1. Walk through why Trapping Rain Water's two-pointer solution is correct without fully scanning the region between the pointers.**
Whenever `leftMax` is updated, the branch condition guarantees a strictly taller bar was observed on the right side at that same moment, at a position `left` never overtakes afterward — that observed bar is a permanent witness that the true right-side maximum stays at least as large as `leftMax` for as long as it's being used.

**Q2. Name three approaches to Trapping Rain Water, in order of optimization.**
Brute force (per-position rescanning, O(n²)/O(1)), prefix/suffix max arrays (precompute both directions, O(n)/O(n)), two pointers (running max per side, O(n)/O(1)).

**Q3. Is Merge Sorted Array opposite-ends or fast-slow?**
Neither — it's From-the-Back: filling the result starting from the last index specifically to avoid overwriting not-yet-merged elements.

**Q4. Why doesn't Sort Colors reduce to opposite-ends or fast-slow?**
It uses three pointers marking three regions rather than two pointers converging or scanning together.

**Q5. What's the actual difference between 3Sum's two-pointer mechanism and Boats to Save Most People's?**
Both converge from opposite ends, but 3Sum's movement is purely comparison-driven toward an exact or closest sum; Boats adds a genuine branch — pair, or strand the extreme one — a different decision rule on the same convergence shape.

**Q6. If a new, unfamiliar two-pointer problem doesn't obviously match any named variant, what's the right move?**
Don't force it into an existing box — identify the actual invariant the problem needs, and name it as its own thing if it genuinely is one.

---

## Day 14 — Sliding Window Begins (LC 121, LC 643)

**Q1. Why is a sliding window O(n) even though the code often has a loop nested inside another loop?**
The inner pointer's total movement across the entire run, not per outer iteration, is bounded by n — each element is added to the window's aggregate at most once and removed at most once overall.

**Q2. What's the actual mechanistic relationship between Sliding Window and the fast-slow Two Pointers variant?**
Both move two pointers only rightward, never backward. Fast-slow uses one pointer as a scanner and the other purely as a write-position marker; Sliding Window promotes both pointers to meaningful range boundaries with a running aggregate over everything between them.

**Q3. Is LC 121 (Best Time to Buy/Sell Stock) a true sliding window?**
Not in the strict sense — no explicit window boundary or aggregate-over-a-range is tracked, just a running minimum and an implicit left edge that can jump forward arbitrarily. It shares the "carry state forward, never rescan" idea without matching the textbook shape.

**Q4. What distinguishes a fixed-size from a variable-size window?**
Fixed-size: length is a given constant, both edges move in lockstep (LC 643). Variable-size: the window grows and shrinks based on whether a stated constraint currently holds — arriving in Week 3.

**Q5. In LC 643, why is updating the sum incrementally better than resumming each window?**
Consecutive windows share k-1 elements; incremental update touches only the two elements that actually changed, turning O(k) per-window work into O(1) and the overall complexity from O(n·k) to O(n).

**Q6. Why wasn't any extra Sliding Window practice added this week?**
The pattern is only opening — Week 3 immediately continues it with 12 more required problems spanning every major variant, so extra practice now risks duplicating problems Week 3 was always going to require.

---

## Index by Topic (Quick Reference)

- **Two Pointers — variant classification:** Day 13, Q3–Q6 (also see the full 16-problem answer key in `Day13_Resource_Book.md`).
- **Two Pointers — proof techniques:** Day 11 Q3 (exchange argument), Day 12 Q2–Q3 (invariant), Day 13 Q1 (witness argument).
- **Recursion:** Day 8, all.
- **JVM Memory Model / pass-by-value:** Day 9, Q1–Q5.
- **Primitives / overflow / Integer cache:** Day 10, Q3–Q7.
- **String Internals:** Day 11, Q5–Q8.
- **Collections Internals:** Day 12, Q5–Q7.
- **Sliding Window mechanism:** Day 14, all.
