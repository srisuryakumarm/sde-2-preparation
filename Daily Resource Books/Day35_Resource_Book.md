# Day 35 (Sunday) — Linked Lists Continue, and Week 5 Consolidation

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 34 Resource Book](Day34_Resource_Book.md)
**Next ▶:** [Day 36 Resource Book](Day36_Resource_Book.md)
**Companion to:** Day 35 of `Week_05_Revised.md`

---

## Recap

Yesterday delivered two separate linked-list techniques — iterative reversal, and fast/slow middle-finding — largely independently. Today's first problem combines them directly, back to back, with no re-explanation of either mechanism, only the combination itself. The second problem introduces one small, genuinely new technique (a dummy head) that then gets reused immediately in both of today's extra-practice problems. This is also the last day of the week: a full consolidation closes out the file below, tallying Binary Search's actual close-out against what was planned.

---

## Self-Check (10 min)

Before starting today's new material: pick one Binary Search problem from earlier this week and solve it cold, closed-notes. For real signal, pick one from each framing if there's time — one "on the input" (rotated search or boundary search) and one "on the answer" (Koko or Ship Capacity) — since Day 33 flagged that the second framing is the one that's actually harder to recognize under pressure, not just harder to implement once recognized.

---

## Learning Objectives

By the end of today, without notes:

1. Combine fast/slow middle-finding and iterative reversal to check a linked list for a palindrome in O(1) space, and explain precisely why the middle node ends up compared to itself without needing special-case handling.
2. Explain the dummy-head technique and why it removes the need to special-case "is this the first node of the result."
3. Reverse only a sub-range of a linked list in one pass, reusing the dummy-head technique to avoid special-casing a reversal that starts at the head.
4. Merge k sorted lists via two genuinely different O(N log k) approaches, and state the real trade-off between them.

---

## Concept Dependency Map

```
Day 34: fast/slow middle-finding (LC 876)
Day 34: iterative reversal (LC 206)
        │
        ▼
Today, Problem 3: Palindrome Linked List (LC 234)
   fast/slow + reversal, COMBINED — no new mechanism, only a new combination

Day 17: PriorityQueue (binary heap)
Day 29: anonymous inner classes (needed for a custom Comparator)
Day 8: recursion, call-stack-depth-as-space
        │
        ▼
Today, Problem 4: Merge Two Sorted Lists (LC 21)
   NEW: dummy head — avoids special-casing an empty result list
        │
        ├──▶ Extra Practice 1: Reverse Linked List II (LC 92) —
        │        reuses today's dummy head, generalizes Day 34's full reversal
        │
        └──▶ Extra Practice 2: Merge k Sorted Lists (LC 23) —
                 reuses today's exact merge routine (divide & conquer) AND
                 Day 17's PriorityQueue + Day 29's anonymous Comparator (heap)
```

---

# Part 1 — Linked Lists Continue

### Prerequisites (confirmed)

- Fast/slow pointers, iterative reversal — Day 34.

## Problem 3: Palindrome Linked List (LeetCode 234, Easy) — Pattern: Fast/Slow + Reversal, Combined

**Statement:** Given the head of a singly linked list, determine whether it reads the same forward and backward.

### Approach 1 — Brute force: copy to an array, then two-pointer check

