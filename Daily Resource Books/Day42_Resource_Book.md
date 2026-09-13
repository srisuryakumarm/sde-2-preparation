# Day 42 (Sunday) — Consolidation, and Monotonic Stack Continues

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 41 Resource Book](Day41_Resource_Book.md)
**Next ▶:** [Day 43 Resource Book](Day43_Resource_Book.md)
**Companion to:** Day 42 of `Week_06_Revised.md`
**Also see:** [Week 6 Interview Question Bank](Week6_Interview_Questions.md)

---

## Recap

The week opened with cycle detection and closes with two more Monotonic Stack problems — both direct reapplications of the index-based invariant from Days 40–41, with no new mechanism to learn today. That leaves today genuinely lighter on new material, which is exactly the point of a consolidation day: today is for confirming the week's heaviest lift — Linked Lists closing, Stacks/Monotonic Stack opening, and a full second track of Java concurrency — actually landed, not for adding to it.

---

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Solve a Linked List problem from earlier this week cold, with no reference material.
2. Apply the index-based Monotonic Stack pattern to Daily Temperatures without being told it's the same mechanism as Next Greater Element II.
3. Explain why Evaluate RPN must pop its two operands in a specific order, with a concrete example where reversing that order silently produces a wrong (not crashing) answer.
4. State, precisely, how many problems this week added to the running total, and how many of those were genuinely new versus recapped.

---

## Concept Dependency Map

```
Day 34–39: full Linked List technique set (fast/slow, reversal,
  dummy heads, HashMap node-mapping, doubly linked lists)
        │
        ▼
Today: Self-Check — one of these, attempted cold, no notes

Day 40–41: index-based Monotonic Stack, amortized O(n) proof
        │
        ├──▶ Today: Daily Temperatures (LC 739) — same mechanism,
        │      computing a day-GAP instead of returning a value
        │
        └──▶ Today: Evaluate RPN (LC 150) — general-purpose stack,
               NOT monotonic — the LIFO property itself is the
               algorithm, same as Valid Parentheses (Day 39)
```

---

## Self-Check (15 min)

Pick **one** of the following, attempted cold — blank file, no notes, no looking back at this week's books until you're either done or genuinely stuck:

- **Linked List Cycle II** (Day 36) — can you still reproduce the phase-2 distance-math proof, not just the two-phase code?
- **Reorder List** (Day 38) — can you still name the three techniques being chained, without needing to look them up?
- **LRU Cache** (Day 39) — can you still design it from the two requirements (O(1) lookup, O(1) recency-tracking) rather than recalling the HashMap+DLL shape as a memorized template?

If it comes back cleanly, that's a real signal the week's foundational work is solid. If it doesn't, that's useful information now, at low stakes — better to surface a gap on a Sunday than mid-interview. Whichever happens, it's worth being honest about it in the diagnostic list at the end of today's consolidation, below.

---

## Part 1 — Daily Temperatures

### Problem 6: Daily Temperatures (LeetCode 739, Medium) — Pattern: Monotonic Decreasing Stack

**Statement:** Given daily temperatures, return an array where each entry is the number of days you'd have to wait for a *warmer* temperature — `0` if no future day is ever warmer.

**What's being reused, stated first:** this is Day 41's exact index-based Monotonic Stack mechanism from Next Greater Element II — the only difference is what gets stored as the answer. Instead of recording the *value* that resolved a stack entry, record the **day-gap** (`i - j`) between the resolving day and the day being resolved.

```java
public int[] dailyTemperatures(int[] temperatures) {
    int n = temperatures.length;
    int[] answer = new int[n];
    Deque<Integer> stack = new ArrayDeque<>();   // indices, decreasing temps bottom to top

    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
            int j = stack.pop();
            answer[j] = i - j;
        }
        stack.push(i);
    }
    return answer;
}
```

**Worked trace, `[73,74,75,71,69,72,76,73]`:**

