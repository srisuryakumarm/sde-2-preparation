# Week 5 Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Companion to:** `Week_05_Revised.md`
**Source books:** [Day 29](Day29_Resource_Book.md) · [Day 30](Day30_Resource_Book.md) · [Day 31](Day31_Resource_Book.md) · [Day 32](Day32_Resource_Book.md) · [Day 33](Day33_Resource_Book.md) · [Day 34](Day34_Resource_Book.md) · [Day 35](Day35_Resource_Book.md)

This bank pulls every interview question from all seven of this week's Resource Books into one place, in the same order they were taught, for cold review without re-opening each day individually. 64 questions total, covering Binary Search's full close-out (rotated arrays, boundary search, on-the-answer feasibility) and Linked Lists' opening (reversal, fast/slow, merging), plus the JVM concurrency model and REST/Spring Boot theory.

---

## Day 29 — Binary Search Continues; Threads and the JVM Concurrency Model

**Q1. Search Insert Position reuses LC 704's exact-match template almost unchanged. What's the one thing that actually differs?** Only the no-match return value — LC 704 returns `-1`; this problem returns `left`, the index where the search loop naturally converges once `left` crosses `right`.

**Q2. Prove why `left` is the correct insertion point when the loop ends.** The loop maintains the invariant that everything before `left` is `< target` and everything after `right` is `>= target`. Since the loop only exits when `right == left - 1`, at that moment the entire array is partitioned exactly at `left` — everything before is smaller, everything from `left` onward is `>=` — which is the definition of the correct insertion point.

**Q3. What's the core insight that lets Find Peak Element binary search an array with no global sort order?** Binary search only needs a rule that safely discards half the search space at each step without discarding the answer — a sorted array is one way to get that, but a purely local comparison (`nums[mid]` vs. `nums[mid+1]`) works too, as long as it's proven to always leave a peak on the kept side.

**Q4. Walk through why an ascending slope at `mid` guarantees a peak strictly to the right.** Walking right from `mid+1`, the sequence either keeps rising until the (virtual `-infinity`) right boundary, making the last real element a peak, or it turns downward somewhere, and that turning point is a peak. Either way a peak exists in `[mid+1, right]`.

**Q5. Why doesn't the array need to have exactly one peak for this to work?** The problem only asks for *any* valid peak, and the proof only ever claims "a peak exists somewhere in this range" — it never claims uniqueness, so multiple peaks don't break the argument or the algorithm.

**Q6. Precisely, what's the difference between a thread and a process?** A process is an independently running program instance with its own memory space, managed by the OS. A thread is one path of execution inside a process; a process can contain multiple threads, and every thread in that process shares the process's memory.

**Q7. Why can two threads corrupt a shared object's fields but never corrupt each other's local `int` variables?** Every thread gets its own private call stack, which is where local primitives live — so two threads' local variables are physically different memory, zero interaction possible. All objects, regardless of which thread created them, live in one heap shared by every thread in the process — so two threads holding references to the same heap object can both modify it, with no guarantee about how those modifications interleave.

**Q8. Why prefer `implements Runnable` over `extends Thread`?** Java allows a class to `extend` only one class. Extending `Thread` permanently spends that slot; implementing `Runnable` costs nothing and leaves the class free to extend whatever its actual type hierarchy calls for, while still being runnable on a thread.

**Q9. What actually happens if you call `.run()` instead of `.start()`?** `run()` executes as an ordinary method call on the *current* thread — no new thread is created, so nothing runs concurrently; the code that looks "multi-threaded" silently behaves as if it were sequential.

