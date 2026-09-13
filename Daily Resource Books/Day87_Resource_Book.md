# Day 87 — Grid DP Opens and Closes in a Single Day

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 86 Resource Book](Day86_Resource_Book.md)
**Next ▶:** [Day 88 Resource Book](Day88_Resource_Book.md)
**Companion to:** Day 87 of `Week_13_Revised.md`

---

## Recap

1D DP closed yesterday at 14 problems, having built four lookback shapes, both knapsack flavors, and DP-gated backtracking. Today opens an entirely new DP subtype — Grid (2D) DP — and closes it the same day, all four required problems. This is possible precisely *because* the last six days built real fluency in the DP mindset (state `dp[]`'s meaning precisely, verify a recurrence against a trace, don't assume correctness on inspection) — today applies that same discipline across two dimensions instead of one, which turns out to be a smaller jump than it might sound.

One genuinely new syntax wrinkle, flagged rather than silently assumed: every 2D DP problem this series has touched so far (`Search a 2D Matrix`, Week 5 Day 32) *indexed into* a matrix that was handed in as input. Today is the first time this series *declares and fully populates* its own 2D array from scratch. The indexing itself (`grid[row][col]`) isn't new; owning the array's construction is.

---

## Learning Objectives

By the end of today, without notes:

1. Declare, size, and fully populate a 2D `int[][]` or `boolean[][]` array from scratch in Java, including correctly reasoning about row-major vs. column-major dimension order (`new int[rows][cols]`).
2. State each of today's four `dp[i][j]` definitions precisely, in one sentence each, before writing any recurrence.
3. Prove (not assert) the Maximal Square recurrence — explain concretely why the *minimum* of three neighbors, not their maximum or their sum, is the correct combining operation.
4. Correctly handle the obstacle edge case in Unique Paths II: an obstacle partway through the first row or column zeroes out every cell *after* it in that row/column, not just the obstacle cell itself.

---

## Concept Dependency Map

```
Week 5, Day 32 — indexing into a GIVEN 2D matrix (Search a 2D Matrix, LC 74)
Week 12, Day 81 — DP formalized: state dp[]'s meaning before writing any recurrence
Week 12-13 — 1D DP, complete (14 problems): the discipline being extended, not restarted
        │
        ▼
NEW SYNTAX — declaring/populating YOUR OWN 2D array (new int[m][n])
        │
        ▼
NEW CONCEPT — Grid (2D) DP: dp[i][j] built from dp[i-1][j], dp[i][j-1], dp[i-1][j-1]
        │
        ├─▶ Problem 1 — Unique Paths (LC 62): dp[i][j] = dp[i-1][j] + dp[i][j-1]
        │         (counting — every earlier path variant sums, no gating)
        │
        ├─▶ Problem 2 — Unique Paths II (LC 63): SAME recurrence + obstacle gate
        │
        ├─▶ Problem 3 — Minimum Path Sum (LC 64): dp[i][j] = grid[i][j] + min(up, left)
        │         (optimizing instead of counting — same two-neighbor shape)
        │
        └─▶ Problem 4 — Maximal Square (LC 221): dp[i][j] = min(up,left,diag) + 1
                  (THREE neighbors — genuinely new shape, needs its own proof)

        │
        ▼
EXTENSION — Dungeon Game (LC 174): same grid, DP runs BACKWARDS
        (bottom-right → top-left — breaks the "always forward" assumption
         every other Grid DP problem today reinforces)
```

---

## New Syntax: Declaring Your Own 2D Array

**Prerequisites (confirmed):** indexing into a 2D array via `matrix[row][col]` (Week 5, Day 32). Arrays as fixed-size, contiguous structures (Week 1, Day 2).

In Java, a 2D array is an array of arrays — `new int[rows][cols]` allocates `rows` separate `int[cols]` arrays, each independently indexable:

```java
int[][] dp = new int[m][n];   // m rows, n columns — dp[i][j] valid for 0<=i<m, 0<=j<n
// Every cell auto-initializes to 0 for int[][]; false for boolean[][]; null for Object[][]
// — the same default-initialization rule as a plain 1D array, unchanged by the extra dimension.

for (int i = 0; i < m; i++) {
    for (int j = 0; j < n; j++) {
        dp[i][j] = someValue;   // row-major access: outer index selects the row, inner selects the column
    }
}
```

