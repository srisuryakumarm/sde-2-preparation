# Day 97 — Isolating a Singleton: XOR Revisited, and Counting Mod 3

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 96 Resource Book](Day96_Resource_Book.md)
**Next ▶:** [Day 98 Resource Book](Day98_Resource_Book.md)
**Companion to:** Day 97 of `Week_14_Revised.md`

---

## Recap

Days 95–96 built XOR's cancellation properties and the `n&(n-1)` identity, and used both directly. Today starts by reusing XOR's cancellation unextended (Missing Number), then hits a case where XOR alone genuinely **isn't enough** — and generalizing it, precisely, is today's new technique.

---

## Learning Objectives

By the end of today, without notes:

1. Derive Missing Number's XOR solution from the cancellation argument, and explain the overflow risk in the alternative sum-formula approach precisely (citing Week 2, Days 10–11).
2. Explain exactly why plain XOR fails on Single Number II's "everything appears three times" setup, with a concrete counterexample — not just "it doesn't work."
3. Derive per-bit frequency counting as the mod-3 generalization of XOR's mod-2 cancellation.

---

## Concept Dependency Map

```
Day 95: XOR cancellation (mod-2 parity per bit)
Day 10/11 (Wk2): int overflow wraps silently; long needed near int's boundary
        │
        ▼
LC 268 Missing Number — XOR cancellation, reused directly (no extension)
        │
        ▼
Why plain XOR FAILS when groups are size 3, not 2 (counterexample)
        │
        ▼
Per-bit frequency counting (NEW) — mod-3 generalization of mod-2 cancellation
        │
        ▼
LC 137 Single Number II

[Extra Practice] LC 461 Hamming Distance — combines Day 95's XOR + Day 95/96's bit-counting
```

---

## Problem: Missing Number (LeetCode 268, Easy)

**Statement:** An array of `n` distinct numbers drawn from `[0, n]` is missing exactly one value from that range. Find it.

### Approach 1 — Sum formula

```java
public static int missingNumberSum(int[] nums) {
    int n = nums.length;
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : nums) actualSum += num;
    return expectedSum - actualSum;
}
```

