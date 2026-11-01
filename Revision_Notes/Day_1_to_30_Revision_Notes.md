# SDE-2 Quick Revision Notes — Java Foundations and Repository Work

**Purpose:** Restore working memory after a break without rereading all 16 resource books. These notes are based on the supplied `Day*_Resource_Book.md` files and `ALL_FILES_COMBINED(2).md`.

**Target session:** about **90 minutes**. Use the 2-hour version if several concepts feel unfamiliar.

---

## 1. How to use this guide

Do not turn this into another five-day reading exercise. Read a concept, look away, and explain it aloud in your own words. If you cannot explain it, mark it for a second pass instead of stopping to reread an entire resource book.

| Time | What to do |
|---:|---|
| 0–5 min | Read the mental map and the one-line takeaways below. |
| 5–52 min | Review the 16 theory topics. Aim for roughly 2–3 minutes per topic; don't redo DSA exercises. |
| 52–72 min | Review the repository commits in chronological order, using the file/class names as recall cues. |
| 72–85 min | Answer the active-recall questions without looking. |
| 85–90 min | Pick the 3 weakest topics and reopen only those exact sections in the original resource books. |

**For the 2-hour version:** spend an extra 15 minutes explaining the code from the repository out loud, and another 15 minutes on the questions you missed.

### One-line mental map

- **Arrays:** direct indexed access because position can be calculated; fixed size.
- **ArrayList:** array-backed, resizable; fast index access and amortized-fast append.
- **HashMap/HashSet:** hash to a bucket, then use equality to locate the exact key.
- **Stack/Queue:** LIFO/FIFO; `ArrayDeque` is the preferred general-purpose implementation in these notes.
- **OOP:** encapsulate state; use inheritance/polymorphism to vary behavior behind a common type.
- **SOLID:** principles that make changes safer and dependencies easier to replace or test.
- **Recursion:** ordinary method calls that build stack frames; every path must reach a base case.
- **Pass-by-value:** Java copies the argument value; for an object, that value is the reference.
- **Strings:** immutable; repeated `+=` inside a loop repeatedly copies accumulated content.
- **Collections internals:** memory layout explains operation costs and real-world performance.
- **Generics:** compile-time type safety; type parameters are erased at runtime.
- **Ordering:** `Comparable` defines natural ordering; `Comparator` defines an external ordering.
- **Exceptions:** `Throwable` splits into `Error` and `Exception`; try-with-resources closes resources safely.
- **Records/sealed types:** concise value carriers and explicitly restricted type hierarchies.
- **Threads:** separate stacks, shared heap; scheduling is nondeterministic and shared mutable state needs coordination.

---

## 2. Theory refresh — all 16 mapped days

### Day 2 — Arrays and basic OOP

**Arrays**

- An array is a fixed-size structure whose elements have one declared type. Its contiguous layout lets the runtime calculate the location of index `i` directly: `base address + i × element size`. That is why `arr[i]` is **O(1)** rather than a search.
- Valid indexes are `0` through `arr.length - 1`. `array.length` is a field; `String.length()` is a method; `ArrayList.size()` is a method.
- Default array values: numeric primitives become zero, `boolean` becomes `false`, and reference-type slots become `null`.
- Use classic `for` when you need an index, reverse traversal, or coordinated positions. Use for-each when you only need each value.
- A Java two-dimensional array is an **array of arrays**; each row can be a separate array.

**Strings and equality**

- `String` is an object, not a primitive. `String` is immutable: operations such as `toUpperCase()`, `substring()` and concatenation return a new string rather than modifying the original.
- For objects, `==` compares references (whether they refer to the same object); `.equals()` checks logical/content equality when the class implements it that way. String literals can share a pooled instance, so `==` may appear to work and still be the wrong value-comparison rule. Use `.equals()` for content equality.

**OOP building blocks**