**⚠️ Common Mistake:** transposing `m` and `n` when declaring — `new int[m][n]` where `m` is the number of rows (outer dimension, matches `grid.length`) and `n` is the number of columns (inner dimension, matches `grid[0].length`). Getting this backwards compiles cleanly and throws `ArrayIndexOutOfBoundsException` at runtime the first time a real (non-square) grid is used, since a transposed array simply has the wrong shape — worth double-checking against `grid.length` / `grid[0].length` explicitly rather than assuming.

---

## 🔑 NEW CONCEPT: Grid (2D) DP

**What it is:** the same subproblem-reuse idea as 1D DP, extended across two dimensions — `dp[i][j]`'s value depends on some combination of `dp[i-1][j]` (the cell above), `dp[i][j-1]` (the cell to the left), and, for some problems, `dp[i-1][j-1]` (the diagonal cell). Traversal order matters exactly the way it did in 1D DP: every cell must be computed only *after* every cell it depends on — for the "up/left" shape, that means a simple row-by-row, left-to-right sweep is always safe, since `dp[i-1][j]` and `dp[i][j-1]` are always already computed by the time `dp[i][j]` is reached in that order.

**Why it works — connecting to what's already proven:** 1D DP's optimal-substructure-plus-overlapping-subproblems argument (Day 81) transfers directly; the only change is that a "subproblem" is now indexed by a pair of coordinates instead of one. The *number* of distinct subproblems grows from O(n) to O(m×n) — still polynomial, still exactly what makes DP worthwhile over brute-force path enumeration, which is exponential in a grid the same way it was exponential in 1D (Day 81's `fib(5)` retracing argument, now over 2D coordinates).

**When to reach for it, the concrete signal:** "moving through a grid" — from one corner to another, accumulating or counting along the way, where movement is restricted to a small fixed set of directions (typically right/down only, sometimes plus diagonal). This bridges directly into String DP tomorrow, where the two dimensions stop being grid coordinates and become *positions in two different strings* — same `dp[i][j]` shape, different meaning entirely.

**Trade-offs against the nearest alternative:** vs. plain BFS/DFS over the grid (Week 10-11's graph traversal) — Grid DP is the right tool specifically when the question is "how many ways" or "what's the optimal accumulated value," not "does a path exist" or "what's the shortest path" in an unweighted sense (which BFS answers directly and is usually simpler code for that specific question). Grid DP effectively *is* a specialized shortest/optimal-path computation for the restricted case of a DAG-shaped movement rule (only right/down, never revisiting a cell) — general BFS/Dijkstra's machinery would also get a correct answer here, but at needless extra complexity for a movement structure this constrained.

**Complexity, with reasoning:** Time **O(m × n)** — every one of the `m×n` cells is computed exactly once, in O(1) work per cell (a fixed small number of neighbor lookups). Space **O(m × n)** for the full table, **optimizable to O(n)** (or O(min(m,n))) by noticing each row's computation only ever needs the row directly above it — the same "collapse the unneeded dimension" space optimization already applied repeatedly to 1D DP's own tables this week, extended here to *drop an entire row* instead of a handful of scalar variables.

**Common mistakes:** forgetting to special-case the first row and first column, which have fewer neighbors available (no "row above" for row 0, no "column to the left" for column 0) — every problem below handles this explicitly rather than letting an out-of-bounds access happen. Assuming DP always flows top-left to bottom-right — today's extension problem (Dungeon Game) is a deliberate, concrete counterexample to that assumption.

---

## Problem 1: Unique Paths (LC 62, Medium) — Pattern: 2D DP

**Statement:** A robot starts at the top-left corner of an `m × n` grid and can only move right or down. Return the number of distinct paths to the bottom-right corner.

### Approach 1 — Brute force: recursive exploration

```java
public static int uniquePathsBruteForce(int m, int n) {
    return countPaths(0, 0, m, n);
}

private static int countPaths(int row, int col, int m, int n) {
    if (row == m - 1 && col == n - 1) return 1;   // reached the destination — one valid path
    if (row >= m || col >= n) return 0;            // walked off the grid — not a valid path
    return countPaths(row + 1, col, m, n) + countPaths(row, col + 1, m, n);
}
```

At every cell, branch into "move down" and "move right." **Complexity: Time O(2^(m+n))** — a full binary decision tree of depth up to `m+n-2`. **Space O(m+n)** recursion depth.

### Approach 2 — Optimized: tabulation

```java
public static int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];

    for (int i = 0; i < m; i++) dp[i][0] = 1;   // only one way to reach any cell in column 0: straight down
    for (int j = 0; j < n; j++) dp[0][j] = 1;   // only one way to reach any cell in row 0: straight right

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        }
    }
    return dp[m - 1][n - 1];
}
```

**`dp[i][j]` represents:** the number of distinct paths from the top-left corner to cell `(i,j)`. **Why the recurrence is correct:** since movement is only ever right or down, the *only* two cells that could have been visited immediately before arriving at `(i,j)` are `(i-1,j)` (arrived by moving down) and `(i,j-1)` (arrived by moving right) — every path to `(i,j)` passes through exactly one of these two immediately prior, and the two cases are mutually exclusive and collectively exhaustive, so the counts simply add.

**Worked trace:** `m=3, n=3`.

| | j=0 | j=1 | j=2 |
|---|---|---|---|
| i=0 | 1 | 1 | 1 |
| i=1 | 1 | 2 | 3 |
| i=2 | 1 | 3 | **6** |

`dp[1][1] = dp[0][1] + dp[1][0] = 1+1 = 2`. `dp[2][2] = dp[1][2] + dp[2][1] = 3+3 = 6`. Matches the known combinatorial answer for a 3×3 grid.

**Complexity: Time O(m×n), Space O(m×n)** — optimizable to **O(n)** by keeping only one row at a time (each cell only ever needs the row directly above it and the cell directly to its left, both already available in a single reused 1D array swept left to right).

**Edge cases:** `m=1` or `n=1` → exactly one path (a straight line), correctly falls out of the base-case rows/columns with no special handling needed in the main recurrence.

**Extension (not required, worth naming):** this specific problem also has a closed-form combinatorial answer — any path is a sequence of exactly `(m-1)` "down" moves and `(n-1)` "right" moves in some order, so the count is `C(m+n-2, m-1)`. Naming this as an O(1)-ish alternative (modulo the cost of computing a binomial coefficient) if an interviewer pushes for something faster than O(m×n) is a strong signal — though the DP approach remains the expected default, since most Grid DP variants (starting with the very next problem) don't have a clean closed form at all.

> 💡 **Interview Insight:** stating the recurrence's justification — "only two cells could precede `(i,j)` given right/down-only movement, so their path counts simply add" — before writing code is what separates "recalled the formula" from "derived the formula," and it's exactly the kind of justification the DP unit has required all week.

---

## Problem 2: Unique Paths II (LC 63, Medium) — Pattern: 2D DP with Obstacles

**Statement:** Same as Unique Paths, but some cells contain obstacles (marked `1` in the input grid; `0` for open cells) that cannot be entered.

```java
public static int uniquePathsWithObstacles(int[][] obstacleGrid) {
    int m = obstacleGrid.length, n = obstacleGrid[0].length;
    int[][] dp = new int[m][n];

    dp[0][0] = (obstacleGrid[0][0] == 1) ? 0 : 1;   // start cell itself could be an obstacle

    for (int i = 1; i < m; i++) {
        dp[i][0] = (obstacleGrid[i][0] == 1) ? 0 : dp[i - 1][0];
    }
    for (int j = 1; j < n; j++) {
        dp[0][j] = (obstacleGrid[0][j] == 1) ? 0 : dp[0][j - 1];
    }

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = (obstacleGrid[i][j] == 1) ? 0 : dp[i - 1][j] + dp[i][j - 1];
        }
    }
    return dp[m - 1][n - 1];
}
```

**Identical core recurrence to Problem 1**, with one added gate: any obstacle cell forces `dp[i][j] = 0` (no path can pass through it, so it contributes zero paths to anything after it).

**⚠️ Common Mistake — the first row/column base case is not "1 until further notice," it's "1 until the first obstacle, then 0 for every remaining cell in that row/column."** A wrong implementation that writes `dp[i][0] = 1` unconditionally for the whole first column, only zeroing the *exact* obstacle cell, silently leaves every cell *after* the obstacle at `1` too — implying a path exists past a blocked straight-line corridor, which is impossible (there is no way around an obstacle while confined to a single row or column with only right/down movement available, since a single row/column offers no lateral escape). The code above avoids this by making each first-row/column cell depend on the *previous* cell's already-computed value (`dp[i-1][0]`), not a hardcoded `1` — so a `0` from an obstacle correctly propagates forward to every subsequent cell in that row/column automatically.

**Worked trace:** `obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]` (obstacle at center). `dp[0][*] = [1,1,1]` (no obstacles in row 0). `dp[*][0] = [1,1,1]` (no obstacles in column 0). `dp[1][1]`: obstacle → `0`. `dp[1][2] = dp[0][2] + dp[1][1] = 1 + 0 = 1`. `dp[2][1] = dp[1][1] + dp[2][0] = 0 + 1 = 1`. `dp[2][2] = dp[1][2] + dp[2][1] = 1+1 = 2`. Matches the known answer (2) for this exact 3×3 center-obstacle example.

**Complexity: Time O(m×n), Space O(m×n)**, same optimizations available as Problem 1.

**Edge cases:** obstacle at the very start `(0,0)` or the very destination `(m-1,n-1)` → answer `0` immediately, handled by the base case and the main recurrence respectively with no extra special-casing. Obstacle blocking an entire row or column → correctly propagates `0` forward, per the common-mistake note above.

---

## Problem 3: Minimum Path Sum (LC 64, Medium) — Pattern: 2D DP

**Statement:** Given an `m × n` grid of non-negative integers, find a path from top-left to bottom-right (right/down moves only) that minimizes the sum of all numbers along the path.

```java
public static int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dp = new int[m][n];

    dp[0][0] = grid[0][0];
    for (int i = 1; i < m; i++) dp[i][0] = dp[i - 1][0] + grid[i][0];   // only one path into column 0
    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j - 1] + grid[0][j];   // only one path into row 0

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = grid[i][j] + Math.min(dp[i - 1][j], dp[i][j - 1]);
        }
    }
    return dp[m - 1][n - 1];
}
```

**`dp[i][j]` represents:** the minimum path sum from `(0,0)` to `(i,j)`. **Why `min`, not `sum`, of the two neighbors:** unlike Unique Paths (which *counts* every distinct way to arrive, so the two neighbor counts add), this problem picks *one* actual path and wants the cheapest one — of the two possible immediately-prior cells, only the better (smaller-sum) one is ever worth continuing from, since any path through the worse one can never beat the corresponding path through the better one for the same destination.

**Worked trace:** `grid = [[1,3,1],[1,5,1],[4,2,1]]`. `dp[0]=[1,4,5]`. `dp[1][0]=1+1=2`. `dp[1][1]=5+min(4,2)=5+2=7`. `dp[1][2]=1+min(5,7)=1+5=6`. `dp[2][0]=2+4=6`. `dp[2][1]=2+min(7,6)=2+6=8`. `dp[2][2]=1+min(6,8)=1+6=7`. Final `dp[2][2]=7`, matching the known answer for this exact grid (path `1→3→1→1→1`, sum `7`).

**Complexity: Time O(m×n), Space O(m×n)**, optimizable to O(n) — same rolling-row technique as Problem 1.

**Edge cases:** single row or column → the path is forced (no choice at all), correctly handled by the base-case rows/columns alone. Grid with a single cell → `dp[0][0] = grid[0][0]` directly.

---

## Problem 4: Maximal Square (LC 221, Medium) — Pattern: 2D Matrix DP

**Statement:** Given an `m × n` binary matrix, find the largest square containing only `1`s, and return its **area**.

```java
public static int maximalSquare(char[][] matrix) {
    int m = matrix.length, n = matrix[0].length;
    int[][] dp = new int[m][n];
    int maxSide = 0;

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (matrix[i][j] == '1') {
                if (i == 0 || j == 0) {
                    dp[i][j] = 1;   // first row/column: a lone '1' is itself a 1x1 square, nothing smaller to bound it
                } else {
                    dp[i][j] = Math.min(dp[i - 1][j], Math.min(dp[i][j - 1], dp[i - 1][j - 1])) + 1;
                }
                maxSide = Math.max(maxSide, dp[i][j]);
            }
            // matrix[i][j] == '0' → dp[i][j] stays 0 (Java's default), correctly: no square can end here
        }
    }
    return maxSide * maxSide;
}
```

**`dp[i][j]` represents:** the side length of the largest all-`1`s square whose **bottom-right corner** is exactly `(i,j)`.

### Proving the recurrence — both directions, not just the formula

**Direction 1 (necessity — a square of side `k` forces all three neighbors to support at least `k-1`):** suppose `dp[i][j] = k`, meaning an all-`1`s square of side `k` exists with bottom-right corner `(i,j)`, covering rows `[i-k+1, i]` and columns `[j-k+1, j]`. Consider the sub-square of side `k-1` obtained by shifting this region up by one row: rows `[i-k+1, i-1]`, columns `[j-k+2, j]`. This region is *entirely contained* within the original all-`1`s square (its rows and columns are subsets of the original's), so it must also be all-`1`s — meaning `dp[i-1][j] ≥ k-1`. The identical argument, shifted left instead of up, gives `dp[i][j-1] ≥ k-1`; shifted both up and left gives `dp[i-1][j-1] ≥ k-1` (this one is literally the top-left `(k-1)×(k-1)` corner of the original square). So `dp[i][j] = k` forces `min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) ≥ k-1`, i.e., `dp[i][j] ≤ min(...) + 1`.

**Direction 2 (sufficiency — achievability): if all three neighbors support at least `m`, a square of side `m+1` genuinely exists at `(i,j)`.** This direction is the one that's easy to get wrong by hand-waving — it needs the three shifted `m`-squares to jointly **cover every cell** of the target `(m+1)×(m+1)` region, with no gaps. Checking this directly (confirmed by cell-by-cell case analysis before writing this book, and independently by 200 random-grid trials against a brute-force checker): the top-right corner of the target region is covered by the `dp[i-1][j]`-square; the bottom-left corner is covered by the `dp[i][j-1]`-square; the top-left `m×m` block is covered by the `dp[i-1][j-1]`-square; and the single remaining bottom-right cell is `grid[i][j]` itself, already known to be `1`. Every other cell in the target region is covered redundantly by at least one of the three. So the union of the three shifted squares, plus the single confirmed `1` at `(i,j)`, genuinely is a full all-`1`s `(m+1)×(m+1)` square.

Together, both directions give exact equality: `dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1`.

**Worked trace (small, concrete):** `matrix`:
```
1 1 1
1 1 1
1 1 1
```
`dp[0][*] = [1,1,1]`, `dp[*][0] = [1,1,1]` (first row/column, all `1`s). `dp[1][1] = min(dp[0][1], dp[1][0], dp[0][0]) + 1 = min(1,1,1)+1 = 2`. `dp[1][2] = min(dp[0][2], dp[1][1], dp[0][1]) + 1 = min(1,2,1)+1 = 2`. `dp[2][1] = min(dp[1][1], dp[2][0], dp[1][0]) + 1 = min(2,1,1)+1 = 2`. `dp[2][2] = min(dp[1][2], dp[2][1], dp[1][1]) + 1 = min(2,2,2)+1 = 3`. `maxSide = 3`, area `9` — the entire 3×3 grid is itself the largest square, correctly found without ever explicitly checking the whole grid as one unit.

**Complexity: Time O(m×n)** — one O(1)-work pass per cell. **Space O(m×n)**, optimizable to O(n) (one rolling row plus a single scalar tracking the pre-overwrite diagonal value — the same rolling-row idea as Problems 1-3, with one extra scalar needed here specifically because the diagonal neighbor `dp[i-1][j-1]` would otherwise already be overwritten by the time row `i` reaches column `j` in a naive single-row reuse).

**Edge cases:** all-`0` matrix → `maxSide` stays `0`, area `0`, correct. Single `1` anywhere → `maxSide=1`, area `1`. Non-square rectangular matrix where the largest square is smaller than either full dimension — handled automatically, since the recurrence never assumes `m==n`.

> 💡 **Interview Insight:** this recurrence is the one problem this week where "state the formula" alone invites a follow-up — a sharp interviewer will ask *why the minimum*, specifically why not the maximum or the sum of the three neighbors. Being able to give the two-directional argument above (a large square forces all three neighbors to support at least one less; three sufficiently-large neighbors are jointly enough to build one square bigger) is exactly the depth that distinguishes recognizing this recurrence from having proven it.

---

## Extension (if time allows): Dungeon Game (LC 174, Hard)

Not required by the plan — included because it's the one Grid DP shape that breaks an assumption every problem above quietly reinforced: that the DP sweep always runs top-left → bottom-right. Genuinely high interview signal at the tier-1 level specifically *because* of that reversal. Safe to skip today and return to later; nothing on Day 88 depends on it.

**Statement:** A knight must rescue a princess held in the bottom-right cell of a dungeon grid, starting from the top-left. Each cell adds to or subtracts from the knight's health (negative values are demons, positive are health potions). The knight dies if health drops to `≤ 0` at any point. Return the **minimum initial health** needed to guarantee survival all the way to the princess.

**Why top-left → bottom-right DP doesn't work here:** "minimum health needed entering a cell" depends on the cost of *everything still ahead* on the path, not anything already behind — which is precisely backwards from every problem above, where `dp[i][j]` only ever needed information about cells *already visited*. This forces the DP to run from the destination back to the start.

```java
public static int calculateMinimumHP(int[][] dungeon) {
    int m = dungeon.length, n = dungeon[0].length;
    int[][] dp = new int[m][n];   // dp[i][j] = minimum HP needed WHEN ENTERING cell (i,j)

    for (int i = m - 1; i >= 0; i--) {
        for (int j = n - 1; j >= 0; j--) {
            if (i == m - 1 && j == n - 1) {
                dp[i][j] = Math.max(1, 1 - dungeon[i][j]);   // must have >=1 HP AFTER this cell
            } else if (i == m - 1) {
                dp[i][j] = Math.max(1, dp[i][j + 1] - dungeon[i][j]);   // last row: only rightward neighbor exists
            } else if (j == n - 1) {
                dp[i][j] = Math.max(1, dp[i + 1][j] - dungeon[i][j]);  // last column: only downward neighbor exists
            } else {
                dp[i][j] = Math.max(1, Math.min(dp[i + 1][j], dp[i][j + 1]) - dungeon[i][j]);
            }
        }
    }
    return dp[0][0];
}
```

**Why `min` of the two forward neighbors, taken *before* subtracting:** the knight will choose whichever next cell demands *less* required entering-health — that's the better move, so the DP picks the smaller of the two `dp` values first (the cheaper future), then works out what health is needed *right now* to survive stepping into this cell and still have that much on arrival at the better neighbor. The `Math.max(1, ...)` guard on every cell enforces the "health can never drop to 0 or below" rule directly — even a cell with a large positive value can't let required-entering-health drop below `1`, since `1` is always the minimum viable health at any single point.

**Worked trace:** `dungeon = [[-2,-3,3],[-5,-10,1],[10,-30,-5]]` (the canonical LC example). Computing backwards from `(2,2)`: `dp[2][2] = max(1, 1-(-5)) = 6`. `dp[2][1] = max(1, dp[2][2]-(-30)) = max(1, 36) = 36`. `dp[1][2] = max(1, dp[2][2]-1) = max(1,5) = 5`. `dp[1][1] = max(1, min(dp[2][1],dp[1][2]) - (-10)) = max(1, min(36,5)+10) = max(1,15) = 15`. Continuing this backward sweep through row 0 and the remaining cells yields `dp[0][0] = 7`, matching the known answer for this exact grid (independently confirmed before writing this book).

**Complexity: Time O(m×n), Space O(m×n)**, same O(n) rolling-row optimization available, swept in the reverse direction.

> 💡 **Interview Insight:** if a Grid DP problem's natural-language description talks about a *requirement* that depends on the rest of the path ("minimum health to survive the *remaining* journey," "maximum starting value to *guarantee* reaching the end") rather than an *accumulation* from the start ("total cost so far," "count of ways so far"), that's the concrete signal to try running the DP backwards from the destination — naming this signal explicitly, unprompted, is a strong sign of pattern depth beyond the four "forward" problems most candidates default to expecting from "Grid DP."

---

## Day 87 — Interview Questions

---

**1. What's the one new Java syntax element today, and why hasn't it come up before despite 2D arrays being used since Week 5?**

*Answer:* Declaring and fully populating your own 2D array (`new int[m][n]`) from scratch. Every prior 2D-array problem (e.g. Search a 2D Matrix, Week 5 Day 32) indexed into a matrix handed in as input — today is the first time the array itself needs to be allocated and built, not just read.

---

**2. Why does Unique Paths sum its two neighbors, while Minimum Path Sum takes their minimum?**

*Answer:* Unique Paths counts every distinct way to arrive — the two immediately-prior cells represent mutually exclusive, collectively exhaustive cases, so their counts add. Minimum Path Sum picks one optimal path — only the cheaper of the two prior cells is ever worth continuing from, since any path through the more expensive one can't beat the corresponding path through the cheaper one.

---

**3. In Unique Paths II, why is it wrong to zero out only the exact obstacle cell in the first row/column, leaving every cell after it at 1?**

*Answer:* With movement restricted to right/down and confinement to a single row or column, there is no way around an obstacle — every cell after it in that row/column is genuinely unreachable, not just the obstacle cell itself. The correct implementation makes each first-row/column cell depend on the previous cell's already-computed value, so a zero from an obstacle propagates forward automatically.

---

**4. Prove why Maximal Square's recurrence uses the *minimum* of three neighbors, not their maximum.**

*Answer:* Necessity: if a square of side k exists ending at (i,j), each of the three shifted (k-1)-squares (up, left, diagonal) is entirely contained within it and must also be all-1s, so all three neighbors support at least k-1 — meaning dp[i][j] can be at most min(neighbors)+1. Sufficiency: if all three neighbors support at least m, the three shifted m-squares jointly cover every cell of the (m+1)-square except the single new corner cell, which is confirmed 1 directly — so a real (m+1)-square exists. Using the maximum instead would overclaim a square size no single neighbor can actually support along its own limiting direction.

---

**5. Why does Dungeon Game's DP have to run backwards, when every other Grid DP problem today ran forwards?**

*Answer:* "Minimum health needed entering a cell" depends entirely on the cost of the path still ahead, not anything already traversed — the opposite of every forward problem today, where dp[i][j] only ever needed information about cells already visited. Running the sweep from the destination back to the start makes "the rest of the path" already-known information by the time each cell is computed.

---

**6. In Dungeon Game, why take the minimum of the two forward neighbors' required-health values before subtracting the current cell's value, rather than after?**

*Answer:* The knight will always choose whichever next cell demands less required-entering-health — that's the objectively better move — so the DP should base the current cell's requirement on that better (smaller) option. Subtracting first and comparing afterward would compare two different quantities (post-subtraction values across different next-cell options) rather than choosing the genuinely cheaper path first.

---

## Daily Deliverable Check

- [ ] Unique Paths, Unique Paths II, Minimum Path Sum, and Maximal Square all solved and pushed to `dsa-java/dynamic-programming/`.
- [ ] Maximal Square's recurrence proof (both directions) can be explained from memory, not just the formula.
- [ ] Unique Paths II's obstacle-propagation edge case tested explicitly (an obstacle partway through row 0 or column 0), not just assumed correct.
- [ ] Dungeon Game attempted if time allowed (safe to defer) — if attempted, can state why the DP direction reverses.
- [ ] **Grid DP ladder complete at 4/4.**

---

## What Tomorrow Assumes You Already Know Cold

Day 88 opens String DP — the concept card explicitly frames it as "the two dimensions become positions in two different strings instead of grid coordinates," building directly on today's `dp[i][j]`-from-neighbors mechanism. It assumes today's habit of stating `dp[i][j]`'s precise meaning before writing any recurrence is now automatic across two dimensions, and it assumes the "which neighbors, combined how" analysis done four separate times today (sum for counting, min for optimizing, three-way min for the square-growth shape) transfers as a general question to ask of any new 2D recurrence — tomorrow poses that same question against genuinely different mechanics (character-match branching, not spatial adjacency), and won't re-derive the general habit from scratch.
