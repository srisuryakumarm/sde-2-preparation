# Part 1 (Primer) — # Lambdas, Functional Interfaces & Method References — Quick Revision

## 1. Functional Interface

**Functional Interface**

- An interface with exactly **one abstract method**.
- A lambda expression can provide the implementation of that method.
- `@FunctionalInterface` tells the compiler to verify this rule.

Example:
```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}
```

Default and static methods are allowed because they do not add another abstract method.

---

## 2. Lambda Expression

**Lambda**

- A shorter way to implement a functional interface.
- Avoids writing an anonymous inner class when only one method needs implementation.
- Uses the `->` operator.

### Anonymous Inner Class
```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};
```

### Lambda Expression
```java
Runnable task = () -> System.out.println("Hello");
```

Both provide the implementation of `Runnable.run()`.

### How does Java map a lambda to a method?

Consider:
```java
Runnable task = () -> System.out.println("Hello");
```

Java follows this process:

1. Checks the target type: `Runnable`.
2. Finds its only abstract method: `run()`.
3. Uses the lambda body as the implementation of `run()`.

**Important:** Java doesn't guess the method name. It identifies the method from the functional interface.

### Lambda Syntax
```java
(parameters) -> expression
```

Examples:
```java
() -> System.out.println("Hello");  // No parameters

x -> x * 2;                         // One parameter

(a, b) -> a + b;                    // Two parameters
```

### Expression Body vs Block Body

**Expression body:** Returns the result automatically when the method requires a return value.
```java
(a, b) -> a + b
```

**Block body:** Uses braces and requires an explicit `return` when the method returns a value.
```java
(a, b) -> {
    int result = a + b;
    return result;
}
```

### Parameter Type Inference

Java determines parameter types from the functional interface.

```java
Comparator<Integer> c = (a, b) -> Integer.compare(a, b);
```

Here, Java knows `a` and `b` are integers from `Comparator<Integer>`.

---

## 3. Six Common Functional Interfaces

All are available in `java.util.function`, except `Runnable` and `Comparator`, which belong to other Java packages.

| Interface | Abstract method | Purpose |
|---|---|---|
| `Runnable` | `void run()` | Perform an operation |
| `Supplier<T>` | `T get()` | Provide a value |
| `Consumer<T>` | `void accept(T)` | Accept a value and perform an operation |
| `Function<T,R>` | `R apply(T)` | Transform input into output |
| `Predicate<T>` | `boolean test(T)` | Check a condition |
| `Comparator<T>` | `int compare(T,T)` | Compare two values |

### Examples

**Runnable — Perform work**
```java
Runnable task = () -> System.out.println("Hello");
```

**Supplier — Provide a value**
```java
Supplier<String> s = () -> "Hello";
String result = s.get();
```

**Consumer — Accept and use a value**
```java
Consumer<String> c = name -> System.out.println(name);
c.accept("Surya");
```

**Function — Transform a value**
```java
Function<String, Integer> f = name -> name.length();
int length = f.apply("Surya");
```

**Predicate — Check a condition**
```java
Predicate<Integer> p = n -> n > 10;
boolean result = p.test(15); // true
```

**Comparator — Compare two values**
```java
Comparator<Integer> c =
    (a, b) -> Integer.compare(a, b);
```

### Remember

- `Runnable` → Do something.
- `Supplier` → Give something.
- `Consumer` → Receive something.
- `Function` → Transform something.
- `Predicate` → Check something.
- `Comparator` → Compare two things.

---

## 4. Lambda Expressions with Threads (`Runnable`)

`Runnable` is a functional interface.

```java
public interface Runnable {
    void run();
}
```

### Using a Lambda
```java
Runnable task = () -> System.out.println("Running");

Thread thread = new Thread(task);
thread.start();
```

### Direct Lambda
```java
Thread thread = new Thread(
    () -> System.out.println("Running")
);

thread.start();
```

**What happens?**

1. The lambda provides the implementation of `Runnable.run()`.
2. The `Thread` constructor receives that task.
3. Calling `start()` starts a new thread.
4. The new thread executes the task through `run()`.

### Remember

```text
Runnable → Defines the task
Lambda   → Provides the task implementation
Thread   → Executes the task
start()  → Starts a new thread
run()    → Executes the task's method
```

**Important:** A lambda does not create a thread by itself.

---

## 5. Method References (`::`)

**Method Reference**

- A shorter way to refer to an existing method.
- Used when a lambda simply calls an existing method.
- Uses the `::` operator.
- The target functional interface determines how the method is used.

### Four Types

