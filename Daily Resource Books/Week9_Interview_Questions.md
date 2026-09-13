# Week 9 — Interview Question Bank

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**Covers:** Days 57–63 — Heaps (closing), Tries (opening and closing its core), Backtracking (opening), CAP Theorem, Consistent Hashing, Replication Models, Spring AOP, multi-module Maven.

Every question below is pulled verbatim from its day's Resource Book. Organized by day for traceability back to full worked examples, traces, and code; use this file for pure review once the individual days are solid.

---

## Day 57 — Heaps: Scheduling Patterns

**Q1. Why must the "hold out for one round" entry be re-offered *before* checking the heap's next poll, not after?** Because the very next character placed must come from what's currently the most frequent *eligible* candidate — if the held-out entry isn't back in the heap in time, the algorithm could pick a worse choice than necessary, though correctness (no two adjacent) is still preserved either way since the hold-out itself is what guarantees non-adjacency. The ordering mainly affects whether you get *a* valid answer versus a well-formed one built by the intended greedy rule.

**Q2. Derive the impossibility boundary for Reorganize String and explain why it's `(n+1)/2`, not `n/2`.** A character with `maxFreq` copies needs `maxFreq - 1` gaps of at least one other character between its copies. If `maxFreq > ⌈n/2⌉`, there aren't enough remaining characters to fill every gap. `(n+1)/2` computed with Java integer division equals `⌈n/2⌉` exactly for positive integers — `n/2` alone gives the floor, which is off by one whenever `n` is odd.

**Q3. Why is greedy-by-frequency provably correct for Reorganize String, not just a heuristic that happens to work?** Any strategy that ever defers placing the currently-most-frequent character risks that character's remaining copies clustering together with fewer future opportunities to interleave them — deferring never creates more spacing options, only fewer. Always placing the max keeps the remaining distribution as spread out as it can possibly be at every step.

**Q4. In Task Scheduler, what does the notation collision between the cooldown parameter `n` and "input size" cause, and how do you avoid it in an interview?** The problem overloads `n` for cooldown, which conflicts with the usual complexity-analysis convention of `n` for input size. State explicitly which `n` you mean when giving Big-O, or switch to a different variable name (this book uses `T` for task count) to keep the two unambiguous out loud.

**Q5. Walk through the closed-form Task Scheduler derivation.** Take the max-frequency task type, lay out `maxFreq - 1` blocks of (1 task + n cooldown slots), add one final instance of that type, add one instance of every other type tied for the max frequency (they can't all fit into earlier gaps without exceeding those gaps' capacity), and everything with lower frequency is guaranteed to fit into the resulting gaps for free. Take the max of that count against the raw task count, to cover the case with no idle time at all.

**Q6. Why does the heap-based simulation remain worth knowing even though the closed form is asymptotically better?** The closed form is a sharp, problem-specific insight that stops working the moment the problem is generalized (e.g., different cooldowns per task type, or a priority beyond pure frequency); the heap simulation is the general technique that survives those variants with minimal changes — showing both, and explaining the trade-off, is exactly what a tier-1 interview is checking for.

**Q7. What forces a "forced idle" tick in the heap simulation, and how is it represented in the code?** The max-heap is empty (nothing eligible to run right now) but the cooldown queue isn't (something is still waiting to become eligible) — `time` still advances, but no `count` is decremented and no character is appended to output, since there's genuinely nothing schedulable that tick.

---

## Day 58 — Heaps Capstone: Two-Heap and Merge Patterns

**Q1. State the two-heap invariant for Find Median from Data Stream precisely.** `lower` (max-heap) holds the smaller half of all numbers seen; `upper` (min-heap) holds the larger half; `lower.size()` equals `upper.size()` or is exactly one more; every value in `lower` is ≤ every value in `upper`.

**Q2. Why push to `lower` unconditionally rather than deciding up front which heap a new number belongs in?** It avoids comparing against a heap that might currently be empty (which would throw on `peek()`), and achieves correct placement through one unconditional push plus at most one rebalancing step — simpler to get right than branch-then-insert, same `O(log n)` cost either way.

**Q3. Given the invariant, derive `findMedian()`'s two cases.** If `lower` has one more element than `upper`, the median is `lower`'s top alone (the middle element in an odd-length stream). If they're equal in size, the median is the average of both tops (the two middle elements in an even-length stream).

**Q4. Why is Merge k Sorted Lists' comparator written with `Integer.compare(a.val, b.val)` instead of `a.val - b.val`?** Subtracting two `int`s that sit near opposite ends of the representable range can overflow and silently flip the sign, corrupting heap ordering; `Integer.compare` never has this failure mode since it doesn't compute an intermediate difference.

