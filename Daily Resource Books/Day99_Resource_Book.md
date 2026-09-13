# Day 99 — Bit Manipulation's Final Two Problems, Tries Fully Closes, and Kubernetes Autoscaling

**Series:** SDE-2 Interview Prep · Week 15, Day 99 (of 105 DSA-phase days)
**Curriculum Map:** [00_Curriculum_Map.md](./00_Curriculum_Map.md)
**◀ Previous:** [Day 98](./Day98_Resource_Book.md) (Week 14) · **Next ▶:** [Day 100](./Day100_Resource_Book.md)
**Companion to:** Day 1 of `Week_15_Revised.md`

---

## Recap

Week 14 closed Dynamic Programming entirely (35/35 required across three weeks) and then opened Bit Manipulation from zero: place value, two's complement derived from Week 2's overflow proof, all seven bitwise operators, XOR's algebraic properties, `n & (n-1)` (clears the lowest set bit) and `n & (-n)` (isolates it), and per-bit frequency counting. That run closed at 8/10 required + 1 extra (Hamming Distance), with exactly two required problems deliberately deferred to today — Reverse Bits and Maximum XOR of Two Numbers in an Array — because the second of those two needs something Bit Manipulation alone can't supply: the Trie structure from Week 9. Today closes both patterns in a single problem.

Today also opens an entirely new theory thread — Kubernetes — which has never appeared in this series before. The plan's own framing assumes some Kubernetes vocabulary (Pods, `kubectl`, Minikube) is already in place. It isn't. That gap is real, not a plan typo, and today's Theory Block is restructured to build the missing foundation first — flagged explicitly below, not silently patched over — before reaching the Horizontal Pod Autoscaler itself.

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Reverse the bits of a 32-bit integer in a single pass, and explain precisely why the shift must be unsigned.
2. Build a **Bit Trie** — a Trie whose nodes branch on individual bits instead of characters — and use it to find the maximum XOR of any two numbers in an array in O(n) time, with a proof of why the greedy bit-by-bit walk is correct.
3. State, from memory, the final tallies for both Bit Manipulation (10/10 required) and Tries (7/7 required) and name every distinct sub-shape each pattern tested.
4. Explain the core Kubernetes object model — Pod, ReplicaSet, Deployment, Service — and the single declarative "reconcile toward desired state" idea that all four (and the HPA) are built from.
5. Write a correct `HorizontalPodAutoscaler` manifest (`autoscaling/v2`) that scales a Deployment on CPU utilization, and explain what "70% utilization" is actually measured against.

## Concept Dependency Map for Today

```
Week 14 (Bit Manipulation core,        Week 9 (Trie: insert/search,
8/10 required)                         constrained + branching descent)
  │  two's complement, XOR,                │  TrieNode structure,
  │  n&(n-1), n&(-n),                      │  DFS over a Trie
  │  per-bit frequency counting            │
  └──────────────────┬─────────────────────┘
                      ▼
        Bit Trie (NEW: same Trie shape,
        but nodes branch on BITS, not
        letters — children[0]/children[1])
                      │
                      ▼
      Maximum XOR of Two Numbers in an Array (LC 421)
      — closes Bit Manipulation (10/10) AND Tries (7/7)
      simultaneously

Week 14 (>> vs >>> divergence,          Docker (Week 7) — containers,
  proven for negative operands)         images, "already-known" building
      │                                 block for today's Pod concept
      ▼                                       │
  Reverse Bits (LC 190)                       ▼
  — needs >>> specifically,          Kubernetes fundamentals (NEW today):
    not >>, to avoid sign            Pod → ReplicaSet → Deployment → Service
    extension corrupting the         (the declarative reconcile-loop model)
    bit pattern                                │
                                                ▼
                                    Horizontal Pod Autoscaler (HPA)
                                    — a controller running that SAME
                                    reconcile loop, but computing its
                                    own desired replica count from
                                    live CPU metrics
```

---

## Part 1 — Reverse Bits (LeetCode #190, Easy)

**Statement:** Given a 32-bit unsigned integer `n`, return the integer obtained by reversing the bits of its binary representation.

### Brute-force / only approach

There isn't really a slower "brute force" distinct from the direct approach here — the direct bit-by-bit construction *is* the natural approach, so this problem is really about doing that construction correctly rather than choosing between approaches.

**Mechanism:** build the answer one bit at a time. For 32 iterations: take the lowest bit of `n` (`n & 1`), append it to the *low end* of a growing `result` after first shifting `result` left by one to make room, then discard the bit you just consumed from `n` by shifting `n` right by one.

```java
public int reverseBits(int n) {
    int result = 0;
    for (int i = 0; i < 32; i++) {
        result <<= 1;              // make room for the next bit
        result |= (n & 1);         // append n's current lowest bit
        n >>>= 1;                  // unsigned shift — bring the next bit into position 0
    }
    return result;
}
```

