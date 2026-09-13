# Week 2 (Revised): Two Pointers Completes (16 Problems), Sliding Window Begins

**What changed:** Two Pointers now closes at 16 problems instead of 11 (4Sum and Boats to Save Most People are new additions, alongside the 3 you already picked up in Week 1). Every theory topic from the original Week 2 is preserved in full — just re-sequenced by a day or two to fit around the extra practice.

---

## Day 8 — Two Pointers Continues, and Recursion Deep Dive

### DSA Block (2.5 hrs)
- Problem 6: Remove Duplicates from Sorted Array — LeetCode #26 — Easy — Pattern: Two Pointers (fast-slow)
  - Hint: `slow` marks where the next unique element goes; `fast` scans ahead looking for it.
  - Complexity: Time O(n) | Space O(1)
- Problem 7: Move Zeroes — LeetCode #283 — Easy — Pattern: Two Pointers (fast-slow)
  - Hint: `slow` tracks the next non-zero slot; swap whenever `fast` finds a non-zero.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Recursion, Properly
- Every recursive method needs a base case and a recursive case. Each call gets its own stack frame — this is why deep recursion can throw `StackOverflowError`, and it's the mechanism behind every tree/graph traversal and backtracking solution later in this plan.
- Coding exercise: implement factorial and Fibonacci recursively. Trace Fibonacci(5)'s call tree by hand and count how many times `fib(2)` gets recomputed — your first hands-on look at why memoization (coming with Dynamic Programming) matters.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `RecursionPractice` class — factorial, Fibonacci, and a method that recursively sums a number's digits.
- Definition of done: each method has a comment naming its base case and recursive case explicitly; pushed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: follow up on Day 6's connection requests.

### Daily Deliverable
- [ ] Remove Duplicates and Move Zeroes solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain, out loud, why unbounded recursion crashes and what a stack frame is.
- [ ] `RecursionPractice` pushed.

---

## Day 9 — Two Pointers: Sorted-Array Variants, and the JVM Memory Model

### DSA Block (2.5 hrs)
- Problem 8: Squares of a Sorted Array — LeetCode #977 — Easy — Pattern: Two Pointers (opposite ends)
  - Hint: the largest squares come from the extremes — compare `|left|` vs `|right|` and fill the result array from the back.
  - Complexity: Time O(n) | Space O(1) excl. output
- Problem 9: Two Sum II (Input Array Is Sorted) — LeetCode #167 — Medium — Pattern: Two Pointers (opposite ends)
  - Hint: start pointers at both ends; move `left` up if the sum is too small, `right` down if too large.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: JVM Memory Model — Stack vs. Heap
- Local variables and method call frames live on the stack; objects live on the heap, and variables referencing them just hold an address. Java is strictly pass-by-value — for objects, the "value" being passed is a reference, which is why a method can mutate a passed object's fields but reassigning the parameter itself doesn't affect the caller. There's no way to make a Java method reassign the caller's original variable — the language simply doesn't offer that capability, which is worth knowing precisely rather than assuming otherwise.
- Coding exercise: write a method that takes an `int` and tries to modify it (no effect on the caller), then a method that takes an object and mutates a field (caller *does* see the change) — side by side.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `PassByValueDemo` class implementing the exercise above, with inline comments on what's on the stack vs. the heap at each line.
- Definition of done: pushed, and you can explain it without notes.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 3 Engineering Managers at target companies.

### Daily Deliverable
- [ ] Squares of a Sorted Array and Two Sum II solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain pass-by-value semantics for primitives and objects, precisely.
- [ ] `PassByValueDemo` pushed.

---

## Day 10 — Two Pointers on Triplets, and the Integer Cache Trap

### DSA Block (2.5 hrs)
- Problem 10: 3Sum — LeetCode #15 — Medium — Pattern: Sorting + Two Pointers
  - Hint: sort first, fix one element, then use two pointers on the remainder to find pairs summing to its negation. Skip duplicate fixed elements.
  - Complexity: Time O(n²) | Space O(1) excl. output
