# Day 65 Resource Book — Backtracking: Phone Letters and Parentheses, and Feign Clients

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 64 Resource Book](Day64_Resource_Book.md)
**Next ▶:** [Day 66 Resource Book](Day66_Resource_Book.md)
**Companion to:** Day 65 of `Week_10_Revised.md`

---

## Recap

Yesterday's two problems both *selected elements from an array* — the recursion's job was choosing which candidates belong in a combination. Both of today's problems build a **string**, one character at a time, from a small, per-position set of choices — a genuinely different shape for the same choose/explore/un-choose template, not a variant of yesterday's forward-index mechanism. On the platform side: yesterday's Resilience4j handled what happens when a downstream call *fails*; today's Feign handles *making* that downstream call in the first place, declaratively.

**🔗 Confirming a note from the curriculum map:** Letter Combinations of a Phone Number (today's Problem 7) was seriously considered as Week 9 *extra* practice, then confirmed — checking `Week_10_Revised.md` directly — to already be required here. It was correctly **not** added last week. This is that confirmation: LC 17 appears exactly once in this series, right here, marked required.

---

## Learning Objectives

By the end of today, without notes:

1. Solve Letter Combinations of a Phone Number and Generate Parentheses, explaining why both are backtracking despite neither selecting from an array of candidates.
2. Explain, with a proof (not an assertion), why Generate Parentheses' `close < open` condition is sufficient to guarantee every generated string is well-formed.
3. Contrast building a path with a mutable `StringBuilder` (needs an explicit undo) against building it via immutable `String` concatenation (doesn't) — and state which one this series has been using all along, and why.
4. Explain what Feign generates from an interface, and why calling a real Product module (not a mocked one) makes today's project a meaningfully more honest exercise than testing against a placeholder.

---

## Concept Dependency Map for Today

```
Day 61 — Backtracking template (choose → explore → un-choose)
Day 8  — Recursion, call stack
        │
        ▼
Today, Problem 1 — Letter Combinations of a Phone Number (LC 17)
  NEW shape: build a STRING one position at a time; choices come from a
  small fixed per-digit letter set, not from indices into the input itself
        │
        ▼
Today, Problem 2 — Generate Parentheses (LC 22)
  Same string-building shape, but the choice at each step isn't "pick
  from a fixed small set" — it's "is '(' legal here? is ')' legal here?" —
  validity is tracked via two running counters, not a lookup table

Day 34 — REST/Spring Boot, HTTP verbs        Day 64 — Resilience4j (proxy,
Day 2  — Interfaces                            fallback methods, reflection
        │                                       match)
        ▼                                             │
Feign Clients (NEW) — a declarative interface,         │
  proxy-generated at startup into a real HTTP          ◀───────────────────┘
  client; fallback shares Resilience4j's exact
  "same signature, extra param" convention
```

---

# Part 1 — Letter Combinations of a Phone Number (LeetCode 17, Medium) — Pattern: Backtracking

**Statement:** Given a string `digits` containing digits `2`–`9`, return every possible letter combination the number could represent, using the standard telephone keypad mapping (`2`→"abc", `3`→"def", ..., `7`→"pqrs", `8`→"tuv", `9`→"wxyz"). Return an empty list if `digits` is empty.

## Why this is a new backtracking shape, not a variant of Days 61–64's

Every backtracking problem so far has picked a *subset or arrangement of elements already sitting in an input array*. This problem builds a brand-new string, one character at a time, where the set of legal choices at each step comes from a **small, fixed, external lookup** (the keypad mapping) — not from "which array indices are still available." The recursion depth here is always exactly `digits.length()` (one recursive call per digit, no early variable-length stopping the way Subsets or Combination Sum had) — the *branching factor* varies (3 or 4 letters per digit), but the *depth* doesn't.

### Approach — Backtrack one digit position at a time

```java
public static List<String> letterCombinations(String digits) {
    List<String> result = new ArrayList<>();
    if (digits == null || digits.isEmpty()) {
        return result;   // required: LeetCode specifies [] for empty input, not [""]
    }
    String[] mapping = {"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
    backtrack(digits, 0, new StringBuilder(), mapping, result);
    return result;
}

private static void backtrack(String digits, int index, StringBuilder path,
                               String[] mapping, List<String> result) {
    if (index == digits.length()) {
        result.add(path.toString());   // base case: one letter chosen per digit, done
        return;
    }
    String letters = mapping[digits.charAt(index) - '0'];
    for (char letter : letters.toCharArray()) {
        path.append(letter);                              // choose
        backtrack(digits, index + 1, path, mapping, result);  // explore
        path.deleteCharAt(path.length() - 1);              // un-choose
    }
}
```

**Why `mapping` is a plain `String[]` indexed by digit, not a `HashMap<Character,String>`:** the exact same reasoning Day 5's Valid Anagram used for choosing a fixed-size array over a HashMap — the key space (digits 0–9) is small, known in advance, and contiguous. `digits.charAt(index) - '0'` converts the character digit directly to its array index (the same char-arithmetic trick from Day 5's frequency arrays), avoiding both hashing overhead and `Character`/`String` autoboxing for a 10-entry lookup that never changes.

**Why `StringBuilder` + explicit undo here, when earlier problems' `path` was a `List<Integer>`:** the mechanism is identical in spirit — a single shared mutable structure represents the current partial answer, appended to on choose and shrunk on un-choose. `StringBuilder.deleteCharAt(length - 1)` is `StringBuilder`'s equivalent of `List.remove(size - 1)`. Strings are immutable (Day 11); building one via repeated concatenation instead (`path + letter`, no explicit undo needed at all, since each call would get its own independent `String`) is possible and *would* still be correct backtracking — it just trades away Day 11's `+=`-in-a-loop-is-O(n²) lesson for an easier mental model. `StringBuilder` avoids that cost, which is exactly why it's the right choice here, not `StringBuilder` because "backtracking requires a shared mutable structure" as a rigid rule.

### The required empty-input edge case, made explicit

Without the guard at the top, `backtrack("", 0, ...)` would immediately satisfy `index == digits.length()` (`0 == 0`) on the very first call and add the empty string `""` to the result — producing `[""]`. LeetCode's specification requires `[]` for empty input, not a list containing one empty string, so this guard isn't defensive style, it's a correctness requirement.

### Trace: `digits = "23"`

```
backtrack(index=0, path="")
  digit '2' → letters "abc"
  'a': path="a" → backtrack(index=1, path="a")
         digit '3' → letters "def"
         'd': path="ad" → backtrack(index=2) → index==len(2) → ADD "ad"
         'e': path="ae" → ADD "ae"
         'f': path="af" → ADD "af"
       (path restored to "a", then to "")
  'b': path="b" → same three: ADD "bd","be","bf"
  'c': path="c" → same three: ADD "cd","ce","cf"
```

**Result: `["ad","ae","af","bd","be","bf","cd","ce","cf"]`** — 9 combinations, matching `3 × 3` (three letters for `'2'`, three for `'3'`).

### Complexity

**Time: O(4ⁿ)** where `n = digits.length()` — the worst-case branching factor is 4 (digits `7` and `9` each map to 4 letters), and the tree has depth `n`, so the total number of leaf combinations (and thus total work, since building each string costs O(n) but that factor is often folded into a looser combined bound) is bounded by `4ⁿ`. **Space: O(n)** for the recursion depth and the `StringBuilder`, excluding the output list itself.

### Common Mistakes and Edge Cases

- ⚠️ **Forgetting the empty-input guard** — produces `[""]` instead of the required `[]`.
- ⚠️ **Using `digits.charAt(index) - '0'` without validating input is actually `2`–`9`** — the problem guarantees this, but it's worth knowing the assumption is there rather than silently relying on it.
- ⚠️ **Rebuilding a new `String` via concatenation inside the loop instead of mutating one shared `StringBuilder`+undo** — not incorrect, but reintroduces the O(n²)-in-a-loop cost pattern from Day 11 across the whole recursion, for no benefit here.
- Edge case: single-digit input (e.g., `"7"`) → 4 single-character results, no recursion depth beyond 1.
- Edge case: digits `7` and `9` specifically (4-letter groups) are the actual worst-case branching factor driving the `O(4ⁿ)` bound — worth naming these two digits specifically if asked to justify the base of the exponent.

> 💡 **Interview Insight:** This problem is frequently used to check whether "backtracking" is understood as a general template or as "the Subsets/Permutations code." Explicitly naming that today's choices come from an *external mapping* rather than *array indices*, while the choose/explore/un-choose skeleton stays identical, is exactly the kind of pattern-versus-instance distinction a tier-1 interview is listening for.

---

# Part 2 — Generate Parentheses (LeetCode 22, Medium) — Pattern: Backtracking

**Statement:** Given `n` pairs of parentheses, generate all combinations of well-formed (validly matched) parentheses strings.

## A third choice model, in the same three-problem arc

Problem 1 (today) chose from a small *external* set at each position. This problem's choices are neither "an array index" nor "an external lookup" — at each step, the only question is **"is `(` legal here? is `)` legal here?"**, answered by two running counters. This is worth naming explicitly as a third distinct flavor of "what determines the legal choices at this recursion depth," alongside forward-index-into-an-array (Days 63–64) and external-lookup-per-position (Problem 1 above).

### Approach — Track open and close counts, prune illegal branches immediately

```java
public static List<String> generateParenthesis(int n) {
    List<String> result = new ArrayList<>();
    backtrack(new StringBuilder(), 0, 0, n, result);
    return result;
}

private static void backtrack(StringBuilder path, int openCount, int closeCount,
                               int n, List<String> result) {
    if (path.length() == 2 * n) {
        result.add(path.toString());
        return;
    }
    if (openCount < n) {
        path.append('(');
        backtrack(path, openCount + 1, closeCount, n, result);
        path.deleteCharAt(path.length() - 1);   // un-choose
    }
    if (closeCount < openCount) {
        path.append(')');
        backtrack(path, openCount, closeCount + 1, n, result);
        path.deleteCharAt(path.length() - 1);   // un-choose
    }
}
```

**Why this needs no separate "is this string valid?" check at the end, the way a naive brute force would:** a brute force that generated *every* string of `2n` parentheses characters and then validated each one afterward would work, but would explore a huge number of dead branches (any string with more `)` than `(` at some prefix is invalid, and there are many such strings among all `2^(2n)` possibilities). This solution instead makes it **structurally impossible** to ever build an invalid prefix in the first place, by only ever offering a legal next character — which is the same "prune the instant a branch can't succeed" idea as yesterday's `remaining < 0` check, just enforced as a build-time constraint instead of a post-hoc check.

## Proving `closeCount < openCount` is sufficient — not just asserting it

**The claim to prove:** if a string is built by only ever appending `(` when `openCount < n`, and only ever appending `)` when `closeCount < openCount`, every fully-built string of length `2n` is well-formed.

**A string of parentheses is well-formed exactly when, scanning left to right, the running count of `(` minus `)` never goes negative, and ends at exactly zero.** (This is the standard, precise definition — worth stating exactly this way if asked, not "the parens match up.")

**Proof, by the invariant the two conditions maintain at every single character appended:**

1. Every time `)` is appended, it's only because `closeCount < openCount` held *before* appending. After appending, the new `closeCount' = closeCount + 1 ≤ openCount`. So immediately after *any* `)` is appended, `openCount − closeCount' ≥ 0` — the running balance never goes negative, by construction, at every single step, not just at the end.
2. The string only stops growing once `path.length() == 2n`. Since every appended character increments exactly one of `openCount` or `closeCount`, and `openCount` can never exceed `n` (guarded directly by `openCount < n`) while `closeCount` can never exceed `openCount` (guarded by step 1) — the only way to reach total length `2n` is `openCount = n` **and** `closeCount = n` simultaneously (if `openCount` were less than `n`, `closeCount ≤ openCount < n` too, and the total length would be under `2n`; contradiction). So the string ends with balance exactly `n − n = 0`.