**Q10. The JVM's `Thread.State` enum has six values. Name them, and say which of the five-stage conceptual model each maps to.** `NEW` → New. `RUNNABLE` → covers both "Runnable" and "Running" (the JVM doesn't distinguish "eligible" from "actually executing"). `BLOCKED` → waiting specifically for a lock. `WAITING` and `TIMED_WAITING` → waiting on a coordination call, without and with a timeout respectively (both fall under the conceptual model's "Blocked/Waiting"). `TERMINATED` → Terminated.

**Q11. Why does running `ThreadInterleavingDemo` three times produce three different outputs, even though the code never changed?** Which thread the OS scheduler grants CPU time to, at any given moment, depends on factors entirely outside the program's control (system load, scheduling policy, timing). `Thread.sleep()` only guarantees a minimum wait, not what happens immediately after — so the exact interleaving of both threads' print statements isn't fixed by the code.

---

## Day 30 — Binary Search: Rotated Arrays

**Q1. Why is at least one half of a rotated sorted array always guaranteed to be normally sorted, no matter where you split it?** The array is a single ascending sequence cut once and swapped. A contiguous slice only fails to be sorted if it straddles that cut point, and any split at `mid` can put the cut in at most one of the two resulting halves — so the other one is guaranteed clean.

**Q2. What does `nums[left] <= nums[mid]` actually test?** Whether the slice `[left, mid]` is free of the rotation's wraparound. A genuinely sorted slice never has its first element exceed its last, so if this holds, the left half is safely sorted; if it fails, the wraparound must be inside `[left, mid]`, meaning the right half is the sorted one instead.

**Q3. Once you know which half is sorted, what's the second check, and why is it needed?** Whether `target` actually falls within that sorted half's own value range. Knowing a half is sorted doesn't mean the target is in it — the algorithm still has to compare `target` against that half's own min/max before deciding to recurse there.

**Q4. Give a concrete input where trusting `nums[left] <= nums[mid]` without duplicate-handling produces a wrong answer.** `nums = [1, 0, 1, 1, 1]`, `target = 0`: `nums[0] <= nums[2]` is `1 <= 1`, true, incorrectly signaling the left slice `[1,0,1]` is sorted — it isn't — which routes the search away from index 1, where the actual `0` lives, producing a false negative.

**Q5. What's the fix for that failure, and what does it cost?** When `nums[left] == nums[mid] == nums[right]`, neither half can be trusted, so shrink the range from both ends by one and retry. The cost is the complexity guarantee: this fallback can fire on every iteration for an all-duplicate array, degrading from O(log n) average to O(n) worst case.

**Q6. Does the LC 81 version ever return an actually wrong answer, or just a slower one, compared to LC 33's algorithm run on a duplicate-free case?** Just slower in the worst case — the fallback never discards a range that could contain the target, only elements it's already directly confirmed (via the triple-equality check) can't be a useful match at that boundary, so correctness holds throughout; only the time bound is lost.

**Q7. If the array turns out not to be rotated at all (pivot at index 0), what happens to this algorithm?** It degrades gracefully — the "which half is sorted" check finds the entire remaining range sorted at every step, so it behaves identically to plain binary search, with no special-casing needed.

**Q8. Why does LC 81 return a `boolean` while LC 33 returns an `int` index?** That's simply how each problem is specified on LeetCode — with duplicates present, a target's index isn't even well-defined if it's stated to only require existence; the algorithm itself barely changes either way, only the return type and the final `-1`-vs-`false` convention.

---

## Day 31 — Binary Search: Minimum in Rotated Arrays, and Boundary Search

**Q1. Why compare `nums[mid]` to `nums[right]` rather than `nums[left]` in Find Minimum in Rotated Sorted Array?** Because `mid` can equal `left` in a two-element range (comparing a value to itself, losing the signal at exactly the corner case that needs it most), but `mid` never equals `right` given this midpoint formula — so comparing to `right` always compares two genuinely distinct elements.

**Q2. In that same problem, when `nums[mid] > nums[right]`, why is it safe to discard `mid` entirely?** Because that inequality proves the rotation's "drop" happens strictly after `mid` — a normally sorted slice from `mid` to `right` would require `nums[mid] <= nums[right]`, so violating that means the minimum can't be at or before `mid`.

**Q3. What real information does `nums[mid] == nums[right]` fail to give you, in the duplicate version?** It can't distinguish "mid is a high pre-rotation value that happens to numerically match a low post-rotation duplicate at right" from "mid is already inside the low segment alongside right" — both are consistent with the same equality, so no directional decision can be made safely.

**Q4. Why is discarding `right` (not `mid`, not `left`) the safe move in that ambiguous case?** Because an identical value still exists at `mid`, which stays inside the search range — so nothing reachable is lost by dropping the one redundant duplicate at `right`.

**Q5. In Find First and Last Position, why doesn't the algorithm just return immediately on the first match, like ordinary binary search does?** Because a match doesn't guarantee it's the *boundary* match — there could be more occurrences further in the direction being searched, so the match is recorded as a candidate and the search keeps narrowing until that's ruled out.

**Q6. State the exact rule for which direction to narrow after a match, for "first" vs. "last."** Finding the first occurrence: after a match, narrow left (`right = mid - 1`). Finding the last: after a match, narrow right (`left = mid + 1`).

**Q7. Why skip the second search entirely when the first one returns `-1`?** A `-1` from the first search means the target isn't in the array at all — running a second search to find a "last occurrence" of something proven absent would just re-derive the same `-1`, at real (if asymptotically harmless) extra cost.

**Q8. How does Find Smallest Letter Greater Than Target relate to Find First and Last Position?** Same boundary-search shape — one binary search converging on the first index past a threshold — but simplified to a single condition (`<=` moves right, else moves left) instead of two independently-biased searches, since only one boundary is needed here, not both a first and a last.

---

## Day 32 — Binary Search: 2D Matrices, and Binary Search on the Answer

**Q1. What exact structural guarantee makes it valid to treat LC 74's matrix as one flattened sorted array?** Each row is individually sorted, and each row's first value exceeds the previous row's last value — together those two facts mean reading the matrix row by row, left to right, produces one single globally sorted sequence.

**Q2. Given a virtual 1D index `mid` into an `m × n` matrix, how do you recover the real cell?** `row = mid / cols`, `col = mid % cols` — integer division and remainder translate a position in the row-major flattened reading into the actual 2D coordinates.

**Q3. LC 240 looks almost identical to LC 74 but needs a completely different technique. What guarantee is missing?** The cross-row guarantee — LC 240 only promises each row and each column is individually sorted, not that one row's values are entirely above the previous row's. Without that, the row-major flattening isn't globally sorted, so binary-search-as-1D no longer applies.

**Q4. What's the correct technique for LC 240, and what's its complexity?** Start at the top-right corner; move left when the current value is too big (eliminating a whole column), move down when it's too small (eliminating a whole row). O(m+n) — a different complexity class from LC 74's O(log(mn)), not just a different constant.

**Q5. What does "binary search on the answer" mean, and how does Koko Eating Bananas qualify?** Searching over a range of candidate answer values (not array indices) using a monotonic feasibility check at each candidate. Koko qualifies because "can she finish within `h` hours at speed `k`" is monotonic in `k` — any speed faster than a working speed also works.

**Q6. Which earlier problem this series already used this exact shape, and what changed?** Day 28's First Bad Version (LC 278) — same shape, a monotonic boolean condition binary searched over a range. Only the range changed (version numbers → eating speeds) and the condition became a numeric feasibility check instead of a single boolean flag.

**Q7. Why is `hoursNeeded` declared as `long` rather than `int`?** With piles up to `10^9` and up to `10^4` piles, the summed hour count across a worst-case feasibility check can approach `int`'s overflow boundary; since Java's `int` overflow wraps silently rather than throwing, an undetected overflow here would produce a wrong answer with no visible error.

**Q8. Why does the algorithm set `right = mid` (not `mid - 1`) when `mid` turns out to be feasible?** Because `mid` being feasible doesn't rule it out as the *minimum* feasible speed — it needs to stay inside the search range in case nothing smaller also works, exactly mirroring Day 31's Find Minimum logic.

**Q9. Why is `max(piles)`, not `sum(piles)`, the correct upper bound for the search range?** A speed equal to `max(piles)` already guarantees finishing in exactly `n` hours (one full pile per hour, worst case) — no speed larger than that could ever be a smaller, more minimal answer, so the true minimum is always within `[1, max(piles)]`.

---

## Day 33 — Binary Search Capstone: On-the-Answer, Completed and Reviewed

**Q1. What's the one-sentence definition of "binary search on the answer"?** Searching over a range of candidate answer values (not array positions) using a monotonic feasibility function, rather than searching over positions in a given array.

**Q2. What phrasing in a problem statement should make you suspect an "on the answer" problem?** "Minimum/maximum X such that Y is achievable" — the answer being optimized isn't sitting in any array; it has to be searched over as a value.

**Q3. Ship Capacity and Koko Eating Bananas share almost the entire algorithm. What's the one part that's genuinely different between them?** Only the feasibility check's internals — Koko sums ceiling-divided hours per pile; Ship simulates a greedy day-by-day loading process. The outer binary-search-on-a-monotonic-condition shell is identical.

**Q4. Why does Minimum Days to Make m Bouquets need an upfront `-1` check that none of the other on-the-answer problems this week needed?** Because feasibility there can be impossible in principle — if `m × k` exceeds the total number of flowers, no day, however large, ever creates enough of them, since waiting doesn't create new flowers. Every other problem this week is guaranteed feasible for some value in its search range.

**Q5. Why would counting total bloomed flowers and dividing by `k` give a wrong answer for the Bouquets problem?** Because bouquets need `k` *adjacent* bloomed flowers, not just `k` bloomed flowers anywhere — three isolated bloomed flowers with no two adjacent can make zero bouquets of size two, even though naive division would suggest one.

**Q6. What's different about Magnetic Force Between Two Balls compared to every other "on the answer" problem this week?** It maximizes the answer instead of minimizing it — a larger minimum distance is harder to achieve with a fixed number of balls, so the feasible region and the binary search's shrink direction both flip relative to the minimize-type problems.

**Q7. In Magnetic Force, why does a successful feasibility check push `left` up instead of pulling `right` down?** Because the goal is the *largest* feasible distance — on success, the algorithm records the current candidate as the best answer so far and searches strictly above it for something even larger, rather than searching below it for something smaller.

**Q8. Classify Search a 2D Matrix and Koko Eating Bananas: on the input, or on the answer?** Search a 2D Matrix is on the input — the flattened matrix supplies the sorted structure being searched. Koko is on the answer — the search is over eating speeds, using a constructed feasibility check, with no array being searched at all.

**Q9. Across all 15 Binary Search problems from Weeks 4 and 5, roughly how evenly split are the two framings?** Close to even — 9 on the input, 6 on the answer — which is itself worth remembering: "on the answer" isn't a rare exception, it accounts for close to half the pattern's real interview weight.

---

## Day 34 — Linked Lists Begin, and Spring Boot Initialization

**Q1. Mechanically, what is a linked list node's `next` field?** A heap reference — the same mechanism Day 9 already covered for every object reference — pointing to another node, or `null` at the end of the chain. Nothing new is happening in memory; it's the same reference mechanism, used deliberately to chain nodes.

**Q2. Why is array access O(1) but linked list access O(n)?** Array access uses direct address arithmetic (`base + i × elementSize`), needing no traversal. A linked list has no such formula — nodes are scattered wherever the heap allocated them, so reaching node `i` requires walking `i` reference hops from the head.

**Q3. Is "linked list insertion is O(1)" always true?** Only if you already hold a reference to the node you're inserting at. If you first have to *find* that node, the search costs O(n) on a singly linked list (no random access to speed it up), which dominates the O(1) insertion that follows it.

**Q4. In the iterative reversal, why must `curr.next` be saved before it's overwritten?** Because `curr.next` is the only remaining path to the rest of the original list — overwriting it first (`curr.next = prev`) before saving it anywhere permanently disconnects everything after `curr`, with nothing left pointing to it.

**Q5. What's the space complexity of the recursive reversal, and why?** O(n) — one stack frame is held per node until the base case is reached, directly following Day 8's call-stack-depth-equals-space-cost lesson; it's not O(1) just because no explicit data structure is declared.

**Q6. Why does fast/slow only need one pass, while counting-then-jumping needs two?** Fast/slow discovers "halfway" *while* traversing, by advancing two references at different speeds — no upfront count is needed, since the moment `fast` runs out, `slow` has necessarily covered exactly half the distance.

**Q7. Are the two approaches to Middle of the Linked List different in Big-O?** No — both are O(n) time, O(1) space. Fast/slow's advantage is a single pass instead of two, and being the direct foundation for later problems (Palindrome Linked List, cycle detection) — not an asymptotic win.

**Q8. What's the mechanical difference between `@Override` and `@RestController`?** `@Override` is a compile-time-only check with zero runtime effect. `@RestController` (and Spring's other annotations) are read at application startup via reflection — the running program inspecting its own classes and methods — and used to build a routing table connecting HTTP requests to the right methods.

