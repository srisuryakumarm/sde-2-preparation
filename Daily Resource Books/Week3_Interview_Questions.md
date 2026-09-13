# Week 3 — Consolidated Interview Question Bank

**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Source books:** [Day 15](Day15_Resource_Book.md) · [Day 16](Day16_Resource_Book.md) · [Day 17](Day17_Resource_Book.md) · [Day 18](Day18_Resource_Book.md) · [Day 19](Day19_Resource_Book.md) · [Day 20](Day20_Resource_Book.md) · [Day 21](Day21_Resource_Book.md)

46 questions across 7 days: Sliding Window's full closing arc (Days 15–20, including its four extra-practice problems), HashMap Internals, Generics, Comparable/Comparator + TreeMap/PriorityQueue, Exception Handling, and the opening of Prefix Sum & Kadane's (Day 21).

**How to use this:** work a day at a time, cold, before checking against the answer — the value is in retrieving the *reasoning*, not recognizing the answer once you see it. For the proof-style questions (Day 16 Q1, Day 20 Q1/Q3, Day 21 Q2/Q3), being able to say the argument out loud in your own words is the actual bar, not matching the wording below.

---

## Day 15 — Variable-Size Sliding Window, HashMap Internals

**Q1. Walk me through what happens, mechanically, when you call `map.get(key)`.**
A: Compute `key.hashCode()`, spread it (`h ^ (h>>>16)`), mask with `(n-1)` to get a bucket index, then walk that bucket's chain (list or tree) comparing `key.equals()` against each entry until a match is found or the chain is exhausted.

**Q2. Why does the bucket-index formula use `(n-1) & hash` instead of `hash % n`?**
A: They're equivalent whenever `n` is a power of two, but the bitwise AND is cheaper than a modulo — which is exactly why HashMap enforces power-of-two capacities.

**Q3. When does a bucket treeify, and into what?**
A: When that specific bucket's chain reaches 8 entries (`TREEIFY_THRESHOLD`) **and** the table's overall capacity is at least 64 (`MIN_TREEIFY_CAPACITY`); it converts from a linked list of `Node`s to a red-black tree. Below capacity 64, the table resizes instead of treeifying.

**Q4. If `hashCode()` always returns the same constant, does `HashMap.get()` still return the correct value?**
A: Yes — every key lands in the same bucket, so the map degrades to a single long chain, but `.equals()` still correctly identifies the right entry during the scan. It's a performance bug (toward O(n)), not a correctness bug.

**Q5. Give an example where `get()` returns null for a key you know is logically present.**
A: Override `.equals()` to compare by value but leave `.hashCode()` unoverridden (or inconsistent) — two "equal" instances get different identity-based hash codes, land in different buckets, and `get()` on one instance never even looks in the bucket where the other lives.

**Q6. Why is the total runtime of a variable-size sliding window O(n) and not O(n²), given the nested loop?**
A: `right` advances at most n times total; `left` only ever moves forward and also advances at most n times total across the *entire* run (not per outer iteration) — so total work is O(n) + O(n) = O(n), regardless of the syntactic nesting.

**Q7. What's the concrete signal in a problem statement that tells you "variable-size window," as opposed to fixed-size?**
A: Wording like "longest/shortest subarray such that…" or "at most k…" — the window's size itself is what you're solving for, versus fixed-size wording like "of size k."

**Q8. In Max Consecutive Ones III, why do you only decrement `zeroCount` when the element leaving the window was a zero?**
A: `zeroCount` tracks how many zeros are *currently inside* the window; removing a 1 from the window doesn't change that count, so decrementing unconditionally would undercount.

**Q9. In Longest Substring Without Repeating Characters, why is a single shrink step (versus a while-loop of shrinks) sometimes not enough?**
A: If the window contains other characters between the old occurrence of `c` and the current position, they need to be evicted one at a time until the specific blocking duplicate is gone — that can take more than one removal, which is exactly why it's a `while`, not an `if`.