**Q5. Name two genuinely different algorithms that both solve Merge k Sorted Lists in O(N log k), and explain the mechanism difference.** Min-heap of current list-heads (processes one node at a time across all lists simultaneously) and divide-and-conquer pairwise merging (processes whole lists per round, halving the list count each round) — same asymptotic bound, structurally different approaches.

**Q6. Why does the heap in Merge k Sorted Lists never exceed size k, regardless of how large N is?** It holds exactly one node per still-active input list at any time — popping a node and immediately offering its successor keeps the count bounded by the number of lists, never by total node count.

**Q7. Across Weeks 8–9, name the distinct roles a heap has played, and why that variety matters for interview prep.** Min-heap for kth-largest queries, max-heap for keeping the k best, min-heap for minimizing combination cost, heap as a candidate generator, max-heap for greedy scheduling, and two-heap balance-invariant statistics plus k-way merge coordination — six genuinely different applications of one structure, which is why the pattern needs no further padding: the goal was recognizing *when* to reach for a heap and *how* to shape it, not memorizing 13 individual solutions.

---

## Day 59 — Tries Begin, and the CAP Theorem

**Q1. Why does a Trie generalize `TreeNode` rather than introduce a wholly new recursive shape?** Both are self-referential structures holding pointers to further nodes of the same type; a `TrieNode` just holds up to 26 such pointers (indexed by character) instead of 2 named ones (`left`/`right`) — the recursion pattern over the structure is identical, only the branching factor and what each edge represents changes.

**Q2. When would you choose a `HashSet<String>` over a Trie, and when the reverse?** HashSet for exact-match-only membership questions (faster, O(1) average, simpler); Trie specifically when prefix queries are needed (autocomplete, "does any word start with X"), since a HashSet has no efficient way to answer those without a full scan.

**Q3. Why does `search` need the `isEndOfWord` flag when `startsWith` doesn't?** Both walk an identical path; without the flag, there's no way to distinguish a prefix that merely happens to lead to a longer stored word (e.g., `"app"` as a prefix of stored `"apple"`) from `"app"` having actually been inserted as its own complete word.

**Q4. In Map Sum Pairs, why compute a delta rather than adding `val` directly on every insert?** The problem requires overwrite, not accumulation, semantics for a repeated key — adding directly would double-count a re-inserted key. Tracking the key's previous value and applying only the *difference* to every node on the path keeps every ancestor's accumulated sum correct after an overwrite, without needing to re-walk or recompute anything else.

**Q5. Why does the root node also need to track `sum` in Map Sum Pairs?** So `sum("")` — the empty prefix, matching every key — is answered by the exact same walk-then-return-node.sum logic as any other prefix, with no special-cased branch required.

**Q6. State the CAP theorem precisely, and explain why "pick 2 of 3" is a common oversimplification.** During a network partition, a distributed system must choose between consistency (every read reflects the latest write, or errors) and availability (every request gets a response, possibly stale). Partition tolerance isn't a real design choice for a genuinely distributed system — partitions happen regardless — so it's really "C or A during a partition," with both available when no partition is occurring, not an unconditional 2-of-3 trade-off.

**Q7. Are MongoDB and Cassandra permanently "CP" and "AP" respectively?** No — those describe their *defaults* (MongoDB's single-primary model defaults toward consistency; Cassandra's leaderless quorum model defaults toward availability), but both expose tunable consistency levels per query/operation, so treating either classification as fixed and immutable is a mistake an interviewer may specifically probe for.

---

## Day 60 — Tries Continue, and Consistent Hashing

**Q1. Why does Longest Word in Dictionary's DFS only descend into children where `isEndOfWord` is true?** That condition is precisely "the path so far is itself a complete word," which is the problem's buildability requirement — a child that exists (because a longer word passes through it) but isn't itself a complete word represents an unbuildable path and must be pruned.

**Q2. Why does processing Trie children in index order (`'a'` to `'z'`) resolve the lexicographic tie-break without an explicit final comparison?** A DFS that always tries `'a'` before `'b'` naturally visits candidate words in lexicographic order at each branch point; combined with only replacing the answer on a strictly longer match (never on a tie), the first max-length word encountered is guaranteed to be the lexicographically smallest.

**Q3. What's the actual worst-case time complexity of Design Add and Search Words' `search`, and when is the commonly-cited "O(m)" figure accurate?** Worst case is O(26^k × (m−k)) where k = number of wildcards, simplifying to O(26^m) when the entire word is wildcards. "O(m)" is only accurate for the wildcard-free fast path, where the search follows exactly one path with no branching.

