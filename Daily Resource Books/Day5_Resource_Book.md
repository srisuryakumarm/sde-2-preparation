# Day 5 Resource Book — The HashMap/HashSet Interview Pattern (12 Problems) and the Four OOP Pillars

**Series:** Week 1 SDE-2 Prep · [← Curriculum Map](./00_Curriculum_Map.md) · [← Day 4](./Day4_Resource_Book.md) · Next: Day 6 →

**Companion to:** Day 5 of `Week_01_Revised.md`

---

## Recap: what today is

Day 4 gave you `HashMap`/`HashSet` as tools. Today formalizes them as a **recognized interview pattern family** — the single highest-leverage pattern in early-to-mid interview prep, because it converts an enormous number of naturally-O(n²) "check every pair / every combination" problems into O(n). Your plan lists 7 problems; this book covers those 7 in full depth, plus **5 extra problems** using the same underlying techniques, so the pattern becomes reflexive rather than tied to memorizing 7 specific answers. It then formally names the four OOP pillars you've been using — some since Day 2 — and introduces one genuinely new concept needed for today's project: the **abstract class**.

## Learning Objectives

By the end of today, without notes:

1. Recognize, from a problem statement alone, which of the five HashMap/HashSet sub-patterns applies: complement lookup, membership, frequency counting, canonical-key grouping, or smart-starting-point traversal.
2. Solve all 12 problems in this book, explaining brute force, optimized approach, and the reasoning connecting problem structure to data structure choice — out loud, unprompted.
3. Name and define all four OOP pillars, each anchored to a concrete example you've already built.
4. Explain the difference between an interface and an abstract class, and know which one a given design calls for.

## Concept Dependency Map for Today

```
HashMap/HashSet as raw tools (Day 4)
        │
        ▼
Five sub-patterns, each a distinct reason to reach for HashMap/HashSet:
  1. Complement lookup       (Two Sum)
  2. Membership               (Contains Duplicate)
  3. Frequency counting        (Valid Anagram, Ransom Note, First Unique Character, Majority Element)
  4. Canonical-key grouping    (Group Anagrams)
  5. Smart-starting-point      (Longest Consecutive Sequence)
  + two-way mapping             (Isomorphic Strings, Word Pattern)
  + prefix sum + lookup         (Subarray Sum Equals K — combines pattern 1's shape with a new technique)
        │
        ▼
OOP Four Pillars (needs: encapsulation/inheritance/interfaces — Day 2)
        │
        ▼
Abstract classes (NEW — needed for a design where subclasses share real implementation, not just a contract)
        │
        ▼
enum with per-constant behavior (needs: abstract methods, polymorphism)
        │
        ▼
Practice: Account class hierarchy
```

---

# Part 1 — The HashMap/HashSet Pattern Family

The single idea underneath every problem in this section: **whenever a brute-force solution checks every pair, every combination, or repeatedly asks "have I seen X," a HashMap or HashSet can very often collapse an O(n²) scan into an O(n) pass**, by trading the *searching* for a *calculation* — exactly the mechanism from Day 4, Section 2. Recognizing *which* of the five sub-patterns above applies to a new, unfamiliar problem is the actual skill being built here — not memorizing 12 solutions.

---

## Problem 1: Two Sum (LeetCode 1, Easy) — Pattern: Complement Lookup

**Statement:** Given an array of integers `nums` and an integer `target`, return the indices of the two numbers that add up to `target`. Exactly one solution exists; you may not use the same element twice.

### Approach 1 — Brute force

```java
public static int[] twoSumBruteForce(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] == target) {
                return new int[]{i, j};
            }
        }
    }
    throw new IllegalArgumentException("No solution found");
}
```

Check every pair directly. Correct, but **O(n²)** — for each of the n elements, scan up to n more.

### Approach 2 — Optimized: HashMap complement lookup

```java
public static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();   // value -> index
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        seen.put(nums[i], i);
    }
    throw new IllegalArgumentException("No solution found");
}
```

**The reframe that makes this work:** instead of asking, for every *pair*, "do these two sum to target?" — ask, for every *single* element, "have I already seen the specific number that would complete this pair?" That second question is exactly what a HashMap answers in O(1) average time. This turns n² pair-checks into n independent O(1) lookups.

**Why check for the complement *before* inserting the current number:** this ordering prevents matching an element with itself. If `target = 6` and the current number is `3`, checking *before* inserting means the map doesn't yet contain this `3` when we ask "is `3` (the complement) already present?" — so we won't accidentally pair index `i` with itself. This ordering also correctly handles genuine duplicates: for `nums = [3, 3]`, `target = 6` — at `i = 0`, complement `3` isn't in the (empty) map yet, so we insert `3 → 0`; at `i = 1`, complement `3` *is* now found (from index 0), correctly returning `[0, 1]`.

**Complexity: Time O(n) — one pass, O(1) average per lookup/insert; Space O(n) — worst case, the answer is the last two elements, so nearly everything gets stored before a match is found.**

> 💡 **Interview Insight:** Two Sum is the canonical exemplar for this entire pattern family — nearly every problem below is a variation on "reframe a pairwise/repeated check as a single-pass lookup." The strongest opening move is stating the brute force, naming its O(n²) cost, and *explicitly* proposing the space-for-time trade before writing the optimized version — narrating that trade-off is exactly what's being evaluated, not just arriving at working code.

---

## Problem 2: Contains Duplicate (LeetCode 217, Easy) — Pattern: Membership

**Statement:** Given an integer array, return `true` if any value appears at least twice.

This problem is worth working through with **three** approaches rather than two — it's a genuinely good vehicle for the time/space trade-off discussion, since there's a real middle ground between brute force and optimal.

```java
// Approach 1 — Brute force: check every pair. O(n^2) time, O(1) space.
public static boolean containsDuplicateBruteForce(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] == nums[j]) return true;
        }
    }
    return false;
}

// Approach 2 — Sort, then check adjacent pairs. O(n log n) time, O(1) extra space.
public static boolean containsDuplicateSorting(int[] nums) {
    int[] sorted = nums.clone();     // avoid mutating the caller's array — a deliberate design choice
    Arrays.sort(sorted);
    for (int i = 1; i < sorted.length; i++) {
        if (sorted[i] == sorted[i - 1]) return true;
    }
    return false;
}

// Approach 3 — HashSet. O(n) time, O(n) space.
public static boolean containsDuplicate(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    for (int num : nums) {
        if (!seen.add(num)) {   // add() returns FALSE if the element was already present
            return true;
        }
    }
    return false;
}
```

**A small API detail worth knowing:** `Set.add()` returns a `boolean` — `true` if the element was newly added, `false` if it was already present. This lets the HashSet approach combine "check membership" and "record it" into a single call, rather than the more verbose `if (seen.contains(num)) return true; seen.add(num);`.

**Note on Approach 2:** sorting a *copy* (`nums.clone()`) rather than sorting `nums` directly is a deliberate choice worth stating out loud — mutating a caller's input as an unannounced side effect is a real, common source of subtle bugs elsewhere in a program; a strong candidate flags this trade-off rather than silently picking one.

