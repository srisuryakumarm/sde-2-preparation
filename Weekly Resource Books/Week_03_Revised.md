# Week 3 (Revised): Sliding Window Completes (14 Problems), and the Leave Week Begins

**What changed:** Sliding Window grows from 10 problems to 14 (Max Consecutive Ones III, Longest Subarray of 1's After Deleting One Element, Longest Substring with At Most K Distinct Characters, and Subarrays with K Different Integers are new). Day 21 is the **first day of the leave week** — flagging this clearly since it's the day to have actually started your leave from work by.

---

## Day 15 — Sliding Window Continues, and HashMap Internals

### DSA Block (2.5 hrs)
- Problem 3: Max Consecutive Ones III — LeetCode #1004 — Medium — Pattern: Sliding Window (variable) **(new)**
  - Hint: the window is valid while it contains at most `k` zeros. Shrink from the left the moment a new zero would push you past `k`.
  - Complexity: Time O(n) | Space O(1)
- Problem 4: Longest Substring Without Repeating Characters — LeetCode #3 — Medium — Pattern: Sliding Window (variable)
  - Hint: track characters in the current window with a HashSet; on a duplicate, shrink from the left until it's gone.
  - Complexity: Time O(n) | Space O(min(m,n))

### Theory Block (2 hrs)
- Topic: HashMap Internals
- Keys are hashed via `.hashCode()` into a bucket array; collisions within a bucket are handled with a linked list, converted to a red-black tree if a single bucket gets too crowded (treeification, since Java 8) to keep worst-case lookups from degrading to O(n). This is why the `.equals()`/`.hashCode()` contract matters — two "equal" objects must return the same hash code, or a HashMap will hide them in different buckets, since it never even looks in the bucket where the "equal" object actually lives.
- Coding exercise: create a class with a broken `hashCode()` (always returns 0) and demonstrate a `HashMap` lookup silently failing to find an object you know you inserted.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `HashCodeContractDemo` class proving the broken-contract failure above, then fixed with a correct `equals()`/`hashCode()` override.
- Definition of done: pushed, with a comment stating the contract rule in your own words.

### Career Block (1 hr)
- LinkedIn: Post 5 — HashMap internals, illustrated with the treeification detail.
- Networking: send 5 connection requests to the Backend Engineers identified yesterday.

### Daily Deliverable
- [ ] Max Consecutive Ones III and Longest Substring Without Repeating Characters solved, pushed to `dsa-java/sliding-window/`.
- [ ] Can explain treeification and the equals/hashCode contract without notes.
- [ ] `HashCodeContractDemo` pushed. LinkedIn Post 5 published.

---

## Day 16 — Sliding Window with Anagrams, and Generics

### DSA Block (2.5 hrs)
- Problem 5: Longest Repeating Character Replacement — LeetCode #424 — Medium — Pattern: Sliding Window (variable) + frequency counting
  - Hint: the window is valid if `(window length) - (count of the most frequent character in it) <= k`. Shrink from the left when it isn't.
  - Complexity: Time O(n) | Space O(1)
- Problem 6: Permutation in String — LeetCode #567 — Medium — Pattern: Sliding Window (fixed size) + frequency counting
  - Hint: maintain a fixed-size window equal to the target string's length; compare frequency arrays instead of re-sorting each window.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Generics
- `<T>` lets a class or method work over any type while keeping compile-time type safety. Bounded wildcards (`? extends T`, `? super T`) control what you can read vs. write through a generic reference. Type erasure: generic type info exists only at compile time — at runtime `List<String>` and `List<Integer>` are both just `List`, with no way to ask a `List` object what type it was declared to hold. This is a deliberate design trade-off (it kept generics compatible with code written before generics existed in Java) with real, concrete consequences: it's why `new T[]` doesn't compile, and why certain reflection tricks that feel like they should work, don't.
- Coding exercise: write a generic `ResponseWrapper<T>` class wrapping any payload with a success flag and a timestamp — you'll reuse this shape once real API work starts.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `ResponseWrapper<T>` class, plus a small generic `Pair<A, B>` class for practice.
- Definition of done: both compile and are demonstrated with at least two different type parameters each; pushed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: comment meaningfully on 5 posts from existing connections.

### Daily Deliverable
- [ ] Longest Repeating Character Replacement and Permutation in String solved, pushed.
- [ ] Can explain type erasure and why `new T[]` doesn't compile.
- [ ] `ResponseWrapper<T>` and `Pair<A, B>` pushed.

---

## Day 17 — Sliding Window with HashMap, and Comparable vs. Comparator

### DSA Block (2.5 hrs)
- Problem 7: Find All Anagrams in a String — LeetCode #438 — Medium — Pattern: Sliding Window + HashMap
  - Hint: identical skeleton to Permutation in String, but collect every valid starting index instead of stopping at the first.
  - Complexity: Time O(n) | Space O(1)
- Problem 8: Minimum Size Subarray Sum — LeetCode #209 — Medium — Pattern: Sliding Window (variable)
  - Hint: expand right to grow the sum; shrink left greedily while the window's sum still meets the target, tracking the minimum length seen.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Comparable vs. Comparator, and Heap-Adjacent Structures
- `Comparable` (`compareTo`) defines a class's single natural ordering. `Comparator` defines an external, swappable ordering — as many as you like for the same class. `TreeMap` (Red-Black tree, O(log n), keys always sorted) and `PriorityQueue` (binary heap, O(log n) insert/poll, O(1) peek) both rely on one or the other.
- Coding exercise: write a `Transaction` record (id, amount, timestamp). Sort a list by amount using a lambda `Comparator`, then re-sort by timestamp descending.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `Transaction` record and the two sort variations above, in a `TransactionSorting` class.
- Definition of done: both sorts print correctly ordered output; pushed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: follow up on any recruiter responses.

### Daily Deliverable
- [ ] Find All Anagrams in a String and Minimum Size Subarray Sum solved, pushed.
- [ ] Can state the difference between Comparable and Comparator in one sentence.
- [ ] `Transaction` sorting exercise pushed.

---

## Day 18 — Sliding Window: At-Most-K-Distinct, and Exception Handling

### DSA Block (2.5 hrs)
- Problem 9: Fruit Into Baskets — LeetCode #904 — Medium — Pattern: Sliding Window (at-most-K-distinct)
  - Hint: this is "longest window with at most 2 distinct values" wearing a costume — track counts of each fruit type, shrink when a third type appears.
  - Complexity: Time O(n) | Space O(1)
- Problem 10: Longest Subarray of 1's After Deleting One Element — LeetCode #1493 — Medium — Pattern: Sliding Window (variable) **(new)**
  - Hint: equivalent to "longest window with at most one zero" — you're always allowed to delete exactly one element, so a window with one zero in it is still valid.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Exception Handling
- Checked exceptions (must be declared or caught) vs. unchecked (`RuntimeException` subclasses, no compiler enforcement — for programmer errors). `try-with-resources` auto-closes anything implementing `AutoCloseable`, eliminating the classic "forgot to close it in `finally`" bug. Whether checked exceptions were a good design decision is a genuinely long-running debate inside the Java community itself — plenty of production codebases lean almost entirely on unchecked exceptions for exactly this reason, and having an opinion on it, backed by a real reason, is worth being able to articulate.
- Coding exercise: create a custom unchecked `CacheMissException`. Write a `try-with-resources` block simulating a file read that logs cleanly on failure.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `CacheMissException` class and the demo above.
- Definition of done: pushed, with a comment on when you'd choose checked vs. unchecked for your own exception types.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: reach out to 2 college alumni at target companies.

### Daily Deliverable
- [ ] Fruit Into Baskets and Longest Subarray of 1's After Deleting One Element solved, pushed.
- [ ] Can explain checked vs. unchecked exceptions and when try-with-resources helps.
- [ ] `CacheMissException` and demo pushed.

---

## Day 19 — Sliding Window: Two-Pointer Hard Tier

### DSA Block (3 hrs) — no new theory today; both problems are Hard-tier and earn the full block
- Problem 11: Longest Substring with At Most K Distinct Characters — LeetCode #340 — Medium — Pattern: Sliding Window + HashMap **(new; LeetCode Premium in some regions — if inaccessible, treat Fruit Into Baskets as the K=2 case and mentally generalize)**
  - Hint: identical shrink-condition logic to Fruit Into Baskets, generalized from "2 distinct" to "K distinct."
  - Complexity: Time O(n) | Space O(k)
- Problem 12: Minimum Window Substring — LeetCode #76 — Hard — Pattern: Sliding Window (variable, two-pointer)
  - Hint: expand right until the window contains every required character; shrink left as far as possible while still valid, recording the smallest valid window found.
  - Complexity: Time O(n+m) | Space O(n+m)

### Project Block (1 hr)
- Repository: `java-fundamentals`.
- Task: none new — use this slot to write a short comparison note: how does Minimum Window Substring's "shrink while still valid" differ from Minimum Size Subarray Sum's (Day 17) "shrink while still valid"? They look identical on the surface; the validity check itself is what differs (a sum threshold vs. a character-coverage check), and being able to articulate that distinction cleanly is exactly the kind of thing an interviewer probes for.
- Definition of done: a few sentences, saved alongside your solutions in `dsa-java/sliding-window/`.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: reach out to 1 peer about their interview experience.

### Daily Deliverable
- [ ] Longest Substring with At Most K Distinct Characters and Minimum Window Substring solved, pushed.
- [ ] Comparison note on the two "shrink while valid" problems written.

---

## Day 20 — Sliding Window Capstone (Monotonic Deque), and Pattern Wrap-Up

### DSA Block (3 hrs)
- Problem 13: Sliding Window Maximum — LeetCode #239 — Hard — Pattern: Monotonic Deque
  - Hint: keep a deque of indices in decreasing order of value; pop from the back while the new element is larger, pop from the front when the front index leaves the window.
  - Complexity: Time O(n) | Space O(k)
- Problem 14: Subarrays with K Different Integers — LeetCode #992 — Hard — Pattern: Sliding Window (exactly-K via at-most-K) **(new, stretch — if you're short on time this specific day, it's fine to revisit this one during a later revision pass rather than solving it cold today)**
  - Hint: "exactly K distinct" = "at most K distinct" − "at most K−1 distinct." Write one helper function for "at most K," call it twice.
  - Complexity: Time O(n) | Space O(k)

**This closes Sliding Window: 14 problems, Easy through Hard — up from 10 in the original plan.**

### Theory Block (1 hr)
- Topic: Sliding Window, Reviewed — Fixed vs. Variable, Side by Side
- Same exercise as Day 13's Two Pointers review: write down which of today's 14 problems used a fixed-size window and which used a variable one, and what specifically in each problem's wording signaled that choice.

### Project Block (1 hr)
- Repository: none new.
- Task: make sure all 14 Sliding Window solutions are pushed and organized in `dsa-java/sliding-window/`.

### Career Block (1 hr)
- LinkedIn: Post 6 — Sliding Window pattern guide (fixed vs. variable window, with a code snippet of each).
- Networking: reach out to 2 college alumni at target companies.

### Daily Deliverable
- [ ] Sliding Window Maximum and Subarrays with K Different Integers solved (or the latter flagged for revisit) — Sliding Window ladder complete.
- [ ] Fixed vs. variable reflection written. LinkedIn Post 6 published.

---

## Day 21 (Sunday) — Leave Week Begins: Prefix Sum & Kadane's Algorithm

**This is Day 1 of the leave week.** If you're taking formal leave from work to cover this, note that Day 21 and Day 27 (the last day of this stretch) fall on what would normally be your weekend anyway — you'd realistically only need to request leave for Days 22–26, five working days, not the full seven.

### Self-Check (10 min)
- [ ] Solve one Sliding Window problem from this week cold, without hints.

### DSA Block (4-5 hrs at leave-week intensity)

**Concept Card — Prefix Sum & Kadane's Algorithm**
- What: precompute cumulative sums (or products) so any range's total is a single subtraction away, instead of a fresh scan every time. Kadane's Algorithm is the running-maximum special case: track the best subarray sum ending *at* each position.
- Why: this pattern is genuinely absent from a lot of DSA prep plans despite being one of the most iconic array techniques in interviews — Maximum Subarray (Kadane's) is arguably *the* canonical array problem.
- Where: any "range sum/product query," "maximum/minimum subarray," or "does some subarray satisfy X" problem.
- Interview signal: "contiguous subarray," "range sum," "maximum sum subarray" — if the brute force is "try every subarray," prefix sum or Kadane's usually collapses it to O(n).
- Prerequisites: arrays ✅, HashMap ✅ (both from Week 1).

- Problem 1: Range Sum Query - Immutable — LeetCode #303 — Easy — Pattern: Prefix Sum **(new)**
  - Hint: precompute `prefix[i] = sum(nums[0..i])`. Any range sum `[i,j]` is `prefix[j] - prefix[i-1]`.
  - Complexity: Time O(1) per query after O(n) preprocessing | Space O(n)
- Problem 2: Maximum Subarray — LeetCode #53 — Medium — Pattern: Kadane's Algorithm **(new)**
  - Hint: at each position, decide whether to extend the previous subarray or start fresh from here — `currentSum = max(num, currentSum + num)`. Track the running max separately.
  - Complexity: Time O(n) | Space O(1)
- Problem 3: Product of Array Except Self — LeetCode #238 — Medium — Pattern: Prefix Product × Suffix Product **(new)**
  - Hint: one pass left-to-right builds prefix products, one pass right-to-left builds suffix products (or fold both into the output array directly). No division allowed — that's the actual point of the problem.
  - Complexity: Time O(n) | Space O(1) excl. output

### Career Block (30 min, leave-week days stay light on career tasks by design)
- LinkedIn: engagement — a quick 10-15 minutes commenting on posts.

### Daily Deliverable
- [ ] Range Sum Query - Immutable, Maximum Subarray, and Product of Array Except Self solved, pushed to `dsa-java/prefix-sum-kadanes/`.
- [ ] Leave week officially underway — no theory/project block distraction today, pure pattern immersion.
