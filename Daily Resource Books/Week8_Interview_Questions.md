# Week 8 Interview Questions — Consolidated Bank

**Series:** SDE-2 Interview Prep Resource Books · [Curriculum Map](./00_Curriculum_Map.md)
**Source:** Days 50–56 · Trees closes (BST, construction, general LCA, serialization, Median of Two Sorted Arrays) · Heaps opens (mechanism through frequency/generation patterns)

This bank pulls every question from all seven of this week's daily Resource Books into one review document, numbered continuously. Section headers mark which day each block of questions came from, for tracing back to full explanations, code, and worked traces in the source day's book.

---

## Day 50 — Binary Search Trees Begin, and Kafka Consumers

**1. State the BST ordering invariant precisely — not "left is smaller," but the full rule.**

*Answer:* For every node, every value in its entire left subtree is smaller than the node's value, and every value in its entire right subtree is larger — not just the immediate children, the whole subtree in each direction.

---

**2. Why does checking only `node.left.val < node.val < node.right.val` fail to validate a BST correctly?**

*Answer:* It only checks one level down. A node can locally look fine against its immediate parent while still violating a constraint from a higher ancestor (e.g., a node in a right subtree that's smaller than the root two levels up) — a purely local check has no way to see that inherited bound.

---

**3. Why does the bounds-passing DFS for Validate BST use `Integer` instead of `int` for the min/max parameters?**

*Answer:* `Integer` can be `null`, giving a clean way to represent "no bound yet" at the root (and along the unbounded side at every level) without needing a sentinel value that might collide with a legitimate node value.

---

**4. Is a BST's O(log n) search guaranteed by the ordering rule alone?**

*Answer:* No. The ordering rule alone permits a fully skewed tree (e.g., inserting strictly increasing values) with height O(n). O(log n) requires the tree to also be balanced — a separate property a plain BST doesn't enforce on its own. Self-balancing variants like Red-Black trees (which back `TreeMap`, Day 17) exist specifically to guarantee it.

---

**5. Why does inorder traversal of a BST visit nodes in ascending order, while it has no particular meaning on a plain binary tree?**

*Answer:* Inorder visits left subtree, then the node, then right subtree. On a BST specifically, "everything in the left subtree" is guaranteed smaller and "everything in the right subtree" is guaranteed larger by the ordering invariant — so visiting left-node-right necessarily visits values in increasing order. A plain tree has no such guarantee, so the same traversal order carries no sorted meaning there.

---

**6. In Kth Smallest Element in a BST, why is a single-element `int[]` used for `count` and `result` instead of plain `int` locals?**

*Answer:* A plain local `int` can't be mutated by an inner recursive call in a way that's visible to the caller — the same limitation Diameter of Binary Tree (Day 48) hit with its running max. A single-element array (or an instance field) gives a mutable reference the recursion can update.

---

**7. What does BST Iterator's `next()` do differently from Kth Smallest's inorder traversal, mechanically?**

*Answer:* Kth Smallest runs one recursive inorder pass to completion (or until the kth element is found) in a single call. BST Iterator uses an explicit stack that mirrors what the recursive call stack would look like mid-traversal, so the "next" node can be produced one at a time across arbitrarily many separate `next()` calls, rather than all at once.

---

**8. Why is BST Iterator's `next()` described as "average O(1)," not worst-case O(1)?**

*Answer:* A single call can do up to O(h) work if the popped node has a right child with a long left spine to push. But across the iterator's entire lifetime, every node is pushed and popped exactly once — total work is O(n), which averages to O(1) per call even though individual calls vary.

---

**9. In Kafka, what exactly does "consumer group" control?**

*Answer:* It determines partition assignment and duplicate-avoidance: within one consumer group, each partition is read by exactly one consumer at a time, so instances in the same group split the work. A different, separately-named consumer group reads the entire topic independently from scratch — group identity, not the topic, determines what's "already been read."

---

**10. What specifically can go wrong with Kafka auto-commit, and when?**

*Answer:* If the consumer crashes after an offset auto-commits but before it finishes processing the message that commit covers, that message is effectively lost — on restart, the consumer resumes past an offset it never actually finished handling, with no error or exception marking the loss.

---

**11. If you switch to manual commit to fix that risk, what new requirement does it introduce?**

*Answer:* Processing must become idempotent. Manual commit fixes message loss by only committing after processing finishes, but a crash between finishing processing and committing causes the same message to be redelivered on restart — the application must handle reprocessing the same message safely.

---

**12. What mechanism does `@KafkaListener` share with `@RestController`/`@GetMapping` from Day 34?**

*Answer:* Both are wired up via reflection at startup — the framework inspects annotated methods at runtime and connects them to their trigger (an HTTP route for `@GetMapping`, a topic subscription for `@KafkaListener`) without either being called directly by name anywhere in your own code.

---

## Day 51 — BST and Construction, and Spring Cloud Config

**13. Why does Lowest Common Ancestor of a BST only need to check ONE direction at each step, unlike Validate BST?**

*Answer:* The BST ordering invariant means if both target values are smaller than the current node, the entire subtree containing both must be the left subtree — no need to also check the right subtree. Validate BST needs to bound both directions because it's confirming a global property holds everywhere, not making a single navigational decision.

---

**14. What makes LCA of a BST solvable iteratively in O(1) space, while the general-tree version typically needs recursion?**

*Answer:* The ordering invariant lets you *compute* which single direction to go at every step, with no need to search both sides. A plain tree gives no such shortcut — you have to actually search both children and see what each returns, which is naturally expressed as a recursive combine.

---

**15. State the three steps of divide-and-conquer precisely.**

*Answer:* Divide — split the problem into independent subproblems. Conquer — solve each recursively, with a base case. Combine — merge the subproblem results into the overall answer.

---

**16. Every tree DFS problem since Day 46 already followed a divide/conquer/combine shape. What's actually new about Construct Binary Tree from Preorder and Inorder?**

*Answer:* In every earlier problem, the divide step was free — the tree handed you `node.left` and `node.right` directly. Here, there's no tree yet, only two flat arrays; figuring out which elements belong to the left subtree and which to the right requires real computed work (finding the root's index in `inorder`) before recursion can even begin.

