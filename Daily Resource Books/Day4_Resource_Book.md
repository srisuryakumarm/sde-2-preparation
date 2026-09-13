# Day 4 Resource Book — HashSet, HashMap, Stack, and Queue: The Rest of Your DSA Toolkit

**Series:** Week 1 SDE-2 Prep · [← Curriculum Map](./00_Curriculum_Map.md) · [← Day 3](./Day3_Resource_Book.md) · Next: Day 5 →

**Companion to:** Day 4 of `Week_01_Revised.md`

---

## Recap: what today builds on

Big-O (Day 3) is what makes "O(1) average" — the headline claim for both structures you're meeting today — actually mean something precise rather than being a vague promise. `.equals()` (Day 2) is about to become load-bearing in a way it wasn't before: `HashMap` and `HashSet` use it directly, together with a new concept introduced today (`hashCode()`), to decide whether two keys are "the same." Today rounds out your core toolkit — after today, essentially every DSA pattern for the rest of this plan draws from `ArrayList`, `HashSet`, `HashMap`, or `ArrayDeque`.

## Learning Objectives

By the end of today, without notes:

1. Explain what a hash function does and, mechanically, why it makes `HashMap`/`HashSet` operations average O(1) instead of O(n).
2. State the `equals()`/`hashCode()` contract precisely, and explain what breaks if it's violated.
3. Choose correctly between `ArrayList`, `HashSet`, `HashMap`, and `ArrayDeque` for a described problem, and justify the choice.
4. Solve Valid Parentheses (and three related stack problems) using a stack, and articulate *why* the problem's structure signals "stack" before writing any code.

## Concept Dependency Map for Today

```
Big-O (Day 3) + .equals() (Day 2)
        │
        ▼
Hash functions → buckets → average O(1) lookup
        │
        ▼
HashSet (built on the same hashing mechanism as HashMap, but storing only keys)
        │
        ▼
HashMap (key → value, using hashCode() + equals() together)
        │
        ▼
Stack (LIFO) & Queue (FIFO) via ArrayDeque — independent of hashing, needs only Day 1-2 basics
        │
        ▼
Practice: word frequency (HashMap) · unique elements (HashSet) · Valid Parentheses + 3 extra stack problems
```

---

# Section 1 — HashSet

### What it is, in one sentence

A **`HashSet`** is a collection that holds only **unique** elements and answers "have I seen this before?" (`.contains()`) in **average O(1)** time — dramatically faster than scanning a list, which would need up to O(n) comparisons in the worst case.

### Why it exists — the problem it solves

Checking membership in an `ArrayList` (`list.contains(x)`) requires scanning element by element until a match is found or the list is exhausted — O(n) in the worst case, every single time you ask. `HashSet` trades that linear scan for a fundamentally different mechanism (Section 2 explains exactly how), turning "have I seen this?" into a near-constant-time question regardless of how many elements are already stored.

### API

```java
import java.util.HashSet;

HashSet<Integer> seen = new HashSet<>();
seen.add(5);
seen.add(10);
seen.add(5);              // no-op — 5 is already present; a Set can never hold duplicates

boolean has = seen.contains(5);   // true, average O(1)
seen.remove(10);
int size = seen.size();
```

> 💡 **Interview Insight:** "Contains duplicates," "have you seen this element before," "return only the unique values" — this exact vocabulary in a problem statement is a direct, strong signal to reach for `HashSet`. Naming that signal explicitly out loud ("this is asking about membership/uniqueness, so I'd reach for a HashSet") before writing code is exactly the kind of stated reasoning tier-1 interviews are listening for.

---

# Section 2 — HashMap

### What it is

