# Day 59 — Tries Begin, and the CAP Theorem

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 58 Resource Book](Day58_Resource_Book.md)
**Next ▶:** [Day 60 Resource Book](Day60_Resource_Book.md)
**Companion to:** Day 59 of `Week_09_Revised.md`

---

## Recap

Heaps closed yesterday at 10/10 required, 13/13 distinct. Today opens an entirely new structure — the Trie — built from two pieces already solid: **recursion** (Day 8, Week 2 — call stack, base/recursive case) and the **node-with-children shape** first built for `TreeNode` (Day 46, Week 7 — a self-referential structure with pointers to further nodes of the same type). A Trie is that exact shape, generalized: instead of exactly two named children (`left`, `right`), a `TrieNode` holds up to 26 — one slot per possible next character. Nothing about *recursing over a node-with-pointers structure* is new; only the branching factor and what each edge represents changes.

## Learning Objectives

By the end of today, without notes:

1. Build a `TrieNode` and implement `insert`, `search`, and `startsWith` from first principles, and explain why each runs in `O(m)` regardless of how many words are stored.
2. Explain precisely why a Trie beats a `HashSet<String>` for prefix queries specifically, while conceding where the HashSet is actually better.
3. Implement a Trie that accumulates a value per key and answers prefix-sum queries in `O(m)`, including correctly handling key overwrites.
4. State the CAP theorem precisely (not the common oversimplified version), explain why "partition tolerance" isn't really optional, and give a concrete system that would prefer each of C and A during a partition.

## Concept Dependency Map

```
Recursion (Day 8)          Node-with-pointers shape (Day 46, TreeNode)
        │                              │
        └──────────────┬───────────────┘
                        ▼
              NEW: TrieNode — up to 26 children, not 2
                        │
        ┌───────────────┴────────────────┐
        ▼                                 ▼
insert / search / startsWith      NEW: per-node accumulated value
   (LC 208)                          (LC 677 — Map Sum Pairs)
        │                                 │
        └────────────────┬────────────────┘
                          ▼
              🔗 Day 60–61: DFS-validated word
              building, wildcard search, dictionary
              substitution, matrix backtracking
                          │
                          ▼
   System Design, independent track:
   HashMap mechanism (Day 4) → CAP Theorem (today, NEW)
```

---

## Part 1 — The Trie (Prefix Tree)

### Prerequisites (confirmed)

- Recursion — call stack, base/recursive case (Day 8).
- Self-referential node structure with pointers to more nodes of the same type (Day 46, `TreeNode`).
- Array indexing, specifically mapping a character to an index via `c - 'a'` (Day 2).

### What it is, and why the shape works

A Trie (from re**trie**val, pronounced either "try" or "tree") is a tree where **each edge represents one character**, and the path from the root to any node spells out a shared prefix of every word that passes through it. Two words with a common prefix literally share the same nodes for that prefix's length — that sharing is the entire point.

```java
class TrieNode {
    TrieNode[] children = new TrieNode[26]; // index i = 'a' + i
    boolean isEndOfWord = false;
}
```

Compare this directly to `TreeNode` from Day 46: same idea (a node holding pointers to further nodes of the same type), generalized from exactly 2 named pointers (`left`, `right`) to 26 indexed ones. Recursing over it is the same *kind* of recursion you've already written — walk to a child, recurse, combine or return.

### Why not just a `HashSet<String>`?

A `HashSet<String>` answers "is this exact string present?" in O(1) average time — genuinely faster than a Trie's O(m) for that one question. But it cannot answer "does *any* stored word start with this prefix?" without a full O(n) scan of every stored string, because a hash of the *complete* string tells you nothing about partial matches. A Trie answers both questions in the same O(m) walk, because the prefix-membership question *is* "did I successfully walk this far without hitting a missing child" — no separate check needed. That's the trade Tries make: slightly slower exact-match lookups (`O(m)` vs. `O(1)` average) in exchange for prefix queries the HashSet structurally cannot do efficiently at all.

