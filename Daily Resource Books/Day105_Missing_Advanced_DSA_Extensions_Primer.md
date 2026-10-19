# Day 105 — Missing Primer (STRETCH): Advanced DSA Extensions — KMP, Rolling Hash, Bitmask DP, Bridges, Difference Arrays, Math, Sampling

**Do this on Day 105 (Sunday, ≈ 2.5 h) or spread across the Sundays of Days 105/112/119 — after the Day 105A DSA mock, then continue with the regular plan.** Skip if you are behind; these are the techniques that show up in *hard* problems, not mediums.
**Needed by:** hard-level questions at product companies; Day 105A/113A/123A/132A mocks may brush them.
**Why it is missing:** Day 47's book says KMP "hasn't been covered in this series". KMP, Rabin–Karp, bitmask DP, Tarjan, GCD/sieve/modular arithmetic and reservoir sampling are only mentioned in passing; "difference array" is never named (Day 26's Meeting Rooms II hints at sweep line). Sparse tables, binary lifting and digit DP are deliberately left out — they are rare in SDE-2 loops.
**Already in place:** DSA patterns through Day 105 (trees, graphs, DP, heaps, tries, segment tree), bit manipulation (Day 95).

---

## 1. KMP (string search in O(n + m))
The **prefix function** `pi[i]` = length of the longest proper prefix of `s[0..i]` that is also a suffix. It lets a mismatch skip ahead instead of restarting.
```java
static int[] prefixFunction(String s) {
    int[] pi = new int[s.length()];
    for (int i = 1, k = 0; i < s.length(); i++) {
        while (k > 0 && s.charAt(i) != s.charAt(k)) k = pi[k - 1];
        if (s.charAt(i) == s.charAt(k)) k++;
        pi[i] = k;
    }
    return pi;
}
static int indexOf(String text, String pat) {
    if (pat.isEmpty()) return 0;
    int[] pi = prefixFunction(pat);
    for (int i = 0, k = 0; i < text.length(); i++) {
        while (k > 0 && text.charAt(i) != pat.charAt(k)) k = pi[k - 1];
        if (text.charAt(i) == pat.charAt(k)) k++;
        if (k == pat.length()) return i - k + 1;
    }
    return -1;
}
```
Practice: LC 28 *Find the Index of the First Occurrence*, LC 459 *Repeated Substring Pattern*, LC 214 *Shortest Palindrome*.

## 2. Rolling hash (Rabin–Karp)
Hash every substring in O(1) after an O(n) prefix build; compare hashes, verify on a match.
```java
final long MOD = 1_000_000_007L, BASE = 911_382_323L;
long[] h = new long[n + 1], p = new long[n + 1]; p[0] = 1;
for (int i = 0; i < n; i++) { h[i+1] = (h[i] * BASE + s.charAt(i)) % MOD; p[i+1] = p[i] * BASE % MOD; }
long hash(int l, int r) { return ((h[r] - h[l] * p[r - l]) % MOD + MOD) % MOD; }   // substring s[l, r)
```
Practice: LC 187 *Repeated DNA Sequences*, LC 1044 *Longest Duplicate Substring* (binary search on length + hash set).

## 3. Bitmask DP (subsets as numbers)
When n ≤ ~20, encode "which items are used" as a bitmask: state `dp[mask][last]`.
```java
int FULL = 1 << n, INF = 1_000_000_000;
int[][] dp = new int[FULL][n];
for (int[] row : dp) Arrays.fill(row, INF);
for (int i = 0; i < n; i++) dp[1 << i][i] = 0;
for (int mask = 1; mask < FULL; mask++)
    for (int last = 0; last < n; last++) {
        if ((mask >> last & 1) == 0 || dp[mask][last] == INF) continue;
        for (int nxt = 0; nxt < n; nxt++) if ((mask >> nxt & 1) == 0)
            dp[mask | 1 << nxt][nxt] = Math.min(dp[mask | 1 << nxt][nxt], dp[mask][last] + dist[last][nxt]);
    }
// answer: min over last of dp[FULL - 1][last]        — O(2^n · n^2)
```
Practice: LC 698 *Partition to K Equal Sum Subsets*, LC 847 *Shortest Path Visiting All Nodes*, LC 1125 *Smallest Sufficient Team*.

