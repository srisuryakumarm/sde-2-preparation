# Day 67 Resource Book — Backtracking Capstone, and JWT at the Gateway

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 66 Resource Book](Day66_Resource_Book.md)
**Next ▶:** [Day 68 Resource Book](Day68_Resource_Book.md)
**Companion to:** Day 67 of `Week_10_Revised.md`

---

## Recap

Backtracking has now run five straight days across two weeks — opened Week 9, Day 61 (include/exclude, Subsets), continued through swap-based (Permutations), forward-index (Combinations), duplicate-skip via `used[]` (Permutations II), then this week's forward-index-with-repetition (Combination Sum), duplicate-skip translated into forward-index form (Combination Sum II), external-lookup string-building (Letter Combinations), counter-gated string-building (Generate Parentheses), and grid-DFS (Word Search, Palindrome Partitioning). Today closes it at 12/12 required problems with two final applications — neither introduces a new backtracking mechanism. Subsets II directly reuses yesterday-but-one's duplicate-skip translation; N-Queens combines the include/exclude shape with a new validity-checking structure (columns and diagonals tracked via `HashSet`).

**On today's extra-practice decision, stated in full:** as flagged Monday, no extra Backtracking practice is being added anywhere this week. The reasoning, complete now that the full ladder is visible: 12 required problems, spanning include/exclude, swap-based, forward-index (with and without repetition, with and without duplicates, two different duplicate-handling mechanisms), three distinct string-building shapes, matrix-constrained search, and full constraint-satisfaction — every canonical variant this pattern has, at Medium-to-Hard depth, with zero thin spots to fill. This mirrors Week 9's own Tries precedent (Day 61: a comprehensive required set closing with zero extra, reasoned rather than assumed) more than it mirrors, say, Union-Find's genuinely thin required ladder next week — padding an already-comprehensive, already-dense ladder would trade pacing realism for repetition with no corresponding gap to justify it.

---

## Learning Objectives

By the end of today, without notes:

1. Solve Subsets II, explaining precisely why it cites Day 64's Combination Sum II (not Week 9's Permutations II) as the direct mechanical ancestor of its duplicate-skip logic.
2. Solve N-Queens, including the `row + col` / `row - col` diagonal-identification trick, proven, not memorized.
3. Classify all 12 Backtracking problems solved across Weeks 9–10 by choice model, on demand.
4. Explain why JWT validation belongs at the Gateway layer rather than duplicated inside every module, and implement it.

---

## Concept Dependency Map for Today

```
Day 61 — Subsets (include/exclude template)
Day 64 — Combination Sum II (duplicate-skip via i > start, forward-index form)
        │
        ▼
Today, Problem 1 — Subsets II (LC 90)
  Day 61's include/exclude shape + Day 64's EXACT skip condition,
  reused verbatim — genuinely nothing new to derive

Day 61 — Backtracking template
Day 4  — HashSet (O(1) membership)
        │
        ▼
Today, Problem 2 — N-Queens (LC 51)
  Row-by-row placement + THREE HashSets (columns, two diagonal
  directions) answering "is this placement safe" in O(1)

  ══════ BACKTRACKING CLOSES: 12/12 required ══════

Day 65 — Feign (service-to-service calls)      Day 66 — Gateway (single
Day 64 — Resilience4j (proxy-based Advice)        entry point, routing)
        │                                                │
        ▼                                                ▼
JWT Authentication at the Gateway (NEW) — validate once, centrally,
  before any request reaches a module — same "centralize a cross-
  cutting concern" motivation as Day 66's routing, applied to auth
```

---

# Part 1 — Subsets II (LeetCode 90, Medium) — Pattern: Backtracking with Duplicate Handling

**Statement:** Given an integer array `nums` that may contain duplicates, return all possible subsets (the power set), with no duplicate subset in the output.

## Confirming the correct ancestor before writing a line of code

