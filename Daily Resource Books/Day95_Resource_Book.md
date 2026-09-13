# Day 95 — Bit Manipulation Opens: Binary, Two's Complement, and XOR

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 94 Resource Book](Day94_Resource_Book.md)
**Next ▶:** [Day 96 Resource Book](Day96_Resource_Book.md)
**Companion to:** Day 95 of `Week_14_Revised.md`

---

## Recap

Dynamic Programming closed yesterday at 35/35. Today opens **Bit Manipulation** — a genuinely new top-level pattern, built from zero the way Trees, Heaps, Tries, Backtracking, Graphs, Union-Find, and Dijkstra's Algorithm each were.

Two things you've already seen make today less unfamiliar than it might feel:

- **Week 3, Day 15's HashMap Internals** briefly showed `h ^ (h >>> 16)` (hash spreading) and `(n - 1) & hash` (bucket indexing) — presented then as "here's what the JDK does internally," not as general-purpose tools. By the end of today you'll be able to look back at that exact line and derive why it's written the way it is, not just accept it.
- **Week 2, Day 10** proved, precisely, that `int` overflow wraps via two's complement — `Integer.MAX_VALUE + 1` flips every trailing `1` to `0` and carries into the sign bit, landing on `Integer.MIN_VALUE`, no exception thrown. Today formalizes **why** two's complement is the representation in the first place, using that exact mechanism as the starting point.

---

## Learning Objectives

By the end of today, without notes:

