# Day 113 Resource Book — LLD #5: The ATM Machine — Chain of Responsibility, and State Reapplied

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 112 Resource Book](Day112_Resource_Book.md) *(Week 16 — Library Management System, Mock #2)*
**Next ▶:** [Day 114 Resource Book](Day114_Resource_Book.md)
**Companion to:** Day 113 of `Week_17_Revised.md`

---

## Recap

Week 16 closed with Library Management System (Day 112) and five days of hard evidence that the 5-step LLD framework, all ten Structural/Behavioral patterns, and the judgment to recognize *when a pattern doesn't fit* are all genuinely reflexive now — not something today needs to re-establish. Week 17 is the second half of the LLD phase: six more systems, three more mock interviews, closing with all ten systems done and five mocks deep before System Design even begins.

Today reuses State exactly as built on Day 109 (Vending Machine) — same mechanism, a new system — and introduces one genuinely new pattern, Chain of Responsibility, this series' eleventh. The DSA block is pure spaced-repetition revision, not new material: one Heap problem, solved cold.

## Learning Objectives

By the end of today, without notes:

1. Design and implement the ATM's card/PIN/withdrawal lifecycle using the State pattern, citing Day 109's mechanism directly rather than re-deriving it.
2. Explain Chain of Responsibility precisely — what it decouples, why a sender neither knows nor cares how many handlers exist or which one(s) act — and justify it against the plain alternative (a single method with an if/else per denomination).
3. Implement a Chain-of-Responsibility cash dispenser that always uses the fewest notes for a canonical denomination set, prove *why* the greedy largest-first choice is safe here, and explain precisely where that same greedy idea stops being safe (Coin Change, Week 12).
4. Identify and fix a real check-then-act correctness bug in a naive dispense implementation — not just patch it, but explain why the naive version corrupts state on partial failure.
5. Solve K Closest Points to Origin (LC 973) cold, narrating the heap mechanism from memory.

## Concept Dependency Map

```
Week 16 Day 106: 5-step LLD framework (clarify → objects → relationships → patterns → code)
Week 16 Day 109: State pattern, fully earned (Vending Machine — NoCoin/HasCoin/
                 Dispensing/SoldOut; context delegates to an interface reference,
                 states drive their own transitions, zero dispatch conditionals)
Week 1 Day 6:    SOLID — Open/Closed Principle
Week 4 Day 23:   Greedy Algorithms — a locally-best choice, justified by an
                 exchange argument, not just asserted
Week 12 Day 84:  Coin Change (LC 322) — DP, because greedy provably fails there
        │
        ▼
Today: ATM Machine
  ├─ State REAPPLIED (no new mechanism) — Idle/HasCard/CorrectPIN/Dispensing
  │
  └─ Chain of Responsibility (NEW — needs: interfaces, Wk1 D2; recursion, Wk2 D8)
        ├─ DenominationHandler chain: ₹2000 → ₹500 → ₹100
        ├─ Naive (mutate-as-you-go) version — a real correctness bug on
        │  partial failure, worked through deliberately, not just patched
        ├─ Fixed: two-phase canDispense() / commitDispense() split
        └─ The chain's dispatch order is a GREEDY choice — proven safe here
           (each denomination is an exact multiple of the next), proven
           UNSAFE in general (worked counterexample, {1,3,4}, target 6) —
           this is exactly why Coin Change needed DP instead
        │
        ▼
DSA Revision: K Closest Points to Origin (LC 973)
  (needs: Heap mechanism — Wk8 D54; custom Comparator — Wk8 D55)
```

---

# DSA Revision Block (1 hr)

This week carries six DSA revision blocks — spaced-repetition, cold-solve checks on already-closed patterns, not new material. Today's is Heap. **Solve it cold, without notes, before reading the solution below** — that's the actual exercise; what follows is the reference to check yourself against.

## K Closest Points to Origin (LeetCode 973, Medium) — Pattern: Max-Heap of Size k

**Originally taught:** Week 8, Day 55, as one of that day's two required Heap problems (paired with Kth Largest Element in an Array).

**Statement:** Given an array of points on the X-Y plane and an integer `k`, return the `k` points closest to the origin `(0, 0)`, in any order.

**The core idea:** maintain a **max-heap of size k** holding the k closest points *found so far*. For each new point: if the heap has fewer than k points, add it unconditionally. Once the heap is full, compare the new point's distance against the heap's current *worst* (largest) distance — the root of a max-heap. If the new point is closer, evict the root and insert the new point; otherwise discard it. The heap's root is always the current "worst of the best" — exactly what needs evicting the instant something closer shows up.

```java
public int[][] kClosest(int[][] points, int k) {
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>(new Comparator<int[]>() {
        @Override
        public int compare(int[] a, int[] b) {
            int distA = a[0] * a[0] + a[1] * a[1];
            int distB = b[0] * b[0] + b[1] * b[1];
            return distB - distA;   // reversed subtraction — max-heap by distance
        }
    });

    for (int[] point : points) {
        maxHeap.offer(point);
        if (maxHeap.size() > k) {
            maxHeap.poll();   // evict the current worst (largest distance)
        }
    }

    int[][] result = new int[k][2];
    for (int i = 0; i < k; i++) {
        result[i] = maxHeap.poll();
    }
    return result;
}
```

**Why max-heap and not min-heap:** this is the detail worth being able to say unprompted. A min-heap of *all* n points would need every point inserted, then k pops from the front — O(n log n). A **bounded** max-heap of size k only ever holds the k best candidates seen so far; the moment it's full, the O(log k) eviction check is cheap, and it's a max-heap specifically because the operation needed is "find and discard the current worst" — which is the max-heap's root. Reaching for a min-heap here (a common reflex, since the problem asks for the *closest*) is the wrong reflex; the heap ordering should match what you need to evict, not what you need to keep.

**No squared-distance subtlety to skip:** comparing `dist²` instead of `dist` avoids a `Math.sqrt()` call entirely — distance ordering is preserved under squaring since all distances are non-negative, and floating-point square roots are both slower and a source of precision bugs versus plain integer arithmetic.

**Complexity: Time O(n log k)** — n insertions, each O(log k) against a heap bounded at size k (not size n). **Space O(k)** for the heap, O(k) for the output — genuinely better than sorting all n points by distance (O(n log n)) whenever k is meaningfully smaller than n, which is the usual case this problem is testing for.

**Edge cases:** `k == points.length` (heap never evicts — equivalent to returning everything); duplicate points at the same distance (no tie-breaking specified, any k of the tied set is valid — don't over-engineer a tie-break the problem doesn't ask for); a single point with `k=1` (trivial, heap of size 1).

**💡 Interview Insight:** this is the same "bounded max-heap of size k" role from Week 8 Day 55's Kth Largest Element in an Array — cite that lineage directly rather than re-deriving the reasoning. If asked to extend this to a data *stream* (points arriving one at a time, indefinitely, "give me the k closest so far at any moment"), the exact same heap already answers that with zero design changes — worth saying out loud, since it signals the design generalizes rather than being a one-shot answer to this exact array-input phrasing.

---

# Part 1 — State Pattern, Reapplied

No new mechanism here — this is Day 109's State pattern, applied to a new system, cited rather than re-derived. **Prerequisites confirmed:** State's mechanism (a context delegating to an interface reference; each state drives its own transitions; states are typically parameterless, reusable singletons) — Week 16, Day 109.

## The ATM's Four States

**Requirements clarified first** (per the 5-step framework, Day 106): withdrawal only, for today — deposits and balance checks are out of scope, matching the plan's own framing. PIN validation is in scope. The machine stocks a fixed set of denominations (today: ₹2000, ₹500, ₹100).

Four states, matching the plan directly: `Idle` (no card), `HasCard` (card in, PIN not yet verified), `CorrectPIN` (PIN verified, awaiting a withdrawal amount), `Dispensing` (cash actively being counted out).

```java
public interface ATMState {
    void insertCard(ATM atm, Card card);
    void enterPIN(ATM atm, int pin);
    void requestWithdrawal(ATM atm, int amount);
    void ejectCard(ATM atm);
}
```

Every method is declared on every state, even ones that are illegal from that state — each concrete state answers all four, usually by throwing. This is deliberate: a state that silently ignored an illegal call would hide a caller's bug; throwing surfaces it immediately, which is the same "fail loud on a contract violation" instinct as an unchecked array index.

```java
public class IdleState implements ATMState {
    @Override
    public void insertCard(ATM atm, Card card) {
        atm.setCurrentCard(card);
        atm.setState(atm.getHasCardState());
        System.out.println("Card inserted. Enter your PIN.");
    }
    @Override
    public void enterPIN(ATM atm, int pin) {
        throw new IllegalStateException("Insert a card first.");
    }
    @Override
    public void requestWithdrawal(ATM atm, int amount) {
        throw new IllegalStateException("Insert a card first.");
    }
    @Override
    public void ejectCard(ATM atm) {
        throw new IllegalStateException("No card is inserted.");
    }
}

public class HasCardState implements ATMState {
    @Override
    public void insertCard(ATM atm, Card card) {
        throw new IllegalStateException("A card is already inserted.");
    }
    @Override
    public void enterPIN(ATM atm, int pin) {
        if (atm.getCurrentCard().getPin() == pin) {
            atm.setState(atm.getCorrectPINState());
            System.out.println("PIN correct. Select a withdrawal amount.");
        } else {
            atm.setCurrentCard(null);
            atm.setState(atm.getIdleState());
            System.out.println("Incorrect PIN. Card ejected.");
        }
    }
    @Override
    public void requestWithdrawal(ATM atm, int amount) {
        throw new IllegalStateException("Enter your PIN first.");
    }
    @Override
    public void ejectCard(ATM atm) {
        atm.setCurrentCard(null);
        atm.setState(atm.getIdleState());
        System.out.println("Card ejected.");
    }
}

public class CorrectPINState implements ATMState {
    @Override
    public void insertCard(ATM atm, Card card) {
        throw new IllegalStateException("A card is already inserted.");
    }
    @Override
    public void enterPIN(ATM atm, int pin) {
        throw new IllegalStateException("PIN already verified.");
    }
    @Override
    public void requestWithdrawal(ATM atm, int amount) {
        atm.setState(atm.getDispensingState());
        boolean success = atm.getCashHandlerChain().dispense(amount);
        if (success) {
            System.out.println("Please collect Rs." + amount + ".");
        } else {
            System.out.println("Cannot dispense that exact amount. Transaction cancelled.");
        }
        atm.setCurrentCard(null);
        atm.setState(atm.getIdleState());
    }
    @Override
    public void ejectCard(ATM atm) {
        atm.setCurrentCard(null);
        atm.setState(atm.getIdleState());
        System.out.println("Card ejected.");
    }
}

public class DispensingState implements ATMState {
    @Override
    public void insertCard(ATM atm, Card card) {
        throw new IllegalStateException("ATM is busy dispensing cash.");
    }
    @Override
    public void enterPIN(ATM atm, int pin) {
        throw new IllegalStateException("ATM is busy dispensing cash.");
    }
    @Override
    public void requestWithdrawal(ATM atm, int amount) {
        throw new IllegalStateException("A withdrawal is already in progress.");
    }
    @Override
    public void ejectCard(ATM atm) {
        throw new IllegalStateException("Cannot eject mid-dispense.");
    }
}
```

```java
public class ATM {
    private final ATMState idleState = new IdleState();
    private final ATMState hasCardState = new HasCardState();
    private final ATMState correctPINState = new CorrectPINState();
    private final ATMState dispensingState = new DispensingState();

    private ATMState currentState = idleState;
    private Card currentCard;
    private final CashHandler cashHandlerChain;

    public ATM(CashHandler cashHandlerChain) {
        this.cashHandlerChain = cashHandlerChain;
    }

    public void insertCard(Card card)        { currentState.insertCard(this, card); }
    public void enterPIN(int pin)             { currentState.enterPIN(this, pin); }
    public void requestWithdrawal(int amount) { currentState.requestWithdrawal(this, amount); }
    public void ejectCard()                   { currentState.ejectCard(this); }

    // Package-visible — only the State classes reach into these.
    void setState(ATMState state)      { this.currentState = state; }
    ATMState getIdleState()            { return idleState; }
    ATMState getHasCardState()         { return hasCardState; }
    ATMState getCorrectPINState()      { return correctPINState; }
    ATMState getDispensingState()      { return dispensingState; }
    void setCurrentCard(Card card)     { this.currentCard = card; }
    Card getCurrentCard()              { return currentCard; }
    CashHandler getCashHandlerChain()  { return cashHandlerChain; }
}
```

**🔑 Key Takeaway:** `ATM` contains **zero** `if (state == ...)` dispatch — every method is a one-line delegation to `currentState`. Verify this directly against the code the same way Day 109 verified it against `VendingMachine`, rather than taking it on faith: scan every method body above and confirm none of them branches on what state it's in.

**💡 Interview Insight — `DispensingState`'s other three methods are transient but not dead code.** In this synchronous design, `DispensingState` is entered and exited inside a single `requestWithdrawal` call — no other method ever actually gets invoked while in it. It's still fully implemented, and that matters the moment this stops being synchronous: a real ATM's cash dispenser is a physical device with its own latency, and a production version would transition into `DispensingState`, *await a hardware callback* confirming the notes were actually counted and released, and only then transition out — at which point another thread genuinely could call `ejectCard()` mid-dispense, and the throw becomes load-bearing rather than defensive. Naming this unprompted is a good signal; building it is out of scope today, since the plan reserves concurrency for BookMyShow later this week.

---

# Part 2 — Chain of Responsibility, Precisely (New)

**Prerequisites confirmed:** interfaces (Week 1, Day 2); recursion over a linked structure (Week 2, Day 8's recursion; Week 5, Day 34's self-referential `Node` shape).

## What problem it solves

A request needs to be handled, but the sender doesn't know — and shouldn't need to know — how many objects are capable of handling it, or which one(s) actually will. Each candidate handler gets a chance, in sequence: it either fully handles the request, partially handles it and passes the remainder along, or declines and passes the whole thing along untouched. The **sender only ever talks to the first handler in the chain.**

For cash dispensing, the naive alternative is a single method with an `if/else` per denomination:

```java
// The alternative NOT being built — shown once, to be explicit about what's rejected and why.
public void dispenseNaive(int amount) {
    if (amount >= 2000) { /* dispense some ₹2000 notes */ }
    if (amount % 2000 >= 500) { /* dispense some ₹500 notes */ }
    // ... etc, one branch per denomination, all in one method
}
```

This works, but it violates **Open/Closed** (Week 1, Day 6) directly: adding a new denomination — a ₹50 note, say — means *editing* this existing, already-tested method, exactly the kind of modification-of-working-code Open/Closed exists to avoid. Chain of Responsibility's actual payoff isn't "it produces a different number" — the naive version and the chain produce identical output — it's that **each denomination's logic lives in its own class**, independently testable, and a new denomination is a new class plus one line wiring it into the chain, touching nothing that already works.

## Building the chain

```java
public abstract class CashHandler {
    protected final int denomination;
    protected int availableNotes;
    protected final CashHandler nextHandler;

    protected CashHandler(int denomination, int availableNotes, CashHandler nextHandler) {
        this.denomination = denomination;
        this.availableNotes = availableNotes;
        this.nextHandler = nextHandler;
    }
    // dispense() defined below, after a deliberate first attempt that's broken.
}

public class TwoThousandHandler extends CashHandler {
    public TwoThousandHandler(int availableNotes, CashHandler nextHandler) {
        super(2000, availableNotes, nextHandler);
    }
}
public class FiveHundredHandler extends CashHandler {
    public FiveHundredHandler(int availableNotes, CashHandler nextHandler) {
        super(500, availableNotes, nextHandler);
    }
}
public class HundredHandler extends CashHandler {
    public HundredHandler(int availableNotes, CashHandler nextHandler) {
        super(100, availableNotes, nextHandler);
    }
}
```

Constructed tail-first, since each handler needs a reference to the next one before it can exist:

```java
CashHandler hundred = new HundredHandler(50, null);
CashHandler fiveHundred = new FiveHundredHandler(20, hundred);
CashHandler chain = new TwoThousandHandler(10, fiveHundred);   // the chain's head — this is what the ATM holds
```

## The broken first attempt — worth seeing explicitly

```java
// BROKEN — mutates inventory before the full chain has confirmed success.
public boolean dispenseBroken(int amount) {
    if (amount == 0) return true;
    int notesToUse = Math.min(amount / denomination, availableNotes);
    availableNotes -= notesToUse;                       // mutated NOW
    int remainder = amount - notesToUse * denomination;
    if (remainder == 0) return true;
    if (nextHandler != null) return nextHandler.dispenseBroken(remainder);
    return false;                                        // chain exhausted, money still owed
}
```

**⚠️ Common Mistake, traced concretely:** suppose the ₹2000 handler has only 1 note left, the ₹500 handler has plenty, and the ₹100 handler has *zero* notes. Request ₹2300. The ₹2000 handler dispenses one note (`availableNotes` drops to 0), passes ₹300 down. The ₹500 handler can't make ₹300 from ₹500 notes, passes the full ₹300 down unchanged. The ₹100 handler needs 3 notes, has 0, dispenses nothing, remainder stays ₹300, no `nextHandler` — returns `false`. The overall call correctly reports failure — **but the ₹2000 handler's inventory has already been decremented**, as if a note that was never actually handed to anyone left the machine. Call `dispenseBroken` again on a *different* request afterward, and the machine is now lying about how much cash it holds. This is a genuine, silent correctness bug, not a cosmetic one.

## The fix: a two-phase check-then-commit split

```java
public abstract class CashHandler {
    protected final int denomination;
    protected int availableNotes;
    protected final CashHandler nextHandler;

    protected CashHandler(int denomination, int availableNotes, CashHandler nextHandler) {
        this.denomination = denomination;
        this.availableNotes = availableNotes;
        this.nextHandler = nextHandler;
    }

    /** Phase 1 — read-only. Never mutates availableNotes. */
    public boolean canDispense(int amount) {
        if (amount == 0) return true;
        int notesUsable = Math.min(amount / denomination, availableNotes);
        int remainder = amount - notesUsable * denomination;
        if (remainder == 0) return true;
        if (nextHandler != null) return nextHandler.canDispense(remainder);
        return false;
    }

    /** Phase 2 — only ever called after canDispense() has already confirmed success. */
    private void commitDispense(int amount) {
        if (amount == 0) return;
        int notesUsable = Math.min(amount / denomination, availableNotes);
        availableNotes -= notesUsable;
        if (notesUsable > 0) {
            System.out.println("Dispensing " + notesUsable + " x Rs." + denomination + " note(s)");
        }
        int remainder = amount - notesUsable * denomination;
        if (remainder > 0 && nextHandler != null) {
            nextHandler.commitDispense(remainder);
        }
    }

    public boolean dispense(int amount) {
        if (!canDispense(amount)) {
            return false;
        }
        commitDispense(amount);
        return true;
    }
}
```

**Why this is correct, not just differently broken:** `canDispense` reads `availableNotes` but never writes it, so calling it — even recursively through the entire chain, even when it ultimately returns `false` — leaves every handler's inventory untouched. `commitDispense` is `private` and is only ever reached through `dispense()`, *after* `canDispense` has already confirmed the exact same computation will succeed at every level. Since nothing else mutates `availableNotes` between the two calls (single-threaded, synchronous), `commitDispense` is guaranteed to retrace `canDispense`'s exact path and succeed everywhere it needs to.

**🔗 Forward reference:** that last sentence — "since nothing else mutates state between the check and the act" — is precisely the check-then-act assumption Day 111 showed is *false* under concurrency (`findAvailableSpot`/`assignVehicle`'s race). If this ATM needed to serve concurrent withdrawal requests against shared inventory, `canDispense`-then-`commitDispense` would need the same fix Parking Lot got: a lock held across both phases. Out of scope today — the plan reserves concurrency for BookMyShow (Days 116–117) — but worth having ready if an interviewer asks "what if two withdrawals happen at once?"

## Worked trace: ₹4700

```
canDispense(4700):
  ₹2000 handler: 4700/2000=2, min(2,10)=2 usable → remainder 700 → ask next
  ₹500 handler:  700/500=1, min(1,20)=1 usable  → remainder 200 → ask next
  ₹100 handler:  200/100=2, min(2,50)=2 usable  → remainder 0   → TRUE
canDispense returns true up the whole chain → commitDispense(4700) runs the
identical path for real: dispenses 2×₹2000, 1×₹500, 2×₹100 = 5 notes, ₹4700 exactly.
```

## Why greedy (largest-first) is safe here — proven, not asserted

The chain always tries the **largest** denomination first. That's a greedy choice, and Week 4's discipline applies directly: a greedy choice needs an exchange argument, not just a plausible feel.

**Claim:** for the denomination set {₹2000, ₹500, ₹100}, always preferring the largest usable note never produces more total notes than any other valid combination.

**Proof sketch (exchange argument):** ₹2000 = 4 × ₹500 = 20 × ₹100, and ₹500 = 5 × ₹100 — **each denomination is an exact integer multiple of the next-smaller one.** Suppose some alternative solution uses fewer than the maximum possible number of ₹2000 notes for a given amount, making up the difference with ₹500s and ₹100s. Since 4 notes of ₹500 (or any combination of ₹500s/₹100s summing to ₹2000) can always be swapped for exactly 1 note of ₹2000 with no change in total value, that swap strictly *reduces* the note count whenever it's available — so any solution using fewer large notes than greedy can always be improved by making this swap, meaning greedy is never worse. This is the same swap-and-compare shape as every other exchange argument this series has used (Week 4's interval-swap and domination arguments; the Cut Property, Week 11).

**Where this breaks — the counterexample that motivates Coin Change:** the "each denomination divides evenly into the next" property is what makes greedy safe, and it does **not** hold for arbitrary denomination sets. Take {1, 3, 4}, target 6: greedy takes 4 first (largest ≤ 6), leaving 2, which greedy fills with two 1s — **3 coins total (4+1+1)**. The actual optimum is **2 coins (3+3)**. Greedy fails here because 4 is not a multiple of 3, so there's no clean swap — taking the 4 forecloses a better combination the exchange argument can no longer repair. **This is exactly why Coin Change (LC 322, Week 12, Day 84) needed Dynamic Programming instead of greedy** — arbitrary denominations have no guaranteed swap property, so the only correct approach is exploring the actual subproblem space. Cite this connection directly if asked "why not just use greedy for Coin Change too."

**Complexity:** `dispense()` runs in **O(h)** time, where h is the number of handlers (a small constant — 3 today) — not dependent on the withdrawal amount or the number of notes in stock. **Space O(h)** for the recursion depth.

**Edge cases:** amount not a multiple of the smallest denomination (₹100) — `canDispense` correctly returns `false` once the final handler's remainder is nonzero and non-zero notes can't cover it (e.g., ₹150 is uncovarable with this set); amount of 0 (trivially `true`, dispenses nothing); a handler with 0 notes in stock (correctly skipped — `Math.min(needed, 0) = 0`, remainder passes through untouched).

---

# Part 3 — Full System Assembly (5-Step Framework)

1. **Requirements:** withdrawal only; PIN validation in scope; fixed denomination stock (₹2000/₹500/₹100), no restocking modeled today.
2. **Core objects:** `ATM` (context), `ATMState` (+ 4 concrete states), `Card`, `CashHandler` (+ 3 concrete handlers).
3. **Relationships:** `ATM` holds one `CashHandler` (the chain head) and one `ATMState` (the current state) at a time; `CashHandler`s form a singly-linked chain; `ATMState`s are singletons owned by `ATM`, referencing it back only through method parameters (never storing a reference to it — this keeps every state trivially reusable, the same design already established Day 109).
4. **Patterns applied deliberately:** State (lifecycle), Chain of Responsibility (dispensing) — two independent patterns solving two genuinely different problems in the same system, not one pattern doing double duty.
5. **Code:** above, in full.

```java
public class Card {
    private final String cardNumber;
    private final int pin;
    public Card(String cardNumber, int pin) { this.cardNumber = cardNumber; this.pin = pin; }
    public int getPin() { return pin; }
}

public class ATMDemo {
    public static void main(String[] args) {
        CashHandler hundred = new HundredHandler(50, null);
        CashHandler fiveHundred = new FiveHundredHandler(20, hundred);
        CashHandler chain = new TwoThousandHandler(10, fiveHundred);

        ATM atm = new ATM(chain);
        Card card = new Card("4242", 1234);

        atm.insertCard(card);
        atm.enterPIN(1234);
        atm.requestWithdrawal(4700);   // dispenses 2x2000 + 1x500 + 2x100
    }
}
```

---

# Project Block Guide

**Repository:** `lld-java/atm/`. Build the classes above (or your own equivalent honoring the same two patterns). **Definition of done:** a ₹4700 withdrawal dispenses via the fewest possible notes (verify: 2×₹2000, 1×₹500, 2×₹100 = 5 notes); an amount the current stock can't make exactly (e.g., request more ₹100-multiples than remain) cleanly returns failure with **zero inventory corruption** — verify this explicitly by checking `availableNotes` on every handler before and after a failed request; they must be identical. Pushed.

# Career Block Guide

**LinkedIn (20 min):** engagement only today — comment on 3–5 posts, no new post required.
**Networking:** continue this week's targeted connection-building — Rippling, Google, Databricks, Stripe, Uber, Atlassian, Walmart Global Tech. This is deliberately front-loaded ahead of Day 118's application push — warm connections take time to convert into real conversations, which is exactly why the plan starts this now rather than the day applications go out.

---

# Day 113 — Interview Questions

**Q1. Does `ATM` contain any conditional logic that inspects which state it's in?** No — every public method is a single-line delegation to `currentState`; the states themselves hold all the branching logic, which is the entire point of the pattern.

**Q2. Why does `ATMState` declare all four methods on every concrete state, including the illegal ones?** So an illegal call fails loudly (an explicit exception) rather than being silently swallowed — a caller invoking `enterPIN` with no card inserted has a real bug, and hiding that is worse than surfacing it immediately.

**Q3. What specifically does Chain of Responsibility decouple, and from what?** The sender (the ATM, requesting a dispense) from the handlers — the sender never knows how many denomination handlers exist or which ones actually end up dispensing anything; it only ever talks to the chain's head.

**Q4. Why is a single method with an if/else per denomination the wrong design, even though it computes the same answer?** It violates Open/Closed — adding a new denomination means editing an existing, already-tested method. Chain of Responsibility makes a new denomination a new class plus one line of wiring, touching nothing that already works.

**Q5. Walk through the exact bug in the naive `dispenseBroken` implementation.** It mutates `availableNotes` as it recurses, before knowing whether the *entire* chain will succeed. If a later handler in the chain can't cover its share, the method correctly returns `false` overall, but earlier handlers have already decremented inventory for notes that were never actually given to anyone — silent, persistent state corruption.

**Q6. How does the `canDispense`/`commitDispense` split fix that bug?** `canDispense` is read-only — it computes the exact same recursive path but never writes `availableNotes`. Only after it confirms success does `commitDispense` run, retracing the identical computation and mutating for real. Since nothing else can modify state between the two calls (single-threaded), `commitDispense` is guaranteed to succeed everywhere `canDispense` said it would.

**Q7. Under what condition would the check-then-commit split itself become unsafe?** Under concurrency — if two withdrawal requests could interleave between the `canDispense` check and the `commitDispense` act, both could pass the check against inventory that's no longer accurate by the time either commits. This is the identical check-then-act shape Day 111's Parking Lot race demonstrated; the fix would be the same, a lock held across both phases.

**Q8. Prove that always dispensing the largest possible denomination first never produces more notes than any other valid combination, for {₹2000, ₹500, ₹100}.** Each denomination is an exact multiple of the next-smaller one (₹2000 = 4×₹500 = 20×₹100). Any solution using fewer large notes than greedy can have some combination of its smaller notes swapped for one larger note of equal value with no change in total value paid, strictly reducing note count — so greedy can never be beaten, only matched.

**Q9. Why does that same greedy approach fail for Coin Change (LC 322)?** Coin Change's denominations are arbitrary and not guaranteed to divide evenly into each other — the counterexample {1,3,4}, target 6: greedy gives 4+1+1 (3 coins), optimal is 3+3 (2 coins). Without the divides-evenly property, there's no guaranteed swap, so only exploring the actual subproblem space (DP) is correct in general.

**Q10. In K Closest Points to Origin, why a max-heap rather than a min-heap?** The heap needs to answer "what's currently the worst of my k best candidates, so I can evict it?" — that's the max-heap's root. A min-heap would need every point inserted and k pops from the front, O(n log n) instead of the bounded max-heap's O(n log k).

**Q11. Why compare squared distances instead of actual distances in K Closest Points?** Squaring preserves ordering for non-negative distances while avoiding a `Math.sqrt()` call entirely — cheaper, and sidesteps floating-point precision issues a square root could introduce.

---

# Daily Deliverable Check

- [ ] ATM Machine LLD complete — `State` for the lifecycle, `Chain of Responsibility` for dispensing, both pushed to `lld-java/atm/`.
- [ ] `ATM` verified to contain zero state-dispatch conditionals (Day 109's discipline, reapplied).
- [ ] The naive `dispenseBroken` bug reproduced deliberately, then fixed via `canDispense`/`commitDispense` — inventory confirmed unchanged after a failed request.
- [ ] Can prove, not just state, why largest-first dispensing is safe for ₹2000/₹500/₹100 and unsafe in general (Coin Change contrast) — without notes.
- [ ] K Closest Points to Origin (LC 973) solved cold; heap-direction reasoning (max, not min) explainable from memory.

---

## What Tomorrow Assumes You Already Know Cold

Day 114 reuses State a *third* time (Elevator: Idle/MovingUp/MovingDown/DoorsOpen) with zero re-explanation — if today's ATM states still feel like they need re-deriving rather than being a direct pattern-match, that's worth closing before tomorrow. Tomorrow also introduces SCAN/LOOK dispatch, which needs nothing from today directly, and formalizes the Machine Coding interview format via a from-scratch Token Bucket rebuild — today's Chain of Responsibility discipline (a naive version shown broken, then fixed with a clear mechanism) is the same rigor tomorrow's rate-limiter recap expects you to bring to your own cold implementation.

**Next:** [Day 114 Resource Book](Day114_Resource_Book.md) — Elevator System (SCAN/LOOK), and Mock Interview #3 (Machine Coding Format).
