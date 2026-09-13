# Day 23 — Greedy & Intervals Begins: Naming What You've Already Been Doing

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 22 Resource Book](Day22_Resource_Book.md)
**Next ▶:** [Day 24 Resource Book](Day24_Resource_Book.md)
**Companion to:** Day 23 of `Week_04_Revised.md`

---

## Recap

Yesterday closed Prefix Sum & Kadane's at 9 distinct problems. Today opens a second pattern family with zero presence in the original 17-week plan — **Greedy & Intervals**, 11 required problems across Days 23–27. Unlike yesterday, today's opening has no new *data structure* prerequisite at all: greedy algorithms need only sorting (Week 1, Day 3) and array fundamentals (Week 1, Day 2), both long since automatic.

There's a more interesting kind of prerequisite, though — one you already have without having named it. Two problems solved weeks ago were tagged in `00_Curriculum_Map.md` with the word "greedy" attached, informally, before today's formal treatment existed:

- **LC 881, Boats to Save Most People** (Week 2, Day 11) — tagged "Two Pointers — Opposite Ends, Greedy Pairing."
- **LC 11, Container With Most Water** (Week 2, Day 12) — tagged "Two Pointers — Opposite Ends, Provable Greedy."

Both of those solutions made a locally-best choice at each step and never looked back — that's greedy, in substance, weeks before it had a name. Today gives it the name, the formal justification technique, and — just as important — the boundary of where it stops working.

Per the pattern's own opening precedent (Sliding Window opened Week 2, Day 14 with zero extra practice; Prefix Sum & Kadane's opened Week 3, Day 21 the same way), **no extra practice is added today** — the pattern is still opening, not closing, and adding extras this early risks colliding with problems Week 4's own later days already require. Extras for Greedy & Intervals start tomorrow, once there's enough of the pattern established to pick genuinely complementary reps.

---

## Learning Objectives

By the end of today, without notes:

1. State the formal definition of a greedy algorithm, and the exchange-argument shape used to prove one is safe — not just assert it.
2. Name a problem where greedy provably fails, and explain precisely *why* it fails, sharpening recognition of when the technique actually applies.
3. Recognize the "extend the farthest-reachable frontier" greedy shape for reachability problems (Jump Game), and its natural extension into a level-counting shape for minimum-step problems (Jump Game II).
4. Explain, via a telescoping argument, why summing every positive day-over-day price delta correctly solves unlimited-transaction stock trading — and why this differs fundamentally from Day 14's single-transaction version.

---

## Concept Dependency Map

```
Already known, informally, without the word "greedy" attached yet:
├─ Week 2, Day 11: LC 881 Boats to Save Most People — "opposite-ends, greedy pairing"
└─ Week 2, Day 12: LC 11 Container With Most Water — "opposite-ends, provable greedy"
        │
        ▼
TODAY — Greedy, Formalized
├─ Definition: locally-best choice at each step, never revisited
├─ Proof technique: the exchange argument (swap any optimal solution's choice
│  for the greedy one without making it worse — if that's always possible, greedy is safe)
├─ Counter-example: where greedy provably FAILS (0/1 Knapsack, sketched)
        │
        ▼
Needs: Day 14's LC 121 (Best Time to Buy/Sell Stock, ONE transaction) as contrast
├─ LC 122 Best Time to Buy/Sell Stock II — UNLIMITED transactions, greedy
        │
        ▼
Needs: arrays (Day 2) only — no new structure
├─ LC 55 Jump Game — greedy, "farthest reachable" frontier tracking
└─ LC 45 Jump Game II — greedy, level-counting variant of LC 55's frontier idea
        │
        ▼
Tomorrow: Gas Station (harder greedy, circular) + Intervals begins (a sort-and-sweep sibling pattern)
```

---

# Part 1 — Greedy, Formalized

**Definition:** a greedy algorithm builds a solution one step at a time, at each step making whichever choice looks locally best *right now*, and never reconsidering that choice later. No backtracking, no exploring alternatives once a choice is made.

That definition alone doesn't tell you anything about *correctness* — plenty of locally-best-looking choices lead to a globally wrong answer. **The entire skill in "being greedy" is proving that, for this specific problem, local optimality happens to compose into global optimality.** Interviewers who ask a greedy question are almost never satisfied with "I'll be greedy here" as a standalone justification; they're listening for the proof.

### The exchange argument — the standard proof template

**The shape of the argument:** take *any* optimal solution. If it doesn't already make the same choice your greedy strategy would make at some step, show that swapping in the greedy choice instead — "exchanging" one decision for another — can only keep the solution just as good, never make it worse. If that swap is always possible, then some optimal solution exists that agrees with greedy at every step, which means greedy itself achieves the optimum.