```java
public static boolean isPalindromeBruteForce(ListNode head) {
    List<Integer> values = new ArrayList<>();
    for (ListNode node = head; node != null; node = node.next) {
        values.add(node.val);
    }
    int left = 0, right = values.size() - 1;
    while (left < right) {
        if (!values.get(left).equals(values.get(right))) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

Time O(n), Space O(n) — the array copy is exactly what the optimized version below avoids.

### Approach 2 — Optimized: find the middle, reverse the second half, compare

```java
public static boolean isPalindrome(ListNode head) {
    if (head == null || head.next == null) {
        return true;
    }

    // Step 1: find the middle — Day 34's fast/slow, unchanged
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // Step 2: reverse the second half — Day 34's iterative reversal, unchanged, starting at slow
    ListNode secondHalfHead = reverseList(slow);

    // Step 3: compare the first half against the reversed second half
    ListNode firstHalfPtr = head;
    ListNode secondHalfPtr = secondHalfHead;
    boolean result = true;
    while (secondHalfPtr != null) {
        if (firstHalfPtr.val != secondHalfPtr.val) {
            result = false;
            break;
        }
        firstHalfPtr = firstHalfPtr.next;
        secondHalfPtr = secondHalfPtr.next;
    }

    return result;
}
```

**🔑 Key Takeaway — this is genuinely just yesterday's two techniques, run in sequence:** neither `reverseList` nor the fast/slow walk needed to change at all to be reused here. The only new work is deciding *what order* to run them in and *what to do* with the two resulting pieces.

**⚠️ Worth reasoning through carefully — what happens at the middle node on odd-length lists:** reversing *starting at* `slow` mutates the original list — specifically, `slow`'s own `next` field becomes `null` (it becomes the new tail of the reversed second half). This has a clean side effect: on an odd-length list, the middle node ends up being the **last node of both halves simultaneously** — the traversal from `head` reaches it and then hits `null` (since its `next` was zeroed), and the traversal from `secondHalfHead` also reaches it last (since it's the final node of the reversed piece). It gets compared to itself, trivially always equal, and both pointers become `null` on the same iteration — no special-casing needed for odd length at all; it falls out of the mechanism for free.

**Worked trace, odd length:** `1 → 2 → 3 → 2 → 1` (nodes `A,B,C,D,E`). `slow` lands on `C` (verified by Day 34's fast/slow trace pattern for length 5). `reverseList(C)` reverses `C→D→E→null` into `E→D→C→null`; critically, `C.next` becomes `null`, and `B.next` (untouched by the reversal) still points to `C`. Comparing `A(1)` vs `E(1)` → equal; `B(2)` vs `D(2)` → equal; `C(3)` vs `C(3)` → equal (same node); both pointers now `null` — loop ends, `true`.

**Worked trace, a non-palindrome, even length:** `1 → 2 → 3 → 4`. `slow` lands on the third node (`val=3`, per Day 34's even-length trace pattern). Reversing from there turns `3→4→null` into `4→3→null`. Comparing `head(1)` against the new second-half head `(4)`: `1 != 4` immediately → `false`. Correct.

**Complexity:** Time O(n) — fast/slow is O(n), the reversal touches at most half the nodes, the comparison touches at most half — all linear, summing to O(n). Space **O(1)** — no extra data structure, only pointer rewiring, a genuine improvement over the brute force's O(n) array.

**Edge cases:**
- Single node or empty list — caught by the early return, trivially a palindrome.
- All values identical — works the same as any other case; no special path needed.

**⚠️ Common Mistake / worth flagging in an interview even though it's not asked:** this approach **mutates the input list** (the second half stays reversed after the function returns, unless it's explicitly reversed back). The problem statement doesn't require preserving the input, but naming this trade-off unprompted — and offering to restore the list with one more reversal pass if the interviewer cares — reads as more careful engineering than silently leaving a side effect unmentioned.

**💡 Interview Insight:** name this explicitly as "three techniques you already have, combined" — the exact framing Week 6's Reorder List (fast/slow + reversal + merge) will reuse again, one level more complex. Recognizing that pattern-combination is itself a recurring move, not a one-off trick, is worth stating out loud.

---

## Problem 4: Merge Two Sorted Lists (LeetCode 21, Easy) — Pattern: Two Pointers with a Dummy Head (NEW)

**Statement:** Merge two sorted linked lists into one sorted list by splicing their nodes together, and return the head.

### Approach 1 — Brute force: extract, sort, rebuild

Copy every value from both lists into one array, sort it, then build an entirely new list from the sorted values. Time O((m+n) log(m+n)) for the sort, Space O(m+n) for new nodes — and it wastes the fact that both inputs are *already* individually sorted.

### Approach 2 — Optimized: splice existing nodes with a dummy head

```java
public static ListNode mergeTwoLists(ListNode list1, ListNode list2) {
    ListNode dummy = new ListNode(-1);
    ListNode tail = dummy;

    while (list1 != null && list2 != null) {
        if (list1.val <= list2.val) {
            tail.next = list1;
            list1 = list1.next;
        } else {
            tail.next = list2;
            list2 = list2.next;
        }
        tail = tail.next;
    }

    tail.next = (list1 != null) ? list1 : list2;   // attach whichever list has leftovers
    return dummy.next;
}
```

**🔑 New Syntax/Technique: the dummy head.** Without it, building a result list from scratch needs an if/else on every insertion: "if the result is still empty, set its head; otherwise, append to the current tail" — exactly the kind of edge-case-prone branching a dummy node eliminates entirely. `dummy` is a throwaway node that's never part of the real answer — it exists purely so `tail` always has *something* valid to hang the first real node off of. The real answer starts at `dummy.next`, which is discarded along with `dummy` itself once the function returns a reference to what comes after it.

**Why splice instead of copying values into new nodes:** the problem only asks to *reorder* the two lists into one, not to duplicate their data — reusing the existing nodes keeps this O(1) extra space (beyond the single dummy), and is the more idiomatic reading of "merge."

**Worked trace:** `list1 = 1→3→5`, `list2 = 2→4→6`. `1<=2`→attach `1`; `3<=2`? no →attach `2`; `3<=4`→attach `3`; `5<=4`? no→attach `4`; `5<=6`→attach `5`; now `list1` is exhausted (`null`) → loop ends → `tail.next = list2` (which is `6`). Result: `1→2→3→4→5→6`. Correct.

**Complexity:** Time O(m+n) — every node visited exactly once total. Space O(1), excluding the single dummy node and the output itself.

**Edge cases:**
- One list empty from the start — the `while` loop never runs (its guard fails immediately), and `tail.next` is set directly to whichever list is non-empty — correctly returns that list untouched.
- Both empty — `dummy.next` stays `null` throughout, correctly returns `null`.

**⚠️ Common Mistake:** forgetting to advance `tail` itself (only advancing `list1`/`list2`) — every subsequent `tail.next =` assignment would silently overwrite the same position instead of extending the chain, corrupting the result.

**⚠️ Worth being precise about, not just "using `<=`":** using `<` instead of `<=` doesn't break overall sortedness of the output either way — it only changes *which* list's node comes first when values tie. The problem doesn't require that kind of stability, but knowing this is a stability choice rather than a correctness requirement is worth stating precisely rather than treating `<=` as an arbitrary default.

**💡 Interview Insight:** name the dummy-head pattern immediately and explicitly — it's a small, specific, widely reusable trick for avoiding empty-result special-casing in list-building problems, not something to silently use without naming.

---

# Part 2 — Extra Practice (Extension): Two More Reps

Linked Lists opened yesterday and continues into Week 6 — following this series' established discipline (no extra reps land on a pattern's opening day; see Day 14, Day 21, Day 23, Day 28 in the curriculum map), today is the first day extras are added, reinforcing what's already been taught rather than the opening material itself. Both are clearly beyond today's strict 2-hour DSA budget.

---

## Extra Practice 1 (Extension): Reverse Linked List II (LeetCode 92, Medium) — Pattern: Bounded Reversal, Reusing Today's Dummy Head

**Statement:** Reverse only the nodes from position `left` to position `right` (1-indexed, inclusive), leaving the rest of the list unchanged, in one pass.

### Approach — dummy head + repeated "move next node to the front of the sub-range"

```java
public static ListNode reverseBetween(ListNode head, int left, int right) {
    ListNode dummy = new ListNode(-1);
    dummy.next = head;
    ListNode pre = dummy;

    for (int i = 0; i < left - 1; i++) {
        pre = pre.next;             // walk to the node just before position `left`
    }

    ListNode curr = pre.next;        // curr stays fixed — it becomes the TAIL of the reversed range
    for (int i = 0; i < right - left; i++) {
        ListNode nodeToMove = curr.next;
        curr.next = nodeToMove.next;
        nodeToMove.next = pre.next;
        pre.next = nodeToMove;
    }

    return dummy.next;
}
```

**🔗 Direct reuse of today's dummy head, for exactly the reason it exists:** without it, reversing starting at `left = 1` (the very head) would need a special case, since there'd be no "node before position 1" to serve as `pre`. With the dummy in place, `pre` simply stays at `dummy` in that scenario, and `dummy.next` correctly reflects the new head once the loop finishes.

**Mechanism:** `curr` never moves during the inner loop — it represents the node that starts the sub-range and ends up as its tail. Each iteration takes the node immediately after `curr` and re-links it to the front of the sub-range (right after `pre`), repeatedly "peeling" nodes off the front of the not-yet-reversed remainder and inserting them at the head of the growing reversed segment.

**Worked trace:** `1→2→3→4→5`, `left=2, right=4`. `pre` walks one step to node `1`. `curr = pre.next = node 2`.

*Iteration 1:* `nodeToMove = 3`. `curr.next (2.next) = 3.next = 4` → `2→4`. `nodeToMove.next (3.next) = pre.next = 2` → `3→2`. `pre.next (1.next) = 3` → `1→3`. State: `1→3→2→4→5`.

*Iteration 2:* `nodeToMove = curr.next = 2.next = 4`. `curr.next (2.next) = 4.next = 5` → `2→5`. `nodeToMove.next (4.next) = pre.next = 3` → `4→3`. `pre.next (1.next) = 4` → `1→4`. State: `1→4→3→2→5`.

Final: `1 → 4 → 3 → 2 → 5`. Reversing positions 2–4 (values `2,3,4`) of `[1,2,3,4,5]` gives exactly `[1,4,3,2,5]`. Correct.

**Complexity:** Time O(right) — a single pass, no separate "reverse then reconnect" phase. Space O(1).

**Edge cases:**
- `left == right` — the inner loop runs zero times; the list is returned completely unchanged.
- `left == 1` — `pre` stays at `dummy`; handled without any special case, exactly the payoff of using the dummy head.
- `right == n` (the reversal reaches the true end) — no different from any other case; nothing about the algorithm depends on there being nodes after `right`.

**⚠️ Common Mistake:** assuming `curr` should advance during the inner loop. It shouldn't — `curr` is the fixed tail of the reversed sub-range throughout; only `pre.next` and the node being relocated change.

---

## Extra Practice 2 (Extension): Merge k Sorted Lists (LeetCode 23, Hard) — Pattern: k-Way Merge

**Statement:** Given an array of `k` sorted linked lists, merge them into one sorted list.

### Approach 1 — Brute force

Extract every value from all `k` lists into one array, sort it (O(N log N) where `N` is the total node count across all lists), rebuild a new list. Correct, but ignores that each individual list is already sorted, and allocates entirely new nodes.

### Approach 2 — Heap-based k-way merge (reuses Day 17's `PriorityQueue` + Day 29's anonymous classes)

```java
public static ListNode mergeKListsHeap(ListNode[] lists) {
    PriorityQueue<ListNode> minHeap = new PriorityQueue<ListNode>(new Comparator<ListNode>() {
        @Override
        public int compare(ListNode a, ListNode b) {
            return a.val - b.val;
        }
    });

    for (ListNode list : lists) {
        if (list != null) {
            minHeap.offer(list);
        }
    }

    ListNode dummy = new ListNode(-1);
    ListNode tail = dummy;
    while (!minHeap.isEmpty()) {
        ListNode smallest = minHeap.poll();
        tail.next = smallest;
        tail = tail.next;
        if (smallest.next != null) {
            minHeap.offer(smallest.next);
        }
    }
    return dummy.next;
}
```

**Mechanism:** the heap holds at most one node per list at any time — that list's current, not-yet-merged front. Repeatedly pull the smallest across all `k` fronts, append it to the result, and if the node it came from has a successor, push that successor in to take its place as that list's new front.

**⚠️ Common Mistake:** getting the `Comparator`'s sign backwards. `a.val - b.val` is negative when `a` should sort first, which is exactly what a **min**-heap needs; flipping it would silently build a max-heap and merge in descending order with no compile-time warning.

### Approach 3 — Divide and conquer (reuses today's exact `mergeTwoLists`, unmodified)

```java
public static ListNode mergeKListsDivideAndConquer(ListNode[] lists) {
    if (lists == null || lists.length == 0) {
        return null;
    }
    return mergeRange(lists, 0, lists.length - 1);
}

