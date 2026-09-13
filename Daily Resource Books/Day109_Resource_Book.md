# Day 109 — LLD #2: Vending Machine (State Pattern), and Mock Interview #1

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 108 Resource Book](Day108_Resource_Book.md)
**Next ▶:** [Day 110 Resource Book](Day110_Resource_Book.md)
**Companion to:** Day 109 of `Week_16_Revised.md`

---

## Recap

Day 108 ran the 5-step framework live for the first time and correctly concluded no pattern was needed. Today runs it again, against a genuinely harder system, where a pattern **is** the point: State (introduced Day 107, applied only to a small `TrafficLight` example there) gets its real system-level test. The framework's five steps aren't re-explained today — Day 106 taught why each one matters, Day 108 proved they work end to end; today just runs them, faster, against a bigger system.

---

## Learning Objectives

By the end of today, without notes:

1. Design and implement a Vending Machine using the State pattern with **zero** state-dispatch conditionals in the context class.
2. Explain precisely how a state transition happens mechanically (which object calls `setState`, and why the state classes — not the context — decide what comes next).
3. Distinguish State from Strategy in two sentences, and give a concrete tell for which one a design actually needs.
4. Run and debrief a 45-minute LLD mock interview, and self-assess against the "clarified requirements first?" checkpoint.
5. Solve Longest Substring Without Repeating Characters (LC 3) cold, without hints.

---

## Concept Dependency Map

```
Day 106: 5-step LLD framework
Day 107: State pattern (TrafficLight — small illustrative example)
Day 108: framework applied live, end-to-end (Tic-Tac-Toe) — no re-explanation needed today
Week 3 Day 15: Sliding Window, variable-size (today's revision problem)
        │
        ▼
Today: framework applied live again — Vending Machine
  Step 1-4: requirements, objects, relationships, patterns (State — now justified, not illustrative)
  Step 5: Product/Inventory + State interface (NoCoinState, HasCoinState,
          DispensingState, SoldOutState) + VendingMachine context
        │
        ▼
Mock Interview #1 — Tic-Tac-Toe as subject, framework under time pressure
        │
        ▼
Theory: State vs. Strategy — the intent distinction, now that both are
concretely built (TrafficLight/VendingMachine vs. yesterday's DiscountStrategy)
        │
        ▼
🔗 Forward: Day 110-111's Parking Lot is the next full system — single-threaded
   first, then concurrency. Neither needs State; noticing that (like Day 108's
   Tic-Tac-Toe) is itself part of the exercise.
```

---

# Part 1 — DSA Revision Block (1 hr)

## Revision — Longest Substring Without Repeating Characters (LeetCode 3, Medium)

**🔗 Originally taught in full depth:** Week 3, Day 15 — Sliding Window, variable-size, the pattern's second day.

**Statement:** given a string `s`, return the length of the longest substring that contains no repeated characters.

**Attempt cold before reading on.**

### Recap: The Approach

A variable-size sliding window (`left`, `right`), plus a `HashMap<Character, Integer>` recording the **most recent index** each character was seen at. Expand `right` one step at a time; when the character at `right` was already seen *inside the current window* (its last-seen index is `>= left`), jump `left` directly to one past that duplicate's position — not one step at a time, straight there — since every index between the old `left` and the duplicate's position is now guaranteed to also be inside a window that still contains the duplicate.

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int maxLength = 0;
    int left = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
            left = lastSeen.get(c) + 1;   // jump left past the duplicate directly
        }
        lastSeen.put(c, right);
        maxLength = Math.max(maxLength, right - left + 1);
    }

    return maxLength;
}
```

### Fresh Trace — A Different Example Than Day 15 Used

`s = "pwwkew"`:

```
right=0 'p': not seen.              lastSeen={p:0}         window=[0,0] len=1  max=1
right=1 'w': not seen.              lastSeen={p:0,w:1}     window=[0,1] len=2  max=2
right=2 'w': seen at 1, 1>=left(0)  left=2                 window=[2,2] len=1  max=2
              lastSeen={p:0,w:2}
