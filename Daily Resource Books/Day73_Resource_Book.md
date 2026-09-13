# Day 73 — Graphs Capstone: Word Ladder, Floyd-Warshall, and Shortest Path Foundations

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 72 Resource Book](Day72_Resource_Book.md)
**Next ▶:** [Day 74 Resource Book](Day74_Resource_Book.md)
**Companion to:** Day 73 of `Week_11_Revised.md`

---

## Recap

Graphs sits at 12/12 required after today, closing a pattern that opened Day 68 with adjacency lists and the `visited` requirement, added 3-state coloring for directed-cycle detection Day 69, multi-source distance Day 70, topological sort Day 71, and reversed multi-source reachability Day 72. Today's two problems are deliberately the pattern's edge cases, not its center: one is BFS applied to a graph that's never actually built as a data structure (Word Ladder), and the other isn't a traversal at all (Floyd-Warshall) — closing the pattern with the two problems that best test whether "graph" was understood as an abstraction, not just an adjacency list.

---

## Learning Objectives

By the end of today, without notes:

1. Explain what it means for a graph to be "implicit," and generate Word Ladder's neighbor set on the fly without ever materializing an adjacency list.
2. State why Word Ladder is single-source BFS, explicitly distinguishing it from Day 70/72's multi-source techniques rather than blurring the two.
3. Implement Floyd-Warshall from memory, state precisely why the loop order (`k` outermost) is required for correctness, and trace the distance matrix's evolution by hand.
4. Classify every one of this week's Graph problems by what specifically it needed — shortest-path BFS, pure reachability, ordering/cycle-detection, or neither — without defaulting to a false BFS-vs-DFS binary.

---

## Concept Dependency Map

```
Day 49: BFS — queue.size() level-by-level scaffold
Day 68: Graphs — "a grid is a graph in disguise" (adjacency computed, not stored)
        │
        ▼
TODAY (Part 1): Word Ladder (LC 127)
├─ Graph is IMPLICIT — adjacency generated per-word, on demand
├─ SINGLE-source BFS (from beginWord) — not multi-source,
│  a deliberate contrast with Days 70 and 72
└─ Shortest transformation sequence = BFS, unconditionally
        │
        ▼
TODAY (Part 2): Floyd-Warshall / Find the City (LC 1334)
├─ NOT a traversal — a different algorithm family entirely
├─ All-pairs shortest path via incremental tabulation
│  (a DP-flavored preview, same spirit as Week 3's Kadane's —
│  formal Dynamic Programming isn't named until next week)
└─ Correctness REQUIRES k as the outermost loop — proven, not asserted
        │
        ▼
Graphs BFS/DFS: CLOSES at 12/12 required (+2 extra from Week 10 = 14 distinct)
```

---

# Part 1 — Word Ladder

## Problem 11: Word Ladder (LeetCode 127, Hard) — Pattern: BFS for Shortest Path (Unweighted, Implicit Graph)

**Prerequisites, confirmed:** BFS's level-by-level mechanism, `queue.size()` isolating one layer at a time (Day 49) ✅; the "shortest/minimum → BFS, unconditionally" interview signal (Day 68) ✅; `HashSet` for O(1) membership checks (Day 5) ✅.

**Statement:** given `beginWord`, `endWord`, and a `wordList`, find the length of the shortest transformation sequence from `beginWord` to `endWord`, where each step changes exactly one letter and every intermediate word must exist in `wordList`. Return 0 if no such sequence exists.

**What "implicit graph" means here, precisely:** there is no adjacency list to build ahead of time — the graph's nodes are every word in `wordList` (plus `beginWord`), and an edge exists between two words exactly when they differ in exactly one letter position. Instead of materializing every edge up front, each word's neighbors are generated on demand: for a word of length `M`, try substituting each of its `M` positions with each of 26 letters, and check whether the resulting string is in the word set. "A grid is a graph in disguise" (Day 68) generated adjacency from coordinate arithmetic; a word ladder generates adjacency from character substitution — same underlying idea, a different generation rule.