Both halves of the well-formedness definition — never negative, ends at zero — are guaranteed at construction time, not verified afterward.

### Trace: `n = 2`

```
backtrack(path="", open=0, close=0)
  open<2: append '(' → path="(" , backtrack(open=1, close=0)
    open<2: append '(' → path="((" , backtrack(open=2, close=0)
      open<2? NO (open==n)
      close<open? 0<2 yes: append ')' → path="(()" , backtrack(open=2, close=1)
        open<2? NO
        close<open? 1<2 yes: append ')' → path="(())" , backtrack(open=2, close=2)
          length==4 → ADD "(())"
    close<open? 0<1 yes: append ')' → path="()" , backtrack(open=1, close=1)
      open<2: append '(' → path="()(" , backtrack(open=2, close=1)
        close<open? 1<2 yes: append ')' → path="()()" → ADD "()()"
      close<open? 1<1 NO — nothing further from here
```

**Result: `["(())", "()()"]`** — matches the known Catalan-number count for `n=2` (2 combinations). Notice `)` is only ever offered when strictly fewer closes than opens have happened, and this alone — with no post-hoc validity check anywhere — produced exactly the well-formed set.

### Complexity

**Time: O(4ⁿ/√n)** — this is the `n`-th Catalan number's asymptotic growth rate (the exact count of valid strings for a given `n` *is* the `n`-th Catalan number, `C(n) = (2n)! / ((n+1)! × n!)`, which grows as `4ⁿ/(n^1.5 × √π)`). This bound reflects that the pruning above means the recursion only ever explores paths that lead to valid results — unlike a naive generate-then-filter approach, whose tree would include the much larger set of *all* `2^(2n)` binary strings before filtering. **Space: O(n)** for recursion depth and the `StringBuilder` (excluding the output).

