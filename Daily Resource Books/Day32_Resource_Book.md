# Day 32 — Binary Search: 2D Matrices, and Binary Search on the Answer

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 31 Resource Book](Day31_Resource_Book.md)
**Next ▶:** [Day 33 Resource Book](Day33_Resource_Book.md)
**Companion to:** Day 32 of `Week_05_Revised.md`

---

## Recap

Every binary search this week so far has searched over **array indices** — a rotated array, a boundary within a sorted array, a peak. Today's second problem changes what's being searched entirely: Koko Eating Bananas searches over a **range of possible answer values** (eating speeds), using a feasibility check instead of an array comparison. This isn't actually new — Day 28's First Bad Version (LC 278) was already exactly this shape, just with a boolean condition instead of a numeric one. Today makes that connection explicit and gives it real depth, since the plan itself flags it as "the one people miss cold in interviews." First, though, one more index-based variant: extending exact-match search to two dimensions.

No theory or project block today — pure DSA, then Career.

---

## Learning Objectives

By the end of today, without notes:

1. State precisely which structural guarantee about a 2D matrix makes "treat it as one flattened sorted array" valid, and translate a 1D binary-search index into a 2D cell.
2. Recognize a matrix that looks similar but lacks that guarantee, and name why the 1D approach breaks for it.
3. Explain "binary search on the answer" as a direct generalization of Day 28's boundary search — a monotonic feasibility check over a range of *candidate answers*, not array indices — and say exactly what monotonicity property makes it valid here.
4. Defend, with a concrete input, why the feasibility check's running total needs `long`, not `int`.

---

## Concept Dependency Map

```
Day 28: LC 704 (exact match, on the input) vs. LC 278 (boundary search, ON THE ANSWER)
Day 10 / Day 11: int overflow, and why running sums near the constraint boundary need `long`
        │
        ├──▶ Today, Problem 9: Search a 2D Matrix (LC 74)
        │        exact-match template + index math — still "on the input,"
        │        just a 2D input read in row-major order
        │        │
        │        └──▶ Worth Knowing: LC 240 — a similar-looking matrix where
        │             the SAME technique breaks, and why
        │
        └──▶ Today, Problem 10: Koko Eating Bananas (LC 875)
                 binary search ON THE ANSWER, formally reused from Day 28 —
                 search over candidate SPEEDS, not indices, using a monotonic
                 feasibility check (continues tomorrow, Day 33)
```

---

## Problem 9: Search a 2D Matrix (LeetCode 74, Medium) — Pattern: Binary Search, Treat as 1D

**Statement:** An `m × n` matrix where every row is sorted left to right, **and** the first element of each row is greater than the last element of the previous row. Given `target`, return whether it exists. Required in O(log(m×n)).

### Approach 1 — Brute force

```java
public static boolean searchMatrixBruteForce(int[][] matrix, int target) {
    for (int[] row : matrix) {
        for (int value : row) {
            if (value == target) return true;
        }
    }
    return false;
}
```

Time O(m×n), Space O(1).

### Approach 2 — Meaningfully distinct middle ground: binary search each row

Since each row is individually sorted, binary search within a row is valid on its own — the only question is which row(s) could plausibly contain `target` (a row can be skipped entirely if `target` falls outside `[row[0], row[n-1]]`). This gets to O(m log n): still touches every row in the worst case, but each row lookup is fast. Real improvement over brute force, but doesn't yet use the *cross-row* guarantee at all.

### Approach 3 — Optimized: treat the whole matrix as one flattened sorted array

```java
public static boolean searchMatrix(int[][] matrix, int target) {
    int rows = matrix.length, cols = matrix[0].length;
    int left = 0, right = rows * cols - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int row = mid / cols;
        int col = mid % cols;
        int value = matrix[row][col];
        if (value == target) {
            return true;
        } else if (value < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return false;
}
```

**Why this is valid — name the exact guarantee, don't just apply it:** the matrix's stated property — each row sorted, *and* each row's first value exceeds the previous row's last value — means reading the matrix row by row, left to right, produces **one single, globally sorted sequence** of length `m×n`. That's not a looser analogy; it's literally the same situation as having one big sorted array, just stored in a 2D shape. `row = mid / cols` and `col = mid % cols` are the only new piece — pure index translation from "position `mid` in the virtual flattened array" to "the actual cell holding that value." Everything else is exactly Day 28's LC 704 template, unmodified.

