# Day 114 Resource Book — LLD #6: The Elevator System — SCAN/LOOK Dispatch, and Mock Interview #3

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 113 Resource Book](Day113_Resource_Book.md)
**Next ▶:** [Day 115 Resource Book](Day115_Resource_Book.md)
**Companion to:** Day 114 of `Week_17_Revised.md`

---

## Recap

Day 113 built the ATM on State (reapplied, zero new mechanism) and Chain of Responsibility (new — a naive mutate-as-you-go dispatcher shown broken, then fixed with a read-only check phase ahead of a commit phase). Today reuses State a *third* time — same mechanism, a third system — and pairs it with a genuinely new idea that has nothing to do with GoF's catalog: SCAN/LOOK, an OS disk-scheduling algorithm repurposed for elevator dispatch. Today also opens this series' third mock interview, in a format not used before: Machine Coding.

## Learning Objectives

By the end of today, without notes:

1. Design the Elevator's lifecycle on State — a third reapplication — and explain precisely what changed and what didn't versus Day 109 and Day 113.
2. State the exact difference between SCAN and LOOK, implement LOOK correctly (including direction persistence — continuing the current direction until genuinely nothing remains ahead, only then reversing), and prove it beats naive FIFO dispatch on a worked example.
3. Explain what a Machine Coding interview is actually evaluating, and why the discipline that serves an LLD interview (think deeply before writing code) can actively hurt here if taken too far.
4. Rebuild a Token Bucket rate limiter cold, from the spec alone, citing its two guarantees precisely (bounded burst, bounded average rate) without re-deriving them from scratch.
5. Solve Design Add and Search Words Data Structure (LC 211) cold, narrating the wildcard-branching mechanism from memory.

## Concept Dependency Map

```
Week 16 Day 109: State pattern, fully earned
Week 17 Day 113: State reapplied once already (ATM) — confirms the pattern
                 generalizes, not a one-off
Week 9  Day 60:  TreeMap navigation (ceilingKey/firstKey) — Consistent Hashing
Week 10 Day 68:  Token Bucket rate limiter — bounded burst (capacity) + bounded
                 average rate (continuous refill); needs synchronized (Wk6 D37)
        │
        ▼
Today: Elevator System
  ├─ State reapplied AGAIN (3rd time) — Idle/MovingUp/MovingDown/DoorsOpen
  │     models the elevator's OWN lifecycle — not each floor button's state
  │
  └─ SCAN/LOOK dispatch (NEW — needs: TreeSet navigation, same family as
     Wk9 D60's TreeMap.ceilingKey; enums, Wk1 D5)
        ├─ Precise SCAN vs LOOK distinction
        ├─ Direction-persistence: continue current direction until nothing
        │  remains ahead, THEN reverse — proven against naive FIFO
        └─ Worked trace: 8 floors of travel (LOOK) vs 17 (FIFO), same requests
        │
        ▼
Mock Interview #3 — Machine Coding format (NEW skill, not a data structure)
  └─ Token Bucket rebuilt cold (cites Wk10 D68 directly — no new algorithm)
        │
        ▼
DSA Revision: Design Add and Search Words Data Structure (LC 211)
  (needs: Trie mechanism — Wk9 D59; recursion — Wk2 D8)
```

---

# Part 1 — State Pattern, Reapplied a Third Time

**Prerequisites confirmed:** State's mechanism, Day 109; its generalization, Day 113. No new explanation of *why* State works is needed — only what's genuinely different about this system.

**What's different from the ATM:** the ATM's states were driven entirely by *external* events (a card inserted, a PIN typed). The elevator's states are driven by a mix of external events (a floor requested) and **internal progress** (arriving at a floor, one step at a time) — the state machine advances on its own, not only in response to input. That's a real difference worth naming, not a re-derivation of the pattern itself.

```java
public enum Direction { UP, DOWN }

public interface ElevatorState {
    void handleRequest(Elevator elevator, int floor);
    void step(Elevator elevator);   // one unit of progress — one floor, or doors closing
}
```