---

**17. Why is preorder's first element always the root, and why does that fact alone not tell you the subtree sizes?**

*Answer:* Preorder visits root, then left subtree, then right subtree — so the first element is always whatever root is currently being placed. But it says nothing about how many of the remaining elements belong to the left subtree vs. the right; that split has to come from a second source — inorder's root position, in this problem.

---

**18. Why must the left recursive call happen before the right one when using a shared `preorderIndex` counter?**

*Answer:* The shared counter only stays synchronized with what preorder would visit next if the recursive calls consume it in the same order preorder itself lists — root, then the entire left subtree, then the entire right subtree. Building right before left would consume index positions meant for the left subtree while building the right one, silently producing a wrong tree.

---

**19. Why does a HashMap turn Construct Binary Tree from Preorder and Inorder from O(n²) into O(n)?**

*Answer:* Without it, finding a root value's position in `inorder` requires an O(n) linear scan, repeated once per node placed — O(n²) total. Precomputing every value's index once up front makes each lookup O(1), for O(n) total across all n placements.

---

**20. Going from preorder+inorder to inorder+postorder, what two things flip, and why?**

*Answer:* The root's position flips from the front of the array (preorder lists root first) to the back (postorder lists root last), so the pointer must start at the end and walk backward. The build order flips from left-then-right to right-then-left, because reading postorder backward from the root encounters the right subtree's elements before the left subtree's.

---

**21. Why does the problem's "no duplicate values" guarantee matter for the HashMap-lookup approach to tree construction?**

