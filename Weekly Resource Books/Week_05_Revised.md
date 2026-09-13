# Week 5 (Revised): Binary Search Completes, Linked Lists Begins, Spring Boot Initialization

**What changed:** Binary Search closes at 11 core problems (up from 8) — Median of Two Sorted Arrays is deliberately **not** included here; it's scheduled right after Trees complete, exactly as the original plan intended but never actually delivered. Linked Lists grows from 9 problems to 12.

---

## Day 29 — Binary Search Continues, and Threads/JVM Concurrency

### DSA Block (2.5 hrs)
- Problem 3: Search Insert Position — LeetCode #35 — Easy
  - Hint: standard binary search; when the loop ends without a match, `left` is the correct insertion index.
  - Complexity: Time O(log n) | Space O(1)
- Problem 4: Find Peak Element — LeetCode #162 — Medium — Pattern: Binary Search on an unsorted array **(new)**
  - Hint: compare `nums[mid]` to `nums[mid+1]`. If it's rising, a peak exists to the right; if falling, one exists at `mid` or to the left. The array's edges count as `-infinity`.
  - Complexity: Time O(log n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Threads and the JVM's Concurrency Model
- A process can run multiple threads sharing the same heap — exactly why concurrent code can corrupt shared state. Creating a thread: `extends Thread` vs. `implements Runnable` (prefer `Runnable`, since Java only allows single inheritance and you may want to extend something else later). Thread lifecycle: New → Runnable → Running → Blocked/Waiting → Terminated. This is the raw model underneath everything — you'll get to Java's higher-level concurrency tools (`ExecutorService`, and eventually reactive patterns) later, but understanding what's actually happening at the thread level first makes those higher-level tools make sense rather than feel like magic.
- Coding exercise: spawn two threads, each printing numbers 1–5 with a 100ms sleep, and observe the interleaved output.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: `ThreadInterleavingDemo` implementing the exercise above. Run it 3 times and note the output order changes.
- Definition of done: pushed, with a one-line comment on what "non-deterministic" means in this context.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 3 companies with strong engineering blogs.

### Daily Deliverable
- [ ] Search Insert Position and Find Peak Element solved, pushed to `dsa-java/binary-search/`.
- [ ] Can explain the thread lifecycle without notes.
- [ ] `ThreadInterleavingDemo` pushed.

---

## Day 30 — Binary Search: Rotated Arrays

### DSA Block (2.5 hrs)
- Problem 5: Search in Rotated Sorted Array — LeetCode #33 — Medium
  - Hint: one half of the array (split at `mid`) is always properly sorted — figure out which, then check if the target lies within that half's range.
  - Complexity: Time O(log n) | Space O(1)
- Problem 6: Search in Rotated Sorted Array II — LeetCode #81 — Medium — duplicates break the clean "one half is always sorted" guarantee **(new)**
  - Hint: when `nums[left] == nums[mid] == nums[right]`, you can't tell which half is sorted — shrink both ends by one and try again. Worst case degrades to O(n).
  - Complexity: Time O(log n) average, O(n) worst case | Space O(1)

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: none new — write a short note comparing today's two problems: exactly what does one duplicate value break about yesterday's clean binary-search-on-rotated-array logic, and why does the fix cost you the O(log n) guarantee?
- Definition of done: a few sentences, saved alongside your solutions.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 3 companies known for strong backend engineering blogs (these will matter once `todo-api` is live in a couple of days).

### Daily Deliverable
- [ ] Search in Rotated Sorted Array and Search in Rotated Sorted Array II solved, pushed.
- [ ] Comparison note written.

---

## Day 31 — Binary Search: Minimum in Rotated Arrays, and Boundary Search

### DSA Block (2.5 hrs)
- Problem 7: Find Minimum in Rotated Sorted Array — LeetCode #153 — Medium
  - Hint: compare `nums[mid]` to `nums[right]` — if greater, the minimum is to the right of `mid`; otherwise it's `mid` or to the left.
  - Complexity: Time O(log n) | Space O(1)
- Problem 8: Find First and Last Position of Element in Sorted Array — LeetCode #34 — Medium — Pattern: Binary Search (boundary variant)
  - Hint: run binary search twice — once biased to keep narrowing left on a match (first occurrence), once biased right (last occurrence).
  - Complexity: Time O(log n) | Space O(1)

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: comment meaningfully on 5 posts from existing connections.

### Daily Deliverable
- [ ] Find Minimum in Rotated Sorted Array and Find First/Last Position solved, pushed.

---

## Day 32 — Binary Search: 2D and On-the-Answer

### DSA Block (2.5 hrs)
- Problem 9: Search a 2D Matrix — LeetCode #74 — Medium — Pattern: Binary Search (treat as 1D)
  - Hint: if rows are sorted and each row's first element exceeds the previous row's last, treat the grid as one sorted array using index math (`row = idx / n`, `col = idx % n`).
  - Complexity: Time O(log(m×n)) | Space O(1)
- Problem 10: Koko Eating Bananas — LeetCode #875 — Medium — Pattern: Binary Search on Answer
  - Hint: binary search the eating speed `k` from 1 to `max(piles)`; for each candidate, check whether Koko finishes within `H` hours.
  - Complexity: Time O(n log m) | Space O(1)

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: reach out to 1 peer about their interview experience.

### Daily Deliverable
- [ ] Search a 2D Matrix and Koko Eating Bananas solved, pushed.

---

## Day 33 — Binary Search Capstone (On-the-Answer, Continued)

### DSA Block (2 hrs)
- Problem 11: Capacity To Ship Packages Within D Days — LeetCode #1011 — Medium — Pattern: Binary Search on Answer **(new)**
  - Hint: same skeleton as yesterday's Koko problem — binary search the ship's capacity; for each candidate, greedily simulate how many days it takes to ship everything.
  - Complexity: Time O(n log(sum)) | Space O(1)

**This closes Binary Search: 11 problems, Easy through Medium — up from 8 in the original plan. Median of Two Sorted Arrays (LeetCode #4, Hard) is intentionally not here — it's scheduled right after Trees complete, once divide-and-conquer intuition is properly established, which is exactly what the original plan promised on its own Day 21 and then never delivered. Flagging it again now so it doesn't get lost twice.**

### Theory Block (1 hr)
- Topic: Binary Search, Reviewed — "On the Input" vs. "On the Answer"
- Write down, in one sentence, the difference between problems 1–9 (search directly on a sorted array) and problems 10–11 (search on a range of possible *answers*, checking feasibility at each candidate). The second variant is the one people miss cold in interviews — the array isn't sorted, but the *answer space* is monotonic, and recognizing that is the whole skill.

### Project Block (1 hr)
- Repository: none new — make sure all 11 solutions are pushed and organized in `dsa-java/binary-search/`.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 3 companies with strong engineering blogs.

### Daily Deliverable
- [ ] Capacity To Ship Packages Within D Days solved — Binary Search ladder complete at 11 problems.
- [ ] "On the input vs. on the answer" reflection written.

---

## Day 34 — Linked Lists Begin, and Spring Boot Initialization

### DSA Block (2.5 hrs)

**Concept Card — Linked Lists**
- What: a chain of nodes, each holding data and a reference to the next. No contiguous memory requirement, unlike arrays.
- Why: O(1) insertion/deletion at a known position, at the cost of O(n) access to an arbitrary index.
- Where: the literal implementation behind `LinkedList`, and the mental model behind pointer manipulation you'll reuse in trees and graphs.
- Interview signal: "reverse," "merge," "detect a cycle," "find the middle" — usually solvable with a small, fixed number of pointers and O(1) extra space.
- Prerequisites: OOP ✅ (a `ListNode` is just a class with a `val` and a `next` field).

- Problem 1: Reverse Linked List — LeetCode #206 — Easy — Pattern: Iterative Pointer Reversal
  - Hint: keep `prev`, `curr`, `next`. Save `curr.next` before repointing it to `prev`, or you'll lose the rest of the list.
  - Complexity: Time O(n) | Space O(1)
- Problem 2: Middle of the Linked List — LeetCode #876 — Easy — Pattern: Fast/Slow Pointers
  - Hint: `fast` moves two steps for every one of `slow`'s; when `fast` hits the end, `slow` is at the middle.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: REST APIs and Spring Boot Initialization
- HTTP methods (GET/POST/PUT/DELETE) map to CRUD; status codes communicate outcome (2xx/4xx/5xx). REST: resources as URLs, statelessness, using the right verb and status code for what actually happened. Spring exposes all of this through annotations — `@RestController` marks a class as handling HTTP requests and serializing responses to JSON automatically; `@GetMapping`, `@PostMapping`, and similar annotations map a method to a specific HTTP verb and path.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `todo-api` (new).
- Task: initialize via Spring Initializr with Web, JPA, PostgreSQL, and Validation. Create a `/health` endpoint returning JSON via a `ResponseWrapper<T>` (Week 3's generic class).
- Definition of done: the app starts on port 8080; `curl localhost:8080/health` returns 200 OK.

### Career Block (1 hr)
- LinkedIn: Post 8 — "Today I shipped my first Spring Boot endpoint."
- Networking: identify 3 companies known for strong backend engineering blogs.

### Daily Deliverable
- [ ] Reverse Linked List and Middle of the Linked List solved, pushed to `dsa-java/linked-lists/`.
- [ ] `todo-api` initialized, `/health` live.
- [ ] LinkedIn Post 8 published.

---

## Day 35 (Sunday) — Linked Lists Continues, and Week Consolidation

### Self-Check (10 min)
- [ ] Solve one Binary Search problem from this week cold, without hints.

### DSA Block (2 hrs)
- Problem 3: Palindrome Linked List — LeetCode #234 — Easy **(new)**
  - Hint: find the middle (fast/slow, same trick as yesterday), reverse the second half in place, then compare both halves.
  - Complexity: Time O(n) | Space O(1)
- Problem 4: Merge Two Sorted Lists — LeetCode #21 — Easy — Pattern: Two Pointers with Dummy Head
  - Hint: `ListNode dummy = new ListNode(-1)` anchors the result so you never special-case an empty starting list.
  - Complexity: Time O(m+n) | Space O(1)

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 35, five weeks in, **70 total DSA problems solved.** Binary Search fully closed at 11 (up from 8), with Median of Two Sorted Arrays deliberately reserved for right after Trees. Linked Lists underway with 4 of its expanded 11-problem set done. `todo-api` is live with its first endpoint.

### Daily Deliverable
- [ ] Palindrome Linked List and Merge Two Sorted Lists solved, pushed.
- [ ] Weekly ritual and scorecard complete.