```java
public class IdleState implements ElevatorState {
    @Override
    public void handleRequest(Elevator elevator, int floor) {
        elevator.getPendingRequests().add(floor);
        if (floor > elevator.getCurrentFloor()) {
            elevator.setLastDirection(Direction.UP);
            elevator.setState(elevator.getMovingUpState());
        } else if (floor < elevator.getCurrentFloor()) {
            elevator.setLastDirection(Direction.DOWN);
            elevator.setState(elevator.getMovingDownState());
        } else {
            elevator.setState(elevator.getDoorsOpenState());   // requested its own floor
        }
    }
    @Override
    public void step(Elevator elevator) {
        // nothing pending — idle has nothing to advance
    }
}

public class MovingUpState implements ElevatorState {
    @Override
    public void handleRequest(Elevator elevator, int floor) {
        elevator.getPendingRequests().add(floor);   // LOOK will pick it up in the right order on its own
    }
    @Override
    public void step(Elevator elevator) {
        elevator.setCurrentFloor(elevator.getCurrentFloor() + 1);
        if (elevator.getPendingRequests().contains(elevator.getCurrentFloor())) {
            elevator.getPendingRequests().remove(elevator.getCurrentFloor());
            elevator.setState(elevator.getDoorsOpenState());
        }
    }
}

public class MovingDownState implements ElevatorState {
    @Override
    public void handleRequest(Elevator elevator, int floor) {
        elevator.getPendingRequests().add(floor);
    }
    @Override
    public void step(Elevator elevator) {
        elevator.setCurrentFloor(elevator.getCurrentFloor() - 1);
        if (elevator.getPendingRequests().contains(elevator.getCurrentFloor())) {
            elevator.getPendingRequests().remove(elevator.getCurrentFloor());
            elevator.setState(elevator.getDoorsOpenState());
        }
    }
}
```

`DoorsOpenState` is where dispatch actually happens — deferred to Part 2, since it directly calls the LOOK logic that hasn't been taught yet. **🔑 Key Takeaway:** three states in, the pattern-match check is unchanged from Day 109 — scan `Elevator` itself (below) and confirm it contains no `if (state == ...)` branching. It doesn't; every method is a one-line delegation, exactly as before.

```java
public class Elevator {
    private int currentFloor;
    private ElevatorState currentState;
    private Direction lastDirection = Direction.UP;   // arbitrary initial bias, corrected on first real request

    private final ElevatorState idleState = new IdleState();
    private final ElevatorState movingUpState = new MovingUpState();
    private final ElevatorState movingDownState = new MovingDownState();
    private final ElevatorState doorsOpenState = new DoorsOpenState();
    private final TreeSet<Integer> pendingRequests = new TreeSet<>();

    public Elevator(int startFloor) {
        this.currentFloor = startFloor;
        this.currentState = idleState;
    }

    public void requestFloor(int floor) { currentState.handleRequest(this, floor); }
    public void step()                  { currentState.step(this); }

    int getCurrentFloor()                    { return currentFloor; }
    void setCurrentFloor(int floor)          { this.currentFloor = floor; }
    void setState(ElevatorState state)       { this.currentState = state; }
    Direction getLastDirection()             { return lastDirection; }
    void setLastDirection(Direction d)       { this.lastDirection = d; }
    ElevatorState getIdleState()             { return idleState; }
    ElevatorState getMovingUpState()         { return movingUpState; }
    ElevatorState getMovingDownState()       { return movingDownState; }
    ElevatorState getDoorsOpenState()        { return doorsOpenState; }
    TreeSet<Integer> getPendingRequests()    { return pendingRequests; }

    Integer getNextStopInDirection(boolean up) {
        return up ? pendingRequests.higher(currentFloor) : pendingRequests.lower(currentFloor);
    }
}
```

---

# Part 2 — SCAN and LOOK, Precisely (New)

**Prerequisites confirmed:** `TreeSet`/`TreeMap` navigation methods — `higher`, `lower`, `ceiling`, `floor` — are the same family already used for `TreeMap.ceilingKey()` in Consistent Hashing (Week 9, Day 60); enums (Week 1, Day 5).

## The naive alternative, and why it's bad

Servicing floor requests in **arrival order** (FIFO) is the obvious first design — and it's a real correctness-adjacent problem, not just a style complaint: an elevator that zigzags based on request arrival order (up to floor 9 because that request came in first, then back down to floor 2 because that one came in second) burns far more travel distance, and real elevators would be unusable this way.

## SCAN vs. LOOK — the distinction that's easy to blur

Both are classic OS disk-scheduling algorithms, repurposed here. They're often used interchangeably in casual conversation, which is exactly the kind of imprecision worth avoiding:

