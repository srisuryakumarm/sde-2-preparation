# SDE-2 Resource Book Series
## Day 19 — Sliding Window: Two-Pointer Hard Tier

**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**← Previous:** [Day 18](Day18_Resource_Book.md) &nbsp;|&nbsp; **Next →:** [Day 20](Day20_Resource_Book.md)
**Companion to:** Day 19 of `Week_03_Revised.md`

No new theory today — both problems are Hard-tier and earn the full DSA block, exactly as the plan intends.

---

### Recap

Problem 11 is Day 18's Fruit Into Baskets (at-most-2-distinct) generalized: the shrink condition becomes `window.size() > K` for a parameter `K`, instead of the hardcoded `> 2`. Problem 12 returns to the shrink-while-valid (minimize) loop shape from Day 17's Minimum Size Subarray Sum, but with a validity check built from character coverage instead of a numeric sum — today's project block asks you to nail down exactly what's the same and what's different between the two.

---

### Learning Objectives

By the end of today, without notes, you should be able to:
1. Write `atMostKDistinct(s, k)` as a standalone, reusable helper, and explain why Day 18's Fruit Into Baskets was really a special case of it.
2. Solve Minimum Window Substring using a `required`/`formed` counter pair, and explain precisely when `formed` increments and decrements (and why it isn't simply "every time a needed character is seen").
3. State, precisely, what's identical and what's different between Minimum Size Subarray Sum (Day 17) and Minimum Window Substring's shrink loops.

---

### Concept Dependency Map

```
Day 18 — LC 904 (at-most-2-distinct, HashMap.size() > 2)
        │
        ▼
Day 19: LC 340 — same shape, K as a parameter
   (window.size() > k)

Day 17 — LC 209 (shrink-while-valid, sum threshold ≥ target)
        │
        ▼
Day 19: LC 76 — shrink-while-valid, NEW check:
   character coverage via required/formed counters
        │
        ▼
  Project: comparison note — LC 209 vs. LC 76,
  identical loop shape, different validity check
```

---

## Problem 11: Longest Substring with At Most K Distinct Characters

**LeetCode #340 — Medium — Pattern: Sliding Window + HashMap**

**Statement:** given a string `s` and integer `k`, return the length of the longest substring containing at most `k` distinct characters.

**Brute force:** for every substring, count distinct characters. O(n²) or O(n³).

**Optimal — the direct generalization of yesterday's Fruit Into Baskets:**

```java
public int lengthOfLongestSubstringKDistinct(String s, int k) {
    if (k == 0) return 0;
    Map<Character, Integer> window = new HashMap<>();
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        window.merge(s.charAt(right), 1, Integer::sum);
        while (window.size() > k) {
            char leftChar = s.charAt(left);
            window.put(leftChar, window.get(leftChar) - 1);
            if (window.get(leftChar) == 0) window.remove(leftChar);
            left++;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Why it works:** this *is* yesterday's LC 904, with `2` replaced by `k`. Nothing else changes — same shrink-until-valid shape, same "remove the key once its count hits zero" bookkeeping.

**Trace:** `s = "eceba"`, `k = 2`. Window grows to `{e:1,c:1}` (best=2), then `{e:2,c:1}` at `right=2` (best=3, "ece"). At `right=3` (`'b'`), size hits 3 → shrink out `'c'` → `{e:1,b:1}` (best stays 3). At `right=4` (`'a'`), size hits 3 again → shrink out `'e'` → `{b:1,a:1}` (best stays 3). Final answer: **3**.

**Complexity:** Time O(n), Space O(k) — the window holds at most `k+1` distinct keys at any instant (the extra one being what triggers the shrink).

**Edge cases & mistakes:**
- ⚠️ `k = 0`: no substring can have "at most 0" distinct characters except the empty one — handle explicitly, since the shrink-while-`size()>0` loop would otherwise empty the window entirely and still (incorrectly) try to proceed.
- ⚠️ `k >= s.length()`: the whole string is trivially valid; the loop needs no special case here — verify rather than assume.

**💡 Interview framing:** open by naming the generalization directly: "this is at-most-K-distinct — I solved the K=2 special case yesterday as Fruit Into Baskets, so I'd usually write this as one reusable helper and call it with different K values." That sentence also sets up tomorrow's Subarrays with K Different Integers, which reuses this exact helper.

---

## Problem 12: Minimum Window Substring

**LeetCode #76 — Hard — Pattern: Sliding Window (variable, shrink-while-valid)**

**Statement:** given strings `s` and `t`, return the smallest substring of `s` that contains every character of `t` (including duplicates — if `t` has two `'A'`s, the window needs two).

**Brute force:** for every `(start, end)` pair, check whether the substring covers `t`. O(n² · |t|) or worse.

**Optimal — track "how many distinct required characters are currently satisfied," not just "is each character present":**

```java
public String minWindow(String s, String t) {
    if (s.length() < t.length()) return "";

    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);

    int required = need.size();   // distinct characters t needs
    int formed = 0;               // distinct characters currently fully satisfied
    Map<Character, Integer> window = new HashMap<>();

    int left = 0, bestLen = Integer.MAX_VALUE, bestStart = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        window.merge(c, 1, Integer::sum);
        if (need.containsKey(c) && window.get(c).intValue() == need.get(c).intValue()) {
            formed++;
        }

        while (formed == required) {
            if (right - left + 1 < bestLen) {
                bestLen = right - left + 1;
                bestStart = left;
            }
            char leftChar = s.charAt(left);
            window.put(leftChar, window.get(leftChar) - 1);
            if (need.containsKey(leftChar) && window.get(leftChar) < need.get(leftChar)) {
                formed--;
            }
            left++;
        }
    }

    return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestStart, bestStart + bestLen);
}
```

**Why `formed` increments/decrements exactly when it does — this is worth being precise about, since it's the easiest part of this problem to get subtly wrong:** `formed` counts distinct characters whose window count has reached *exactly* their required count — not "every time a needed character is seen." Adding a character that's already satisfied (e.g., a third `'A'` when only two are required) must **not** increment `formed` again — the `== `comparison (not `>=`) is what prevents double-counting. Symmetrically, `formed` only decrements when removing a character drops its count *below* the requirement — going from "exactly enough" to "one short," not from "more than enough" to "still enough."

**Trace (condensed — the key moments):** `s = "ADOBECODEBANC"`, `t = "ABC"` → `need = {A:1,B:1,C:1}`, `required = 3`.

`formed` first reaches `3` at `right=5` (`s[0..5] = "ADOBEC"`), triggering the first shrink pass: it records `bestLen=6` at `[0,5]`, then shrinks left until removing `'A'` (at `left=0`) drops `formed` back to 2. The window re-expands, and `formed` reaches `3` again at `right=10`; shrinking this time only gets as far as `left=6` before the count is exhausted (no improvement over 6). A third pass, triggered at `right=12`, shrinks all the way to `left=9`, finding progressively smaller valid windows — length 5, then length 4 — before `formed` finally drops. That final pass is where the true minimum is found: **`bestLen=4`, `bestStart=9`**, giving `s.substring(9,13) = "BANC"`.

**Complexity:** Time O(n + m) — `n = s.length()`, `m = t.length()` for building `need`; the main scan is O(n) by the total-movement argument, with O(1) HashMap operations per step (amortized). Space O(n + m) — `need` is bounded by `t`'s distinct characters, `window` by `s`'s.

**Edge cases & mistakes:**
- ⚠️ Using `>=` instead of `==` when incrementing `formed` — silently double-counts a character that already satisfied its requirement, which can make the algorithm think a window is "formed" too early.
- ⚠️ `s.length() < t.length()`: no valid window can exist — guard before the loop rather than letting the main loop run and return an empty result the slow way.
- ⚠️ `t` has duplicate characters (e.g., `t = "AAB"`): `need` correctly captures `{A:2, B:1}` via `merge`, and the window must match those *counts*, not just presence — a window with only one `'A'` does not satisfy this `t`, and the algorithm handles this correctly precisely because `need`/`window` store counts, not booleans.

**💡 Interview framing:** "I'll track two integers — `required`, how many distinct characters `t` needs, and `formed`, how many are currently exactly satisfied — instead of comparing full frequency maps on every step; that turns an O(|t|)-per-step check into O(1)." This is the single detail that most separates a working-but-slow solution from the expected one here.

---

## Project Block Guide (1 hr)

No new coding task — write the comparison note the plan calls for. Here's the shape it should take, and the actual answer:

> **Minimum Size Subarray Sum (Day 17) vs. Minimum Window Substring (today): what's actually the same, and what's actually different.**
>
> Both use the exact same loop shape — shrink-*while*-valid, recording the answer inside the shrink loop, stopping the shrink only once it would break validity. That shape is identical, and it's identical for a reason: both problems ask for the *minimum* window satisfying a condition, and shrink-while-valid is the general template for "minimum window" questions the way shrink-until-valid is the template for "maximum window" questions.
>
> What's different is entirely the **validity check itself**. LC 209's check is a numeric threshold: `runningSum >= target`, a single comparison against a single accumulated number. LC 76's check is a coverage condition over multiple independent requirements at once: `formed == required`, where `formed` itself depends on tracking each distinct required character's count separately. The *complexity* of a sliding-window problem often lives entirely in the validity check, not the loop shape around it — recognizing "I've seen this loop shape before" gets you the skeleton for free; the actual work is figuring out what "valid" means for the specific problem in front of you.

Save this alongside your solutions in `dsa-java/sliding-window/`.

---

## Career Block Guide (1 hr)

- **LinkedIn engagement (20 min):** 3–5 posts, substantive comments.
- **Networking — 1 peer, about their interview experience:** ask specific questions (which round they found hardest, what they wish they'd drilled more) rather than an open-ended "how'd it go" — specific questions get specific, useful answers.

---

## Day 19 — Interview Questions

**Q1. How does Longest Substring with At Most K Distinct Characters relate to yesterday's Fruit Into Baskets?**
A: It's the exact same algorithm with `K` as a parameter instead of a hardcoded `2` — Fruit Into Baskets is the `K=2` special case.

**Q2. In Minimum Window Substring, why does `formed` use `==` rather than `>=` when checking whether to increment?**
A: `>=` would re-increment `formed` every time an already-satisfied character is seen again (e.g., a third `'A'` when only two were required), double-counting a requirement that was already met and corrupting the `formed == required` check.

**Q3. What's identical, and what's different, between Minimum Size Subarray Sum and Minimum Window Substring?**
A: Identical: the shrink-while-valid loop shape, since both are "minimum window" problems. Different: the validity check — a single numeric sum threshold versus a multi-character coverage count built from `required`/`formed`.

**Q4. Why is Minimum Window Substring's space complexity O(n + m) and not O(1)?**
A: Unlike the fixed-26-letter frequency arrays used earlier this week, `need` and `window` here are general character maps sized by the actual distinct characters present in `t` and `s` respectively — not bounded by a small constant alphabet in the general case (e.g., if the input included Unicode characters beyond a small fixed set).

**Q5. What would you say out loud, before coding, to signal you recognize Minimum Window Substring's pattern immediately?**
A: "This is shrink-while-valid, like a minimum-window sum problem, but validity here means full character coverage — I'll track that with a required/formed counter pair instead of comparing frequency maps directly each step, to keep each check O(1)."

---

## Daily Deliverable Check

- [ ] Longest Substring with At Most K Distinct Characters (LC 340) and Minimum Window Substring (LC 76) solved, pushed.
- [ ] Comparison note on the two "shrink while valid" problems written and saved.

---

### What Tomorrow Assumes You Already Know Cold

Day 20 introduces a genuinely new mechanic — the monotonic deque — for Sliding Window Maximum, which does **not** reduce to any shrink-until/while-valid shape you've used so far; it's presented fresh. Subarrays with K Different Integers, the day's other problem, directly reuses today's `atMostKDistinct` helper twice (once for K, once for K−1) — that reuse is assumed, not re-explained.