| Approach | Time | Space |
|---|---|---|
| Brute force | O(n²) | O(1) |
| Sort then scan | O(n log n) | O(1) extra (in-place sort) |
| HashSet | O(n) | O(n) |

**Why HashSet is the default answer, while still being able to discuss the alternative:** O(n) is strictly faster than O(n log n) — reach for the HashSet by default. But being able to say "if extra space were constrained, I'd sort in place instead, trading time for space" — and meaning it — is exactly the trade-off articulation this whole series has been building toward, and it's a genuinely fair thing for an interviewer to probe on this specific problem.

**Complexity: as tabled above. Edge cases:** empty array (no duplicates possible — `false`); single element (`false`); all identical elements (`true`, detected on the second element in every approach).

---

## Problem 3: Valid Anagram (LeetCode 242, Easy) — Pattern: Frequency Counting

**Statement:** Given strings `s` and `t`, return `true` if `t` is an anagram of `s` (same letters, same counts, any order).

### Approach 1 — Sorting

```java
public static boolean isAnagramSorting(String s, String t) {
    if (s.length() != t.length()) return false;
    char[] sChars = s.toCharArray();
    char[] tChars = t.toCharArray();
    Arrays.sort(sChars);
    Arrays.sort(tChars);
    return Arrays.equals(sChars, tChars);
}
```

Two anagrams, sorted, must produce identical character sequences. Correct, but **O(n log n)** (dominated by the two sorts), and O(n) space for the char arrays — necessary because Strings are immutable (Day 2) and can't be sorted directly.

### Approach 2 — Optimized: frequency array

```java
public static boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] freq = new int[26];
    for (int i = 0; i < s.length(); i++) {
        freq[s.charAt(i) - 'a']++;
        freq[t.charAt(i) - 'a']--;
    }
    for (int count : freq) {
        if (count != 0) return false;
    }
    return true;
}
```

**The trick, worth stating explicitly:** increment for `s`, decrement for `t`, in the *same* pass. A character appearing equally often in both strings nets to exactly zero; any imbalance shows up directly as a non-zero entry. This is a direct reuse of Day 2's character-frequency-array technique (Problem 3, that day) — you already built the core mechanism.

**Why a fixed-size array beats a `HashMap<Character, Integer>` here, specifically:** the "key space" — lowercase English letters — is small, known in advance, and fixed. A HashMap pays hashing overhead *and* autoboxing overhead (`Character`/`Integer` objects, Day 3) for what's really just 26 possible buckets. Reaching for a more specific, cheaper structure when the constraints justify it — rather than defaulting to HashMap on a "pattern day" purely out of habit — is itself a sophistication signal worth having ready to articulate.

**Complexity: Time O(n), Space O(1)** — the array is always exactly 26 elements, regardless of string length.

**Edge cases:** different lengths (early-return false, O(1)); empty strings (trivially anagrams of each other — `true`); this simple version assumes lowercase English letters only — worth stating as an explicit assumption, with `Character.toLowerCase()` or a general `HashMap<Character, Integer>` as the fix for full case-insensitive/Unicode generality.

---

## Problem 4: Ransom Note (LeetCode 383, Easy) — Pattern: Frequency Counting

**Statement:** Given `ransomNote` and `magazine`, return `true` if `ransomNote` can be constructed using `magazine`'s letters (each letter in `magazine` usable at most once).

```java
public static boolean canConstruct(String ransomNote, String magazine) {
    if (ransomNote.length() > magazine.length()) return false;
    int[] freq = new int[26];
    for (char c : magazine.toCharArray()) {
        freq[c - 'a']++;
    }
    for (char c : ransomNote.toCharArray()) {
        freq[c - 'a']--;
        if (freq[c - 'a'] < 0) {
            return false;   // used more of this letter than the magazine actually has
        }
    }
    return true;
}
```

**The same mechanism as Valid Anagram, with a genuinely different stopping condition — worth being explicit about the distinction:** Valid Anagram needs *exact* equality — every count must land on precisely zero. Ransom Note only needs `magazine` to provide *enough* of each letter — a coverage/subset check, not an exact match, since `magazine` is allowed leftover, unused letters. The mechanism (frequency counting via a fixed array) is identical; the check at the end (all-zero vs. never-goes-negative) is what actually differs, and recognizing "same tool, different success condition" is the real transferable insight.

**The early length check** (`ransomNote.length() > magazine.length()`) is an O(1) short-circuit: if the note needs more total letters than the magazine has, it's immediately impossible regardless of *which* letters — not strictly required for correctness (the per-character negative check would eventually catch it too), but a good "reject impossible input fast" instinct.

**Complexity: Time O(m + n)** where m = magazine length, n = ransomNote length — stated with two separate variables deliberately, per Day 3's "different inputs get different variables" rule, since the two strings aren't guaranteed the same length. **Space O(1).**

**Edge cases:** empty `ransomNote` (trivially constructible — `true`); empty `magazine` with non-empty `ransomNote` (caught by the length check); `magazine` with unused extra letters (correctly ignored — never checked).


---

## Problem 5: Isomorphic Strings (LeetCode 205, Easy) — Pattern: Two-Way Mapping

**Statement:** Given strings `s` and `t`, determine if `s`'s characters can be replaced to get `t`, such that: each character in `s` maps to exactly one character in `t`, **and** no two different characters in `s` map to the same character in `t` (the mapping must be injective in both directions).

This is the first problem in this pattern family where a single, natural-looking approach is *silently wrong* rather than merely slow — worth working through carefully, since spotting this kind of bug in your own code is a real skill.

### The broken, single-map attempt — and why it fails

```java
// BROKEN — only checks the s -> t direction
public static boolean isIsomorphicBroken(String s, String t) {
    Map<Character, Character> map = new HashMap<>();
    for (int i = 0; i < s.length(); i++) {
        char sc = s.charAt(i), tc = t.charAt(i);
        if (map.containsKey(sc)) {
            if (map.get(sc) != tc) return false;
        } else {
            map.put(sc, tc);
        }
    }
    return true;
}
```

Trace `s = "ab"`, `t = "aa"` (which should be **invalid** — both `'a'` and `'b'` would need to map to `'a'`, violating the two-different-source-characters rule): `i=0`: `sc='a'`, `tc='a'`, not yet in map → put `a→a`. `i=1`: `sc='b'`, `tc='a'` — `'b'` isn't a *key* in the map yet (only `'a'` is), so this branch also just inserts `b→a`. The loop finishes having never found a conflict, and **incorrectly returns `true`.** The bug: a single forward map checks that `s→t` is a valid *function* (each `s`-character maps to exactly one `t`-character), but never checks that it's *injective* — nothing stops two different `s`-characters from both mapping to the same `t`-character.

### Correct approach: two maps (or one map + one "used" set)

