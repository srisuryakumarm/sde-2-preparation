# Day 62 — Backtracking Continues, and the Flagship Platform Initializes

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 61 Resource Book](Day61_Resource_Book.md)
**Next ▶:** [Day 63 Resource Book](Day63_Resource_Book.md)
**Companion to:** Day 62 of `Week_09_Revised.md`

---

## Recap

Yesterday's Subsets used include/exclude branching over a fixed set of positions — every element gets exactly one binary decision, independent of every other element. Today's Permutations is deliberately **not** a repeat of that shape: every position's choice depends on which elements earlier positions already claimed, so the mechanism has to change from "two static choices per element" to "swap a candidate into position, recurse, swap back." Same backtracking template (choose → explore → un-choose) from Day 61, applied to a genuinely different decision structure.

On the platform side: Spring Boot, Maven, Docker/Compose, and the full testing stack are all already solid from `todo-api` (Weeks 5–8). Today's Spring AOP is new mechanism; today's multi-module Maven structure is a new *organization* of already-known tools, not new tools themselves.

## Learning Objectives

By the end of today, without notes:

1. Solve Permutations using in-place swapping, and explain precisely why swapping back after each recursive call is what keeps every sibling branch correct — not just a cleanup formality.
2. State what Aspect, Pointcut, and Advice each mean in Spring AOP, and explain the proxy mechanism that makes `@Transactional`-style annotations work — including its self-invocation limitation.
3. Set up a multi-module Maven project with a shared parent POM, and explain the specific difference between `<dependencies>` and `<dependencyManagement>`.

## Concept Dependency Map

```
Backtracking template (Day 61)
        │
        ▼
NEW variant: swap-based choice (position-dependent,
   not independent per element)
        │
        ▼
Permutations (LC 46)
        │
        ▼
🔗 Day 63: Combinations (forward-index, avoids swapping
   entirely) and Permutations II (duplicate handling
   layered on today's swap mechanism)

Independent track — Platform:
Spring Boot / REST (Day 34) + Docker/Compose (Day 43-44)
        │
        ▼
NEW: Spring AOP — Aspect / Pointcut / Advice, proxy-based
        │
        ▼
NEW: Multi-module Maven — parent/child POM, reactor build
        │
        ▼
scalable-ecommerce-platform initializes
        │
        ▼
🔗 Week 10: Resilience4j, Feign, Gateway — all built
   directly on today's module skeleton
```

---

## Part 1 — Permutations

### Problem 12: Permutations (LeetCode 46, Medium) — Pattern: Backtracking

**Statement:** Given an array of distinct integers, return every possible permutation (ordering) of them.

### Approach 1 — Used-array + building a list

```java
// Track a boolean[] used and a growing List<Integer> path;
// at each position, try every not-yet-used number, recurse, then mark unused again.
// Correct, but the boolean[] costs O(n) extra space beyond recursion depth.
```

### Approach 2 — Optimized: in-place swapping

