# Day 93 — State Machine DP Closes: Bounding the Transaction Count

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 92 Resource Book](Day92_Resource_Book.md)
**Next ▶:** [Day 94 Resource Book](Day94_Resource_Book.md)
**Companion to:** Day 93 of `Week_14_Revised.md`

---

## Recap

Yesterday built State Machine DP from zero and used it twice: a 2-state machine (`hold`/`cash`) for Transaction Fee, extended to 3 states (`hold`/`sold`/`rest`) for Cooldown — with the extra state existing specifically to structurally forbid one illegal transition (buying the day after a sale). Both machines allowed **unlimited** transactions.

Today adds a genuinely different kind of dimension: not a new *situation* (like cooldown), but a **budget** — at most 2 transactions, then at most `k`. This isn't a bigger version of yesterday's states; it's a new axis entirely, and it needs its own place in the index.

---

## Learning Objectives

By the end of today, without notes:

1. Extend `dp[i][state]` to `dp[i][k][state]`, and explain precisely what the new dimension `k` counts and when it advances.
2. Derive the 4-variable solution to "at most 2 transactions" directly from the general `dp[i][k][state]` recurrence, rather than memorizing it as a fixed shape.
3. Generalize to arbitrary `k`, including the specific optimization that prevents wasted work when `k` is large relative to `n`.
4. State, unprompted, why State Machine DP closes today with zero added practice problems.

---

## Concept Dependency Map

```
Day 92: 2-state (hold/cash) and 3-state (hold/sold/rest) machines,
        both with UNLIMITED transactions
        │
        ▼
NEW DIMENSION: transaction count k, a BUDGET, not a situation
dp[i][k][state] — "best outcome by day i, having started at most k
                   transactions, ending day i in `state`"
        │
        ├──▶ LC 123 (at most 2) — k fixed at 2, unrolled into 4 named variables
        │
        └──▶ LC 188 (at most k) — k generalized to a parameter, array-indexed
                    │
                    └──▶ reduction: k ≥ n/2 collapses to LC 122 (Wk4 D23, unlimited/greedy)

STATE MACHINE DP CLOSES: 4/4 required (LC 714, 309, 123, 188)
```

---

# Part 1 — Bounding the Transaction Count

## Prerequisites (confirmed)

- The 2-state `hold`/`cash` machine and its recurrence, in full (Day 92).
- Rolling-variable space optimization, and why it's valid when `dp[i]` depends only on `dp[i-1]` (Day 92, generalizing 0/1 Knapsack's capacity-dimension optimization, Day 85).

## The general recurrence

Let `k` count **transactions started** (i.e., buys made), capped at some maximum `K`. Define:

```
dp[i][k][hold] = best profit by day i, having started exactly k transactions, currently holding
dp[i][k][cash] = best profit by day i, having started exactly k transactions, currently not holding
```

```
dp[i][k][hold] = max( dp[i-1][k][hold],                    // keep holding
                       dp[i-1][k-1][cash] - price[i] )      // buy today — STARTS transaction k
dp[i][k][cash] = max( dp[i-1][k][cash],                     // stay in cash
                       dp[i-1][k][hold] + price[i] )         // sell today — completes transaction k (k unchanged)
```

`k` only ever advances on a **buy**, never on a sell — a full transaction is "buy, then eventually sell," so the count needs to be charged at the point of commitment, not completion. The final answer is `max` over every `k` from `0` to `K` of `dp[n-1][k][cash]` — "at most `K`" means the best `k` might genuinely be *less* than `K`, so every value up to the cap must be considered, not just the cap itself.

