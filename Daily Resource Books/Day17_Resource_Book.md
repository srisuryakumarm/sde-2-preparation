# SDE-2 Resource Book Series
## Day 17 — Sliding Window with HashMap, and Comparable vs. Comparator

**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**← Previous:** [Day 16](Day16_Resource_Book.md) &nbsp;|&nbsp; **Next →:** [Day 18](Day18_Resource_Book.md)
**Companion to:** Day 17 of `Week_03_Revised.md`

---

### Recap

Problem 7 today reuses Day 16's Permutation in String skeleton (LC 567) verbatim — same fixed window, same frequency-array match — with one change: collect every matching start index instead of returning on the first. Problem 8 returns to variable-size windows (Day 15), but with a loop shape you haven't seen yet: shrinking *while* the window is still valid, to find the **smallest** satisfying window, rather than shrinking *until* valid to find the **largest** one.

⚠️ **A note on today's plan wording before we start:** the plan's project block calls the practice class a "`Transaction` record." Java's `record` keyword isn't taught until Week 4, Day 28 (Records and Sealed Classes) — using it here would mean using untaught syntax. Today's `Transaction` is written as a plain class instead, hand-rolling exactly what `record` will later generate for you in one line. When you reach Day 28, this class is the thing to look back at.

---

### Learning Objectives

By the end of today, without notes, you should be able to:
1. Extend a fixed-window frequency-match solution to collect all matches, not just detect one.
2. Explain, precisely, how "shrink while valid" (minimize) differs in loop structure from "shrink until valid" (maximize) — not just that they look different.
3. State the one-sentence difference between `Comparable` and `Comparator`, and know when to reach for each.
4. Explain why `TreeMap` and `PriorityQueue` are each O(log n), from their underlying structures, not by memorized label.

---

### Concept Dependency Map

```
Day 16 — LC 567 fixed-window skeleton         Day 15B — variable-size window template
        │                                               │
        ▼                                               ▼
Day 17: LC 438 — same skeleton,           Day 17: LC 209 — shrink WHILE valid
   collect ALL match indices                  (minimize), contrasted with
   (List<Integer> instead of boolean)          Days 15–16's shrink-UNTIL-valid (maximize)

Day 2 — interfaces                    Day 3 — Big-O, O(log n)
        │                                        │
        ▼                                        ▼
Day 17: Comparable (compareTo) vs. Comparator (compare)
        │
        ├──▶ TreeMap — Red-Black tree, keys always sorted, O(log n)
        └──▶ PriorityQueue — binary heap (array), O(log n) insert/poll, O(1) peek
        │
        ▼
  Transaction (plain class, hand-rolled) + TransactionSorting (Project)
  🔗 record itself arrives Week 4, Day 28 — this class previews exactly what it automates
```

---

## Problem 7: Find All Anagrams in a String

**LeetCode #438 — Medium — Pattern: Sliding Window + HashMap (frequency array)**

**Statement:** given strings `s` and `p`, return the starting indices of all of `p`'s anagrams in `s`.

**Brute force:** for every window the size of `p`, sort both and compare, or rebuild a frequency map from scratch. O(n · m log m) or O(n · m).

**Optimal — identical skeleton to Day 16's Permutation in String, generalized to collect every match:**

```java
public List<Integer> findAnagrams(String s, String p) {
    List<Integer> result = new ArrayList<>();
    if (p.length() > s.length()) return result;

    int[] need = new int[26];
    int[] window = new int[26];
    for (char c : p.toCharArray()) need[c - 'a']++;

    int windowSize = p.length();
    for (int right = 0; right < s.length(); right++) {
        window[s.charAt(right) - 'a']++;
        if (right >= windowSize) {
            window[s.charAt(right - windowSize) - 'a']--;
        }
        if (right >= windowSize - 1 && Arrays.equals(need, window)) {
            result.add(right - windowSize + 1);
        }
    }
    return result;
}
```

**Why it works:** exactly LC 567's reasoning (frequency-profile equality is order-independent, so it correctly detects any anagram) — the only change is `return true` becomes `result.add(startIndex)`, and the loop runs to completion instead of short-circuiting.

**Trace:** `s = "cbaebabacd"`, `p = "abc"`. `need = {a:1,b:1,c:1}`, `windowSize = 3`.