| i | temp | stack before (indices) | pops → `answer[j] = i-j` | stack after |
|---|---|---|---|---|
| 0 | 73 | `[]` | — | `[0]` |
| 1 | 74 | `[0]` | pop 0 → answer[0]=1 | `[1]` |
| 2 | 75 | `[1]` | pop 1 → answer[1]=1 | `[2]` |
| 3 | 71 | `[2]` | none (75 not < 71... i.e. 71 isn't > 75) | `[2,3]` |
| 4 | 69 | `[2,3]` | none | `[2,3,4]` |
| 5 | 72 | `[2,3,4]` | pop 4 → answer[4]=1; pop 3 → answer[3]=2 | `[2,5]` |
| 6 | 76 | `[2,5]` | pop 5 → answer[5]=1; pop 2 → answer[2]=4 | `[6]` |
| 7 | 73 | `[6]` | none | `[6,7]` |

Stack ends with `[6,7]` unresolved — `answer[6]` and `answer[7]` stay at their default `0`. **Final: `[1,1,4,2,1,1,0,0]`** — matching the well-known expected result for this exact input.

**Complexity: Time O(n), Space O(n)** — identical amortized argument to every Monotonic Stack problem so far this week.

**💡 Interview Insight:** worth naming immediately — "this is the same pattern as Next Greater Element, computing a distance instead of a value" — since a candidate who has to re-derive the whole approach from scratch on a problem this close to one already solved is a weaker signal than one who recognizes the reuse instantly.

---

## Part 2 — Evaluate Reverse Polish Notation

### Problem 7: Evaluate Reverse Polish Notation (LeetCode 150, Medium) — Pattern: Stack

**Statement:** Evaluate an arithmetic expression given in postfix (Reverse Polish) notation — operators follow their operands (`"2 1 +"` means `2 + 1`). Division truncates toward zero.

**How this differs from this week's Monotonic Stack problems, worth naming explicitly:** there's no invariant being maintained here — this is a *general-purpose* stack, the same role it played in Valid Parentheses (Day 39): the LIFO property itself directly *is* the algorithm, since the two most recently seen operands are always exactly the ones the next operator applies to.

```java
public int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();

    for (String token : tokens) {
        if (token.equals("+") || token.equals("-") || token.equals("*") || token.equals("/")) {
            int right = stack.pop();   // pushed most recently — the SECOND operand, "b" in "a b op"
            int left = stack.pop();    // pushed before that — the FIRST operand, "a" in "a b op"
            if (token.equals("+")) {
                stack.push(left + right);
            } else if (token.equals("-")) {
                stack.push(left - right);
            } else if (token.equals("*")) {
                stack.push(left * right);
            } else {
                stack.push(left / right);
            }
        } else {
            stack.push(Integer.parseInt(token));
        }
    }

    return stack.pop();
}
```

**Why the pop order matters — proof by example, not assertion, since `+`/`*` wouldn't reveal a bug here but `-`/`/` will.** Trace `["4","13","5","/","+"]`, representing `4 + (13 / 5)`:

| token | stack before | action | stack after |
|---|---|---|---|
| 4 | `[]` | push | `[4]` |
| 13 | `[4]` | push | `[4,13]` |
| 5 | `[4,13]` | push | `[4,13,5]` |
| `/` | `[4,13,5]` | `right=5`, `left=13` → `13/5=2` (truncated) | `[4,2]` |
| `+` | `[4,2]` | `right=2`, `left=4` → `4+2=6` | `[6]` |

**Result `6`, correctly matching `4 + (13/5) = 4 + 2 = 6`.** Now the counter-check: if `right`/`left` were assigned in the opposite order (first pop as `left`, second as `right`), the division step would compute `5/13 = 0` instead of `13/5 = 2` — silently producing `4 + 0 = 4`, a **wrong answer with no crash or exception to reveal the bug.** This is exactly why the pop order needs to be stated and justified, not just "whichever happens to compile."

**Complexity: Time O(n) — one pass, O(1) work per token. Space O(n)** worst case, if many operands are pushed before the first operator appears.

**Edge cases:** Java's native integer division already truncates toward zero (matching the problem's requirement directly, with no extra handling needed) — worth knowing this isn't true in every language (Python's `//` floors instead, which differs from truncation for negative results). A single-token input (just one number, no operators) pushes once and returns it directly.