- **Class** = blueprint; **object** = one instance with its own state.
- **Fields** hold object state; local variables belong to a method call. A **constructor** initializes a new object, has the class name, and has no return type. If you write any constructor, Java no longer supplies the default no-argument constructor automatically.
- `this.field = field` distinguishes the current object's field from a same-named parameter. `title = title` just assigns the parameter to itself.
- **Encapsulation** uses private state and controlled methods to preserve valid state/invariants; it is not merely about hiding information.
- **Inheritance:** `extends` creates a subclass; `super(...)` calls the superclass constructor and must come first in the constructor body. `@Override` lets the compiler catch a signature that does not actually override.
- **Overloading** = same method name, different parameter list, resolved at compile time. **Overriding** = subclass supplies the same method signature, selected at runtime based on the object's actual type.
- **Interface** = contract. A class can implement several interfaces, but can extend only one class. `Shape[]` can contain both `Circle` and `Rectangle`, and each object's `area()` implementation is selected at runtime: this is polymorphism.
- Java is **always pass-by-value**; the copied value for an array/object is a reference. A method can mutate the shared object, but assigning its parameter to a new object does not reassign the caller's variable. Day 9 explains the memory mechanism.

**Recall cue:** Explain why array indexing is O(1), why `a == b` is unreliable for String content, and why `this.field = field` matters.

### Day 3 — Big-O, amortized analysis, and ArrayList

**Big-O** describes the growth of work as input size grows, not exact execution time. Drop constant factors and lower-order terms: `O(2n) = O(n)` and `O(n² + n) = O(n²)`.

| Shape | Usual example |
|---|---|
| `O(1)` | Array index access |
| `O(log n)` | Repeatedly halve the search space |
| `O(n)` | One pass over input |
| `O(n log n)` | Efficient sort, then possibly a linear scan |
| `O(n²)` | Two loops nested over the same `n` values |
| `O(2ⁿ)` | Naive recursive Fibonacci (loose upper bound) |
| `O(n!)` | Enumerating every ordering/permutation |

- Nested loops over the same input often multiply: `O(n) × O(n) = O(n²)`. Separate sequential loops add: `O(n) + O(n) = O(n)`.
- State the case you mean. Interviews generally expect worst-case complexity unless specified; average case is about input distributions, not the same idea as amortized analysis.
- **Amortized analysis** averages the cost across a sequence of operations on a growing structure and gives a guarantee for the sequence, not a probability over typical inputs.
- `ArrayList` uses a backing array. `get(i)`/`set(i)` are `O(1)`. Most end `add()` calls are `O(1)`; when capacity runs out, it allocates a larger array and copies existing elements, costing `O(n)` for that particular append. Capacity doubling makes the total copies over `n` appends a geometric sum bounded by `O(n)`, hence **amortized `O(1)` append**. Growing by a fixed amount would cause repeated large copies and a worse total cost.
- `contains()` is `O(n)` because it scans. Inserting/removing at a middle/front position requires shifting elements, also `O(n)`.
- Generics do not accept primitives: use `ArrayList<Integer>`, not `ArrayList<int>`. **Autoboxing** converts `int` to `Integer`; **unboxing** converts back. Boxing creates/uses objects and can matter in large performance-sensitive loops.
- Gotcha: on `ArrayList<Integer>`, `remove(1)` chooses `remove(int index)` and removes the item at index 1. To remove the value 1, use `remove(Integer.valueOf(1))`.

**Recall cue:** Why is `ArrayList.add()` amortized O(1), not worst-case O(1)? Why do two adjacent loops not automatically mean O(n²)?

### Day 4 — HashMap, HashSet, Stack, and Queue

- **`HashSet`** stores unique elements. `add` of an already-present element changes nothing; membership checks are average `O(1)`. Use it for uniqueness and “have I seen this?” questions.
- **`HashMap<K,V>`** stores key → value. A key's `hashCode()` is used to choose a bucket; `equals()` distinguishes matching keys within the bucket. Insert/get/remove are average `O(1)`, with worse behavior possible when collisions are concentrated.
- A **collision** is when different keys land in the same bucket. It is expected and does not break correctness; it adds lookup work within the bucket.
- Contract: if `a.equals(b)` is true, `a.hashCode()` must equal `b.hashCode()`. The reverse is not required—different keys may collide. If equal objects produce different hashes, they may go to different buckets and lookup can return `null` even though an equal key was inserted.
- `map.get(missingKey)` returns `null`, not zero. `getOrDefault(key, fallback)` is useful for safe frequency counting: `map.put(word, map.getOrDefault(word, 0) + 1)`.
- **Stack = LIFO** (last in, first out); **queue = FIFO** (first in, first out). `ArrayDeque` can implement both. Common stack methods: `push`, `pop`, `peek`; deque ends can be manipulated with `addFirst`/`addLast` and `pollFirst`/`pollLast`.
- The old `Stack` class inherits from synchronized `Vector`; that synchronization overhead is usually unnecessary for a simple single-threaded stack. `ArrayDeque` is the preferred fit in these notes.

