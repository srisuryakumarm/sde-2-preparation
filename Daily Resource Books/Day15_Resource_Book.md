# SDE-2 Resource Book Series
## Day 15 — Sliding Window Continues, and HashMap Internals

**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**← Previous:** [Day 14](Day14_Resource_Book.md) &nbsp;|&nbsp; **Next →:** [Day 16](Day16_Resource_Book.md)
**Companion to:** Day 15 of `Week_03_Revised.md`

---

### Recap

Day 14 closed Week 2 by introducing Sliding Window as a named pattern — the direct evolution of same-direction (fast/slow) Two Pointers — and used it exactly once, in its simplest form: **fixed-size**, via Maximum Average Subarray I (LC 643), where both edges of the window move in lockstep. Today introduces the other half of the pattern — **variable-size** windows, where the two edges move independently based on a constraint — which is genuinely new mechanism, not yet used anywhere in the series.

Today's theory also reaches back further, to Day 4, where HashMap was first taught (key→value, hash-function-into-buckets intuition, and the equals()/hashCode() contract stated as a rule). Today we open that box up and look at the actual mechanism.

---

### Learning Objectives

By the end of today, without notes, you should be able to:
1. Write the variable-size sliding window template from memory and explain why its total runtime is O(n) despite a while-loop nested inside a for-loop.
2. Solve Max Consecutive Ones III and Longest Substring Without Repeating Characters using that template.
3. Explain, step by step, how a HashMap turns a key into a bucket index — including *why* the spreading step exists, with a concrete example of a collision it prevents.
4. State precisely when a bucket treeifies, what it treeifies into, and why.
5. Distinguish a hashCode() that's merely *inefficient* from one that actually *violates the equals()/hashCode() contract* — and explain why only the second one causes `get()` to silently return null for a key you know you inserted.

---

### Concept Dependency Map

```
Day 4 — HashMap/HashSet basics                Day 4 — HashSet (membership)
  equals()/hashCode() contract STATED                  │
        │                                              │
        ▼                                              ▼
Day 15A — HashMap Internals (DEEPENING)      Day 15B — Variable-Size Window (NEW)
  hashCode() → spread → bucket index           expand right → check valid →
  collision chaining → treeification            shrink left while invalid →
  (Java 8, threshold 8, min capacity 64)         record answer
        │                                              │
        ▼                                              ▼
  HashCodeContractDemo (Project)          LC 1004 — shrink while zeroCount > k
                                            LC 3 — HashSet, shrink while duplicate
                                            LC 1695 (Extra) — HashSet + running sum

Day 3 — Amortized Analysis (ArrayList.add() doubling)
        │
        ├──reused for──▶ HashMap resize/rehash (amortized O(1) put/get)
        └──reused for──▶ "total left/right movement ≤ 2n" argument below
```

---

## Part A — HashMap Internals

### A.1 What you already know (Day 4), and what's new today

Day 4 gave you the *contract as a rule*: two objects that are `.equals()` must return the same `.hashCode()`. Today gives you the *mechanism that makes the rule necessary* — because once you see exactly how a HashMap decides which bucket a key lives in, "why" stops being something you have to memorize and becomes something you can derive on the spot, which is what a tier-1 interviewer is actually checking for.

### A.2 From hashCode() to a bucket index

A `HashMap<K,V>` is, underneath, an array of buckets (`Node<K,V>[] table`). Every key needs to land in exactly one bucket, deterministically. The process:

**Step 1 — raw hash.** Call `key.hashCode()`. This returns some `int` — could be anything across the full 32-bit range, positive or negative.

**Step 2 — spread.** Java doesn't use that raw int directly. It computes:
```java
static int spread(int h) {
    return h ^ (h >>> 16);
}
```
This XORs the hash with its own upper 16 bits shifted down into the lower 16 bits.

