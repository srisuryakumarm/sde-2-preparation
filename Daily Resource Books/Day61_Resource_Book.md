# Day 61 — Tries Core Complete, Backtracking Begins, and Replication Models

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 60 Resource Book](Day60_Resource_Book.md)
**Next ▶:** [Day 62 Resource Book](Day62_Resource_Book.md)
**Companion to:** Day 61 of `Week_09_Revised.md`

---

## Recap

Yesterday's two traversal styles — constrained descent (Longest Word in Dictionary) and branching descent (wildcard search) — both reappear today. Replace Words reuses constrained descent almost exactly. Word Search II combines Trie traversal with something genuinely new: a board where every cell has up to 4 neighbors, explored via choose-recurse-undo — which today's second half formally names as **Backtracking**, built on recursion (Day 8), the one prerequisite this pattern actually needs.

## Learning Objectives

By the end of today, without notes:

1. Solve Replace Words and Word Search II, and give the full accounting of why Tries needs no extra practice at closure.
2. State the general backtracking template (choose → explore → un-choose) and explain what distinguishes it from plain recursion/DFS and from dynamic programming.
3. Solve Subsets via the include/exclude decision tree, and trace it completely for a 3-element input.
4. Explain the concrete trade-off between single-leader, multi-leader, and leaderless replication, and match each to a workload that favors it.

## Concept Dependency Map

```
Trie constrained descent (Day 60, LC 720)
        │
        ▼
Replace Words (LC 648) — same descent, applied to
   dictionary-root substitution
        │
Trie construction (Day 59) + Matrix DFS shape (new)
        │
        ▼
Word Search II (LC 212) — Trie prunes a board backtrack
   that would otherwise repeat work per word
        │
        ▼
Tries CLOSE at 6/6 core (7th — Bit Manipulation pairing — deferred)
        │
        ▼
Recursion (Day 8) ──────────────┐
                                 ▼
                  NEW: Backtracking, formalized
                  choose → explore → un-choose
                                 │
                                 ▼
                     Subsets (LC 78) — include/exclude
                                 │
                                 ▼
                  🔗 Day 62–63: Permutations (swap-based),
                  Combinations (forward-index), Permutations II
                  (duplicate handling)

Independent track — System Design:
CAP Theorem (Day 59) → Consistent Hashing (Day 60) → Replication Models (today)
```

---

## Part 1 — Replace Words

### Problem 9: Replace Words (LeetCode 648, Medium) — Pattern: Trie

**Statement:** Given a dictionary of word "roots" and a sentence, replace every word in the sentence with the **shortest** root that is a prefix of it, if any root matches; otherwise leave the word unchanged.

```java
public static String replaceWords(List<String> roots, String sentence) {
    TrieNode trieRoot = new TrieNode();
    for (String root : roots) {
        TrieNode node = trieRoot;
        for (char c : root.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEndOfWord = true;
    }

    StringBuilder result = new StringBuilder();
    for (String word : sentence.split(" ")) {
        if (result.length() > 0) result.append(" ");
        result.append(shortestRoot(trieRoot, word));
    }
    return result.toString();
}

private static String shortestRoot(TrieNode trieRoot, String word) {
    TrieNode node = trieRoot;
    for (int i = 0; i < word.length(); i++) {
        int idx = word.charAt(i) - 'a';
        if (node.children[idx] == null) return word;           // no root matches at all
        node = node.children[idx];
        if (node.isEndOfWord) return word.substring(0, i + 1);  // shortest match found
    }
    return word; // walked the whole word, never hit isEndOfWord
}
```

**Why stopping at the *first* `isEndOfWord` hit is correct, not just convenient:** every character walked deeper down the same Trie path is, by construction, a strictly longer string. The first `isEndOfWord` encountered while walking left-to-right is therefore necessarily the **shortest** root that's a prefix of `word` — there's no shorter one to miss, since anything shorter would have to be a *different* path, not a continuation of this one.

### Complexity