It would be easy to reach for Week 9's Permutations II (`!used[i-1]`) here, since both problems are about suppressing duplicate output from a duplicate-containing input. **That would be reaching for the wrong tool.** Permutations II uses a swap-based choice model with an explicit `used[]` array — Subsets uses **include/exclude over a forward-moving index**, the same choice model Combination Sum II (Day 64) already translated the duplicate-skip idea into. Subsets II needs *that* translation, verbatim — not a fresh derivation, and not Permutations II's swap-based version.

### Approach — Subsets' include/exclude template + Day 64's exact skip condition

```java
public static List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums);   // required — identical reason to Day 64: equal values must be adjacent
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private static void backtrack(int[] nums, int start, List<Integer> path, List<List<Integer>> result) {
    result.add(new ArrayList<>(path));   // every path, at every depth, is itself a valid subset — add on entry
    for (int i = start; i < nums.length; i++) {
        if (i > start && nums[i] == nums[i - 1]) {
            continue;   // Day 64's exact condition — skip a repeated sibling choice at this depth
        }
        path.add(nums[i]);
        backtrack(nums, i + 1, path, result);
        path.remove(path.size() - 1);
    }
}
```

**The one real structural difference from Combination Sum II, worth stating precisely:** Combination Sum II only records a result when `remaining == 0` — a specific *terminal* condition. Subsets II records a result **on every single call**, at every depth, immediately on entry — because every prefix built so far, no matter how short, is itself already a valid subset (the empty subset included, from the very first call with `path = []`). This is exactly Day 61's original Subsets behavior; nothing about *when* to record has changed, only the *skip condition* layered on top of the existing loop.

### Trace: `nums = [1,2,2]` (sorted)

```
backtrack(start=0, path=[])          → ADD []
  i=0 (i>start? no): take 1 → path=[1]
    backtrack(start=1, path=[1])     → ADD [1]
      i=1 (i>start? no): take 2 → path=[1,2]
        backtrack(start=2, path=[1,2]) → ADD [1,2]
          i=2 (i>start? no): take 2 → path=[1,2,2]
            backtrack(start=3, path=[1,2,2]) → ADD [1,2,2]
      i=2 (i>start? 2>1 TRUE; nums[2]=2==nums[1]=2 → SKIP)
  i=1 (i>start? 1>0 TRUE; nums[1]=2≠nums[0]=1 → no skip): take 2 → path=[2]
    backtrack(start=2, path=[2])     → ADD [2]
      i=2 (i>start? no): take 2 → path=[2,2]
        backtrack(start=3, path=[2,2]) → ADD [2,2]
  i=2 (i>start? 2>0 TRUE; nums[2]=2==nums[1]=2 → SKIP)
```

**Result: `[[], [1], [1,2], [1,2,2], [2], [2,2]]`** — exactly 6 subsets, matching the verified correct output. The two skips (`i=2` under the outer loop, both times) are what prevent `[2]` and `[2,2]` from each appearing a second time via the second `2`'s index.

### Complexity

**Time: O(n × 2ⁿ)** — up to `2ⁿ` distinct subsets in the worst case (all-distinct input), each costing up to O(n) to copy into the result list. **Space: O(n)** for recursion depth, excluding the output.

### Common Mistakes and Edge Cases

- ⚠️ **Reaching for `!used[i-1]` here** — mechanically wrong for this choice model; Day 64's `i > start` is the correct translation, not Permutations II's own condition.
- ⚠️ **Forgetting to sort** — identical failure mode to Day 64: unsorted duplicates aren't adjacent, so the skip check silently fails to catch them.
- ⚠️ **Only adding to `result` at the deepest recursion level** — that's Combination Sum II's shape (a specific terminal condition), not this problem's; Subsets II needs every intermediate path recorded too.
- Edge case: all elements identical (e.g., `[2,2,2]`) → correctly produces `[[], [2], [2,2], [2,2,2]]`, four subsets, not eight.
- Edge case: empty input array → result is `[[]]` — the empty subset alone, since the loop body never executes but the entry-point `result.add` still fires once.