```java
public static List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, result);
    return result;
}

private static void backtrack(int[] nums, int start, List<List<Integer>> result) {
    if (start == nums.length) {
        List<Integer> perm = new ArrayList<>();
        for (int n : nums) perm.add(n);
        result.add(perm);
        return;
    }
    for (int i = start; i < nums.length; i++) {
        swap(nums, start, i);
        backtrack(nums, start + 1, result);
        swap(nums, start, i); // undo — restores nums to what it was before this iteration
    }
}

private static void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

**The mechanism:** `backtrack(nums, start, ...)` fixes everything before index `start` as already decided, and tries every remaining candidate (`i` from `start` to the end) *in that position* by swapping it into place, recursing on the next position, then swapping back. "Swapping back" isn't cleanup — it's what makes the *next* iteration of the `for` loop start from the correct array state again. Without it, the second iteration (`i = start+1`) would recurse on an array already corrupted by the first iteration's swap, producing wrong permutations for every subsequent choice at this level.

**Why this uses O(1) extra space (beyond the recursion stack), unlike the used-array approach:** no auxiliary boolean array is needed — "which elements are already placed" is encoded directly in the array's own layout (indices `0..start-1` are placed, `start..end` are still available), maintained entirely through swaps.

### Worked trace

`nums = [1, 2, 3]`. Tracing `backtrack(nums, 0, ...)`:

`start=0, i=0`: swap(0,0) → `[1,2,3]` (no-op). Recurse `start=1`.
&nbsp;&nbsp;`start=1, i=1`: swap(1,1) → `[1,2,3]`. Recurse `start=2`.
&nbsp;&nbsp;&nbsp;&nbsp;`start=2, i=2`: swap(2,2) → `[1,2,3]`. Recurse `start=3` → **add `[1,2,3]`**. Undo swap(2,2) → `[1,2,3]`.
&nbsp;&nbsp;Undo swap(1,1) → `[1,2,3]`.
&nbsp;&nbsp;`start=1, i=2`: swap(1,2) → `[1,3,2]`. Recurse `start=2`.
&nbsp;&nbsp;&nbsp;&nbsp;`start=2, i=2`: swap(2,2) → `[1,3,2]`. Recurse `start=3` → **add `[1,3,2]`**. Undo → `[1,3,2]`.
&nbsp;&nbsp;Undo swap(1,2) → `[1,2,3]`.
Undo swap(0,0) → `[1,2,3]`.
`start=0, i=1`: swap(0,1) → `[2,1,3]`. Recurse `start=1`.
&nbsp;&nbsp;`start=1, i=1`: swap(1,1) → `[2,1,3]`. Recurse `start=2` → **add `[2,1,3]`**. Undo.
&nbsp;&nbsp;`start=1, i=2`: swap(1,2) → `[2,3,1]`. Recurse `start=2` → **add `[2,3,1]`**. Undo → `[2,1,3]`.
Undo swap(0,1) → `[1,2,3]`.
`start=0, i=2`: swap(0,2) → `[3,2,1]`. Recurse `start=1`.
&nbsp;&nbsp;`start=1, i=1`: swap(1,1) → `[3,2,1]`. Recurse `start=2` → **add `[3,2,1]`**. Undo.
&nbsp;&nbsp;`start=1, i=2`: swap(1,2) → `[3,1,2]`. Recurse `start=2` → **add `[3,1,2]`**. Undo → `[3,2,1]`.
Undo swap(0,2) → `[1,2,3]`.

**Result, in order produced:** `[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,2,1], [3,1,2]` — 6 permutations, `3! = 6`. Every single "undo" step above was load-bearing: skip any one of them and the very next sibling iteration starts from a corrupted array, producing a wrong (or duplicate, or missing) permutation from that point forward.

### Complexity

**Time: O(n × n!)** — `n!` permutations, each requiring `O(n)` to copy into the result.
**Space: O(n)** auxiliary — recursion depth only, no extra array (the input itself is mutated in place and restored).

### Edge cases

- Single-element array — one permutation, the trivial base case, correctly handled without any swap ever occurring meaningfully.
- The swap-back step when `i == start` (a "no-op" swap, as seen in the trace above) — still executes correctly; swapping an element with itself is harmless and doesn't need a special-cased skip.

### Interview framing

**Say before coding:** "I'll fix one position at a time by swapping each remaining candidate into it, recursing on the rest, then swapping back so the next candidate at this position starts from the correct state. This avoids a separate 'used' array — the array's own layout tracks what's already placed."
**Likely follow-up:** "Why not the used-array-plus-growing-list version?" — functionally equivalent output, but costs `O(n)` extra space for the boolean array; the swap version is the tighter, more commonly expected answer once `O(1)` auxiliary space is possible.
**No extra practice today** — Backtracking is still within its opening arc (2 problems in, 10 more required across this week and next), and Week 10's already-dense required ladder (2 problems/day) makes padding unnecessary; see Day 63's closing note for the full reasoning.

---

## Part 2 — Spring AOP (Aspect-Oriented Programming)

### What it is

AOP lets you inject behavior — logging, security checks, transaction management — **around** existing methods, without modifying those methods' own source code. Three vocabulary terms, each answering a different question:

- **Aspect** — *what* to inject (the behavior itself — e.g., "log how long this took").
- **Pointcut** — *where* to inject it (which methods qualify — e.g., "every method annotated `@LogExecutionTime`").
- **Advice** — *when*, relative to the target method's execution: `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`, or `@Around` (which wraps the call entirely, controlling whether/how it even executes).

Spring uses exactly this mechanism internally for `@Transactional` — the annotation itself does nothing; an Aspect intercepts calls to annotated methods and wraps them in transaction begin/commit/rollback logic.

### How it actually works: proxies

Spring AOP is **proxy-based**: for a class with AOP-advised methods, Spring doesn't modify your class's bytecode — it creates a **proxy object** that sits in front of your real object. Every external call goes through the proxy first, which runs the relevant Advice, then delegates to the real method. Two proxy mechanisms, chosen automatically: **JDK dynamic proxies** (when the target implements an interface) or **CGLIB** (subclassing the target class directly, when no interface exists).

**The self-invocation limitation — a real, commonly-tested gotcha:** because the proxy sits *outside* your object, a call from one method to *another method on the same object* (`this.otherMethod()`) never goes through the proxy at all — it's a direct, in-JVM method call, bypassing the Aspect entirely. This is precisely why an object calling its own `@Transactional`-annotated method internally silently gets **no** transactional behavior — the annotation is real, but nothing ever routes the call through the proxy that would have honored it.

> ⚠️ **Common Mistake:** assuming an `@Around`-advised (or `@Transactional`) method behaves consistently regardless of *how* it's called. It doesn't — called from outside the class (through the proxy), the Advice fires; called from another method on the same instance (self-invocation), it silently doesn't. The usual fix is restructuring so the advised method is called on an *injected* reference to the bean (going back through the proxy) rather than via `this`, or splitting the advised logic into a separate bean entirely.

### Custom annotation example

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
}

@Aspect
@Component
public class LoggingAspect {

    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.nanoTime();
        Object result = joinPoint.proceed(); // this IS the actual method call happening
        long elapsedMs = (System.nanoTime() - start) / 1_000_000;
        System.out.printf("%s executed in %dms%n", joinPoint.getSignature(), elapsedMs);
        return result;
    }
}
```

