# Day 2 Resource Book — Arrays, Strings, and Object-Oriented Programming

**Series:** Week 1 SDE-2 Prep · [← Curriculum Map](./00_Curriculum_Map.md) · [← Day 1](./Day1_Resource_Book.md) · Next: Day 3 →

**Companion to:** Day 2 of `Week_01_Revised.md`

---

## Recap: what today builds on

Day 1 gave you variables, primitive types, operators, control flow, loops, and methods — including pass-by-value for primitives. Today introduces the first two real *data structures* (arrays and, in effect, Strings) and the entire mental model behind object-oriented programming. Two things promised on Day 1 get finished today: the full `==` vs. `.equals()` picture (Strings are the first objects you'll actually use), and the full pass-by-value story for arrays and objects, which behaves differently from primitives in a way that trips up almost everyone the first time.

## Learning Objectives

By the end of today, without notes, you should be able to:

1. Declare, index, and iterate an array both ways (classic `for` and for-each), and explain precisely why `arr[i]` is a fast, constant-time operation.
2. Explain exactly when `==` breaks on Strings, why it sometimes *appears* to work, and why you should never rely on it.
3. Explain what happens when you pass an array or object into a method, and how that differs from passing a primitive.
4. Explain class vs. object, what a constructor is for, and why encapsulation matters — not just what it is.
5. Read and write a small inheritance hierarchy, correctly using `extends`, `super()`, and `@Override`.
6. Explain what an interface is and why `Shape shapes[] = {new Circle(...), new Rectangle(...)}` is meaningful.

## Concept Dependency Map for Today

```
Arrays (needs: variables, loops — Day 1)
        │
        ▼
Array iteration (classic for vs. for-each) ── needs arrays + loops
        │
        ▼
Strings as objects (needs: arrays, conceptually — String is backed by a char array internally)
        │
        ▼
== vs .equals() on Strings (needs: == from Day 1, now applied to objects instead of primitives)
        │
        ▼
Why OOP / Class vs Object (needs: nothing new — a genuinely fresh mental model)
        │
        ▼
Fields → Constructors → `this` (needs: class/object)
        │
        ▼
Encapsulation (needs: constructors, fields)
        │
        ▼
Inheritance: extends, super(), @Override (needs: class, constructors)
        │
        ▼
Interfaces (needs: class, method signatures)
        │
        ▼
Pass-by-value completed for arrays/objects (needs: arrays AND objects — this is why it waited until today)
```

---

# Section 1 — Arrays

### What an array actually is

An **array** is a fixed-size, contiguous block of memory holding elements that are all the same type. Two words in that definition are doing all the work, and both have direct consequences:

- **Contiguous** — every element sits right next to the previous one in memory, in one unbroken block, with no gaps.
- **Fixed-size** — the size is decided the moment the array is created and can never change afterward (Section 1 covers what to do when this is a problem).

### Why `arr[i]` is O(1) — the actual mechanism, not just the label

🔗 **Forward reference:** the formal meaning of "O(1)" — and the whole vocabulary of Big-O — is Day 3's entire topic. For today, take it to mean: *the amount of work this takes does not depend on how big the array is.* Whether the array has 10 elements or 10 million, accessing `arr[500]` takes the same, small, fixed amount of work. That's the property being described, and the mechanism below is *why* it's true.

Because the array is one contiguous block, the computer doesn't need to search for element `i` at all — it can **calculate** exactly where that element lives in memory, directly:

```
address of arr[i]  =  base_address  +  (i × size_of_one_element)
```

If an `int` array starts at memory address `1000`, and each `int` takes 4 bytes, then `arr[3]` lives at exactly `1000 + (3 × 4) = 1012`. The computer jumps straight there. No searching, no scanning, no dependence on how many elements exist before or after it — one arithmetic calculation, then one memory read. This is the entire reason indexed array access is fast and predictable, and it's a genuinely different mechanism from, say, searching a list for a value (which — without more information — has no choice but to potentially look at every element).

### The trade-off: this same design is also the limitation

The property that makes `arr[i]` fast — one contiguous, fixed-size block reserved up front — is *inseparable* from the property that an array can't grow. There's no "room" reserved next to it in memory to expand into; whatever sits in the next memory addresses may already belong to something else entirely. 🔗 **Forward reference:** this exact limitation is *why* `ArrayList` exists, arriving Day 3 — it gives you array-like O(1) access while handling the resizing problem for you, at a cost we'll analyze precisely once Big-O is formal.

### Declaring, initializing, indexing

```java
int[] numbers = new int[5];          // an array of 5 ints, all defaulting to 0
int[] scores = {90, 85, 78, 92, 88};  // declare AND initialize with actual values, size inferred as 5

numbers[0] = 10;      // assign to index 0
int first = numbers[0];  // read index 0 → 10
```

Arrays are **zero-indexed** — valid indices run from `0` to `length - 1`. Accessing `numbers[5]` on a 5-element array throws an `ArrayIndexOutOfBoundsException` at runtime — Java does not silently allow you to read or write outside the array's bounds, and this is a genuinely common off-by-one bug (using `<=` instead of `<` in a loop condition is the classic way to trigger it — walk through why: if `arr.length` is `5`, valid indices are `0..4`, so a loop condition of `i <= arr.length` would attempt `arr[5]` on its last pass, which doesn't exist).

> ⚠️ **Common Mistake:** `array.length` is a **field**, accessed with no parentheses. This is genuinely different from `String.length()`, which is a **method** (Section 3) — mixing these up (`array.length()` or `str.length`) is a compile error, and it's one of the most common early mistakes precisely because both concepts share the name "length" but aren't the same kind of thing.

**Default values** if you create an array without explicit values: numeric types default to `0` (or `0.0` for `double`/`float`), `boolean` defaults to `false`, and object-type arrays (like `String[]`) default every slot to `null` — not an empty string, `null`, meaning "no object at all yet." Attempting to call a method on a `null` slot (e.g., `stringArray[0].length()` before assigning it anything) throws a `NullPointerException`.

### Iterating: two ways, and when each is the right call

