# Week 1 — Consolidated Interview Question Bank

**Series:** Week 1 SDE-2 Prep · [← Curriculum Map](./00_Curriculum_Map.md)

This document consolidates every interview question from every day's resource book into a single review sheet. Use it the way you'd use a spaced-repetition deck: work through it fresh a few days after finishing the week, not immediately after — testing yourself once material isn't fresh anymore is what actually reveals what you've genuinely retained versus what you could only answer with the explanation still in front of you.

Each section here is intentionally more concise than the full day book — the goal is fast self-testing, not re-teaching. If an answer doesn't fully click, that's your signal to go back to the relevant Resource Book section (linked at the top of each day's block below) rather than trying to memorize the short version here.

**Status:** Complete — all 7 days consolidated below.

---

## Day 1 — Programming Foundations, Control Flow, Methods

*Full explanations: [Day 1 Resource Book](./Day1_Resource_Book.md)*

1. **What's the difference between compilation and interpretation, and how does Java use both?**
   Compilation translates source ahead of execution (`javac`: `.java` → `.class` bytecode). Interpretation executes instructions one at a time at runtime. Java does both — compiles once to bytecode, then the JVM interprets (and JIT-compiles hot paths of) that bytecode at runtime.

2. **What does "write once, run anywhere" mean, and what makes it possible?**
   Compiled bytecode is platform-independent; the JVM is platform-specific. Each OS has its own JVM that translates the same bytecode into instructions for that machine, so one compiled file runs unmodified everywhere a JVM exists.

3. **Difference between JDK, JRE, and JVM?**
   JVM executes bytecode. JRE = JVM + standard libraries needed to run programs. JDK = JRE + development tools (including the compiler) — needed to write and compile Java, not just run it.

4. **Why must a `.java` file's name match its public class name exactly?**
   Hard compiler requirement in Java, not convention — mismatched names fail to compile.

5. **What determines how much memory a variable uses?**
   Its declared type — each primitive has a fixed, compile-time-known size (`int` = 4 bytes, `double` = 8 bytes, etc.).

6. **What does `7 / 2` evaluate to in Java, and how do you get a decimal result?**
   `3` — integer division truncates. Use `7 / 2.0` or cast one operand to `double` to get `3.5`.

7. **What is `-7 % 3` in Java?**
   `-1` — Java's `%` takes the sign of the dividend, not a strictly non-negative mathematical modulo.

8. **Difference between `=` and `==`; what bug does confusing them cause?**
   `=` assigns, `==` compares. Using `=` inside a condition silently reassigns instead of comparing — Java's requirement that `if` conditions be `boolean` catches many but not all instances.

9. **What is short-circuit evaluation, and why does `arr != null && arr.length > 0` rely on it?**
   `&&`/`||` stop evaluating once the result is determined by the left side alone. In the example, if `arr` is `null`, `arr.length` is never evaluated, avoiding a crash. Order matters.

10. **`for` vs. `while` — when do you choose each?**
    `for` when the iteration count/bookkeeping is known up front. `while` when the number of iterations depends on a runtime condition not naturally expressed as a counter.

11. **`while` vs. `do-while`?**
    `while` checks its condition before the first run (may execute zero times). `do-while` checks after (always runs at least once).

12. **What happens without `break` in a classic `switch`?**
    Execution falls through into subsequent cases regardless of their labels, until a `break` or the end of the `switch`. The modern arrow-syntax `switch` has no fall-through.

13. **What is method overloading, and when is the overload resolved?**
    Same method name, different parameter lists. Resolved at **compile time** based on argument types/count (static binding) — contrast with overriding, resolved at runtime (dynamic binding).

14. **What does "pass by value" mean for a primitive parameter?**
    The method receives a copy of the value in its own local parameter. Reassigning the parameter inside the method never affects the caller's original variable.

15. **[Code reading]** `int x = 5; int y = x; x = 10;` — what is `y`?
    `5` — `y` received a copy of `x`'s value at assignment time; later changes to `x` don't propagate.

16. **[Debug]** A FizzBuzz that checks `%3`, then `%5`, then `%15` in that order — what breaks?
    `%15` is unreachable — every multiple of 15 is also a multiple of 3, so the `%3` branch fires first and `"FizzBuzz"` never prints. Fix: check the most specific condition (`%15`) first.

17. **Why does checking divisors only up to `√n` correctly determine primality?**
    If `n = a × b` with `a ≤ b`, then `a ≤ √n` (since `a × a ≤ a × b = n`). Any composite number is guaranteed a factor at or below its square root, so finding none by `√n` proves primality. O(√n) vs. O(n) brute force.

18. **Why `i * i <= n` instead of `i <= Math.sqrt(n)`?**
    Avoids floating-point call overhead and potential precision-related off-by-one errors at the boundary; integer multiplication is cheaper than a square root computation.


---

## Day 2 — Arrays, Strings, Object-Oriented Programming

*Full explanations: [Day 2 Resource Book](./Day2_Resource_Book.md)*

1. **Why is `arr[i]` a constant-time operation?**
   Contiguous memory layout lets the address of any element be directly calculated (`base + i × element_size`) rather than searched for — same small amount of work regardless of size or index.

2. **What trade-off comes with that same fast-access property?**
   Fixed size — no guaranteed room to expand into, which is exactly why `ArrayList` exists.

3. **When do you choose classic `for` over for-each for array iteration?**
   Whenever you need the index itself — comparing neighbors, writing to a different index than you read, iterating backward, or walking two arrays in lockstep.

4. **Difference between `array.length` and `string.length()`?**
   `array.length` is a field, no parentheses. `string.length()` is a method call. Mixing them up is a compile error.

5. **Why does `"a" == "a"` often evaluate `true` for Strings, despite `==` being unreliable for content comparison?**
   The String pool reuses identical literals as the same object, so both variables reference the same pooled object. This is exactly why the bug is dangerous — it works by coincidence for literals and fails for non-pooled Strings (concatenation, user input, etc.).

6. **What does `new String("hello")` do differently from a literal?**
   Forces a brand-new object outside the pool, so `==` against a pooled equivalent is `false` even though `.equals()` is `true`.

7. **Why is String immutable, and what's the benefit?**
   Every "modifying" operation returns a new String; the original never changes. This makes Strings safe to share across code and safe as HashMap keys.

8. **What's the hidden cost of `result += x` in a loop, and the fix?**
   Each `+=` copies everything accumulated so far (immutability), giving roughly quadratic total work across n iterations. `StringBuilder` mutates one buffer in place for linear total work.

9. **Class vs. object?**
   Class = blueprint (fields/methods description). Object = a specific instance built from that blueprint with real values.

10. **What does a constructor do? What breaks if you add a parameterized one without a no-arg one?**
    Puts a new object into a valid starting state. Defining any constructor removes Java's free default no-arg constructor — `new Book()` would fail to compile without an explicit no-arg version.

11. **Why `this.title = title;` instead of `title = title;`?**
    The parameter shadows the field by name; a bare `title` resolves to the parameter. `this.title` explicitly targets the field. Without it, the field silently keeps its default value.

12. **What does encapsulation protect against, concretely?**
    Prevents external code from putting an object into an invalid state. Example: `Book.isAvailable` only changes through `checkOut()`/`returnBook()`, guaranteeing the "no double checkout" rule — impossible to guarantee with a public field.

13. **`private` vs. `protected` vs. `public`?**
    `private`: same class only. `protected`: same class + package + subclasses elsewhere. `public`: everywhere.

14. **What does `super(...)` do, and why must it come first?**
    Calls the parent constructor to initialize inherited state before the subclass adds its own. Omitting it inserts an implicit no-arg `super()`, which fails to compile if the parent lacks a no-arg constructor.

15. **Overloading vs. overriding?**
    Overloading: same name, different parameters, same class — resolved at compile time (static binding). Overriding: same signature, redefined in a subclass — resolved at runtime based on actual object type (dynamic binding); the basis of polymorphism.

16. **Why can't you instantiate an interface directly?**
    It declares method signatures with no implementation — nothing to actually run. Only implementing concrete classes can be instantiated.

17. **[Code reading]** `Shape[] shapes = {new Circle(5), new Rectangle(3,4)};` then `s.area()` in a loop — what determines which implementation runs?
    Dynamic binding: Java dispatches to whichever concrete class's `area()` belongs to the actual object at runtime, regardless of the array's declared `Shape` type.

18. **Pass-by-value for objects: mutating through a reference vs. reassigning the reference — what does the caller see?**
    Mutating through the reference (`arr[0] = 999`) is visible to the caller (shared object). Reassigning the parameter itself is never visible (only changes the local copy of the reference) — Java is always pass-by-value; for objects, the value passed is the reference itself.

19. **[Debug]** `if (userInput == "quit")` where `userInput` came from user input — what's wrong?
    `==` compares references; `userInput` isn't the pooled literal even if content matches, so this can silently fail. Fix: `"quit".equals(userInput)` (null-safe, since it's called on the known-non-null literal).


