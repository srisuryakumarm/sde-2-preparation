# Day 98 — Isolating Two Singletons, Addition Without +, and Week 14 Consolidation

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 97 Resource Book](Day97_Resource_Book.md)
**Next ▶:** Day 99 Resource Book (Week 15)
**Companion to:** Day 98 of `Week_14_Revised.md`

---

## Recap

This week built, in order: two's complement negation (`~x+1`, Day 95), XOR's cancellation properties (Day 95), `n&(n-1)` to clear the lowest set bit (Day 95, applied Days 95–96), and per-bit frequency counting as a mod-3 generalization of XOR (Day 97). Today combines the *first two* of those — negation and AND — into one final derived identity, then closes the week.

---

## Learning Objectives

By the end of today, without notes:

1. Derive `n & (-n)` (isolates the lowest set bit) from the two's-complement negation formula, and state precisely why it differs from `n & (n-1)`.
2. Solve Single Number III by combining XOR cancellation, bit isolation, and a partition argument — and explain why partitioning by that specific bit is guaranteed to separate the two singletons.
3. Explain Sum of Two Integers as a direct simulation of a hardware adder, with XOR and AND+shift mapped to their exact roles.
4. Reconstruct Week 14's full planned-vs-actual tally and know precisely what Week 15 assumes going in.

---

## Concept Dependency Map

```
Day 95: negation (-n = ~n+1)         Day 95/96: n&(n-1) clears LOWEST set bit
        │                                       │
        └───────────────┬───────────────────────┘
                         ▼
        n & (-n) (NEW) — ISOLATES the lowest set bit (doesn't clear it — keeps ONLY it)
                         │
                         ▼
Day 95: XOR cancellation ──┐
                            ▼
                    LC 260 Single Number III
                    (XOR-all, isolate a differing bit, partition, XOR each half)

Day 95: XOR = sum without carry (per-bit truth table)
Day 95: AND = carry generated (per-bit truth table)
        │
        ▼
    LC 371 Sum of Two Integers (ripple-carry simulation)

WEEK 14 CONSOLIDATION
```

---

# Part 1 — Isolating a Single Bit

## Deriving `n & (-n)`

Let `n`'s lowest set bit sit at position `k`: `n = X 1 0₁...0ₖ` (prefix `X`, a `1` at position `k`, `k` zeros below — the same setup Day 95 used to derive `n&(n-1)`).

