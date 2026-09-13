# Day 56 Resource Book — Consolidation, and Heaps: Frequency and Scheduling

**Series:** SDE-2 Interview Prep Resource Books (Week 8) · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 55](./Day55_Resource_Book.md) · **Next ▶:** Day 57 (Week 9)
**Companion to:** Day 56 of `Week_08_Revised.md`

---

## Recap: what today is

Today closes Week 8 — two more required Heap problems, a self-check, and the full week's consolidation. Top K Frequent Elements is the canonical "count with a HashMap, then bound with a heap" combination this week's problems have been building toward without naming it as its own explicit shape. Ugly Number II is a different kind of heap problem from anything else this week — instead of being handed a fixed input to extract the top/bottom k from, it **generates its own candidates**, pulling the heap's smallest-value-in-sorted-order guarantee (Day 54's mechanism) into a candidate-generation loop rather than a fixed-collection query.

One optional extra today, kept deliberately light: Sort Characters by Frequency is Top K Frequent's direct sibling — same technique, different output shape — marked as skippable given Sunday's shorter schedule.

## Learning Objectives

By the end of today, without notes:

1. Solve Top K Frequent Elements with a HashMap-plus-heap combination, and explain the O(n)-via-bucket-sort alternative and when it's worth reaching for over the heap.
2. Solve Ugly Number II, explaining precisely why generating candidates from confirmed-ugly numbers is correct where testing arbitrary integers for "ugliness" is not just slower but structurally wasteful.
3. Recite Heaps' status at the end of Week 8 — required count, extra count, and what's still to come in Week 9.
4. State Trees' full closure and Heaps' current progress as part of a coherent account of everything this week actually built.

## Concept Dependency Map for Today

```
Day 4/5: HashMap frequency counting
Day 54-55: heap-of-size-k (multiple variants this week)
        │
        ▼
Problem 5: Top K Frequent Elements (LC 347)
  needs: HashMap frequency count + min-heap-of-size-k, the two combined explicitly for the first time

Day 54: heap mechanism — root is always the current min, by construction
        │
        ▼
Problem 6: Ugly Number II (LC 264)
  needs: heap as a CANDIDATE GENERATOR, not a fixed-collection query — new usage of the same mechanism

Extra Practice (optional, light): Sort Characters By Frequency (LC 451)
  needs: today's Problem 5 skeleton, mirrored to a different output shape

Self-Check: one Tree problem from this week, cold
        │
        ▼
Week 8 Consolidation
```

---

# Part 1 — Top K Frequent Elements

## Problem 5: Top K Frequent Elements (LeetCode 347, Medium) — Pattern: HashMap + Heap

**Statement:** Given an integer array and an integer `k`, return the `k` most frequent elements.

### Approach 1 — HashMap + min-heap of size k

```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freqMap = new HashMap<>();
    for (int num : nums) {
        freqMap.put(num, freqMap.getOrDefault(num, 0) + 1);
    }

    PriorityQueue<Map.Entry<Integer, Integer>> minHeap = new PriorityQueue<>(
        new Comparator<Map.Entry<Integer, Integer>>() {
            @Override
            public int compare(Map.Entry<Integer, Integer> a, Map.Entry<Integer, Integer> b) {
                return a.getValue() - b.getValue();   // order by FREQUENCY, not by the element's own value
            }
        }
    );

    for (Map.Entry<Integer, Integer> entry : freqMap.entrySet()) {
        minHeap.offer(entry);
        if (minHeap.size() > k) {
            minHeap.poll();   // evict the currently-least-frequent kept entry
        }
    }

    int[] result = new int[k];
    for (int i = k - 1; i >= 0; i--) {
        result[i] = minHeap.poll().getKey();
    }
    return result;
}
```

**The combination, named explicitly:** count with a HashMap (Day 4/5's frequency-counting pattern), then bound with a heap-of-size-k (this week's recurring shape) — ordered not by an element's own value, but by its **computed frequency**, a second custom-comparator instance following directly from yesterday's distance-based comparator.

**Complexity: Time O(n log k)** — O(n) to build the frequency map, O(d log k) for the heap operations where `d` is the count of distinct elements (`d ≤ n`). **Space O(n)** for the map.

### Approach 2 — An even better approach: bucket sort

**The insight the heap approach doesn't exploit:** frequency in an array of length `n` is bounded — no element can appear more than `n` times. That bound means frequencies can be used directly as array indices, avoiding a heap (and its `log k` factor) entirely.

