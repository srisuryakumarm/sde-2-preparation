# Day 30 — Missing Primer: Lambdas, Functional Interfaces, and Method References

**Do this first on Day 30 (≈ 60 min), then continue with Day 30's regular plan.**
**Needed by:** Day 34 onward — Spring, JUnit/AssertJ (Day 45), Mockito (Day 46), Testcontainers (Day 47), Kafka (Day 48) and Resilience4j (Day 64) are all written with lambdas.
**Why it is missing:** Day 106's book says this series "has never taught lambda expressions, method references, or functional interfaces" — yet the books already use them: Day 17 (`Comparator.comparing(Employee::getName)`), Days 24–26 (`(a, b) -> Integer.compare(a[0], b[0])`), Day 57 (`new PriorityQueue<>((a, b) -> b[1] - a[1])`), Days 76–79 (`computeIfAbsent(k -> new ArrayList<>())`), Days 45–47 (`assertThatThrownBy(() -> ...)`). Until today you have been copying them; after today you can read and write them.
**Already in place:** interfaces (Day 5), anonymous inner classes for `Runnable`/`Comparator` (Days 17, 29), generics (Day 16).

---

## 1. One idea, three spellings

```java
// 1) Anonymous inner class — what you wrote on Days 17 and 29
Comparator<int[]> byStart = new Comparator<int[]>() {
    @Override public int compare(int[] a, int[] b) { return Integer.compare(a[0], b[0]); }
};

// 2) Lambda — identical meaning. The compiler already knows the interface (Comparator)
//    and its single method (compare), so you only write the parameters and the body.
Comparator<int[]> byStart2 = (a, b) -> Integer.compare(a[0], b[0]);

// 3) Method reference — when the lambda does nothing except call one existing method
Comparator<String> byLength = Comparator.comparing(String::length);
```

## 2. Functional interface = an interface with exactly one abstract method
A lambda can be used anywhere the target type is a functional interface. The six you will meet:

| Interface | Its one method | Lambda shape | Where you will meet it |
|---|---|---|---|
| `Runnable` | `void run()` | `() -> ...` | threads (Day 29), callbacks |
| `Supplier<T>` | `T get()` | `() -> value` | `orElseThrow(() -> ...)`, Testcontainers `postgres::getJdbcUrl` (Day 47) |
| `Consumer<T>` | `void accept(T)` | `x -> ...` | `forEach`, `ifPresent` |
| `Function<T,R>` | `R apply(T)` | `x -> y` | `map`, `computeIfAbsent(k -> new ArrayList<>())` |
| `Predicate<T>` | `boolean test(T)` | `x -> boolean` | `filter`, `removeIf` |
| `Comparator<T>` | `int compare(T,T)` | `(a, b) -> int` | `sort`, `PriorityQueue` |

Two-argument cousins: `BiFunction`, `BinaryOperator<T>` (this is what `Integer::sum` is). You can declare your own: `@FunctionalInterface interface Validator<T> { boolean isValid(T t); }` — the annotation just makes the compiler check the "exactly one abstract method" rule.

## 3. Syntax rules
- `(a, b) -> a + b` is an expression body (value returned automatically). `(a, b) -> { ...; return x; }` is a block body and needs `return`.
- One parameter: parentheses optional (`x -> x * 2`). No parameters: `() -> ...`.
- Parameter types are inferred from the target type. You rarely write them.
- **A lambda may only use local variables that are effectively final** (assigned once, never changed):
  ```java
  int count = 0;
  list.forEach(x -> count++);   // compile error: local variables referenced from a lambda
                                 // expression must be final or effectively final
  ```
  Why: the lambda may run later or on another thread (Day 29). Fixes: use an `AtomicInteger`, a field, or just a normal `for` loop.
- Inside a lambda, `this` means the *enclosing* object (unlike an anonymous class, where `this` is the anonymous object).

## 4. Method references — four kinds

| Kind | Example | Same as lambda |
|---|---|---|
| static method | `Integer::sum` | `(a, b) -> Integer.sum(a, b)` |
| method on a specific object | `System.out::println` | `x -> System.out.println(x)` |
| method on "whatever object comes in" | `String::length` | `s -> s.length()` |
| constructor | `ArrayList::new` | `() -> new ArrayList<>()` |

## 5. Decode the lambdas you have already copied

| You copied | It means | Note |
|---|---|---|
| `Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]))` | sort rows by first element (`Comparator<int[]>`) | prefer `Integer.compare` over `a[0] - b[0]`: subtraction can overflow |
| `new PriorityQueue<>((a, b) -> b[1] - a[1])` | max-heap on index 1 | `b - a` reverses the order; safe only if the difference cannot overflow |
| `map.merge(key, 1, Integer::sum)` | absent → put 1; present → `Integer.sum(old, 1)` | `Integer::sum` is a `BinaryOperator<Integer>` |
| `graph.computeIfAbsent(u, k -> new ArrayList<>()).add(v)` | get u's list, creating it if missing | the lambda runs only when the key is absent |
| `Comparator.comparing(Employee::getName)` | build a comparator from a "key extractor" | chain `.thenComparing(...)`, `.reversed()` |
| `assertThatThrownBy(() -> service.get(1L))` | hand over *code to run later* so the assertion can catch what it throws | without `() ->`, the exception would fire before the assertion starts |
| `registry.add("spring.datasource.url", postgres::getJdbcUrl)` | a `Supplier` evaluated later, after the container is up | |

## 6. Exercises (30 min)
1. Rewrite Day 17's `Transaction` sort three ways — anonymous class → lambda → `Comparator.comparing(...).thenComparing(...)`. Confirm identical output.
2. Declare `@FunctionalInterface interface Validator<T> { boolean isValid(T value); }`. Write two lambdas (non-blank string; positive int) and `static <T> List<T> keepValid(List<T> in, Validator<T> v)`.
3. Predict, then compile: `int count = 0; list.forEach(x -> count++);`. Read the error. Fix it two ways (`AtomicInteger`; a plain loop).
4. Write `static <T> T retry(int times, Supplier<T> action)`: call `action.get()` up to `times` times, return the first success, rethrow the last exception. (Resilience4j's `@Retry` on Day 64 is this idea with configuration.)

## Checklist
- [ ] Can turn an anonymous `Comparator` into a lambda and into `Comparator.comparing(...)`.
- [ ] Can name the six interfaces in §2 and say what each lambda shape looks like.
- [ ] Can explain "effectively final" and why it exists.
- [ ] Exercises 1–4 pushed to `java-fundamentals`.
- [ ] Continue with Day 30's regular plan.