```java
public static boolean isIsomorphic(String s, String t) {
    if (s.length() != t.length()) return false;
    Map<Character, Character> mapST = new HashMap<>();
    Map<Character, Character> mapTS = new HashMap<>();
    for (int i = 0; i < s.length(); i++) {
        char sc = s.charAt(i);
        char tc = t.charAt(i);
        if (mapST.containsKey(sc) && mapST.get(sc) != tc) return false;
        if (mapTS.containsKey(tc) && mapTS.get(tc) != sc) return false;
        mapST.put(sc, tc);
        mapTS.put(tc, sc);
    }
    return true;
}
```

Re-tracing `s = "ab"`, `t = "aa"` against this version: `i=0`: neither map has entries — put `a→a` (mapST) and `a→a` (mapTS). `i=1`: `sc='b'`, `tc='a'` — `mapTS` **already contains** `'a'` (mapped to `'a'` from `i=0`); check `mapTS.get('a') != sc` → `'a' != 'b'` → **mismatch, correctly returns `false`.** The second map is precisely what catches the violation the single-map version missed.

An equivalent, sometimes-preferred alternative — a single forward map plus a `HashSet` tracking which target characters are already "claimed" by some source character — is exactly what your plan's own hint offers as an option, and works identically:

```java
public static boolean isIsomorphicAlt(String s, String t) {
    if (s.length() != t.length()) return false;
    Map<Character, Character> mapping = new HashMap<>();
    Set<Character> usedTargets = new HashSet<>();
    for (int i = 0; i < s.length(); i++) {
        char sc = s.charAt(i), tc = t.charAt(i);
        if (mapping.containsKey(sc)) {
            if (mapping.get(sc) != tc) return false;
        } else {
            if (usedTargets.contains(tc)) return false;   // tc already claimed by a DIFFERENT source char
            mapping.put(sc, tc);
            usedTargets.add(tc);
        }
    }
    return true;
}
```

**Complexity: Time O(n), Space O(1)** — the maps/set hold at most as many entries as the alphabet size, which is bounded regardless of how long the input strings are (the same "fixed, bounded key space counts as O(1)" reasoning as the frequency-array problems above).

**Edge cases:** different lengths (early false); single-character strings (trivially isomorphic); all-identical characters in both (`"aaa"`/`"bbb"` — valid, consistent one-to-one mapping); empty strings (trivially isomorphic).

> 💡 **Interview Insight:** Not every problem's difficulty is "optimize a slow brute force" — this one's difficulty is "notice that an intuitive-looking solution is subtly incomplete." Explicitly checking your own solution against a small adversarial example (like `"ab"`/`"aa"`) *before* declaring it done is a habit worth having automatic, and mentioning that you're doing this check is itself a good signal.

---

## Problem 6: Group Anagrams (LeetCode 49, Medium) — Pattern: HashMap Keyed by Canonical Form

**Statement:** Given an array of strings, group the anagrams together.

### Approach 1 — Sorted string as the key

```java
public static List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String str : strs) {
        char[] chars = str.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(str);
    }
    return new ArrayList<>(groups.values());
}
```

**The core idea: a canonical form.** Anagrams are, by definition, rearrangements of the same multiset of characters — sorting any string produces a single, deterministic ordering of its characters, so two strings are anagrams of each other **if and only if** their sorted forms are identical. This "map many different-but-equivalent inputs to one shared representative key" technique generalizes well beyond anagrams specifically.

**A useful API worth knowing: `computeIfAbsent`.** `groups.computeIfAbsent(key, k -> new ArrayList<>())` returns the list already stored at `key` if one exists, or — if not — computes a new one (here, an empty `ArrayList`), stores it, and returns *that*. Either way, the returned list is immediately ready to `.add(str)` onto. This replaces the more verbose `if (!groups.containsKey(key)) { groups.put(key, new ArrayList<>()); } groups.get(key).add(str);` and is the idiomatic way to write "group items by a computed key," a shape that recurs constantly.

### Approach 2 — Optimized: frequency-count key instead of sorting

```java
public static List<List<String>> groupAnagramsOptimized(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String str : strs) {
        int[] count = new int[26];
        for (char c : str.toCharArray()) {
            count[c - 'a']++;
        }
        StringBuilder keyBuilder = new StringBuilder();
        for (int i = 0; i < 26; i++) {
            keyBuilder.append('#').append(count[i]);   // delimiter avoids ambiguous concatenation
        }
        groups.computeIfAbsent(keyBuilder.toString(), k -> new ArrayList<>()).add(str);
    }
    return new ArrayList<>(groups.values());
}
```

**Why this is strictly better, and a subtlety in its own design worth flagging:** sorting each string costs O(k log k) (k = string length); counting each string's letters costs O(k), with a fixed O(26) = O(1) pass to build the key afterward — genuinely faster per string. **The `'#'` delimiter matters:** without a separator between counts, values like `[1, 10, 0, ...]` and `[11, 0, 0, ...]` could produce ambiguous, colliding concatenated keys (`"110..."` parses multiple ways). This is a real, easy-to-miss edge case in one's *own* solution design, not just the problem's input — worth naming unprompted.

**Complexity:** sorted-key approach: **Time O(n × k log k), Space O(n × k)** (n = number of strings, k = max string length — exactly as your plan states). Frequency-key approach: **Time O(n × k), Space O(n × k)** — same space, strictly better time.

**Edge cases:** empty input (empty result); a single string (one group of one); strings that are exact duplicates (correctly grouped — trivially anagrams of themselves).

> 💡 **Interview Insight:** The canonical-form idea — reduce many equivalent inputs to one shared, comparable key, then group by that key in a HashMap — is a transferable pattern well beyond this specific problem. Naming it as such ("this is a group-by-canonical-key problem") is a stronger signal than just producing working code.


---

## Problem 7: Longest Consecutive Sequence (LeetCode 128, Medium) — Pattern: HashSet, Smart Starting Point

**Statement:** Given an unsorted array of integers, find the length of the longest run of consecutive integers (e.g., `[100, 4, 200, 1, 3, 2]` → the run `1, 2, 3, 4` → answer `4`). The problem explicitly asks for **O(n)** — better than the O(n log n) a sort-based approach would give.

This is the hardest of the seven, and it's worth working through **three** versions — including a subtly-wrong-complexity middle version — because the reasoning about *why* the final version is O(n) is exactly the kind of thing a sharp interviewer will push on.

### Approach 1 — Sorting

```java
public static int longestConsecutiveSorting(int[] nums) {
    if (nums.length == 0) return 0;
    int[] sorted = nums.clone();
    Arrays.sort(sorted);
    int longest = 1, current = 1;
    for (int i = 1; i < sorted.length; i++) {
        if (sorted[i] == sorted[i - 1]) {
            continue;                          // duplicate — doesn't break OR extend a streak
        } else if (sorted[i] == sorted[i - 1] + 1) {
            current++;
        } else {
            current = 1;                       // streak broken, restart
        }
        longest = Math.max(longest, current);
    }
    return longest;
}
```

Correct, and a perfectly reasonable *first* approach to state — but **O(n log n)**, which doesn't meet the problem's explicit requirement.