**This is single-source BFS — worth stating explicitly, not glossing over.** Unlike Day 70's Rotting Oranges (multiple sources seeded at once) or Day 72's reversed multi-source searches, there is exactly **one** starting point here: `beginWord`. The "shortest transformation sequence" question is a plain shortest-path-from-one-source question, and BFS answers it for exactly the reason Day 68 established: BFS explores in strict distance order, so the *first* time `endWord` is dequeued, the number of steps taken to reach it is guaranteed minimal.

```java
public static int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> wordSet = new HashSet<>(wordList);
    if (!wordSet.contains(endWord)) return 0;

    Queue<String> queue = new ArrayDeque<>();
    queue.offer(beginWord);
    wordSet.remove(beginWord);   // reuse the set itself as the visited structure

    int steps = 1;
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        for (int i = 0; i < levelSize; i++) {
            String word = queue.poll();
            if (word.equals(endWord)) return steps;

            char[] chars = word.toCharArray();
            for (int pos = 0; pos < chars.length; pos++) {
                char original = chars[pos];
                for (char c = 'a'; c <= 'z'; c++) {
                    if (c == original) continue;
                    chars[pos] = c;
                    String candidate = new String(chars);
                    if (wordSet.contains(candidate)) {
                        wordSet.remove(candidate);   // mark visited
                        queue.offer(candidate);
                    }
                }
                chars[pos] = original;   // restore before mutating the next position
            }
        }
        steps++;
    }
    return 0;   // queue exhausted without reaching endWord
}
```

**Brute force, for contrast:** materialize the full graph first — compare every pair of words in the list against each other (`O(N²×M)` just to build the adjacency list, since each of `N` words gets compared against `N-1` others at `O(M)` per comparison), *then* run BFS on the explicit structure. It's correct, but pays for edges that BFS may never even need to visit if `endWord` is reached early. Generating neighbors lazily, per word actually dequeued, only ever does work proportional to what BFS actually explores.

**⚠️ Common Mistake:** forgetting to restore `chars[pos]` to its `original` value before mutating the *next* position. Without the restore, the second position's substitutions would be applied to an already-mutated array from the first position's loop, silently generating garbage candidates that don't correspond to any real one-letter-away word.

**⚠️ Common Mistake:** checking `word.equals(endWord)` only when *generating* neighbors, rather than immediately upon dequeuing. Checking at generation time can return one step too many if `endWord` happens to be generated as a candidate but the check is written to only fire on the *next* iteration's dequeue.

### Worked Trace

`beginWord = "hit"`, `endWord = "cog"`, `wordList = ["hot","dot","dog","lot","log","cog"]`.

| Level (`steps` during processing) | Queue entering level | Word checked against `endWord` | New neighbors found | Queue for next level |
|---|---|---|---|---|
| 1 | `[hit]` | hit ≠ cog | hot | `[hot]` |
| 2 | `[hot]` | hot ≠ cog | dot, lot | `[dot, lot]` |
| 3 | `[dot, lot]` | dot ≠ cog; lot ≠ cog | dog (from dot), log (from lot) | `[dog, log]` |
| 4 | `[dog, log]` | dog ≠ cog; log ≠ cog | cog (from dog only — `wordSet` already had `cog` removed by the time `log` checks, since `dog` claimed it first this same level) | `[cog]` |
| 5 | `[cog]` | **cog = cog** → return `steps = 5` | — | — |

Final answer: **5** — the sequence `hit → hot → dot → dog → cog` (5 words), matching the known correct result for this exact input. Note the tie that never surfaces as a bug: `hit → hot → lot → log → cog` is an equally short alternative path, but because `cog` is claimed and removed from `wordSet` the instant `dog` reaches it, `log`'s later attempt to also claim `cog` simply finds it already gone — no double-counting, no incorrect shorter/longer result, just one valid shortest path returned, which is all the problem asks for.

