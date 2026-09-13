# Day 64 Resource Book — Combination Sum, Its Duplicate-Handling Twin, and Resilience4j

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 63 Resource Book](Day63_Resource_Book.md)
**Next ▶:** [Day 65 Resource Book](Day65_Resource_Book.md)
**Companion to:** Day 64 of `Week_10_Revised.md`

---

## Recap

Day 63 closed Week 9 with Backtracking sitting at 4/12 required problems — Subsets (include/exclude), Permutations (swap-based), Combinations (forward-index), and Permutations II (forward-index... actually swap-based + a duplicate-skip layered on top: `!used[i-1]`, proven via a full `[1,1,2]` trace). Zero extra practice was added anywhere in Week 9 — Backtracking's own opening day (Day 61) correctly withheld extras per this series' standing precedent, and Days 62–63 withheld them too, for reasons specific to each day (a dense Week 10 required ladder already ahead, and one live collision — LC 17 — actually caught and avoided, not just a hypothetical worry).

Today continues that ladder: two more required problems (5 and 6 of 12), both genuinely new, both building directly on Day 63's forward-index and duplicate-skip machinery rather than introducing a new backtracking shape from scratch. **A week-level note worth stating up front:** Backtracking closes this week (Day 67) at a dense 12-problem required ladder that already spans every canonical variant this pattern has — include/exclude, swap-based, forward-index, with-repetition, with-duplicates (twice, via two different mechanisms), string-building, matrix-constrained, and full constraint-satisfaction (N-Queens). Given that density, and mirroring Week 9's own Tries precedent (a comprehensive required set closing with zero extra practice, reasoned explicitly rather than assumed), **no extra Backtracking practice is being added this week** — the reasoning is given in full when the pattern actually closes, Day 67.

On the platform side, today opens an entirely new theory thread — **Resilience4j and circuit breakers** — the first of four straight days building real microservices resilience and gateway patterns directly into `scalable-ecommerce-platform`.

---

## Learning Objectives

By the end of today, without notes:

1. Solve Combination Sum and Combination Sum II, explaining brute force, optimized backtracking, and — for the second problem specifically — *prove*, not just state, why the duplicate-skip condition is `i > start`, not `i > 0`.
2. Explain precisely how Combination Sum's "reuse the same element" rule changes exactly one line of Day 63's forward-index template, and name that line.
3. Explain what a circuit breaker actually does mechanically — the three states, the specific cascading-failure mode it exists to prevent — and connect its proxy-based implementation directly to Day 62's Spring AOP mechanism.
4. Configure and verify a working `@CircuitBreaker` on a Spring service method, including a fallback.

---

## Concept Dependency Map for Today

```
Day 61 — Backtracking template (choose → explore → un-choose)
Day 63 — Forward-index recursion (Combinations, LC 77)
Day 63 — Duplicate-value suppression, swap-based (Permutations II, LC 47)
        │
        ▼
Today, Problem 1 — Combination Sum (LC 39)
  forward-index template + ONE new wrinkle: elements may repeat
  (recurse WITHOUT advancing the index when an element is taken)
        │
        ▼
Today, Problem 2 — Combination Sum II (LC 40)
  Problem 1's shape + duplicate-value suppression, TRANSLATED from
  Day 63's swap/used[] mechanism into an index/start-pointer mechanism
  (since this template has no used[] array to check)
        │
        ▼
Day 62 — Spring AOP: proxy-based Advice, self-invocation limitation
        │
        ▼
Resilience4j Circuit Breaker (NEW) — a proxy-intercepted annotation,
  mechanically the same proxy shape as Day 62's AOP, applied to a new
  problem: cascading failure between services, not cross-cutting logging
```

---

# Part 1 — Combination Sum (LeetCode 39, Medium) — Pattern: Backtracking

**Statement:** Given an array of **distinct** positive integers `candidates` and a target integer `target`, return all unique combinations of `candidates` where the chosen numbers sum to `target`. The **same number may be chosen from `candidates` an unlimited number of times.** Two combinations are unique if the frequency of at least one chosen number differs.

## Why this isn't Day 63's Combinations problem wearing a new name

