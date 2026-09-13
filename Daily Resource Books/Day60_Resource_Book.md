# Day 60 — Tries Continue, and Consistent Hashing

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 59 Resource Book](Day59_Resource_Book.md)
**Next ▶:** [Day 61 Resource Book](Day61_Resource_Book.md)
**Companion to:** Day 60 of `Week_09_Revised.md`

---

## Recap

Yesterday built the Trie's core walk (`insert`/`search`/`startsWith`) and a variant that accumulates values along the path. Today extends the *walk* itself in two directions that don't change the underlying node structure at all: constraining which paths are legal to walk down in the first place (today's first problem), and allowing a single step to branch into every possible child rather than one specific one (today's second problem). Both reuse yesterday's `TrieNode` shape completely unchanged.

## Learning Objectives

By the end of today, without notes:

1. Use a Trie's `isEndOfWord` flag as a *traversal constraint* (not just a final check), and explain why this specific problem needs that reframing.
2. Implement wildcard search over a Trie via recursive branching, and state its true worst-case complexity — not the shorthand version.
3. Set up basic consistent hashing with a `TreeMap`-backed ring, and explain precisely why it remaps far fewer keys than naive modulo hashing when the server count changes.

## Concept Dependency Map

```
Trie core walk (Day 59)
        │
        ├─ NEW: constrained traversal — only descend into
        │    children marked isEndOfWord (LC 720)
        │
        └─ NEW: branching traversal — '.' explores every
             child instead of one (LC 211)
        │
        ▼
🔗 Day 61: both traversal styles combine — Trie + DFS
   for dictionary substitution, then Trie + full
   matrix backtracking for Word Search II

Independent track — System Design:
HashMap hashing/buckets (Day 4) → CAP Theorem (Day 59)
        │
        ▼
NEW: Consistent Hashing — ring topology, TreeMap.ceilingKey()
```

---

## Part 1 — Longest Word in Dictionary

### Problem 7: Longest Word in Dictionary (LeetCode 720, Medium) — Pattern: Trie + DFS

**Statement:** Given a list of words, find the longest word that can be built one character at a time by other words in the list — i.e., every prefix of the answer, obtained by removing characters from the end one at a time down to length 1, must *also* be a complete word in the list. If there's a tie in length, return the lexicographically smallest.

### Approach 1 — Brute force

For each word, check every one of its prefixes against a `HashSet` of all words. Building each prefix as a substring costs `O(m)`, done for `m` prefixes per word, across `n` words: **O(n × m²)**.

### Approach 2 — Middle ground: sort + greedy HashSet

Sort all words lexicographically. Walk them in order, maintaining a `goodWords` set. A word qualifies (gets added to `goodWords`, and considered for the answer) if its length is 1, or if the word minus its last character is already in `goodWords`. Because of the sort order, any valid shorter prefix is guaranteed to have been processed already. **O(n log n + n × m)** — genuinely competitive with the Trie approach, and a legitimate thing to mention as an alternative.

### Approach 3 — Optimized (this pattern's technique): Trie + constrained DFS

```java
public static String longestWord(String[] words) {
    TrieNode root = new TrieNode();
    for (String word : words) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEndOfWord = true;
    }

    StringBuilder best = new StringBuilder();
    dfs(root, new StringBuilder(), best);
    return best.toString();
}

private static void dfs(TrieNode node, StringBuilder path, StringBuilder best) {
    if (path.length() > best.length()
            || (path.length() == best.length() && path.toString().compareTo(best.toString()) < 0)) {
        best.replace(0, best.length(), path.toString());
    }
    for (int i = 0; i < 26; i++) {
        TrieNode child = node.children[i];
        if (child != null && child.isEndOfWord) {   // the constraint: only descend into "buildable" nodes
            path.append((char) ('a' + i));
            dfs(child, path, best);
            path.deleteCharAt(path.length() - 1);   // undo — this IS backtracking, previewed ahead of Day 61
        }
    }
}
```

**The key reframing:** insert every word normally, but the *DFS afterward* only ever descends into a child whose `isEndOfWord` is `true`. That single condition is exactly "the path so far, including this next character, is itself a complete word" — which is precisely the problem's buildability requirement. A child that exists in the Trie (because some *longer* word passes through it) but isn't itself `isEndOfWord` represents a path that's a valid prefix of *something*, but not buildable one word at a time — correctly pruned.

**Why processing children in index order (`i = 0` to `25`, i.e., `'a'` to `'z'`) handles the lexicographic tie-break for free:** a DFS that always tries `'a'` before `'b'` before `'c'`, etc., naturally visits words in lexicographic order at each branching point. Combined with the `>` (strictly longer) comparison replacing `best` — never replacing on a tie in length via `>=` — the *first* word of the maximum length encountered is kept, and because of the traversal order, that first one is guaranteed to be the lexicographically smallest among all words of that length.