1. Convert between binary and decimal by place value, and state the exact bit width and range of `int`/`long` from memory (Day 10's table, cited directly).
2. Derive the two's-complement negation formula (`~x + 1`), and prove it's correct rather than reciting it.
3. Define all six Java bitwise operators precisely, and prove — with a concrete negative-number example — exactly why `>>` and `>>>` differ, and exactly when they don't.
4. Prove XOR's algebraic properties (not just state them), and use them to solve Single Number in one pass.
5. Derive the `n & (n - 1)` lowest-set-bit-clearing identity from binary subtraction mechanics, and use it to count set bits.

---

## Concept Dependency Map

```
Day 10 (Wk 2): int/long sizes & ranges; overflow wraps via two's complement (proved)
Day 15 (Wk 3): HashMap's h ^ (h>>>16), (n-1)&hash — glimpsed, not explained
        │
        ▼
Binary representation & place value (NEW)
        │
        ▼
Two's complement, formalized (NEW) — WHY, not just "how Day 10 already showed"
        │
        ▼
Six bitwise operators: & | ^ ~ << >> >>>  (NEW)
        │
        ├──▶ >> vs >>> — proved distinct only for negative operands
        │
        ▼
XOR's algebraic properties, proved by truth table (NEW)
        │
        ├──▶ LC 136 Single Number
        │
        ▼
n & (n-1): clears the lowest set bit, derived from subtraction borrow (NEW)
        │
        └──▶ LC 191 Number of 1 Bits
```

---

# Part 1 — Binary, Two's Complement, and the Bitwise Operators

## Prerequisites (confirmed)

- `int`/`long` exact bit widths and ranges, and the proof that overflow wraps via two's complement rather than throwing (Week 2, Day 10).

## Binary representation, briefly formalized

Every integer type in Java is stored as a fixed-width sequence of bits, each position worth a power of 2 — bit 0 (rightmost) worth `2⁰=1`, bit 1 worth `2¹=2`, bit 2 worth `4`, and so on up to bit 31 for an `int`. `1011` in binary is `1·8 + 0·4 + 1·2 + 1·1 = 11`. This is the exact same place-value idea as decimal, base 2 instead of base 10 — nothing new in the *concept*, only in the base.

## Two's complement, formalized

Day 10 already proved the *effect*: `Integer.MAX_VALUE + 1` wraps to `Integer.MIN_VALUE` because ordinary binary addition doesn't special-case the sign bit — it's just another bit, and carrying into it is just carrying. Today's question is **why represent negative numbers this way at all.**

**The negation formula:** for any `x`, `-x` is represented as `~x + 1` (flip every bit, then add 1).

**Proof this is correct:** flipping every bit of an `n`-bit number `x` produces `(2ⁿ - 1) - x` — each bit flip turns a `1` into a `0` and vice versa, which is exactly what subtracting each bit's value from an all-`1`s number does. So `~x = (2ⁿ - 1) - x`, and:

```
~x + 1 = (2ⁿ - 1) - x + 1 = 2ⁿ - x
```

Under fixed-width, wraparound (mod `2ⁿ`) arithmetic — which is exactly what Day 10 already proved Java's `int`/`long` arithmetic does — `2ⁿ - x` **is** the representation of `-x`, because `x + (2ⁿ - x) = 2ⁿ ≡ 0 (mod 2ⁿ)`, i.e., `x` plus its supposed negation wraps around to exactly `0`, which is the defining property `-x` must have.

**Concrete check:** `x = 5 = 00000101` (8 bits, for readability). `~x = 11111010`. `~x + 1 = 11111011`. Add back: `00000101 + 11111011 = 100000000` — 9 bits, and the leading `1` is discarded by fixed-width wraparound, leaving `00000000 = 0`. Confirms `11111011` correctly represents `-5`.

**Why this representation, and not something simpler like "a sign bit plus the magnitude":** two's complement makes addition and subtraction use **one single circuit**, with no special-casing for sign — exactly the mechanism Day 10 already demonstrated (positive overflow flips the sign bit through ordinary carrying; the same carrying logic correctly handles every positive/negative combination with zero extra logic). A sign-and-magnitude scheme would need genuinely different hardware logic depending on the operands' signs, and would also have two representations of zero (`+0` and `-0`) — two's complement has exactly one.

## The six bitwise operators, precisely

| Operator | Name | Rule (per bit pair) |
|---|---|---|
| `&` | AND | `1` only if **both** bits are `1` |
| `\|` | OR | `1` if **either** bit is `1` |
| `^` | XOR | `1` if the bits **differ** |
| `~` | NOT | flips every bit (unary — one operand) |
| `<<` | Left shift | shifts bits left, fills with `0` on the right |
| `>>` | Arithmetic (signed) right shift | shifts bits right, fills with the **sign bit** on the left |
| `>>>` | Logical (unsigned) right shift | shifts bits right, fills with `0` on the left, **always** |

**Why `>>` and `>>>` are both needed, and exactly when they diverge:** `>>` preserves sign by design — shifting a negative number right should still produce a negative-ish result if it's meant to behave like division by a power of 2 (floor division, specifically). `>>>` deliberately ignores sign entirely, treating the bit pattern as a raw, unsigned quantity — which matters whenever the bits themselves are the point (hashing, bit-flag manipulation), not whatever number they might represent.

**Proof they differ only for negative operands**, with a concrete example: `n = -8` (32-bit: `0xFFFFFFF8`).

- `n >> 1`: sign-extends (fills with the sign bit, `1`) → `0xFFFFFFFC` = **`-4`**. (Matches floor(`-8`/`2`) = `-4`.)
- `n >>> 1`: fills with `0` → `0x7FFFFFFC` = **`2147483644`**. Wildly different from `-4` — `>>>` has no concept of "this was negative" at all; it just moved every bit right and dropped a `0` in from the top.

Now `n = 8` (positive, `0x00000008`):

- `n >> 1` = `0x00000004` = `4`.
- `n >>> 1` = `0x00000004` = `4`. **Identical.**

**Why they're identical for any non-negative number:** the sign bit of a non-negative number is already `0` — `>>` "sign-extends" by filling with the current sign bit, which for a non-negative number *is* `0`, the exact same fill value `>>>` always uses. They can only possibly diverge when the sign bit being extended is `1` — i.e., only for negative operands.

**⚠️ Common Mistake:** using `>>` when the intent is a raw bit shift on what's conceptually unsigned data (e.g., manipulating hash bits, as Day 15's `h >>> 16` does) — `>>` would incorrectly sign-extend and corrupt the upper bits whenever the value happens to have its sign bit set, even though nothing about "signedness" is conceptually meaningful for a hash's raw bits in the first place. This is exactly why Java's own `HashMap.hash()` uses `>>>`, not `>>` — worth re-reading that Day 15 line now that this is fully justified rather than just observed.

