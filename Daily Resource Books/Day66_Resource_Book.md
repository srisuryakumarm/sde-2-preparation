# Day 66 Resource Book — Backtracking on Grids and Palindromes, and the API Gateway

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 65 Resource Book](Day65_Resource_Book.md)
**Next ▶:** [Day 67 Resource Book](Day67_Resource_Book.md)
**Companion to:** Day 66 of `Week_10_Revised.md`

---

## Recap

Yesterday closed out this week's three string-building choice models (external-lookup, counter-based). Today returns to *structural* backtracking — one problem that searches a 2D grid, one that partitions a string — and both are genuinely lighter lifts than their surface difficulty suggests, because **today's first problem is almost entirely a recap.** Week 9, Day 61 built Word Search II (LC 212): a Trie combined with grid backtracking, using an overwrite-and-restore visited-marking technique explicitly flagged at the time as *"the established technique going forward for any grid-backtracking problem."* Today's Word Search (LC 79) is the single-word version of exactly that problem, minus the Trie. On the platform side, today assembles the single entry point every request will pass through: the Gateway.

---

## Learning Objectives

By the end of today, without notes:

1. Solve Word Search by recognizing it as Day 61's Word Search II with the Trie removed — not as a new mechanism to learn from scratch.
2. Solve Palindrome Partitioning, explaining why its backtracking-over-partition-points shape is structurally closer to Combinations' forward-index idea than to Word Search's grid-DFS.
3. Explain precisely why a Gateway centralizes routing (and, in later days, auth and rate limiting) instead of duplicating that logic inside every module.
4. Configure Spring Cloud Gateway routes forwarding to multiple backend modules correctly.

---

## Concept Dependency Map for Today

```
Day 61 — Word Search II (Trie + matrix backtracking, overwrite-restore
          visited marking, established as the go-to grid technique)
        │
        ▼
Today, Problem 1 — Word Search (LC 79)
  Day 61's exact grid-DFS mechanism, MINUS the Trie — single word,
  not multiple; genuinely new: the complexity analysis WITHOUT a Trie's
  shared-prefix pruning

Day 63 — Forward-index recursion (never revisit an earlier position)
Day 7  — Two Pointers, opposite-ends (palindrome check)
        │
        ▼
Today, Problem 2 — Palindrome Partitioning (LC 131)
  Forward-index-over-STRING-POSITIONS (not array elements) + a
  palindrome-validity gate on each candidate substring

Day 34 — REST/HTTP, Spring Boot            Day 65 — Feign (service-to-
        │                                     service calls already exist)
        ▼                                             │
Spring Cloud Gateway (NEW) — single entry point,       │
  centralizes routing that would otherwise be          │
  duplicated inside every module                       ◀──────────────────┘
```

---

# Part 1 — Word Search (LeetCode 79, Medium) — Pattern: Matrix Backtracking

**Statement:** Given an `m × n` grid of characters and a word, return `true` if the word can be formed by a sequence of adjacent cells (horizontally or vertically neighboring), using each cell at most once per search path.

## This is a recap of a mechanism, not a new one — stated plainly

Day 61's Word Search II solved the harder version of this exact problem: find *every* word from a whole dictionary that exists in the grid, using a Trie to share exploration across words with common prefixes. That solve established, explicitly, that **marking a cell visited by temporarily overwriting its character (then restoring it when backtracking out) is the standard technique for any grid-backtracking problem going forward** — avoiding a separate `visited[][]` boolean array entirely. Today's problem needs exactly that mechanism, applied to one word instead of many, with no Trie at all.

### Approach — DFS from every cell, overwrite-and-restore