> 🔑 **Key Takeaway:** reach for a Trie specifically when a problem asks about *prefixes* — autocomplete, "does any word start with X," building words character-by-character with validity checks at each step. If the problem only ever asks about complete, exact strings, a HashSet is simpler and faster; don't reach for a Trie by default just because strings are involved.

### Problem 5: Implement Trie (Prefix Tree) (LeetCode 208, Medium) — Pattern: Trie

**Statement:** Implement `insert(String word)`, `boolean search(String word)`, and `boolean startsWith(String prefix)`.

```java
class Trie {
    private final TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.isEndOfWord = true;
    }

    public boolean search(String word) {
        TrieNode node = walk(word);
        return node != null && node.isEndOfWord;
    }

    public boolean startsWith(String prefix) {
        return walk(prefix) != null;
    }

    private TrieNode walk(String s) {
        TrieNode node = root;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}
```

**Why `search` and `startsWith` differ by exactly one check:** both walk the identical path; `startsWith` only cares whether the walk completed without hitting a missing child, while `search` additionally requires that the node it lands on was explicitly marked as a complete word's end. Without the `isEndOfWord` flag, there'd be no way to distinguish "app" being a stored word from "app" merely being a prefix of stored "apple" — both would walk to the same node.

**⚠️ Common Mistake:** forgetting `isEndOfWord` entirely and treating "the walk completed" as equivalent to "this exact word was inserted." Trace it: insert `"apple"` only. `search("app")` would incorrectly return `true` without the flag, since the walk for `"app"` completes successfully (those nodes exist, as part of spelling out `"apple"`) — but `"app"` itself was never inserted as a complete word.

### Complexity

**`insert`, `search`, `startsWith`: O(m)** each, where `m` = length of the word/prefix — independent of how many words `n` are already stored, since each operation is a single walk down one path.
**Space: O(N × M)** worst case (N words, average length M) if the words share no prefixes at all — each node holds a fixed 26-slot array regardless of how many children are actually populated, so the constant-factor overhead is real even though the asymptotic bound only reflects total inserted characters.

### Edge cases

- Empty string insert/search — `insert("")` should mark the *root* itself as `isEndOfWord`; `search("")` and `startsWith("")` both trivially succeed at the root without consuming any characters.
- Searching a word that's a strict prefix of a stored word (traced above) — must return `false` without the flag check.
- Duplicate inserts of the same word — idempotent; no special handling needed, `isEndOfWord` just gets set to `true` again.

### Interview framing

**Say before coding:** "Each node holds up to 26 child pointers, one per letter, plus a flag marking whether a complete word ends here. Insert and search both walk character by character; search additionally checks that end-of-word flag where insert doesn't need to."
**Likely follow-up:** "What if the alphabet isn't just lowercase English?" → swap the fixed `TrieNode[26]` array for a `Map<Character, TrieNode>` — more memory-efficient for sparse/large alphabets, at the cost of hashing overhead per step instead of direct array indexing.

---

### Problem 6: Map Sum Pairs (LeetCode 677, Medium) — Pattern: Trie Storing Cumulative Values

**Statement:** Implement `insert(String key, int val)` and `int sum(String prefix)`, where `sum` returns the total value of every inserted key that starts with `prefix`. A repeated `insert` on the same key **replaces**, not adds to, its value.

### Approach — Delta propagation along the insert path

A naive approach would walk to the prefix's node on every `sum` call and then DFS every descendant to total their values — correct, but that DFS can cost `O(n × m)` in the worst case, undoing the whole point of using a Trie. The optimized version instead has **every node accumulate a running sum** of all values passing through it, updated incrementally on each `insert`:

```java
class MapSum {
    static class Node {
        Node[] children = new Node[26];
        int sum = 0;
    }
    private final Node root = new Node();
    private final Map<String, Integer> keyValue = new HashMap<>(); // tracks prior value, for overwrites

    public void insert(String key, int val) {
        int delta = val - keyValue.getOrDefault(key, 0);
        keyValue.put(key, val);

        Node node = root;
        node.sum += delta;
        for (char c : key.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new Node();
            node = node.children[idx];
            node.sum += delta;
        }
    }

    public int sum(String prefix) {
        Node node = root;
        for (char c : prefix.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return 0;
            node = node.children[idx];
        }
        return node.sum;
    }
}
```

**Why the delta, not just adding `val` directly, is the crux of this problem:** if `insert` blindly added `val` to every node's sum along the path, inserting the *same key* twice would double-count it (violating the "replace, not add" rule stated in the problem). Computing `delta = val - previousVal` and applying *that* to every node on the path means a repeated insert with a different value correctly adjusts every ancestor's accumulated sum by exactly the difference — no re-walk or recomputation of anything else needed.

**Why the root itself also accumulates `sum`:** so that `sum("")` — the empty prefix, matching every key — works through the exact same walk-then-return-node.sum logic as any other prefix, with no special-casing required.

### Worked trace

`insert("apple", 3)`: `delta = 3 - 0 = 3`. `keyValue = {"apple": 3}`. Walk `a→p→p→l→e`, adding 3 at root and every node along the path — every node's `sum` is now 3.

