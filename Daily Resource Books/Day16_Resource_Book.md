# SDE-2 Resource Book Series
## Day 16 — Sliding Window with Anagrams, and Generics

**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**← Previous:** [Day 15](Day15_Resource_Book.md) &nbsp;|&nbsp; **Next →:** [Day 17](Day17_Resource_Book.md)
**Companion to:** Day 16 of `Week_03_Revised.md`

---

### Recap

Today reuses yesterday's variable-size window template (Day 15, Part B) without re-deriving it, and returns to fixed-size windows (Day 14) with a new twist: comparing the window's *contents*, not just its size. Both of today's problems lean on frequency counting, a pattern family formalized back on Day 5. Generics, today's theory topic, has no dependency on Sliding Window at all — it reaches back to Day 2 (classes, methods) and Day 3, where you used `ArrayList<T>` without yet having the formal vocabulary for what `<T>` actually means.

---

### Learning Objectives

By the end of today, without notes, you should be able to:
1. Solve Longest Repeating Character Replacement, and explain *why* it's still correct to compare against a "stale" (not-recomputed) frequency count.
2. Solve Permutation in String using a fixed-size window with frequency-array comparison.
3. Declare a generic class or method with `<T>`, and explain type erasure precisely enough to say why `new T[]` doesn't compile.
4. Choose correctly between `<? extends T>` and `<? super T>` for a given method signature, using PECS.

---

### Concept Dependency Map

```
Day 15B — Variable-size window template (established)     Day 14 — Fixed-size window (established)
        │                                                          │
        ▼                                                          ▼
Day 16: LC 424 — freq array + running maxFreq            Day 16: LC 567 — freq array, exact match
   (validity: winLen − maxFreq ≤ k)                          (window slides one char at a time)
        │                                                          │
        └────────────────── both reuse ──────────────────▶ Day 5's frequency-counting pattern
                                                                    │
                                                                    ▼
                                                         LC 1052 (Extra) — fixed window, gain-max variant

Day 2 — Classes & methods            Day 3 — ArrayList<T> (used informally)
        │                                        │
        ▼                                        ▼
Day 16: Generics — <T>, bounded wildcards, PECS, type erasure
        │
        ▼
  ResponseWrapper<T>, Pair<A,B> (Project)
```

---

## Problem 5: Longest Repeating Character Replacement

**LeetCode #424 — Medium — Pattern: Sliding Window (variable) + frequency counting**

**Statement:** given a string `s` of uppercase letters and an integer `k`, you may replace up to `k` characters with any other uppercase letter. Return the length of the longest substring you can make consist of a single repeated letter.

**Brute force:** for every substring, count letter frequencies and check whether `(length − mostFrequentCount) <= k`. O(n²) or O(n³) depending on how frequencies are recomputed. Space O(1) (26-letter alphabet).

**Optimal — variable window with a running (possibly stale) maxFreq:**

```java
public int characterReplacement(String s, int k) {
    int[] freq = new int[26];
    int left = 0, maxFreq = 0, best = 0;

    for (int right = 0; right < s.length(); right++) {
        freq[s.charAt(right) - 'A']++;
        maxFreq = Math.max(maxFreq, freq[s.charAt(right) - 'A']);

        int windowLen = right - left + 1;
        if (windowLen - maxFreq > k) {
            freq[s.charAt(left) - 'A']--;
            left++;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Why it works — this is the "prove it, don't assert it" case of the week.**

Notice two things that look suspicious at first glance: (1) when the window is invalid, we shrink by exactly **one**, not in a `while` loop until valid; (2) `maxFreq` is **never recomputed downward** after a shrink — it only ever goes up or stays the same, even though the actual most-frequent-letter count in the *current* window might genuinely be lower after removing a character.

🔑 **The proof this is still correct:** we're hunting for the *longest* valid window, so once we've established that a window of length `L` is achievable (with some `maxFreq = M` at the time, satisfying `L − M ≤ k`), we never care about accepting anything *shorter* than `L` again — we only care whether we can ever beat `L`. For the window to grow to length `L+1` and stay valid, we'd need a `maxFreq'` at that point satisfying `(L+1) − maxFreq' ≤ k`, i.e. `maxFreq' ≥ M+1` — **strictly more than the previous maxFreq.** If it isn't more, the arithmetic guarantees invalidity, so there's no case where growing past `L` succeeds without `maxFreq` also growing. This means the window's size never actually needs to shrink below its previous best — sliding both edges forward by one (rather than genuinely shrinking) loses nothing, because we were never going to accept a smaller answer anyway. The "staleness" isn't a bug tolerated for convenience; it's structurally irrelevant to a *longest*-window question.