- **SCAN:** the arm (elevator) moves in one direction, servicing every request along the way, and continues **all the way to the physical end** (the top floor, or the bottom) even if no request remains ahead — only reversing once it physically cannot go further.
- **LOOK:** the arm services every request along the way, but reverses **as soon as no request remains ahead in the current direction** — it never travels to the physical boundary just because that's where the building ends. This is what real elevators do, and it's what the plan is actually describing ("continue in the current direction... only reversing once nothing remains ahead in that direction").

**What's being built today is LOOK, not pure SCAN** — worth naming precisely if an interviewer uses the term "SCAN" loosely, since correcting the distinction (briefly, not pedantically) signals real understanding rather than pattern-matched vocabulary. *(Extension, not needed today: C-SCAN and C-LOOK are circular variants — service in one direction only, then jump back to the start without servicing on the return leg, which gives more uniform wait times at the cost of extra travel. Skippable under time pressure; mentioned here only in case it comes up.)*

## Implementing LOOK's dispatch decision

```java
public class DoorsOpenState implements ElevatorState {
    @Override
    public void handleRequest(Elevator elevator, int floor) {
        elevator.getPendingRequests().add(floor);   // can still take a request mid-stop
    }
    @Override
    public void step(Elevator elevator) {
        Direction preferred = elevator.getLastDirection();
        Integer sameDirection = elevator.getNextStopInDirection(preferred == Direction.UP);
        if (sameDirection != null) {
            elevator.setState(preferred == Direction.UP ? elevator.getMovingUpState() : elevator.getMovingDownState());
            return;
        }
        Integer opposite = elevator.getNextStopInDirection(preferred != Direction.UP);
        if (opposite != null) {
            Direction reversed = (preferred == Direction.UP) ? Direction.DOWN : Direction.UP;
            elevator.setLastDirection(reversed);
            elevator.setState(reversed == Direction.UP ? elevator.getMovingUpState() : elevator.getMovingDownState());
            return;
        }
        elevator.setState(elevator.getIdleState());   // truly nothing pending, either direction
    }
}
```

**Why `TreeSet.higher`/`lower` are exactly the right tool:** `higher(currentFloor)` returns the smallest pending floor strictly greater than the current one — precisely "the next stop if I keep going up" — in O(log n) against the red-black tree backing `TreeSet`, with zero manual scanning. `lower` is the symmetric query going down. This is the same navigable-structure family Consistent Hashing used to find "the next node clockwise" (Week 9, Day 60) — a different domain, an identical query shape.

**🔑 Key Takeaway — direction persistence is the entire algorithm.** The one-line difference between this and the broken "always check up first" version is `Direction preferred = elevator.getLastDirection()` — checking the *current* direction first, not an arbitrary fixed one. Get this backwards (always prefer UP regardless of where the elevator was actually headed) and the algorithm silently degrades into unnecessary direction reversals — still correct, but no longer LOOK.

## Worked trace: LOOK vs. FIFO, same requests

Elevator starts at floor 1, idle. Requests arrive in this order: **5, 2, 9, 6.**

```
requestFloor(5): Idle, 5 > 1 → lastDirection=UP, state=MovingUp. pending={5}
requestFloor(2): pending={2,5}
requestFloor(9): pending={2,5,9}
requestFloor(6): pending={2,5,6,9}

step: MovingUp → floor 2. In pending? yes → remove. pending={5,6,9}. → DoorsOpen
step: DoorsOpen. preferred=UP. higher(2)=5 → continue UP. → MovingUp
step: MovingUp → floor 3. not pending.
step: MovingUp → floor 4. not pending.
step: MovingUp → floor 5. pending? yes → remove. pending={6,9}. → DoorsOpen
step: DoorsOpen. preferred=UP. higher(5)=6 → continue UP. → MovingUp
step: MovingUp → floor 6. pending? yes → remove. pending={9}. → DoorsOpen
step: DoorsOpen. preferred=UP. higher(6)=9 → continue UP. → MovingUp
step: MovingUp → floor 7. not pending.
step: MovingUp → floor 8. not pending.
step: MovingUp → floor 9. pending? yes → remove. pending={}. → DoorsOpen
step: DoorsOpen. higher(9)=null. lower(9)=null (pending empty). → Idle
```

**LOOK's path:** 1 → 2 → 5 → 6 → 9. **Total travel: |1−2|+|2−5|+|5−6|+|6−9| = 1+3+1+3 = 8 floors.**

**FIFO's path** (service in arrival order: 5, 2, 9, 6): 1 → 5 → 2 → 9 → 6. **Total travel: 4+3+7+3 = 17 floors.**