*Answer:* The technique relies on a value's position in `inorder` uniquely identifying it. With duplicate values, a value could appear at more than one index, and a single HashMap entry couldn't correctly distinguish which occurrence is the actual root being placed.

---

**22. What's the practical difference between Spring profiles and Spring Cloud Config Server?**

*Answer:* Spring profiles let one service hold multiple named configuration variants, chosen at its own startup via `spring.profiles.active` — solving the single-service, multi-environment problem. Spring Cloud Config Server is a separate microservice that centrally serves configuration to many services at once, typically Git-backed and capable of live updates via `@RefreshScope` — a different tool for centrally managing config across many services, not just switching one service between environments.

---

**23. How does `@Profile` differ from a profile-specific property in `application-dev.yml`?**

*Answer:* A profile-specific property overrides a configuration *value*. `@Profile` controls whether an entire Spring *bean* is registered at all — useful for swapping a whole component (e.g., a mock vs. a real implementation) based on the active profile, not just a setting on one shared component.

---

## Day 52 — Trees Beyond BST: General LCA and Serialization, and WireMock

**24. Why can't Lowest Common Ancestor of a Binary Tree (general) use the same single-direction approach as the BST version?**

*Answer:* There's no ordering invariant on a plain tree to compute which direction a target is in — both children genuinely have to be searched, with the results combined via postorder recursion, rather than a direction being computable in O(1) at each node.

---

**25. In the general LCA solution, what does it mean when `leftResult` and `rightResult` are both non-null at some node?**

*Answer:* It means one target was found somewhere in the left subtree and the other somewhere in the right subtree — that node is exactly the point where the paths to each target diverge, which is the LCA by definition.

---

**26. What does it mean when only one side's result is non-null, and why is propagating it upward still correct?**