**Complexity:** Time **O(M²×N)** — for each of up to `N` words processed, generating candidates costs `O(M)` positions × 26 letters × `O(M)` to build each candidate string = `O(26M²) = O(M²)` per word, across `N` words. Space **O(N×M)** — the word set and the queue each hold at most `N` words of length `M`; this is tightened from a looser `O(M²×N)` bound sometimes cited (which conflates the *total work done* across the run with the *maximum simultaneous memory footprint* — the two aren't the same quantity here).

**Edge cases:** `endWord` not in `wordList` — return 0 immediately, since no valid final step could ever exist. `beginWord` already equals `endWord` — not a case the constraints allow, but worth naming that the loop's dequeue-time check would correctly return 1 immediately if it somehow occurred. A `wordList` containing words of a different length than `beginWord` — never matches any single-substitution candidate (candidates are always generated at the same length), so they're implicitly and correctly ignored without special-casing.

**💡 Interview Insight:** the single highest-value thing to say before coding is the reframe itself — "the graph here isn't given, it's implicit; nodes are words, edges are one-letter differences, and I'll generate them on the fly rather than precompute them." That sentence signals the pattern was recognized as BFS-on-an-implicit-graph, not "some string problem." A common, reasonable follow-up: "what if you needed the actual sequence, not just its length?" — track a parent pointer (or the path itself) alongside each queued word, exactly the same modification Course Schedule → Course Schedule II made two days ago (Day 71) to go from existence-check to construction.

---

# Part 2 — Floyd-Warshall: All-Pairs Shortest Path

## Concept Card — Floyd-Warshall

**Prerequisites, confirmed:** none from the graph-traversal side — this is deliberately **not** a BFS/DFS descendant. The only real prerequisite is arrays and nested loops (Day 1–2) ✅.

**What it is:** an algorithm that computes the shortest path between **every pair** of nodes in a weighted graph simultaneously, by incrementally allowing more and more intermediate nodes into the paths it considers.

**A note on where this sits, since it's genuinely different from everything else this week:** Floyd-Warshall isn't a traversal at all — there's no queue, no stack, no `visited` array. It builds up an answer table the way Week 3's Kadane's Algorithm did: an entry is computed from smaller, already-solved entries, and reused rather than recomputed. That "build from smaller solved subproblems, reuse instead of recompute" shape is Dynamic Programming's core idea, previewed here exactly the way Kadane's previewed it back then — flagged, not yet formally named. DP itself opens next week.

**Why it beats running a single-source algorithm once per node:** if you only need shortest paths from *one* source, a single-source algorithm is the right tool (that territory opens next week with Dijkstra's). But if you need shortest paths between **every pair**, running a single-source algorithm `V` separate times costs `V × O(that algorithm's time)`. For a small enough `V`, Floyd-Warshall's direct `O(V³)` computes the same complete answer in one pass, without `V` separate re-derivations.

**Mechanism:** initialize a distance matrix `dist[i][j]` — direct edge weight if one exists, `0` on the diagonal, infinity otherwise. Then, for every possible intermediate node `k` (0 to `V-1`), for every pair `(i, j)`, check: is the path `i → k → j` shorter than the best path found for `i → j` so far? If so, update it.

```java
public static int[][] floydWarshall(int n, int[][] edges) {
    int[][] dist = new int[n][n];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE / 2);  // avoid overflow on addition
    for (int i = 0; i < n; i++) dist[i][i] = 0;

    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], w = edge[2];
        dist[u][v] = w;
        dist[v][u] = w;   // undirected; drop this line for a directed graph
    }

    // k MUST be the outermost loop — see "Why it Works" below
    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    return dist;
}
```

### Why It Works — proven, not asserted

Define `dist_k[i][j]` as the shortest path from `i` to `j` using only intermediate nodes from `{0, ..., k}`. The recurrence `dist_k[i][j] = min(dist_{k-1}[i][j], dist_{k-1}[i][k] + dist_{k-1}[k][j])` says: either the best path avoids node `k` entirely (unchanged from the previous layer), or it uses `k` exactly once, splitting into an `i→k` leg and a `k→j` leg — and a shortest path never needs to visit `k` twice, since re-visiting any node can only add distance, never remove it.

**Why the *in-place* update (a single 2D array, not a fresh copy per `k`) is still safe:** `dist_k[i][k]` is provably identical to `dist_{k-1}[i][k]` — allowing `k` itself as an available intermediate can't shorten the path *to* `k`, because that would require the path to revisit `k`, which never helps. The same holds for `dist_k[k][j]`. So even if `dist[i][k]` or `dist[k][j]` gets "touched" earlier in the same `k`-th pass, its value is already correct to read.

**Why `k` must be the *outermost* loop specifically:** the argument above depends on every pair `(i,j)` having been fully finalized for intermediate set `{0,...,k-1}` *before* any of them starts using `k` as a bridge. Making `k` outermost is exactly what guarantees that "finish an entire layer before starting the next" ordering. If `i` or `j` were outermost instead, some pairs would finish being updated using a mix of different intermediate-node completeness levels, breaking the layer-by-layer invariant the whole proof rests on. The relative order of `i` and `j` (whichever is placed second and third) doesn't matter — only `k`'s position as the outermost loop does.

**Complexity:** Time **O(V³)** — three nested loops over every node. Space **O(V²)** — the distance matrix itself.

### Worked Trace

4 cities, undirected weighted edges: `0-1: 3`, `1-2: 1`, `1-3: 4`, `2-3: 1`. Initial matrix (∞ shown as `–`):

```
     0    1    2    3