```java
public int[] topKFrequentBucket(int[] nums, int k) {
    Map<Integer, Integer> freqMap = new HashMap<>();
    for (int num : nums) {
        freqMap.put(num, freqMap.getOrDefault(num, 0) + 1);
    }

    List<Integer>[] buckets = new List[nums.length + 1];   // index = frequency, value = elements with that frequency
    for (Map.Entry<Integer, Integer> entry : freqMap.entrySet()) {
        int freq = entry.getValue();
        if (buckets[freq] == null) {
            buckets[freq] = new ArrayList<>();
        }
        buckets[freq].add(entry.getKey());
    }

    int[] result = new int[k];
    int idx = 0;
    for (int freq = buckets.length - 1; freq >= 0 && idx < k; freq--) {
        if (buckets[freq] != null) {
            for (int num : buckets[freq]) {
                result[idx++] = num;
                if (idx == k) break;
            }
        }
    }
    return result;
}
```

**Complexity: Time O(n), Space O(n)** — strictly better than the heap approach's O(n log k) on paper.

**Which to actually reach for in an interview:** the heap approach generalizes more naturally — it extends cleanly to a *streaming* version of this problem (new elements arriving continuously, `k` most frequent needed at any moment), where bucket sort's "know the max possible frequency up front" assumption breaks down. Bucket sort is the better answer when the full input is known up front and squeezing out the log-factor genuinely matters. Naming both, and the trade-off between them, is a stronger answer than presenting only one as "the" solution.

**Edge cases:** `k` equal to the number of distinct elements (every distinct value gets returned); all elements identical (a single bucket at the maximum frequency, holding one element); `k = 1` (both approaches degrade gracefully to "find the single most frequent element").

---

# Part 2 — Heap as a Candidate Generator

## Problem 6: Ugly Number II (LeetCode 264, Medium) — Pattern: Min-Heap Generating Candidates in Order

**Statement:** An "ugly number" is a positive integer whose only prime factors are 2, 3, and 5 (`1` counts as ugly by convention). Return the nth ugly number.

### Approach 1 — Brute force: test every integer

```java
public int nthUglyNumberBruteForce(int n) {
    int count = 0;
    long candidate = 0;
    while (count < n) {
        candidate++;
        if (isUgly(candidate)) {
            count++;
        }
    }
    return (int) candidate;
}

private boolean isUgly(long num) {
    for (int factor : new int[]{2, 3, 5}) {
        while (num % factor == 0) {
            num /= factor;
        }
    }
    return num == 1;
}
```

Test every positive integer in order, dividing out factors of 2, 3, and 5 repeatedly; if only `1` remains, it's ugly. **This is not just slow, it's structurally wasteful** — ugly numbers thin out rapidly as numbers grow (the nth ugly number is generally much larger than `n` itself), meaning this approach spends most of its work testing integers that turn out **not** to be ugly at all, for no benefit.

### Approach 2 — Optimized: generate candidates, don't test them

**The reframe:** rather than testing arbitrary integers, generate **only** numbers that are provably ugly, directly — every ugly number greater than 1 is exactly `2×`, `3×`, or `5×` some smaller ugly number. Starting from the one known ugly number (`1`) and repeatedly multiplying every confirmed-ugly value by 2, 3, and 5 generates every subsequent ugly number, with nothing wasted testing non-candidates.

```java
public int nthUglyNumber(int n) {
    PriorityQueue<Long> minHeap = new PriorityQueue<>();
    Set<Long> seen = new HashSet<>();
    minHeap.offer(1L);
    seen.add(1L);

    long ugly = 1;
    int[] factors = {2, 3, 5};

    for (int i = 0; i < n; i++) {
        ugly = minHeap.poll();
        for (int factor : factors) {
            long next = ugly * factor;
            if (!seen.contains(next)) {
                seen.add(next);
                minHeap.offer(next);
            }
        }
    }
    return (int) ugly;
}
```