**💡 Interview Insight:** naming the truncation behavior as "Java's default `/` already does what this problem wants" is a small but real signal of language fluency — knowing when the standard library's default behavior happens to already match a requirement, rather than reflexively adding defensive code for a case that doesn't need it.

---

## Career Block Guide (1 hr)

**Weekly Industry Awareness Ritual (20 min):** clear the TLDR Newsletter backlog; read one engineering blog post in full, not skimmed.

**Weekly Scorecard:** see the full Week 6 Consolidation below — the numbers there are this week's scorecard, in the same "planned vs. actual" shape the plan itself has used at each week's close.

---

## Day 42 — Interview Questions

**Q1. What's identical between Daily Temperatures and Next Greater Element II, and what's the one thing that changes?** Both use an index-based, decreasing Monotonic Stack with the identical amortized O(n) push-once/pop-at-most-once argument. The only difference is what's recorded when a stack entry resolves — Daily Temperatures records the day-gap (`i - j`); Next Greater Element II records the resolving value itself.

**Q2. Why is Evaluate RPN's stack usage fundamentally different in kind from this week's Monotonic Stack problems?** There's no ordering invariant being maintained — the stack is used purely for its LIFO property, since the two most recently seen operands are always exactly what the next operator should apply to. It's the same *kind* of stack usage as Valid Parentheses, not a Monotonic Stack problem at all.

**Q3. Why does swapping the pop order in Evaluate RPN produce a wrong answer instead of a crash?** Both pop calls succeed regardless of order — nothing throws. Addition and multiplication are commutative, so the bug is invisible on those tokens, but subtraction and division are order-sensitive, and the swapped order computes the mathematically wrong (but syntactically valid) result silently.

**Q4. How many DSA problems did Week 6 add to the running cumulative total, and how many were genuinely new versus recapped?** 14 required slots were filled this week, but only 12 were newly solved — 2 (Valid Parentheses, Implement Queue using Stacks) were recaps of Week 1 solves, already counted before this week began. Including 4 extra-practice problems, Week 6 added **16 genuinely new distinct problems** to the running total.

**Q5. What closed this week, and what opened?** Linked Lists closed at 11 required + 4 extra = 15 distinct problems total. Stacks/Monotonic Stack opened, currently at 7/12 required (2 of which were recaps) + 2 extra = 9 distinct so far, continuing into Week 7.

---

## Daily Deliverable Check

- [ ] Self-check completed honestly — one Linked List problem attempted cold, result noted (clean recall or a real gap) in the diagnostic list below.
- [ ] Daily Temperatures (LC 739) and Evaluate Reverse Polish Notation (LC 150) solved, pushed to `dsa-java/stacks/`.
- [ ] Can state why RPN's pop order matters with a concrete example, not just "order matters."
- [ ] Weekly ritual complete (TLDR backlog cleared, one engineering blog post read).
- [ ] `Week6_Interview_Questions.md` reviewed end to end — not just today's slice.
- [ ] `00_Curriculum_Map.md` confirmed updated with this week's problem table rows, concept index entries, and running totals.

---

# Week 6 Consolidation

## What Actually Got Built

**DSA:** Linked Lists closed — Cycle Detection (Floyd's, and its extension to finding the cycle's start), fixed-offset two pointers, math simulation on a list, a three-technique combination (Reorder List), HashMap-based node cloning against a new pointer shape, and a doubly linked list built specifically to close the pattern with LRU Cache. Stacks/Monotonic Stack opened — LIFO fundamentals formalized (already informally in use since Week 1), then a full Monotonic Stack sub-pattern: the decreasing invariant proven (not just asserted), applied across a plain array, a circular array, and a stateful stream of calls (Online Stock Span), plus its mirror-image increasing variant (Remove K Digits).

**`todo-api`:** went from a REST-only skeleton to a genuinely persistent service — a real `@Entity` mapped to PostgreSQL via Spring Data JPA (Day 36), then a versioned Flyway migration replacing auto-generated schema changes with an auditable, immutable history (Day 40).