---

## Day 3 — Big-O Notation, ArrayList

*Full explanations: [Day 3 Resource Book](./Day3_Resource_Book.md)*

1. **What does Big-O actually measure?**
   The shape of how work grows as input grows — ignoring constants, hardware, and language — not an exact operation count.

2. **Why does O(2n) simplify to O(n)?**
   Both grow at the same rate as input doubles; Big-O captures growth trend, not exact count.

3. **Why does O(n² + n) simplify to O(n²)?**
   The n² term dwarfs n at scale (n=1000 → 1,000,000 vs. 1,000) — the lower-order term becomes negligible.

4. **Why should two differently-sized inputs get different variables (e.g., O(m+n)) instead of both being "n"?**
   Collapsing them can hide real scaling differences between the two inputs.

5. **Sequential loops vs. nested loops — complexity difference?**
   Sequential: add (O(n)+O(n)=O(n)). Nested: multiply (O(n)×O(n)=O(n²)). Check whether the second loop is inside or after the first.

6. **Complexity of a loop that halves its value each iteration?**
   O(log n) — "how many times can you halve n before reaching 1" is what a base-2 logarithm answers.

7. **Best/worst/average case — which does an interview default to?**
   Worst case, unless stated otherwise. State it explicitly when giving a Big-O answer.