**Q4. Why is a wildcard's worst-case bound rarely hit in practice, despite being technically exponential?** The bound is only tight when every node the wildcard touches has close to all 26 possible children genuinely populated; in most real dictionaries, far fewer than 26 children exist at most nodes, so branches dead-end quickly and actual performance is much better than the worst case suggests.

**Q5. Why does naive `hash(key) % N` remap nearly all keys when `N` changes, while consistent hashing remaps only a small fraction?** Modulo hashing has no locality — changing the divisor changes essentially every key's computed result simultaneously. Consistent hashing's ring locality means only the keys in the specific segment adjacent to the added/removed server are affected; every other segment's nearest-clockwise-server computation is untouched.

**Q6. What problem do virtual nodes solve in consistent hashing, and how?** Uneven server placement on the ring (a real risk with few physical servers) can give some servers disproportionately large key ranges. Hashing each physical server onto the ring multiple times under distinct virtual identifiers smooths the distribution without changing the core ceilingKey-based lookup mechanism.

---

## Day 61 — Tries Core Complete, Backtracking Begins, and Replication Models

**Q1. Why does Replace Words return the *first* `isEndOfWord` node hit while walking, rather than checking all matching roots and picking the shortest?** Every character walked further down the same Trie path produces a strictly longer string; the first `isEndOfWord` hit while walking left-to-right is therefore necessarily the shortest matching root — there is no shorter match to miss on that same path.

**Q2. In Word Search II, why does using a single shared Trie beat searching the board once per word, mechanistically (not just "it's shared")?** Every DFS step checks whether any remaining candidate word shares the current prefix via `node.children[...]`; if not, that branch dies in O(1) regardless of the original word count. Words sharing a prefix (e.g., "oath"/"oats") also walk the same Trie nodes for that shared portion exactly once per board path, rather than once per word.

**Q3. Why does Word Search II mark cells visited by overwriting the board instead of a separate `visited` array?** It saves O(m×n) auxiliary space and avoids the bug class where a visited-array reset is accidentally skipped on some exit path — restoring the original character at the end of each DFS call is itself the backtracking "undo" step.

**Q4. State precisely why Tries needed no extra practice at closure today.** The required six problems already span every major Trie role (basic ops, value aggregation, constrained descent, branching descent, substitution, Trie-pruned board search); and the pattern opens and closes within 3 days, leaving no "mid-pattern, non-opening" day this series' own convention would use to place deferred extras.

**Q5. What structurally distinguishes backtracking from a plain recursive DFS, like the tree traversals from Weeks 7–8?** Backtracking explicitly undoes a choice after exploring it, because it typically mutates one shared structure across the entire decision tree; plain tree DFS recurses into independent subtrees (left/right) that never need restoring, since each recursive branch never touches the other's data.

**Q6. Why must `result.add(new ArrayList<>(path))` copy `path`, rather than adding the reference directly, in Subsets?** `path` is one shared, mutable list reused and mutated across the entire recursion tree; storing a direct reference would mean every earlier "added" subset silently changes whenever `path` is mutated again later — a copy freezes that subset's contents at the moment it was complete.

**Q7. Match a banking ledger and a social media like-counter to a replication strategy, and justify each.** A ledger needs synchronous, strongly-consistent replication (single-leader or strict quorum) since an incorrect or lost balance is unacceptable and write throughput is rarely the constraint. A like-counter favors asynchronous, leaderless or multi-leader replication, trading brief staleness or rare small losses for much higher availability and write throughput — a workload where that trade is essentially free.

---

## Day 62 — Backtracking Continues, and the Flagship Platform Initializes

**Q1. In the swap-based Permutations approach, why is "swap back" not just cleanup — what breaks if it's omitted?** The next iteration of the same `for` loop needs the array in its pre-swap state to correctly place its *own* candidate at that position; skipping the undo leaves the array corrupted for every subsequent sibling branch at that level, producing wrong or missing permutations from that point on.

**Q2. Why does the swap-based approach use O(1) extra space where a used-array approach needs O(n)?** "Which elements are already placed" is encoded directly in the array's own layout (indices before `start` are decided, at/after are available) via swapping, rather than tracked in a separate boolean array.

**Q3. Define Aspect, Pointcut, and Advice, each in one sentence.** Aspect — the behavior being injected. Pointcut — which methods it applies to. Advice — when it runs relative to the target method (before/after/around/etc.).

**Q4. How does Spring AOP actually intercept a method call, mechanically?** Via a proxy object (JDK dynamic proxy for interfaces, CGLIB subclassing otherwise) that sits in front of the real bean; every *external* call goes through the proxy first, which runs the relevant Advice before delegating to the real method.