### Approach 2 — HashSet *without* the smart-starting-point check (a genuine trap, worth seeing explicitly)

```java
// LOOKS like it should be O(n) because it uses a HashSet — it is NOT.
public static int longestConsecutiveNaiveHashSet(int[] nums) {
    Set<Integer> numSet = new HashSet<>(Arrays.asList(Arrays.stream(nums).boxed().toArray(Integer[]::new)));
    int longest = 0;
    for (int num : numSet) {
        int currentNum = num;
        int currentStreak = 1;
        while (numSet.contains(currentNum + 1)) {
            currentNum++;
            currentStreak++;
        }
        longest = Math.max(longest, currentStreak);
    }
    return longest;
}
```

This is worth sitting with, because it's a genuine, common trap: **merely using a HashSet doesn't automatically make an algorithm O(n).** Here, the inner `while` loop runs starting from *every single element*, not just the true start of each run. For an input that's one long consecutive run, `[1, 2, 3, ..., n]`: starting from `1` counts `n` steps; starting from `2` counts `n-1` steps; starting from `3` counts `n-2`; and so on. The total work is `n + (n-1) + (n-2) + ... + 1`, which sums to **O(n²)** — actually *worse* than the sorting approach above, despite "using a HashSet." The HashSet makes each individual `contains` check fast; it does nothing by itself to prevent massively redundant, overlapping counting.

### Approach 3 — Optimized: HashSet *with* the smart-starting-point check

```java
public static int longestConsecutive(int[] nums) {
    Set<Integer> numSet = new HashSet<>();
    for (int num : nums) {
        numSet.add(num);
    }

    int longest = 0;
    for (int num : numSet) {
        // Only count from a number that is the START of a run —
        // i.e., num - 1 is NOT in the set, so nothing extends this run to the left.
        if (!numSet.contains(num - 1)) {
            int currentNum = num;
            int currentStreak = 1;
            while (numSet.contains(currentNum + 1)) {
                currentNum++;
                currentStreak++;
            }
            longest = Math.max(longest, currentStreak);
        }
    }
    return longest;
}
```

**Trace, to confirm correctness:** `nums = [100, 4, 200, 1, 3, 2]` → `numSet = {100, 4, 200, 1, 3, 2}`.
- `num=100`: `contains(99)`? No → start. Streak: `100` only (`contains(101)`? No). `longest = 1`.
- `num=4`: `contains(3)`? **Yes** → skip (not a run-start).
- `num=200`: `contains(199)`? No → start. Streak: `200` only. `longest` stays `1`.
- `num=1`: `contains(0)`? No → start. `1→2→3→4` (each `+1` found), streak `4`. `contains(5)`? No, stop. `longest = 4`.
- `num=3`: `contains(2)`? Yes → skip.
- `num=2`: `contains(1)`? Yes → skip.

Final answer: `4`. Correct.

### Why this version genuinely is O(n), despite the nested loop shape

This is the crux, and it's worth being able to state precisely rather than gesturing at it: **the inner `while` loop only ever executes starting from a true run-start** — a number whose predecessor is *not* in the set. Every other number is rejected by the `if` check in **O(1)** and contributes nothing further. Across the *entire* outer loop's execution, **every individual number in the set gets visited by some inner `while` loop at most once, total** — specifically, by whichever run-start's while-loop walk reaches it. This is not "nested loops multiply" (Day 3, Rule 2); it's the same *amortized* shape as Day 3's `ArrayList` analysis and Day 4's two-stack queue: most outer iterations do O(1) work, and the total work done across all the "expensive" iterations, summed together, is bounded by n — not n *per outer iteration*.

**Complexity: Time O(n), Space O(n)** (the HashSet stores all n elements).

**Edge cases:** empty array (`0`); single element (`1`); all-duplicate values (e.g., `[1,1,1,1]` — the Set collapses duplicates to `{1}`, correctly yielding streak length `1`, not 4); negative numbers (no special handling needed — HashSet works identically for any integers); several disjoint runs (each found independently via its own run-start, correctly).

> 💡 **Interview Insight:** This problem rewards narrating the "why doesn't this look-like-nested-loops end up O(n²)?" argument *before* an interviewer has to ask it. Presenting Approach 3 silently, with no acknowledgment that it superficially resembles the O(n²) trap in Approach 2, leaves a sharp interviewer unconvinced even if the code is correct — walking through the amortized argument unprompted is exactly the depth this series has been building toward.


---

# Part 2 — Extra Practice: Five More Reps of the Same Patterns

Your plan's 7 problems establish each sub-pattern once. These five add repetition — same underlying techniques, fresh surfaces — which is what actually builds reflexive pattern recognition rather than memorized answers to 7 specific problems.

## Extra Practice 1: Majority Element (LeetCode 169, Easy) — Pattern: Frequency Counting

**Statement:** Given an array of size n, return the element that appears more than `n/2` times. (Guaranteed to exist.)

```java
// Approach 1 — HashMap frequency counting. O(n) time, O(n) space.
public static int majorityElementHashMap(int[] nums) {
    Map<Integer, Integer> counts = new HashMap<>();
    for (int num : nums) {
        int count = counts.getOrDefault(num, 0) + 1;
        counts.put(num, count);
        if (count > nums.length / 2) {
            return num;   // exit the instant a count crosses the majority threshold
        }
    }
    throw new IllegalStateException("No majority element found");
}
```

**A genuinely valuable bonus approach — Boyer-Moore Voting, O(1) space:**

```java
public static int majorityElementBoyerMoore(int[] nums) {
    int candidate = nums[0];
    int count = 0;
    for (int num : nums) {
        if (count == 0) {
            candidate = num;
        }
        count += (num == candidate) ? 1 : -1;
    }
    return candidate;
}
```

**Why this works — a genuinely clever cancellation argument:** think of it as a running "vote." Seeing the current `candidate` again increments the count; seeing anything else decrements it. If the count ever reaches zero, the current candidate is abandoned and replaced with whatever number comes next. Because the true majority element appears *more* than n/2 times, it can never be fully cancelled out by every other element combined (which together number *fewer* than n/2) — so whichever candidate survives to the very end must be the majority element.

> ⚠️ **A precondition worth stating explicitly:** this algorithm relies entirely on a majority element being *guaranteed* to exist. Without that guarantee, Boyer-Moore can return a wrong answer (some non-majority value can survive the cancellation process if nothing actually holds a true majority) — it is not a general "find the most frequent element" algorithm, and presenting it as one would be a real correctness gap.

| Approach | Time | Space |
|---|---|---|
| HashMap frequency | O(n) | O(n) |
| Boyer-Moore Voting | O(n) | O(1) |

**Complexity/edge cases:** single-element array (trivially the majority); all-identical elements (immediately confirmed).

---

## Extra Practice 2: Intersection of Two Arrays (LeetCode 349, Easy) — Pattern: Membership

**Statement:** Given two integer arrays, return their intersection — each element appearing at most once in the result.

