# Week 1 (Revised): Foundations From Scratch, HashMap/HashSet Pattern Expanded, Two Pointers Begins

**What changed from the previous draft:** every concept below is now taught directly and completely on its own terms — what it is, why it exists, how it works — with no reference to any other language as a shortcut. The schedule itself is unchanged (foundations still close in 5 days, exactly as confirmed), because that pacing was never actually dependent on the comparisons — it's dependent on this material being genuinely fast to teach well, which it is. The HashMap/HashSet pattern is still expanded to 7 problems, and Two Pointers still begins today at its expanded 16-problem count.

---

## Day 1 — What Is a Program? Environment Setup, and Control Flow

### Foundations Block (3.5 hrs)
- **What is a computer program?** A precise, ordered sequence of instructions a computer executes, one at a time, from start to finish. Nothing in the machine "understands" what the code means — it follows the instructions mechanically, exactly as written, with no judgment or interpretation involved.
- **How your code becomes something that runs:** you write source code in a `.java` file. A compiler (`javac`) translates that source into bytecode — a `.class` file, a set of instructions in a format no real CPU speaks natively. The Java Virtual Machine (JVM) then reads that bytecode and either interprets it line by line or JIT-compiles the frequently-run parts into actual machine code for your specific CPU. This two-step design — compile once to a universal bytecode format, then let a JVM installed on any machine run it — is the whole idea behind "write once, run anywhere."
- Install the JDK (use JDK 21 — later weeks use modern language features) and an IDE. IntelliJ IDEA Community Edition is recommended for this whole plan.
- Command line basics: `cd` (change directory), `ls`/`dir` (list contents), `mkdir` (make a directory) — enough to navigate confidently before Git commands are introduced.
- Write `HelloWorld.java`. Compile it with `javac HelloWorld.java`, run it with `java HelloWorld` from the terminal. Then open the same file in your IDE and run it with one click — you'll use the terminal for quick checks and the IDE for everything else from here forward.
- **Variables and primitive types:** a variable is a named location in memory holding a value of a specific type. `int` (whole numbers), `double` (decimal numbers), `boolean` (true/false), `char` (a single character) — each reserves a fixed, known amount of memory, which is part of why primitive operations are fast and predictable. Declaring, assigning, and printing with `System.out.println`.
- **Basic operators:** arithmetic (`+ - * / %`), comparison (`== != < > <= >=`). One trap worth internalizing immediately, because it causes a huge share of early bugs in any language with this syntax: `=` *assigns* a value ("make this variable hold this"), while `==` *compares* two values ("are these equal"). Writing `if (x = 5)` when you meant `if (x == 5)` is a classic, costly mistake — Java's compiler actually catches this specific case for `boolean` variables, but not for numeric ones, so the habit of double-checking still matters.
- **Why control flow matters:** a program that can only execute one fixed sequence of steps, exactly once, isn't useful for much. Real programs need to make decisions and repeat work. `if / else if / else` for decisions; the `switch` statement for choosing among several fixed options (classic syntax, then the modern Java 14+ arrow syntax `case X -> ...`, which avoids the classic version's fall-through trap). Loops: `for` (reach for this when you know the iteration count up front), `while` (when you don't), `do-while` (when the body must run at least once no matter what).
- **Methods exist to avoid repeating yourself:** once logic is written once inside a named, callable unit, it can be reused anywhere instead of copy-pasted. Declaring one (return type, name, parameter list), calling it, returning a value. Method overloading lets the same method name serve different parameter combinations, resolved by the compiler based on what you pass in.
- **Practice — building fluency, not solving LeetCode yet:**
  - FizzBuzz (1–100; multiples of 3 → "Fizz", of 5 → "Buzz", of both → "FizzBuzz").
  - A method converting Celsius to Fahrenheit.
  - A method that checks whether a number is prime — your first loop doing real decision-making work, not just repeating an action.

### Git & Environment Block (1 hr)
- **What Git is and why you need it:** a system that records snapshots of your code over time, so you can undo mistakes safely and — for your job search — show a public, consistent history of real work.
- Install Git. Configure your identity (`git config --global user.name` / `user.email`).
- Core commands for today: `git init`, `git add`, `git commit -m`, `git remote add origin`, `git push`.
- Create a GitHub account if needed. Create the `dsa-java` and `java-fundamentals` repositories.
- **Starting today: every solved LeetCode problem gets committed to `dsa-java`, organized by pattern** — e.g., `dsa-java/hashmap-hashset/two-sum/`, `dsa-java/two-pointers/valid-palindrome/`. You'll want this organized and browsable months from now when you're reviewing for interviews.
- **Task:** initialize both repos locally, commit `HelloWorld.java`, and push. This is your first proof the whole toolchain works end to end.

### Career Block (1 hr)
- Create a special repository at `github.com/[yourusername]/[yourusername]` for your GitHub profile README.
- Add a `README.md`: personal statement ("Backend engineer building expertise in distributed systems. 3 years of experience. Currently: SDE-2 transformation."), tech stack badges (shields.io), a "Currently Building" section.
- LinkedIn: Post 1 — journey announcement.
- Networking: identify 5 target Backend Engineers at companies you admire, just to start observing what they post.

### Daily Deliverable
- [ ] JDK and IDE installed and working.
- [ ] `HelloWorld.java` compiled and run from both terminal and IDE.
- [ ] Comfortable writing `if/else`, `switch`, and all three loop types from memory.
- [ ] FizzBuzz, temperature converter, and prime checker written, tested, pushed.
- [ ] Git installed and configured. `dsa-java` and `java-fundamentals` repos created and pushed with a first commit.
- [ ] GitHub profile README live. LinkedIn Post 1 published.

---

## Day 2 — Arrays, Strings, and Object-Oriented Programming

### Foundations Block (3.5 hrs)
- **What an array is:** a fixed-size, contiguous block of memory holding same-typed elements. This is *why* `arr[i]` is O(1) — the computer calculates the exact memory address directly (`base address + i × element size`) instead of searching for it.
- **Why the fixed size matters:** an array's strength (fast, predictable access) is inseparable from its limitation (you can't resize it once created) — you'll meet `ArrayList` tomorrow specifically to solve this.
- Declaring, initializing, and indexing arrays; iterating with a classic `for` loop and the enhanced `for-each`, and when each is appropriate (for-each when you don't need the index, classic `for` when you do). Two-dimensional arrays, briefly (`int[][] grid`) — just enough to recognize the shape; matrix problems arrive properly later.
- **Strings:** a `String` is an object, not a primitive, but behaves like text in everyday use. Common methods: `.length()`, `.charAt(i)`, `.substring()`, `.split()`, `.equals()`. Critically: **`==` compares whether two references point to the exact same object in memory; `.equals()` compares the actual content of two objects.** Two strings can hold identical text and still fail an `==` check if they're different objects — this single distinction trips up nearly everyone learning Java and must never be confused.
- **Practice:**
  - Find the maximum value in an array.
  - Reverse an array in place.
  - Count how many times each character appears in a string, using a plain array of size 26 for lowercase letters — deliberately the same trick Valid Anagram will use once real DSA starts, so it's already familiar.
  - Check if a string is a palindrome.
- **Why object-oriented programming:** as programs grow past a handful of functions, grouping related data and the behavior that acts on it — instead of scattering loose variables and standalone functions everywhere — keeps code organized, reusable, and easier to reason about. This is the mental model behind almost every Java library you'll touch, including the Collections Framework starting tomorrow.
- **Class vs. object:** a class is a blueprint describing what fields and methods something has; an object is one specific instance built from that blueprint, with its own actual values in memory.
- **Constructors:** the special method that runs when an object is created with `new`, responsible for putting the new object into a valid starting state. The `this` keyword refers to the current object, most commonly used to disambiguate a field from a same-named constructor/method parameter.
- **Encapsulation:** making fields `private` and exposing controlled access through public getter/setter methods. This matters because it prevents an object from ever being put into an invalid state by code outside the class — the class itself is the only thing that can touch its own internals directly.
- **Basic inheritance:** `extends` lets one class build on another, inheriting its fields and methods. `super()` calls the parent's constructor. `@Override` marks a method that replaces the parent's version with its own behavior.
- **Interfaces, briefly:** a contract listing method signatures with no implementation attached. A class that `implements` an interface promises to provide real bodies for every method in that contract. (Full depth on interfaces vs. abstract classes comes in a few days — today is just enough to *use* interfaces comfortably.)
- **Practice:**
  - A `Book` class (title, author, `isAvailable`) with a constructor, getters/setters, `checkOut()`/`returnBook()`.
  - A `Shape` interface with `area()`, implemented by `Circle` and `Rectangle`.

### Project Block (1 hr)
- Repository: `java-fundamentals`. Create an `arrays` package and an `oop` package with today's exercises.
- Definition of done: everything runs correctly via a `main` method demonstrating it; pushed with a clear commit message.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting meaningfully on posts from your target list.
- Networking: send 2–3 connection requests with a short, genuine note.

### Daily Deliverable
- [ ] Comfortable declaring, indexing, and iterating arrays.
- [ ] Know exactly when `==` breaks on Strings and why.
- [ ] Can explain class vs. object and what a constructor does, without notes.
- [ ] Array/string exercises and the `Book`/`Shape` hierarchy written, tested, pushed.

---

## Day 3 — Big-O Notation, and Your First Java Collection: ArrayList

### Foundations Block (3.5 hrs)

**Concept Card — Big-O Notation**
- **What:** a way to describe how an algorithm's time or space requirements grow as input size grows, deliberately ignoring constants and specific hardware.
- **Why:** it's the shared vocabulary every DSA hint and interview answer in this plan uses ("Time O(n) | Space O(1)") — you can't meaningfully evaluate or compare solutions without it.
- **Where:** every remaining day of this plan, and every technical interview you'll sit.
- **Today's scope:** recognize O(1), O(log n), O(n), O(n log n), O(n²) from a short code snippet and rank them by how fast they grow.

**Concept Card — ArrayList**
- **What:** a resizable list, backed internally by a plain array that grows (typically doubles in size) automatically when it runs out of room.
- **Why:** a plain array can't grow once created — `ArrayList` gives you array-like O(1) indexed access while letting the collection resize as elements are added.
- **Where:** anywhere you'd instinctively reach for "a list of things," and in a large share of the DSA problems ahead.
- **Interview signal:** need a dynamically-sized *ordered* collection → `ArrayList`, unless something more specific (a Map, a Set, a Deque) fits better.
- **Java API today:** `.add()`, `.get(i)`, `.set(i, val)`, `.remove(i)`, `.size()`, `.contains()`, for-each iteration.
- **Practice:** redo two of Day 2's array exercises using `ArrayList<Integer>` instead of a raw array. Notice what got easier (no fixed size to worry about) and what got slightly more verbose (autoboxing — Java has to wrap each `int` in an `Integer` object to store it in a generic collection).

### Project Block (1 hr)
- Repository: `java-fundamentals`. Create a `collections` package (you'll keep adding to it this week). Add `ArrayListPractice`.
- Definition of done: pushed, with a comment in your own words on why `ArrayList` can grow and a raw array can't.

### Career Block (1 hr)
- LinkedIn: Post 2 — one thing that surprised you about how Java actually runs.
- Networking: reach out to 2 college alumni at companies on your radar.

### Daily Deliverable
- [ ] Can classify a short snippet's time complexity across the five common classes.
- [ ] Comfortable with `ArrayList`'s core methods.
- [ ] LinkedIn Post 2 published.

---

## Day 4 — Sets, Maps, Stacks, and Queues: The Rest of Your DSA Toolkit

### Foundations Block (3.5 hrs)

**Concept Card — HashSet**
- **What:** a collection that holds only unique elements, backed internally by a `HashMap` (full internals arrive later — today, treat it as "a set that answers `contains()` in O(1) average time").
- **Why:** whenever a problem cares about *membership* or *uniqueness*, not order or count.
- **Where:** deduplication, "have I visited this before" in graph traversal, fast existence checks.
- **Interview signal:** "contains duplicates," "have you seen this," "unique elements" → `HashSet`.
- **Java API:** `.add()`, `.contains()`, `.remove()`, `.size()`.

**Concept Card — HashMap**
- **What:** stores key → value pairs. A hash function converts each key into a number, which maps that key toward a specific bucket in an internal array — giving average O(1) insert/lookup/delete, since you go almost directly to the right bucket instead of scanning everything.
- **Why:** whenever you need to associate information with something and look it up fast — counting frequencies, remembering "have I seen this value and at what index," grouping items by a shared property.
- **Where:** caching layers, database indexes, deduplication pipelines — and later in this plan, a Redis cache is conceptually a distributed HashMap.
- **Interview signal:** "find a pair," "count frequency," "group by," "find the complement" → HashMap is almost always the first thing worth trying.
- **Prerequisites:** solid on `.equals()` (Day 2) — Java uses `.equals()` and `.hashCode()` together to decide which bucket a key lives in and whether two keys are "the same." Full internals (buckets, treeification) come later; today is correct *usage*.
- **Java API:** `.put()`, `.get()`, `.getOrDefault()`, `.containsKey()`, iterating via `.entrySet()`.

**Concept Card — Stack and Queue (via `ArrayDeque`)**
- **What:** a Stack is Last-In-First-Out — picture a stack of plates, where you can only add or remove from the top. A Queue is First-In-First-Out — picture a line at a shop, where the first person in line is served first. In Java, `ArrayDeque` implements both efficiently and is generally preferred over the legacy `Stack` class and over `LinkedList` for this purpose.
- **Why:** any "undo the most recent thing" problem maps to a Stack; any "process things in the order they arrived" problem maps to a Queue.
- **Java API:** as a stack — `.push()`, `.pop()`, `.peek()`. As a queue — `.offer()`, `.poll()`, `.peek()`.

**Practice — previewing patterns you'll meet formally soon:**
- Count the frequency of every word in a sentence using `HashMap<String, Integer>`.
- Given a list of numbers, return only the unique ones using `HashSet`.
- Check whether a string of brackets like `"({[]})"` is balanced, using a `Deque` as a stack.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`. Add `HashSetPractice`, `HashMapPractice`, `StackQueuePractice` to `collections`.
- Definition of done: all three exercises implemented, tested with sample inputs in `main`, pushed.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3–5 posts.
- Networking: light day, no specific outreach.

### Daily Deliverable
- [ ] Comfortable choosing between `ArrayList`/`HashSet`/`HashMap`/`ArrayDeque` based on what a problem asks for.
- [ ] All three practice exercises implemented and pushed.

---

## Day 5 — The HashMap/HashSet Interview Pattern (Expanded), and the Four OOP Pillars

### DSA Block (3 hrs) — the pattern bank starts here

**Concept Card — HashMap/HashSet as an Interview Pattern**
- **What:** you used these yesterday as raw tools; today formalizes them as a recognized interview pattern family.
- **Why:** O(1) average lookup turns an O(n²) brute-force "check every pair" into O(n).
- **Interview signal:** "find a pair," "check duplicates," "count frequency," "group by."

- Problem 1: Two Sum — LeetCode #1 — Easy — Pattern: HashMap complement lookup
  - Hint: store each number's index as you scan; before inserting, check if `target - num` is already a key.
  - Complexity: Time O(n) | Space O(n)
- Problem 2: Contains Duplicate — LeetCode #217 — Easy — Pattern: HashSet membership
  - Hint: add each number to a HashSet; if it's already there, you've found a duplicate.
  - Complexity: Time O(n) | Space O(n)
- Problem 3: Valid Anagram — LeetCode #242 — Easy — Pattern: Frequency counting
  - Hint: an `int[26]` frequency array beats a HashMap here — fixed alphabet, no hashing overhead.
  - Complexity: Time O(n) | Space O(1)
- Problem 4: Ransom Note — LeetCode #383 — Easy — Pattern: Frequency counting **(new)**
  - Hint: same shape as Valid Anagram — build a frequency array from the magazine's letters, decrement as you consume letters for the note; if anything goes negative, it's impossible.
  - Complexity: Time O(n) | Space O(1)
- Problem 5: Isomorphic Strings — LeetCode #205 — Easy — Pattern: Two-way HashMap mapping **(new)**
  - Hint: one map isn't enough — the mapping must be injective both ways (two different characters in `s` can't map to the same character in `t`). Use two maps, or one map plus one "already used" set.
  - Complexity: Time O(n) | Space O(1)
- Problem 6: Group Anagrams — LeetCode #49 — Medium — Pattern: HashMap keyed by canonical form **(new)**
  - Hint: sort each string's characters to get a canonical key (`"eat"` and `"tea"` both become `"aet"`); group the original strings under that key in a HashMap.
  - Complexity: Time O(n × k log k) | Space O(n × k)
- Problem 7: Longest Consecutive Sequence — LeetCode #128 — Medium — Pattern: HashSet, smart starting point **(new)**
  - Hint: put every number in a HashSet; only start counting a sequence from a number whose `num - 1` is *not* in the set. This is the trick that keeps it O(n) instead of O(n log n) — without it, you'd recount overlapping sequences from every element.
  - Complexity: Time O(n) | Space O(n)

**All 7 committed to `dsa-java/hashmap-hashset/`, one folder per problem.**

### Theory Block (1.5 hrs)
- Topic: OOP, Layer 2 — The Four Pillars, Named
- You've already *used* encapsulation, inheritance, and interfaces since Day 2. Today: formally name and connect them — **Encapsulation** (private state + controlled access), **Inheritance** (`extends`, code reuse through hierarchy), **Polymorphism** (one interface, many implementations — your `Shape.area()` calls were already this), **Abstraction** (hiding complexity behind a simple contract).
- `enum` can hold logic, not just constant names — each constant compiles to its own class instance.
- Coding exercise: write `enum AccountType { SAVINGS, CHECKING }` with an abstract `calculateInterest()` implemented differently per constant.

### Project Block (1 hr)
- Repository: `java-fundamentals`.
- Task: build an `Account` class hierarchy using inheritance, encapsulation, and the `AccountType` enum from above.
- Definition of done: the hierarchy can't be instantiated via an abstract base class; fields are strictly private with getters/setters; pushed.

### Career Block (1 hr)
- **Accountability:** post on r/developersIndia or LinkedIn looking for an SDE-2 prep partner for weekly mock interviews. Line this up *now* — the revised plan uses mock interviews far more heavily than earlier drafts did, starting in the LLD phase, so you'll want a working partnership well before then.
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.

### Daily Deliverable
- [ ] All 7 HashMap/HashSet pattern problems solved and pushed to `dsa-java`.
- [ ] Can name and explain all four OOP pillars with your own example for each.
- [ ] `Account` hierarchy pushed. Accountability-partner post published.

---

## Day 6 — Two Pointers Begins

### DSA Block (2.5 hrs)

**Concept Card — Two Pointers**
- **What:** two indices moving through a structure — either from opposite ends inward, or both from the same end at different speeds — instead of nested loops.
- **Why:** collapses many O(n²) brute-force scans into O(n) by exploiting sorted order or a structural property.
- **Where:** merge steps in merge sort, deduplication, palindrome checks, capacity/container problems.
- **Interview signal:** sorted array + "find a pair/triplet," "reverse in place," "partition without extra space."
- **Prerequisites:** arrays and iteration ✅ (Day 2).

- Problem 1: Valid Palindrome — LeetCode #125 — Easy — Pattern: Two Pointers (opposite ends)
  - Hint: skip non-alphanumeric characters from both ends while comparing; lowercase everything first.
  - Complexity: Time O(n) | Space O(1)
- Problem 2: Reverse String — LeetCode #344 — Easy — Pattern: Two Pointers (opposite ends)
  - Hint: swap characters at `left`/`right`, move both inward until they meet.
  - Complexity: Time O(n) | Space O(1)
- Problem 3: Merge Sorted Array — LeetCode #88 — Easy — Pattern: Two Pointers (from the back) **(new)**
  - Hint: merge starting from the *back* of both arrays inward — this avoids overwriting elements in the first array before you've had a chance to read them.
  - Complexity: Time O(m+n) | Space O(1)

### Theory Block (1.5 hrs)
- Topic: SOLID Principles
- Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion.
- Coding exercise: write a `NotificationService` that violates Dependency Inversion by instantiating `EmailSender` directly, then refactor it to accept a `MessageSender` interface via constructor injection.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: apply SOLID to Day 5's `Account` hierarchy — extract a `TaxCalculator` interface so new tax rules can be added without modifying `Account`.
- Definition of done: account state and tax logic are cleanly separated; pushed.

### Career Block (1 hr)
- LinkedIn: Post 3 — one insight from the Four Pillars / SOLID work.
- Networking: send 5 connection requests with a short personal note.

### Daily Deliverable
- [ ] Valid Palindrome, Reverse String, and Merge Sorted Array solved, pushed to `dsa-java/two-pointers/`.
- [ ] Can explain all five SOLID principles with a one-line example each.
- [ ] `TaxCalculator` refactor pushed.

---

## Day 7 (Sunday) — Two Pointers Continues, and Week 1 Consolidation

### Self-Check (10 min)
- [ ] Without looking anything up: state, in one sentence each, when you'd reach for `ArrayList`, `HashSet`, `HashMap`, and `ArrayDeque`.

### DSA Block (2 hrs)
- Problem 4: Is Subsequence — LeetCode #392 — Easy — Pattern: Two Pointers (single forward pointer each) **(new)**
  - Hint: advance a pointer in `s` only when the current characters match; advance a pointer in `t` every single step. If `s`'s pointer reaches the end, `s` is a subsequence of `t`.
  - Complexity: Time O(n) | Space O(1)
- Problem 5: Valid Palindrome II — LeetCode #680 — Easy — Pattern: Two Pointers (allow one deletion) **(new)**
  - Hint: on the first mismatch, you have exactly two options — skip the left character or skip the right one. Try both; if either leaves a palindrome, it's valid.
  - Complexity: Time O(n) | Space O(1)

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 7, 7 days in, **12 total DSA problems solved** (7 HashMap/HashSet + 5 Two Pointers). Foundations fully closed in 5 days. Two Pointers is already a third of the way through its expanded 16-problem set (up from 11 in the earliest draft of this plan).

### Daily Deliverable
- [ ] Is Subsequence and Valid Palindrome II solved, pushed to `dsa-java/two-pointers/`.
- [ ] Weekly ritual and scorecard complete.

---

## Where this leaves you for Week 2

Foundations closed in 5 days, every concept taught fully on its own terms. The HashMap/HashSet pattern fully closed at 7 problems. Two Pointers already 5 of 16 problems in, all committed to `dsa-java`, organized by pattern, from Day 1 forward. Week 2 finishes Two Pointers (11 problems remaining) and begins Sliding Window.