**Recall cue:** Tell the difference between `hashCode()` selecting the bucket and `equals()` confirming the key; define LIFO and FIFO without hesitation.

### Day 5 — OOP pillars, abstract classes, and enum behavior

The four pillars, expressed as design outcomes:

1. **Encapsulation:** protect state and control state changes through methods.
2. **Inheritance:** derive a specialized type from a base type to reuse/extend behavior.
3. **Polymorphism:** write to a common superclass/interface and let runtime dispatch select the object's concrete implementation.
4. **Abstraction:** expose the required contract while hiding irrelevant implementation details.

**Interface vs abstract class**

- Use an **interface** when types need to promise the same capability/contract but do not share a common stateful implementation (e.g., unrelated shapes each have `area()`). A class may implement multiple interfaces.
- Use an **abstract class** when related subclasses share fields and working methods, but one or more behaviors must be supplied by subclasses. It can hold state and constructors, cannot be instantiated directly, and uses the single class-inheritance slot.
- In the `Account` design, the base class owns common holder/balance state and deposit/withdraw rules, while subclasses implement the differing interest calculation. Callers can use an `Account` reference while the correct subclass method runs at runtime.
- An enum can have per-constant class bodies that override an abstract method. `SAVINGS` and `CHECKING` can each implement `calculateInterest(...)` differently; this is polymorphism, not a completely separate mechanism.

**Recall cue:** Why is `Account` an abstract class rather than an interface in this design? Which pillar is demonstrated by invoking `calculateInterest()` through an `Account` reference?

### Day 6 — SOLID and dependency injection

| Principle | Core meaning | Example signal |
|---|---|---|
| **S — SRP** | One reason to change per class. | Separate interest calculation from statement formatting. |
| **O — OCP** | Open for extension, closed for modification. | Add a new account subclass rather than editing a long type-checking `if/else`. |
| **L — LSP** | A subtype must preserve the behavioral expectations of its base type. | A `Square` that violates `Rectangle`'s width/height expectations is a bad substitute. |
| **I — ISP** | Clients should not depend on methods they do not use. | Split a large `Worker` interface into smaller capabilities such as `Workable` and `Eatable`. |
| **D — DIP** | High-level and low-level modules depend on abstractions, not directly on each other. | `NotificationService` depends on `MessageSender`, not hard-coded `EmailSender`. |

- **Dependency injection** means supplying a dependency from outside instead of constructing it internally. **Constructor injection** supplies it through the constructor; it makes required dependencies explicit and supports testing/replacement.
- In the `TaxCalculator` project, `TaxService` receives an `Account` and a `TaxCalculator`; the service asks the account for interest and delegates tax calculation through the interface. `StandardTaxCalculator` is one concrete strategy. This separates policy from the service that orchestrates it.

**Recall cue:** Explain how DIP improves both extensibility and unit testing. Give one behavioral LSP violation rather than defining LSP only as “is-a”.

### Day 8 — Recursion

- Recursion is a method calling itself on a smaller or simpler input. Each call creates a **new stack frame** with its own parameters, locals, return location, and bookkeeping—even when the same method calls itself.
- Every correct recursive function needs a **base case** (returns without recursing) and a **recursive case** that moves strictly closer to a base case. Reason about correctness like induction: base case is correct; assuming the smaller case is correct, show the current case builds the correct answer.
- If a base case is missing or unreachable, frames keep accumulating until `StackOverflowError`. In Java, that is an `Error` (`StackOverflowError → VirtualMachineError → Error → Throwable`), not a `RuntimeException`.
- Recursion often expresses trees, divide-and-conquer, and backtracking naturally, but uses `O(depth)` call-stack space and can overflow for deep input. Java does **not** eliminate recursive frames via tail-call optimization.
- Naive Fibonacci recomputes overlapping subproblems (e.g., `fib(2)` appears in multiple branches). Time grows exponentially (often stated loosely as `O(2ⁿ)`); the maximum simultaneous stack depth is only `O(n)`. Memoization later avoids recomputing completed subproblems.