**LOOK does the identical job in under half the travel** — 8 versus 17 — on the exact same request set. This is the proof that "SCAN/LOOK beats naive FIFO" isn't a vague design preference; it's a more-than-2x reduction in physical travel for this example, and the gap only widens as request volume grows.

**Complexity:** each dispatch decision is **O(log n)** against `n` pending requests (`TreeSet.higher`/`lower`). **Space O(n)** for the pending set.

**Edge cases:** a request for the elevator's *current* floor while idle (handled directly — straight to `DoorsOpenState`, no travel); a request arriving for a floor *behind* the elevator's current direction (correctly deferred — added to `pendingRequests`, picked up automatically once LOOK reverses, never causes an immediate detour); the pending set going empty mid-direction (both `higher` and `lower` return `null`, falls through to `IdleState` — verified above at floor 9).

---

# Mock Interview #3 — Machine Coding Format

**Prerequisites confirmed:** everything needed to build a rate limiter (Token Bucket, Week 10 Day 68) — this section supplies the recap, not new algorithmic material.

## What Machine Coding actually evaluates

This is a genuinely different format from the LLD interviews run so far this week, and from any DSA interview before that — worth naming precisely rather than treating as "LLD but faster."

**LLD interviews** are conversational and incremental: an interviewer probes design choices in real time, values the *discussion* of trade-offs as much as the code, and rarely demands a fully working, end-to-end-tested system in the room. **Machine Coding interviews** hand over a spec (often LLD-shaped, sometimes smaller) and then go largely silent for 60–90 minutes — the deliverable is **actual compiling, running code** that correctly handles the spec, evaluated after the fact. Design discussion barely factors in; whether it *runs* is close to the entire signal.

**💡 Interview Insight — the instinct that helps LLD interviews can actively hurt here.** Spending the first 30–40 minutes perfecting a class diagram before writing a line that executes is a defensible LLD strategy (the interviewer is watching that reasoning happen and scoring it). In Machine Coding, that same instinct, taken too far, produces a beautiful design and a rushed, broken implementation — and a design nobody saw is worth nothing if the code doesn't run.

## The actual discipline

1. **Get a compiling skeleton first.** Stub every class and method — even with a body that just `return null` or does nothing — before fully implementing any single one. The system should never be in a broken, non-compiling state for longer than a few minutes at a time; running out of time mid-refactor with red code scores far worse than a simpler solution that fully works.
2. **Implement incrementally, testing as you go.** A `main` method exercising two or three concrete cases is usually enough given the time budget — full JUnit coverage is a nice-to-have, not the priority, when the clock is the binding constraint.
3. **Resist gold-plating.** Implement exactly what the spec asks. Time spent on an unrequested feature is time not spent making the requested behavior correct.
4. **If time runs low, cut scope, not correctness.** A rate limiter that only implements the single required strategy, fully correct, beats one that half-implements two strategies and is broken in both.

## Today's task: rebuild Token Bucket, cold

**Recap only — this is not new material.** Token Bucket was taught in full on Week 10, Day 68. Its two guarantees, stated precisely: **bounded burst** — the bucket holds at most `capacity` tokens, so no burst of requests can ever exceed that ceiling regardless of how long the system was idle beforehand — and **bounded long-run average rate** — tokens refill continuously at `refillRatePerSecond`, so sustained throughput can never exceed that rate either. Both guarantees are enforced simultaneously, which is exactly what a naive fixed-window counter fails to do: a fixed window resets its count at a hard boundary, which means up to `2×capacity` requests can land back-to-back if they straddle the reset instant (a burst right at the end of one window, immediately followed by a burst right at the start of the next) — continuous refill has no such boundary to exploit.

```java
public class TokenBucketRateLimiter {
    private final long capacity;
    private final long refillRatePerSecond;
    private double availableTokens;
    private long lastRefillTimestampMillis;

    public TokenBucketRateLimiter(long capacity, long refillRatePerSecond) {
        this.capacity = capacity;
        this.refillRatePerSecond = refillRatePerSecond;
        this.availableTokens = capacity;   // start full
        this.lastRefillTimestampMillis = System.currentTimeMillis();
    }

    public synchronized boolean allowRequest() {
        refill();
        if (availableTokens >= 1) {
            availableTokens -= 1;
            return true;
        }
        return false;
    }

    private void refill() {
        long now = System.currentTimeMillis();
        double elapsedSeconds = (now - lastRefillTimestampMillis) / 1000.0;
        double tokensToAdd = elapsedSeconds * refillRatePerSecond;
        if (tokensToAdd > 0) {
            availableTokens = Math.min(capacity, availableTokens + tokensToAdd);
            lastRefillTimestampMillis = now;
        }
    }
}
```