---

## Day 16 — Anagram-Style Windows, Generics

**Q1. In Longest Repeating Character Replacement, why is it safe for `maxFreq` to go stale after a shrink?**
A: Because the question asks for the *longest* valid window — once a window of length L is known achievable, any shorter window is irrelevant, so the algorithm never needs to "notice" the window has become momentarily invalid at a smaller size; it only needs to detect when a strictly larger `maxFreq` unlocks a strictly longer valid window.

**Q2. Why is the shrink in LC 424 an `if`, not a `while`, when Day 15's template used a `while`?**
A: At most one violation can accumulate per step here, since `windowLen` and `maxFreq` both change by exactly one unit per iteration — a single shrink always restores the (stale) validity check.

**Q3. What's the difference between what Permutation in String and Longest Substring Without Repeating Characters (Day 15) each track in their window?**
A: LC 567 tracks a fixed-size window's exact frequency profile, comparing for equality against a target; LC 3 tracks a variable-size window's membership only (a HashSet), growing as long as there's no duplicate.

**Q4. What does type erasure actually erase, and when?**
A: Generic type parameter information — it exists during compilation (for type-checking and automatic cast insertion) and is discarded before/at bytecode generation; at runtime, `List<String>` and `List<Integer>` are indistinguishable, both just `List`.

**Q5. Why doesn't `new T[]` compile?**
A: Java arrays are reified (they track and enforce their component type at runtime), but erasure means the JVM has no concrete type for `T` at runtime — there's nothing to give the array to create it correctly.

**Q6. State PECS and apply it: would you use `? extends` or `? super` for a method parameter that only ever calls `.add()` on the list you're given?**
A: Producer Extends, Consumer Super. A method that only adds is *consuming* values into the list, so `? super T`.

**Q7. Why can't you overload `process(List<String>)` and `process(List<Integer>)` in the same class?**
A: After erasure both become `process(List)` — identical signatures, which the compiler rejects as a duplicate method.

---

## Day 17 — Anagram Collection, Minimum Windows, Comparable/Comparator

**Q1. How does Find All Anagrams in a String differ from Permutation in String, mechanically?**
A: Nothing about the sliding-window mechanism changes — only what happens on a match: return `true` immediately versus record the index and keep scanning.

**Q2. Explain the structural difference between "shrink while valid" and "shrink until valid."**
A: "Shrink until valid" (Days 15–16) shrinks only when the window is currently *invalid*, to restore validity, and records the answer once validity is restored — it's solving for the *longest* valid window. "Shrink while valid" (Day 17 onward) shrinks precisely *because* the window is currently valid, recording the answer at every step of the shrink, stopping only once shrinking would break validity — it's solving for the *shortest* valid window.

**Q3. State the difference between `Comparable` and `Comparator` in one sentence.**
A: `Comparable` defines a class's one built-in natural ordering (inside the class, via `compareTo`); `Comparator` defines any number of external, swappable orderings (outside the class, via `compare`).

**Q4. Why is `(a, b) -> a.getAmount() - b.getAmount()` a risky Comparator, even though it often "works"?**
A: It risks integer overflow for large-magnitude values, and for floating-point fields, implicit narrowing to the `int` a Comparator must return can truncate and misorder close values; `Type.compare(a, b)` avoids both failure modes.

**Q5. Why is `TreeMap`'s `get()` O(log n) while `HashMap`'s is O(1) average — and what do you get in exchange for the slower guarantee?**
A: `TreeMap` is a balanced binary search tree, so any operation walks one root-to-leaf path bounded by O(log n); `HashMap` computes a bucket directly. In exchange, `TreeMap` keeps keys in sorted order at all times and supports range/ordered queries `HashMap` cannot offer at any speed.