**Recall cue:** For any recursive method, point to its base case, prove the recursive input approaches it, and state the stack-space cost.

### Day 9 — JVM stack/heap and pass-by-value

- **Stack:** per-thread; each active method call has a frame containing parameters and local variables. Primitive locals hold primitive values. Object-typed locals/parameters hold **references**.
- **Heap:** shared region where object instances and arrays live. Objects can outlive the method that created them while references remain; garbage collection can reclaim unreachable objects.
- Java always passes arguments **by value**. For a primitive, the copied value is the primitive. For an object, the copied value is the reference.
  - Mutating a field or array element through the copied reference changes the shared heap object, so the caller can observe it.
  - Reassigning the method parameter to a new object changes only the local copy of the reference, not the caller's variable.
- An array is also an object, so the same rule applies to arrays and collections. A `String` is immutable, so methods cannot mutate its character content in place.

**Recall cue:** Predict the result of three methods: `x = 50` for an `int`; `box.value = 50`; and `box = new Box()`.

### Day 10 — Primitive ranges, overflow, and Integer cache

| Type | Size | Key range/precision |
|---|---:|---|
| `byte` | 8-bit | -128 to 127 |
| `short` | 16-bit | -32,768 to 32,767 |
| `int` | 32-bit | -2³¹ to 2³¹−1 |
| `long` | 64-bit | -2⁶³ to 2⁶³−1 |
| `float` | 32-bit | About 6–7 significant decimal digits |
| `double` | 64-bit | About 15–17 significant decimal digits |

- Java's integral types are signed. Ordinary `int`/`long` arithmetic **wraps silently** on overflow; `Integer.MAX_VALUE + 1` becomes `Integer.MIN_VALUE`. If checked overflow is desired, Java has `Math.addExact()` and `Math.multiplyExact()`.
- `Integer.valueOf()`—also used by autoboxing—caches at least values `-128` through `127`. Two boxed small values may refer to the same cached object, making `==` true. Outside the cache, two equal values may be separate objects and `==` can be false. Use `.equals()` to compare values.
- `new Integer(...)` forces a fresh object and bypasses the cache; the constructor is deprecated in favor of `valueOf()`.

**Recall cue:** Why can `Integer a = 100; Integer b = 100; a == b` be true while the same comparison with 200 can be false? Why is overflow dangerous in arithmetic-heavy algorithms?

### Day 11 — String internals and performance

- Strings are immutable. An apparent modification such as `s.concat("x")` returns a new string; if the result is not stored, the original remains unchanged and the new value is effectively discarded.
- Literals use the **String pool**: equal literals can share one object. `new String("hello")` creates a separate object. This is another reason reference comparison can mislead; use `.equals()` for content.
- In a loop, `result += piece` creates a new string each iteration and copies the accumulated characters. Total copied content is `1 + 2 + ... + n`, producing `O(n²)` work.
- A compiler may use `StringBuilder` for a concatenation statement, but it does not generally turn repeated separate loop iterations into one shared accumulator. The repeated-copy cost remains.
- `StringBuilder` has a mutable resizable buffer. Appending is amortized `O(1)` because occasional resizing doubles capacity; building an entire result in a loop is therefore typically linear in total output length. It is the practical choice for repeated concatenation.

**Recall cue:** Explain why `+` can be fine for a short expression but harmful inside a large loop, and why the compiler optimization does not eliminate the loop's repeated copying.

### Day 12 — Collection internals and benchmark interpretation

| Structure | Layout | Strong fit | Cost reminders |
|---|---|---|---|
| `ArrayList` | Contiguous resizable array | Index access, append, iteration | `get/set O(1)`; end append amortized `O(1)`; insert/remove at front or middle `O(n)` due to shifting. |
| `LinkedList` | Separate doubly-linked nodes | Changes at head/tail when the node is already known | Add/remove at ends `O(1)`; `get(i) O(n)` because it traverses nodes. Per-node allocation/pointer chasing hurts locality. |
| `ArrayDeque` | Circular resizable array | Stack/queue/deque operations at both ends | Add/remove at ends amortized `O(1)`; good locality; no indexed-access role; does not allow `null`. |

