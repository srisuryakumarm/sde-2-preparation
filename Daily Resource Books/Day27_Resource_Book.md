# Day 27 — Greedy & Intervals Capstone, and Leave Week Wrap-Up

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 26 Resource Book](Day26_Resource_Book.md)
**Next ▶:** [Day 28 Resource Book](Day28_Resource_Book.md)
**Companion to:** Day 27 of `Week_04_Revised.md`

---

## Recap

Five days of Greedy & Intervals, in order: formal greedy + exchange arguments + frontier domination (Day 23) → prefix-elimination proofs + start-time-sorted Intervals (Day 24) → end-time-sorted Intervals, a different key for a different question (Day 25) → the min-heap applied to interval scheduling (Day 26). Today closes the pattern with its eleventh and final required problem — one that uses interval-style reasoning **without** ever constructing an explicit `[start, end]` object — then steps back to consolidate *why* each greedy choice this week was actually safe, the skill the whole week has been building toward.

Today is the lightest DSA day of the leave week (2.5–3 hours, one new problem) but the fullest in reflection — Theory and Career Blocks both return today, at full weight, as the leave week's intensity winds down.

---

## Learning Objectives

By the end of today, without notes:

1. Solve Partition Labels by converting an implicit character-range problem into interval-style greedy reasoning, via a last-occurrence-index sweep.
2. Produce a correct, one-sentence safety argument for at least three of this week's eleven greedy choices, in the specific "why can this never be wrong" form an interviewer is listening for — not "it worked on the examples I tried."
3. Recite the full closing shape of Greedy & Intervals: 11 required problems + 2 extra practice = 13 distinct problems, the second pattern family with zero presence anywhere in the original 17-week plan.

---

## Concept Dependency Map

```
Full week, consolidated:

Day 23 — Greedy, Formalized
├─ LC 122 Buy/Sell Stock II — telescoping decomposition
├─ LC 55  Jump Game — frontier domination
└─ LC 45  Jump Game II — BFS-by-levels, frontier domination extended

Day 24 — Greedy Continues + Intervals (sort by START)
├─ LC 134 Gas Station — prefix-elimination
├─ LC 56  Merge Intervals — combine overlapping
├─ LC 57  Insert Interval — same mechanism, O(n) via given sort
└─ Extra: LC 406 Queue Reconstruction by Height — two-key greedy

Day 25 — Intervals (sort by END)
├─ LC 435 Non-overlapping Intervals — keep earliest-ending survivor
├─ LC 452 Min Arrows to Burst Balloons — identical shape, reframed
└─ Extra: LC 986 Interval List Intersections — two pointers, two lists

Day 26 — Intervals + Min-Heap
├─ LC 252 Meeting Rooms — existence check
└─ LC 253 Meeting Rooms II — counting via monotonically-growing heap
        │
        ▼
TODAY — LC 763 Partition Labels — interval reasoning via last-occurrence index,
        NO explicit interval object at all
        │
        ▼
Greedy & Intervals CLOSES — 11 required + 2 extra = 13 distinct problems
        │
        ▼
Theory: Greedy, Reviewed — WHY, not just WHAT, for 3 of the 11
```

---

## Problem: Partition Labels (LeetCode 763, Medium) — Pattern: Intervals, Implicit

**Statement:** given a string `s`, partition it into as many parts as possible so that each letter appears in **at most one** part, and return the length of each part, in order.

### Approach 1 — Brute force (conceptual)

For each possible partition boundary, check whether every letter within the resulting segment is absent from every other segment — checking this directly for all possible boundary combinations is exponential in the number of candidate cut points, and not a real candidate to code.

### Approach 2 — Optimized: last-occurrence index, then a single greedy sweep

