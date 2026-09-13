# Day 34 — Linked Lists Begin, and Spring Boot Initialization

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 33 Resource Book](Day33_Resource_Book.md)
**Next ▶:** [Day 35 Resource Book](Day35_Resource_Book.md)
**Companion to:** Day 34 of `Week_05_Revised.md`

---

## Recap

Binary Search closed yesterday. Today opens a new data structure — but it's worth being precise about what's actually new: a linked list's `next` field is a heap reference, mechanically identical to every object reference Day 9's JVM Memory Model already covered. Nothing new is happening in memory; what's new is *using* that mechanism deliberately, node by node, to build a chain. The trade-offs against `ArrayList` were even previewed already, back on Day 12's Collections Internals ("contiguous array vs. linked nodes vs. circular array, cache locality") — today cashes that preview in with real depth.

The theory switches to something genuinely new: REST APIs and Spring Boot, the first backend-framework material in this series, and the first use of `todo-api`, a new repository that stays live for the rest of the plan.

**⚠️ Technical accuracy note on today's totals:** `Week_05_Revised.md`'s own header states Linked Lists "grows from 9 problems to 12," but `Week_06_Revised.md`'s header says it "closes at 11" — and today's own Day 35 Weekly Scorecard (in the plan, ahead) independently says "expanded 11-problem set." Counting directly: 4 problems land in Week 5 (today and tomorrow) and 7 are required in Week 6, giving 11 total — matching two of the three sources and the direct arithmetic. This book uses **11** throughout as the correct total; the "12" in Week 5's header appears to be a stale figure the rest of the plan's own text doesn't agree with.

---

## Learning Objectives

By the end of today, without notes:

1. Explain a linked list node's `next` field as a heap reference — the same mechanism as any object reference — and explain why that's what makes O(1) insertion/deletion possible at a known node.
2. State the ArrayList-vs-LinkedList trade-off precisely, including the "still O(n) if you have to search first" nuance that a shallow explanation glosses over.
3. Reverse a singly linked list iteratively (three-pointer walk) and recursively, and state exactly why the recursive version's space cost is O(n), not O(1).
4. Find a linked list's middle in one pass with fast/slow pointers, and explain why that's the same Two Pointers variant from Week 1, applied to a structure with no index arithmetic.
5. Explain HTTP verbs, status codes, and REST's statelessness principle, and translate that into a working Spring Boot `@RestController` endpoint.

---

## Concept Dependency Map

```
Day 2: classes, objects, references
Day 9: JVM Memory Model — heap references, mechanism-level
Day 3 / Day 12: ArrayList amortized doubling; Collections Internals
                (ArrayList vs. LinkedList vs. ArrayDeque, PREVIEWED, not yet proven)
Day 6/7: Two Pointers — same-direction (fast/slow) variant
Day 8: Recursion — call stack, base/recursive case, StackOverflowError
        │
        ▼
Today: Linked Lists (NEW SHAPE, no new memory mechanism)
        ├─ Problem 1: Reverse Linked List (LC 206) — iterative AND recursive
        └─ Problem 2: Middle of the Linked List (LC 876) — fast/slow, reused directly from Week 1

Day 16: Generics (<T>, bounded type parameters)
        │
        ▼
Today: REST APIs and Spring Boot
        ├─ HTTP verbs/status codes, REST's statelessness
        ├─ NEW: reflection (a program inspecting its own classes/annotations at RUNTIME)
        ├─ @RestController / @GetMapping — Spring's routing mechanism
        └─ ResponseWrapper<T> — a plain, unbounded generic class (Day 16's simplest case)
```

---

# Part 1 — Linked Lists

### Prerequisites (confirmed)

- Classes, objects, and object references — Day 2.
- Heap references, mechanism-level — Day 9 (today extends this directly, introduces no new memory concept).
- `ArrayList`'s amortized-O(1) append and array memory-address indexing — Day 3, Day 2.
- Collections Internals — Day 12 (ArrayList vs. LinkedList vs. ArrayDeque previewed at the conceptual level; today proves it).
- Fast/slow pointers — Day 6/7's Two Pointers.

