# Day 96 — Bit Manipulation Continues: Power of Two, and DP Fuses In

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 95 Resource Book](Day95_Resource_Book.md)
**Next ▶:** [Day 97 Resource Book](Day97_Resource_Book.md)
**Companion to:** Day 96 of `Week_14_Revised.md`

---

## Recap

Yesterday derived `n & (n-1)` from binary subtraction (clears the lowest set bit) and used it to count set bits in Number of 1 Bits. Today reuses that exact identity twice more — once directly, once **fused with 1D DP's lookback discipline** (Week 12, Day 81 onward) — the same kind of cross-pattern fusion this series named explicitly once before (Word Break II fusing DP and Backtracking, Week 13, Day 86).

---

## Learning Objectives

By the end of today, without notes:

1. Recognize why exactly one condition — `n & (n-1) == 0` — characterizes every power of two, and why the `n > 0` guard is not optional.
2. Build Counting Bits' recurrence by directly reusing yesterday's identity as the DP transition, and state precisely why the recurrence is well-founded (every reference points to a strictly smaller, already-computed index).
3. Compare the independent-per-number approach against the DP approach with the correct complexity reasoning for a *range* of numbers, not a single one.

---

## Concept Dependency Map

```
Day 95: n & (n-1) clears the lowest set bit (derived from subtraction)
        │
        ├──▶ LC 231 Power of Two — direct application: exactly one set bit ⟺ n&(n-1)==0
        │
        └──▶ LC 338 Counting Bits — FUSED with 1D DP (Day 81 onward):
                   dp[i] = dp[i & (i-1)] + 1
                   (bit trick supplies the transition; DP supplies the reuse)
```

---

## Problem: Power of Two (LeetCode 231, Easy)

**Statement:** Given an integer `n`, return `true` if it is a power of two.

### Why exactly one condition characterizes this

Every power of two (`1, 2, 4, 8, 16, ...`) has **exactly one bit set** — `2^k` is a `1` at bit position `k` and `0` everywhere else, by definition of what a power of two's binary representation is. Yesterday's derivation proved `n & (n-1)` clears the lowest set bit, leaving every other bit untouched. If `n` has exactly one set bit, clearing "the lowest" clears the *only* one — the result is `0`. If `n` has two or more set bits, clearing only the lowest leaves at least one other set bit behind — the result is **not** `0`. The check is therefore an iff, not a heuristic: `n & (n-1) == 0` **exactly when** `n` has zero or one set bits.

### Approach 1 — Naive: repeated division

```java
public static boolean isPowerOfTwoNaive(int n) {
    if (n <= 0) return false;
    while (n % 2 == 0) {
        n /= 2;
    }
    return n == 1;
}
```

Repeatedly strips a trailing zero bit via integer division; a true power of two reduces all the way to `1`, anything else stalls on an odd, non-`1` remainder first. **Time:** O(log n) — one division per bit.

### Approach 2 — Optimized: the bit trick