## 4. Tarjan: bridges (and the low-link idea behind SCCs)
`disc[u]` = discovery time; `low[u]` = earliest discovery time reachable from `u`'s subtree using at most one back edge. An edge `(u, v)` is a **bridge** if `low[v] > disc[u]`.
```java
void dfs(int u, int parent) {
    disc[u] = low[u] = ++timer;
    for (int v : adj.get(u)) {
        if (v == parent) continue;
        if (disc[v] == 0) { dfs(v, u); low[u] = Math.min(low[u], low[v]); if (low[v] > disc[u]) bridges.add(List.of(u, v)); }
        else low[u] = Math.min(low[u], disc[v]);
    }
}
```
Practice: LC 1192 *Critical Connections in a Network*. (Parallel edges need edge ids instead of the parent check.)

## 5. Difference array and sweep line
**Difference array:** many range updates, one pass to read.
```java
int[] diff = new int[n + 1];
for (int[] u : updates) { diff[u[0]] += u[2]; diff[u[1] + 1] -= u[2]; }   // add u[2] to [l..r]
for (int i = 0, run = 0; i < n; i++) { run += diff[i]; arr[i] = run; }
```
**Sweep line:** turn intervals into ± events, sort, accumulate.
```java
List<int[]> ev = new ArrayList<>();
for (int[] m : meetings) { ev.add(new int[]{m[0], 1}); ev.add(new int[]{m[1], -1}); }
ev.sort((a, b) -> a[0] != b[0] ? Integer.compare(a[0], b[0]) : Integer.compare(a[1], b[1]));   // ends (-1) before starts (+1) at the same time
int cur = 0, best = 0; for (int[] e : ev) { cur += e[1]; best = Math.max(best, cur); }
```
Practice: LC 370 *Range Addition*, LC 1109 *Corporate Flight Bookings*, LC 253 *Meeting Rooms II*, LC 1094 *Car Pooling*.

## 6. Math toolkit
```java
static long gcd(long a, long b) { return b == 0 ? a : gcd(b, a % b); }
static long modPow(long b, long e, long m) { long r = 1; b %= m; while (e > 0) { if ((e & 1) == 1) r = r * b % m; b = b * b % m; e >>= 1; } return r; }
boolean[] composite = new boolean[n + 1];                         // sieve of Eratosthenes, O(n log log n)
for (int i = 2; (long) i * i <= n; i++) if (!composite[i]) for (int j = i * i; j <= n; j += i) composite[j] = true;
```
Practice: LC 204 *Count Primes*, LC 50 *Pow(x, n)*, LC 1071 *Greatest Common Divisor of Strings*. (Modular answers "modulo 10⁹+7": reduce after every multiply and add.)

## 7. Sampling
**Fisher–Yates shuffle** (uniform): `for (int i = n - 1; i > 0; i--) swap(a, i, rnd.nextInt(i + 1));`
**Reservoir sampling** (one item from a stream of unknown length): `seen++; if (rnd.nextInt(seen) == 0) chosen = x;`
Practice: LC 384 *Shuffle an Array*, LC 382 *Linked List Random Node*, LC 398 *Random Pick Index*.

## Checklist
- [ ] Can explain the prefix function and write KMP from memory; can say when a rolling hash beats it.
- [ ] Can set up `dp[mask][last]` and state its complexity.
- [ ] Can state the bridge condition `low[v] > disc[u]`.
- [ ] Can apply a difference array and a sweep line.
- [ ] At least one practice problem per section attempted (or consciously skipped).