**Concept Card — Linked Lists**
- **What:** a chain of nodes, each holding a value and a reference to the next node. No contiguous-memory requirement, unlike arrays.
- **Why:** O(1) insertion/deletion at a known node, at the cost of O(n) access to an arbitrary position.
- **Where:** the literal implementation behind `LinkedList`, and the pointer-manipulation mental model reused constantly in trees and graphs.
- **Interview signal:** "reverse," "merge," "detect a cycle," "find the middle" — usually solvable with a small, fixed number of pointers and O(1) extra space.
- **Prerequisites:** OOP ✅ — a `ListNode` is just a class with a value field and a reference field.

## What a Linked List Actually Is

```java
public class ListNode {
    int val;
    ListNode next;

    ListNode(int val) {
        this.val = val;
        this.next = null;
    }
}
```

That's the entire structure. `next` is a reference — the exact same mechanism Day 9 already established for every object reference — pointing to another `ListNode` on the heap, or `null` to mark the end of the chain.

```
head
 │
 ▼
┌─────┬──────┐      ┌─────┬──────┐      ┌─────┬──────┐
│ val:1│ next │ ───▶ │ val:2│ next │ ───▶ │ val:3│ next │ ───▶ null
└─────┴──────┘      └─────┴──────┘      └─────┴──────┘
  (heap)               (heap)               (heap)
```

Each box is a separate heap allocation — nothing requires them to be adjacent in memory, unlike an array. Traversal is a straight walk:

```java
ListNode current = head;
while (current != null) {
    // process current.val
    current = current.next;
}
```

## Why the Trade-offs Are What They Are — Mechanism, Not Just Labels

**🔑 Key Takeaway — this is Day 12's preview, proven:** an array (and `ArrayList`, built on one) gives O(1) access to `arr[i]` because of the memory-address formula from Day 2 — `baseAddress + i × elementSize` — a direct calculation, no traversal needed. A linked list has **no such formula**, because nodes are scattered wherever the heap happened to allocate them; there is no relationship between a node's position in the chain and its memory address. Reaching node `i` requires walking `i` heap-reference hops from the head — O(n) in the worst case, with no way around it.