**Q6. Why is `PriorityQueue.peek()` O(1) but `poll()` is O(log n)?**
A: The minimum is always sitting at array index 0 by the heap invariant, so peeking is a direct array read. Removing it requires promoting a new root (swap in the last element, then sift it down to restore the heap invariant), which costs up to the tree's height in swaps.

**Q7. Why does Day 17's `Transaction` use a hand-written class instead of `record`?**
A: `record` isn't taught until Week 4, Day 28 — using it here would use syntax before its prerequisite lesson. The hand-written version does the same job and previews exactly what `record` will later automate.

---

## Day 18 — At-Most-K-Distinct Windows, Exception Handling

**Q1. Fruit Into Baskets is secretly which more general pattern?**
A: "Longest window with at most K distinct values," with K fixed at 2.

**Q2. In Fruit Into Baskets, why remove a key from the map entirely once its count hits zero, rather than leaving it at 0?**
A: `basket.size()` is what drives the validity check — a lingering zero-count key would still count toward `size()`, making the window look invalid when it's actually fine.

**Q3. Why must you subtract 1 from the "at most one zero" window length in Longest Subarray of 1's After Deleting One Element, even for an all-ones array?**
A: The deletion is mandatory regardless of whether a zero is available — if there's a zero, delete it (−1 from the window); if there isn't, you still must delete something, one of the 1s (also −1). Either way the answer is `window length − 1`.

**Q4. Draw (or describe) where `StackOverflowError` sits in the `Throwable` hierarchy, and why that placement matters.**
A: `Throwable → Error → StackOverflowError`. It's an `Error`, not a `RuntimeException`, meaning the JVM doesn't expect it to be routinely caught and handled — it signals a serious runtime condition (stack exhaustion), not an ordinary programmer error.

**Q5. What's the actual bug `try-with-resources` fixes, versus manual `try/finally`?**
A: If both the try block and the cleanup code throw, manual `finally` lets the cleanup's exception silently replace the original, losing the real cause; `try-with-resources` instead propagates the original exception and attaches the cleanup's exception as suppressed, preserving both.

**Q6. Give one argument for checked exceptions and one against.**
A: For: they force callers to explicitly acknowledge conditions the API author knows can fail. Against: in practice they push toward boilerplate (empty catches, blanket `throws Exception`) and interact poorly with functional-style/lambda code.

**Q7. In Frequency of the Most Frequent Element, why sort the array first?**
A: Sorting means a window's maximum is always its rightmost element, and since you can only increment (never decrement) values, the cheapest target for a window is always that window's own maximum — sorting is what makes "the rightmost element" and "the maximum" the same thing.

---

## Day 19 — Hard-Tier Windows (No New Theory)

**Q1. How does Longest Substring with At Most K Distinct Characters relate to Fruit Into Baskets?**
A: It's the exact same algorithm with `K` as a parameter instead of a hardcoded `2` — Fruit Into Baskets is the `K=2` special case.

**Q2. In Minimum Window Substring, why does `formed` use `==` rather than `>=` when checking whether to increment?**
A: `>=` would re-increment `formed` every time an already-satisfied character is seen again (e.g., a third `'A'` when only two were required), double-counting a requirement that was already met and corrupting the `formed == required` check.

**Q3. What's identical, and what's different, between Minimum Size Subarray Sum and Minimum Window Substring?**
A: Identical: the shrink-while-valid loop shape, since both are "minimum window" problems. Different: the validity check — a single numeric sum threshold versus a multi-character coverage count built from `required`/`formed`.

**Q4. Why is Minimum Window Substring's space complexity O(n + m) and not O(1)?**
A: Unlike the fixed-26-letter frequency arrays used earlier this week, `need` and `window` here are general character maps sized by the actual distinct characters present in `t` and `s` respectively — not bounded by a small constant alphabet in the general case.

