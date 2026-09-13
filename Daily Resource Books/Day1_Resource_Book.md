# Day 1 Resource Book — What Is a Program? Environment Setup, and Control Flow

**Series:** Week 1 SDE-2 Prep · [← Curriculum Map](./00_Curriculum_Map.md) · Next: Day 2 →

**Companion to:** Day 1 of `Week_01_Revised.md`

---

## How to use this book

This is Day 1 of a from-zero foundation. You have professional software engineering experience, but for the purposes of this series, treat yourself as someone who has never written a line of code — every concept is built from nothing, and nothing is used before it's explained. That discipline matters more than it might seem: tier-1 interviewers can tell within two sentences whether you actually understand *why* something works or you've just memorized *that* it works, and the only way to reliably land on the right side of that line is to actually build the foundation properly once.

**Today's shape:** you'll go from "what even is a program" to writing, compiling, and running real Java, making decisions and repeating work in code, packaging logic into reusable methods, and applying all of it to three practice problems that are treated with the same rigor this series will later give to LeetCode problems — because the reasoning skill is identical, only the difficulty changes.

---

## Learning Objectives

By the end of today, you should be able to, without notes:

1. Explain what happens between writing Java source code and a program actually running, including the role of the compiler and the JVM.
2. Declare and use the four core primitive types correctly, and explain what "type" means and why it matters.
3. Explain the difference between `=` and `==`, and why confusing them causes bugs.
4. Write `if/else`, `switch` (both forms), and all three loop types from memory.
5. Write a method with parameters and a return type, and explain what "pass by value" means for primitives.
6. Solve FizzBuzz, a temperature converter, and a prime checker — and explain, out loud, why your prime checker is efficient.
7. Explain what Git is for and run the core commands to get code onto GitHub.

---

## Concept Dependency Map for Today

```
What is a program? (mechanical execution, no "understanding")
        │
        ▼
Source code → Compiler → Bytecode → JVM → Running program
        │
        ▼
Dev environment (JDK, IDE, terminal) ── enables you to actually run the above
        │
        ▼
HelloWorld.java (class, main method, System.out.println)
        │
        ▼
Variables & primitive types (int, double, boolean, char)
        │
        ▼
Operators (arithmetic, comparison, assignment, logical)
        │
        ▼
Control flow: if/else, switch  ──┐
        │                        │  (both need operators to form conditions)
        ▼                        │
Loops: for, while, do-while  ◄───┘
        │
        ▼
Methods (bundle the above into reusable, named units)
        │
        ▼
Practice: FizzBuzz · Celsius→Fahrenheit · Prime Checker
```

Nothing above requires arrays, objects, or Strings-as-objects — those are Day 2. Where Java's standard library requires you to *type* something that looks like an object (e.g. `"Hello, World!"` is technically a `String` object), you'll be told just enough to proceed correctly, with an explicit note that the full explanation is coming on Day 2. This is intentional — flagging it explicitly is safer than silently glossing over it.

---

# Section 1 — What Is a Program?

A **computer program** is a precise, ordered sequence of instructions that a computer executes one at a time, from start to finish (with the ability to jump around based on conditions, which is what control flow — Section 7 — is for).

The single most important mental model to internalize before writing a line of code:

> 🔑 **Key Takeaway:** The computer does not understand anything. It has no judgment, no intent-reading, no "I think you meant." It executes exactly what you wrote, exactly as written, mechanically. Every bug you will ever write comes down to this: you told the machine to do something precisely, and it did precisely that — just not what you actually wanted.

This reframing is more useful than it sounds. When code doesn't work the way you expect, the productive question is never "why is the computer being weird" — it's "what did I actually tell it to do, versus what did I mean to tell it to do." Debugging, at every level of seniority, is this same question asked more carefully.

A program is built out of a small set of building blocks, and literally everything you'll ever write in any language is some combination of these:

- **Sequence** — do this, then this, then this (the default: one line after another).
- **Selection** — do this *or* that, depending on a condition (Section 7: `if`/`switch`).
- **Iteration** — do this *repeatedly*, until some condition changes (Section 8: loops).
- **Abstraction** — give a chunk of logic a name so you can reuse it without rewriting it (Section 9: methods).

Every program you will ever read or write — from FizzBuzz to a distributed database — is these four things, composed and nested. Keeping that in mind will make even unfamiliar code less intimidating: you're never looking at something conceptually new, just an unfamiliar composition of these four ideas.

---

# Section 2 — From Source Code to a Running Program

You don't write instructions a CPU understands directly (that would be raw machine code — sequences of 1s and 0s specific to one exact processor architecture). Instead, you write **source code** in a human-readable language, and something translates it. How Java does this translation is one of its defining design decisions, and interviewers do ask about it, so it's worth understanding precisely rather than approximately.

### The pipeline

```
YourFile.java  ──(javac, the compiler)──►  YourFile.class  ──(JVM)──►  Program runs
 (source code)                              (bytecode)                (on YOUR specific CPU)
```

1. **You write source code** in a file ending in `.java`. This is for humans — the compiler and eventually you, six months later, debugging it.
2. **`javac` (the Java compiler) translates it into bytecode**, a `.class` file. Bytecode is not source code and it is not machine code for any real CPU — it's an intermediate format, a set of instructions for an *imaginary* machine that doesn't physically exist.
3. **The JVM (Java Virtual Machine) reads that bytecode and executes it** on your actual, physical CPU. It does this two ways, both of which happen in the same run of your program:
   - **Interpretation** — the JVM reads bytecode instructions one at a time and executes them immediately. This starts your program fast, but is slower for code that runs many, many times.
   - **JIT (Just-In-Time) compilation** — the JVM watches which parts of your bytecode run *frequently* (a "hot" loop, for example) and compiles *those specific parts* directly into real machine code for your exact CPU, so subsequent runs of that hot code are as fast as if you'd written it in a compiled language like C. This happens automatically, while your program is running — hence "just in time."

### Why go through this two-step translation at all?

This is the part that actually gets asked in interviews, so internalize the *reason*, not just the mechanism: **a `.class` file full of bytecode is identical no matter what operating system or CPU produced it.** The JVM is the only thing that differs per platform — there's a Windows JVM, a Mac JVM, a Linux JVM, each one knowing how to translate the *same* bytecode into instructions for *that* specific machine.

This is the entire idea behind Java's famous slogan: **"write once, run anywhere."** You compile your `.java` file into bytecode exactly once. That single `.class` file will run correctly on any machine that has a JVM installed — Windows, Mac, Linux, a server, an embedded device — without you recompiling anything. Contrast this with a language that compiles straight to native machine code: you'd need a separate compiled binary for every target platform.

> 💡 **Interview Insight:** If asked "what's the difference between compiled and interpreted languages, and where does Java fit?" — the correct answer is that Java is *both*. It's compiled from source to bytecode (that's the `javac` step), and that bytecode is then interpreted (and selectively JIT-compiled) at runtime by the JVM. It doesn't cleanly fit the traditional "compiled vs. interpreted" binary, and saying so, with the reasoning above, signals real understanding rather than a memorized label.

### JDK vs. JRE vs. JVM — a distinction worth having crisp

These three terms get used loosely, but they mean different things:

| Term | What it is |
|---|---|
| **JVM** (Java Virtual Machine) | The engine that actually executes bytecode. This is the "runs anywhere" part. |
| **JRE** (Java Runtime Environment) | JVM + the standard library classes your compiled programs need to actually run (things like `System`, which you'll use in the next section). |
| **JDK** (Java Development Kit) | JRE + the tools you need to *develop* Java: the compiler (`javac`), a debugger, and more. |

You need the **JDK** to write and compile Java. Anyone who only wants to *run* an already-compiled Java program technically only needs the JRE — though in practice today, almost everyone just installs the JDK.

---

# Section 3 — Setting Up Your Environment

### Install the JDK

Install **JDK 21** (a Long-Term Support release — later weeks of your plan use modern language features that need it). Get it from [Adoptium (Eclipse Temurin)](https://adoptium.net) or [Oracle's JDK page](https://www.oracle.com/java/technologies/downloads/) — pick the installer for your OS and run it. Confirm it worked by opening a terminal and running:

```bash
java -version
javac -version
```

Both should report version 21.

### Install an IDE

**IntelliJ IDEA Community Edition** (free) from [jetbrains.com/idea](https://www.jetbrains.com/idea/download/) is recommended for this whole plan. An IDE (Integrated Development Environment) gives you autocomplete, in-line error checking before you even compile, a debugger with breakpoints, and one-click run — all of which meaningfully speed up learning, since you get feedback in seconds instead of after a full manual compile cycle.

### Command line basics

You'll use the terminal constantly — for quick compiles today, and for Git from here forward. Four commands to be fluent in before moving on:

| Command | Does | Example |
|---|---|---|
| `pwd` | Print working directory — "where am I right now?" | `pwd` |
| `cd` | Change directory | `cd Documents/dsa-java` (go in), `cd ..` (go up one level) |
| `ls` (Mac/Linux) / `dir` (Windows) | List the contents of the current directory | `ls` |
| `mkdir` | Make a new directory | `mkdir dsa-java` |

> ⚠️ **Common Mistake:** Running a command and getting "command not found" or "no such file or directory" almost always means you're not where you think you are. Run `pwd` first, then `ls`, to see reality before assuming the tool is broken.

---

# Section 4 — Your First Program

Create a file named exactly `HelloWorld.java` and write:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

### Breaking down every single token

This looks like ceremony right now. It stops looking like ceremony once you know what each piece is actually for — and every piece here is something you'll use in literally every Java program you ever write, so it's worth five careful minutes now.

- **`public class HelloWorld`** — declares a class named `HelloWorld`. A class is a blueprint/container (full depth on Day 2); for now, think of it as "the named box that holds this program's code." `public` means this class is visible/usable from outside this file.
- **The filename must be `HelloWorld.java`, exactly matching the public class name, including capitalization.** This isn't a style convention — it's a hard rule the compiler enforces. A public class named `HelloWorld` living in a file called `hello.java` or `Helloworld.java` will fail to compile.
- **`public static void main(String[] args)`** — this exact signature is the designated entry point. When you run a Java program, the JVM looks specifically for a method matching this signature and starts execution there.
  - `public` — visible from outside the class, which the JVM needs in order to call it from outside your program entirely.
  - `static` — belongs to the class itself, not to any particular object built from it. (This will click fully once objects exist, Day 2 — for now: it means you can run this method without first creating a `HelloWorld` object, which matters because *nothing exists yet* when your program is just starting up.)
  - `void` — this method doesn't return a value back to whatever called it. (Methods that *do* return values are in Section 9.)
  - `main` — the name the JVM specifically looks for. Not a convention you could rename — it's a hard requirement.
  - `(String[] args)` — a parameter: an array of `String`s (command-line arguments passed in when you run the program). You won't use these for a while, but the JVM requires this exact parameter to exist in your entry point.
- **`System.out.println("Hello, World!")`** — prints text to the console, followed by a new line.
  - `System` is a built-in class the JDK provides.
  - `out` is a stream (specifically, `System.out`) representing "standard output" — your terminal/console.
  - `println` is a method on that stream that writes text and then moves to a new line (`print` does the same without the new line).
  - `"Hello, World!"` — text wrapped in double quotes like this is called a **String** in Java. 🔗 **Forward reference:** Strings are actually full objects with their own methods and a specific equality gotcha — that's all coming properly on Day 2. For today, all you need is: text in double quotes prints as-is.
- **The semicolon `;`** ends a statement. Java doesn't care about line breaks or whitespace for meaning — it cares about semicolons. Forgetting one is probably the single most common first-week compile error.
- **Curly braces `{ }`** mark the start and end of a block — the body of the class, and the body of the method. Every `{` needs a matching `}`.

### Compiling and running — both ways

**From the terminal**, in the same directory as the file:

```bash
javac HelloWorld.java
```

This produces `HelloWorld.class` (the bytecode from Section 2) in the same folder. Then:

```bash
java HelloWorld
```

> ⚠️ **Common Mistake:** Note there's no `.class` (and no `.java`) in that second command — you compile *the file* (`HelloWorld.java`) but you run *the class* (`HelloWorld`), because `java` is invoking the JVM against the compiled class, and classes don't have file extensions as far as the JVM is concerned.

**From IntelliJ**: open the file, and you'll see a green "run" arrow next to the `main` method (or right-click the file → Run). This does the exact same `javac` + `java` sequence under the hood, just automated with one click, and gives you a built-in console pane showing the output.

Get comfortable with both. You'll use the terminal for quick sanity checks and scripting later; you'll use the IDE for everything else, including the debugger, from here forward.

---

*(Continued in the next section of this file — Variables, Operators, and Control Flow.)*

# Section 5 — Variables and Primitive Types

### What a variable actually is

A **variable** is a named location in memory that holds a value of a specific type. Three things are true of every variable, and all three matter:

1. It has a **name** (an identifier you choose).
2. It has a **type**, fixed at the moment you declare it, which determines what kind of value it can hold and how much memory it occupies.
3. It has a **value**, which can change over the variable's lifetime (that's *why* it's called "variable" — the value varies; the type does not).

### Why type matters — and why it's not just bureaucracy

Java is a **statically typed** language: every variable's type is known at compile time, and the compiler enforces it — you cannot put a decimal number into a variable declared to hold whole numbers without explicitly saying you mean to. This might feel like friction coming from nothing, but the payoff is real: an entire category of bugs (accidentally treating text as a number, accidentally losing decimal precision) becomes a compile-time error you fix in seconds, instead of a runtime failure a user hits in production.

The type also directly determines **how much memory is reserved** and **what operations are valid.** This is *why* primitive operations are fast and predictable: the compiler knows exactly how many bytes a value needs before the program ever runs, so no runtime bookkeeping is needed just to store or read a primitive.

### The four primitive types your plan asks for today

| Type | Holds | Size | Example |
|---|---|---|---|
| `int` | Whole numbers | 4 bytes (32 bits) | `int age = 28;` |
| `double` | Decimal (floating-point) numbers | 8 bytes (64 bits) | `double price = 19.99;` |
| `boolean` | `true` or `false` — nothing else | 1 bit (conceptually) | `boolean isActive = true;` |
| `char` | A single character | 2 bytes (16 bits) | `char grade = 'A';` |

> ⚠️ **Common Mistake:** `char` uses **single** quotes (`'A'`), `String` uses **double** quotes (`"A"`). These are not interchangeable, and mixing them up is a compile error, not a warning.

### The full primitive picture (good to know, even though today's plan uses four of them)

Java has eight primitive types total. You'll mostly live in `int`, `double`, `boolean`, and `char`, but knowing the rest exists — and *why* — pays off the first time you're summing a billion-scale value and silently overflow an `int`:

| Type | Size | Range (roughly) |
|---|---|---|
| `byte` | 1 byte | -128 to 127 |
| `short` | 2 bytes | -32,768 to 32,767 |
| `int` | 4 bytes | ≈ -2.1 billion to 2.1 billion |
| `long` | 8 bytes | ≈ -9.2 × 10¹⁸ to 9.2 × 10¹⁸ |
| `float` | 4 bytes | decimal, less precision than `double` |
| `double` | 8 bytes | decimal, the default choice for decimals |
| `boolean` | ~1 bit | `true` / `false` |
| `char` | 2 bytes | single Unicode character |

> 💡 **Interview Insight:** "What happens if you add 1 to `Integer.MAX_VALUE`?" is a real question. The answer is **integer overflow** — it silently wraps around to `Integer.MIN_VALUE` (a large negative number), with no error or warning. This is exactly why problems involving large sums (common in DSA — summing large arrays, computing products) often specify `long` instead of `int`, or explicitly warn about overflow. Knowing this and mentioning it unprompted when a problem involves large numbers is a strong interview signal.

### Declaring, assigning, initializing

```java
int x;        // declaration — reserves the name and type, no value yet
x = 5;        // assignment — now it holds a value
int y = 10;   // declaration + assignment in one line — "initialization"
int a = 1, b = 2, c = 3;  // multiple variables of the same type, one line
```

> ⚠️ **Common Mistake — and a genuinely important Java rule:** a **local variable** (one declared inside a method) does *not* get a default value. If you declare `int x;` and try to use `x` before assigning it a value, **the code will not compile.** This is different from *fields* on a class (Day 2), which *do* get automatic defaults (`0` for numeric types, `false` for `boolean`, `null` for objects). Java draws this line deliberately: using an uninitialized local variable is almost always a bug, so the compiler refuses to let you.

### Naming rules and convention

- Must start with a letter, `_`, or `$` — not a digit.
- Case-sensitive (`age` and `Age` are different variables).
- Convention (not a compiler rule, but a strong professional expectation): variables and methods use **camelCase** (`totalScore`, `isValid`); classes use **PascalCase** (`HelloWorld`, `BankAccount`, Day 2). Interviewers and reviewers *do* notice naming convention — it's a small, free signal of professionalism.

### Type casting, briefly

```java
int wholeNumber = 5;
double decimalNumber = wholeNumber;   // 5.0 — implicit "widening": always safe, no data can be lost

double pi = 3.14159;
int truncatedPi = (int) pi;           // 3 — explicit "narrowing": you must ask for this, because data (the .14159) is lost
```

Widening (small type → larger type, e.g. `int` → `double`) happens automatically because no information can possibly be lost. Narrowing (large type → smaller type, e.g. `double` → `int`) requires an explicit cast — the `(int)` in front — because you're telling the compiler "yes, I know I might be throwing information away, and I mean to."

---

# Section 6 — Operators

Operators are how you actually *do* something with variables and values — compute, compare, combine.

### Arithmetic operators

| Operator | Meaning | Example |
|---|---|---|
| `+` | Addition | `5 + 3` → `8` |
| `-` | Subtraction | `5 - 3` → `2` |
| `*` | Multiplication | `5 * 3` → `15` |
| `/` | Division | see below — this one has a trap |
| `%` | Modulo (remainder) | `7 % 2` → `1` |

> ⚠️ **Common Mistake — the single most important gotcha in this entire section:** **integer division in Java truncates the decimal part.** `7 / 2` evaluates to `3`, not `3.5` — because both operands are `int`, so the result is computed as an `int`. To get a decimal result, at least one operand must be a `double`:

```java
int a = 7, b = 2;
System.out.println(a / b);          // 3  (integer division — truncated)
System.out.println(a / (double) b); // 3.5  (one operand is now a double)
System.out.println(7.0 / 2);        // 3.5  (7.0 is already a double)
```

This exact trap will resurface the moment you write `celsius * 9 / 5` later today instead of `celsius * 9.0 / 5` — see the temperature converter walkthrough in Section 10.

**Modulo with negative numbers** — also worth having exactly right, since it comes up constantly in DSA (checking evenness, wrapping array indices, hashing):

```java
System.out.println(-7 % 3);  // -1, not 2
System.out.println(7 % -3);  //  1, not -2
```

In Java, the result of `%` takes the **sign of the dividend** (the left-hand operand) — not the sign of the divisor, and not a mathematician's "always non-negative" modulo (which is how some other languages, like Python, define it). If you ever need a strictly non-negative result regardless of sign, the idiom is `((a % n) + n) % n`.

### Comparison (relational) operators

| Operator | Meaning |
|---|---|
| `==` | Equal to |
| `!=` | Not equal to |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less than or equal to |
| `>=` | Greater than or equal to |

Every one of these evaluates to a `boolean` — `true` or `false`. That boolean result is precisely what `if` statements and loop conditions consume (Sections 7-8).

### Assignment operators

```java
int x = 10;
x += 5;   // same as: x = x + 5;   → 15
x -= 3;   // same as: x = x - 3;   → 12
x *= 2;   // same as: x = x * 2;   → 24
x /= 4;   // same as: x = x / 4;   → 6
x %= 4;   // same as: x = x % 4;   → 2
```

These compound assignment operators are pure convenience — they compile to exactly the same thing as the spelled-out version. Use them; they're standard, expected, idiomatic Java, and reduce visual noise in loops especially.

### Logical operators

| Operator | Meaning | Example |
|---|---|---|
| `&&` | AND — both sides must be true | `x > 0 && x < 10` |
| `\|\|` | OR — at least one side must be true | `x < 0 \|\| x > 100` |
| `!` | NOT — flips a boolean | `!isValid` |

**Short-circuit evaluation** is the detail that actually matters here, and it's not optional trivia — it's load-bearing in real code you'll write constantly:

```java
if (array != null && array.length > 0) {
    // safe: if array IS null, Java never evaluates array.length at all —
    // it short-circuits and the whole expression is immediately false
}
```

`&&` stops evaluating the moment the left side is `false` (the overall result can't possibly be `true` anymore, so there's no reason to check the right side). Symmetrically, `||` stops the moment the left side is `true`. This isn't just an optimization — it's frequently used *deliberately* as a safety guard, exactly as in the null-check example above. Writing the condition in the other order (`array.length > 0 && array != null`) would crash with a `NullPointerException` whenever `array` actually is `null`, because Java would try to read `.length` before the null-check ever ran. 🔗 **Forward reference:** you'll get the full picture of `null` and objects on Day 2 — for today, just know that order matters in `&&`/`||` chains specifically because of short-circuiting.

### The trap your plan explicitly calls out: `=` vs. `==`

```java
int x = 5;

if (x == 5) { ... }   // comparison — "is x equal to 5?" → evaluates to a boolean
if (x = 5)  { ... }   // assignment — sets x to 5, and the if checks the ASSIGNED value
```

`=` **assigns** a value: "make this variable hold this." `==` **compares** two values: "are these equal?" Writing `if (x = 5)` when you meant `if (x == 5)` is a classic, costly bug in any C-family language (Java, C, C++, JavaScript, C# — this list is long, which is exactly why the habit of double-checking matters beyond just this one language).

Java's compiler actually catches this specific mistake *for `boolean` variables* — `if (isValid = true)` won't compile, because `if` requires a `boolean` and the assignment `isValid = true` isn't automatically treated as one in that position for a `boolean`-only context... but for **numeric** variables, `if (x = 5)` **does compile**, because the assignment expression `x = 5` evaluates to `5`, and in older C-family reasoning that gets tested for truthiness. Java specifically requires an actual `boolean` in an `if` condition, so `if (x = 5)` where `x` is an `int` will *not* compile either, actually — Java is stricter here than C. Where this bug genuinely bites in Java is more subtle (e.g. inside more complex boolean expressions), but the *habit* of writing conditions carefully and reading them back is what actually protects you across every language you'll ever touch, which is why it's worth over-learning now.

---

# Section 7 — Control Flow: Making Decisions

### Why control flow matters

A program that executes one fixed sequence of steps, exactly once, isn't useful for much. Real programs need to make decisions (this section) and repeat work (Section 8) — these two capabilities, combined, are what make a "sequence of instructions" into something that can actually solve problems.

### `if` / `else if` / `else`

```java
int score = 85;

if (score >= 90) {
    System.out.println("A");
} else if (score >= 80) {
    System.out.println("B");
} else if (score >= 70) {
    System.out.println("C");
} else {
    System.out.println("F");
}
```

Execution checks each condition **top to bottom** and runs the **first** block whose condition is `true`, then skips every remaining `else if`/`else` entirely — it does not continue checking. This ordering detail matters a lot in practice: conditions should generally go from most specific to least specific (you'll see exactly this principle drive a real bug in the FizzBuzz walkthrough, Section 10).

> ⚠️ **Common Mistake:** Always use `{ }` braces, even for a single-statement body. Java allows you to omit them for a one-line body, but doing so is a well-known source of bugs the moment someone (including future-you) adds a second line to the block without noticing the braces aren't there — the second line silently falls outside the `if`.

### `switch` — two forms

The **classic** form:

```java
int day = 3;
switch (day) {
    case 1:
        System.out.println("Monday");
        break;
    case 2:
        System.out.println("Tuesday");
        break;
    case 3:
        System.out.println("Wednesday");
        break;
    default:
        System.out.println("Invalid day");
        break;
}
```

> ⚠️ **Common Mistake — the classic switch's fall-through trap:** if you omit `break`, execution **falls through** to the next case and keeps running, regardless of whether that next case's label matches:

```java
int day = 1;
switch (day) {
    case 1:
        System.out.println("Monday");
    case 2:
        System.out.println("Tuesday");
    default:
        System.out.println("Invalid");
}
// Prints ALL THREE lines — "Monday", "Tuesday", "Invalid" —
// because without break, execution just keeps falling into the next block.
```

The **modern arrow syntax** (Java 14+, which JDK 21 fully supports) fixes this by design — each case is a self-contained expression/block with no fall-through possible:

```java
int day = 3;
String dayName = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 4 -> "Thursday";
    case 5 -> "Friday";
    case 6, 7 -> "Weekend";
    default -> "Invalid day";
};
System.out.println(dayName);
```

Notice two things here beyond the fall-through fix: **multiple labels can share one arm** (`case 6, 7 ->`), and a `switch` can be used as an **expression** that directly produces a value you assign to a variable, rather than only as a statement that does something. This second form is what you should reach for by default going forward — it's shorter, and an entire category of bugs (forgetting a `break`) simply cannot happen.

> 💡 **Interview Insight:** `switch` vs. a chain of `if/else if` is functionally similar for simple equality checks, but `switch` communicates intent more clearly when you're branching on the *exact value* of one variable (especially with many branches), and the modern form is measurably safer. If asked when to prefer one, that's the honest answer — not "switch is always faster" (it usually isn't meaningfully different for a handful of branches; the real win is intent and safety, not raw performance).

---

# Section 8 — Control Flow: Loops

Loops repeat a block of code while some condition holds. Java gives you three, and picking the right one for a given situation is itself a small but real interview signal.

### `for` — reach for this when you know the iteration count up front

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);   // prints 0, 1, 2, 3, 4
}
```

Three parts, always in this order:
1. **Initialization** (`int i = 0`) — runs once, before the loop starts.
2. **Condition** (`i < 5`) — checked *before every iteration, including the first*. The loop continues only while this is `true`.
3. **Update** (`i++`) — runs after every iteration's body finishes, before the condition is checked again.

`i++` is shorthand for `i = i + 1` (there's also `i--` for `i = i - 1`). You'll write `for` loops constantly in DSA — iterating an array by index is the canonical use, arriving Day 2.

### `while` — reach for this when you don't know the count in advance

```java
int count = 0;
while (count < 5) {
    System.out.println(count);
    count++;
}
```

Same idea as `for`, but the initialization and update aren't baked into a fixed header — you manage them yourself. Use `while` when the number of iterations genuinely depends on runtime conditions you can't compute in advance (e.g., "keep reading input until the user types 'quit'", or later, "keep dividing by 2 until you reach 1").

> ⚠️ **Common Mistake:** Forgetting the update step (`count++` above) inside a `while` loop produces an **infinite loop** — the condition never becomes false because nothing about the loop's state ever changes. This is one of the most common bugs beginners hit, and the fix is always the same: make sure something the condition depends on actually changes inside the loop body, every single iteration.

### `do-while` — when the body must run at least once, no matter what

```java
int n = 10;
do {
    System.out.println(n);
    n++;
} while (n < 5);
// Prints "10" exactly once, even though the condition (n < 5) is false from the start
```

The defining difference: a `while` loop checks its condition **before** the first iteration (so the body might run zero times), while `do-while` checks it **after** the first iteration (so the body always runs at least once). This form is genuinely less common in everyday code, but it's the exact right tool whenever the logical shape of a problem is "do the thing, then decide whether to do it again" (a real example: retry-until-success logic, where you must attempt the operation at least once before you can even evaluate whether it succeeded).

### `break` and `continue`

Not explicitly named in your plan, but a direct, necessary extension of loops — you'll need both within the next few days for early-exit logic in DSA problems (e.g., stopping a scan the moment you find what you're looking for, which is the difference between an unnecessary full pass and an efficient early return).

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;       // exits the loop entirely, immediately
    }
    System.out.println(i);   // prints 0, 1, 2, 3, 4, then stops
}

for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) {
        continue;    // skips the rest of THIS iteration, moves to the next one
    }
    System.out.println(i);   // prints 1, 3, 5, 7, 9 (even numbers are skipped)
}
```

`break` exits the entire loop immediately — nothing after it in the loop body runs, and the loop does not continue to further iterations. `continue` skips only the *rest of the current iteration* and jumps straight to the next one (the update step, in a `for` loop). Mixing these up — using `break` when you meant to just skip one iteration — is a real and common bug; think through which behavior you actually want before typing either.

> 💡 **Interview Insight:** Using `break` to exit a loop the moment you've found an answer (rather than scanning the entire structure regardless) is often the difference between a correct-but-wasteful solution and an efficient one. Interviewers notice, and value, this kind of "stop as soon as you can" instinct — you'll see it formally as a technique starting with Two Pointers on Day 6.

### Nested loops — a brief preview

Loops can contain other loops:

```java
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        System.out.println(i + ", " + j);
    }
}
// The inner loop runs completely (all 3 iterations) for EVERY single iteration of the outer loop
// Total: 3 x 3 = 9 print statements
```

🔗 **Forward reference:** this "inner loop runs fully for every outer iteration" shape is *exactly* what produces O(n²) time complexity — formalized properly on Day 3. Filing this pattern away now (nested loop → work multiplies, not adds) will make Day 3's Big-O section click much faster, and it's directly relevant today too: the brute-force prime checker in Section 10 and the brute-force approach to nearly every "check every pair" problem you'll meet starting Day 5 has exactly this nested-work shape, just with one of the loops sometimes implicit.


# Section 9 — Methods

### Why methods exist

Once a piece of logic is written exactly once, inside a named, callable unit, it can be reused anywhere it's needed instead of being copy-pasted every time. This is the **abstraction** building block from Section 1's list, and it's foundational for a reason beyond just "less typing": a bug fixed inside a method is fixed everywhere that method is called, instantly. A bug in copy-pasted code has to be found and fixed in every single copy, and it's extremely easy to miss one.

### Anatomy of a method

```java
public static double celsiusToFahrenheit(double celsius) {
    double fahrenheit = celsius * 9.0 / 5.0 + 32;
    return fahrenheit;
}
```

- **`public static`** — for now, mirror what you used in `main`: `public` (callable from outside), `static` (belongs to the class, not to an object — full meaning arrives Day 2 with OOP).
- **`double`** (the first one) — the **return type**: the type of value this method hands back to whatever called it.
- **`celsiusToFahrenheit`** — the method's name. Convention: camelCase, and ideally a verb or verb phrase describing what it *does*.
- **`(double celsius)`** — the **parameter list**. This method accepts exactly one input, a `double`, which will be locally known inside the method as `celsius`.
- **`return fahrenheit;`** — hands a value back to the caller. A method declared to return `double` **must** return a `double` on every possible code path — the compiler checks this.
- **`void` methods** (like `main`) don't have a `return` statement with a value — they can have a bare `return;` to exit early, or no `return` at all, and simply finish when execution reaches the closing brace.

### Calling a method

```java
public static void main(String[] args) {
    double result = celsiusToFahrenheit(100.0);
    System.out.println(result);   // 212.0
}
```

The value `100.0` is passed in as an **argument** (the concrete value supplied at the call site — as distinct from a **parameter**, which is the placeholder name in the method's own declaration; people use these two terms almost interchangeably in casual conversation, but knowing the precise distinction reads as careful, and interviewers do sometimes probe it).

### Pass-by-value — for primitives (the full picture, including objects, is Day 2)

This is a genuinely important Java semantic, and it's worth getting exactly right rather than approximately right:

```java
public static void tryToModify(int x) {
    x = 100;
    System.out.println("Inside the method: x = " + x);   // 100
}

public static void main(String[] args) {
    int number = 5;
    tryToModify(number);
    System.out.println("After the method call: number = " + number);   // still 5!
}
```

When you pass `number` into `tryToModify`, Java **copies the value** (`5`) into the method's own local parameter `x`. Inside the method, `x` and the caller's `number` are two completely separate pieces of memory that happen to start out equal. Reassigning `x` inside the method only ever changes that local copy — it has no way to reach back and affect `number` in `main`. This is what "pass by value" means, precisely: the *value* is what gets passed, not the variable itself.

> 🔗 **Forward reference:** Java is *always* pass-by-value — there's no exception — but the behavior looks different once you're passing arrays and objects (Day 2), because what gets copied for an object is a *reference* (an address), not the object's actual contents. That distinction — and the real, practical consequence it has for writing correct code — gets its own full treatment on Day 2, once arrays and objects actually exist to demonstrate it with.

### Method overloading

The same method name can be reused for genuinely different parameter lists — different types, and/or a different number of parameters:

```java
public static int add(int a, int b) {
    return a + b;
}

public static double add(double a, double b) {
    return a + b;
}

public static int add(int a, int b, int c) {
    return a + b + c;
}
```

The compiler decides which version to call based entirely on **what you pass in** — this is resolved at **compile time**, by matching the argument types and count against the available overloads. Call `add(2, 3)` and the compiler picks the `int` version; call `add(2.5, 3.5)` and it picks the `double` version; call `add(1, 2, 3)` and it picks the three-parameter version. If no overload matches what you're passing, you get a compile error, not a runtime surprise.

> 💡 **Interview Insight:** Overloading is resolved at *compile time* based on the declared types of the arguments — this is called **static binding** or **static dispatch**. This is worth contrasting later (Day 2/5) with method **overriding** in inheritance, which is resolved at *runtime* based on the actual object's type (**dynamic binding**). Confusing these two — "overloading" vs. "overriding" — is an extremely common vocabulary slip in interviews, and interviewers do notice which one you actually mean.

---

# Section 10 — Practice Problems (Full Depth)

These three problems are today's proving ground for everything above. They're not LeetCode-numbered, but from here forward, every practice problem in this series — including these — gets the same rigor a tier-1 interview expects: state the approach, justify it, analyze it, know its edges. Building that habit starting today, on genuinely simple problems, means it's automatic by the time the problems get hard.

## Problem 1: FizzBuzz

**Statement:** Print the numbers from 1 to `n`. For multiples of 3, print "Fizz" instead of the number. For multiples of 5, print "Buzz" instead. For multiples of both 3 and 5, print "FizzBuzz" instead.

### Approach

```java
public static void fizzBuzz(int n) {
    for (int i = 1; i <= n; i++) {
        if (i % 15 == 0) {
            System.out.println("FizzBuzz");
        } else if (i % 3 == 0) {
            System.out.println("Fizz");
        } else if (i % 5 == 0) {
            System.out.println("Buzz");
        } else {
            System.out.println(i);
        }
    }
}
```

> ⚠️ **The bug nearly everyone writes on their first attempt:** checking `i % 3 == 0` *before* `i % 15 == 0`. Walk through why this breaks: for `i = 15`, `15 % 3 == 0` is `true`, so the `if/else if` chain prints `"Fizz"` and — because `else if` only runs when everything above it was `false` — **never even checks** the `%5` or `%15` conditions. `"FizzBuzz"` never prints for *any* input, because every multiple of 15 is also a multiple of 3, and the more general condition was checked first and "stole" the branch. This is a direct, concrete instance of the ordering principle from Section 7: **check the most specific condition first.** `i % 15 == 0` (divisible by both) is strictly more specific than `i % 3 == 0` alone, so it must come first in the chain.

An equally correct alternative avoids the ordering trap entirely by checking both conditions explicitly instead of relying on order:

```java
if (i % 3 == 0 && i % 5 == 0) {
    System.out.println("FizzBuzz");
} else if (i % 3 == 0) {
    System.out.println("Fizz");
} else if (i % 5 == 0) {
    System.out.println("Buzz");
} else {
    System.out.println(i);
}
```

### A second, more extensible approach — string-building

```java
public static void fizzBuzz(int n) {
    for (int i = 1; i <= n; i++) {
        String output = "";
        if (i % 3 == 0) output += "Fizz";
        if (i % 5 == 0) output += "Buzz";
        System.out.println(output.isEmpty() ? String.valueOf(i) : output);
    }
}
```

This builds the answer piece by piece rather than branching on every combination explicitly. It looks like more code for exactly three rules, but it scales far better: adding a fourth rule ("multiples of 7 → Bazz") means adding one `if` line in this version, versus potentially doubling the number of branches in the `if/else if` version (you'd need a branch for every *combination* of rules that could apply). This ternary (`condition ? valueIfTrue : valueIfFalse`) is new syntax — a compact one-line `if/else` that produces a value — worth recognizing even though you haven't written one yet.

### Complexity

**Time: O(n)** — exactly one pass over the numbers 1 through n, constant work per number. **Space: O(1)** — no data structure grows with `n`; you're only ever holding one number and one string at a time (this counts *extra* space, not the output itself, which is standard practice in complexity analysis — output size is not counted against your solution's space complexity).

> 💡 **Interview Insight:** FizzBuzz is famous as a filter question — its entire purpose is confirming a candidate can translate a simple specification into working, correct code without hand-holding. At an SDE-2 tier-1 bar, you won't be asked this in isolation, but the instinct it tests (read the spec precisely, order conditions by specificity, verify against edge cases before declaring done) is exactly what shows up one layer deeper in every problem from Day 5 onward. A strong candidate also proactively mentions the extensibility trade-off between the two approaches above without being asked — that's the "evaluate different approaches and justify your decision" expectation this series keeps building toward.

---

## Problem 2: Celsius to Fahrenheit

**Statement:** Write a method that converts a temperature in Celsius to Fahrenheit. Formula: `F = C × 9/5 + 32`.

### Approach

```java
public static double celsiusToFahrenheit(double celsius) {
    return celsius * 9.0 / 5.0 + 32;
}
```

> ⚠️ **Common Mistake, and it's the exact trap from Section 6:** writing `celsius * 9 / 5` instead of `celsius * 9.0 / 5.0`. If `celsius` is a `double`, `celsius * 9` is already a `double` (an `int` literal multiplied by a `double` promotes to `double`), so `/ 5` would actually still produce a correct decimal result *in this specific case* — but relying on that is fragile and easy to get wrong the moment the input type changes. Writing `9.0` and `5.0` explicitly removes any ambiguity about intent and protects you if this logic is ever copied into a context where the input starts as an `int`.

### Design choice worth narrating out loud: why return a value instead of printing directly?

```java
// Weaker design:
public static void celsiusToFahrenheitPrint(double celsius) {
    System.out.println(celsius * 9.0 / 5.0 + 32);
}

// Better design:
public static double celsiusToFahrenheit(double celsius) {
    return celsius * 9.0 / 5.0 + 32;
}
```

The second version is more reusable: the caller decides what to *do* with the result — print it, store it, use it in further math, display it in a UI, write it to a file. The first version has decided for them, permanently, that the only possible use is printing to console. **Separating computation from output** is a small design habit that scales directly into how you'll structure real production code, and articulating *why* you chose one form over the other is exactly the "explain your reasoning" muscle tier-1 interviews are built to test.

### Complexity

**Time: O(1), Space: O(1)** — fixed arithmetic, no dependence on any input size.

### Edge cases worth considering out loud

- Negative Celsius values (e.g., freezing point, `0°C` → `32°F`) — the formula handles this correctly with no special-casing needed, and pointing that out (rather than silently assuming it) is good practice.
- Extremely negative "impossible" temperatures (below absolute zero, `-273.15°C`) — whether to validate this depends entirely on the method's actual contract/use case; a strong answer here is "I'd ask whether input validation is expected, or whether this is trusted internal input," not silently deciding either way.

---

## Problem 3: Prime Checker

**Statement:** Write a method that determines whether a given integer is prime. (A prime number is a whole number greater than 1 with no positive divisors other than 1 and itself.)

This is today's most valuable problem, precisely because it's the first one with a genuine brute-force-vs-optimized distinction — the exact shape of "explain every possible approach, justify why one wins" that this entire series (and tier-1 interviews) will keep asking for, starting with something simple enough to fully internalize the *pattern* of that reasoning.

### Approach 1 — Brute Force: check every number up to n

```java
public static boolean isPrimeBruteForce(int n) {
    if (n < 2) return false;          // 0, 1, and negative numbers are not prime, by definition
    for (int i = 2; i < n; i++) {
        if (n % i == 0) {
            return false;             // found a divisor other than 1 and n → not prime
        }
    }
    return true;                      // no divisor found anywhere → prime
}
```

**Why it works:** by definition, `n` is prime exactly when nothing between `2` and `n - 1` divides it evenly. This checks literally every candidate in that range, so it's correct — but it's doing more work than it needs to.

**Complexity: Time O(n), Space O(1).**

### Approach 2 — Optimized: only check up to √n

```java
public static boolean isPrime(int n) {
    if (n < 2) return false;
    if (n == 2) return true;
    if (n % 2 == 0) return false;              // handle the only even prime, then skip all other evens
    for (int i = 3; i * i <= n; i += 2) {      // only odd candidates from here
        if (n % i == 0) {
            return false;
        }
    }
    return true;
}
```

**Why this is correct — the actual mathematical justification (interviewers will ask "why does this work," not just "does it work"):**

Suppose `n` is *not* prime. Then `n = a × b` for some integers `a` and `b`, both greater than 1. Without loss of generality, say `a ≤ b`. Since `a ≤ b` and `a × b = n`, it follows that `a × a ≤ a × b = n`, which means `a ≤ √n`. In other words: **if `n` has any factor at all, it must have one that is ≤ √n** — because factors always come in pairs `(a, b)` with `a × b = n`, and at least one member of that pair can never be larger than the square root (if both were larger than √n, their product would exceed n). So if you've checked every candidate up to and including `√n` and found no divisor, there is no possible pair left to find — you're done, correctly, without checking anything beyond that point.

**Why `i * i <= n` instead of `i <= Math.sqrt(n)`:** both are logically equivalent, but `i * i <= n` avoids a floating-point function call and any floating-point precision edge cases (`Math.sqrt` returning a value *very* slightly off due to floating-point representation, which could — in rare cases — cause an off-by-one error at the boundary). Multiplying two integers is also simply cheaper than a square-root computation. This is a small detail, but naming it unprompted is exactly the kind of trade-off-awareness a tier-1 interview is listening for.

**The extra optimizations layered in, and why each is valid:**
- `if (n == 2) return true;` then `if (n % 2 == 0) return false;` — handles the one even prime explicitly, then immediately rules out every other even number. This is valid because *any* even number greater than 2 is, by definition, divisible by 2, so it can never be prime.
- `for (int i = 3; ...; i += 2)` — having already ruled out all even divisors above, only odd candidates (`3, 5, 7, 9, ...`) need checking. This roughly halves the number of iterations for free.

**Complexity: Time O(√n), Space O(1).** For a mid-sized input like `n = 1,000,000`, that's the difference between checking on the order of a million candidates (brute force) versus about a thousand (optimized) — a real, measurable difference, not just a theoretical one.

### A further optimization worth knowing about, even though it's beyond what's asked today

All primes greater than 3 are of the form `6k ± 1` for some integer `k` — so you can advance the loop by checking only candidates of that shape, skipping roughly two out of every three odd numbers you'd otherwise check. This is a genuine further speedup, but it's a "nice to mention if it comes up" footnote, not something expected unprompted at this stage — over-engineering a warm-up question past what's asked can itself read as a minor interview miss (not reading the room on expected scope).

### Edge cases — a checklist worth having automatic

| Input | Correct output | Why |
|---|---|---|
| `n = 0` | `false` | Not prime by definition |
| `n = 1` | `false` | Not prime by definition — this trips people up more than any other case |
| `n = 2` | `true` | The only even prime |
| `n = 3` | `true` | Smallest odd prime |
| Negative `n` | `false` | Primality is only defined for positive integers |
| Large `n` (e.g. near `Integer.MAX_VALUE`) | Correct, but consider `i * i` overflow | If `i` and `n` are both `int` and `i` is large, `i * i` can overflow before the loop condition is even checked — a real, subtle bug worth being aware of for very large inputs; using `long` for the loop variable is one fix. |

### Interview framing

> 💡 **Interview Insight:** A prime checker is a classic warm-up precisely because it has this brute-force/optimized split baked in, so it directly tests whether you *default* to reasoning about efficiency or need to be prompted for it. The expected flow in a tier-1 interview: state the brute-force solution first if it's not immediately obvious the interviewer wants you to skip it, briefly note its complexity, then proactively say something like *"this is O(n) — we can do better by only checking up to the square root, since factors pair up around it,"* and implement the optimized version, explaining the `√n` justification as you go rather than after being asked. A very common, entirely fair follow-up: **"now find all primes up to N."** That's a different, well-known algorithm (the **Sieve of Eratosthenes**, achieving O(n log log n) by eliminating multiples rather than testing each number individually) — worth knowing this follow-up exists and having a name ready for it, even though implementing the sieve itself isn't part of today's scope.


---

# Section 11 — Git & Version Control Basics

### What Git is, and why it matters beyond "backup"

**Git** is a system that records snapshots of your code over time. Two distinct benefits fall out of that one idea, and both matter for this plan specifically:

1. **You can undo mistakes safely.** Every snapshot ("commit") is preserved — you can always get back to a known-good state, which means you can experiment and refactor fearlessly.
2. **For your job search specifically: it shows a public, consistent history of real work.** A `dsa-java` repository with 100+ commits, organized by pattern, accumulated daily over months, is itself a credible signal to a hiring manager before a single interview happens.

### The mental model: three areas

```
Working Directory  ──git add──►  Staging Area  ──git commit──►  Local Repository  ──git push──►  Remote (GitHub)
  (your actual files,               (changes you've                (a permanent,                  (a copy hosted
   as you're editing them)           marked as "ready                snapshotted                    online, e.g.
                                      to be committed")               version, saved                 on GitHub)
                                                                       locally)
```

You edit files in your **working directory** like normal. `git add` marks specific changes as ready to be included in the next snapshot (the **staging area** — this exists so you can commit *some* of your changes without being forced to commit everything you've touched). `git commit` actually takes that snapshot, permanently, into your **local repository**, with a message describing what changed. `git push` uploads your local commits to a **remote** repository (GitHub, in your case) so they exist outside your machine too.

### Core commands for today

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
# One-time setup — Git stamps every commit with this identity.

git init
# Turns the current directory into a Git repository. Run once, in the project's root folder.

git add HelloWorld.java
# or: git add .        (stages EVERY changed file in the current directory and below)

git commit -m "Add HelloWorld.java"
# Takes the snapshot. The -m flag lets you supply the message inline.
# Write commit messages as a short, present-tense description of what changed and why.

git remote add origin https://github.com/yourusername/dsa-java.git
# Links your local repository to a specific location on GitHub. Run once per repository.

git push -u origin main
# Uploads your commits to GitHub. The -u flag remembers this remote+branch pairing,
# so future pushes from this repo can just be `git push`.
```

### Today's task, concretely

1. Create a GitHub account if you don't have one.
2. Create two repositories on GitHub: `dsa-java` and `java-fundamentals`.
3. In each local project folder: `git init`, then `git add`, `git commit -m "..."`, then `git remote add origin <url>`, then `git push -u origin main`.
4. Commit `HelloWorld.java` as your very first commit — this is your proof the entire toolchain (editor → compiler → JVM → Git → GitHub) works end to end, which matters because every day from here forward depends on this pipeline working.

> 💡 **Practical tip:** Add a `.gitignore` file to each repository (a plain text file, one pattern per line) so compiled output and IDE clutter never get committed — for Java/IntelliJ, at minimum:
> ```
> *.class
> .idea/
> out/
> target/
> ```

**Going forward:** starting today, every solved LeetCode problem gets committed to `dsa-java`, organized by pattern — e.g. `dsa-java/hashmap-hashset/two-sum/`, `dsa-java/two-pointers/valid-palindrome/`. Set this folder structure up now; you'll want it organized and easily browsable months from now, both for your own review and for anyone (including an interviewer) who looks at your GitHub profile.

---

# Section 12 — Career Block Guide

### GitHub profile README

Create a repository named exactly `github.com/[yourusername]/[yourusername]` — GitHub treats a repo with this exact naming pattern specially, rendering its `README.md` directly on your profile page. Structure worth using:

- **A short personal statement** — role, experience, and current focus, stated plainly. E.g.: *"Backend engineer building expertise in distributed systems. 3 years of experience. Currently: SDE-2 transformation."* Specific and factual reads better than generic enthusiasm.
- **Tech stack badges** — [shields.io](https://shields.io) generates small badge images (e.g., "Java", "Git") you embed with a single line of Markdown each; a common, clean way to make a tech list visually scannable rather than a plain bullet list.
- **A "Currently Building" section** — 2-3 lines on what you're actively working on. This is worth updating weekly as this plan progresses; a stale "currently building" section reads worse than not having one.

### LinkedIn Post 1 — the journey announcement

Today's post is specifically an announcement, not a technical post (those come later in the week, once you have technical substance to share). A strong version is short, specific, and concrete rather than aspirational-sounding — state what you're doing (a structured SDE-2 transformation, starting from Java fundamentals through system design), roughly how long it'll run, and that you'll be sharing progress. Concrete beats motivational; a hiring manager skimming LinkedIn responds better to "here's specifically what I'm doing and why" than to generic enthusiasm about growth.

### Networking — identifying targets

Identify 5 Backend Engineers at companies you admire. Today's task is purely observational — follow them, read what they post, get a feel for what technical content in your target space actually looks like. You are not reaching out yet; that starts Day 2. Skipping straight to cold outreach before you've absorbed what good engineering content in your space looks like tends to produce generic, easy-to-ignore messages — a day of observation first measurably improves what you'll write later.

---

# Day 1 — Interview Questions

Test yourself against these before moving to Day 2. Cover the answer, answer out loud as if to an interviewer, then check.

---

**1. What's the difference between compilation and interpretation, and how does Java use both?**

*Answer:* Compilation translates source code into another form ahead of execution (Java's `javac` compiles `.java` source into `.class` bytecode). Interpretation executes instructions one at a time as the program runs. Java does both: `javac` compiles source to bytecode once, and then the JVM interprets that bytecode at runtime — additionally using JIT compilation to convert frequently-executed bytecode into native machine code on the fly, for speed.

---

**2. What does "write once, run anywhere" mean, and what actually makes it possible?**

*Answer:* A single compiled `.class` file (bytecode) is platform-independent — the same bytecode runs unmodified on Windows, Mac, or Linux. What makes this possible is that the JVM (not the bytecode) is platform-specific: each OS has its own JVM implementation that knows how to translate that identical bytecode into instructions for its specific machine.

---

**3. What's the difference between the JDK, JRE, and JVM?**

*Answer:* JVM is the engine that executes bytecode. JRE is the JVM plus the standard library classes needed to run compiled programs. JDK is the JRE plus development tools, including the compiler (`javac`) — you need the JDK to *write and compile* Java, not just run it.

---

**4. Why must a Java file's name exactly match its public class name?**

*Answer:* It's a hard compiler requirement in Java, not a convention — a `.java` file containing a `public class Foo` must be named `Foo.java`, exactly, including capitalization, or it will fail to compile.

---

**5. What determines how much memory a variable uses?**

*Answer:* Its declared type. Each primitive type has a fixed, known size (e.g., `int` is 4 bytes, `double` is 8 bytes) determined entirely at compile time, which is part of why primitive operations are fast — no runtime bookkeeping is needed to know how much space a value needs.

---

**6. What happens when you divide two `int`s in Java, e.g., `7 / 2`? How do you get a decimal result?**

*Answer:* Integer division truncates — `7 / 2` evaluates to `3`, discarding the remainder, because the result of dividing two `int`s is computed as an `int`. To get a decimal result, at least one operand must be a `double` — e.g., `7 / 2.0`, or `(double) 7 / 2`.

---

**7. What is the output of `-7 % 3` in Java, and why?**

*Answer:* `-1`. In Java, the result of `%` takes the sign of the dividend (the left operand), not a mathematically "always non-negative" definition of modulo.

---

**8. Explain the difference between `=` and `==`. What kind of bug does confusing them cause?**

*Answer:* `=` is assignment — it sets a variable's value. `==` is comparison — it checks whether two values are equal, producing a `boolean`. Writing `=` where `==` was intended inside a condition silently reassigns a variable instead of comparing it — Java's requirement that `if` conditions be an actual `boolean` catches many, but not all, instances of this mistake, so the habit of double-checking still matters.

---

**9. What is short-circuit evaluation, and why does it matter for code like `if (arr != null && arr.length > 0)`?**

*Answer:* `&&` stops evaluating as soon as the left operand is `false` (the result can't be `true` regardless of the right side), and `||` stops as soon as the left operand is `true`. In the example, if `arr` is `null`, Java never evaluates `arr.length` at all — it short-circuits after the first condition fails. Writing the check in the opposite order would attempt to read `.length` on a `null` reference and crash.

---

**10. When would you choose a `for` loop over a `while` loop?**

*Answer:* `for` when the number of iterations (or at least the start/stop/step logic) is known up front — it keeps that bookkeeping together in one line. `while` when the number of iterations depends on a condition that can only be evaluated at runtime and isn't naturally expressed as a counter.

---

**11. What's the difference between `while` and `do-while`?**

*Answer:* `while` checks its condition *before* the first iteration, so the body may run zero times. `do-while` checks its condition *after* the first iteration, so the body is guaranteed to run at least once, even if the condition is false from the start.

---

**12. What happens if you forget a `break` in a classic `switch` statement?**

*Answer:* Execution falls through into the next `case` block and keeps running, regardless of whether that next case's label actually matches — continuing until it hits a `break` or reaches the end of the `switch`. This is why the modern arrow-syntax `switch` (`case X -> ...`) is generally preferable — it has no fall-through behavior at all.

---

**13. What is method overloading, and how does the compiler decide which overloaded version to call?**

*Answer:* Overloading is defining multiple methods with the same name but different parameter lists (different types and/or different counts). The compiler picks which one to call at **compile time**, based on the number and types of arguments at the call site — this is static binding, distinct from the runtime dynamic binding used in method overriding (inheritance, arriving Day 2).

---

**14. Java is often described as "pass by value." What does that mean for a primitive parameter?**

*Answer:* When a primitive is passed to a method, the method receives a copy of the value, stored in its own local parameter. Reassigning that parameter inside the method has no effect on the caller's original variable, because they're separate pieces of memory that only started out equal.

---

**15. [Code reading] What does this print, and why?**

```java
int x = 5;
int y = x;
x = 10;
System.out.println(y);
```

*Answer:* `5`. `int y = x;` copies the *value* of `x` at that moment (`5`) into `y` — `y` and `x` are independent variables from that point on. Reassigning `x` afterward has no effect on `y`. (This is the same "copy the value" idea from pass-by-value, just without a method call involved.)

---

**16. [Debug] What's wrong with this FizzBuzz snippet, and why?**

```java
for (int i = 1; i <= n; i++) {
    if (i % 3 == 0) {
        System.out.println("Fizz");
    } else if (i % 5 == 0) {
        System.out.println("Buzz");
    } else if (i % 15 == 0) {
        System.out.println("FizzBuzz");
    } else {
        System.out.println(i);
    }
}
```

*Answer:* The `%15` check is unreachable. Every multiple of 15 is also a multiple of 3, so `i % 3 == 0` is checked first and is already `true` — the `if/else if` chain runs that branch and never reaches the `%15` check at all. `"FizzBuzz"` will never print, for any input. The fix is to check the most specific condition (`%15`, divisible by both) *first* in the chain.

---

**17. Walk through your optimized prime-checking algorithm's time complexity, and explain why checking divisors only up to `√n` is sufficient.**

*My Answer:* Since any divisor greater than the square root must have a corresponding divisor smaller than the square root, checking up to √n is sufficient to cover all possible factor pairs. This reduces the time complexity from $O(n)$ to $O(√n)$.

*Answer:* If `n` is not prime, it factors as `n = a × b` with `1 < a ≤ b < n`. Since `a ≤ b`, it follows that `a × a ≤ a × b = n`, so `a ≤ √n`. This means any composite number is guaranteed to have a factor at or below its square root — so if no divisor is found by `√n`, none exists at all, and `n` must be prime. This gives O(√n) time, versus O(n) for checking every candidate up to `n - 1`.

---

**18. Why does the optimized prime check use `i * i <= n` instead of `i <= Math.sqrt(n)`?**

*Answer:* They're logically equivalent, but `i * i <= n` avoids a floating-point function call and any floating-point precision issues near the boundary (a `Math.sqrt` result that's very slightly off could cause an incorrect off-by-one comparison). It's also computationally cheaper — integer multiplication versus a square-root computation.

---

## Daily Deliverable Check

Cross-reference against your plan's checklist — by this point you should be able to check off every item:

- [ ] JDK and IDE installed and working
- [ ] `HelloWorld.java` compiled and run from both terminal and IDE
- [ ] Comfortable writing `if/else`, `switch` (both forms), and all three loop types from memory
- [ ] FizzBuzz, temperature converter, and prime checker written, tested, and pushed
- [ ] Git installed and configured; `dsa-java` and `java-fundamentals` repos created and pushed with a first commit
- [ ] GitHub profile README live; LinkedIn Post 1 published

---

## What Tomorrow Assumes You Already Know Cold

Day 2 builds directly on: variables and primitive types, all operators (especially the `=`/`==` distinction and short-circuiting), `if`/`switch`, all three loop types, and writing/calling methods including passing arguments. If any of the 18 questions above gave you trouble, that's the signal to revisit this book before starting Day 2 — everything from here forward assumes this is automatic, not something you're still reasoning through from scratch.

**Next:** [Day 2 Resource Book](./Day2_Resource_Book.md) — Arrays, Strings, and Object-Oriented Programming.