0  [ 0    3    –    – ]
1  [ 3    0    1    4 ]
2  [ –    1    0    1 ]
3  [ –    4    1    0 ]
```

**`k=0`:** node 0 connects only to node 1 — no other pair can route usefully through it. No updates.

**`k=1`:** every pair now considers routing through node 1.
- `dist[0][2] = min(–, dist[0][1]+dist[1][2]) = min(–, 3+1) = 4`
- `dist[0][3] = min(–, dist[0][1]+dist[1][3]) = min(–, 3+4) = 7`

```
     0    1    2    3
0  [ 0    3    4    7 ]
1  [ 3    0    1    4 ]
2  [ 4    1    0    1 ]
3  [ 7    4    1    0 ]
```

**`k=2`:** every pair now considers routing through node 2.
- `dist[0][3] = min(7, dist[0][2]+dist[2][3]) = min(7, 4+1) = 5` — **improves**, using the `dist[0][2]=4` that `k=1` just finished establishing.
- `dist[1][3] = min(4, dist[1][2]+dist[2][3]) = min(4, 1+1) = 2` — **improves**, the direct edge (4) was never actually the shortest path.

```
     0    1    2    3
0  [ 0    3    4    5 ]
1  [ 3    0    1    2 ]
2  [ 4    1    0    1 ]
3  [ 5    2    1    0 ]
```

**`k=3`:** every remaining pair checked through node 3 — no further improvements (every path through 3 is already longer than what's established).

**Final matrix confirmed stable.** Notice `dist[0][3]` improved twice, in two different passes (∞→7 at `k=1`, 7→5 at `k=2`) — a concrete demonstration that the algorithm keeps refining an entry as more intermediate nodes become available, exactly as the recurrence predicts, not a one-shot computation.

---

## Problem 12: Find the City With the Smallest Number of Neighbors at a Threshold Distance (LeetCode 1334, Medium) — Pattern: Floyd-Warshall

**Statement:** `n` cities, weighted (undirected) roads between some pairs, and a `distanceThreshold`. Return the city that can reach the **fewest** other cities within `distanceThreshold`; break ties by returning the city with the **greatest** index.

**Approach:** run Floyd-Warshall once to get every pairwise distance, then for each city, count how many other cities are within `distanceThreshold`, tracking the minimum (using `<=` when comparing, so that later — larger-indexed — cities win ties, since the loop visits indices in increasing order).

```java
public static int findTheCity(int n, int[][] edges, int distanceThreshold) {
    int[][] dist = floydWarshall(n, edges);   // from the Concept Card above

    int bestCity = -1, minCount = Integer.MAX_VALUE;
    for (int i = 0; i < n; i++) {
        int count = 0;
        for (int j = 0; j < n; j++) {
            if (i != j && dist[i][j] <= distanceThreshold) count++;
        }
        if (count <= minCount) {   // <= deliberately: later (larger) index wins ties
            minCount = count;
            bestCity = i;
        }
    }
    return bestCity;
}
```

**Verifying against the worked trace above** with `distanceThreshold = 4`: city 0's distances to `{1,2,3}` are `{3,4,5}` → 2 within threshold. City 1's distances `{3,1,2}` → 3 within threshold. City 2's distances `{4,1,1}` → 3 within threshold. City 3's distances `{5,2,1}` → 2 within threshold. Cities 0 and 3 tie at 2 — the smallest count — and the `<=` comparison correctly keeps overwriting `bestCity` through the tie, landing on **city 3**, the larger index.

**Complexity:** Time O(V³) (dominated by Floyd-Warshall itself) + O(V²) for the counting pass = O(V³). Space O(V²) for the distance matrix.

**Edge cases:** a city with no roads at all reachable within the threshold — its count is 0, and 0 is the smallest possible count, so an isolated city will always be a strong (often winning) candidate unless another city ties it. `distanceThreshold` smaller than every edge weight — every count is 0, and the tie-break rule alone decides the answer (the largest index, always).

**⚠️ Common Mistake:** using `<` instead of `<=` in the final comparison, which would keep the *first* (smallest-index) city on a tie instead of the required *last* (largest-index) one — a one-character bug that silently returns the wrong tie-break winner without ever throwing an error.

**💡 Interview Insight:** the honest, correct framing to say out loud is that this problem is small-`n`-friendly by construction — Floyd-Warshall's `O(V³)` is only reasonable because city counts in this problem stay small. If asked "what would you do if `n` were 10,000," the right answer is "switch to running Dijkstra's from every node instead" (next week's algorithm) rather than trying to make Floyd-Warshall scale — naming that boundary honestly is worth more than pretending one algorithm is always the right one.

---

**This closes Graphs BFS/DFS at 12/12 required — up from 9 in the original plan.** Combined with the 2 extra practice problems added in Week 10, the pattern closes at **14 distinct problems** total.

**Note on extra practice:** none added today. Word Ladder (Hard-tier) and Floyd-Warshall (explicitly, per the plan itself, "not a pattern to over-invest in") are each single, deliberately-scoped exposures rather than open patterns needing reinforcement — and Graphs as a whole is closing this week already comprehensive at 14 distinct problems, the same reasoning applied on Days 71 and 72.

---

# Part 3 — Theory Block: Graphs, Reviewed (1 hr)

**Topic:** When BFS beats DFS, and when neither is enough.

This week's problems don't split cleanly into a BFS-vs-DFS binary — a more precise classification, one sentence each:

| Problem | What it actually needed | Why |
|---|---|---|
| Course Schedule / Course Schedule II (Day 71) | Ordering / cycle-detection — neither pure shortest-path nor pure reachability | The question is "does a valid sequence exist," answered via in-degree tracking (Kahn's) or 3-state coloring, a third category distinct from both of the rows below |
| Surrounded Regions (Day 72) | Pure reachability — DFS or BFS, interchangeably | The question is only "connected to the border or not," with no distance component at all |
| Pacific Atlantic Water Flow (Day 72) | Pure reachability, reversed — DFS or BFS, interchangeably | Same as above, plus a reversed traversal direction; still no distance component |
| Word Ladder (Day 73) | BFS, specifically and non-negotiably | The question is "shortest transformation sequence" — a true minimum-steps question, exactly where BFS's level-order guarantee is required, not optional |
| Find the City (Day 73) | **Neither** — a different algorithm family entirely | All-pairs shortest path via incremental tabulation (Floyd-Warshall), not a traversal at all — no queue, no stack, no `visited` |

**🔑 Key Takeaway:** "is this a graph problem" is never the useful question — it's always true this week. The useful question is "what, precisely, am I computing" — a minimum, a yes/no reachability, a valid ordering, or an all-pairs table — because that answer, not the word "graph," determines which tool is correct.

---

## Project Block Guide (1 hr)

**Repository:** `dsa-java/graphs/`. No new feature work — confirm all 12 required Graph solutions from this week and last are present, pushed, and organized in this directory, matching the file layout established since Day 68.

**Definition of done:** all 12 solutions confirmed present and organized.

---

## Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** review 3 more engineering manager profiles at target companies — a lighter, research-only task today, appropriate for a shorter theory/project day.

---

## Day 73 — Interview Questions

**Q1. What makes Word Ladder's graph "implicit," and why generate neighbors on the fly instead of precomputing them?**
*A:* No adjacency list exists ahead of time — nodes are words, edges are one-letter differences, generated per word via character substitution. Generating on demand only does work proportional to what BFS actually explores, rather than paying to build every edge up front.

**Q2. Is Word Ladder multi-source or single-source, and why does that distinction matter here?**
*A:* Single-source — there's exactly one `beginWord`. It matters because it correctly separates this problem from Days 70 and 72's multi-source techniques, which solve a structurally different kind of question (multiple simultaneous starting points, not one).

**Q3. Why is BFS specifically required for Word Ladder, not just convenient?**
*A:* The problem asks for the *shortest* transformation sequence — a true minimum-steps question. BFS's level-by-level exploration guarantees the first time the target is dequeued, that's the minimum step count; DFS gives no such guarantee.

**Q4. State the Floyd-Warshall recurrence and explain each term.**
*A:* `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])` — either the current best path from `i` to `j` stays unchanged, or a path routing through intermediate node `k` is shorter, checked incrementally as `k` ranges over every node.

**Q5. Why must `k` be the outermost loop in Floyd-Warshall?**
*A:* Correctness requires every pair `(i,j)` to be fully finalized for intermediate set `{0,...,k-1}` before any pair starts using `k` as a bridge. Only making `k` outermost guarantees that full-layer-before-next-layer ordering; `i` and `j`'s relative order doesn't matter.

**Q6. Why is it safe to update the distance matrix in place, using a single 2D array rather than a fresh copy per `k`?**
*A:* `dist[i][k]` and `dist[k][j]` can't be improved by allowing `k` itself as an intermediate — that would require revisiting `k`, which never shortens a shortest path. So reading those values mid-pass is always safe, even if they were "touched" earlier in that same pass.

**Q7. Why is Floyd-Warshall not classified as a graph traversal, despite operating on a graph?**
*A:* There's no queue, stack, or `visited` array — it builds an answer table from smaller already-solved subproblems and reuses them, the same DP-flavored shape previewed back in Week 3's Kadane's Algorithm, not a BFS/DFS descendant.

**Q8. When would you prefer Floyd-Warshall over running a single-source shortest-path algorithm multiple times?**
*A:* When you need shortest paths between every pair of nodes and the node count is small — Floyd-Warshall computes the full all-pairs table in one `O(V³)` pass, versus `V` separate single-source runs.

**Q9. In Find the City, why does the tie-break comparison use `<=` rather than `<`?**
*A:* The rule requires returning the largest-indexed city among ties. Iterating indices in increasing order and using `<=` means a later equal count always overwrites the stored answer, correctly landing on the largest index.

**Q10. Classify this week's five DSA-block problems by what each one actually needed — not just "graph."**
*A:* Course Schedule/II: ordering and cycle-detection. Surrounded Regions and Pacific Atlantic: pure reachability (BFS or DFS interchangeably). Word Ladder: BFS specifically, for a true shortest-path question. Find the City: neither — a distinct, non-traversal algorithm family (Floyd-Warshall).

---

## Daily Deliverable Check

- [ ] Word Ladder and Find the City With the Smallest Number of Neighbors at a Threshold Distance solved, pushed — Graphs ladder complete at 12/12 required.
- [ ] BFS-vs-DFS-vs-neither reflection written, covering all 6 of this week's Graph problems.
- [ ] All 12 required Graph solutions confirmed present and organized in `dsa-java/graphs/`.
- [ ] 3 engineering manager profiles reviewed.

---

## Week 11 So Far — Interim Note

Graphs BFS/DFS is now fully closed: **12 required + 2 extra (Week 10) = 14 distinct problems**, spanning basic traversal, cycle detection (undirected and directed), topological ordering, multi-source distance, multi-source reachability (forward and reversed), single-source shortest path on an implicit graph, and all-pairs shortest path. Union-Find opens tomorrow — a genuinely new data structure, taught fully from scratch, with zero assumed carryover from Graphs beyond "connectivity questions are common" as motivation.

---

## What Tomorrow Assumes You Already Know Cold

Day 74 opens Union-Find as an entirely new structure — it assumes nothing from today's algorithms specifically, only general comfort with arrays (Day 1) and the idea, reinforced all week, that "are these connected" is a question worth a dedicated tool rather than a fresh traversal every time it's asked. Today's closing note on Floyd-Warshall's non-traversal nature is worth holding onto loosely: tomorrow introduces a structure that *also* isn't a traversal, for the same underlying reason — some questions are answered faster by a purpose-built structure than by any graph search, however cleverly seeded.

**Next:** [Day 74 Resource Book](Day74_Resource_Book.md) — Union-Find Begins, and AWS Networking Fundamentals.
