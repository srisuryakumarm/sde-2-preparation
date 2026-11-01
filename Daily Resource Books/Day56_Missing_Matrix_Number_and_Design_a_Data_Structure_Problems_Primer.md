# Day 56 — Missing Primer: Matrix Traversal, Number/String Implementation, In-Place Array Tricks, and "Design a Data Structure" Problems

**Do this on Day 56 (Sunday; ≈ 2 h — split it into two sittings if you like, or spread it across the Sundays of Days 56 / 63 / 70), then continue with the regular plan. Do it before the Day 105A DSA mock.** Not a "hard-problem" extension (that is Day 105's file) — these are **medium, high-frequency "just implement it" questions** that show up as the first question of a phone screen.
**Needed by:** Day 68 (*Number of Islands* and every grid DFS/BFS assume direction arrays and bounds checks), Day 87 (grid DP), the mocks on Days 105A / 113A / 123A / 132A, and any screen that opens with "rotate this matrix" or "implement `pow`".
**Why it is missing:** the plan's inventory contains 66 of the 69 free Blind 75 problems; the three absent ones are all matrix problems — **Spiral Matrix (LC 54), Rotate Image (LC 48), Set Matrix Zeroes (LC 73)**. The same audit found these other families absent by name from all 148 books: Game of Life (289), Valid Sudoku (36), Multiply Strings (43), Reverse Integer (7), String to Integer (8), Roman to Integer (13), Decode String (394), Rotate Array (189), Next Permutation (31), Largest Number (179), Insert Delete GetRandom O(1) (380), Time Based Key-Value Store (981), Design Twitter (355), Sort List (148), Reverse Nodes in k-Group (25). (`pow`, `sqrt` and `gcd` appear only as ideas in Day 105's math toolkit.) The plan's own principle is *sequencing over volume* — so this is one pass over ~20 problems, **technique first**, not 20 new days.
**Already in place:** arrays and 2D arrays (Day 2), Two Pointers (Days 6–13), binary search incl. 2D and "on the answer" (Days 28–33), stacks (Days 39–45), linked lists and merge of two sorted lists (Days 34–39), heaps (Days 54–55), `HashMap` design (Days 15, 39).

---