Combinations (LC 77, Day 63) picks exactly `k` numbers from `1..n`, no repeats allowed, order doesn't matter. Combination Sum picks *any number* of values from `candidates`, summing to an exact target, and **explicitly allows reusing the same value any number of times.** That single rule change — repetition allowed — is the entire delta from Day 63's forward-index template. Everything else (never revisit an *earlier* index, so no duplicate combination is ever produced in two different orders) transfers unchanged.

### Approach 1 — Brute force: generate every multiset, filter by sum

A "brute force" here isn't really a separate algorithm so much as backtracking with no early termination: explore every possible combination of any length using any candidate any number of times, and check the sum only once a branch is fully built. This is worth naming as a concept even though the code below *is* the eventual real solution's skeleton, because the real optimization isn't a different algorithm — it's **pruning**: stopping a branch the instant its running sum exceeds the target, rather than building the whole thing first and checking at the end.

```java
// Naive: explores every branch to completion before checking the sum. No pruning.
public static List<List<Integer>> combinationSumNaive(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    backtrackNaive(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private static void backtrackNaive(int[] candidates, int target, int start,
                                    List<Integer> path, List<List<Integer>> result) {
    int sum = path.stream().mapToInt(Integer::intValue).sum();   // recomputed every call — wasteful
    if (sum == target) {
        result.add(new ArrayList<>(path));
        return;
    }
    if (sum > target || start == candidates.length) {
        return;
    }
    for (int i = start; i < candidates.length; i++) {
        path.add(candidates[i]);
        backtrackNaive(candidates, target, i, path, result);   // i, not i+1 — reuse allowed
        path.remove(path.size() - 1);
    }
}
```

**Why this is worth calling out as the "brute force," despite being real backtracking code already:** recomputing `sum` from scratch on every call is O(path length) of wasted work, every single call. The optimized version below fixes this the same way Day 21's Prefix Sum avoided recomputing a range sum from scratch — carry a running value (here, `remaining target`) through the recursion instead of recalculating it.

### Approach 2 — Optimized: track `remaining`, prune negative branches

```java
public static List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private static void backtrack(int[] candidates, int remaining, int start,
                               List<Integer> path, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(path));   // found a valid combination
        return;
    }
    if (remaining < 0) {
        return;   // overshot — prune this branch immediately, don't explore further
    }
    for (int i = start; i < candidates.length; i++) {
        path.add(candidates[i]);                                   // choose
        backtrack(candidates, remaining - candidates[i], i, path, result);  // explore — i, NOT i+1
        path.remove(path.size() - 1);                               // un-choose
    }
}
```

**The one-line delta from Day 63's Combinations, made explicit:** Day 63's forward-index template recursed with `i + 1` — once an index was used, it could never be chosen again on that path, which is exactly right when each element is available exactly once. Here, the recursive call passes **`i`, not `i + 1`.** That's the entire "repetition allowed" rule, expressed as a single changed argument: candidate `i` remains eligible to be chosen *again* on the very next recursive call. The loop still starts from `start` (never an earlier index than the current position), which is what still guarantees `[2,2,3]` is only ever generated once, in strictly non-decreasing index order — never also as `[2,3,2]` or `[3,2,2]`.

**Pruning via `remaining < 0`:** this is *not* the same thing as sorting the input (the problem doesn't require it, and this solution doesn't sort). It's a plain early-exit: the moment the running total overshoots, every number left in `candidates` is positive, so nothing later in this branch can ever bring the sum back down — continuing to explore is guaranteed wasted work. This check turns "explore everything, filter at the end" into "stop the instant a branch is provably dead," which is the real difference between the naive and optimized versions above; the *shape* of the recursion tree they explore is otherwise identical.

### Worked trace: `candidates = [2,3,6,7]`, `target = 7`