8. **Why "amortized O(1)" for `ArrayList.add()` rather than plain O(1)?**
   Most calls are O(1), but resizes are O(n). Amortized O(1) is the precise claim that total cost across n appends is O(n), so average cost per call is O(1) — a guarantee about the whole sequence.

9. **Why is total resize-copying work O(n) across n appends, not more?**
   Doubling means resizes happen at sizes 1,2,4,...,n — a geometric series summing to under 2n regardless of n.

10. **Why does `ArrayList` double capacity instead of growing by a fixed amount?**
    Fixed growth keeps resize frequency constant while each resize copies more data as the list grows, giving O(n²) total copying and O(n) amortized cost. Doubling makes resizes exponentially rarer, keeping total copying at O(n).

11. **What is autoboxing, and why does `ArrayList<Integer>` need it?**
    Generics only accept object types, not primitives. Wrapper classes (`Integer`, etc.) are objects holding a primitive value; autoboxing converts automatically between the two.

12. **Performance cost of autoboxing?**
    Each boxed value needs real heap allocation, unlike a raw primitive — negligible at small scale, real at large scale.

13. **Difference between `list.remove(1)` and `list.remove(Integer.valueOf(1))`?**
    `remove(int)` removes by index; `remove(Object)` removes by value. A literal `int` argument always resolves to the index overload.

14. **Why is `Integer a=100; Integer b=100; a==b` true, but `Integer c=200; Integer d=200; c==d` false?**
    Integer caching covers -128 to 127; cached values share a reference (`==` true), values outside the range are freshly allocated (`==` false). Always use `.equals()` for wrapper types.

15. **Is `ArrayList.get(i)` still O(1)?**
    Yes — it's backed by a real array and delegates directly to indexed access; only `add()`/`remove()` touch the resizing machinery.


---

## Day 4 — HashSet, HashMap, Stack, Queue

*Full explanations: [Day 4 Resource Book](./Day4_Resource_Book.md)*

1. **Why does `HashSet.contains()` average O(1) vs. O(n) for `ArrayList`?**
   Hashing computes a bucket directly (calculation) instead of scanning every element (search).

2. **Mechanically, what happens inside `HashMap.put(key, value)`?**
   Hash function → deterministic hash code → bucket index (e.g. `hashCode % numBuckets`) → stored there. `get()` recomputes the same hash to find the same bucket, then uses `.equals()` within it.

