# Day 107 — Behavioral Patterns, and TDD Practice

**Series:** SDE-2 Interview Prep Resource Books
**Curriculum Map:** [00_Curriculum_Map.md](00_Curriculum_Map.md)
**◀ Previous:** [Day 106 Resource Book](Day106_Resource_Book.md)
**Next ▶:** [Day 108 Resource Book](Day108_Resource_Book.md)
**Companion to:** Day 107 of `Week_16_Revised.md`

---

## Recap

Yesterday closed out Structural patterns (Adapter, Decorator, Facade, Proxy, Composite) and named the third GoF category, Behavioral, without opening it. Today opens it. Everything from yesterday's foundation still holds — interfaces (Week 1, Day 2), the Four Pillars (Week 1, Day 5), SOLID (Week 1, Day 6), and the "no lambdas yet" constraint — plus one thing specific to today: yesterday's Decorator showed an object *wrapping* another object to add behavior; today's Observer shows an object *watching* another object and reacting to its changes, a genuinely different relationship, worth not conflating just because both involve one object holding a reference to another.

On the DSA side, today's two revision problems reach back to **Week 11, Day 71** (Course Schedule, Topological Sort) and **Week 12, Day 84** (Coin Change, 1D Unbounded Knapsack) — both fully taught, both now due for a cold, unaided re-solve as a spaced-repetition check, the first of five such checks this week.

---

## Learning Objectives

By the end of today, without notes:

1. For each of Observer, Strategy, State, Command, and Template Method: explain the mechanism, the concrete signal that calls for it, and its nearest alternative.
2. Explain the Red-Green-Refactor cycle and why each of its three phases exists, not just its name.
3. Build the Observer pattern (`WeatherStation` → multiple `Display` subscribers) test-first, from a failing test through to a passing, refactored implementation.
4. Solve Course Schedule (LC 207) and Coin Change (LC 322) cold, without hints, confirming both patterns are still genuinely reflexive rather than half-remembered.

---

## Concept Dependency Map

```
Week 1 Day 2:   interfaces (needed for every Behavioral pattern below)
Week 15 D100–101: Creational patterns — comparison target for today's taxonomy recap
Day 106:        Structural patterns; GoF's 3-category taxonomy; "wrap to add behavior" (Decorator)
Week 11 Day 71: Topological Sort — Kahn's Algorithm (today's Graph revision)
Week 12 Day 84: 1D DP — Unbounded Knapsack (today's DP revision)
        │
        ▼
Today: Behavioral Patterns — "how objects communicate, and vary behavior at runtime":
  Observer         (needs: interfaces — D2; "wrap and hold a reference" instinct — D106 Decorator)
  Strategy         (needs: interfaces — D2)
  State            (needs: interfaces — D2)
  Command          (needs: interfaces — D2)
  Template Method  (needs: abstract classes — D2; abstract-method-in-base-class — Week 15 D101 Factory Method)
        │
        ▼
Test-Driven Development — Red / Green / Refactor (NEW methodology, applied to Observer below)
        │
        ▼
WeatherStation → CurrentConditionsDisplay, StatisticsDisplay (lld-java, built test-first)
DataProcessor Template Method sketch
        │
        ▼
🔗 Forward: Day 109 gives State its full worked system (Vending Machine) and directly
   compares it against today's Strategy. Week 17 gives Strategy two full systems
   (Splitwise's Split types, Food Delivery's PartnerMatchingStrategy).
```

---

# Part 1 — Behavioral Patterns

Structural patterns (yesterday) answer "how are these objects wired together." Behavioral patterns answer a different question: **"how do these objects talk to each other, and how does an object's own behavior vary, without a growing if/else chain naming every case?"**

---

## Observer

**What it is:** Observer lets subscribers react to state changes in a subject without tight coupling — a Weather Station pushing updates to multiple Display screens, none of which the station needs to know about specifically.

**Mechanism:** a `Subject` maintains a list of `Observer`s it knows nothing about beyond a shared interface. When the subject's state changes, it iterates that list and calls a fixed method (`update(...)`) on each one. Each observer decides for itself what to do with the update.

**When to reach for it — the concrete signal:** "when X happens, an unknown-in-advance, possibly-changing set of other things need to react to it" — and critically, X (the subject) shouldn't need to be modified every time a new kind of reactor is added.

**Trade-off against the nearest alternative:** the alternative is the subject directly calling each specific reactor by name (`currentConditionsDisplay.refresh(); statisticsDisplay.refresh(); ...`, hardcoded). That means every new display type requires editing the subject's own code — a direct Open/Closed violation (Week 1, Day 6). Observer's cost is one more interface and a list to maintain; its payoff is that new subscribers register themselves, and the subject's code never changes to accommodate them.

**Complexity:** notifying `n` observers is `O(n)` — a fixed, small cost per state change, dominated in practice by whatever work each observer does in its own `update()`, not by the notification mechanism itself.