> ⚠️ **Common Mistake:** using `>=` instead of `>` when comparing lengths, intending to "always take the latest." This actually breaks the tie-break, since a *later*, lexicographically larger word of the same max length would incorrectly overwrite the correct answer. The explicit length-then-lexicographic comparison shown above is the correct, complete condition — worth stating both branches out loud, not just the length check.

### Worked trace

`words = ["a","banana","app","appl","ap","apply","apple"]`.

Check `"apple"`: needs `a`, `ap`, `app`, `appl`, `apple` all present as complete words — `a`✓, `ap`✓, `app`✓, `appl`✓, `apple`✓ — buildable, length 5.
Check `"apply"`: needs `a`, `ap`, `app`, `appl`, `apply` — all present — buildable, also length 5. **Tie.**
`"apple"` vs `"apply"` lexicographically: `a=a, p=p, p=p, l=l`, then `e` vs `y` — `e < y`, so `"apple"` is smaller. **Answer: `"apple"`.** The DFS reaches `"apple"` before `"apply"` naturally (since `'e' < 'y'` in child-index order), so the `>` -only replacement correctly keeps `"apple"` without needing an explicit tie-break comparison at the end.

### Complexity

**Building the Trie: O(N × M)** (N words, average length M). **DFS: O(N × M)** worst case — every node is visited at most once, and `path.toString()` calls cost `O(M)` each in the worst case. **Total: O(N × M).**
**Space: O(N × M)** for the Trie.

### Edge cases

- No word of length 1 exists in the input for some letter — that entire branch is simply never buildable, correctly excluded by the `isEndOfWord` check at depth 1.
- Multiple words tie at the maximum length — handled by traversal order, as traced above; don't add a separate explicit comparison pass unless you're not confident the traversal order alone is sufficient (state the reasoning if asked).
- Single-character words only — each is independently valid at length 1; the longest among them, lexicographically smallest on a tie, is returned correctly by the same logic.

### Interview framing

**Say before coding:** "I'll insert every word into a Trie, then DFS from the root — but only descend into a child if it's itself marked as a complete word, since that's exactly the 'buildable one step at a time' constraint. Processing children in alphabetical order and only replacing my answer on a strictly longer path gives me the lexicographic tie-break for free."
**Likely follow-up:** "Could you solve this without a Trie?" → yes, the sort + greedy HashSet approach above, same overall complexity class — worth naming to show the Trie isn't the *only* tool, just the one this week is building reps in.

---

## Part 2 — Design Add and Search Words Data Structure

### Problem 8: Design Add and Search Words Data Structure (LeetCode 211, Medium) — Pattern: Trie + DFS

**Statement:** Implement `addWord(String)` and `boolean search(String word)`, where `search` supports `.` as a wildcard matching any single character.

```java
class WordDictionary {
    private final TrieNode root = new TrieNode();

    public void addWord(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEndOfWord = true;
    }

    public boolean search(String word) {
        return dfs(root, word, 0);
    }

    private boolean dfs(TrieNode node, String word, int idx) {
        if (idx == word.length()) return node.isEndOfWord;
        char c = word.charAt(idx);
        if (c == '.') {
            for (TrieNode child : node.children) {
                if (child != null && dfs(child, word, idx + 1)) return true;
            }
            return false;
        } else {
            TrieNode child = node.children[c - 'a'];
            return child != null && dfs(child, word, idx + 1);
        }
    }
}
```

**Mechanism:** an ordinary character follows exactly one child, same as yesterday's `search`. A `.` instead tries **all** 26 possible children recursively, returning `true` if *any* of them leads to a successful match for the rest of the word. This is the same "try every option, recurse, and check if any succeeds" shape that Day 61 formalizes as Backtracking — today's `.` handling is a preview of that idea, not yet named.

### Complexity — corrected from the plan's shorthand

The plan's own hint states this as "O(m)," which is only true for the **wildcard-free case** — a word with zero `.` characters walks exactly one path, `O(m)` where `m` is the word length, identical to plain `search`. That's the fast path, and it's worth stating clearly, but it isn't the *worst case*, and a tier-1 interviewer will specifically probe the worst case here.

**Worst case, precisely: O(26^k × (m − k))** where `m` = word length and `k` = number of `.` characters in it — each wildcard can branch into up to 26 children, and those branches multiply. In the fully-degenerate case where the entire word is dots (`k = m`), this simplifies to the commonly-cited bound **O(26^m)**. This is only *tight* when the Trie is maximally "bushy" at every relevant depth (every one of the 26 possible children genuinely exists at every level the wildcard touches) — in practice, most branches dead-end almost immediately because far fewer than 26 children actually exist at most nodes, so real-world performance is usually far better than the worst-case bound suggests. Both figures — the fast path and the honest worst case — are worth having ready; giving only "O(m)" invites the exact follow-up question this book is flagging.