**`java-fundamentals`:** a full second pass through concurrency, this time about correctness and coordination rather than just execution order — a genuine, provable data-corruption bug fixed with `ReentrantLock` (Day 37), coordinated waiting via `Condition` in a working Producer-Consumer system (Day 38), and a concrete, measured demonstration of why `ConcurrentHashMap`'s bucket-level locking outperforms a single whole-map lock (Day 39).

**New theory, start to finish:** Spring Data JPA / ORM mechanics, `synchronized`/`ReentrantLock`, `wait()`/`notify()`/`Condition`, `ConcurrentHashMap`/CAS, and SQL fundamentals (keys, normalization, joins, indexes) — five substantial topics in five days, each building on confirmed prior material (Day 34's reflection, Day 29's thread model, Day 15's HashMap internals) rather than starting cold.

## Planned vs. Actual

| | Planned (per `Week_06_Revised.md`) | Actual |
|---|---|---|
| Required DSA problems | 14 (2/day × 7 days) | 14 slots filled — **12 newly solved, 2 recapped** (LC 20, LC 232 — both already solved Week 1, flagged since then) |
| Extra practice problems | 0 specified | **4** — LC 202, LC 287 (Day 36, Floyd's generalized); LC 901, LC 402 (Day 41, Monotonic Stack, using the day's built-in schedule slack) |
| New distinct problems added to the running total | 14 (if every slot were new) | **16** (12 new-required + 4 extra; the 2 recaps don't add to the distinct count — they were already there) |
| Required-ladder cumulative (Weeks 1–6) | — | **84** — matches `Week_06_Revised.md`'s own Day 42 claim exactly (70 at the end of Week 5 + 14 required slots this week) |
| Distinct-total cumulative (Weeks 1–6) | — | **111** (95 at the end of Week 5 + 16 new this week) |
| Theory/Project blocks | 7 days' worth (one day, 41, deliberately has neither, per the original plan) | Matched exactly — Day 41's missing blocks were preserved, not silently filled in, and its freed time was spent on extra Monotonic Stack practice instead |

**Two deliberate prerequisite insertions, not in the plan's own text, flagged as they happened:** a brief `synchronized` primer before `ReentrantLock` (Day 37 — needed as a baseline for the comparison the plan's own theory content makes), and doubly linked lists taught from scratch before LRU Cache (Day 39 — every prior linked-list day used only a single `next` pointer).

## Diagnostic List

Worth an honest check against each of these before treating the week as fully closed:

- [ ] The Cycle II distance-math proof (`a = (n−1)c + (c−b)`) reproducible without looking it up.
- [ ] Reorder List nameable as "three known techniques, chained" without hesitation.
- [ ] LRU Cache designable from its two requirements, not recalled as a memorized shape.
- [ ] The Monotonic Stack amortized-O(n) argument ("pushed once, popped at most once") reproducible on demand, for any of this week's four applications of it (array, circular, streaming, or the increasing variant).
- [ ] The `while`-not-`if` rule for `wait()`/`await()`, and both independent reasons behind it, stated without notes.
- [ ] `ReentrantLock`'s real advantages over `synchronized` listed without claiming reentrancy itself as one of them.
- [ ] The `LEFT JOIN`-produces-`NULL` vs. `INNER JOIN`-excludes-the-row distinction, stated precisely, not just "they're different."
- [ ] Today's self-check result — honestly recorded, whichever way it went.

## What Week 7 Assumes

Stacks/Monotonic Stack continues directly into its Hard tier (Asteroid Collision, Basic Calculator II, Largest Rectangle in Histogram, Maximal Rectangle, Basic Calculator) — every one of those builds on this week's decreasing-invariant proof and amortized argument being fully solid, with no re-derivation planned. Trees begins fresh, leaning on Week 2's recursion fundamentals and this week's (and Week 5's) comfort with self-referential node classes — the same shape, applied to two children instead of one `next` pointer. On the engineering side, Week 7 introduces Docker, Docker Compose, JUnit 5/AssertJ, Mockito, TestContainers, and Kafka — an entirely fresh run of theory with no direct prerequisite from this week's concurrency or SQL content, but every one of those tools operates directly on `todo-api` as it stands today: a real JPA-backed entity, a Flyway-versioned schema, and — starting Week 7 — containerized, properly tested infrastructure around both.