## 1. The matrix toolkit (five moves cover everything below)
1. **Dimensions first:** `int m = g.length, n = g[0].length;` — never assume square.
2. **Direction array + bounds check** (reused in Day 68's grids):
   `int[][] DIRS = {{0,1},{1,0},{0,-1},{-1,0}};` then `nr = r + d[0]; nc = c + d[1]; if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;`
3. **Flatten ↔ coordinates:** `idx = r * n + c` and `r = idx / n, c = idx % n` (Day 32's *Search a 2D Matrix* in one line).
4. **Shrinking boundaries** (`top, bottom, left, right`) for layer-by-layer walks.
5. **In-place transforms:** transpose + reverse; or encode two states in one cell (bits) / use the first row and column as scratch space.

## 2. Matrix problems (all compiled and tested, including 1-row, 1-column and non-square cases)
**Spiral Matrix (LC 54)** — walk the four edges, shrink the boundary after each. **The two `if` guards are the whole trick:** without them a single remaining row or column is walked twice.
```java
static List<Integer> spiralOrder(int[][] m) {
    List<Integer> out = new ArrayList<>();
    int top = 0, bottom = m.length - 1, left = 0, right = m[0].length - 1;
    while (top <= bottom && left <= right) {
        for (int c = left; c <= right; c++) out.add(m[top][c]);
        top++;
        for (int r = top; r <= bottom; r++) out.add(m[r][right]);
        right--;
        if (top <= bottom) { for (int c = right; c >= left; c--) out.add(m[bottom][c]); bottom--; }
        if (left <= right) { for (int r = bottom; r >= top; r--) out.add(m[r][left]); left++; }
    }
    return out;
}
```
O(m·n) time, O(1) extra (besides the output).

**Rotate Image (LC 48)** — 90° clockwise = **transpose, then reverse every row** (counter-clockwise: reverse rows first, then transpose). In place, O(n²).
```java
static void rotateClockwise(int[][] m) {
    int n = m.length;
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++) { int t = m[i][j]; m[i][j] = m[j][i]; m[j][i] = t; }   // transpose
    for (int[] row : m)
        for (int l = 0, r = n - 1; l < r; l++, r--) { int t = row[l]; row[l] = row[r]; row[r] = t; }  // reverse rows
}
```

**Set Matrix Zeroes (LC 73)** — O(1) extra space: **use the first row and first column as the flag arrays**, after remembering separately whether they themselves held a zero.
```java
static void setZeroes(int[][] m) {
    int rows = m.length, cols = m[0].length;
    boolean firstRowZero = false, firstColZero = false;
    for (int c = 0; c < cols; c++) if (m[0][c] == 0) firstRowZero = true;
    for (int r = 0; r < rows; r++) if (m[r][0] == 0) firstColZero = true;
    for (int r = 1; r < rows; r++)
        for (int c = 1; c < cols; c++)
            if (m[r][c] == 0) { m[r][0] = 0; m[0][c] = 0; }
    for (int r = 1; r < rows; r++)
        for (int c = 1; c < cols; c++)
            if (m[r][0] == 0 || m[0][c] == 0) m[r][c] = 0;
    if (firstRowZero) Arrays.fill(m[0], 0);
    if (firstColZero) for (int r = 0; r < rows; r++) m[r][0] = 0;
}
```
(The simple version — two `boolean[]` of size `m` and `n` — is O(m+n) space and a perfectly good first answer; say so, then optimise.)

**Game of Life (LC 289)** — all cells must update *simultaneously*, so store the next state **in the same cell** using two bits: bit 0 = current state, bit 1 = next state. Count neighbours with `& 1` (the current bit), write the next bit with `|= 2`, then shift everything right once.
```java
static void gameOfLife(int[][] b) {
    int R = b.length, C = b[0].length;
    for (int r = 0; r < R; r++)
        for (int c = 0; c < C; c++) {
            int live = 0;
            for (int dr = -1; dr <= 1; dr++)
                for (int dc = -1; dc <= 1; dc++) {
                    if (dr == 0 && dc == 0) continue;
                    int nr = r + dr, nc = c + dc;
                    if (nr >= 0 && nr < R && nc >= 0 && nc < C) live += b[nr][nc] & 1;
                }
            boolean alive = (b[r][c] & 1) == 1;
            if (alive ? (live == 2 || live == 3) : live == 3) b[r][c] |= 2;
        }
    for (int r = 0; r < R; r++) for (int c = 0; c < C; c++) b[r][c] >>= 1;
}
```
*(`&`, `|=`, `>>=` are the bit operators Day 95 teaches in depth; here you only need "bit 0 and bit 1 are two independent flags".)*

**Valid Sudoku (LC 36)** — one pass, three sets of seen digits. Box index = `(r / 3) * 3 + c / 3`. Bitmasks (`int[9]` each for rows, columns, boxes; `bit = 1 << (digit - 1)`) are the compact form of three `HashSet`s.

## 3. Numbers and strings: implement the thing without the library
Interviewers use these to test **overflow, edge cases and carrying**, not cleverness. Always state the edge cases first: sign, zero, empty, overflow, leading zeros.
```java
static double myPow(double x, int n) {                 // LC 50 — fast exponentiation, O(log n)
    long e = n;                                        // Integer.MIN_VALUE: -n would overflow an int
    if (e < 0) { x = 1 / x; e = -e; }
    double result = 1;
    while (e > 0) { if ((e & 1) == 1) result *= x; x *= x; e >>= 1; }
    return result;
}
static int mySqrt(int x) {                             // LC 69 — binary search on the answer (Days 32–33)
    if (x < 2) return x;
    int lo = 1, hi = x / 2, ans = 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (mid <= x / mid) { ans = mid; lo = mid + 1; }   // mid*mid <= x, written to avoid int overflow
        else hi = mid - 1;
    }
    return ans;
}
static int[] plusOne(int[] d) {                        // LC 66 — carry from the right; all 9s → new array
    for (int i = d.length - 1; i >= 0; i--) { if (d[i] < 9) { d[i]++; return d; } d[i] = 0; }
    int[] res = new int[d.length + 1]; res[0] = 1; return res;
}
static String multiply(String a, String b) {           // LC 43 — grade-school multiplication into int[m+n]
    if (a.equals("0") || b.equals("0")) return "0";
    int[] p = new int[a.length() + b.length()];
    for (int i = a.length() - 1; i >= 0; i--)
        for (int j = b.length() - 1; j >= 0; j--) {
            int sum = (a.charAt(i) - '0') * (b.charAt(j) - '0') + p[i + j + 1];
            p[i + j + 1] = sum % 10;                   // digit i of a × digit j of b lands at positions i+j and i+j+1
            p[i + j] += sum / 10;
        }
    StringBuilder sb = new StringBuilder();
    for (int v : p) if (!(sb.length() == 0 && v == 0)) sb.append(v);
    return sb.toString();
}
static int reverseInt(int x) {                         // LC 7 — test overflow BEFORE multiplying by 10
    int rev = 0;
    while (x != 0) {
        int d = x % 10; x /= 10;
        if (rev > Integer.MAX_VALUE / 10 || (rev == Integer.MAX_VALUE / 10 && d > 7)) return 0;
        if (rev < Integer.MIN_VALUE / 10 || (rev == Integer.MIN_VALUE / 10 && d < -8)) return 0;
        rev = rev * 10 + d;
    }
    return rev;
}
```
**String to Integer (LC 8)** — skip leading spaces → optional `+`/`-` → digits until a non-digit → **clamp** to `Integer.MIN/MAX_VALUE` (accumulate in a `long` and check after every digit). **Roman to Integer (LC 13)** — scan left to right; if the current symbol is smaller than the next one, *subtract* it, else add (`IV` = −1 + 5). **Decode String (LC 394)** — two stacks (counts and the strings built so far): on `[` push and reset, on `]` pop and append `repeat(count)`. It is the Day 40–45 stack pattern with strings.

## 4. In-place array tricks
```java
static void rotateArray(int[] a, int k) {              // LC 189 — three reversals, O(n) time, O(1) space
    k %= a.length;
    reverse(a, 0, a.length - 1); reverse(a, 0, k - 1); reverse(a, k, a.length - 1);
}
static void reverse(int[] a, int l, int r) { while (l < r) { int t = a[l]; a[l++] = a[r]; a[r--] = t; } }

static void nextPermutation(int[] a) {                 // LC 31 — find the pivot, swap, reverse the suffix
    int i = a.length - 2;
    while (i >= 0 && a[i] >= a[i + 1]) i--;            // 1. rightmost position where a[i] < a[i+1]
    if (i >= 0) {
        int j = a.length - 1;
        while (a[j] <= a[i]) j--;                      // 2. rightmost element larger than a[i]
        int t = a[i]; a[i] = a[j]; a[j] = t;
    }
    reverse(a, i + 1, a.length - 1);                   // 3. the suffix was descending → make it ascending (smallest)
}
static String largestNumber(int[] nums) {              // LC 179 — a custom comparator: which order gives the bigger string?
    String[] s = Arrays.stream(nums).mapToObj(String::valueOf).toArray(String[]::new);
    Arrays.sort(s, (x, y) -> (y + x).compareTo(x + y));
    return s[0].equals("0") ? "0" : String.join("", s);     // "00" → "0"
}
```
Next Permutation in words (say this out loud): *"scan from the right for the first dip; swap it with the smallest larger element to its right; then reverse what is to the right of the dip."* If there is no dip, the array is the last permutation — reversing the whole array gives the first.

## 5. "Design a data structure" — pick the structure that makes each operation O(1) or O(log n)
**Insert Delete GetRandom O(1) (LC 380).** `ArrayList` for O(1) random access + `HashMap<value, index>` for O(1) lookup. **Delete = overwrite the hole with the last element, then remove the last** (removing from the end is O(1); removing from the middle is not).
```java
static class RandomizedSet {
    private final List<Integer> items = new ArrayList<>();
    private final Map<Integer, Integer> pos = new HashMap<>();
    private final Random rnd = new Random();
    boolean insert(int v) { if (pos.containsKey(v)) return false; pos.put(v, items.size()); items.add(v); return true; }
    boolean remove(int v) {
        Integer i = pos.remove(v);
        if (i == null) return false;
        int last = items.remove(items.size() - 1);
        if (i < items.size()) { items.set(i, last); pos.put(last, i); }   // skip when we removed the last item itself
        return true;
    }
    int getRandom() { return items.get(rnd.nextInt(items.size())); }
}
```
**Time Based Key-Value Store (LC 981).** `HashMap<String, List<(timestamp, value)>>`; timestamps for a key arrive in increasing order, so `get(key, t)` is a **binary search for the last entry with `ts <= t`** (the Day 28–31 "rightmost ≤" variant). O(1) `set` (append), O(log n) `get`.
**Design Twitter (LC 355).** Per-user list of `(time, tweetId)`; a user's feed = **merge the last 10 tweets of the user and everyone they follow** with a max-heap (the k-way merge from Day 55). The follow graph is `Map<Integer, Set<Integer>>`. Say the production caveat: this is "fan-out on read"; Day 129's news feed discusses fan-out on write.
**LFU Cache (LC 460)** — the hard sibling of the LRU you built on Day 39 (the plan has no LFU problem), a stretch sketch: `Map<key, value+freq>`, `Map<freq, LinkedHashSet<key>>` (insertion order breaks ties = least *recently* used among the least frequent) and an `int minFreq`. On every `get`/`put`, move the key from bucket `f` to `f+1` (if bucket `f` empties and `f == minFreq`, bump `minFreq`); on eviction remove the first key of `buckets[minFreq]`. All O(1). Day 18's file has the `LinkedHashMap` one-liner for plain LRU.

## 6. Linked-list stragglers (sketches — do them as stretch)
- **Sort List (LC 148):** merge sort on a list. Find the middle with slow/fast, cut it, sort both halves recursively, `merge` them (LC 21 — which you have done). O(n log n), O(log n) stack. *(Day 102 covers merge sort on arrays; the list version is the one interviewers prefer.)*
- **Reverse Nodes in k-Group (LC 25):** a dummy head; for each group, check that *k* nodes exist (`kth`); reverse the group by re-pointing `next` towards `groupNext`; re-attach `groupPrev.next = kth` and move `groupPrev` to the old first node. Leave a short tail untouched. Draw it with k = 3 before coding.

## 7. Exercise (≈ 1 h 15 min of coding — put the solutions in `dsa-java`)
1. **Warm-up (15 min):** Spiral Matrix, Rotate Image, Set Matrix Zeroes from a blank file. Test each on a 1×1, 1×4, 4×1, 3×4 and square input.
2. **Core (35 min):** Game of Life, Valid Sudoku (bitmasks), `myPow` (include `n = Integer.MIN_VALUE`), `mySqrt` (include `2147395599` → `46339`), Multiply Strings (`"999" × "999"` → `"998001"`), Reverse Integer (`1534236469` → `0`).
3. **Interview-shaped (25 min):** Next Permutation (trace `[1,5,8,4,7,6,5,3,1]` by hand first), Rotate Array, RandomizedSet, TimeMap. Say the complexity of each out loud.
4. **Log them** in the Curriculum Map's inventory style (LC number, pattern, "solved cold?") so Day 105A's mock can draw from them.

## Checklist
- [ ] Can write spiral traversal with the two guards, and say why a single row/column breaks it without them.
- [ ] Can rotate a matrix in place (transpose + reverse) and set zeroes in O(1) extra space.
- [ ] Can encode two states in one cell (Game of Life) and name the overflow checks in Reverse Integer.
- [ ] Can write fast exponentiation (negative and `MIN_VALUE` exponents) and `sqrt` by binary search without overflow.
- [ ] Can explain RandomizedSet's "swap with last" delete and TimeMap's binary search in one sentence each.
- [ ] Continue with the regular plan.