- Problem 11: 3Sum Closest — LeetCode #16 — Medium — Pattern: Sorting + Two Pointers
  - Hint: same skeleton as 3Sum, but track the minimum absolute difference between the current sum and the target.
  - Complexity: Time O(n²) | Space O(1)

### Theory Block (2 hrs)
- Topic: Primitives, Precisely
- Sizes and ranges of `byte`/`short`/`int`/`long`/`float`/`double`; overflow wraps silently, no exception. The Integer cache: `Integer.valueOf(-128)` through `127` returns cached instances, so `==` happens to work below 128 and silently breaks above it — a subtle gotcha rooted entirely in how Java's autoboxing was implemented, worth knowing cold rather than discovering it live.
- Coding exercise: deliberately overflow an `int`; separately, prove the cache trap comparing `Integer.valueOf(100) == Integer.valueOf(100)` (true) against `Integer.valueOf(200) == Integer.valueOf(200)` (false).

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `TypesAndCache` class demonstrating both, heavily commented on why each happens.
- Definition of done: pushed.

### Career Block (1 hr)
- LinkedIn: Post 4 — the Integer cache trap.
- Networking: send connection requests to the 3 EMs identified yesterday.

### Daily Deliverable
- [ ] 3Sum and 3Sum Closest solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain the Integer cache trap from memory.
- [ ] `TypesAndCache` pushed. LinkedIn Post 4 published.

---

## Day 11 — Two Pointers: Extending to Four, and String Internals

### DSA Block (2.5 hrs)
- Problem 12: 4Sum — LeetCode #18 — Medium — Pattern: Sorting + Two Pointers **(new)**
  - Hint: fix the first two elements with nested loops, then run the same two-pointer sweep 3Sum used on the remainder. Skip duplicates at every level, not just the outer one.
  - Complexity: Time O(n³) | Space O(1) excl. output
- Problem 13: Boats to Save Most People — LeetCode #881 — Medium — Pattern: Two Pointers (greedy pairing) **(new)**
  - Hint: sort by weight; always try to pair the lightest person with the heaviest. If they fit together, both go; if not, the heaviest goes alone.
  - Complexity: Time O(n log n) | Space O(1)

### Theory Block (2 hrs)
- Topic: String Internals
- Strings are immutable — every "modification" creates a new object. The String pool caches literals for reuse. This is why `+=` concatenation in a loop is quietly O(n²), and why `StringBuilder` (which mutates an internal buffer) is the fix.
- Coding exercise: concatenate a string 10,000 times with `+=` and time it; do the same with `StringBuilder.append()`. Compare.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `StringPerformance` benchmark class, with the explanation in the class Javadoc.
- Definition of done: pushed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: reach out to 1 peer or former coworker to stay warm on the relationship.

### Daily Deliverable
- [ ] 4Sum and Boats to Save Most People solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain why naive String concatenation in a loop is O(n²).
- [ ] `StringPerformance` benchmark pushed.

---

## Day 12 — Two Pointers Near-Capstone, and Collections Internals

### DSA Block (2.5 hrs)
- Problem 14: Container With Most Water — LeetCode #11 — Medium — Pattern: Two Pointers (opposite ends)
  - Hint: area is capped by the shorter line — always move the pointer at the shorter line inward.
  - Complexity: Time O(n) | Space O(1)
- Problem 15: Sort Colors — LeetCode #75 — Medium — Pattern: Two Pointers (Dutch National Flag)
  - Hint: three pointers — `low`, `mid`, `high` — partition into 0s, 1s, and 2s in one pass.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Collections Internals — `ArrayList`, `LinkedList`, `ArrayDeque`
- `ArrayList` is a resizable array — O(1) random access, O(n) front-insertion (everything shifts). `LinkedList` gives O(1) front insertion but pays with pointer-chasing overhead and poor cache locality. `ArrayDeque` uses a circular array, giving O(1) amortized insertion/removal at *both* ends without `LinkedList`'s overhead — why it's the modern default for stacks and queues.
- Coding exercise: benchmark inserting 100,000 elements at index 0 of an `ArrayList` vs. an `ArrayDeque`.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `CollectionsBenchmark` class running the exercise above, with comments on *why* the difference exists.
- Definition of done: pushed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 5 Backend Engineers at Tier B companies.