**⚠️ Common Mistake — the `synchronized` is load-bearing, not defensive boilerplate.** Without it, two threads can both read `availableTokens` before either writes it back, both see enough tokens available, and both proceed — the exact Counter-corruption shape from Week 6, Day 37, applied to a different field. Dropping `synchronized` here isn't a performance micro-optimization; it's a correctness bug, precisely as Day 68 established.

**In the actual mock:** build this cold, from the spec above (bounded burst + bounded average rate, nothing else), without looking at the reference implementation first — that's the exercise. Compare afterward, not before.

---

# DSA Revision Block (1 hr)

## Design Add and Search Words Data Structure (LeetCode 211, Medium) — Pattern: Trie with Wildcard Branching

**Originally taught:** Week 9, Day 60.

**Statement:** implement a data structure supporting `addWord(word)` and `search(word)`, where `search` may contain `.` as a wildcard matching any single character.

```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEndOfWord = false;
}

public class WordDictionary {
    private final TrieNode root = new TrieNode();

    public void addWord(String word) {
        TrieNode current = root;
        for (char c : word.toCharArray()) {
            int index = c - 'a';
            if (current.children[index] == null) {
                current.children[index] = new TrieNode();
            }
            current = current.children[index];
        }
        current.isEndOfWord = true;
    }

    public boolean search(String word) {
        return searchHelper(word, 0, root);
    }

    private boolean searchHelper(String word, int index, TrieNode node) {
        if (node == null) return false;
        if (index == word.length()) return node.isEndOfWord;
        char c = word.charAt(index);
        if (c == '.') {
            for (TrieNode child : node.children) {
                if (searchHelper(word, index + 1, child)) {
                    return true;
                }
            }
            return false;
        }
        return searchHelper(word, index + 1, node.children[c - 'a']);
    }
}
```