**Step 3 — bucket index.** `index = (n - 1) & hash`, where `n` is the table's current length. Because `n` is *always* a power of two, `n - 1` in binary is a solid run of 1-bits (e.g., `n=16` → `n-1=15=0b1111`), so `(n-1) & hash` is a bitmask that keeps only the low bits of `hash` — mathematically equivalent to `hash % n`, but a bitwise AND is cheaper than a modulo, which is why HashMap enforces power-of-two capacities specifically to make this trick legal.

### A.3 Why the spreading step exists — a concrete collision it prevents

🔑 **Key Takeaway:** without spreading, `(n-1)&hash` only ever looks at the *low* bits of the hash. If two keys' hashCodes differ only in *high* bits, they'd collide every time in a small table — spreading folds the high bits down so they influence the low bits too.

Take two keys with hashCodes `h1 = 0x00010000` and `h2 = 0x00020000` (they differ only in bits 16–17; their low 16 bits are both zero). Table size `n = 16`, so `n-1 = 0xF`.

*Without* spreading: `h1 & 0xF = 0`, `h2 & 0xF = 0` — **both land in bucket 0.** Guaranteed collision, purely from a table that's too small to see the difference.

*With* spreading: `h1 >>> 16 = 0x0001`, so `spread(h1) = 0x00010000 ^ 0x00000001 = 0x00010001`, and `spread(h1) & 0xF = 1`. Meanwhile `spread(h2) = 0x00020000 ^ 0x00000002 = 0x00020002`, and `spread(h2) & 0xF = 2`. **Different buckets.** The high-bit difference got folded down where the mask could see it.

### A.4 Collision handling and treeification

Multiple keys landing in the same bucket is expected and fine — collisions don't break correctness, only speed. Within a bucket, entries are stored as a **linked list** of `Node<K,V>` by default. A `get(key)` computes the bucket index, then walks that bucket's list checking `key.equals(entry.key)` (with a `==` short-circuit first) until it finds a match.

If a *single bucket's* list grows to `TREEIFY_THRESHOLD = 8` entries **and** the table's overall capacity is at least `MIN_TREEIFY_CAPACITY = 64`, that bucket converts its linked list into a small **red-black tree** (ordered by hash, tie-broken by class comparison) — worst-case lookup within that one bucket drops from O(k) to O(log k). If the table is smaller than 64 buckets when a bucket hits 8, HashMap resizes (doubles) the whole table *instead* of treeifying — a bigger table usually spreads that one crowded bucket back out, so treeifying a table that's just plain undersized would be treating the symptom, not the cause. If removals later shrink a treeified bucket back down to `UNTREEIFY_THRESHOLD = 6`, it converts back to a plain list (a tree has more per-node overhead than a linked list, not worth carrying for a nearly-empty bucket).

⚠️ **Common Mistake:** thinking every HashMap bucket is a tree. It isn't — treeification is a rare safety net for pathologically bad hashCode() distributions (or adversarial input designed to force collisions), not the normal case. In normal use, with a reasonable hashCode(), you never see it.

### A.5 Resize — connecting back to Day 3's amortized analysis

