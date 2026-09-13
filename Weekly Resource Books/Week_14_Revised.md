# Week 14 (Revised): Dynamic Programming Completes Entirely, Bit Manipulation Begins

**What changed:** State Machine DP and Tree DP close out Dynamic Programming completely — 35 problems across every subtype (1D, Grid, String, Interval, State Machine, Tree), the single largest pattern in this plan, exactly as thorough as originally intended. Bit Manipulation begins its run toward 10 problems (up from 8).

---

## Day 92 — State Machine DP Begins, and Java 21 Pattern Matching

### DSA Block (2.5 hrs)

**Concept Card — State Machine DP**
- What: `dp[i][state]` — the DP dimension isn't just your position in the array, it's also which of a small, fixed set of states you're in at that position.
- Why: problems with a natural "mode" (holding a stock vs. not, in a cooldown period vs. active) are much cleaner modeled as explicit states than as a pile of ad hoc conditionals. Draw the state transition diagram before writing any code, every time — it's genuinely faster than trying to reason it out directly in code.
- Interview signal: "buy/sell with constraints," anything with a clear small number of named states and explicit rules for moving between them.

- Problem 1: Best Time to Buy and Sell Stock with Cooldown — LeetCode #309 — Medium — Pattern: State Machine DP
  - Hint: three states — `hold`, `sold`, `rest`. Draw the transition diagram first: from `hold` you can stay or move to `sold`; from `sold` you must move to `rest`; from `rest` you can stay or move to `hold`.
  - Complexity: Time O(n) | Space O(1)
- Problem 2: Best Time to Buy and Sell Stock with Transaction Fee — LeetCode #714 — Medium — Pattern: State Machine DP
  - Hint: only two states needed this time — `hold` and `cash`. Subtract the fee at the moment you transition from `hold` to `cash` (i.e., when selling).
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Java 21 Pattern Matching
- Record Patterns let you destructure a record's fields directly inside a pattern, in one expression, instead of casting and then manually pulling fields out one at a time. Pattern Matching for `switch` extends this to branch on a value's exact shape, with guard conditions (`when` clauses) letting you add extra conditions to a specific branch without breaking the switch's exhaustiveness.
- Coding exercise: take the `PaymentState` sealed interface from earlier in this plan and rewrite its `switch` using Java 21 Record Patterns with a guard clause — for example, `case Success(var id, var amount) when amount > 10000 -> ...` for a high-value success case handled differently from a normal one.

### Project Block (1.5 hrs)
- Repository: `java-fundamentals`.
- Task: the Record Pattern rewrite above.
- Definition of done: pushed, compiles on Java 21, guard clause working correctly.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: review progress; prepare for a personal Month 3 milestone check given how much ground the last two leave weeks covered.

### Daily Deliverable
- [ ] Best Time to Buy/Sell Stock with Cooldown and with Transaction Fee solved, pushed to `dsa-java/dynamic-programming/`.
- [ ] Record Pattern rewrite pushed.

---

## Day 93 — State Machine DP Completes

### DSA Block (2.5 hrs)
- Problem 3: Best Time to Buy and Sell Stock III — LeetCode #123 — Hard — Pattern: State Machine DP
  - Hint: at most two transactions allowed — track four explicit states: `buy1`, `sell1`, `buy2`, `sell2`, where each later state can only build on the state immediately before it.
  - Complexity: Time O(n) | Space O(1)
- Problem 4: Best Time to Buy and Sell Stock IV — LeetCode #188 — Hard — Pattern: State Machine DP
  - Hint: generalize yesterday's four explicit states into `2k` states for `k` allowed transactions — `buy[i]`/`sell[i]` arrays instead of individually named variables.
  - Complexity: Time O(n×k) | Space O(k)

**This closes State Machine DP: 4 problems.**

### Theory Block (2 hrs)
- Topic: The Dead Letter Queue Pattern
- If a Kafka listener fails to process a message and retries forever, that failure blocks every message waiting behind it in the same partition. A Dead Letter Queue routes a message to a separate topic after a fixed number of failed attempts, letting normal processing continue while the failed message sits somewhere for manual inspection or a dedicated reprocessing job later.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: implement a DLQ for the platform's Kafka listener — after 3 failed processing attempts, route the message to an `order-events-dlq` topic.
- Definition of done: simulating a processing error eventually pushes the message to the DLQ topic, verified via console consumer.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: reach out to a peer to schedule next week's mock interview.

### Daily Deliverable
- [ ] Best Time to Buy/Sell Stock III and IV solved — State Machine DP ladder complete at 4 problems.
- [ ] DLQ pattern live and verified on the platform.

---

## Day 94 — Tree DP Completes, and Dynamic Programming Is Entirely Done

### DSA Block (2.5 hrs)