```java
// Classic for — use this when you need the index itself
for (int i = 0; i < numbers.length; i++) {
    System.out.println("Index " + i + ": " + numbers[i]);
}

// Enhanced for-each — use this when you only need the values, not their positions
for (int num : numbers) {
    System.out.println(num);
}
```

The for-each form is more concise and eliminates an entire class of bugs (off-by-one errors, forgetting to update `i`) — reach for it by default. Fall back to the classic `for` loop the moment you need the index for anything: comparing an element to its neighbor, writing to a *different* index than you're reading from, iterating backward, or iterating two arrays in lockstep by position. A useful rule of thumb: if you find yourself calling `numbers[i]` inside a for-each loop to work around not having `i`, that's the signal you actually needed the classic `for` loop from the start.

### Two-dimensional arrays, briefly

```java
int[][] grid = new int[3][4];   // 3 rows, 4 columns, all defaulting to 0
grid[0][0] = 1;
grid[2][3] = 99;

int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6}
};   // 2 rows, 3 columns, explicit values
```

A 2D array in Java is really an array of arrays — `grid[i]` is itself an `int[]` (one full row), and `grid[i][j]` indexes into that row. Iterating one requires a nested loop — exactly the shape flagged on Day 1 as the structural source of O(n²) work, now with a concrete data structure to attach that idea to:

```java
for (int i = 0; i < grid.length; i++) {           // rows
    for (int j = 0; j < grid[i].length; j++) {     // columns within this row
        System.out.print(grid[i][j] + " ");
    }
    System.out.println();
}
```

Matrix-specific problems arrive properly later in this plan — today's scope is just recognizing the shape and being able to declare, index, and iterate one correctly.

---

# Section 2 — Array Practice Problems

## Problem 1: Find the Maximum Value in an Array

**Statement:** Given an array of integers, return the largest value.

```java
public static int findMax(int[] arr) {
    if (arr == null || arr.length == 0) {
        throw new IllegalArgumentException("Array must not be null or empty");
    }
    int max = arr[0];
    for (int i = 1; i < arr.length; i++) {
        if (arr[i] > max) {
            max = arr[i];
        }
    }
    return max;
}
```

**Why this doesn't have a brute-force/optimized split — and why that's worth saying out loud in an interview.** Not every problem has room for a cleverer approach. Finding the maximum requires looking at every element at least once — with no additional information (the array isn't sorted, there's no auxiliary structure), any element could be the maximum, so skipping even one element risks missing it. This single O(n) pass *is* the optimal solution, and recognizing — and stating — that a problem has no further optimization available is itself a genuine interview skill. Reflexively searching for a "clever trick" where the honest answer is "this is already optimal, and here's why" is exactly the kind of well-calibrated reasoning a tier-1 interviewer is listening for.

> ⚠️ **Common Mistake:** Initializing `max` to `0` instead of `arr[0]`. This silently breaks the moment every element in the array is negative — the function would incorrectly return `0`, a value that was never actually in the array. Always seed your "running best" with a real element from the input, never an assumed baseline.

**Edge cases:** empty/`null` array (handled above by throwing — an equally valid alternative in an interview is to explicitly ask the interviewer what behavior is expected for empty input, since "throw," "return `Integer.MIN_VALUE`," and "return an `Optional<Integer>`" are all defensible depending on the contract); a single-element array (loop body never runs, correctly returns that one element); all-duplicate values (works correctly since `>` rather than `>=` doesn't matter for correctness here, only for which *index* of a tied value you'd keep, if that mattered).

**Complexity: Time O(n), Space O(1).**

---

## Problem 2: Reverse an Array In Place

**Statement:** Reverse the order of elements in an array, without allocating a second array proportional to the input size.

### Approach 1 — Brute force: build a new reversed array

```java
public static int[] reverseNewArray(int[] arr) {
    int[] result = new int[arr.length];
    for (int i = 0; i < arr.length; i++) {
        result[arr.length - 1 - i] = arr[i];
    }
    return result;
}
```

Correct, but allocates an entirely new array the same size as the input — O(n) extra space. The phrase "in place" in the problem statement is specifically ruling this approach out as the final answer, though it's a completely reasonable first thing to say out loud before optimizing.

### Approach 2 — Optimized: swap from both ends toward the middle

```java
public static void reverseInPlace(int[] arr) {
    int left = 0;
    int right = arr.length - 1;
    while (left < right) {
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```

**Why this works:** maintain two indices, one starting at each end. Swap the elements they point to, then move both inward — `left` forward, `right` backward — until they meet or cross. Each swap correctly places two elements into their final reversed positions simultaneously, and once `left` and `right` meet (or pass each other), every element has been placed exactly once.

🔗 **Forward reference:** this "two indices moving toward each other from opposite ends" technique is formalized as a named interview pattern — **Two Pointers** — starting Day 6. You've now written it once, from first principles, before it has a name; that's deliberate, and it'll make the formal pattern immediately recognizable rather than new.

**Why the `left < right` condition (not `left <= right` or `left != right`):** for an *odd*-length array, `left` and `right` eventually point to the same middle index — that element doesn't need swapping with itself, so the loop correctly stops the moment they'd meet, rather than performing a needless self-swap. For an *even*-length array, `left` and `right` cross by one and the loop stops correctly there too. Using `left <= right` would attempt one harmless-but-wasteful extra self-swap on the odd case; using `left != right` would work but is a less common, marginally less readable idiom than `<`.

**Complexity: Time O(n) — precisely n/2 swaps, but constant factors are dropped in Big-O notation (Day 3 makes this rule precise); Space O(1) — only two index variables and one temp variable, regardless of array size.**

**Edge cases:** empty array (`right` starts at `-1`, so `left < right` is `0 < -1`, false immediately — loop never runs, correctly a no-op); single element (`left == right == 0` immediately, loop never runs, correctly a no-op).

### Interview framing