*Answer:* It means either both targets are found deeper within that one side (and the true LCA hasn't been identified yet, so it needs to keep propagating up for a later call to resolve), or only one target is on that side with the other elsewhere still to be found — in both cases, passing the non-null result upward is the correct action.

---

**27. Why does a bare preorder sequence like `[1,2,3]`, with no null markers, fail to uniquely determine a tree's shape?**

*Answer:* Multiple distinct tree shapes can produce the identical preorder sequence when only real nodes are recorded — for example, `2` could be `1`'s left child with `3` below it, or `2` and `3` could both be direct children of `1`. Without a second traversal (like inorder) or explicit markers for absent children, there's no way to distinguish these.

---

**28. Why does adding explicit null markers let tree serialization use only ONE traversal, where Construct Binary Tree needed two?**

*Answer:* The null markers directly record where every subtree ends, rather than requiring that boundary to be inferred from comparing two different traversal orders. The ambiguity that made two traversals necessary is exactly what the null markers remove.

---

**29. How does deserialization know when it's finished consuming one subtree and should move to the next, without tracking explicit index ranges the way Construct Binary Tree did?**

*Answer:* The recursive structure itself does this — each call consumes exactly the tokens belonging to one subtree (a value token followed recursively by its left subtree's full token sequence, then its right subtree's), because that's the order they were written in. A shared `Queue`, consumed front-to-back, naturally stays synchronized with this as long as deserialization follows the identical recursive shape serialization used to write it.

---

**30. Why is the delimiter choice load-bearing for correctness in Serialize/Deserialize Binary Tree?**

*Answer:* Without a consistent separator between every token, multi-digit or negative values couldn't be unambiguously split back apart from the raw string — the delimiter is what makes `Integer.parseInt` on each token safe regardless of a value's sign or digit count.

---

**31. What's the core difference in motivation between TestContainers and WireMock, even though both are testing tools introduced in this project?**

*Answer:* TestContainers provides a real instance of a dependency you own, because you want realistic behavior (Postgres via H2 wasn't realistic enough). WireMock provides a fake stand-in for a dependency you don't own, because you want deterministic, controllable behavior (specific failure conditions) that a real, uncontrolled external service can't reliably be forced to produce.

---

**32. Why is WireMock being introduced ahead of Resilience4j actually arriving in the plan?**

*Answer:* Testing a circuit breaker meaningfully requires reliably forcing the exact failure conditions (timeouts, repeated errors) that trip it — which isn't possible against a real, uncontrolled external dependency. Having the WireMock stubbing capability already in place means it's ready the moment Resilience4j needs it.

---

## Day 53 — Trees Capstone: The Median Fix, and Divide-and-Conquer Reviewed

**33. Why can't an efficient merge of the two arrays meet Median of Two Sorted Arrays' required O(log(min(m,n))) bound, even done optimally at O(m+n)?**

*Answer:* O(m+n) is linear in the combined size; the required bound is logarithmic in the smaller array alone — a fundamentally different growth rate. Reaching it requires not examining most of the input at all, which a merge, by definition touching every element up to the target position, structurally cannot do.

---

**34. State the two conditions a correct partition of both arrays must satisfy.**

*Answer:* A size condition — the combined left part must hold exactly `⌈(m+n)/2⌉` elements. A value condition — every element in the combined left part must be ≤ every element in the combined right part.

---

**35. Why does fixing the partition index `i` in the smaller array also fully determine `j`, the partition index in the larger array?**

*Answer:* The size condition fixes the total number of elements required in the combined left part. Once `i` elements are taken from the smaller array, the remaining slots needed to reach that total are forced — `j = halfLen - i` — leaving nothing left to search independently.

---

**36. What does it mean if `left1 > right2` during the partition search, and what's the correct response?**

*Answer:* It means the smaller array's left part reaches too far right — it contains a value larger than something that should be in the combined right part. `i` needs to shrink, so the search moves its right boundary down.

---

**37. Why does the algorithm binary search over the smaller array specifically, rather than either array?**

*Answer:* The search range is bounded by the array being searched — searching the smaller array bounds the iteration count by `log(min(m,n))`, matching the required complexity. Searching the larger array would still be correct, just slower: `O(log(max(m,n)))`.

---

**38. For an odd combined length, why is `max(left1, left2)` the median, with no need to look at either right value?**

*Answer:* The partition is sized so the combined left part has exactly one more element than the right part when the total is odd — meaning the left part's own maximum value IS the middle element of the full combined order.

---

**39. Could a real array value equal to `Integer.MIN_VALUE` or `MAX_VALUE` break the sentinel logic in the median algorithm?**

*Answer:* No — even if a genuine value equals the sentinel, treating it as behaving like -∞ or +∞ is harmless, since nothing in a valid int array can be smaller than `Integer.MIN_VALUE` or larger than `Integer.MAX_VALUE` anyway; every comparison the algorithm performs behaves identically either way.

---

**40. In one sentence: why does BST LCA get to skip checking both subtrees, while general-tree LCA can't?**

*Answer:* The BST's ordering invariant lets you compute, in O(1), which single subtree could contain both targets, so the other is never searched; a plain tree carries no such information, so both children must actually be searched and combined.

---

**41. Is general-tree LCA a "harder algorithm" than BST LCA?**

*Answer:* No — it's not a difference in sophistication. A BST simply carries extra information (the ordering invariant) that a plain tree doesn't have, and that extra information is what turns a search into a computation. Without it, searching both sides is the correct and necessary approach, not an inferior one.

---

**42. Why does Day 53 add no extra practice problem, when every other "thin" pattern that week got one?**

*Answer:* The partition-based binary search in Median of Two Sorted Arrays is a genuinely singular technique with no comparable-difficulty sibling that reinforces the exact same mechanism — reaching for a loosely-related "binary search on the answer" problem would reinforce pattern-matching on surface similarity rather than the actual technique, which does more harm than good.

---

## Day 54 — Heaps Begin, and Frequency Patterns

**43. How is a binary heap stored in memory, and what index formulas connect a node to its children and parent?**

*Answer:* As a plain array, with no pointers. For a node at index `i`: children live at `2i+1` and `2i+2`; the parent lives at `(i-1)/2` (integer division).

---

**44. What are the two guarantees a heap makes, and which one is structural versus which is about ordering?**

*Answer:* Structural: it's always a complete binary tree — every level full except possibly the last, filled left to right with no gaps. Ordering: every parent's value is ≤ (min-heap) or ≥ (max-heap) both of its children's values.

---

**45. Why is a heap's O(log n) guarantee unconditional, where a bare BST's was not?**

*Answer:* A heap's complete-binary-tree structure is enforced on every insert by construction — a new element always goes into the next array slot, with no insertion order able to produce an uneven shape. A complete tree with n nodes has height exactly ⌊log₂n⌋, always — so operations bounded by tree height are provably O(log n), with no balancing algorithm needed because no imbalance is possible in the first place. A bare BST's shape depends entirely on insertion order, with no such enforcement.

---

**46. Walk through why sift-up (heap insert) is O(log n).**

*Answer:* The new element starts at the last array slot and is compared to its parent; if it violates the heap property, it swaps with the parent and the comparison repeats one level up. Each step moves exactly one level toward the root, and the tree has at most O(log n) levels, so at most O(log n) swaps occur.

---

**47. During sift-down (remove-root), why does the LAST array element get moved to the root, rather than promoting one of the root's children directly?**

*Answer:* Moving the last element preserves the array's completeness (no gaps), which the index-math formulas for parent/child depend on. Promoting a child directly would leave a hole that breaks that structural guarantee.

---

**48. During sift-down, why swap with the SMALLER of the two children (for a min-heap), not either child?**

*Answer:* Swapping with the smaller child guarantees the heap property is satisfied between the new parent and the child that wasn't swapped (since it's ≥ the smaller one, and the smaller one is now placed above it in the correct position) — swapping with the larger child could leave the smaller child violating the property relative to its new parent.

---

**49. In Kth Largest Element in a Stream, why is a min-heap correct, when the question asks about the LARGEST elements?**

*Answer:* The heap tracks only a bounded window of the k largest values seen so far, not the whole stream. Once that window has k elements, its own smallest member is, by definition, the kth largest overall — exactly what a min-heap's O(1) peek answers directly, with O(log k) maintenance per update.

---

**50. Why can't a max-heap of the entire stream answer "what's the kth largest" as efficiently?**

*Answer:* A max-heap's peek gives the single overall maximum, not the kth-ranked value — reaching the kth position would require repeated polls (destroying the heap's other contents) or O(n) work, rather than maintaining a bounded top-k window incrementally the way the min-heap-of-size-k approach does.