| Type | Example | Equivalent Lambda |
|---|---|---|
| Static method | `Integer::sum` | `(a, b) -> Integer.sum(a, b)` |
| Specific object method | `System.out::println` | `x -> System.out.println(x)` |
| Method on incoming object | `String::length` | `s -> s.length()` |
| Constructor reference | `ArrayList::new` | `() -> new ArrayList<>()` |

### Remember

```java
Integer::sum
```
Refers to an existing static method.

```java
System.out::println
```
Refers to a method on a specific object.

```java
String::length
```
Calls `length()` on the incoming String.

```java
ArrayList::new
```
Refers to the ArrayList constructor.

**Important:** A method reference provides behavior to be executed later. It does not necessarily execute the method immediately.

---

## 6. Decode Common Java Code Patterns

### A. Sorting Intervals
```java
Arrays.sort(intervals,
    (a, b) -> Integer.compare(a[0], b[0])
);
```

- Uses `Comparator<int[]>`.
- Compares the first element of two interval arrays.
- Sorts intervals by starting value.
- Negative → `a` comes first; zero → equal; positive → `b` comes first.

**Prefer `Integer.compare(a, b)` over `a - b` to avoid integer overflow.**

### B. PriorityQueue — Max Heap
```java
PriorityQueue<int[]> pq =
    new PriorityQueue<>(
        (a, b) -> Integer.compare(b[1], a[1])
    );
```

- Uses `Comparator<int[]>`.
- Compares the second element of two arrays.
- Reverses the ordering to make larger values come first.
- Creates a max-heap based on index `1`.

### C. Map Merge
```java
map.merge(key, 1, Integer::sum);
```

- If key is absent → inserts `1`.
- If key exists → adds the old value and `1`.
- `Integer::sum` is equivalent to `(a, b) -> a + b`.
- Uses a `BinaryOperator<Integer>`.

Useful for frequency counting.

### D. computeIfAbsent()
```java
graph.computeIfAbsent(
    u, k -> new ArrayList<>()
).add(v);
```

- Checks whether key `u` has a mapping.
- If absent (or mapped to `null`) → creates a new `ArrayList`.
- If present → returns the existing list.
- `.add(v)` adds the neighbor to that list.
- Uses a `Function` to create the missing value.

### E. Comparator.comparing()
```java
Comparator.comparing(Employee::getName);
```

- Creates a comparator based on an object's property.
- `Employee::getName` extracts the employee's name.
- Equivalent in principle to comparing `a.getName()` and `b.getName()`.

Additional operations:
```java
.thenComparing(Employee::getAge)
.reversed()
```

- `thenComparing()` → Adds a secondary sorting condition.
- `reversed()` → Reverses the ordering.

### F. assertThatThrownBy()
```java
assertThatThrownBy(() -> service.get(1L))
    .isInstanceOf(NotFoundException.class);
```

- Provides an operation for the testing framework to execute.
- The framework executes it and checks the exception.
- Without the lambda, the method would execute immediately, before the assertion could inspect its exception.

### G. Supplier in Testcontainers
```java
registry.add(
    "spring.datasource.url",
    postgres::getJdbcUrl
);
```

- Passes a reference to `getJdbcUrl()`.
- The receiving code can call it when the value is needed.
- Acts as a value supplier rather than immediately calling the method.

---

## 7. Effectively Final Variables

A local variable captured by a lambda must be `final` or **effectively final**.

**Valid:**
```java
int multiplier = 2;

Function<Integer, Integer> f =
    number -> number * multiplier;
```

`multiplier` is not reassigned.

**Invalid:**
```java
int count = 0;

list.forEach(x -> count++);
```

`count` is being modified inside the lambda.

### Remember

- Local variables captured by lambdas cannot be reassigned.
- An object's contents can still be modified if its reference is not reassigned.
- Use a normal loop or an appropriate mutable object when a changing counter is required.

---

## 8. `this` Inside a Lambda

| Lambda | Anonymous Inner Class |
|---|---|
| `this` refers to the enclosing object. | `this` refers to the anonymous class object. |

---

# SDE-2 Mental Model

Remember these six statements:

1. **Functional Interface** defines one abstract method.
2. **Lambda** provides the implementation of that method.
3. **Java identifies the method** using the target functional interface.
4. **Method Reference (`::`)** is a shorter way to refer to an existing method.
5. **Standard Functional Interfaces** represent common operations such as providing, consuming, transforming, checking, and comparing values.
6. **Lambda does not execute automatically.** The receiving code decides when to execute it.

### One-line Summary

> A lambda is a concise way to provide the implementation of a functional interface's single abstract method, while a method reference is a shorter way to refer to an existing method that matches the expected interface.