> 💡 **Interview Insight:** "In place" is a term worth having a precise, ready definition for: it means modifying the existing structure using only a constant (O(1)) amount of *extra* memory, rather than allocating new memory proportional to the input size. It does **not** mean "without using any extra variables" — the `temp`, `left`, and `right` variables above are fine; what's disallowed is a second array or structure that grows with the input. Stating this definition unprompted, and explicitly contrasting the O(n) extra space of the brute-force version against the O(1) of the optimized one, is exactly the "explain the trade-off" behavior expected at this level.

*(Continued in the next section of this file — Strings and Object-Oriented Programming.)*
---

# Section 3 — Strings

### Strings are objects, not primitives

Every type you met on Day 1 (`int`, `double`, `boolean`, `char`) is a **primitive** — a raw value, stored directly. `String` is different: it's a **class**, and every piece of text you write (`"Hello, World!"`) is actually an **object** — an instance of that class, with its own internal data and its own methods. You've been using Strings since Day 1's `System.out.println("Hello, World!")` without needing this distinction yet; today it starts to matter, because objects behave differently from primitives in ways that cause real bugs if you don't know about them.

### Common String methods

```java
String name = "Claude";

int len = name.length();            // 6 — a METHOD, with parentheses (contrast: array.length, no parens)
char c = name.charAt(0);             // 'C' — the character at index 0
String sub = name.substring(1, 4);   // "lau" — from index 1 (inclusive) to 4 (exclusive)
String[] parts = "a,b,c".split(","); // ["a", "b", "c"] — splits into an array
boolean same = name.equals("Claude"); // true — compares actual CONTENT
```

`substring(start, end)` follows the same "start inclusive, end exclusive" convention you'll see constantly in Java's standard library — worth committing to memory now rather than re-deriving it each time.

### Strings are immutable — and why that's a deliberate design choice, not a limitation

**Immutable** means: once a `String` object is created, its content can never be changed. Every method that looks like it "modifies" a String — `substring`, `toUpperCase`, concatenation with `+` — actually returns a **brand-new** String object, leaving the original completely untouched:

```java
String original = "hello";
String upper = original.toUpperCase();
System.out.println(original);  // "hello" — UNCHANGED
System.out.println(upper);     // "HELLO" — a NEW String object
```

This is deliberate, and the reasons are worth knowing: immutable objects are inherently safe to share — if you pass a String to ten different methods, none of them can corrupt it for the others, because none of them *can* modify it at all, only produce new Strings. This also makes Strings safe to use as keys in a `HashMap` (arriving Day 4) — a mutable key that changed after being inserted would break the data structure's internal bookkeeping.

### `==` vs. `.equals()` — the single most important gotcha in this book so far

This is worth getting exactly right, because it's arguably *the* most common Java correctness bug among engineers coming from other languages, and it will show up in your DSA problems constantly from Day 5 onward.

```java
String a = "hello";
String b = "hello";
String c = new String("hello");

System.out.println(a == b);        // true
System.out.println(a == c);        // false
System.out.println(a.equals(c));   // true
```

**Why `a == b` is `true`:** Java maintains a special region of memory called the **String pool** (or intern pool). When you write a String **literal** (text directly in double quotes, like `"hello"`), Java first checks whether an identical String already exists in the pool. If it does, the new variable is pointed at that *same, already-existing* object rather than a fresh one being created. Since `a` and `b` are both literal `"hello"`, they end up pointing at the exact same object in memory — so `==`, which compares memory addresses (references), correctly reports `true`.

**Why `a == c` is `false`:** `new String("hello")` explicitly forces the creation of a **brand-new object**, deliberately bypassing the pool, even though its content is identical to what's already pooled. `a` and `c` now point to two different objects in memory that happen to hold the same text — so `==` (still just comparing addresses) correctly reports `false`.