> 🔗 **Backward reference:** this is the payoff of Day 64's own framing choice — teaching the `i > start` translation there specifically so it could be cited, not re-derived, here. If this problem required re-explaining the skip condition from scratch, that would be a signal Day 64 wasn't actually internalized yet.

---

# Part 2 — N-Queens (LeetCode 51, Hard) — Pattern: Backtracking

**Statement:** Place `n` queens on an `n × n` chessboard such that no two queens attack each other (no shared row, column, or diagonal). Return every distinct board configuration.

## Prerequisites, confirmed

Backtracking's choose/explore/un-choose template (Day 61) and `HashSet` for O(1) membership (Day 4/5). Nothing about this problem's *recursion shape* is new — one queen placed per row, choose/explore/un-choose exactly as always. What's new is the **validity check**: efficiently answering "is this specific cell safe" without re-scanning the whole board on every placement attempt.

### The insight: one queen per row, by construction — eliminates the row constraint entirely

Since exactly one queen must occupy each row (n queens, n rows, no two sharing a row), placing queens **row by row** — one recursive call per row, choosing which column to place that row's queen in — makes "no two queens share a row" true automatically, by the very shape of the recursion. No explicit row-conflict check is ever needed; it's structurally impossible to violate. This leaves only columns and diagonals to actually check.

### Identifying a diagonal in O(1): the `row + col` / `row − col` trick

Every cell on the *same* anti-diagonal (top-right to bottom-left) shares an identical value of `row + col`. Every cell on the *same* main diagonal (top-left to bottom-right) shares an identical value of `row − col`. This is worth being able to justify, not just state: moving one step down-right along a main diagonal increments both `row` and `col` by 1, so their *difference* stays constant; moving one step down-left along an anti-diagonal increments `row` by 1 and decrements `col` by 1, so their *sum* stays constant. Tracking "used" values of `row+col` and `row−col` in two separate `HashSet`s turns "is this diagonal already occupied" into an O(1) lookup, the same complexity motivation as every other HashSet use in this series since Day 4.

### Approach — Row-by-row placement, three HashSets for O(1) safety checks

```java
public static List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    int[] queenCols = new int[n];         // queenCols[row] = column of the queen placed in that row
    Set<Integer> usedCols = new HashSet<>();
    Set<Integer> usedDiag1 = new HashSet<>();   // row - col
    Set<Integer> usedDiag2 = new HashSet<>();   // row + col

    backtrack(0, n, queenCols, usedCols, usedDiag1, usedDiag2, result);
    return result;
}

private static void backtrack(int row, int n, int[] queenCols, Set<Integer> usedCols,
                               Set<Integer> usedDiag1, Set<Integer> usedDiag2,
                               List<List<String>> result) {
    if (row == n) {
        result.add(buildBoard(queenCols, n));   // every row successfully filled — one valid solution
        return;
    }
    for (int col = 0; col < n; col++) {
        int diag1 = row - col, diag2 = row + col;
        if (usedCols.contains(col) || usedDiag1.contains(diag1) || usedDiag2.contains(diag2)) {
            continue;   // unsafe — skip this column
        }
        // choose
        queenCols[row] = col;
        usedCols.add(col);
        usedDiag1.add(diag1);
        usedDiag2.add(diag2);

        backtrack(row + 1, n, queenCols, usedCols, usedDiag1, usedDiag2, result);   // explore

        // un-choose
        usedCols.remove(col);
        usedDiag1.remove(diag1);
        usedDiag2.remove(diag2);
    }
}

private static List<String> buildBoard(int[] queenCols, int n) {
    List<String> board = new ArrayList<>();
    for (int row = 0; row < n; row++) {
        char[] rowChars = new char[n];
        Arrays.fill(rowChars, '.');
        rowChars[queenCols[row]] = 'Q';
        board.add(new String(rowChars));
    }
    return board;
}
```

