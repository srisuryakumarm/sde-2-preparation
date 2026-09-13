# Week 15 — Interview Question Bank

**Series:** SDE-2 Interview Prep · Week 15 Consolidated Review
**Curriculum Map:** [00_Curriculum_Map.md](./00_Curriculum_Map.md)
**Covers:** [Day 99](./Day99_Resource_Book.md) · [Day 100](./Day100_Resource_Book.md) · [Day 101](./Day101_Resource_Book.md) · [Day 102](./Day102_Resource_Book.md) · [Day 103](./Day103_Resource_Book.md) · [Day 104](./Day104_Resource_Book.md) · [Day 105](./Day105_Resource_Book.md)

Every question from every day's Resource Book this week, in one place, in the order it was originally taught. 76 questions total: Bit Manipulation's and Tries' closing problems, Kubernetes fundamentals and HPA, Segment Trees (open and close), all three Creational design patterns, sorting algorithms from scratch, Quickselect, and the full two-day SQL practice track (subqueries through window functions). Use this for spaced review — the individual day books remain the place to go for full derivations, worked traces, and code; this bank is for fast recall checks.

---

## Day 99 — Bit Manipulation Closes, Tries Closes, Kubernetes & HPA

**Q1. Why does Reverse Bits require `>>>` instead of `>>`, specifically?**

*Answer:* The two operators diverge only when the operand is negative (Week 14, Day 95). `>>` sign-extends — it fills newly-vacated high bits with copies of the *original* sign bit, which corrupts the bit pattern being read if `n`'s sign bit was `1`. `>>>` always fills with `0`, preserving the raw bit pattern regardless of sign. Since this problem treats `n` as a bit *pattern*, not a signed magnitude, `>>>` is the only correct choice.

---

**Q2. Reversing the bits of a small positive number like `1` produces a very large negative number in Java. Is that a bug?**

*Answer:* No — it's an unavoidable consequence of Java having no unsigned 32-bit integer type. Reversing puts the original bit 0 into bit 31, and bit 31 is the sign bit for a Java `int`. Any odd input (lowest bit set) produces a result with its sign bit set, i.e., a negative number, after reversal.

---

**Q3. What's the time complexity of Reverse Bits, and why is it O(1) rather than O(n)?**

*Answer:* O(1). The loop always runs exactly 32 times, fixed by the width of a 32-bit integer — there's no variable "n" whose growth the runtime scales with. A problem's input *size* here is constant by definition (always exactly 32 bits), which is what makes this O(1) rather than O(32) written loosely as if it were input-dependent.

---

**Q4. Why does a Trie work for maximizing XOR — what's the actual reframe that makes this a Trie problem?**

*Answer:* Instead of comparing every pair directly (O(n²)), insert every number's binary representation into a tree where a shared prefix (shared high-order bits) is stored once. Then, for each number, ask the tree: "what's the closest thing to my exact *bitwise opposite* that you contain?" — answered by walking the tree once, preferring the opposite-bit child at each level. That single O(bit-length) walk replaces comparing against every other number individually.

---

**Q5. Prove that the greedy opposite-bit choice at each level of the Bit Trie walk is actually optimal — don't just assert it.**

*Answer:* For k-bit numbers, a `1` at bit position `i` contributes `2^i`. The maximum possible combined contribution of *every* bit position below `i` is `2^(i-1) + ... + 2^0 = 2^i − 1`, which is strictly less than `2^i`. So no combination of lower bits can ever outweigh a single higher bit — meaning whichever choice maximizes a higher bit position is always safe to commit to first, permanently, regardless of what it forces at lower positions. That's exactly what greedily preferring the opposite bit from the most significant bit downward does.

---

**Q6. What happens during the query walk when the opposite-bit child doesn't exist?**

*Answer:* You're forced to descend into the only child that does exist (the matching-bit child), which contributes a `0` at that bit position in the result. This isn't a failure case — it's just a position where no number in the tree so far happens to differ from the query number at that bit; the walk continues normally from there.

---

**Q7. What's the time and space complexity of the Bit Trie approach, and how does it compare to brute force?**

*Answer:* O(n) time (n insertions + n queries, each O(31), with 31 treated as a constant) and O(n) space (at most 31n nodes), versus brute force's O(n²) time. The improvement comes from the same source every Trie problem in this series has used: a shared prefix is stored once and traversed once, instead of re-scanned per comparison.

