# Day 57 — Heaps: Scheduling Patterns

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 56 Resource Book](Day56_Resource_Book.md)
**Next ▶:** [Day 58 Resource Book](Day58_Resource_Book.md)
**Companion to:** Day 57 of `Week_09_Revised.md`

---

## Recap

Week 8 opened Heaps from zero on Day 54 (array-backed complete binary tree, sift-up/sift-down, the unconditional-O(log n)-vs-BST contrast) and put it to work through three genuinely different roles: min-heap for "kth largest" (Day 54–55), max-heap for "keep the k best" (Day 54's Last Stone Weight, Day 55's K Closest Points), and a heap as a **candidate generator** rather than a query structure (Day 56's Ugly Number II). That's 6/10 required problems plus 3 extra (Minimum Cost to Connect Sticks, Kth Smallest in a Sorted Matrix, Sort Characters By Frequency) — 9 distinct heap problems solid before today.

Today adds a **fourth** role: a max-heap driving a **greedy scheduling decision**, repeatedly asking "what's the best choice *right now*, given everything already placed." Both of today's problems reuse the exact frequency-counting mechanism from Day 5 (Week 1's HashMap pattern) and Day 56 (Top K Frequent's HashMap + heap combination) — only the *use* of the frequency data is new, not how it's built.

## Learning Objectives

By the end of today, without notes:

1. Build a frequency-ordered max-heap and use it to drive a greedy placement decision, explaining precisely why greedy-by-frequency is correct here (not just that it works).
2. Solve Reorganize String and Task Scheduler, including the pigeonhole-based feasibility check both problems reduce to.
3. State Task Scheduler's O(1)-space closed-form alternative, and explain why the heap simulation is still worth knowing even though the formula is asymptotically better.
4. Correctly compute the "impossible" boundary condition using integer arithmetic, without an off-by-one error.

## Concept Dependency Map

```
HashMap frequency counting (Week 1, Day 5)
        │
        ▼
Heap mechanism, max-heap variant (Day 54)
        │
        ▼
HashMap + heap combination (Day 56, Top K Frequent / Ugly Number II)
        │
        ▼
NEW: Greedy-by-frequency placement
  ├─ Reorganize String — place most frequent char, hold it out one round
  └─ Task Scheduler — place most frequent task, cooldown queue tracks re-entry
        │
        ▼
🔗 Day 58: same greedy-heap family generalizes to two-heap (median)
   and k-way merge (Merge k Sorted Lists) — closes Heaps at 10/10
```

---

## Part 1 — Reorganize String

### Problem 1: Reorganize String (LeetCode 767, Medium) — Pattern: Max-Heap by Frequency

**Statement:** Given a string `s`, rearrange its characters so that no two adjacent characters are the same. Return any valid rearrangement, or `""` if impossible.

### Feasibility check, first

Before any rearranging: is it even possible? If one character's count exceeds `⌈n/2⌉` (n = string length), it's impossible — with that many copies of one character, you cannot separate every pair, no matter how you interleave the rest. In Java, `⌈n/2⌉` is computed as `(n + 1) / 2` using integer division.

**⚠️ Common Mistake:** writing `n / 2` instead of `(n + 1) / 2` for the ceiling. Verify against a boundary case, not just the general shape: `n = 5`, `maxFreq = 3` — is `"aaabb"` arrangeable? `(5+1)/2 = 3`, and `3 > 3` is false, so the check says *possible*. Confirm by construction: `a b a b a` — 3 a's, 2 b's, no two adjacent, valid. If you'd used `n/2 = 2` instead, `3 > 2` would wrongly report impossible.

### Approach — Max-heap, hold-out-one-round

```java
public static String reorganizeString(String s) {
    int n = s.length();
    int[] freq = new int[26];
    int maxFreq = 0;
    for (char c : s.toCharArray()) {
        freq[c - 'a']++;
        maxFreq = Math.max(maxFreq, freq[c - 'a']);
    }
    if (maxFreq > (n + 1) / 2) {
        return "";
    }

    PriorityQueue<int[]> maxHeap = new PriorityQueue<>((a, b) -> b[1] - a[1]); // [char, count] by count desc
    for (int i = 0; i < 26; i++) {
        if (freq[i] > 0) {
            maxHeap.offer(new int[]{i, freq[i]});
        }
    }

    StringBuilder result = new StringBuilder();
    int[] previous = null; // the entry withheld from the heap for exactly one round

    while (!maxHeap.isEmpty()) {
        int[] current = maxHeap.poll();
        result.append((char) ('a' + current[0]));
        current[1]--;

        if (previous != null && previous[1] > 0) {
            maxHeap.offer(previous);
        }
        previous = current; // this becomes next round's "on hold" entry
    }
    return result.toString();
}
```

**The mechanism, precisely:** at every step, place the currently-most-frequent remaining character — but hold the *just-placed* character out of the heap for exactly one round before it's eligible again. Since it can't be picked again until at least one other character has been placed in between, adjacency is structurally impossible. `previous` is what implements the hold: it's excluded from `maxHeap` during the round it was placed, then re-offered (if it still has remaining count) right after the *next* character goes in.

**Why greedy-by-frequency (not just "any legal choice") is necessary, not just sufficient:** always placing the *most* frequent remaining character is what prevents you from painting yourself into a corner later. If you instead placed a low-frequency character while a high-frequency one kept accumulating unplaced copies, you could reach a point where the only remaining character *is* the one you just placed — with nothing left to interleave it with. Placing greedily by frequency keeps every character's remaining count as spread out across the remaining string as possible.

> 🔑 **Key Takeaway:** "hold the last placement out for one round" is a reusable technique — it's the same shape you'll use tomorrow for Task Scheduler's cooldown, just generalized from a 1-round hold to an n-round hold.

### Worked trace

`s = "aab"`, n = 3. Frequencies: a→2, b→1. `maxFreq = 2`, `(3+1)/2 = 2`, `2 > 2` false → possible.

Heap: `[(a,2), (b,1)]`.

| Step | Poll | Append | `previous` re-offered? | Heap after | `previous` now |
|---|---|---|---|---|---|
| 1 | (a,2)→(a,1) | "a" | none yet | `[(b,1)]` | (a,1) |
| 2 | (b,1)→(b,0) | "ab" | (a,1), count>0 → yes | `[(a,1)]` | (b,0) |
| 3 | (a,1)→(a,0) | "aba" | (b,0), count=0 → no | `[]` | (a,0) |

Result: `"aba"` — no two adjacent characters equal. Correct.

### Complexity

At most 26 distinct characters ever sit in the heap, so every push/pop is `O(log 26)` — a constant — but it's conventional (and what an interviewer expects to hear) to state it in terms of the general alphabet size `k`:

**Time: O(n log k)** — n characters placed, each costing one poll and up to one offer, each `O(log k)`. For a fixed 26-letter alphabet this is `O(n)`.
**Space: O(k)** — the heap and frequency array, bounded by alphabet size, not string length.

### Edge cases

- `maxFreq > (n+1)/2` → return `""` immediately, don't attempt to build.
- Single-character string (`n=1`) → trivially valid, returned as-is.
- All characters distinct → heap drains without ever needing to hold anything back in a way that matters; still correct, just never actually constrained.

### Interview framing

**Say before coding:** "First I'll check feasibility with a pigeonhole argument — if any character needs more than half the slots (rounding up), it's impossible. Then I'll greedily place the most frequent remaining character each step, holding the one I just placed out of contention for exactly one round so it can't repeat immediately."

**Likely follow-up:** "Can you do this without a heap?" — yes: since the alphabet is fixed at 26, you can sort characters by frequency once and place them at even/odd indices (most frequent character fills all even indices first, then odds), achieving `O(n + k log k)` (or `O(n)` if you count the fixed 26-element sort as constant). This is genuinely faster in practice but doesn't generalize to unbounded alphabets the way the heap approach does — worth mentioning as extension material if there's time, but the heap version is what this week is building reps in.

---

## Part 2 — Task Scheduler

### Problem 2: Task Scheduler (LeetCode 621, Medium) — Pattern: Max-Heap + Cooldown Queue

**Statement:** Given an array of CPU tasks (each a character A–Z) and a non-negative cooldown `n`, the same task type must be separated by at least `n` intervals (idle or other tasks). Return the minimum total time (including idles) to finish all tasks.

**⚠️ Notation collision, flagged up front:** the problem's own cooldown parameter is *also* called `n` — completely unrelated to "n = input size," which is how `n` was used in every complexity bound so far this series. This resource book uses `n` only for the cooldown value (matching the problem statement) and `T` for the total number of tasks, to avoid silently reusing one letter for two different things mid-explanation.

### Approach 1 — Max-heap + cooldown queue (this pattern's core technique)

```java
public static int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;

    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
    for (int f : freq) {
        if (f > 0) maxHeap.offer(f);
    }

    Queue<int[]> cooldown = new LinkedList<>(); // [remainingCount, availableAtTime]
    int time = 0;

    while (!maxHeap.isEmpty() || !cooldown.isEmpty()) {
        time++;
        if (!maxHeap.isEmpty()) {
            int count = maxHeap.poll() - 1;
            if (count > 0) {
                cooldown.offer(new int[]{count, time + n});
            }
        }
        // otherwise this tick is a forced idle — heap was empty but cooldown wasn't

        if (!cooldown.isEmpty() && cooldown.peek()[1] == time) {
            maxHeap.offer(cooldown.poll()[0]);
        }
    }
    return time;
}
```

**Mechanism:** each tick, run the currently-most-frequent remaining task type (max-heap), then park it in `cooldown` with the tick at which it becomes eligible again (`time + n`). Each tick, also check whether anything's cooldown has just expired and, if so, return it to the heap. If the heap is empty but something is still cooling down, the tick is a forced idle — the loop still advances `time` but has nothing to schedule.

**Why the greedy "always run the highest-remaining-count task" choice is correct:** any task type with a large remaining count is the one most likely to run out of "room" later — the more instances remain, the more total cooldown-spanning gaps that type alone will need. Running it now, whenever it's eligible, is what keeps its remaining copies as spread out as possible; deferring it in favor of a lower-count task only makes its eventual scheduling *harder*, never easier.

### Approach 2 — Closed-form counting (no heap, O(1) extra space)

There's a well-known formula that gets the same answer without simulating time step by step at all:

```java
public static int leastIntervalFormula(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;

    int maxFreq = 0;
    for (int f : freq) maxFreq = Math.max(maxFreq, f);

    int countOfMaxFreq = 0;
    for (int f : freq) if (f == maxFreq) countOfMaxFreq++;

    int intervalCount = (maxFreq - 1) * (n + 1) + countOfMaxFreq;
    return Math.max(tasks.length, intervalCount);
}
```

**The argument, not just the formula:** take the most frequent task type and lay out `maxFreq - 1` complete "blocks," each one instance of that task followed by `n` cooldown slots — that's `(maxFreq - 1) * (n + 1)` slots, then add the final instance of that task type (no trailing cooldown needed after the very last one) plus one instance of every *other* task type that's tied for the max frequency (they can't all fit in earlier blocks' gaps without also hitting the max-frequency-task limit). Every task that *isn't* at the max frequency is guaranteed to fit into the gaps this skeleton already creates, without ever needing to extend the schedule further — the `Math.max(tasks.length, ...)` handles the one case where there's no idle time at all (enough distinct low-frequency tasks to fill every gap and then some), in which case the answer is simply the task count itself.

> 💡 **Interview Insight:** state both. Lead with the heap simulation — it's this week's pattern and generalizes better to variants (e.g., "task scheduler with different cooldowns per type," where the closed form breaks down but the heap approach barely changes). Then mention the closed form as the "can you do better?" follow-up: same problem, `O(T)` time, `O(1)` extra space (26-bucket array aside), no heap needed at all. Interviewers who ask this follow-up are checking whether you over-index on "the pattern of the week" versus recognizing when a problem has a sharper, non-general solution.

### Worked trace (closed form)

`tasks = ["A","A","A","B","B","B"]`, `n = 2`. `freq: A=3, B=3`. `maxFreq = 3`, `countOfMaxFreq = 2` (both A and B tied). `intervalCount = (3-1)*(2+1) + 2 = 6 + 2 = 8`. `tasks.length = 6`. `max(6, 8) = 8`.

Confirm by construction: `A B idle A B idle A B` = 8 slots, and no A or B repeats within 2 slots of itself. Matches.

### Complexity

**Heap approach — Time: O(T)** where T = total number of tasks (the loop runs once per tick, and total ticks is bounded by the final answer, itself `O(T · n)` in the worst case, but each heap operation is `O(log 26)`, a constant — so this is linear in the schedule length, not in `T` alone, in the strict sense; in practice this is the number reported in interviews and is fine to state as `O(T)` for a fixed, bounded alphabet). **Space: O(1)** beyond the 26-bucket frequency array (heap and queue both bounded by 26 entries).
**Closed-form — Time: O(T)** (one pass to count frequencies). **Space: O(1)** (fixed 26-element array).

### Edge cases

- `n = 0` → no cooldown at all; answer is always `tasks.length`.
- One task type only, e.g., `["A","A","A"]`, `n = 2` → `(3-1)*3 + 1 = 7`, correctly accounting for two full cooldown gaps.
- Every task type distinct → `countOfMaxFreq` could be very large; `Math.max` correctly falls back to `tasks.length`.

### Interview framing

**Say before coding:** "I'll count frequencies, then greedily run the highest-remaining-count task each tick, parking a just-run task in a cooldown structure until it's eligible again."
**Likely follow-up:** "Can you avoid simulating every tick?" → the closed-form derivation above.
**Justify unprompted:** the heap approach generalizes to variants the closed form can't handle (per-type cooldowns, tasks with priorities beyond frequency) — say this even if not asked, since it shows you understand *why* you're choosing the heavier tool.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts, same cadence as every non-posting day this series.
**Networking:** apply to 2 Tier B companies.

---

## Day 57 — Interview Questions

**Q1. Why must the "hold out for one round" entry be re-offered *before* checking the heap's next poll, not after?** Because the very next character placed must come from what's currently the most frequent *eligible* candidate — if the held-out entry isn't back in the heap in time, the algorithm could pick a worse choice than necessary, though correctness (no two adjacent) is still preserved either way since the hold-out itself is what guarantees non-adjacency. The ordering mainly affects whether you get *a* valid answer versus a well-formed one built by the intended greedy rule.

**Q2. Derive the impossibility boundary for Reorganize String and explain why it's `(n+1)/2`, not `n/2`.** A character with `maxFreq` copies needs `maxFreq - 1` gaps of at least one other character between its copies. If `maxFreq > ⌈n/2⌉`, there aren't enough remaining characters to fill every gap. `(n+1)/2` computed with Java integer division equals `⌈n/2⌉` exactly for positive integers — `n/2` alone gives the floor, which is off by one whenever `n` is odd.

**Q3. Why is greedy-by-frequency provably correct for Reorganize String, not just a heuristic that happens to work?** Any strategy that ever defers placing the currently-most-frequent character risks that character's remaining copies clustering together with fewer future opportunities to interleave them — deferring never creates more spacing options, only fewer. Always placing the max keeps the remaining distribution as spread out as it can possibly be at every step.

**Q4. In Task Scheduler, what does the notation collision between the cooldown parameter `n` and "input size" cause, and how do you avoid it in an interview?** The problem overloads `n` for cooldown, which conflicts with the usual complexity-analysis convention of `n` for input size. State explicitly which `n` you mean when giving Big-O, or switch to a different variable name (this book uses `T` for task count) to keep the two unambiguous out loud.

**Q5. Walk through the closed-form Task Scheduler derivation.** Take the max-frequency task type, lay out `maxFreq - 1` blocks of (1 task + n cooldown slots), add one final instance of that type, add one instance of every other type tied for the max frequency (they can't all fit into earlier gaps without exceeding those gaps' capacity), and everything with lower frequency is guaranteed to fit into the resulting gaps for free. Take the max of that count against the raw task count, to cover the case with no idle time at all.

**Q6. Why does the heap-based simulation remain worth knowing even though the closed form is asymptotically better?** The closed form is a sharp, problem-specific insight that stops working the moment the problem is generalized (e.g., different cooldowns per task type, or a priority beyond pure frequency); the heap simulation is the general technique that survives those variants with minimal changes — showing both, and explaining the trade-off, is exactly what a tier-1 interview is checking for.

**Q7. What forces a "forced idle" tick in the heap simulation, and how is it represented in the code?** The max-heap is empty (nothing eligible to run right now) but the cooldown queue isn't (something is still waiting to become eligible) — `time` still advances, but no `count` is decremented and no character is appended to output, since there's genuinely nothing schedulable that tick.

---

## Daily Deliverable Check

- [ ] Reorganize String solved — feasibility check verified against the `n=5, maxFreq=3` boundary case, not just the general shape.
- [ ] Task Scheduler solved with **both** the heap simulation and the closed-form formula, and can explain the trade-off between them unprompted.
- [ ] Both pushed to `dsa-java/heaps/`.
- [ ] Can trace the "hold out for one round" mechanism on a fresh 3-character example without looking back at today's trace.

---

## What Tomorrow Assumes You Already Know Cold

Day 58 assumes today's max-heap-driven greedy placement is solid, since tomorrow immediately extends the *same* "always act on the current best candidate" instinct into two new heap roles: a **two-heap** structure (splitting a data stream into halves for O(1) median queries) and a heap as a **merge coordinator** across k independent streams. Neither is a repeat of today's mechanism, but both assume you're no longer deriving "why a heap here" from scratch — that judgment call should already be reflexive heading in.

**Next:** [Day 58 Resource Book](Day58_Resource_Book.md) — Heaps close at 10/10: Two-Heap Median, and k-Way Merge.