**Q9. Why does REST favor statelessness, and what does it cost?** Statelessness lets any server instance handle any request, since no instance needs to remember a specific client — the basis for straightforward horizontal scaling. The cost is that clients must resend identifying context on every request, rather than relying on a server-side session.

**Q10. What does `ResponseWrapper<T>`'s unbounded `<T>` mean, concretely?** That the class never calls any method specific to `T` — it only ever stores and returns whatever `T` is — so there's no reason to constrain it with a bound; it's Day 16's simplest generics case in practice.

**Q11. Why 404 versus 400 versus 422 — what's the precise distinction?** `400` means the request itself is malformed (bad syntax, missing required fields). `404` means the request is well-formed but the specific resource doesn't exist. `422` means the request is well-formed and the resource concept is valid, but the actual data fails a semantic rule (e.g., a negative price).

---

## Day 35 — Linked Lists Continue, and Week 5 Consolidation

**Q1. Why does Palindrome Linked List's middle-plus-reversal approach need no special case for odd-length lists?** Reversing starting at the middle node sets that node's own `next` to `null`. On odd length, that same node is the last node reached from both the original head's traversal and the reversed second half's traversal — it gets compared to itself (trivially equal), and both pointers hit `null` on the same step.

**Q2. What does the dummy head actually solve?** It removes the need to branch on "is the result list still empty" every time a node is appended — the dummy always gives `tail` something valid to attach to, and the real answer is read off `dummy.next` once the loop finishes.