---

**Q8. Bit Manipulation just closed at 10 required problems. Name the distinct techniques it covered, without looking anything up.**

*Answer:* XOR self-cancellation for a lone value (Single Number); two ways to count set bits, one O(1)-per-check naive and one using `n&(n-1)`'s popcount-bounded loop (Number of 1 Bits); `n&(n-1)==0` as an iff-test for exactly one set bit (Power of Two); a DP transition built directly on `n&(n-1)` (Counting Bits); XOR cancellation as a missing-value detector, contrasted against an overflow-prone sum formula (Missing Number); per-bit frequency counting generalized past mod-2 (Single Number II); `n&(-n)` isolating the lowest set bit to partition a XOR-fold into two independent halves (Single Number III); bitwise addition simulating a hardware full-adder (Sum of Two Integers); full 32-bit reversal (Reverse Bits); and today's Bit Trie fusion with Tries (Maximum XOR).

---

**Q9. What is a Pod, precisely — how is it different from a container?**

*Answer:* A container (Week 7) is one isolated packaged process. A Pod is Kubernetes' smallest deployable *unit* — a wrapper around one or more containers that are always scheduled together on the same machine, sharing a network namespace. Most Pods wrap exactly one container; the multi-container case exists but wasn't needed today. The key distinction: you don't run containers directly on a Kubernetes cluster, you run Pods, which contain containers.

---

**Q10. Explain the declarative reconcile-loop model — why is a Deployment fundamentally different from a `docker run` command?**

*Answer:* `docker run` is imperative — a one-shot action, executed once, done. A Deployment declares a desired *ongoing* state (e.g., "3 replicas of this Pod template should exist, always"), and a controller continuously compares that declared state against actual reality, taking whatever action closes any gap, in a loop that never stops. If a Pod dies, the controller notices and creates a replacement without anyone re-running a command.

---

**Q11. What's the actual chain of "who manages whom" — Deployment, ReplicaSet, Pod?**

*Answer:* You typically create and edit a Deployment. The Deployment creates and manages a ReplicaSet (and creates a *new* ReplicaSet on top of the old one during a rolling update). The ReplicaSet creates and manages the actual Pods, continuously ensuring the live Pod count matches its `replicas` target.

---

**Q12. Why is a Service needed at all — why can't other parts of the system just talk to Pods directly by IP?**

*Answer:* Pods are disposable and their IPs are not stable — a crashed Pod's replacement gets a new IP, not the old one. A Service provides one stable address that load-balances across whatever set of Pods currently match its selector, so nothing else in the system needs to track individual Pod identities as they come and go.

---

**Q13. An HPA is configured with `averageUtilization: 70`, but replica count never increases even under heavy load. What's the most likely misconfiguration?**

*Answer:* The target Deployment's containers probably don't have `resources.requests.cpu` set. HPA's utilization percentage is computed relative to the CPU *request*, not the node's total capacity or a `limits` value — without a request set, there's no baseline to compute a percentage against, and CPU-based scaling can't function correctly.

---

**Q14. Why does HPA treat scaling up and scaling down asymmetrically?**

*Answer:* Reacting quickly to a genuine load spike is desirable — under-provisioning during real demand costs users a slow or failing service. Reacting quickly to a *dip* is riskier: a brief lull followed by another spike would cause replicas to be destroyed and immediately recreated (each with real startup cost), a wasteful thrashing pattern. HPA's `behavior` field lets scale-up and scale-down be tuned with separate stabilization windows for exactly this reason, with scale-down deliberately more conservative by default.

---

## Day 100 — Segment Trees Open, Singleton (DCL + Enum)

**Q1. Why isn't a plain Prefix Sum array good enough for Range Sum Query - Mutable?**

*Answer:* Prefix Sum gives O(1) queries, but a single `update` invalidates every prefix sum from that index onward, forcing an O(n) rebuild. This problem calls both `update` and `sumRange` repeatedly, so O(n) updates are unacceptable — exactly the gap a Segment Tree closes by landing both operations at O(log n).

---

**Q2. Walk through why a Segment Tree's range query is O(log n) — don't just state it.**

*Answer:* At each level of the recursion, node ranges partition the array into disjoint blocks. A node is either fully outside the query (return in O(1)), fully inside it (return the precomputed aggregate in O(1)), or straddles one of the query's two boundaries (must recurse into both children). At most two nodes per level can straddle a boundary — one per edge of the query range — so each level contributes only a small constant number of "still recursing" nodes. With O(log n) levels total, the whole query visits O(log n) nodes.