**Q5. Why does an `@Transactional` (or any AOP-advised) method silently lose its behavior when called from another method on the same class?** Self-invocation (`this.method()`) is a direct in-JVM call that never passes through the proxy sitting outside the object — the proxy is what runs the Advice, so bypassing it means the Advice never fires, even though the annotation is technically present.

**Q6. What's the exact difference between `<dependencies>` and `<dependencyManagement>` in a parent Maven POM?** `<dependencyManagement>` only centralizes version numbers for dependencies a child *might* use — it adds nothing automatically. A child module must still declare the dependency itself (without a version) in its own `<dependencies>` block to actually receive it on its classpath.

**Q7. Why must the parent POM use `<packaging>pom</packaging>`?** The parent holds no compiled code of its own — it exists only to declare shared configuration (modules list, dependency management) — `pom` packaging signals there's nothing to compile into a `jar`/`war` at that level.

---

## Day 63 — Consolidation, and Backtracking Continues

**Q1. Why does forward-only recursion (`i+1`, never revisiting) fully solve Combinations' duplicate-avoidance problem with no extra structure needed?** Since order doesn't matter, every valid combination has exactly one increasing-order representation; only ever choosing forward from the last pick means the recursion can only ever produce that one canonical increasing form, so no duplicate can ever arise in the first place.

**Q2. Why would forward-only recursion be the *wrong* choice for Permutations?** Order matters there — every element must eventually be tried in every position, including positions "behind" where an earlier choice landed. Forward-only recursion would silently skip every reordering, undercounting the true permutation set.

**Q3. State the Permutations II duplicate-skip condition precisely, and explain why it's `!used[i-1]`, not `used[i-1]`.** Skip candidate `i` if `nums[i] == nums[i-1]` **and** `!used[i-1]` (the previous identical value is *not* currently part of the active path). `!used[i-1]` true means we already fully explored and backtracked past using the previous copy at this exact depth — choosing this copy now would just re-explore an equivalent sibling branch. `used[i-1]` being true instead means the previous copy is actively part of the *current* path one level up, and choosing this copy now is legitimate reuse of a duplicate value within one permutation, not a forbidden repeat.

**Q4. Give the complete reasoning for why Backtracking received zero extra practice this week.** The pattern is still in its genuine opening arc with no natural non-opening reinforcement day yet; Week 10's required ladder is already dense (2 problems/day, no gaps) and comprehensively spans every canonical variant; and a specific candidate (Letter Combinations of a Phone Number) was checked against `Week_10_Revised.md` and found to already be its required Day 65 problem — confirming the overlap-check process catching a real collision, not just a theoretical safeguard.

**Q5. What's the cumulative distinct-problem count through Day 63, and how does it reconcile against the plan's own "123" figure?** 145 distinct problems through Week 8 (110 required + 35 extra across the series, per the curriculum map's running totals) plus this week's 14 newly-solved (Heaps' remaining 4 + Tries' 6 + Backtracking's 4, zero extra) — 159 distinct total. The plan's own "123" figure (Day 63's scorecard in `Week_09_Revised.md`) undercounts by one: it implies 13 required problems for Week 9, but the plan's own day-by-day list for this week sums to 14 (Day 61 alone carries three — Replace Words, Word Search II, and Subsets — one more than any other day this week, the likely source of the drift). 110 + 14 = 124 required through Week 9, and 145 + 14 = 159 distinct overall — this book uses the verified row-by-row count rather than propagating the plan's own scorecard total.

**Q6. Name the six distinct heap roles demonstrated across Weeks 8–9.** Min-heap for kth-largest queries; max-heap for keeping the k best; min-heap for minimizing combination cost; heap as a candidate generator; max-heap for greedy scheduling (Day 57); and two-heap balance-invariant statistics tracking plus heap-as-merge-coordinator (Day 58).

**Q7. Why did Tries close in exactly 3 days with zero extra practice, while Backtracking — also opening this week — gets a full 2 weeks and still adds none either?** Different reasons for the same outcome: Tries' required set (6) already spans every major role and the pattern is inherently short (no non-opening day exists within its own arc to place extras on). Backtracking's required set spans 2 full weeks and 12 problems specifically *because* it's a larger, higher-interview-weight pattern — but that larger required ladder is itself already comprehensive enough that no padding is needed; the absence of extras reflects thoroughness of the required set in both cases, via two different mechanisms (short-and-complete vs. long-and-complete).

---

**Total: 49 questions across 7 days.** Every LeetCode problem, every system-design topic, and every platform-engineering concept from Week 9 is represented above at least once. For full code, worked traces, and complexity derivations behind any answer, follow the day link at the top of that section.