**Q3. Why would `left == 1` need a special case in Reverse Linked List II without a dummy head?** Because there'd be no real node "before position 1" to serve as `pre` — the dummy head supplies that role uniformly, so reversing from the very head needs no different code path than reversing from anywhere else in the list.

**Q4. In Reverse Linked List II, why doesn't `curr` move during the inner loop?** `curr` represents the fixed tail of the sub-range being reversed — each iteration relocates the node immediately after `curr` to the front of the sub-range instead, which is why `curr` stays put while everything around it changes.

**Q5. Merge k Sorted Lists has two O(N log k) approaches. What's the actual trade-off between them, not just "they're the same Big-O"?** The heap approach needs O(k) extra space for the heap itself; divide-and-conquer needs only O(log k) for its recursion's call stack, and reuses the exact two-list merge routine already in hand rather than needing `PriorityQueue` and a custom `Comparator` at all.

**Q6. Walk through why divide-and-conquer's merging is O(N log k), specifically.** Each round of pairwise merging touches every node across all lists exactly once — O(N) total per round, however that's split across the round's parallel merges — and the number of lists halves each round, giving O(log k) rounds. O(N) per round × O(log k) rounds = O(N log k).

**Q7. What does the plan's "70 total DSA problems" figure actually count, and how does it differ from `00_Curriculum_Map.md`'s own running total?** It's a required-ladder-only count (57 through Week 4, plus 13 required in Week 5), the same convention Week 4's plan used internally for its own "57" checkpoint. The curriculum map's own total (95 through Week 5) additionally counts every extra-practice problem solved beyond the plan's required list.

**Q8. What does Week 6 assume is already solid from this week's theory, and why does it matter concretely?** Day 29's Threads/JVM concurrency model — specifically stack-per-thread, one shared heap — because Week 6, Day 37 builds actual locking (`ReentrantLock`) directly on it, and demonstrates real data corruption from a race condition for the first time, not just the non-deterministic ordering Day 29 showed.