- Big-O is not the whole runtime story. `ArrayList` and `ArrayDeque` have contiguous storage and good cache locality; `LinkedList` allocates nodes and follows pointers. Those constant-factor/memory-layout effects explain why a theoretically `O(1)` linked-list insertion is not automatically faster in real workloads.
- The repository benchmark inserts 100,000 items at the front. `ArrayList.add(0, x)` shifts existing elements and gives quadratic total work; `ArrayDeque.addFirst()` and `LinkedList.addFirst()` do not shift all existing elements. Use the run to compare trends, not as a reliable universal microbenchmark: runtime, JVM warm-up, machine load, and timer/reporting choices affect raw numbers.

**Recall cue:** Why is front insertion a poor `ArrayList` use case? Why can `ArrayDeque` beat `LinkedList` despite both supporting O(1)-style operations at the ends?

### Day 15 — HashMap internals and the hashCode contract

**Lookup path:** call `key.hashCode()` → spread/fold high bits into lower bits → compute bucket index → inspect that bucket and use equality to identify the exact key.

- The notes describe Java's `HashMap` index as `(n - 1) & hash`, where table capacity `n` is a power of two. Spreading (`h ^ (h >>> 16)`) lets high-bit differences affect the low bits used for bucket selection.
- Different keys can collide; a normal bucket stores entries in a linked list. If a bucket is unusually crowded, Java 8 HashMap can treeify it into a red-black tree when the bucket reaches the threshold (8) and capacity is at least 64; below that capacity it grows the table instead. A treeified bucket can untreeify at 6 entries.
- Default initial capacity in the notes: 16; default load factor: 0.75. When size exceeds capacity × load factor, the table grows (doubles) and entries are redistributed. The occasional `O(n)` resize still gives amortized `O(1)` insertion, just like `ArrayList` and `StringBuilder` resizing.
- Crucial distinction:
  - **Poor but legal hash:** a constant hash code sends every key to one bucket. Equality can still find the right key; correctness remains, but lookup becomes slow.
  - **Broken contract:** equal objects can have different hash codes, so a lookup can search a different bucket and return `null`. That is incorrect behavior, not merely slow behavior.
- For mutable keys, changing fields that participate in `equals()`/`hashCode()` after insertion can similarly make the entry difficult to retrieve; use stable key state.

**Recall cue:** Explain both the “constant hash code” case and the “equals overridden but hashCode not overridden” case. Why are the symptoms different?

### Day 16 — Generics, wildcards, and type erasure

- Generics let code be written once for many types while allowing compile-time type checking. Without them, collections exposed `Object` and callers had to cast, risking runtime `ClassCastException`.
- `<T>` is a type parameter; `Box<String>` makes the compiler treat `get()` as returning `String` without an explicit cast. Bounded parameters such as `<T extends Comparable<T>>` let code safely call methods guaranteed by the bound.
- **PECS:** **Producer Extends, Consumer Super.** Use `? extends T` when a method mainly reads values as `T` from the collection; it is not safe to add a particular subtype. Use `? super T` when the method writes `T` values into a destination; reads are only guaranteed to be `Object`.
- **Type erasure:** generic type information is mainly a compile-time construct. At runtime `List<String>` and `List<Integer>` are both the raw `List` shape. The compiler inserts appropriate casts; erasure preserves compatibility with pre-generics Java bytecode.
- Consequences: `new T[]`, `obj instanceof T`, and `T.class` do not compile because the runtime type argument is not available as a concrete type. Two overloads differing only in `List<String>` vs `List<Integer>` also collide after erasure.

**Recall cue:** Pick `extends` or `super` based on whether a method reads from or writes to a collection. Explain why `new T[]` is not just an arbitrary compiler restriction.

### Day 17 — Comparable, Comparator, TreeMap, PriorityQueue