**⚠️ Common Mistake — the exact overflow lesson from Week 2, Days 10–11, resurfacing:** `n * (n + 1)` is computed *before* dividing by `2`, and for `n` large enough (roughly `n ≥ 46,341`, since `46341² ≈ 2.147×10⁹`, right at `int`'s boundary), that intermediate product itself overflows `int` — silently wrapping, exactly as Day 10 proved, with no exception. The fix, if `n`'s range isn't tightly bounded by the problem's own constraints, is the same one Day 11 required for 4Sum: compute in `long` (`(long) n * (n + 1) / 2`) and cast back only if a narrower return type is truly needed.

### Approach 2 — Optimized: XOR cancellation

```java
public static int missingNumber(int[] nums) {
    int result = nums.length;         // seed with n, since the loop below only covers indices 0..n-1
    for (int i = 0; i < nums.length; i++) {
        result ^= i ^ nums[i];
    }
    return result;
}
```

**Derivation:** the goal is `XOR(0, 1, ..., n) ^ XOR(nums[0], ..., nums[n-1])`. `nums` contains every value in `{0,...,n}` **except** the missing one, each exactly once. So in the combined XOR, every value in `{0,...,n}` that *is* present in `nums` appears exactly **twice** (once from the `0..n` sequence, once from `nums` itself) and cancels via self-inverse (Day 95); the missing value appears only **once** (from the `0..n` sequence alone) and survives. Seeding `result` with `n` and then XOR-ing in `i` and `nums[i]` together for `i` from `0` to `n-1` builds exactly this combined XOR in one pass.

**Worked trace:** `nums = [3, 0, 1]` (`n=3`, range `[0,3]`, missing `2`).

| i | nums[i] | result = result ^ i ^ nums[i] |
|---|---|---|
| start | — | `3` (seed) |
| 0 | 3 | `3 ^ 0 ^ 3 = 0` |
| 1 | 0 | `0 ^ 1 ^ 0 = 1` |
| 2 | 1 | `1 ^ 2 ^ 1 = 2` |

Final `result = 2`. Matches the missing value.

**Complexity:** Time **O(n)**, Space **O(1)**. **Zero overflow risk**, unlike the sum approach — XOR never carries or grows past 32 bits, which is the actual, concrete reason to prefer it over the sum formula once `n`'s upper bound isn't known to be safely small.

**Edge cases:**
- `n = 0` (empty array): `result` starts and stays at `0` (the loop never executes) — correctly, the only possible "missing number" from an empty range `[0,0]` is `0` itself.
- Missing value is `0` or `n` itself (the range's endpoints): no special-casing needed — the cancellation argument doesn't treat endpoints differently from any other value.

**💡 Interview Insight:** Volunteer the overflow trade-off unprompted — "the sum approach is O(n) too, but XOR avoids a real overflow risk the sum formula has for large n" is a stronger answer than presenting only one approach, and it's a genuine, previously-established concern (Days 10–11), not a generic disclaimer.

---

## Problem: Single Number II (LeetCode 137, Medium)

**Statement:** Every element appears exactly **three** times, except one, which appears once. Find it. (Constant extra space required.)

### Why plain XOR fails here — a concrete counterexample

XOR's cancellation is really: at each bit position, count how many numbers have that bit set, and take the count **mod 2**. A pair contributes `2` to that count, and `2 mod 2 = 0` — that's the whole mechanism. A *triple* contributes `3` to the count, and `3 mod 2 = 1`, **not** `0` — a tripled number's bit does **not** reliably cancel under plain XOR. Concretely: `2 ^ 2 ^ 2 = 2` (not `0`) — three copies of the same number XOR to that number itself, not to nothing, because `2^2=0` but then `0^2=2` again. Plain XOR-folding every element here would incorrectly mix tripled numbers' bits into the result alongside the true singleton's.

### Approach — per-bit frequency counting, mod 3

**Derivation:** generalize "count each bit's occurrences, take mod 2" to "count each bit's occurrences, take **mod 3**." A number that appears exactly three times contributes a multiple of `3` to every one of its bits' counts, and any multiple of `3` is `0 mod 3` — tripled numbers' bits now correctly cancel out. The true singleton contributes exactly `1` to each of its own set bits' counts, and `1 mod 3 = 1 ≠ 0` — its bits are exactly the ones that survive.

```java
public static int singleNumberII(int[] nums) {
    int result = 0;
    for (int bit = 0; bit < 32; bit++) {
        int count = 0;
        for (int num : nums) {
            count += (num >> bit) & 1;
        }
        if (count % 3 != 0) {
            result |= (1 << bit);
        }
    }
    return result;
}
```

`(num >> bit) & 1` extracts bit position `bit` of `num` as a `0`/`1` value (shift the bit of interest down to position `0`, then mask everything else off) — summed across every number, this is exactly "how many numbers have this bit set." `result |= (1 << bit)` sets that same bit position in the answer whenever the count survives the mod-3 reduction.

**Worked trace:** `nums = [2, 2, 3, 2]` (`2` appears three times, `3` once; expected answer `3`).

| bit | values' bit (2,2,3,2) | count | count % 3 | result bit set? |
|---|---|---|---|---|
| 0 | 0,0,1,0 | 1 | 1 | yes |
| 1 | 1,1,1,1 | 4 | 1 | yes |
| 2+ | 0,0,0,0 | 0 | 0 | no |

Result: bit 0 set, bit 1 set, rest clear → `011 = 3`. Matches.

**Complexity:** Time **O(32n) = O(n)** — 32 fixed outer iterations (one per bit position), each an O(n) inner scan. Space **O(1)** — no data structure grows with input size.

**Edge cases:**
- Single-element array: every bit count is either `0` or `1`; `1 % 3 = 1` for every set bit of that lone element — correctly returns it.
- Negative numbers present: `(num >> bit) & 1` reads a real bit position regardless of sign — no special-casing needed, since the technique operates on raw bit patterns, exactly like every operator built Day 95.

**💡 Interview Insight:** Lead with the counterexample (`2^2^2=2`, not `0`) *before* presenting the fix — showing precisely where the "obvious" approach breaks is stronger than jumping straight to the correct one, and it's the exact kind of "prove it, don't assert it" reasoning this series has emphasized since Day 85's loop-order counterexamples. Likely follow-up: *"generalize to 'everything appears k times except one'"* — answer: mod `k` instead of mod `3`, same argument, no other change.

---

## [Extra Practice] Hamming Distance (LeetCode 461, Easy)

*One extra today — low marginal teaching cost, since it composes two already-fully-taught mechanisms (Day 95's XOR, and Day 95/96's bit-counting) with no new technique, in the same spirit as this series' past near-zero-cost additions (e.g., Delete Operation for Two Strings, Week 13, Day 88).*

**Statement:** Given two integers `x` and `y`, return the number of bit positions at which they differ.

```java
public static int hammingDistance(int x, int y) {
    int xorResult = x ^ y;      // a 1 in exactly the positions where x and y differ
    int count = 0;
    while (xorResult != 0) {
        xorResult &= (xorResult - 1);   // Day 95/96's identity: clear the lowest set bit
        count++;
    }
    return count;
}
```

**Why this composition is correct:** XOR is `1` exactly where two bits **differ** (the truth table proved this directly, Day 95) — so `x^y`'s set bits are, by definition, exactly the differing positions. Counting those set bits is exactly yesterday's Number of 1 Bits problem, unchanged.

**Trace:** `x=1 (001), y=4 (100)`. `xorResult = 001^100 = 101 = 5`. Clear lowest bit: `5 & 4 = 4`, count=1. Clear again: `4 & 3 = 0`, count=2. Loop ends. Result `2` — matches (bits 0 and 2 differ; bit 1 is `0` in both).

**Complexity:** Time O(1) (bounded by 32 bits), Space O(1).

**Edge cases:** `x == y`: `xorResult = 0` immediately, loop never runs, correctly returns `0`.

---

# Part 2 — Theory Block: StatefulSets and DaemonSets

## What they are

Two Kubernetes workload controllers, each solving a problem plain `Deployment`s don't:

- **StatefulSet:** manages Pods that need a **stable identity** and **stable, dedicated storage** — each replica gets a predictable name (`pod-0`, `pod-1`, `pod-2`, ...) that persists across restarts, and its own `PersistentVolumeClaim` that follows *that specific* replica, not a shared pool. Used for stateful systems like databases or Kafka brokers, where "which specific instance am I, and where's my own data" genuinely matters.
- **DaemonSet:** ensures exactly one copy of a Pod runs on **every** node in the cluster (or a selected subset), automatically scheduling a new copy onto any node that joins later. Used for node-level infrastructure — log collectors, monitoring agents, network plugins — where the point is "one of these, everywhere," not "N replicas, load-balanced."

## Why each works

A plain `Deployment`'s replicas are interchangeable by design — any pod can be killed and replaced by an identical one with a new, arbitrary name and, by default, no guaranteed storage continuity. `StatefulSet` exists specifically because that interchangeability is *wrong* for stateful workloads: replacing `pod-1` must still produce something that identifies as `pod-1` and reattaches to `pod-1`'s own volume, not a fresh, anonymous replacement. `DaemonSet` exists because "one per node" isn't a replica *count* at all — it's a binding to the cluster's own node topology, which a `Deployment`'s replica count has no concept of and would need external, fragile tooling to approximate.

## Trade-offs against the nearest alternative (plain Deployment)

| | Deployment | StatefulSet | DaemonSet |
|---|---|---|---|
| Pod identity | Anonymous, interchangeable | Stable, ordinal (`-0`, `-1`, ...) | One per node, tied to that node |
| Storage | Shared or ephemeral by default | Dedicated PVC per replica, persists across restarts | Typically none, or node-local |
| Scaling model | Arbitrary replica count | Ordered, one at a time by default (predictable startup/teardown order) | Tied to node count, not set independently |

**⚠️ Common Mistake:** reaching for a `Deployment` with a `PersistentVolumeClaim` shared across replicas for something like a database cluster — multiple replicas writing to the same volume with no ordering or identity guarantees is a correctness problem waiting to happen, precisely the situation `StatefulSet` exists to avoid.

---

## Project Block Guide

**Repository:** `scalable-ecommerce-platform`. If any component in the project is genuinely stateful (a cache node, a queue broker) and currently modeled as a plain `Deployment`, note in the README whether it *should* be a `StatefulSet` instead, and why — this is a documentation/design-review task today, not necessarily a full migration.

## Career Block Guide

Continue outreach cadence per this week's plan.

---

## Day 97 — Interview Questions

**Q1. Why is the sum-formula approach to Missing Number a real overflow risk, and what's the fix?** `n*(n+1)` is computed before dividing by 2; for `n` roughly ≥46,341 this product itself can overflow `int`, silently wrapping (Day 10's exact mechanism). Fix: compute in `long`, matching Day 11's requirement for 4Sum.

**Q2. Prove, with a counterexample, why plain XOR fails on Single Number II.** `2^2^2 = 2`, not `0` — three copies of the same value XOR to that value itself, since `2^2=0` then `0^2=2` again; a triple's bit contributes an odd count (`3 mod 2 = 1`) to plain XOR, so it doesn't reliably cancel.

**Q3. Derive per-bit frequency counting as a generalization, don't just state the mod-3 rule.** XOR is really "count each bit's occurrences, take mod 2." Generalizing to groups of three: a tripled number contributes a multiple of 3 to each bit's count, and any multiple of 3 is `0 mod 3` — swap "mod 2" for "mod 3" and tripled bits correctly cancel, leaving only the true singleton's bits.

**Q4. What's the time complexity of the per-bit counting approach, and why?** O(n) — 32 fixed outer iterations (bit positions) times an O(n) inner scan per bit, and 32 is a constant independent of input size.

**Q5. Why is Hamming Distance a reasonable "cheap" extra to add this week?** It composes two already-fully-taught mechanisms — XOR (Day 95) to find differing positions, and set-bit counting (Days 95–96) to count them — with no new technique, near-zero marginal teaching cost, matching this series' past practice for exactly this kind of reinforcement addition.

**Q6. When should a Kubernetes workload be a `StatefulSet` instead of a `Deployment`?** When replicas need a stable, persistent identity and their own dedicated storage that follows that specific replica across restarts — anything where "which instance, and its own data" matters, not just "N interchangeable copies."

---

## Daily Deliverable Check

- [ ] LC 268 (Missing Number) solved via XOR, with the sum-formula overflow risk noted in comments, pushed to `dsa-java/bit-manipulation/`.
- [ ] LC 137 (Single Number II) solved via per-bit frequency counting, with the `2^2^2=2` counterexample reproducible from memory.
- [ ] LC 461 (Hamming Distance) solved, same directory.
- [ ] StatefulSet vs. Deployment noted for any stateful component in `scalable-ecommerce-platform`'s README.

---

## What Tomorrow Assumes You Already Know Cold

Day 98 assumes every identity built this week — two's complement negation (`~x+1`), XOR cancellation, `n&(n-1)`, and per-bit frequency counting — is fully reflexive, and combines two of them (two's complement's negation formula, and XOR) into one new derived identity: `n & (-n)` isolates a number's lowest set bit. That derivation is built fresh tomorrow, not assumed, but it leans on today's and Day 95's mechanics being solid enough to not need re-explanation.