**Concept Card — Tree DP**
- What: a DFS that returns more than one value per node — typically "the best answer if this node is included" and "the best answer if it's excluded" — which the parent then combines.
- Why: many tree optimization problems genuinely need information from *both* children before the parent can decide anything, which plain top-down recursion doesn't give you on its own without this returned-pair technique.
- Interview signal: "maximum sum," "maximum independent set," or any optimization problem stated over a tree shape.

- Problem 1: House Robber III — LeetCode #337 — Medium — Pattern: DP on Trees
  - Hint: each node returns two values — the max money obtainable if this node is robbed, and the max if it isn't. If robbed, neither child can be robbed. If not robbed, each child can independently be robbed or not — take whichever of its two returned values is larger.
  - Complexity: Time O(n) | Space O(n)
- Problem 2: Binary Tree Maximum Path Sum — LeetCode #124 — Hard — Pattern: DFS / DP on Trees
  - Hint: at each node, compute the max path sum that can extend *upward to its parent* (which can only use one child's contribution, since a path can't branch twice) — while separately updating a global maximum for paths that curve *through* the node using both children at once.
  - Complexity: Time O(n) | Space O(n)

**This closes Tree DP: 2 problems, and closes Dynamic Programming entirely: 35 problems across 1D, Grid, String, Interval, State Machine, and Tree DP — the single largest pattern in this plan, given the room it genuinely needs and now actually got.**

### Theory Block (1 hr)
- Topic: Dynamic Programming, Fully Reviewed
- Before moving on: pick three problems from across the entire DP block — one from 1D, one from String DP, one from State Machine or Tree DP — and state each one's `dp[]` definition from memory, unprompted. If any of the three feel shaky, that's worth a short revisit now, while the whole pattern is still fresh, rather than discovering the gap during an actual interview months from now.

### Project Block (1 hr)
- Repository: none new — make sure every DP solution across all six subtypes is pushed and cleanly organized in `dsa-java/dynamic-programming/`.

### Career Block (1 hr)
- LinkedIn: Post 18 — a short retrospective on the DP block specifically: which subtype clicked fastest, which took longest, and why.
- Networking: reflect on progress; you're now past the halfway point of the DSA phase's hardest single pattern.

### Daily Deliverable
- [ ] House Robber III and Binary Tree Maximum Path Sum solved — Tree DP complete, **Dynamic Programming entirely complete at 35 problems.**
- [ ] Three-problem DP review complete. LinkedIn Post 18 published.

---

## Day 95 — Bit Manipulation Begins, and Service Discovery

### DSA Block (2.5 hrs)