### Daily Deliverable
- [ ] Container With Most Water and Sort Colors solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain precisely why `ArrayDeque` beats `LinkedList` for stack/queue use.
- [ ] `CollectionsBenchmark` pushed.

---

## Day 13 — Two Pointers Capstone (Hard Tier), and Pattern Consolidation

### DSA Block (2.5 hrs)
- Problem 16: Trapping Rain Water — LeetCode #42 — Hard — Pattern: Two Pointers (opposite ends)
  - Hint: track `leftMax`/`rightMax` as pointers close inward; water trapped at any position is bounded by the smaller of the two maxes.
  - Complexity: Time O(n) | Space O(1)

**This closes Two Pointers: 16 problems, Easy through Hard — up from 11 in the original plan.**

### Theory Block (1.5 hrs)
- Topic: Two Pointers, Reviewed — Opposite-Ends vs. Fast-Slow, Side by Side
- Before moving on, write down (in your own words, no looking back at hints) which of today's 16 problems used opposite-ends pointers and which used fast-slow, and *why* each problem's structure called for that variant. This is the single highest-leverage 20 minutes you'll spend this week — pattern recognition comes from articulating the "why," not just from having solved the problem once.
- Coding exercise: none — the written reflection above is the exercise.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: none new — use this slot to make sure all 16 Two Pointers solutions are actually pushed to `dsa-java/two-pointers/` with clear file/folder names, since you'll want to find these fast during interview-prep review months from now.
- Definition of done: `dsa-java/two-pointers/` contains all 16, browsable at a glance.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: reach out to 1 peer or former coworker to stay warm on the relationship.

### Daily Deliverable
- [ ] Trapping Rain Water solved — Two Pointers ladder complete at 16 problems.
- [ ] Written opposite-ends vs. fast-slow reflection complete.
- [ ] `dsa-java/two-pointers/` fully organized.

---

## Day 14 (Sunday) — Consolidation, and Sliding Window Begins

### Self-Check (15 min)
- [ ] Can you solve a Two Pointers problem cold, without hints, right now? Pick one from earlier this week and try before continuing.

### DSA Block (2 hrs)

**Concept Card — Sliding Window**
- What: a window (a contiguous subarray/substring) that expands and contracts over the data instead of re-scanning from scratch for every starting point.
- Why: turns an O(n²) or O(n³) "check every subarray" brute force into O(n), since each element enters and leaves the window at most once.
- Where: streaming/rate-limiting logic, network buffer analysis, any "best/longest/shortest contiguous X" problem.
- Interview signal: "contiguous subarray/substring," "longest/shortest window satisfying condition," "at most K distinct."
- Prerequisites: arrays/strings ✅, HashMap/HashSet ✅ (both from Week 1).

- Problem 1: Best Time to Buy and Sell Stock — LeetCode #121 — Easy — Pattern: Sliding Window (fixed-start tracking)
  - Hint: track the minimum price seen so far; at each day, check if selling today beats your best profit.
  - Complexity: Time O(n) | Space O(1)
- Problem 2: Maximum Average Subarray I — LeetCode #643 — Easy — Pattern: Sliding Window (fixed size)
  - Hint: compute the sum of the first `k` elements, then slide by subtracting the outgoing element and adding the incoming one.
  - Complexity: Time O(n) | Space O(1)

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** 14 days in, **25 total DSA problems solved** — 7 HashMap/HashSet (Week 1) + 16 Two Pointers, now fully closed (5 done Week 1 + 11 done this week) + 2 Sliding Window problems today. That matches the original plan's count at this same Day-14 checkpoint (also 25) — the difference isn't speed, it's depth: Two Pointers closed at 16 problems instead of 11, and Sliding Window opens with its expanded 14-problem set still ahead, instead of the original's 10. Same pace, more pattern underneath it.

### Daily Deliverable
- [ ] Best Time to Buy/Sell Stock and Maximum Average Subarray I solved, pushed to `dsa-java/sliding-window/`.
- [ ] Weekly ritual and scorecard complete.