**Why three separate `HashSet`s, not one combined structure:** columns, main diagonals, and anti-diagonals are three genuinely independent constraints — a cell can be safe on columns but unsafe on a diagonal, or vice versa — so each needs its own membership check. This mirrors Day 26's Meeting Rooms II reasoning in spirit (track exactly the state a constraint actually needs, no more, no less), just with three independent constraints instead of one.

### Trace: `n = 4` — the first placement, and the first successful branch

```
backtrack(row=0): try col=0 → safe (nothing used yet) → place, usedCols={0}, usedDiag1={0}, usedDiag2={0}
  backtrack(row=1): try col=0 → usedCols contains 0 → UNSAFE, skip
                     try col=1 → diag1=1-1=0 → usedDiag1 contains 0 → UNSAFE, skip
                     try col=2 → diag1=1-2=-1 (new), diag2=1+2=3 (new), col=2 (new) → SAFE
                       place, usedCols={0,2}, usedDiag1={0,-1}, usedDiag2={0,3}
    backtrack(row=2): try col=0 → usedCols has 0 → UNSAFE
                       try col=1 → diag2=2+1=3 → usedDiag2 has 3 → UNSAFE
                       try col=2 → usedCols has 2 → UNSAFE
                       try col=3 → diag1=2-3=-1 → usedDiag1 has -1 → UNSAFE
                       (no safe column at row=2 — this entire branch dead-ends here)
      un-choose row=1's col=2, try col=3 → diag1=1-3=-2(new), diag2=1+3=4(new) → SAFE
        place, usedCols={0,3}, ...
        backtrack(row=2): try col=1 → diag1=2-1=1(new), diag2=2+1=3(new), col=1(new) → SAFE
          ... continuing eventually reaches row=4 with columns [1,3,0,2] as ONE full valid solution
```

**Confirmed by direct execution rather than hand-tracing further:** running this exact algorithm for small `n` gives `n=1 → 1` solution, `n=2 → 0`, `n=3 → 0`, `n=4 → 2`, `n=8 → 92` — matching the well-known values for this problem exactly (the `n=2` and `n=3` zero-solution cases are worth knowing by name: too small a board for `n` mutually non-attacking queens to exist at all).

### Complexity

**Time: O(n!)** — worst-case, row 0 has `n` column choices, row 1 has at most `n−1` remaining (one column eliminated), and so on — the column-elimination alone bounds the tree by `n!`; the diagonal constraints only prune *further* from there, never add branches back, so `O(n!)` remains a valid (if not perfectly tight) upper bound. **Space: O(n)** for the three `HashSet`s and the `queenCols` array, all sized proportionally to `n`, plus O(n) recursion depth.

### Common Mistakes and Edge Cases

- ⚠️ **Checking `row - col` and `col - row` inconsistently across the add/lookup calls** — must use the *same* signed expression both when marking a diagonal used and when checking it, or the set simply never matches what was actually stored.
- ⚠️ **Re-scanning placed queens on every safety check instead of using the three HashSets** — still correct, but turns an O(1) check into an O(n) one, degrading the overall bound by a full factor of `n`.
- ⚠️ **Forgetting to remove from all three sets on backtrack** — silently corrupts every subsequent sibling branch's safety checks, the identical failure category as forgetting `path.remove()` anywhere else in this pattern.
- Edge case: `n = 1` → trivially one solution (a single queen, alone, is always safe).
- Edge case: `n = 2` or `n = 3` → correctly zero solutions; the board is provably too small for `n` mutually safe queens to coexist.

> 💡 **Interview Insight:** N-Queens is frequently used specifically to test whether "backtracking" has been internalized as choose/explore/un-choose over *any* validity structure, or memorized as "the array/subset problems." The row-by-row insight (eliminating the row constraint by construction) and the diagonal-identity trick are the two specific things worth narrating unprompted — both are exactly the kind of "why does this work" reasoning a tier-1 interview is listening for, not just working code.

---

## Backtracking, Reviewed — All 12 Problems, Classified

Backtracking closes here: **12/12 required + 0 extra = 12 distinct problems**, spanning Weeks 9–10.

