# Day 45 — Stacks Capstone, and JUnit 5/AssertJ

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 44 Resource Book](Day44_Resource_Book.md)
**Next ▶:** [Day 46 Resource Book](Day46_Resource_Book.md)
**Companion to:** Day 45 of `Week_07_Revised.md`

---

## Recap

Today closes Stacks/Monotonic Stack at 12 of 12 required problems with the hardest of the three calculator-family problems — Basic Calculator adds parentheses on top of Day 43's Basic Calculator II, which means the stack now has to save and restore context across nested scopes, not just track a running computation. Nothing about the LIFO mechanism itself is new; what's new is *what* gets pushed. The theory block then pivots entirely: JUnit 5 and AssertJ open a fresh thread that Day 46's Mockito builds on directly tomorrow.

## Learning Objectives

By the end of today, without notes:

1. Solve Basic Calculator, and explain precisely why a naive left-to-right scan that ignores parentheses gives the wrong answer.
2. State exactly what gets pushed onto the stack at `(` and how it's used at `)`.
3. Recite the full Stacks/Monotonic Stack ladder — all 14 problems, general-purpose LIFO vs. true monotonic-invariant — well enough to place a new, unseen stack problem into one of those two buckets.
4. Write a JUnit 5 test class using `@Test`, `@BeforeEach`, and `@ParameterizedTest`, with AssertJ fluent assertions.

## Concept Dependency Map

```
Day 43 — Basic Calculator II: stack of terms, corrected in place on */÷
        │
        ▼
LC 224 Basic Calculator — SAME stack-of-state idea, but now the thing
being saved/restored is an entire (result, sign) CONTEXT, one per
level of paren nesting
        │
        ▼
**Stacks/Monotonic Stack CLOSES: 12/12 required + 2 extra (Week 6:
LC 901, 402) = 14 distinct total**
        │
        ▼
NEW: JUnit 5 — @Test / @BeforeEach / @AfterEach / @ParameterizedTest;
AssertJ fluent assertions (assertThat(x).isEqualTo(y))
        │
        ▼
Day 46 (tomorrow) — Mockito, extends JUnit 5's structure directly
```

---

## Problem 12: Basic Calculator (LeetCode 224, Hard) — Pattern: Stack

**Statement:** Evaluate a string expression containing non-negative integers, `+`, `-`, and parentheses (no `*` or `/`).

### The broken naive attempt — and why it fails

A tempting first instinct, especially right after Basic Calculator II: strip the parentheses entirely and evaluate left to right, the same way Day 43's `lastSign` trick did.

```java
// BROKEN — ignores what parentheses actually mean
public static int calculateBroken(String s) {
    s = s.replace("(", "").replace(")", "");
    int result = 0, sign = 1, num = 0;
    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '+' || c == '-') {
            result += sign * num;
            num = 0;
            sign = (c == '+') ? 1 : -1;
        }
    }
    return result + sign * num;
}
```

Trace `"2-(5-6)"`: stripping parens gives `"2-5-6"`, which evaluates to `2 - 5 - 6 = -9`. The correct answer is `2 - (5 - 6) = 2 - (-1) = 3` — off by 12. **The bug:** a `-` sign immediately before a `(` needs to flip the sign of *every* term inside those parentheses, not just the first one. Naively dropping the parens applies the outer `-` only to the number immediately following it (`5`), then treats the inner `-6` as if it were still at the outer nesting level with its own independent sign — which is exactly wrong, since that inner `-` was relative to the *paren's own* local context, not the outer expression's.

### Optimized: stack saves `(result, sign)` context

```java
public static int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int result = 0;
    int sign = 1;
    int num = 0;

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (Character.isDigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '+') {
            result += sign * num;
            num = 0;
            sign = 1;
        } else if (c == '-') {
            result += sign * num;
            num = 0;
            sign = -1;
        } else if (c == '(') {
            stack.push(result);     // save what we'd built so far...
            stack.push(sign);       // ...and the sign that applies to the WHOLE upcoming group
            result = 0;
            sign = 1;
        } else if (c == ')') {
            result += sign * num;   // finalize the last number inside this group
            num = 0;
            result *= stack.pop();  // apply the group's outer sign to the ENTIRE group at once
            result += stack.pop();  // add back whatever had been built before this '('
        }
    }
    result += sign * num;           // finalize any trailing number after the loop
    return result;
}
```