*(Full implementation below, built test-first — see Part 2.)*

---

## Strategy

**What it is:** Strategy lets you swap an algorithm's implementation at runtime behind a common interface, chosen by the client.

**Mechanism:** an interface declares one method representing "the algorithm" (however it's actually computed); multiple concrete classes implement it, each a different algorithm; a context class holds a reference to the interface (not to any concrete implementation) and calls it without knowing which one it's holding.

```java
public interface DiscountStrategy {
    double apply(double originalPrice);
}

public class PercentageDiscount implements DiscountStrategy {
    private final double percentage;   // e.g. 0.10 for 10% off

    public PercentageDiscount(double percentage) {
        this.percentage = percentage;
    }

    @Override
    public double apply(double originalPrice) {
        return originalPrice * (1 - percentage);
    }
}

public class FlatDiscount implements DiscountStrategy {
    private final double flatAmount;

    public FlatDiscount(double flatAmount) {
        this.flatAmount = flatAmount;
    }

    @Override
    public double apply(double originalPrice) {
        return Math.max(0, originalPrice - flatAmount);
    }
}

public class NoDiscount implements DiscountStrategy {
    @Override
    public double apply(double originalPrice) {
        return originalPrice;
    }
}

public class Checkout {
    private DiscountStrategy discountStrategy;

    public Checkout(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    public void setDiscountStrategy(DiscountStrategy discountStrategy) {   // swappable at runtime
        this.discountStrategy = discountStrategy;
    }

    public double checkout(double cartTotal) {
        return discountStrategy.apply(cartTotal);
    }
}
```

`Checkout` never has an `if (discountType == PERCENTAGE) ... else if (discountType == FLAT) ...` chain — it just calls `apply()` on whatever `DiscountStrategy` it was handed, and the caller decides which one that is, even changing it between calls via `setDiscountStrategy`.

**When to reach for it — the concrete signal:** "the *client* (external caller) needs to choose which algorithm runs, and that choice can vary independently of everything else the class does." Multiple payment methods, multiple sorting comparators, multiple discount rules — all Strategy.

**Trade-off against the nearest alternative:** the alternative is a type-branching conditional inside the class itself (`if/else` or `switch` over a discount-type enum). That's a direct parallel to Week 15 Day 101's Simple-Factory-vs-Factory-Method distinction — the conditional violates Open/Closed (a new discount type means editing existing code), where Strategy's new implementation is a wholly new class touching nothing existing.

**Complexity:** `O(1)` — selecting and invoking a strategy is a single reference call; whatever complexity exists lives inside that specific strategy's own `apply()`, not in the selection mechanism.

**🔗 Forward reference:** Strategy gets two full system-level applications next week — Splitwise's equal/exact/percentage `Split` types (Week 17, Day 115) and Food Delivery's `NearestPartnerStrategy`/`HighestRatedPartnerStrategy` (Week 17, Day 118). Tomorrow's Theory Block draws the State-vs-Strategy distinction directly, so keep this mechanism fresh overnight.

---

## State

**What it is:** State lets an object change its own behavior when its internal state changes, without a giant conditional block anywhere.

**Mechanism:** a `State` interface declares the operations whose behavior varies by state; concrete state classes implement it, one per state; the **context object holds a reference to its current state object** and delegates to it — and, critically, a state transition means the context replaces *which* state object it's holding, not that it flips an internal flag a big conditional then checks.

```java
public interface TrafficLightState {
    void next(TrafficLight light);   // advance to whatever state comes after this one
    String getColor();
}

public class RedState implements TrafficLightState {
    @Override
    public void next(TrafficLight light) {
        light.setState(new GreenState());   // Red -> Green
    }

    @Override
    public String getColor() {
        return "RED";
    }
}

public class GreenState implements TrafficLightState {
    @Override
    public void next(TrafficLight light) {
        light.setState(new YellowState());  // Green -> Yellow
    }

    @Override
    public String getColor() {
        return "GREEN";
    }
}

public class YellowState implements TrafficLightState {
    @Override
    public void next(TrafficLight light) {
        light.setState(new RedState());     // Yellow -> Red
    }

    @Override
    public String getColor() {
        return "YELLOW";
    }
}

public class TrafficLight {
    private TrafficLightState currentState = new RedState();   // starts Red

    public void setState(TrafficLightState state) {
        this.currentState = state;
    }

    public void advance() {
        currentState.next(this);   // delegate — TrafficLight itself never checks "if red, go green..."
    }

    public String getColor() {
        return currentState.getColor();
    }
}
```

Notice `TrafficLight` itself contains **zero** conditional logic about what comes after what — every transition rule lives inside the state class it belongs to. Calling `advance()` three times cycles Red → Green → Yellow → Red, entirely through delegation.

**When to reach for it — the concrete signal:** the requirement describes an object with a **small, enumerable set of internal states**, where *what the object does in response to the same method call* genuinely differs by which state it's currently in — and, ideally, the states themselves know what comes next.

**Trade-off against the nearest alternative:** the alternative is one field (`currentColor`) plus a conditional in every method that behaves differently by state. That conditional grows by one branch per method per state (methods × states, not just states) and lives in one increasingly tangled class; State distributes it into one small class per state instead.

**Complexity:** `O(1)` per transition — swapping which object `currentState` points to is a single reference assignment, regardless of how many states exist.

**🔗 Forward reference:** this traffic light is deliberately small — Day 109's Vending Machine is where State gets its full, real system treatment (`HasCoinState`, `NoCoinState`, `DispensingState`, `SoldOutState`), and tomorrow's Theory Block puts State and Strategy side by side directly. Keep both mechanisms sharp overnight; the comparison assumes both are already solid, not re-taught.

---

## Command

**What it is:** Command encapsulates a request as an object, which is what enables queuing it, logging it, or undoing it later.

**Mechanism:** a `Command` interface declares an `execute()` method (and, when undo matters, an `undo()` method); each concrete command wraps everything needed to perform one specific action — including a reference to whatever actually does the work (the **receiver**). An **invoker** holds and triggers commands without knowing what any of them actually do.

```java
public interface Command {
    void execute();
    void undo();
}

// The receiver — does the actual work. Knows nothing about commands or invokers.
public class Light {
    private boolean on = false;

    public void turnOn() {
        on = true;
        System.out.println("Light is ON");
    }

    public void turnOff() {
        on = false;
        System.out.println("Light is OFF");
    }

    public boolean isOn() {
        return on;
    }
}

public class LightOnCommand implements Command {
    private final Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.turnOn();
    }

    @Override
    public void undo() {
        light.turnOff();   // the inverse of execute()
    }
}

public class LightOffCommand implements Command {
    private final Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.turnOff();
    }

    @Override
    public void undo() {
        light.turnOn();
    }
}

// The invoker — triggers commands, keeps a history for undo, never references Light directly.
public class RemoteControl {
    private final Deque<Command> history = new ArrayDeque<>();

    public void press(Command command) {
        command.execute();
        history.push(command);
    }

    public void pressUndo() {
        if (!history.isEmpty()) {
            history.pop().undo();
        }
    }
}
```

```java
Light livingRoomLight = new Light();
RemoteControl remote = new RemoteControl();

remote.press(new LightOnCommand(livingRoomLight));    // Light is ON
remote.press(new LightOffCommand(livingRoomLight));   // Light is OFF
remote.pressUndo();                                    // Light is ON  (undoes the OFF)
```

**Why the same mechanism enables queuing and logging too, not just undo:** `history` above is just a `Deque<Command>` — the exact same "hold commands as objects in a collection" idea directly supports a work queue (a `List<Command>` processed later, on a different thread, or in a different order than requested) or an audit log (each command serializing its own state before `execute()` runs). Undo, queuing, and logging aren't three separate features requiring three separate designs — they're three consequences of the same underlying move: **a request that would otherwise be an immediate, untraceable method call becomes a first-class object that can be stored, inspected, delayed, or reversed.**

**When to reach for it — the concrete signal:** a requirement mentions undo, a history/audit trail, scheduling work for later, or queuing operations — any case where "do this now" needs to become "remember that this should be done" as a distinct, storable thing.

**Trade-off against the nearest alternative:** the alternative is the invoker calling the receiver's methods directly (`remote.turnOnPressed()` calling `light.turnOn()` inline). That's simpler for a single, immediate action, but the request is never a first-class *thing* — there's nothing to store in a history, log, or hand to a queue. Command's cost is one class per distinct action; its payoff is that "an action" becomes data, not just a call.

**Complexity:** `O(1)` to execute or undo a single command; a full undo history of depth `k` costs `O(k)` space, the same bound as any stack-backed history.

> ⚠️ **Common Mistake:** giving the receiver's logic (the actual `turnOn()`/`turnOff()` work) to the *command* class instead of the *receiver*. Commands should stay thin — wrap a reference to the receiver and call into it — so the same receiver logic isn't duplicated across a `LightOnCommand` and, say, a hypothetical voice-activated trigger that also needs to turn the light on.

---

## Template Method

**What it is:** Template Method defines the skeleton of an algorithm in a base class, with specific steps deferred to subclasses — the overall sequence stays fixed, but individual steps can vary.

**Mechanism:** an abstract class defines one **concrete, non-overridable** method containing the fixed sequence of steps; that method calls several **abstract** methods for the steps that genuinely vary; subclasses implement only those abstract steps, never touching the sequence itself.

**When to reach for it — the concrete signal:** "several variants of a process share the exact same overall sequence of steps, but one or two specific steps differ per variant" — read, process, write is always the order; only *how* to read and *how* to process legitimately differs.

**Trade-off against the nearest alternative:** the alternative is each variant reimplementing the entire sequence independently. That duplicates the shared steps (and the *order* they must run in) across every variant, and a bug fix to the shared sequence needs to be applied everywhere it was copied. Template Method's cost is a slightly less obvious control flow (the sequence lives in the base class, not visibly in each subclass); its payoff is that the sequence exists exactly once, and subclasses genuinely cannot get the order wrong.

### Sketch — `DataProcessor`

```java
public abstract class DataProcessor {

    // The template method: fixed sequence, NOT overridable (no subclass can reorder or skip a step).
    public final void run() {
        readData();
        process();
        writeData();
    }

    // Concrete, shared step — every subclass reads the same way. Override only if a subclass genuinely needs to.
    protected void readData() {
        System.out.println("Reading data from default source...");
    }

    // Abstract step — every subclass MUST supply its own processing logic.
    protected abstract void process();

    // Concrete, shared step — same reasoning as readData().
    protected void writeData() {
        System.out.println("Writing data to default destination...");
    }
}

public class CsvDataProcessor extends DataProcessor {
    @Override
    protected void process() {
        System.out.println("Parsing rows as CSV...");
    }
}

public class JsonDataProcessor extends DataProcessor {
    @Override
    protected void process() {
        System.out.println("Parsing as JSON...");
    }
}
```

```java
DataProcessor processor = new CsvDataProcessor();
processor.run();   // readData() -> process() [CSV-specific] -> writeData(), in that fixed order, always
```

**🔗 Direct connection to Week 15, Day 101's Factory Method:** the shape — a base class defining structure, with one or more `abstract` methods subclasses are compelled to fill in — is the exact same "abstract creator + abstract method overridden per concrete subclass" mechanism Factory Method used, just aimed at a different job. Factory Method uses it so subclasses can vary *what gets created*; Template Method uses it so subclasses can vary *one step of an algorithm* while the surrounding sequence stays fixed and un-overridable (note `run()` above is `final` specifically to make the sequence non-negotiable — Factory Method's creation method typically isn't `final`, since the whole point there is usually to let it be one replaceable piece within a larger flow, a small but real distinction worth having precise).

**Complexity:** `O(1)` structurally — `run()` always executes exactly three steps in a fixed order, regardless of what each step's own internal complexity happens to be.

> ⚠️ **Common Mistake:** making the template method (`run()` above) non-`final`, or failing to make the varying steps `abstract`. Either mistake lets a subclass silently override the sequence itself, which defeats the entire pattern — the guarantee Template Method exists to provide is precisely that the *order* can't drift per subclass.

---

## 🔑 Key Takeaway — Telling the Five Apart

| Pattern | Solves | One-line signal |
|---|---|---|
| **Observer** | An unknown, possibly-growing set of things need to react to a change | "When X happens, N other things (not fixed in advance) need to know." |
| **Strategy** | The *client* needs to choose which algorithm runs | "The caller picks the approach; the class doesn't branch on a type internally." |
| **State** | An object's *own* behavior changes as its internal lifecycle progresses | "The object itself transitions between a small, enumerable set of states." |
| **Command** | A request needs to become a storable, undoable, queueable *thing* | "Undo, audit log, or 'do this later' shows up in the requirements." |
| **Template Method** | Several variants share one fixed sequence, differing in a couple of steps | "Same steps, same order, every variant — only *how* one or two steps work differs." |

---

# Part 2 — Test-Driven Development: Red, Green, Refactor

TDD is a workflow, not a design pattern — it governs *the order in which code and tests get written*, not how classes relate to each other. Three phases, repeated in a tight loop:

1. **🔴 Red — write a failing test first.** Before any production code exists (or before the next small increment of it does), write a test expressing what the code *should* do. Run it. It fails — often because it doesn't even compile yet, since the class or method it calls doesn't exist. That failure is confirmed and expected, not a mistake.
2. **🟢 Green — write the minimum code to pass.** Not the most elegant version, not the fully general version — the smallest change that makes the failing test pass. Run it again to confirm.
3. **🔵 Refactor — clean up, with the safety net already in place.** Now that a passing test exists, restructure the code (extract a method, rename something, remove duplication) with the confidence that the test will immediately flag anything the refactor accidentally breaks.

**Why each phase exists, not just what it's called:** Red first, before any implementation, forces the *interface* to be designed from the caller's perspective — what would calling code actually want to write? — rather than designing an implementation and bolting a test onto it afterward, which tends to test what the code *happens* to do rather than what it's *supposed* to do. Green being deliberately minimal keeps each step small and reversible — a failing test after a tiny change has one obvious cause, where a failing test after a large change requires hunting. Refactor exists because Green's "just make it pass" code is often not code you'd want to keep — the passing test is what makes it *safe* to improve afterward instead of leaving it as first-draft code forever.

### Applying TDD — Building `WeatherStation` → `Display` Test-First

**🔴 Red.** Before `WeatherStation` or `Display` exist at all:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class WeatherStationTest {
    @Test
    void registeredDisplayReceivesTemperatureUpdate() {
        WeatherStation station = new WeatherStation();
        RecordingDisplay display = new RecordingDisplay();   // a test-only Display that just remembers what it was told

        station.registerDisplay(display);
        station.setTemperature(72.5);

        assertEquals(72.5, display.getLastTemperature(), 0.001);
    }
}
```

This doesn't compile yet — `WeatherStation`, `Display`, `RecordingDisplay` don't exist. That's the honest starting point of Red: the test states the API as it *should* look from a caller's point of view, before a single line of that API exists.

**🟢 Green.** Write exactly enough to pass — the `Display` interface, the `Subject` interface it's paired with, `WeatherStation` implementing `Subject`, and the test-only `RecordingDisplay`:

```java
public interface Display {
    void update(double temperature);
}

public interface Subject {
    void registerDisplay(Display display);
    void removeDisplay(Display display);
    void notifyDisplays();
}

public class WeatherStation implements Subject {
    private final List<Display> displays = new ArrayList<>();
    private double temperature;

    @Override
    public void registerDisplay(Display display) {
        displays.add(display);
    }

    @Override
    public void removeDisplay(Display display) {
        displays.remove(display);
    }

    @Override
    public void notifyDisplays() {
        for (Display display : displays) {
            display.update(temperature);
        }
    }

    public void setTemperature(double temperature) {
        this.temperature = temperature;
        notifyDisplays();   // every state change immediately pushes to every registered display
    }
}
```

```java
// Test-only stand-in — deliberately minimal, exists only to make the assertion checkable.
class RecordingDisplay implements Display {
    private double lastTemperature;

    @Override
    public void update(double temperature) {
        this.lastTemperature = temperature;
    }

    double getLastTemperature() {
        return lastTemperature;
    }
}
```

Run the test again: it passes. This is Green — not the most interesting code yet, but correct and minimal.

**🔵 Refactor.** With the safety net in place, add the two *real* displays the requirement actually asked for, and a second test locking in that Observer's core promise — multiple displays, each reacting independently:

```java
public class CurrentConditionsDisplay implements Display {
    private double currentTemperature;

    @Override
    public void update(double temperature) {
        this.currentTemperature = temperature;
        System.out.println("Current conditions: " + currentTemperature + "°");
    }
}

public class StatisticsDisplay implements Display {
    private double minTemperature = Double.MAX_VALUE;
    private double maxTemperature = Double.MIN_VALUE;
    private double sum = 0;
    private int count = 0;

    @Override
    public void update(double temperature) {
        minTemperature = Math.min(minTemperature, temperature);
        maxTemperature = Math.max(maxTemperature, temperature);
        sum += temperature;
        count++;
        System.out.println("Stats — min: " + minTemperature + ", max: " + maxTemperature + ", avg: " + (sum / count));
    }
}
```

```java
@Test
void multipleDisplaysAllReceiveTheSameUpdateIndependently() {
    WeatherStation station = new WeatherStation();
    RecordingDisplay a = new RecordingDisplay();
    RecordingDisplay b = new RecordingDisplay();
    station.registerDisplay(a);
    station.registerDisplay(b);

    station.setTemperature(68.0);

    assertEquals(68.0, a.getLastTemperature(), 0.001);
    assertEquals(68.0, b.getLastTemperature(), 0.001);   // both received it — WeatherStation never named either by type
}

@Test
void removedDisplayNoLongerReceivesUpdates() {
    WeatherStation station = new WeatherStation();
    RecordingDisplay display = new RecordingDisplay();
    station.registerDisplay(display);
    station.removeDisplay(display);

    station.setTemperature(100.0);

    assertEquals(0.0, display.getLastTemperature(), 0.001);   // never updated — still at its default
}
```

**What actually got refactored here, precisely:** nothing about `WeatherStation`'s own code needed to change to support `CurrentConditionsDisplay` and `StatisticsDisplay` — which is Observer's Open/Closed payoff, made concretely visible rather than just claimed. The refactor step added *new* observer implementations and *new* tests locking in behavior (multiple independent subscribers, correct removal) that Green's minimal version hadn't yet proven.

> 🔑 **Key Takeaway:** TDD and Observer reinforce each other here in a way worth noticing explicitly — Observer's whole value proposition is "the subject's code never needs to change when a new observer is added." `removedDisplayNoLongerReceivesUpdates` and `multipleDisplaysAllReceiveTheSameUpdateIndependently` are tests that would **only be worth writing at all** because Observer makes that guarantee — they're effectively testing the pattern's own promise, not just this specific weather-station example.

---

# Part 3 — DSA Revision Block (1.5 hrs)

**Format for both problems below:** the statement and constraints are restated, but the *approach* is a recap, not a from-scratch re-teach, per this series' own established convention for material a prior week already covered in full. **Attempt each one cold, without looking past the statement, before reading the recap that follows it** — that's the actual point of a spaced-repetition check; reading the answer first defeats it.

---

## Revision 1 — Course Schedule (LeetCode 207, Medium)

**🔗 Originally taught in full depth:** Week 11, Day 71 — Topological Sort, Kahn's Algorithm / Cycle Detection.

**Statement:** given `numCourses` and a list of prerequisite pairs `[a, b]` (meaning course `a` requires course `b` first), return `true` if it's possible to finish all courses, `false` if a cycle makes that impossible.

### Recap: The Approach

Build a directed graph where an edge `b → a` means "b must be taken before a," and track each course's **in-degree** (how many prerequisites it still has). **Kahn's Algorithm:** start a BFS queue with every course whose in-degree is already 0 (no prerequisites); repeatedly dequeue a course, "complete" it, and decrement the in-degree of every course it points to — any course whose in-degree drops to 0 as a result joins the queue. If the total number of courses processed this way equals `numCourses`, every course was reachable in some valid order — no cycle. If it's less, some courses' in-degrees never reached 0, which only happens if they're stuck in a cycle with each other.

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> graph = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) graph.add(new ArrayList<>());
    int[] inDegree = new int[numCourses];

    for (int[] pair : prerequisites) {
        int course = pair[0], prereq = pair[1];
        graph.get(prereq).add(course);   // edge: prereq -> course
        inDegree[course]++;
    }

    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (inDegree[i] == 0) queue.add(i);
    }

    int processed = 0;
    while (!queue.isEmpty()) {
        int current = queue.poll();
        processed++;
        for (int next : graph.get(current)) {
            if (--inDegree[next] == 0) queue.add(next);
        }
    }

    return processed == numCourses;
}
```

### Fresh Trace — A Different Example Than Day 71 Used

`numCourses = 4`, `prerequisites = [[1,0],[2,0],[3,1],[3,2]]` — course 1 needs 0; course 2 needs 0; course 3 needs both 1 and 2.

```
Edges: 0→1, 0→2, 1→3, 2→3
In-degrees: [0]=0, [1]=1, [2]=1, [3]=2
Queue starts: [0]              (only course with in-degree 0)

Dequeue 0, processed=1. Decrement 1 (→0, enqueue), decrement 2 (→0, enqueue). Queue: [1, 2]
Dequeue 1, processed=2. Decrement 3 (→1, not yet 0).                          Queue: [2]
Dequeue 2, processed=3. Decrement 3 (→0, enqueue).                            Queue: [3]
Dequeue 3, processed=4. No outgoing edges.                                    Queue: []

processed == 4 == numCourses  →  true
```

**Complexity:** `O(V + E)` time — every course processed once, every prerequisite edge traversed once. `O(V + E)` space for the adjacency list plus the in-degree array and queue.

### Common Mistakes Checklist

- [ ] Getting the edge direction backwards (`course → prereq` instead of `prereq → course`) — this silently computes the wrong in-degrees and produces wrong results without crashing, the most dangerous kind of bug.
- [ ] Forgetting a self-loop (`[0, 0]`) is an immediate cycle — course 0 depending on itself can never reach in-degree 0.
- [ ] Only checking `queue.isEmpty()` at the start instead of comparing `processed == numCourses` at the end — courses stuck in a cycle never enter the queue at all, so an empty *starting* queue check alone misses cycles that don't happen to involve every course.
- [ ] Forgetting that disconnected components (a course with no prerequisites and nothing depending on it) are still valid and must still be counted in `processed`.

**If any of this needed re-deriving rather than confirming:** go back to Week 11, Day 71 for the full original treatment, including the DFS three-state-coloring alternative and its own cycle-detection argument.

---

## Revision 2 — Coin Change (LeetCode 322, Medium)

**🔗 Originally taught in full depth:** Week 12, Day 84 — 1D DP, Unbounded Knapsack.

**Statement:** given coin denominations `coins[]` and a target `amount`, return the fewest number of coins needed to make exactly `amount`, or `-1` if it's impossible.

### Recap: The Approach

`dp[i]` = minimum coins needed to make amount `i`. Base case `dp[0] = 0` (zero coins needed to make zero). For every amount `i` from `1` to `amount`, and every coin `c`: if `c <= i`, then `dp[i] = min(dp[i], dp[i - c] + 1)` — "the best way to make `i` using coin `c` at least once is one more than the best way to make `i - c`." Because the same coin can be reused (this is the **unbounded** part of unbounded knapsack), `dp[i - c]` is allowed to already include uses of `c` itself — there's no "used coins" tracking needed, just the amount remaining. Initialize every `dp[i]` (`i > 0`) to a sentinel larger than any possible real answer (`amount + 1` works, since no valid answer can exceed using all 1s); if `dp[amount]` is still at that sentinel at the end, no combination works — return `-1`.

```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);   // sentinel: larger than any real answer
    dp[0] = 0;

    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }

    return dp[amount] > amount ? -1 : dp[amount];
}
```

### Fresh Trace — A Different Example Than Day 84 Used

`coins = [1, 2, 5]`, `amount = 11`.

```
dp[0]=0
dp[1] = dp[0]+1 = 1                                          (coin 1)
dp[2] = min(dp[1]+1=2, dp[0]+1=1) = 1                          (coin 2)
dp[3] = min(dp[2]+1=2, dp[1]+1=2) = 2
dp[4] = min(dp[3]+1=3, dp[2]+1=2) = 2
dp[5] = min(dp[4]+1=3, dp[3]+1=3, dp[0]+1=1) = 1               (coin 5)
dp[6] = min(dp[5]+1=2, dp[4]+1=3, dp[1]+1=2) = 2
dp[7] = min(dp[6]+1=3, dp[5]+1=2, dp[2]+1=2) = 2
dp[8] = min(dp[7]+1=3, dp[6]+1=3, dp[3]+1=3) = 3
dp[9] = min(dp[8]+1=4, dp[7]+1=3, dp[4]+1=3) = 3
dp[10]= min(dp[9]+1=4, dp[8]+1=4, dp[5]+1=2) = 2               (5 + 5)
dp[11]= min(dp[10]+1=3, dp[9]+1=4, dp[6]+1=3) = 3               (5 + 5 + 1, or 5 + 2 + 2 + 2 — both cost 3)

dp[11] = 3  →  matches the known correct answer for coins=[1,2,5], amount=11
```

**Complexity:** `O(amount × numCoins)` time — the nested loop's exact bound. `O(amount)` space for the `dp` array.

### Common Mistakes Checklist

- [ ] Initializing `dp[i]` to `0` instead of a large sentinel — `0` would wrongly imply "zero coins needed" for every unreached amount, corrupting every `min()` comparison that touches it.
- [ ] Reaching for a **greedy** approach (always take the largest coin that fits) instead of DP — greedy fails on `coins = [1, 3, 4]`, `amount = 6`: greedy takes `4 + 1 + 1` (3 coins), but the true optimum is `3 + 3` (2 coins). Worth having this exact counterexample ready — it's the fastest way to justify DP over the "obviously simpler" greedy idea if an interviewer pushes on it.
- [ ] Off-by-one on the `dp` array size — it needs `amount + 1` slots (indices `0` through `amount` inclusive), not `amount`.
- [ ] Forgetting the final `dp[amount] > amount` check and returning the sentinel value directly instead of `-1` when the amount is genuinely unreachable (e.g., `coins = [2]`, `amount = 3`).

**If any of this needed re-deriving rather than confirming:** go back to Week 12, Day 84 for the full original treatment, including how this connects forward to Word Break (Day 84, same day) and back to Climbing Stairs' recurrence shape (Week 12, Day 81).

---

# Project Block Guide (1.5 hrs)

**Repository:** `lld-java`, `design-patterns` module.

**Task:** the `WeatherStation`/`Display` implementation from Part 2, built via the TDD cycle demonstrated there.

**Definition of done:**
- All code and tests from Part 2's Red-Green-Refactor walkthrough pushed to `lld-java/design-patterns/observer/`.
- A short `TDD_NOTES.md` in the same folder documenting the actual Red → Green → Refactor sequence you followed — which test you wrote first, what the minimal Green implementation looked like, and what specifically changed in the Refactor step. This is the "Red-Green-Refactor practiced and documented" deliverable the plan asks for — a real record of the order things happened in, not a polished-after-the-fact description.
- `DataProcessor`/`CsvDataProcessor`/`JsonDataProcessor` sketch committed alongside it under `template-method/` — sketch depth, no full test suite required, matching Composite's precedent from yesterday.

---

# Career Block Guide (1 hr)

**LinkedIn:** engagement — 20 minutes commenting on 3–5 posts from people at target companies or in the SDE-2/backend space.

**Networking — schedule Mock Interview #1 for this weekend.** This is the first of five LLD mocks across this phase (Day 109 does the first one, using Tic-Tac-Toe as the subject) — get it on the calendar with the accountability partner found back in Week 1, Day 5, rather than letting "I'll schedule it later" slip.

**Worth knowing now, even though the full treatment is Week 21:** Google's "Googleyness," Databricks' own leadership principles, and Atlassian's five named values are each evaluated as their **own axis** in those companies' interview loops — not folded into generic behavioral questions. Real candidates report being downleveled specifically for treating these casually, as if any confident-sounding answer would do. Filing this away now, three weeks before it's directly actionable, is deliberate — it's the kind of thing that benefits from marinating rather than being crammed the night before Week 21.

---

# Day 107 — Interview Questions

**Q1. Distinguish Observer from Strategy — both involve a class holding a reference to something implementing an interface.** Observer is 1-to-many and reactive: a subject pushes updates to a *set* of observers whenever its own state changes, and the observers don't choose anything. Strategy is 1-to-one and client-directed: a client hands a context exactly *one* algorithm implementation to use, chosen deliberately, not triggered by a state change.

**Q2. Why does `WeatherStation.notifyDisplays()` never need to change when a new `Display` type is added?** It iterates a `List<Display>` and calls `update()` polymorphically — it depends only on the `Display` interface, never on any concrete display class by name. A new display type just needs to implement `Display` and register itself.

**Q3. What's the actual distinguishing question between State and Strategy, in one sentence?** Does the object's own internal lifecycle drive the behavior change (State), or does an external client choose the behavior (Strategy)? Both look structurally similar in code; the difference is *who's in control of the swap and why it happens*.

**Q4. Why is `TrafficLight` itself free of any conditional logic about what state comes next?** Every transition rule lives inside the state class it belongs to (`RedState.next()` creates a `GreenState`, etc.) — `TrafficLight.advance()` only ever calls `currentState.next(this)`, delegating the decision entirely rather than checking `if (currentColor == RED) ...`.

**Q5. How does Command enable undo, queuing, and logging from the same underlying mechanism?** All three come from making a request a stored *object* rather than an immediate method call — a `Deque<Command>` history supports undo (pop and call the inverse), the identical structure as a `List<Command>` supports queuing (process later, in any order), and a command serializing its own state before executing supports logging. One design decision, three payoffs.

**Q6. Why must Template Method's `run()` be declared `final`?** If a subclass could override it, it could reorder or skip steps entirely, defeating the pattern's core guarantee — that the sequence is fixed across every variant, with only specific named steps left to vary.

**Q7. What is the actual purpose of TDD's Red phase, beyond "write a test"?** It forces the API to be designed from the caller's perspective before any implementation exists, since the test is the first piece of code to actually *use* the not-yet-built class — this tends to produce simpler, more caller-friendly interfaces than writing the implementation first and testing it afterward.

**Q8. Why does Green deliberately aim for the *minimum* code to pass, rather than the best implementation right away?** Small, minimal steps keep each change easy to reason about — if the test fails after a tiny change, the cause is almost always that change. Writing a large, "final" implementation before any test has passed reintroduces the exact untested-guesswork risk TDD exists to avoid.

**Q9. In Course Schedule, why does an edge go from the prerequisite to the dependent course (`prereq → course`), not the other way?** Kahn's Algorithm processes nodes with in-degree 0 first — courses with no remaining prerequisites. An edge `prereq → course` correctly increments `course`'s in-degree for each prerequisite it has; reversing the edge direction would compute in-degrees for the wrong relationship and silently produce wrong results.

**Q10. Why does Coin Change's DP allow `dp[i - coin]` to already include uses of the same coin?** The problem allows unlimited reuse of each denomination (unbounded knapsack) — there's no constraint tracking "how many of this coin have I used," only "how much amount remains," so reusing the same coin type multiple times toward the same total is exactly the intended behavior, not a bug.

**Q11. Give a concrete counterexample showing why a greedy approach fails for Coin Change.** `coins = [1, 3, 4]`, `amount = 6`: greedy takes the largest fitting coin first (`4`), then is forced into `1 + 1`, totaling 3 coins; the true optimum is `3 + 3`, totaling 2 coins. Greedy's locally-largest choice isn't always part of the globally optimal solution here.

---

## Daily Deliverable Check

- [ ] Observer pattern implemented via TDD; `TDD_NOTES.md` documents the actual Red-Green-Refactor sequence.
- [ ] Template Method sketch (`DataProcessor` + two subclasses) complete.
- [ ] Course Schedule (LC 207) and Coin Change (LC 322) both solved cold, without hints.
- [ ] Mock Interview #1 scheduled for this weekend.

---

## What Tomorrow Assumes You Already Know Cold

Day 108 assumes the full 5-step LLD framework (Day 106) and all ten patterns taught across these two days (five Structural, five Behavioral) are fluent enough to reach for without flipping back through this book — tomorrow is the first day the framework gets applied end-to-end, live, against a real system, rather than one piece at a time against isolated examples. It also assumes Course Schedule and Coin Change's underlying patterns (Topological Sort, Unbounded Knapsack) are confirmed solid from today's cold-solve check, since tomorrow's own revision problem (one Backtracking problem, cold) is a continuation of the same spaced-repetition discipline, not a one-off.