```java
public static boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

**⚠️ Common Mistake — dropping the `n > 0` guard.** Without it, `n = 0` passes incorrectly: `0 - 1 = -1` (all 32 bits set, by two's complement), and `0 & (-1) = 0` — the check would wrongly report `0` as a power of two, even though `0` has **zero** set bits, not one, and is explicitly excluded by the problem (powers of two start at `2⁰=1`). The "exactly one set bit" argument above only applies once `n` is confirmed positive; the guard is what makes that precondition actually hold before the bit check runs.

**Worked check:** `n = 16 = 10000`. `n-1 = 15 = 01111`. `n & (n-1) = 10000 & 01111 = 00000 = 0`. `n>0` and the AND is `0` → `true`. Contrast `n = 12 = 1100`: `n-1=1011`. `1100 & 1011 = 1000 ≠ 0` → `false` (`12` has two set bits: `8+4`).

**Complexity:** Time **O(1)**, Space **O(1)** — a strict, genuine improvement over the naive O(log n) loop this time (not the "technically both O(1)" nuance from yesterday's Number of 1 Bits comparison — here the naive approach's iteration count truly does grow with `n`, since each division only strips one bit).

**Edge cases:**
- `n = 0`: excluded explicitly by the guard, as derived above.
- `n = 1 = 2⁰`: `n-1=0`, `1 & 0 = 0` → correctly `true`.
- Negative `n`: excluded by the guard — no power of two is negative, and the bit-level argument was never proven for negative operands in the first place.
- `Integer.MIN_VALUE`: worth naming explicitly — `n > 0` correctly excludes it before any bit check runs, sidestepping any concern about `n-1` on the most negative representable value.

**💡 Interview Insight:** State the iff-argument ("exactly one set bit, and clearing the lowest one either empties it or doesn't") before writing the one-liner — the code alone looks like a memorized trick; the argument is what proves it's understood.

---

## Problem: Counting Bits (LeetCode 338, Easy) — Bit Manipulation Fuses With DP

**Statement:** Given `n`, return an array `ans` of length `n+1` where `ans[i]` is the number of `1` bits in `i`, for every `i` from `0` to `n`.

### Approach 1 — Independent: yesterday's technique, called once per number

```java
public static int[] countBitsIndependent(int n) {
    int[] ans = new int[n + 1];
    for (int i = 0; i <= n; i++) {
        ans[i] = hammingWeight(i);   // yesterday's n&(n-1) loop, Day 95
    }
    return ans;
}
```

**Complexity:** counting the set bits of a single value `i` costs time proportional to how many bits `i` actually needs — up to `O(log i)`. Summed across every `i` from `0` to `n`, the total is **O(n log n)** — this is the standard, correct way to reason about it here, distinct from yesterday's single-number case: yesterday there was exactly *one* number, whose bit-width is a fixed constant regardless of its value; today there's a whole *range* of numbers, and the average bit-width genuinely grows as the range grows, so the sum is not simply O(n).

### Approach 2 — Optimized: DP, reusing yesterday's identity as the transition

```java
public static int[] countBits(int n) {
    int[] dp = new int[n + 1];      // dp[0] = 0 by array default — 0 has no set bits
    for (int i = 1; i <= n; i++) {
        dp[i] = dp[i & (i - 1)] + 1;
    }
    return dp;
}
```

**Why this is a DP recurrence, precisely, not just "a formula":** `dp[i]` is defined in terms of `dp` at a **smaller** index — `i & (i-1)`, yesterday's lowest-set-bit-clearing identity — plus O(1) extra work (`+1`, for the bit that clearing just removed). That's exactly the 1D DP shape from Day 81 onward: define `dp[i]` precisely, express it via already-solved smaller subproblems, and reuse rather than recompute.

**Why the recurrence is well-founded (every reference is to an already-computed, strictly smaller index):** Day 95 proved `n & (n-1)` clears `n`'s lowest set bit; clearing a set bit strictly decreases the value whenever one exists (`i ≥ 1` in this loop always has at least one set bit). So `i & (i-1) < i` for every `i` in the loop — the recurrence never reaches forward, only backward, and a simple ascending loop from `1` to `n` guarantees `dp[i & (i-1)]` is always already computed by the time it's read.

**Alternate, equally valid recurrence, worth knowing:** `dp[i] = dp[i >> 1] + (i & 1)` — shift `i` right by one (dropping its lowest bit entirely, whether that bit was `0` or `1`), then add back `1` if the dropped bit *was* a `1` (`i & 1` extracts exactly that). Both recurrences are correct; today's version is presented primarily because it reuses yesterday's exact identity, but recognizing the `i>>1` alternative — and that both are valid *decompositions* of the same underlying fact — is worth being able to produce if asked for a second approach.

**Worked trace:** `n = 5`.

| i | i & (i-1) | dp[i & (i-1)] | dp[i] | check (binary of i) |
|---|---|---|---|---|
| 0 | — | — | `0` (base case) | `0` — 0 bits |
| 1 | `1 & 0 = 0` | `0` | `1` | `1` — 1 bit |
| 2 | `2 & 1 = 0` | `0` | `1` | `10` — 1 bit |
| 3 | `3 & 2 = 2` | `1` | `2` | `11` — 2 bits |
| 4 | `4 & 3 = 0` | `0` | `1` | `100` — 1 bit |
| 5 | `5 & 4 = 4` | `1` | `2` | `101` — 2 bits |

`dp = [0,1,1,2,1,2]` — every value matches its number's true popcount. Matches.

**Complexity:** Time **O(n)** — O(1) work per index, `n+1` indices, a genuine asymptotic improvement over Approach 1's O(n log n). Space **O(n)** for the output array — required by the problem itself (it explicitly asks for the full array), not an avoidable cost of this particular approach.

**Edge cases:**
- `n = 0`: loop never executes; `dp = [0]`, correctly just the base case.
- Every power of two `i` in range: `i & (i-1) = 0` (today's earlier proof), so `dp[i] = dp[0] + 1 = 1` — correctly `1` bit, falling out of the general recurrence with no special-casing needed.

**💡 Interview Insight:** Naming this as "yesterday's bit trick, reused as a DP transition" *before* writing the recurrence is the single strongest signal available here — it demonstrates the technique was actually understood on Day 95, not just pattern-matched to "problems with bits in the name." A near-certain follow-up: *"can you do it without the extra `O(n)` array?"* — the honest answer is no, not fully, since the problem's own return type requires producing all `n+1` values; the O(n) space is inherent to what's being asked, not a missed optimization.

---

## No Extra Practice Today

Same reasoning as Day 95: the two required problems already deliver meaningfully distinct lessons (a direct iff-application of yesterday's identity, and a genuine cross-pattern fusion with DP) rather than repetition of each other. A dedicated, low-marginal-cost extra is deferred to Day 97, once one more required technique (per-bit frequency counting) is in place to reinforce alongside it — mirroring how this series has placed single extras at the point of maximum reinforcement value before, rather than mechanically on every day a pattern is active.

---

# Part 2 — Theory Block: ConfigMaps and Secrets

## What they are

Both are Kubernetes objects for externalizing configuration away from a container image, so the same image can run identically across environments (dev/staging/prod) with only the surrounding config changing:

- **ConfigMap:** holds non-sensitive key-value configuration — feature flags, service URLs, log levels, anything safe to see in plain text.
- **Secret:** holds sensitive data — passwords, API keys, TLS certificates. Structurally almost identical to a ConfigMap, but **base64-encoded**, not encrypted, by default — a distinction worth being precise about, since it's a common misconception.

Both can be consumed by a Pod either as **environment variables** or as **mounted files** in a volume.

## Why the base64-encoding-is-not-encryption distinction matters

Base64 is a reversible *encoding*, not a cipher — anyone with `kubectl` read access to a `Secret` object can trivially decode it back to plaintext (`base64 -d`). The actual confidentiality guarantee a `Secret` provides, by default, is only that it's kept separate from a `ConfigMap` (different RBAC rules can apply, and it avoids showing sensitive values in plain `kubectl describe` output) — genuine encryption-at-rest requires additional configuration (e.g., an external secrets manager, or Kubernetes' own encryption-at-rest feature for its backing store), not something `Secret` alone guarantees out of the box.

## When to reach for which

The signal is simply: would this value be a security problem if someone read it in plain text? If yes, `Secret` (accepting that it's separation-and-access-control, not cryptographic protection, unless additional measures are layered on); if no, `ConfigMap`.

## Trade-offs against the nearest alternative (baking config into the image)

| | Config baked into the image | ConfigMap / Secret |
|---|---|---|
| Changing a value | Requires rebuilding and redeploying the image | Update the object; Pods can pick it up without a rebuild |
| Environment portability | One image per environment, or conditional logic inside it | One image, environment-specific config supplied externally |
| Sensitive data handling | Ends up committed to source control if not careful | Kept as a distinct, separately-access-controlled object |

**⚠️ Common Mistake:** treating a `Secret` as sufficient protection on its own and pasting sensitive values into it without any further access control — the base64 encoding provides essentially no confidentiality against anyone who can already read Kubernetes objects in that namespace.

---

## Project Block Guide

**Repository:** `scalable-ecommerce-platform`. Extract at least one piece of hardcoded configuration (a service URL, a feature flag, a log level) into a `ConfigMap`, and, if any credential is currently hardcoded or in a plain properties file, move it into a `Secret` — with a comment noting explicitly that base64 alone isn't encryption, so the team's actual threat model is documented, not assumed.

## Career Block Guide

Continue outreach cadence per this week's plan.

---

## Day 96 — Interview Questions

**Q1. Why does `n & (n-1) == 0` exactly characterize powers of two (and zero)?** A power of two has exactly one set bit; clearing the lowest set bit (yesterday's proven identity) either empties a number with exactly one set bit, or leaves at least one set bit behind if there were two or more — the check is an iff, not a heuristic.

**Q2. Why is the `n > 0` guard required in Power of Two?** Without it, `n=0` passes incorrectly: `0-1=-1` (all bits set), and `0 & -1 = 0`, wrongly matching the "cleared to zero" condition even though `0` has zero set bits, not one.

**Q3. What makes Counting Bits' recurrence a genuine DP recurrence, not just a formula?** `dp[i]` is defined via `dp` at a strictly smaller, already-computed index (`i & (i-1)`, proven smaller by yesterday's identity) plus O(1) extra work — the exact shape every 1D DP recurrence in this series has had since Day 81.

**Q4. Give the alternate recurrence for Counting Bits and explain what it decomposes differently.** `dp[i] = dp[i>>1] + (i&1)` — drops the lowest bit entirely via a shift, then adds it back only if that dropped bit was itself a `1`; a different but equally valid way of expressing "one smaller, already-known popcount, plus a local correction."

**Q5. Compare the complexity of counting bits independently per-number versus via DP, and justify the difference precisely.** Independent: O(n log n) — each number's bit count costs time proportional to its own bit-width, and the average bit-width grows as the range grows. DP: O(n) — O(1) work per index by reusing an already-computed smaller result. Genuinely different this time, unlike yesterday's single-number O(1)-vs-O(1) comparison, because a *range* of numbers is involved, not one fixed-width value.

**Q6. Is a Kubernetes `Secret` encrypted?** Not by default — it's base64-encoded, which is a reversible encoding, not a cipher; anyone with read access to the object can trivially decode it. Genuine encryption-at-rest requires additional configuration beyond using `Secret` alone.

---

## Daily Deliverable Check

- [ ] LC 231 (Power of Two) solved via the bit trick, with the `n>0` guard justified in comments, pushed to `dsa-java/bit-manipulation/`.
- [ ] LC 338 (Counting Bits) solved via the DP-fused recurrence, same directory.
- [ ] Can state, unprompted, why `i & (i-1) < i` always holds in Counting Bits' loop.
- [ ] At least one config value extracted to a `ConfigMap`, and one credential (if applicable) to a `Secret`, in `scalable-ecommerce-platform`.

---

## What Tomorrow Assumes You Already Know Cold

Day 97 assumes today's and yesterday's bit-manipulation and XOR mechanics are fully reflexive, and introduces a genuinely new technique — per-bit frequency counting — as a **generalization** of XOR's "count mod 2" cancellation to "count mod 3." Missing Number reuses yesterday's XOR-cancellation argument directly, unextended; Single Number II needs the new generalization built fresh.