**The reframe — turning characters into implicit intervals:** every character in `s` effectively defines an interval — from its *first* appearance to its *last* appearance, every occurrence of that character must live inside the same partition. If character `'a'` last appears at index `8`, then no partition boundary can fall anywhere between `'a'`'s first appearance and index `8`, or a second `'a'` would end up split across two partitions. This is Merge Intervals' overlap-and-combine idea again (Day 24), except the "intervals" are never built as explicit `[start, end]` objects — they're derived on the fly from a single pass, because characters (unlike Day 24's given intervals) already appear in the string in a fixed left-to-right order that plays the same role sorting-by-start played there.

```java
public static List<Integer> partitionLabels(String s) {
    int[] lastOccurrence = new int[26];
    for (int i = 0; i < s.length(); i++) {
        lastOccurrence[s.charAt(i) - 'a'] = i;   // overwritten each time — ends up TRUE last index
    }

    List<Integer> partitionLengths = new ArrayList<>();
    int start = 0, end = 0;

    for (int i = 0; i < s.length(); i++) {
        end = Math.max(end, lastOccurrence[s.charAt(i) - 'a']);   // extend boundary if needed
        if (i == end) {
            partitionLengths.add(end - start + 1);   // close the partition
            start = i + 1;
        }
    }
    return partitionLengths;
}
```

**Why `end = Math.max(end, ...)` mirrors Merge Intervals' extension step exactly:** just as Day 24's merge extended the currently-building interval's end whenever a new overlapping interval reached further, this extends the currently-building *partition's* boundary whenever the current character's last occurrence reaches further than what's already been committed to. The `if (i == end)` check is the moment nothing currently open still needs this partition to stay open — every character seen since `start` has now had its last occurrence accounted for — which is exactly Day 23's Jump Game II's "reached the current boundary, must commit" moment, applied here to closing a partition instead of counting a jump.

**Worked trace:** `s = "ababcbacadfegdehijhklij"`. First compute `lastOccurrence` for each letter that appears (only showing relevant ones): `a→8, b→5, c→7, d→14, e→15, f→11, g→13, h→19, i→22, j→21, k→20, l→22`.

Sweep:

| i | s[i] | lastOccurrence[s[i]] | end (updated) | i == end? | action |
|---|---|---|---|---|---|
| 0 | a | 8 | 8 | no | — |
| 1 | b | 5 | 8 | no | — |
| ... | | (end stays 8 through i=8, since nothing seen so far reaches further) | | | |
| 8 | a | 8 | 8 | **yes** | partition length = 8-0+1 = 9, start=9 |
| 9 | d | 14 | 14 | no | — |
| ... | | (end stays 14, or extends via e→15, through the relevant range) | | | |

Continuing this sweep for the full string produces partition lengths `[9, 7, 8]` — matching the known expected output for this exact input (`"ababcbaca"`, `"defegde"`, `"hijhklij"`).