```java
public static boolean exist(char[][] board, String word) {
    int rows = board.length, cols = board[0].length;
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (dfs(board, word, r, c, 0)) {
                return true;   // found a starting cell that leads to a full match
            }
        }
    }
    return false;
}

private static boolean dfs(char[][] board, String word, int r, int c, int index) {
    if (index == word.length()) {
        return true;   // every character matched — base case
    }
    if (r < 0 || r >= board.length || c < 0 || c >= board[0].length
            || board[r][c] != word.charAt(index)) {
        return false;   // out of bounds, or this cell doesn't match the needed character
    }

    char original = board[r][c];
    board[r][c] = '#';   // mark visited — overwrite, exactly Day 61's technique

    boolean found = dfs(board, word, r + 1, c, index + 1)
                 || dfs(board, word, r - 1, c, index + 1)
                 || dfs(board, word, r, c + 1, index + 1)
                 || dfs(board, word, r, c - 1, index + 1);

    board[r][c] = original;   // restore on backtrack — same as Day 61
    return found;
}
```

**Why `'#'` (or any character guaranteed absent from valid input) works as the visited marker:** once a cell is temporarily overwritten, it can never again match `word.charAt(index)` for any subsequent character in the *same* search path (since the search only compares against the actual letters the word needs), which is exactly what "this cell is already used in the current path" needs to mean. Restoring the original character before returning is what makes the cell available again for a *different* starting position's search — without the restore, one failed search path would permanently corrupt the board for every subsequent attempt.

**Why the `||` short-circuit matters beyond just being idiomatic:** `dfs(...) || dfs(...) || ...` stops trying further directions the instant one succeeds — this isn't just style, it avoids exploring dead branches once the answer is already known, the same "stop the instant a branch is provably settled" instinct as yesterday's parentheses pruning, just applied to a boolean success signal instead of a numeric bound.

## What's genuinely new here: the complexity *without* a Trie's shared pruning

Day 61's Word Search II shared exploration across every word sharing a prefix — a Trie let the search abandon *all* remaining words at once the instant a prefix stopped matching anything in the dictionary. Today, there's exactly one word, so there's nothing to share; every search starts fresh from scratch.

**Time: O(m × n × 4ˡ)** where `l = word.length()` — for each of the `m × n` starting cells, a DFS explores up to 4 directions at each of up to `l` steps deep, giving `4ˡ` work per starting cell in the worst case. **Space: O(l)** — the recursion depth, bounded by the word's length (the board itself is mutated in place, not copied, so it contributes no additional space beyond its own existing storage).

### Trace: confirming the technique against a small board

`board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]]`, `word = "ABCCED"`:

Starting at `(0,0)='A'` (matches `word[0]`): mark `(0,0)='#'` → try `(1,0)='S'` (≠`'B'`, fail), `(0,1)='B'` (matches `word[1]`) → mark `(0,1)='#'` → try `(0,2)='C'` (matches `word[2]`) → mark `(0,2)='#'` → try `(1,2)='C'` (matches `word[3]`) → mark `(1,2)='#'` → try `(1,3)='S'`(≠`'E'`), `(1,1)='F'`(≠`'E'`), `(2,2)='E'` (matches `word[4]`) → mark → try `(2,3)='E'`(≠`'D'`), `(2,1)='D'` (matches `word[5]`, the last character) → `index == word.length()` → **`true`**. Every cell visited along this path gets restored as the recursion unwinds back up, regardless of the eventual `true`/`false` result — matches the verified expected result for this classic example.

### Common Mistakes and Edge Cases

- ⚠️ **Reaching for a separate `boolean[][] visited` array out of habit**, without recognizing the overwrite-restore technique is both simpler and already-established from Day 61 — not wrong, just an unnecessary extra structure once the established technique is reflexive.
- ⚠️ **Forgetting to restore the original character before returning** — corrupts the board for every subsequent starting-cell attempt, causing silently wrong `false` results on inputs that should return `true`.
- ⚠️ **Checking `board[r][c] != word.charAt(index)` before the bounds check** — accessing `board[r][c]` on an out-of-bounds `r`/`c` throws `ArrayIndexOutOfBoundsException`; the bounds check must short-circuit first (Java's `||`/`&&` evaluate left-to-right and stop early — the order in the `if` condition above is load-bearing, not stylistic).
- Edge case: `word` longer than `rows × cols` → impossible on a no-repeat-cell path; the search still correctly returns `false`, just after doing genuinely less work than the worst-case bound suggests, since it can never reach the base case.
- Edge case: the same letter appears many times in `board` — handled correctly regardless, since each search path's own overwrite marking is what prevents reuse, not any global letter-position bookkeeping.

> 🔗 **Backward reference:** if Day 61's Word Search II is fresh, this problem should require close to zero new reasoning — the entire value here is recognizing "I've already built this exact mechanism" rather than re-deriving grid backtracking from first principles. If it *doesn't* feel immediately familiar, that's a signal worth spending a few minutes revisiting Day 61 directly, rather than pushing ahead on a shakier foundation.

---

# Part 2 — Palindrome Partitioning (LeetCode 131, Medium) — Pattern: Backtracking

**Statement:** Given a string `s`, partition it such that every substring of the partition is a palindrome. Return all possible palindrome partitionings.

## Prerequisites, confirmed

Backtracking's forward-index idea (Day 63, Day 64) and the opposite-ends palindrome check (Day 7's Valid Palindrome). Nothing new is needed to *check* a palindrome — the new piece is applying forward-index recursion over **string positions**, where each "element" being chosen is a *substring*, not a single array entry.

