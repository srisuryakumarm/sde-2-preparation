# Day 107 — Missing Primer: The Design Patterns the Plan Skips (Abstract Factory, Prototype, Bridge, Flyweight, Iterator, Memento, Mediator, Visitor, Interpreter)

**Do this first on Day 107 (≈ 90 min), then continue with Day 107's regular plan.**
**Needed by:** Days 108–119 (LLD problems and mocks), and LLD interviews where "name the pattern" and "when would you use X?" are standard follow-ups.
**Why it is missing:** Days 100–107 cover Singleton, Factory Method, Builder, Adapter, Decorator, Proxy, Facade, Composite, Chain of Responsibility, Command, Observer, State, Strategy and Template Method. **Nine of the 23 Gang-of-Four patterns appear nowhere in the repo**: Abstract Factory, Prototype, Bridge, Flyweight, Iterator, Memento, Mediator, Visitor, Interpreter.
**Already in place:** SOLID (Day 6), the patterns above (Days 100–107), interfaces/composition (Day 5), sealed types and pattern `switch` (Day 28).

---

| Pattern | Intent (one line) | Use it when | Minimal Java shape |
|---|---|---|---|
| **Abstract Factory** | create a **family** of related objects without naming concrete classes | UI themes, per-vendor DB/cloud clients | `interface Theme { Button button(); Checkbox checkbox(); }` with `DarkTheme`, `LightTheme`; client depends only on `Theme` |
| **Prototype** | create new objects by **copying** an existing one | construction is expensive or configuration-heavy | `Shape copy()` via a **copy constructor** (Java's `Cloneable` is shallow and awkward — avoid) |
| **Bridge** | split an abstraction from its implementation so both vary independently | avoids N×M subclasses (`Urgent/Normal` × `Email/Sms/Push`) | `abstract class Notification { protected final Channel channel; … }` — compose, don't inherit |
| **Flyweight** | **share** immutable intrinsic state across many objects | millions of similar objects (chess pieces, text glyphs, map tiles) | a factory returning cached instances; `Integer.valueOf`'s −128..127 cache (Day 10) is a flyweight |
| **Iterator** | traverse a collection without exposing its structure | custom traversals (tree in-order, paginated results) | implement `Iterable<T>` returning an `Iterator<T>`; fail-fast iterators throw `ConcurrentModificationException` |
| **Memento** | capture and later **restore** an object's state without breaking encapsulation | undo/redo, checkpoints | `Editor.save()` returns an opaque `Snapshot`; a `History` (caretaker) keeps a stack and calls `editor.restore(s)` |
| **Mediator** | route all communication through **one coordinator** | many peers that would otherwise know each other (chat room, air-traffic control, UI dialogs) | `ChatRoom.send(from, msg)` delivers to members; members hold only the `ChatRoom` |
| **Visitor** | add new operations to a class hierarchy **without editing it** (double dispatch) | stable hierarchy, many changing operations (AST, shapes → area/export) | `interface Visitor<R> { R visit(Circle c); R visit(Rect r); }` + `accept(Visitor)`; in Java 21 a **sealed interface + pattern `switch`** often replaces it |
| **Interpreter** | represent a small **grammar** as objects and evaluate it | expression/rule evaluators, filters | `sealed interface Expr` with `Num`, `Add`, `Mul` records and an `eval` switch |

## Notes that win interview points
- **Undo:** *Command* stores the **operation** (and its inverse); *Memento* stores the **state**. Use Command for fine-grained undo, Memento for snapshots; real editors mix both.
- **Mediator vs Observer:** Observer = one subject notifies many listeners (one-way); Mediator = peers talk to each other **through** a hub (many-to-many, rules live in the hub). A Kafka topic or Spring's `ApplicationEventPublisher` is mediator-like.
- **Visitor vs sealed + `switch`:** adding a *new operation* is easy either way; adding a *new type* breaks every visitor/switch — the compiler tells you where (Day 79's exercise).
- **Abstract Factory vs Factory Method:** Factory Method = one method, subclasses decide *which* product; Abstract Factory = an object that creates a *family* of products that must go together.
- **Bridge vs Strategy:** same shape, different intent — Bridge structures a two-dimensional hierarchy, Strategy swaps one algorithm at runtime.

## Exercise (≈ 40 min, `lld-java`)
1. **Bridge:** `Notification` (Urgent, Normal) × `Channel` (Email, SMS, Push) — implement without more than 5 classes.
2. **Memento:** a text `Editor` with `type`, `save`, `restore`; add undo through a `History` stack. Then redo the same with **Command** and write two sentences on the difference.
3. **Mediator:** a `ChatRoom` with three `User`s; users never reference one another.
4. **Visitor vs sealed switch:** model `Circle`/`Rect`; compute area and perimeter first with a Visitor, then with a sealed interface and pattern `switch`. Which is shorter? Which breaks louder when you add `Triangle`?
5. One paragraph each for Abstract Factory, Prototype, Flyweight, Iterator, Interpreter: a real example *outside* UI toolkits (e.g. Flyweight = String pool).

## Checklist
- [ ] Can state each of the nine intents in one sentence.
- [ ] Can contrast Command vs Memento, Mediator vs Observer, Bridge vs Strategy, Abstract Factory vs Factory Method.
- [ ] Can name a JDK example for Flyweight (`Integer` cache), Iterator, and Prototype's pitfall (`Cloneable`).
- [ ] Exercises 1–4 pushed. Continue with Day 107's regular plan.