The window first matches at `right=2` (window `[0,2] = "cba"`) → add start index `0`. It stays mismatched through `right=3..7` as `'e'` and repeated letters cycle through. It matches again at `right=8` (window `[6,8] = "bac"`) → add start index `6`. Final result: **`[0, 6]`**.

**Complexity:** Time O(n), Space O(1) (fixed 26-length arrays, output array excluded by convention).

**Edge cases & mistakes:**
- ⚠️ `p.length() > s.length()`: no window of that size can exist — guard before the loop.
- ⚠️ Off-by-one on the reported start index: it's `right - windowSize + 1`, not `right - windowSize` — verify against the trace above if this ever feels uncertain.

**💡 Interview framing:** "same fixed-window frequency-match as Permutation in String, generalized to collect every occurrence." Naming the reuse explicitly (rather than re-deriving it) is itself a signal to the interviewer that you recognize the pattern family, not just the individual problem.

---

## Problem 8: Minimum Size Subarray Sum

**LeetCode #209 — Medium — Pattern: Sliding Window (variable), shrink-while-valid**

**Statement:** given an array of positive integers `nums` and a target, return the length of the shortest contiguous subarray whose sum is ≥ target, or 0 if none exists.

**Brute force:** for every starting index, extend right until the sum meets the target, recording the shortest such extension. O(n²).

**Optimal — shrink *while* valid, not *until* valid:**

```java
public int minSubArrayLen(int target, int[] nums) {
    int left = 0, sum = 0, best = Integer.MAX_VALUE;
    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];
        while (sum >= target) {
            best = Math.min(best, right - left + 1);
            sum -= nums[left];
            left++;
        }
    }
    return best == Integer.MAX_VALUE ? 0 : best;
}
```

**Why the loop shape is genuinely different, not just cosmetically different, from Days 15–16:** in every "longest window" problem so far, the `while` fires when the window becomes **invalid**, and its job is to restore validity — you record the answer *after* the loop, once you know the window is valid again. Here, the `while` fires when the window is **still valid**, and its job is to keep shrinking *precisely because* it's valid — you're actively trying to prove you can do with less, recording the answer *inside* the loop, every single time shrinking preserves validity. The loop only stops once shrinking *would* break validity. Same syntax shape (`while` with a `left++` inside), opposite intent.

**Trace:** `target = 7`, `nums = [2,3,1,2,4,3]`.

| right | sum after add | while (sum≥7)? | best updates | sum after shrink | left |
|---|---|---|---|---|---|
| 0 | 2 | no | — | 2 | 0 |
| 1 | 5 | no | — | 5 | 0 |
| 2 | 6 | no | — | 6 | 0 |
| 3 | 8 | yes | best=min(∞,4)=4 | 6 (left→1) | 1 |
| 4 | 10 | yes ×2 | best=min(4,4)=4, then best=min(4,3)=**3** | 6 (left→2, then left→3) | 3 |
| 5 | 9 | yes ×2 | best=min(3,3)=3, then best=min(3,2)=**2** | 3 (left→4, then left→5) | 5 |

Final answer: **2** (the subarray `[4,3]`).

**Complexity:** Time O(n) — same total-movement argument as always: `right` moves n times, `left` moves at most n times total across the whole run. Space O(1).

**Edge cases & mistakes:**
- ⚠️ No subarray meets the target at all — `best` stays `Integer.MAX_VALUE`; return `0`, not the sentinel value itself.
- ⚠️ This exact "shrink-while-valid" shape is the one to compare against LC 76 (Minimum Window Substring) tomorrow's project note asks you to write up — start noticing now that the *validity check itself* (a numeric sum vs. a character-coverage count) is the only real difference, not the loop shape.

**💡 Interview framing:** open by naming the shape explicitly: "longest-window problems shrink to *restore* validity; this one shrinks to *exploit* validity — I'm looking for the minimum, so every valid window is a candidate answer worth recording before I give it up." That sentence alone answers the most common follow-up before it's asked.

---

## Theory: Comparable vs. Comparator, and Heap-Adjacent Structures

### Comparable — a class's one natural ordering

```java
public class Employee implements Comparable<Employee> {
    private final String name;
    private final int salary;
    // constructor, getters omitted

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);
    }
}
```