right=3 'k': not seen.              lastSeen={...,k:3}     window=[2,3] len=2  max=2
right=4 'e': not seen.              lastSeen={...,e:4}     window=[2,4] len=3  max=3
right=5 'w': seen at 2, 2>=left(2)  left=3                 window=[3,5] len=3  max=3
              lastSeen={...,w:5}

Result: 3   ("wke" or "kew")
```

**Complexity:** `O(n)` time — `right` visits every index exactly once, and `left` only ever moves forward, so across the *entire* run `left` also advances at most `n` total steps, not `n` steps *per* iteration of `right`; the two pointers together do `O(n)` total work, not `O(n²)`. Space is `O(min(n, k))` where `k` is the size of the character set in play (26 for lowercase-only input, 128 for ASCII) — the map can never hold more distinct entries than either the string's length or the alphabet size, whichever is smaller.

### Common Mistakes Checklist

- [ ] Omitting the `lastSeen.get(c) >= left` check and jumping on *any* previously-seen occurrence, even one that fell **outside** the current window (already excluded by an earlier jump). Without this check, `left` can be moved **backwards**, corrupting the window entirely — this is the single most common way this exact problem goes wrong, and it's silent: the code still runs and returns *a* number, just the wrong one on inputs where an old, stale occurrence exists outside the live window.
- [ ] Off-by-one on the window length — it's `right - left + 1` (both endpoints inclusive), not `right - left`.
- [ ] Reaching for a full `O(n²)` or `O(n³)` brute-force check-every-substring approach under time pressure without first trying to justify the sliding-window jump — worth having the "why can `left` skip multiple positions at once and still be correct" argument ready to state proactively, not just as a response to being asked.

**If any of this needed re-deriving rather than confirming:** Week 3, Day 15 has the full original treatment, including the fixed-size sliding window that opened the pattern the day before.

---

# Part 2 — LLD #2: Vending Machine, State Pattern Applied for Real

### Steps 1–4, Briskly — the Framework Is Established, Not Re-Explained

**Requirements, clarified:** a machine stocks a small, fixed catalog of products, each with its own price and remaining quantity. Payment is coin-based, coins can be inserted incrementally (balance accumulates), and change is returned if a payment exceeds the selected product's price. No card payment, no restocking flow, no multi-currency handling — all explicitly out of scope for today.

**Core objects:** `Product` (code, name, price, quantity), `VendingMachine` (the context — owns the inventory, the running balance, and the current state), and a `VendingMachineState` interface with four implementers: `NoCoinState`, `HasCoinState`, `DispensingState`, `SoldOutState`.

**Relationships:** `VendingMachine` composes a `Map<String, Product>` inventory and holds a reference to its current `VendingMachineState`; the four state classes are stateless singletons — created once, held as fields on `VendingMachine`, and swapped between, never re-instantiated per transition.

**Pattern, deliberately:** State — the machine's behavior for the same two actions (`insertCoin`, `selectProduct`) genuinely differs depending on which of four phases the transaction is in, and the transitions are driven by the machine's own operations, not by an external caller choosing a mode. This is precisely State's concrete signal from Day 107, now actually load-bearing rather than illustrative.

### Step 5 — Code the Core

```java
public interface VendingMachineState {
    void insertCoin(VendingMachine machine, int coinValue);
    void selectProduct(VendingMachine machine, String productCode);
    void dispense(VendingMachine machine);
}
```

```java
public class NoCoinState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine, int coinValue) {
        machine.addBalance(coinValue);
        machine.setState(machine.getHasCoinState());
        System.out.println("Coin accepted. Balance: " + machine.getBalance());
    }

    @Override
    public void selectProduct(VendingMachine machine, String productCode) {
        System.out.println("Insert a coin first.");
    }

    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Cannot dispense — no payment received.");
    }
}

public class HasCoinState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine, int coinValue) {
        machine.addBalance(coinValue);
        System.out.println("Coin accepted. Balance: " + machine.getBalance());
    }

    @Override
    public void selectProduct(VendingMachine machine, String productCode) {
        Product product = machine.getInventory().get(productCode);
        if (product == null || product.getQuantity() == 0) {
            System.out.println("Selected item is out of stock.");
            return;   // stays in HasCoinState — balance is preserved, a different item can still be chosen
        }
        if (machine.getBalance() < product.getPrice()) {
            System.out.println("Insufficient balance. Need " + (product.getPrice() - machine.getBalance()) + " more.");
            return;   // stays in HasCoinState — more coins can still be inserted
        }
        machine.setSelectedProductCode(productCode);
        machine.setState(machine.getDispensingState());
        machine.getState().dispense(machine);   // transition, then immediately act on it
    }

    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Select a product first.");
    }
}