The reverse trade holds for insertion: inserting into the *middle* of an `ArrayList` costs O(n), because every element after the insertion point has to physically shift over one slot (Day 3's amortized O(1) only covers **appending at the end** — inserting in the middle is a different operation with a different cost, worth being precise about rather than treating "ArrayList insert" as one uniform O(1) claim). Inserting into a linked list at a **known node** costs O(1) — it's purely a matter of rewiring two `next` references, no shifting, regardless of how long the list is.

| | `ArrayList` | `LinkedList` (singly linked) |
|---|---|---|
| Memory layout | contiguous | scattered, one allocation per node |
| Random access `[i]` | O(1) — direct address math | O(n) — must walk from head |
| Insert/delete at end | O(1) amortized (Day 3) | O(1) if you hold the tail reference |
| Insert/delete at a **known** node | O(n) — must shift | O(1) — just rewire references |
| Cache locality | good — adjacent memory, CPU prefetch-friendly (Day 12) | poor — nodes can be anywhere on the heap |
| Extra memory per element | none beyond the value | one reference per node beyond the value |

**⚠️ Common Mistake — the nuance a shallow explanation skips:** "O(1) insertion" for a linked list quietly assumes you *already hold a reference* to the node you're inserting at. If you don't — if you first have to **find** the right spot — that search costs O(n) on a singly linked list (no random access means no way to binary-search your way there), and the O(1) insertion is dwarfed by the O(n) search that had to happen first. "Linked lists are faster for insertion" is only true relative to the *insertion itself*, not the whole operation of locating-then-inserting.

**💡 Interview Insight — a real, likely follow-up:** *"Can you delete a node given only a reference to that node itself — no head, no reference to its predecessor?"* For a singly linked list, deleting node `X` normally requires rewiring `X`'s **predecessor's** `next` field — which requires having a reference to the predecessor, not just `X`. The classic workaround, when you truly only have `X` and `X` isn't the last node: copy the *next* node's value into `X`, then delete the next node instead (`X.val = X.next.val; X.next = X.next.next;`) — from the outside, this looks identical to deleting `X`, even though what actually got unlinked from the heap was `X`'s former successor. This trick fails if `X` is the last node, since there's no successor to borrow from — worth naming that limitation unprompted if this comes up.

---

## Problem 1: Reverse Linked List (LeetCode 206, Easy) — Pattern: Iterative Pointer Reversal

**Statement:** Given the head of a singly linked list, reverse it and return the new head.

### Approach 1 — Iterative: three-pointer walk

```java
public static ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
    while (curr != null) {
        ListNode next = curr.next;   // save BEFORE overwriting — see the mistake below
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

**Mechanism:** walk the list once, and at each node, flip its `next` reference to point *backward* instead of forward. Three pointers are needed simultaneously: `prev` (the new predecessor to link to), `curr` (the node being flipped), and a temporary `next` (where to continue the walk, since `curr.next` is about to be overwritten and would otherwise be lost).

**⚠️ Common Mistake — the classic list-losing bug, shown concretely:** writing `curr.next = prev;` *before* saving the original `curr.next` anywhere. Trace it on `1 → 2 → 3`: `curr.next = prev` sets `node1.next = null` — but nothing anywhere still holds a reference to `node2` or `node3` at that point (the only path to them was through `node1.next`, which was just overwritten). They're unreachable, permanently, and the "reversed" result is just a single node, `1 → null` — the rest of the list is silently gone. Saving `next` first, before any mutation, is not optional style — it's the only thing preventing this.

**Worked trace:** `1 → 2 → 3 → null`.

| prev | curr | next (saved) | after `curr.next = prev` |
|---|---|---|---|
| null | 1 | 2 | `1 → null` |
| 1 | 2 | 3 | `2 → 1` |
| 2 | 3 | null | `3 → 2` |

Loop ends when `curr` becomes `null`; `prev` is now `3`, the new head. Full result: `3 → 2 → 1 → null`. Correct.

**Complexity:** Time O(n), Space O(1).

### Approach 2 — Recursive (meaningfully distinct, citing Day 8)

```java
public static ListNode reverseListRecursive(ListNode head) {
    if (head == null || head.next == null) {
        return head;   // base case: an empty list or single node is already "reversed"
    }
    ListNode newHead = reverseListRecursive(head.next);
    head.next.next = head;   // the node after head now points back to head
    head.next = null;        // head becomes the new tail
    return newHead;
}
```

**Mechanism, using Day 8's recursion model directly:** the recursive call fully reverses everything *after* `head` first, and returns the new head of that reversed sublist — which, crucially, is also the head of the *entire* reversed list, and doesn't change again as the recursion unwinds. On the way back up each stack frame, `head.next` still points to whatever immediately followed `head` in the *original* list — but that node has already been fully processed as the current tail of the reversed portion. `head.next.next = head` attaches `head` as that tail's new successor; `head.next = null` makes `head` the new tail.

**Worked trace:** `1 → 2 → 3 → null`. `reverseListRecursive(1)` calls `reverseListRecursive(2)`, which calls `reverseListRecursive(3)` — base case (`3.next == null`), returns `3`. Back at the `head=2` frame: `newHead=3`; `head.next.next = head` → `node3.next = node2` (`3→2`); `head.next = null` → `node2.next = null`. Back at the `head=1` frame: `newHead=3` (unchanged, propagated up); `head.next.next = head` → `node2.next = node1` (`2→1`); `head.next = null` → `node1.next = null`. Final structure: `3 → 2 → 1 → null`. Same result as the iterative version.

**Complexity:** Time O(n). **Space O(n)** — not O(1) — because the call stack holds one frame per node until the base case is reached (Day 8's call-stack-depth-equals-space-cost lesson, made concrete on a real structure).

**⚠️ Trade-off, worth stating unprompted:** a list of, say, 100,000 nodes would risk `StackOverflowError` (Day 8) on the recursive version, despite it being perfectly correct — this is exactly why the **iterative** version, not just "a working solution," is the one to reach for by default, even though the recursive version often reads more elegantly.

**Edge cases:**
- Empty list (`head == null`) — iterative: loop never runs, returns `prev = null`. Recursive: hits the base case immediately, returns `null`. Both correct.
- Single node — iterative: one iteration, node's `next` becomes `null` (unchanged, since it already was), returns that same node. Recursive: hits the base case (`head.next == null`) immediately, returns `head` unchanged.

**💡 Interview Insight:** default to the iterative version; have the recursive one ready as a follow-up, and lead with its space cost unprompted the moment it comes up — naming the O(n) call-stack cost before being asked is a stronger signal than producing correct code that "happens" to be O(n) space without registering it.

---

## Problem 2: Middle of the Linked List (LeetCode 876, Easy) — Pattern: Fast/Slow Pointers

**Statement:** Return the middle node. If there are two middle nodes (even length), return the **second** one.

### Approach 1 — Two-pass: count, then walk halfway

```java
public static ListNode middleNodeTwoPass(ListNode head) {
    int count = 0;
    ListNode node = head;
    while (node != null) {
        count++;
        node = node.next;
    }
    ListNode curr = head;
    for (int i = 0; i < count / 2; i++) {
        curr = curr.next;
    }
    return curr;
}
```

Time O(n) — two full passes, still linear overall. Space O(1).

### Approach 2 — Optimized: fast/slow pointers, one pass

```java
public static ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

**🔗 This is not a new technique — it's Week 1's fast/slow Two Pointers, applied where index arithmetic isn't available:** Day 6/7 established same-direction fast/slow as one of Two Pointers' seven named variants, used there on arrays. A linked list has no `arr[i]` to jump to directly, which is exactly why the "count then jump to n/2" approach feels natural on an array but clumsy here — fast/slow sidesteps needing an index at all, advancing `fast` twice for every one of `slow`'s steps, so when `fast` runs out, `slow` has covered exactly half the distance.

**⚠️ Honest complexity note — resist the urge to oversell this:** both approaches here are O(n) time, O(1) space — fast/slow is **not** asymptotically better, only a single pass instead of two (a real but modest win) and, more importantly, the direct foundation for tomorrow's Palindrome Linked List and later cycle detection. Claiming a complexity-class win that isn't actually there would be exactly the kind of overstated claim worth avoiding in an interview.

**Worked trace, even length:** `1 → 2 → 3 → 4 → null`. `slow=1, fast=1`. Step 1: `fast(1).next=2` exists → `slow=2, fast=1.next.next=3`. Step 2: `fast(3).next=4` exists → `slow=3, fast=3.next.next=null`. Step 3: `fast == null` → loop ends. Returns `slow = 3` — correctly the **second** of the two middles (`2` and `3`).

**Worked trace, odd length:** `1 → 2 → 3 → 4 → 5 → null`. `slow=1, fast=1`. Step 1: `slow=2, fast=3`. Step 2: `slow=3, fast=5`. Step 3: `fast(5).next == null` → loop condition false, loop ends. Returns `slow = 3` — the single unambiguous middle.

**Complexity:** Time O(n), Space O(1).

**Edge cases:**
- Single node — `fast.next` is `null` immediately, loop never runs, returns `head` itself.
- Two nodes — one iteration, returns the second node (matches the "second middle on a tie" rule).

**⚠️ Common Mistake:** checking the loop condition in a different order (e.g., `fast.next != null && fast.next.next != null`) looks superficially similar but returns the *first* of the two middles on even-length lists instead of the second — the exact condition and the exact order of advancing `slow` then `fast` both matter, not just "having roughly the right idea."

**💡 Interview Insight:** name the Week 1 connection out loud — "this is the same fast/slow shape as [specific Week 1 problem], just without array indices to fall back on" — before writing code. It signals pattern recognition across data structures, not just within one.

---

# Part 2 — REST APIs and Spring Boot Initialization

### Prerequisites (confirmed)

- Generics, unbounded and bounded — Day 16 (`ResponseWrapper<T>` below is the simplest possible case: no bound needed).
- Classes, annotations as used so far (`@Override`) — Day 2, Day 5.

## HTTP and REST, From Zero

An HTTP request has a **method** (verb) and a **path**. The four verbs that map onto CRUD:

| Verb | CRUD operation | Typical use |
|---|---|---|
| `GET` | Read | Fetch a resource, no side effects |
| `POST` | Create | Create a new resource |
| `PUT` | Update | Replace a resource entirely |
| `DELETE` | Delete | Remove a resource |

A response carries a **status code**, grouped by leading digit: `2xx` success (`200 OK`, `201 Created`, `204 No Content`), `4xx` client error (`400 Bad Request` — the client sent malformed data; `404 Not Found` — the resource genuinely doesn't exist; worth knowing precisely, `422 Unprocessable Entity` — the data is well-formed but semantically invalid, e.g. a negative price), `5xx` server error (`500 Internal Server Error`).

**REST** (**Re**presentational **S**tate **T**ransfer) is a set of conventions for designing this kind of API: resources are identified by **URLs as nouns**, not verbs (`/tasks/42`, not `/getTask?id=42`), and the verb + status code together communicate what actually happened, rather than encoding the action into the URL itself.

**🔑 Key Takeaway — statelessness is a real trade-off, not an arbitrary rule:** each request must carry everything the server needs to handle it (identity, auth, any relevant context) — the server keeps no memory of a specific client between requests. This is what makes horizontal scaling straightforward: since no server instance has to "remember" a particular client, *any* instance behind a load balancer can handle *any* request, with no session state to keep synchronized across machines. The cost is that clients resend identifying information on every single request, rather than relying on a server-remembered session.

## Spring Boot: What the Annotations Actually Do

Spring Boot is a framework built on top of the wider Spring Framework, with an **embedded server** (Tomcat, by default) — there's no separate server to install and configure; running the application *is* running the server.

```java
@RestController
public class HealthController {

    @GetMapping("/health")
    public ResponseWrapper<String> health() {
        return new ResponseWrapper<>(true, "OK", null);
    }
}
```

**⚠️ Mechanism worth being precise about — this is a different kind of annotation than `@Override`:** `@Override` is checked by the **compiler**, at compile time — it has no effect at runtime at all, purely a compile-time correctness check. `@RestController` and `@GetMapping` are checked by **Spring itself, at startup**, using **reflection** — a running Java program's ability to inspect its own classes, methods, and annotations *at runtime*, something not touched on anywhere in this series until now. When the application starts, Spring scans the codebase, finds classes annotated `@RestController`, and finds methods annotated `@GetMapping`/`@PostMapping`/etc. inside them, then builds an internal routing table mapping (verb, path) pairs to the exact method to invoke. `@RestController` also tells Spring to automatically serialize whatever a method returns into JSON (via the Jackson library, working underneath Spring — worth knowing the name exists, not worth going deeper into today).

## `ResponseWrapper<T>` — Day 16's Generics, Applied

```java
public class ResponseWrapper<T> {
    private final boolean success;
    private final T data;
    private final String message;

    public ResponseWrapper(boolean success, T data, String message) {
        this.success = success;
        this.data = data;
        this.message = message;
    }

    public boolean isSuccess() { return success; }
    public T getData() { return data; }
    public String getMessage() { return message; }
}
```

This is Day 16's plainest generics case: `T` is completely unbounded — `ResponseWrapper` never calls any method *on* `T`, it just holds and returns whatever it's given, so there's no reason to constrain it the way a bounded type parameter would. Wrapping every endpoint's response in the same `{success, data, message}` shape gives every API consumer one consistent structure to parse, regardless of which endpoint they called.

**⚠️ Common Mistake, flagged now for Day 36:** today's `/health` endpoint takes no request body, so it doesn't need `@RequestBody` — but the moment an endpoint needs to *receive* JSON (a `POST /tasks` accepting a new task's fields, coming Week 6), the method parameter needs `@RequestBody` to tell Spring to deserialize the incoming JSON into a Java object automatically. Forgetting it is a very common first mistake, worth having in mind before it's actually needed.

---

## Project Block Guide (1.5 hrs)

**Repository:** `todo-api` (new).

**Step 1 — Initialize:** use Spring Initializr (start.spring.io) with dependencies: **Web**, **JPA**, **PostgreSQL**, **Validation**. Download and extract the generated project.

**Step 2 — Build `ResponseWrapper<T>`** exactly as above, in its own file.

**Step 3 — Build `HealthController`** exactly as above, returning `ResponseWrapper<String>` with `success=true`, `data="OK"`.

**Step 4 — Run it:** start the application; it should come up on port `8080` by default.

**Definition of done:** the app starts cleanly with no errors, and `curl localhost:8080/health` returns HTTP `200 OK` with a JSON body shaped like `{"success":true,"data":"OK","message":null}`.

---

## Career Block Guide (1 hr)

**LinkedIn Post 8 — "Today I shipped my first Spring Boot endpoint."** A short, specific post: what `/health` does, why a health-check endpoint is a standard first thing to build (it's the simplest possible proof the service is actually running, and often the first thing infrastructure tooling checks), and one genuine thing that felt new (reflection-based routing is a good candidate, since it's a real mechanism most beginners gloss over).

**Networking:** the plan repeats "identify 3 companies known for strong backend engineering blogs" from Day 30 verbatim here — worth noticing rather than mechanically redoing the same search. `todo-api` is live *today*, which is the moment that list actually becomes useful — a better use of today's networking time is pulling up the list from Day 30 and actually reading one post from each, rather than re-identifying the same three companies a second time.

---

## Day 34 — Interview Questions

**Q1. Mechanically, what is a linked list node's `next` field?** A heap reference — the same mechanism Day 9 already covered for every object reference — pointing to another node, or `null` at the end of the chain. Nothing new is happening in memory; it's the same reference mechanism, used deliberately to chain nodes.

**Q2. Why is array access O(1) but linked list access O(n)?** Array access uses direct address arithmetic (`base + i × elementSize`), needing no traversal. A linked list has no such formula — nodes are scattered wherever the heap allocated them, so reaching node `i` requires walking `i` reference hops from the head.

**Q3. Is "linked list insertion is O(1)" always true?** Only if you already hold a reference to the node you're inserting at. If you first have to *find* that node, the search costs O(n) on a singly linked list (no random access to speed it up), which dominates the O(1) insertion that follows it.

**Q4. In the iterative reversal, why must `curr.next` be saved before it's overwritten?** Because `curr.next` is the only remaining path to the rest of the original list — overwriting it first (`curr.next = prev`) before saving it anywhere permanently disconnects everything after `curr`, with nothing left pointing to it.

**Q5. What's the space complexity of the recursive reversal, and why?** O(n) — one stack frame is held per node until the base case is reached, directly following Day 8's call-stack-depth-equals-space-cost lesson; it's not O(1) just because no explicit data structure is declared.

**Q6. Why does fast/slow only need one pass, while counting-then-jumping needs two?** Fast/slow discovers "halfway" *while* traversing, by advancing two references at different speeds — no upfront count is needed, since the moment `fast` runs out, `slow` has necessarily covered exactly half the distance.

**Q7. Are the two approaches to Middle of the Linked List different in Big-O?** No — both are O(n) time, O(1) space. Fast/slow's advantage is a single pass instead of two, and being the direct foundation for later problems (Palindrome Linked List, cycle detection) — not an asymptotic win.

**Q8. What's the mechanical difference between `@Override` and `@RestController`?** `@Override` is a compile-time-only check with zero runtime effect. `@RestController` (and Spring's other annotations) are read at application startup via reflection — the running program inspecting its own classes and methods — and used to build a routing table connecting HTTP requests to the right methods.

**Q9. Why does REST favor statelessness, and what does it cost?** Statelessness lets any server instance handle any request, since no instance needs to remember a specific client — the basis for straightforward horizontal scaling. The cost is that clients must resend identifying context on every request, rather than relying on a server-side session.

**Q10. What does `ResponseWrapper<T>`'s unbounded `<T>` mean, concretely?** That the class never calls any method specific to `T` — it only ever stores and returns whatever `T` is — so there's no reason to constrain it with a bound; it's Day 16's simplest generics case in practice.

**Q11. Why 404 versus 400 versus 422 — what's the precise distinction?** `400` means the request itself is malformed (bad syntax, missing required fields). `404` means the request is well-formed but the specific resource doesn't exist. `422` means the request is well-formed and the resource concept is valid, but the actual data fails a semantic rule (e.g., a negative price).

---

## Daily Deliverable Check

- [ ] Reverse Linked List and Middle of the Linked List solved (both iterative and recursive for the former), pushed to `dsa-java/linked-lists/`.
- [ ] Can state the ArrayList-vs-LinkedList trade-off precisely, including the "still O(n) if you have to search first" nuance.
- [ ] `todo-api` initialized via Spring Initializr; `/health` live, returning `200 OK` via `ResponseWrapper<String>`.
- [ ] LinkedIn Post 8 published.

---

## What Tomorrow Assumes You Already Know Cold

Day 35 assumes today's fast/slow (Middle of the Linked List) and iterative reversal are both fully reflexive, since tomorrow's Palindrome Linked List combines both techniques directly, back to back, with no re-explanation of either mechanism — only the *combination* is new. It also assumes the trade-off table above (ArrayList vs. LinkedList) is solid enough to defend without notes, since it won't be re-derived again this series, only cited.