**Space: O(N × M)** for the Trie, same as every prior Trie problem this week.

> 💡 **Interview Insight:** when a problem's *stated* or commonly-repeated complexity is a simplification, say so and give the precise bound — "the fast path is O(m) with no wildcards; worst case, with the word being all dots, it's O(26^m), though pruning makes that rare in practice" reads as far stronger than confidently repeating an incomplete number.

### Worked trace

`addWord("bad")`, `addWord("dad")`, `addWord("mad")`.

`search("pad")` → follow `p`: no child at index `'p'-'a'` from root → `false`.
`search("bad")` → follow `b→a→d` exactly, land on a node with `isEndOfWord = true` → `true`.
`search(".ad")` → `.` at index 0 tries all root children that exist: `b`, `d`, `m`. Each leads to `→a→d`, all three are complete words → first one tried returns `true` → overall `true`.
`search("b..")` → follow `b`, then `.` tries `a` (the only child of `b`) → then `.` tries `d` (the only child of `b→a`) → lands on `isEndOfWord = true` → `true`.

### Edge cases

- `search` on an empty string — `idx == word.length()` is true immediately at the root; returns `root.isEndOfWord` (true only if `addWord("")` was ever called).
- A wildcard as the *first* character with no words sharing any first letter in common — degenerates to a fast `false`, no real branching occurs despite the wildcard.
- All-wildcard search matching a length that no stored word has — the DFS still explores every branch down to that depth and finds none reaching `isEndOfWord` at exactly that depth; correctly returns `false`, just at the cost of full exploration.

### Interview framing

**Say before coding:** "An ordinary character in the search word follows one specific child, same as yesterday's Trie search. A wildcard tries every existing child recursively and succeeds if any branch matches the rest of the word."
**Likely follow-up:** "What's the actual worst-case complexity?" — this is the question the plan's shorthand invites; answer with the precise `O(26^k × (m-k))` bound above, not just "O(m)."

---

## Part 3 — Consistent Hashing

### What it is, and the problem it solves

Naive hash-based sharding — `hash(key) % N` where `N` is the number of servers — has a specific, costly failure mode: changing `N` (adding or removing even one server) changes the result of the modulo operation for almost *every* key, forcing a near-total remap of data across servers. Consistent hashing avoids this by placing both servers and keys onto the same conceptual **ring**: hash each server's identifier onto a circular hash space, and a key belongs to whichever server is the next one clockwise from the key's own hash position. Adding or removing a single server only reshuffles the keys that were mapped to the ring segment immediately affected — everything else stays put.

### Implementation sketch

```java
public class ConsistentHashRing {
    private final TreeMap<Integer, String> ring = new TreeMap<>();

    public void addServer(String server) {
        ring.put(hash(server), server);
    }

    public void removeServer(String server) {
        ring.remove(hash(server));
    }

    public String getServerForKey(String key) {
        if (ring.isEmpty()) throw new IllegalStateException("no servers");
        int keyHash = hash(key);
        Map.Entry<Integer, String> entry = ring.ceilingEntry(keyHash);
        if (entry == null) {
            entry = ring.firstEntry(); // wrap around the ring
        }
        return entry.getValue();
    }

    private int hash(String s) {
        return s.hashCode(); // a real system uses SHA-1/MD5 for better distribution
    }
}
```

**Why `TreeMap` specifically:** it's a balanced binary search tree keyed by hash value, giving `O(log S)` lookup (S = number of servers) via `ceilingKey()` — "the smallest key ≥ this value," which is exactly "the next server clockwise." When no such key exists (the key's hash falls past every server's position), `firstEntry()` wraps back to the beginning of the ring — the ring is circular, not a line.

**Why this remaps far fewer keys than naive modulo hashing:** removing or adding one server only affects the contiguous ring segment between that server and its immediate neighbor — every key that was already mapped to any *other* segment is entirely untouched, since its nearest-clockwise-server computation never involved the changed server at all. Naive `% N` has no such locality: changing `N` changes the divisor for every single key's computation simultaneously.

### Extension: virtual nodes

A real weakness of the basic version above: with few servers, their positions on the ring can land unevenly, giving some servers disproportionately large segments (and thus disproportionate load). The standard fix is **virtual nodes** — each physical server is hashed onto the ring multiple times under different virtual identifiers (e.g., `server1#0`, `server1#1`, ... `server1#150`), smoothing out the distribution without changing the core lookup mechanism at all.

### Trade-offs against the nearest alternative

