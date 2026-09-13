# Day 92 — State Machine DP Opens: Transaction Fee and Cooldown

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 91 Resource Book](Day91_Resource_Book.md)
**Next ▶:** [Day 93 Resource Book](Day93_Resource_Book.md)
**Companion to:** Day 92 of `Week_14_Revised.md`

---

## Recap

Yesterday (Day 91) closed Interval DP and, with it, all four of the DP subtypes that could be built without an explicit "which situation am I in" dimension — 1D, Grid, String, and Interval DP each compute `dp[state]` where "state" is just a position (an index, or a pair of indices). Week 13 Consolidation left DP at 28/35 required, with exactly two subtypes remaining: **State Machine DP** (today and tomorrow) and **Tree DP** (Day 94) — 6 required problems, closing Dynamic Programming entirely by Wednesday.

State Machine DP isn't arriving from nowhere. Two things already touched it without naming it:

- **0/1 Knapsack** (Day 85) — every `dp[capacity]` value implicitly encoded "best value achievable *given how much room is left*," which is a one-dimensional state. Today generalizes "room left" to any small, enumerable set of situations — *holding a share* vs. *not holding one*, for instance.
- **Predict the Winner** (Day 91) — the score-difference framing tracked "whose turn it effectively is" as an implicit second dimension riding along with the interval. Today makes that kind of dimension explicit and names it.

More directly, two problems you've already **solved** are about to get solved again, on purpose, through a different lens:

| LC # | Problem | Where | How it was solved then |
|---|---|---|---|
| 121 | Best Time to Buy and Sell Stock | Week 2, Day 14 | Sliding Window — single-pass min-tracking |
| 122 | Best Time to Buy and Sell Stock II | Week 4, Day 23 | Greedy — telescoping decomposition |

Neither of those techniques survives what today adds. That failure is the actual motivation for State Machine DP, and it's worth understanding precisely rather than taking on faith — see "Why Greedy Breaks" below.

---

## Learning Objectives

By the end of today, without notes:

1. Define a DP state machine precisely: what `dp[i][state]` means, and what makes a transition between states legal.
2. Explain, with a concrete counterexample, exactly where LC 122's greedy telescoping argument stops working once a fee or cooldown is introduced.
3. Derive and code the 2-state (`hold`/`cash`) machine for Transaction Fee (LC 714) from the recurrence up, including the space-optimized rolling-variable version.
4. Extend the 2-state machine to 3 states for Cooldown (LC 309), and justify *why* 2 states are insufficient here specifically.
5. Write a Java 21 exhaustive `switch` over a sealed hierarchy using record patterns and guarded case labels.

---

## Concept Dependency Map

```
DP foundations (Day 81: optimal substructure + overlapping subproblems)
0/1 Knapsack (Day 85, informal state preview) ─┐
Predict the Winner (Day 91, informal state preview) ─┤
                                                       ▼
LC 121 (Wk2 D14, solved via Sliding Window) ──┐   State Machine DP (NEW)
LC 122 (Wk4 D23, solved via Greedy) ───────────┤   dp[i][state], transitions = legal actions
  — both re-examined as unconstrained          │        │
    2-state machines with no swap-argument      │        ▼
    counterexample yet to break them             ├──▶ LC 714 (Transaction Fee) — 2 states
                                                  │        │
                                                  │        ▼
                                                  └──▶ LC 309 (Cooldown) — 3 states, extends 714

Records + sealed interfaces (Week 4, Day 28, Java 17)
        │
        ▼
Java 21 Pattern Matching (NEW) — record patterns, guarded switch cases
```

---

# Part 1 — State Machine Dynamic Programming

## Prerequisites (confirmed)