**Why this fixes the bug above:** on `(`, the *entire result built so far* and the *sign that was pending* both get pushed, and `result`/`sign` reset to start the group fresh, as if it were its own independent sub-expression. On `)`, the group's own result is finalized first, then multiplied by the sign that was pending *before* the `(` was ever seen — which is exactly what correctly propagates a `-` across every term inside the group, not just the first one — and finally added back onto whatever had been accumulated before entering the group.

**Trace:** `s = "2-(5-6)"`.

| i | c | action | result | sign | num | stack (bottom→top) |
|---|---|---|---|---|---|---|
| 0 | `2` | digit | — | 1 | 2 | `[]` |
| 1 | `-` | finalize 2: `result += 1*2` | 2 | −1 | 0 | `[]` |
| 2 | `(` | push result(2), push sign(−1); reset | 0 | 1 | 0 | `[2, −1]` |
| 3 | `5` | digit | 0 | 1 | 5 | `[2, −1]` |
| 4 | `-` | finalize 5: `result += 1*5` | 5 | −1 | 0 | `[2, −1]` |
| 5 | `6` | digit | 5 | −1 | 6 | `[2, −1]` |
| 6 | `)` | finalize 6: `result += (−1)*6 → 5−6=−1`; `×pop(−1)→1`; `+pop(2)→3` | 3 | −1 | 0 | `[]` |

End of loop: `result += sign*num = 3 + (−1×0) = 3`. **Correct** — matches `2 − (5 − 6) = 3` exactly, unlike the broken version's `−9`.

**Complexity: Time O(n) — one pass. Space O(n)** worst case — `n` levels of nested parentheses each push two values onto the stack before any are popped.