`~n` flips every bit: `~n = (~X) 0 1₁...1ₖ`. Adding `1` (Day 95's negation formula, `-n = ~n+1`) ripples a carry through all `k` trailing `1`s (each becomes `0`), until it reaches position `k` (currently `0`), which absorbs the carry and becomes `1`, with nothing left to carry further:

```
-n = ~n + 1 = (~X) 1 0₁...0ₖ
```

Now AND the two:

```
  n  =   X   1  0...0
 -n  =  ~X   1  0...0
  &  =  X&~X 1  0...0
```

At every position in the prefix, `X & ~X = 0` — a bit ANDed with its own complement is always `0` (directly from the NOT truth table: `1&0=0` and `0&1=0`, with no way to get `1`). At position `k`, both `n` and `-n` have a `1`, so the AND is `1`. Below position `k`, both are `0`.

```
n & (-n) = 0...0 1 0...0   —   exactly n's lowest set bit, and NOTHING else
```

**How this differs from `n&(n-1)`, precisely:** `n&(n-1)` **clears** the lowest set bit, keeping everything else. `n&(-n)` **isolates** it — keeping *only* that one bit and clearing everything else. They're complementary operations on the exact same bit position, derived from two different combinations of today's and Day 95's identities.

**Concrete check:** `n = 12 = 1100`. `-12` in two's complement (8-bit, for readability): `~12 = 11110011`, `+1 = 11110100`. `n & (-n) = 00001100 & 11110100 = 00000100 = 4`. `12`'s lowest set bit is indeed worth `4` (`12 = 8+4`). Matches.

---

## Problem: Single Number III (LeetCode 260, Medium)

**Statement:** Exactly two elements appear once each; every other element appears exactly twice. Find both singletons, in any order.

### Approach — XOR, isolate, partition, XOR again

```java
public static int[] singleNumberIII(int[] nums) {
    int xorAll = 0;
    for (int num : nums) xorAll ^= num;          // = unique1 ^ unique2 (every pair cancels)

    int diff = xorAll & (-xorAll);                 // isolate ONE bit where unique1, unique2 differ

    int a = 0, b = 0;
    for (int num : nums) {
        if ((num & diff) != 0) {
            a ^= num;
        } else {
            b ^= num;
        }
    }
    return new int[]{a, b};
}
```

**Derivation, step by step:**

1. **XOR everything.** Every paired value cancels (Day 95); what's left is `unique1 ^ unique2` — some nonzero value, since the two singletons are guaranteed distinct.
2. **Isolate one set bit of that XOR, via today's new identity.** Every set bit of `unique1 ^ unique2` is, by XOR's own definition, a position where `unique1` and `unique2` **differ** — isolating any one of them (the lowest, via `diff & -diff`) gives one concrete bit position guaranteed to split the two singletons apart.
3. **Partition every number by that bit.** `unique1` and `unique2` land in *different* groups, by construction. Every *paired* number's two copies are bit-for-bit identical, so both copies agree on the `diff` bit and land in the *same* group as each other — meaning each group still has its pairs intact, and exactly one singleton.
4. **XOR within each group independently** — ordinary Single Number (Day 95) inside each group, isolating each singleton separately.

**Worked trace:** `nums = [1, 2, 1, 3, 2, 5]` (expected: `{3, 5}`, order-independent).

`xorAll = 1^2^1^3^2^5`. Pairs cancel (`1^1=0`, `2^2=0`), leaving `3^5`. `3=011`, `5=101`. `3^5 = 110 = 6`.

`diff = 6 & (-6)`. `6 = 0110`. `-6`: `~6=1001`, `+1=1010`. `6 & (-6) = 0110 & 1010 = 0010 = 2` — bit position 1 isolated.

Partition by bit 1 (`num & 2`):

| num | binary | `num & 2` | group |
|---|---|---|---|
| 1 | 001 | 0 | b |
| 2 | 010 | 2 (nonzero) | a |
| 1 | 001 | 0 | b |
| 3 | 011 | 2 (nonzero) | a |
| 2 | 010 | 2 (nonzero) | a |
| 5 | 101 | 0 | b |

Group `a`: `2, 3, 2` → XOR = `2^3^2 = 3`. Group `b`: `1, 1, 5` → XOR = `1^1^5 = 5`. Result `{3, 5}`. Matches.

**Complexity:** Time **O(n)** (three linear passes: XOR-all, partition-and-XOR — still O(n) total, not O(n) per pass in a way that changes the asymptotic class). Space **O(1)**.

**Edge cases:**
- Exactly two elements (both singletons, no pairs at all): `xorAll = unique1^unique2` directly; the rest of the derivation is unaffected — no special-casing needed.
- The two singletons share every bit except one: still fully handled — the argument only ever needed *one* differing bit, not many, and one is always guaranteed to exist since the values are distinct.

**💡 Interview Insight:** Narrate all four derivation steps *before* writing code — this problem is frequently used specifically to test whether Single Number's technique (Day 95) was understood as a *building block* that composes, not a fixed answer to one fixed problem shape. Stating "I need one bit where the two answers differ, and I already have a tool that finds a number's lowest set bit" unprompted is the strongest possible signal here.

---

## Problem: Sum of Two Integers (LeetCode 371, Medium)

**Statement:** Compute `a + b` without using `+` or `-`.

### Mapping the truth table to hardware addition

Adding two bits `a`, `b` (ignoring any incoming carry) has exactly four cases: `0+0=0` (no carry), `0+1=1` (no carry), `1+0=1` (no carry), `1+1=0, carry 1`. Compare directly to two operators already fully proven (Day 95):

- **XOR's truth table matches the "sum digit" column exactly**: `0^0=0, 0^1=1, 1^0=1, 1^1=0`.
- **AND's truth table matches the "generates a carry" column exactly**: `0&0=0, 0&1=0, 1&0=0, 1&1=1`.

So `a^b` computes every bit's sum **as if no carry existed anywhere**, and `a&b` marks exactly the positions that **do** generate a carry into the *next* position — which is why that carry gets shifted left by one (`<< 1`) before being folded back in.

```java
public static int getSum(int a, int b) {
    while (b != 0) {
        int carry = (a & b) << 1;
        a = a ^ b;
        b = carry;
    }
    return a;
}
```

Each iteration is one step of carry propagation, exactly the way you'd add two binary numbers by hand, one column at a time, except every column is computed simultaneously and any resulting carries are folded back in as a whole new pass, rather than left-to-right one column at a time.

**Worked trace:** `a=12 (1100), b=7 (0111)`. Expected `19`.

| iter | a | b | a & b | carry = (a&b)<<1 | new a = a^b |
|---|---|---|---|---|---|
| 1 | 1100 | 0111 | 0100 (=4) | 1000 (=8) | 1100^0111 = **1011** (=11) |
| 2 | 1011 | 1000 | 1000 (=8) | 10000 (=16) | 1011^1000 = **0011** (=3) |
| 3 | 0011 | 10000 | 00000 | 0 | 00011^10000 = **10011** (=19) |

Loop ends (`b=0`). `a=19`. Matches `12+7`.

**Termination — bounded, not open-ended:** because `int` is a fixed 32-bit type and `<<` discards any bit shifted past position 31, a carry cannot propagate indefinitely — it either resolves to `0` or eventually shifts entirely out of the 32-bit register. The worst case (e.g., `Integer.MAX_VALUE + 1`, where the carry ripples through every one of `MAX_VALUE`'s 31 set bits before landing in the sign bit) takes roughly 32 iterations — a **fixed** bound tied to the word size, not one that grows with the numeric value of `a` or `b`.

**Complexity:** Time **O(1)** — bounded by the fixed 32-bit word size, the same style of bound Day 95 used for Number of 1 Bits' naive loop. Space **O(1)**.

**Edge cases:**
- `b = 0` from the start: loop never executes, correctly returns `a` unchanged.
- One or both operands negative: no special-casing anywhere — every operator involved works on raw two's-complement bit patterns regardless of sign, which is exactly the point of having built two's complement as carefully as Day 95 did.
- `Integer.MAX_VALUE + 1`: correctly reproduces Java's own silent-overflow wraparound to `Integer.MIN_VALUE` (Day 10's original proof) — this bit-level simulation and native `+` agree exactly, because they're implementing the identical underlying arithmetic.

**💡 Interview Insight:** If asked for the time complexity, resist "O(log(a+b))" as a first answer unless immediately qualified — the precise, defensible statement is "bounded by the fixed word size (32 for `int`), not by the numeric value of the inputs," which is a subtly different and more accurate claim than one that sounds input-dependent.

---

# Part 2 — Week 14 Consolidation

## What actually got built

**Dynamic Programming closed entirely** — 35/35 required slots across Weeks 12–14 (34 newly taught, 1 fulfilled by recap — reconciled in full on Day 94). State Machine DP (4 required, 0 extra — comprehensive by design, reasoning stated explicitly Day 93) and Tree DP (2 required, 1 extra: `LC 968`) were this week's share.

**Bit Manipulation opened** and reached **8/10 required** — every core isolation trick this pattern tests: XOR-cancellation (`136`, `268`), bit-counting (`191`, `338`, fused with DP), a direct `n&(n-1)` application (`231`), mod-3 generalized counting (`137`), and today's `n&(-n)` isolation composed with partitioning (`260`) and a full hardware-adder simulation (`371`). Two more required problems (`190`, `421`) close it out at 10 next week, exactly matching `Week_14_Revised.md`'s own stated target ("up from 8").

## Planned vs. actual

| Day | Required (plan) | Extra added | Day total |
|---|---|---|---|
| 92 | 2 | 0 | 2 |
| 93 | 2 | 0 | 2 |
| 94 | 2 | 1 (`968`) | 3 |
| 95 | 2 | 0 | 2 |
| 96 | 2 | 0 | 2 |
| 97 | 2 | 1 (`461`) | 3 |
| 98 | 2 | 0 | 2 |
| **Total** | **14** | **2** | **16** |

**Cumulative distinct problems through Week 14: 229 (through Week 13) + 16 = 245.**

## Diagnostic list

- **Zero recaps needed this week, in either direction** — every one of Week 14's 14 required problems was confirmed absent from the full inventory before being taught (see the curriculum map's Week 14 overlap section). This is the **second consecutive week** with zero recaps, matching Week 13's precedent immediately before it.
- **One old problem resurfaced as a citation, correctly handled as a recap rather than a re-teach:** `LC 543` (Diameter of Binary Tree, Week 7 Day 48) was cited when introducing Tree DP on Day 94, not re-solved — its mechanism (a running value tracked outside the return) is exactly what Max Path Sum extends, and citing it directly served the same purpose a redundant re-solve would have, without the redundancy.
- **One deliberate reordering, flagged where it happened:** Day 92 taught Transaction Fee (2 states) before Cooldown (3 states), reversed from `Week_14_Revised.md`'s own stated order, so the state count escalates from the pattern's simplest case outward — noted explicitly in Day 92's own book, not silently absorbed.
- **The "35 vs. 34" DP figure is fully reconciled, not an open discrepancy** — both numbers are correct, answering slightly different questions (total required slots including one recap, vs. distinct newly-taught problems); see Day 94 for the full derivation.