**The mechanism, recapped:** `addWord` is the ordinary Trie insert (Day 59's mechanism, unchanged). `search`'s only real addition is what happens on `.` — instead of descending into exactly one child, it **branches into all 26**, returning `true` the instant any branch succeeds. Every non-wildcard character behaves exactly like standard Trie search — a single deterministic descent.

**Complexity:** `addWord` is **O(L)**, L = word length, identical to Day 59. `search` is **O(L)** when there are no wildcards; in the worst case — a query that's all wildcards — it's **O(26^L)**, since every level branches into all 26 children with no way to prune. In practice this worst case is loose: real branching at any node is bounded by however many distinct words actually share that prefix, not a full 26, so actual performance is almost always far better than the bound suggests.

**Edge cases:** an empty-string query (`index == word.length()` immediately — returns whether the root itself is a complete word); a query longer than any stored word (recursion correctly bottoms out at `node == null`); consecutive wildcards (`"..."` — branches at every level, exponential in the worst case, exactly as the bound predicts).

**💡 Interview Insight:** if asked "what if the alphabet weren't just lowercase a–z" — swap the fixed `TrieNode[26]` array for a `HashMap<Character, TrieNode>`; the traversal logic is otherwise unchanged. Worth saying unprompted, since it signals the array-vs-map choice was a deliberate trade (dense small alphabet → array; sparse/large alphabet → map), not the only way to build a Trie.

---

# Project Block Guide

**Repository:** `lld-java/elevator/`. Build the classes above. **Definition of done:** feed the exact request sequence (5, 2, 9, 6) from a starting floor of 1 and verify the service order is 2 → 5 → 6 → 9 (not arrival order); confirm total simulated travel is 8 floors, not 17. Pushed.

# Mock Interview #3 — Logistics

**Format:** Machine Coding, 60–90 minutes, solo and timed. **Task:** build the in-memory rate limiter above from the spec, without referring to the reference implementation until finished. **Grading yourself afterward:** did it compile and run throughout, not just at the end? Did both guarantees (bounded burst, bounded average rate) actually hold when tested? Would a stranger's test harness calling `allowRequest()` in a tight loop get correct behavior?

# Career Block Guide

**LinkedIn (20 min):** engagement only — no new post required today.
**Networking:** continue this week's outreach ahead of Day 118's application push.

---

# Day 114 — Interview Questions

**Q1. What changed about State's usage in the Elevator versus the ATM?** The mechanism is identical — a context delegating to an interface reference, states driving their own transitions — but the Elevator's states are also driven by internal progress (`step()`, arriving at a floor), not purely external events like the ATM's card/PIN input.

**Q2. State the precise difference between SCAN and LOOK.** SCAN travels all the way to the physical boundary (top or bottom floor) before reversing, even with nothing pending there. LOOK reverses as soon as nothing remains pending ahead in the current direction, without traveling to the boundary. Real elevators implement LOOK.

**Q3. Why does `DoorsOpenState.step()` check `getLastDirection()` before deciding where to go next, instead of always checking "up" first?** Checking a fixed direction first would ignore which way the elevator was actually already headed, causing unnecessary direction reversals — genuinely different (worse) behavior than LOOK, even though it would still eventually serve every request correctly.

**Q4. Why is `TreeSet.higher()`/`lower()` the right tool for finding the next stop, instead of iterating over the pending requests?** They run in O(log n) against the tree structure with no manual scanning, and are the same navigable-structure family already used for `TreeMap.ceilingKey()` in Consistent Hashing (Week 9) — a different problem, an identical query shape.

**Q5. What does a Machine Coding interview evaluate that an LLD interview doesn't?** Whether the code actually compiles and runs correctly against the spec, evaluated largely after the candidate works in near-silence — design discussion barely factors in, versus an LLD interview where the real-time trade-off conversation is a large part of the signal.

**Q6. Why can the "think deeply before coding" instinct that serves LLD interviews well actively hurt in Machine Coding?** Spending too much time perfecting a design before writing any code that runs risks a rushed, broken implementation at the end — and an unseen design scores nothing if the code doesn't execute. The discipline that wins here is a compiling skeleton first, incremental implementation second.

**Q7. State Token Bucket's two guarantees precisely.** Bounded burst — never more than `capacity` tokens available at once, regardless of idle time beforehand. Bounded average rate — sustained throughput can't exceed `refillRatePerSecond` long-run, since tokens only arrive that fast.

**Q8. Why does a naive fixed-window counter fail to provide both guarantees simultaneously?** A hard window boundary lets a burst at the end of one window and a burst at the start of the next both succeed, allowing up to roughly `2×capacity` requests in a short span straddling the reset — continuous refill has no boundary to exploit that way.

**Q9. Is `synchronized` on `allowRequest()` a performance optimization or a correctness requirement?** Correctness. Without it, two threads can both read `availableTokens` before either writes back the decrement, both see enough tokens, and both proceed — the same race shape as Week 6's Counter bug, just applied to a different field.

**Q10. In Design Add and Search Words, what specifically happens differently on a `.` versus a normal character?** A normal character descends into exactly one child, deterministically. A `.` branches into all 26 possible children, returning true the instant any one of them leads to a successful match on the rest of the query.

**Q11. What's the worst-case complexity of `search` with wildcards, and why is it usually much better in practice?** O(26^L) worst case, when the query is all wildcards, since every level branches fully with no way to prune. In practice, actual branching at any node is bounded by however many real words share that prefix — almost always far less than 26 — so realistic performance is much closer to O(L).

---

# Daily Deliverable Check

- [ ] Elevator System LLD complete — State for lifecycle, LOOK for dispatch, pushed to `lld-java/elevator/`.
- [ ] LOOK dispatch verified against the worked trace: service order 2→5→6→9, total travel 8 floors, versus FIFO's 17.
- [ ] Can state the SCAN-vs-LOOK distinction precisely, unprompted, without conflating the two.
- [ ] Mock Interview #3 completed — Token Bucket rebuilt cold, both guarantees verified against a self-written test.
- [ ] Design Add and Search Words Data Structure (LC 211) solved cold; wildcard-branching mechanism explainable from memory.

---

## What Tomorrow Assumes You Already Know Cold

Day 115 (Splitwise) needs Strategy — not re-taught, cited directly back to Day 107 as "its first full system-level application" — so today's third State reapplication should have you comfortable pattern-matching a GoF pattern onto a new system from a citation alone, with no re-derivation. Tomorrow also introduces a genuinely new heap role (a *pair* of independent heaps used for greedy matching, distinct from Week 9's balanced two-heap median technique) — today's exchange-argument proof style (largest-first dispensing, ATM) is exactly the rigor tomorrow's greedy settlement algorithm needs, including a place where the greedy approach's real limits get named honestly rather than oversold.

**Next:** [Day 115 Resource Book](Day115_Resource_Book.md) — Splitwise (Strategy, applied for real; dual-heap greedy settlement).