public class DispensingState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine, int coinValue) {
        System.out.println("Please wait — dispensing in progress.");
    }

    @Override
    public void selectProduct(VendingMachine machine, String productCode) {
        System.out.println("Please wait — dispensing in progress.");
    }

    @Override
    public void dispense(VendingMachine machine) {
        Product product = machine.getInventory().get(machine.getSelectedProductCode());
        product.decrementQuantity();
        int change = machine.getBalance() - product.getPrice();
        System.out.println("Dispensing: " + product.getName() + ". Change returned: " + change);
        machine.resetBalance();

        machine.setState(machine.isSoldOut() ? machine.getSoldOutState() : machine.getNoCoinState());
    }
}

public class SoldOutState implements VendingMachineState {
    @Override
    public void insertCoin(VendingMachine machine, int coinValue) {
        System.out.println("Machine is sold out. Coin rejected.");
    }

    @Override
    public void selectProduct(VendingMachine machine, String productCode) {
        System.out.println("Machine is sold out.");
    }

    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Machine is sold out. Cannot dispense.");
    }
}
```

```java
public class Product {
    private final String code;
    private final String name;
    private final int price;
    private int quantity;

    public Product(String code, String name, int price, int quantity) {
        this.code = code;
        this.name = name;
        this.price = price;
        this.quantity = quantity;
    }

    public String getCode() { return code; }
    public String getName() { return name; }
    public int getPrice() { return price; }
    public int getQuantity() { return quantity; }
    public void decrementQuantity() { quantity--; }
}
```

```java
public class VendingMachine {
    private final Map<String, Product> inventory = new HashMap<>();
    private int balance = 0;
    private String selectedProductCode;

    private final VendingMachineState noCoinState = new NoCoinState();
    private final VendingMachineState hasCoinState = new HasCoinState();
    private final VendingMachineState dispensingState = new DispensingState();
    private final VendingMachineState soldOutState = new SoldOutState();

    private VendingMachineState currentState = noCoinState;

    public void addProduct(Product product) {
        inventory.put(product.getCode(), product);
    }

    // The ENTIRE public transaction surface. Neither method contains a single
    // if/else or switch on what state the machine is in — both delegate immediately.
    public void insertCoin(int coinValue) {
        currentState.insertCoin(this, coinValue);
    }

    public void selectProduct(String productCode) {
        currentState.selectProduct(this, productCode);
    }

    public void setState(VendingMachineState state) { this.currentState = state; }
    public VendingMachineState getState() { return currentState; }
    public VendingMachineState getNoCoinState() { return noCoinState; }
    public VendingMachineState getHasCoinState() { return hasCoinState; }
    public VendingMachineState getDispensingState() { return dispensingState; }
    public VendingMachineState getSoldOutState() { return soldOutState; }

    public void addBalance(int amount) { balance += amount; }
    public void resetBalance() { balance = 0; }
    public int getBalance() { return balance; }

    public Map<String, Product> getInventory() { return inventory; }
    public void setSelectedProductCode(String code) { this.selectedProductCode = code; }
    public String getSelectedProductCode() { return selectedProductCode; }

    public boolean isSoldOut() {
        for (Product product : inventory.values()) {
            if (product.getQuantity() > 0) {
                return false;
            }
        }
        return true;
    }
}
```

### Worked Trace — Full Purchase Flow, Including an Out-of-Stock Detour

Machine stocked with `Soda` (price 25, qty 1) and `Chips` (price 30, qty 2). Start: `NoCoinState`, balance 0.

```
insertCoin(50)
  → NoCoinState.insertCoin: balance=50, state→HasCoinState
