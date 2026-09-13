# Day 52 Resource Book — Trees Beyond BST: General LCA and Serialization, and WireMock

**Series:** SDE-2 Interview Prep Resource Books (Week 8) · [Curriculum Map](./00_Curriculum_Map.md)
**◀ Previous:** [Day 51](./Day51_Resource_Book.md) · **Next ▶:** [Day 53](./Day53_Resource_Book.md)
**Companion to:** Day 52 of `Week_08_Revised.md`

---

## Recap: what today is

Yesterday's Lowest Common Ancestor leaned entirely on the BST ordering invariant to compute a single direction at every step. Today asks the same question — find the LCA of two nodes — on a **plain** binary tree, where that invariant doesn't exist. Both children genuinely have to be searched, and the results combined; there's no shortcut left to exploit. Today's second problem, Serialize and Deserialize Binary Tree, isn't new from nothing either — Day 47's brute-force approach to Same Tree already used explicit null markers to serialize two trees and compare the resulting strings. Today builds the other half of that idea: given a serialized string, rebuild the actual tree.

**No extra practice today** — both required problems are substantial (297 is this book's first Hard-rated tree problem), and the day's schedule already carries new theory (WireMock) plus the usual project and career blocks. Where a day is already full with two genuinely hard-earned problems, adding a third for the sake of "extra reps" would cost more in rushed depth than it would add in reinforcement — this is exactly the "don't pad a pattern that's genuinely full already" judgment call the reps philosophy for this series has used before.

## Learning Objectives

By the end of today, without notes:

1. Solve Lowest Common Ancestor of a Binary Tree (general), explaining precisely why it needs postorder combination of both children where yesterday's BST version needed only one.
2. Solve Serialize and Deserialize Binary Tree, explaining why explicit null markers let a single traversal (preorder alone) reconstruct a tree that would otherwise need two traversals to disambiguate.
3. Explain what WireMock stubs, why that's a genuinely different testing need from TestContainers (Day 47), and write a working stub.

## Concept Dependency Map for Today

```
Day 46: universal recursive template (base case null, combine left/right)
Day 51: LCA of a BST — used ordering to compute ONE direction
        │
        ▼
Problem 13: Lowest Common Ancestor of a Binary Tree, general (LC 236)
  needs: postorder combine — BOTH children searched, no invariant to skip either

Day 47: Same Tree brute force — serialize both trees (explicit null markers) + compare strings
  (seeded but did not build: reconstructing a tree FROM a serialized string)
        │
        ▼
Problem 14: Serialize and Deserialize Binary Tree (LC 297)
  needs: preorder traversal + the null-marker idea, extended to the reverse direction

Day 45: JUnit 5 (@Test, assertions)
Day 47: TestContainers — a REAL dependency (Postgres), containerized, for realistic integration tests
        │
        ▼
NEW: WireMock — a FAKE dependency's HTTP responses, stubbed, for CONTROLLED failure-mode testing
  (opposite motivation from TestContainers — contrast this explicitly, not just two unrelated tools)
```

---

# Part 1 — General Tree Lowest Common Ancestor

## Problem 13: Lowest Common Ancestor of a Binary Tree (LeetCode 236, Medium) — Pattern: DFS, Postorder

**Statement:** Given the root of a binary tree (not necessarily a BST) and two nodes `p` and `q`, both guaranteed to exist, find their lowest common ancestor.

### Why yesterday's approach doesn't transfer

Yesterday's BST version worked by *computing* a single direction at each node — no ordering invariant exists here, so there's no way to know which child, if either, actually contains `p` or `q` without searching. Both children have to be searched, unconditionally, and the results combined — the same "combine left/right" shape from Day 46's universal template, but with a genuinely new combination rule.

### Approach — postorder DFS, combine what each side reports

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) {
        return root;
    }

    TreeNode leftResult = lowestCommonAncestor(root.left, p, q);
    TreeNode rightResult = lowestCommonAncestor(root.right, p, q);

    if (leftResult != null && rightResult != null) {
        return root;                                  // p and q were found on OPPOSITE sides — root is the split point
    }

    return leftResult != null ? leftResult : rightResult;   // propagate whichever side found something (or null if neither did)
}
```

**What the base case actually encodes:** `root == null` means "nothing found down this path." `root == p` or `root == q` means "found one of the targets — report it upward," *without* needing to keep searching below it (a node is allowed to be its own ancestor for this problem's definition, and once a target is found, nothing further below it needs checking for this purpose).

**What the combine step means, precisely:** if `leftResult` and `rightResult` are **both** non-null, that means one target was found somewhere in the left subtree and the other somewhere in the right subtree — the current node is exactly the point where paths to each target diverge, which is the definition of their LCA. If only one side is non-null, that side either contains *both* targets deeper down (and the actual LCA hasn't been identified yet, so keep propagating that result upward for a higher call to resolve) or contains just one target with the other target's ancestor still to be found elsewhere — either way, propagating the non-null side upward is correct in both cases.

**Worked trace:** tree rooted at `3`, with `3.left=5`, `3.right=1`, `5.left=6`, `5.right=2`, `2.left=7`, `2.right=4`. Find `LCA(6, 4)`.

- `lca(3,6,4)`: not null, not 6, not 4. Recurse both sides.
  - `lca(5,6,4)`: not null, not 6, not 4. Recurse both sides.
    - `lca(6,6,4)`: `root == p(6)` → **returns 6**.
    - `lca(2,6,4)`: not null, not 6, not 4. Recurse both sides.
      - `lca(7,6,4)`: neither null-match nor target-match; both children null → returns `null`.
      - `lca(4,6,4)`: `root == q(4)` → **returns 4**.
      - Back at node `2`: `left=null`, `right=4` → only right is non-null → **propagate 4 upward**.
    - Back at node `5`: `left=6`, `right=4` — **both non-null** → node `5` **is the LCA**. Return `5`.
  - `lca(1,6,4)`: neither `6` nor `4` exists anywhere under `1` → eventually returns `null`.
  - Back at root `3`: `left=5`, `right=null` → only left non-null → propagate `5` upward.
- Final answer: `5`. Correct — `6` is `5`'s direct child, `4` is `5`'s grandchild through `2`, and `5` is exactly where their paths diverge.

**Complexity: Time O(n)** — every node visited at most once in the worst case (both targets deep in different subtrees, or the search has to traverse most of the tree to rule paths out). **Space O(h)** for the recursion stack.

**Edge cases:** `p` is an ancestor of `q` (correctly returns `p`, since the base case fires the moment `root == p` without needing to also confirm `q` is somewhere below it — the problem's guarantee that both nodes exist makes this safe); `p` and `q` are the same node (trivially, that node is its own LCA); a tree that's just a single node equal to both `p` and `q` in a degenerate input.

> 🔑 **Key Takeaway:** yesterday's BST version and today's general version solve the *same question* with a *different amount of information available*. The BST's ordering invariant is what let yesterday's version skip a whole subtree computationally; without it, "search both, combine" is the only option — not a weaker technique, just the one required when there's nothing to compute a shortcut from. Day 53 asks you to state this distinction crisply in one sentence each — worth having it clear now.

---

# Part 2 — Serialize and Deserialize Binary Tree

## Problem 14: Serialize and Deserialize Binary Tree (LeetCode 297, Hard) — Pattern: DFS (Preorder) with Null Markers

**Statement:** Design an algorithm to serialize a binary tree to a string, and deserialize that string back to the exact original tree structure. No constraints on the algorithm's format beyond round-tripping correctly.

### 🔗 Where this idea already appeared

Day 47's brute-force approach to Same Tree serialized both input trees — using explicit markers for `null` children — and compared the two resulting strings for equality. That was a one-directional use of the idea: turn a tree into a string, for comparison purposes only. Today needs the **other direction**: given a string produced this way, rebuild the actual tree it came from. That's the genuinely new half.

### Why explicit null markers are the whole trick

A **plain** preorder traversal — visiting only real nodes, with nothing recorded for an absent child — is fundamentally ambiguous to reconstruct from alone. Given just `[1, 2, 3]` as a preorder sequence with no other information, there's no way to know whether `2` is `1`'s left child with `3` somewhere below it, or whether `2` and `3` are both direct children of `1`, or several other shapes — multiple distinct trees share the same "real node" preorder sequence. This is exactly why Construct Binary Tree from Preorder and Inorder (yesterday) needed a **second** traversal (`inorder`) to disambiguate subtree sizes.

**Recording an explicit sentinel for every `null` child removes that ambiguity entirely.** With nulls written into the sequence, the string doesn't just say *which* values exist — it says exactly *where every subtree ends*, because a subtree's boundary is now marked directly rather than needing to be inferred from a second traversal's structure. This is why serialization only needs **one** traversal order (preorder, by convention — any consistent order would work) instead of two.

### Approach — preorder serialize, replay the same order to deserialize

```java
public String serialize(TreeNode root) {
    StringBuilder sb = new StringBuilder();
    serializeHelper(root, sb);
    return sb.toString();
}