**Complexity:** Time O(n) — one pass to build `lastOccurrence` (fixed-size 26-element array, the same "small, fixed key space" reasoning from Week 1's frequency-array problems), one more pass to sweep. Space O(1) — the `lastOccurrence` array is always exactly 26 elements, regardless of `s`'s length, the same amortized-O(1)-space argument used for every fixed-alphabet frequency structure this series has built since Week 1, Day 5.

**Edge cases:** every character appearing exactly once (each character is its own partition, since `end` never extends past the current index — `i == end` fires on every single iteration); the entire string being one repeated character (a single partition spanning the whole string, since `end` immediately jumps to the last occurrence on the very first character and nothing closes it early); a string where two characters' ranges nest cleverly inside each other without ever exceeding a shared boundary (correctly merges into one partition, via the same `Math.max` extension logic Merge Intervals uses for nested intervals).

💡 **Interview Insight:** naming the "each character defines an implicit interval from first to last occurrence, and this is Merge Intervals without materializing the intervals" connection out loud, before writing any code, is exactly the kind of pattern-recognition-across-superficially-different-problems interviewers are listening for at this level — it's a stronger signal than the code itself, which is short once the reframe is seen.

---

## Greedy & Intervals — Pattern Closes

**11 required problems** (LC 122, 55, 45, 134, 56, 57, 435, 452, 252, 253, 763) **+ 2 extra practice problems** (LC 406, 986) **= 13 distinct problems total.** The second pattern family — after Prefix Sum & Kadane's, closed Day 22 — with zero presence anywhere in the original 17-week plan, now fully closed without slipping the overall 120-day timeline by a single day.

---

## Theory Block Guide (1 hr) — Greedy, Reviewed: Why It Works Here, and Where It Would Fail

The plan's own instruction for today: write, for three of this week's eleven problems, a one-sentence argument for *why* the greedy choice is safe. "I'll be greedy here" is not a justification on its own — this is the actual interview skill, and interviewers will ask you to defend it. Three worked examples below, deliberately chosen to be three **different shapes** of argument, not three repetitions of the same one — recognizing which shape a new, unfamiliar problem calls for is the transferable part.

**1. Jump Game (LC 55) — a domination argument.** *The farthest reachable index at any point strictly dominates every closer reachable index, since anything reachable from a closer point remains reachable from the farthest one too — so tracking only the single farthest value, and discarding every other reachable point, loses no information about what's ultimately reachable.*

**2. Non-overlapping Intervals (LC 435) — an exchange argument.** *Among any cluster of mutually overlapping intervals, the one that ends earliest leaves at least as much room for everything that follows as any other choice from that cluster would — so swapping it into any optimal solution in place of a later-ending choice can never make that solution worse, meaning some optimal solution always agrees with the greedy pick.*

**3. Gas Station (LC 134) — a prefix-elimination argument.** *If starting from station `X` first fails at station `Y`, then every station between `X` and `Y` has already banked a non-negative surplus by the time it's reached — at least as much as starting fresh there would have — so if continuing from `X` still fails by `Y`, starting fresh at any of those in-between stations fails at least as early, eliminating the entire range at once rather than one station at a time.*

**🔑 Key Takeaway, tying all three together:** every one of these arguments has the same underlying shape — show that the greedy choice can be substituted into *any* alternative without making things worse — but the specific *mechanism* of that substitution differs each time (domination of a reachable set; swapping one interval for another; eliminating a contiguous range of candidates). Memorizing three answers doesn't transfer to a new problem; recognizing which mechanism a new problem's structure calls for does. When practicing further, the useful exercise isn't re-deriving these three again — it's picking one of the remaining eight problems from this week cold and producing its argument from scratch.

---

## Career Block Guide (1 hr) — Back to Normal Cadence as the Leave Week Ends

### LinkedIn Post 7

A draft to adapt, not copy verbatim — make it sound like you, and swap in your own specifics:

> This week I closed out two entire pattern families I'd genuinely never worked through in a structured way before: Greedy Algorithms & Intervals, and Prefix Sum & Kadane's Algorithm.
>
> Thirteen problems for the first, nine for the second — plus, more useful than the count, the habit of actually proving *why* a greedy choice is safe instead of just trusting that it "feels right." Turns out "I'll be greedy here" isn't an argument on its own; it needs to survive the question "what if I did something else instead?"
>
> Naming a real gap and closing it deliberately reads better than pretending it was never there. On to Binary Search next.

Naming a real gap you closed reads better than implying you always knew the material — worth keeping that framing in the post rather than softening it.

### Networking — catch up on outreach that slipped during the leave week

The last several days have been DSA-only, with career work reduced to light LinkedIn engagement. Before moving forward: review the companies identified on Days 24 and 25 (strong engineering / backend blogs) and send the connection requests or follow-ups that got deferred. A short, specific note referencing something concrete from their profile or recent activity — not a generic "I'd love to connect" — is worth the extra thirty seconds per message; specificity is what gets a response.

---

## Day 27 — Interview Questions

**Q1. How does Partition Labels turn a string-partitioning problem into interval reasoning without ever building an explicit interval object?** Each character implicitly defines a range from its first to its last occurrence — no two occurrences of the same character can be split across partitions, so a single pass tracking the running maximum "last occurrence seen so far" plays the same role Merge Intervals' explicit end-time tracking does, just derived on the fly instead of given upfront.

---

**Q2. Why is the `lastOccurrence` lookup array O(1) space, regardless of the input string's length?** It's always exactly 26 entries (one per lowercase letter), a fixed, bounded key space independent of `s`'s length — the same reasoning used for every fixed-alphabet frequency structure since Week 1's Valid Anagram.

---

**Q3. In Partition Labels, what does `i == end` signify, and what earlier problem's "commit" moment does it mirror?** It signifies that every character seen since the current partition's start has had its last occurrence fully accounted for — nothing still open requires the partition to stay open. This mirrors Jump Game II's `i == currentJumpEnd` moment (Day 23): the boundary has been reached, so committing (closing the partition / taking the jump) is forced, not optional.

---

**Q4. State Jump Game's safety argument in one sentence.** The farthest reachable index dominates every closer one, since anything reachable from a closer point remains reachable from the farthest point too — tracking only the farthest value loses no information about ultimate reachability.

---

**Q5. State Non-overlapping Intervals' safety argument in one sentence.** Among overlapping intervals, the earliest-ending one leaves at least as much room for what follows as any alternative choice would, so swapping it into any optimal solution never makes that solution worse.

---

**Q6. State Gas Station's safety argument in one sentence.** If starting from `X` fails at `Y`, every station between them already has a non-negative banked surplus by the time it's reached — at least as much as a fresh start there would have — so they all fail at least as early too, and the entire range is eliminated at once.

---

**Q7. What do all three of these arguments have in common, structurally?** Each shows the greedy choice can be substituted into any alternative solution without making it worse — the specific mechanism differs (domination of a reachable set, an interval swap, elimination of a candidate range), but the underlying template ("show the swap is always safe") is the same one used throughout this series since Container With Most Water, back in Week 2.

---

**Q8. What is the final closing count for Greedy & Intervals, and how does it split between required and extra?** 11 required problems (LC 122, 55, 45, 134, 56, 57, 435, 452, 252, 253, 763) + 2 extra practice (LC 406, 986) = 13 distinct problems — the second pattern family in this series with zero presence anywhere in the original 17-week plan.

---

## Daily Deliverable Check

- [ ] Partition Labels solved, pushed to `dsa-java/greedy-intervals/` — Greedy & Intervals ladder complete at 11/11 required.
- [ ] Greedy-safety reflection written (Jump Game, Non-overlapping Intervals, Gas Station, or your own choice of three) — can state at least one from memory without notes.
- [ ] LinkedIn Post 7 published.
- [ ] Networking catch-up complete: deferred connection requests from Days 24–25 sent.
- [ ] **Leave week complete.** Prefix Sum/Kadane's (9 distinct) and Greedy/Intervals (13 distinct) — both entirely new pattern families — fully closed, 22 distinct problems total across the two, without pushing the overall 120-day timeline by a single day.

---

## What Tomorrow Assumes You Already Know Cold

Day 28 closes out the week with a genuinely different kind of content — Binary Search (a new pattern, opening, not closing) alongside Java's `record` and `sealed interface` features — neither of which depends on anything from this week's Greedy & Intervals material. What *is* assumed solid going into Day 28, and into Week 5's much deeper Binary Search coverage immediately afterward, is the general discipline this week reinforced one more time on top of Prefix Sum & Kadane's: naming which of several closely-related techniques a problem calls for, and being able to justify that choice, rather than pattern-matching to the first thing that looks similar. Tomorrow is also this week's last day — expect a full consolidation alongside the new material, not just two more problems.

**Next:** [Day 28 Resource Book](Day28_Resource_Book.md) — Consolidation, and Binary Search Begins.