**Why `a.equals(c)` is `true`:** `.equals()` is a method, and `String` specifically **overrides** it (you'll write your own overrides of methods like this once inheritance is covered later today) to compare **actual character content**, not memory addresses. This is what you almost always actually want to know: "do these two Strings hold the same text?" — not "do these two variable names happen to point at the exact same object in memory?"

> 🔑 **Key Takeaway:** `==` on any object type (not just String) compares references — "are these the same object?" `.equals()` compares content, *if and only if the class has overridden it to do so* — "do these hold equal values?" String overrides `.equals()` correctly; **never use `==` to compare String content.** The fact that `==` sometimes appears to work (any time both sides happen to be pooled literals) is precisely what makes this bug dangerous — it can pass casual testing and then fail the moment a String arrives from user input, string concatenation, `substring()`, or anywhere else that doesn't hit the pool, none of which are rare in real programs.

> ⚠️ **Common Mistake:** This exact bug is disguised inside a huge number of "why is my code returning the wrong answer" reports from people learning Java: comparing two Strings read from different sources (e.g., one hardcoded, one built by concatenation) using `==`, having it work in one test case by pool-coincidence, and fail in the next.

### String concatenation and why `StringBuilder` exists

```java
String result = "";
for (int i = 0; i < 5; i++) {
    result += i;   // looks harmless...
}
```

Because Strings are immutable, **every single `+=` here creates an entirely new String object**, copying every character accumulated so far, plus the new addition. On iteration 1, it copies ~0 characters; on iteration 2, ~1; on iteration 3, ~2; and so on. 🔗 **Forward reference:** formal Big-O notation is Day 3, but the intuitive shape here is worth having now — the *total* copying work across all iterations grows roughly like `1 + 2 + 3 + ... + n`, which grows quadratically (proportional to n²) as `n` increases, not linearly. For a handful of iterations this is invisible; inside a loop processing large input, it's a genuine, measurable performance bug.

`StringBuilder` is Java's mutable, resizable alternative, purpose-built for exactly this situation:

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 5; i++) {
    sb.append(i);   // mutates ONE internal buffer in place — no repeated full copying
}
String result = sb.toString();   // convert to a String only once, at the end
```

`StringBuilder` maintains one internal, resizable buffer and appends to it directly, avoiding the repeated-copy trap entirely. **Rule of thumb going forward:** if you're building up a String piece-by-piece inside a loop, reach for `StringBuilder`; if you're just combining a small, fixed number of known pieces once (like `"Hello, " + name + "!"`), plain `+` concatenation is perfectly fine and more readable — the compiler even optimizes a single chained expression like that into an equivalent `StringBuilder` automatically under the hood.

> 💡 **Interview Insight:** Being asked "what's the time complexity of building this string in a loop with `+=`?" and correctly identifying the hidden O(n²) behavior — then naming `StringBuilder` as the O(n) fix — is a genuine, common tier-1 signal. It's a small piece of knowledge that reliably separates "has written a lot of Java" from "is learning Java."

---

# Section 4 — String Practice Problems

## Problem 3: Character Frequency Count

**Statement:** Given a lowercase string, count how many times each character appears.

```java
public static int[] countCharacterFrequency(String s) {
    int[] freq = new int[26];
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        freq[c - 'a']++;
    }
    return freq;
}
```

**The trick worth fully understanding, not just copying:** `char` values are, underneath, just numbers — specifically, Unicode code points (16-bit values). `'a'` is code point 97, `'b'` is 98, and so on through `'z'` at 122. Subtracting `'a'` from any lowercase letter gives you that letter's **position in the alphabet**, zero-indexed: `'a' - 'a' = 0`, `'b' - 'a' = 1`, ..., `'z' - 'a' = 25`. This maps every lowercase letter onto exactly the valid index range of a 26-element array — `freq[c - 'a']` is therefore always a valid, correct index for any lowercase input character. This exact trick — mapping a small, fixed alphabet onto array indices via character arithmetic — is one of the most reused building blocks in string/array interview problems.

**Edge cases:** uppercase letters or non-letter characters would compute an index outside `0-25` (or even negative), causing an `ArrayIndexOutOfBoundsException` — worth stating explicitly as an assumption ("I'm assuming lowercase-only input per the problem statement; here's what I'd add if that's not guaranteed") rather than silently ignoring the possibility. A robust guard, if needed: `Character.toLowerCase(c)` before the subtraction, plus a check that the character is actually a letter.

**Complexity: Time O(n) where n is the string's length — one pass; Space O(1) — the array is always exactly 26 elements regardless of input size, so its size doesn't grow with n. (This "fixed-size regardless of input" property is exactly what makes something O(1) space rather than O(n) — filing this away now will make Day 3's formal treatment land faster.)**

🔗 **Forward reference:** this exact frequency-counting technique, applied to comparing two strings' letter counts, *is* the optimized solution to **Valid Anagram (LeetCode 242)**, arriving as one of Day 5's core HashMap/HashSet-pattern problems. You've already built the core mechanism.

---

## Problem 4: Palindrome Check

**Statement:** Given a string, determine whether it reads the same forward and backward.

### Approach 1 — Brute force: reverse and compare

```java
public static boolean isPalindromeBruteForce(String s) {
    String reversed = new StringBuilder(s).reverse().toString();
    return s.equals(reversed);
}
```

Correct and short — `StringBuilder` has a built-in `.reverse()` method. But it allocates a full second String (and an intermediate `StringBuilder`) the same size as the input — O(n) extra space — purely to answer a yes/no question.

### Approach 2 — Optimized: two pointers from both ends

```java
public static boolean isPalindrome(String s) {
    int left = 0;
    int right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

**Why this works:** a string is a palindrome exactly when its first character matches its last, its second matches its second-to-last, and so on, working inward. Checking each such pair directly — without ever constructing a reversed copy — confirms the same property with no extra allocation.

**Why this is strictly better, not just different:** beyond the O(1) vs. O(n) space difference, this version can **exit early** the instant a mismatch is found (`return false` on the first bad pair), whereas the brute-force version must fully build the reversed string *before* it can compare anything. For an input that fails early (e.g., the very first and last characters already don't match), the optimized version does a small, fixed amount of work; the brute-force version always does the full O(n) reversal regardless.

🔗 **Forward reference:** this is your second independent derivation of the Two Pointers shape today (the first was array reversal, Problem 2) — a strong sign it's a genuinely fundamental technique, not a one-off trick. **Valid Palindrome (LeetCode 125)**, one of Day 6's core Two Pointers problems, is a direct extension of exactly this — with the added complications of ignoring non-alphanumeric characters and ignoring case, which this simple version deliberately doesn't handle yet.

**Edge cases:** empty string (`right = -1`, so `left < right` is `0 < -1`, false immediately — an empty string is trivially considered a palindrome, which matches the mathematical definition even if it feels like a strange case to reason about); single character (trivially a palindrome, loop never runs); case sensitivity — is `"Aa"` a palindrome? This simple version says no (`'A' != 'a'` as distinct char codes), which is a legitimate, definable behavior *as long as you state it as an assumption* rather than leaving it ambiguous — exactly the kind of scope question worth asking an interviewer up front rather than guessing.

**Complexity: Time O(n) — worst case (no early exit) still looks at roughly half the characters; Space O(1).**


---

# Section 5 — Why Object-Oriented Programming

### The problem OOP actually solves

Consider modeling a library book using only what Day 1 gives you:

```java
String bookTitle = "Clean Code";
String bookAuthor = "Robert Martin";
boolean bookIsAvailable = true;

public static void checkOutBook(String title, String author, boolean isAvailable) {
    // ...but this can't actually change the ORIGINAL variables (pass-by-value for primitives/booleans!),
    // so this design is already broken before it even gets more complicated
}
```

This falls apart almost immediately. There's no way to represent *one specific book* as a single thing — its title, author, and availability are three completely separate, loosely-related variables that only stay associated with each other by convention and careful naming, not by any enforced structure. Model two books, and you need `bookTitle2`, `bookAuthor2`, `bookIsAvailable2` — the problem doesn't scale; it multiplies.

**Object-oriented programming's core idea:** bundle related data and the behavior that operates on that data into a single unit — an **object**. A `Book` object *is* its title, author, and availability, together, as one coherent thing, with its own methods for the actions that make sense on it (`checkOut()`, `returnBook()`). Model a second book, and it's simply a second, independent `Book` object — no numbered variable suffixes, no risk of accidentally mixing up which title belongs to which availability flag.

### Class vs. Object — the distinction to have perfectly crisp

- A **class** is a *blueprint* — it describes what fields (data) and methods (behavior) something of that type will have. It does not, by itself, represent any actual book.
- An **object** is a specific *instance* built from that blueprint, with actual values filled in. `new Book("Clean Code", "Robert Martin")` creates one real object; you could create a hundred more `Book` objects from the same `Book` class, each with its own independent title, author, and availability.

> 🔑 **Key Takeaway:** "Class" is to "object" as "cookie cutter" is to "cookie." One cutter (class) can stamp out many cookies (objects), each a separate physical thing, all sharing the same shape (structure) but each with its own independent existence.

---

# Section 6 — Fields, Constructors, and `this`

```java
public class Book {
    // FIELDS (instance variables) — every Book object gets its OWN independent copy of these
    private String title;
    private String author;
    private boolean isAvailable;

    // CONSTRUCTOR — runs automatically when `new Book(...)` executes
    public Book(String title, String author) {
        this.title = title;
        this.author = author;
        this.isAvailable = true;   // every new book starts out available — a sensible default
    }
}
```

### Fields

**Fields** (also called instance variables) are variables declared directly inside a class, outside any method. Unlike a local variable inside a method (which disappears the moment the method returns, per Day 1), a field lives for as long as the object itself does — it's part of the object's permanent state.

### Constructors

A **constructor** is a special method that runs automatically the moment `new` creates an object, and its job is to put that new object into a valid starting state. Two rules define it structurally: it has **exactly the same name as the class**, and it has **no return type at all** — not even `void`.

If you don't write any constructor yourself, Java silently provides a free, empty, no-argument one. **The moment you write even one constructor of your own, that free default disappears entirely** — if you also want a no-argument constructor available, you now have to write it explicitly. This is a genuinely common early mistake: adding a parameterized constructor, then being surprised that `new Book()` (no arguments) no longer compiles.

### The `this` keyword — and the bug it exists to prevent

`this` refers to *the current object* — the specific instance whose method or constructor is currently executing. Its most common job is resolving exactly the naming collision in the constructor above: the parameter is named `title`, and the field is *also* named `title` (deliberately — this is standard, idiomatic Java, not a mistake). Inside the constructor body, a bare `title` refers to the closer one — the **parameter**, because Java resolves names from the innermost scope outward. `this.title` explicitly means "the field belonging to this object," disambiguating it from the parameter.

```java
// WITHOUT this — a real, classic bug:
public Book(String title, String author) {
    title = title;    // assigns the parameter to itself. Does NOTHING. The field is never touched,
                       // and stays at its default value (null) forever.
}

// WITH this — correct:
public Book(String title, String author) {
    this.title = title;   // assigns the PARAMETER's value into the FIELD. This is the actual point.
}
```

> ⚠️ **Common Mistake:** `title = title;` compiles without any error or warning — it's completely valid Java, it's just useless. This is exactly the kind of bug that's silent and easy to miss precisely because nothing about it looks wrong at a glance; it only becomes visible when the field mysteriously stays `null` at runtime. Always reach for `this.field = parameter;` whenever a constructor or method parameter shares a name with a field.

---

# Section 7 — Encapsulation

**Encapsulation** means keeping a class's fields `private` (inaccessible directly from outside the class) and exposing controlled access through public methods — conventionally, **getters** (read a value) and, where appropriate, **setters** (change a value).

```java
public class Book {
    private String title;
    private boolean isAvailable;

    public Book(String title) {
        this.title = title;
        this.isAvailable = true;
    }

    public String getTitle() {
        return title;
    }

    public boolean isAvailable() {     // convention: boolean getters are named isXxx(), not getXxx()
        return isAvailable;
    }

    public void checkOut() {
        if (!isAvailable) {
            System.out.println("Book is already checked out.");
            return;
        }
        isAvailable = false;
    }

    public void returnBook() {
        isAvailable = true;
    }
}
```

### Why this matters — not just what it is

Notice there's **no public setter for `title`** at all — once a `Book` is constructed, its title can never be changed from outside the class, by design. And notice `isAvailable` is never set *directly* from outside either — it only ever changes through `checkOut()` and `returnBook()`, which means **every** state change to availability passes through code that can enforce rules (here: you can't check out an already-checked-out book). If `isAvailable` were a public field, any code anywhere could write `myBook.isAvailable = true;` at any time, bypassing every rule entirely — there'd be no way to guarantee the object ever stays in a valid, consistent state.

> 🔑 **Key Takeaway:** encapsulation isn't about hiding data out of secrecy — it's about **controlling how an object's state can change**, so the class itself can guarantee its own invariants (rules that must always hold true) instead of trusting every piece of code that ever touches it to behave correctly. A class with only getters and no setters at all — every field set once in the constructor and never changed again — is called **immutable**. You've already met exactly this idea once today: `String` is immutable for precisely this reason.

---

# Section 8 — Inheritance

### `extends`, fields shared with subclasses, and `super()`

Inheritance lets one class (a **subclass**) acquire the fields and methods of another (a **superclass**), then add or change behavior on top:

```java
public class Employee {
    protected String name;
    protected double baseSalary;

    public Employee(String name, double baseSalary) {
        this.name = name;
        this.baseSalary = baseSalary;
    }

    public double calculatePay() {
        return baseSalary;
    }
}

public class Manager extends Employee {
    private double bonus;

    public Manager(String name, double baseSalary, double bonus) {
        super(name, baseSalary);   // calls Employee's constructor to initialize the inherited fields
        this.bonus = bonus;
    }

    @Override
    public double calculatePay() {
        return baseSalary + bonus;   // baseSalary is accessible here because it's `protected`, not `private`
    }
}
```

**A new access modifier: `protected`.** `private` fields are invisible even to subclasses — `Manager` couldn't read `baseSalary` at all if `Employee` had declared it `private`. `protected` opens visibility specifically to subclasses (and other classes in the same package), while still blocking arbitrary outside code — a middle ground between `private`'s full lock-down and `public`'s complete openness.

| Modifier | Same class | Same package | Subclass (different package) | Everywhere |
|---|:---:|:---:|:---:|:---:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(default, no keyword)* | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**`super(...)`** explicitly calls the parent class's constructor, and if used, **must be the first statement** in the subclass constructor. If you don't write a `super(...)` call yourself, Java automatically inserts an implicit, no-argument `super()` as the very first line — which fails to compile if the parent class doesn't actually have a no-argument constructor available (a real, common error message worth recognizing on sight: *"implicit super constructor Employee() is undefined"* — the fix is almost always to add an explicit `super(...)` call with the right arguments).

### `@Override` — and closing the loop from Day 1

`Manager.calculatePay()` **overrides** `Employee.calculatePay()` — same method signature, a new implementation in the subclass. The `@Override` annotation isn't strictly required by the compiler for this to work, but writing it is a strong, universal best practice: it tells the compiler "I intend this to override a parent method," so if you make a typo in the signature (wrong parameter types, wrong name) and accidentally create a brand-new, unrelated method instead of actually overriding anything, the compiler catches that mistake immediately — instead of you silently ending up with a bug where the "override" never actually runs.

> 🔗 **Forward reference resolved:** Day 1 flagged that **overloading** and **overriding** are commonly confused and promised the distinction properly once inheritance existed. Now it can be stated precisely:
> - **Overloading** — same method *name*, different parameter list, all within the *same* class. Resolved at **compile time**, based on the declared types of the arguments at the call site (**static binding**).
> - **Overriding** — same method *signature* (name **and** parameters), redefined in a **subclass**. Resolved at **runtime**, based on the actual type of the object the method is called on (**dynamic binding**) — this is the mechanism behind polymorphism, which you'll see demonstrated concretely in the next section and formalized as one of OOP's four pillars on Day 5.

---

# Section 9 — Interfaces (Introductory)

An **interface** defines a *contract* — a set of method signatures with no implementation — that any class choosing to `implements` it must fulfill completely, or the code will not compile:

```java
public interface Shape {
    double area();
}

public class Circle implements Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle implements Shape {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }
}
```

An interface **cannot be instantiated directly** — `new Shape()` will not compile, because `Shape` has no actual implementation to run; only concrete classes that implement it (like `Circle` and `Rectangle`) can be constructed. A class can `implements` **multiple** interfaces at once (unlike `extends`, which only ever allows **one** direct superclass) — a flexibility worth noting now, with the full design implications deferred to when this plan covers interfaces vs. abstract classes in depth.

### Why this matters immediately — a concrete look at polymorphism

```java
Shape[] shapes = { new Circle(5), new Rectangle(3, 4) };
for (Shape s : shapes) {
    System.out.println(s.area());   // calls the CORRECT area() for whichever actual object this is
}
```

This loop doesn't know or care whether each element is actually a `Circle` or a `Rectangle` — it only knows each one *is a* `Shape`, and every `Shape` guarantees an `area()` method exists. At runtime, Java calls whichever concrete class's `area()` actually belongs to that specific object — `Circle`'s formula for the `Circle`, `Rectangle`'s for the `Rectangle` — automatically, via the same dynamic-binding mechanism from Section 8's `@Override` discussion. This — treating different concrete types uniformly through a shared contract — is **polymorphism**, one of OOP's four pillars, formally named and fully explored on Day 5. You've now built and used it once, concretely, before the label — same deliberate approach as the Two Pointers preview earlier today.


---

# Section 10 — Completing the Pass-by-Value Story (Arrays and Objects)

Day 1 promised this once arrays and objects existed to demonstrate it with. Now they do, and this is worth reading slowly — it's the single most common correctness bug for engineers arriving in Java from languages with different reference semantics.

**Java is *always* pass-by-value. There is no exception, ever — not even here.** What changes for arrays and objects is *what the "value" actually is*. For a primitive, the value is the raw data itself (a copy of the number). For an array or object, the value is a **reference** — conceptually, a memory address pointing to where the actual data lives. That reference is what gets copied, not the underlying array or object itself.

```java
public static void modifyArray(int[] arr) {
    arr[0] = 999;              // mutates the object THROUGH the reference — the shared data changes
}

public static void reassignArray(int[] arr) {
    arr = new int[]{1, 2, 3};  // reassigns the LOCAL COPY of the reference to point elsewhere — the
                                // caller's reference is completely untouched by this
}

public static void main(String[] args) {
    int[] myArray = {10, 20, 30};

    modifyArray(myArray);
    System.out.println(myArray[0]);   // 999 — the mutation IS visible to the caller!

    reassignArray(myArray);
    System.out.println(myArray[0]);   // still 999 — the reassignment did NOT propagate back
}
```

### Why — visualized

```
main():        myArray ────────┐
                                │
                                ▼
                         [10, 20, 30]     ← the actual array object, living in memory
                                ▲
                                │
modifyArray(arr):          arr ┘   ← arr is a COPY of the reference, but it points to the SAME object

    arr[0] = 999   →   reaches through the reference and changes the SHARED object
                        → myArray[0] is 999 too, because there was only ever ONE array all along


reassignArray(arr):     arr ──────► [1, 2, 3]   ← a brand NEW object
                          (this only changes what the LOCAL "arr" points to)

    main()'s "myArray" was never touched — it still points to the ORIGINAL [10, 20, 30]-turned-[999,20,30] object
```

### The rule, stated precisely

- **Mutating an object's contents *through* a reference** (`arr[0] = 999`, or calling a method on an object that changes its internal fields, like `myBook.checkOut()`) **is visible to the caller**, because caller and callee are working with two separate references that both point at the *same* underlying object.
- **Reassigning the parameter itself** to point at a different object entirely (`arr = new int[]{...}`) **is never visible to the caller**, because that only ever changes the method's own *local copy* of the reference — it cannot reach back and change what variable the caller is holding.

> 💡 **Interview Insight:** A very standard interview question tests exactly this: "if I pass an `ArrayList` into a method and the method calls `.add()` on it, does the caller see the new element? What if the method instead does `list = new ArrayList<>();` inside itself?" The correct answer, with the *reasoning* above (not just the memorized yes/no), is precisely the kind of "explain why, not just what" depth this whole series is built around.

> My Answer: "Java is pass-by-value. When an object is passed to a method, the value being copied is the reference to the object. Therefore, both the caller's variable and the method parameter initially refer to the same ArrayList object. Calling add() mutates that shared object, so the caller sees the change. However, assigning list = new ArrayList<>() only changes the method's local copy of the reference. The caller's reference still points to the original ArrayList, so it is unaffected.".

---

# Section 11 — OOP Practice Problems

## Practice 1: The `Book` Class

Full implementation, combining everything from Sections 6-7:

```java
public class Book {
    private String title;
    private String author;
    private boolean isAvailable;

    public Book(String title, String author) {
        this.title = title;
        this.author = author;
        this.isAvailable = true;
    }

    public String getTitle() {
        return title;
    }

    public String getAuthor() {
        return author;
    }

    public boolean isAvailable() {
        return isAvailable;
    }

    public void checkOut() {
        if (!isAvailable) {
            System.out.println(title + " is already checked out.");
            return;
        }
        isAvailable = false;
        System.out.println(title + " checked out successfully.");
    }

    public void returnBook() {
        isAvailable = true;
        System.out.println(title + " returned successfully.");
    }

    public static void main(String[] args) {
        Book book = new Book("Clean Code", "Robert Martin");
        System.out.println(book.getTitle() + " by " + book.getAuthor());
        book.checkOut();
        book.checkOut();     // should print "already checked out" — proves the guard works
        book.returnBook();
        book.checkOut();     // should succeed again — proves state actually reset
    }
}
```

**Design points worth being able to narrate:** `title` and `author` have getters but no setters — once a book is created, its identity shouldn't change, so this class is *partially* immutable by design (only `isAvailable` is mutable, and only through controlled methods, never directly). The `checkOut()` guard against double-checkout is exactly encapsulation earning its keep in a genuinely realistic way: it's protecting an invariant ("a book cannot be checked out twice in a row without being returned first") that a public boolean field could never enforce.

## Practice 2: The `Shape` Interface Hierarchy

Already built in Section 9 (`Shape`, `Circle`, `Rectangle`) — here's the demonstration tying it together with a `main` method, worth running yourself to see polymorphism actually execute rather than just reading about it:

```java
public class ShapeDemo {
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle(5),
            new Rectangle(3, 4),
            new Circle(2)
        };

        double totalArea = 0;
        for (Shape s : shapes) {
            System.out.println("Area: " + s.area());
            totalArea += s.area();
        }
        System.out.println("Total area: " + totalArea);
    }
}
```

Notice the loop body never mentions `Circle` or `Rectangle` by name at all — it's written entirely in terms of the `Shape` contract, yet correctly computes each shape's specific area. Adding a third shape type later (a `Triangle`, say) would require zero changes to this loop — it would simply start working, as long as `Triangle implements Shape` correctly. This is the practical payoff of programming against an interface rather than concrete classes, and it's worth being able to state as a deliberate design benefit, not just an incidental feature.

---

# Section 12 — Project Block

In your `java-fundamentals` repository, create two packages:

- `arrays/` — `MaxFinder.java`, `ArrayReverser.java`, `CharFrequency.java`, `PalindromeChecker.java` (or combined sensibly — either is fine as long as it's organized and each piece is easy to find)
- `oop/` — `Book.java`, `Shape.java`, `Circle.java`, `Rectangle.java`, `ShapeDemo.java`

**Definition of done** (same bar as Day 1): everything compiles and runs correctly, demonstrated via a `main` method that actually exercises the interesting cases (not just the happy path — show the double-checkout guard working, show polymorphism iterating mixed shape types), committed with a clear message, and pushed.

---

# Section 13 — Career Block

### LinkedIn engagement (≈20 minutes)

Spend this block **commenting**, not posting. On 5-8 posts from your target-engineer list (identified Day 1), leave comments that reference something *specific* in the post and add a genuine reaction, question, or related detail of your own — "Great post!" or "So true!" is functionally invisible and does nothing for your visibility; a comment that references a specific detail from the post and adds something real gets noticed, both by the poster and by anyone else reading the thread.

### Networking — first outreach

Send 2-3 connection requests to people from your target list, each with a short, genuine note (not a template-feeling copy-paste): who you are, one specific, real reason you're reaching out to *them specifically* (something from their content, their role, or their company — not "I'd love to connect!" with nothing behind it), and a low-pressure framing — you're not asking for a referral on message one; you're starting a real connection.


---

# Day 2 — Interview Questions

Same protocol as Day 1: cover the answer, answer out loud, then check yourself.

---

**1. Explain precisely why `arr[i]` is a constant-time operation.**

*Answer:* Because the array is stored as one contiguous block in memory, the address of any element can be directly calculated — `base_address + (i × size_of_one_element)` — rather than searched for. This calculation takes the same small amount of work regardless of the array's size or which index is being accessed.

---

**2. What trade-off is inseparable from that same fast-access property?**

*Answer:* The array must be a fixed size, decided at creation. The contiguous-block layout that makes indexed access fast leaves no guaranteed room to expand into, so the array cannot grow — this is exactly why `ArrayList` exists.

---

**3. When would you choose a classic `for` loop over a for-each loop for iterating an array?**

*Answer:* Whenever the index itself is needed — comparing an element to a neighbor, writing to a different index than you're reading, iterating backward, or walking two arrays in lockstep by position. Default to for-each otherwise; it's more concise and eliminates off-by-one risk.

---

**4. What's the difference between `array.length` and `string.length()`?**

*Answer:* `array.length` is a field (no parentheses) — a direct property of the array object. `string.length()` is a method call (parentheses required) on the `String` class. Confusing the two is a compile error.

---

**5. `==` on Strings is described as unreliable, yet `"a" == "a"` often evaluates to `true`. Why?**

*Answer:* Java's String pool: identical String literals are reused as the same object rather than each creating a new one, so two variables both assigned the literal `"a"` end up pointing at the same pooled object, and `==` (which compares references) reports `true`. This is exactly what makes the bug dangerous — it appears to work by coincidence for literals, then fails the moment a String comes from a non-pooled source like concatenation or user input.

---

**6. What does `new String("hello")` do differently from writing the literal `"hello"`?**

*Answer:* It explicitly forces creation of a brand-new object outside the String pool, even though an identical pooled literal may already exist — so `==` against a pooled `"hello"` will be `false`, while `.equals()` will still correctly report `true`, since it compares content.

---

**7. Why is String immutable, and what's the actual benefit?**

*Answer:* Every "modifying" operation (substring, concatenation, case conversion) returns a new String rather than changing the original. This makes Strings inherently safe to share across code without risk of one piece of code corrupting a String another piece of code is relying on, and makes them safe to use as HashMap keys, since a key that could mutate after insertion would break the structure's internal bookkeeping.

---

**8. What's the hidden performance issue with `result += x` inside a loop, and what's the fix?**

*Answer:* Because Strings are immutable, each `+=` creates an entirely new String, copying everything accumulated so far. Across n iterations, total copying work grows roughly quadratically (like 1+2+...+n), not linearly. `StringBuilder` fixes this by mutating one internal buffer in place, giving linear total work.

---

**9. What's the difference between a class and an object?**

*Answer:* A class is a blueprint describing what fields and methods something will have. An object is a specific instance built from that blueprint, with actual values — many independent objects can be created from one class.

---

**10. What is a constructor's job? What happens if you define a parameterized constructor and someone then calls `new Book()` with no arguments?**

*Answer:* A constructor puts a newly created object into a valid starting state. Java only provides a free, no-argument default constructor if you define *no* constructor at all — the moment you write any constructor yourself, that free one disappears, so `new Book()` would fail to compile unless a no-argument constructor is also explicitly defined.

---

**11. Why write `this.title = title;` instead of just `title = title;` inside a constructor?**

*Answer:* When a parameter shares a name with a field, a bare reference resolves to the closer scope — the parameter. `title = title;` assigns the parameter to itself and does nothing; the field is never touched and silently keeps its default value. `this.title` explicitly targets the field, disambiguating it from the parameter.

---

**12. What does encapsulation actually protect against? Give a concrete example.**

*Answer:* It prevents an object from being placed into an invalid state by code outside the class, by routing all state changes through methods that can enforce rules. Example: `Book`'s `isAvailable` field has no public setter — it can only change via `checkOut()`/`returnBook()`, which means the "can't check out an already-checked-out book" rule is guaranteed to hold, something a public boolean field could never enforce.

---

**13. Difference between `private`, `protected`, and `public`?**

*Answer:* `private` — visible only within the same class. `protected` — visible within the same class, same package, and subclasses even in other packages. `public` — visible everywhere.

---

**14. What does `super(...)` do, and why must it be the first statement in a subclass constructor?**

*Answer:* It explicitly calls the parent class's constructor, to properly initialize the inherited fields before the subclass adds its own. It must come first because a subclass object can't have its own state set up meaningfully before its inherited (parent) state is valid. If omitted, Java inserts an implicit no-argument `super()` automatically — which fails to compile if the parent has no no-argument constructor.

---

**15. Distinguish method overloading from method overriding.**

*Answer:* Overloading: same method name, different parameter lists, within the same class — resolved at compile time based on argument types (static binding). Overriding: same signature, redefined in a subclass — resolved at runtime based on the object's actual type (dynamic binding), which is the mechanism behind polymorphism.

---

**16. Why can't you write `new Shape()` when `Shape` is an interface?**

*Answer:* An interface only declares method signatures with no implementation — there's no actual behavior to run. Only concrete classes that `implements` the interface (and provide real method bodies) can be instantiated.

---

**17. [Code reading]** Given `Shape[] shapes = {new Circle(5), new Rectangle(3,4)};` and a loop calling `s.area()` on each — explain what determines which `area()` implementation actually runs for each element.

*Answer:* The loop is written entirely in terms of the `Shape` interface, but at runtime, Java dispatches to whichever concrete class's `area()` actually belongs to that specific object — `Circle`'s implementation for the `Circle` instance, `Rectangle`'s for the `Rectangle` instance — via dynamic binding. This is polymorphism: uniform code, type-specific behavior, resolved at runtime.

---

**18. In terms of what the caller observes, what's the difference between mutating an object through a method parameter versus reassigning that parameter to a new object?**

*Answer:* Mutating the object's contents through the reference (e.g. `arr[0] = 999`) is visible to the caller, because caller and method share a reference to the same underlying object. Reassigning the parameter itself to point at a different object only changes the method's local copy of the reference and is never visible to the caller — Java is always pass-by-value, and for objects, the "value" being passed is the reference itself, not the object's contents.

---

**19. [Debug]** What's wrong here, and what's the fix?

```java
String userInput = scanner.nextLine();
if (userInput == "quit") {
    System.out.println("Goodbye!");
}
```

*Answer:* `userInput` comes from user input, not a literal, so it will not be the same pooled object as the literal `"quit"` even if the text matches exactly — `==` compares references, so this comparison can silently fail even when the content is identical. Fix: `userInput.equals("quit")` (or, to also guard against a `null` `userInput`, `"quit".equals(userInput)`, which is null-safe since it calls `.equals()` on the known-non-null literal).

---

## Daily Deliverable Check

- [ ] Comfortable declaring, indexing, and iterating arrays both ways
- [ ] Can state exactly when `==` breaks on Strings and why, without hedging
- [ ] Can explain class vs. object and a constructor's purpose without notes
- [ ] Array/string exercises and the Book/Shape hierarchy written, tested, and pushed to `java-fundamentals`

---

## What Tomorrow Assumes You Already Know Cold

Day 3 builds directly on: arrays (declaring, indexing, iterating, the O(1)-access intuition) and loops. The informal "O(1)" language used throughout today's array discussion gets its full, formal treatment tomorrow — if array access still feels like something you're taking on faith rather than something you could re-derive from the memory-address argument in Section 1, that's worth revisiting before moving on, since Day 3 builds the entire vocabulary of algorithmic efficiency on top of exactly that intuition.

**Next:** [Day 3 Resource Book](./Day3_Resource_Book.md) — Big-O Notation and ArrayList.