- **`Comparable<T>` / `compareTo()`** defines the class's single natural/default ordering. It is implemented by the class being sorted.
- **`Comparator<T>` / `compare(a,b)`** defines an external ordering. You can have many orderings for the same type, including for types you do not own. Example: sort transactions by amount, then by timestamp.
- Avoid subtraction in comparators such as `a.amount - b.amount`: numeric overflow can reverse the result. Prefer `Integer.compare`, `Long.compare`, `Double.compare`, or `Comparator.comparing(...)`.
- **`TreeMap`** keeps keys sorted using a Red-Black tree, so `get/put/remove` are `O(log n)` and range/order operations are available. It pays more than HashMap's average `O(1)` lookup to maintain ordering.
- **`PriorityQueue`** is a binary min-heap by default, stored in an array: the minimum is at the root. `peek()` is `O(1)`; `offer/add` and `poll/remove` are `O(log n)` because elements sift up/down the heap. It is not a fully sorted iterator/collection; it efficiently exposes the current min (or max with a reversing comparator).

**Recall cue:** One sentence distinguishing `Comparable` and `Comparator`; one sentence distinguishing `TreeMap` from `PriorityQueue`.

### Day 18 — exception handling

```text
Throwable
├── Error                  (e.g., StackOverflowError, OutOfMemoryError)
└── Exception
    ├── checked exceptions (Exception subclasses other than RuntimeException)
    └── RuntimeException   (unchecked: e.g., NullPointerException)
```

- **Checked exceptions** must be caught or declared with `throws`; the compiler enforces this. Examples in the notes include `IOException` and `SQLException`.
- **Unchecked exceptions** (`RuntimeException` and subclasses) do not require compiler-enforced handling and often represent programming errors such as null dereference or invalid index.
- `Error` is also unchecked, but it belongs to a distinct hierarchy branch; `StackOverflowError` is not a `RuntimeException`.
- **Try-with-resources** works with `AutoCloseable`. It invokes `close()` automatically, even when the body throws, and closes multiple resources in reverse declaration order.
- If both the try body and `close()` throw, try-with-resources preserves the main exception and attaches the close failure as a **suppressed exception**. Manual `finally` cleanup can accidentally replace the original exception.
- The notes highlight a legitimate design debate: checked exceptions can force useful handling for recoverable external failures, but can also cause boilerplate or poor practices such as empty catch blocks or blanket `throws Exception`.

**Recall cue:** Draw the hierarchy, explain why `StackOverflowError` is an `Error`, and describe the suppressed-exception behavior.

### Day 28 — records and sealed classes/interfaces

- A `record`, e.g. `record Point(int x, int y) {}`, is a concise data carrier. It generates a canonical constructor, private final component fields, accessors named `x()`/`y()` (not JavaBean `getX()`), component-based `equals()`/`hashCode()`, and `toString()`. Records are implicitly `final`; they cannot extend another class, but can implement interfaces.
- A record is appropriate when a type's job is to carry a fixed set of values—not when the design requires mutable instance state or class inheritance.
- A `sealed interface` plus `permits` restricts which direct types may implement/extend the hierarchy. Each permitted direct subtype must be `final`, `sealed`, or `non-sealed`. Records are implicitly final and fit naturally as permitted variants.
- An exhaustive type-pattern `switch` can cover every permitted variant and catch a newly added case omission at compile time. **Version distinction from the resource book:** records became standard in Java 16; sealed types in Java 17; the type-pattern `switch` shown without preview flags became standard in Java 21.

**Recall cue:** What does a record generate? Why can the payment-state switch omit `default` when all permitted cases are handled? Which Java version standardizes that switch feature?

### Day 29 — threads and JVM concurrency model

- A **process** is a running program instance with its own memory space. A **thread** is a path of execution within that process.
- Extend Day 9's memory model: there is **one stack per thread**, but the **heap is shared** by the threads in the process. Separate local primitives are independent; two threads mutating the same shared object without coordination can race.
- `Runnable` describes work; `Thread` provides the mechanism to run it. `implements Runnable` is preferred in the notes because a class keeps its single `extends` slot available and work is separated from thread mechanics.
- `.start()` asks the JVM to start a new thread and execute `run()` on it. Calling `.run()` directly is an ordinary method call on the current thread; it does not start a new thread.
- The OS scheduler chooses when threads run, so print interleaving is not deterministic. `Thread.sleep(ms)` pauses the current thread for at least the requested duration but does not promise which thread runs next.
- Conceptual lifecycle: New → Runnable → Running → Blocked/Waiting → Terminated. The JVM's actual `Thread.State` enum has six values: `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, `TERMINATED`. `RUNNABLE` covers both eligible and currently executing; `BLOCKED` specifically means waiting for a monitor lock, while `WAITING` and `TIMED_WAITING` mean waiting indefinitely or with a timeout.
- `InterruptedException` is checked. If caught but not fully handled, restoring the interrupt flag with `Thread.currentThread().interrupt()` preserves the cancellation signal rather than swallowing it.
- Nondeterministic output order is not automatically data corruption. A race condition requires shared mutable state and unsynchronized competing access.