A class implements `Comparable<T>` **once**, defining its single default sort order. `Collections.sort(list)` (no second argument) uses this.

### Comparator — as many external orderings as you want

```java
Comparator<Employee> byNameThenSalaryDesc =
    Comparator.comparing(Employee::getName)
              .thenComparing(Employee::getSalary, Comparator.reverseOrder());
```

`Comparator<T>` lives *outside* the class — you can define any number of them, including for classes you don't own (built-ins, or third-party library types you can't edit). `list.sort(comparator)` or `Collections.sort(list, comparator)` uses one.

🔑 **Key Takeaway — the one-sentence version:** `Comparable` is "how do I sort by default," defined once, inside the class. `Comparator` is "how do I sort *this time*," defined however many times you need, outside the class.

⚠️ **Common Mistake — subtraction comparators.** `(a, b) -> a.getAmount() - b.getAmount()` looks fine and often *is* fine for small values, but it's a real bug waiting to happen: for large-magnitude values it can silently overflow (wrapping to a wrong-signed result), and for `double`/`float` fields it truncates when the lambda's result gets narrowed to the `int` a `Comparator` must return. Always prefer `Integer.compare(a, b)`, `Double.compare(a, b)`, `Long.compare(a, b)`, or `Comparator.comparing(...)` — never raw subtraction.

### TreeMap — sorted keys, backed by a Red-Black tree

`TreeMap<K,V>` keeps every key in sorted order at all times (via `Comparable` or a supplied `Comparator`), backed by a self-balancing binary search tree (a Red-Black tree) — balancing is what guarantees the tree's height stays O(log n) even on adversarial/sorted insertion order, unlike a naive unbalanced BST, which can degrade to a straight line (height O(n)) on sorted input. `get`/`put`/`remove` are therefore **O(log n)** — you walk one root-to-leaf path.

```java
TreeMap<String, Integer> scores = new TreeMap<>();
scores.put("Charlie", 90);
scores.put("Alice", 95);
scores.put("Bob", 88);
System.out.println(scores); // {Alice=95, Bob=88, Charlie=90} — always sorted, no extra sort() call
```

Unlike `HashMap` (O(1) average, no ordering guarantee at all), `TreeMap` trades a bit of speed for genuine ordered operations — `firstKey()`, `lastKey()`, `floorKey()`, `ceilingKey()`, range views — none of which `HashMap` can offer at any cost, since it has no concept of "next" or "previous" key.

### PriorityQueue — a binary heap, not a fully sorted structure

`PriorityQueue<T>` is backed by a **binary min-heap** stored in a resizable array (min by default — smallest element has highest priority). For a node at array index `i`: its children live at `2i+1` and `2i+2`, its parent at `(i-1)/2`.

- **`peek()` — O(1):** the minimum is always at array index 0 by the heap's core invariant (every parent ≤ its children).
- **`offer()`/`add()` — O(log n):** append at the end, then "sift up" — repeatedly swap with the parent while smaller than it. At most tree-height swaps.
- **`poll()`/`remove()` — O(log n):** swap the root with the last element, remove the (old root, now last), then "sift down" from the root — repeatedly swap with the smaller child while larger than it. At most tree-height swaps.

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5); minHeap.offer(1); minHeap.offer(3);
minHeap.peek(); // 1 — O(1)
minHeap.poll(); // 1 — removes and returns it, O(log n)

// Max-heap: supply a reversing Comparator
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
```

🔑 **Key Takeaway — TreeMap vs. PriorityQueue, the actual trade-off:** `TreeMap` gives you full sorted access to *everything*, by key. `PriorityQueue` only ever gives you fast access to the single smallest (or largest) element — a narrower guarantee, but cheaper to maintain (simpler array-backed structure, no tree-node/pointer overhead), which is exactly the shape needed by "repeatedly grab the current min/max" problems. 🔗 That specific shape is the Top-K / heap pattern family — not formally introduced yet, but Week 4's Meeting Rooms II (Day 26) leans on a min-heap of end times, so file this away.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals` · **Task:** `Transaction` (plain class) and `TransactionSorting`