**Worked trace:** `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, `target = 16`. `rows=3, cols=4`, so `left=0, right=11`.

| left | right | mid | row = mid/4 | col = mid%4 | value | comparison | action |
|---|---|---|---|---|---|---|---|
| 0 | 11 | 5 | 1 | 1 | 11 | 11 < 16 | `left = 6` |
| 6 | 11 | 8 | 2 | 0 | 23 | 23 > 16 | `right = 7` |
| 6 | 7 | 6 | 1 | 2 | 16 | match | return `true` |

**Complexity:** Time O(log(m×n)) — equivalently O(log m + log n), same complexity class either way. Space O(1).

**Edge cases:**
- Target smaller than `matrix[0][0]` or larger than `matrix[m-1][n-1]` — the search correctly exhausts without finding a match.
- Single row, or single column — degenerates cleanly to plain 1D binary search.
- Empty matrix — worth an explicit guard (`matrix.length == 0 || matrix[0].length == 0 → return false`) even though the problem's constraints guarantee a non-empty input; a defensive check costs nothing and signals the habit of not trusting constraints blindly.

**⚠️ Common Mistake:** trying to index `matrix[mid]` directly, forgetting `mid` is a *virtual* 1D position that needs translating into a real `(row, col)` pair first.

**💡 Interview Insight:** state the enabling property out loud before writing any code: "reading this matrix row-major gives one globally sorted sequence, because of how the rows relate to each other — so this reduces entirely to LC 704 plus one line of index math." Naming *why* it reduces, not just that it does, is what separates recognizing a template from getting lucky with one.

---

### 💡 Worth Knowing — A Similar-Looking Matrix Where This Technique Breaks (LeetCode 240)

This is exactly the kind of contrast interviewers like to probe with as a follow-up, so it's worth having ready even though it isn't today's required problem.

**LeetCode 240 (Search a 2D Matrix II)** looks almost identical: each row sorted left to right, each **column** sorted top to bottom. The property that's *missing*, compared to today's LC 74, is the cross-row guarantee — nothing says row `i+1`'s first value exceeds row `i`'s last value. Concretely: `[[1,4,7],[2,5,8],[3,6,9]]` satisfies LC 240's rules (every row and column is individually sorted) but flattened row-major gives `[1,4,7,2,5,8,3,6,9]` — **not sorted** (`7` then `2` is a decrease). Binary search treating this as one flattened array would be meaningless.

**The correct technique instead:** start at the top-right corner. At each step, if the current cell's value is greater than `target`, move left (everything below the current cell, in that column, is even bigger — eliminate the whole column); if it's less than `target`, move down (everything to the left, in that row, is even smaller — eliminate the whole row). This is a "staircase" walk that eliminates one full row *or* column per step — O(m + n), a genuinely different complexity class from O(log(m×n)), not just a different constant. (Binary-searching each row independently, as in Approach 2 above, also still works here — each row remains individually sorted — giving O(m log n); the staircase walk is asymptotically better than that whenever `m` and `n` are comparable in size, since `O(m+n)` beats `O(m log n)` for large inputs.)

**🔑 Key Takeaway:** the two problems look nearly identical in their statements but need genuinely different techniques, because the cross-row guarantee that makes LC 74's flattening valid is exactly what LC 240 doesn't have. If asked "what if the matrix looked like *this* instead," naming precisely which guarantee disappeared — not just "a different algorithm is needed" — is the signal worth giving.

---

## Problem 10: Koko Eating Bananas (LeetCode 875, Medium) — Pattern: Binary Search on the Answer

**Statement:** `piles[i]` bananas in pile `i`. Koko eats at a constant integer speed `k` bananas/hour, one pile per hour — if a pile has fewer than `k` bananas left, she finishes it and the rest of that hour is wasted (no carryover to another pile). Return the **minimum** integer `k` such that she can finish every pile within `h` hours.

### Approach 1 — Brute force

```java
public static int minEatingSpeedBruteForce(int[] piles, int h) {
    int maxPile = 0;
    for (int pile : piles) maxPile = Math.max(maxPile, pile);
    for (int speed = 1; speed <= maxPile; speed++) {
        if (canFinish(piles, speed, h)) {
            return speed;
        }
    }
    return maxPile;
}
```

Try every possible speed from `1` upward, return the first that works. Time O(maxPile × n), where `n = piles.length` — and `maxPile` can be up to `10^9` per the problem's constraints, making this hopelessly slow in practice despite being "technically" correct.

### Approach 2 — Optimized: binary search on the answer

```java
public static int minEatingSpeed(int[] piles, int h) {
    int left = 1, right = 0;
    for (int pile : piles) {
        right = Math.max(right, pile);
    }
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canFinish(piles, mid, h)) {
            right = mid;        // mid works — a smaller speed might also work
        } else {
            left = mid + 1;     // mid too slow — need strictly more
        }
    }
    return left;
}