---

**51. Why does Last Stone Weight's simulation always terminate, never loop indefinitely?**

*Answer:* Every round removes exactly two stones and adds back at most one, so the total stone count strictly decreases by at least one every round — the process necessarily reaches 0 or 1 remaining stones in a bounded number of rounds.

---

## Day 55 — Heaps Continue: Two-Heap and Kth-Order Patterns

**52. State precisely what Day 12's Dutch National Flag partition and Quickselect's partition share, and what differs.**

*Answer:* Both rearrange an array into ordered zones in a single pass around a pivot value. They differ in zone count (three, around fixed 0/1/2 values, for Dutch National Flag; two, around an element drawn from the array itself, for Quickselect) and in where the pivot comes from (a known constant vs. an actual array element).

---

**53. Why is Quickselect O(n) average while quicksort, built on the same partition step, is O(n log n)?**

*Answer:* Quicksort recurses into both sides of every partition (`T(n) = 2T(n/2) + O(n)`, resolving to O(n log n)). Quickselect only recurses into the one side containing the target index (`T(n) = T(n/2) + O(n)` on average), a geometric series summing to O(n) with no second branch to add a log n factor.

---

**54. What causes Quickselect's O(n²) worst case, and what's the standard mitigation?**

*Answer:* Consistently poor pivot choices (e.g., always picking the smallest or largest remaining element, which happens with a fixed last-element pivot on sorted or reverse-sorted input) shrink the search space by only one element per partition. Randomizing the pivot choice makes this worst case vanishingly unlikely rather than eliminating it, giving an expected O(n) regardless of input order.