private static ListNode mergeRange(ListNode[] lists, int start, int end) {
    if (start == end) {
        return lists[start];
    }
    int mid = start + (end - start) / 2;
    ListNode left = mergeRange(lists, start, mid);
    ListNode right = mergeRange(lists, mid + 1, end);
    return mergeTwoLists(left, right);   // today's Problem 4 routine, reused directly, unmodified
}
```

**Mechanism:** pair up the `k` lists and merge them two at a time, recursively, halving the number of still-unmerged lists at each level, until one list remains.

**Worked trace:** `lists = [[1,4,5], [1,3,4], [2,6]]` (the classic example). `mergeRange` splits into `merge(lists[0], lists[1])` and `lists[2]` alone. `mergeTwoLists([1,4,5], [1,3,4])` — running today's exact algorithm — produces `1→1→3→4→4→5`. That result is then merged with `[2,6]` via the same routine again, producing the final `1→1→2→3→4→4→5→6`. (Every step here is literally today's `mergeTwoLists`, called twice — nothing new is happening algorithmically, only the recursive structure deciding *which two lists* to feed it at each step.)

**Complexity — walk through why both optimized approaches land on O(N log k), via different reasoning:**

- **Heap:** the heap never holds more than `k` elements (one per list). Every one of the `N` total nodes is pushed and popped exactly once, and each push/pop on a heap of size ≤ `k` costs O(log k). Total: O(N log k) time, O(k) extra space.
- **Divide and conquer:** think in terms of merge *rounds*. Round 1 pairs up the `k` lists into `k/2` merged pairs — collectively, every node across all `k` lists is touched exactly once during this round, so the round's total cost is O(N), regardless of how it's split across the `k/2` parallel merges. Round 2 merges those `k/2` results into `k/4`, again touching every node once, again O(N) total. This halves the list count each round, for O(log k) rounds — O(N) work per round × O(log k) rounds = O(N log k) time. Space: O(log k) for the recursion's call stack (Day 8's call-stack-depth-as-space-cost, again) — no separate heap structure needed at all.

**🔑 Key Takeaway — same time complexity, genuinely different trade-off:** both approaches are O(N log k) time, but divide-and-conquer uses less extra space (O(log k) call stack vs. O(k) heap storage) and needs nothing beyond today's own merge routine plus recursion — no `PriorityQueue`, no custom `Comparator`. The strongest opening names both, states they match asymptotically, and picks divide-and-conquer as the tighter-space option, with the heap version as the alternative worth having ready if asked for a second approach.

**Complexity summary:** Brute force O(N log N) time, O(N) space. Heap O(N log k) time, O(k) space. Divide & conquer O(N log k) time, O(log k) space.

**Edge cases:**
- Empty `lists` array — return `null` immediately (guarded explicitly in the divide-and-conquer entry point).
- Some entries in `lists` are `null` (empty lists mixed in with real ones) — the heap approach skips `null` entries before offering them; `mergeTwoLists` already handles a `null` input list correctly (traced on Day 34/today), so divide-and-conquer needs no extra guard beyond the top-level empty-array check.
- `k == 1` — both approaches correctly degenerate to returning the single list unchanged.

**💡 Interview Insight:** this is the natural hard-tier follow-up to Merge Two Sorted Lists — open by naming both viable approaches and their shared complexity class before picking one to implement; that framing alone signals more than jumping straight to code.

---

## Career Block Guide (1 hr)

**Weekly Industry Awareness Ritual (20 min):** clear the TLDR Newsletter backlog; read one engineering blog post in full — ideally from one of the companies identified on Day 29/30/34, closing the loop on this week's networking groundwork rather than letting the list sit unused.

**Weekly Scorecard — Day 35, five weeks in.**

**⚠️ Technical accuracy note before the numbers:** the plan's own scorecard line reads **"70 total DSA problems solved."** This is the *required-ladder-only* count — the same convention `Week_04_Revised.md` used internally for its own "57 problems" Day 28 checkpoint (per `00_Curriculum_Map.md`'s explicit note on that figure), not a count of every distinct problem actually solved including extra practice. Verified directly: `57` (Week 4's required-only checkpoint) `+ 13` (Week 5's own required problems: 2 each on Days 29–32 and 34–35, 1 on Day 33) `= 70` — matching the plan's figure exactly.

The fuller picture, matching how `00_Curriculum_Map.md` has tracked things since Week 1 (distinct problems, including every extra rep):

- **Weeks 1–4, distinct problems solved (verified against the curriculum map): 76.**
- **Week 5 required: 13** (LC 35, 162, 33, 81, 153, 34, 74, 875, 1011, 206, 876, 234, 21).
- **Week 5 extra practice: 6** (LC 154, 744 on Day 31; LC 1482, 1552 on Day 33; LC 92, 23 today).
- **Week 5 distinct total: 19.**
- **Cumulative distinct problems solved, Weeks 1–5: 95.**

**Binary Search fully closed at 11 required (up from 8) + 4 extra = 15 distinct**, with Median of Two Sorted Arrays deliberately reserved for right after Trees. **Linked Lists underway, 4 of its 11-problem ladder done** (the "11" here matches `Week_06_Revised.md`'s own header and this week's own internal scorecard text — see the accuracy note at the top of Day 34 for why the "12" in `Week_05_Revised.md`'s header appears to be a stale figure). `todo-api` is live with its first endpoint, built on REST fundamentals and Spring Boot's reflection-based routing, both entirely new this week.

---

## Week 5 Consolidation

### What Actually Got Built

Five days closed out Binary Search (Days 29–33: 9 more required problems plus 4 extra reps, bringing the pattern to 11 required / 15 distinct total across Weeks 4–5), then pivoted into Linked Lists (Days 34–35: 4 required problems plus 2 extra reps). Two full theory threads opened from zero — Threads and the JVM's concurrency model (Day 29), and REST APIs with Spring Boot (Day 34) — and `todo-api`, a new repository that stays live for the rest of the plan, shipped its first working endpoint.

### Planned vs. Actual

| | Planned | Actual |
|---|---|---|
| Binary Search required | 9 new (closing at 11) | 9 new, closed at 11 — exactly on plan |
| Binary Search extra practice | not specified by the plan | 4 added (LC 154, 744, 1482, 1552) |
| Linked Lists required | 4 | 4 — exactly on plan |
| Linked Lists extra practice | not specified by the plan | 2 added (LC 92, 23) |
| Linked Lists total ladder | "12" per `Week_05_Revised.md`'s header; **11** confirmed correct (see accuracy notes, Days 34–35) | 4 of 11 done, continuing Week 6 |
| `todo-api` | initialized, `/health` live | delivered exactly as planned |
| New theory threads | Threads/JVM concurrency, REST/Spring Boot | both delivered in full |

### Diagnostic — Confirm Before Moving On

- [ ] Can state, from memory, which of Binary Search's 15 solved problems are "on the input" vs. "on the answer," and why that distinction is the one that gets missed under interview pressure.
- [ ] Can prove — not just state — why comparing to `nums[right]` rather than `nums[left]` is what makes rotated-array search unambiguous.
- [ ] Can explain a linked list node's `next` field as a heap reference, tying it directly back to Day 9, without hesitation.
- [ ] Can reverse a linked list both iteratively and recursively, and state precisely why their space complexities differ.
- [ ] Can explain the dummy-head technique and name at least one other situation this week it was reused in (Reverse Linked List II).
- [ ] Can explain Spring's `@RestController`/`@GetMapping` as reflection-based startup configuration, distinct from `@Override`'s compile-time-only role.

### What Week 6 Assumes

Week 6 closes Linked Lists (Days 36–39: cycle detection, harder pointer manipulation, LRU Cache) before opening Stacks. It assumes today's fast/slow and reversal mechanics are fully reflexive — Linked List Cycle (Day 36) reuses fast/slow with no re-explanation, exactly the way today reused Day 34's without one. It assumes `todo-api` is live and reachable, since Day 36 adds Spring Data JPA and a real database-backed entity directly on top of what was initialized today. And critically, it assumes Day 29's Threads/JVM concurrency model — stack-per-thread, one shared heap, the thread lifecycle — is solid, since Week 6, Day 37 builds `ReentrantLock` and a concurrent `Counter` directly on that foundation, the first time this series demonstrates *actual data corruption* from a race condition, not just non-deterministic ordering.

---

## Day 35 — Interview Questions

**Q1. Why does Palindrome Linked List's middle-plus-reversal approach need no special case for odd-length lists?** Reversing starting at the middle node sets that node's own `next` to `null`. On odd length, that same node is the last node reached from both the original head's traversal and the reversed second half's traversal — it gets compared to itself (trivially equal), and both pointers hit `null` on the same step.

**Q2. What does the dummy head actually solve?** It removes the need to branch on "is the result list still empty" every time a node is appended — the dummy always gives `tail` something valid to attach to, and the real answer is read off `dummy.next` once the loop finishes.

**Q3. Why would `left == 1` need a special case in Reverse Linked List II without a dummy head?** Because there'd be no real node "before position 1" to serve as `pre` — the dummy head supplies that role uniformly, so reversing from the very head needs no different code path than reversing from anywhere else in the list.

**Q4. In Reverse Linked List II, why doesn't `curr` move during the inner loop?** `curr` represents the fixed tail of the sub-range being reversed — each iteration relocates the node immediately after `curr` to the front of the sub-range instead, which is why `curr` stays put while everything around it changes.

**Q5. Merge k Sorted Lists has two O(N log k) approaches. What's the actual trade-off between them, not just "they're the same Big-O"?** The heap approach needs O(k) extra space for the heap itself; divide-and-conquer needs only O(log k) for its recursion's call stack, and reuses the exact two-list merge routine already in hand rather than needing `PriorityQueue` and a custom `Comparator` at all.

**Q6. Walk through why divide-and-conquer's merging is O(N log k), specifically.** Each round of pairwise merging touches every node across all lists exactly once — O(N) total per round, however that's split across the round's parallel merges — and the number of lists halves each round, giving O(log k) rounds. O(N) per round × O(log k) rounds = O(N log k).

**Q7. What does the plan's "70 total DSA problems" figure actually count, and how does it differ from `00_Curriculum_Map.md`'s own running total?** It's a required-ladder-only count (57 through Week 4, plus 13 required in Week 5), the same convention Week 4's plan used internally for its own "57" checkpoint. The curriculum map's own total (95 through Week 5) additionally counts every extra-practice problem solved beyond the plan's required list.

**Q8. What does Week 6 assume is already solid from this week's theory, and why does it matter concretely?** Day 29's Threads/JVM concurrency model — specifically stack-per-thread, one shared heap — because Week 6, Day 37 builds actual locking (`ReentrantLock`) directly on it, and demonstrates real data corruption from a race condition for the first time, not just the non-deterministic ordering Day 29 showed.

---

## Daily Deliverable Check

- [ ] Palindrome Linked List and Merge Two Sorted Lists solved, pushed.
- [ ] Extra practice: Reverse Linked List II and Merge k Sorted Lists (both the heap and divide-and-conquer approaches) solved, pushed — both checked against `00_Curriculum_Map.md` and `Week_06_Revised.md` beforehand; no collisions found.
- [ ] Weekly ritual and scorecard complete, with the required-only vs. distinct-including-extras distinction understood, not just the raw numbers copied down.
- [ ] Week 5 Consolidation diagnostic checklist genuinely passes — not skimmed.

---

## What Week 6 Assumes You Already Know Cold

Covered in full in the **Week 5 Consolidation** section above — Linked Lists' fast/slow and reversal mechanics, the dummy-head technique, and Day 29's Threads/JVM concurrency model, which Week 6's Day 37 builds real locking on top of.