---

**Q3. What's the index math for navigating the tree, and where has it been seen before in this series?**

*Answer:* Node `i`'s children are at `2i+1` and `2i+2`, parent at `(i-1)/2` — identical to Week 8's array-backed heap indexing. The structures store different things (a heap node holds one prioritized element; a Segment Tree node holds a range aggregate), but the navigation math is unchanged.

---

**Q4. Segment Tree vs. Fenwick Tree (BIT) — what's the actual trade-off?**

*Answer:* Fenwick Trees get the same O(log n) update/query for sums with a much smaller constant factor (a flat array, iterative, no node objects) — but they rely on invertibility (range = prefix(r) − prefix(l−1)), so they naturally support sum/XOR but not min/max, which have no inverse. Segment Trees handle any associative combine function uniformly, at the cost of more code and a larger constant factor.

---

**Q5. If this problem needed range updates (add x to every element in `[l, r]`) instead of point updates, how would that change the approach?**

*Answer:* Lazy propagation — deferring a pending update at a node until a query actually needs to descend past it, so a range update doesn't have to eagerly touch every individual leaf in the range. Named here as the correct answer; not built today, since today's exposure is deliberately kept to point updates.

---

**Q6. Name all three Creational patterns from today and, in one line each, what each one solves.**

*Answer:* Singleton — exactly one instance of a class, globally reachable. Factory Method — delegates *which* concrete class gets instantiated to a subclass or dedicated method, decoupling the caller from concrete constructors. Builder — constructs a complex object step by step, avoiding a constructor with a long list of optional parameters.

---

**Q7. Why is the naive lazy Singleton (`if (instance == null) instance = new X();`, no locking) broken under concurrency?**

*Answer:* Two threads can both evaluate `instance == null` as true before either finishes constructing an instance — both then proceed to construct and assign, producing two distinct instances. The same race-condition shape as Week 6, Day 37's `Counter` example, applied to object construction instead of an integer increment.

---

**Q8. Why does Double-Checked Locking check `instance == null` twice instead of once?**

*Answer:* The first check (unsynchronized) is a fast path — once the instance exists, the overwhelming majority of calls return immediately without ever touching the lock. The second check, inside the `synchronized` block, exists because multiple threads may have all passed the first check and be queued at the lock; only the first one through should construct the instance, and every thread behind it must re-check to discover it's already done, or each would construct its own.

---

**Q9. Why does Double-Checked Locking specifically need `volatile` on the instance field — what actually breaks without it?**

*Answer:* `instance = new X()` is really three steps — allocate, run the constructor, assign the reference. Without `volatile`, the JVM/CPU may legally reorder the constructor's field writes and the reference assignment, since that reordering is invisible to the single thread doing the construction. A second thread could then see a non-null `instance` (published early) before the constructor has actually finished running, and could observe a partially-initialized object — default field values instead of real ones. `volatile` forbids exactly this reordering for its own reads/writes and establishes a happens-before relationship, guaranteeing any thread that observes a non-null `instance` also sees the fully-completed construction.

---

**Q10. Is `volatile` a substitute for `synchronized` here?**

*Answer:* No — they solve different problems. `synchronized` provides mutual exclusion (only one thread constructs the instance). `volatile` provides visibility/ordering (a thread that sees the reference also sees the fully-constructed object behind it). Double-Checked Locking needs both; neither alone is sufficient.

---

**Q11. Why is Enum Singleton thread-safe with no explicit lock or `volatile` at all?**

*Answer:* JVM class initialization is specified to be thread-safe — the classloader effectively holds an implicit lock during initialization, and a class is guaranteed to initialize exactly once regardless of how many threads reference it concurrently for the first time. The enum constant is created during that initialization, so it inherits this guarantee directly.

---

**Q12. How is Enum Singleton immune to reflection-based attacks that a conventional Singleton isn't?**

*Answer:* A conventional Singleton's private constructor can still be invoked via reflection (`setAccessible(true)` + `newInstance()`), silently creating a second instance. The JDK explicitly guards against this for enums: `Constructor.newInstance()` checks whether the declaring class is an enum and throws `IllegalArgumentException("Cannot reflectively create enum objects")` if so — a deliberate, hard-coded safeguard specific to enum types.