**Time: O(D + S)** where `D` = total characters across all roots (Trie build) and `S` = total characters across the sentence (each word walked once, character by character, bounded by that word's own length). **Space: O(D)** for the Trie.

### Edge cases

- A word with no matching root at all — walked to completion or to a missing child, correctly returned unchanged.
- Multiple roots where one is a prefix of another (e.g., `"cat"` and `"catalog"` both as roots, word `"cats"`) — the shorter root (`"cat"`) is hit first during the walk and wins, matching the problem's "shortest root" requirement exactly.
- Empty sentence or empty root list — degenerate but handled without special-casing by the loops as written.

### Interview framing

**Say before coding:** "I'll build a Trie of the dictionary roots, then for each sentence word walk the Trie character by character — the moment I hit a node marked as a complete root, that's the shortest matching root, so I substitute immediately and stop."
**Justify unprompted:** why the walk order guarantees shortest-first — interviewers will often ask this even if you get the code right, since it's the actual correctness argument, not just an implementation detail.

---

## Part 2 — Word Search II

### Problem 10: Word Search II (LeetCode 212, Hard) — Pattern: Trie + Matrix Backtracking

**Statement:** Given an `m × n` board of letters and a list of words, return every word from the list that can be traced as a path of adjacent cells (up/down/left/right), never reusing a cell within one word's path.

### Approach 1 — Brute force: search each word independently

For each of `W` words, run a full DFS/backtracking search over the board from every cell. Cost: **O(W × m × n × 4ˡ)** where `l` is average word length — every word re-walks the board from scratch, redoing shared work whenever words share prefixes.

### Approach 2 — Optimized: one Trie, one board pass

```java
public static List<String> findWords(char[][] board, String[] words) {
    TrieNode root = new TrieNode();
    for (String word : words) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.word = word; // store the full word at its end node, instead of a boolean flag
    }

    List<String> result = new ArrayList<>();
    int rows = board.length, cols = board[0].length;
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            dfs(board, r, c, root, result);
        }
    }
    return result;
}

private static void dfs(char[][] board, int r, int c, TrieNode node, List<String> result) {
    if (r < 0 || r >= board.length || c < 0 || c >= board[0].length || board[r][c] == '#') return;
    char ch = board[r][c];
    TrieNode child = node.children[ch - 'a'];
    if (child == null) return; // prune: no remaining word shares this prefix

    if (child.word != null) {
        result.add(child.word);
        child.word = null; // avoid adding the same word twice
    }

    board[r][c] = '#';           // mark visited by overwriting, no extra visited-set needed
    dfs(board, r + 1, c, child, result);
    dfs(board, r - 1, c, child, result);
    dfs(board, r, c + 1, child, result);
    dfs(board, r, c - 1, child, result);
    board[r][c] = ch;            // undo — the backtracking step
}
```

**Why one Trie beats one search per word — the actual mechanism, not just "it's shared":** every DFS call checks `node.children[ch - 'a']` — if no remaining candidate word shares the prefix built so far, that branch dies immediately, in `O(1)`, regardless of how many words were in the original list. Words that *share* a prefix (e.g., `"oath"` and `"oats"`) walk the *same* Trie nodes for their shared portion, meaning that shared exploration happens exactly once for the whole board pass, not once per word.

**Marking visited by overwriting the cell (`'#'`), then restoring it, instead of a separate `boolean[][] visited` array:** saves `O(m×n)` auxiliary space and avoids a whole class of bugs where a visited-array reset is forgotten on one exit path — the restore (`board[r][c] = ch`) at the end of `dfs` is the backtracking "undo" step, structurally identical to what Part 3 formalizes below.

> ⚠️ **Common Mistake:** forgetting to set `child.word = null` after adding it to `result`. Without this, if the *same* board path could somehow be re-reached (it structurally can't in this exact traversal, but a near-miss version of this code sometimes calls `dfs` from multiple valid starting cells for the same word), the word could be added twice. Setting it to `null` immediately is a cheap, unconditionally-safe habit.

### Complexity

Using this problem's own conventional notation: `m×n` = board dimensions, `l` = maximum word length, `k` = number of words.

**Time: O(m × n × 4ˡ)** — a DFS/backtracking pass from every cell, each path bounded in depth by the Trie (pruned the instant no word shares the current prefix), with up to 4 directions explored per step. (The true per-step branching factor is ≤4 only on the very first move from a cell and ≤3 afterward, since the just-visited cell itself is excluded — a minor refinement worth mentioning if pushed, though the loose 4ˡ bound is what's conventionally quoted.)
**Space: O(k × l)** for the Trie holding all `k` words of length up to `l`.

### Edge cases

- Two words share a full prefix except the last character (e.g., `"oath"` / `"oati"`) — both correctly found independently once the Trie branches at the differing character.
- A word longer than any path the board can offer — the DFS naturally dead-ends when `child` is `null` at some depth, no special length check needed.
- The same word appears twice in the input list — inserted once into the Trie (repeated insert is idempotent), found and added to `result` once.

### Worked trace (mechanism only, on a small board)

`board = [["o","a"],["e","t"]]`, `words = ["oa","oe","at"]`. Trie: `o→a` (word="oa"), `o→e` (word="oe"), `a→t` (word="at").

DFS from `(0,0)='o'`: `root.children['o']` exists → descend. Not a word yet. Mark `(0,0)='#'`. Try neighbors: `(1,0)='e'` → `node.children['e']` exists (from `o→e`) → descend, `child.word="oe"` → add `"oe"`, clear it. `(0,1)='a'` → `node.children['a']` exists (from `o→a`) → descend, `child.word="oa"` → add `"oa"`, clear it. Restore `(0,0)='o'`.

DFS from `(0,1)='a'`: `root.children['a']` exists → descend. Not a word yet. Mark `(0,1)='#'`. Try neighbor `(1,1)='t'` → `child.word="at"` → add `"at"`. Restore.

Result: `["oe", "oa", "at"]` (order may vary by traversal) — all three found, each via a single shared board pass, with the `oa`/`oe` shared prefix (`o→`) walked only once from `(0,0)`.

### Interview framing

**Say before coding:** "Rather than searching the board once per word, I'll build one Trie of all the words and do a single backtracking pass over the board — the Trie lets me prune a path the instant no remaining word could match, and words sharing a prefix share that exploration too."
**Likely follow-up:** "How would you avoid duplicate results if a word could be reached two different ways?" → the `child.word = null` clear-on-find, already in the code, handles this directly.
**Extension, if time allows (mark as extension, not core):** after fully exploring from a node, if it has no remaining children and isn't itself a pending word, that node can be pruned from the Trie entirely — shrinking the search space for the *rest* of the board scan. Skippable under time pressure; not needed to pass, but a well-known LC 212 optimization worth naming if asked "can you make this faster."

---

## Tries Close: 6/6 Core Required — Why No Extra Practice

Tries opened Day 59 at 0/6 (per this series' established convention, no extras land on a pattern's true opening day) and close today at 6/6: `insert`/`search`/`startsWith`, cumulative-value accumulation, constrained-descent word validation, wildcard branching search, dictionary-root substitution, and Trie-pruned matrix backtracking. The 7th canonical Trie problem (Maximum XOR of Two Numbers in an Array) is deliberately deferred to pair with Bit Manipulation later, exactly as the plan specifies — not an oversight, a scheduled dependency.

No extra practice is added, for two independent reasons that both have to hold, and both do:

1. **The required six already span every major Trie role** an SDE-2 interview draws on — basic prefix operations, value aggregation along a path, constrained traversal, branching traversal, substitution, and Trie-pruned board search. A seventh problem drawn from this same pool would reinforce a role already demonstrated twice, not add new pattern recognition.
2. **The pattern closes too fast to ever reach a genuine "reinforcement day."** Every other pattern that received deferred extras this series (Trees, Heaps) had multiple non-opening days *after* its opening day to place them on. Tries opens Day 59 and closes today, Day 61 — the same day Backtracking opens — leaving no day that's "mid-Tries, not opening" to defer anything to.

---

## Part 3 — Backtracking, Formalized

### What it is

Backtracking explores a decision tree by making a choice, recursing into the consequences of that choice, and then explicitly **undoing** the choice before trying the next option at the same level. The general template:

```java
void backtrack(State state, List<Result> results) {
    if (isCompleteSolution(state)) {
        results.add(copyOf(state));
        return; // or don't return, if partial solutions also count — problem-dependent
    }
    for (Choice choice : choicesAvailableFrom(state)) {
        makeChoice(state, choice);       // choose
        backtrack(state, results);       // explore
        undoChoice(state, choice);       // un-choose
    }
}
```

### Why it works — and why it isn't just "recursion" or "DFS"

Backtracking *is* a form of DFS over an implicit tree of choices — but naming it separately matters because of the explicit **undo** step. Plain recursive DFS (like every tree traversal so far this series) doesn't need to undo anything, because each recursive call operates on its own independent piece of the input (a node's left or right subtree) rather than mutating shared, reused state. Backtracking typically builds up one shared, mutable structure (a path, a partial arrangement) across the *entire* tree of possibilities — so after exploring one branch fully, that shared state must be restored to exactly what it was before the branch began, or the next sibling branch would start from corrupted state.

**Distinction from Dynamic Programming, precisely:** DP applies when subproblems genuinely *overlap* and you want a count or an optimal value — memoizing avoids redoing identical work. Backtracking applies when you need to enumerate **every** valid solution (or find *any* one), and there's typically no overlapping-subproblem structure to exploit — each full path through the decision tree is usually a genuinely distinct combinatorial object, not a repeated computation. This is exactly why many backtracking problems are inherently exponential: you're enumerating a combinatorially large solution space, not computing one number.

### Prerequisites (confirmed)

- Recursion — call stack, base/recursive case (Day 8).
- **No canonical "Easy" LeetCode backtracking problem exists to ease into this.** Before any real problem: generate every binary string of length 3 by hand, on paper. At each of 3 positions, two choices (`0` or `1`); draw the resulting decision tree — 3 levels deep, 8 leaves (`000, 001, 010, ..., 111`). That tree — not any specific code — *is* the backtracking shape every problem below reduces to.

### Problem 11: Subsets (LeetCode 78, Medium) — Pattern: Backtracking

**Statement:** Given an array of distinct integers, return every possible subset (the power set).

```java
public static List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private static void backtrack(int[] nums, int index, List<Integer> path, List<List<Integer>> result) {
    if (index == nums.length) {
        result.add(new ArrayList<>(path)); // copy — path keeps mutating after this
        return;
    }
    // choice 1: exclude nums[index]
    backtrack(nums, index + 1, path, result);

    // choice 2: include nums[index]
    path.add(nums[index]);
    backtrack(nums, index + 1, path, result);
    path.remove(path.size() - 1); // undo
}
```

**Why `result.add(new ArrayList<>(path))` copies, rather than adding `path` itself:** `path` is one shared, mutable list reused across the entire recursion tree — every subsequent choice keeps appending to and removing from the *same* object. Adding `path` directly would store a reference that keeps changing after the fact, silently corrupting every previously "added" subset the moment `path` is mutated again. This is a structural requirement of backtracking with shared mutable state, not a style preference.

**Why every leaf of this tree is reached exactly once, with no possibility of a missed or duplicate subset:** at each of the `n` positions there are exactly 2 independent choices (include or exclude), made once per position, with no choice depending on what happened at any other position — this is precisely a complete binary tree of depth `n`, with `2ⁿ` leaves, one per subset, no two leaves identical since each leaf corresponds to one unique include/exclude sequence.

### Worked trace

`nums = [1, 2, 3]`.

```
                    []
          /                    \
      exclude 1              include 1
        []                      [1]
      /      \                /       \
  excl 2   incl 2          excl 2    incl 2
   []       [2]            [1]       [1,2]
   / \       / \            / \        / \
  e3 i3    e3  i3          e3  i3     e3  i3
 [] [3]  [2][2,3]        [1][1,3]  [1,2][1,2,3]
```

Eight leaves, left to right: `[]`, `[3]`, `[2]`, `[2,3]`, `[1]`, `[1,3]`, `[1,2]`, `[1,2,3]` — all 8 = 2³ subsets of a 3-element set, each appearing exactly once.

### Complexity

**Time: O(n × 2ⁿ)** — 2ⁿ leaves (subsets), each requiring up to `O(n)` to copy into the result.
**Space: O(n)** auxiliary (recursion depth and `path`'s max size), not counting the output itself.

### Edge cases

- Empty input array — a single subset, the empty set itself, correctly returned as the one and only leaf.
- All elements identical (not applicable here — problem states distinct integers, worth confirming that constraint out loud, since Day 63's Permutations II specifically handles the *non*-distinct case as a deliberate contrast).

### Interview framing

**Say before coding:** "At each element I have two independent choices — include it or don't — so I'll recurse on both branches, and copy the current path into my results whenever I've made a decision for every element."
**Likely follow-up:** "Can you do this iteratively?" → cascading approach: start with `[[]]`, and for each new number, take every subset already built and add a copy of it with the new number appended — same `O(2ⁿ)` output, avoids recursion. Worth naming as extension material; not required to pass, but shows range.
**No extra practice today**, matching this series' established convention: Backtracking is opening fresh (its actual first problem), so reinforcement is deliberately deferred rather than risking a collision with what's coming next.

---

## Part 4 — Replication Models

### The three shapes

- **Single-leader (primary-replica):** one node accepts all writes, replicating them to one or more read replicas. Simple to reason about; the leader is both a single point of failure and a write-throughput bottleneck.
- **Multi-leader:** more than one node accepts writes independently, requiring **conflict resolution** when the same data is modified concurrently on different leaders (last-write-wins, vector clocks, or application-specific merge logic).
- **Leaderless (quorum-based):** any node can accept a write; reads and writes both require agreement from a **quorum** of nodes (commonly `R + W > N`, where `N` = total replicas, `W` = nodes acknowledging a write, `R` = nodes consulted on a read) — the Dynamo-style model.

### Synchronous vs. asynchronous replication

**Synchronous:** the leader waits for a replica to confirm receipt before acknowledging the write to the client — guarantees durability (a replica genuinely has the data before you're told it's safe) at the cost of added latency (and, in a single-leader-with-one-sync-replica setup, availability risk if that replica is unreachable). **Asynchronous:** the leader acknowledges immediately and replicates in the background — fast, but a leader failure between acknowledgment and successful replication can lose the most recent write(s) entirely.

### Matching a model to a workload

A **banking ledger** needs every read to reflect the true, current balance — synchronous, single-leader (or a quorum-based leaderless design with a strict quorum) is the right default; losing or misordering a write is unacceptable, and the write throughput of a ledger is rarely the bottleneck anyway. A **social media "like" counter** can tolerate a replica being briefly stale, or even losing a handful of increments during a rare leader failure, in exchange for much higher write throughput and availability — asynchronous, leaderless (or multi-leader) is the right default there; a like count off by a few for a few seconds has no real consequence.

> 🔗 **Direct connection to Day 59's CAP Theorem:** single-leader-synchronous designs lean CP (consistency prioritized, at some availability cost during a leader outage); leaderless-asynchronous designs lean AP (availability prioritized, tolerating temporarily inconsistent reads). Today's replication-model choice and yesterday's CAP framing are the same underlying trade-off, viewed from two different angles.

### Interview framing

**When it comes up:** "design a system where two reads must never disagree" → single-leader, synchronous (or quorum with `R+W>N`) — name the durability/latency cost explicitly. "Design a system that must stay available and fast even during regional issues" → leaderless or multi-leader, asynchronous — name the eventual-consistency cost explicitly. Always state what's being traded away, not only what's being gained.

---

## Project Block (1.5 hrs)

**Repository:** `todo-api`.
**Task:** a design-note comment block in `TaskService` on where you'd route reads vs. writes with a leader/replica Postgres setup, and the "read your own writes" risk it introduces.
**Definition of done:** pushed as a design note. **This is `todo-api`'s last new task** — from tomorrow, active project work moves to the flagship platform.

**Practical guidance:** the "read your own writes" risk, concretely: a client writes to the leader, then immediately reads from a replica that hasn't yet received that write via (likely asynchronous) replication — the client's own just-written data appears to have vanished. Name at least one mitigation in the comment (e.g., routing a client's own subsequent reads to the leader for some short window after a write by that same client, or a "read-your-writes" session-consistency guarantee) even though implementing it fully is out of scope for this note.

---

## Career Block (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.
**Networking:** apply to 1 Tier B target company.

---

## Day 61 — Interview Questions

**Q1. Why does Replace Words return the *first* `isEndOfWord` node hit while walking, rather than checking all matching roots and picking the shortest?** Every character walked further down the same Trie path produces a strictly longer string; the first `isEndOfWord` hit while walking left-to-right is therefore necessarily the shortest matching root — there is no shorter match to miss on that same path.

**Q2. In Word Search II, why does using a single shared Trie beat searching the board once per word, mechanistically (not just "it's shared")?** Every DFS step checks whether any remaining candidate word shares the current prefix via `node.children[...]`; if not, that branch dies in O(1) regardless of the original word count. Words sharing a prefix (e.g., "oath"/"oats") also walk the same Trie nodes for that shared portion exactly once per board path, rather than once per word.

**Q3. Why does Word Search II mark cells visited by overwriting the board instead of a separate `visited` array?** It saves O(m×n) auxiliary space and avoids the bug class where a visited-array reset is accidentally skipped on some exit path — restoring the original character at the end of each DFS call is itself the backtracking "undo" step.

**Q4. State precisely why Tries needed no extra practice at closure today.** The required six problems already span every major Trie role (basic ops, value aggregation, constrained descent, branching descent, substitution, Trie-pruned board search); and the pattern opens and closes within 3 days, leaving no "mid-pattern, non-opening" day this series' own convention would use to place deferred extras.

**Q5. What structurally distinguishes backtracking from a plain recursive DFS, like the tree traversals from Weeks 7–8?** Backtracking explicitly undoes a choice after exploring it, because it typically mutates one shared structure across the entire decision tree; plain tree DFS recurses into independent subtrees (left/right) that never need restoring, since each recursive branch never touches the other's data.

**Q6. Why must `result.add(new ArrayList<>(path))` copy `path`, rather than adding the reference directly, in Subsets?** `path` is one shared, mutable list reused and mutated across the entire recursion tree; storing a direct reference would mean every earlier "added" subset silently changes whenever `path` is mutated again later — a copy freezes that subset's contents at the moment it was complete.

**Q7. Match a banking ledger and a social media like-counter to a replication strategy, and justify each.** A ledger needs synchronous, strongly-consistent replication (single-leader or strict quorum) since an incorrect or lost balance is unacceptable and write throughput is rarely the constraint. A like-counter favors asynchronous, leaderless or multi-leader replication, trading brief staleness or rare small losses for much higher availability and write throughput — a workload where that trade is essentially free.

---

## Daily Deliverable Check

- [ ] Replace Words and Word Search II solved — **Tries core complete at 6/6.**
- [ ] Subsets solved — Backtracking begun; full 8-leaf decision tree traced by hand for a 3-element input.
- [ ] Can state the choose/explore/un-choose template from memory and explain why the "un-choose" step exists (shared mutable state, not habit).
- [ ] Replication model comparison written (banking ledger vs. social "like" counter, 150 words).
- [ ] Design-note comment block pushed in `TaskService` — `todo-api`'s last new task.

---

## What Tomorrow Assumes You Already Know Cold

Day 62 assumes today's include/exclude decision-tree shape is solid, since Permutations introduces a *structurally different* backtracking mechanism (swap-based, not include/exclude) that Day 62's book will present as a genuinely new variant, not a repeat. It also assumes the design-note thinking from today's Project Block — reasoning about a system's shape before writing implementation code — carries directly into tomorrow's multi-module Maven skeleton, which is a design decision before it's any specific class.

**Next:** [Day 62 Resource Book](Day62_Resource_Book.md) — Backtracking Continues, and the Flagship Platform Initializes.
