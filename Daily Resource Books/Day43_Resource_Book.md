# Day 43 — Stack Simulation Problems, and Docker Basics

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 42 Resource Book](Day42_Resource_Book.md)
**Next ▶:** [Day 44 Resource Book](Day44_Resource_Book.md)
**Companion to:** Day 43 of `Week_07_Revised.md`

---

## Recap

Week 6 opened Stacks/Monotonic Stack (Day 39's concept card, Day 40–41's increasing/decreasing invariant and the amortized "each element pushed and popped at most once" argument) and got it to 7 of 12 required problems. Today and the next two days finish it off: five problems, three of them Hard, all reusing the same LIFO discipline and the same amortized argument — none of them re-derive it from scratch. (This week's overlap check against `00_Curriculum_Map.md` and `Week_08_Revised.md` came back clean — nothing below is a repeat, and nothing added here collides with what Week 8 requires.)

Today's two problems are both "use a stack to hold state you'll need later" rather than strictly monotonic — Asteroid Collision simulates physical collisions, Basic Calculator II tracks pending arithmetic. The theory block starts an entirely fresh thread: Docker, which today's project wraps around the `todo-api` you've been building since Day 34.

## Learning Objectives

By the end of today, without notes:

1. Solve Asteroid Collision and Basic Calculator II, explaining why each reaches for a stack specifically — what state is being deferred, and why LIFO order is the correct order to resolve it in.
2. State the brute-force approach for each problem and its complexity, and argue precisely why the stack approach is faster.
3. Explain the difference between a Docker image and a container, and trace exactly which layer of a `Dockerfile` gets rebuilt when source code changes.
4. Write a multi-stage `Dockerfile` and explain why the final image doesn't contain Maven or your source `.java` files.

## Concept Dependency Map

```
Day 39 — Stack (LIFO), formal pattern
Day 40/41 — Monotonic Stack invariant + amortized "pushed/popped once" proof
        │
        ▼
Today: Stack as a SIMULATION tool (not strictly monotonic)
  ├─ LC 735 Asteroid Collision — stack holds "asteroids still in flight,
  │  not yet resolved" — LIFO because the most recently pushed (i.e.
  │  nearest) survivor is exactly the one a new left-mover meets first
  └─ LC 227 Basic Calculator II — stack holds "terms pending a final
     +/- sum" — a pushed term can still be corrected in place when a
     later '*' or '/' arrives, which is exactly what stack.pop() then
     push(corrected) gives you
        │
        ▼
NEW: Docker — image (layered snapshot) vs. container (running instance);
Dockerfile layer caching, ordered least-to-most-frequently-changing
        │
        ▼
Project: todo-api gets a multi-stage Dockerfile
```

---

# Part 1 — Stack as a Simulation Tool

Days 40–41 used the stack to maintain a strictly monotonic invariant (find the next-greater element, etc.). Today's two problems use the stack differently: as a place to hold *provisional, still-correctable* results — a survivor that might still get destroyed, a running term that might still get multiplied. The LIFO discipline still matters for the same underlying reason it always does: **the thing you need to inspect or correct next is always the most recently deferred thing** — the nearest asteroid, the most recently pushed term.

---

## Problem 8: Asteroid Collision (LeetCode 735, Medium) — Pattern: Stack Simulation

**Statement:** Given an array of integers representing asteroids in a row, where each asteroid's absolute value is its size and its sign is its direction (positive = right, negative = left, all moving at the same speed), return the state of the asteroids after all collisions. Same-direction asteroids never meet. On collision, the smaller explodes; equal sizes both explode.

### Approach 1 — Brute force: repeatedly rescan for the next collision

```java
public static int[] asteroidCollisionBruteForce(int[] asteroids) {
    List<Integer> list = new ArrayList<>();
    for (int a : asteroids) list.add(a);

    boolean collisionHappened = true;
    while (collisionHappened) {
        collisionHappened = false;
        for (int i = 0; i < list.size() - 1; i++) {
            int left = list.get(i), right = list.get(i + 1);
            if (left > 0 && right < 0) {           // the ONLY adjacent pattern that can collide
                if (left < -right) {
                    list.remove(i);                  // left explodes
                } else if (left == -right) {
                    list.remove(i + 1);               // both explode
                    list.remove(i);
                } else {
                    list.remove(i + 1);               // right explodes
                }
                collisionHappened = true;
                break;                                // restart — a new adjacency may now exist
            }
        }
    }
    int[] result = new int[list.size()];
    for (int i = 0; i < result.length; i++) result[i] = list.get(i);
    return result;
}
```

Every collision can only ever happen between a *right-mover immediately followed by a left-mover* — that's the only adjacent pair that's moving toward each other. This scans for that pattern, resolves the first one found, and restarts from the top, because resolving one collision can create a brand-new adjacency further left (a chain reaction). Correct, but each restart is a fresh O(n) scan, and in the worst case (a long chain of collisions) you restart O(n) times: **O(n²)**.

### Approach 2 — Optimized: single-pass stack

```java
public static int[] asteroidCollision(int[] asteroids) {
    Deque<Integer> stack = new ArrayDeque<>();  // asteroids "still alive so far", left to right
    for (int asteroid : asteroids) {
        boolean alive = true;
        while (alive && asteroid < 0 && !stack.isEmpty() && stack.peek() > 0) {
            int top = stack.peek();
            if (top < -asteroid) {
                stack.pop();               // top explodes; current keeps moving left, checks the next top
            } else if (top == -asteroid) {
                stack.pop();                // both explode
                alive = false;
            } else {
                alive = false;              // current explodes; top survives
            }
        }
        if (alive) stack.push(asteroid);
    }
    int[] result = new int[stack.size()];
    for (int i = result.length - 1; i >= 0; i--) result[i] = stack.pop();  // stack is bottom-to-top; reverse it
    return result;
}
```

**Why this is a single pass, not a rescan:** the stack holds exactly "every asteroid that has survived everything seen *so far*, nearest-first from the top." A right-mover (`asteroid > 0`) can never collide with anything already on the stack — it's moving away from everyone behind it — so it's always just pushed. A left-mover only collides with the stack's *top* (the nearest survivor); if that top loses, it's popped and the same left-mover keeps checking the new top, which correctly handles a left-mover that's strong enough to destroy several right-movers in a row. Each asteroid is pushed at most once and popped at most once — the same amortized argument from Day 41, applied to a new problem shape rather than re-derived.

**Trace:** `asteroids = [10, 2, -5]`.
| asteroid | action | stack (bottom→top) |
|---|---|---|
| 10 | positive, push | [10] |
| 2 | positive, push | [10, 2] |
| -5 | top=2: `2 < 5` → 2 explodes, pop | [10] |
| (continuing) | top=10: `10 < 5`? No. `10 == 5`? No. → -5 explodes, `alive=false` | [10] |

Final stack `[10]` → result `[10]`. Matches the expected output for this exact input.

**Complexity: Time O(n) — Day 41's amortized argument, restated: total pushes and pops across the whole run are each bounded by n. Space O(n)** — worst case, nothing collides (e.g., all moving right) and every asteroid survives onto the stack.

**Edge cases:** two equal-magnitude colliding asteroids (both explode — the `==` branch, easy to drop by accident if you only write `<` and `>`); a left-mover strong enough to destroy multiple stacked right-movers in sequence (the `while`, not `if`, is what makes this work — using `if` instead is the single most common bug on this problem); asteroids moving apart or in the same direction (never collide — `[−2, −1, 1, 2]` is already final).

> 💡 **Interview Insight:** State out loud, before coding, exactly which adjacent configuration can ever collide (right-mover immediately followed by left-mover) — it's the fact that makes "stack, LIFO" the obvious tool rather than something you have to justify after the fact. The `while` vs. `if` distinction above is a near-guaranteed follow-up if your first draft uses `if`.

---

## Problem 9: Basic Calculator II (LeetCode 227, Medium) — Pattern: Stack

**Statement:** Evaluate a string expression containing non-negative integers and the operators `+ - * /` (integer division truncates toward zero), with no parentheses.

### Approach 1 — Brute force: two-pass token splicing

```java
public static int calculateBruteForce(String s) {
    List<String> tokens = tokenize(s);   // e.g. "3+5/2" -> ["3","+","5","/","2"]

    boolean sawMultDiv = true;
    while (sawMultDiv) {
        sawMultDiv = false;
        for (int i = 1; i < tokens.size() - 1; i++) {
            String op = tokens.get(i);
            if (op.equals("*") || op.equals("/")) {
                int left = Integer.parseInt(tokens.get(i - 1));
                int right = Integer.parseInt(tokens.get(i + 1));
                int result = op.equals("*") ? left * right : left / right;
                tokens.set(i - 1, String.valueOf(result));
                tokens.remove(i + 1);
                tokens.remove(i);
                sawMultDiv = true;
                break;                     // restart the scan after every splice
            }
        }
    }

    int result = Integer.parseInt(tokens.get(0));
    for (int i = 1; i < tokens.size(); i += 2) {
        int next = Integer.parseInt(tokens.get(i + 1));
        result = tokens.get(i).equals("+") ? result + next : result - next;
    }
    return result;
}
```

Tokenize once (O(n)), then resolve every `*`/`/` by repeatedly scanning the token list and splicing out the three tokens involved, restarting after each splice — because `List.remove(i)` shifts every later element down by one, each splice costs O(n), and there can be O(n) operators, giving **O(n²)**. The final left-to-right `+`/`-` pass is O(n) and doesn't change the overall bound.

### Approach 2 — Optimized: single pass, `lastSign` trick

```java
public static int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int num = 0;
    char lastSign = '+';   // the operator that applies to the number currently being built

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (Character.isDigit(c)) {
            num = num * 10 + (c - '0');
        }
        if ((!Character.isDigit(c) && c != ' ') || i == s.length() - 1) {
            switch (lastSign) {
                case '+': stack.push(num); break;
                case '-': stack.push(-num); break;
                case '*': stack.push(stack.pop() * num); break;
                case '/': stack.push(stack.pop() / num); break;
            }
            lastSign = c;
            num = 0;
        }
    }

    int result = 0;
    for (int val : stack) result += val;
    return result;
}
```

**The key idea:** every `+` or `-` term just gets pushed (with its sign folded in) — it's final the moment it's pushed, and nothing later can change it. A `*` or `/`, though, needs to *correct* the term that's already on the stack: pop it, combine it with the new number, push the corrected value back. That's precisely why a stack — not a running total — is the right container: the most recently pushed term is exactly the one a `*` or `/` needs to reach back and fix. At the very end, the stack holds a set of already-signed terms with no operators left to apply, so the answer is just their sum.

`lastSign` always holds the operator that precedes the number *currently being accumulated* — not the operator you just read. That's why it's updated to `c` only *after* the current number has been finalized against the *previous* `lastSign`.

**Trace:** `s = "3+5/2"`.
| i | c | num | action | stack | lastSign after |
|---|---|---|---|---|---|
| 0 | `3` | 3 | digit, keep building | — | `+` |
| 1 | `+` | — | finalize 3 with `lastSign='+'` → push 3 | `[3]` | `+` |
| 2 | `5` | 5 | digit, keep building | `[3]` | `+` |
| 3 | `/` | — | finalize 5 with `lastSign='+'` → push 5 | `[3, 5]` | `/` |
| 4 | `2` | 2 | digit; **also last index** → finalize 2 with `lastSign='/'` → pop 5, push `5/2=2` | `[3, 2]` | — |

Sum of stack `= 3 + 2 = 5`. `3 + 5/2` with integer division is `3 + 2 = 5`. Correct.

**Complexity: Time O(n) — one pass, O(1) work per character amortized. Space O(n)** — worst case, every term is `+`/`-` and stays on the stack (a run of `*`/`/` collapses the stack, so this is the true worst case, not an average).

**Edge cases:** leading/embedded spaces (skipped — they fail both parts of the trigger condition unless at the very last index, which the `!= ' '` check specifically excludes even there); integer division truncates toward zero, not floors — `-3 / 2` is `-1` in Java, not `-2` (Day 10's overflow/primitives block covered this rounding behavior; it applies unchanged here); a single number with no operator at all (the end-of-string branch still fires and pushes it).

> 💡 **Interview Insight:** Narrate the "why a stack, specifically" reasoning before coding: `+`/`-` terms are final on arrival, `*`/`/` terms require reaching back and correcting the most recent entry — that asymmetry is the whole justification. A likely follow-up is "how would you extend this to handle parentheses?" — the honest answer previews Day 45's Basic Calculator: you'd need to save `(result, sign)` context on the stack every time you enter a `(`, which is a materially different problem, not a small tweak to this one.

---

# Part 2 — Docker Basics

### Prerequisites (confirmed)

None from the DSA/Java track — this is a fresh theory thread, the same way Threads (Day 29) and SQL (Day 40) each started fresh. It does assume the `todo-api` project exists and currently runs via `mvn spring-boot:run` against a local Postgres (Day 36, Flyway-managed since Day 40) — today wraps that existing, unchanged application; nothing about its code or schema changes today.

### Image vs. Container

An **image** is a static, read-only, layered snapshot — your application plus everything it needs to run (a JRE, OS libraries, your `.jar`), built once from a `Dockerfile` and then never modified. A **container** is a running instance of an image, with a thin writable layer on top for anything the running process changes (logs, temp files) — that writable layer is discarded when the container is removed, while the image underneath is untouched and can spawn any number of fresh containers.

This is the same relationship as a **class and an object** (Day 2): the image is the template, the container is a live instance — and just as multiple objects can be built from one class without affecting each other, multiple containers can run from one image without affecting each other or the image itself.

### The `Dockerfile` and Layer Caching

Each instruction in a `Dockerfile` (`FROM`, `COPY`, `RUN`, ...) produces one **layer**, stacked on the ones before it. Docker caches every layer by hashing its instruction plus its inputs. On a rebuild, Docker walks the `Dockerfile` top to bottom and reuses a cached layer **only if that exact layer is unchanged from the last build** — the instant one layer's inputs differ, that layer is rebuilt, and *every layer after it* is rebuilt too, cache or not, because each layer is built on top of the specific layer before it.

This is the entire reason layer *order* matters:

```dockerfile
# Naive ordering — cache-hostile
FROM eclipse-temurin:21-jdk
WORKDIR /app
COPY . .                          # <-- includes source AND pom.xml
RUN mvn dependency:go-offline     # re-downloads every dependency, every single code change
RUN mvn package -DskipTests
```

```dockerfile
# Ordered least-to-most-frequently-changing
FROM eclipse-temurin:21-jdk
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline     # cached — only re-runs when pom.xml itself changes
COPY src ./src                    # changes on nearly every commit
RUN mvn package -DskipTests
```

`pom.xml` changes rarely (only when a dependency is added or bumped); `src/` changes on almost every commit. Copying `pom.xml` and resolving dependencies *before* copying source means that layer is reused on nearly every build — only the `COPY src` layer and everything after it gets rebuilt on a normal code change, instead of re-downloading the entire dependency tree from the internet every time.

> ⚠️ **Common Mistake:** `COPY . .` as the very first `COPY`, before dependencies are resolved. It looks harmless — it's fewer lines — but it means *any* file change, including a one-line source edit, invalidates the dependency-download layer too, since Docker only compares whether the layer's inputs changed, not whether they changed *in a way that matters to that specific `RUN` command*.

### Multi-Stage Builds

A single-stage image built with the `Dockerfile` above would ship with the full JDK, Maven, and your source tree baked in — none of which the *running* application needs. A **multi-stage build** uses one stage to compile, and a second, separate, minimal stage to run — only the compiled artifact crosses between them:

```dockerfile
# ---- Stage 1: build ----
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# ---- Stage 2: run ----
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/todo-api-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

`COPY --from=build` reaches into Stage 1's filesystem and pulls out only the built `.jar` — Maven, the JDK compiler, and your `.java` sources never make it into the final image at all. The final image is built on a `-jre-alpine` base (a JRE, not a full JDK, on a minimal Linux distribution) — smaller to pull and start, and with a meaningfully smaller attack surface, since there's no compiler or build tooling present for an attacker to abuse even if the container were compromised.

### Common Mistakes

- ⚠️ **`COPY . .` before dependency resolution** — invalidates the expensive dependency-download layer on every source change (above).
- ⚠️ **Single-stage builds for compiled languages** — ships the entire build toolchain (JDK, Maven, and its cached `~/.m2` dependencies) inside the image you actually deploy, often 3–5× larger than necessary.
- ⚠️ **No `.dockerignore`** — without one, `COPY . .` sends your local `target/`, `.git/`, and IDE folders into the build context, bloating build time and occasionally leaking files you didn't mean to ship.
- ⚠️ **Treating the image as a VM** — a container shares the host's kernel (no separate OS boot, no hypervisor), which is exactly why it starts in milliseconds rather than the seconds-to-minutes a VM takes, but it also means container isolation is weaker than VM isolation — worth knowing as a trade-off, not just a performance win.

---

# Project Block Guide (1.5 hrs)

**Repository:** `todo-api`. Write the two-stage `Dockerfile` above (adjusted for your actual `artifactId`/version in `pom.xml` if it differs from the placeholder), plus a `.dockerignore` excluding at minimum `target/`, `.git/`, and `*.iml`.

**Definition of done:** `docker build -t todo-api .` succeeds, and `docker run -p 8080:8080 todo-api` serves requests correctly on `localhost:8080` — confirm with the same endpoint you've been hitting locally since Day 34. Worth doing once: run `docker images` and note the size difference between a quick single-stage build and your multi-stage one, since that's the concrete number your LinkedIn post below is asking you to measure.

---

# Career Block Guide (1 hr)

### LinkedIn — Post 10

> Spent today wrapping `todo-api` in a multi-stage Docker build — one stage to compile with Maven, a second, minimal stage that only ships the JRE and the built jar.
>
> Single-stage image: ~[X] MB. Multi-stage: ~[Y] MB — about [Z]% smaller, since the final image never carries Maven, the JDK compiler, or my source tree, only what's actually needed to run.
>
> Small change, but it's the same principle as everywhere else in engineering: ship exactly what's needed to do the job, nothing the job doesn't require.
>
> Day 43 of SDE-2 prep. #buildinpublic #docker #springboot

Fill in `[X]`, `[Y]`, `[Z]` from your own `docker images` output before posting — a real, measured number is the entire point of this post.

### Networking

Apply to 2 Tier C ("practice") companies — the lower-stakes applications this plan uses to get real interview reps in before Tier A/B loops matter.

---

# Day 43 — Interview Questions

**Q1. Why does Asteroid Collision reach for a stack instead of, say, two pointers?**
The problem requires comparing each new left-moving asteroid against the *most recently surviving* asteroid to its left, and a collision can cascade backward through several previous survivors. A stack's LIFO order gives you exactly "the nearest survivor" in O(1), and popping destroyed asteroids off the top naturally exposes the next-nearest one to check against.

**Q2. In Asteroid Collision, why is the collision-check a `while` loop and not an `if`?**
A single large left-mover can destroy multiple smaller right-movers in a row. An `if` would only resolve one collision per asteroid and incorrectly leave the rest of the destroyed chain on the stack; the `while` keeps resolving against the new top until the current asteroid is destroyed, survives outright, or the stack no longer opposes it.

**Q3. What's the one adjacency pattern that can ever produce a collision, and why does that fact justify the stack approach?**
Only a right-mover immediately followed (at some point) by a left-mover can collide — anything moving apart or in the same direction never meets. This is exactly what lets a single left-to-right pass work: by the time you reach a left-mover, everything that could possibly collide with it is already sitting on the stack.

**Q4. Walk through why Basic Calculator II's brute-force splicing approach is O(n²) while the stack approach is O(n).**
Splicing removes tokens from a `List`, and each removal shifts every subsequent element down by one — an O(n) operation — repeated for up to O(n) operators, giving O(n²) total. The stack approach makes one pass, doing O(1) amortized work per character (a push, or a pop-then-push), for O(n) total.

**Q5. What does `lastSign` actually track, and why is it updated *after* processing the current number rather than *when* the operator character is first seen?**
`lastSign` holds the operator that applies to the number currently being accumulated — which is the operator seen *before* this number started, not the one just encountered. The current character is only ever the operator for the *next* number, so it's stored into `lastSign` only after the just-finished number has been finalized against the *previous* value of `lastSign`.

**Q6. Why does a `*` or `/` in Basic Calculator II require popping from the stack, while `+` and `-` don't?**
A `+`/`-` term is final the instant it's pushed — nothing later in a left-to-right scan can change it. A `*`/`/` needs to correct the term that's already on top of the stack (multiply or divide it by the new number), so the stack has to expose — and let you replace — its most recently pushed value, which is exactly what pop-then-push gives you.

**Q7. What's the difference between a Docker image and a container?**
An image is a static, layered, read-only snapshot built once from a `Dockerfile`. A container is a running instance of that image, with a thin writable layer on top for runtime changes; that writable layer is discarded when the container is removed, and the underlying image is never modified by running (or removing) a container.

**Q8. If you edit one line in `TaskController.java` and rebuild, which Docker layers get rebuilt, assuming the cache-friendly `Dockerfile` ordering from today?**
Only the `COPY src ./src` layer and everything after it (the `mvn package` layer) — the earlier `COPY pom.xml .` and `RUN mvn dependency:go-offline` layers are unaffected by a source change and are reused straight from cache.

**Q9. Why does putting `COPY . .` before dependency resolution hurt build times, specifically?**
It bundles your source files into the same layer's cache key as your dependency list. Since source changes on nearly every commit, that layer's cache key changes on nearly every commit too — invalidating the (expensive, network-bound) dependency-download step on every single build, even though the dependencies themselves didn't actually change.

**Q10. What does a multi-stage build remove from the final image that a single-stage build would include, and why does that matter?**
It removes the JDK compiler, Maven itself, and the full source tree — only the compiled `.jar` crosses from the build stage into the minimal runtime stage. This produces a smaller image (faster to pull and start) and a smaller attack surface, since there's no build tooling present in the deployed container at all.

---

# Daily Deliverable Check

- [ ] Asteroid Collision (LC 735) and Basic Calculator II (LC 227) solved — both approaches understood, not just the optimized one — pushed to `dsa-java/stacks/`.
- [ ] `todo-api` has a working multi-stage `Dockerfile` and a `.dockerignore`.
- [ ] `docker run -p 8080:8080 todo-api` serves requests correctly.
- [ ] Single-stage vs. multi-stage image size measured with `docker images`.
- [ ] LinkedIn Post 10 published with the real measured numbers filled in.
- [ ] Applied to 2 Tier C companies.

---

## What Tomorrow Assumes You Already Know Cold

Day 44 assumes today's `Deque<Integer>` stack mechanics are fully reflexive (no re-explanation of push/pop/peek), and that you can push and pop *indices* just as comfortably as values — Largest Rectangle in Histogram stores indices on the stack, not heights directly, which is a small but real shift from today's two problems. It also assumes today's image-vs-container distinction is solid, since Docker Compose (tomorrow's theory) is about orchestrating *multiple* containers together, not a new mental model for what a single container is.