private void serializeHelper(TreeNode node, StringBuilder sb) {
    if (node == null) {
        sb.append("#,");
        return;
    }
    sb.append(node.val).append(",");
    serializeHelper(node.left, sb);
    serializeHelper(node.right, sb);
}

public TreeNode deserialize(String data) {
    Queue<String> tokens = new LinkedList<>(Arrays.asList(data.split(",")));
    return deserializeHelper(tokens);
}

private TreeNode deserializeHelper(Queue<String> tokens) {
    String token = tokens.poll();
    if (token.equals("#")) {
        return null;
    }
    TreeNode node = new TreeNode(Integer.parseInt(token));
    node.left = deserializeHelper(tokens);
    node.right = deserializeHelper(tokens);
    return node;
}
```

**Why deserialization can just "replay" the same recursive shape:** the tokens were written in preorder — root, then everything in the left subtree (including its own nulls), then everything in the right subtree. A `Queue`, consumed strictly front-to-back, presents tokens in exactly that same order regardless of how deep the recursion currently is — so `deserializeHelper` can build the root, then immediately recurse to consume "the rest of the left subtree's tokens" before touching a single right-subtree token, mirroring how the write side produced them. This is the same shared-pointer-during-recursion idea from yesterday's `preorderIndex` counter, except here the "pointer" walks through serialized tokens, and a null marker is actual **data** consumed from the queue — not merely the absence of a recursive call.

**Complexity: Time O(n)** for both `serialize` and `deserialize` — every real node and every null marker touched exactly once. **Space O(n)** for the resulting string/token list, plus **O(h)** recursion stack for each operation.

**Edge cases:** an empty tree (`root == null` at the top level — serializes to `"#,"` alone, deserializes back to `null` correctly); a tree that's a single node (serializes to `"val,#,#,"`); negative values or multi-digit values (the comma delimiter is what makes `Integer.parseInt(token)` safe regardless of digit count or sign — a fixed-width or no-delimiter format would need a different, more careful encoding).

> ⚠️ **Common Mistake:** using a delimiter that could also appear inside a value itself (e.g., using a single character with no separator between multi-digit numbers) — this silently corrupts round-tripping for any value that isn't a single digit. A consistent delimiter (comma, here) between *every* token, including nulls, is what keeps parsing unambiguous.

> 💡 **Interview Insight:** the "why do you need null markers at all" question is the one to be ready for — the strongest answer is the ambiguity argument above (a bare `[1,2,3]` preorder sequence doesn't uniquely determine a tree shape), not just "because it works." Being able to construct the specific two-tree ambiguity example on the spot is a strong signal.

---

# Part 3 — WireMock

## What it stubs, and why

`todo-api`'s eventual features will depend on services it doesn't control — a payment gateway is the example the plan uses. Testing how your code behaves when that gateway succeeds is easy enough. Testing how it behaves when the gateway returns a `500`, or times out, or returns malformed data, is not — you can't reliably make a real third-party service misbehave on command, and even if you could, doing so against a live external system for routine test runs would be slow, flaky, and occasionally expensive.

**WireMock** solves this by running a real, local HTTP server inside your test process, which you configure ("stub") to return whatever response you want, for whatever request pattern you specify — status code, headers, body, and even artificial latency, all fully under your control and fully deterministic.

```java
@Test
void handlesSuccessfulPayment() {
    stubFor(post(urlEqualTo("/payment"))
        .willReturn(aResponse()
            .withStatus(200)
            .withHeader("Content-Type", "application/json")
            .withBody("{\"status\":\"success\",\"transactionId\":\"txn-123\"}")));

    // code under test makes a real HTTP call to http://localhost:8089/payment here,
    // and receives exactly the canned response configured above — no real payment gateway involved
}
```

## WireMock vs. TestContainers — opposite motivations, not just two different tools

Day 47 introduced TestContainers specifically because H2 (an in-memory database) doesn't enforce Postgres-specific behavior — the fix was to spin up a **real** Postgres, containerized, so tests exercise genuinely realistic behavior for a dependency you actually control and own.

WireMock solves the **opposite** problem. There's no "use the real payment gateway in a container" option — it's an external system you don't own, can't spin up locally, and critically, can't reliably force into specific failure states even if you could reach it. WireMock exists to fake that dependency's *responses*, deliberately and controllably, precisely because getting realistic-but-uncontrolled behavior isn't the goal here — getting a **specific, repeatable** condition (a timeout, a `500`, a malformed body) on demand is.

| | TestContainers (Day 47) | WireMock (today) |
|---|---|---|
| What it provides | A **real** instance of a dependency you own (Postgres) | A **fake** stand-in for a dependency you don't own (a third-party HTTP API) |
| Why | H2 doesn't match Postgres's real behavior — you want realism | You can't reliably force a real external API into a specific failure state — you want control |
| Runs as | An actual Docker container | An in-process (or locally-bound) HTTP server your test configures directly |

## Forward reference

The plan's own note is worth repeating here: this WireMock stub is being built now specifically so it's *already in place* once Resilience4j (a circuit-breaker and retry library) arrives in a few weeks. Testing a circuit breaker meaningfully requires being able to reliably force the exact failure conditions — timeouts, repeated errors — that trip it. That's not possible against a real, uncontrolled external dependency; it's exactly what a WireMock stub is built to provide on demand.

---

# Section — Project Block

**Repository:** `todo-api`. **Task:** the WireMock stub above (`http://localhost:8089/payment` returning `200 OK` with fixed JSON), written as a JUnit test fixture.