### Common Mistakes and Edge Cases

- ⚠️ **Checking `closeCount < n` instead of `closeCount < openCount`** — this is the single most common bug on this problem. `closeCount < n` only prevents *too many total* closing parens; it does nothing to prevent a closing paren from appearing before its matching open, so it would happily generate `")("` as a "valid" two-character prefix. The comparison must be against `openCount`, the *currently open* count, not the fixed target `n`.
- ⚠️ **Forgetting the undo (`deleteCharAt`) on either branch** — since both `(` and `)` share one `StringBuilder`, skipping the undo on one branch corrupts every subsequent sibling exploration silently (no exception, just wrong output).
- Edge case: `n = 0` → the only well-formed string of length 0 is `""` itself; the base case triggers immediately with an empty path, correctly returning `[""]` (unlike Problem 1's *empty input*, which correctly returns `[]` — worth not conflating these two different "empty" edge cases, since they have different correct answers for different reasons).
- Edge case: `n = 1` → exactly one result, `"()"`.

> ⚠️ **Common Mistake:** treating "prune invalid branches early" as unique to this problem. It's the same idea as yesterday's `remaining < 0` check in Combination Sum — the *specific* condition differs per problem, but "structurally prevent an invalid branch rather than filtering results after the fact" is a transferable backtracking optimization, not a one-off trick.

> 💡 **Interview Insight:** Being asked to justify `closeCount < openCount` rather than just writing it from memory is common precisely because it's easy to memorize without understanding. Walking through the two-part proof above — balance never goes negative, and total length forces both counts to `n` — unprompted, is a strong signal of genuine understanding versus pattern-matched recall.

---

# Part 3 — Feign Clients for Service-to-Service Communication

## Prerequisites, confirmed

REST/HTTP verbs and status codes (Day 34), interfaces (Day 2), and Day 64's Resilience4j — specifically its "annotate something, get real behavior generated underneath via reflection" shape, which Feign shares.

## What Feign actually removes

Before today, "one service calling another over HTTP" would mean hand-building that call: open a connection (or use a library like `RestTemplate`/`WebClient`), construct the URL, set headers, serialize the request body, send it, deserialize the response, handle the failure cases — all boilerplate, repeated at every single call site. Feign replaces all of that with a plain Java **interface**: you declare what the call looks like (method signature, HTTP mapping annotations), and a real, working HTTP client implementing that interface is generated for you at startup.

```java
@FeignClient(name = "product-service", url = "${product.service.url}",
             fallback = ProductServiceClientFallback.class)
public interface ProductServiceClient {

    @GetMapping("/api/products/{id}")
    ProductDto getProduct(@PathVariable("id") String id);
}
```

```java
@Component
public class ProductServiceClientFallback implements ProductServiceClient {
    @Override
    public ProductDto getProduct(String id) {
        return ProductDto.unavailable(id);   // safe default when Product module is unreachable
    }
}
```

```java
@Service
public class OrderService {
    private final ProductServiceClient productServiceClient;

    public OrderService(ProductServiceClient productServiceClient) {
        this.productServiceClient = productServiceClient;   // Spring injects the GENERATED implementation
    }

    public OrderDetails createOrder(String productId, int quantity) {
        ProductDto product = productServiceClient.getProduct(productId);   // a REAL HTTP call, one line
        // ... build and return the order using real product data
    }
}
```

**What's mechanically happening, worth understanding rather than just copying:** at application startup, Spring scans for `@FeignClient`-annotated interfaces and — using the exact same reflection mechanism Day 34 introduced for `@RestController`/`@GetMapping` (a running program inspecting its own annotations *at runtime*, not compile time) — generates a real implementing class on the fly. That generated class translates each interface method call into an actual outbound HTTP request matching the method's mapping annotations, then deserializes the JSON response back into the declared return type. `OrderService` never sees any of this; it just calls `productServiceClient.getProduct(id)` as an ordinary method call on an ordinary-looking interface.

## Why calling the real Product module (not a placeholder) is a meaningfully different exercise

Every prior "call another service" exercise in typical prep material (and, per the plan's own framing, the *original* version of this exact exercise) points a client interface at a mocked or placeholder endpoint — something that always returns a fixed canned response regardless of what's actually asked. Here, because `scalable-ecommerce-platform`'s Product module already exists as a real, independently-running module (Day 62), `ProductServiceClient` is calling **real, working code that can actually fail in real ways** — a genuinely different product not found, a real network timeout if the Product module is down, a real deserialization mismatch if the two modules' DTOs drift out of sync. This is the same "TestContainers vs. WireMock" distinction Day 52 drew (a real dependency for realism vs. a fake one for control) — except here it's the *actual production code path*, not a test, that's benefiting from the realism.

## The fallback's relationship to Day 64's fallback method — same idea, different mechanism

Resilience4j's fallback (Day 64) is a **method** matched by signature via reflection. Feign's fallback (`fallback = ProductServiceClientFallback.class`) is a **whole class implementing the same interface** — every method on `ProductServiceClient` needs a corresponding implementation in the fallback class, not just the one method that happens to fail. Both exist to answer the same question ("what happens when the real call can't complete"), and both are wired in declaratively rather than via a manual try/catch at every call site — but Feign's unit of fallback is the entire client interface, where Resilience4j's is a single guarded method.

### When to reach for Feign vs. a manually-built HTTP client

| | Manual (`RestTemplate`/`WebClient`) | Feign |
|---|---|---|
| URL construction, headers, serialization | Written by hand, every call site | Generated from annotations |
| Adding a new endpoint call | New boilerplate each time | One new interface method |
| Fallback wiring | Manual try/catch per call | One fallback class, declared once |
| Best fit when... | A single one-off external call, or fine-grained control over the HTTP client itself is genuinely needed | Frequent, structured calls between your *own* services — exactly this platform's Order→Product shape |

### Common Mistakes

- ⚠️ **Forgetting the fallback class must implement every method on the interface**, not just the one expected to fail — an incomplete fallback class fails to compile, since it's a real class satisfying a real interface contract.
- ⚠️ **Assuming Feign retries automatically** — by itself it doesn't; retry behavior is a separate, explicitly-configured concern (often paired with Resilience4j's own `@Retry`, distinct from `@CircuitBreaker`), not a Feign default.
- ⚠️ **Conflating the fallback class with the circuit breaker from yesterday** — they can be combined (a Feign client protected by a circuit breaker, whose OPEN state triggers Feign's own fallback), but Feign's fallback alone doesn't track failure *rate* or transition through OPEN/HALF_OPEN — it's a simpler, always-available default.

> 💡 **Interview Insight:** "Feign generates an HTTP client from an interface via reflection at startup" is the specific, defensible mechanism-level answer a tier-1 interviewer wants — "it's an easier way to make HTTP calls" is true but shallow. Naming *how* it works, not just *that* it's convenient, is the difference.

---

## Project Block

**Repository:** `scalable-ecommerce-platform`.

**Task:** define `ProductServiceClient`, a `@FeignClient` interface in the Order module, calling the real Product module's existing product-lookup endpoint. Add a fallback for when the Product module is unreachable.

**Practical guidance:** confirm the Product module's actual endpoint path and DTO shape before writing the Feign interface's mapping annotations — a mismatch here is a runtime failure (a 404 or a deserialization error), not a compile error, the same category of "matches by convention, checked at runtime" gotcha as yesterday's fallback-method signature. Test the fallback path directly by temporarily stopping the Product module and confirming `OrderService` still returns a graceful placeholder rather than propagating an exception.

**Definition of done:** the Feign client compiles and correctly fetches real product data from the running Product module; the fallback returns a placeholder response when the Product module is stopped, verified by actually stopping it and observing the fallback fire — not assumed from the code alone.

---

## Career Block

**LinkedIn Post 14** — "Declarative HTTP clients: how Feign saves you from writing boilerplate REST calls." A concrete, specific post (what Feign actually generates, why the annotation-driven approach reduces repeated boilerplate) reads as far more credible than a generic "learned about Feign today" — the same posting guidance established Day 10.

**Networking:** identify 5 Target companies (Tier C) for early interview practice.

---

# Day 65 — Interview Questions

---

**1. What's genuinely new about today's two backtracking problems compared to Days 61–64's?**

*Answer:* Every prior backtracking problem selected a subset or arrangement of elements from an input array. Today's problems build a new string one character at a time — Letter Combinations chooses from a small external per-digit lookup at each position; Generate Parentheses chooses based on two running counters, not a lookup or an array index at all. The choose/explore/un-choose template is unchanged; what determines the *legal choices* at each step is what's different.

---

**2. Why does Letter Combinations need an explicit empty-input guard?**

*Answer:* Without it, `backtrack("", 0, ...)` immediately satisfies the base case (`index == digits.length()`, `0 == 0`) and adds the empty string to the result, producing `[""]`. LeetCode requires `[]` for empty input — a genuinely different, required output the guard exists to produce.

---

**3. Prove that `closeCount < openCount` alone guarantees every generated parentheses string is well-formed.**

*Answer:* Every `)` appended only happens when `closeCount < openCount` held beforehand, so immediately after appending, `closeCount' ≤ openCount` — the running balance (open minus close) never goes negative at any point during construction. Since the string only stops at length `2n`, and `openCount` is capped at `n` while `closeCount` is capped at `openCount`, reaching length `2n` forces `openCount = closeCount = n` exactly — balance zero at the end. Both halves of well-formedness (never negative, ends at zero) are guaranteed by construction.

---

**4. Why is `closeCount < n` (instead of `closeCount < openCount`) a bug, not just a less-efficient version?**

*Answer:* `closeCount < n` only limits the *total* number of closing parens used; it does nothing to prevent a `)` from being placed before its matching `(`. It would happily generate an invalid prefix like `")("`. The comparison must be against `openCount` — the currently-open count — specifically because well-formedness is about the running balance at every prefix, not just the final totals.

---

**5. Why doesn't Generate Parentheses need a separate validity check on the finished string?**

*Answer:* The two build-time conditions make it structurally impossible to ever construct an invalid prefix — every character appended is proven, at the moment it's appended, to keep the running balance non-negative. Validity is guaranteed by construction, so checking afterward would be redundant.

---

**6. `StringBuilder` + `deleteCharAt` versus building a new `String` via concatenation at each step — both are "correct" backtracking. What's the actual trade-off?**

*Answer:* Both correctly explore the same recursion tree. `StringBuilder` mutates one shared structure and needs an explicit undo, avoiding Day 11's O(n²)-string-concatenation-in-a-loop cost. Building a fresh `String` via concatenation at each call needs no explicit undo (each call's string is independent) but re-pays that concatenation cost repeatedly across the recursion — correct, but strictly more expensive for no benefit here.

---

**7. Mechanically, what does `@FeignClient` actually generate, and when?**

*Answer:* At application startup, Spring scans for `@FeignClient`-annotated interfaces and, via the same runtime-reflection mechanism used for `@RestController`/`@GetMapping`, generates a real implementing class. That generated class translates each interface method call into an actual HTTP request matching the method's mapping annotations, and deserializes the response into the declared return type.

---

**8. Why is calling the platform's real Product module a meaningfully different exercise than calling a mocked stand-in?**

*Answer:* A mocked endpoint always returns a fixed canned response regardless of the actual request. The real Product module can fail in genuinely real ways — an actual not-found product, an actual network timeout if it's down, an actual deserialization mismatch if the two modules' DTOs drift — making the exercise (and its fallback path) test something real rather than a scripted stand-in.

---

**9. How does Feign's fallback differ from Resilience4j's, in scope?**

*Answer:* Resilience4j's fallback is a single method, matched by signature via reflection, guarding one specific method call. Feign's fallback is an entire class implementing the whole client interface — every method the interface declares needs a real implementation in the fallback class, not just the one expected to fail.

---

**10. Does Feign retry failed calls automatically?**

*Answer:* No — retry behavior is a separate, explicitly configured concern (often layered on via Resilience4j's own `@Retry`), not something Feign does by default just by being present.

---

## Daily Deliverable Check

- [ ] Letter Combinations of a Phone Number (LC 17) solved — empty-input edge case handled explicitly.
- [ ] Generate Parentheses (LC 22) solved — the `closeCount < openCount` proof reproducible without notes.
- [ ] Both problems pushed to `dsa-java/backtracking/`.
- [ ] `ProductServiceClient` Feign interface with fallback pushed, calling the real Product module — fallback verified by actually stopping the Product module, not assumed.
- [ ] LinkedIn Post 14 published; 5 Tier C target companies identified.

---

## What Tomorrow Assumes You Already Know Cold

Day 66 assumes today's string-building shape is solid, since Palindrome Partitioning extends it once more (building a *partition*, one substring at a time, rather than one character at a time) — and it assumes the choose/explore/un-choose template is now automatic across all three of this week's distinct choice models (forward-index, external-lookup, counter-based) so a fourth variation doesn't need the base mechanism re-explained. It also assumes today's Feign/reflection-generated-client idea is solid, since tomorrow's Gateway sits in front of the Feign-calling Order module and every other module, and the Gateway theory builds on "requests get routed somewhere" without re-deriving how any individual service-to-service call actually happens underneath.

**Next:** [Day 66 Resource Book](./Day66_Resource_Book.md) — Backtracking on Grids and Palindromes, and the API Gateway.