**Why the heap, specifically:** multiplying a given ugly number by 2, 3, and 5 doesn't produce the *next* ugly number directly in order — it produces three candidates that need to be considered alongside every other pending candidate from every earlier ugly number. A min-heap is exactly the structure that always surfaces the smallest pending candidate next, in O(log(heap size)) — this is the heap's "root is always the current minimum" guarantee (Day 54's mechanism) applied to a growing pool of self-generated candidates, rather than a fixed input collection.

**Why the `seen` set is required, not optional:** the same ugly number is frequently reachable multiple ways — `6 = 2×3 = 3×2`. Without deduplication, the heap accumulates duplicate entries, and popping a duplicate wastes an iteration without producing a genuinely new nth value, silently under-counting toward `n`.

**Worked trace, `n = 10`** (expected sequence: `1,2,3,4,5,6,8,9,10,12` — note `7` is skipped, since 7 isn't divisible by only 2/3/5):

| i | popped (ugly) | new candidates generated |
|---|---|---|
| 0 | 1 | 2, 3, 5 |
| 1 | 2 | 4, 6, 10 |
| 2 | 3 | 9, 15 (6 already seen) |
| 3 | 4 | 8, 12, 20 |
| 4 | 5 | 25 (10, 15 already seen) |
| 5 | 6 | 18, 30 (12 already seen) |
| 6 | 8 | 16, 24, 40 |
| 7 | 9 | 27, 45 (18 already seen) |
| 8 | 10 | 50 (20, 30 already seen) |
| 9 | **12** | ... |

After the 10th iteration (`i=9`), `ugly = 12` — the 10th ugly number. Matches the expected sequence exactly.

**Complexity: Time O(n log n)** — n iterations, up to 3 heap operations each, heap size bounded by O(n) across the run (at most `3n` total insertions), giving `O(log n)` per operation. **Space O(n)** for the heap and the `seen` set.

**Edge cases:** `n = 1` (returns `1` immediately, no multiplication needed); the overflow risk this problem shares with Day 10/11's habit — candidate values can approach or exceed `Integer.MAX_VALUE` during generation even though the *final answer* is guaranteed to fit within a 32-bit int for this problem's constraints, which is exactly why heap entries and the running candidate value are typed `long`, not `int`, throughout.

> ⚠️ **Common Mistake:** omitting the `seen` set to "simplify" the code. It compiles, it runs, and it silently produces a wrong answer for any `n` large enough that a duplicate candidate gets generated before the true nth value is reached — a correctness bug, not a performance one.

---

# Part 3 — Extra Practice (Optional, Light)

## Extra Practice 1: Sort Characters By Frequency (LeetCode 451, Medium) — Pattern: HashMap + Heap, Frequency Output

**Statement:** Given a string, sort its characters in decreasing order of frequency, returning the sorted string (any valid ordering among equal-frequency characters is accepted).

**Marked optional today, deliberately:** Sunday's DSA block is shorter, and this is a direct sibling of Problem 5 above — same HashMap-plus-heap combination, ordered by frequency, just producing a different output shape (a full reconstructed string instead of a fixed top-k list). If time is tight, understanding *why* it's the same underlying technique is worth more than grinding through the full implementation — but the code is here if there's room for it.

```java
public String frequencySort(String s) {
    Map<Character, Integer> freqMap = new HashMap<>();
    for (char c : s.toCharArray()) {
        freqMap.put(c, freqMap.getOrDefault(c, 0) + 1);
    }

    PriorityQueue<Character> maxHeap = new PriorityQueue<>(new Comparator<Character>() {
        @Override
        public int compare(Character a, Character b) {
            return freqMap.get(b) - freqMap.get(a);
        }
    });
    maxHeap.addAll(freqMap.keySet());

    StringBuilder sb = new StringBuilder();
    while (!maxHeap.isEmpty()) {
        char c = maxHeap.poll();
        int freq = freqMap.get(c);
        for (int i = 0; i < freq; i++) {
            sb.append(c);
        }
    }
    return sb.toString();
}
```

**Complexity: Time O(n + d log d)** where `d` is the count of distinct characters (`d ≤ n`) — O(n) to count, O(d log d) to heap-sort the distinct characters, O(n) to rebuild the output string. **Space O(n).**

**Edge cases:** a string of all-identical characters (single heap entry, output is the original string unchanged); an empty string (returns empty immediately).

---

# Part 4 — Self-Check (15 minutes)

**Solve one Tree problem from earlier this week, cold, without hints.** Validate Binary Search Tree (bounds-passing DFS) and Lowest Common Ancestor of a Binary Tree, general (postorder combine, no ordering invariant) are the two strongest candidates — they're the week's two most structurally distinct Tree techniques, so successfully re-deriving either from a blank editor is a genuinely different signal than re-deriving, say, another BST problem that leans on the same bounds-passing idea you'd have just used.

**Why cold recall specifically, not just re-reading the solution:** recognizing an approach when it's shown to you and independently producing it under blank-page conditions test different things — an interview is the second one, not the first. If either problem doesn't come back cleanly without notes, that's real, useful information about where this week's material hasn't fully settled yet — better to find that out now than under real interview pressure.

---

# Section — Career Block

**Weekly Industry Awareness Ritual (20 minutes):** clear the TLDR Newsletter backlog, read one engineering blog post.

**Weekly Scorecard — Day 56, eight weeks in:**

Matching the plan's own required-problems-only convention: **110 total DSA problems solved** (97 through Week 7 + 13 required this week — 7 closing Trees, 6 opening Heaps). Trees is fully closed at 15 problems, up from 11 in the original plan, and the Median of Two Sorted Arrays gap the original plan's Day 21 left open is now genuinely closed rather than just promised. Heaps is 6 problems into its expanded 10, already meaningfully past where the original plan's thin 6-problem treatment stopped entirely.

**Tracking every extra practice problem alongside the required ladder** (the fuller count this series has kept since Week 1): Week 8 added **18 newly-solved problems** — 13 required + 5 extra (BST Iterator, Construct Binary Tree from Inorder/Postorder, Minimum Cost to Connect Sticks, Kth Smallest Element in a Sorted Matrix, Sort Characters by Frequency). **Cumulative distinct problems solved, Weeks 1–8: 145.**

---

# Week 8 Consolidation

## What actually got built

- **Trees closed** at 15 required + 5 extra = 20 distinct problems total, spanning BST traversal and validation, tree construction via divide-and-conquer, general-tree LCA, and serialization — the full pattern now closed across Weeks 7 and 8 combined.
- **Heaps opened**, reaching 6 of 10 required problems (Kth Largest Element in a Stream, Last Stone Weight, Kth Largest Element in an Array, K Closest Points to Origin, Top K Frequent Elements, Ugly Number II) + 3 extra (Minimum Cost to Connect Sticks, Kth Smallest Element in a Sorted Matrix, Sort Characters by Frequency) = 9 distinct so far, continuing into Week 9.
- **Two old debts resolved:** Median of Two Sorted Arrays, promised on the original plan's Day 21 and never delivered across its remaining 99 days, is now genuinely solved. Quickselect, named as a future destination back on Day 12 ("resurfaces with Quickselect later"), showed up for real on Day 55.
- **`todo-api` gained:** a Kafka consumer (`@KafkaListener`, completing the produce-consume loop Day 48 started), environment-specific configuration via Spring profiles (`dev`/`prod`), and a WireMock stub laying groundwork for Resilience4j, still a few weeks out.
- **One plan discrepancy flagged, not silently absorbed:** Day 55's title referenced a "Spring Cloud Config Wrap-Up" that the plan's own body never actually specified — handled by following the body (no new theory that day) rather than inventing content the plan didn't actually call for.

## Planned vs. actual

| | Plan's own required-ladder count | This series' full count (required + extra) |
|---|---|---|
| Through Week 7 | 97 | 127 |
| Week 8 alone | 13 | 18 |
| Through Week 8 | **110** (matches the plan's own Day 56 claim exactly) | **145** |

## Diagnostic — solid vs. worth revisiting

**Should be fully reflexive without notes:**
- The BST ordering invariant, and precisely why it lets LCA compute a single direction while validation needs bounds threaded through both.
- Divide-and-conquer's three steps, and specifically what made Construct Binary Tree's divide step new (computed, not free) versus every earlier tree DFS problem.
- The heap mechanism end to end — array index math, sift-up, sift-down, and the unconditional-vs-conditional O(log n) contrast with a bare BST.
- Quickselect's partition step, its O(n)-average-vs-O(n log n) distinction from quicksort, and its worst case plus mitigation.

**Worth a second pass if any of these felt shaky:**
- The Median of Two Sorted Arrays partition proof — specifically, being able to state both correctness conditions (size, value) without looking them up.
- The cost-accounting exchange argument for why Minimum Cost to Connect Sticks combines smallest-first, in contrast to Last Stone Weight's largest-first rule.
- The staircase-count mechanism from Kth Smallest Element in a Sorted Matrix, and its connection back to Day 32's unfinished LC 240 mention.

## What Week 9 assumes

Week 9 closes Heaps (4 more required problems: Reorganize String, Task Scheduler, Find Median from Data Stream, Merge k Sorted Lists — Days 57–58), closes Tries' core 6 problems, opens Backtracking, and initializes `scalable-ecommerce-platform`, the flagship project, five weeks earlier than the original plan had it. It assumes this week's full heap mechanism — not just "know how to call `PriorityQueue`," but the array-backed structure and why its complexity bound is unconditional — is completely solid, since none of Week 9's four remaining Heap problems re-derive it from scratch. It assumes recursion (Day 8) is fully reflexive for two entirely new structures: Tries (a new self-referential shape, each node holding up to 26 children rather than 2) and Backtracking (explore-recurse-undo, with no canonical "Easy" problem to ease into — Day 59's plan opens by hand-tracing a 3-level decision tree before any real code). It assumes `todo-api`'s Docker, Docker Compose, and full testing stack (JUnit 5, AssertJ, Mockito, TestContainers) — all established Week 7, untouched this week — are stable, since Week 9 adds no further infrastructure changes to `todo-api` at all; from Day 62 onward, active project work moves entirely to the new flagship platform, with `todo-api` staying exactly as it is, complete and no longer receiving new features.

---

# Day 56 — Interview Questions

**1. What two techniques does Top K Frequent Elements combine, and what does each contribute?**

*Answer:* HashMap frequency counting (Day 4/5) determines how often each element appears; a min-heap of size k (this week's recurring shape) bounds the result to the k most frequent, ordered by that computed frequency rather than the elements' own values.

---

**2. Why does the bucket-sort approach to Top K Frequent Elements reach O(n), beating the heap approach's O(n log k)?**

*Answer:* Frequency in an array of length n is bounded by n itself, so frequencies can be used directly as array indices (buckets), avoiding a heap's log-factor entirely — counting is O(n), and scanning buckets from highest frequency down is also O(n).

---

**3. When would you prefer the heap approach over bucket sort for this problem, despite bucket sort's better asymptotic complexity?**

*Answer:* When the input isn't fully known up front — a streaming variant needing "current top k" at any moment fits a heap naturally, while bucket sort's fixed-size bucket array assumes the maximum possible frequency is known in advance.

---

**4. Why is brute-force testing of every integer for Ugly Number II described as structurally wasteful, not just slow?**

*Answer:* Ugly numbers thin out as integers grow — the nth ugly number is typically much larger than n — so most integers tested turn out not to be ugly at all, wasting work on candidates that were never going to contribute to the answer.

---

**5. Why does generating candidates by multiplying confirmed-ugly numbers by 2, 3, and 5 avoid that waste?**

*Answer:* Every ugly number greater than 1 is provably 2×, 3×, or 5× some smaller ugly number — so this generation process only ever produces genuinely ugly candidates, with nothing spent testing numbers that turn out not to qualify.

---

**6. Why is a min-heap specifically needed here, rather than just iterating through generated candidates in the order they're produced?**

*Answer:* Multiplying one ugly number by 2, 3, and 5 doesn't produce the next ugly number in sorted order — it produces three candidates that must be weighed against every other pending candidate from every earlier ugly number. A min-heap always surfaces the smallest pending candidate next, regardless of generation order.

---

**7. Why is the `seen` set required for correctness, not just a minor optimization?**

*Answer:* The same ugly number is often reachable multiple ways (e.g., 6 = 2×3 = 3×2). Without deduplication, the heap holds duplicate entries, and popping one wastes an iteration without advancing toward a genuinely new nth value — silently under-counting the result.

---

**8. Why does Ugly Number II use `long` for heap values, given the final answer is guaranteed to fit in a 32-bit int?**

*Answer:* Intermediate candidate values generated during the search (an already-large ugly number multiplied by 2, 3, or 5) can approach or exceed `Integer.MAX_VALUE` even though the guarantee only covers the final answer — the same overflow-awareness habit established Day 10/11.

---

**9. What does Week 8's Complete Problem Inventory now show for Trees and Heaps?**

*Answer:* Trees is fully closed at 15 required + 5 extra = 20 distinct total, spanning Weeks 7–8. Heaps has reached 6 of an eventual 10 required + 3 extra = 9 distinct so far, continuing into Week 9.

---

**10. What two old debts did Week 8 resolve that had been sitting open since much earlier in the plan?**

*Answer:* Median of Two Sorted Arrays, promised on the original plan's Day 21 and never delivered anywhere in its remaining days, and Quickselect, explicitly named as a future destination back on Day 12's Sort Colors.

---

## Daily Deliverable Check

- [ ] Top K Frequent Elements and Ugly Number II solved, pushed to `dsa-java/heaps/`.
- [ ] Extra Practice: Sort Characters by Frequency attempted if time allowed — understood as Top K Frequent's direct sibling either way.
- [ ] Self-Check complete — one Tree problem solved cold, without hints, and any gaps it revealed noted honestly.
- [ ] Weekly ritual and scorecard complete.
- [ ] Week 8 Consolidation reviewed — planned-vs-actual counts, diagnostic list, and what Week 9 assumes, all clear before moving on.

---

## What Next Week Assumes You Already Know Cold

See "What Week 9 assumes," above — this is the same closing handoff every prior week's consolidation has ended on, carried forward in full rather than summarized away.

**Next:** Day 57, Week 9 — Heaps: Scheduling Patterns (`Week_09_Revised.md`).