---

**55. Why does K Closest Points to Origin need a hand-written Comparator, where earlier heap problems that week didn't?**

*Answer:* "Closeness" is a computed property of an `int[]` point, not something Java's natural integer ordering applies to directly — a custom comparison rule is required, using the same anonymous-inner-class technique introduced for a custom Comparator back on Day 35.

---

**56. Why is comparing squared distance instead of true distance both correct and preferable?**

*Answer:* Squaring and square-rooting are both monotonically increasing for non-negative numbers, so comparing squared values produces identical ordering to comparing true distances — while avoiding an unnecessary square root computation and any floating-point imprecision it could introduce.

---

**57. Why does K Closest Points to Origin need a max-heap, while Kth Largest Element in an Array's heap approach needs a min-heap?**

*Answer:* K Closest needs to efficiently find and evict the current farthest point whenever the kept set exceeds size k — a max-heap's specialty. Kth Largest needs to efficiently find and evict the current smallest of the kept top-k values — a min-heap's specialty. Both maintain a bounded window of size k; which extreme needs evicting determines which heap type is correct.

---

**58. Minimum Cost to Connect Sticks and Last Stone Weight both repeatedly combine two heap elements. Why does one use a min-heap and the other a max-heap?**

*Answer:* Minimum Cost to Connect Sticks is minimizing a total cost, where each combination's cost is paid again every time that combined value is combined further — combining the smallest values first (min-heap) minimizes how many times any value's cost gets re-counted. Last Stone Weight isn't optimizing a cost at all; it's simulating a specific process the problem defines (smash the two heaviest), which requires a max-heap purely to efficiently track the two current largest values as the rules dictate.

---

**59. In Kth Smallest Element in a Sorted Matrix's staircase count, why does moving right after a "≤ target" match count `row + 1` elements at once, rather than checking them individually?**

*Answer:* Columns are sorted ascending top-to-bottom, so if the current cell is ≤ target, every cell above it in the same column (a smaller row index, hence a smaller or equal value) is also guaranteed ≤ target — all `row + 1` of them can be counted in one step without individually checking each.

---

**60. How does the "binary search on value" approach for Kth Smallest in a Sorted Matrix connect to Weeks 4–5's Binary Search framing?**

*Answer:* It's a direct instance of "binary search on the answer" — searching over a range of candidate answer values (not array positions) using a monotonic feasibility check (count of elements ≤ candidate) rather than directly indexing into a sorted array, the same shape formalized on Day 33.

---

**61. Which of the two Kth Smallest in a Sorted Matrix approaches scales better as k approaches n², and why?**

*Answer:* The binary-search-on-value approach — its cost depends on the value range and n (via the O(n) count step), not on k at all, while the heap approach's cost grows directly with k (O(k log n) for the extractions).

---

## Day 56 — Consolidation, and Heaps: Frequency and Scheduling

**62. What two techniques does Top K Frequent Elements combine, and what does each contribute?**