- Optimal substructure and overlapping subproblems (Day 81).
- 1D DP recurrence discipline — state a `dp[]` meaning precisely, then prove the recurrence by exhaustive, disjoint cases (Day 82 onward, most recently Day 91).
- Greedy exchange arguments and where they can fail (Week 4, Day 23's 0/1 Knapsack counterexample).

## What it is

Every DP subtype so far has had a `dp` array indexed by *position only* — an index into an array or string, or a pair of indices bounding an interval. **State Machine DP adds a second, small, enumerable dimension**: on day `i`, you are always in exactly one of a fixed, finite set of *states*, and the correct decision — and the correct value — depends on which one.

Formally: `dp[i][s]` = the best achievable outcome by day `i`, **given that you end day `i` in state `s`**. A transition from `dp[i-1][s']` to `dp[i][s]` is legal only if the action that moves you from `s'` to `s` is legal under the problem's rules, and the recurrence takes the best over every legal incoming transition:

```
dp[i][s] = max over every state s' with a legal transition s' → s of:
               dp[i-1][s'] + (the value gained/lost making that transition on day i)
```

This is still nothing but "best answer here, built from the best answer one step back" (Kadane's own shape, all the way back to Day 81) — the only new ingredient is that "here" now means "here, in this specific situation," not just "here, at this index."

## Why it works

Optimal substructure still holds, for the same reason it always has: the best way to be in state `s` on day `i` is fully determined by the best way to have been in *some* state on day `i-1`, plus today's transition — nothing about the future affects that. Overlapping subproblems still holds: many different action sequences reach state `s` on day `i`, and they all share the exact same optimal-future-decisions from that point forward, so solving `dp[i][s]` once and reusing it (rather than replaying every path that reaches it) is exactly the same computational win DP has delivered since fib(5).

The only genuinely new claim to justify is: **why do you need `s` in the index at all?** Because on some days, the *best* outcome achievable **isn't monotonic in the obvious way** unless you track which situation produced it — the best profit "if I currently hold a share" and the best profit "if I currently don't" are frequently maximized by two different, incompatible histories, and collapsing them into one number would silently let the recurrence use a history that isn't actually reachable given today's action.

## Why Greedy Breaks

LC 122's greedy solution sums every positive day-over-day delta: `profit = Σ max(0, prices[i] - prices[i-1])`. This is provably correct **only because you may buy and sell on the same day with no penalty** — the sum is a valid decomposition of any multi-day hold into consecutive single-day steps, and every step you'd skip (a negative delta) can simply be omitted, since nothing stops you from re-entering the very next day.

The moment either constraint below is added, that decomposition becomes invalid:

- **A fee per transaction (LC 714):** decomposing one long hold into many single-day steps multiplies the number of transactions, and therefore the total fee paid. The greedy sum silently assumes decomposition is *free* — it isn't anymore.
- **A cooldown (LC 309):** decomposing a hold into steps requires re-buying the very next day after each "sell" — which cooldown explicitly forbids. The decomposition isn't just costly now; it's often **illegal**.

Concretely: `prices = [1, 3, 2]`, fee `= 2`. Greedy would take the `1→3` step (`+2`) and skip `3→2` (negative), for a naive total of `2` — but paying the fee once for a single buy-at-1/sell-at-3 transaction nets `3 - 1 - 2 = 0`, and there's no better option (buying at 1 and holding to the end never beats selling at the local peak once, and any two-transaction plan pays the fee twice for less total spread). Either way, the *right* answer is `0`, and the plain greedy sum overstates it to `2` because it isn't accounting for the fee at all. This is precisely the "greedy's local choice can't be blindly trusted once a real constraint enters the picture" lesson from 0/1 Knapsack (Week 4, Day 23) resurfacing in a new shape.

**🔑 Key Takeaway:** State Machine DP isn't a different *topic* from what solved LC 121 and LC 122 — it's the general-purpose tool that both of those techniques turn out to be special, unconstrained cases of. Sliding-window min-tracking (LC 121) is the 2-state machine limited to exactly one buy/sell pair; greedy telescoping (LC 122) is the same 2-state machine with zero transaction constraints, where an exchange argument happens to prove the greedy shortcut equals the DP optimum. Add *any* constraint that breaks that exchange argument, and you need the machine itself, explicitly.

## When to reach for it

The signal: a problem describes a **sequence of steps** (days, rounds, positions) where at each step you make a **discrete choice among a small, fixed set of options**, and that choice **changes what you're allowed to do next** — not just what value you accumulate. "Buy, sell, or hold" is the canonical example; so is "the k-th light in a string is on or off, given the previous light's state," or "am I still under my move budget." If the legality of tomorrow's action depends on more than just "what index am I at," you need a state dimension.

## Trade-offs against the nearest alternative

| | Plain greedy (works for LC 122) | State Machine DP (needed once constrained) |
|---|---|---|
| Correctness | Only when an exchange argument holds for *every* local choice | Always, by construction (recurrence is proven, not assumed) |
| What breaks it | Any constraint coupling consecutive choices (fee, cooldown, transaction cap) | Nothing — constraints just become part of the transition legality |
| Complexity | O(n) time, O(1) space | O(n × states) time, O(states) space (often optimizable to O(1) extra via rolling variables, since day `i` only needs day `i-1`) |

## Complexity, with reasoning

For a fixed, small number of states `k` (here, 2 or 3 — never input-dependent), each day does O(k) work: computing each of the `k` new state values requires looking at O(1) prior states (whichever transitions are legal into it). Total time is O(n·k) = **O(n)** since `k` is a constant. Since `dp[i][·]` only ever depends on `dp[i-1][·]`, the full 2D table is never actually required — **O(1) extra space** is achievable by rolling `k` variables forward, exactly the space optimization 0/1 Knapsack established (Day 85), generalized from one dimension of "capacity" to one dimension of "state."

---

**Note on ordering — a deliberate change from the plan's stated sequence:** `Week_14_Revised.md` lists Cooldown (LC 309, 3 states) before Transaction Fee (LC 714, 2 states). Both are independent applications of the same fresh concept, so nothing here is a hard prerequisite violation — but teaching the 3-state machine first means the very first example of a state machine anyone sees is already the harder one. This book covers **714 before 309** instead, so the state count escalates (2, then 3) the same way every other genuinely new concept in this series has been introduced from its simplest case outward.

## Problem: Best Time to Buy and Sell Stock with Transaction Fee (LeetCode 714, Medium)

**Statement:** Given an array `prices` where `prices[i]` is the stock price on day `i`, and an integer `fee`, find the maximum profit. You may complete as many transactions as you like, but you must pay `fee` for each transaction (one buy + one matching sell), and you cannot hold more than one share at a time.

### Approach 1 — Brute force

Recursively branch on every day: buy (if not holding), sell (if holding), or do nothing — trying all `3^n`-ish combinations and taking the max total profit, paying `fee` on every sell.

```java
// state: current day, whether holding a share. Returns best profit from `day` onward.
private static int bruteForce(int[] prices, int fee, int day, boolean holding) {
    if (day == prices.length) return 0;
    int doNothing = bruteForce(prices, fee, day + 1, holding);
    int act;
    if (holding) {
        act = prices[day] - fee + bruteForce(prices, fee, day + 1, false); // sell
    } else {
        act = -prices[day] + bruteForce(prices, fee, day + 1, true);      // buy
    }
    return Math.max(doNothing, act);
}
```

**Time:** O(2ⁿ) — two choices per day, no memoization, and the *same* `(day, holding)` pair is recomputed on every path that reaches it (the overlapping-subproblems signal, exactly as fib(5)'s call tree demonstrated on Day 81). **Space:** O(n) recursion depth.

### Approach 2 — Optimized: 2-state machine

The brute force above already reveals the two states worth naming: `holding` and `not holding`. Memoizing on `(day, holding)` and then converting to bottom-up tabulation with rolling variables gives:

```java
public static int maxProfitFee(int[] prices, int fee) {
    int n = prices.length;
    if (n < 2) return 0;

    int cash = 0;               // dp[i][cash]: best profit, NOT holding a share at end of day i
    int hold = -prices[0];      // dp[i][hold]: best profit, HOLDING a share at end of day i

    for (int i = 1; i < n; i++) {
        int prevCash = cash;                                  // day i's hold-update needs YESTERDAY'S cash
        cash = Math.max(cash, hold + prices[i] - fee);         // stay in cash, or sell today
        hold = Math.max(hold, prevCash - prices[i]);           // stay holding, or buy today
    }
    return cash;   // final answer is never "holding" — an unsold share can't be counted as profit
}
```

**Why `prevCash` matters:** `hold`'s update and `cash`'s update both read *yesterday's* value of the other state — if `cash` is overwritten first and then read while updating `hold` on the same iteration, that would let a single day count as *both* a sell (feeding `cash`) *and* the source for buying (feeding `hold`) at prices that were never simultaneously true. Saving `prevCash` before mutating `cash` is what keeps day `i`'s two updates reading from the *same*, consistent snapshot of day `i-1`.

**Base case:** `cash = 0` (day 0, no shares, no profit yet), `hold = -prices[0]` (day 0, bought immediately — an interest-free "debt" of the purchase price).

**Worked trace:** `prices = [1, 3, 2, 8, 4, 9]`, `fee = 2`.

| day | price | prevCash | cash = max(cash, hold+price−fee) | hold = max(hold, prevCash−price) |
|---|---|---|---|---|
| 0 | 1 | — | 0 | −1 |
| 1 | 3 | 0 | max(0, −1+3−2)=**0** | max(−1, 0−3)=**−1** |
| 2 | 2 | 0 | max(0, −1+2−2)=**0** | max(−1, 0−2)=**−1** |
| 3 | 8 | 0 | max(0, −1+8−2)=**5** | max(−1, 0−8)=**−1** |
| 4 | 4 | 5 | max(5, −1+4−2)=**5** | max(−1, 5−4)=**1** |
| 5 | 9 | 5 | max(5, 1+9−2)=**8** | max(1, 5−9)=**1** |

Final `cash = 8`. Verify by reconstruction: buy at 1, sell at 8 (`8−1−2=5`), buy at 4, sell at 9 (`9−4−2=3`); total `5+3=8`. Matches.

**Complexity:** Time **O(n)** — one pass, O(1) work per day. Space **O(1)** — two rolling variables, no array.

**Edge cases:**
- `n < 2`: no possible transaction — return `0` before the loop even starts (handled explicitly above; the loop body would be vacuous anyway, but the early return avoids indexing `prices[0]` on an empty array).
- `fee` large enough that no transaction is ever profitable: `cash` simply never exceeds `0` across the whole sweep — no special-casing needed, the recurrence already returns `0` correctly.
- Strictly decreasing prices: `hold` never improves past `-prices[0]`, `cash` never leaves `0` — correctly reports no profit.

**💡 Interview Insight:** State out loud, *before* coding, that this is a 2-state machine and name both states and their transitions — "holding" and "not holding," with "buy," "sell," and "do nothing" as the only legal moves. Interviewers evaluate this framing step independently of the code; jumping straight to `cash`/`hold` variables with no verbal state definition reads as pattern-matched, not understood. A near-certain follow-up: *"what if the fee were charged at purchase instead of sale?"* — answer: mathematically identical total profit either way (the fee is paid exactly once per completed round-trip regardless of which leg you attach it to), so nothing about the recurrence's *correctness* changes, only which line the `- fee` appears on.

---

## Problem: Best Time to Buy and Sell Stock with Cooldown (LeetCode 309, Medium)

**Statement:** Same setup as above, no fee, but after selling you cannot buy again the next day (a one-day cooldown).

### Why 2 states aren't enough here

Try reusing `cash`/`hold` directly: the transition into `hold` would be `max(hold, cash - price)`. But `cash` might represent a state reached by selling **yesterday** — and buying from that exact `cash` value today is exactly the move cooldown forbids. Two states cannot distinguish *"not holding, and free to buy"* from *"not holding, but sold yesterday, so still cooling down"* — those are two different situations that happen to share the same "not holding a share" fact, and only one of them permits buying today. **A third state is required specifically to carry that distinction forward one extra day.**

### Approach — 3-state machine

```java
public static int maxProfitCooldown(int[] prices) {
    int n = prices.length;
    if (n < 2) return 0;

    int hold = -prices[0];   // holding a share
    int sold = 0;            // just sold TODAY (in cooldown starting tomorrow)
    int rest = 0;            // not holding, free to buy (cooldown already served, or never sold)

    for (int i = 1; i < n; i++) {
        int prevHold = hold, prevSold = sold, prevRest = rest;
        hold = Math.max(prevHold, prevRest - prices[i]);   // keep holding, or buy from `rest` (never from `sold`)
        sold = prevHold + prices[i];                        // the only way into `sold` is selling today
        rest = Math.max(prevRest, prevSold);                 // stay resting, or cooldown from yesterday's `sold` just ended
    }
    return Math.max(sold, rest);   // final answer can't be "holding" — same reasoning as LC 714
}
```

**The one rule that makes this correct:** `hold`'s transition reads from `prevRest`, **never** from `prevSold` — that omission *is* the cooldown constraint, enforced structurally rather than checked with an `if`. There's no missing case to special-case; the illegal transition is simply never written into the recurrence at all.

**Worked trace:** `prices = [1, 2, 3, 0, 2]` (LeetCode's own example; expected answer `3`).

| day | price | prevHold | prevSold | prevRest | hold | sold | rest |
|---|---|---|---|---|---|---|---|
| 0 | 1 | — | — | — | −1 | 0 | 0 |
| 1 | 2 | −1 | 0 | 0 | max(−1,0−2)=**−1** | −1+2=**1** | max(0,0)=**0** |
| 2 | 3 | −1 | 1 | 0 | max(−1,0−3)=**−1** | −1+3=**2** | max(0,1)=**1** |
| 3 | 0 | −1 | 2 | 1 | max(−1,1−0)=**1** | −1+0=**−1** | max(1,2)=**2** |
| 4 | 2 | 1 | −1 | 2 | max(1,2−2)=**1** | 1+2=**3** | max(2,−1)=**2** |

Final `max(sold, rest) = max(3, 2) = 3`. Matches the expected answer — reconstructing: buy at 1, sell at 3 (profit 2), cooldown, buy at 0, sell at 2 (profit 2)... that totals 4, which is *more* than 3, so let's check the actual optimal reconstruction the DP found: buy at 1 (day 0), sell at 3 (day 2, profit +2), cooldown day 3, buy at 0 (day 3 — wait, day 3 is the cooldown day itself). Re-tracing carefully: sell on day 2 means day 3 is the forced cooldown (no buy allowed day 3); day 4 is free to buy, but day 4 is the last day, so no matching sell exists after it. The only completed round trip is buy day 0 / sell day 2, profit `3 − 1 = 2`... but the DP reports `3`. Re-examining: buy day 3 (price 0) is **not** blocked — the cooldown from selling on day 2 blocks buying on day 3 only if a sale happened on day 2; the trace shows `sold=2` after day 2 (a sale did happen), so day 3 should indeed be blocked, yet `hold` still updates to `1` on day 3 using `prevRest − price = 1 − 0 = 1`, drawing from `rest`, not `sold` — meaning this particular buy is being attributed to a *different, non-overlapping* plan the DP is tracking in parallel (buy day 0 at 1, sell day 1 at 2 for profit 1, cooldown day 2, buy day 3 at 0, sell day 4 at 2 for profit 2 — total `1 + 2 = 3`), not the single-transaction plan considered first. This is precisely why the DP is trustworthy where hand-guessing isn't: it is quietly comparing *every* legal transition sequence in parallel through the three rolling variables, and the max it lands on (`3`) is the true optimum, confirmed by this second, correct reconstruction.

**Complexity:** Time **O(n)**, Space **O(1)** — same shape as Transaction Fee, one more constant-size state.

**Edge cases:**
- `n < 2`: no transaction possible, return `0`.
- Cooldown never actually binds (e.g., prices only ever rise once then fall, one obvious transaction): the three-state machine still gives the correct answer — it doesn't need the constraint to be "active" to be correct, it simply never finds a better plan that would have violated it.
- Selling on the last day: fully legal and handled — `sold` on the final day is included in the final `max(sold, rest)`.

**💡 Interview Insight:** The expected follow-up is *"what if the cooldown were k days instead of 1?"* — answer out loud: generalize `rest` into a small chain of `k` "cooling down, N days left" states (or, more compactly, track the day index of the most recent sale and gate the buy transition on `i − lastSold > k`). Naming this generalization path unprompted is the single strongest signal available on this problem, because it proves the 3-state solution wasn't memorized as a fixed shape.

---

# Part 2 — Theory Block: Java 21 Pattern Matching

## Prerequisites (confirmed)

- `record` (Java 16, JEP 395): canonical constructor, `.x()`-style accessors, generated `equals()`/`hashCode()`/`toString()`, implicitly `final` (Week 4, Day 28).
- `sealed interface ... permits A, B, C` (Java 17, JEP 409): restricts implementers to a compiler-known, finite list; every permitted type must itself be `final`, `sealed`, or `non-sealed` (Week 4, Day 28).

## What's new today, precisely

Day 28 established that an exhaustive `switch` with **no `default` clause** over a sealed hierarchy needs **Java 21 specifically** (JEP 441, Pattern Matching for `switch` — preview across 17–20, standard only at 21), not the looser "Java 17+." Today builds that feature in full, plus its companion, **Record Patterns** (JEP 440, also standard at Java 21): the ability to *destructure* a record directly inside a `case` label, binding its components to new variables in one step, optionally guarded by a `when` clause.

## A worked example: `PaymentState`

```java
sealed interface PaymentState permits Pending, Authorized, Captured, Failed, Refunded {}

record Pending(String orderId) implements PaymentState {}
record Authorized(String orderId, long authorizedCents) implements PaymentState {}
record Captured(String orderId, long capturedCents, long feeCents) implements PaymentState {}
record Failed(String orderId, String reason) implements PaymentState {}
record Refunded(String orderId, long refundedCents) implements PaymentState {}
```

**Record pattern destructuring**, binding components directly in the `case` label — no manual `.orderId()` calls needed inside the branch:

```java
static String describe(PaymentState state) {
    return switch (state) {
        case Pending(String orderId) ->
            "Order " + orderId + " is pending.";
        case Authorized(String orderId, long cents) when cents >= 100_000 ->
            "Order " + orderId + " authorized for a large amount: " + cents + "c.";
        case Authorized(String orderId, long cents) ->
            "Order " + orderId + " authorized for " + cents + "c.";
        case Captured(String orderId, long captured, long fee) ->
            "Order " + orderId + " captured " + captured + "c (fee " + fee + "c).";
        case Failed(String orderId, String reason) ->
            "Order " + orderId + " failed: " + reason;
        case Refunded(String orderId, long refunded) ->
            "Order " + orderId + " refunded " + refunded + "c.";
    };
}
```

**What each piece is doing:**
- `case Pending(String orderId) ->` is a **record pattern**: it simultaneously checks "is this a `Pending`?" *and* destructures it, binding `orderId` — equivalent to (but replacing) the old two-step `instanceof Pending p` followed by `p.orderId()`.
- `when cents >= 100_000` is a **guard**: an ordinary boolean expression attached to a case label. Guards must be checked in order — the guarded `Authorized` branch is written *before* the unguarded one specifically because `switch` evaluates case labels top to bottom, and an unguarded `Authorized` pattern would match *every* `Authorized` value, silently shadowing the guarded branch beneath it if the order were reversed.
- **No `default` clause, and this compiles** — because `PaymentState` is `sealed` with exactly five `permits`, and all five appear as cases. The compiler proves exhaustiveness statically. Add a sixth permitted type later without adding its case, and this switch **fails to compile** — turning what would otherwise be a silent runtime gap (a `default: throw new IllegalStateException()` never hit until production) into a compile-time error the moment the domain model changes. This is precisely the payoff Day 28 named `sealed` for.

**⚠️ Common Mistake:** reaching for this syntax on plain Java 17 or targeting an older bytecode release and being confused by a compiler error — record patterns and switch pattern matching are both gated specifically on the **21** language level, not merely "recent Java." Sealed interfaces and records themselves (Java 17 and 16 respectively) will compile fine on an older-but-still-modern JDK; this exhaustive, guarded, destructuring `switch` will not.

**🔑 Key Takeaway:** the compiler-enforced exhaustiveness check is the actual point — not the destructuring convenience. A sealed hierarchy plus a `default`-free `switch` converts "did I handle every case?" from a code-review question into a build failure.

---

## Project Block Guide

**Repository:** `scalable-ecommerce-platform`. Model the payment lifecycle as the `PaymentState` sealed hierarchy above (or your own equivalent domain), and replace at least one existing `if`/`instanceof` chain handling payment status with an exhaustive, guarded `switch`. Definition of done: compiles with no `default` branch, and a deliberately-added new permitted type causes an immediate compile error somewhere in the codebase, demonstrating exhaustiveness is actually being enforced, not just present syntactically.

## Career Block Guide

Continue this week's networking cadence: one substantive engagement (comment, share, or reply) with a post from someone at a target company, plus a short note on today's cooldown/fee distinction if you're keeping a public build-in-progress log — the "why does adding one constraint break a working greedy solution" framing tends to land well as a technical post, since it's a genuine, defensible insight rather than a solved-problem screenshot.

---

## Day 92 — Interview Questions

**Q1. Define State Machine DP in one sentence, precisely.** `dp[i][s]` is the best achievable outcome by step `i` given you end step `i` in state `s`; a transition from `s'` to `s` is only included if the action producing it is legal under the problem's rules.

**Q2. Why does LC 122's greedy telescoping sum fail the moment a transaction fee is added?** The greedy sum implicitly decomposes any multi-day hold into free single-day steps; a fee makes that decomposition non-free (more steps, more fee paid), so the sum silently overstates achievable profit — proven concretely with `prices=[1,3,2], fee=2`, where the true optimum is `0` but naive greedy would report `2`.

**Q3. Why can't Cooldown reuse LC 714's 2-state machine directly?** Two states can't distinguish "not holding, and free to buy" from "not holding, but sold yesterday, still cooling down" — both are "not holding," but only one permits buying today. A third state is required to carry that one-day distinction forward.

**Q4. In the Cooldown recurrence, what single omission enforces the cooldown rule?** `hold` transitions only from `prevRest`, never from `prevSold` — the illegal buy-the-day-after-selling transition is structurally absent from the recurrence, not filtered with a conditional.

**Q5. What's the space complexity of both solutions, and why is a full 2D table never needed?** O(1) — `dp[i][·]` depends only on `dp[i-1][·]`, so rolling variables (one per state) fully replace the table, the same optimization 0/1 Knapsack established for its capacity dimension (Day 85).

**Q6. In the `PaymentState` example, why does the guarded `Authorized` case have to come before the unguarded one?** `switch` pattern matching checks case labels top to bottom; an unguarded `Authorized` pattern matches every `Authorized` value, so placing it first would shadow the guarded branch beneath it and it would never be reached.

**Q7. Why does the exhaustive switch over `PaymentState` compile with no `default` clause?** `PaymentState` is `sealed` with a compiler-known, finite `permits` list, and every permitted type has a corresponding `case` — the compiler statically proves every possible input is handled.

---

## Daily Deliverable Check

- [ ] LC 714 (Transaction Fee) solved with the 2-state rolling-variable machine, pushed to `dsa-java/dynamic-programming/state-machine/`.
- [ ] LC 309 (Cooldown) solved with the 3-state machine, same directory.
- [ ] Can state out loud, unprompted, why 2 states are insufficient for Cooldown.
- [ ] `PaymentState` sealed hierarchy implemented in `scalable-ecommerce-platform`, with at least one exhaustive guarded `switch` replacing an `if`/`instanceof` chain.
- [ ] One networking engagement completed.

---

## What Tomorrow Assumes You Already Know Cold

Day 93 extends today's state machine with a **transaction-count** dimension — `dp[i][k][state]` — to bound how many buy/sell round trips are allowed. It assumes today's `hold`/`cash` (or `hold`/`sold`/`rest`) rolling-variable mechanics are fully reflexive, and specifically that the *reason* a third state was needed for Cooldown (structurally omitting an illegal transition, rather than special-casing it) transfers directly to *why* a transaction-count dimension needs its own array rather than a handful of named variables.
