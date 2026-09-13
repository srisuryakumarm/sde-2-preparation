# Day 106 — The LLD Interview Framework, and Structural Patterns

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 105 Resource Book](Day105_Resource_Book.md)
**Next ▶:** [Day 107 Resource Book](Day107_Resource_Book.md)
**Companion to:** Day 106 of `Week_16_Revised.md`

---

## Recap

Day 105 closed the entire DSA phase of this series: 197 required + 53 extra = 249 distinct DSA problems across Weeks 1–15, plus a 10-problem SQL track — 259 problems total, every pattern from HashMap/HashSet through Sorting now assumed fully reflexive. Today opens something structurally different: **Low-Level Design (LLD)**, the phase this entire plan has been building toward since Week 1's very first `Shape`/`Circle`/`Rectangle` hierarchy.

Everything today needs is already in place:
- **Interfaces and abstract classes** (Week 1, Day 2) — every pattern below is, mechanically, a specific way of arranging interfaces, abstract classes, and concrete classes to solve one recurring kind of problem.
- **The OOP Four Pillars** (Week 1, Day 5) — encapsulation, inheritance, polymorphism, abstraction. Every pattern today leans on polymorphism specifically: code written against a shared interface, correctly invoking each concrete implementation's own behavior.
- **SOLID** (Week 1, Day 6), especially **Open/Closed** ("open for extension, closed for modification") and **Dependency Inversion** ("depend on abstractions, not concretions") — Week 15 Day 101 already used Open/Closed to formally distinguish Factory Method from Simple Factory; today's patterns are evaluated against the same principle throughout.
- **Recursion** (Week 2, Day 8) — Composite, below, is a direct application of the same base-case/recursive-case shape, just applied to a class hierarchy instead of a function call.
- **All three Creational patterns** (Week 15, Days 100–101) — Singleton, Factory Method, Builder, all implemented and tested in `lld-java/design-patterns`. Today's Structural patterns are a different category solving a different kind of problem (composing objects together, not creating them), but the repo, the module layout, and the "implement it, test it, push it" discipline all carry forward unchanged.

