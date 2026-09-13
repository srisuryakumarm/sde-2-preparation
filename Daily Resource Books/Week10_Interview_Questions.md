# Week 10 Interview Questions — Backtracking Capstone, and Graphs Begin

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Companion to:** `Week_10_Revised.md`

All 70 questions below are pulled verbatim from each day's Resource Book (Days 64–70) into one running review document — nothing rewritten, nothing added. Use this file for spaced review across the whole week's material without reopening seven separate files.

## Contents

| Day | Topic | Questions |
|---|---|---|
| [Day 64](#day-64--combination-sum-combination-sum-ii-resilience4j) | Combination Sum, Combination Sum II, Resilience4j | 1–10 |
| [Day 65](#day-65--letter-combinations-generate-parentheses-feign-clients) | Letter Combinations, Generate Parentheses, Feign Clients | 11–20 |
| [Day 66](#day-66--word-search-palindrome-partitioning-api-gateway) | Word Search, Palindrome Partitioning, API Gateway | 21–30 |
| [Day 67](#day-67--subsets-ii-n-queens-backtracking-closes-jwt-at-the-gateway) | Subsets II, N-Queens, Backtracking closes, JWT at the Gateway | 31–40 |
| [Day 68](#day-68--graphs-open-number-of-islands-max-area-of-island-token-bucket) | Graphs open, Number of Islands, Max Area of Island, Token Bucket | 41–50 |
| [Day 69](#day-69--clone-graph-is-graph-bipartite-kafka-schema-registry) | Clone Graph, Is Graph Bipartite?, Kafka Schema Registry | 51–60 |
| [Day 70](#day-70--rotting-oranges-all-paths-source-to-target-saga-choreography) | Rotting Oranges, All Paths Source to Target, Saga Choreography | 61–70 |

---

## Day 64 — Combination Sum, Combination Sum II, Resilience4j

---
## Day 64 — Combination Sum, Combination Sum II, Resilience4j

---

**1. What's the exact code-level difference between Day 63's Combinations template and today's Combination Sum?**

*Answer:* Combinations recurses with `i + 1` (each index usable once); Combination Sum recurses with `i` (the current index remains eligible again, since repetition is allowed). Every other part of the forward-index template — the loop starting at `start`, the choose/explore/un-choose shape — is unchanged.

---

**2. Why does the `remaining < 0` check matter for correctness vs. for performance?**

*Answer:* It doesn't affect correctness — the `remaining == 0` check alone would eventually find every valid combination regardless. It matters purely for performance: without it, the algorithm keeps exploring branches that are already provably dead (every remaining candidate is positive, so an already-negative remainder can never recover), doing significant wasted work.

---

**3. In Combination Sum II, what does the condition `i > start` actually test, in your own words?**

*Answer:* Whether the current candidate is the *first* one being offered at this specific recursion depth, or a *later sibling* choice at that same depth. Only later siblings that duplicate the immediately preceding value get skipped — the first choice at any depth is always allowed, regardless of what value appeared at a shallower depth.

---

**4. Why does `i > 0` (instead of `i > start`) produce a wrong answer, concretely?**

*Answer:* At any recursion depth past the first, `start > 0`, so `i > 0` is true even when `i == start` — the legitimate first choice at that depth. If that first choice's value happens to equal `candidates[i-1]` (a value from a *shallower*, unrelated position), it gets wrongly skipped, silently dropping valid combinations that reuse a duplicate value across two genuinely different path positions — proven directly against `[1,1,2]`, target `4`, where the correct answer `[1,1,2]` is never found under this bug.

---

**5. Why must `candidates` be sorted before Combination Sum II's skip logic works?**

*Answer:* The skip check compares `candidates[i]` to `candidates[i-1]` — the immediately preceding array slot. That only reliably catches duplicate *values* if equal values are guaranteed to sit adjacent to each other, which sorting guarantees and an arbitrary input order does not.

---

**6. Contrast Combination Sum II's duplicate-skip with Permutations II's `!used[i-1]` — same idea, why different code?**

*Answer:* Both suppress a duplicate value from being chosen as an equivalent sibling branch. Permutations II uses a swap-based choice model with an explicit `used[]` array, so it checks *array membership state* (`!used[i-1]`). Combination Sum II uses a forward-index model with no `used[]` array at all — eligibility is purely "index ≥ start" — so it checks *loop position* (`i > start`) instead. The underlying idea transfers across backtracking's different choice models; the exact code does not.

---

**7. What specifically breaks without a circuit breaker, mechanically — not just "it gets slow"?**

*Answer:* A slow or failing downstream dependency causes every calling thread that hits that code path to block waiting on it. Under sustained load, a fixed-size thread pool fills up entirely with threads stuck waiting on that one dependency, leaving no threads available to handle *any* other request — including ones unrelated to the failing dependency. This is cascading failure: one dependency's slowness starves the whole service of the resources needed to do anything else.

---

**8. Name the three circuit breaker states and what each one does.**

*Answer:* CLOSED — normal operation, calls pass through, failures are counted. OPEN — failure rate crossed the threshold; the real call is skipped entirely and a fallback returns immediately, giving the dependency room to recover. HALF_OPEN — after a wait period, a small number of trial calls are allowed through to test recovery; success returns to CLOSED, continued failure returns to OPEN.

---

**9. Why does self-invocation bypass `@CircuitBreaker`, and where has this exact issue appeared before in this series?**

*Answer:* `@CircuitBreaker` is implemented via a proxy that intercepts calls to the bean from *outside* it; a method calling another method on `this` within the same class never goes through that proxy. This is the identical self-invocation limitation Day 62 established for Spring AOP and `@Transactional` — both are proxy-based cross-cutting mechanisms with the same structural blind spot.

---

**10. What does a plain try/catch with a hardcoded fallback fail to do that a circuit breaker does?**

*Answer:* A try/catch handles each individual call's failure correctly but still *attempts* the real call every single time, paying the full cost (including any timeout) on every attempt even during a sustained outage. A circuit breaker's OPEN state stops attempting the real call at all once failure is established, which is what actually prevents resource exhaustion from a sustained failure — a try/catch alone does nothing to stop that.

---

## Day 65 — Letter Combinations, Generate Parentheses, Feign Clients

---

**11. What's genuinely new about today's two backtracking problems compared to Days 61–64's?**

*Answer:* Every prior backtracking problem selected a subset or arrangement of elements from an input array. Today's problems build a new string one character at a time — Letter Combinations chooses from a small external per-digit lookup at each position; Generate Parentheses chooses based on two running counters, not a lookup or an array index at all. The choose/explore/un-choose template is unchanged; what determines the *legal choices* at each step is what's different.

---

**12. Why does Letter Combinations need an explicit empty-input guard?**

*Answer:* Without it, `backtrack("", 0, ...)` immediately satisfies the base case (`index == digits.length()`, `0 == 0`) and adds the empty string to the result, producing `[""]`. LeetCode requires `[]` for empty input — a genuinely different, required output the guard exists to produce.

---

**13. Prove that `closeCount < openCount` alone guarantees every generated parentheses string is well-formed.**

*Answer:* Every `)` appended only happens when `closeCount < openCount` held beforehand, so immediately after appending, `closeCount' ≤ openCount` — the running balance (open minus close) never goes negative at any point during construction. Since the string only stops at length `2n`, and `openCount` is capped at `n` while `closeCount` is capped at `openCount`, reaching length `2n` forces `openCount = closeCount = n` exactly — balance zero at the end. Both halves of well-formedness (never negative, ends at zero) are guaranteed by construction.

---

**14. Why is `closeCount < n` (instead of `closeCount < openCount`) a bug, not just a less-efficient version?**

*Answer:* `closeCount < n` only limits the *total* number of closing parens used; it does nothing to prevent a `)` from being placed before its matching `(`. It would happily generate an invalid prefix like `")("`. The comparison must be against `openCount` — the currently-open count — specifically because well-formedness is about the running balance at every prefix, not just the final totals.

---

**15. Why doesn't Generate Parentheses need a separate validity check on the finished string?**

*Answer:* The two build-time conditions make it structurally impossible to ever construct an invalid prefix — every character appended is proven, at the moment it's appended, to keep the running balance non-negative. Validity is guaranteed by construction, so checking afterward would be redundant.

---

**16. `StringBuilder` + `deleteCharAt` versus building a new `String` via concatenation at each step — both are "correct" backtracking. What's the actual trade-off?**

*Answer:* Both correctly explore the same recursion tree. `StringBuilder` mutates one shared structure and needs an explicit undo, avoiding Day 11's O(n²)-string-concatenation-in-a-loop cost. Building a fresh `String` via concatenation at each call needs no explicit undo (each call's string is independent) but re-pays that concatenation cost repeatedly across the recursion — correct, but strictly more expensive for no benefit here.

---

**17. Mechanically, what does `@FeignClient` actually generate, and when?**

*Answer:* At application startup, Spring scans for `@FeignClient`-annotated interfaces and, via the same runtime-reflection mechanism used for `@RestController`/`@GetMapping`, generates a real implementing class. That generated class translates each interface method call into an actual HTTP request matching the method's mapping annotations, and deserializes the response into the declared return type.

---

**18. Why is calling the platform's real Product module a meaningfully different exercise than calling a mocked stand-in?**

*Answer:* A mocked endpoint always returns a fixed canned response regardless of the actual request. The real Product module can fail in genuinely real ways — an actual not-found product, an actual network timeout if it's down, an actual deserialization mismatch if the two modules' DTOs drift — making the exercise (and its fallback path) test something real rather than a scripted stand-in.

---

**19. How does Feign's fallback differ from Resilience4j's, in scope?**

*Answer:* Resilience4j's fallback is a single method, matched by signature via reflection, guarding one specific method call. Feign's fallback is an entire class implementing the whole client interface — every method the interface declares needs a real implementation in the fallback class, not just the one expected to fail.

---

**20. Does Feign retry failed calls automatically?**

*Answer:* No — retry behavior is a separate, explicitly configured concern (often layered on via Resilience4j's own `@Retry`), not something Feign does by default just by being present.

---

## Day 66 — Word Search, Palindrome Partitioning, API Gateway

---

**21. What makes today's Word Search different from a brand-new problem, mechanism-wise?**

*Answer:* It's the exact overwrite-and-restore grid-DFS technique Day 61's Word Search II established, applied to a single word instead of a whole Trie-backed dictionary — the marking mechanism, the bounds/match checks, and the restore-on-backtrack shape are all identical; only the absence of a Trie (and the resulting complexity, since there's no shared-prefix pruning) is genuinely new.

---

**22. Why must the bounds check happen before the character-match check in Word Search's DFS?**

*Answer:* `board[r][c]` on an out-of-bounds index throws an exception; Java's short-circuit evaluation means the bounds condition must be checked first in the combined `if`, so an out-of-bounds access is never attempted before the function has already decided to return `false`.

---

**23. Why is Word Search's complexity `O(m × n × 4ˡ)` rather than something involving the dictionary size, the way Word Search II's was?**

*Answer:* There's only one target word this time, so there's no shared-prefix pruning across multiple words to account for — every one of the `m × n` starting cells independently explores up to 4 directions at each of the word's `l` characters, giving `4ˡ` work per start with no dictionary term in the bound at all.

---

**24. What determines the legal "next choice" in Palindrome Partitioning, and how does that differ from Word Search's four-directional search?**

*Answer:* Palindrome Partitioning only ever searches forward along the string (never backward or sideways), choosing how long the *next* substring is, gated by whether that candidate substring is itself a palindrome. Word Search explores outward in four directions from the current cell with no such gating condition — any adjacent unvisited cell matching the next needed character is a legal move.

---

**25. Why does `backtrack(s, end, ...)` — not `end - 1` or `end + 1` — correctly avoid both skipping and re-including characters?**

*Answer:* `s.substring(start, end)` is end-exclusive, so `end` is already the index of the first character *not* included in the just-chosen substring — recursing with exactly `end` as the next `start` picks up precisely where the previous substring left off, with no gap and no overlap.

---

**26. Why is the DP optimization for Palindrome Partitioning mentioned but not built today?**

*Answer:* Precomputing a palindrome-lookup table so each check becomes O(1) is a genuine, standard optimization to this exact problem — but it requires Dynamic Programming, which hasn't been formally taught yet in this series. It's named as a known next step, not built, the same deliberate deferral pattern used for other DP-dependent problems earlier in the series.

---

**27. Why does centralizing routing in a Gateway matter more than it might first appear — what's the actual cost being avoided?**

*Answer:* Without a Gateway, any cross-cutting concern that should apply to every external request — routing, and later authentication and rate limiting — would need to be implemented separately inside every individual module, then kept in sync across all of them as requirements change. A Gateway means that logic exists in exactly one place.

---

**28. Does Day 65's Feign call from Order to Product go through the Gateway?**

*Answer:* No — the Gateway is the entry point for external traffic reaching the platform from outside. Internal service-to-service calls, like Order's Feign client calling Product directly, continue to address each other's own ports directly and bypass the Gateway entirely.

---

**29. What actually happens if a Gateway route's path predicate is written as `Path=/products` instead of `Path=/products/**`?**

*Answer:* It would match only the exact literal path `/products` and fail to match any nested path like `/products/42` — most real requests to that module would simply not route at all, since the predicate has no wildcard to cover sub-paths.

---

**30. If a Gateway route points at the wrong port, when does that failure actually surface?**

*Answer:* At request time, not at Gateway startup — the Gateway doesn't validate that a configured target is actually reachable until a real request tries to use that route, so a misconfigured `uri` shows up as a connection failure on the first real call through it, not as a startup error.

---

## Day 67 — Subsets II, N-Queens, Backtracking closes, JWT at the Gateway

---

**31. Why does Subsets II cite Day 64's Combination Sum II rather than Week 9's Permutations II as its duplicate-skip ancestor?**

*Answer:* Subsets II uses the same include/exclude, forward-index choice model as Combination Sum II — both check `i > start` against the loop's own starting position. Permutations II uses a swap-based model with an explicit `used[]` array and a structurally different check (`!used[i-1]`). The underlying *idea* (suppress a repeated sibling choice) traces back to Permutations II conceptually, but the actual *code* Subsets II reuses is Combination Sum II's.

---

**32. In Subsets II, why is a result added to `result` on every single recursive call, rather than only at a base case?**

*Answer:* Every prefix built so far — at any depth, including the very first call with an empty path — is itself already a valid subset. There's no single "terminal" condition the way Combination Sum II's `remaining == 0` was; the entire point of the power set is that every intermediate state qualifies.

---

**33. Why does placing one queen per row eliminate the need to separately check for row conflicts in N-Queens?**

*Answer:* Since the recursion places exactly one queen in each successive row, by construction, no two queens can ever share a row — it's structurally impossible for the algorithm to place two queens in the same row in the first place, so there's nothing to check.

---

**34. Justify the `row + col` and `row - col` diagonal-identity claim — don't just state it.**

*Answer:* Moving one step along a main diagonal (top-left to bottom-right) increases both row and column by 1, so their difference (`row - col`) stays constant along that entire diagonal. Moving one step along an anti-diagonal (top-right to bottom-left) increases row by 1 while decreasing column by 1, so their sum (`row + col`) stays constant along that diagonal instead. Each diagonal is therefore uniquely identified by one fixed value of the corresponding expression.

---

**35. What's the actual cost of re-scanning placed queens instead of using the three HashSets in N-Queens?**

*Answer:* Correctness is unaffected, but each safety check degrades from O(1) to O(n) (scanning all previously placed queens), multiplying the overall time bound by an extra factor of n compared to the HashSet version.

---

**36. Name the six distinct backtracking choice models this series has now covered, across all 12 problems.**

*Answer:* Include/exclude (Subsets, Subsets II), swap-based (Permutations, Permutations II), forward-index — bare, with repetition, and with duplicate-skip (Combinations, Combination Sum, Combination Sum II), string-building via external lookup (Letter Combinations), string-building via counter-gating (Generate Parentheses), grid-DFS with overwrite-restore marking (Word Search, and forward-index-over-positions for Palindrome Partitioning), and row-by-row placement with derived O(1) constraint tracking (N-Queens).

---

**37. Why does JWT validation belong at the Gateway rather than inside each individual module?**

*Answer:* The same centralization argument as the Gateway's routing itself: without it, every module would need to independently implement and keep in sync identical signature-and-expiry validation logic. Centralizing it means an unauthenticated request is rejected once, before ever reaching any module, and every module downstream can simply trust that anything reaching it already passed authentication.

---

**38. Is what's built today "OAuth 2.0"? Why or why not?**

*Answer:* No. Today's work is custom JWT issuance and validation — this platform signs and verifies its own tokens directly. OAuth 2.0 is a separate authorization protocol governing how a token gets issued to a third party; it frequently uses JWTs as its token format, but a system can use JWTs without OAuth 2.0 at all, which is exactly what's built here.

---

**39. Why doesn't JWT validation require a database round-trip on every request?**

*Answer:* A JWT is self-contained — the claims needed for validation (identity, roles, expiry) are encoded directly inside the signed token. The Gateway can verify the signature against its own signing key and check the expiry claim purely from the token's own contents, with no need to look anything up externally.

---

**40. What specifically goes wrong if the JWT filter's `getOrder()` is set incorrectly, letting routing run first?**

*Answer:* An unauthenticated or invalid-token request would be routed to and handled by the underlying module before authentication is ever checked — defeating the entire purpose of validating at the Gateway, since the module would process the request regardless of whether the token was ever valid.

---

## Day 68 — Graphs open, Number of Islands, Max Area of Island, Token Bucket

---

**41. Why did no tree traversal in this series, from Day 46 through Day 61's Word Search II, ever need an explicit `visited` structure?**

*Answer:* A tree is, by definition, connected and acyclic, with exactly one path between any two nodes — it's structurally impossible for a tree traversal to revisit a node it's already seen. A general graph carries no such guarantee and can contain cycles, which is exactly why an explicit `visited` structure becomes mandatory starting today.

---

**42. When is an adjacency matrix actually the better choice over an adjacency list?**

*Answer:* When the graph is dense (a large fraction of all possible edges actually exist) or when checking whether one *specific* edge exists is a frequent operation — the matrix answers that in O(1) where a list requires scanning a neighbor list. For sparse graphs, the list's O(V+E) space is strictly better than the matrix's unconditional O(V²).

---

**43. Given a problem statement, what's the specific phrasing that signals BFS over DFS?**

*Answer:* "Shortest path," "minimum steps," or "fewest moves" on an unweighted graph — BFS's level-by-level exploration guarantees the first time it reaches a target is via the fewest possible edges. DFS doesn't inherently find the shortest anything; it's the right default for "does a path exist" or "explore everything reachable."

---

**44. Why is Number of Islands' DFS not backtracking, despite the recursive, explore-neighbors code shape?**

*Answer:* Backtracking requires an un-choose step specifically because it mutates one shared structure across a whole decision tree that must be restored for sibling branches. Number of Islands marks a cell visited permanently — once part of a confirmed island, there's never a scenario requiring that mark to be undone. With nothing to restore, there's no un-choose step, which is exactly what makes this plain graph traversal rather than backtracking.

---

**45. In Max Area of Island, why does the out-of-bounds/water base case return `0` and not `1`?**

*Answer:* That branch represents a cell contributing nothing to the island's area — it's either off the grid or water, neither of which is land. Returning `1` there would incorrectly count a non-land cell as part of the area, inflating every island's computed size.

---

**46. What's the direct structural connection between Max Area of Island's DFS and Day 46's Maximum Depth of Binary Tree?**

*Answer:* Both are "base case contributes a fixed value, combine recursive results from every neighbor" — `1 + max(depth(left), depth(right))` for a tree's two children versus `1 + dfs(...) + dfs(...) + dfs(...) + dfs(...)` summed over a grid cell's up to four neighbors. The recursive shape is identical; only the number of neighbors (2 fixed vs. up to 4) and the combining operation (max vs. sum) differ.

---

**47. Name the two independent guarantees a token bucket provides, and why a naive "N requests per fixed minute window" counter fails to provide one of them.**

*Answer:* Bounded burst (via capacity) and bounded long-run average rate (via refill rate). A fixed-window counter allows up to 2N requests clustered arbitrarily close together across a window boundary — N at the very end of one window and N at the very start of the next — which a continuously-refilling token bucket doesn't permit, since tokens accumulate smoothly over time rather than resetting in discrete jumps.

---

**48. Why is `synchronized` genuinely required on `tryConsume()`, not just good practice?**

*Answer:* Under concurrent requests, two threads could both read the same `currentTokens` value before either writes back its decrement, allowing more requests through than the bucket's capacity should permit — a real correctness bug (more tokens consumed than actually existed), the same category of unguarded-shared-state corruption Day 37 demonstrated directly.

---

**49. What breaks if token refill doesn't cap at `capacity`?**

*Answer:* Tokens would accumulate without bound during any sufficiently long idle period, letting a client that's been quiet burst far beyond the intended capacity the moment it resumes — defeating the bounded-burst guarantee the capacity is supposed to enforce.

---

**50. Why does a grid need no explicit adjacency list built before running DFS/BFS on it?**

*Answer:* A grid's neighbor relationship (up/down/left/right) is directly computable from a cell's own coordinates — `(r±1, c)` and `(r, c±1)` — rather than needing to be looked up from a stored structure. The grid itself already encodes the graph; there's nothing separate to construct.

---

## Day 69 — Clone Graph, Is Graph Bipartite?, Kafka Schema Registry

---

**51. Concretely, why does a plain recursive `clone(node)` with no bookkeeping infinite-loop on a graph containing a cycle?**

*Answer:* With nothing tracking which nodes are already being cloned, a cycle causes the recursion to keep rediscovering nodes it's already visited and recursing into them again — for example, cloning node 1 recurses into neighbor 2, which recurses back into neighbor 1, which recurses into 2 again, with no base case ever reached. It's a genuine non-terminating program, not just an inefficient one.

---

**52. Why must `visited.put(node, clone)` happen before the neighbor loop, not after, in Clone Graph?**

*Answer:* If the map were populated only after processing all neighbors, a cyclic path leading back to a node still mid-recursion wouldn't find it in the map yet (since that original call hasn't reached its own `put` statement), and the cycle would still cause infinite recursion. Populating immediately after creating the clone — before touching any neighbor — is what guarantees a cyclic path finds the in-progress node already mapped.

---

**53. Why is `HashMap<Node,Node>` keyed by reference, not by value, correct here?**

*Answer:* Two genuinely different original nodes could coincidentally share the same value; keying by value would incorrectly treat them as the same node and merge their clones. Reference-based keys correctly distinguish every distinct node object regardless of value collisions.

---

**54. In Is Graph Bipartite, why must every disconnected component be checked independently, via an outer loop?**

*Answer:* A single BFS/DFS from one starting node only reaches nodes within that node's own connected component. A graph with multiple disconnected pieces would leave every node outside the first component's reach completely unchecked without an outer loop iterating over every node as a potential unstarted component.

---

**55. State the precise mathematical characterization of bipartiteness that the coloring algorithm is actually checking for.**

*Answer:* A graph is bipartite if and only if it contains no cycle of odd length. The 2-coloring algorithm doesn't explicitly search for cycles — it discovers the same fact indirectly, since an odd cycle is exactly what forces two directly-connected nodes into the same color during BFS/DFS coloring.

---

**56. (Extension) How does Find Eventual Safe States' third state differ in purpose from Is Graph Bipartite's two colors?**

*Answer:* Bipartite's two colors both represent "resolved, assigned a definite category" — the check is whether two adjacent resolved nodes contradict each other. Safe States' three states include one — "currently visiting, on the active DFS path" — that is deliberately *not yet resolved*; encountering a node in that specific state is what signals a cycle has been found, a distinction two states alone can't express.

---

**57. What's the concrete failure this platform would hit without Schema Registry, and where would it actually surface?**

*Answer:* If the Order module changed its published event's shape (a renamed field, a changed type) with no registry enforcing compatibility, the Notification module — deployed and evolving independently — would either throw deserialization errors or silently misread the new shape, discovered as a runtime failure inside Notification, disconnected from its actual cause inside Order.

---

**58. Why does this specifically need Avro rather than plain JSON messages?**

*Answer:* Avro requires a schema to exist and be registered before a message can be serialized against it, which is what makes registry-enforced compatibility checking possible at all — there's a concrete schema to compare a new version against. Plain JSON carries no such enforced contract; any shape can be published, and a mismatch is only discovered when a consumer actually fails to read an expected field.

---

**59. Why keep the JPA entity and the Avro event schema as two separate things, rather than publishing the entity directly?**

*Answer:* The JPA entity is a private implementation detail of the Order module's own persistence, free to include internal fields no other service should depend on. The Avro schema is a public contract other services rely on. Publishing the entity directly would leak internal persistence details into that contract and would force every future internal refactor to also consider whether it silently breaks external consumers.

---

**60. Does the registry rejecting an incompatible schema change guarantee the change is "safe" in every sense?**

*Answer:* No — it guarantees the change satisfies the specifically configured compatibility mode (commonly backward compatibility). A change can pass that check and still represent a meaningful business-logic change worth its own review; schema compatibility and business correctness are related but distinct concerns.

---

## Day 70 — Rotting Oranges, All Paths Source to Target, Saga Choreography

---

**61. What's the one-sentence change that turns single-source BFS into multi-source BFS?**

*Answer:* Seed the queue with every starting node before the first iteration, instead of just one — the level-by-level processing mechanism (Day 49's `queue.size()` trick) is otherwise completely unchanged.

---

**62. In Rotting Oranges, why does `minutes++` sit outside the inner per-cell loop rather than inside it?**

*Answer:* `minutes` should increment once per fully-processed BFS level, not once per cell — incrementing inside the inner loop would count once per cell processed within a level, producing a far larger number than the actual elapsed time.

---

**63. Does All Paths From Source to Target's lack of a `visited` structure contradict Day 68's rule that graphs need one?**

*Answer:* No — Day 68's rule was conditioned on cycles being possible, not an unconditional mandate. This problem's input is explicitly guaranteed to be a DAG, ruling out cycles entirely, so the specific failure mode `visited` exists to prevent simply cannot occur here regardless of whether it's present.

---

**64. Beyond being unnecessary, why would adding a global `visited` set to All Paths From Source to Target actually produce a wrong answer?**

*Answer:* The goal is enumerating every distinct path to the target, and a single node can legitimately appear on multiple different valid paths. A global visited set would mark a node "used" the first time any path reached it, incorrectly blocking a different, equally valid path from revisiting that same node later — silently dropping correct answers.

---

**65. Is All Paths From Source to Target backtracking, or plain graph DFS like Day 68's Number of Islands? Justify it.**

*Answer:* It's backtracking — it maintains one shared, mutable `path` structure across the whole exploration and genuinely needs the un-choose step (`path.remove`), since a node added while exploring one neighbor must be removed before a sibling neighbor's exploration begins, or the sibling's path would incorrectly include it. This is the opposite classification from Number of Islands, which permanently marks cells with nothing ever needing restoration — two visually similar recursive graph traversals, two different correct answers to "is this backtracking."

---

**66. (Extension) What's the one structural difference between Rotting Oranges and 01 Matrix's BFS, given they share the same multi-source seeding?**

*Answer:* Rotting Oranges tracks one shared `minutes` counter incremented per level, since it only needs a single global answer. 01 Matrix computes each cell's own distance directly as `dist[source] + 1` when first reached, since every cell individually needs its own answer — no shared level counter is needed for that.

---

**67. Why can't a single database transaction span Order, Payment, and Notification directly?**

*Answer:* Each service is independently deployed with its own separate database; a database transaction can only guarantee atomicity within one database. There is no single transaction mechanism that spans multiple independent databases the way a monolith's one database could.

---

**68. Why is a distributed transaction (two-phase commit) the wrong fix here, connecting back to Day 64?**

*Answer:* Two-phase commit requires every participating service to hold resources locked and remain available for the entire commit duration — a single slow or unavailable participant blocks the whole transaction, the same resource-exhaustion cascading-failure shape Day 64's circuit breaker exists to prevent, now at the scale of an entire business transaction instead of one call.

---

**69. Why is "compensating transaction" not the same idea as "database rollback"?**

*Answer:* By the time a later step fails and triggers compensation, the earlier local transaction has already committed successfully in its own database — there's nothing left to roll back. Undoing the business effect requires a new, explicit, forward-moving operation (like marking an order cancelled), not a database mechanism reversing an unfinished transaction.

---

**70. What's the actual trade-off choreography makes compared to orchestration — not just "which is better"?**

*Answer:* Choreography needs no central coordinator, and reuses the pub/sub mechanism already in place — but the full transaction's logic ends up scattered across every participating service's own listeners rather than visible in one place, making the complete flow harder to see and reason about all at once. Orchestration centralizes that visibility at the cost of a coordinator every participant now depends on.

---

---

## How this maps to Week 10's Resource Books

| Topic | Full treatment |
|---|---|
| Combination Sum, Combination Sum II | [Day 64](./Day64_Resource_Book.md) |
| Letter Combinations, Generate Parentheses | [Day 65](./Day65_Resource_Book.md) |
| Word Search, Palindrome Partitioning | [Day 66](./Day66_Resource_Book.md) |
| Subsets II, N-Queens — **Backtracking closes (12/12)** | [Day 67](./Day67_Resource_Book.md) |
| Number of Islands, Max Area of Island — **Graphs open** | [Day 68](./Day68_Resource_Book.md) |
| Clone Graph, Is Graph Bipartite?, (ext.) Find Eventual Safe States | [Day 69](./Day69_Resource_Book.md) |
| Rotting Oranges, All Paths Source to Target, (ext.) 01 Matrix — **Week 10 Consolidation** | [Day 70](./Day70_Resource_Book.md) |

**Next:** `Week11_Interview_Questions.md` (generated alongside Week 11's daily books).