```
backtrack(remaining=7, start=0, path=[])
├─ i=0, take 2 → path=[2], backtrack(remaining=5, start=0, path=[2])
│    ├─ i=0, take 2 → path=[2,2], backtrack(remaining=3, start=0, path=[2,2])
│    │    ├─ i=0, take 2 → path=[2,2,2], backtrack(remaining=1, start=0)
│    │    │    ├─ i=0, take 2 → remaining=-1 → PRUNE
│    │    │    ├─ i=1, take 3 → remaining=-2 → PRUNE
│    │    │    ├─ i=2, take 6 → remaining=-5 → PRUNE
│    │    │    └─ i=3, take 7 → remaining=-6 → PRUNE
│    │    │    (path=[2,2,2] un-chosen back to [2,2])
│    │    ├─ i=1, take 3 → path=[2,2,3], backtrack(remaining=0, start=1)
│    │    │    → remaining==0 → ADD [2,2,3] ✓
│    │    ├─ i=2, take 6 → remaining=-3 → PRUNE
│    │    └─ i=3, take 7 → remaining=-4 → PRUNE
│    ├─ i=1, take 3 → path=[2,3], backtrack(remaining=2, start=1)
│    │    ├─ i=1, take 3 → remaining=-1 → PRUNE
│    │    ├─ i=2, take 6 → remaining=-4 → PRUNE
│    │    └─ i=3, take 7 → remaining=-5 → PRUNE
│    ├─ i=2, take 6 → remaining=-1 → PRUNE
│    └─ i=3, take 7 → remaining=-2 → PRUNE
├─ i=1, take 3 → path=[3], backtrack(remaining=4, start=1)
│    ├─ i=1, take 3 → path=[3,3], backtrack(remaining=1, start=1) → all four children prune
│    ├─ i=2, take 6 → remaining=-2 → PRUNE
│    └─ i=3, take 7 → remaining=-3 → PRUNE
├─ i=2, take 6 → path=[6], backtrack(remaining=1, start=2) → both children prune
└─ i=3, take 7 → path=[7], backtrack(remaining=0, start=3)
     → remaining==0 → ADD [7] ✓
```

**Result: `[[2,2,3], [7]]`** — matches the known correct output for this exact classic input. Notice `start` never decreases as the path deepens (0 → 0 → 0/1 → 1 → 2 → 3), which is the forward-index guarantee doing its job: `[3,2,2]` is never explored as a separate branch from `[2,2,3]` at all, not filtered out after the fact.

### Complexity

**Time: O(2ⁿ)** where — stated precisely, not left vague — a tighter, fully rigorous bound treats this as a tree with branching factor bounded by the number of candidates and depth bounded by `target / min(candidates)` (the most times the smallest candidate could possibly be reused before overshooting): worst case **O(k^(target/min(candidates) + 1))**, where `k = candidates.length`. The plan's `O(2ⁿ)` shorthand is the common, looser way this gets cited — treating each step along a path as roughly a binary "keep extending this branch or don't" decision — and is fine to say out loud as an order-of-magnitude signal, but the `k^(target/min)` form is the one to have ready if an interviewer asks you to actually justify the bound.

**Space: O(target/min(candidates))** — the recursion depth, which is also `path`'s maximum size (excluding the output list itself).

> 🔑 **Key Takeaway:** "repetition allowed" in a backtracking problem is almost always exactly this: recurse with `i`, not `i + 1`. It's one of the smallest-looking changes in this entire pattern family and one of the easiest to get backwards under pressure — say it out loud before coding so a wrong index doesn't slip in silently.

### Common Mistakes and Edge Cases