---

**Q13. How is Enum Singleton immune to serialization-based attacks?**

*Answer:* Deserializing an ordinary Singleton with Java's default mechanism reconstructs the object's fields directly, without calling any constructor — producing a second, distinct instance, unless `readResolve()` is explicitly implemented to redirect back to the existing one. Enums are serialized specially: only the constant's name is written, and deserialization resolves it via `Enum.valueOf()`, which looks up the existing constant rather than constructing anything — structurally incapable of producing a duplicate.

---

**Q14. What's the real trade-off of choosing Enum Singleton over Double-Checked Locking?**

*Answer:* Enum Singleton is simpler and safer by construction — but an enum implicitly extends `java.lang.Enum`, so it cannot extend any other superclass (it can still implement interfaces). Double-Checked Locking allows normal class inheritance at the cost of needing the `volatile`/locking reasoning to be exactly right.

---

## Day 101 — Segment Trees Close, Factory Method, Builder

**Q1. What's different about how today's Segment Tree is indexed, compared to yesterday's?**

*Answer:* Yesterday's tree was indexed by array *position* — leaf `i` held `nums[i]`. Today's tree is indexed by *value* (after coordinate compression) — each leaf represents a possible value, and holds a count of how many times that value has been inserted so far. Same structure, genuinely different mental model of what a leaf means.

---

**Q2. Why does Count of Smaller Numbers After Self process the array right to left?**

*Answer:* At the moment index `i` is processed, only elements at indices greater than `i` should have been inserted into the tree — those are exactly the elements "to the right of `i`." Processing right to left guarantees that invariant by construction: nothing at or left of `i` has been touched when `i` is queried.

---

**Q3. What is coordinate compression, and why is it needed here rather than indexing directly by raw value?**

*Answer:* Sort the array, deduplicate, and map each distinct value to its rank (its position in the sorted-unique list) — collapsing an arbitrary, possibly huge or sparse value range into a dense `0..k-1` index space. It's needed because the tree needs to be sized by *number of distinct values*, not by the raw magnitude of those values, which could be enormous or negative.

---

**Q4. Why does the query use range `[0, rank-1]` instead of `[0, rank]`?**

*Answer:* The problem wants strictly smaller elements. Since ranks are assigned to distinct (deduplicated) values, `[0, rank-1]` counts only values strictly less than the current one; including `rank` itself would incorrectly count values equal to the current one as "smaller."

---

**Q5. What's the overall time complexity, and what dominates it?**

*Answer:* O(n log n) — sorting for coordinate compression is O(n log n), and each of the n insert/query pairs against the Segment Tree costs O(log k) where k ≤ n. Both terms are O(n log n); neither dominates the other asymptotically.

---

**Q6. How does a Fenwick Tree's indexing connect to material from Week 14?**

*Answer:* A Fenwick Tree's `update`/`prefixSum` loops both step by `i & (-i)` — the exact "isolate the lowest set bit" identity derived in Week 14, Day 98. That value determines each index's "span of responsibility" within the structure.

---

**Q7. Why can't a Fenwick Tree be used for a range-minimum-query version of this kind of problem?**

*Answer:* Fenwick Trees compute a range aggregate as `prefixSum(r) − prefixSum(l−1)`, which requires the aggregate to be invertible (undoable via subtraction). Sum and XOR support this; min and max do not — there's no way to "subtract out" a minimum from a combined result. Min/max range queries need a Segment Tree instead.

---

**Q8. What's the actual difference between a "Simple Factory" and the true GoF Factory Method pattern?**

*Answer:* A Simple Factory is one static method with conditional branching (switch/if-else) choosing which concrete class to instantiate — adding a new type means editing that method. True Factory Method is an abstract creator with an abstract method that each concrete subclass overrides to produce its own product — adding a new type means adding a new subclass, with zero existing code touched. The second one respects the Open/Closed Principle; the first doesn't.

---

**Q9. What problem does Builder solve that a constructor with many optional parameters doesn't?**

*Answer:* It avoids the "telescoping constructor" problem — an unreadable, error-prone pile of positional arguments, especially risky once two parameters share a type and can be silently swapped with no compiler error. Builder's fluent, named method calls make every field self-documenting at the call site.

---

**Q10. What specifically is lost if a "Builder" is implemented over a target class that still has mutable, publicly-settable fields?**