selectProduct("SODA")
  → HasCoinState.selectProduct: Soda found, qty=1>0, balance(50)>=price(25) ✓
    selectedProductCode="SODA", state→DispensingState, dispense() called immediately
  → DispensingState.dispense: Soda.decrementQuantity() → qty=0
    change = 50-25 = 25. Print "Dispensing: Soda. Change returned: 25". balance→0
    isSoldOut()? Chips qty=2>0 → false → state→NoCoinState

insertCoin(50)
  → NoCoinState.insertCoin: balance=50, state→HasCoinState
selectProduct("SODA")
  → HasCoinState.selectProduct: Soda found, qty=0 → "out of stock" → state STAYS HasCoinState, balance STAYS 50
selectProduct("CHIPS")
  → HasCoinState.selectProduct: Chips found, qty=2>0, balance(50)>=price(30) ✓
    selectedProductCode="CHIPS", state→DispensingState, dispense() called immediately
  → DispensingState.dispense: Chips.decrementQuantity() → qty=1
    change = 50-30 = 20. Print "Dispensing: Chips. Change returned: 20". balance→0
    isSoldOut()? Chips qty=1>0 → false → state→NoCoinState
```

**What this trace actually proves, beyond "the happy path works":** the out-of-stock detour shows `HasCoinState` correctly staying active with the balance intact when a selection fails — the balance isn't silently lost, and a *different*, in-stock product can still be purchased with the same money, without ever re-inserting a coin. That behavior falls out of the state design directly (an out-of-stock or insufficient-balance selection simply doesn't call `setState`), not from any special-cased handling.

**Complexity:** every state transition is `O(1)` — swapping which object `currentState` points to is a single reference assignment, unrelated to catalog size. `isSoldOut()` is `O(k)` where `k` is the number of distinct products in the catalog — it must check every product's quantity, since any one of them still being in stock changes the answer.

> 🔑 **Key Takeaway — the "no giant if-else" constraint, checked directly:** `VendingMachine.insertCoin()` and `selectProduct()` are the machine's *entire* public transaction surface, and neither contains a single conditional about what state the machine is currently in — both immediately delegate to `currentState`. Every rule about what's allowed in which phase lives inside the four state classes, each one small and focused (high cohesion, directly reusing Day 108's lens) on exactly its own phase's rules.

> ⚠️ **Common Mistake:** creating a *new* state object on every transition (`machine.setState(new NoCoinState())`) instead of reusing the pre-built singleton fields (`machine.getNoCoinState()`). The four state classes here hold no per-transaction data of their own — they're stateless with respect to any individual purchase — so re-instantiating them on every transition wastes allocation for no behavioral benefit; treating them as fixed, reusable singletons (created once, referenced repeatedly) is the correct, idiomatic version of this pattern whenever the state classes themselves don't need to carry data.

---

# Part 3 — Mock Interview #1

**Format:** 45 minutes, with the accountability partner from Day 107's scheduling. One of you interviews, using **Tic-Tac-Toe** (Day 108) as the subject — the interviewer should *not* simply hand over Day 108's design; the value is in re-deriving it live, under a clock, the way an actual interview works.

**Suggested time allocation**, worth stating to the interviewer up front so both sides know what's coming:

- **~5 min — Requirements.** The candidate asks clarifying questions; the interviewer answers as a real interviewer would (a decision already made, not "whatever you think is best").
- **~10 min — Objects and class diagram.** Talked through out loud and sketched, not written as final code yet.
- **~25 min — Code the core.** Live implementation, talking through decisions as they're made.
- **~5 min — Wrap-up.** Interviewer gives direct feedback; candidate asks any questions.

**After the mock, alone, answer honestly — the plan's specific checkpoint:** *did you clarify requirements first, before designing anything?* If the honest answer is no — if the board size, win conditions, or scope got assumed rather than asked — that's exactly Day 106's Step 1 failure mode, now caught in a live setting rather than a reading exercise, which is the entire reason this mock exists this early and against a system you've already built once.

**Two more worth answering honestly, beyond the plan's one:**
- Did the class diagram actually get referenced while coding, or abandoned the moment typing started?
- Was there a moment a pattern got reached for out of habit rather than because a requirement called for it?

---

# Part 4 — Theory: State vs. Strategy (1 hr)

Both patterns share the *exact* same code shape — a context class holding a reference to an interface it delegates to, with multiple interchangeable implementations behind that interface. Structurally, `VendingMachine.currentState` and yesterday's `Checkout.discountStrategy` look almost identical. The difference is intent, not mechanism.

**The two-sentence version, as the plan asks for directly:** State and Strategy share the same code shape — a context delegating to an interface it holds a reference to — but differ in who drives the swap and why. In State, the object's *own* operations trigger transitions between a small, closed set of internal phases, and the states typically know about and reference each other; in Strategy, an *external client* deliberately chooses which algorithm to plug in, that choice doesn't emerge from the object's own behavior, and the strategies are typically unaware of each other's existence.

**A second, practical tell, worth noticing in the actual code already written this week:** State implementations here (`NoCoinState`, `HasCoinState`, ...) carry **no constructor parameters** — they're stateless with respect to any individual transaction, which is exactly why they can be safely reused as singletons. Yesterday's `PercentageDiscount(0.10)` **does** take a constructor parameter — a strategy instance is frequently configured with data specific to that particular choice. Not a universal rule (a parameterless Strategy or a stateful State both exist), but a genuinely useful, checkable signal when a design is ambiguous: if the candidate implementations need their own configuration data, that leans Strategy; if they're interchangeable, parameterless behaviors that mainly know how to hand off to *each other*, that leans State.

| | State | Strategy |
|---|---|---|
| Who chooses the active implementation? | The object itself, via its own operations | An external client |
| Do implementations reference each other? | Usually yes (`HasCoinState` creates/points to `DispensingState`) | Usually no |
| Typical constructor shape | Often parameterless (stateless singletons) | Often carries configuration data |
| Client's mental model | "This object is *in* a phase" | "This object was *given* a policy" |

> 💡 **Interview Insight:** if a design genuinely could be read either way, that's worth *saying* out loud rather than silently picking one — "this could be modeled as State if transitions are driven internally, or Strategy if the caller explicitly selects the mode; here I'm treating it as State because..." demonstrates the actual distinction is understood, not just memorized as two separate vocabulary words.

---

# Project Block Guide (2.5 hrs)

**Repository:** `lld-java`. **Module:** new — `vending-machine/`.

**Task:** the full implementation above — `Product`, `VendingMachineState` and its four implementers, `VendingMachine`.

**Definition of done:**
- Full purchase flow works end-to-end: insert coin(s) → select product → dispense → change returned → correct next state (`NoCoinState` if stock remains, `SoldOutState` if the entire catalog is exhausted).
- **Zero conditionals dispatching on machine state** anywhere in `VendingMachine` — confirm this by rereading `insertCoin()`/`selectProduct()` and checking neither contains an `if`/`switch` referencing "what state am I in."
- At least one JUnit test covering the out-of-stock-then-successful-alternate-purchase flow from the worked trace above — this is the scenario most likely to be silently broken by a careless implementation (e.g., one that resets balance on a failed selection, which the design above deliberately does not do).
- Pushed to `lld-java/vending-machine/`.

---

# Career Block Guide (1 hr)

**LinkedIn engagement:** 20 minutes. **Networking:** continue replying to recruiter inbound, same cadence as established.

**Resume checkpoint.** Update the resume to reflect where the plan actually stands today — a real revision, not a placeholder note to "do this later":

- **Platform architecture**, described by what it demonstrates, not just a technology list: a Spring Boot service with JWT-secured endpoints, Token Bucket rate limiting, Kafka-based eventing with schema registry, Resilience4j circuit breakers around downstream calls, Feign clients, a Spring Cloud Gateway front door, Saga choreography for distributed transactions, deployed on Kubernetes with Helm and horizontal pod autoscaling.
- **LLD systems**, as their own line or bullet cluster — Tic-Tac-Toe and Vending Machine so far, phrased around the design skill demonstrated (state-driven behavior with zero conditional dispatch, applying a formal 5-step design process under time constraints) rather than just naming them.
- **A concrete bullet-writing pattern worth using directly:** action verb + what was built + the specific technique or technology + the outcome or property it produced. For example: *"Designed a Vending Machine's transaction flow using the State pattern, eliminating all state-dispatch conditionals from the core class and isolating each transaction phase's rules independently."* Specific enough to justify in an interview immediately if asked to elaborate — the actual test for whether a bullet is doing real work or just listing a buzzword.
- **On the 259-problem DSA practice figure specifically:** this isn't resume-bullet material on its own (raw problem counts read as noise to most reviewers) — it's better represented indirectly, through a pinned GitHub repo (`dsa-java`) a resume can link to, or as brief context if directly asked in an interview about prep process.

---

# Day 109 — Interview Questions

**Q1. What's the two-sentence distinction between State and Strategy?** They share the same code shape — a context delegating to an interface reference — but differ in who drives the swap: State's transitions are triggered by the object's own operations between a small, closed set of internal phases, with states often referencing each other; Strategy's choice is made deliberately by an external client, and strategies are typically unaware of one another.

**Q2. Give the practical constructor-shape tell for distinguishing an ambiguous State-or-Strategy design.** State implementations are often parameterless, stateless, and safely reusable as singletons; Strategy implementations frequently carry their own configuration data in the constructor. Not universal, but a useful, checkable signal when the intent alone doesn't settle it.

**Q3. Why are `VendingMachine`'s four state objects created once as fields, rather than `new`'d on every transition?** They hold no per-transaction data — they're stateless with respect to any individual purchase — so re-instantiating them on every transition allocates memory for no behavioral benefit; reusing fixed singleton instances is the idiomatic version of State whenever the state classes carry no data of their own.

**Q4. Trace what happens if `selectProduct` is called while the machine is in `SoldOutState`.** `SoldOutState.selectProduct` runs — it prints "Machine is sold out" and performs no state change, no balance change, and no dispense. The call is fully absorbed by the current state object; `VendingMachine` itself does nothing state-specific at all.

**Q5. Why does a failed selection (out of stock, or insufficient balance) leave the machine in `HasCoinState` rather than reverting to `NoCoinState`?** The customer's balance is still present and still valid — reverting to `NoCoinState` would either strand that balance or force it to be re-entered; staying in `HasCoinState` correctly allows a different, valid selection using the same already-inserted money.

**Q6. Why does `HasCoinState.selectProduct` transition to `DispensingState` and then immediately call `dispense()` on it, rather than letting a later call trigger dispensing?** Once payment is validated as sufficient, dispensing isn't a separate customer-triggered action — it's a direct, immediate consequence of a successful selection, so the transition and the action happen together rather than waiting for a call that doesn't correspond to anything the customer does.

**Q7. In Longest Substring Without Repeating Characters, why must the duplicate-index check include `>= left`, not just check whether the character was ever seen before?** A character seen earlier but **outside** the current window (before `left`) isn't actually a duplicate *within* the live window — treating it as one would incorrectly move `left` backwards, corrupting the window and producing a wrong (too-small or logically inconsistent) answer.

**Q8. Why is the two-pointer scan for LC 3 O(n) overall rather than O(n²), given `left` moves inside the loop that `right` also drives?** `left` only ever moves forward, never backward, across the algorithm's *entire* run — so even though it moves inside `right`'s loop, the total number of steps `left` takes across every iteration combined is bounded by `n`, not by `n` per iteration; the two pointers together do a bounded `O(n)` amount of total work.

---

## Daily Deliverable Check

- [ ] Longest Substring Without Repeating Characters (LC 3) solved cold, without hints.
- [ ] Vending Machine complete: full purchase flow works via state transitions; zero state-dispatch conditionals in `VendingMachine` itself.
- [ ] Mock Interview #1 completed and debriefed, including the "did I clarify requirements first?" self-check.
- [ ] State vs. Strategy two-sentence write-up complete.
- [ ] Resume updated to reflect current platform architecture, LLD systems built, and prep progress.

---

## What Tomorrow Assumes You Already Know Cold

Day 110 assumes the full framework — now run live twice, against two real systems — no longer needs any explanation at all, only application; tomorrow moves straight into Parking Lot's own requirements with no framework recap. It also assumes today's State-vs-Strategy distinction is settled enough to recognize, by contrast, that Parking Lot needs **neither** — the pattern-recognition instinct being built this week is as much about correctly saying "no pattern here" (Day 108, and again tomorrow) as it is about applying one correctly (today).
