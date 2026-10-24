# Day 30 — Missing Primer: Debugging and Reading Stack Traces

**Do this first on Day 30 (≈ 60 min, before the Lambdas and Streams primers), then continue with Day 30's regular plan.**
**Needed by:** Day 34 onward (every Spring startup error arrives as a stack trace), Day 45 (failing tests), Day 75 (thread dumps), Day 108 (existing-codebase rounds are mostly reading and debugging) — and every day you get a wrong answer on a DSA problem.
**Why it is missing:** across all 148 books the words "stack trace" appear nowhere and "debugger"/"breakpoint" appear once, on Day 1. Day 18 teaches exceptions but never how to *read* one or how to *find* a bug.
**Already in place:** IntelliJ and the terminal (Day 1), exceptions (Day 18), recursion and the call stack (Days 7–9), logging with SLF4J (Day 32's file).

---

## 1. Debugging is a method, not a talent
**Reproduce → isolate → hypothesize → test the hypothesis → fix → prevent a repeat.** Change one thing at a time. Never "fix" what you cannot reproduce. After the fix, add a test or a check so the bug cannot return.

## 2. Reading a Java stack trace
```
Exception in thread "main" java.lang.NullPointerException:
    Cannot invoke "java.lang.Integer.intValue()" because the return value of "java.util.Map.get(Object)" is null
        at com.you.shop.StockService.available(StockService.java:42)      ← YOUR code: start here
        at com.you.shop.OrderService.place(OrderService.java:17)
        at com.you.shop.Main.main(Main.java:9)
Caused by: java.io.IOException: ...                                        ← the ROOT cause is the LAST "Caused by"
```
1. **First line:** the exception type and message. Modern JVMs (Java 15+) print a *helpful* `NullPointerException` message that names the null expression.
2. **Frames** list the call stack, **most recent call first**. Skip framework frames (`org.springframework…`, `java.base…`) and find the **first frame in your own package** — that is where to look.
3. **`Caused by:`** chains show wrapped exceptions; the **last** one is usually the real problem. Spring wraps startup failures several layers deep — read to the bottom.
4. Never swallow exceptions (`catch (Exception e) {}`) and never lose the cause: log with `log.error("order failed", e)`, not `e.getMessage()`.

| Exception | Usual meaning |
|---|---|
| `NullPointerException` | you used a reference that is `null` (often `Map.get` of a missing key) |
| `ArrayIndexOutOfBoundsException` / `IndexOutOfBounds` | off-by-one, or a loop bound `<=` that should be `<` |
| `ClassCastException`, `NumberFormatException` | wrong type or malformed input |
| `ConcurrentModificationException` | you changed a collection while iterating it |
| `StackOverflowError` | recursion without a reachable base case |
| `OutOfMemoryError` | see Day 75's file |

## 3. The IntelliJ debugger (macOS defaults — check the Run menu if yours differ)
- **Breakpoint:** click the line-number gutter. **Debug** (the bug icon) instead of Run.
- **Step Over** `F8` (run this line) · **Step Into** `F7` (enter the call) · **Step Out** `⇧F8` · **Resume** `⌥⌘R` · **Evaluate Expression** `⌥F8` (run any expression on the paused state).
- **Variables / Watches / Frames** panes: inspect values, pin an expression, and click a frame to jump *up the call stack* and see its variables.
- **Conditional breakpoint** (right-click it): stop only when `i == 998`. **Log breakpoint** (uncheck *Suspend*): print a message without stopping. **Exception breakpoint** (*Run → View Breakpoints → Java Exception Breakpoints*): stop at the throw, wherever it happens.
- **Spring Boot:** run the main class in Debug, or start `./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005"` and attach IntelliJ with *Run → Attach to Process* / a Remote JVM configuration on port 5005.
- **Threads:** the Threads & Variables tab shows every thread (Day 29/37); a frozen program is a deadlock until proven otherwise.

## 4. Debugging without the debugger
Print/log the *state* (not "here 1"); write the smallest input that fails (a **minimal reproduction**); **dry-run** a DSA solution on a 3-element input in a table of variables; assert your assumptions (`assert x > 0`); bisect the *history* with `git bisect` (Day 139's file) or the *code* by commenting out half; read the docs and the library source (*Go to Declaration*, *Download Sources*); explain the bug aloud ("rubber duck") — the bug often appears mid-sentence.

## 5. Exercise (≈ 35 min, `java-fundamentals`)
For each snippet: **predict** the failure, **run** it, **read** the trace, **set** a breakpoint (or exception breakpoint), **fix** it, then add a check in `main` that would have caught it.
```java
// 1. NullPointerException
Map<String, Integer> stock = new HashMap<>();
stock.put("pen", 10);
int n = stock.get("pencil");

// 2. ConcurrentModificationException
List<Integer> xs = new ArrayList<>(List.of(1, 2, 3, 4));
for (Integer x : xs) if (x % 2 == 0) xs.remove(x);

// 3. StackOverflowError
static long fact(long n) { return n == 1 ? 1 : n * fact(n - 1); }   // call fact(0)

// 4. ArrayIndexOutOfBoundsException
int[] a = {3, 1, 2};
for (int i = 0; i <= a.length; i++) System.out.println(a[i]);
```
Then take one of your own wrong-answer LeetCode attempts from Days 20–29 and find the bug **with the debugger** instead of by staring.

## Checklist
- [ ] Can read a trace: exception, message, first frame in my code, and the last `Caused by`.
- [ ] Can use breakpoints, Step Over/Into/Out, Evaluate Expression, a conditional breakpoint and an exception breakpoint.
- [ ] Can describe the reproduce-isolate-hypothesize loop and the minimal-reproduction habit.
- [ ] Exercise 1–4 done and pushed. Continue with Day 30's regular plan.