*Answer:* The immutability and atomic-validation benefits — arguably the more important half of the pattern. A mutable target can exist in a partially-configured, invalid intermediate state at any point, and there's no single moment where "is this object valid" gets enforced. Only the readability of the fluent call syntax survives; the correctness guarantees Builder is usually reached for do not.

---

**Q11. Why does making the built object's fields `final` matter beyond just "good practice"?**

*Answer:* It gives thread-safety for free — a fully immutable object (all fields final, set once in a private constructor, no mutation possible afterward) can be freely shared and read across multiple threads with zero synchronization needed, since there's no mutable state for concurrent access to corrupt.

---

## Day 102 — Merge Sort, Quicksort, Quickselect, DSA Retrospective

**Q1. Prove merge sort's O(n log n) bound — don't just cite it.**

*Answer:* The recurrence is `T(n) = 2T(n/2) + O(n)` — two half-sized recursive calls plus O(n) merge work. By the Master Theorem, `a=2, b=2` gives `n^(log_b a) = n`, matching `f(n) = O(n)` exactly (Case 2), so `T(n) = Θ(n log n)`. This holds in every case — best, average, worst — because the split is always even regardless of input values, unlike quicksort's data-dependent partitioning.

---

**Q2. Why is merge sort stable, mechanically — not just "because it is"?**

*Answer:* The merge step compares with `<=`, so when two front elements are equal, the element from the left half is always taken first. Since the left half's elements originally preceded the right half's in the pre-split array, this preserves their relative order. The mechanism is copy-based (never swaps), which is what makes this guarantee possible in the first place.

---

**Q3. Give a concrete input that triggers quicksort's O(n²) worst case with "always pick the last element" pivot selection, and explain why.**

*Answer:* An already-sorted array. Every partition call picks the largest remaining element as pivot, splitting into a group of size `n-1` and a group of size `0` — O(n) work per call, but the problem only shrinks by 1 each time. Total work is `n + (n-1) + ... + 1 = O(n²)`, the same arithmetic-series shape as other naive O(n²) approaches in this series.

---

**Q4. How does randomizing the pivot fix quicksort's worst case?**

*Answer:* It doesn't change the theoretical worst case — an unlucky sequence of random choices is still possible. It removes the ability for any *fixed input* to reliably trigger that worst case, since the pivot no longer depends on array position. This gives expected O(n log n) regardless of input arrangement, which is the standard mitigation.

---

**Q5. Is quicksort, as implemented today, stable? Why or why not?**

*Answer:* No. The partition step swaps elements to maintain its boundary invariant, and a swap can reorder two equal elements relative to each other with no logic protecting their original order — fundamentally different from merge sort's copy-based, order-preserving merge.

---

**Q6. Why does Java's `Arrays.sort()` use dual-pivot quicksort for primitives but TimSort for objects — give the actual mechanism, not just the names.**

*Answer:* Three reasons: (1) stability matters for objects (two `Person`s with the same last name have a reasonable expectation of keeping relative order) but is meaningless for primitives (two equal ints are indistinguishable); (2) object comparisons are user-supplied, potentially expensive or even adversarial, making merge sort's guaranteed-worst-case O(n log n) the safer choice, while primitive comparisons are cheap, fixed hardware operations where quicksort's average-case speed and in-place nature are a bigger practical win; (3) in-place swapping's cache-locality advantage matters more for contiguous primitive data than for arrays of object references, which are already one level of indirection removed from their actual data.

---

**Q7. What is TimSort, precisely — is it accurate to call it "a variant of merge sort"?**

*Answer:* Yes, accurately — its core mechanism is merging sorted subsequences, the same fundamental idea as merge sort. It's adaptive on top of that: it detects existing sorted "runs" in the input and merges those directly (fast on partially-sorted real-world data), and uses insertion sort for small runs since insertion sort's low overhead beats merge sort's on small inputs.

---

**Q8. Quickselect and quicksort both use the exact same `partition` method. Why is Quickselect's average complexity O(n) and not O(n log n) like quicksort's?**

*Answer:* Quicksort recurses into *both* sides of each partition — `T(n) = 2T(n/2) + O(n)`, giving O(n log n). Quickselect recurses into only *one* side, discarding the other entirely, since only one side can contain the target index — `T(n) = T(n/2) + O(n)`. Expanded out, this is the geometric series `n + n/2 + n/4 + ... ≈ 2n = O(n)`, dominated by its first term — a fundamentally different shape from quicksort's recurrence, not just a smaller constant on the same shape.