**Why this is provably correct and not just plausible:** exactly the same disjoint-exhaustive-case argument this series has used since House Robber (Day 82) — on any given day, in any given state, there are only ever two disjoint possibilities (act, or don't), and the recurrence takes the max over precisely those two, so nothing reachable is ever excluded, and nothing unreachable is ever included (an illegal transition, like advancing `k` on a sell, is simply never written into the recurrence — the same structural-omission style Cooldown used yesterday, not a runtime check).

---

## Problem: Best Time to Buy and Sell Stock III (LeetCode 123, Hard)

**Statement:** Same as before, but at most **2** transactions total.

### Deriving the 4-variable solution from the general recurrence

With `K = 2` fixed, unroll `dp[i][k][state]` into four named variables — `buy1, sell1` for `k=1`, `buy2, sell2` for `k=2` — and note `dp[i][0][cash]` is always `0` (zero transactions, zero profit, never needs its own variable):

```java
public static int maxProfitTwoTransactions(int[] prices) {
    int n = prices.length;
    if (n < 2) return 0;

    int buy1 = -prices[0], sell1 = 0;
    int buy2 = -prices[0], sell2 = 0;

    for (int i = 1; i < n; i++) {
        buy1  = Math.max(buy1, -prices[i]);              // dp[i][1][hold]: buy from dp[i-1][0][cash]=0
        sell1 = Math.max(sell1, buy1 + prices[i]);        // dp[i][1][cash]
        buy2  = Math.max(buy2, sell1 - prices[i]);        // dp[i][2][hold]: buy from dp[i][1][cash] — SAME iteration
        sell2 = Math.max(sell2, buy2 + prices[i]);        // dp[i][2][cash]
    }
    return sell2;
}
```

**⚠️ Common Mistake — reading `buy2`'s line as a bug:** `buy2` reads `sell1` *after* `sell1` was just updated on this same iteration, not yesterday's `sell1`. This is deliberate, not an ordering accident: it correctly allows the second transaction to begin using a sale that happened on the *same day* as this iteration — which is exactly what "at most 2 transactions, no gap required" permits. Forcing `buy2` to read a `sell1` snapshotted from before this iteration would incorrectly forbid same-day sell-then-buy sequences.

**Why the final answer is `sell2`, not `max(sell1, sell2)`:** because `sell2`'s own recurrence already carries forward `max(sell2, buy2 + price)`, and `buy2` was itself defined as `max(buy2, sell1 - price)` — `sell2` can never fall below what a single-transaction plan would have achieved, since "not taking the second transaction" is already one of the two options `sell2`'s own `max` considers implicitly through `buy2`'s history. Tracking `max(sell1, sell2)` would be redundant, not wrong — just unnecessary given `sell2` already dominates.

**Worked trace:** `prices = [3, 3, 5, 0, 0, 3, 1, 4]` (LeetCode's own example; expected answer `6`).

| i | price | buy1 | sell1 | buy2 | sell2 |
|---|---|---|---|---|---|
| 0 | 3 | −3 | 0 | −3 | 0 |
| 1 | 3 | max(−3,−3)=−3 | max(0,0)=0 | max(−3,−3)=−3 | max(0,0)=0 |
| 2 | 5 | max(−3,−5)=−3 | max(0,2)=**2** | max(−3,−3)=−3 | max(0,2)=**2** |
| 3 | 0 | max(−3,0)=**0** | max(2,0)=2 | max(−3,2)=**2** | max(2,2)=2 |
| 4 | 0 | max(0,0)=0 | max(2,0)=2 | max(2,2)=2 | max(2,2)=2 |
| 5 | 3 | max(0,−3)=0 | max(2,3)=**3** | max(2,0)=2 | max(2,5)=**5** |
| 6 | 1 | max(0,−1)=0 | max(3,1)=3 | max(2,2)=2 | max(5,3)=5 |
| 7 | 4 | max(0,−4)=0 | max(3,4)=**4** | max(2,0)=2 | max(5,6)=**6** |

Final `sell2 = 6`. Matches. Reconstruction: buy at 0 (day 3), sell at 5... wait — buy at day 2's dip isn't right either; trust the DP over hand-guessing (Day 92 already demonstrated why): the actual plan the numbers support is buy day 3 (price 0) / sell day 5 (price 3, profit 3) as transaction 1, buy day 6 (price 1) / sell day 7 (price 4, profit 3) as transaction 2 — total `3 + 3 = 6`.

**Complexity:** Time **O(n)**, Space **O(1)** — four rolling variables, `K=2` baked in as a constant.

**Edge cases:**
- `n < 2`: no transaction possible, `0`.
- Best result uses **fewer than 2** transactions: already handled — nothing forces `buy2`/`sell2` to be used; if a second transaction never helps, `sell2` simply never exceeds `sell1`'s contribution baked into it.
- Monotonically decreasing prices: both transactions correctly contribute `0`.

**💡 Interview Insight:** State explicitly, before coding, that this is "the 2-state machine from yesterday, unrolled across an explicit transaction-count dimension" — not a new technique. The likely follow-up is exactly tomorrow's generalization: *"what if it were k transactions, not 2?"* Naming the array-indexed generalization unprompted, before being asked, is the strongest signal this problem offers.

---

## Problem: Best Time to Buy and Sell Stock IV (LeetCode 188, Hard)

**Statement:** Same setup, at most `k` transactions, `k` given as an input.

### Approach — generalize the unrolled variables into arrays

```java
public static int maxProfitKTransactions(int k, int[] prices) {
    int n = prices.length;
    if (n < 2 || k == 0) return 0;

    if (k >= n / 2) {
        // More budget than the array can ever use: at most n/2 non-overlapping
        // transactions fit in n days, so the cap stops binding — this is exactly
        // LC 122's unconstrained problem again (Week 4, Day 23, greedy).
        int profit = 0;
        for (int i = 1; i < n; i++) {
            if (prices[i] > prices[i - 1]) profit += prices[i] - prices[i - 1];
        }
        return profit;
    }

    int[] buy = new int[k + 1];
    int[] sell = new int[k + 1];
    Arrays.fill(buy, Integer.MIN_VALUE);   // sell[] defaults to 0, which is exactly dp[i][t][cash]'s correct base case

    for (int price : prices) {
        for (int t = 1; t <= k; t++) {
            buy[t]  = Math.max(buy[t], sell[t - 1] - price);
            sell[t] = Math.max(sell[t], buy[t] + price);
        }
    }
    return sell[k];
}
```

**Why the `k ≥ n/2` guard exists, precisely:** every transaction needs at least one distinct buy day and one distinct sell day, and transactions can't overlap — so the maximum number of transactions that could ever possibly fit in `n` days is `⌊n/2⌋`. Once `k` reaches that ceiling, "at most `k`" and "unlimited" describe the exact same achievable set of plans, and the O(n) greedy sum is both correct and far cheaper than running the O(n·k) array version with a `k` that could otherwise be arbitrarily, wastefully large. Skipping this guard doesn't produce a wrong *answer* — it produces needlessly expensive, and for a large enough `k`, wasteful array allocation for a bound that was never actually reachable.

**Why `Integer.MIN_VALUE` for `buy[]`'s initial value, not `0`:** `buy[t]` represents "holding, having started transaction `t`" — before any price has been seen, that state is **not yet reachable** for any `t ≥ 1`, and using `0` instead of a sentinel would incorrectly let `sell[t]` treat an unreached `buy[t]` as if a transaction had already, freely, started. `sell[]` defaults to `0` correctly, because `dp[i][t][cash]` **is** validly reachable with zero profit before any transaction (simply never having transacted yet).

**Trace (abbreviated — same shape as LC 123's table, one more `t` layer):** `k=2, prices=[3,2,6,5,0,3]` (LeetCode's own example; expected answer `7`). Since `k=2 < n/2=3`, the array path runs. Running the same update rule shown above through all six prices lands on `sell[2] = 7`, achieved by buy day 1 (price 2) / sell day 2 (price 6, profit 4) as transaction 1, buy day 4 (price 0) / sell day 5 (price 3, profit 3) as transaction 2 — total `4 + 3 = 7`.

**Complexity:** Time **O(n·k)** in the general branch (or O(n) whenever the `k ≥ n/2` guard fires). Space **O(k)** — two arrays of size `k+1`; still no dependency on `i` beyond the current day, since `buy[]`/`sell[]` are overwritten in place each day (this is valid for the *same* reason the 2- and 3-state rolling variables were valid on Day 92 — `dp[i][·][·]` only ever needs `dp[i-1][·][·]`, or values already updated earlier in the *same* day's inner loop, exactly like `buy2` reading the current day's `sell1` in LC 123 above).

**Edge cases:**
- `k = 0`: zero transactions allowed, return `0` immediately (guarded explicitly).
- `k` far larger than `n` (e.g., `k = 1000` on a 5-day array): the `k ≥ n/2` guard fires, avoiding an oversized, mostly-wasted array.
- `k = 1`: the array path correctly degenerates to the single-transaction case — equivalent to (though not identical code to) LC 121, since `buy[1]`/`sell[1]` alone reduce to exactly that recurrence.

**💡 Interview Insight:** Volunteer the `k ≥ n/2` optimization even if the interviewer's stated constraints don't obviously require it — it demonstrates you're thinking about the algorithm's behavior across its *entire* input space, not just the examples given, which is precisely the kind of unprompted defensive reasoning tier-1 interviewers are listening for. If pressed on why the bound is `n/2` specifically and not, say, `n`, have the one-line non-overlap argument ready rather than citing it as a memorized constant.

---

## State Machine DP Closes: 4/4 Required

`LC 714, 309, 123, 188` — every required State Machine DP problem is now solved. **Zero extra practice was added across both days, and that's worth stating explicitly rather than leaving as a silent zero** (the same practice this series used for Dijkstra's Algorithm, Week 12, Day 80, when its required ladder was already comprehensive). The four problems together already span every shape this pattern tests in real interviews: unconstrained-but-fee-bearing (2 states), a structural constraint coupling consecutive days (3 states), a small fixed transaction budget (a new counting dimension, unrolled), and an arbitrary transaction budget (the same counting dimension, generalized, including the reduction back to the unconstrained case). A fifth problem here would necessarily repeat one of these four shapes rather than add a genuinely new one — exactly the "don't pad a pattern that's already comprehensive" judgment call this series has made before.

---

# Part 2 — Theory Block: The Dead Letter Queue Pattern

## What it is

In an asynchronous message-processing system (Kafka, SQS, RabbitMQ, or similar), a **Dead Letter Queue (DLQ)** is a separate topic/queue that a message is routed to after it has **failed processing repeatedly**, instead of being retried forever or silently dropped. A consumer typically retries a failed message some bounded number of times (often with backoff between attempts); once that budget is exhausted, the message is published to the DLQ along with metadata about the failure (the error, the retry count, the original topic/partition/offset) rather than being reprocessed again.

## Why it works — what problem it actually solves

Without a DLQ, a system has exactly two bad options when a message can't be processed: **retry forever**, which can permanently stall the main queue behind one poison-pill message (blocking every message behind it if ordering matters, or burning consumer throughput indefinitely if it doesn't), or **drop it silently**, which loses the failure with no record it ever happened. A DLQ is a third option: stop blocking or looping on the bad message, but **preserve it and the fact that it failed**, so a human or an automated remediation process can inspect, fix, and optionally replay it later without having lost any information.