3. **What is a hash collision and how is it handled?**
   Two different keys hashing to the same bucket — unavoidable in general (pigeonhole principle). Handled by letting a bucket hold multiple entries, disambiguated via `.equals()`.

4. **State the equals()/hashCode() contract.**
   Equal objects (per `.equals()`) must produce equal `hashCode()`. Unequal objects may share a hash code (an ordinary collision).

5. **What breaks if a class overrides `.equals()` but not `hashCode()`?**
   "Equal" objects can land in different buckets — a `HashMap` may silently fail to find a key that's genuinely already present, returning `null` instead of the expected value.

6. **What does `map.get(missingKey)` return, and what's the danger?**
   `null` — assigning it directly to a primitive triggers a `NullPointerException` via failed auto-unboxing. Use `getOrDefault`.

7. **What does the `getOrDefault(key, 0) + 1` idiom do?**
   Handles first-occurrence (returns 0 → stores 1) and increment (returns current count → stores count+1) in one line, no explicit branching.

8. **Relationship between HashSet and HashMap internally?**
   `HashSet` wraps a `HashMap`, storing elements as keys and ignoring values.

9. **Stack vs. Queue?**
   Stack: LIFO (undo, bracket matching). Queue: FIFO (arrival-order processing, level-order traversal).

10. **Why `ArrayDeque` over legacy `Stack`?**
    `Stack extends Vector`, which is synchronized (unneeded locking overhead) and exposes Vector's full API, undermining the LIFO restriction.

11. **Why `ArrayDeque` over `LinkedList`?**
    `LinkedList` needs per-element node allocation and pointer bookkeeping with poor cache locality; `ArrayDeque` is array-backed with `ArrayList`-style doubling.

12. **What signals "use a stack" in a problem statement?**
    Matching/undoing/referencing the most recently seen or opened item — a LIFO relationship.

13. **Why is Valid Parentheses naturally a stack problem?**
    Each closer must match the most recently opened, still-unclosed bracket — a LIFO relationship, checked in one linear pass via push-on-open, pop-on-close.

14. **Two classic bugs in a stack-based Valid Parentheses solution?**
    Missing `isEmpty()` check before popping, and missing the final `isEmpty()` check after the loop (unmatched openers left on the stack).

15. **Why can't one stack alone produce FIFO order, and why do two stacks work?**
    One stack reverses order once (still LIFO). Moving elements to a second stack reverses again, restoring original (FIFO) order for the transferred batch.

16. **Amortized complexity of two-stack queue operations?**
    O(1) amortized — each element is transferred between the two stacks at most once across its lifetime, bounding total transfer work at O(n) over any n operations.

17. **Why can cancellations cascade in Remove Adjacent Duplicates, and why does that need a stack?**
    Removing one pair can expose a new adjacent pair. A stack keeps the prior character available and correctly ordered to catch this immediately.

18. **Why does `StringBuilder`-as-stack avoid a final reversal, unlike `Deque<Character>`?**
    `deleteCharAt(length-1)` only removes from the end, preserving left-to-right order; popping a `Deque` yields most-recent-first, the opposite order.

19. **HashSet or HashMap: need "have I seen this" plus one more piece of info (e.g., first index seen)?**
    `HashMap` — `HashSet` only answers membership; associating extra info requires a Map.


---

## Day 5 — HashMap/HashSet Pattern (12 Problems), OOP Four Pillars

*Full explanations: [Day 5 Resource Book](./Day5_Resource_Book.md)*

1. **Core reframe that makes Two Sum O(n)?**
   Ask "have I seen the complement?" per element (O(1) lookup) instead of checking every pair.

2. **Why check the complement before inserting in Two Sum?**
   Prevents matching an element with itself; correctly handles duplicates.

3. **Three approaches to Contains Duplicate?**
   Brute force O(n²)/O(1); sort-then-scan O(n log n)/O(1) extra; HashSet O(n)/O(n).

4. **Why does Valid Anagram use `int[26]` instead of HashMap?**
   Fixed, small, known key space — cheaper than hashing/autoboxing overhead.

5. **Valid Anagram vs. Ransom Note — same mechanism, different check?**
   Anagram needs exact equality (all counts zero); Ransom Note needs coverage (no count goes negative).