**New syntax, introduced as used:** binary literals (`0b1010` = `10`) and hexadecimal literals (`0x1F` = `31`) are valid Java integer literals — useful for writing bit patterns directly instead of converting by hand. `Integer.toBinaryString(n)` returns a `String` of `n`'s two's-complement bits (no sign, no leading zeros for positive `n`; a full 32 characters for negative `n`, since the sign bit itself is part of the pattern) — a genuinely useful debugging aid for visually checking a bit-manipulation result, worth reaching for anytime a trace is hard to follow by hand.

---

# Part 2 — XOR's Algebraic Properties

## Proof by truth table

| a | b | a ^ b |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

Every property below follows directly from this table, per-bit, and therefore holds for multi-bit integers too (each bit position is independent):

- **Commutative:** `a^b = b^a` — the table is symmetric in `a`/`b`.
- **Associative:** `(a^b)^c = a^(b^c)` — verify the 8 three-variable rows by hand once; every one agrees regardless of grouping. (Consequence: XOR-ing a list of numbers gives the same result in **any order**.)
- **Identity:** `a^0 = a` — row 1 and row 3 of the table say exactly this: XOR-ing with `0` never changes a bit.
- **Self-inverse:** `a^a = 0` — rows 1 and 4: a bit XOR'd with itself is always `0`.

**Why this makes XOR exactly the right tool for "find the element that appears once, when everything else appears in pairs":** XOR-ing the entire list, in *any* order (associativity), pairs every duplicated value with itself, and each such pair cancels to `0` (self-inverse) — those zeros then vanish against everything else (identity) — leaving only the single unpaired value standing, because it never had a partner to cancel against.

---

## Problem: Single Number (LeetCode 136, Easy)

**Statement:** Every element in an array appears exactly twice, except one, which appears once. Find it. (Constant extra space required.)

### Approach 1 — Brute force: HashMap frequency count

```java
public static int singleNumberHashMap(int[] nums) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int num : nums) freq.merge(num, 1, Integer::sum);
    for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
        if (entry.getValue() == 1) return entry.getKey();
    }
    throw new IllegalArgumentException("no single number found");
}
```

**Time:** O(n). **Space:** O(n) — violates the problem's own constant-space requirement, which is precisely the signal pointing toward XOR below.

### Approach 2 — Optimized: XOR fold

```java
public static int singleNumber(int[] nums) {
    int result = 0;
    for (int num : nums) {
        result ^= num;
    }
    return result;
}
```