*Answer:* HashMap frequency counting (Day 4/5) determines how often each element appears; a min-heap of size k (this week's recurring shape) bounds the result to the k most frequent, ordered by that computed frequency rather than the elements' own values.

---

**63. Why does the bucket-sort approach to Top K Frequent Elements reach O(n), beating the heap approach's O(n log k)?**

*Answer:* Frequency in an array of length n is bounded by n itself, so frequencies can be used directly as array indices (buckets), avoiding a heap's log-factor entirely — counting is O(n), and scanning buckets from highest frequency down is also O(n).

---

**64. When would you prefer the heap approach over bucket sort for Top K Frequent Elements, despite bucket sort's better asymptotic complexity?**

*Answer:* When the input isn't fully known up front — a streaming variant needing "current top k" at any moment fits a heap naturally, while bucket sort's fixed-size bucket array assumes the maximum possible frequency is known in advance.

---

**65. Why is brute-force testing of every integer for Ugly Number II described as structurally wasteful, not just slow?**

*Answer:* Ugly numbers thin out as integers grow — the nth ugly number is typically much larger than n — so most integers tested turn out not to be ugly at all, wasting work on candidates that were never going to contribute to the answer.

---

**66. Why does generating candidates by multiplying confirmed-ugly numbers by 2, 3, and 5 avoid that waste?**

*Answer:* Every ugly number greater than 1 is provably 2×, 3×, or 5× some smaller ugly number — so this generation process only ever produces genuinely ugly candidates, with nothing spent testing numbers that turn out not to qualify.

---

**67. Why is a min-heap specifically needed for Ugly Number II, rather than just iterating through generated candidates in the order they're produced?**

*Answer:* Multiplying one ugly number by 2, 3, and 5 doesn't produce the next ugly number in sorted order — it produces three candidates that must be weighed against every other pending candidate from every earlier ugly number. A min-heap always surfaces the smallest pending candidate next, regardless of generation order.

---

**68. Why is the `seen` set required for correctness in Ugly Number II, not just a minor optimization?**

*Answer:* The same ugly number is often reachable multiple ways (e.g., 6 = 2×3 = 3×2). Without deduplication, the heap holds duplicate entries, and popping one wastes an iteration without advancing toward a genuinely new nth value — silently under-counting the result.

---

**69. Why does Ugly Number II use `long` for heap values, given the final answer is guaranteed to fit in a 32-bit int?**

*Answer:* Intermediate candidate values generated during the search (an already-large ugly number multiplied by 2, 3, or 5) can approach or exceed `Integer.MAX_VALUE` even though the guarantee only covers the final answer — the same overflow-awareness habit established Day 10/11.

---

**70. What does Week 8's Complete Problem Inventory show for Trees and Heaps by the end of Day 56?**

*Answer:* Trees is fully closed at 15 required + 5 extra = 20 distinct total, spanning Weeks 7–8. Heaps has reached 6 of an eventual 10 required + 3 extra = 9 distinct so far, continuing into Week 9.

---

**71. What two old debts did Week 8 resolve that had been sitting open since much earlier in the plan?**

*Answer:* Median of Two Sorted Arrays, promised on the original plan's Day 21 and never delivered anywhere in its remaining days, and Quickselect, explicitly named as a future destination back on Day 12's Sort Colors.

---

**72. What does Week 9 assume is already solid before it begins?**

*Answer:* The full heap mechanism (not just the `PriorityQueue` API, but why its complexity bound is unconditional), since none of Week 9's remaining four Heap problems re-derive it. Recursion fully reflexive for two new structures — Tries and Backtracking. And `todo-api`'s Docker/Compose/testing stack stable and untouched, since Week 9 adds no further infrastructure changes there before active project work moves to the flagship platform.

---

## Index by Topic

For targeted review rather than sequential read-through:

- **BST mechanics (bounds, ordering, LCA):** Q1–5, Q13–14, Q40–41
- **Divide and conquer / tree construction:** Q15–21
- **Serialization:** Q27–30
- **General-tree LCA:** Q24–26
- **Median of Two Sorted Arrays:** Q33–39, Q42
- **Heap mechanism (array, sift-up/down, complexity):** Q43–48
- **Heap-of-size-k patterns:** Q49–50, Q55–58
- **Quickselect:** Q52–54
- **Frequency + heap combinations:** Q62–64, Q70–71
- **Ugly Number II (candidate generation):** Q65–69
- **Kth Smallest in a Sorted Matrix / binary search on value:** Q59–61
- **Kafka Consumers:** Q9–12
- **Spring Profiles / Config:** Q22–23
- **WireMock:** Q31–32
- **Week 8 status and forward-looking:** Q72