**Concept Card — Bit Manipulation**
- What: working directly with a number's binary representation using `& | ^ ~ << >> >>>`.
- Why: some problems have an elegant O(1)-space, single-pass bitwise solution where the "obvious" approach needs a HashMap or an extra array — interviewers use these specifically to test whether you understand what's actually happening underneath the abstractions you use every day.
- Interview signal: "without extra space," "single number appears once," anything framed directly around binary representation.
- Key facts: `x ^ x = 0` and `x ^ 0 = x` (XOR cancels duplicates, and is both commutative and associative — order doesn't matter); `n & (n-1)` clears the lowest set bit.

- Problem 1: Single Number — LeetCode #136 — Easy — Pattern: Bitwise XOR
  - Hint: XOR every element together — every duplicate pair cancels itself out to 0, leaving only the single unpaired number.
  - Complexity: Time O(n) | Space O(1)
- Problem 2: Number of 1 Bits — LeetCode #191 — Easy — Pattern: Bit Manipulation
  - Hint: `n & (n-1)` flips the lowest set bit to 0 — count how many times you can repeat this before `n` reaches 0.
  - Complexity: Time O(log n) | Space O(1)

### Theory Block (2 hrs)
- Topic: Service Discovery
- A hardcoded service IP breaks the moment that service restarts and gets a new one. Service discovery lets services find each other by name instead — Eureka is the classic Spring Cloud answer; Kubernetes' built-in CoreDNS is the Kubernetes-native equivalent, resolving a service's DNS name to whichever pod is currently healthy.
- Coding exercise: none — the project below is the exercise.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: register each module with Eureka so they're discoverable by name rather than a hardcoded address.
- Definition of done: the Eureka dashboard shows every module registered and discoverable.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: identify 3 Target Tier B companies.

### Daily Deliverable
- [ ] Single Number and Number of 1 Bits solved, pushed to `dsa-java/bit-manipulation/`.
- [ ] Platform modules discoverable by name via Eureka.

---

## Day 96 — Bit Manipulation Continues, and ConfigMaps/Secrets

### DSA Block (2.5 hrs)
- Problem 3: Power of Two — LeetCode #231 — Easy — Pattern: Bit Manipulation **(new)**
  - Hint: a power of two has exactly one bit set — so `n > 0 && (n & (n-1)) == 0` checks it in one line.
  - Complexity: Time O(1) | Space O(1)
- Problem 4: Counting Bits — LeetCode #338 — Easy — Pattern: DP + Bit Manipulation
  - Hint: `dp[i] = dp[i >> 1] + (i & 1)` — the bit count of `i` is the bit count of `i/2`, plus one more if `i` is odd.
  - Complexity: Time O(n) | Space O(n)

### Theory Block (2 hrs)
- Topic: Kubernetes ConfigMaps and Secrets
- ConfigMaps hold non-sensitive configuration values; Secrets hold sensitive ones — base64-encoded by default, which is encoding, not real encryption, so genuine security still needs an external secrets provider or a tool like Sealed Secrets on top. Both can be injected into a pod as environment variables or mounted as files, decoupling configuration from the container image itself.
- Coding exercise: write a Kubernetes `secret.yaml` (base64-encoded) and a `deployment.yaml` injecting it as a `DB_PASSWORD` environment variable.

### Project Block (1.5 hrs)
- Repository: `scalable-ecommerce-platform`.
- Task: the Secret/ConfigMap exercise above, applied to the platform's actual database credentials.
- Definition of done: the deployment pulls `DB_PASSWORD` from the Secret, not a hardcoded value.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: send connection requests to the 3 Tier B targets from yesterday.

### Daily Deliverable
- [ ] Power of Two and Counting Bits solved, pushed.
- [ ] Secret-based DB credential injection live on the platform.

---

## Day 97 — Bit Manipulation: Isolation Tricks, and StatefulSets/DaemonSets

### DSA Block (2.5 hrs)
- Problem 5: Missing Number — LeetCode #268 — Easy — Pattern: Bitwise XOR
  - Hint: XOR every index and every value together — everything present cancels out in pairs, leaving only the missing number.
  - Complexity: Time O(n) | Space O(1)
- Problem 6: Single Number II — LeetCode #137 — Medium — Pattern: Bit Manipulation **(new)**
  - Hint: for each bit position, count how many numbers have that bit set. Since every number except one appears exactly 3 times, the correct bit of the answer is whichever value makes each bit-count divisible by 3.
  - Complexity: Time O(n) | Space O(1)

### Theory Block (2 hrs)
- Topic: StatefulSets and DaemonSets
- Deployments treat pods as interchangeable, which is fine for stateless services but wrong for anything needing stable network identity and storage tied to a specific instance, like a database — StatefulSets exist to provide exactly that. DaemonSets guarantee exactly one pod runs on every node, used for node-level agents like log shippers or monitoring collectors.
- Coding exercise: none — sketch when you'd reach for a StatefulSet instead of a Deployment for a hypothetical self-hosted Postgres running inside Kubernetes.

### Project Block (1 hr)
- Repository: none new — the sketch above is today's task, saved as a short design note in the platform's `docs/` folder.

### Career Block (1 hr)
- LinkedIn: engagement — 20 minutes commenting on 3-5 posts.
- Networking: apply to 1 Tier B target company.

### Daily Deliverable
- [ ] Missing Number and Single Number II solved, pushed.
- [ ] StatefulSet vs. Deployment note pushed.

---

## Day 98 (Sunday) — Bit Manipulation: Splitting Duplicates, and Consolidation

### Self-Check (15 min)
- [ ] Solve one Bit Manipulation problem from this week cold, without hints.

### DSA Block (2 hrs)
- Problem 7: Single Number III — LeetCode #260 — Medium — Pattern: Bitwise XOR
  - Hint: XOR everything together to get `A XOR B`, the two unique numbers combined. Find the lowest set bit of that result (`diff &= -diff`) — this bit must differ between A and B specifically, so using it to split every number into two groups puts A in one group and B in the other, cleanly.
  - Complexity: Time O(n) | Space O(1)
- Problem 8: Sum of Two Integers — LeetCode #371 — Medium — Pattern: Bit Manipulation
  - Hint: XOR gives you the sum *without* carrying; AND, shifted left by one, gives you exactly the carries that XOR dropped. Repeat the process using the new sum-without-carry and the new carry, until there's no carry left.
  - Complexity: Time O(1), bounded by integer width | Space O(1)

### Career Block (1 hr)
- **Weekly Industry Awareness Ritual (20 min):** TLDR Newsletter backlog, one engineering blog post.
- **Weekly Scorecard:** Day 98, fourteen weeks in, **198 total DSA problems solved. Dynamic Programming — the largest pattern in this entire plan — is completely closed at 35 problems**, delivered at the depth it actually deserved rather than rushed to protect a date on a calendar. Bit Manipulation is 6 of its expanded 10 problems in. The platform now has Eureka service discovery, Secrets-based config, and StatefulSet awareness alongside everything from the last month of work.

### Daily Deliverable
- [ ] Single Number III and Sum of Two Integers solved, pushed.
- [ ] Weekly ritual and scorecard complete.