**Trace:** `s = "AABABBA"`, `k = 1`.

| right | char | freq (nonzero) | maxFreq | winLen | valid? | action | window |
|---|---|---|---|---|---|---|---|
| 0 | A | A:1 | 1 | 1 | 1−1=0≤1 | — | [0,0] |
| 1 | A | A:2 | 2 | 2 | 2−2=0≤1 | — | [0,1] |
| 2 | B | A:2,B:1 | 2 | 3 | 3−2=1≤1 | — | [0,2] |
| 3 | A | A:3,B:1 | 3 | 4 | 4−3=1≤1 | — | [0,3] |
| 4 | B | A:3,B:2 | 3 (stale — actual current max is B:2, not 3) | 5 | 5−3=2>1 | shrink: remove `s[0]='A'`, left=1 | [1,4] |
| 5 | B | A:2,B:3 | 3 (now genuinely current) | 5 | 5−3=2>1 | shrink: remove `s[1]='A'`, left=2 | [2,5] |
| 6 | A | A:2,B:3 | 3 | 5 | 5−3=2>1 | shrink: remove `s[2]='B'`, left=3 | [3,6] |

Final window `[3,6]`, length **4**. Note the row at `right=4`: the algorithm accepted the window as "still needing a shrink" using `maxFreq=3`, even though the *actual* max frequency inside `[1,4]` ("ABAB") is only 2 — that's the staleness in action, and it doesn't produce a wrong final answer, exactly as the proof above predicts.

**Complexity:** Time O(n) — each index enters and leaves the window at most once (Day 15's total-movement argument). Space O(1) — the frequency array is fixed at 26 entries regardless of input size.

**Edge cases & mistakes:**
- ⚠️ Recomputing `maxFreq` by scanning the whole `freq` array every step "to be safe" — this still works, but costs an extra O(26) per step for zero benefit; the proof above is exactly why it's unnecessary.
- ⚠️ Using `while` instead of `if` for the shrink — harmless here (shrinking by one always restores validity in this problem, since `windowLen` and `maxFreq` change by exactly 1 in the shrink), but worth knowing it's an `if` specifically because at most one violation can accumulate per step.
- ⚠️ `k >= s.length()`: whole string can become one repeated letter; verify your loop doesn't need a special case (it doesn't).

**💡 Interview framing:** state the validity condition out loud first — "the window is valid when everything except the most frequent letter fits inside the replacement budget k." Likely follow-up: "why don't you need to shrink maxFreq back down?" — this is your cue to give the proof above; interviewers ask this specifically to check you understand the algorithm rather than having memorized it.

---

## Problem 6: Permutation in String

**LeetCode #567 — Medium — Pattern: Sliding Window (fixed size) + frequency counting**

**Statement:** given strings `s1` and `s2`, return true if any permutation of `s1` is a contiguous substring of `s2`.

**Brute force:** for every window of `s2` the size of `s1`, sort both and compare, or build a frequency map from scratch. O(n · m log m) with sorting, or O(n · m) rebuilding frequency counts each time, where `n = s2.length()`, `m = s1.length()`.

**Optimal — fixed window, frequency arrays maintained incrementally:**

```java
public boolean checkInclusion(String s1, String s2) {
    if (s1.length() > s2.length()) return false;
    int[] need = new int[26];
    int[] window = new int[26];
    for (char c : s1.toCharArray()) need[c - 'a']++;

    int windowSize = s1.length();
    for (int right = 0; right < s2.length(); right++) {
        window[s2.charAt(right) - 'a']++;
        if (right >= windowSize) {
            window[s2.charAt(right - windowSize) - 'a']--;
        }
        if (right >= windowSize - 1 && Arrays.equals(need, window)) {
            return true;
        }
    }
    return false;
}
```

**Why it works:** a window of `s2` is a permutation of `s1` if and only if the two strings have identical letter-frequency profiles — order doesn't matter, only counts. Sliding a fixed-size window one character at a time (add the incoming character, remove the outgoing one once the window is full) keeps `window` exactly synchronized with the current substring's frequencies at O(1) update cost per step.

**Trace:** `s1 = "ab"`, `s2 = "eidbaooo"`. `need = {a:1, b:1}`, `windowSize = 2`.

Stepping through: at `right=4` (`s2[4]='a'`), after adding `'a'` and removing `s2[2]='d'` (the character leaving a two-wide window), `window` holds exactly `{a:1, b:1}` — the substring `s2[3..4] = "ba"`, a permutation of `"ab"`. `Arrays.equals(need, window)` is true → return `true`.