```java
public static int[] intersection(int[] nums1, int[] nums2) {
    Set<Integer> set1 = new HashSet<>();
    for (int num : nums1) {
        set1.add(num);
    }
    Set<Integer> resultSet = new HashSet<>();
    for (int num : nums2) {
        if (set1.contains(num)) {
            resultSet.add(num);
        }
    }
    int[] result = new int[resultSet.size()];
    int i = 0;
    for (int num : resultSet) {
        result[i++] = num;
    }
    return result;
}
```

**Why two HashSets, not one:** `set1` turns membership-checking against `nums1` into O(1) average (versus an O(m) scan of the raw array for every element of `nums2`, which would give an O(m × n) brute force). A *second* Set for the result isn't just convenient — it directly satisfies the "each element at most once" requirement automatically, with no separate deduplication step needed.

**Complexity: Time O(m + n), Space O(m + n)** — two differently-sized inputs, named accordingly (Day 3's rule, applied again).

**Edge cases:** no overlap (empty result); one array empty (empty result); duplicate values within an array (correctly deduplicated by the Set-based result); full overlap (result is every unique shared value).

---

## Extra Practice 3: First Unique Character in a String (LeetCode 387, Easy) — Pattern: Frequency Counting

**Statement:** Return the index of the first character in a string that doesn't repeat. If none exists, return `-1`.

```java
public static int firstUniqChar(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) {
        freq[c - 'a']++;
    }
    for (int i = 0; i < s.length(); i++) {
        if (freq[s.charAt(i) - 'a'] == 1) {
            return i;
        }
    }
    return -1;
}
```

**Why this genuinely needs two passes — worth being ready to justify, since "can't you do this in one pass?" is a fair, common follow-up:** you cannot know whether a character is unique until you've seen the *entire* string — a character appearing once in the first half might reappear later. The first pass builds complete frequency information; the second pass then scans **in original order** (to correctly identify "*first*") against that now-complete data.

**Complexity: Time O(n)** — two passes, but `2n` simplifies to O(n) under Day 3's constant-dropping rule; **Space O(1)** — the fixed 26-element array.

**Edge cases:** no unique character exists (`"aabbcc"` → `-1`); single-character string (trivially unique, index `0`); the first character itself is the unique one (correctly returned immediately in the second pass).


---

## Extra Practice 4: Word Pattern (LeetCode 290, Easy) — Pattern: Two-Way Mapping

**Statement:** Given a `pattern` (e.g., `"abba"`) and a string `s` of space-separated words, determine whether `s` follows the same pattern — a bijection between pattern letters and words.

This is Isomorphic Strings' exact shape, one level removed: mapping *characters to words* instead of *characters to characters* — deliberately included to prove the pattern transfers, not just repeats.

```java
public static boolean wordPattern(String pattern, String s) {
    String[] words = s.split(" ");
    if (pattern.length() != words.length) return false;

    Map<Character, String> charToWord = new HashMap<>();
    Map<String, Character> wordToChar = new HashMap<>();

    for (int i = 0; i < pattern.length(); i++) {
        char c = pattern.charAt(i);
        String word = words[i];

        if (charToWord.containsKey(c) && !charToWord.get(c).equals(word)) return false;
        if (wordToChar.containsKey(word) && wordToChar.get(word) != c) return false;

        charToWord.put(c, word);
        wordToChar.put(word, c);
    }
    return true;
}
```

**A detail worth being deliberate about, since it's easy to get backward:** `charToWord.get(c).equals(word)` uses `.equals()`, because it's comparing two `String` objects — Day 2's lesson, directly relevant again. `wordToChar.get(word) != c` uses primitive `!=`, because `c` is a raw `char`, and `wordToChar.get(word)` auto-unboxes its `Character` result for the comparison — so `!=` is correctly comparing primitive values, not object references. This single problem requires both forms, correctly, on adjacent lines, for two different underlying reasons — a genuinely good self-check for whether the Day 2 `==`/`.equals()` lesson actually stuck.

**Complexity: Time O(n), Space O(n)** where n = pattern length (= word count).

**Edge cases:** mismatched pattern length vs. word count (early false); repeated pattern letters correctly requiring repeated words (`pattern="abba"`, `s="dog cat cat dog"` → `true`); a word reused for two different pattern letters (correctly caught by `wordToChar`, mirroring the Isomorphic Strings bug this structure is designed to prevent).

---

## Extra Practice 5: Subarray Sum Equals K (LeetCode 560, Medium) — Pattern: Prefix Sum + Lookup

**Statement:** Given an array of integers and an integer `k`, return the number of continuous subarrays whose sum equals `k`. Array elements may be negative.

This is the most advanced problem in this book — a deliberate stretch, combining Two Sum's complement-lookup *shape* with a genuinely new technique: the **prefix sum**.

### Approach 1 — Brute force

```java
public static int subarraySumBruteForce(int[] nums, int k) {
    int count = 0;
    for (int start = 0; start < nums.length; start++) {
        int sum = 0;
        for (int end = start; end < nums.length; end++) {
            sum += nums[end];      // extend the running sum incrementally — avoids re-summing from scratch
            if (sum == k) count++;
        }
    }
    return count;
}
```

Every possible `(start, end)` subarray, sum computed incrementally as `end` extends — **O(n²)** time, O(1) space.

### Approach 2 — Optimized: prefix sum + HashMap

**The prefix sum concept:** define `prefixSum[i]` as the sum of all elements before index `i` (so `prefixSum[0] = 0`, the sum of an empty prefix). The key identity: **the sum of any subarray from index `i` to `j` (inclusive) equals `prefixSum[j+1] - prefixSum[i]`** — "everything up to and including `j`," minus "everything before `i`," leaves exactly the elements in between.

Rearranged, that identity becomes: we want pairs where `prefixSum[j+1] - prefixSum[i] == k`, i.e., `prefixSum[i] == prefixSum[j+1] - k`. Scanning left to right while maintaining a running sum, at each position we ask: **"how many earlier prefix sums equal my current running sum minus k?"** — exactly Two Sum's complement-lookup shape, applied to running sums instead of raw array values, and answerable in O(1) average time if every prefix sum seen so far is tracked in a HashMap.

```java
public static int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixSumCounts = new HashMap<>();
    prefixSumCounts.put(0, 1);   // the empty prefix (sum 0) has occurred once, before reading anything

    int runningSum = 0;
    int count = 0;

    for (int num : nums) {
        runningSum += num;
        count += prefixSumCounts.getOrDefault(runningSum - k, 0);
        prefixSumCounts.put(runningSum, prefixSumCounts.getOrDefault(runningSum, 0) + 1);
    }

    return count;
}
```

**Trace, to confirm correctness:** `nums = [1, 2, 3]`, `k = 3` — expected subarrays: `[1,2]` and `[3]`, so answer `2`.
- Start: `prefixSumCounts = {0: 1}`, `runningSum = 0`, `count = 0`.
- `num=1`: `runningSum=1`. Look for `1-3=-2`: absent. Record `1`. Map: `{0:1, 1:1}`.
- `num=2`: `runningSum=3`. Look for `3-3=0`: **found (count 1)** → `count=1`. Record `3`. Map: `{0:1,1:1,3:1}`.
- `num=3`: `runningSum=6`. Look for `6-3=3`: **found (count 1)** → `count=2`. Record `6`.

Final: `count = 2`. Correct.

**Two initialization/ordering details that are common, real bugs:**

1. **`prefixSumCounts.put(0, 1)` before the loop is not optional.** Without it, a subarray starting at index `0` that itself sums to exactly `k` would have no match — the running sum would equal `k`, we'd look for `runningSum - k = 0`, and find nothing, because `0` was never recorded as a seen prefix sum. This under-counts silently.
2. **The lookup must happen *before* updating the map with the current running sum, not after.** Looking up first (against the map's state *before* this iteration's sum is added) ensures a subarray can't incorrectly "match itself" via its own just-inserted entry — order matters here, and swapping it introduces a subtle over-counting bug.

**Why this needs prefix sum + HashMap rather than a sliding window (a fair thing to be asked, since sliding window is coming later in this plan):** a sliding window's efficiency relies on sums growing and shrinking *predictably* as the window expands or contracts — which only holds when all elements are non-negative. This problem explicitly allows negative numbers, which breaks that monotonicity assumption entirely; prefix sum + HashMap works correctly regardless of sign, since it never relies on the running sum moving in one direction.

**Complexity: Time O(n) — one pass, O(1) average per HashMap operation; Space O(n) — up to n distinct running sums stored.**

**Edge cases:** negative numbers present (exactly why this approach, not sliding window, is needed); `k = 0` (correctly counts subarrays summing to zero via the same logic); single-element array.

> 💡 **Interview Insight:** This problem is a genuine synthesis — Two Sum's complement-lookup *shape*, applied to a new auxiliary quantity (prefix sums) rather than raw values. Naming that connection explicitly ("this is Two Sum's lookup trick, just applied to running sums instead of the array values themselves") demonstrates the pattern has actually generalized in your understanding, rather than being 12 memorized, disconnected solutions.


---

# Part 3 — OOP, Layer 2: The Four Pillars, Named

You've been *using* three of these four pillars since Day 2, without the formal vocabulary attached. Today connects the name to the mechanism you already have working code for, and introduces the fourth (Abstraction) plus one genuinely new tool needed for today's project.

## The Four Pillars

### 1. Encapsulation

**Definition:** keeping an object's internal state (`private` fields) inaccessible from outside the class, exposed only through deliberately controlled methods.

**Where you already did this:** Day 2's `Book` class — `title`, `author`, `isAvailable` are all `private`; `isAvailable` changes only through `checkOut()`/`returnBook()`, which enforce the "can't double-check-out" rule.

**Why it matters:** it lets a class *guarantee its own invariants* — rules that must always hold — regardless of what external code does, because external code is never given a direct path to violate them.

### 2. Inheritance

**Definition:** a subclass acquiring the fields and methods of a superclass via `extends`, then adding to or overriding that behavior.

**Where you already did this:** Day 2's `Employee`/`Manager` hierarchy — `Manager extends Employee`, calling `super(name, baseSalary)` and overriding `calculatePay()`.

**Why it matters:** shared structure and behavior across related types get written *once*, in the superclass, rather than duplicated across every subclass — and a fix or improvement to shared logic in the superclass automatically benefits every subclass.

### 3. Polymorphism

**Definition:** code written against a shared supertype (an interface or superclass) that correctly invokes each concrete subtype's *own* specific implementation, at runtime, without needing to know which concrete type it's actually holding.

**Where you already did this:** Day 2's `ShapeDemo` — `Shape[] shapes = {new Circle(5), new Rectangle(3,4)}`, then `s.area()` in a loop that never mentions `Circle` or `Rectangle` by name, yet calls the correct formula for each.

**Why it matters:** new types can be added later (a `Triangle`, say) with **zero changes** to code that already operates polymorphically against the shared `Shape` contract — a real, practical extensibility benefit, not just an abstract nicety.

### 4. Abstraction

**Definition:** exposing a simple, stable *contract* while hiding the complexity of *how* that contract is fulfilled.

**Where you already did this, without the name attached:** every interface you've written *is* abstraction — `Shape.area()` tells you *what* you can ask for, and hides *how* each shape actually computes it. Callers of `s.area()` never need to know whether that's a simple multiplication (`Rectangle`) or involves `Math.PI` (`Circle`) — the contract is all that's visible from outside.

**Why it matters:** it lets the *implementation* of something change freely — a faster algorithm, a bug fix, an entirely different internal representation — without breaking any code that only ever depended on the stable contract.

> 🔑 **Key Takeaway:** these four pillars aren't four separate, disconnected features — they compose. Abstraction defines *what's* exposed; encapsulation controls *how state changes* behind that exposure; inheritance lets related types *share* structure; polymorphism lets code written once work correctly across every type that honors the shared contract. Today's `Account` hierarchy (Part 4) uses all four simultaneously, which is the realistic, ordinary way they actually show up in real code.

---

# Part 4 — Abstract Classes (New)

Today's project needs a tool that hasn't been introduced yet: a class that, like an interface, **cannot be instantiated directly** — but unlike an interface, **can** contain real, shared implementation (actual method bodies, actual fields) alongside method declarations that subclasses are *required* to implement themselves.

```java
public abstract class Account {
    private String accountHolder;
    private double balance;

    public Account(String accountHolder, double balance) {
        this.accountHolder = accountHolder;
        this.balance = balance;
    }

    // CONCRETE method — shared, working, identical for every subclass. No reimplementation needed.
    public double getBalance() {
        return balance;
    }

    // ABSTRACT method — no body. Every subclass MUST provide its own implementation, or fail to compile.
    public abstract double calculateInterest();
}
```

`new Account("Alice", 1000)` **will not compile.** The `abstract` keyword on the class declares it explicitly incomplete — `calculateInterest()` has no body, so there's nothing to actually execute if a raw `Account` were somehow created and that method called. Only a concrete subclass that provides a real implementation for every abstract method can be instantiated.

### When to reach for an abstract class instead of an interface

| | Interface | Abstract class |
|---|---|---|
| Can hold shared, working implementation? | No (beyond default methods, out of scope for today) | Yes |
| Can hold shared state (fields)? | No | Yes |
| A class can have how many? | Multiple (`implements A, B, C`) | One (`extends` — single inheritance only) |
| Best fit when... | Unrelated types share *only* a contract, no implementation (`Shape` — a `Circle` and `Rectangle` share nothing except "has an `area()`") | Related types genuinely share *state and behavior*, differing in one specific piece of logic (every `Account` shares a holder, a balance, deposit/withdraw rules — they differ only in *how interest is calculated*) |

This is precisely why today's `Account` hierarchy calls for an abstract class rather than an interface: every account type needs the *same* `accountHolder`, `balance`, `getBalance()`, `deposit()`, and `withdraw()` — reimplementing all of that identically in every subclass would be exactly the duplication inheritance exists to eliminate. Only `calculateInterest()` genuinely varies.

---

# Part 5 — `enum` With Per-Constant Behavior

An `enum` isn't limited to naming a fixed set of constants — each constant can carry its **own** implementation of a method, different from every other constant's:

```java
public enum AccountType {
    SAVINGS {
        @Override
        public double calculateInterest(double balance) {
            return balance * 0.04;   // 4%
        }
    },
    CHECKING {
        @Override
        public double calculateInterest(double balance) {
            return balance * 0.01;   // 1%
        }
    };

    public abstract double calculateInterest(double balance);
}
```

**What's mechanically happening here — worth understanding, not just copying:** each enum constant (`SAVINGS`, `CHECKING`) actually compiles to its **own anonymous subclass** of the `AccountType` enum, each overriding the abstract method with its own specific logic. This is, genuinely, the exact same polymorphism mechanism from Section "Polymorphism" above — just a new *syntax* applying a mechanism you already understand, not a new mechanism to learn from scratch.

```java
AccountType type = AccountType.SAVINGS;
double interest = type.calculateInterest(1000);   // 40.0
```


---

# Part 6 — Practice: The `Account` Class Hierarchy

Putting all four pillars, the abstract class, and the enum together:

```java
public abstract class Account {
    private String accountHolder;
    private double balance;

    public Account(String accountHolder, double balance) {
        this.accountHolder = accountHolder;
        this.balance = balance;
    }

    public String getAccountHolder() {
        return accountHolder;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        balance += amount;
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        if (amount > balance) {
            throw new IllegalArgumentException("Insufficient funds");
        }
        balance -= amount;
    }

    public abstract double calculateInterest();
}

public class SavingsAccount extends Account {
    private static final double INTEREST_RATE = 0.04;

    public SavingsAccount(String accountHolder, double balance) {
        super(accountHolder, balance);
    }

    @Override
    public double calculateInterest() {
        return getBalance() * INTEREST_RATE;
    }
}

public class CheckingAccount extends Account {
    private static final double INTEREST_RATE = 0.01;

    public CheckingAccount(String accountHolder, double balance) {
        super(accountHolder, balance);
    }

    @Override
    public double calculateInterest() {
        return getBalance() * INTEREST_RATE;
    }
}
```

**A refinement worth noticing versus Day 2's `Employee`/`Manager`:** that earlier example used `protected` fields, letting subclasses reach in directly. This version uses **`private`** fields with a public getter (`getBalance()`) that subclasses use exactly like any other caller would. This is slightly more disciplined encapsulation — the base class retains full control over its own invariants even from its own subclasses, rather than granting them a side door around the public API.

**A small new piece of syntax:** `private static final double INTEREST_RATE`. `static` — shared at the class level, not duplicated per object (the same meaning from Day 1's `main` method, now applied to a field). `final` — can never be reassigned after initialization, making it a true constant. Together, `static final` is Java's standard idiom for "a named constant."

**Demonstration — polymorphism and the abstract-class restriction, both visible at once:**

```java
public static void main(String[] args) {
    Account savings = new SavingsAccount("Alice", 1000);
    Account checking = new CheckingAccount("Bob", 500);

    Account[] accounts = {savings, checking};
    for (Account acc : accounts) {
        System.out.println(acc.getAccountHolder() + ": interest = " + acc.calculateInterest());
    }

    // new Account("Carol", 100);   // COMPILE ERROR — Account is abstract, cannot be instantiated directly
}
```

Exactly the same "iterate over a shared supertype, get correct type-specific behavior" shape as Day 2's `ShapeDemo` — now applied to a design that also shares real, concrete implementation (`deposit`, `withdraw`, `getBalance`) across every subclass, which is precisely the scenario an abstract class (rather than a bare interface) is for.

---

# Section — Project Block

In `java-fundamentals`, build the `Account` hierarchy above (or your own equivalent design following the same principles).

**Definition of done:** the hierarchy cannot be instantiated via its abstract base class (verify this by confirming `new Account(...)` genuinely fails to compile — don't just assume it); every field is strictly `private`, accessed only through getters/setters; pushed.

---

# Section — Career Block

### Accountability partner

Post on r/developersIndia or LinkedIn looking for an SDE-2 prep partner for weekly mock interviews. **Worth lining this up now, deliberately ahead of when it's needed:** later phases of this plan (particularly the LLD/system-design phase) lean heavily on mock interviews, and a working accountability partnership takes real time to establish — starting the search only once you actually need it creates an avoidable bottleneck right when momentum matters most.

### Weekly Industry Awareness Ritual (20 minutes)

Clear your TLDR Newsletter backlog and read one engineering blog post. A small, low-effort, recurring habit — worth protecting precisely because it's easy to let slip once daily DSA/project work fills the available time.

---

# Day 5 — Interview Questions

---

**1. What's the core reframe that turns Two Sum from O(n²) into O(n)?**

*Answer:* Instead of checking every pair directly, ask — for each single element — "have I already seen the specific value that would complete this pair with the target?" That question is answerable in O(1) average time via a HashMap, turning n² pair-checks into n independent lookups.

---

**2. Why must Two Sum check for the complement *before* inserting the current element into the map?**

*Answer:* Checking first prevents matching an element with itself, and correctly handles genuine duplicate values — inserting first could cause a number to be paired with its own just-inserted entry.

---

**3. Name the three approaches to Contains Duplicate and their time/space trade-offs.**

*Answer:* Brute force (O(n²), O(1)); sort then scan adjacent (O(n log n), O(1) extra); HashSet (O(n), O(n)). HashSet is the default for its better time complexity; sorting is the answer if extra space is constrained.

---

**4. Why is `Set.add()` returning a `boolean` useful for Contains Duplicate?**

*Answer:* It combines "check if already present" and "record it" into a single call — `if (!seen.add(num))` is true exactly when `num` was already in the set, since `add()` returns `false` in that case.

---

**5. Why does Valid Anagram use an `int[26]` array rather than a `HashMap<Character, Integer>`?**

*Answer:* The key space (lowercase letters) is small and fixed in advance. A HashMap pays hashing and autoboxing overhead for what's really just 26 known buckets — a fixed-size array is strictly cheaper when the constraints justify it.

---

**6. Valid Anagram and Ransom Note both use frequency counting. How do their success conditions differ?**

*Answer:* Valid Anagram needs exact equality — every count must land on precisely zero. Ransom Note only needs `magazine` to provide *enough* of each letter — a coverage check where leftover unused letters in `magazine` are fine, checked via "does any count ever go negative."

---

**7. Why does Isomorphic Strings need two maps (or one map plus a "used" set), not just one?**

*Answer:* A single forward map only verifies the mapping is a valid function (each source character maps to one target character) — it never checks that the mapping is injective (that no two different source characters map to the same target character). A second map (or a used-targets set) is what catches that specific violation.

---

**8. [Trace]** Given `s="ab"`, `t="aa"`, why does a single-map-only solution incorrectly return `true`?**

*Answer:* `'a'` maps to `'a'` first; then `'b'` — which isn't yet a *key* in the map — is freely mapped to `'a'` as well, since nothing checks whether `'a'` (the target) is already claimed by a different source character. The loop finds no conflict and wrongly reports valid.

---

**9. What is a "canonical form," and how does Group Anagrams use one?**

*Answer:* A canonical form maps many different-but-equivalent inputs to one shared representative key. Group Anagrams sorts each string's characters (or builds a frequency-count signature) so that all anagrams of each other produce an identical key, then groups by that key in a HashMap.

---

**10. Why is a frequency-count key better than a sorted-string key for Group Anagrams?**

*Answer:* Sorting each string costs O(k log k); counting costs O(k) with a fixed O(1) pass to build the key from the count array — strictly faster per string, giving O(n×k) instead of O(n×k log k) overall.

---

**11. What is the "smart starting point" trick in Longest Consecutive Sequence, and why is it necessary for O(n)?**

*Answer:* Only start counting a run's length from a number whose predecessor (`num - 1`) is *not* in the set — i.e., only from true run-starts. Without this check, every element (not just run-starts) triggers a count, causing massive redundant overlapping work that degrades to O(n²).

---

**12. Why doesn't the nested while-inside-for structure in the optimized Longest Consecutive Sequence make it O(n²)?**

*Answer:* The inner while loop only ever runs from true run-starts; every other element is rejected in O(1). Across the entire outer loop, every individual number gets visited by some inner while-loop walk at most once total — the total work across all "expensive" iterations is bounded by n, not multiplied by n.

---

**13. What happens if you use a HashSet in Longest Consecutive Sequence but skip the smart-starting-point check?**

*Answer:* It's still correct, but no longer O(n) — starting the count from every element (not just run-starts) causes overlapping, redundant counting, degrading to O(n²) in the worst case (e.g., one long consecutive run) — actually worse than the O(n log n) sorting approach, despite using a HashSet.

---

**14. Explain Boyer-Moore Voting's core insight for Majority Element, and its precondition.**

*Answer:* Treat matches to the current candidate as +1 votes and mismatches as -1; if the count hits zero, switch candidates. Because a true majority element appears more than n/2 times, it can never be fully cancelled by all other elements combined, so whichever candidate survives to the end must be it. Precondition: a majority element must be guaranteed to exist, or the algorithm can return a wrong answer.

---

**15. Why does Intersection of Two Arrays use a Set for the *result*, not just for the lookup structure?**

*Answer:* A second Set for the result automatically enforces the "each element appears at most once" requirement, with no separate deduplication step needed.

---

**16. Why does First Unique Character need two passes rather than one?**

*Answer:* Uniqueness can't be determined until the entire string has been seen — a character appearing once early could repeat later. The first pass builds complete frequency data; the second scans in original order to correctly find the *first* unique one.

---

**17. In Word Pattern, why does one line use `.equals()` and an adjacent line use `!=` for what looks like a similar comparison?**

*Answer:* `.equals()` compares two `String` objects by content. `!=` compares a primitive `char` (auto-unboxed from a `Character`) by value — different types being compared, each correctly using the appropriate mechanism.

---

**18. What is a prefix sum, and how does it relate to any subarray's sum?**

*Answer:* `prefixSum[i]` is the sum of all elements before index `i`. Any subarray sum from index `i` to `j` equals `prefixSum[j+1] - prefixSum[i]` — the difference between two prefix sums isolates exactly the elements in between.

---

**19. Why must `prefixSumCounts.put(0, 1)` be initialized before the loop in Subarray Sum Equals K?**

*Answer:* Without it, a subarray starting at index 0 that itself sums to exactly k would never find a match, since a running sum equal to k would look for a prefix sum of 0, which wouldn't be recorded yet — silently undercounting.

---

**20. Why must the lookup happen before updating the map with the current running sum, not after?**

*Answer:* Looking up first, against the map's state before this iteration's sum is recorded, prevents a subarray from incorrectly matching against its own just-inserted entry — reversing the order introduces a subtle over-counting bug.

---

**21. Why can't Subarray Sum Equals K use a sliding window, given the array may contain negative numbers?**

*Answer:* Sliding window efficiency relies on the window's sum changing predictably (monotonically) as it expands or contracts, which only holds when all elements are non-negative. Negative numbers break that assumption entirely, so prefix sum + HashMap — which doesn't depend on monotonic growth — is needed instead.

---

**22. Name and define all four OOP pillars, each with an example from this series.**

*Answer:* Encapsulation — private state + controlled access (`Book`'s `isAvailable`, changeable only via `checkOut`/`returnBook`). Inheritance — shared structure via `extends` (`Employee`/`Manager`). Polymorphism — one contract, many implementations, resolved at runtime (`Shape[]` calling `.area()`). Abstraction — a simple exposed contract hiding implementation detail (any interface — `Shape.area()` hides *how* each shape computes it).

---

**23. What's the difference between an interface and an abstract class? When would you choose each?**

*Answer:* An interface holds only method signatures, no shared implementation or state, and a class can implement multiple. An abstract class can hold real shared implementation and fields, but a class can extend only one. Choose an interface when unrelated types share only a contract (`Shape`); choose an abstract class when related types genuinely share state and behavior, differing in one specific piece of logic (`Account`).

---

**24. Why can't `Account` be instantiated directly?**

*Answer:* It's declared `abstract` and contains at least one abstract method (`calculateInterest()`) with no body — there'd be nothing to actually execute if that method were called on a raw `Account`. Only a concrete subclass providing a real implementation can be instantiated.

---

**25. Mechanically, how does an enum constant get its own implementation of an abstract method?**

*Answer:* Each enum constant with a body compiles to its own anonymous subclass of the enum type, overriding the abstract method with its own logic — the same polymorphism mechanism used elsewhere in OOP, applied via enum syntax.

---

## Daily Deliverable Check

- [ ] All 12 HashMap/HashSet pattern problems (7 required + 5 extra) solved, with brute force and optimized approaches both understood
- [ ] All 7 required problems pushed to `dsa-java/hashmap-hashset/`, one folder per problem
- [ ] Can name and explain all four OOP pillars with your own example for each, without notes
- [ ] `Account` hierarchy pushed; confirmed the abstract base class genuinely cannot be instantiated
- [ ] Accountability-partner post published

---

## What Tomorrow Assumes You Already Know Cold

Day 6 introduces Two Pointers — a pattern that, like today's, will show up constantly for the rest of this plan. It assumes the HashMap/HashSet pattern-recognition instinct from today is now close to automatic, since one of Day 7's Two Pointers extras (3Sum) directly combines *both* patterns together. If any of today's 12 problems still require re-deriving the approach from scratch rather than recognizing the pattern quickly, that repetition is worth doing before moving forward — today was, by design, the heaviest day in Week 1.

**Next:** [Day 6 Resource Book](./Day6_Resource_Book.md) — Two Pointers Begins, and SOLID Principles.