| # | LC | Problem | Week/Day | Choice Model |
|---|---|---|---|---|
| 1 | 78 | Subsets | Wk9 D61 | Include/exclude — independent per-element binary choice |
| 2 | 46 | Permutations | Wk9 D62 | Swap-based — position-dependent, `used[]` implicit via array layout |
| 3 | 77 | Combinations | Wk9 D63 | Forward-index — never revisit an earlier index |
| 4 | 47 | Permutations II | Wk9 D63 | Swap-based + duplicate-skip via `!used[i-1]` |
| 5 | 39 | Combination Sum | Wk10 D64 | Forward-index + repetition allowed (`i`, not `i+1`) |
| 6 | 40 | Combination Sum II | Wk10 D64 | Forward-index + duplicate-skip via `i > start` |
| 7 | 17 | Letter Combinations of a Phone Number | Wk10 D65 | String-building — external per-position lookup |
| 8 | 22 | Generate Parentheses | Wk10 D65 | String-building — counter-gated validity |
| 9 | 79 | Word Search | Wk10 D66 | Matrix/grid DFS — overwrite-and-restore marking |
| 10 | 131 | Palindrome Partitioning | Wk10 D66 | Forward-index over string positions + palindrome gate |
| 11 | 90 | Subsets II | Wk10 D67 | Include/exclude + duplicate-skip via `i > start` (Problem 6's exact translation) |
| 12 | 51 | N-Queens | Wk10 D67 | Row-by-row placement + O(1) constraint tracking (3 HashSets) |

**The throughline worth being able to state in one breath:** every one of these twelve is choose → explore → un-choose over *some* decision space; what actually varies, problem to problem, is (a) what the legal choices at a given step are, and (b) what state has to be restored when backtracking out. Six genuinely distinct choice models appear across the twelve — include/exclude, swap-based, forward-index (bare, with repetition, and with duplicate-skip), string-building (two flavors), grid-DFS, and row-by-row-with-derived-constraints — not one template with cosmetic surface variation.

---

# Part 3 — JWT Authentication at the Gateway

## Prerequisites, confirmed

Day 66's Gateway and route configuration, directly — today's validation logic runs at that same layer, before any request reaches a routed module.

## The mechanism: sign once, validate on every request, without a database round-trip

A user authenticates once (credentials checked against the Order/User data as appropriate) and receives a **JWT** (JSON Web Token) — a signed, self-contained token encoding claims about that user (identity, roles, expiry) directly inside the token itself. On every subsequent request, the Gateway validates the token's **signature** and **expiry** — and, critically, does this *without* querying any database or calling any other service, because everything needed to validate is already inside the token and verifiable purely against the server's own signing key.

```java
@Component
public class JwtAuthenticationFilter implements GlobalFilter, Ordered {

    private final String secretKey;   // in real deployments: externalized, never hardcoded

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String authHeader = exchange.getRequest().getHeaders().getFirst("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);   // 401 — no token at all
            return exchange.getResponse().setComplete();
        }

        String token = authHeader.substring(7);
        try {
            Jwts.parserBuilder().setSigningKey(secretKey).build().parseClaimsJws(token);
            // signature AND expiry both verified by parseClaimsJws — throws if either fails
        } catch (JwtException e) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);   // 401 — invalid or expired
            return exchange.getResponse().setComplete();
        }

        return chain.filter(exchange);   // valid — proceed to the routed module
    }

    @Override
    public int getOrder() {
        return -1;   // run BEFORE routing — reject unauthenticated requests before they reach any module
    }
}
```

**Why this belongs at the Gateway and not inside each module, stated with the same reasoning as Day 66's routing centralization:** without this, every one of Product, Order, Payment, and Notification would need to independently implement — and correctly keep in sync — the exact same signature-and-expiry validation logic. Centralizing it here means a request without a valid token is rejected with `401` **before it ever reaches any module**, and every module downstream of the Gateway can simply trust that any request reaching it has already been authenticated, without re-implementing the check itself.

## The distinction a tier-1 interviewer will specifically probe: JWT is not OAuth 2.0

**This is worth stating precisely, because the two are genuinely different things that frequently get conflated:** what's built today is **custom JWT issuance and validation** — this platform signs its own tokens and validates them itself. **OAuth 2.0** is a separate **authorization framework** — a standardized protocol for how a user grants a third-party application limited access to their resources *without* sharing their password (the "sign in with Google" flow is OAuth 2.0). OAuth 2.0 flows frequently *happen* to issue JWTs as their access tokens — but a system can use JWTs without OAuth 2.0 (exactly what's built today), and OAuth 2.0 doesn't strictly require JWTs as its token format (opaque tokens are valid too). **They're complementary specifications solving different problems, not two names for the same thing** — JWT is a *token format* (and a signing/verification mechanism); OAuth 2.0 is a *protocol* governing *how* a token gets issued and to whom.