private static boolean canFinish(int[] piles, int speed, int h) {
    long hoursNeeded = 0;
    for (int pile : piles) {
        hoursNeeded += (pile + speed - 1) / speed;   // ceiling division, integer-only
    }
    return hoursNeeded <= h;
}
```

**🔑 Key Takeaway — this is not a new pattern, it's Day 28's LC 278 with numbers instead of a boolean flip:** First Bad Version searched a range of **version numbers** using a monotonic condition (`isBadVersion`: false, false, ..., false, true, true, ..., true) and converged on the flip point. Here the range is **eating speeds** (`1` to `max(piles)`), and the condition (`canFinish`) is monotonic the same way: if speed `k` finishes in time, every faster speed also finishes in time (going faster never hurts), so the feasible speeds form exactly the same false-false-...-false-true-true-...-true shape, and the same convergence logic applies. The array being searched isn't sorted — there *is* no array — but that was never actually the requirement; the requirement is a monotonic condition over an ordered range, which speed provides just as well as a sorted array index does.

**Say this out loud before coding, unprompted:** *"The array itself isn't sorted, and I'm not searching indices at all — I'm searching over the space of possible eating speeds, and binary search applies because 'can Koko finish in time' is monotonic in speed."* This single sentence is the highest-value thing to say in this entire binary-search unit — the plan itself calls this exact framing "the one people miss cold in interviews."

**Why `right = mid` (not `mid - 1`) when `mid` is feasible:** identical logic to Day 31's Find Minimum — `mid` being feasible doesn't rule out `mid` itself as the true minimum answer, so it stays in range rather than being excluded.

**Why ceiling division, and why it's computed as `(pile + speed - 1) / speed`:** a pile of `11` bananas at speed `4` takes `4, 4, 3` — three hours, not `11/4 = 2` (which truncates). `ceil(11/4) = 3`. Adding `speed - 1` before integer-dividing pushes any nonzero remainder over the next integer boundary using only integer arithmetic — no floating-point division or rounding function needed.

**⚠️ Why `hoursNeeded` is declared `long`, not `int` — direct citation, not a new lesson:** per the problem's own constraints, a single pile can hold up to `10^9` bananas, and there can be up to `10^4` piles. Summing ceiling-divided hours across worst-case inputs can approach values that risk `int` overflow — exactly Day 10's silent-wraparound danger, applied the same way Day 11's 4Sum needed `long` for its running sums. This is a defensive habit, not a hypothetical: `int` overflow doesn't throw in Java, it silently wraps, so a missed `long` here wouldn't crash — it would return a wrong answer with no error at all.

**Worked trace:** `piles = [3, 6, 7, 11]`, `h = 8`. `left=1, right=11` (max pile).

| left | right | mid | hours: ⌈3/mid⌉+⌈6/mid⌉+⌈7/mid⌉+⌈11/mid⌉ | total | ≤ 8? | action |
|---|---|---|---|---|---|---|
| 1 | 11 | 6 | 1+1+2+2 | 6 | yes | `right = 6` |
| 1 | 6 | 3 | 1+2+3+4 | 10 | no | `left = 4` |
| 4 | 6 | 5 | 1+2+2+3 | 8 | yes | `right = 5` |
| 4 | 5 | 4 | 1+2+2+3 | 8 | yes | `right = 4` |

`left == right == 4`, loop ends → returns `4`. (Matches the well-known expected answer for this exact input.)

**Complexity:** Let `n = piles.length`, `m = max(piles)`. Time O(n log m) — O(log m) iterations of the outer binary search, each doing an O(n) feasibility check. Space O(1) beyond the input.

**Edge cases:**
- `h == piles.length` exactly — the tightest possible constraint; forces `k = max(piles)`, since with exactly one hour per pile available, Koko needs to clear even the biggest pile in a single hour.
- `h` very large relative to the piles — `k` can converge to a small value, even `1`.
- All piles equal — every candidate speed produces the same per-pile hour count, no asymmetry to reason about.
- Single pile — reduces to `k = ceil(pile / h)`, and the binary search still finds it correctly, just via more machinery than strictly needed.

**⚠️ Common Mistake:** using plain integer division (`pile / speed`) instead of ceiling division — silently *undercounts* the hours needed whenever a pile doesn't divide evenly by the speed, which can make an infeasible speed look feasible.

**⚠️ Common Mistake:** initializing `right` to `sum(piles)` instead of `max(piles)`. It isn't *wrong* (the true answer is still found, since it's still somewhere in that wider range) — but it's a looser bound than necessary, since a speed of `max(piles)` already guarantees finishing in exactly `n` hours (one pile per hour, worst case), so no speed larger than that could ever be the *minimum* answer. Using the tight bound isn't just neater; it directly reduces the number of iterations, which is worth naming as the reason, not just doing it out of habit.

---

## Career Block Guide (1 hr)

**LinkedIn (20 min):** comment meaningfully on 3–5 posts.

**Networking:** reach out to one peer about their interview experience — a real conversation, not a generic ask; something specific to react to (their role, their company, a post they made about interviewing) gets a real reply far more often than "how was your interview process?" cold.

---

## Day 32 — Interview Questions

**Q1. What exact structural guarantee makes it valid to treat LC 74's matrix as one flattened sorted array?** Each row is individually sorted, and each row's first value exceeds the previous row's last value — together those two facts mean reading the matrix row by row, left to right, produces one single globally sorted sequence.

**Q2. Given a virtual 1D index `mid` into an `m × n` matrix, how do you recover the real cell?** `row = mid / cols`, `col = mid % cols` — integer division and remainder translate a position in the row-major flattened reading into the actual 2D coordinates.

**Q3. LC 240 looks almost identical to LC 74 but needs a completely different technique. What guarantee is missing?** The cross-row guarantee — LC 240 only promises each row and each column is individually sorted, not that one row's values are entirely above the previous row's. Without that, the row-major flattening isn't globally sorted, so binary-search-as-1D no longer applies.

**Q4. What's the correct technique for LC 240, and what's its complexity?** Start at the top-right corner; move left when the current value is too big (eliminating a whole column), move down when it's too small (eliminating a whole row). O(m+n) — a different complexity class from LC 74's O(log(mn)), not just a different constant.

**Q5. What does "binary search on the answer" mean, and how does Koko Eating Bananas qualify?** Searching over a range of candidate *answer values* (not array indices) using a monotonic feasibility check at each candidate. Koko qualifies because "can she finish within `h` hours at speed `k`" is monotonic in `k` — any speed faster than a working speed also works.

**Q6. Which earlier problem this series already used this exact shape, and what changed?** Day 28's First Bad Version (LC 278) — same shape, a monotonic boolean condition binary searched over a range. Only the range changed (version numbers → eating speeds) and the condition became a numeric feasibility check instead of a single boolean flag.

**Q7. Why is `hoursNeeded` declared as `long` rather than `int`?** With piles up to `10^9` and up to `10^4` piles, the summed hour count across a worst-case feasibility check can approach `int`'s overflow boundary; since Java's `int` overflow wraps silently rather than throwing, an undetected overflow here would produce a wrong answer with no visible error.

**Q8. Why does the algorithm set `right = mid` (not `mid - 1`) when `mid` turns out to be feasible?** Because `mid` being feasible doesn't rule it out as the *minimum* feasible speed — it needs to stay inside the search range in case nothing smaller also works, exactly mirroring Day 31's Find Minimum logic.

**Q9. Why is `max(piles)`, not `sum(piles)`, the correct upper bound for the search range?** A speed equal to `max(piles)` already guarantees finishing in exactly `n` hours (one full pile per hour, worst case) — no speed larger than that could ever be a smaller, more minimal answer, so the true minimum is always within `[1, max(piles)]`.

---

## Daily Deliverable Check

- [ ] Search a 2D Matrix and Koko Eating Bananas solved, pushed.
- [ ] Can state, from memory, the specific guarantee that makes LC 74's flattening valid — and why LC 240's superficially similar matrix doesn't share it.

---

## What Tomorrow Assumes You Already Know Cold

Day 33 assumes "binary search on the answer" — the monotonic feasibility check framing, not just today's specific `canFinish` function — is solid enough to apply to a brand new feasibility check without re-deriving the *idea* from scratch, since tomorrow's Capacity to Ship Packages problem (and its own extra practice) reuses this exact skeleton with a different simulation inside the check. It also assumes today's `long`-for-accumulated-totals habit carries forward automatically, since tomorrow's feasibility checks face the same overflow risk.