## What Week 15 assumes

Every bit-manipulation identity built this week — XOR cancellation, `n&(n-1)`, `n&(-n)`, and per-bit frequency counting — is assumed fully reflexive going into Week 15, which closes Bit Manipulation at Day 99 with Reverse Bits (`LC 190`) and Maximum XOR of Two Numbers in an Array (`LC 421`). The latter specifically combines this week's bit mechanics with **Tries** (Week 9) into a Bit Trie — meaning Week 9's trie mechanism needs to be equally solid, not just this week's material; that combination has been deliberately deferred since Week 9 specifically to pair with Bit Manipulation once it existed. Dynamic Programming is fully closed and assumed to need no further reinforcement — Week 15 moves to Segment Trees, a full from-scratch Sorting review, and SQL, none of which lean on this week's material directly.

---

## Career Block Guide — Weekly Ritual and Scorecard

Close out the week's own accounting, separate from this map's cumulative tracking: tally applications sent, referrals requested, and networking touches made against the week's targets; note any interview loop currently in progress and its next step; and carry forward anything incomplete into Week 15's own career block rather than letting it silently drop. If a technical post is still owed for the week, today's `n&(-n)` derivation (built from two already-known pieces, combined into something new) is a clean, self-contained example of exactly the kind of "here's how two things I learned separately combined into a third thing" narrative that reads well publicly.