## When to reach for it

The signal: an asynchronous consumer whose processing can genuinely fail for reasons unrelated to *transient* infrastructure issues (which a simple bounded retry with backoff already handles) — a malformed payload, a business-rule violation, a downstream dependency that will never succeed for this specific message no matter how many times it's retried. If every failure is expected to be transient, retries alone may suffice; a DLQ becomes valuable specifically once some failures are **not** expected to resolve on their own.

## Trade-offs against the nearest alternative

| | Retry forever (or drop silently) | Dead Letter Queue |
|---|---|---|
| Poison-pill messages | Can permanently stall the queue, or vanish with no trace | Isolated after a bounded retry budget; main queue keeps flowing |
| Observability | Failures are invisible unless explicitly logged elsewhere | Failure + metadata is durably captured in the DLQ itself |
| Operational cost | Lower — nothing extra to build or monitor | Higher — needs its own monitoring/alerting, and a replay/remediation process, or failures accumulate unseen |

**⚠️ Common Mistake:** standing up a DLQ and never building anything that *consumes* it. A DLQ with no monitoring or replay tooling just relocates where messages silently pile up — the durability and visibility only pay off if something is actually watching it and acting on what lands there.

---

## Project Block Guide

**Repository:** `scalable-ecommerce-platform`. Add a DLQ topic alongside an existing consumer (order-processing, payment-webhook handling, or similar) — bounded retry with backoff on the primary path, publish to the DLQ with failure metadata (exception type/message, retry count, original topic/partition/offset) once the retry budget is exhausted. Definition of done: a deliberately-malformed test message demonstrably lands in the DLQ, not stuck retrying forever and not silently dropped.