**Definition of done:** pushed, test passes against the stubbed endpoint — confirm the test actually exercises the stub (not accidentally hitting a real endpoint, and not trivially passing regardless of the stub's response) before considering this done.

# Section — Career Block

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts.

**Networking:** review 3 engineering manager profiles — note anything worth referencing in a future outreach message (a shared connection, a specific project they've posted about, a team they lead that matches your target roles).

---

# Day 52 — Interview Questions

**1. Why can't Lowest Common Ancestor of a Binary Tree (general) use the same single-direction approach as yesterday's BST version?**

*Answer:* There's no ordering invariant on a plain tree to compute which direction a target is in — both children genuinely have to be searched, with the results combined via postorder recursion, rather than a direction being computable in O(1) at each node.

---

**2. In the general LCA solution, what does it mean when `leftResult` and `rightResult` are both non-null at some node?**

*Answer:* It means one target was found somewhere in the left subtree and the other somewhere in the right subtree — that node is exactly the point where the paths to each target diverge, which is the LCA by definition.

---

**3. What does it mean when only one side's result is non-null, and why is propagating it upward still correct?**

*Answer:* It means either both targets are found deeper within that one side (and the true LCA hasn't been identified yet, so it needs to keep propagating up for a later call to resolve), or only one target is on that side with the other elsewhere still to be found — in both cases, passing the non-null result upward is the correct action.

---

**4. Why does a bare preorder sequence like `[1,2,3]`, with no null markers, fail to uniquely determine a tree's shape?**

*Answer:* Multiple distinct tree shapes can produce the identical preorder sequence when only real nodes are recorded — for example, `2` could be `1`'s left child with `3` below it, or `2` and `3` could both be direct children of `1`. Without a second traversal (like inorder) or explicit markers for absent children, there's no way to distinguish these.

---

**5. Why does adding explicit null markers let serialization use only ONE traversal, where Construct Binary Tree needed two?**

*Answer:* The null markers directly record where every subtree ends, rather than requiring that boundary to be inferred from comparing two different traversal orders. The ambiguity that made two traversals necessary is exactly what the null markers remove.

---

**6. How does deserialization know when it's finished consuming one subtree and should move to the next, without tracking explicit index ranges the way Construct Binary Tree did?**

*Answer:* The recursive structure itself does this — each call consumes exactly the tokens belonging to one subtree (a value token followed recursively by its left subtree's full token sequence, then its right subtree's), because that's the order they were written in. A shared `Queue`, consumed front-to-back, naturally stays synchronized with this as long as deserialization follows the identical recursive shape serialization used to write it.

---

**7. Why is the delimiter choice (a comma, in this implementation) load-bearing for correctness?**

*Answer:* Without a consistent separator between every token, multi-digit or negative values couldn't be unambiguously split back apart from the raw string — the delimiter is what makes `Integer.parseInt` on each token safe regardless of a value's sign or digit count.

---

**8. What's the core difference in motivation between TestContainers and WireMock, even though both are testing tools introduced in this project?**

*Answer:* TestContainers provides a real instance of a dependency you own, because you want realistic behavior (Postgres via H2 wasn't realistic enough). WireMock provides a fake stand-in for a dependency you don't own, because you want deterministic, controllable behavior (specific failure conditions) that a real, uncontrolled external service can't reliably be forced to produce.

---

**9. Why is WireMock being introduced now, ahead of Resilience4j actually arriving in the plan?**

*Answer:* Testing a circuit breaker meaningfully requires reliably forcing the exact failure conditions (timeouts, repeated errors) that trip it — which isn't possible against a real, uncontrolled external dependency. Having the WireMock stubbing capability already in place means it's ready the moment Resilience4j needs it.

---

## Daily Deliverable Check

- [ ] Lowest Common Ancestor of a Binary Tree (general) and Serialize/Deserialize Binary Tree solved, pushed — the contrast with yesterday's BST-LCA approach, and the null-marker ambiguity argument, both explainable without notes.
- [ ] WireMock stub test pushed, confirmed actually exercising the stub rather than trivially passing.

---

## What Tomorrow Assumes You Already Know Cold

Day 53 assumes both of today's LCA approaches — BST and general — are solid enough to compare directly and explain the difference in one sentence each; that's the day's actual theory exercise, not new material. It also assumes yesterday's divide-and-conquer formalization (the divide step requiring real computed work, not a free structural split) is fully reflexive, since Median of Two Sorted Arrays reuses that exact intuition — this time dividing via binary search over a partition point, rather than an index lookup.

**Next:** [Day 53 Resource Book](./Day53_Resource_Book.md) — Trees Capstone: The Median Fix, and Divide-and-Conquer Reviewed.