Default initial capacity is 16, default load factor is 0.75. Once `size > capacity * loadFactor`, the table doubles and every existing entry gets rehashed into the new, larger table. That rehash is O(n) — but exactly like `ArrayList`'s doubling (Day 3) and `StringBuilder`'s buffer doubling (Day 11), it happens rarely enough (each doubling roughly doubles the amount of work you'd have to do again before the *next* one) that the amortized cost per `put()` stays O(1). Same argument, same shape, third time it's shown up — 🔗 cite it exactly as you did on Day 3, don't re-derive it from scratch.

### A.6 The contract, precisely — and the bug it's easy to conflate it with

Here's the rule again, now that you can see the mechanism behind it: **if `a.equals(b)` is true, then `a.hashCode()` must equal `b.hashCode()`.** Given Part A.2–A.3, you can now see exactly why: `get(b)` computes `b`'s bucket from `b.hashCode()` and only ever searches *that* bucket. If `a` and `b` are logically equal but have different hash codes, `a` might live in bucket 5 while `get(b)` searches bucket 12 — it will never find `a`, because it never even looks in bucket 5. This isn't `get()` being slow; it's `get()` returning the *wrong answer* (null, for a key that's logically present).

⚠️ **Common Mistake — the one this week's exercise is built to expose:** people conflate "a bad hashCode()" with "a hashCode()/equals() contract violation." They're different bugs with different symptoms:

| Bug | What it looks like | Symptom |
|---|---|---|
| **Constant hashCode()** (e.g., always returns `0`) | Every key lands in the *same* bucket | Still finds the right value — `.equals()` correctly picks it out during the (now very long) bucket scan. **Slow, not wrong.** Degrades toward O(n) per lookup. |
| **Contract violation** (`.equals()` overridden but `.hashCode()` isn't, or the two disagree) | Two logically-equal objects land in *different* buckets | `get()` on a "duplicate" instance searches the wrong bucket entirely and returns **null**, even though an equal object is sitting in the map. **Wrong, not just slow.** |

You'll build both, side by side, in today's project, so the distinction stops being abstract.

Here is the exact **HashMap `put()` flow**, with the terminology and the internal details we covered:

```text
                              key
                               |
                               ↓
                          hashCode()
                               |
                               ↓
                           raw hash
                         (32-bit int)
                               |
                               ↓
                    split conceptually into
                    upper 16 + lower 16 bits
                               |
                               ↓
                         h >>> 16
                    (upper bits moved down)
                               |
                               ↓
                    h ^ (h >>> 16)
                         = spread hash
                               |
                               ↓
                  n = current table length
                               |
                               ↓
                      n - 1 = bit mask
                               |
                               ↓
                    (n - 1) & spreadHash
                               |
                               ↓
                       bucket index
                               |
                               ↓
                         table[index]
                               |
                               ↓
                     Is bucket empty?
                         /          \
                       yes           no
                        |             |
                        ↓             ↓
                   Add new Node   Compare keys
                                      |
                                      ↓
                                   equals()?
                                   /      \
                                 yes       no
                                  |         |
                                  ↓         ↓
                              Same key    Collision
                                  |         |
                                  ↓         ↓
                               Replace   Next Node
                               value         |
                                            ↓
                                      More nodes?
                                      /       \
                                    yes        no
                                     |          |
                                     ↓          ↓
                                  equals()    Add new Node
                                  again
```

And the **collision/treeification branch** continues as:

```text
                         Collision
                             |
                             ↓
                      Add/search next node
                             |
                             ↓
                    Bucket becomes crowded?
                             |
                             ↓
                   TREEIFY_THRESHOLD ≈ 8
                             |
                    Is capacity >= 64?
                       /             \
                     no               yes
                     |                 |
                     ↓                 ↓
                   Resize          Treeify
                     |                 |
                     ↓                 ↓
                 table grows      Red-Black Tree
                     |                 |
                     ↓                 ↓
               redistribute       O(log k) lookup
                  entries
```

And the **overall resize branch** is separate:

```text
                    HashMap size
                         |
                         ↓
             size > capacity × 0.75 ?
                    /           \
                  no             yes
                  |               |
                  ↓               ↓
             Continue          Resize
                                  |
                                  ↓
                           16 → 32 → 64 → ...
                                  |
                                  ↓
                     Existing hashes are reused
                                  |
                                  ↓
                       Entries redistributed
                                  |
                           +------+------+
                           |             |
                      same bucket   old + oldCapacity
```

The single flow you should keep in your head is:

```text
key
 ↓
hashCode()
 ↓
raw hash
 ↓
spread
 ↓
bucket index
 ↓
table[index]
 ↓
equals()
 ↓
exact key
 ↓
value / collision
```

And the three concepts that should **never be mixed together** are:

```text
hashCode()  → produces the hash
capacity    → determines how the hash becomes a bucket index
equals()    → determines whether the key in that bucket is actually the same key
```

---

## Part B — Variable-Size Sliding Window (New Mechanism)

### B.1 The template

Day 14's fixed-size window always had `right - left + 1` equal to a constant `k` — both pointers moved together. A **variable-size** window instead grows and shrinks based on whether the current window still satisfies some constraint:

```java
public int variableWindowTemplate(int[] nums /*, other params */) {
    int left = 0;
    int best = 0; // or Integer.MAX_VALUE, if you're minimizing a window size instead
    // running state describing the window's current contents, e.g. a count or sum

    for (int right = 0; right < nums.length; right++) {
        // 1) Expand: fold nums[right] into the running state.

        // 2) Shrink while [left, right] violates the constraint.
        while (/* window is currently invalid */) {
            // remove nums[left] from the running state
            left++;
        }

        // 3) [left, right] is now valid — update the answer.
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

### B.2 Why this is O(n), not O(n²) — the argument, not just the label

⚠️ **Common Mistake:** looking at a `while` loop nested inside a `for` loop and assuming O(n²). That reasoning is about *syntax*, not about *actual total work done* — and here it's wrong.

🔑 **The real argument:** `right` is incremented exactly once per outer-loop iteration, so it moves at most `n` times, total, across the whole run. `left` only ever moves *forward* — it never resets backward — so across the **entire algorithm's execution**, not per outer iteration, `left` can also be incremented at most `n` times before it runs off the end of the array. Total work is therefore bounded by (work per `right` step) + (work per `left` step, summed across every inner-loop execution combined) = O(n) + O(n) = **O(n)**, regardless of how the loops are nested syntactically. This is the same style of "total pointer movement" argument you've used since Two Pointers — the nesting looks scarier than the bound actually is.

### B.3 Contrast with Day 14

|  | Fixed-size (Day 14, e.g. LC 643) | Variable-size (today) |
|---|---|---|
| Both edges move together? | Yes, in lockstep | No — right always advances; left advances only on violation |
| Window size | Constant `k` | Changes every iteration |
| Signal in the wording | "of size k," "of length k" | "longest / shortest subarray such that…", "at most k…" |

---

## Problem 3: Max Consecutive Ones III

**LeetCode #1004 — Medium — Pattern: Sliding Window (variable)**

**Statement:** given a binary array `nums` and an integer `k`, return the length of the longest subarray containing only 1s after flipping at most `k` zeros to 1s.

**Brute force:** for every pair `(i, j)`, count the zeros in `nums[i..j]`; keep the longest range with zero-count ≤ k. Time O(n²) (or O(n³) if you recount zeros from scratch instead of extending a running window), Space O(1).

**Optimal — variable window:** track `zeroCount` in the current window. A window is valid while `zeroCount <= k`.

```java
public int longestOnes(int[] nums, int k) {
    int left = 0, zeroCount = 0, best = 0;
    for (int right = 0; right < nums.length; right++) {
        if (nums[right] == 0) zeroCount++;
        while (zeroCount > k) {
            if (nums[left] == 0) zeroCount--;
            left++;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Why it works:** this is a direct instance of the Part B template — "invalid" means `zeroCount > k`; shrinking always removes exactly the leftmost element from the running zero count.

**Trace:** `nums = [1,0,1,1,0,1]`, `k = 1`.

| right | nums[right] | zeroCount | shrink? | left | window len | best |
|---|---|---|---|---|---|---|
| 0 | 1 | 0 | no | 0 | 1 | 1 |
| 1 | 0 | 1 | no (1≤1) | 0 | 2 | 2 |
| 2 | 1 | 1 | no | 0 | 3 | 3 |
| 3 | 1 | 1 | no | 0 | 4 | 4 |
| 4 | 0 | 2 | yes → left 0→1 (no zero removed)→2, left 1→2 (zero removed)→1 | 2 | 3 | 4 |
| 5 | 1 | 1 | no | 2 | 4 | **4** |

Answer 4 — matches "flip the zero at index 1, giving `[1,1,1,1,0,1]`, longest run 4."

**Complexity:** Time O(n) (Part B.2's argument). Space O(1).

**Edge cases & mistakes:**
- ⚠️ `k = 0`: degenerates to "longest run of 1s with no flips" — the template still handles it correctly (zeroCount can never exceed 0, so any 0 immediately forces a shrink until it's expelled).
- ⚠️ `k >= (count of all zeros in nums)`: the whole array becomes the answer — verify your loop doesn't special-case this; it shouldn't need to.
- ⚠️ Forgetting to decrement `zeroCount` *only* when the element leaving the window was actually a zero (a very easy off-by-one to introduce under interview pressure).

**💡 Interview framing:** say out loud, before coding: "this is 'longest window where a violation count stays ≤ k' — I'll expand right, track zeros, and shrink from the left only when zeros exceed k." Likely follow-up: "what if you needed to flip 0s to 1s *and* 1s to 0s, whichever is cheaper?" — gesture at running two symmetric windows (or generalizing the "violation" definition), without needing to solve it live.

---

## Problem 4: Longest Substring Without Repeating Characters

**LeetCode #3 — Medium — Pattern: Sliding Window (variable)**

**Statement:** given a string `s`, return the length of the longest substring without repeating characters.

**Brute force:** for every substring, check for duplicates using a HashSet. O(n³) naively (O(n²) substrings × O(n) to build/check each), or O(n²) if you build the set incrementally per starting index. Space O(min(n, charset)).

**Optimal — variable window with a HashSet:**

```java
public int lengthOfLongestSubstring(String s) {
    Set<Character> window = new HashSet<>();
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        while (window.contains(c)) {
            window.remove(s.charAt(left));
            left++;
        }
        window.add(c);
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Why it works:** "invalid" here means "the incoming character is already in the window." Shrinking one character at a time from the left is guaranteed to eventually remove the *specific* duplicate blocking `c`, because `c` can only appear once in the current window before this step.

**Trace:** `s = "abcabcbb"`.

Expand a,b,c cleanly to `best=3`. At `right=3` (`'a'`), `'a'` is already in the window `{a,b,c}` → shrink once (remove `s[0]='a'`, `left=1`) → `'a'` no longer present → add it, window `{b,c,a}`, length 3. At `right=6` (`'b'`), `'b'` is in `{a,b,c}` → shrink twice in a row (removing `s[3]='a'` then `s[4]='b'`, since the first shrink still leaves `'b'` in the window) before `'b'` is finally absent. Final answer: **3** (`"abc"`).

**Complexity:** Time O(n) (each character added/removed from the set at most once, per B.2's argument). Space O(min(n, charset size)) — the window can never hold more distinct characters than exist in the alphabet in play.

**Edge cases & mistakes:**
- ⚠️ Empty string → should return 0; the loop simply never executes, `best` stays 0. Verify this rather than assuming it.
- ⚠️ All-identical-characters string (`"aaaa"`) → answer should be 1; make sure your shrink loop can fire multiple times per `right` step if needed (it can, and here it will each time).
- 💡 **Optimization worth knowing, not required today:** a HashMap of `char → lastSeenIndex` lets you jump `left` directly to `lastSeenIndex + 1` instead of shrinking one character at a time — same O(n) bound, smaller constant factor. Mentioned as extension material; the HashSet version above is the one to have cold.

**💡 Interview framing:** name the pattern immediately: "sliding window, shrink on a HashSet membership violation." Likely follow-up is exactly the HashMap-index optimization above — have the one-sentence version ready even if you don't implement it live.

---

## Extra Practice: Maximum Erasure Value

**LeetCode #1695 — Medium — Pattern: Sliding Window (variable, HashSet uniqueness + running aggregate)**

✅ **Overlap check:** not present anywhere in `00_Curriculum_Map.md`'s problem inventory, and not required by `Week_04_Revised.md`. Added because Sliding Window is this week's marquee pattern (grows to 14 required problems) and deserves reps beyond the plan's two-per-day minimum — this one pairs directly with LC 3's technique above, adding a genuinely new wrinkle: a running *aggregate* (sum) tracked alongside the uniqueness constraint, not just a length.

**Statement:** given an array of positive integers, choose a contiguous subarray with all-unique elements; your score is its sum. Return the maximum possible score.

**Brute force:** check every subarray for uniqueness (HashSet) and sum it. O(n²) with an O(1)-amortized-per-element check inside, or O(n³) naively. Space O(n).

**Optimal:**

```java
public int maximumUniqueSubarray(int[] nums) {
    Set<Integer> window = new HashSet<>();
    int left = 0, windowSum = 0, best = 0;
    for (int right = 0; right < nums.length; right++) {
        while (window.contains(nums[right])) {
            window.remove(nums[left]);
            windowSum -= nums[left];
            left++;
        }
        window.add(nums[right]);
        windowSum += nums[right];
        best = Math.max(best, windowSum);
    }
    return best;
}
```

**Why it works:** identical shrink condition to LC 3 (duplicate membership), but now the running state carries *two* pieces of information in lockstep — the set (for the validity check) and the sum (for the answer) — both updated together on every expand and every shrink step. This is the first time this week you're tracking more than one piece of window state at once; Day 16 and beyond lean on this same "set/map plus a scalar" shape repeatedly.

**Trace:** `nums = [4,2,4,5,6]`. Expand 4, 2 cleanly (`sum=6`). At `right=2` (`nums[2]=4`), 4 is already in the window → shrink once (remove `nums[0]=4`, `sum=6-4=2`, `left=1`) → add the new 4 → window `{2,4}`, `sum=6`. Expand 5 (`sum=11`), expand 6 (`sum=17`). Final answer: **17** (the subarray `[2,4,5,6]`, formed by erasing the leading duplicate 4).

**Complexity:** Time O(n), Space O(n) worst case (window can hold up to n distinct elements). Reasoning identical to LC 3's.

**Edge cases & mistakes:**
- ⚠️ Subtracting from `windowSum` *before* removing the element from the set (or vice versa) doesn't matter for correctness here since they're independent statements, but keep them adjacent in your code — separating them is a common source of "I updated one and forgot the other" bugs under time pressure.
- ⚠️ All-unique input array → the shrink loop never fires; make sure `best` still gets updated every iteration (it does, since step 3 of the template runs unconditionally).

**💡 Interview framing:** "same shrink-on-duplicate skeleton as Longest Substring Without Repeating Characters, but the thing I'm optimizing is a sum instead of a length — so I carry an extra running total through the same expand/shrink steps."

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals` · **Task:** `HashCodeContractDemo`

Build one class with two clearly-separated, well-commented demonstrations — matching the distinction from Part A.6:

1. **`demoConstantHashCode()`** — a `BadHashPoint` class with a correct `equals()` (compares `x`/`y`) but `hashCode()` hardcoded to return `0`. Insert ~100,000 instances into a `HashMap`, then `get()` one back by an equal key. It **succeeds** — print the value found — but add a comment explaining that every entry lives in one bucket's chain, so this `get()` did a near-linear scan instead of O(1).
2. **`demoBrokenContract()`** — a `BrokenContractPoint` class with a correct `equals()` but **no** `hashCode()` override (inherits `Object`'s identity-based one). `put()` one instance, then `get()` using a *different* instance that's `.equals()` to it. It returns **null** — print that, with a comment explaining why (different identity hash → different bucket → never found).
3. **Fix:** override `hashCode()` on `BrokenContractPoint` consistently with `equals()` (e.g., `Objects.hash(x, y)`), re-run the same lookup, and show it now succeeds.

**Definition of done:** all three demos run and print their described behavior; a comment in your own words states the equals()/hashCode() contract rule.

---

## Career Block Guide (1 hr)

- **LinkedIn Post 5 — HashMap internals:** lead with the treeification detail specifically (it's the most "I didn't know that" fact of today's theory) — e.g., open with the fact that a HashMap bucket can silently become a tree, then explain why. Keep it to a few paragraphs; a code snippet of the bucket-index formula is a good visual anchor.
- **Networking — 5 connection requests to Backend Engineers:** since these are follow-ups to targets you identified the prior day, personalize each request with one specific, genuine reason you're reaching out (a shared alma mater, a project of theirs you looked at, a role that matches your target) rather than sending the default blank invite — a one-line personalized note meaningfully changes acceptance rates.

---

## Day 15 — Interview Questions

**Q1. Walk me through what happens, mechanically, when you call `map.get(key)`.**
A: Compute `key.hashCode()`, spread it (`h ^ (h>>>16)`), mask with `(n-1)` to get a bucket index, then walk that bucket's chain (list or tree) comparing `key.equals()` against each entry until a match is found or the chain is exhausted.

**Q2. Why does the bucket-index formula use `(n-1) & hash` instead of `hash % n`?**
A: They're equivalent whenever `n` is a power of two, but the bitwise AND is cheaper than a modulo — which is exactly why HashMap enforces power-of-two capacities.

**Q3. When does a bucket treeify, and into what?**
A: When that specific bucket's chain reaches 8 entries (`TREEIFY_THRESHOLD`) **and** the table's overall capacity is at least 64 (`MIN_TREEIFY_CAPACITY`); it converts from a linked list of `Node`s to a red-black tree. Below capacity 64, the table resizes instead of treeifying.

**Q4. If `hashCode()` always returns the same constant, does `HashMap.get()` still return the correct value?**
A: Yes — every key lands in the same bucket, so the map degrades to a single long chain, but `.equals()` still correctly identifies the right entry during the scan. It's a performance bug (toward O(n)), not a correctness bug.

**Q5. Give an example where `get()` returns null for a key you know is logically present.**
A: Override `.equals()` to compare by value but leave `.hashCode()` unoverridden (or inconsistent) — two "equal" instances get different identity-based hash codes, land in different buckets, and `get()` on one instance never even looks in the bucket where the other lives.

**Q6. Why is the total runtime of a variable-size sliding window O(n) and not O(n²), given the nested loop?**
A: `right` advances at most n times total; `left` only ever moves forward and also advances at most n times total across the *entire* run (not per outer iteration) — so total work is O(n) + O(n) = O(n), regardless of the syntactic nesting.

**Q7. What's the concrete signal in a problem statement that tells you "variable-size window," as opposed to fixed-size?**
A: Wording like "longest/shortest subarray such that…" or "at most k…" — the window's size itself is what you're solving for, versus fixed-size wording like "of size k."

**Q8. In Max Consecutive Ones III, why do you only decrement `zeroCount` when the element leaving the window was a zero?**
A: `zeroCount` tracks how many zeros are *currently inside* the window; removing a 1 from the window doesn't change that count, so decrementing unconditionally would undercount.

**Q9. In Longest Substring Without Repeating Characters, why is a single shrink step (versus a while-loop of shrinks) sometimes not enough?**
A: If the window contains other characters between the old occurrence of `c` and the current position, they need to be evicted one at a time until the specific blocking duplicate is gone — that can take more than one removal, which is exactly why it's a `while`, not an `if`.

---

## Daily Deliverable Check

- [ ] Max Consecutive Ones III (LC 1004) and Longest Substring Without Repeating Characters (LC 3) solved, pushed to `dsa-java/sliding-window/`.
- [ ] **Extra:** Maximum Erasure Value (LC 1695) solved, same folder.
- [ ] Can explain treeification and the equals()/hashCode() contract without notes — including the constant-hashCode-vs-contract-violation distinction.
- [ ] `HashCodeContractDemo` pushed, with both demos and the fix.
- [ ] LinkedIn Post 5 published.

---

### What Tomorrow Assumes You Already Know Cold

Day 16 reuses today's variable-size window template without re-deriving it — Longest Repeating Character Replacement will be introduced as "the same shrink-on-violation shape, with a new validity check." It also assumes HashSet/HashMap usage is fully fluent (no mechanism explanation will be repeated). Generics, today's theory's neighbor, has no hard dependency on today's material — it builds on Day 2's class fundamentals and Day 3's ArrayList as a generic example you've already used without the formal vocabulary for it.