**Why `@Retention(RetentionPolicy.RUNTIME)` is required, specifically:** Spring's AOP proxy mechanism inspects annotations *at runtime* to decide which Advice applies to which method call. A `SOURCE`- or `CLASS`-retention annotation would be discarded before the JVM ever runs, making it invisible to anything checking for it at runtime — the annotation would compile fine but silently do nothing.

**Why `joinPoint.proceed()` sits in the middle, not at the start:** everything before it is "before" behavior (starting the timer); everything after is "after" behavior (computing and logging elapsed time); `proceed()` itself is the actual wrapped method call — this is what makes `@Around` strictly more powerful than `@Before`/`@After` combined: it can skip the call entirely, call it multiple times, or catch and transform its exceptions, none of which the simpler advice types can do.

### Complexity / cost note

AOP's proxy indirection adds a small, constant per-call overhead (one extra method dispatch through the proxy) — negligible for anything doing real work (a DB call, a computation), worth mentioning only if an interviewer specifically asks about AOP's runtime cost.

### Interview framing

**Say before coding:** "An Aspect defines the injected behavior, a Pointcut defines which methods it applies to, and Advice defines the timing relative to the method call. Spring implements this via a proxy that intercepts external calls — which is also exactly why self-invocation bypasses it."
**Likely follow-up:** "Why didn't my `@Transactional` method work when called from elsewhere in the same class?" — this is the self-invocation gotcha; naming it before being asked is a strong, concrete signal of real (not just textbook) Spring experience.

---

## Project Block (1.5 hrs)

**Repository:** `scalable-ecommerce-platform` (new — this is now your one flagship project, not a second throwaway one).
**Task:** initialize a multi-module Maven project (Product, Order, Payment, Notification modules) from the start. Apply your `@LogExecutionTime` aspect to a dummy endpoint in the Order module.
**Definition of done:** all four modules compile independently; the app starts; curling the annotated endpoint prints the execution time via the AOP interceptor.

### Multi-module Maven, precisely

**Parent POM** (`pom.xml` at the repo root):

```xml
<project>
    <groupId>com.yourname.ecommerce</groupId>
    <artifactId>scalable-ecommerce-platform</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>  <!-- NOT jar/war — a parent has no code of its own -->

    <modules>
        <module>product-service</module>
        <module>order-service</module>
        <module>payment-service</module>
        <module>notification-service</module>
    </modules>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-aop</artifactId>
                <version>3.3.0</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

**Each child module's POM** (e.g., `order-service/pom.xml`):

```xml
<project>
    <parent>
        <groupId>com.yourname.ecommerce</groupId>
        <artifactId>scalable-ecommerce-platform</artifactId>
        <version>1.0.0</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <artifactId>order-service</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
            <!-- no version here — inherited from the parent's dependencyManagement -->
        </dependency>
    </dependencies>