---

## Day 98 — Interview Questions

**Q1. Derive `n & (-n)` and state what it isolates.** Writing `n`'s lowest set bit at position `k` as `X 1 0...0`, `-n = ~n+1` works out to `~X 1 0...0` (the same carry-ripple argument as `n-1`, applied to `~n` instead). ANDing: the prefix cancels (`X & ~X = 0` at every position), position `k` gives `1&1=1`, and everything below is `0&0=0` — leaving exactly `n`'s lowest set bit, alone.

**Q2. Why does partitioning by one differing bit correctly separate Single Number III's two answers?** Every set bit of `unique1^unique2` is, by XOR's definition, a position where the two values differ — so partitioning on any one of them puts the two singletons in different groups by construction, while every paired number's identical copies always agree on that bit and stay together, keeping their cancellation intact within each group.

**Q3. Map XOR and AND to their roles in Sum of Two Integers, precisely.** XOR's truth table matches "sum digit ignoring carry" exactly (`1^1=0`, matching `1+1`'s digit before carry); AND's truth table matches "generates a carry" exactly (`1&1=1`, the only case addition actually carries) — the loop repeatedly folds that shifted carry back in until none remains.

**Q4. What's the real time-complexity bound for Sum of Two Integers, stated precisely?** O(1), bounded by the fixed 32-bit word size — the carry can propagate at most ~32 positions before it's shifted entirely out of the register, a bound tied to the type's width, not to the numeric value of the inputs.

**Q5. Reconcile Week 14's "16 total problems" against "14 required."** 14 is exactly `Week_14_Revised.md`'s own required count (6 DP + 8 Bit Manipulation); 16 includes the two extras added this week (`LC 968` on Day 94, `LC 461` on Day 97) — both numbers are correct, describing required-only versus required-plus-extra.

**Q6. What does Week 15's Maximum XOR of Two Numbers problem specifically require from two different earlier weeks?** This week's Bit Manipulation mechanics (XOR, bit isolation) **and** Week 9's Trie structure, combined into a Bit Trie — a combination deliberately deferred since Week 9 specifically so it could pair with Bit Manipulation once this week existed.

---

## Daily Deliverable Check

- [ ] LC 260 (Single Number III) solved via the full four-step derivation, pushed to `dsa-java/bit-manipulation/`.
- [ ] LC 371 (Sum of Two Integers) solved, with the XOR/AND-to-hardware-adder mapping reproducible from memory.
- [ ] Can derive `n & (-n)` live, from `-n = ~n+1`, not recite the conclusion alone.
- [ ] Week 14's planned-vs-actual table reproducible from memory: 14 required, 2 extra, 245 cumulative.
- [ ] Weekly scorecard updated; anything incomplete explicitly carried into Week 15 rather than dropped.

---

## What Tomorrow Assumes You Already Know Cold

Day 99 (Week 15) closes Bit Manipulation at 10/10 required, assuming every identity from this week — negation, XOR cancellation, `n&(n-1)`, `n&(-n)`, per-bit frequency counting — is solid without re-explanation, and additionally assumes Week 9's Trie mechanism is equally reflexive, since Maximum XOR of Two Numbers in an Array combines both into a Bit Trie rather than extending either one alone.