---

**Q9. Why does Quickselect's loop convert "k-th largest" into `n - k` as a target index?**

*Answer:* If the array were fully sorted in ascending order, the k-th largest element would sit at index `n - k` (0-indexed) — e.g., the 1st largest (the maximum) is at index `n-1`. Partitioning repeats until the pivot's own final position exactly equals that target index.

---

**Q10. What are the real, practical trade-offs between the heap approach and Quickselect for this exact problem?**

*Answer:* The heap is O(n log k) time, O(k) space, guaranteed worst case, and works on a stream without needing the full array in memory upfront. Quickselect is O(n) average time, O(1) extra space, but has a real (if unlikely, with randomization) O(n²) worst case, requires random access to the whole array upfront, and mutates the input array in place — a genuine problem if the caller needs the original order preserved.

---

## Day 103 — SQL Part 1: Subqueries, Aggregation, Self-Joins

**Q1. State SQL's logical query processing order, and explain what it's actually used to determine.**

*Answer:* `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`. It determines what's legal to reference where — most importantly, why `WHERE` cannot filter on an aggregate result (it runs before grouping/aggregation exist) while `HAVING` can (it runs after).

---

**Q2. Why does `SELECT MAX(salary) FROM Employee WHERE salary < (SELECT MAX(salary) FROM Employee)` correctly return `NULL`, rather than erroring, when every salary is identical?**

*Answer:* If every salary is identical, no row satisfies `salary < (the max)`, so the `WHERE` clause matches zero rows. `MAX()` applied to an empty result set returns `NULL` in standard SQL rather than erroring or returning no rows — which is exactly the output LC176 expects, with no special-casing required.

---

**Q3. Why is `DISTINCT` required in the `LIMIT 1 OFFSET 1` approach to Second Highest Salary?**

*Answer:* Without it, a duplicated highest salary occupies two rows before any lower value appears — `ORDER BY salary DESC LIMIT 1 OFFSET 1` would return the *second row*, which could still be the same (highest) value, not the second-highest *distinct* value. `DISTINCT` collapses duplicate values first, so "offset 1" correctly means "the next distinct value."

---

**Q4. What does it mean for a subquery to be "correlated," mechanically?**

*Answer:* A correlated subquery references a column from the outer query's current row (e.g., `e2.departmentId = e.departmentId`, where `e` is the outer alias). Conceptually, it's re-evaluated once per outer row, each time producing a value specific to that row's context — unlike an independent subquery, which can be evaluated once regardless of the outer query.

---

**Q5. Why does the correlated-subquery approach to Department Highest Salary correctly return tied employees, while a plain `GROUP BY departmentId` with `MAX(salary)` would not?**

*Answer:* `GROUP BY departmentId` collapses each department to a single output row, which can't represent multiple tied top earners at once without extra machinery. Comparing each individual employee's salary against their department's correlated max (`WHERE e.salary = (correlated MAX)`) keeps every row and independently checks each one — so every employee tied for their department's maximum satisfies the condition and appears in the result.

---

**Q6. How can indexing change a correlated subquery's actual performance, concretely?**

*Answer:* Without an index, the inner correlated lookup can effectively become a full scan of the inner table for every outer row — O(n) per outer row, O(n²) overall in the worst case. An index on the correlated column (here, `(departmentId, salary)`) turns that inner lookup into a fast index range scan instead of a full scan, the same O(log n)-vs-O(n) distinction established for indexes back in Week 6.

---

**Q7. What's the precise difference between `WHERE` and `HAVING`, and why can't they be swapped?**

*Answer:* `WHERE` filters individual rows before any grouping occurs; `HAVING` filters groups after `GROUP BY` has collapsed rows and computed aggregates. `WHERE` can't reference an aggregate because none exists yet at that stage of processing; `HAVING` can, because it runs strictly after aggregation.

---

**Q8. Why does Rising Temperature's self-join use `DATEDIFF(...) = 1` instead of comparing adjacent row IDs?**

*Answer:* If the table has any gaps in recorded dates, "the next row" and "the next calendar day" are no longer guaranteed to be the same thing — an ID-adjacency join would silently compare across a gap. `DATEDIFF` checks the actual calendar relationship directly, which stays correct regardless of whether the data happens to be gapless.