A **`HashMap`** stores **key → value** pairs and gives average **O(1)** insert, lookup, and delete — by key. (In fact, `HashSet` is — quite literally, in Java's actual implementation — a thin wrapper around a `HashMap`, storing your elements as keys and ignoring the value slot entirely. Understanding `HashMap`'s mechanism *is* understanding `HashSet`'s mechanism.)

### The mechanism: how a hash function turns "search" into "calculate"

This is the part worth understanding at the level of *mechanism*, not just accepting as a label — it's genuinely one of the highest-value pieces of conceptual knowledge in this whole series, because it explains *why* the O(1) claim is true rather than asking you to take it on faith.

Internally, a `HashMap` keeps an array of "buckets" (conceptually similar to the arrays from Day 2). When you call `map.put(key, value)`:

1. A **hash function** is run on the key, producing a number (the **hash code**) — deterministically: the *same* key always produces the *same* hash code, every time.
2. That number is used (typically via something like `hashCode % numberOfBuckets`) to pick **which bucket** this key-value pair belongs in.
3. The pair is stored in that bucket.

When you later call `map.get(key)`, the *exact same* hash function runs on that key, producing the *exact same* number, pointing at the *exact same* bucket — so instead of scanning every stored entry to find a match (O(n)), the `HashMap` **calculates** exactly where to look, in one step, and jumps straight there. This is structurally the same idea as Day 2's array-indexing argument (`base_address + offset` instead of scanning) — just one layer more abstract: instead of computing a memory address directly from an index, a hash function computes a *bucket index* from an arbitrary key.

### Collisions — and why they don't break correctness

Two genuinely different keys can, by chance, hash to the *same* bucket — this is called a **collision**, and with a fixed number of buckets and a potentially unlimited variety of keys, it's mathematically unavoidable in general (this is the pigeonhole principle: more possible keys than buckets means some bucket must eventually hold more than one). Java's `HashMap` handles this by letting each bucket hold more than one entry (conceptually, a small list of entries that all happen to hash to that bucket) — so a collision costs a little extra work checking the (usually very few) entries sharing that bucket, but never breaks correctness. 🔗 **The deeper mechanics of collision handling — including how Java optimizes very crowded buckets — are genuinely more advanced than today's scope; today's goal is correct, confident *usage*, not implementing a hash table from scratch.**

### The contract that makes all of this actually work: `equals()` and `hashCode()`

This is the piece explicitly worth getting exactly right, because getting it wrong produces bugs that are notoriously confusing to debug — code that *looks* completely correct, compiles fine, and silently fails at runtime.

**The rule:** if two objects are equal according to `.equals()`, they **must** produce the same `hashCode()`. (The reverse is not required — two *unequal* objects are allowed to share a hash code; that's just an ordinary collision, already handled per above.)

**Why this rule has to hold, mechanically:** `HashMap` uses `hashCode()` to decide *which bucket* to look in, and only then uses `.equals()` to find the *exact* matching entry within that bucket (in case of a collision). If a class violates the contract — say, it overrides `.equals()` to compare by meaningful content, but doesn't correspondingly override `hashCode()` to match — then two objects that are genuinely "equal" by that class's own definition could still compute *different* hash codes, land in *different* buckets, and the `HashMap` would never even look in the right place to find them as the same key. `map.put(keyA, "value")` followed by `map.get(keyB)`, where `keyA.equals(keyB)` is `true`, could silently return `null` instead of `"value"` — not a crash, not an exception, just quietly wrong data, which is exactly the kind of bug that's expensive to track down.

> 🔑 **Key Takeaway:** `hashCode()` gets you to the right *neighborhood* (bucket); `.equals()` confirms the exact *match* once you're there. Both must agree on what "the same key" means, or the whole mechanism silently breaks.

For everything in this plan so far, this is handled for you — `String` and the wrapper classes (`Integer`, etc.) correctly implement matching `equals()`/`hashCode()` pairs as part of the standard library, which is exactly why they've been safe to use as `HashMap` keys without a second thought. The contract becomes something *you* have to actively honor the day you start using your own custom classes as `HashMap` keys — worth having this reasoning ready now, even before that day arrives.

> 💡 **A concrete, well-known detail worth knowing:** Java's `String.hashCode()` is computed with a specific, published formula — `s[0]×31^(n−1) + s[1]×31^(n−2) + ... + s[n−1]`, using 31 as the multiplier specifically because it's an odd prime, and the JVM can optimize multiplication by 31 into a cheap bit-shift-and-subtract (`31 × i == (i << 5) − i`). Knowing this exists — not necessarily reproducing the formula from memory — is the kind of small, concrete detail that signals real depth rather than surface familiarity.

### API

```java
import java.util.HashMap;
import java.util.Map;

HashMap<String, Integer> map = new HashMap<>();
map.put("apple", 1);              // insert (or overwrite, if "apple" already existed)
map.put("banana", 2);

Integer count = map.get("apple");            // 1 — returns null (NOT a default!) if the key is absent
int safeCount = map.getOrDefault("cherry", 0);  // 0 — "cherry" isn't present, so the fallback is used

boolean hasKey = map.containsKey("apple");    // true

for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}
```

> ⚠️ **Common Mistake:** `map.get(missingKey)` returns **`null`**, not `0` or any other default. If you assign that directly into a primitive — `int x = map.get("missingKey");` — Java attempts to *auto-unbox* `null` into an `int`, which is impossible, and throws a `NullPointerException` at runtime. This is precisely why `.getOrDefault(key, fallback)` exists, and it's the standard, idiomatic way to avoid this trap.

### The idiom you'll use constantly from here forward

```java
map.put(word, map.getOrDefault(word, 0) + 1);
```

This single line correctly handles *both* cases a frequency counter needs: if `word` hasn't been seen yet, `getOrDefault` returns `0`, so this stores `0 + 1 = 1` (first occurrence, recorded correctly). If `word` has already been seen, `getOrDefault` returns its current count, so this stores `count + 1` (a correct increment). One line, no explicit `if/else` branching for "is this the first time or not" — this exact idiom is the backbone of nearly every frequency-counting problem you'll meet from tomorrow onward.


---

# Section 3 — Stack and Queue (via `ArrayDeque`)

### The two mental models

- **Stack — Last-In-First-Out (LIFO).** Picture a stack of plates: you can only add to, or remove from, the *top*. The most recently added item is always the first one to come back out.
- **Queue — First-In-First-Out (FIFO).** Picture a line at a shop: the first person to join the line is the first one served. Items come out in the *same order* they went in.

### Why `ArrayDeque`, specifically

Java's `ArrayDeque` ("Array Double-Ended Queue") efficiently implements *both* behaviors, and is the modern, preferred choice over two older alternatives, for concrete reasons worth knowing rather than just accepting as "best practice":

- **The legacy `java.util.Stack` class** predates Java's modern Collections Framework and, as a design choice that's now considered a mistake, `extends Vector` — which is *synchronized* (thread-safe via locking) by default. That locking overhead is pure waste for the overwhelmingly common single-threaded case, making it needlessly slower. It also inherits every one of `Vector`'s general-purpose methods, which undermines the whole point of a Stack being a *restricted*, LIFO-only interface.
- **`LinkedList`** can also function as a Deque, but each element requires its own separately-allocated node with extra bookkeeping (pointers to neighboring nodes) and — since those nodes aren't stored contiguously — worse memory-cache performance. `ArrayDeque` is backed by a resizable array (the same doubling-based growth strategy you fully understand from Day 3's `ArrayList` deep dive), giving it better real-world performance for this specific job.

### API

```java
import java.util.ArrayDeque;
import java.util.Deque;

// AS A STACK
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
stack.push(3);
int top = stack.peek();   // 3 — look without removing
int popped = stack.pop(); // 3 — remove and return the top

// AS A QUEUE
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1);
queue.offer(2);
queue.offer(3);
int front = queue.peek(); // 1 — look without removing
int polled = queue.poll(); // 1 — remove and return the front
```

> 💡 **Interview Insight:** "Undo the most recent action," "match the most recently opened X," "reverse a sequence using only removal from one end" → Stack. "Process in the order things arrived," "level-by-level traversal" (a preview — this becomes central once trees/graphs arrive) → Queue. Naming *which* shape a problem has, out loud, before reaching for `ArrayDeque`, is the same "connect problem structure to data structure" reasoning HashSet/HashMap asked for above — it's a pattern that repeats across this entire plan.

---

# Section 4 — Practice: HashMap and HashSet

## Practice 1: Word Frequency Counter

**Statement:** Given a sentence, count how many times each word appears.

```java
public static Map<String, Integer> wordFrequency(String sentence) {
    Map<String, Integer> freq = new HashMap<>();
    String[] words = sentence.toLowerCase().split(" ");
    for (String word : words) {
        freq.put(word, freq.getOrDefault(word, 0) + 1);
    }
    return freq;
}
```

**Design choices worth narrating:** `.toLowerCase()` before splitting, so `"The"` and `"the"` count as the same word — a real, common ambiguity worth resolving deliberately and stating out loud, rather than leaving it as an unstated assumption. `.split(" ")` is a simple splitter that assumes single-space-separated words with no punctuation — worth explicitly flagging as a simplification (real text would need a more robust split, e.g. on any whitespace and stripped punctuation) rather than silently pretending the input is always this clean.

**Complexity:** Time **O(n)** where n is the number of words — one pass, and each `put`/`getOrDefault` is average O(1). Space **O(k)** where k is the number of *distinct* words — worth stating in terms of the *right* variable (distinct words), not conflating it with `n` (total words), exactly matching Day 3's "different inputs get different variables" principle.

## Practice 2: Unique Elements

**Statement:** Given a list of numbers, return only the unique values.

```java
public static Set<Integer> uniqueElements(int[] nums) {
    Set<Integer> unique = new HashSet<>();
    for (int num : nums) {
        unique.add(num);   // duplicates are silently ignored — a Set can never hold them
    }
    return unique;
}
```

**Why this is correct with essentially no extra logic:** the defining property of a `Set` — it cannot contain duplicates — does all the actual work; `.add()` on an already-present value is simply a safe no-op. This is worth explicitly contrasting with the equivalent `ArrayList`-based approach, which would require manually checking `.contains()` before every `.add()` — correct, but both more verbose *and* asymptotically worse (an O(n) `.contains()` scan on every insertion, versus `HashSet`'s O(1) average check built directly into `.add()`).

**Complexity:** Time **O(n)** — one pass, O(1) average per insertion; Space **O(k)** where k is the number of distinct values (worst case k = n, if every element is unique).


---

# Section 5 — Stack Practice: Valid Parentheses and Three Extra Reps

Your plan's own practice problem — "check whether a string of brackets is balanced" — is, precisely, **LeetCode 20: Valid Parentheses**, one of the most common stack-recognition problems in interviewing. It's treated here with full depth, plus three additional problems using the same core technique, so the *shape* of "this is a stack problem" becomes automatic rather than tied to one specific example.

## Problem: Valid Parentheses (LeetCode 20, Easy)

**Statement:** Given a string containing only `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`, determine whether it's valid. Valid means: every opening bracket is closed by the same *type* of bracket, and brackets are closed in the correct *order* (e.g., `"([)]"` is invalid even though it has matching counts of every bracket type).

### Approach 1 — Brute force: repeatedly cancel adjacent matched pairs

```java
public static boolean isValidBruteForce(String s) {
    boolean removedSomething = true;
    while (removedSomething) {
        String before = s;
        s = s.replace("()", "").replace("[]", "").replace("{}", "");
        removedSomething = !s.equals(before);
    }
    return s.isEmpty();
}
```

Repeatedly strip out any adjacent matched pair, anywhere in the string, until a full pass removes nothing further. If the string is empty at the end, it was valid. This is *correct*, but expensive: each `.replace()` call scans the (immutable — Day 2!) string and builds an entirely new one, and the outer loop may need to repeat up to O(n) times in the worst case (e.g., `"((((()))))"` peels off one layer per pass) — giving **O(n²)** overall, with extra hidden cost from all the intermediate String allocations along the way.

### Approach 2 — Optimized: single pass with a stack

```java
public static boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '{' || c == '[') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) {
                return false;   // a closing bracket with nothing open to match
            }
            char top = stack.pop();
            boolean mismatch = (c == ')' && top != '(')
                             || (c == '}' && top != '{')
                             || (c == ']' && top != '[');
            if (mismatch) {
                return false;
            }
        }
    }
    return stack.isEmpty();     // anything left open at the end means unmatched brackets
}
```

**The reasoning to state before writing any code:** whenever a closing bracket appears, it must match the **most recently opened, still-unclosed** bracket — never an earlier one. "Most recent, still open" is precisely a **LIFO** relationship — which is exactly what a stack models directly. Pushing every opening bracket and popping on every closing bracket lets each closing bracket check itself against exactly the right candidate, in one pass, with no repeated scanning.

**Why this beats the brute force, precisely:** one linear pass, O(n), versus a potentially quadratic sequence of full-string scans-and-rebuilds. Beyond the raw complexity, the stack version also **fails fast** — it returns `false` the instant a mismatch or an unmatched closer is found, rather than continuing to rewrite the whole string on every pass regardless.

> ⚠️ **Common Mistake:** Forgetting the `stack.isEmpty()` check *before* popping — attempting to pop from an already-empty stack throws an exception (or, depending on the API, silently returns `null`/an error state), rather than correctly signaling "this closer has no match." Equally common: forgetting the *final* `stack.isEmpty()` check — a string like `"((("` never triggers a mismatch during the loop (nothing ever tries to close), so without checking the stack is empty at the very end, this invalid input would incorrectly be reported valid.

**Edge cases:** empty string (vacuously valid — the loop never runs, the stack starts and ends empty, correctly returns `true`); a string of only opening brackets (fails the final check); a string of only closing brackets (fails immediately, empty-stack check, on the very first character); odd-length input (never needs special-casing — it's structurally guaranteed to fail either the mismatch check or the final empty check, so no separate length check is needed).

**Complexity: Time O(n) — one pass; Space O(n) worst case — an all-opening-brackets string pushes every character onto the stack.**

> 💡 **Interview Insight:** This is a canonical "recognize the shape" problem. The strongest opening move isn't jumping straight to code — it's stating the LIFO observation out loud first ("closing brackets must match the most recently opened one, which is inherently last-in-first-out, so I'd reach for a stack"), *then* implementing it. That sequencing — structure-first, code-second — is exactly what separates "solved it" from "clearly understood why this approach is correct," which is the distinction tier-1 interviews are actually evaluating.

---

## Extra Practice 1: Baseball Game (LeetCode 682, Easy)

**Statement:** Given a sequence of operations — an integer (record that score), `"+"` (record a score equal to the sum of the previous two), `"D"` (record double the previous score), or `"C"` (invalidate/remove the previous score) — return the sum of all valid scores at the end.

```java
public static int calPoints(String[] operations) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (String op : operations) {
        switch (op) {
            case "+" -> {
                int top = stack.pop();
                int second = stack.peek();
                stack.push(top);            // restore what we popped to peek beneath it
                stack.push(top + second);   // then push the new score
            }
            case "D" -> stack.push(stack.peek() * 2);
            case "C" -> stack.pop();
            default -> stack.push(Integer.parseInt(op));
        }
    }
    int sum = 0;
    for (int score : stack) {
        sum += score;
    }
    return sum;
}
```

**Why this is a stack problem:** every operation cares *only* about the most recently recorded score(s) — `"D"` and `"C"` reference exactly the most recent one; `"+"` references the two most recent. "Only ever care about the most recent few" is the same LIFO signal as Valid Parentheses, wearing a different costume — a scoring simulation instead of bracket matching, but structurally identical.

**A fair question worth having an answer to: could you use an `ArrayList` instead, operating on its last index?** Yes — and in fact that *would* functionally be a stack usage pattern, just via a different concrete type. `ArrayDeque` is still the better choice here because its entire API is purpose-built around exactly this "only touch the top" access pattern, making the code's intent immediately clear and structurally preventing an accidental stray operation on a non-top element — a correctness benefit, not just a style preference.

**Complexity: Time O(n) — each operation is O(1) except the final summation, which is O(n) over however many scores remain; Space O(n).**

---

## Extra Practice 2: Implement Queue using Stacks (LeetCode 232, Easy)

**Statement:** Implement a FIFO queue using only stack operations (push/pop/peek from one end).

```java
public class MyQueue {
    private final Deque<Integer> inStack = new ArrayDeque<>();
    private final Deque<Integer> outStack = new ArrayDeque<>();

    public void push(int x) {
        inStack.push(x);
    }

    public int pop() {
        transferIfNeeded();
        return outStack.pop();
    }

    public int peek() {
        transferIfNeeded();
        return outStack.peek();
    }

    public boolean empty() {
        return inStack.isEmpty() && outStack.isEmpty();
    }

    private void transferIfNeeded() {
        if (outStack.isEmpty()) {
            while (!inStack.isEmpty()) {
                outStack.push(inStack.pop());
            }
        }
    }
}
```

**Why two stacks, and why this actually produces FIFO order:** a single stack can never produce FIFO order using only push/pop — reversing order once is fundamentally what a stack does, and one reversal alone always gives you LIFO, never FIFO. The trick is reversing **twice**: everything pushed goes onto `inStack` (LIFO order, most recent on top). The moment something needs to come *out*, if `outStack` is empty, every element is moved from `inStack` to `outStack` — and moving elements one at a time from the top of one stack to the top of another **reverses their order**. Reversed-once is LIFO; this second reversal restores the *original* insertion order — exactly FIFO — for everything transferred in that batch.

**The amortized-cost connection back to Day 3, worth stating explicitly:** a single call to `pop()` or `peek()` *can* trigger a full O(n) transfer, if `outStack` happens to be empty at that moment. But look at any *individual* element's total lifetime in this structure: it's pushed onto `inStack` exactly once (O(1)), and moved from `inStack` to `outStack` **at most once**, ever — after that, it sits in `outStack` until popped directly (O(1)), and is never moved again. Across any sequence of n operations, the *total* transfer work across all of them is bounded by O(n) — one move per element, maximum — giving an **amortized O(1)** cost per operation, in exactly the same sense as Day 3's `ArrayList.add()` analysis: individual calls vary, but the total across a sequence is what the honest complexity claim is actually about.

**Complexity: Amortized O(1) per operation; Space O(n) across both stacks combined.**

---

## Extra Practice 3: Remove All Adjacent Duplicates In String (LeetCode 1047, Easy)

**Statement:** Repeatedly remove pairs of adjacent, identical letters from a string until none remain. Return the final result.

### Approach 1 — Brute force: repeated scan-and-rebuild

Structurally identical to Valid Parentheses' brute force: repeatedly scan for any adjacent identical pair, remove it, and repeat until a full pass finds nothing left to remove. Same cost profile: up to O(n) passes, each doing O(n) work and allocating new Strings along the way — **O(n²)** overall.

### Approach 2 — Optimized: use a `StringBuilder` as the stack directly

```java
public static String removeDuplicates(String s) {
    StringBuilder stack = new StringBuilder();
    for (char c : s.toCharArray()) {
        int lastIndex = stack.length() - 1;
        if (lastIndex >= 0 && stack.charAt(lastIndex) == c) {
            stack.deleteCharAt(lastIndex);   // "pop" — the new character cancels the top
        } else {
            stack.append(c);                  // "push"
        }
    }
    return stack.toString();
}
```

**A technique worth flagging explicitly:** this uses `StringBuilder` itself *as* the stack — `.append()` is push, `.deleteCharAt(length - 1)` is pop, `.charAt(length - 1)` is peek — rather than a separate `Deque<Character>` followed by a final reversal-and-conversion step. This sidesteps an ordering headache entirely: a `Deque`-based version builds the result in *reverse* internally and needs an explicit `.reverse()` at the end (worth being able to explain *why*, if asked: popping a `Deque` yields elements top-first, which is the *opposite* of the original left-to-right order, since the top of the stack is whatever was pushed *last*). `StringBuilder` naturally maintains left-to-right order throughout, since `deleteCharAt` only ever removes from the *end*, matching the scan direction directly — a nice, concrete callback to Day 2's `StringBuilder` introduction, now solving a genuinely different kind of problem.

**Why this is the same shape as Valid Parentheses, despite looking like a different problem:** cancellation here can **cascade** — removing one pair can expose a *new* adjacent pair that wasn't adjacent before. Trace `"abba"`: push `'a'` → `"a"`; `'b'` doesn't match top (`'a'`) → push → `"ab"`; `'b'` **matches** top → pop → `"a"`; `'a'` **matches** the new top → pop → `""`. The final result is the empty string — two cancellations, the second one only possible *because* the first one exposed it. This cascading-match behavior is exactly why a stack (not a simple single left-to-right pass without one) is necessary: each cancellation can immediately re-expose the previous, not-yet-examined element, and a stack is precisely the structure that keeps that "previous element" available and correctly ordered.

**Complexity: Time O(n) — every character is pushed and popped at most once across the entire run; Space O(n) worst case (no cancellations at all, e.g. `"abcdef"`).**


---

# Section 6 — Project Block

In `java-fundamentals/collections`, add `HashSetPractice.java`, `HashMapPractice.java`, and `StackQueuePractice.java`, covering today's four exercises (word frequency, unique elements, Valid Parentheses, and at least one of the three extra stack problems — all four if time allows, since the extra reps are exactly what turns "solved it once" into genuine pattern comfort).

**Definition of done:** all three files implemented, each tested against sample inputs directly in a `main` method (not just the happy path — for Valid Parentheses specifically, test at least one valid string, one mismatched-type string like `"([)]"`, and one unmatched-closer string like `")("`), and pushed.

---

# Section 7 — Career Block

Light day by design: 20 minutes of LinkedIn engagement — meaningful comments on 3-5 posts from your target list, same bar as Day 1 (reference something specific, add a real reaction or question). No outreach task today; today's real investment is the technical depth above.

---

# Day 4 — Interview Questions

---

**1. Why does `HashSet.contains()` run in average O(1) time, versus O(n) for `ArrayList.contains()`?**

*Answer:* `ArrayList` has no choice but to scan element by element until a match is found. `HashSet` (via the same mechanism as `HashMap`) computes a hash code from the value being checked and jumps directly to the corresponding bucket — a calculation, not a search — checking only the (typically very few) entries that happen to share that bucket.

---

**2. Walk through what happens, mechanically, inside a `HashMap` when `put(key, value)` is called.**

*Answer:* A hash function computes a deterministic hash code from the key. That hash code determines which internal bucket the pair belongs in (e.g., via `hashCode % numberOfBuckets`). The pair is stored in that bucket. A later `get(key)` recomputes the identical hash code, lands on the identical bucket, and checks the (usually few) entries there using `.equals()` to find the exact match.

---

**3. What is a hash collision, and why is it unavoidable in general? How does `HashMap` handle one?**

*Answer:* A collision is two different keys hashing to the same bucket. It's unavoidable in general because there's a fixed number of buckets but a potentially unlimited variety of possible keys (the pigeonhole principle). `HashMap` handles it by allowing a bucket to hold more than one entry, checked via `.equals()` when needed — correctness is preserved, at the cost of slightly more work only for colliding entries.

---

**4. State the `equals()`/`hashCode()` contract precisely.**

*Answer:* If two objects are equal according to `.equals()`, they must produce the same `hashCode()`. The converse isn't required — unequal objects may share a hash code; that's an ordinary, harmless collision.

---

**5. What breaks, concretely, if a class overrides `.equals()` but not `hashCode()`?**

*Answer:* Two objects that are genuinely equal by the class's own `.equals()` definition can compute different hash codes and land in different buckets. A `HashMap` would then never even check the right bucket to find them as the same key — `get()` could silently return `null` for a key that was, by the class's own definition, already inserted. No crash, no exception — just quietly wrong data.

---

**6. What does `map.get(missingKey)` return, and why is unboxing it directly into a primitive dangerous?**

*Answer:* It returns `null`, not a default value. Assigning `null` directly into a primitive (`int x = map.get(missingKey);`) triggers auto-unboxing of `null`, which is impossible, and throws a `NullPointerException` at runtime.

---

**7. What does `map.put(key, map.getOrDefault(key, 0) + 1)` accomplish, and why is it written this way?**

*Answer:* It handles both the first-occurrence and subsequent-occurrence cases in one line: if `key` is absent, `getOrDefault` returns `0`, storing `1`. If present, it returns the current count, storing `count + 1`. No explicit `if/else` branch is needed for "have I seen this before."

---

**8. What's the relationship between `HashSet` and `HashMap` internally?**

*Answer:* `HashSet` is implemented as a thin wrapper around a `HashMap`, storing elements as keys and ignoring the value slot. Understanding `HashMap`'s hashing mechanism directly explains `HashSet`'s behavior.

---

**9. Distinguish Stack (LIFO) from Queue (FIFO), with an example use case for each.**

*Answer:* Stack: last item in is the first out — undo functionality, matching the most recently opened bracket. Queue: first item in is the first out — processing tasks in arrival order, level-by-level traversal (previewed, formalized later).

---

**10. Why is `ArrayDeque` preferred over the legacy `java.util.Stack` class?**

*Answer:* `Stack` extends `Vector`, which is synchronized by default — unnecessary locking overhead for typical single-threaded use — and inherits `Vector`'s full general-purpose API, undermining the intended LIFO-only restriction.

---

**11. Why is `ArrayDeque` generally preferred over `LinkedList` for stack/queue use?**

*Answer:* `LinkedList` requires a separately allocated node per element with extra pointer bookkeeping and worse memory-cache locality (scattered, not contiguous). `ArrayDeque` is backed by a resizable array with the same doubling growth strategy as `ArrayList`, giving better real-world performance for this use case.

---

**12. What signal in a problem statement suggests reaching for a stack?**

*Answer:* Any variant of "match/undo/reference the most recently seen or opened item" — a LIFO relationship. Concretely: bracket matching, undo history, or cascading adjacent-cancellation problems.

---

**13. Why is Valid Parentheses naturally a stack problem — what's the underlying LIFO relationship?**

*Answer:* Every closing bracket must match the most recently opened, still-unclosed bracket — never an earlier one. "Most recent, still open" is precisely a LIFO relationship, so pushing openers and popping on closers lets each closer check itself against exactly the right candidate in one linear pass.

---

**14. What are the two classic implementation mistakes in a stack-based Valid Parentheses solution?**

*Answer:* Forgetting to check `stack.isEmpty()` *before* popping (a closer with nothing open would otherwise error or misbehave), and forgetting to check `stack.isEmpty()` *after* the loop (a string like `"((("` never triggers a mismatch mid-scan and would incorrectly report valid without this final check).

---

**15. Why can't a single stack alone produce FIFO order? Why does using two stacks work?**

*Answer:* A stack inherently reverses order once; a single reversal always yields LIFO, never FIFO. Using two stacks and moving elements from one to the other reverses order a *second* time, which restores the original insertion order — true FIFO — for whatever's transferred.

---

**16. What's the amortized complexity of `MyQueue`'s operations (built from two stacks), and why?**

*Answer:* Amortized O(1). Any single call can trigger a full O(n) transfer if the output stack is empty, but each individual element is moved from the input stack to the output stack at most once across its entire lifetime in the structure — so total transfer work across any sequence of n operations is bounded by O(n), giving O(1) average per operation.

---

**17. In Remove Adjacent Duplicates, why can cancellations cascade, and why does that require a stack rather than a single simple pass?**

*Answer:* Removing one adjacent matching pair can expose a new pair that wasn't adjacent before removal (e.g., `"abba"` → removing the inner `"bb"` exposes an now-adjacent `"aa"`). A stack keeps the previous, not-yet-finalized character available and correctly ordered so this cascading match can be detected immediately as it's exposed.

---

**18. Why does the `StringBuilder`-as-stack approach to that problem avoid a final `.reverse()`, while a `Deque<Character>`-based version needs one?**

*Answer:* `StringBuilder.deleteCharAt(length - 1)` only ever removes from the end, so it naturally preserves left-to-right order throughout. Popping a `Deque` yields the most-recently-pushed element first — the opposite of original left-to-right order — so a `Deque`-based version must explicitly reverse the collected result at the end.

---

**19. [Debug]** This Valid Parentheses attempt fails on input `"((("`. Why?

```java
public static boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '{' || c == '[') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            stack.pop();
        }
    }
    return true;   // BUG
}
```

*Answer:* It always returns `true` at the end regardless of what's left on the stack. `"((("` never hits the `else` branch at all (no closers appear), so the loop finishes with three unmatched openers still on the stack, and the hardcoded `return true;` incorrectly reports it as valid. Fix: `return stack.isEmpty();`.

---

**20. If a problem needs "have I seen this value before" *and* also needs to remember something else about it (like the index it first appeared at), would you reach for `HashSet` or `HashMap`? Why?**

*Answer:* `HashMap` — `HashSet` only answers membership (yes/no), with no way to associate extra information with an element. The moment a problem needs to remember *anything more* than "seen or not" (an index, a count, a running value), that's the signal a `Map` is needed instead of a `Set`, even if the problem is still fundamentally about "have I seen this."

---

## Daily Deliverable Check

- [ ] Comfortable choosing between `ArrayList`/`HashSet`/`HashMap`/`ArrayDeque` based on what a problem actually asks for, and can justify the choice out loud
- [ ] `HashSetPractice`, `HashMapPractice`, `StackQueuePractice` implemented, tested against non-trivial cases, and pushed
- [ ] Can state the `equals()`/`hashCode()` contract from memory, including *why* it matters

---

## What Tomorrow Assumes You Already Know Cold

Day 5 is where everything from today gets put to direct, repeated use — the `HashMap`/`HashSet` mechanism you just learned becomes a **named interview pattern**, exercised across twelve problems. If the `getOrDefault` frequency-counting idiom, or the reasoning behind average-O(1) lookup, still takes conscious effort to reconstruct, that's worth another pass through Sections 1-2 first — tomorrow assumes reaching for `HashMap`/`HashSet` is close to automatic, not something you're re-deriving per problem.

**Next:** [Day 5 Resource Book](./Day5_Resource_Book.md) — The HashMap/HashSet Interview Pattern (12 problems) and the Four OOP Pillars.