- ⚠️ **Recursing with `i + 1` out of habit** (carried over from Day 63's Combinations) — silently forbids reuse, producing a strict-subset-and-wrong answer with no runtime error to flag it.
- ⚠️ **Checking `remaining == 0` only at the very end of the loop**, instead of at the top of every call — misses combinations early and can cause redundant continued exploration past a valid stopping point.
- ⚠️ **Forgetting the `remaining < 0` prune** — still produces a correct final answer (the `remaining == 0` check alone is sufifcient for correctness) but loses the entire practical benefit of pruning, degrading real-world runtime substantially on larger inputs.
- Edge case: no combination sums to target → return an empty list, not `null` — the loop simply never hits `remaining == 0`.
- Edge case: a single candidate exactly equals target → found on the very first `i=0` branch, one-element combination.
- Edge case: `target` smaller than every candidate → every branch prunes immediately at depth 1.

> 💡 **Interview Insight:** State the "repetition allowed → recurse without advancing the index" rule *before* writing code, explicitly contrasted against Combinations' `i+1`. An interviewer who has seen a hundred of these is listening for exactly that one sentence — it signals you understand *why* the index changes, not that you memorized a template.

---

# Part 2 — Combination Sum II (LeetCode 40, Medium) — Pattern: Backtracking with Duplicate Handling

**Statement:** Given a **possibly-duplicate-containing** array `candidates` and a target, return all unique combinations summing to target, where **each number may be used at most once this time** (positionally — if `candidates` has two `1`s, each can be used once, in different combinations, but not both in the same combination twice from the *same* index).

## Why this needs a genuinely new mechanism, not just "remove the reuse line"

Removing today's Problem 1 wrinkle (recurse with `i+1` instead of `i`, restoring one-use-per-position) gets you *most* of the way there — but not all the way. `candidates` here can contain **duplicate values** (e.g., `[1,1,2]`), and naively running Combinations' plain forward-index template over an array with duplicate values produces duplicate *combinations* in the output — not because the code is wrong about which indices to use, but because two different indices holding the *same value* are, from the output's perspective, indistinguishable, and the template has no way to know that yet.

This is the same *category* of problem Day 63's Permutations II (LC 47) solved with `!used[i-1]` — but that mechanism was built for a **swap-based** choice model with an explicit `used[]` boolean array. Today's template is **forward-index-based**, with no `used[]` array at all (eligibility is just "index ≥ `start`"). The idea transfers; the code does not. Today's version is the one Day 67's Subsets II will cite directly, since Subsets II uses this exact same forward-index shape.

### Approach — Sort, then skip a repeated value at the same recursion depth

```java
public static List<List<Integer>> combinationSum2(int[] candidates, int target) {
    Arrays.sort(candidates);   // REQUIRED — duplicate values must be adjacent for the skip check to work
    List<List<Integer>> result = new ArrayList<>();
    backtrack(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private static void backtrack(int[] candidates, int remaining, int start,
                               List<Integer> path, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(path));
        return;
    }
    if (remaining < 0) {
        return;
    }
    for (int i = start; i < candidates.length; i++) {
        if (i > start && candidates[i] == candidates[i - 1]) {
            continue;   // skip a repeated value AT THIS RECURSION DEPTH
        }
        path.add(candidates[i]);
        backtrack(candidates, remaining - candidates[i], i + 1, path, result);   // i+1 — each used once
        path.remove(path.size() - 1);
    }
}
```

**The condition, precisely: `i > start`, not `i > 0`.** This is the exact detail worth being able to defend, because the two look almost identical and only one is correct.

- `start` is *this specific recursive call's* own loop starting point — it moves deeper as the path grows (0 → 1 → 2 → ...).
- `i > start` asks: "am I past the very first candidate offered *at this recursion depth*?" If yes, and this candidate equals the previous one, the previous identical value was already tried as *this exact position's* choice, and its entire subtree was already fully explored — trying the same value again here would just re-walk an equivalent sibling branch, producing a duplicate combination.
- If `i == start`, this is the *first* candidate being offered at this depth — it must always be allowed, even if an identical value was used at a **shallower** depth (a different, earlier position in the path). Two different positions in the combination are allowed to hold the same duplicate value; only *repeating a value as a sibling choice at the same position* is forbidden.

`i > 0` would incorrectly compare against index `0` globally, at every recursion depth — not against `start`, which moves. That distinction only shows up once recursion goes at least one level deep, so it's exactly the kind of bug that can look like it works on a shallow manual check and then fail for real.

### Trace proving `i > start` is correct and `i > 0` is not

`candidates = [1,1,2]` (sorted), `target = 4`.

**Correct version (`i > start`):**

```
backtrack(remaining=4, start=0, path=[])
  i=0 (i>start? 0>0 false — never skipped): take 1 → path=[1]
    backtrack(remaining=3, start=1, path=[1])
      i=1 (i>start? 1>1 false — never skipped): take 1 → path=[1,1]
        backtrack(remaining=2, start=2, path=[1,1])
          i=2 (i>start? 2>2 false): take 2 → path=[1,1,2]
            backtrack(remaining=0, start=3) → ADD [1,1,2] ✓
      i=2 (i>start? 2>1 TRUE; candidates[2]=2 ≠ candidates[1]=1 → no skip): take 2 → path=[1,2]
        backtrack(remaining=1, start=3) → loop empty, no result
  i=1 (i>start? 1>0 TRUE; candidates[1]=1 == candidates[0]=1 → SKIP)
  i=2 (i>start? 2>0 TRUE; candidates[2]=2 ≠ candidates[1]=1 → no skip): take 2 → path=[2]
    backtrack(remaining=2, start=3) → loop empty, no result
```

**Result: `[[1,1,2]]`** — the one genuinely valid combination, found exactly once. The `i=1` branch at the top level (trying to start a fresh combination with the *second* `1`) is correctly skipped — it would only ever rediscover `[1,1,2]` a second time via a different, redundant path.

**The buggy version, run for real (`i > 0` in place of `i > start`):** at the deeper call `backtrack(remaining=3, start=1, path=[1])`, index `i=1` now incorrectly evaluates `i > 0` → `true`, then checks `candidates[1] == candidates[0]` → `1 == 1` → **true**, so it wrongly skips `i=1` — even though `i == start` at this depth, meaning it's the legitimate *first* choice being offered here, not a repeated sibling. That skip prevents `path=[1,1]` from ever being built at all, which means `[1,1,2]` — the only correct answer for this input — is **never found**. Running both versions confirms this exactly: the correct version returns `[[1,1,2]]`; the `i > 0` version returns an empty list.

### Complexity

**Time: O(2ⁿ)**, same shorthand and same caveat as Problem 1 above — more precisely bounded by the size of the (now duplicate-pruned) recursion tree, at most `2^n` where `n = candidates.length`, since each element is now used at most once, giving a genuine binary "in or out" ceiling per element (this bound *is* tight in the include/exclude sense, unlike Problem 1's repetition-driven tree). **Space: O(n)** — recursion depth is now bounded by the array length itself, not by `target/min`, since each element is usable at most once.

### Common Mistakes and Edge Cases

- ⚠️ **`i > 0` instead of `i > start`** — proven above to silently drop valid combinations that legitimately reuse a duplicate value across two *different* path positions.
- ⚠️ **Forgetting to sort first** — the skip check assumes equal values are adjacent; on unsorted input it simply doesn't catch the duplicates it's meant to, and the output silently contains repeats.
- ⚠️ **Recursing with `i`, not `i + 1`** — this is Problem 1's line, carried over by habit; here it would incorrectly allow reusing the *same index* and produce Problem 1's repetition-allowed behavior on a problem that explicitly forbids it.
- Edge case: all candidates identical (e.g., `[2,2,2]`, target `4`) — the skip logic still produces exactly one `[2,2]`, not three.
- Edge case: no valid combination — empty result, same as Problem 1.
- Edge case: a candidate larger than target on its own — pruned immediately via `remaining < 0`.

> ⚠️ **Common Mistake:** conflating this skip condition with Day 63's `!used[i-1]` and trying to reuse that exact code shape here. They solve the same *category* of duplicate-suppression problem but are mechanically distinct — `!used[i-1]` inspects a `used[]` array asking "is the previous identical value currently part of my active path," which only makes sense in a swap-based model; `i > start` inspects the *loop position itself*, which only makes sense in a forward-index model. Know which choice model a given backtracking problem uses before reaching for either.

> 💡 **Interview Insight:** This problem is a strong vehicle for demonstrating you can *translate* a technique across a different code shape, not just recall it. Naming the connection to Permutations II *and* correctly explaining why the code differs is a stronger signal than silently producing a correct `i > start` check with no acknowledgment of where the idea came from.

---

# Part 3 — Circuit Breakers with Resilience4j

## Prerequisites, confirmed

This section needs REST/Spring Boot fundamentals (Day 34), the multi-module `scalable-ecommerce-platform` structure (Day 62), and — directly — Day 62's Spring AOP: proxy-based Advice and the self-invocation limitation. All three are established and unchanged.

## The failure mode this exists to prevent

Every service call made so far in this series — `todo-api`'s own REST endpoints, its database queries — has one property in common: a monolith calling its own code fails in a monolith's usual ways (a thrown exception, a bug), but it doesn't fail because *some unrelated part of the system* is slow. A system built from multiple independently-deployed services (`scalable-ecommerce-platform`'s four modules) introduces a genuinely new failure mode: **one dependency being slow or down can exhaust the *caller's* own resources while it waits**, and that exhaustion cascades to whatever was calling *the caller*.

Concretely: if the Order module calls a slow downstream dependency and that call takes 30 seconds to time out, every thread handling an Order request that happens to hit this code path is now blocked for 30 seconds, holding a thread (and whatever connection/resources it owns) the whole time. Under real load, this doesn't stay contained — a fixed-size thread pool fills up with threads stuck waiting on the *one* slow dependency, and requests that have *nothing to do with* that dependency start failing too, purely because there are no threads left to handle them. This specific failure shape — one slow dependency starving an entire service of the resources needed to do anything else — is called **cascading failure**, and it's the reason circuit breakers exist.

## The mechanism: three states, not a binary on/off

A circuit breaker watches the failure rate of calls to a specific dependency and transitions between three states:

- **CLOSED** (normal operation): calls pass through to the real dependency as usual; failures are counted.
- **OPEN**: once the failure rate crosses a configured threshold, the breaker "trips" — it stops attempting the real call *entirely* for a configured wait duration, immediately returning a fallback (or failing fast) instead. This is the actual fix for the cascading-failure mode above: no thread blocks waiting on the struggling dependency, because no call to it is even attempted.
- **HALF_OPEN**: after the wait duration elapses, the breaker allows a small number of *trial* calls through. If they succeed, it transitions back to CLOSED (the dependency has recovered); if they still fail, it returns to OPEN and waits again.

> 🔑 **Key Takeaway:** the entire point of OPEN state is that it gives the struggling dependency **room to recover** instead of continuously hitting it with an ever-growing pile of retried, waiting requests — which is often exactly what *keeps* a recovering service from ever actually recovering. A circuit breaker is a form of self-protective backing-off, applied automatically rather than left to whatever ad-hoc retry logic each caller happens to write.

## Configuration and fallback, in Spring

```java
@Service
public class OrderService {

    private final ProductClient productClient;   // stands in for today's slow dependency

    public OrderService(ProductClient productClient) {
        this.productClient = productClient;
    }

    @CircuitBreaker(name = "productService", fallbackMethod = "getProductFallback")
    public ProductInfo getProductInfo(String productId) {
        return productClient.fetchProduct(productId);   // the real, potentially-slow call
    }

    // Fallback signature requirement: same return type, same parameters,
    // PLUS one additional trailing Throwable parameter.
    public ProductInfo getProductFallback(String productId, Throwable t) {
        return ProductInfo.unavailable(productId);   // a safe, immediate default
    }
}
```

```yaml
resilience4j.circuitbreaker:
  instances:
    productService:
      failure-rate-threshold: 50        # % of calls that must fail to trip OPEN
      minimum-number-of-calls: 3        # don't evaluate the rate until this many calls have happened
      wait-duration-in-open-state: 10s  # how long OPEN lasts before trying HALF_OPEN
      permitted-number-of-calls-in-half-open-state: 2
```

**Why the fallback method's signature must match exactly:** Resilience4j needs to locate `getProductFallback` by reflection at startup and confirm it's call-compatible with `getProductInfo` — same parameters (so it can be invoked with the same arguments the real call would have received) plus a trailing `Throwable` (so the fallback can inspect *why* the real call failed, if it needs to). A signature mismatch is a runtime configuration error, not a compile error, since the matching happens via reflection rather than the compiler checking an interface contract — worth testing directly rather than assuming a plausible-looking method name is enough.

## Why this is Day 62's AOP mechanism again, not a new one

**This is the citation worth making explicitly, not leaving implicit:** `@CircuitBreaker` works exactly the way Day 62's Spring AOP `@Transactional`-style Advice worked — a **proxy** sits between the caller and the real `OrderService` bean, intercepting the call to `getProductInfo` *before* it reaches the real method. The proxy is what actually tracks the failure count, decides whether the circuit is OPEN or CLOSED, and redirects to the fallback when appropriate — none of that logic lives inside `OrderService` itself. This has the exact same direct consequence Day 62 named for `@Transactional`: **self-invocation bypasses it.** If some other method *inside* `OrderService` called `this.getProductInfo(...)` directly, that call would never pass through the proxy at all, and the circuit breaker would simply never activate — the call would go straight to the real (possibly failing) dependency every time, with no protection. This is the same proxy-boundary gotcha, showing up for a second, structurally different cross-cutting concern.

## When to reach for this vs. its nearest alternative

| | Plain try/catch with a hardcoded fallback | Circuit Breaker |
|---|---|---|
| Stops attempting the real call once it's clearly failing? | No — every single call still tries, and still pays the full timeout cost, even during a sustained outage | Yes — OPEN state skips the real call entirely |
| Gives the failing dependency room to recover? | No | Yes — by design |
| Adapts automatically as the dependency's health changes? | No — static, always-on fallback logic | Yes — CLOSED/OPEN/HALF_OPEN transitions based on live failure rate |

A bare try/catch handles *one call's* failure correctly, but does nothing to stop a *sustained* failure from repeatedly consuming full-timeout resources on every subsequent call — which is precisely the resource-exhaustion mechanism a circuit breaker exists to interrupt.

### Common Mistakes

- ⚠️ **Assuming the circuit breaker prevents the *first* failure.** It doesn't — it needs `minimum-number-of-calls` failures to even evaluate the rate. Its job is stopping *sustained* failure from cascading, not preventing any single call from ever failing.
- ⚠️ **Calling the guarded method via `this.` from inside the same class** — silently bypasses the proxy entirely, identical to Day 62's `@Transactional` self-invocation trap.
- ⚠️ **Writing a fallback method with a mismatched signature** — fails at runtime (or silently isn't picked up), not at compile time, since the match happens via reflection.

> 💡 **Interview Insight:** A tier-1 interviewer asking about resilience patterns is very often specifically listening for the phrase "cascading failure" and the resource-exhaustion mechanism behind it — not just "it retries" or "it's a fallback." Being able to state precisely *what breaks without one* (a slow dependency exhausting the caller's thread pool) is what separates "has used the annotation" from "understands why it exists."

---

## Project Block

**Repository:** `scalable-ecommerce-platform`.

**Task:** in the Order module, create a standalone endpoint that mimics a slow third-party API using `Thread.sleep(...)` to simulate latency past a reasonable timeout. Add `@CircuitBreaker` to a service method that calls it, configured to open after 3 failures, with a fallback response.

**Practical guidance:** set `minimum-number-of-calls: 3` and a low `failure-rate-threshold` so the breaker trips deterministically within a small number of test calls, rather than needing a large volume of traffic to observe the transition. Log the circuit breaker's state on each call (Resilience4j exposes this via `CircuitBreaker.getState()`) so opening is *visible*, not just inferred from response shape.

**Definition of done:** after 3 timeouts, the circuit opens and every subsequent call immediately returns the fallback response — verified by observing the logged state transition from CLOSED → OPEN, not just by seeing fallback responses appear.

---

## Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts in your network. Genuine, specific comments (referencing something concrete in the post) build far more real visibility than generic ones, the same guidance from Day 10's networking note.

**Networking:** apply to 2 more Tier C companies. Keep applications moving in parallel with the deeper prep work — Tier C roles are lower-stakes practice for the interview loop itself, ahead of Tier A/B applications later in the plan.

---

# Day 64 — Interview Questions

---

**1. What's the exact code-level difference between Day 63's Combinations template and today's Combination Sum?**

*Answer:* Combinations recurses with `i + 1` (each index usable once); Combination Sum recurses with `i` (the current index remains eligible again, since repetition is allowed). Every other part of the forward-index template — the loop starting at `start`, the choose/explore/un-choose shape — is unchanged.

---

**2. Why does the `remaining < 0` check matter for correctness vs. for performance?**

*Answer:* It doesn't affect correctness — the `remaining == 0` check alone would eventually find every valid combination regardless. It matters purely for performance: without it, the algorithm keeps exploring branches that are already provably dead (every remaining candidate is positive, so an already-negative remainder can never recover), doing significant wasted work.

---

**3. In Combination Sum II, what does the condition `i > start` actually test, in your own words?**

*Answer:* Whether the current candidate is the *first* one being offered at this specific recursion depth, or a *later sibling* choice at that same depth. Only later siblings that duplicate the immediately preceding value get skipped — the first choice at any depth is always allowed, regardless of what value appeared at a shallower depth.

---

**4. Why does `i > 0` (instead of `i > start`) produce a wrong answer, concretely?**

*Answer:* At any recursion depth past the first, `start > 0`, so `i > 0` is true even when `i == start` — the legitimate first choice at that depth. If that first choice's value happens to equal `candidates[i-1]` (a value from a *shallower*, unrelated position), it gets wrongly skipped, silently dropping valid combinations that reuse a duplicate value across two genuinely different path positions — proven directly against `[1,1,2]`, target `4`, where the correct answer `[1,1,2]` is never found under this bug.

---

**5. Why must `candidates` be sorted before Combination Sum II's skip logic works?**

*Answer:* The skip check compares `candidates[i]` to `candidates[i-1]` — the immediately preceding array slot. That only reliably catches duplicate *values* if equal values are guaranteed to sit adjacent to each other, which sorting guarantees and an arbitrary input order does not.

---

**6. Contrast Combination Sum II's duplicate-skip with Permutations II's `!used[i-1]` — same idea, why different code?**

*Answer:* Both suppress a duplicate value from being chosen as an equivalent sibling branch. Permutations II uses a swap-based choice model with an explicit `used[]` array, so it checks *array membership state* (`!used[i-1]`). Combination Sum II uses a forward-index model with no `used[]` array at all — eligibility is purely "index ≥ start" — so it checks *loop position* (`i > start`) instead. The underlying idea transfers across backtracking's different choice models; the exact code does not.

---

**7. What specifically breaks without a circuit breaker, mechanically — not just "it gets slow"?**

*Answer:* A slow or failing downstream dependency causes every calling thread that hits that code path to block waiting on it. Under sustained load, a fixed-size thread pool fills up entirely with threads stuck waiting on that one dependency, leaving no threads available to handle *any* other request — including ones unrelated to the failing dependency. This is cascading failure: one dependency's slowness starves the whole service of the resources needed to do anything else.

---

**8. Name the three circuit breaker states and what each one does.**

*Answer:* CLOSED — normal operation, calls pass through, failures are counted. OPEN — failure rate crossed the threshold; the real call is skipped entirely and a fallback returns immediately, giving the dependency room to recover. HALF_OPEN — after a wait period, a small number of trial calls are allowed through to test recovery; success returns to CLOSED, continued failure returns to OPEN.

---

**9. Why does self-invocation bypass `@CircuitBreaker`, and where has this exact issue appeared before in this series?**

*Answer:* `@CircuitBreaker` is implemented via a proxy that intercepts calls to the bean from *outside* it; a method calling another method on `this` within the same class never goes through that proxy. This is the identical self-invocation limitation Day 62 established for Spring AOP and `@Transactional` — both are proxy-based cross-cutting mechanisms with the same structural blind spot.

---

**10. What does a plain try/catch with a hardcoded fallback fail to do that a circuit breaker does?**

*Answer:* A try/catch handles each individual call's failure correctly but still *attempts* the real call every single time, paying the full cost (including any timeout) on every attempt even during a sustained outage. A circuit breaker's OPEN state stops attempting the real call at all once failure is established, which is what actually prevents resource exhaustion from a sustained failure — a try/catch alone does nothing to stop that.

---

## Daily Deliverable Check

- [ ] Combination Sum (LC 39) solved and understood — brute force, optimized, the "recurse with `i`" rule stated explicitly.
- [ ] Combination Sum II (LC 40) solved and understood — the `i > start` condition proven, not just applied, including why `i > 0` fails.
- [ ] Both problems pushed to `dsa-java/backtracking/`.
- [ ] Circuit breaker live in `scalable-ecommerce-platform`'s Order module — verified opening after 3 timeouts and correctly falling back, with the state transition actually observed (logged), not just inferred.
- [ ] LinkedIn engagement (3–5 comments) and 2 Tier C applications submitted.

---

## What Tomorrow Assumes You Already Know Cold

Day 65 assumes today's forward-index template is fully reflexive — Letter Combinations of a Phone Number extends it into a genuinely different shape (building a *string* one character at a time from per-position choices, rather than selecting a *subset* of array elements), so today's choose/explore/un-choose mechanics need to be automatic, not re-derived. It also assumes the Resilience4j proxy mechanism is solid, since tomorrow's Feign client work sits in the exact same "declarative annotation, real behavior generated via a proxy/reflection mechanism underneath" family — the specific new piece tomorrow is what Feign generates (an HTTP client), not the general shape of "annotate an interface, get real behavior for free."

**Next:** [Day 65 Resource Book](./Day65_Resource_Book.md) — Backtracking: Phone Letters and Parentheses, and Feign Clients.