---

**Q9. In Employees Earning More Than Their Managers, why does `INNER JOIN` correctly exclude employees with no manager, without any extra `WHERE` clause needed?**

*Answer:* A manager-less employee has `managerId IS NULL`. Any comparison involving `NULL`, including `NULL = e2.id`, evaluates to `UNKNOWN` under SQL's three-valued logic — never `TRUE`. A `JOIN` condition only keeps rows where it evaluates to `TRUE`, so that employee's row simply never matches and is dropped, correctly and by direct design — the same NULL-comparison principle already established in Week 6's `NOT IN` trap.

---

**Q10. A self-join is written using two aliases of the same table. What does each alias conceptually represent, and why is aliasing necessary at all?**

*Answer:* Each alias lets the same physical table be referenced as if it were two separate tables playing two different roles in the query — e.g., "yesterday" vs. "today," or "employee" vs. "their manager." Aliasing is necessary because SQL needs some way to distinguish which occurrence of a column (e.g., which table's `recordDate`) is being referred to in the `JOIN`/`WHERE` clauses; without distinct aliases, the query can't disambiguate the two roles.

---

## Day 104 — SQL Part 2: Window Functions

**Q1. What's the fundamental difference between a window function and `GROUP BY`?**

*Answer:* `GROUP BY` collapses rows sharing a value into one output row per group. A window function does not collapse anything — every input row still appears in the output, with an additional computed column reflecting some aggregate or ranking calculated over a related set of rows (the "window").

---

**Q2. On a dataset with a tie, walk through exactly how `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` would each number the rows.**

*Answer:* For descending salaries `[100, 100, 90, 80]`: `ROW_NUMBER()` gives `1, 2, 3, 4` — always unique, breaking the tie arbitrarily. `RANK()` gives `1, 1, 3, 4` — tied rows share a rank, but the next distinct value's rank skips ahead by the number of tied rows. `DENSE_RANK()` gives `1, 1, 2, 3` — tied rows share a rank, and the next distinct value's rank increases by exactly 1, with no gap.

---

**Q3. Why can't a window function's output be referenced directly in that same query's `WHERE` clause?**

*Answer:* Window functions are evaluated after `WHERE`/`GROUP BY`/`HAVING` in SQL's logical processing order — conceptually alongside the final `SELECT` list. At the point `WHERE` runs, the window function's result doesn't exist yet. Filtering on it requires wrapping the windowed query in a subquery (or CTE) and filtering in the outer query instead.

---

**Q4. Why is `DENSE_RANK()` specifically the right choice for "Nth highest salary," rather than `RANK()` or `ROW_NUMBER()`?**

*Answer:* `DENSE_RANK()` numbers distinct *values*, not rows — which matches "Nth highest salary"'s actual intent (a question about distinct salary values). It's the same underlying idea as yesterday's `SELECT DISTINCT salary ... LIMIT/OFFSET` approach, expressed through ranking instead of deduplication, rather than a different technique entirely.

---

**Q5. What does `LAG(num, 2) OVER (ORDER BY id)` return for the first two rows of the ordering, and why?**

*Answer:* `NULL` for both — `LAG` looks a fixed number of positions backward within the ordering, and there simply isn't a row two positions before the 1st or 2nd row. This falls out of the mechanism directly, with no special-casing required, and further downstream comparisons against a `NULL` correctly evaluate to `UNKNOWN`/not-true under SQL's three-valued logic.

---

**Q6. In Trips and Users, why is `Users` joined twice instead of once?**

*Answer:* The query needs to check the banned status of two different people playing two different roles in the same trip — the client and the driver — both of whom are looked up in the same `Users` table. Joining it twice, under two different aliases, lets each role's banned status be checked independently within the same query.

---

**Q7. Explain exactly why `SUM(CASE WHEN status != 'completed' THEN 1 ELSE 0 END)` correctly counts cancelled trips within a group.**

*Answer:* The `CASE` expression evaluates to `1` for each row where the trip isn't completed (i.e., was cancelled) and `0` otherwise — per row. `SUM()` then adds up those per-row values across the group; since only matching rows contribute a nonzero amount (exactly `1` each), the total equals the count of matching rows in that group.

---

**Q8. In Exchange Seats, why is the `id != (SELECT MAX(id) FROM Seat)` check necessary in the first `CASE` branch?**

*Answer:* Without it, the last seat — if its id is odd (i.e., the total seat count is odd) — would still try to swap with a nonexistent next seat (`id + 1`), which doesn't exist. That check specifically detects "this is a trailing, unpaired odd id" and routes it to the fallback branch (stay in place) instead of attempting an invalid swap.

---

**Q9. Prove, with a concrete counter-example, why `RANK()` would give a wrong answer for Department Top Three Salaries.**

*Answer:* Take salaries `[90000, 90000, 85000, 80000]` in one department, with a tie for first. `RANK()` gives both `90000`s rank 1, then jumps to rank 3 for `85000` (correct so far) but rank 4 for `80000` — pushing it past a `rnk <= 3` filter and wrongly excluding it, even though `80000` is genuinely the 3rd-highest *distinct* salary in the department. `DENSE_RANK()` gives `1, 1, 2, 3` instead, correctly keeping `80000` at rank 3.

---

**Q10. How does `PARTITION BY` relate to what yesterday's correlated subquery (Department Highest Salary) was doing?**

*Answer:* Yesterday's correlated subquery recomputed `MAX(salary)` independently for each outer row's department — effectively "restarting" the calculation per group, but only capable of expressing a single aggregate value (the max) per group. `PARTITION BY departmentId` generalizes that same "restart independently per group" idea into the window-function framework, but supports full rankings (not just a single max), which is exactly what "top 3 per department" needs and a correlated `MAX` subquery structurally cannot provide without much more complexity.

---

## Day 105 — Consolidation & DSA Curriculum Retrospective

*(A consolidation day's interview questions test recall and synthesis across the whole DSA phase, not new material — treat these as a spot-check on the self-check exercise above, not a new quiz.)*

**Q1. What's the actual difference between "197 required DSA problems" and "249 distinct DSA problems solved" — why are both numbers correct?**

*Answer:* 197 counts only what the plan's own week-by-week ladder explicitly called for. 249 additionally includes every extra-practice problem this series added on top of that ladder, specifically for patterns judged to need more reps than the plan alone provided (53 extra problems across the whole series). Both are accurate; they answer different questions — "what was assigned" versus "what was actually solved."

---

**Q2. Name three patterns that this series added or substantially expanded beyond the original plan, and why.**

*Answer:* Greedy/Intervals and Prefix Sum/Kadane's didn't exist anywhere in the original plan and were added as genuine gaps. Heaps, Union-Find, Dijkstra's, and Tries were flagged by an early audit as too thin in the original plan and were expanded to comprehensive depth. The SQL practice track (10 problems) was added entirely outside the original plan's scope this week.

---

**Q3. Without looking anything up: what's the one-sentence interview signal for Sliding Window?**

*Answer:* "Contiguous subarray/substring" combined with a size, sum, or distinct-count constraint.

---

**Q4. Without looking anything up: what's the one-sentence interview signal for Union-Find?**

*Answer:* "Are these connected" — dynamic connectivity queries, or redundant-edge detection.

---

**Q5. What's the difference between recognizing a pattern and remembering its exact code — and why does today's self-check specifically test the former?**

*Answer:* Remembering exact code is of limited value in a live interview, where the actual challenge is hearing an unfamiliar problem statement and quickly identifying which of the many techniques covered actually applies. Today's random-pattern, cold-solve exercise specifically measures that recognition reflex — how fast the right tool gets identified — not whether exact syntax is memorized.

---

**Q6. This week added zero extra-practice problems across every pattern it touched. Is that a gap, or a deliberate choice — and how would you defend it if asked?**

*Answer:* Deliberate, and defensible pattern by pattern: Bit Manipulation was closing with an already-comprehensive 11-problem history; Segment Trees and the SQL track were both explicitly scoped by the plan as fixed-size, deliberately-light tracks (2 and 10 problems respectively) rather than open-ended patterns; Sorting's real deliverable was a from-scratch implementation, not additional LeetCode reps. None of these were silent defaults — each was reasoned through and stated explicitly in that day's own material.

---

**Q7. If, during this week's self-check, one pattern's cold-solve genuinely stalled — what's the correct response?**

*Answer:* Treat it as real, useful diagnostic information, not a failure to dismiss. Revisit that specific pattern's original Resource Book before moving on, focusing on the interview signal and the core mechanism rather than re-solving every problem from that pattern — the goal is restoring recognition speed, not re-doing the entire pattern from scratch.
