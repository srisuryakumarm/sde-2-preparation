# Week 6 — Consolidated Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Source:** every "Day N — Interview Questions" section from [Day 36](Day36_Resource_Book.md) through [Day 42](Day42_Resource_Book.md), reorganized by topic for review. 59 questions total. Each entry cites its source day — if an answer feels thin, the source day's full Resource Book has the complete derivation, worked trace, and common-mistake context behind it.

---

## 1. Linked Lists — Cycle Detection (Floyd's) *(5 questions, Day 36)*

**Q1. Explain the fast/slow pointer mechanism, and prove it must detect a cycle if one exists.** Two pointers start at `head`; each iteration, `slow` advances one node, `fast` advances two. If a cycle exists, once both pointers are inside it, the gap between them (measured along the cycle) shrinks by exactly 1 every iteration — since it can never skip past 0, it must hit exactly 0 within at most the cycle's length in further steps, forcing a meeting.

**Q2. Why does Cycle II reset a pointer to `head` instead of continuing from the meeting point?** Because the distance from `head` to the cycle's start (`a`) is provably equal to the distance from the meeting point forward to the cycle's start (`c − b`), modulo whole trips around the cycle — derived from `a + b + nc = 2(a+b)`. Advancing both pointers one step at a time from those two starting points therefore lands them on the same node — the cycle's start — at the same time.

**Q3. Does the two-phase approach ever fail to find the true cycle start?** No — the derivation doesn't depend on which specific meeting point Phase 1 happens to land on; any valid `b` satisfies the same equation, so Phase 2 is correct regardless of exactly where inside the cycle the pointers first meet.

**Q4. How would you recognize that Happy Number is a cycle-detection problem, given that it never mentions a linked list?** Any deterministic process with a well-defined "next state" and a bounded set of possible states either terminates or eventually revisits a state — which is structurally a linked list with a cycle, whether or not any object with a `.next` field exists.

**Q5. Why does Find the Duplicate Number specifically forbid sorting or extra data structures, and how does that shape the solution?** Those constraints (no modification, O(1) space) rule out both an O(n) HashSet and an in-place sort's implicit assumptions, forcing a technique that uses no extra memory — which is exactly what Floyd's, reused via treating array values as implicit pointers, provides.

---

## 2. Linked Lists — Pointer Manipulation *(8 questions, Days 37–38)*

**Q6. Why does Remove Nth Node From End need a gap of `n + 1`, not `n`?** *(Day 37)* `slow` needs to stop one node *before* the target so `slow.next = slow.next.next` can perform the removal; leading `fast` by exactly `n` would land `slow` directly on the target instead of just before it.

**Q7. Why is a dummy head necessary for Remove Nth Node From End specifically?** *(Day 37)* If the node to remove is the head itself, there's no preceding node to redirect without one — the dummy gives every case, including removing the head, a uniform predecessor to operate on.

**Q8. In Add Two Numbers, why does the loop need to check `carry != 0` in addition to both lists being non-null?** *(Day 37)* Both lists can be fully exhausted while a final carry still needs to produce one more output digit (e.g., 99 + 1) — omitting this check would silently drop that digit.

**Q9. Why is reverse-digit order actually convenient for Add Two Numbers, rather than an obstacle?** *(Day 37)* Addition by hand starts from the least-significant digit, which is exactly where these lists already begin — no reversal is needed before summing.