## Career Block Guide

Continue the week's outreach cadence — today's technical thread (why a transaction budget needs an explicit DP dimension, distinct from yesterday's cooldown-as-a-situation) is a reasonable follow-up post if yesterday's landed well; otherwise, spend the block on mock-interview prep, narrating today's `k ≥ n/2` reduction and the DLQ trade-off table out loud, unprompted, as if fielding the natural follow-up question in each case.

---

## Day 93 — Interview Questions

**Q1. What does the new dimension `k` count in `dp[i][k][state]`, and when does it advance?** The number of transactions started (buys made) so far, capped at `K`; it advances only on a buy, never a sell, since a transaction is charged at the point of commitment.

**Q2. Derive `buy1`'s recurrence from the general form.** `dp[i][1][hold] = max(dp[i-1][1][hold], dp[i-1][0][cash] - price[i])`; since `dp[·][0][cash]` is always `0` (zero transactions, zero profit), this simplifies to `max(buy1, -price[i])` — exactly the code.

**Q3. Why does `buy2` read the current day's `sell1`, not yesterday's?** It's deliberate: "at most 2 transactions" permits a same-day sell-then-buy, so `buy2` must be allowed to start from a sale that just happened this same iteration, not one frozen from the prior day.

**Q4. What does the `k ≥ n/2` check in LC 188 protect against, and why `n/2` specifically?** It avoids running the O(n·k) array path with a `k` that could never actually bind — at most `⌊n/2⌋` non-overlapping transactions fit in `n` days (each needs a distinct buy day and sell day), so once `k` reaches that ceiling, "at most k" and "unlimited" are the same problem, solvable by LC 122's O(n) greedy sum instead.

**Q5. Why is `buy[]` initialized to `Integer.MIN_VALUE` but `sell[]` to `0`?** `sell[t]` (zero profit, `t` transactions not yet started) is validly reachable with no prices seen; `buy[t]` for `t ≥ 1` (holding, having started transaction `t`) is not reachable at all before any price is seen, and defaulting it to `0` would let `sell[t]` treat an impossible state as if it were freely available.

**Q6. Why does State Machine DP add zero extra practice problems this week?** The four required problems already span every shape the pattern tests — unconstrained-with-fee (2 states), a structural coupling constraint (3 states), a small fixed transaction budget, and an arbitrary budget with its reduction case — so a fifth problem would necessarily repeat one of the four rather than add a new shape.

**Q7. What's the actual risk of running a Dead Letter Queue with no consumer watching it?** Failures still get isolated from the main queue (so throughput/ordering isn't blocked), but silently accumulate unseen — the durability the DLQ provides only pays off if something monitors and acts on what lands there.

---

## Daily Deliverable Check

- [ ] LC 123 (III) solved with the 4-variable derivation, pushed to `dsa-java/dynamic-programming/state-machine/`.
- [ ] LC 188 (IV) solved with the array-generalized version, including the `k ≥ n/2` guard.
- [ ] Can derive the 4-variable LC 123 solution live from the general `dp[i][k][state]` recurrence, not recite it from memory.
- [ ] DLQ added to one consumer in `scalable-ecommerce-platform`, demonstrated against a deliberately-malformed message.
- [ ] One outreach/mock-interview activity completed.

---

## What Tomorrow Assumes You Already Know Cold

Day 94 leaves State Machine DP behind and opens **Tree DP** — the recursion returns to trees (Days 46–52), but now each node's return value(s) *are* a fully-defined DP state, not a side-effect scalar. Today's discipline (state precisely what each variable/array slot means before writing any transition, then prove the recurrence by exhaustive disjoint cases) is assumed fully transferable to a tree's postorder-combine shape without re-derivation. Specifically assumed solid, not re-explained: House Robber's take-or-skip recurrence and its disjoint-exhaustive-cases proof (Week 12, Day 82) — tomorrow extends that exact recurrence onto a tree directly.