6. **Why does Isomorphic Strings need two maps?**
   One map only checks the mapping is a function; a second map (or used-set) checks it's injective — no two source characters may map to the same target.

7. **What is a canonical form, and how does Group Anagrams use one?**
   A shared representative key for equivalent inputs (sorted chars, or a frequency signature) — group by that key in a HashMap.

8. **Why is a frequency-count key faster than a sorted-string key for Group Anagrams?**
   O(k) counting beats O(k log k) sorting per string.

9. **What's the "smart starting point" trick in Longest Consecutive Sequence, and why does it matter?**
   Only count from true run-starts (`num-1` not in set); without it, redundant overlapping counts degrade to O(n²) despite using a HashSet.

10. **Why doesn't the nested while-in-for in that solution make it O(n²)?**
    Every element is visited by some inner while-loop at most once, total, across the whole run — not once per outer iteration.

11. **Boyer-Moore Voting's core insight and precondition?**
    Cancel non-matching votes against the current candidate; the true majority (>n/2) can never be fully cancelled. Requires a majority to actually exist.

12. **Why two HashSets in Intersection of Two Arrays?**
    One for O(1) lookup against `nums1`; a second automatically deduplicates the result.

13. **Why does First Unique Character need two passes?**
    Uniqueness can't be known until the whole string is seen; pass one builds frequencies, pass two finds the first with count 1.

14. **Word Pattern: why `.equals()` on one line and `!=` on the next?**
    `.equals()` compares Strings by content; `!=` compares an auto-unboxed primitive `char` by value.

15. **What is a prefix sum, and how does it relate to subarray sums?**
    `prefixSum[i]` = sum before index i. Subarray sum(i,j) = `prefixSum[j+1] - prefixSum[i]`.

16. **Why must `prefixSumCounts.put(0,1)` be initialized before the loop in Subarray Sum Equals K?**
    Otherwise a subarray starting at index 0 summing to k finds no match (prefix sum 0 not yet recorded).

17. **Why must the lookup happen before the map update in that same problem?**
    Prevents a subarray from matching against its own just-inserted entry.

18. **Why can't Subarray Sum Equals K use a sliding window?**
    Negative numbers break the monotonic-growth assumption sliding window relies on.

19. **Name all four OOP pillars with an example each.**
    Encapsulation (`Book.isAvailable`), Inheritance (`Employee`/`Manager`), Polymorphism (`Shape[]` calling `.area()`), Abstraction (any interface contract).

20. **Interface vs. abstract class — when does each apply?**
    Interface: unrelated types sharing only a contract, multiple per class. Abstract class: related types sharing real state/behavior, differing in one piece of logic; single per class.

21. **Why can't `Account` be instantiated directly?**
    It's `abstract` with at least one no-body abstract method (`calculateInterest()`) — nothing to execute without a concrete override.

22. **Mechanically, how does an enum constant get its own method implementation?**
    Each constant with a body compiles to its own anonymous subclass overriding the abstract method — ordinary polymorphism via enum syntax.


---

## Day 6 — Two Pointers Begins, SOLID Principles

*Full explanations: [Day 6 Resource Book](./Day6_Resource_Book.md)*

1. **What structural property makes Two Pointers valid?**
   Some guarantee (usually sortedness) proving a pointer move can't skip a valid answer — it doesn't apply to arbitrary unsorted data.

2. **Three Two Pointers variants covered?**
   Opposite ends converging; from the back; same-direction fast/slow (read/write).

3. **Why is Valid Palindrome's nested while-in-while still O(n)?**
   Every character is examined once, total, across the run — not multiplied per outer iteration.

4. **Why does Reverse String operate on `char[]`, not `String`?**
   Strings are immutable — "reverse in place" is only achievable on a mutable structure.

5. **Why merge Merge Sorted Array from the back?**
   Merging from the front would overwrite not-yet-read data; the back's empty padding gives safe write room.

6. **Why `while (j >= 0)` alone, not `i>=0 && j>=0`, in Merge Sorted Array?**
   If nums1 finishes first, remaining nums2 elements must still be copied; if nums2 finishes first, nums1's leftovers are already correct.

7. **Two Sum II (sorted) vs. Day 5's Two Sum (unsorted) — trade-off?**
   Sortedness enables O(1)-space two pointers vs. O(n)-space HashMap for the unsorted version.