## Why this is forward-index-over-positions, not grid-DFS

Today's first problem searched *outward in four directions* from a cell. This problem searches **forward only**, along a single string, exactly like Combinations' (Day 63) and Combination Sum's (Day 64) `start`-pointer discipline — never revisit an earlier position. The genuinely new piece: instead of choosing one array element at a time, each choice here is "how long is the *next* substring," gated by whether that substring is itself a palindrome.

### Approach — Try every prefix substring; recurse only into valid palindromes

```java
public static List<List<String>> partition(String s) {
    List<List<String>> result = new ArrayList<>();
    backtrack(s, 0, new ArrayList<>(), result);
    return result;
}

private static void backtrack(String s, int start, List<String> path, List<List<String>> result) {
    if (start == s.length()) {
        result.add(new ArrayList<>(path));   // reached the end — every piece so far was a valid palindrome
        return;
    }
    for (int end = start + 1; end <= s.length(); end++) {
        String candidate = s.substring(start, end);
        if (isPalindrome(candidate)) {
            path.add(candidate);                       // choose this substring
            backtrack(s, end, path, result);            // explore — start the next search right after it
            path.remove(path.size() - 1);                // un-choose
        }
        // if NOT a palindrome, simply don't recurse — the loop moves to try a longer candidate
    }
}

private static boolean isPalindrome(String str) {
    int left = 0, right = str.length() - 1;
    while (left < right) {                              // Day 7's opposite-ends check, unchanged
        if (str.charAt(left) != str.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}
```

**Why this is a direct structural cousin of Combinations/Combination Sum, worth stating explicitly:** `start` advances forward only, `end` ranges over every possible stopping point past it, and recursion picks up exactly where the chosen candidate left off (`backtrack(s, end, ...)`, never `end - 1` or earlier) — the identical "never revisit earlier ground" guarantee that made Day 63's Combinations produce each combination exactly once. Here it guarantees every *partition* is produced exactly once, with pieces always read left to right, never reordered or overlapping.

**The `if (isPalindrome(candidate))` gate is the entire new idea:** unlike Combinations (where every remaining element is always a legal next choice) or Combination Sum (where every remaining candidate is always legal, pending the sum), here a candidate substring might simply not qualify — the loop tries it, and if it fails the palindrome check, that branch is never entered at all. This is a different flavor of "prune illegal choices" than yesterday's numeric bounds check, but the same underlying idea: don't recurse into a branch that's already known to be invalid.

### Trace: `s = "aab"`

```
backtrack(start=0, path=[])
  end=1: candidate="a" → isPalindrome? yes → path=["a"]
    backtrack(start=1, path=["a"])
      end=2: candidate="a" → isPalindrome? yes → path=["a","a"]
        backtrack(start=2, path=["a","a"])
          end=3: candidate="b" → isPalindrome? yes → path=["a","a","b"]
            backtrack(start=3) → start==len(3) → ADD ["a","a","b"] ✓
      end=3: candidate="ab" → isPalindrome? "ab"≠"ba" reversed → NO → skip (no recursion)
  end=2: candidate="aa" → isPalindrome? yes → path=["aa"]
    backtrack(start=2, path=["aa"])
      end=3: candidate="b" → isPalindrome? yes → path=["aa","b"]
        backtrack(start=3) → ADD ["aa","b"] ✓
  end=3: candidate="aab" → isPalindrome? "aab"≠"baa" reversed → NO → skip
```