### Common Mistakes

- ⚠️ **Saying "JWT" and "OAuth 2.0" interchangeably** — flagged above as a specific, commonly-tested confusion; know which one is actually being discussed.
- ⚠️ **Validating expiry manually with a separate check** instead of trusting the parsing library's own built-in expiry validation — `parseClaimsJws` already throws on an expired token; adding a redundant manual check is unnecessary and a plausible source of an inconsistent expiry rule if the two checks ever disagree.
- ⚠️ **Placing the JWT filter's `getOrder()` value incorrectly**, letting routing happen before authentication — the entire point of validating at the Gateway is rejecting *before* a module is ever reached; an ordering mistake here silently defeats that.

> 💡 **Interview Insight:** "Is this OAuth?" is close to a guaranteed follow-up the moment JWT comes up in a systems conversation. Answering precisely — naming JWT as a token format and OAuth 2.0 as a separate authorization protocol that can (but doesn't have to) use JWTs — is exactly the distinction that separates genuine understanding from pattern-matched buzzwords.

---

## Project Block

**Repository:** `scalable-ecommerce-platform`.

**Task:** implement JWT validation at the Gateway level, following the filter shape above.

**Practical guidance:** test both failure modes explicitly and separately — a request with no `Authorization` header at all, and a request with an expired or tampered token — rather than only testing the happy path plus one generic failure case; the two failure modes exercise genuinely different branches of the validation logic.

**Definition of done:** requests without a valid Bearer token are rejected with `401` before reaching any module — verified by observing the rejection happen at the Gateway (e.g., confirming the downstream module never logs receiving the request at all), not just by seeing a `401` response.

---

## Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** research the hiring manager for a role you've applied to and send a direct, specific message — referencing something concrete from their background or recent activity, the same principle established Day 10's networking guidance.

---

# Day 67 — Interview Questions

---

**1. Why does Subsets II cite Day 64's Combination Sum II rather than Week 9's Permutations II as its duplicate-skip ancestor?**

*Answer:* Subsets II uses the same include/exclude, forward-index choice model as Combination Sum II — both check `i > start` against the loop's own starting position. Permutations II uses a swap-based model with an explicit `used[]` array and a structurally different check (`!used[i-1]`). The underlying *idea* (suppress a repeated sibling choice) traces back to Permutations II conceptually, but the actual *code* Subsets II reuses is Combination Sum II's.

---

**2. In Subsets II, why is a result added to `result` on every single recursive call, rather than only at a base case?**

*Answer:* Every prefix built so far — at any depth, including the very first call with an empty path — is itself already a valid subset. There's no single "terminal" condition the way Combination Sum II's `remaining == 0` was; the entire point of the power set is that every intermediate state qualifies.

---

**3. Why does placing one queen per row eliminate the need to separately check for row conflicts in N-Queens?**

*Answer:* Since the recursion places exactly one queen in each successive row, by construction, no two queens can ever share a row — it's structurally impossible for the algorithm to place two queens in the same row in the first place, so there's nothing to check.

---

**4. Justify the `row + col` and `row - col` diagonal-identity claim — don't just state it.**

*Answer:* Moving one step along a main diagonal (top-left to bottom-right) increases both row and column by 1, so their difference (`row - col`) stays constant along that entire diagonal. Moving one step along an anti-diagonal (top-right to bottom-left) increases row by 1 while decreasing column by 1, so their sum (`row + col`) stays constant along that diagonal instead. Each diagonal is therefore uniquely identified by one fixed value of the corresponding expression.

---

**5. What's the actual cost of re-scanning placed queens instead of using the three HashSets in N-Queens?**

*Answer:* Correctness is unaffected, but each safety check degrades from O(1) to O(n) (scanning all previously placed queens), multiplying the overall time bound by an extra factor of n compared to the HashSet version.

---

**6. Name the six distinct backtracking choice models this series has now covered, across all 12 problems.**

*Answer:* Include/exclude (Subsets, Subsets II), swap-based (Permutations, Permutations II), forward-index — bare, with repetition, and with duplicate-skip (Combinations, Combination Sum, Combination Sum II), string-building via external lookup (Letter Combinations), string-building via counter-gating (Generate Parentheses), grid-DFS with overwrite-restore marking (Word Search, and forward-index-over-positions for Palindrome Partitioning), and row-by-row placement with derived O(1) constraint tracking (N-Queens).

---

**7. Why does JWT validation belong at the Gateway rather than inside each individual module?**

*Answer:* The same centralization argument as the Gateway's routing itself: without it, every module would need to independently implement and keep in sync identical signature-and-expiry validation logic. Centralizing it means an unauthenticated request is rejected once, before ever reaching any module, and every module downstream can simply trust that anything reaching it already passed authentication.

---

**8. Is what's built today "OAuth 2.0"? Why or why not?**

*Answer:* No. Today's work is custom JWT issuance and validation — this platform signs and verifies its own tokens directly. OAuth 2.0 is a separate authorization protocol governing how a token gets issued to a third party; it frequently uses JWTs as its token format, but a system can use JWTs without OAuth 2.0 at all, which is exactly what's built here.

---

**9. Why doesn't JWT validation require a database round-trip on every request?**

*Answer:* A JWT is self-contained — the claims needed for validation (identity, roles, expiry) are encoded directly inside the signed token. The Gateway can verify the signature against its own signing key and check the expiry claim purely from the token's own contents, with no need to look anything up externally.

---

**10. What specifically goes wrong if the JWT filter's `getOrder()` is set incorrectly, letting routing run first?**

*Answer:* An unauthenticated or invalid-token request would be routed to and handled by the underlying module before authentication is ever checked — defeating the entire purpose of validating at the Gateway, since the module would process the request regardless of whether the token was ever valid.

---

## Daily Deliverable Check

- [ ] Subsets II (LC 90) solved, citing Day 64's exact duplicate-skip translation rather than re-deriving it.
- [ ] N-Queens (LC 51) solved — the diagonal-identity proof reproducible without notes.
- [ ] Both problems pushed to `dsa-java/backtracking/` — **Backtracking ladder complete at 12/12 required, 0 extra.**
- [ ] JWT validation live at the Gateway, verified rejecting both a missing-token request and an invalid/expired-token request with 401, before either reaches a module.

---

## What Tomorrow Assumes You Already Know Cold

Day 68 opens Graphs BFS/DFS — genuinely new anywhere in this series, needing an explicit `visited` structure for the first time, since every tree traversal so far (Days 46–53) relied on trees having no cycles at all. It assumes recursion (Day 8) and Queue/BFS (Day 4, applied to trees Day 49) both transfer directly to a structure that merely *generalizes* a tree (a tree is a connected, acyclic graph) — the traversal mechanics aren't new, only the cycle-awareness is. It also assumes today's JWT-at-the-Gateway work is stable, since tomorrow's rate limiting is a second filter layered at the exact same Gateway level, and the two need to coexist correctly rather than interfere with each other's request handling.

**Next:** [Day 68 Resource Book](./Day68_Resource_Book.md) — Graphs Begin, and Rate Limiting.