8. **Why does Squares of Sorted Array fill its result from the back?**
   Largest squares come from the array's ends; comparing both ends yields largest-to-smallest order naturally.

9. **How does Remove Duplicates' two-pointer shape differ from opposite-ends convergence?**
   Both pointers move the same direction at different speeds — fast scans, slow marks the write boundary.

10. **Why does Remove Duplicates need no HashSet?**
    Sorted arrays guarantee duplicates are adjacent, so comparing only to the last confirmed-unique value suffices.

11. **SRP, with an example?**
    One reason to change; separate `InterestCalculator` from `StatementFormatter`.

12. **OCP, and how does Day 5's `Account` already satisfy it?**
    Open for extension, closed for modification; a new account type is a new subclass, no changes to `Account` itself.

13. **The Square/Rectangle LSP violation, precisely?**
    `Square extends Rectangle` overriding `setWidth()` to also change height breaks code that expects independent width/height behavior from any `Rectangle`.

14. **ISP, with an example?**
    Don't force unused methods on clients; split a bloated `Worker` interface into `Workable`/`Eatable`.

15. **DIP — why does the original `NotificationService` violate it?**
    It directly instantiates concrete `EmailSender` instead of depending on an abstraction, blocking substitution and testability.

16. **What is constructor injection?**
    Supplying a dependency via the constructor rather than constructing it internally — avoids partially-initialized state.


---

## Day 7 — Two Pointers Continues, Week 1 Consolidation

*Full explanations: [Day 7 Resource Book](./Day7_Resource_Book.md)*

1. **Is Subsequence's Two Pointers variant vs. Remove Duplicates'?**
   One forward pointer each across two *different* structures, vs. two pointers within one structure.

2. **Recursive vs. iterative Is Subsequence — the trade-off?**
   Same time complexity; recursion costs O(s+t) space (call stack), iteration is O(1).

3. **Is Subsequence's follow-up for many queries against one `t`?**
   Preprocess `t` into a char→sorted-indices map once, then binary-search each query instead of rescanning.

4. **Why does Valid Palindrome II check only the first mismatch?**
   Everything before it is already correct; the one allowed deletion must resolve that first disagreement, or nothing can fix it with only one deletion.

5. **Why is skipping both characters on a mismatch wrong in Valid Palindrome II?**
   That uses two deletions, exceeding the allowed budget of one.

6. **Move Zeroes: why swap instead of overwrite (unlike Remove Duplicates)?**
   Every element must be preserved (zeroes relocated, not discarded); overwriting would lose values Remove Duplicates was allowed to discard.

7. **Container With Most Water: why is moving the shorter pointer always correct?**
   Moving the taller pointer is provably dominated — width shrinks while the limiting (shorter) height can't improve — so it can never yield a better area.

8. **How does 3Sum reduce to Two Sum II?**
   Fixing one element turns the rest into "find two numbers summing to `-nums[i]`" in the sorted remainder — exactly Two Sum II.

9. **Why does 3Sum sort first — both reasons?**
   Enables the two-pointer sub-search, and makes duplicate values adjacent for simple skip-based deduplication.

10. **Why does `if (nums[i] > 0) break;` safely end 3Sum early?**
    Sorted + positive means everything after is also positive; three non-negative numbers (one strictly positive) can never sum to zero.

11. **3Sum's time complexity, and why does O(n²) dominate the O(n log n) sort?**
    O(n²) overall — n outer iterations × O(n) inner scan; n² grows faster than n log n at scale.

12. **Remove Element vs. Remove Duplicates — precondition difference?**
    Remove Element's check is against a fixed value (works unsorted); Remove Duplicates compares to a neighbor (needs sortedness to guarantee adjacency).

13. **[Self-check]** One sentence each for ArrayList/HashSet/HashMap/ArrayDeque?
    Ordered+growable+indexed (ArrayList); O(1) uniqueness/membership (HashSet); O(1) key→extra-info lookup (HashMap); LIFO/FIFO from the ends (ArrayDeque).

14. **Name a problem this week combining two different patterns.**
    3Sum (HashMap-era "fix one, reduce" + Two Pointers) or Subarray Sum Equals K (Two Sum's complement-lookup shape + prefix sums).