> ⚠️ **Common Mistake:** using `n >>= 1` (signed right shift) instead of `n >>>= 1`. Week 14, Day 95 proved `>>` and `>>>` diverge exactly when the operand is negative — `>>` sign-extends, copying the *original* sign bit into every newly-vacated high bit, so if `n`'s bit 31 was `1` (making `n` negative as a signed Java `int`), a signed shift would flood the high bits with `1`s that were never actually part of `n`'s remaining unread bits. That corrupts every bit read after the first sign-extended shift. `reverseBits` is reading `n` as a raw 32-bit *pattern*, not as a signed magnitude, so the shift must be the pattern-preserving one — `>>>`. This is Day 95's lesson, now with a concrete case where picking the wrong operator produces a silently wrong answer rather than a crash.

**Why it works:** each of the 32 loop iterations moves exactly one bit from `n`'s current low position into `result`, and because `result` is left-shifted *before* each new bit is OR'd in, the *first* bit read (originally `n`'s bit 0) ends up most-significant in `result`, and the *last* bit read (originally `n`'s bit 31) ends up least-significant. That is precisely a full reversal.

**Worked trace** (using an 8-bit window for readability — the real loop runs 32 times, but the mechanism is identical): `n = 0b00001011` (11).

| Iteration | `n & 1` | `n` after `>>>` | `result` after shift+OR |
|---|---|---|---|
| 1 | 1 | `00000101` | `00000001` |
| 2 | 1 | `00000010` | `00000011` |
| 3 | 0 | `00000001` | `00000110` |
| 4 | 1 | `00000000` | `00001101` |
| 5–8 | 0 (all) | `00000000` | `11010000` |

Reversing `00001011` gives `11010000` — matches the trace exactly.

> ⚠️ **Common Mistake:** expecting a "normal-looking" positive output. Reversing bit 0 into bit 31 means any input with its lowest bit set (i.e., any odd number) produces a result with bit 31 set — and in Java's signed `int`, bit 31 *is* the sign bit. So `reverseBits(1)` returns `-2147483648` (`Integer.MIN_VALUE`), not some small positive number. This is correct, not a bug — Java has no unsigned 32-bit type, so the bit pattern `10000000...0` is unavoidably read back as a negative `int`. Say this out loud before coding, or it looks like a failing test case when it isn't.

**Complexity:** O(1) time — the loop is a hard-coded 32 iterations, not a function of any growing input, so there's no "n" for this to scale with. O(1) space.

> 💡 **Interview Insight — the standard follow-up:** *"If this function is called billions of times, how would you speed it up?"* Precompute the bit-reversal of every possible byte (256 entries, one-time O(256) cost) into a lookup table. Then reverse a 32-bit number by splitting it into 4 bytes, looking up each byte's reversal in O(1), and reassembling them in **reverse byte order** (the byte that was most-significant becomes least-significant, and its *contents* are also internally reversed). This is `[EXTENSION]` — not needed for today's deliverable, but a genuine, commonly-asked follow-up worth being able to sketch on a whiteboard.

**Edge cases:** `n = 0` → result `0` (no bits set, nothing to move). `n = -1` (all 32 bits set, `0xFFFFFFFF`) → result is also all bits set, still `-1`. Both fall out of the loop with no special-casing needed — a good sign the mechanism is genuinely bit-pattern-based rather than value-based.

**Interview framing:** say the mechanism (bit-by-bit shift-and-append) and the `>>>`-not-`>>` reasoning *before* writing a line of code — the operator choice is the one thing an interviewer is actually listening for here, since the loop skeleton itself is short. If asked for the follow-up, sketch the byte-lookup-table idea without necessarily coding it live unless asked.

---

## Part 2 — Maximum XOR of Two Numbers in an Array (LeetCode #421, Medium)

**Statement:** Given an integer array `nums` (each `0 <= nums[i] <= 2³¹ − 1`), find `max(nums[i] XOR nums[j])` over all pairs `i ≠ j`.