**Complexity:** Time O(n) — each character enters and leaves the window exactly once; the 26-length array comparison is O(1) since the alphabet size is constant. Space O(1) (two fixed 26-length arrays).

**Edge cases & mistakes:**
- ⚠️ `s1.length() > s2.length()`: no window that size can even exist — handle this before the loop, not inside it.
- ⚠️ Comparing arrays with `==` instead of `Arrays.equals()` — `==` compares array *references*, not contents, and will always be false for two separately-allocated arrays. This is the array-flavored cousin of the `==` vs `.equals()` gotcha from Day 2.
- 💡 A `matches` counter (incrementing/decrementing only when a specific letter's count transitions to/from equality with `need`) avoids the O(26) comparison each step — doesn't change the asymptotic complexity (26 is already O(1)), just the constant factor. Extension material, not required here.

**💡 Interview framing:** name it immediately as "fixed-size sliding window, frequency-array equality check." Likely follow-up: "what if the alphabet weren't fixed at 26 letters?" — the frequency array becomes a HashMap, and the equality check is no longer O(1); walk through that trade-off verbally.

---

## Extra Practice: Grumpy Bookstore Owner

**LeetCode #1052 — Medium — Pattern: Sliding Window (fixed size), gain-maximization variant**

✅ **Overlap check:** absent from `00_Curriculum_Map.md`'s inventory and from `Week_04_Revised.md`. Added because it's the same fixed-size-window skeleton as LC 567 above, but solves a structurally different kind of question — "which window gives the biggest *gain*" instead of "does this exact window *match*" — a distinction genuinely worth a separate rep.

**Statement:** `customers[i]` customers arrive in minute `i`; `grumpy[i]` is 1 if the owner is grumpy that minute (grumpy customers aren't satisfied) or 0 otherwise. The owner can suppress grumpiness for one contiguous block of `minutes` minutes. Return the maximum number of satisfied customers achievable.

**Brute force:** for every possible starting minute of the suppression window, recompute total satisfied customers from scratch. O(n · minutes) ≈ O(n²) worst case.

**Optimal:**

```java
public int maxSatisfied(int[] customers, int[] grumpy, int minutes) {
    int baseline = 0;
    for (int i = 0; i < customers.length; i++) {
        if (grumpy[i] == 0) baseline += customers[i];
    }

    int windowGain = 0, maxGain = 0;
    for (int right = 0; right < customers.length; right++) {
        if (grumpy[right] == 1) windowGain += customers[right];
        if (right >= minutes) {
            if (grumpy[right - minutes] == 1) windowGain -= customers[right - minutes];
        }
        maxGain = Math.max(maxGain, windowGain);
    }
    return baseline + maxGain;
}
```

**Why it works:** split the problem in two. `baseline` is customers you satisfy *no matter what* (every minute the owner wasn't grumpy anyway). The only minutes the suppression window can possibly help are the currently-grumpy ones — so `windowGain` tracks, for a fixed-size window of length `minutes`, how many *additional* customers get satisfied by suppressing grumpiness there. Sliding that window and taking its maximum is exactly Day 14's fixed-size template, just with "gain" as the tracked quantity instead of a sum or average.

**Trace:** `customers = [1,0,1,2,1,1,7,5]`, `grumpy = [0,1,0,1,0,1,0,1]`, `minutes = 3`.

`baseline` = customers at indices where grumpy=0: `1 + 1 + 1 + 7 = 10`.

Sliding the length-3 gain window: it peaks at the window covering indices `[5,7]` (`grumpy=1,0,1` → customers 1 and 5 are "grumpy-covered" there, contributing `1 + 5 = 6`; index 6 is already non-grumpy and already in baseline). `maxGain = 6`.

Final answer: `10 + 6 = 16`.

**Complexity:** Time O(n) — one pass for baseline, one pass for the sliding gain. Space O(1).

**Edge cases & mistakes:**
- ⚠️ Double-counting: forgetting that non-grumpy minutes inside the suppression window contribute **nothing extra** (they were already in `baseline`) — only grumpy minutes inside the window are gain.
- ⚠️ `minutes >= customers.length`: the whole array is one window; the loop handles this correctly without a special case since the `right >= minutes` removal condition simply never fires.

**💡 Interview framing:** "I'll separate what's guaranteed from what's contestable — baseline satisfied customers, plus the best fixed-size window of *additional* satisfied customers I can buy with the suppression." This framing (split into a guaranteed part + an optimizable window) is a reusable move, not just a trick for this one problem.

---

## Theory: Generics

### Why generics exist

Before generics (pre-Java 5), a collection like `ArrayList` held plain `Object`s — you got no compile-time guarantee about what was inside, and every read required an explicit cast that could fail at runtime with a `ClassCastException`. Generics let a class or method be written once and used at many types, while the compiler checks type-correctness *before* the program ever runs.

### Declaring and using `<T>`

```java
public class Box<T> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

Box<String> stringBox = new Box<>();
stringBox.set("hello");
String s = stringBox.get(); // no cast needed — the compiler already knows it's a String
```

`T` is a placeholder, resolved per-instance to whatever type you parameterize with. You've already used this every time you wrote `ArrayList<Integer>` since Day 3 — today formalizes what that `<Integer>` actually does.

### Bounded type parameters

`<T extends Comparable<T>>` restricts `T` to types that implement `Comparable<T>` — this lets a generic method call `.compareTo()` on values of type `T`, which plain `<T>` wouldn't allow (the compiler only lets you call methods that *every possible T* is guaranteed to have, and unbounded `T` is only guaranteed to have `Object`'s methods).

### Wildcards — PECS

Two flavors, for when you don't need to *name* the exact type parameter, just describe a relationship to it:

```java
// ? extends T — "Producer": safe to READ T (or a subtype) out; NOT safe to add
public double sumAll(List<? extends Number> list) {
    double sum = 0;
    for (Number n : list) sum += n.doubleValue(); // fine — every element IS-A Number
    // list.add(5); // COMPILE ERROR — the list could actually be a List<Integer>,
    //               a List<Double>, etc.; the compiler can't guarantee int is safe to insert
    return sum;
}

// ? super T — "Consumer": safe to WRITE T into; reading only guarantees Object
public void fillWithInts(List<? super Integer> list) {
    for (int i = 0; i < 5; i++) list.add(i); // fine — the list is guaranteed to accept Integer or above
    // Integer x = list.get(0); // COMPILE ERROR — could be a List<Number> or List<Object>
}
```

🔑 **Key Takeaway — PECS:** **P**roducer **E**xtends, **C**onsumer **S**uper. If a parameter only *hands you* values, use `? extends`. If a parameter only *accepts* values from you, use `? super`.

### Type erasure

Generic type information exists **only at compile time**. The compiler uses it to type-check your code and to insert casts automatically where needed (e.g., `stringBox.get()` above compiles to bytecode that calls `Box.get()` returning `Object`, then inserts a cast to `String` — invisibly, because the compiler already proved it's safe). At runtime, `List<String>` and `List<Integer>` are both just `List` — there is no way, via reflection or otherwise, to ask a `List` object what type parameter it was declared with.

🔑 **Key Takeaway — why this trade-off was made:** Java generics arrived in Java 5 (2004), years after Java itself. Erasure was chosen specifically so generic code compiles down to bytecode that's **binary-compatible** with the huge amount of pre-generics code and libraries already in the wild — a generic `List` and a raw pre-Java-5 `List` are, after compilation, the same bytecode shape, so old and new code interoperate without a flag day.

**Concrete consequences, precisely:**
- **`new T[]` doesn't compile.** Java arrays are *reified* — they carry their component type at runtime and enforce it (an attempt to store the wrong type into an array throws `ArrayStoreException` at runtime). Creating an array requires the JVM to know the concrete component type *right now*, at the `new` call — but erasure means `T` isn't known at runtime, so there's nothing concrete to give the array. This is a direct clash between arrays' reified design and generics' erased design, not an arbitrary restriction.
- **`obj instanceof T` doesn't compile**, and **`T.class` doesn't exist** — both would need runtime type information about `T` that erasure has already discarded by the time the JVM runs your code.
- **Workaround, when you truly need a `T[]`:** `@SuppressWarnings("unchecked") T[] arr = (T[]) new Object[size];` — compiles, but is technically unsafe (nothing stops you from later storing something that isn't actually a `T` if the array escapes as an `Object[]` reference elsewhere); or, if you have an actual `Class<T>` token passed in, `(T[]) Array.newInstance(componentType, size)` reflectively creates a properly-typed array.

⚠️ **Common Mistake:** assuming you can overload two methods that differ only by generic type parameter, e.g. `void process(List<String> l)` and `void process(List<Integer> l)` in the same class — this is a compile error, because after erasure both signatures become `void process(List l)`, an illegal duplicate.

---

## Project Block Guide (1.5 hrs)

**Repository:** `java-fundamentals` · **Task:** `ResponseWrapper<T>` and `Pair<A, B>`

- `ResponseWrapper<T>`: fields `boolean success`, `T payload`, `long timestamp` (set via `System.currentTimeMillis()` in the constructor); getters for each. Demonstrate it with at least two different type parameters — e.g. `ResponseWrapper<String>` wrapping a message, and `ResponseWrapper<List<Integer>>` wrapping a result list — to prove the class genuinely works generically, not just for the one type you happened to write it against.
- `Pair<A, B>`: fields `A first`, `B second`; getters for each. Demonstrate with at least two different type-parameter combinations (e.g. `Pair<String, Integer>` and `Pair<Integer, Integer>`).

**Definition of done:** both classes compile, both are exercised with ≥2 distinct type parameterizations each, pushed.

---

## Career Block Guide (1 hr)

- **LinkedIn engagement (20 min):** comment on 3–5 posts. Favor posts from people at your target companies or in similar SDE-2 prep journeys — a substantive comment (not just "Great post!") is itself a small, visible signal of your own understanding.
- **Networking:** comment meaningfully on 5 posts from your *existing* connections — this is relationship maintenance, not new outreach; the goal is staying visible to people who already said yes to connecting with you.

---

## Day 16 — Interview Questions

**Q1. In Longest Repeating Character Replacement, why is it safe for `maxFreq` to go stale after a shrink?**
A: Because the question asks for the *longest* valid window — once a window of length L is known achievable, any shorter window is irrelevant, so the algorithm never needs to "notice" the window has become momentarily invalid at a smaller size; it only needs to detect when a strictly larger `maxFreq` unlocks a strictly longer valid window.

**Q2. Why is the shrink in LC 424 an `if`, not a `while`, when Day 15's template used a `while`?**
A: At most one violation can accumulate per step here, since `windowLen` and `maxFreq` both change by exactly one unit per iteration — a single shrink always restores the (stale) validity check. Day 15's problems could accumulate more than one violation per step (e.g. `zeroCount` could jump straight from valid to `k+2` in principle, though not in that specific problem either — the general template uses `while` because it's correct in general, even when a single shrink wouldn't always suffice).

**Q3. What's the difference between what Permutation in String and Longest Substring Without Repeating Characters (Day 15) each track in their window?**
A: LC 567 tracks a fixed-size window's exact frequency profile, comparing for equality against a target; LC 3 tracks a variable-size window's membership only (a HashSet), growing as long as there's no duplicate.

**Q4. What does type erasure actually erase, and when?**
A: Generic type parameter information — it exists during compilation (for type-checking and automatic cast insertion) and is discarded before/at bytecode generation; at runtime, `List<String>` and `List<Integer>` are indistinguishable, both just `List`.

**Q5. Why doesn't `new T[]` compile?**
A: Java arrays are reified (they track and enforce their component type at runtime), but erasure means the JVM has no concrete type for `T` at runtime — there's nothing to give the array to create it correctly.

**Q6. State PECS and apply it: would you use `? extends` or `? super` for a method parameter that only ever calls `.add()` on the list you're given?**
A: Producer Extends, Consumer Super. A method that only adds is *consuming* values into the list, so `? super T`.

**Q7. Why can't you overload `process(List<String>)` and `process(List<Integer>)` in the same class?**
A: After erasure both become `process(List)` — identical signatures, which the compiler rejects as a duplicate method.

---

## Daily Deliverable Check

- [ ] Longest Repeating Character Replacement (LC 424) and Permutation in String (LC 567) solved, pushed.
- [ ] **Extra:** Grumpy Bookstore Owner (LC 1052) solved, same folder.
- [ ] Can explain type erasure and why `new T[]` doesn't compile, without notes.
- [ ] Can state and apply PECS.
- [ ] `ResponseWrapper<T>` and `Pair<A, B>` pushed, each exercised with ≥2 type parameterizations.

---

### What Tomorrow Assumes You Already Know Cold

Day 17 extends LC 567's exact skeleton (fixed window, frequency-array match) into Find All Anagrams in a String, collecting every match instead of stopping at the first — that reuse will be described, not re-derived. Comparable vs. Comparator, tomorrow's theory, has no dependency on today's Generics beyond general comfort with interfaces (Day 2) — though bounded type parameters (`<T extends Comparable<T>>`, taught today) is exactly the mechanism that lets `Comparable` be written generically, a connection tomorrow's book will point back to explicitly.