</project>
```

**⚠️ Common Mistake — the exact distinction that trips people up:** `<dependencyManagement>` in the parent does **not** add a dependency to any child automatically. It only centralizes *version* management — a child module must still explicitly declare the dependency itself (without a version) inside its own `<dependencies>` block to actually receive it. Skipping the child's own `<dependencies>` entry, expecting the parent's `dependencyManagement` alone to be enough, is a very common and confusing-to-debug mistake, since the build fails with a missing-class error that doesn't obviously point back to a missing POM entry.

**Building everything at once:** run `mvn clean install` from the **root** directory — Maven's reactor resolves inter-module dependencies automatically and builds every module in the correct topological order, without needing to `cd` into each one individually.

**Practical guidance for today's task:** the four modules don't need to depend on each other yet (that starts next week, when Order calls Product via Feign) — today's goal is purely that all four compile and start independently, sharing the AOP dependency version through the parent. Apply `@LogExecutionTime` to one dummy `@GetMapping` endpoint in `order-service` and confirm the console prints an execution time when you curl it.

> 💡 **Interview Insight:** a real production multi-module setup would often add a fifth module (a shared `common`/`api` module for DTOs used across services) rather than having modules depend on each other's internal classes directly — worth naming as the more realistic pattern, even though today's task sticks to the plan's stated four modules.

**Note on `todo-api`:** it stays exactly as it is — a complete, working Spring Boot practice project (health endpoint, Task CRUD, Flyway, Docker/Compose, JUnit/Mockito/TestContainers, Kafka, Trie autocomplete, config profiles, WireMock). It's not getting Kubernetes treatment or further feature work; that infrastructure depth now goes into the platform you'll actually be defending in interviews.

---

## Career Block (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.
**Networking:** apply to 2 more Tier C companies.

---

## Day 62 — Interview Questions

**Q1. In the swap-based Permutations approach, why is "swap back" not just cleanup — what breaks if it's omitted?** The next iteration of the same `for` loop needs the array in its pre-swap state to correctly place its *own* candidate at that position; skipping the undo leaves the array corrupted for every subsequent sibling branch at that level, producing wrong or missing permutations from that point on.

**Q2. Why does the swap-based approach use O(1) extra space where a used-array approach needs O(n)?** "Which elements are already placed" is encoded directly in the array's own layout (indices before `start` are decided, at/after are available) via swapping, rather than tracked in a separate boolean array.

**Q3. Define Aspect, Pointcut, and Advice, each in one sentence.** Aspect — the behavior being injected. Pointcut — which methods it applies to. Advice — when it runs relative to the target method (before/after/around/etc.).

**Q4. How does Spring AOP actually intercept a method call, mechanically?** Via a proxy object (JDK dynamic proxy for interfaces, CGLIB subclassing otherwise) that sits in front of the real bean; every *external* call goes through the proxy first, which runs the relevant Advice before delegating to the real method.

**Q5. Why does an `@Transactional` (or any AOP-advised) method silently lose its behavior when called from another method on the same class?** Self-invocation (`this.method()`) is a direct in-JVM call that never passes through the proxy sitting outside the object — the proxy is what runs the Advice, so bypassing it means the Advice never fires, even though the annotation is technically present.

**Q6. What's the exact difference between `<dependencies>` and `<dependencyManagement>` in a parent Maven POM?** `<dependencyManagement>` only centralizes version numbers for dependencies a child *might* use — it adds nothing automatically. A child module must still declare the dependency itself (without a version) in its own `<dependencies>` block to actually receive it on its classpath.

**Q7. Why must the parent POM use `<packaging>pom</packaging>`?** The parent holds no compiled code of its own — it exists only to declare shared configuration (modules list, dependency management) — `pom` packaging signals there's nothing to compile into a `jar`/`war` at that level.

---

## Daily Deliverable Check

- [ ] Permutations solved via in-place swapping, pushed — full 6-permutation trace for `[1,2,3]` reproduced by hand, matching this book's.
- [ ] Can explain why "swap back" is load-bearing, not cosmetic, without re-deriving it from scratch.
- [ ] `scalable-ecommerce-platform` initialized with its full four-module skeleton; all modules compile independently.
- [ ] `@LogExecutionTime` aspect live and verified on a dummy Order-module endpoint via curl.
- [ ] Can state the self-invocation AOP gotcha unprompted.

---

## What Tomorrow Assumes You Already Know Cold

Day 63 assumes today's swap-based Permutations mechanism is solid, since Permutations II layers duplicate-handling directly on top of it without re-deriving the swap/undo shape from scratch. It also assumes the multi-module Maven skeleton from today is stable and won't need revisiting, since Week 10's Resilience4j and Feign work (Days 64–65) builds new functionality *inside* today's `order-service` module rather than restructuring the project itself.

**Next:** [Day 63 Resource Book](Day63_Resource_Book.md) — Week 9 Consolidation, and Backtracking Continues.