**Result: `[["a","a","b"], ["aa","b"]]`** — matches the known correct output for this classic input exactly, and demonstrates the gate in action: `"ab"` and `"aab"` are both tried as candidates and both correctly rejected without ever recursing into them.

### Complexity

**Time: O(n × 2ⁿ)** — there are up to `2^(n-1)` distinct ways to place cut points between `n` characters (each of the `n-1` gaps is independently either a cut or not — the same "each element gets one binary decision" counting argument named on Day 65 for Combination Sum II's tighter bound), and checking each candidate substring for palindrome-ness costs up to O(n) — giving `O(n × 2ⁿ)` overall in the worst case (a string like `"aaaa...a"`, where nearly every substring is a palindrome and nearly every partition is explored). **Space: O(n)** for recursion depth, excluding the output.

**Extension, beyond what's needed for today's required problems (skippable under time pressure):** the `isPalindrome` check above re-scans each candidate substring from scratch on every call, which is real, avoidable repeated work. A precomputed `boolean[][] isPal` table (`isPal[i][j]` = is `s[i..j]` a palindrome) filled in ahead of time via dynamic programming would turn each check into O(1) — but that specific technique (build a table where each entry reuses a smaller already-computed entry) is Dynamic Programming, formally arriving later in this series. Flagged here as a genuine, standard optimization to this exact problem, deliberately not built out yet, the same way Trees deferred House Robber III and Bit Manipulation deferred LC 421 — not an oversight, a sequencing decision.

### Common Mistakes and Edge Cases

- ⚠️ **Checking the palindrome condition on the wrong substring bounds** — `s.substring(start, end)` in Java is `end`-exclusive; recursing with `backtrack(s, end, ...)` (not `end + 1` or `end - 1`) must line up exactly with that convention, or the next search either skips a character or re-includes one.
- ⚠️ **Assuming a longer candidate is always worth trying even after a shorter prefix already failed** — this solution's loop *does* still try every length regardless (correctly — palindrome-ness at one length says nothing about another), so this is a mistake only in reasoning about the algorithm, not in the code above; worth being explicit that this is why the loop can't short-circuit the way Word Search's `||` did.
- Edge case: single-character string → trivially one partition, the string itself (every single character is its own palindrome).
- Edge case: no palindromic substrings longer than 1 anywhere → the only valid partition is every character individually.
- Edge case: the whole string itself is a palindrome → that's one valid partition among potentially several others (e.g., `"aa"` → both `["a","a"]` and `["aa"]` are valid and both appear in the output).

> 💡 **Interview Insight:** Stating explicitly that this problem is "forward-index recursion, the same discipline as Combinations, with a palindrome gate instead of an unconditional accept" — rather than presenting it as an unrelated new technique — is exactly the kind of pattern-family fluency a tier-1 interview rewards. Mentioning the DP optimization by name (even without building it, since it isn't taught yet) as a known follow-up direction is also worth doing unprompted if time allows; it signals awareness of where this problem goes next without overclaiming a technique not yet in hand.

---

# Part 3 — Spring Cloud Gateway

## Prerequisites, confirmed

REST/HTTP fundamentals (Day 34), and the multi-module platform structure that now has real, independently-callable modules (Day 62, exercised directly Day 65 via Feign).

## Why a single entry point, mechanically

Every module built so far — Product, Order, Payment, Notification — runs as its own independently-deployed service, each with its own address. Without a Gateway, any external caller (a browser, a mobile client, a third-party integration) needs to know each module's specific address directly, and any cross-cutting concern that should apply to *every* incoming request — authentication, rate limiting, logging, routing — has to be reimplemented separately inside each of those four modules. A **Gateway** is a single service every external request passes through *before* reaching any internal module; centralizing that cross-cutting logic in one place means it exists **once**, not duplicated four times (and kept in sync four times, and debugged four times when it inevitably drifts).

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product-route
          uri: http://localhost:8081
          predicates:
            - Path=/products/**
        - id: order-route
          uri: http://localhost:8082
          predicates:
            - Path=/orders/**
        - id: payment-route
          uri: http://localhost:8083
          predicates:
            - Path=/payments/**
```

**What each piece means:** a `predicate` decides *whether* a route applies to a given incoming request (here, purely by URL path prefix); the `uri` is *where* a matching request actually gets forwarded. A request to `/products/42` matches the `product-route` predicate and is forwarded to the Product module's address, with the rest of the path preserved. This is the same declarative, configuration-over-code shape as Day 40's Flyway migrations or Day 44's `docker-compose.yml` — describe the desired routing, let the framework carry it out, rather than hand-writing a dispatcher.

## How this connects to what's already built, not a fresh start

The Gateway doesn't replace anything — Product, Order, Payment, and Notification keep running exactly as before, each still directly reachable on their own ports for internal service-to-service calls (Day 65's Feign client, Order calling Product, still talks *directly* to Product's own address, not through the Gateway — the Gateway is for external traffic reaching the platform from outside, not internal module-to-module calls). What changes is that **external** traffic now has one, single, well-known entry point, with each module's routing rule declared in exactly one place.

### When to reach for a Gateway vs. exposing modules directly

| | Modules exposed directly | API Gateway |
|---|---|---|
| External caller needs to know... | Every individual module's address | One address |
| Cross-cutting concern (auth, rate limiting) implemented... | Once per module (duplicated, drifts) | Once, centrally |
| Adding a new module later | Every existing external integration point is unaffected either way | Just add one new route entry |

### Common Mistakes

- ⚠️ **Assuming the Gateway is required for *internal* service-to-service calls too.** It isn't — Day 65's Feign client bypasses the Gateway entirely, calling Product directly. Conflating "single entry point for external traffic" with "every call in the system must pass through here" is a real, common misunderstanding of what a Gateway is actually for.
- ⚠️ **Forgetting `/**` on the path predicate** — `Path=/products` (no wildcard) would match only the exact literal path `/products`, not `/products/42` or any nested path, silently failing to route most real requests.
- ⚠️ **Hardcoding each module's address instead of reading it from configuration** — the same "don't hardcode what environment-specific config should own" instinct from Day 51's Spring Profiles, now applied to routing targets rather than database URLs.

> 💡 **Interview Insight:** "Why not just let clients call each service directly?" is a near-guaranteed follow-up once a Gateway is mentioned. The strongest answer names the *duplication* cost specifically — cross-cutting logic implemented and maintained separately in every service — rather than a vaguer "it's more organized."

---

## Project Block

**Repository:** `scalable-ecommerce-platform`.

**Task:** set up Spring Cloud Gateway in front of the whole platform; configure routes forwarding `/products/**`, `/orders/**`, and `/payments/**` to their respective modules.

**Practical guidance:** verify each module's actual running port before wiring the `uri` values — a Gateway route pointed at the wrong port fails at request time (connection refused), not at Gateway startup, since the Gateway itself doesn't validate that a target is actually listening until a real request tries to reach it.

**Definition of done:** hitting the Gateway's own address at each of the three path prefixes correctly routes to and returns a response from the matching underlying module — verified by actually sending a request through the Gateway to each route, not just by the configuration file looking correct.

---

## Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** follow up on Tier C applications submitted earlier this week.

---

# Day 66 — Interview Questions

---

**1. What makes today's Word Search different from a brand-new problem, mechanism-wise?**

*Answer:* It's the exact overwrite-and-restore grid-DFS technique Day 61's Word Search II established, applied to a single word instead of a whole Trie-backed dictionary — the marking mechanism, the bounds/match checks, and the restore-on-backtrack shape are all identical; only the absence of a Trie (and the resulting complexity, since there's no shared-prefix pruning) is genuinely new.

---

**2. Why must the bounds check happen before the character-match check in Word Search's DFS?**

*Answer:* `board[r][c]` on an out-of-bounds index throws an exception; Java's short-circuit evaluation means the bounds condition must be checked first in the combined `if`, so an out-of-bounds access is never attempted before the function has already decided to return `false`.

---

**3. Why is Word Search's complexity `O(m × n × 4ˡ)` rather than something involving the dictionary size, the way Word Search II's was?**

*Answer:* There's only one target word this time, so there's no shared-prefix pruning across multiple words to account for — every one of the `m × n` starting cells independently explores up to 4 directions at each of the word's `l` characters, giving `4ˡ` work per start with no dictionary term in the bound at all.

---

**4. What determines the legal "next choice" in Palindrome Partitioning, and how does that differ from Word Search's four-directional search?**

*Answer:* Palindrome Partitioning only ever searches forward along the string (never backward or sideways), choosing how long the *next* substring is, gated by whether that candidate substring is itself a palindrome. Word Search explores outward in four directions from the current cell with no such gating condition — any adjacent unvisited cell matching the next needed character is a legal move.

---

**5. Why does `backtrack(s, end, ...)` — not `end - 1` or `end + 1` — correctly avoid both skipping and re-including characters?**

*Answer:* `s.substring(start, end)` is end-exclusive, so `end` is already the index of the first character *not* included in the just-chosen substring — recursing with exactly `end` as the next `start` picks up precisely where the previous substring left off, with no gap and no overlap.

---

**6. Why is the DP optimization for Palindrome Partitioning mentioned but not built today?**

*Answer:* Precomputing a palindrome-lookup table so each check becomes O(1) is a genuine, standard optimization to this exact problem — but it requires Dynamic Programming, which hasn't been formally taught yet in this series. It's named as a known next step, not built, the same deliberate deferral pattern used for other DP-dependent problems earlier in the series.

---

**7. Why does centralizing routing in a Gateway matter more than it might first appear — what's the actual cost being avoided?**

*Answer:* Without a Gateway, any cross-cutting concern that should apply to every external request — routing, and later authentication and rate limiting — would need to be implemented separately inside every individual module, then kept in sync across all of them as requirements change. A Gateway means that logic exists in exactly one place.

---

**8. Does Day 65's Feign call from Order to Product go through the Gateway?**

*Answer:* No — the Gateway is the entry point for external traffic reaching the platform from outside. Internal service-to-service calls, like Order's Feign client calling Product directly, continue to address each other's own ports directly and bypass the Gateway entirely.

---

**9. What actually happens if a Gateway route's path predicate is written as `Path=/products` instead of `Path=/products/**`?**

*Answer:* It would match only the exact literal path `/products` and fail to match any nested path like `/products/42` — most real requests to that module would simply not route at all, since the predicate has no wildcard to cover sub-paths.

---

**10. If a Gateway route points at the wrong port, when does that failure actually surface?**

*Answer:* At request time, not at Gateway startup — the Gateway doesn't validate that a configured target is actually reachable until a real request tries to use that route, so a misconfigured `uri` shows up as a connection failure on the first real call through it, not as a startup error.

---

## Daily Deliverable Check

- [ ] Word Search (LC 79) solved, explicitly recognized as Day 61's mechanism minus the Trie.
- [ ] Palindrome Partitioning (LC 131) solved — the forward-index-over-positions framing and the palindrome gate both stated clearly.
- [ ] Both problems pushed to `dsa-java/backtracking/`.
- [ ] Gateway routing correctly to all three modules — verified with real requests through the Gateway, not just configuration review.

---

## What Tomorrow Assumes You Already Know Cold

Day 67 closes Backtracking with Subsets II and N-Queens, and assumes every technique from this week — forward-index (Days 64, 66), duplicate-value suppression translated into the forward-index shape (Day 64), and the general choose/explore/un-choose template across four distinct choice models now — is fully reflexive, since neither of tomorrow's two problems introduces a new backtracking *mechanism*, only new applications of what's already in hand. It also assumes today's Gateway is stable and correctly routing, since tomorrow's JWT validation is implemented directly at the Gateway layer, sitting in front of the exact routing configured today.

**Next:** [Day 67 Resource Book](./Day67_Resource_Book.md) — Backtracking Capstone, and JWT at the Gateway.