**Edge cases:** nested parentheses more than one level deep (each level pushes its own `(result, sign)` pair — the stack depth *is* the nesting depth, which is exactly why a stack, not two fixed variables, is required); a `(` immediately preceded by nothing (implicit leading `+`, handled correctly since `sign` starts at 1); spaces anywhere (skipped — they don't match any branch and leave `num`/`result`/`sign` untouched); a negative number is never a token in this problem's input format — negation only ever happens via a leading `-` operator, never a `-` glued directly onto a digit.

> 💡 **Interview Insight:** If asked "how is this different from Basic Calculator II," the precise answer is: II needs to defer a *value* (a pending multiplication or division); this needs to defer an entire *evaluation context* (a result-so-far and a sign) across a nested scope, which is a qualitatively different reason to reach for a stack, even though the container is the same. Being able to articulate that distinction, not just solve both problems, is what a tier-1 interviewer is actually listening for.

---

## Stacks/Monotonic Stack, Reviewed

**Closed today at 12/12 required + 2 extra practice = 14 distinct problems**, spanning Weeks 1, 6, and 7. Two families, worth being able to sort a new problem into on sight:

**General-purpose LIFO** — the stack holds state that needs correcting or restoring, no ordering invariant on the values themselves:

| LC | Problem | First taught |
|---|---|---|
| 20 | Valid Parentheses | Day 4 (Week 1) — recapped Day 39 |
| 232 | Implement Queue using Stacks | Day 4 (Week 1), originally extra practice, folded into the required ladder — recapped Day 40 |
| 155 | Min Stack | Day 41 (Week 6) |
| 150 | Evaluate Reverse Polish Notation | Day 42 (Week 6) |
| 735 | Asteroid Collision | Day 43 (Week 7) |
| 227 | Basic Calculator II | Day 43 (Week 7) |
| 224 | Basic Calculator | Day 45 (Week 7) — today |

**True monotonic stack** — the stack enforces an increasing or decreasing invariant on the values themselves, which is what makes "next greater/smaller" and boundary-area questions solvable in one pass:

| LC | Problem | First taught |
|---|---|---|
| 496 | Next Greater Element I | Day 40 (Week 6) |
| 503 | Next Greater Element II (circular) | Day 41 (Week 6) |
| 901 | Online Stock Span (extra) | Day 41 (Week 6) |
| 402 | Remove K Digits (extra) | Day 41 (Week 6) |
| 739 | Daily Temperatures | Day 42 (Week 6) |
| 84 | Largest Rectangle in Histogram | Day 44 (Week 7) |
| 85 | Maximal Rectangle | Day 44 (Week 7) |

**The signal that separates them:** does the problem ask about a value's relationship to its *neighbors by position* (next greater, span, boundary) — reach for monotonic. Does it ask you to evaluate, simulate, or reverse an *ordered sequence of operations* — reach for general-purpose LIFO.

---

# Part 2 — JUnit 5 and AssertJ

### Prerequisites (confirmed)

None from DSA — like Docker, this is a fresh theory thread. It does assume `todo-api`'s layered structure (controller → service → repository, live since Day 34, persisted via JPA since Day 36) already exists, since today's project writes tests against it.

### `@Test` and the Test Lifecycle

```java
class TaskRepositoryTest {

    @BeforeEach
    void setUp() {
        // runs before EVERY @Test method in this class — fresh state, no leakage between tests
    }

    @Test
    void savesAndRetrievesATask() {
        // one independent test case
    }

    @AfterEach
    void tearDown() {
        // runs after EVERY @Test method — cleanup, even if the test failed
    }
}
```

`@BeforeEach`/`@AfterEach` exist because tests must be independent — a test that only passes when it happens to run after another specific test is a hidden, fragile coupling. Running setup fresh before every single test method (rather than once per class) is what guarantees that independence.

### `@ParameterizedTest`

```java
@ParameterizedTest
@ValueSource(strings = {"", "   ", "\t"})
void rejectsBlankTaskTitles(String blankTitle) {
    assertThatThrownBy(() -> taskService.create(blankTitle))
        .isInstanceOf(IllegalArgumentException.class);
}
```

Runs the same test body once per supplied value, instead of writing three near-identical `@Test` methods that differ only in their input. This matters beyond convenience: with three separate `@Test` methods, a shared bug in the test logic itself would need fixing in three places; with one `@ParameterizedTest`, there's exactly one body to get right.

### AssertJ — Fluent Assertions

```java
// JUnit 5's built-in assertions
assertEquals(3, tasks.size());
assertTrue(tasks.get(0).isComplete());

// AssertJ, chained
assertThat(tasks)
    .hasSize(3)
    .first()
    .matches(Task::isComplete);
```

Both are correct and both work. AssertJ's advantage is that a chain of fluent assertions reads close to a plain-English description of what's actually being verified, and multiple related conditions about the *same* object combine into one chain instead of several disconnected `assertEquals` calls — which also means AssertJ can report a much more specific failure message (e.g., naming exactly which element of a list violated which condition) than a bare `assertTrue` can.

### Common Mistakes

- ⚠️ **Shared mutable state between test methods** — relying on one test's side effects (e.g., a row it inserted) being present in a *later* test. Test execution order isn't guaranteed, and `@BeforeEach` exists specifically to prevent this dependency from ever needing to exist.
- ⚠️ **One assertion buried in a print statement or a comment, instead of an actual `assert*` call** — a test with no assertion always "passes," silently testing nothing.
- ⚠️ **Testing implementation details instead of behavior** — asserting on a private field's exact value rather than the class's observable behavior makes tests break on harmless refactors.

---

# Project Block Guide (1.5 hrs)

**Repository:** `todo-api`. The plan names `@DataJpaTest` for testing `TaskController` — worth being precise here before writing anything: `@DataJpaTest` configures an embedded H2 database and scans only your `@Entity` classes and Spring Data JPA repositories; it does **not** load your web layer at all, so it can't actually exercise `TaskController`'s request/response handling. Testing the controller itself properly would call for `@WebMvcTest` (with `MockMvc`, mocking the service layer) or a full `@SpringBootTest`. What `@DataJpaTest` *is* exactly right for is verifying the persistence layer `TaskController` ultimately depends on — `TaskRepository` and your `Task` entity mapping — which is almost certainly what today's task actually needs: confirming that what gets saved is what comes back out, against a real (if embedded) database rather than mocks.

**Definition of done:** a `@DataJpaTest` class exercising `TaskRepository` — save, findById, findAll, delete — against embedded H2, with AssertJ assertions throughout; `mvn test` passes.

---

# Career Block Guide (1 hr)

### LinkedIn

Engagement day — 20 minutes commenting on 3–5 posts in your network.

### Networking

Identify 5 target Tier C companies for early interview practice — a slightly larger batch today, to build a real pipeline of low-stakes reps before Tier A/B applications start mattering more.

---

# Day 45 — Interview Questions

**Q1. Why does the naive "strip the parentheses and evaluate left to right" approach fail on `"2-(5-6)"`?**
A `-` sign immediately before a `(` needs to flip the sign of every term inside those parentheses, not just the first one. Stripping the parens and evaluating left to right only negates the first number after the `-`, then treats everything after it as if it were still at the outer nesting level — giving `2-5-6=-9` instead of the correct `2-(5-6)=3`.

**Q2. Precisely, what gets pushed onto the stack when a `(` is encountered, and why both values?**
The result accumulated so far, and the sign that was pending before the `(`. Both are needed on the way back out: the pending result is what the finished group gets added onto, and the pending sign is what the *entire* finished group must be multiplied by — which is exactly what correctly propagates a `-` across every term inside the group.

**Q3. What happens at a `)`, in order, and why that specific order?**
First, the last number inside the group is finalized into the group's local `result`. Then that local result is multiplied by the sign popped off the stack (the sign that applied to the whole group). Then the result popped off the stack (what had been built before the `(`) is added back on. Multiplying by the group's sign has to happen before adding back the outer context, or the outer sign would incorrectly apply to the outer context too.

**Q4. How does Basic Calculator's use of the stack differ from Basic Calculator II's, given both are "Stack — general purpose"?**
Basic Calculator II defers a single *value* — a term that a later `*` or `/` might still need to correct. Basic Calculator defers an entire *evaluation context* — a result-so-far and a sign — one full context per level of paren nesting, which is why its stack can grow proportionally to nesting depth rather than to term count.

**Q5. Sort Daily Temperatures and Evaluate Reverse Polish Notation into the two Stack families from today's review, and justify each.**
Daily Temperatures is a true monotonic stack — it maintains a decreasing-height invariant to answer "how far to the next greater element," a positional-neighbor question. Evaluate RPN is general-purpose LIFO — the stack just holds pending operands with no ordering invariant on their values; it's simulating an ordered sequence of operations, not comparing neighbors.

**Q6. Why does `@BeforeEach` run before every single `@Test` method, rather than once per test class?**
Tests must be independent of each other and of execution order, which isn't guaranteed by JUnit. Running setup fresh before every test method guarantees no test can accidentally depend on state left behind by a previous one.

**Q7. What's the actual advantage of `@ParameterizedTest` over three separate `@Test` methods with the same body and different inputs?**
Beyond fewer lines, it means there's exactly one copy of the test logic to get right and to maintain — three separate near-identical methods risk a bug or an update being fixed in one copy and silently missed in the other two.

**Q8. What does `@DataJpaTest` actually configure, and why can't it be used to test `TaskController` directly?**
It spins up an embedded database and scans only `@Entity` classes and Spring Data JPA repositories — it does not load the web layer (controllers, request mapping) at all. Testing `TaskController`'s own behavior needs `@WebMvcTest` or `@SpringBootTest`; `@DataJpaTest` is the right tool for the persistence layer underneath the controller, not the controller itself.

**Q9. Give one concrete advantage of AssertJ's `assertThat(...).hasSize(3).first().matches(...)` chain over the equivalent separate JUnit assertions.**
When one link in the chain fails, AssertJ can report specifically which condition on which element failed, in language close to the assertion's own intent — separate `assertEquals`/`assertTrue` calls each report in isolation and don't carry that same connected context about what larger property was actually being checked.

---

# Daily Deliverable Check

- [ ] Basic Calculator (LC 224) solved — the naive failure mode explained, not just the working code — pushed. **Stacks/Monotonic Stack ladder complete: 12/12 required + 2 extra = 14 distinct.**
- [ ] `@DataJpaTest` suite for `TaskRepository` passing, with AssertJ assertions — and the `@DataJpaTest`-vs-`TaskController` distinction understood.
- [ ] 20 minutes of LinkedIn engagement completed.
- [ ] 5 target Tier C companies identified.

---

## What Tomorrow Assumes You Already Know Cold

Day 46 assumes today's JUnit 5 structure (`@Test`, the lifecycle annotations) is solid, since Mockito's `@Mock`/`@InjectMocks` slot directly into a JUnit 5 test class rather than introducing a new test-running mechanism. On the DSA side, Day 46 assumes none of the Stack lineage directly — Trees is a genuinely new pattern — but it assumes recursion (Day 8) is fully reflexive, and that the self-referential single-`next`-pointer shape from Linked Lists (Weeks 5–6) is comfortable enough to extend, tomorrow, to a node with *two* references instead of one.