**Q10. Walk through Reorder List's approach and name which prior technique each step reuses.** *(Day 38)* Find the middle via fast/slow (Day 34); split and reverse the second half (Day 34's reversal); merge by alternating one node from each half (Day 35's dummy-head merge, adapted from sorted-order merging to alternation).

**Q11. Why does an odd-length list's unpaired middle node correctly end up last in Reorder List, with no special case?** *(Day 38)* The middle-finding split gives the first half one extra node on odd lengths; during the alternating merge, the second half (the reversed tail) runs out one iteration before the first half does, so the first half's final extra node is simply appended last, with the loop ending as soon as `second` is `null`.

**Q12. In Copy List with Random Pointer, why does `oldToNew.get(current.next)` work correctly even when `current.next` is `null`?** *(Day 38)* `HashMap` allows a `null` key; since `put(null, ...)` was never called, `get(null)` returns `null` — exactly the desired result when a node has no next node, with no explicit branch needed.

**Q13. What makes the brute-force approach to Copy List with Random Pointer O(n²) specifically?** *(Day 38)* For each of `n` nodes, finding its `random` target's position requires an O(n) scan of the original list, and then walking that far into the new list costs another O(n) — O(n) work per node, O(n²) total.

---

## 3. Linked Lists — LRU Cache & Doubly Linked Lists *(4 questions, Day 39)*

**Q14. Why does removing a known node take O(1) in a doubly linked list but not a singly linked one?** A doubly linked node holds `prev` directly, so `node.prev.next = node.next` needs no search. A singly linked node has no way to find its predecessor except a traversal from `head`, which is O(n) even with a direct reference to the node itself.

**Q15. Why does LRU Cache need two different data structures rather than one?** No single structure gives both O(1) lookup by key and O(1) reordering by recency — a HashMap alone has no ordering; a linked list alone has O(n) lookup. Combining them, with the map holding direct node references into the list, gives O(1) for both.

**Q16. Why must `get()` in LRU Cache modify the data structure, even though it's conceptually a "read"?** The eviction policy is based on *recency of use*, and a read counts as use — skipping the move-to-front step would make the cache track insertion order instead, which is a different, incorrect policy.

**Q17. What goes wrong if eviction removes a node from the linked list but forgets to remove it from the map?** The map retains a stale entry pointing to a node that's no longer part of the tracked list — a subsequent `get()` on the evicted key incorrectly succeeds and returns a value that should no longer be considered present.

---

## 4. Stacks — Fundamentals *(5 questions, Days 39–42)*

**Q18. Why use `ArrayDeque` rather than `java.util.Stack`?** *(Day 39)* `Stack` extends `Vector` and is fully `synchronized` — every operation pays thread-safety overhead even in ordinary single-threaded use. `ArrayDeque` has none of that overhead and is the standard library's own recommended choice.

**Q19. Why is the two-stack queue's `pop()` amortized O(1) despite occasionally costing O(n)?** *(Day 40)* Every element is pushed onto `inStack` once and, over its total lifetime, moved to `outStack` once and popped from it once — a bounded, constant amount of work per element regardless of how the transfers happen to be distributed across individual calls.

**Q20. Why must Min Stack's secondary stack push using `<=` rather than strict `<`?** *(Day 41)* Using strict `<` skips recording a value tying the current minimum — when that minimum is later popped from the main stack, the secondary stack incorrectly believes the minimum has changed, when in fact an equal value is still present in what remains.

**Q21. Why is Evaluate RPN's stack usage fundamentally different in kind from this week's Monotonic Stack problems?** *(Day 42)* There's no ordering invariant being maintained — the stack is used purely for its LIFO property, since the two most recently seen operands are always exactly what the next operator should apply to. It's the same *kind* of stack usage as Valid Parentheses, not a Monotonic Stack problem at all.

**Q22. Why does swapping the pop order in Evaluate RPN produce a wrong answer instead of a crash?** *(Day 42)* Both pop calls succeed regardless of order — nothing throws. Addition and multiplication are commutative, so the bug is invisible on those tokens, but subtraction and division are order-sensitive, and the swapped order computes the mathematically wrong (but syntactically valid) result silently.

---

## 5. Stacks — Monotonic Stack *(9 questions, Days 40–42)*

**Q23. Prove that the Monotonic Stack in Next Greater Element I is guaranteed to stay decreasing from bottom to top.** *(Day 40)* Any element still on the stack has, by definition, not yet had something bigger appear after it — the `while` loop pops anything smaller the instant something bigger arrives. For any two elements with one below the other, the lower one could only remain below the upper one if the upper one was smaller when it was pushed — otherwise it would have popped the lower one immediately. The invariant is actively enforced on every push, not incidental.

**Q24. Why is Next Greater Element I's stack scan O(n) overall, given the nested `while` inside the `for`?** *(Day 40)* Every element is pushed exactly once and popped at most once across the entire run — total operations are bounded by `2n`, linear, regardless of how many iterations any single element happens to sit on the stack for.

**Q25. What two things does Next Greater Element II add on top of yesterday's monotonic stack logic?** *(Day 41)* Iterating `2n` times with `i % n` to simulate circular wraparound, and storing indices instead of values, since this version of the problem allows duplicate values that a value-only stack couldn't distinguish.

**Q26. Why does Next Greater Element II only push during the first `n` iterations?** *(Day 41)* The second lap exists solely to give already-stacked, still-unresolved indices a chance to resolve against the simulated wraparound — pushing again during the second lap would create duplicate entries for positions already represented on the stack.

**Q27. What makes Online Stock Span's use of a monotonic stack different from an array-based next-greater problem?** *(Day 41)* The stack persists across separate method calls over time rather than resetting for one fixed-array pass — it's the identical amortized mechanism, applied to a stream instead of a static array.

**Q28. Why does Online Stock Span accumulate a popped entry's span instead of discarding it?** *(Day 41)* Every day covered by a popped entry's span was already `≤` that entry's price, and that price is itself `≤` today's — so all of those days are transitively `≤` today's price too, and folding their count directly into today's span avoids re-counting them individually.

**Q29. Why does Remove K Digits use an increasing stack instead of a decreasing one?** *(Day 41)* The goal is to eliminate a larger digit sitting before a smaller one, since a smaller leading digit always produces a smaller number regardless of what follows — the exact mirror of what a decreasing stack tracks.

**Q30. When does Remove K Digits need to trim from the end rather than during the main scan?** *(Day 41)* When the input digits are already non-decreasing throughout, so the main loop's pop condition never fires — removing digits to minimize the number then means dropping the least significant (rightmost) remaining digits instead.

**Q31. What's identical between Daily Temperatures and Next Greater Element II, and what's the one thing that changes?** *(Day 42)* Both use an index-based, decreasing Monotonic Stack with the identical amortized O(n) push-once/pop-at-most-once argument. The only difference is what's recorded when a stack entry resolves — Daily Temperatures records the day-gap (`i - j`); Next Greater Element II records the resolving value itself.

---

## 6. Java Concurrency — Locks *(6 questions, Day 37)*

**Q32. Why isn't `count++` atomic?** It compiles to three separate steps — read the current value, increment it in a local register, write it back — and nothing prevents another thread's own read/increment/write from interleaving between any of those three steps.

**Q33. Walk through a concrete interleaving that causes a lost update on a shared counter.** Two threads both read `count = 0` before either writes back; both compute `1`; both write `1` — one of the two increments is silently lost, and the final value is `1` instead of the correct `2`.

**Q34. Is `ReentrantLock` reentrant in a way `synchronized` is not?** No — this is a common misconception. `synchronized`'s intrinsic locks have always been reentrant; a thread can already re-acquire its own held lock without self-deadlocking. `ReentrantLock`'s real advantages are elsewhere: `tryLock()`/timeouts, configurable fairness, interruptible acquisition, and multiple `Condition` objects per lock.

**Q35. What's the single biggest correctness risk `ReentrantLock` introduces that `synchronized` doesn't have?** Release is no longer automatic — a missed `unlock()` call (e.g., an exception thrown before an `unlock()` that isn't in a `finally` block) holds the lock forever, blocking every other thread waiting on it. `synchronized` guarantees release on any method exit, including via exception.

**Q36. Does locking make a program run faster?** No — it makes a specific critical section execute one thread at a time, which is a deliberate trade of some parallelism for correctness, not a performance optimization.

**Q37. Why does running the unsafe counter demo multiple times produce a different final count each time, but the locked version doesn't?** Thread scheduling is non-deterministic, so which specific interleavings of read/increment/write actually occur — and how many updates get lost — varies run to run. The locked version structurally prevents any interleaving inside the critical section, so its result is deterministic every time.

---

## 7. Java Concurrency — Coordination (`wait`/`notify`/`Condition`) *(5 questions, Day 38)*

**Q38. Why must `wait()` be called inside a `while` loop, never an `if`?** Two independent reasons: spurious wakeups are a documented JVM-level possibility with no `notify()` involved at all, and `notifyAll()` can wake threads whose specific condition still doesn't hold (e.g., another producer, when only consumers should proceed) — both require re-checking the actual condition after waking, not assuming it's now true.

**Q39. What does `Condition` provide that `wait()`/`notify()` on a single object does not?** Multiple independent wait-sets from one lock — `lock.newCondition()` can be called more than once, letting semantically different groups of waiting threads (producers vs. consumers) be signaled separately, rather than every waiter sharing one wait-set and having to individually re-check whether a given wakeup was even relevant to them.

**Q40. Why is `notFull.signal()` safe to use instead of `signalAll()` in a bounded-buffer implementation?** Every thread waiting on `notFull` is, by construction, a producer — there's no risk of waking an unrelated thread, since only producers ever call `notFull.await()`. Exactly one slot became available, so waking exactly one producer is sufficient.

**Q41. Must `wait()`/`notify()` and `Condition.await()`/`signal()` be called while holding the associated lock?** Yes — for `wait()`/`notify()`, the calling thread must hold the object's intrinsic lock (i.e., be inside a `synchronized` block/method on that object) or the JVM throws `IllegalMonitorStateException`; `Condition`'s methods carry the identical requirement relative to the `Lock` that created them.

**Q42. Describe a scenario where two locks deadlock.** Thread A holds Lock 1 and is waiting to acquire Lock 2; Thread B holds Lock 2 and is waiting to acquire Lock 1. Neither can make progress, and neither releases what it already holds — resolved in general by always acquiring multiple locks in one consistent order across every thread.

---

## 8. Java Concurrency — `ConcurrentHashMap` *(3 questions, Day 39)*

**Q43. What's the core difference between `Collections.synchronizedMap()` and `ConcurrentHashMap`, given both are thread-safe?** Both are equally correct under concurrent access. `synchronizedMap` uses one lock for the entire map, so every operation from every thread fully serializes. `ConcurrentHashMap` locks at the level of individual buckets (and uses CAS to avoid locking entirely for some operations), so operations on different buckets can proceed in parallel.

**Q44. What is CAS, and why does it avoid needing a lock?** Compare-And-Swap is an atomic hardware instruction: read a value, and if it still matches an expected value, write a new one — as one indivisible step. Because there's no partial, interruptible "middle" to that operation, no other thread can interleave into it, so correctness doesn't require excluding other threads with a lock.

**Q45. Is it ever safe to use a plain, unwrapped `HashMap` from multiple threads concurrently?** No — concurrent structural modification can corrupt its internal bucket structure or silently lose entries; it's not merely slower than the safe alternatives, it's incorrect.

---

## 9. Spring Data JPA / ORM *(6 questions, Day 36)*

**Q46. What does an ORM actually solve?** The object-relational impedance mismatch — Java objects and relational rows are structurally different shapes; an ORM translates between them automatically, so code works with plain objects instead of hand-written SQL and manual `ResultSet` mapping.

**Q47. What's the relationship between JPA, Hibernate, and Spring Data JPA?** JPA is a specification (interfaces/annotations, no implementation). Hibernate is the most common implementation of that specification, doing the real mapping/SQL work. Spring Data JPA sits on top of both, removing further boilerplate — most visibly, generating a working repository implementation from a bare interface.

**Q48. Mechanically, how does `JpaRepository<Task, Long>` work with no implementation class anywhere?** At startup, Spring scans for interfaces extending `JpaRepository`, and — using reflection — generates a dynamic proxy implementation at runtime, backing every method with real JPA calls. The two generic type parameters tell Spring the entity type and its ID type.

**Q49. What's the risk in leaving `ddl-auto: update` on permanently?** Hibernate can alter or drop columns automatically based on what it currently sees mapped, with no review step — safe for early development, but risky once real data exists, since an unintended schema change (or a dropped column no longer mapped) happens silently.

**Q50. What does `@Enumerated(EnumType.STRING)` protect against, and what happens if you omit it?** The default without this annotation is `EnumType.ORDINAL`, which stores an enum constant's integer position — reordering or inserting a new constant later silently changes what previously stored values mean. `EnumType.STRING` stores the constant's name instead, which is stable across reordering.

**Q51. Why does the `Task` entity need a no-args constructor?** Hibernate instantiates entities via reflection, populating fields directly, before any of your own constructor logic would run — a constructor requiring arguments would make reflective instantiation fail at that step.

---

## 10. SQL Fundamentals & Flyway *(6 questions, Day 40)*

**Q52. What's the exact difference between what `INNER JOIN` and `LEFT JOIN` do with a non-matching row?** `INNER JOIN` excludes the row entirely. `LEFT JOIN` includes it, with `NULL` filled in for every column from the side that had no match.

**Q53. Why can `NOT IN` silently return zero rows when `NOT EXISTS` wouldn't?** If the `NOT IN` subquery's result set contains even one `NULL`, every comparison against it evaluates to `UNKNOWN`, which propagates through the `AND` chain and makes the whole `WHERE` condition `UNKNOWN` — treated as `false` — for every row. `NOT EXISTS` only ever checks whether a matching row exists and never directly compares against a `NULL` value, so it has no equivalent failure mode.

**Q54. What's the actual trade-off an index introduces?** Faster lookups (O(log n) via a B-tree, instead of an O(n) full scan) at the cost of slower writes — every `INSERT`/`UPDATE`/`DELETE` must also update every index on the table, not just the row itself.

**Q55. What does 3NF specifically forbid, with a concrete example?** Transitive dependencies — a non-key column depending on another non-key column rather than on the primary key directly. Storing a user's email on every order row is a violation, since the email depends on the user, not the order; duplicating it risks the copies drifting out of sync.

**Q56. What does `ddl-auto: validate` actually check, and what does it refuse to do?** It compares the actual database schema against what the `@Entity` classes expect and fails startup loudly on any mismatch — unlike `update`, it never modifies the schema itself, closing off the silent-alteration risk that convenience carries.

**Q57. Why does Flyway refuse to start if an already-applied migration file's contents have changed?** Migrations are meant to be immutable, append-only history — Flyway tracks a checksum per applied migration specifically to detect this, and treats a mismatch as a sign the historical record of what was actually run against the database can no longer be trusted; the fix is a new migration file, not an edit to an old one.

---

## 11. Week 6, Meta *(2 questions, Day 42)*

**Q58. How many DSA problems did Week 6 add to the running cumulative total, and how many were genuinely new versus recapped?** 14 required slots were filled this week, but only 12 were newly solved — 2 (Valid Parentheses, Implement Queue using Stacks) were recaps of Week 1 solves, already counted before this week began. Including 4 extra-practice problems, Week 6 added **16 genuinely new distinct problems** to the running total.

**Q59. What closed this week, and what opened?** Linked Lists closed at 11 required + 4 extra = 15 distinct problems total. Stacks/Monotonic Stack opened, currently at 7/12 required (2 of which were recaps) + 2 extra = 9 distinct so far, continuing into Week 7.