**Q5. What would you say out loud, before coding, to signal you recognize Minimum Window Substring's pattern immediately?**
A: "This is shrink-while-valid, like a minimum-window sum problem, but validity here means full character coverage — I'll track that with a required/formed counter pair instead of comparing frequency maps directly each step, to keep each check O(1)."

---

## Day 20 — Monotonic Deque, Pattern Close-Out

**Q1. Why is it safe to permanently discard a value from the back of the monotonic deque when a larger (or equal) one arrives?**
A: The discarded value is dominated — the new value is both more recent (stays in the window at least as long) and at least as large, so the discarded value can never be the maximum of any window that still contains the new one.

**Q2. Why does Sliding Window Maximum store indices in the deque, not values?**
A: Expiration depends on *position* (has this index fallen outside the current window?), which a raw value can't tell you — you need the index to compare against the window's boundary.

**Q3. Derive "exactly K distinct" from "at most K distinct."**
A: `exactly(K) = atMost(K) − atMost(K−1)`, since "at most K−1" is a strict subset of "at most K," and the difference is precisely the subarrays whose distinct count is greater than K−1 and at most K — i.e., exactly K.

**Q4. In the at-most-K helper used for counting, why does `count += right - left + 1` correctly count subarrays rather than just track a window length?**
A: For a fixed `right`, every subarray `[left', right]` with `left' ≥ left` is also valid (shrinking the start can only remove distinct values, never add one), so the number of valid subarrays ending at `right` equals the number of valid starting points, `right - left + 1`.

**Q5. What's the one concrete signal that tells you a Sliding Window problem is fixed-size rather than variable-size?**
A: A number given directly as the window's size in the problem statement ("of size k," "a permutation of," "for k minutes") signals fixed; a constraint to satisfy ("at most k," "longest/shortest such that") signals variable, since the size itself is what's being solved for.

**Q6. Why does Longest Subarray With Absolute Diff ≤ Limit need two deques instead of one?**
A: Its validity check depends on both the window's maximum and minimum simultaneously (`max − min ≤ limit`); a single monotonic deque only gives you one extreme at a time.

---

## Day 21 — Prefix Sum & Kadane's Opens

**Q1. Why does the prefix sum array use `prefix[i] = sum of the first i elements` (with `prefix[0]=0`), rather than aligning `prefix[i]` directly to `nums[i]`?**
A: The `prefix[0]=0` convention means a range starting at index 0 needs no special case — `sumRange(0, j) = prefix[j+1] - prefix[0] = prefix[j+1] - 0`, which is already correct without a branch.

**Q2. Prove Kadane's recurrence — don't just state it.**
A: Let `best[i]` be the max sum of any subarray ending exactly at `i`. Such a subarray either is `nums[i]` alone, or extends the best subarray ending at `i-1` (extending anything less than the best there can only give an equal-or-smaller result after adding the same `nums[i]`). So `best[i] = max(nums[i], best[i-1] + nums[i])`, and the answer is the max of `best[i]` over all `i`.

**Q3. Why doesn't Kadane's need a special case for all-negative arrays?**
A: The recurrence compares the extension against `nums[i]` alone, never against zero — so it never treats "keep a negative running sum" as better than "restart," and correctly converges to the single largest (least negative) element rather than an invalid empty-subarray sum of zero.

**Q4. Why is division disallowed in Product of Array Except Self, precisely?**
A: `total / nums[i]` breaks whenever any element is zero (undefined), and can't correctly express the case of two or more zeros (every output should be 0) — it's not an arbitrary rule, it rules out an approach that's unsound on valid input.

**Q5. Why is the two-pass prefix/suffix technique O(1) extra space, when it still touches every element twice?**
A: The suffix pass folds directly into the existing `output` array via one running variable (`suffixProduct`), rather than allocating a second full-size array — only the required output array and O(1) extra bookkeeping are used.

---

*46 questions total. Cross-reference: [00_Curriculum_Map.md](00_Curriculum_Map.md) for the full problem inventory and concept index this bank draws from.*