`insert("app", 2)`: `delta = 2 - 0 = 2` (not previously in `keyValue`). `keyValue = {"apple":3, "app":2}`. Walk `a→p→p`, adding 2 at root and those three nodes. Root: `3+2=5`. Node `a`: `5`. Node `ap`: `5`. Node `app`: `5` (this node also carries `isEndOfWord` semantics for `"app"` itself, though this problem doesn't need that flag explicitly since `sum` alone answers every query).

`sum("ap")` now: walk `a→p`, return that node's `sum = 5`. Check: `"apple"`(3) + `"app"`(2) = 5. ✅

**Overwrite:** `insert("apple", 2)`: `delta = 2 - 3 = -1` (previous value for `"apple"` was 3). `keyValue = {"apple":2, "app":2}`. Walk `a→p→p→l→e`, subtracting 1 at root and all five nodes. Root: `5-1=4`. Node `ap`: `5-1=4`.

`sum("ap")` now: `4`. Check: `"apple"`(2, updated) + `"app"`(2) = 4. ✅ The overwrite propagated correctly without touching `"app"`'s own contribution.

> 💡 **Interview Insight:** the overwrite case is exactly what a tier-1 interviewer will probe if you present only the "add val directly" version — have the delta explanation and this exact trace ready before they ask, not after.

### Complexity

**`insert`: O(m)**, **`sum`: O(m)** — both are single walks of the key/prefix length, independent of how many keys are stored.
**Space: O(N × M)** for the Trie nodes, plus **O(N)** for the `keyValue` map tracking prior values for delta computation.

### Edge cases

- Querying a prefix that was never inserted as any key or sub-path — `sum` correctly returns 0 via the `null` child check.
- Overwriting a key with the *same* value it already had — `delta = 0`, every node's `sum` is touched but unchanged; correct, just a no-op in effect.
- `sum("")` — matches the earlier design note; returns the root's total, i.e. the sum of every value ever inserted.

### Interview framing

**Say before coding:** "I'll have each Trie node accumulate a running sum of every value that passes through it, so a prefix query is a single O(m) walk that just returns the landed-on node's stored sum — the one wrinkle is handling repeated inserts of the same key as a replace, not an add, which needs tracking the delta against the previous value."
**Likely follow-up:** "What if `sum` needed to be answered without any prior `insert` calls modifying shared state — i.e., could this be parallelized safely?" — flag that concurrent inserts touching overlapping paths would race on the shared `sum` fields; that's outside this problem's scope but worth naming if pushed.

---

## Part 2 — The CAP Theorem

### What it is

For any distributed data store, during a network partition (some nodes can't communicate with others), you must choose between:

- **Consistency** — every read receives the most recent write, or an explicit error; no stale reads are ever silently served.
- **Availability** — every request receives a non-error response, with no guarantee it reflects the most recent write.

You cannot have both simultaneously **while the partition is actually happening**. This is Brewer's theorem, formally proven by Gilbert and Lynch in 2002.

### The common oversimplification, corrected

"Pick 2 of C, A, P" is the popular framing, but it's misleading: **partition tolerance isn't really an optional design choice** for any system that spans more than one machine — network partitions happen regardless of what you'd prefer, so a real distributed system doesn't get to "not choose P." The theorem is more accurately: **when a partition occurs, choose C or A; outside of a partition, you can have both.** "CP vs. AP" is really "C or A *during* a partition," not a permanent, unconditional trade-off.

**Why, mechanically, you can't have both during a partition:** if a partition splits a cluster into two groups, and a client writes to one group, the other group either (a) refuses to serve reads/writes until it can confirm it has the latest data — sacrificing availability to preserve consistency — or (b) keeps serving requests from its own (now possibly stale) data — sacrificing consistency to preserve availability. There's no third option once communication between the groups is actually broken; a node genuinely cannot know it has the latest write if it can't reach whoever might have made one.

### Where real systems land

MongoDB **defaults** toward consistency (a single primary handles writes, with configurable acknowledgment from replicas before confirming a write succeeded); Cassandra **defaults** toward availability (a leaderless, quorum-based design where any node can accept a write, later reconciled). Both words matter: "**defaults**." Both systems expose tunable consistency levels — MongoDB's read/write concern settings, Cassandra's per-query consistency level — so classifying either as an immutable, fixed "CP database" or "AP database" is itself the common mistake to avoid; they ship with a default posture, not a hard-coded one.

> ⚠️ **Common Mistake:** treating a database's CAP classification as a fixed, permanent label rather than a configurable default. An interviewer who asks "is MongoDB CP or AP?" is often specifically checking whether you'll answer with unwarranted certainty or correctly qualify it.

### Extension: PACELC

A well-known refinement, worth naming even briefly: **P**artition → choose **A** or **C**; **E**lse (no partition) → choose **L**atency or **C**onsistency. CAP only describes behavior *during* a partition; PACELC adds that even when nothing is broken, there's still a latency-vs-consistency trade-off (synchronous replication for strong consistency costs latency; asynchronous replication is faster but can serve slightly stale reads). Mentioning PACELC unprompted, briefly, signals depth beyond the textbook CAP framing.

### Interview framing

**When it comes up:** "design a system that needs to stay available during a regional outage" (favors AP) vs. "design a system where two reads must never disagree" (favors CP — a banking ledger is the canonical example, directly connecting to today's project-adjacent Career Block topic and forward to Day 61's Replication Models). State the trade-off explicitly rather than picking one and moving on — naming *what you're giving up*, not just what you're gaining, is the actual signal being evaluated.

---

## Project Block (1.5 hrs)

**Repository:** `todo-api`.
**Task:** implement Trie-based autocomplete for task titles.
**Definition of done:** `/tasks/autocomplete?prefix=X` returns matching titles.

**Practical guidance:** build a `TrieNode`/`Trie` exactly as taught above, but store the *original* title string (not just `isEndOfWord = true`) at the node marking each word's end — autocomplete needs to return the actual titles, not just confirm a prefix exists. On `insert`, walk to the end of each task title's characters and set that node's stored title. On a `startsWith`-style walk to the query prefix's node, DFS every descendant from there, collecting every non-null stored title found. This reuses `startsWith`'s walk directly, with a small DFS added at the end instead of a boolean return.

```java
@GetMapping("/tasks/autocomplete")
public List<String> autocomplete(@RequestParam String prefix) {
    return taskTrie.getWordsWithPrefix(prefix);
}
```

Push once the endpoint returns matching titles for a prefix that matches multiple existing tasks and an empty list for one that matches none.

---

## Career Block (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.
**Networking:** begin researching System Design architectures at target companies — a natural pairing with today's CAP Theorem block, since most real architecture write-ups explicitly justify their consistency/availability posture.

---

## Day 59 — Interview Questions

**Q1. Why does a Trie generalize `TreeNode` rather than introduce a wholly new recursive shape?** Both are self-referential structures holding pointers to further nodes of the same type; a `TrieNode` just holds up to 26 such pointers (indexed by character) instead of 2 named ones (`left`/`right`) — the recursion pattern over the structure is identical, only the branching factor and what each edge represents changes.

**Q2. When would you choose a `HashSet<String>` over a Trie, and when the reverse?** HashSet for exact-match-only membership questions (faster, O(1) average, simpler); Trie specifically when prefix queries are needed (autocomplete, "does any word start with X"), since a HashSet has no efficient way to answer those without a full scan.

**Q3. Why does `search` need the `isEndOfWord` flag when `startsWith` doesn't?** Both walk an identical path; without the flag, there's no way to distinguish a prefix that merely happens to lead to a longer stored word (e.g., `"app"` as a prefix of stored `"apple"`) from `"app"` having actually been inserted as its own complete word.

**Q4. In Map Sum Pairs, why compute a delta rather than adding `val` directly on every insert?** The problem requires overwrite, not accumulation, semantics for a repeated key — adding directly would double-count a re-inserted key. Tracking the key's previous value and applying only the *difference* to every node on the path keeps every ancestor's accumulated sum correct after an overwrite, without needing to re-walk or recompute anything else.

**Q5. Why does the root node also need to track `sum` in Map Sum Pairs?** So `sum("")` — the empty prefix, matching every key — is answered by the exact same walk-then-return-node.sum logic as any other prefix, with no special-cased branch required.

**Q6. State the CAP theorem precisely, and explain why "pick 2 of 3" is a common oversimplification.** During a network partition, a distributed system must choose between consistency (every read reflects the latest write, or errors) and availability (every request gets a response, possibly stale). Partition tolerance isn't a real design choice for a genuinely distributed system — partitions happen regardless — so it's really "C or A during a partition," with both available when no partition is occurring, not an unconditional 2-of-3 trade-off.

**Q7. Are MongoDB and Cassandra permanently "CP" and "AP" respectively?** No — those describe their *defaults* (MongoDB's single-primary model defaults toward consistency; Cassandra's leaderless quorum model defaults toward availability), but both expose tunable consistency levels per query/operation, so treating either classification as fixed and immutable is a mistake an interviewer may specifically probe for.

---

## Daily Deliverable Check

- [ ] Implement Trie and Map Sum Pairs solved, pushed to `dsa-java/tries/`.
- [ ] Can trace Map Sum Pairs' overwrite case (delta computation) from memory, matching this book's worked example.
- [ ] CAP theorem explanation written — 150 words on why MongoDB and Cassandra made different defaults, and what application type would prefer each, correctly using "default" rather than an absolute classification.
- [ ] Autocomplete live in `todo-api`, verified against a prefix matching multiple tasks and one matching none.

---

## What Tomorrow Assumes You Already Know Cold

Day 60 assumes the core Trie walk (`insert`/`search`/`startsWith`) is fully reflexive — tomorrow's two problems both extend it (DFS-validated word building, then wildcard search) without re-deriving the basic node-walk mechanism from scratch. It also assumes you can explain, unprompted, *why* a Trie beats a HashSet for prefix queries specifically, since that judgment call — not the code itself — is what's actually being tested tomorrow and Day 61.

**Next:** [Day 60 Resource Book](Day60_Resource_Book.md) — Tries Continue, and Consistent Hashing.