**Recall cue:** Explain why local `int i` values do not race, but a shared `Counter.count` can. Explain `.start()` vs `.run()` and list the six thread states.

---

## 3. Repository commit revision — chronological pass

Use `ALL_FILES_COMBINED(2).md` as the source of truth for this pass. The file groups the **current versions** of the Java sources by the commit that introduced the work. The goal is not to reread every line; open each named file, predict what it does, then check the implementation.

| Order / commit | File(s) to recall | What your implementation demonstrates |
|---|---|---|
| **Day 2 — `6a1fd0d`** Arrays + basic OOP | `src/arrays/ArrayReverser.java`, `CharFrequency.java`, `MaxFinder.java`, `PalindromeChecker.java`; `src/oop/Book.java`, `Shape.java`, `Circle.java`, `Rectangle.java`, `ShapeDemo.java` | Brute-force vs in-place array reversal; character-frequency array; maximum scan; palindrome check; object state/constructors/encapsulation; interface-based polymorphism through `Shape`. |
| **Day 3 — `e7fe927`** ArrayList | `src/collections/ArrayListPractice.java` | Rejects null/empty input for max; scans for maximum; reverses the list in place with left/right indices and `get`/`set`. |
| **Day 4 — `04c5b4a`** HashMap/HashSet/stack/queue | `HashMapPractice.java`, `HashSetPractice.java`, `StackQueuePractice.java` | Word-frequency map with `getOrDefault`; set-based uniqueness; valid parentheses with `ArrayDeque`; baseball-score stack; queue built from two stacks; adjacent duplicate removal with `StringBuilder` as a stack. |
| **Day 5 — `d4a94c3`** Account hierarchy | `src/oop/Account.java`, `CheckingAccount.java`, `SavingsAccount.java`, `Accounts.java` | Abstract base class owns common holder/balance/deposit/withdraw logic; subclasses supply different interest behavior; calls through `Account` demonstrate polymorphism. |
| **Day 6 — `d9d3802`, `7f583c8`** SOLID / TaxCalculator | `TaxCalculator.java`, `StandardTaxCalculator.java`, `TaxService.java`, updated `Accounts.java` | Tax calculation is an interface; standard tax is one implementation; `TaxService` composes account interest with injected tax calculation rather than hard-coding one calculator. |
| **Day 8 — `71496a7`** Recursion | `src/recursion/RecursionPractice.java` | Factorial, naive Fibonacci and digit sum; identify each base case and recursive step, and recognize repeated work in Fibonacci. |
| **Day 9 — `f9055bd`** Pass by value | `src/utils/PassByValueDemo.java` | Three contrasting cases: primitive copy unchanged in caller; mutation through a shared `Box` reference is visible; reassigning the local parameter doesn't change the caller's reference. |
| **Day 10 — `995c7b0`** Primitives/cache | `src/utils/TypesAndCache.java` | Demonstrates `int` overflow in both directions, `Integer` cache behavior for 100 vs 200, and `.equals()` as the value comparison. |
| **Day 11 — `95c891d`** String performance | `src/utils/StringPerformance.java` | Times repeated `String += i` against one accumulating `StringBuilder`. Treat measured milliseconds as an experiment on that run, not a universal benchmark result. |
| **Day 12 — `bace546`** Collection internals/benchmark | `src/collections/CollectionsBenchmark.java` and the updated `StringPerformance.java` | Compares 100,000 front insertions in `ArrayList`, `ArrayDeque`, and `LinkedList`; connects shifting vs circular-array/node insertion to the observed trend. Revisit timer units before relying on raw values. |
| **Day 15 — `fee9c9b`** hashCode contract | `src/utils/HashCodeContractDemo.java` | Three demonstrations: constant hash is slow but correct; equals/hashCode contract violation can make lookup fail; matched `equals()` and `Objects.hash(x, y)` restore correct lookup. |
| **Day 16 — `b6dddf8`** Generics | `src/generics/GenericsDemo.java` | `ResponseWrapper<T>` stores success/payload/timestamp; `Pair<A,B>` demonstrates multiple generic parameters across String and Integer uses. |
| **Day 17 — `119995c`** Ordering | `src/generics/Transaction.java`, `TransactionSorting.java` | Immutable-style transaction fields; comparator sorting by amount ascending and timestamp descending using `Double.compare` / `Long.compare`, avoiding subtraction. |
| **Day 18 — `5ebb305`** Exceptions/resources | `src/exceptions/CacheMissDemo.java` | Custom unchecked `CacheMissException`; a simulated reader implements `AutoCloseable`; try-with-resources closes even when `read()` throws, then the exception is caught. |
| **Day 28 — `4d43172`** Records/sealed types | `src/modernjava/PaymentStateDemo.java` | `PaymentState` permits `Pending`, `Success`, `Failed`, and `Cancelled` record variants; switch expression handles each state explicitly. |
| **Day 29 — `a769b6c`** Threads | `src/concurrency/ThreadInterleavingDemo.java` | Three threads share a `Runnable` task; output interleaving changes across runs; catch restores the interrupt flag after `InterruptedException`. |