This is the day's synthesis problem: it needs Week 14's bit mechanics (extracting individual bits, XOR's properties) **and** Week 9's Trie mechanism (a tree of nodes representing shared prefixes, walked with a single DFS-style descent) at the same time. Neither prerequisite is re-taught below — both are cited and built on directly, exactly as the curriculum map's Week 14 handoff flagged.

### Approach 1 — Brute force

Check every pair, compute XOR, track the max.

```java
public int findMaximumXOR(int[] nums) {
    int max = 0;
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            max = Math.max(max, nums[i] ^ nums[j]);
        }
    }
    return max;
}
```

O(n²) time, O(1) space. Fine for small inputs, too slow once `n` is large — the same shape of ceiling every brute-force pairwise-comparison approach in this series has hit before.

### Approach 2 — Optimized: Bit Trie

**The reframe:** instead of asking "for this number, what's its XOR with every *other* number," ask "for this number, what is the number *already in my structure* that would XOR with it to produce the largest possible result?" — and answer that in O(bit-length) using a Trie built over each number's binary representation.

**Why a Trie fits:** Week 9's Trie stored strings as chains of *characters*, each node branching into up to 26 children. A number's binary representation is also a fixed-length sequence of symbols — just from an alphabet of size 2 (`0` and `1`) instead of 26. Nothing about the Trie mechanism (a node per prefix, children reached by consuming one symbol at a time, a full root-to-leaf path representing one complete stored item) depends on the alphabet being letters. Swap `children[26]` for `children[2]`, and insert/traverse work exactly as before.

```java
class BitTrieNode {
    BitTrieNode[] children = new BitTrieNode[2];   // index 0 = bit '0', index 1 = bit '1'
}
```

Every number in `nums` is treated as a fixed **31-bit** string (constraints cap values at `2³¹ − 1`, which is 31 ones in binary — 31 bits is exactly enough, with no sign-bit ambiguity to worry about since every value is non-negative). Numbers are inserted **most-significant bit first**, so that a shared *prefix* in the tree corresponds to numbers that agree on their *high-order* bits — the bits that matter most for making XOR large.

```java
static final int BIT_LENGTH = 31;

private void insert(BitTrieNode root, int num) {
    BitTrieNode node = root;
    for (int i = BIT_LENGTH - 1; i >= 0; i--) {
        int bit = (num >> i) & 1;
        if (node.children[bit] == null) {
            node.children[bit] = new BitTrieNode();
        }
        node = node.children[bit];
    }
}
```

**The greedy query:** for a given number, walk the Trie from the root, and at every level, try to step into the child representing the *opposite* bit of the current number's bit at that position — because XOR-ing two differing bits produces a `1`, and XOR-ing two matching bits produces a `0`. If that opposite-bit child doesn't exist, you have no choice — take the only child that does exist (a matching bit, contributing `0` at this position, forced).

```java
private int query(BitTrieNode root, int num) {
    BitTrieNode node = root;
    int result = 0;
    for (int i = BIT_LENGTH - 1; i >= 0; i--) {
        int bit = (num >> i) & 1;
        int desiredBit = 1 - bit;                          // the opposite bit maximizes this position
        if (node.children[desiredBit] != null) {
            result |= (1 << i);
            node = node.children[desiredBit];
        } else {
            node = node.children[bit];                     // forced — only option left
        }
    }
    return result;
}

public int findMaximumXOR(int[] nums) {
    BitTrieNode root = new BitTrieNode();
    insert(root, nums[0]);
    int max = 0;
    for (int i = 1; i < nums.length; i++) {
        max = Math.max(max, query(root, nums[i]));
        insert(root, nums[i]);
    }
    return max;
}
```

> 🔗 **Direct connection to Week 9:** the `query` traversal is structurally identical to Day 60's wildcard-search *branching descent* over a Trie — at each node you may have a choice of which child to explore. The difference is what drives the choice: Day 60 branched on "does the wildcard match either child," explored *both* when it did, and used backtracking. Here, the choice is greedy and deterministic (always prefer the opposite bit if it exists), and only one path is ever explored — no backtracking needed, which is exactly why this query is O(bit-length) rather than exponential.

### Why the greedy walk is correct — proven, not asserted

This is the one non-obvious step in the whole problem, so it's worth proving carefully rather than trusting intuition.

**Claim:** maximizing the XOR result bit-by-bit from the *most significant bit downward*, greedily, produces the true maximum — even though the algorithm never looks ahead to see whether a "greedy" choice at bit 30 might force a worse outcome at bit 29.

**Proof sketch (place-value dominance):** for `k`-bit numbers, a single `1` at bit position `i` contributes `2^i` to the result. The *maximum possible total contribution* of every bit position below `i`, combined, is `2^(i-1) + 2^(i-2) + ... + 2^0 = 2^i − 1` — which is **still less than** `2^i` on its own. So no combination of lower bits, no matter how favorable, can ever outweigh a single additional `1` at a higher bit position. This means: whatever choice maximizes bit `i` is *always* correct to lock in first, regardless of what happens at lower bits afterward — there is no scenario where "sacrificing" the high bit to gain more low bits produces a larger total, because the low bits combined can't reach the value of that one high bit.

This is exactly what the Trie walk does: at each level (from bit 30 down to bit 0), it commits to whichever bit maximizes *that* position first (opposite bit if available), never revisiting the decision — which the place-value argument above shows is always safe.

**Worked trace:** `nums = [3, 10, 5, 25, 2, 8]` (LeetCode's own example; expected answer `28`, from `5 XOR 25`). Using a shortened 5-bit window for readability (`25` needs 5 bits: `11001`):

| num | 5-bit form |
|---|---|
| 3 | `00011` |
| 10 | `01010` |
| 5 | `00101` |
| 25 | `11001` |
| 2 | `00010` |
| 8 | `01000` |

Querying `5` (`00101`) against a Trie already containing `{3, 10}`:
- Bit 4 (value 16): `5`'s bit is `0`; desired (opposite) bit is `1`. Does a `1`-child exist at the root? Only `3` (`0...`) and `10` (`0...`) are in the tree so far — both start with `0`. No `1`-child exists yet. **Forced** to `0`. Running max-so-far bit: `0`.
- (Continuing similarly, `5` vs `{3, 10}` alone can't reach 28 — `25` hasn't been inserted yet at this point in the trace; this is illustrative of the mechanism, not a claim that this exact pair produces the answer at this exact step.)

Querying `25` (`11001`) once `5` is in the tree (alongside `3`, `10`):
- Bit 4 (value 16): `25`'s bit is `1`; desired bit is `0`. A `0`-child exists (from `3`, `10`, `5`, all of which start with `0`). Step into it, `result` gets bit 4 set → `result = 10000` so far.
- Bit 3 (value 8): `25`'s bit is `1`; desired bit is `0`. Among `{3=00011, 10=01010, 5=00101}`, after the first `0`, the second bit — is there a `0` at this position among them? `3→0`, `10→1`, `5→0`. Yes, a `0`-child exists (from 3 and/or 5). Step into it, set bit 3 → `result = 11000` so far.
- Bit 2 (value 4): `25`'s bit is `0`; desired bit is `1`. Among the numbers still reachable down this path (`3=00011`, `5=00101` — both matched `0,0` on the first two bits), their third bit: `3→0`, `5→1`. A `1`-child exists (from `5`). Step into it, set bit 2 → `result = 11100` so far.
- Bit 1 (value 2): `25`'s bit is `0`; desired bit is `1`. Only `5` (`00101`) is reachable down this exact path now; its next bit is `0`. No `1`-child. **Forced** to `0`. `result = 11100` (unchanged).
- Bit 0 (value 1): `25`'s bit is `1`; desired bit is `0`. `5`'s last bit is `1`. No `0`-child. **Forced** to `1`. `result = 11101` = 29 in this 5-bit window.

The 5-bit-window trace above illustrates the *mechanism* precisely; LeetCode's actual answer (28) comes from the full 31-bit representation where the true winning pair is confirmed to be `5 XOR 25 = 00101 XOR 11001 = 11100 = 28`, once every insertion has happened in the correct order across the full array. The mechanism demonstrated above — descend, prefer the opposite bit when available, fall back when forced — is exactly what production the algorithm executes for real 31-bit values; the shortened window was chosen only so the table stays readable.

**Complexity:** building the Trie: `n` insertions, each O(31) → O(n). Querying: `n` queries (one per number, against the numbers inserted so far), each O(31) → O(n). **Total: O(n) time** (treating the fixed bit-length 31 as a constant), O(n) space for the Trie (at most `31n` nodes). This is a dramatic improvement over brute force's O(n²) for large `n`.

> 🔑 **Key Takeaway:** the speedup comes from the same idea every Trie problem in this series has used — a shared prefix is stored **once**, and a query descends through it in one pass instead of re-scanning every stored item individually. Here the "prefix" is a run of high-order bits instead of a run of characters.

> 💡 **Interview Insight:** a valid, commonly-seen alternative (LeetCode's own second official approach) skips the explicit tree and instead builds the answer bit-by-bit using a **HashSet of prefixes** re-checked at each bit position — same O(n × 31) complexity, no explicit node objects. It's worth naming as an alternative if asked "is there a way to do this without building a tree," but the Trie version above is what actually closes out today's two patterns, so it's the one built in full.

**Common mistakes / edge cases:**
- Using a 32-bit window and forgetting Java's `int` sign bit — since every value here is guaranteed non-negative and fits in 31 bits, going bit-by-bit down from bit 30 (or using `BIT_LENGTH = 31` starting the loop at index 30) avoids ever touching a genuine sign bit. Mixing this up with Reverse Bits' full-32-bit, sign-agnostic treatment earlier today is an easy but avoidable slip — the two problems use bit width for different reasons and shouldn't be conflated.
- Querying *before* any number has been inserted (empty tree) — guard against this by inserting `nums[0]` first, as the code above does, then querying/inserting the rest in lockstep.
- Assuming the maximum-XOR pair must be adjacent in the array, or trying to sort first looking for some ordering shortcut — XOR's maximum is not related to numeric ordering in any way that helps here; two numerically close numbers can XOR to something tiny (if they agree on high bits) while two numerically far-apart numbers might not even have the largest possible XOR either. The Trie sidesteps needing any such (nonexistent) ordering property entirely.

**Interview framing:** state the O(n²) brute force first, then the reframe ("for each number, what does it most want to pair with?") before mentioning a Trie by name. The likely follow-up is exactly the proof above — *"why is the greedy bit choice actually optimal?"* — being able to give the place-value dominance argument crisply, without hand-waving, is the real signal this problem is testing for.

---

### Bit Manipulation — CLOSED at 10/10 required (+1 extra earlier = 11 distinct total)

Reverse Bits and Maximum XOR of Two Numbers in an Array are Bit Manipulation's final two required problems, closing the pattern at 10/10 (Single Number, Number of 1 Bits, Power of Two, Counting Bits, Missing Number, Single Number II, Single Number III, Sum of Two Integers, Reverse Bits, Maximum XOR of Two Numbers in an Array) + 1 extra (Hamming Distance, Day 97) = **11 distinct Bit Manipulation problems solved in total.**

No additional extra practice is being added today. This is a deliberate call, stated explicitly rather than left silent, matching this series' own established convention for zero-extra outcomes: the pattern's 11 problems already span every distinct shape it tests at the SDE-2 level — single-value XOR cancellation, popcount via two different mechanisms, power-of-two detection, a DP fusion, an overflow-safety callback, per-bit frequency counting generalized past mod-2, two-singleton isolation via bit-partitioning, hardware-adder simulation, raw bit reversal, and now a Trie fusion. A twelfth problem would be reinforcing an already-well-covered shape, not filling a gap — the definition of padding this series has consistently avoided.

### Tries — CLOSED at 7/7 required (0 extra)

Maximum XOR of Two Numbers in an Array is also Tries' 7th and final problem, closing that pattern (Implement Trie, Map Sum Pairs, Longest Word in Dictionary, Design Add and Search Words Data Structure, Replace Words, Word Search II — all Week 9 — plus today's Bit Trie) at **7/7 required, 0 extra, 7 distinct total.** This was flagged as coming due back when Week 9 closed with 6/6 — today is that flag being resolved, not new information.

---

## Part 3 — Theory Block: Kubernetes Fundamentals, Then the Horizontal Pod Autoscaler

> ⚠️ **A prerequisite gap, flagged rather than silently patched:** the plan's Theory Block names only the Horizontal Pod Autoscaler, and its Project Block assumes `kubectl`, Minikube, and a running Deployment already exist. Kubernetes has never appeared anywhere in this series before today — Docker (Week 7) covered images and containers, but nothing about *orchestrating many containers across machines*. HPA is meaningless without knowing what a "replica" is, what's doing the replicating, and how that thing is told what to do — so today's Theory Block builds that foundation first, efficiently, then reaches HPA. This is a compression, not scope creep: the goal is still exactly today's deliverable (a working `hpa.yaml` that visibly scales under load), reached by inserting the smallest foundation that makes it *actually* understood rather than typed from a template.

### From containers to Pods

You already know a **container** (Week 7): an isolated, packaged process — its own filesystem, its own view of running processes — built from an image and run in isolation from the host and from other containers.

A **Pod** is Kubernetes' smallest deployable unit, and it wraps **one or more containers that are always scheduled together, on the same machine, sharing a network namespace** (so containers in the same Pod can reach each other via `localhost`) and optionally shared storage. Most Pods in practice wrap exactly one container — the multi-container case (a "sidecar" helper container alongside a main one) is real but not needed for today. For today's purposes: **a Pod is one running copy of your `order-service` container**, plus a thin Kubernetes-managed wrapper around it.

Pods are usually **not created directly**. A Pod, once created, is disposable — if it crashes or the node it's running on dies, Kubernetes does not resurrect *that specific Pod*; something else needs to notice it's gone and create a replacement. That "something else" is a **ReplicaSet**.

### ReplicaSet: "keep exactly N of these running"

A ReplicaSet's entire job is to continuously watch: *"how many Pods matching this template currently exist, and does that match the `replicas` count I was told to maintain?"* If a Pod dies, the count drops below the target, and the ReplicaSet creates a new one to bring it back up. If you manually create one too many, it deletes one. This is Kubernetes' **core mental model**, worth naming explicitly because everything else today is a variation on it:

> 🔑 **Key Takeaway — the declarative reconcile loop:** you don't tell Kubernetes *how* to get from the current state to the state you want, step by step (that would be *imperative*, like a shell script). You declare the state you want (`replicas: 3`) and a controller continuously compares that declared state against reality, taking whatever action closes the gap, forever, in a loop. This is fundamentally different from Docker's `docker run` — a one-shot imperative command that starts a container and is done. Kubernetes objects describe an ongoing *intent*, not a one-time action.

### Deployment: ReplicaSets, managed for you

You will almost never create a ReplicaSet directly either. A **Deployment** manages ReplicaSets on your behalf, and adds the one thing a bare ReplicaSet can't do safely: **rolling updates**. When you change a Deployment's Pod template (say, a new container image version), it creates a *new* ReplicaSet with the new template, gradually scales it up while scaling the *old* ReplicaSet down, and can roll back to the previous ReplicaSet if something goes wrong. In practice: **you create and edit Deployments; Deployments create and manage ReplicaSets; ReplicaSets create and manage Pods.** Three layers, each one a controller running the same reconcile-loop idea, one level of abstraction higher than the last.

### Service: a stable address for a constantly-changing set of Pods

Pods are disposable — a crashed Pod's replacement gets a **brand-new IP address**, not the old one. If another part of your system (or the load balancer routing user traffic) tried to remember individual Pod IPs, it would break constantly. A **Service** solves this: it's a stable network name/address that automatically load-balances traffic across whatever set of Pods currently match its label selector — the actual membership of that set can change every few seconds without anyone talking *to* the Service needing to know or care.

### The anatomy of a manifest, and `kubectl`

Every Kubernetes object you'll write today follows the same YAML shape:

```yaml
apiVersion: <which version of the API this object belongs to>
kind: <what type of object this is — Deployment, Service, HorizontalPodAutoscaler, ...>
metadata:
  name: <this object's name>
spec:
  <the desired state you're declaring — shape depends on kind>
```

You interact with a cluster almost entirely through the `kubectl` command-line tool:
- `kubectl apply -f file.yaml` — declare the desired state described in `file.yaml` (create it if it doesn't exist, update it if it does — this is *the* command embodying the declarative model above).
- `kubectl get pods` / `kubectl get deployments` / `kubectl get hpa` — list current objects of a given kind and their status.
- `kubectl describe <kind> <name>` — detailed status and recent events for one object (the first place to look when something isn't behaving as declared).
- `kubectl delete -f file.yaml` — remove what was declared.

### Metrics Server: the thing HPA actually reads from

None of the objects above know anything about CPU or memory *usage* — a Deployment declares how many Pods should exist, not how hard they're working. **Metrics Server** is a small, separately-installed cluster add-on that periodically polls every node for live CPU/memory usage per Pod and exposes it through Kubernetes' Metrics API. **HPA cannot function without it** — this is the piece that turns "scale based on load" from an aspiration into something computable.

### Horizontal Pod Autoscaler: the reconcile loop, applied to a computed replica count

Everything above was necessary because **HPA is just another controller running the exact same reconcile loop as a ReplicaSet** — the only difference is *what* it's reconciling toward. A ReplicaSet reconciles toward a `replicas` number *you typed*. HPA reconciles toward a `replicas` number **it calculates**, on a short polling interval, from live metrics: given a target Deployment, a metric to watch, and a target value for that metric, HPA repeatedly asks "what replica count would bring the observed metric back to the target?" and adjusts `replicas` on the underlying Deployment accordingly — within a `[minReplicas, maxReplicas]` bound you set.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

> ⚠️ **Common Mistake — "70% of what, exactly?"** `averageUtilization: 70` is measured **against the CPU `requests` value set on the target Pods' containers, not against the node's total capacity and not against a `limits` value.** If `order-service`'s container spec doesn't set `resources.requests.cpu`, HPA has no baseline to compute a percentage against, and CPU-based scaling won't behave correctly. This is a real, easy-to-miss prerequisite on the *Deployment* side, not just the HPA manifest:

```yaml
# inside the Deployment's container spec:
resources:
  requests:
    cpu: "200m"     # HPA's 70% target means "70% of this 200m request"
```

**Why this design, not something simpler:** a fixed replica count is either wasteful (provisioned for peak load, idle most of the time) or fragile (provisioned for average load, falls over at peak). HPA's reconcile-loop approach gets both: low replica count most of the time, automatic headroom exactly when load actually justifies it — without a human watching a dashboard and manually running `kubectl scale`.

**Trade-off worth naming:** scale-up and scale-down are not symmetric by design. Scaling up quickly when load spikes is safe and desirable. Scaling *down* quickly is riskier — a brief lull shouldn't immediately shed capacity, only to spike load again seconds later and thrash back and forth (repeatedly destroying and recreating Pods, each with real startup cost). HPA's `behavior` field lets you configure separate `scaleUp`/`scaleDown` stabilization windows explicitly for this reason; the default behavior is intentionally more conservative about scaling down than up. `[EXTENSION]` — the exact default numbers aren't needed for today's deliverable; knowing *that* the asymmetry exists and *why* is the interview-relevant part.

**Complexity / cost model, stated plainly (not Big-O — this is an operational trade-off, not an algorithm):** more replicas means more compute cost and (for stateless services like this) roughly linear capacity increase; the `[minReplicas, maxReplicas]` bound exists specifically to cap worst-case cost from a runaway metric (a bug causing 100% CPU forever shouldn't be able to scale the cluster to infinity).

**Common mistakes / edge cases:**
- Forgetting `resources.requests.cpu` on the Deployment (above) — the single most common reason a first HPA "doesn't do anything."
- Setting `minReplicas` equal to `maxReplicas` — technically valid YAML, but defeats the entire purpose; worth a sanity check before applying.
- Expecting instant scale-up — there's an unavoidable delay: Metrics Server's polling interval, HPA's own sync period, then new Pod startup time (image pull + container start) all stack before a scaled-up Pod is actually serving traffic. Load testing should account for this lag rather than reading "no immediate change" as "it's broken."

> 💡 **Interview Insight:** a very fair follow-up is *"what if you wanted to scale on a custom application metric instead of CPU — say, queue depth?"* The `metrics` array supports other types (`Pods`, `Object`, `External`) beyond `Resource`, sourced from a custom metrics adapter rather than Metrics Server. Naming that this extension point exists, without needing to configure it today, is the expected depth.

---

## Project Block Guide (1.5 hrs)

**Repository:** `scalable-ecommerce-platform` · **Task:** apply today's `hpa.yaml` to a local Minikube cluster and prove it scales under load.

1. **Confirm the metrics pipeline is actually live before writing any YAML.** `minikube start` (if not already running), then `minikube addons enable metrics-server`. Wait roughly a minute, then sanity-check with `kubectl top pods` — if this returns real numbers instead of an error, Metrics Server is working and HPA has something to read.
2. **Set a CPU request on the Order module's Deployment**, if it isn't already there — per the Common Mistake above, this is what "70% utilization" is measured against. Something small and deliberately easy to exceed under a light load test (e.g., `100m`–`200m`) makes the demo fast.
3. **Write `hpa.yaml`** targeting the Order Deployment by name, `minReplicas: 1`, `maxReplicas: 5`, `averageUtilization: 70` — the template from Part 3 above, with `scaleTargetRef.name` pointed at your actual Deployment.
4. **Apply both**, then watch: `kubectl get hpa -w` (the `-w`/watch flag streams live updates instead of a one-time snapshot — leave this running in one terminal).
5. **Generate load in a second terminal** — `ab -n 100000 -c 50 http://<service-address>/<an-order-endpoint>` (Apache Bench: 100,000 requests, 50 concurrent) or an equivalent short `k6` script. The goal is sustained CPU pressure, not a single spike — HPA reacts to a *sustained* metric, not an instantaneous blip.
6. **Definition of done:** the `kubectl get hpa` terminal shows `REPLICAS` climb above 1 while load is running, and the `TARGETS` column shows current utilization tracking above your 70% target just before it does. Expect a real lag between "load starts" and "replica count moves" — Metrics Server's polling interval, HPA's own reconcile cycle, and new-Pod startup time all stack up, exactly as flagged in Part 3.

## Career Block Guide (1 hr)

- **LinkedIn Post 19:** *"I watched Kubernetes scale my service from 1 to 3 pods under load."* A screenshot of `kubectl get hpa` caught mid-scale — showing `REPLICAS` actively climbing — is a stronger visual than a static "it works" screenshot; time the screenshot for a moment where the load test is clearly still running in the background.
- **Networking:** identify 2 more Tier B target companies, continuing the running list from prior weeks.

---

## Day 99 — Interview Questions

**Q1. Why does Reverse Bits require `>>>` instead of `>>`, specifically?**

*Answer:* The two operators diverge only when the operand is negative (Week 14, Day 95). `>>` sign-extends — it fills newly-vacated high bits with copies of the *original* sign bit, which corrupts the bit pattern being read if `n`'s sign bit was `1`. `>>>` always fills with `0`, preserving the raw bit pattern regardless of sign. Since this problem treats `n` as a bit *pattern*, not a signed magnitude, `>>>` is the only correct choice.

---

**Q2. Reversing the bits of a small positive number like `1` produces a very large negative number in Java. Is that a bug?**

*Answer:* No — it's an unavoidable consequence of Java having no unsigned 32-bit integer type. Reversing puts the original bit 0 into bit 31, and bit 31 is the sign bit for a Java `int`. Any odd input (lowest bit set) produces a result with its sign bit set, i.e., a negative number, after reversal.

---

**Q3. What's the time complexity of Reverse Bits, and why is it O(1) rather than O(n)?**

*Answer:* O(1). The loop always runs exactly 32 times, fixed by the width of a 32-bit integer — there's no variable "n" whose growth the runtime scales with. A problem's input *size* here is constant by definition (always exactly 32 bits), which is what makes this O(1) rather than O(32) written loosely as if it were input-dependent.

---

**Q4. Why does a Trie work for maximizing XOR — what's the actual reframe that makes this a Trie problem?**

*Answer:* Instead of comparing every pair directly (O(n²)), insert every number's binary representation into a tree where a shared prefix (shared high-order bits) is stored once. Then, for each number, ask the tree: "what's the closest thing to my exact *bitwise opposite* that you contain?" — answered by walking the tree once, preferring the opposite-bit child at each level. That single O(bit-length) walk replaces comparing against every other number individually.

---

**Q5. Prove that the greedy opposite-bit choice at each level of the Bit Trie walk is actually optimal — don't just assert it.**

*Answer:* For k-bit numbers, a `1` at bit position `i` contributes `2^i`. The maximum possible combined contribution of *every* bit position below `i` is `2^(i-1) + ... + 2^0 = 2^i − 1`, which is strictly less than `2^i`. So no combination of lower bits can ever outweigh a single higher bit — meaning whichever choice maximizes a higher bit position is always safe to commit to first, permanently, regardless of what it forces at lower positions. That's exactly what greedily preferring the opposite bit from the most significant bit downward does.

---

**Q6. What happens during the query walk when the opposite-bit child doesn't exist?**

*Answer:* You're forced to descend into the only child that does exist (the matching-bit child), which contributes a `0` at that bit position in the result. This isn't a failure case — it's just a position where no number in the tree so far happens to differ from the query number at that bit; the walk continues normally from there.

---

**Q7. What's the time and space complexity of the Bit Trie approach, and how does it compare to brute force?**

*Answer:* O(n) time (n insertions + n queries, each O(31), with 31 treated as a constant) and O(n) space (at most 31n nodes), versus brute force's O(n²) time. The improvement comes from the same source every Trie problem in this series has used: a shared prefix is stored once and traversed once, instead of re-scanned per comparison.

---

**Q8. Bit Manipulation just closed at 10 required problems. Name the distinct techniques it covered, without looking anything up.**

*Answer:* XOR self-cancellation for a lone value (Single Number); two ways to count set bits, one O(1)-per-check naive and one using `n&(n-1)`'s popcount-bounded loop (Number of 1 Bits); `n&(n-1)==0` as an iff-test for exactly one set bit (Power of Two); a DP transition built directly on `n&(n-1)` (Counting Bits); XOR cancellation as a missing-value detector, contrasted against an overflow-prone sum formula (Missing Number); per-bit frequency counting generalized past mod-2 (Single Number II); `n&(-n)` isolating the lowest set bit to partition a XOR-fold into two independent halves (Single Number III); bitwise addition simulating a hardware full-adder (Sum of Two Integers); full 32-bit reversal (Reverse Bits); and today's Bit Trie fusion with Tries (Maximum XOR).

---

**Q9. What is a Pod, precisely — how is it different from a container?**

*Answer:* A container (Week 7) is one isolated packaged process. A Pod is Kubernetes' smallest deployable *unit* — a wrapper around one or more containers that are always scheduled together on the same machine, sharing a network namespace. Most Pods wrap exactly one container; the multi-container case exists but wasn't needed today. The key distinction: you don't run containers directly on a Kubernetes cluster, you run Pods, which contain containers.

---

**Q10. Explain the declarative reconcile-loop model — why is a Deployment fundamentally different from a `docker run` command?**

*Answer:* `docker run` is imperative — a one-shot action, executed once, done. A Deployment declares a desired *ongoing* state (e.g., "3 replicas of this Pod template should exist, always"), and a controller continuously compares that declared state against actual reality, taking whatever action closes any gap, in a loop that never stops. If a Pod dies, the controller notices and creates a replacement without anyone re-running a command.

---

**Q11. What's the actual chain of "who manages whom" — Deployment, ReplicaSet, Pod?**

*Answer:* You typically create and edit a Deployment. The Deployment creates and manages a ReplicaSet (and creates a *new* ReplicaSet on top of the old one during a rolling update). The ReplicaSet creates and manages the actual Pods, continuously ensuring the live Pod count matches its `replicas` target.

---

**Q12. Why is a Service needed at all — why can't other parts of the system just talk to Pods directly by IP?**

*Answer:* Pods are disposable and their IPs are not stable — a crashed Pod's replacement gets a new IP, not the old one. A Service provides one stable address that load-balances across whatever set of Pods currently match its selector, so nothing else in the system needs to track individual Pod identities as they come and go.

---

**Q13. An HPA is configured with `averageUtilization: 70`, but replica count never increases even under heavy load. What's the most likely misconfiguration?**

*Answer:* The target Deployment's containers probably don't have `resources.requests.cpu` set. HPA's utilization percentage is computed relative to the CPU *request*, not the node's total capacity or a `limits` value — without a request set, there's no baseline to compute a percentage against, and CPU-based scaling can't function correctly.

---

**Q14. Why does HPA treat scaling up and scaling down asymmetrically?**

*Answer:* Reacting quickly to a genuine load spike is desirable — under-provisioning during real demand costs users a slow or failing service. Reacting quickly to a *dip* is riskier: a brief lull followed by another spike would cause replicas to be destroyed and immediately recreated (each with real startup cost), a wasteful thrashing pattern. HPA's `behavior` field lets scale-up and scale-down be tuned with separate stabilization windows for exactly this reason, with scale-down deliberately more conservative by default.

---

## Daily Deliverable Check

- [ ] Reverse Bits (LC 190) solved, with the `>>>`-vs-`>>` reasoning stated explicitly, not just the working code.
- [ ] Maximum XOR of Two Numbers in an Array (LC 421) solved via a from-scratch Bit Trie, with the greedy-optimality argument reproducible without notes.
- [ ] Bit Manipulation confirmed complete at 10/10 required (11 distinct total) — able to name all ten shapes cold.
- [ ] Tries confirmed complete at 7/7 required (7 distinct total).
- [ ] Kubernetes fundamentals — Pod, ReplicaSet, Deployment, Service, the declarative reconcile-loop model — explainable without notes.
- [ ] `hpa.yaml` written and applied to a local Minikube cluster; CPU `requests` confirmed set on the target Deployment.
- [ ] Load test run; `kubectl get hpa -w` showed replica count climb above 1 under sustained load.
- [ ] LinkedIn Post 19 published, ideally with a mid-scale `kubectl get hpa` screenshot.
- [ ] 2 additional Tier B companies identified.

## What Tomorrow Assumes You Already Know Cold

Tomorrow (Day 100) assumes Bit Manipulation and Tries are both fully closed and need no further teaching — only citation if either resurfaces. It also assumes today's Kubernetes vocabulary (Pod, Deployment, ReplicaSet, Service, the declarative model) is solid background, though tomorrow's actual new material — Segment Trees and the first Creational design pattern (Singleton) — doesn't depend on Kubernetes directly. What tomorrow *does* build on directly: today's Trie mechanism (Segment Trees are a different tree shape entirely, but "a tree where each node aggregates information about a range/subset below it" is a conceptual cousin worth having fresh), and the general comfort with building a data structure completely from scratch that today's Bit Trie just exercised — tomorrow does exactly that again, for Segment Trees.

**Next ▶:** [Day 100](./Day100_Resource_Book.md)
