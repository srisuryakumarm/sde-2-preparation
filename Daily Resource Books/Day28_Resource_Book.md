# Day 28 (Sunday) — Consolidation, and Binary Search Begins

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 27 Resource Book](Day27_Resource_Book.md)
**Next ▶:** [Day 29 Resource Book](Day29_Resource_Book.md)
**Companion to:** Day 28 of `Week_04_Revised.md`

---

## Recap

Four weeks in, two entirely new pattern families closed this week alone (Prefix Sum & Kadane's at 9 distinct problems, Greedy & Intervals at 13), neither of which existed anywhere in the original 17-week plan. Today opens a third pattern — **Binary Search** — while simultaneously closing out Week 4 with a full consolidation. Today is also the lightest DSA day of the week (2 hours, two Easy problems) paired with the heaviest theory-and-project load since early in the leave week: a genuinely new Java language feature pair (`record`, `sealed interface`) that needs its own careful treatment, including one place where the plan's own stated Java version needs a precise correction.

---

## Learning Objectives

By the end of today, without notes:

1. Implement the exact-match binary search template — `mid = left + (right - left) / 2`, narrowing `[left, right]` by comparing `nums[mid]` to `target` — and explain why this specific overflow-avoiding form of `mid` matters, not just state it as a rule.
2. Solve First Bad Version by binary-searching a **boolean condition** rather than an array value, and articulate precisely how this differs from an exact-match search — the "on the answer" framing Week 5 develops much further.
3. Write a `record` and explain exactly what it generates for you (constructor, accessors, `equals`/`hashCode`/`toString`), and connect its auto-generated `equals`/`hashCode` back to the contract established in Week 1, Day 4.
4. Write a `sealed interface` restricting its implementers, and explain precisely which Java version is required for the compiler to check switch-exhaustiveness over that hierarchy without a `default` clause — including why the plan's "Java 17+" framing needs a small, precise correction here.

---

## Concept Dependency Map

```
Week 1, Day 3: sorting, arrays — Binary Search's only stated prerequisites
Week 2, Day 10: int overflow, exact ranges — DIRECTLY relevant to mid-calculation today
        │
        ▼
TODAY, Part 1 — Binary Search Begins
├─ LC 704 Binary Search — exact-match template, ON THE INPUT
└─ LC 278 First Bad Version — boundary search on a MONOTONIC CONDITION, ON THE ANSWER
   (previews Week 5, Days 32-33's much deeper "search the answer space" treatment)

Week 1, Day 2: interfaces, classes, constructors
Week 2, Day 11: String immutability
Week 1, Day 4 / Week 3, Day 15: equals()/hashCode() contract, established and deepened
Week 1, Day 1: switch (classic + arrow syntax)
        │
        ▼
TODAY, Part 2 — Records and Sealed Classes
├─ record — compact immutable data carrier (standard since Java 16)
├─ sealed interface ... permits — restricted hierarchy (standard since Java 17)
└─ Exhaustive switch over a sealed hierarchy — needs Java 21 specifically (see correction below)
        │
        ▼
Week 5: Binary Search continues immediately (9 more required problems, closing at 11),
        then Linked Lists begins, then Spring Boot initialization
```

---

## Self-Check (15 min)

Before today's new material: pick **one** problem from Prefix Sum & Kadane's and **one** from Greedy & Intervals — ideally not the two most recently solved (Partition Labels and whatever you did last are still fresh; pick something from earlier in the week instead, like Contiguous Array or Jump Game) — and re-solve each **cold**, no hints, no looking at your own prior code until you're done or genuinely stuck. This is the real test of whether this week's intensity actually stuck versus just being completed once under guidance. If either one requires re-deriving the core insight from scratch rather than recalling it, that's worth noting honestly in today's diagnostic below, rather than smoothing over.

---

# Part 1 — Binary Search Begins

**Concept Card — Binary Search**

- **What:** repeatedly halve a *sorted* search space by comparing its middle element to your target, discarding the half that provably can't contain the answer.
- **Why:** O(log n) instead of O(n) — for a billion elements, roughly 30 comparisons instead of up to a billion. Each comparison eliminates *half* of whatever remains, so the number of comparisons needed grows only logarithmically with input size.
- **Where:** any already-sorted structure, and — the less obvious, higher-leverage case — any problem where the **answer itself** is monotonic across some range of candidate values, letting you binary search on the *answer* instead of the input. Week 5, Days 32–33 develop this second case in real depth; today's First Bad Version is the first taste of it.
- **Interview signal:** "sorted array" for the direct case; "minimum/maximum value such that some condition holds" for the on-the-answer variant.
- **Prerequisites:** arrays (Week 1, Day 2) ✅. One implementation detail worth having automatic: `mid = left + (right - left) / 2`, not the more obvious-looking `mid = (left + right) / 2` — the reason is Week 2, Day 10's overflow mechanics, applied directly, not a new rule.

**🔗 Backward reference, made concrete:** Week 2, Day 10 established that `int` overflow wraps silently in Java, via two's complement, with no exception thrown. `left + right` can genuinely exceed `Integer.MAX_VALUE` when both are large — not a theoretical concern here, since problems like today's First Bad Version allow `n` up to `2³¹ - 1` — and if it does, the naive `(left + right) / 2` silently computes a garbage, possibly negative, midpoint. `left + (right - left) / 2` never sums two large values together at all; `(right - left)` is bounded by the size of the remaining search space, not by how large `left` and `right` individually are, so it can't overflow the same way.

---

## Problem: Binary Search (LeetCode 704, Easy)

**Statement:** given a sorted (ascending), zero-indexed array of **unique** integers `nums` and an integer `target`, return the index of `target` if it exists, otherwise `-1`.

### Approach 1 — Brute force: linear scan

```java
public static int searchLinear(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] == target) return i;
    }
    return -1;
}
```

Correct, and doesn't even need the array to be sorted — but O(n), ignoring the sortedness the problem hands you for free.

### Approach 2 — Optimized: binary search

```java
public static int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;    // target must be strictly right of mid, if present at all
        } else {
            right = mid - 1;   // target must be strictly left of mid, if present at all
        }
    }
    return -1;   // left > right — search space exhausted, target not present
}
```

**Why the loop invariant guarantees correctness:** at the start of every iteration, *if* `target` is present in `nums` at all, it's guaranteed to lie within `nums[left..right]` (inclusive on both ends) — true initially (the whole array), and preserved by construction on every narrowing step, since each branch only ever discards a half that's been proven (by the sortedness of `nums` and the comparison against `nums[mid]`) not to contain `target`. The loop terminates either by finding `target` directly, or by `left` exceeding `right`, at which point the invariant guarantees the search space is empty and `target` genuinely isn't present.

**Why `left <= right`, not `left < right`, is the correct loop condition here:** when `left == right`, there's still exactly one unchecked candidate index remaining — `nums[left]` itself — and it needs to be checked before concluding failure. Using `left < right` instead would skip checking that final single-element case in some inputs, a genuine off-by-one bug, not just a style choice.

**Worked trace:** `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`.

| left | right | mid | nums[mid] | comparison | action |
|---|---|---|---|---|---|
| 0 | 5 | 2 | 3 | 3 < 9 | left = 3 |
| 3 | 5 | 4 | 9 | **match** | return 4 |

Matches the known expected output for this exact input.

**Complexity:** Time O(log n) — each iteration halves the remaining search space. Space O(1) (iterative version; a recursive version would add O(log n) call-stack space, worth mentioning if asked for the recursive form).

**Edge cases:** empty array (`right = -1` immediately, loop condition `0 <= -1` is false, correctly returns `-1` without ever executing the loop body); single-element array (`left == right == 0` on entry, one comparison resolves it either way); target smaller than every element or larger than every element (the loop correctly narrows to an empty range and exits via `left > right`, without ever needing a special-case check beforehand).

💡 **Interview Insight:** stating the loop invariant explicitly — "if the target exists, it's always within `[left, right]`" — before writing any code is the single strongest thing to say on this problem; it's what makes the `<=` vs `<` boundary choice and the `mid±1` narrowing both provably correct rather than things that merely happen to work on the examples tried.

---

## Problem: First Bad Version (LeetCode 278, Easy) — Pattern: Binary Search on Answer

**Statement:** you're given `n` versions `[1, 2, ..., n]`, and an API `isBadVersion(version)` that returns `true` if that version is bad. Once a version is bad, every version after it is bad too (the "badness" is monotonic across the range). Find the **first** bad version, using as few calls to `isBadVersion` as possible.

**🔑 Key Takeaway, stated before any code:** this is not a search over a *given array* — there's no `nums` at all. It's a search over the **range of possible answers**, `[1, n]`, using a monotonic boolean condition (`isBadVersion`) to decide which half of that range to keep. The mechanics turn out nearly identical to LC 704 above, but the *thing being narrowed* is conceptually different — a preview of exactly the shift Week 5, Days 32–33 develop into full "binary search on the answer" problems (Koko Eating Bananas, Capacity To Ship Packages), where there's no array at all, only a range of candidate answers and a feasibility check.

### Approach 1 — Brute force: linear scan

```java
public static int firstBadVersionLinear(int n) {
    for (int version = 1; version <= n; version++) {
        if (isBadVersion(version)) return version;
    }
    return -1;   // unreachable per problem guarantees, but a defensible fallback
}
```

O(n) calls to `isBadVersion` in the worst case — correct, but the problem explicitly asks to minimize exactly this call count.

### Approach 2 — Optimized: binary search for the leftmost `true`

```java
public static int firstBadVersion(int n) {
    int left = 1, right = n;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (isBadVersion(mid)) {
            right = mid;        // mid COULD be the answer — keep it in range, don't exclude it
        } else {
            left = mid + 1;     // mid is definitely good — answer is strictly after it
        }
    }
    return left;   // left == right, converged on the first bad version
}
```

**Why this loop uses `left < right` (not `<=`) and `right = mid` (not `mid - 1`) — genuinely different from LC 704, and worth being precise about why:** this is searching for a **boundary** — the exact point where a monotonic sequence flips from `false` to `true` — not for an **exact match** against a known value. When `isBadVersion(mid)` is `true`, `mid` itself is a valid *candidate* for "the first bad version" (it might be exactly the boundary, or the boundary might be even earlier) — so it must **stay** in the search range rather than being excluded via `mid - 1`, which is why the update is `right = mid`. The loop condition becomes `left < right` (rather than `<=`) specifically because this narrowing scheme always converges to `left == right` pointing at the answer, rather than to a state where `left > right` signals "not found" — there's no "not found" case here at all, since a bad version is guaranteed to exist.

**Why this genuinely is a different template, not just a cosmetic variation of LC 704:** LC 704 searches for one specific value that may or may not be present, and correctly reports absence. First Bad Version searches for a **boundary** in a sequence that's guaranteed to look like `false, false, ..., false, true, true, ..., true` — there's no "not found" outcome to report, and the exact indices used for narrowing (`right = mid` vs. `right = mid - 1`) differ specifically because "mid might BE the answer" (First Bad Version) is a different situation from "mid definitely ISN'T the answer, having just been checked and ruled out" (LC 704's exact-match case, where a match returns immediately and every non-match is provably not the target).

**Worked trace:** `n = 5`, and suppose version `4` is the first bad one (so `isBadVersion` returns `false, false, false, true, true` for versions `1..5`).

| left | right | mid | isBadVersion(mid) | action |
|---|---|---|---|---|
| 1 | 5 | 3 | false | left = 4 |
| 4 | 5 | 4 | true | right = 4 |
| 4 | 4 | — | loop ends (left==right) | return 4 |

Matches the expected first-bad-version answer for this scenario.

**Complexity:** Time O(log n) calls to `isBadVersion`, the quantity the problem explicitly asks to minimize. Space O(1).

**Edge cases:** version `1` itself is the first bad one (the loop correctly narrows `right` down to `1` without ever needing a special pre-check, since `isBadVersion(1)` being `true` immediately sets `right = mid` at whatever the first computed `mid` happens to be, and subsequent iterations keep narrowing toward `1`); only the last version, `n`, is bad (symmetric — `left` keeps advancing via `left = mid + 1` until it reaches `n`); `n = 1` (loop condition `1 < 1` is false immediately, returns `1` without calling `isBadVersion` at all — correct, since with only one version, it must be the answer by the problem's own guarantee that a bad version exists).

⚠️ **Common Mistake:** copying LC 704's `right = mid - 1` narrowing onto this problem without adjusting for the fact that `mid` is still a live candidate here — doing so can skip over the true boundary entirely, since it wrongly treats a "this could be the answer" signal as a "this is definitely not the answer" signal.

---

# Part 2 — Records and Sealed Classes

### Prerequisites (confirmed)

- Classes, constructors, interfaces (Week 1, Day 2).
- Immutability, as a concept (Week 2, Day 11 — String internals).
- The `equals()`/`hashCode()` contract (Week 1, Day 4; deepened with the bucket mechanism, Week 3, Day 15).
- `switch`, including arrow syntax (Week 1, Day 1) — today extends it, doesn't re-teach it from zero.

### `record` — an immutable data carrier, generated for you

A class whose entire purpose is to hold a fixed set of values, immutably, is extremely common — and writing one by hand (constructor, private final fields, accessor methods, `equals`, `hashCode`, `toString`) is almost entirely repetitive boilerplate. `record` generates all of it from a single declaration:

```java
public record Point(int x, int y) {}
```

This one line generates, automatically:
- A canonical constructor: `Point(int x, int y)`.
- Two `private final` fields, `x` and `y` — genuinely immutable, no setters exist at all.
- **Accessor methods named after the fields directly** — `x()` and `y()`, **not** `getX()`/`getY()`. This is a real, easy-to-trip-on difference from the classic JavaBeans getter convention used everywhere else in this series so far (e.g., Week 4, Day 5's `Account.getBalance()`) — worth having this distinction automatic before it costs a compile error under time pressure.
- `equals()` and `hashCode()`, both based on **all** components — two `Point` records are `.equals()` if and only if every component matches.
- `toString()` — something like `Point[x=3, y=4]`.
- The class is **implicitly `final`** — a record can never be extended.

**🔗 Direct connection to an already-established contract:** Week 1, Day 4 established the `equals()`/`hashCode()` contract as a rule; Week 3, Day 15 deepened it into the actual bucket mechanism a violated contract breaks. A record's auto-generated `equals()`/`hashCode()` pair is **guaranteed to satisfy that contract correctly**, by construction, based on every declared component — it's not possible to accidentally write a record whose `equals()` and `hashCode()` disagree with each other the way a hand-written class's could.

**🔑 Key Takeaway:** a record is not "a class with less typing" as a style preference — the immutability and the auto-correct `equals()`/`hashCode()` pair are the actual point. Reach for a record specifically when a type's entire job is to **carry** a fixed set of values, not when it needs mutable internal state or inheritance from anything other than an interface.

**What a record *cannot* do, worth stating precisely:** it cannot extend another class (it implicitly extends `java.lang.Record`, already using Java's single-inheritance slot) — though it **can** implement any number of interfaces, exactly like an ordinary class. It also cannot declare additional instance fields beyond its components, though it can declare static fields, additional constructors that delegate to the canonical one, and additional methods.

---

### `sealed interface` — a hierarchy with a known, finite set of implementers

An ordinary `interface` can be implemented by literally anything, anywhere, including code you'll never see. `sealed` restricts that:

```java
public sealed interface PaymentState permits Pending, Success, Failed {}
```

**What `permits` guarantees:** only `Pending`, `Success`, and `Failed` are allowed to implement `PaymentState` — nothing else, ever, anywhere in the codebase, checked at compile time. Each permitted type must itself be declared `final` (cannot be extended further), `sealed` (can be extended, but only by its own further-restricted `permits` list), or `non-sealed` (reopens unrestricted extension from that point down) — the compiler enforces that every permitted subtype picks one of these three explicitly.

```java
public record Pending() implements PaymentState {}
public record Success(String transactionId) implements PaymentState {}
public record Failed(String reason) implements PaymentState {}
```

**A synergy worth noticing explicitly:** records are *always* implicitly `final` (established above), so every one of these three automatically satisfies `sealed`'s "must be final, sealed, or non-sealed" requirement with zero extra syntax — records and sealed interfaces compose especially cleanly together for exactly this reason, which is presumably why today's exercise pairs them.

---

### Exhaustive switch over a sealed hierarchy

```java
static String describe(PaymentState state) {
    return switch (state) {
        case Pending p -> "Payment is pending";
        case Success s -> "Payment succeeded: " + s.transactionId();
        case Failed f -> "Payment failed: " + f.reason();
    };
}
```

**Why no `default` clause is needed, or allowed to be missing without one of two conditions holding:** the compiler knows, from `PaymentState`'s own `permits` clause, that `Pending`, `Success`, and `Failed` are the **only** possible implementers that will ever exist. A `switch` expression using type patterns (`case Pending p -> ...`) over a sealed type is checked for **exhaustiveness** against that permitted list — if every permitted type has a `case`, the compiler accepts the switch with no `default` at all; if even one permitted type were missing a case, the code **fails to compile**, catching an entire category of bug — "I added a new payment state and forgot to handle it somewhere" — before the program ever runs, rather than discovering it via an unhandled case at runtime.

⚠️ **A precise correction to the plan's stated Java version, worth getting exactly right rather than gliding past:** the plan's definition of done says "compiles on Java 17+." Records are standard (non-preview) since **Java 16** (JEP 395); sealed classes/interfaces are standard since **Java 17** (JEP 409) — both of those parts of "Java 17+" are accurate. But the specific mechanism above — a `switch` using **type patterns** (`case Pending p -> ...`) that the compiler checks for exhaustiveness against a sealed hierarchy, with no `default` required — is **Pattern Matching for `switch`**, which was a *preview* feature across Java 17 through 20 (JEP 406, then 420, then 427, then 433, refining across releases) and became a **standard, non-preview feature only in Java 21** (JEP 441). Compiling exactly the code above, with no `default` clause and no `--enable-preview` flag, requires **Java 21 or later** — not Java 17–20. By August 2026, Java 21 is comfortably behind the current LTS release, so targeting 21+ for this exercise isn't a stretch in practice; it's simply the version where this specific feature stopped being a preview.

**Extension, beyond what's needed for today's exercise:** `switch` can also **deconstruct** a record's components directly in the case label — `case Success(String txId) -> "Payment succeeded: " + txId` — instead of binding the whole record and calling an accessor. This is a separate, related feature (**record patterns**, JEP 440), also finalized in Java 21 alongside pattern matching for switch. Worth knowing it exists; not required to complete today's exercise, which only needs type patterns.

### Common Mistakes

- ⚠️ **Calling `.getX()` on a record component out of JavaBeans habit.** Records use `.x()`, not `.getX()` — a real compile error, not a style nitpick, the first time this trips someone up.
- ⚠️ **Declaring a permitted subtype without `final`, `sealed`, or `non-sealed`.** The compiler requires one of the three explicitly on every direct permitted implementer of a sealed type — omitting all three is a compile error, though this is moot for records specifically, since they're final automatically.
- ⚠️ **Assuming "compiles on Java 17+" covers exhaustive switch pattern matching.** As corrected above — records and sealed types themselves are fine on 17+; exhaustiveness-checked type-pattern switches specifically need 21+.

---

## Project Block Guide (1 hr)

**Repository:** `java-fundamentals`. **Task:** a `ModernJava` package implementing the `PaymentState` sealed hierarchy above (or your own equivalent design following the same principles — a sealed interface with 2–4 record implementers and an exhaustive switch over it).

**Definition of done, corrected from the plan's stated version:** compiles on **Java 21+** (not 17+ — see the precise correction above), pure `record` syntax for every implementer, an exhaustive `switch` expression with genuinely **no** `default` clause (verify this by temporarily commenting out one `case` and confirming the build actually fails to compile — don't just assume the exhaustiveness check is working, watch it catch a real removed case); pushed.

---

## Career Block Guide (1 hr)

### Weekly Industry Awareness Ritual (20 min)

Clear the TLDR Newsletter backlog; read one engineering blog post — the same small, protectable habit established Week 1, Day 5, worth keeping consistent rather than letting it slide now that the leave week's intensity is ending.

### Weekly Scorecard — Day 28, Four Weeks In

**The plan's own internal checkpoint states "57 total DSA problems solved."** That figure — as `00_Curriculum_Map.md` already flags — tracks only the plan's own required ladder (25 through Day 14, +12 finishing Sliding Window, +18 for the leave week's required problems spanning both Prefix Sum/Kadane's and Greedy/Intervals, +2 Binary Search starters today), and doesn't include any extra practice added across this resource-book series. Here's the fuller, actual picture this series has been tracking:

| Metric | Count |
|---|---|
| Distinct problems solved, Weeks 1–3 (per `00_Curriculum_Map.md`) | 56 |
| Week 4 required problems, newly solved (excludes the LC 560 recap) | 16 |
| Week 4 extra practice, newly solved (LC 974, 152, 406, 986) | 4 |
| **Week 4 total, newly solved** | **20** |
| **Cumulative distinct problems solved, Weeks 1–4** | **76** |

**Pattern status:**
- **Prefix Sum & Kadane's — fully closed:** 7/7 required (3 Week 3, 4 Week 4) + 2 extra (both Week 4) = **9 distinct.**
- **Greedy & Intervals — fully closed:** 11/11 required (all Week 4) + 2 extra (both Week 4) = **13 distinct.**
- **Binary Search — opened today:** 2/11 required, 0 extra (deliberately deferred — the pattern is opening, not closing, mirroring exactly how Sliding Window and Prefix Sum/Kadane's each opened with zero extras). Closes at 11 required in Week 5, Day 33.

**Two entirely new pattern families — present nowhere in the original 17-week plan — fully closed this week, at 22 distinct problems between them, without slipping the overall 120-day timeline by a single day.**

---

# Week 4 Consolidation

## What Actually Got Built

- **Prefix Sum & Kadane's, closed:** the missing HashMap half of the pattern — first-occurrence-index tracking (LC 525, 523) versus frequency-count tracking (LC 560 recapped, LC 974 extra) — plus Kadane's extended to a circular array (LC 918, run twice) and to products (LC 152 extra, tracking max *and* min). 7 required + 2 extra = 9 distinct.
- **Greedy & Intervals, opened and closed in the same week:** formal greedy (exchange arguments, the 0/1 Knapsack counter-example), three distinct proof shapes across the week (frontier domination — Jump Game; prefix elimination — Gas Station; interval-swap exchange — Non-overlapping Intervals), the full Intervals sort-and-sweep mechanism in both its start-time (combine) and end-time (discard/cover) forms, a first application of the already-known min-heap to interval scheduling, and a capstone problem (Partition Labels) solving interval-shaped reasoning with no explicit interval object at all. 11 required + 2 extra = 13 distinct.
- **Binary Search, opened:** the exact-match template and its overflow-safe midpoint calculation (a direct callback to Week 2, Day 10), plus a first look at "binary search on the answer" via First Bad Version — a preview of what Week 5, Days 32–33 develop in full.
- **Two Java theory threads landed alongside the DSA work:** nothing new in the middle of the leave week itself (Days 22–26 were DSA-only, by design, at leave-week intensity), then a return to full Theory/Project/Career blocks on Days 27–28 — greedy-safety reflection, and `record`/`sealed interface`/exhaustive `switch`, including the Java 21 correction to the plan's stated version requirement.

## Planned vs. Actual

| | Planned (required ladder) | Actual (required + extra) |
|---|---|---|
| Prefix Sum & Kadane's | 7 | 9 |
| Greedy & Intervals | 11 | 13 |
| Binary Search (Day 28 only) | 2 | 2 |
| **Week 4 total** | **20** (17 new-required-slots + the 1 recap slot... see note) | **24** |

*Note on the "20" planned figure: the plan's Week 4 problem ladder has 17 slots (4+3+3+2+2+1+2, matching each day's "Problem N" labels), one of which (LC 560) turned out to already be solved — so 16 genuinely new required solves + 1 recap + 4 extra = 21 problems actually worked through this week, closing at 9+13+2=24 distinct problems total when the two carried-over Day-21 Prefix-Sum problems' worth of "planned" context is set aside and only Week 4's own new contribution is counted. The headline number that matters going forward is the **cumulative** one: 76 distinct problems solved across Weeks 1–4.*

## Diagnostic — Spot-Check Before Moving On

Work through these without notes; anywhere you hesitate is worth ten more minutes before Week 5 begins:

- [ ] Can you state, unprompted, when to store a first-occurrence index versus a frequency count in a prefix-sum-plus-HashMap problem?
- [ ] Can you explain Maximum Subarray Sum Circular's all-negative edge case, and *why* the naive formula breaks there, not just that it does?
- [ ] Can you give the 0/1 Knapsack counter-example showing exactly why greedy fails there, with actual numbers, not just "it doesn't always work"?
- [ ] Can you state, from memory, all three of this week's distinct exchange-argument shapes (domination, interval-swap, prefix-elimination) and which problem each belongs to?
- [ ] Can you explain why Meeting Rooms II's heap size, at the end, is already the answer — without needing to re-derive the monotonic-growth argument from scratch?
- [ ] Can you write `mid = left + (right - left) / 2` from memory and explain, precisely, what it avoids?
- [ ] Can you name the exact Java version where exhaustive switch pattern matching over a sealed hierarchy stopped being a preview feature?

## What Week 5 Assumes

Week 5 (`Week_05_Revised.md`) continues Binary Search immediately and intensively — 9 more required problems across Days 29–33, closing the pattern at 11 total, deliberately **not** including Median of Two Sorted Arrays (LC 4, Hard), which stays reserved for right after Trees complete, exactly as the *original* plan intended on its own Day 21 but never actually delivered. It assumes today's exact-match template and overflow-safe midpoint calculation are fully automatic, since Week 5 varies the template repeatedly (rotated arrays, 2D matrices, boundary search, and the "on the answer" variant First Bad Version previewed today) without re-deriving the base mechanism each time. Week 5 then pivots to Linked Lists (growing to 12 problems) and initializes the `todo-api` Spring Boot project — neither of which depends on anything from this week beyond general programming fundamentals long since automatic. Greedy & Intervals and Prefix Sum & Kadane's are both fully closed and won't be re-taught; if either diagnostic item above still feels shaky, that's worth resolving now rather than after Week 5's own new material begins stacking on top.

---

## Day 28 — Interview Questions

**Q1. Why is `mid = left + (right - left) / 2` preferred over `mid = (left + right) / 2`?** The latter can overflow `int` when `left` and `right` are both large, silently wrapping to a garbage (possibly negative) value with no exception, per Week 2, Day 10's overflow mechanics. `(right - left)` is bounded by the size of the remaining search space, not by how large `left` and `right` individually are, so it can't overflow the same way.

---

**Q2. Why does LC 704's binary search use `left <= right` as its loop condition, not `left < right`?** When `left == right`, there's still one unchecked candidate — `nums[left]` itself — that must be examined before concluding the target is absent. `left < right` would skip that final check in some inputs.

---

**Q3. How does First Bad Version's binary search differ mechanically from LC 704's, and why?** It searches for a *boundary* in a monotonic true/false sequence, not an exact match against a known value. Since `mid` might itself be the answer when `isBadVersion(mid)` is true, it must stay in the search range (`right = mid`, not `mid - 1`), and the loop uses `left < right` since the scheme converges to `left == right` pointing at the answer, with no "not found" outcome to signal.

---

**Q4. What does "binary search on the answer" mean, and which of today's two problems previews it?** Instead of searching a given sorted array, you search a *range of candidate answers*, using a monotonic feasibility check to decide which half to keep — there's no array involved at all. First Bad Version previews this: the "array" is conceptually the range `[1, n]`, and `isBadVersion` is the monotonic feasibility check.

---

**Q5. What exactly does declaring `record Point(int x, int y) {}` generate?** A canonical constructor, private final fields `x` and `y`, accessor methods `x()` and `y()` (not `getX()`/`getY()`), `equals()`/`hashCode()` based on all components, a `toString()`, and an implicitly `final` class that can never be extended.

---

**Q6. Why is a record's auto-generated `equals()`/`hashCode()` pair guaranteed to satisfy the Week 1, Day 4 contract?** Both are generated from the exact same set of components, by construction — there's no way to end up with a record whose `equals()` and `hashCode()` disagree with each other, unlike a hand-written class where the two could be implemented inconsistently by mistake.

---

**Q7. What does `sealed interface PaymentState permits Pending, Success, Failed` guarantee, and what must each permitted type declare?** Only the three named types may ever implement `PaymentState`, checked at compile time — nothing else, anywhere in the codebase. Each permitted type must itself be declared `final`, `sealed`, or `non-sealed`, explicitly.

---

**Q8. Why do records satisfy a sealed interface's "must be final/sealed/non-sealed" requirement automatically?** Records are always implicitly `final` — they can never be extended — so this requirement is met with zero extra syntax whenever every permitted implementer is a record.

---

**Q9. The plan states this exercise "compiles on Java 17+." What's imprecise about that, exactly?** Records (Java 16+) and sealed interfaces (Java 17+) are each individually accurate. But exhaustive switch pattern matching over a sealed hierarchy — a `switch` using type patterns with no `default`, checked for completeness against the `permits` list — was a preview feature across Java 17–20 and became standard only in Java 21 (JEP 441). Compiling the exercise exactly as specified, with no `default` and no `--enable-preview` flag, requires Java 21 or later.

---

**Q10. What closed this week, and what opened, in one sentence each?** Prefix Sum & Kadane's closed at 9 distinct problems (7 required + 2 extra); Greedy & Intervals closed at 13 distinct problems (11 required + 2 extra) — both entirely new pattern families with zero presence in the original 17-week plan; Binary Search opened at 2/11 required, with extra practice deliberately deferred until it closes in Week 5.

---

## Daily Deliverable Check

- [ ] Self-check complete: one Prefix Sum/Kadane's and one Greedy/Intervals problem re-solved cold, honestly assessed.
- [ ] Binary Search and First Bad Version solved, pushed to `dsa-java/binary-search/`.
- [ ] `ModernJava` `PaymentState` state machine pushed — compiling on Java 21+, exhaustive switch verified by temporarily breaking it and confirming the build fails.
- [ ] Weekly Industry Awareness Ritual complete.
- [ ] Weekly Scorecard reviewed — cumulative distinct-problem count (76) and both pattern closures (9 + 13) understood, not just the plan's own required-ladder figure.
- [ ] **Week 4 complete.** Leave week fully absorbed — back to standard daily pace from tomorrow.

---

## What Tomorrow Assumes You Already Know Cold

Day 29 (Week 5) continues Binary Search directly — Search Insert Position and Find Peak Element both extend today's exact-match template into new territory (an insertion-point variant, and a genuinely unsorted-array application) without re-explaining the base template from scratch. Day 29 also introduces Threads and the JVM's concurrency model as an entirely fresh theory thread, with no dependency on anything from this week. Today's overflow-safe midpoint calculation, the `left <= right` vs. `left < right` distinction, and the "on the input" vs. "on the answer" framing all need to be fully reflexive going in — Week 5 varies the template repeatedly across increasingly unfamiliar-looking problems (rotated arrays, 2D matrices, capacity/speed feasibility checks) specifically because the base mechanism is assumed solid enough to adapt rather than re-derive each time.

**Next:** Day 29 Resource Book — Binary Search Continues, and Threads/JVM Concurrency. *(Week 5's own generation.)*