**One deliberate constraint, stated explicitly so it reads as a choice, not an oversight:** this series has never taught lambda expressions, method references, or functional interfaces (`Runnable` and `Comparator` have both been implemented via **anonymous inner classes** since Week 5, Day 29 — flagged explicitly at the time as a placeholder for lambdas specifically because lambdas hadn't been taught yet, and reconfirmed as still-true at Week 8, Day 54). That's still true today. Every pattern below is implemented the classic Gang-of-Four way — named concrete classes implementing named interfaces — which also happens to be exactly how the plan itself describes every pattern this week (Day 109's Vending Machine names `HasCoinState`, `NoCoinState`, `DispensingState`, `SoldOutState` as actual classes, not lambdas). If a future day needs lambda syntax, that becomes new material requiring its own from-zero treatment first, not something to reach for quietly because it would make the code shorter.

**A second adaptation, also worth stating up front:** the master brief for this series asks for time/space complexity "with the reasoning shown" for every concept. Big-O measures how cost grows with input size — that maps cleanly onto an algorithm, but most of what a design pattern adds is a small, *fixed* number of indirection hops, not a function of `n`. So this week, "complexity" is handled two ways: where a pattern genuinely wraps a traversal or algorithm (Composite walking a tree, Parking Lot's spot search next week), real Big-O is given, derived the same way it always has been. Where a pattern is just re-wiring how a fixed handful of objects talk to each other, the actual trade-off discussed is **structural cost** — how many extra classes and indirection layers the pattern adds, weighed against the coupling, duplication, or rigidity it removes. That's this week's real currency, stated explicitly rather than silently substituted.

---

## Learning Objectives

By the end of today, without notes:

1. Recite the 5-step LLD interview framework and name the four common mistakes it's designed to prevent.
2. Name the three GoF pattern categories (Creational, Structural, Behavioral) and explain what distinguishes them.
3. For each of Adapter, Decorator, Facade, Proxy, and Composite: explain the mechanism, state the concrete requirement-language signal that calls for it, and name its nearest alternative and why the pattern beats it.
4. Implement the Decorator pattern from scratch (Coffee pricing) with at least three stacked decorators, and explain why `cost()` resolving correctly is the same call-stack mechanism as Day 8's recursion.
5. Sketch the Composite pattern for a file system and explain why a composite's `getSize()` is structurally identical to a tree's postorder DFS combine step.

---

## Concept Dependency Map

```
Week 1 Day 2:  interfaces, abstract classes, inheritance, "this"
Week 1 Day 5:  OOP Four Pillars — encapsulation / inheritance / polymorphism / abstraction
Week 1 Day 6:  SOLID — Open/Closed, Dependency Inversion (+ 3 others)
Week 2 Day 8:  Recursion — base case, recursive case, call stack
Week 15 D100:  Singleton — private ctor + static accessor, volatile, DCL
Week 15 D101:  Factory Method (Simple Factory vs. true GoF) · Builder (immutability)
        │
        ▼
Today: GoF's 3-category taxonomy — Creational / Structural / Behavioral (NEW, named explicitly)
        │
        ▼
The LLD Interview Framework (5 steps) — reused for every remaining system this phase
        │
        ▼
Structural Patterns — "how objects are wired together":
  Adapter    (needs: interfaces — D2)
  Decorator  (needs: interfaces — D2; composition-over-inheritance instinct)
  Facade     (needs: nothing new — composition wrapper only)
  Proxy      (needs: interfaces — D2; forwarding to a wrapped instance)
  Composite  (needs: interfaces — D2; recursion — D8)
        │
        ▼
Coffee Decorator, full implementation + JUnit tests (lld-java/design-patterns)
Composite sketch — Folder/File file system (no full implementation required)
        │
        ▼
🔗 Forward: Day 108's Tic-Tac-Toe applies the 5-step framework end to end for the
   first time. Day 110 gives Composition-over-Inheritance its own dedicated day.
```

---

# Part 1 — The LLD Interview Framework

Every LLD system for the rest of this plan — ten of them, across this week and next — gets built with the same five steps, in the same order. Today is the only day with room to explain *why* each step exists, rather than just naming it, because from Day 108 onward the framework is assumed and applied, not re-taught.

### Step 1 — Clarify Requirements (5–10 min)

Ask what's actually in scope before designing anything. A Parking Lot LLD might or might not need payment processing, multiple vehicle types, or multi-level support — none of that is implied by the two words "parking lot." **This is the single most skipped step, and skipping it is the most common reason a candidate builds the wrong system correctly** — a well-engineered, cleanly-coded solution to a problem the interviewer didn't actually ask.

**Why it matters mechanically, not just as advice:** every one of the next four steps depends on scope being fixed first. Step 2 turns requirements into classes — if the requirements are wrong, the classes are wrong, and everything built on top of them (Step 3's relationships, Step 4's patterns, Step 5's code) inherits that error. There's no later step that corrects a wrong assumption made here; it just compounds.

> 💡 **Interview Insight:** treat this step as a real conversation, not a formality to rush through. Asking two or three sharp, scoping questions *and listening to the answers* signals more seniority than diving straight into a class diagram — it's the single cheapest way to demonstrate you think like someone who's shipped systems that had to survive contact with actual, changing requirements.

### Step 2 — Identify Core Objects

Turn the nouns in the requirements into candidate classes; verbs often become methods on those classes. A "Parking Spot," a "Vehicle," a "Ticket" — each becomes a class. This is a direct, practical extension of Week 1 Day 2's very first exercise (`Book`, `Shape`/`Circle`/`Rectangle`) — the mechanism ("what are the nouns, what state does each one own") hasn't changed; what's new is doing it live, under time pressure, against requirements someone just described verbally rather than a pre-written problem statement.

### Step 3 — Define Relationships, and Draw a Class Diagram

Composition vs. inheritance, associations, multiplicities. This doesn't need to be UML-perfect — a legible sketch showing the classes, their key fields, and how they relate is enough. **Composition vs. inheritance gets its own full dedicated day (Day 110)**, once Parking Lot gives a concrete, motivating case where getting this choice wrong actually hurts; today, it's enough to know the distinction exists and that it's a deliberate choice made at this step, not an afterthought.

### Step 4 — Apply Patterns Deliberately, Not Decoratively

A pattern should solve a specific requirement — multiple payment strategies calls for Strategy (Week 17); a vending machine's mode-dependent behavior calls for State (Day 109) — not be bolted on to demonstrate you know its name. This is the step today's five Structural patterns, and next week's five Behavioral patterns, exist to feed: a working mental catalog of "what problem does each one actually solve," so that in the moment, a requirement maps to a pattern instead of a pattern being forced onto whatever's on screen.

> ⚠️ **Common Mistake:** naming a pattern before it's earned. If a class diagram already cleanly solves the requirement, adding a pattern on top adds indirection with nothing to show for it — more classes, more hops, no requirement actually served. Every pattern below states its own concrete trigger for exactly this reason: the trigger is what makes "deliberately" checkable instead of a vibe.

### Step 5 — Code the Core

You won't finish every method in a real interview; prioritize the class skeletons, the key relationships, and one or two representative method bodies that show you can actually implement what you designed.

**Why this step matters more than it might seem, concretely:** Atlassian's actual LLD round ("Code Design") requires a fully working solution, not a diagram — and their "Data Structures" round explicitly isn't traditional LeetCode; it's a real-world scenario where the choice has to be justified out loud. Every system from here forward in this plan gets built to genuinely compile and run, not sketched and abandoned once the class diagram looks right. A design that only exists as boxes and arrows hasn't actually been tested against the one thing that reliably surfaces a bad design: trying to make it compile.

### The Four Mistakes to Actively Avoid

| Mistake | Why it's a mistake |
|---|---|
| Designing for scale | That's a System Design question (Week 18 onward), not LLD. Sharding, load balancers, and replica counts don't belong in a class diagram for a single-process system. |
| Over-engineering with unwarranted patterns | Every pattern added without a requirement behind it is indirection with no payoff — see Step 4's Common Mistake above. |
| Skipping requirements clarification and guessing wrong | Compounds through every later step, as explained under Step 1. |
| Spending all your time on one class while the rest stay unsketched | An interviewer scores breadth of design *and* depth of implementation; a beautifully complete `Vehicle` class next to four unsketched empty boxes reads as poor time management, not thoroughness. |

---

# Part 2 — Design Patterns: The Three GoF Categories

**"GoF"** — the Gang of Four (Gamma, Helm, Johnson, Vlissides) — has been used as a label since Week 15 Day 101 (`Simple Factory... not one of the 23 GoF patterns`) without the full taxonomy behind it being named. Naming it now: their 1994 catalog splits its 23 patterns into three categories, by **what kind of problem the pattern solves**:

- **Creational** (Week 15) — how objects get *created*. Singleton controls how many instances exist; Factory Method and Builder control *how* construction happens without the caller needing to know the concrete type or assemble a complex object by hand.
- **Structural** (today) — how objects are *composed* into larger structures. Every pattern below takes two or more existing pieces and wires them together — adapting one interface to another, wrapping an object to add behavior, hiding a subsystem behind one simpler interface, standing in front of an object to control access to it, or treating a whole tree of objects uniformly.
- **Behavioral** (Day 107) — how objects *communicate and assign responsibility* at runtime — who talks to whom, in what order, and how behavior can vary without an if/else chain naming every case. Previewed only by name today; full treatment tomorrow.

**Why this taxonomy is worth having explicitly, not just as trivia:** it's the fastest sanity check for Step 4's "deliberately, not decoratively" rule. If a requirement is about *how something gets built*, reach into Creational. If it's about *how existing pieces fit together*, reach into Structural. If it's about *how objects react to each other or vary their own behavior*, reach into Behavioral. A pattern pulled from the wrong category for the problem at hand is usually a sign the requirement wasn't actually understood — which loops straight back to Step 1.

---

# Part 3 — Structural Patterns

All five below take existing pieces and wire them together differently. Each section covers the mechanism, why it works, the concrete signal that calls for it, its nearest alternative and why the pattern wins, and its real (structural or algorithmic) cost.

---

## Adapter

**What it is:** Adapter converts one interface into another that a client expects, bridging two interfaces that were never designed to work together, without modifying either one.

**Mechanism:** a new class implements the interface the client expects, and internally holds — and delegates to — an instance of the incompatible class, translating each call.

```java
// The interface the rest of the application already depends on.
public interface NotificationService {
    void send(String recipient, String message);
}

// A third-party or legacy class this application does not own and cannot change —
// its method signature and its units (a numeric priority) don't match NotificationService at all.
public class LegacySmsGateway {
    public void dispatchSms(String phoneNumber, String body, int priorityLevel) {
        // ... vendor's actual wire protocol, not our concern here
        System.out.println("Dispatched via legacy gateway: " + phoneNumber + " | " + body + " | priority=" + priorityLevel);
    }
}

// The Adapter: implements the interface the app expects, wraps the class it doesn't.
public class SmsGatewayAdapter implements NotificationService {
    private static final int DEFAULT_PRIORITY = 5;
    private final LegacySmsGateway legacyGateway;

    public SmsGatewayAdapter(LegacySmsGateway legacyGateway) {
        this.legacyGateway = legacyGateway;
    }

    @Override
    public void send(String recipient, String message) {
        legacyGateway.dispatchSms(recipient, message, DEFAULT_PRIORITY);
    }
}
```

Any code that depends on `NotificationService` can now be handed a `SmsGatewayAdapter` and never needs to know `LegacySmsGateway` exists.

**Why it works:** this is Dependency Inversion (Week 1, Day 6) applied directly — the rest of the application depends on the `NotificationService` abstraction, never on the concrete `LegacySmsGateway`. The adapter is the *only* class that knows both sides exist.

**When to reach for it — the concrete signal:** a requirements conversation that includes "we need to integrate with [an existing/third-party/legacy system] whose interface doesn't match what our code already expects." If nothing incompatible needs bridging, there's nothing for Adapter to do.

**Trade-off against the nearest alternative:** the alternative is modifying the incompatible class directly — usually impossible (a third-party JAR, a legacy class other teams also depend on) or actively undesirable (changing it could break every other caller). Adapter's cost is one small new class; its payoff is touching zero existing code on either side.

**Complexity:** the adapter method does a fixed, bounded amount of translation work per call — O(1), not data-size-dependent. The interesting cost here is structural, not algorithmic: exactly one new class per incompatible interface being bridged.

> ⚠️ **Common Mistake:** writing an adapter that leaks the wrapped class's types back out through its own return values (e.g., a method that's supposed to return the app's own `Notification` type but returns a `LegacySmsGateway`-specific result object instead). That defeats the entire point — callers are back to depending on the legacy type indirectly.

---

## Decorator

**What it is:** Decorator adds behavior to an individual object dynamically, by wrapping it, instead of needing a new subclass for every possible combination of behaviors.

**Mechanism:** a decorator implements the same interface as the object it wraps, holds a reference to the wrapped object, and adds its own behavior *before or after* delegating to it.

**When to reach for it — the concrete signal:** a requirement that behaviors need to **combine**, in a caller-chosen combination, without the number of subclasses exploding. "A coffee can have any combination of milk, caramel, and whip" is exactly this — three optional add-ons naively subclassed would need up to eight classes (`EspressoWithMilk`, `EspressoWithMilkAndCaramel`, ...) just to name every combination; Decorator needs exactly one class per add-on, composed at runtime.

**Trade-off against the nearest alternative:** the alternative is subclassing per combination. Decorator trades that combinatorial explosion for a small, fixed number of wrapper classes (one per add-on, not one per *combination* of add-ons) plus the run-time cost of the wrapping itself.

### Full Implementation — Pricing a Coffee

```java
// The common interface every component — base drink AND every decorator — implements.
public interface Beverage {
    String getDescription();
    double getCost();
}

// The concrete component being decorated.
public class Espresso implements Beverage {
    @Override
    public String getDescription() {
        return "Espresso";
    }

    @Override
    public double getCost() {
        return 1.99;
    }
}

// The abstract decorator: implements Beverage (so it's substitutable for one), and
// holds — wraps — another Beverage. Concrete decorators extend this and add their own cost/description.
public abstract class CondimentDecorator implements Beverage {
    protected final Beverage wrappedBeverage;

    protected CondimentDecorator(Beverage wrappedBeverage) {
        this.wrappedBeverage = wrappedBeverage;
    }
}

public class Milk extends CondimentDecorator {
    public Milk(Beverage wrappedBeverage) {
        super(wrappedBeverage);
    }

    @Override
    public String getDescription() {
        return wrappedBeverage.getDescription() + ", Milk";
    }

    @Override
    public double getCost() {
        return wrappedBeverage.getCost() + 0.50;
    }
}

public class Caramel extends CondimentDecorator {
    public Caramel(Beverage wrappedBeverage) {
        super(wrappedBeverage);
    }

    @Override
    public String getDescription() {
        return wrappedBeverage.getDescription() + ", Caramel";
    }

    @Override
    public double getCost() {
        return wrappedBeverage.getCost() + 0.75;
    }
}

public class Whip extends CondimentDecorator {
    public Whip(Beverage wrappedBeverage) {
        super(wrappedBeverage);
    }

    @Override
    public String getDescription() {
        return wrappedBeverage.getDescription() + ", Whip";
    }

    @Override
    public double getCost() {
        return wrappedBeverage.getCost() + 0.60;
    }
}
```

```java
public static void main(String[] args) {
    Beverage order = new Espresso();
    order = new Milk(order);
    order = new Caramel(order);
    order = new Whip(order);

    System.out.println(order.getDescription() + " = $" + order.getCost());
    // Espresso, Milk, Caramel, Whip = $3.84
}
```

### Worked Trace — Why `getCost()` Resolves Correctly

`order` after the three wraps is a `Whip` holding a `Caramel` holding a `Milk` holding an `Espresso`. Calling `order.getCost()`:

```
Whip.getCost()
  = wrappedBeverage.getCost() + 0.60        // wrappedBeverage is the Caramel
  = ( Caramel.getCost() )      + 0.60
  = ( wrappedBeverage.getCost() + 0.75 ) + 0.60   // wrappedBeverage is the Milk
  = ( ( Milk.getCost() )       + 0.75 ) + 0.60
  = ( ( wrappedBeverage.getCost() + 0.50 ) + 0.75 ) + 0.60   // wrappedBeverage is the Espresso
  = ( ( 1.99 + 0.50 ) + 0.75 ) + 0.60
  = 3.84
```

**🔗 This is exactly Day 8's recursion mechanism, just applied to object composition instead of function calls.** Each decorator's `getCost()` is a "recursive case" — it doesn't know or compute the total itself; it asks the object it wraps for *that* object's cost first, then adds its own increment. `Espresso.getCost()` is the "base case" — it returns a fixed value with no further delegation. The call chain unwinds exactly the way Day 8's `fib(5)` call tree unwound: work happens on the way back up, not the way down.

**Complexity:** calling `getCost()` on a stack of `k` decorators does `O(k)` work — one hop per layer, each doing O(1) of its own work before delegating. Space is also `O(k)`, both for the chain of wrapper objects and for the call stack unwinding through it. `k` here is "how many condiments were added," not the size of any input collection — small and interview-irrelevant in practice, but worth being able to state precisely rather than waving at "it's fast."

> ⚠️ **Common Mistake:** forgetting that `wrappedBeverage` must be typed as the **interface** (`Beverage`), not a concrete class. Typing it as `Espresso` would make it impossible to wrap a `Milk`-wrapped `Espresso` in a `Caramel` — decorators need to wrap *anything* that satisfies `Beverage`, including other decorators, which is exactly what makes stacking possible at all.

> 💡 **Interview Insight:** when asked "why not just add a `hasMilk`, `hasCaramel`, `hasWhip` boolean to `Espresso` and branch on them in `getCost()`," the strong answer names the actual cost of that alternative: every new condiment means editing `Espresso` again (violates Open/Closed — Week 1, Day 6), and the branching logic for "cost so far" lives in one increasingly complicated method instead of being distributed one small, obvious increment per decorator.

### README Note — Decorator vs. Simple Subclassing (Project Block Deliverable)

> **When to prefer Decorator over subclassing:** subclassing is the right call when there's a small, fixed, *closed* set of variants known ahead of time (a `SavingsAccount` and a `CheckingAccount` really are two distinct, enumerable kinds of `Account` — Week 1, Day 5). Decorator is the right call when behaviors need to combine *in combinations the class's author can't enumerate in advance* — a customer choosing any subset of milk/caramel/whip is exactly that case. The tell: if naming every valid combination as its own subclass would require `2^n` classes for `n` optional behaviors, that's Decorator's signal, not subclassing's.

---

## Facade

**What it is:** Facade provides one simplified interface over a complex subsystem with many moving parts underneath.

**Mechanism:** a facade class holds references to the subsystem's classes and exposes a small number of high-level methods, each of which internally makes several calls into the subsystem in the right order.

```java
// The existing subsystem — real services this platform already has, unrelated to each other's interfaces.
public class InventoryService {
    public boolean reserveStock(String sku, int quantity) {
        System.out.println("Reserved " + quantity + " of " + sku);
        return true;
    }
}

public class PaymentService {
    public boolean charge(String customerId, double amount) {
        System.out.println("Charged " + customerId + " $" + amount);
        return true;
    }
}

public class ShippingService {
    public void scheduleShipment(String orderId, String address) {
        System.out.println("Scheduled shipment for " + orderId + " to " + address);
    }
}

public class NotificationService {
    public void notifyCustomer(String customerId, String message) {
        System.out.println("Notified " + customerId + ": " + message);
    }
}

// The Facade: one method, hiding the correct call order and the coordination between four services.
public class OrderFacade {
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final ShippingService shippingService;
    private final NotificationService notificationService;

    public OrderFacade(InventoryService inventoryService, PaymentService paymentService,
                        ShippingService shippingService, NotificationService notificationService) {
        this.inventoryService = inventoryService;
        this.paymentService = paymentService;
        this.shippingService = shippingService;
        this.notificationService = notificationService;
    }

    public boolean placeOrder(String customerId, String sku, int quantity, double amount, String address) {
        if (!inventoryService.reserveStock(sku, quantity)) return false;
        if (!paymentService.charge(customerId, amount)) return false;
        shippingService.scheduleShipment(sku + "-" + customerId, address);
        notificationService.notifyCustomer(customerId, "Your order is on its way!");
        return true;
    }
}
```

Every caller of `placeOrder(...)` never needs to know these four services exist separately, or that they must be called in this specific order.

**Why it works:** it's a direct application of Interface Segregation and encapsulation together — callers depend on one small surface instead of four larger ones, and the *coordination logic itself* (this order, this error handling) is written and owned in exactly one place instead of being duplicated at every call site that needs to place an order.

**When to reach for it — the concrete signal:** "a client needs to do X, and doing X correctly requires calling several existing subsystem classes, in a specific order, and most callers don't actually care about the individual steps." If a caller genuinely needs fine-grained control over the subsystem's individual pieces, Facade shouldn't be the *only* way in — it's an added convenience layer, not a wall that hides the subsystem entirely.

**Trade-off against the nearest alternative:** the alternative is exposing all four services directly to every caller. That couples every caller to the subsystem's *internal* structure and call order — if the correct sequence changes, every caller needs to change too. Facade's cost is one more class; its payoff is that the sequence lives in exactly one place.

**Complexity:** `O(k)` where `k` is the fixed number of subsystem calls the facade method makes (four, here) — not data-size-dependent, and typically small enough that this is a structural observation, not a real algorithmic one.

> 🔗 This is the same shape as `todo-api`'s service layer, going all the way back to Week 5, Day 34 — a controller talks to one service method, not directly to the repository and validation logic separately. Facade is that same idea, formalized and named.

---

## Proxy

**What it is:** Proxy controls access to another object while presenting the exact same interface as the real thing — logging every call, lazily creating an expensive object only when it's first actually needed, or checking permissions before letting a call through.

**Mechanism:** a proxy implements the same interface as the real object, holds a reference to it (or creates it lazily), and inserts its own logic before and/or after delegating.

**When to reach for it — the concrete signal:** a requirement to add a cross-cutting concern — access control, logging, caching, deferred/lazy construction — to an object **without changing that object's own code**, and ideally without every caller needing to know the concern was added at all.

**Trade-off against the nearest alternative:** the alternative is adding the concern directly inside the real object's own code. That violates Single Responsibility (the object now does its real job *and* logging/access-control/laziness) and can't be turned on or off independently per caller — every caller gets it, always, baked in. Proxy's cost is one more class implementing the same interface; its payoff is that the concern is addable and removable without touching the real object at all.

Two of the plan's three named variants, shown concretely:

### Virtual Proxy — Lazy Initialization

```java
public interface ProductImage {
    void display();
}

// Expensive to construct — imagine this loads a large file from disk or a network call.
public class RealProductImage implements ProductImage {
    private final String filename;

    public RealProductImage(String filename) {
        this.filename = filename;
        loadFromDisk();   // the expensive part
    }

    private void loadFromDisk() {
        System.out.println("Loading " + filename + " from disk (expensive)...");
    }

    @Override
    public void display() {
        System.out.println("Displaying " + filename);
    }
}

public class ProductImageProxy implements ProductImage {
    private final String filename;
    private RealProductImage realImage;   // null until genuinely needed

    public ProductImageProxy(String filename) {
        this.filename = filename;
    }

    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealProductImage(filename);   // the expensive construction, deferred to first use
        }
        realImage.display();
    }
}
```

Creating a `ProductImageProxy` is cheap and immediate; the expensive `RealProductImage` construction only happens the first time `display()` is actually called — and never happens at all if it isn't.

### Logging / Protection Proxy

```java
public interface UserService {
    User getUser(String userId);
}

public class RealUserService implements UserService {
    @Override
    public User getUser(String userId) {
        // ... real lookup logic
        return new User(userId, "Alice");
    }
}

public class LoggingUserServiceProxy implements UserService {
    private final UserService realService;

    public LoggingUserServiceProxy(UserService realService) {
        this.realService = realService;
    }

    @Override
    public User getUser(String userId) {
        System.out.println("Fetching user: " + userId + " at " + System.currentTimeMillis());
        User result = realService.getUser(userId);
        System.out.println("Fetched: " + result);
        return result;
    }
}
```

A **protection proxy** checking permissions before delegating follows this exact same shape — the only change is what happens between "receive the call" and "delegate to the real object": a permission check instead of a log line. Not shown separately since the mechanism is identical; only the inserted logic differs.

**Complexity:** virtual proxy is `O(1)` per call after the first (amortized — the expensive work happens exactly once, ever); logging and protection proxies add a fixed, small `O(1)` of overhead to every call. None of the three variants change the *asymptotic* cost of what they wrap — they add a constant amount of work around it.

> ⚠️ **Common Mistake:** confusing Proxy with Decorator, since both wrap an object behind the same interface. **The distinction is intent, not mechanism:** Decorator *adds new behavior/responsibility* the wrapped object never had (a cost, a description addition). Proxy *controls access* to behavior the wrapped object already fully has — the proxy's job is never to change *what* `display()` or `getUser()` fundamentally does, only to guard, delay, or observe the call to it.

> 💡 **Interview Insight:** if asked to distinguish Proxy from Adapter too — Adapter changes the *interface* (translates one shape of call into another); Proxy keeps the *same* interface and controls access to it. Adapter answers "these two don't speak the same language"; Proxy answers "these two speak the same language, but something needs to happen around the call."

---

## Composite

**What it is:** Composite lets you treat a single object and a group of objects through the same interface, which is what makes recursive tree structures — a file system, a UI component hierarchy — composable without client code needing to know whether it's holding one item or a whole subtree.

**Mechanism:** a common interface is implemented by both **leaf** objects (which have no children) and **composite** objects (which hold a collection of other objects implementing the same interface — leaves, other composites, or a mix). Any operation the interface defines is implemented trivially on leaves and *recursively* (delegate to each child, then combine) on composites.

**When to reach for it — the concrete signal:** the requirement itself describes a **recursive, whole-part structure** — "a folder can contain files and other folders," "a UI panel can contain buttons and other panels" — where client code needs to run the same operation (`getSize()`, `render()`) over the whole structure without special-casing "is this one item or a container of items."

**Trade-off against the nearest alternative:** the alternative is a flat collection plus `instanceof` branches wherever the structure is processed ("if this is a File, do X; if it's a Folder, recurse"). That violates Open/Closed directly — every new operation needs its own `instanceof` chain repeated at every call site, and every new type of node means revisiting every one of those chains. Composite's cost is a slightly less obvious class hierarchy up front; its payoff is that new operations are just new interface methods, and new node types are just new implementers — neither one touches existing client code.

### Sketch — A File System

```java
// Named FileItem rather than File to avoid confusion with java.io.File.
public interface FileSystemComponent {
    long getSize();
    String getName();
    void print(String indent);
}

// LEAF — no children, size is intrinsic.
public class FileItem implements FileSystemComponent {
    private final String name;
    private final long sizeInBytes;

    public FileItem(String name, long sizeInBytes) {
        this.name = name;
        this.sizeInBytes = sizeInBytes;
    }

    @Override
    public long getSize() {
        return sizeInBytes;   // base case — no delegation needed
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public void print(String indent) {
        System.out.println(indent + "- " + name + " (" + sizeInBytes + " bytes)");
    }
}

// COMPOSITE — holds a mix of FileItems and other Folders, all as FileSystemComponent.
public class Folder implements FileSystemComponent {
    private final String name;
    private final List<FileSystemComponent> children = new ArrayList<>();

    public Folder(String name) {
        this.name = name;
    }

    public void add(FileSystemComponent component) {
        children.add(component);
    }

    @Override
    public long getSize() {
        long total = 0;
        for (FileSystemComponent child : children) {
            total += child.getSize();   // recursive case — a Folder child recurses further; a FileItem returns directly
        }
        return total;
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public void print(String indent) {
        System.out.println(indent + "+ " + name + "/");
        for (FileSystemComponent child : children) {
            child.print(indent + "  ");
        }
    }
}
```

```java
Folder root = new Folder("project");
Folder src = new Folder("src");
src.add(new FileItem("Main.java", 1200));
src.add(new FileItem("Utils.java", 800));

Folder docs = new Folder("docs");
docs.add(new FileItem("README.md", 400));

root.add(src);
root.add(docs);
root.add(new FileItem(".gitignore", 50));

System.out.println(root.getSize());   // 2450 — sums the entire subtree, arbitrarily deep
```

**🔗 `getSize()` is exactly Week 7 Day 46's Tree DFS postorder-combine shape, applied to a variable-arity tree instead of a strictly-binary one.** A `FileItem` is a leaf — like a `null` child, it returns a base value directly with no further recursion. A `Folder` is an internal node — like `Maximum Depth of Binary Tree`'s combine step, it doesn't compute its own answer directly; it asks every child for *that child's* answer first, then combines them (there, `1 + max(left, right)`; here, `sum(children)`). The only structural difference from a binary tree is that a `Folder` can have any number of children, not exactly two — the recursive shape (leaf returns directly, internal node recurses-then-combines) is identical.

**Complexity:** `getSize()` visits every node in the subtree exactly once — `O(n)` where `n` is the total number of files and folders under the root, the same bound as any tree traversal. Space is `O(h)` for the recursion's call stack, where `h` is the tree's depth (a deeply nested folder structure costs more stack space than a shallow, wide one holding the same total node count).

> ⚠️ **Common Mistake:** giving `FileItem` (the leaf) a `List<FileSystemComponent> children` field it never uses, "just in case," or giving it an `add()` method that does nothing or throws. A leaf that pretends to support children either wastes memory on an always-empty collection or introduces a confusing runtime failure mode where calling `add()` silently does nothing — worse than simply not implementing `add()` on the leaf-only interface at all. (In a fuller treatment, `add()` would live only on `Folder`, not on the shared `FileSystemComponent` interface, precisely to make this impossible at compile time rather than catching it at runtime — worth naming as the more rigorous version even though this sketch keeps `add()` on `Folder` only for the same reason.)

---

## 🔑 Key Takeaway — Telling the Five Apart

All five wire existing objects together, but each solves a genuinely different problem. This is the fast lookup for Step 4's "deliberately, not decoratively":

| Pattern | Solves | One-line signal |
|---|---|---|
| **Adapter** | Two interfaces don't match | "Integrate with something whose interface we don't control." |
| **Decorator** | Behaviors need to combine freely | "Any combination of optional add-ons, chosen by the caller." |
| **Facade** | A subsystem is complex to use correctly | "Several existing classes, called in a specific order, hidden behind one call." |
| **Proxy** | Access to an object needs controlling | "Same interface as the real thing, but add logging/lazy-init/permission checks around it." |
| **Composite** | A structure is recursively whole-part | "A single item and a group of items need to be treated identically." |

---

# Project Block Guide (1.5 hrs)

**Repository:** `lld-java` (already initialized, Week 15 Day 100 — not a new repo). **Module:** `design-patterns` (already holding Factory Method and Builder from Day 101).

**Task:** the Coffee Decorator implementation above — `Beverage`, `Espresso`, `CondimentDecorator`, `Milk`, `Caramel`, `Whip` — plus the README note distinguishing Decorator from simple subclassing.

**Definition of done:**
- Pushed to `lld-java/design-patterns/decorator/`.
- Tested with **at least 3 stacked decorators** (the Espresso → Milk → Caramel → Whip chain above satisfies this).
- A JUnit test asserting the final cost and description are correct for at least two different stacking orders — worth verifying stacking order actually changes the *description* string (order-dependent) while the *total cost* stays the same regardless of order (addition is commutative) — a genuinely useful thing to confirm with a real test rather than assume:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class DecoratorTest {
    @Test
    void stackingThreeDecoratorsProducesCorrectCostAndDescription() {
        Beverage order = new Whip(new Caramel(new Milk(new Espresso())));
        assertEquals(3.84, order.getCost(), 0.001);
        assertEquals("Espresso, Milk, Caramel, Whip", order.getDescription());
    }

    @Test
    void costIsOrderIndependentButDescriptionIsNot() {
        Beverage a = new Whip(new Milk(new Espresso()));
        Beverage b = new Milk(new Whip(new Espresso()));
        assertEquals(a.getCost(), b.getCost(), 0.001);          // same total either order
        assertEquals("Espresso, Milk, Whip", a.getDescription());
        assertEquals("Espresso, Whip, Milk", b.getDescription()); // different string, same cost
    }
}
```

- Composite sketch (`FileSystemComponent`, `FileItem`, `Folder`) committed alongside it in the same module, under `composite/` — no full test suite required, matching the plan's own "sketch, no full implementation needed" framing for this part specifically.

---

# Career Block Guide (1 hr)

**LinkedIn — Post 20, the Month-3-ish recap.** A draft to adapt, not copy verbatim:

> Roughly three months into a structured SDE-2 prep plan, and today felt like a real inflection point: the entire DSA phase just closed. 259 problems solved across every major pattern — arrays and hashing through dynamic programming, graphs, tries, segment trees, and a full SQL practice track on top of the original scope.
>
> Alongside it, a real platform's been taking shape the whole time — not toy exercises, an actual Spring Boot service with JWT auth, rate limiting, Kafka event streams, Resilience4j circuit breakers, and a Kubernetes deployment with autoscaling, built incrementally, week by week.
>
> Today opens the next phase: Low-Level Design. Ten systems to design and build over the next two weeks — Parking Lot, Vending Machine, an elevator, a booking system — each one coded to actually compile and run, not just diagrammed. First one's already underway.
>
> Genuinely energized about this next stretch. More soon.

Post it, then move to networking: reply to any recruiter inbound from the past week.

---

# Day 106 — Interview Questions

**Q1. Walk through the 5-step LLD framework from memory.** Clarify requirements → identify core objects (nouns become classes, verbs become methods) → define relationships and sketch a class diagram (composition vs. inheritance, associations, multiplicities) → apply patterns deliberately, only where a specific requirement calls for one → code the core (skeletons, key relationships, one or two representative method bodies).

**Q2. Why is skipping requirements clarification specifically dangerous, rather than just a missed nicety?** Every later step builds directly on the assumed scope from Step 1 — wrong requirements produce wrong core objects, which produce a wrong class diagram, which gets wrong patterns applied to it. There's no later step that corrects the error; it only compounds.

**Q3. What are the three GoF pattern categories, and what distinguishes them?** Creational (how objects get created — Singleton, Factory Method, Builder), Structural (how objects are composed into larger structures — today's five), Behavioral (how objects communicate and vary their behavior at runtime — Observer, Strategy, State, Command, Template Method, Day 107).

**Q4. Adapter and Facade both wrap existing code behind a simpler interface. How do they differ?** Adapter bridges exactly two incompatible interfaces so one can stand in for the other — a 1-to-1 translation. Facade simplifies a *multi-class subsystem* into one smaller interface, coordinating several calls in the right order — a many-to-one simplification, not a translation between two shapes.

**Q5. Why can't `CondimentDecorator.wrappedBeverage` be typed as `Espresso` instead of `Beverage`?** Decorators need to wrap *anything* implementing `Beverage`, including other decorators — typing the field as the concrete `Espresso` class would make it impossible to wrap a `Milk`-decorated beverage in a `Caramel` decorator, which is exactly what makes stacking multiple decorators possible.

**Q6. Trace what `getCost()` returns for `new Caramel(new Espresso())`.** `Caramel.getCost()` calls `wrappedBeverage.getCost()` — the `Espresso` — which returns `1.99` directly (no further delegation, the base case); `Caramel` then adds its own `0.75`, returning `2.74`.

**Q7. Why is Decorator's `getCost()` resolution the same mechanism as Day 8's recursion, not just a similar-looking pattern?** Each decorator's `getCost()` is a recursive case — it delegates to the wrapped object for a partial answer before combining it with its own contribution — and the base component (`Espresso`) is the base case, returning a fixed value with no further delegation. The call chain unwinds exactly like a recursive call stack: work happens on the way back up.

**Q8. Distinguish Proxy from Decorator — both wrap an object behind the same interface.** Decorator adds genuinely new behavior or responsibility the wrapped object never had. Proxy controls access to behavior the object already fully has — logging, lazy construction, or permission checks around a call — without ever changing what the underlying operation fundamentally does.

**Q9. Why is a virtual (lazy) proxy's amortized cost O(1) per call, not O(1) for every call including the first?** The expensive construction happens exactly once, on the first call that actually needs the real object — every call after that just delegates to the already-constructed instance. "Amortized" specifically means the one expensive call is being averaged in with all the cheap ones that follow, not that the first call itself is cheap.

**Q10. Why does Composite's `getSize()` on a Folder need to call `getSize()` on each child rather than maintaining a running total field?** A running total field would need to be kept in sync every time any descendant, at any depth, changes size — a correctness burden with many chances to drift out of sync. Recomputing via recursive delegation is always correct by construction, at the cost of re-walking the subtree on every call; if that recomputation cost mattered in practice, caching with explicit invalidation would be the fix, not a naively-maintained running total.

**Q11. Why does `FileSystemComponent` not define an `add()` method, when `Folder` clearly needs one?** A leaf (`FileItem`) has no children by definition — giving it an `add()` method would either do nothing (silently swallowing a caller's mistake) or throw at runtime, when the type system could instead make "you can't add to a file" a compile-time fact by keeping `add()` off the shared interface and only on `Folder`.

**Q12. Give the concrete, requirement-level signal that should make you reach for each of the five Structural patterns.** Adapter — integrating with an interface you don't control. Decorator — behaviors that combine freely, in combinations the class author can't enumerate ahead of time. Facade — a multi-class subsystem that needs a specific call order hidden behind one entry point. Proxy — controlling access to an object without changing its own code. Composite — a recursive, whole-part structure that needs uniform treatment of single items and groups.

---

## Daily Deliverable Check

- [ ] Can recite the 5-step LLD framework and the four common mistakes without notes.
- [ ] Can explain the mechanism, trigger, nearest alternative, and cost for all five Structural patterns.
- [ ] Coffee Decorator implementation pushed to `lld-java/design-patterns/decorator/`, tested with at least 3 stacked decorators.
- [ ] README note on Decorator vs. simple subclassing written.
- [ ] Composite sketch (`FileSystemComponent`/`FileItem`/`Folder`) committed.
- [ ] LinkedIn Post 20 published.

---

## What Tomorrow Assumes You Already Know Cold

Day 107 opens Behavioral patterns directly on top of today's Structural foundation and the 3-category taxonomy — it assumes "Structural patterns compose existing objects; Behavioral patterns govern how objects communicate and vary behavior at runtime" is settled, not re-explained. It also assumes today's "no lambdas yet" constraint is still in force (Observer's subscriber list and Strategy-adjacent material tomorrow are both named-class implementations, exactly like today's decorators) and that the Coffee Decorator's wrap-and-delegate mechanism is fully reflexive, since Day 107 doesn't re-derive "an interface implemented by a wrapper that holds a reference to the wrapped/observed object" from scratch a second time.