**Why this is correct, restated concretely:** exactly the cancellation argument above — every paired value contributes `x^x=0` somewhere in the running XOR (order doesn't matter, by associativity), leaving only the unpaired value.

**Worked trace:** `nums = [4, 1, 2, 1, 2]`.

| step | num | result (running XOR) |
|---|---|---|
| start | — | `0` |
| 1 | 4 | `0^4 = 4` |
| 2 | 1 | `4^1 = 5` |
| 3 | 2 | `5^2 = 7` |
| 4 | 1 | `7^1 = 6` |
| 5 | 2 | `6^2 = 4` |

Final `result = 4`. Matches the single unpaired element.

**Complexity:** Time **O(n)**, Space **O(1)** — meets the problem's own constraint, unlike the HashMap version.

**Edge cases:**
- Single-element array: the loop runs once, `0^x = x` — correctly returns that one element.
- Negative numbers present: XOR operates on raw bit patterns regardless of sign, so nothing changes — the cancellation argument doesn't care what the bits *mean*, only that they match.

**💡 Interview Insight:** Name the constant-space constraint as the reason a HashMap is disqualified *before* being asked — it's stated directly in the problem, and jumping straight to XOR without acknowledging why the "obvious" HashMap approach is off the table skips a signal the interviewer is listening for.

---

# Part 3 — Clearing the Lowest Set Bit

## Deriving `n & (n - 1)`

Write `n` in binary as some prefix `X`, followed by a `1` at its lowest set bit (call that position `k`), followed by `k` zeros: `n = X 1 0₁0₂...0ₖ`.

Subtracting `1` from a number ending in a `1` followed by zeros requires borrowing through every one of those trailing zeros: each `0` becomes `1` (absorbing part of the borrow and passing the rest further left), until the borrow reaches the `1` at position `k`, which absorbs it fully and becomes `0`. Bits above position `k` (the prefix `X`) are never touched by this borrow at all:

```
n     = X 1 0 0 ... 0   (k zeros)
n - 1 = X 0 1 1 ... 1   (k ones)
```

Now AND them, position by position: at position `k`, `1 & 0 = 0`. At every position below `k`, `0 & 1 = 0`. Above position `k`, both numbers are identically `X`, so `X & X = X`, unchanged.

```
n & (n-1) = X 0 0 0 ... 0   =   n, with its lowest set bit cleared, everything else untouched
```

**Concrete check:** `n = 12 = 1100`. `n - 1 = 11 = 1011`. `n & (n-1) = 1100 & 1011 = 1000 = 8`. `12`'s lowest set bit is at position 2 (value `4`); clearing it gives `12 - 4 = 8`. Matches.

---

## Problem: Number of 1 Bits (LeetCode 191, Easy)

**Statement:** Count the number of `1` bits in the binary representation of an integer (this series treats the input as a raw 32-bit pattern, consistent with the operators above never caring about sign).

### Approach 1 — Naive: shift and check, 32 times

```java
public static int hammingWeightNaive(int n) {
    int count = 0;
    for (int i = 0; i < 32; i++) {
        if (((n >>> i) & 1) == 1) count++;
    }
    return count;
}
```

`>>>` is required here, not `>>` — shifting a negative `n` right with sign extension would keep re-introducing `1`s from the left on every iteration, corrupting the check; `>>>` is exactly the "ignore what this bit pattern might represent as a signed number" operator Part 1 built for precisely this situation. **Time:** always exactly 32 iterations, regardless of `n`.

### Approach 2 — Optimized: repeatedly clear the lowest set bit

```java
public static int hammingWeight(int n) {
    int count = 0;
    while (n != 0) {
        n = n & (n - 1);   // clear the lowest set bit
        count++;
    }
    return count;
}
```

**Why this is correct:** each iteration removes exactly one `1` bit (the derivation above), so the loop runs exactly `popcount(n)` times before `n` reaches `0` — every set bit gets counted exactly once, and the loop naturally stops the instant none remain.

**Worked trace:** `n = 11 = 00001011`.

| step | n (binary) | n-1 (binary) | n & (n-1) | count |
|---|---|---|---|---|
| 1 | `1011` | `1010` | `1010` (=10) | 1 |
| 2 | `1010` | `1001` | `1000` (=8) | 2 |
| 3 | `1000` | `0111` | `0000` (=0) | 3 |

Loop ends (`n=0`). `count = 3`. Verify: `11 = 1011`, three `1`s. Matches.

**Complexity — stated precisely, not loosely:** both approaches are, strictly, **O(1)**, since `32` is a fixed constant independent of input, and even `popcount(n)` is bounded by `32`. That equivalence is technically true and **not the whole story worth knowing**: the naive approach always runs exactly `32` iterations; the optimized approach runs exactly `popcount(n)` iterations — anywhere from `0` to `32`, and strictly fewer whenever `n` has any zero bits at all. Stating "both are O(1)" and stopping there, if asked to compare them, misses exactly the distinction the interviewer is checking for.

**Edge cases:**
- `n = 0`: loop never executes, correctly returns `0`.
- `n = -1` (all 32 bits set): the optimized loop runs the full 32 times, clearing one bit each pass — correctly returns `32`.
- Already covered by the derivation: no special-casing needed for the sign bit itself, since `&`/subtraction never treated it specially in the first place.

**💡 Interview Insight:** This exact `n & (n-1)` mechanism is about to resurface directly tomorrow — Power of Two reuses the identity itself, and Counting Bits reuses it inside a DP recurrence. Naming that connection here, today, ahead of being asked, previews the kind of "which two techniques compose" thinking tier-1 interviews reward.

---

## Why No Extra Practice Today

Bit Manipulation opens today, and — following this series' established practice for **every** genuinely new top-level pattern's opening day (Trees, Heaps, Tries, Backtracking, Graphs, Union-Find, Dijkstra's Algorithm, and Dynamic Programming itself each received zero extras on their first day) — today adds none either. The reasoning is the same each time: a brand-new pattern's opening day already carries the full weight of foundational material (here: binary representation, two's complement, all six operators, and XOR's properties, on top of two required problems) — adding a practice problem on top would trade depth on the foundation for shallow repetition, backwards from what an opening day needs. Extra reinforcement, if genuinely warranted, lands on a later day within the arc instead (see Day 97).

---

# Part 4 — Theory Block: Service Discovery

## What it is

In a system made of many service instances whose network locations (IP, port) change constantly — instances scale up/down, restart, get rescheduled onto different hosts — **service discovery** is the mechanism by which one service finds *where* another currently is, without either side hardcoding an address. Two common approaches:

- **Client-side discovery (e.g., Eureka):** each service instance registers itself with a central registry on startup (and sends periodic heartbeats to prove it's still alive). A client wanting to call another service queries the registry directly, gets back a list of currently-healthy instances, and picks one itself (often via client-side load balancing).
- **Server-side / DNS-based discovery (e.g., Kubernetes with CoreDNS):** every Kubernetes `Service` automatically gets a stable DNS name. A client just resolves that name via ordinary DNS — the cluster's own networking layer (`kube-proxy`, or an equivalent) transparently routes the resulting connection to one of the currently-healthy backing Pods. The client never sees a list of instances at all; discovery and load balancing both happen beneath the DNS lookup.

## Why it works

Neither approach requires a human (or a config file) to keep addresses current by hand — both replace "the address is X" with "ask something that always knows the current address," and that "something" is kept accurate automatically (heartbeats for Eureka; the Kubernetes control plane's own knowledge of which Pods are currently scheduled and healthy for CoreDNS).

## Trade-offs against the nearest alternative

| | Client-side (Eureka) | Server-side/DNS-based (Kubernetes + CoreDNS) |
|---|---|---|
| Client complexity | Higher — client needs a discovery-client library, and implements its own load-balancing choice | Lower — client just does an ordinary DNS lookup, no special library |
| Where load-balancing logic lives | In every client | Centralized in the cluster's networking layer |
| Coupling | Clients are coupled to the discovery mechanism itself | Clients are decoupled — a plain DNS name works identically whether the backend is one instance or a thousand |

**⚠️ Common Mistake:** assuming Kubernetes' DNS-based approach means "no discovery is happening" because there's no visible registry API being called — the discovery is real, it's simply been pushed down beneath the DNS layer instead of exposed as a client-facing API.

---

## Project Block Guide

**Repository:** `scalable-ecommerce-platform`. If the project currently hardcodes any inter-service address, replace it with a resolvable name (a Kubernetes `Service` DNS name if deployed there, or a documented equivalent if not) and note in the README which discovery model is in play and why.

## Career Block Guide

Continue outreach cadence. Today's foundational material (binary, two's complement, all six operators) is genuinely dense — if the block's time is tight today, prioritize consolidating today's DSA content over adding new networking volume; a rushed, shallow understanding of two's complement will cost more across the rest of this week than one skipped outreach touch.

---

## Day 95 — Interview Questions

**Q1. Derive the two's-complement negation formula and prove it.** `-x = ~x + 1`. Proof: flipping every bit of an n-bit `x` gives `(2ⁿ-1) - x`; adding `1` gives `2ⁿ - x`; under mod-`2ⁿ` wraparound arithmetic, `x + (2ⁿ - x) = 2ⁿ ≡ 0`, which is exactly the defining property `-x` must satisfy.

**Q2. Why does Java need both `>>` and `>>>`, and when do they actually produce different results?** `>>` sign-extends (preserves the value's sign, useful for arithmetic like floor division by a power of 2); `>>>` always fills with `0` (useful when the bits themselves are the point, not what they represent numerically). They produce identical results for any non-negative operand, since a non-negative number's sign bit is already `0` — they can only diverge when the sign bit being extended is `1`.

**Q3. Prove XOR's self-inverse property and explain why it's the key property for Single Number.** From the truth table: `1^1=0` and `0^0=0`, so `a^a=0` for any `a`, in every bit position. Combined with associativity (order doesn't matter) and identity (`a^0=a`), XOR-ing an entire array cancels every paired value to `0`, leaving only the unpaired element.

**Q4. Derive `n & (n-1)`'s effect from binary subtraction, don't just state it.** If `n`'s lowest set bit is at position `k`, subtracting `1` borrows through all `k` trailing zeros (each becomes `1`) until it reaches that `1` bit, which becomes `0`; bits above position `k` are untouched. ANDing the two: position `k` and below all AND to `0`; everything above is identical in both and passes through unchanged — the net effect is exactly `n` with its lowest set bit cleared.

**Q5. Is `hammingWeightNaive` really worse than the optimized version, given both are O(1)?** Big-O equivalence is technically true and incomplete: the naive version always runs exactly 32 iterations; the optimized version runs exactly `popcount(n)` iterations, strictly fewer whenever `n` has any zero bits — a real, meaningful difference the O(1) label alone doesn't capture.

**Q6. Why does Bit Manipulation add zero extra practice problems on its opening day?** Consistent with this series' practice for every brand-new top-level pattern (Trees, Heaps, Tries, Backtracking, Graphs, Union-Find, Dijkstra's, Dynamic Programming) — the opening day already carries the full foundational load, and adding a practice problem on top would trade depth on that foundation for shallow repetition.

**Q7. What's the actual difference between Eureka-style and Kubernetes-DNS-style service discovery?** Eureka is client-side: each client queries a registry and picks an instance itself. Kubernetes/CoreDNS is server-side: a client does an ordinary DNS lookup, and the cluster's networking layer handles routing and load-balancing transparently beneath that lookup — the client never sees an instance list at all.

---

## Daily Deliverable Check

- [ ] Can state, from memory, the exact rule for all six bitwise operators.
- [ ] Can prove (not recite) why `>>` and `>>>` differ only for negative operands, with a concrete example.
- [ ] LC 136 (Single Number) solved via XOR fold, pushed to `dsa-java/bit-manipulation/`.
- [ ] LC 191 (Number of 1 Bits) solved via the `n & (n-1)` approach, same directory.
- [ ] Can derive `n & (n-1)`'s effect live, from binary subtraction, not from memory of the conclusion alone.
- [ ] Service discovery model documented in `scalable-ecommerce-platform`'s README.

---

## What Tomorrow Assumes You Already Know Cold

Day 96 assumes today's `n & (n-1)` identity is fully reflexive — Power of Two reuses it directly as a one-line check, and Counting Bits reuses it *inside* a DP recurrence, fusing today's bit trick with the 1D DP lookback discipline established Week 12, Day 81 onward. Today's binary/two's-complement foundation (place value, sign extension, the six operators) is assumed solid without re-explanation for the rest of the week.