This isn't a new idea today — it's the same shape of argument you've already produced twice, informally:

- **Container With Most Water (Week 2, Day 12):** moving the pointer at the *shorter* wall was justified by showing that keeping the shorter wall in place could never beat the best already found, since the shorter wall bounds every remaining possibility — an exchange argument, stated without the name.
- **Kadane's Algorithm (Week 3, Day 21):** "extending a suboptimal prior subarray is never better than extending the truly optimal one" is exactly an exchange argument applied to a running-best value instead of a discrete choice.

Today's problems make this proof technique explicit and central, rather than incidental.

### Where greedy fails — a concrete counter-example

Recognizing *when* greedy applies requires knowing what it looks like when it doesn't. The classic example: **0/1 Knapsack.** Given items with weights and values and a weight capacity, choose a subset maximizing total value without exceeding capacity — but each item can only be taken whole or not at all (unlike the "Fractional Knapsack" variant, where items can be split, and greedy-by-value-density *does* work).

Consider capacity `= 10`, and items `(weight, value)`: `(6, 12)`, `(5, 10)`, `(5, 10)`. Value-per-weight is `2, 2, 2` — all identical, so a greedy-by-density strategy has no clear preference and might pick item 1 first (weight 6, value 12), leaving capacity `4` — not enough for either remaining item (weight 5 each). Greedy total: `12`. But the true optimum takes items 2 and 3 together: weight `5+5=10` (exactly fits), value `10+10=20`. **Greedy found 12; the optimum is 20.** The locally-reasonable choice (take the best-density item available) actively blocks a better combination later, and there's no exchange argument that rescues it — swapping item 1 out for either of the other two, alone, doesn't recover the lost value, and greedy has no mechanism to consider *combinations*. This is precisely why 0/1 Knapsack needs Dynamic Programming (previewed briefly on Day 21 as Kadane's "best answer here, built from the best answer one step back" shape) rather than a greedy strategy — DP explores the combination space; greedy commits without ever revisiting.

**🔑 Key Takeaway:** greedy works exactly when an exchange argument can be completed — when swapping the greedy choice into any optimal solution is provably harmless. The moment a choice can *block* a better future combination (as item 1 did above), greedy has no answer, and the interview signal shifts from "justify greedy" to "recognize greedy doesn't apply here."

---

## Problem: Best Time to Buy and Sell Stock II (LeetCode 122, Easy) — Pattern: Greedy

**Statement:** given an array `prices` where `prices[i]` is the stock's price on day `i`, find the maximum profit achievable, given you may complete **as many transactions as you like** (buy then sell, buy then sell again, and so on) — but you can't hold more than one share at a time, and you must sell before buying again.

**🔗 Contrast with Day 14's LC 121:** that problem allowed exactly **one** transaction — track the minimum price seen so far, and the best `price - minSoFar` at each step. Today's problem removes the one-transaction limit entirely, and that single change makes the optimal strategy fundamentally different, not just a small variation.

### Approach 1 — Brute force (conceptual)

Exhaustively trying every combination of buy/sell day pairs, respecting the "must sell before buying again" rule, is exponential in the number of possible transaction boundaries — not a real candidate to code, but worth stating out loud as the naive baseline before presenting the greedy insight, since naming an infeasible brute force and explaining *why* it's infeasible is itself a useful opening move.

### Approach 2 — Optimized: greedy, sum every positive delta

```java
public static int maxProfit(int[] prices) {
    int totalProfit = 0;
    for (int i = 1; i < prices.length; i++) {
        if (prices[i] > prices[i - 1]) {
            totalProfit += prices[i] - prices[i - 1];
        }
    }
    return totalProfit;
}
```

**The greedy claim:** capture every single positive day-over-day price increase, and ignore every decrease entirely.

**Why this is optimal — the exchange/telescoping argument, proven, not asserted:** suppose prices go `a → b → c` with `a < b < c` (a two-day uphill run). Buying at `a` and selling at `c` directly yields profit `c - a`. Buying at `a`, selling at `b`, immediately buying back at `b`, then selling at `c` yields `(b - a) + (c - b)`, which telescopes to exactly `c - a` — **identical profit.** This means any longer uphill run can be freely decomposed into its smallest possible daily up-moves without losing any profit. Since the problem allows unlimited transactions, decomposing all the way down to single-day transactions — buy the day before every price increase, sell the day of it — captures every available up-move at zero cost, and skipping any decrease is free (you simply don't hold a share through it). No strategy can capture more than "every up-move, entirely," because that's already the maximum total upward movement available in the sequence.

**Worked trace:** `prices = [7, 1, 5, 3, 6, 4]`. Day-over-day deltas: `-6, +4, -2, +3, -2`. Sum only the positive ones: `4 + 3 = 7`.

Verify by decomposition: buy at `1` (day 1) sell at `5` (day 2) → profit `4`; buy at `3` (day 3) sell at `6` (day 4) → profit `3`. Total `7`. Matches the known expected output for this exact input.

**Complexity:** Time O(n), Space O(1).

**Edge cases:** strictly decreasing prices (every delta is negative, sum stays `0` — correct, no profitable transaction exists); strictly increasing prices (every delta is positive, sum equals `prices[n-1] - prices[0]` — correct, one continuous hold is optimal, and the decomposition argument shows summing daily deltas gets there without needing to recognize the long run explicitly); a single price or empty array (loop doesn't execute meaningfully, profit stays `0` — correct, no transaction is possible with fewer than two days).

💡 **Interview Insight:** the strongest opening here is stating the telescoping identity (`(b-a)+(c-b) = c-a`) *before* writing the one-line loop — it's the entire proof, and a candidate who states it unprompted has demonstrated the actual skill this problem tests, independent of how trivial the resulting code looks.

---

## Problem: Jump Game (LeetCode 55, Medium) — Pattern: Greedy

**Statement:** given an array `nums` where `nums[i]` is the maximum jump length from index `i`, starting at index `0`, determine whether you can reach the last index.

### Approach 1 — Brute force / backtracking

Try every possible jump length from every reachable position, recursively, backtracking on failure. Exponential in the worst case — from each position with jump length `k`, up to `k` further recursive branches open, compounding across the array.

### Approach 2 — Optimized: greedy, track the farthest reachable frontier

```java
public static boolean canJump(int[] nums) {
    int farthestReachable = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > farthestReachable) {
            return false;   // this position itself is unreachable
        }
        farthestReachable = Math.max(farthestReachable, i + nums[i]);
        if (farthestReachable >= nums.length - 1) {
            return true;    // early exit — the end is already guaranteed reachable
        }
    }
    return true;
}
```

**The greedy claim:** never track *which specific path* got you anywhere — only track the single number "farthest index reachable so far," updated as the maximum over every position visited. If the current position ever exceeds that frontier, nothing reachable could have gotten you here, so the whole attempt fails.

**Why tracking only the frontier (and discarding *how* you got there) loses no information:** the only thing that matters for reachability is *which indices can eventually be reached*, not the specific sequence of jumps used to reach them. If two different paths can both reach index `5`, the only fact worth keeping is "index `5` is reachable" — the specific path is irrelevant to every future decision, since from index `5` onward, both paths have exactly the same options available. Reducing the entire reachable set down to its single farthest point is safe precisely because "farthest reachable" strictly dominates every closer reachable point — anything reachable from a closer point is also reachable from the farthest one, since the farthest one has at least as much jump range to work with at every subsequent step, or *more*. This domination is the exchange argument here: any strategy that stops tracking at a closer point can be replaced by one that tracks the farthest point instead, without ever losing reachability.

**Worked trace:** `nums = [2, 3, 1, 1, 4]`.

| i | i > farthestReachable? | i + nums[i] | farthestReachable (updated) |
|---|---|---|---|
| 0 | no (0>0 false) | 0+2=2 | 2 |
| 1 | no (1>2 false) | 1+3=4 | 4 → `4 >= 4` (last index), **return true** |

Matches the known expected output (`true`) for this input.

Second trace, a failing case: `nums = [3, 2, 1, 0, 4]`.

| i | i > farthestReachable? | i + nums[i] | farthestReachable |
|---|---|---|---|
| 0 | no | 0+3=3 | 3 |
| 1 | no (1>3 false) | 1+2=3 | 3 |
| 2 | no (2>3 false) | 2+1=3 | 3 |
| 3 | no (3>3 false) | 3+0=3 | 3 |
| 4 | **yes** (4>3) | — | **return false** |

Index `4` (the last index) is never reachable — every jump length from indices `0`–`3` tops out at index `3`. Correctly returns `false`, matching the known expected output.

**Complexity:** Time O(n), Space O(1).

**Edge cases:** array of length 1 (already at the last index — loop's first iteration has `i=0`, `farthestReachable` starts at `0`, `0 > 0` is false, so no early false-return, and the loop correctly falls through to `return true` without needing to jump at all); a `0` at index `0` in an array longer than 1 (first real position already traps you — `farthestReachable` stays `0`, and `i=1` immediately triggers `1 > 0`, correctly returning `false`); all-zero array of length > 1 (same trap, caught the same way).

⚠️ **Common Mistake:** trying to simulate actual jump *choices* (which specific jump length to take from each position) instead of just tracking the aggregate frontier — this reintroduces the exponential branching the brute force has, for no benefit, since (as proven above) only the frontier value ever matters.

---

## Problem: Jump Game II (LeetCode 45, Medium) — Pattern: Greedy (BFS by Levels, in Disguise)

**Statement:** same setup as Jump Game, but now the last index is *guaranteed* reachable — return the **minimum number of jumps** needed to reach it.

### Approach 1 — Brute force / backtracking

Try every possible sequence of jumps, tracking the shortest sequence that reaches the end — exponential, for the same reason as Jump Game's brute force.

### Approach 2 — Optimized: greedy jump-boundary tracking (BFS by levels, without an explicit queue)

**The reframe:** think of this as breadth-first search over "how far can I get in exactly `k` jumps," where each BFS *level* is a jump. You don't need an explicit queue, though, because the entire reachable range at each level is a contiguous interval — tracking just its two endpoints (`currentJumpEnd`, the boundary of what's reachable in the current jump count, and `farthestReachable`, the best you could reach with one *more* jump from anywhere already visited) captures everything a queue would have held.

```java
public static int jump(int[] nums) {
    int jumps = 0, currentJumpEnd = 0, farthestReachable = 0;

    for (int i = 0; i < nums.length - 1; i++) {   // stop BEFORE the last index — see below
        farthestReachable = Math.max(farthestReachable, i + nums[i]);
        if (i == currentJumpEnd) {
            jumps++;
            currentJumpEnd = farthestReachable;
        }
    }
    return jumps;
}
```

**Why the loop stops at `nums.length - 2` (i.e., `i < nums.length - 1`), not the last index:** if `i` were allowed to reach the very last index and that index happened to equal `currentJumpEnd`, the code would increment `jumps` one extra, unnecessary time — you're already *at* the destination, and don't need to "jump" from it. Stopping one index early avoids ever evaluating that no-op jump.

**Why `i == currentJumpEnd` is the exact right moment to commit to a jump:** `currentJumpEnd` is the farthest index reachable using the jumps counted *so far*. Every index up to and including `currentJumpEnd` was already reachable without using another jump. The instant `i` reaches `currentJumpEnd` itself, continuing forward requires a jump has already been "spent" to get here from wherever the previous boundary was — so incrementing `jumps` and resetting the boundary to `farthestReachable` (the best possible reach from *anywhere already visited*, tracked continuously) is forced, not optional. This is the BFS level-transition, expressed without an explicit queue: `currentJumpEnd` is "the edge of the current BFS level," and `farthestReachable` is "the edge of the next BFS level, discovered while still processing the current one."

**Worked trace:** `nums = [2, 3, 1, 1, 4]`. Loop runs `i = 0` to `3` (length 5, stop before index 4).

| i | nums[i] | i+nums[i] | farthestReachable | i == currentJumpEnd? | jumps | currentJumpEnd (after) |
|---|---|---|---|---|---|---|
| 0 | 2 | 2 | 2 | yes (0==0) | 1 | 2 |
| 1 | 3 | 4 | 4 | no (1==2? no) | 1 | 2 |
| 2 | 1 | 3 | 4 | yes (2==2) | 2 | 4 |
| 3 | 1 | 4 | 4 | no (3==4? no) | 2 | 4 |

Final `jumps = 2`. Verify: jump from index 0 to index 1 (using `nums[0]=2`'s range, landing anywhere up to index 2 — choosing index 1 since `nums[1]=3` reaches farthest), then from index 1 to index 4 directly (`1+3=4`). Two jumps. Matches the known expected output for this exact input.

**Complexity:** Time O(n) — each index is visited exactly once, and the amortized argument from Week 3 (total pointer/index movement bounded by n) applies directly, since nothing is revisited. Space O(1).

**Edge cases:** array of length 1 (loop condition `i < nums.length - 1` is `i < 0`, never true — loop doesn't execute, `jumps` stays `0`, correct, since you're already at the destination); every jump exactly reaching the next index one at a time (`jumps` correctly accumulates to `n-1`, one per step); a single jump spanning the whole array (`jumps` correctly stays `1`, since `currentJumpEnd` is set to the full reach on the very first iteration and `i` never catches up to it before the loop ends).

💡 **Interview Insight:** if asked to justify why this greedy strategy finds the *minimum* number of jumps (not just *a* valid count), the BFS-by-levels framing is the proof: BFS is well-established to find shortest paths in an unweighted sense (fewest "edges," here fewest jumps) precisely because it explores every node reachable in `k` steps before any node reachable in `k+1` steps — this algorithm computes that level structure implicitly, via the two boundary variables, without ever materializing a queue. Naming the BFS connection explicitly is a strong, non-obvious signal.

---

## Career Block Guide (30 min)

**LinkedIn: engagement.** Same 10–15 minutes as yesterday — commenting meaningfully on 3–5 posts in your feed or network. Worth varying which posts you engage with day to day (not always the same few accounts) so your activity reads as genuine ongoing engagement rather than a narrow, repetitive pattern.

---

## Day 23 — Interview Questions

**Q1. What's the difference between "greedy" as a description of an algorithm's behavior and "greedy" as a justified, correct strategy?** Any algorithm that makes a locally-best choice at each step without reconsidering it is *describable* as greedy — that alone says nothing about correctness. It becomes a justified strategy only once an exchange argument (or equivalent proof) shows the locally-best choice can always be swapped into an optimal solution without making it worse.

---

**Q2. Give an example where greedy provably fails, and say exactly why.** 0/1 Knapsack with capacity 10 and items (6,12), (5,10), (5,10) — greedy-by-value-density picks the first item (density 2, same as the others), leaving capacity 4, insufficient for either remaining item, for a total value of 12. The true optimum takes the second and third items together (weight 10 exactly, value 20). Greedy's early choice blocks a better later combination, and no exchange argument can recover the lost value.

---

**Q3. Why is Best Time to Buy/Sell Stock II's optimal strategy fundamentally different from Day 14's single-transaction version, not just a small variation?** Removing the one-transaction limit means any longer uphill price run can be freely decomposed into consecutive single-day transactions without losing profit (a telescoping identity: `(b-a)+(c-b) = c-a`), so capturing every positive day-over-day delta independently becomes optimal — a strategy that would be actively wrong under a one-transaction limit, where you must commit to a single buy/sell pair.

---

**Q4. In Jump Game, why is it safe to discard *how* a position was reached and track only the single farthest-reachable index?** The farthest reachable index strictly dominates every closer reachable index — anything reachable from a closer point remains reachable from the farthest point too, since the farthest point has at least as much jump range available at every subsequent step. Any strategy tracking a closer point can be replaced by one tracking the farthest point without ever losing reachability.

---

**Q5. In Jump Game II, what does `currentJumpEnd` represent, and why does reaching it force a jump count increment?** It's the farthest index reachable using the jumps already counted. Once the scan reaches that exact index, continuing forward requires a jump has already been used to arrive there, so committing to the next jump (incrementing the count, extending the boundary to `farthestReachable`) is forced, not a choice.

---

**Q6. Why does Jump Game II's loop stop one index before the array's end?** If the loop reached the final index and that index happened to equal `currentJumpEnd`, the code would increment `jumps` one unnecessary time for a "jump" that's never actually needed, since the destination is already reached.

---

**Q7. Why is Jump Game II's greedy strategy described as "BFS by levels, in disguise"?** Each jump corresponds to one BFS level — the set of indices reachable in exactly that many jumps. `currentJumpEnd` marks the boundary of the current level; `farthestReachable`, updated continuously while scanning the current level, becomes the next level's boundary the moment the current one is exhausted. This mirrors BFS's level-by-level exploration without needing an explicit queue, and is exactly why the strategy finds the *minimum* jump count, not just *a* valid one.

---

## Daily Deliverable Check

- [ ] Best Time to Buy and Sell Stock II, Jump Game, and Jump Game II solved, pushed to `dsa-java/greedy-intervals/`.
- [ ] Can state the exchange-argument proof shape from memory, and give the 0/1 Knapsack counter-example unprompted.
- [ ] LinkedIn engagement done (10–15 min).

---

## What Tomorrow Assumes You Already Know Cold

Day 24 assumes today's exchange-argument habit is already automatic — Gas Station's correctness proof (tomorrow) uses a variant of the same reasoning, applied to a circular array, and the resource book won't re-derive what "exchange argument" means from scratch. Tomorrow also opens **Intervals** as a sibling pattern to Greedy — a genuinely new mechanism (sort by start time, sweep once, merge on overlap) that doesn't depend on anything from today beyond arrays and sorting, both long-automatic. If Jump Game II's BFS-by-levels framing still feels like it needs re-deriving rather than simply being recalled, that's worth a few extra minutes before moving on — today was deliberately the conceptually densest of the week's opening days.

**Next:** [Day 24 Resource Book](Day24_Resource_Book.md) — Greedy Continues, and Merge Intervals Begins.