| | Naive `hash(key) % N` | Consistent Hashing |
|---|---|---|
| Lookup | O(1) | O(log S) |
| Keys remapped on scaling | ~100% | ~K/S (proportional share only) |
| Implementation complexity | Trivial | Moderate (ring + virtual nodes for even distribution) |

The O(1) vs. O(log S) difference is almost always worth paying for the dramatically better remap behavior — a distributed cache or database that requires a near-total data shuffle every time it scales is rarely acceptable in practice.

### Interview framing

**Say before coding:** "I'll hash both servers and keys onto the same ring, using a sorted structure so I can find 'the next server clockwise' from any key's position in O(log S), with wraparound handling the case where nothing exists past the key's position."
**Likely follow-up:** "What if servers end up unevenly distributed on the ring?" → virtual nodes, explained above — naming this unprompted, even briefly, signals you know the basic version has a real limitation.

---

## Project Block (1.5 hrs)

**Repository:** `java-fundamentals`.
**Task:** `ConsistentHashingDemo` class. Add/remove a server and print how many keys actually moved.
**Definition of done:** pushed with the before/after key-movement count.

**Practical guidance:** populate the ring with, say, 3 servers and 1,000 sample keys; record each key's assigned server. Add a 4th server; recompute every key's assignment; count how many keys changed servers. Compare that count against what naive `% N` would have produced for the same scenario (nearly all 1,000) — printing both numbers side by side is what makes the demo's point land, not just the consistent-hashing number alone.

---

## Career Block (1 hr)

**LinkedIn:** Post 13 — "Why adding a server shouldn't reshuffle your whole cache."
**Networking:** identify 3 more Target Tier B companies.

---

## Day 60 — Interview Questions

**Q1. Why does Longest Word in Dictionary's DFS only descend into children where `isEndOfWord` is true?** That condition is precisely "the path so far is itself a complete word," which is the problem's buildability requirement — a child that exists (because a longer word passes through it) but isn't itself a complete word represents an unbuildable path and must be pruned.

**Q2. Why does processing Trie children in index order (`'a'` to `'z'`) resolve the lexicographic tie-break without an explicit final comparison?** A DFS that always tries `'a'` before `'b'` naturally visits candidate words in lexicographic order at each branch point; combined with only replacing the answer on a strictly longer match (never on a tie), the first max-length word encountered is guaranteed to be the lexicographically smallest.

**Q3. What's the actual worst-case time complexity of Design Add and Search Words' `search`, and when is the commonly-cited "O(m)" figure accurate?** Worst case is O(26^k × (m−k)) where k = number of wildcards, simplifying to O(26^m) when the entire word is wildcards. "O(m)" is only accurate for the wildcard-free fast path, where the search follows exactly one path with no branching.

**Q4. Why is a wildcard's worst-case bound rarely hit in practice, despite being technically exponential?** The bound is only tight when every node the wildcard touches has close to all 26 possible children genuinely populated; in most real dictionaries, far fewer than 26 children exist at most nodes, so branches dead-end quickly and actual performance is much better than the worst case suggests.

**Q5. Why does naive `hash(key) % N` remap nearly all keys when `N` changes, while consistent hashing remaps only a small fraction?** Modulo hashing has no locality — changing the divisor changes essentially every key's computed result simultaneously. Consistent hashing's ring locality means only the keys in the specific segment adjacent to the added/removed server are affected; every other segment's nearest-clockwise-server computation is untouched.

**Q6. What problem do virtual nodes solve in consistent hashing, and how?** Uneven server placement on the ring (a real risk with few physical servers) can give some servers disproportionately large key ranges. Hashing each physical server onto the ring multiple times under distinct virtual identifiers smooths the distribution without changing the core ceilingKey-based lookup mechanism.

---

## Daily Deliverable Check

- [ ] Longest Word in Dictionary and Design Add and Search Words Data Structure solved, pushed to `dsa-java/tries/`.
- [ ] Can state Design Add and Search Words' precise worst-case complexity (not just "O(m)") from memory.
- [ ] `ConsistentHashingDemo` pushed with the measured key-movement count, compared against naive modulo hashing's behavior for the same scenario.
- [ ] LinkedIn Post 13 published.

---

## What Tomorrow Assumes You Already Know Cold

Day 61 assumes both of today's traversal styles — constrained descent (only into valid-word children) and branching descent (trying every child at once) — are solid, since tomorrow's Word Search II combines Trie traversal with full backtracking over a grid, and Replace Words reuses the constrained-descent idea from Longest Word in Dictionary directly. It also assumes recursion's "try an option, recurse, undo" shape (already previewed today in the `.append()`/`.deleteCharAt()` pairing) is comfortable, since Backtracking — opening later on Day 61 — names and formalizes exactly that shape.

**Next:** [Day 61 Resource Book](Day61_Resource_Book.md) — Tries Close, Backtracking Begins, and Replication Models.