### Commit-by-commit active recall (quick)

For each row above, answer three questions before opening the source: **(1)** What problem does this file solve? **(2)** What design/data-structure choice makes the solution work? **(3)** What is the key complexity or Java-language rule? Spend no more than one minute on a row unless it is one of your weak topics.

---

## 4. Active-recall checkpoint — answer without notes

1. Why does `arr[i]` take O(1) time, and why can the array not grow in place?
2. Explain the difference between `array.length`, `string.length()`, and `list.size()`.
3. What is the exact difference between `==` and `.equals()` for `String` and `Integer`?
4. Why is append to `ArrayList` amortized O(1), even though a resize can cost O(n)?
5. How do `HashMap` and `HashSet` differ? What do `hashCode()` and `equals()` each do?
6. Why does a constant `hashCode()` hurt speed without necessarily breaking correctness, while mismatched hashes for equal objects can break lookup?
7. Define the four OOP pillars, and distinguish an interface from an abstract class.
8. Give one real violation example for each SOLID principle, especially LSP and DIP.
9. What are the two required pieces of recursion? Why does unbounded recursion throw an `Error`?
10. Why is Java still pass-by-value when mutating an object through a method can be visible to the caller?
11. What happens when `Integer.MAX_VALUE + 1` is evaluated as an `int`? What is the default Integer cache range?
12. Why is repeated `String +=` in a loop O(n²), and how does `StringBuilder` change the cost?
13. When would you choose `ArrayList`, `LinkedList`, and `ArrayDeque`? Include memory layout, not only complexity labels.
14. State PECS and explain type erasure. Why does `new T[]` not compile?
15. Distinguish `Comparable` from `Comparator`, then `TreeMap` from `PriorityQueue`.
16. Draw the `Throwable` hierarchy and explain what try-with-resources does when both `read()` and `close()` fail.
17. What does a record generate? What does `sealed ... permits` guarantee? Which Java version standardizes the type-pattern switch shown in Day 28?
18. Explain one-stack-per-thread/shared-heap, `.start()` vs `.run()`, all six `Thread.State` values, and restoring interruption.

### Self-rating

After answering, label each topic:

- **Green:** can explain the mechanism without notes.
- **Yellow:** remember the definition but cannot explain “why”.
- **Red:** cannot reconstruct it.

Reopen only the Yellow/Red source sections. Do not restart the entire guide.

---

## 5. What to do immediately after this revision

You do not need to re-complete old DSA exercises before continuing your plan. Keep a small list of the three weakest theory concepts, spend up to 15 extra minutes on those sections, and then resume at your existing curriculum position. For repository code, use the commit table as a guided tour; the full source remains in `ALL_FILES_COMBINED(2).md`.