```java
public final class Transaction {
    private final String id;
    private final double amount;
    private final long timestamp;

    public Transaction(String id, double amount, long timestamp) {
        this.id = id;
        this.amount = amount;
        this.timestamp = timestamp;
    }

    public String getId() { return id; }
    public double getAmount() { return amount; }
    public long getTimestamp() { return timestamp; }

    @Override
    public String toString() {
        return "Transaction{id='" + id + "', amount=" + amount + ", timestamp=" + timestamp + '}';
    }
}
```

In `TransactionSorting`, build a `List<Transaction>`, then:
1. Sort by `amount`, ascending, with a lambda `Comparator` using `Double.compare`.
2. Re-sort the same list by `timestamp`, descending, with a second lambda using `Long.compare`.

Print the list after each sort to confirm the ordering visually.

**Definition of done:** both sorts print correctly ordered output; pushed. 🔗 Note in a comment that this class is what Day 28's `record` keyword will let you write in one line.

---

## Career Block Guide (1 hr)

- **LinkedIn engagement (20 min):** 3–5 posts, substantive comments.
- **Networking:** follow up on any recruiter responses that have come in — keep it short and concrete (confirm interest, ask the specific next step, propose a time window if scheduling is involved). A prompt, brief reply is worth more here than a long one.

---

## Day 17 — Interview Questions

**Q1. How does Find All Anagrams in a String differ from Permutation in String, mechanically?**
A: Nothing about the sliding-window mechanism changes — only what happens on a match: return `true` immediately versus record the index and keep scanning.

**Q2. Explain the structural difference between "shrink while valid" and "shrink until valid."**
A: "Shrink until valid" (Days 15–16) shrinks only when the window is currently *invalid*, to restore validity, and records the answer once validity is restored — it's solving for the *longest* valid window. "Shrink while valid" (today) shrinks precisely *because* the window is currently valid, recording the answer at every step of the shrink, stopping only once shrinking would break validity — it's solving for the *shortest* valid window.

**Q3. State the difference between `Comparable` and `Comparator` in one sentence.**
A: `Comparable` defines a class's one built-in natural ordering (inside the class, via `compareTo`); `Comparator` defines any number of external, swappable orderings (outside the class, via `compare`).

**Q4. Why is `(a, b) -> a.getAmount() - b.getAmount()` a risky Comparator, even though it often "works"?**
A: It risks integer overflow for large-magnitude values, and for floating-point fields, implicit narrowing to the `int` a Comparator must return can truncate and misorder close values; `Type.compare(a, b)` avoids both failure modes.

**Q5. Why is `TreeMap`'s `get()` O(log n) while `HashMap`'s is O(1) average — and what do you get in exchange for the slower guarantee?**
A: `TreeMap` is a balanced binary search tree, so any operation walks one root-to-leaf path bounded by O(log n); `HashMap` computes a bucket directly. In exchange for the slower guarantee, `TreeMap` keeps keys in sorted order at all times and supports range/ordered queries `HashMap` cannot offer at any speed.

**Q6. Why is `PriorityQueue.peek()` O(1) but `poll()` is O(log n)?**
A: The minimum is always sitting at array index 0 by the heap invariant, so peeking is a direct array read. Removing it requires promoting a new root (swap in the last element, then sift it down to restore the heap invariant), which costs up to the tree's height in swaps.

**Q7. Why does today's `Transaction` use a hand-written class instead of `record`?**
A: `record` isn't taught until Week 4, Day 28 — using it here would use syntax before its prerequisite lesson. The hand-written version does the same job and previews exactly what `record` will later automate.

---

## Daily Deliverable Check

- [ ] Find All Anagrams in a String (LC 438) and Minimum Size Subarray Sum (LC 209) solved, pushed.
- [ ] Can state the difference between Comparable and Comparator in one sentence, and explain TreeMap/PriorityQueue's O(log n) from the underlying structure.
- [ ] `Transaction` class and both sort variations pushed in `TransactionSorting`.

---

### What Tomorrow Assumes You Already Know Cold

Day 18 doesn't build directly on today's Comparable/Comparator/TreeMap/PriorityQueue material — it returns to variable-size windows (Day 15's template) for Fruit Into Baskets and Longest Subarray of 1's After Deleting One Element, and opens Exception Handling as an independent theory thread. What it does assume solidly: the shrink-until-valid (longest-window) shape from Days 15–16, since Fruit Into Baskets is a direct instance of it with a new validity check (distinct-value count instead of a violation counter).
