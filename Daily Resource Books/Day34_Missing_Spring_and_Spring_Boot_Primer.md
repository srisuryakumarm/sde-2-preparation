# Spring & Spring Boot Primer: The Missing Piece Before Day 34

**Where this fits:** insert this as its own block — theory only, no DSA attached — right before Day 34 (`todo-api` initialization). Every day from 34 onward *uses* Spring Boot; this is the one place that explains *what it actually is* underneath the annotations. Everything here is taught from scratch, the same way Week 1 taught Java itself — no assumed prior framework knowledge.

**Why this exists:** the plan teaches you to *use* `@RestController`, `@GetMapping`, `JpaRepository`, and `@KafkaListener` correctly, but never stops to explain the actual framework mechanism underneath — Dependency Injection, the IoC Container, and what a "bean" is. Without this, the annotations feel like magic incantations you're pattern-matching rather than a system you understand. This primer closes that gap once, properly, so everything from Day 34 forward clicks into a real mental model instead of memorized syntax.

**What this primer deliberately does *not* cover:** `@Entity`/`@Id`/`@Column` and JPA mapping specifics (Day 36 already teaches these properly), Spring Cloud Config and profile-switching mechanics (Day 51), AOP internals like `@Aspect`/`@Pointcut` (Day 62), and Kafka's `@KafkaListener` (Day 48/50). Those each get a dedicated theory block exactly where they're needed, so duplicating them here would just be noise. This primer's job is narrower and more foundational: the container, DI, beans, and the handful of everyday annotations that every one of those later days silently assumes you already understand.

---

## Part 1: The Problem Spring Solves

Before touching a single Spring annotation, you need to feel the actual problem Spring exists to fix — otherwise Dependency Injection just sounds like unnecessary ceremony.

Imagine a plain Java `OrderService` that needs to send a confirmation email:

```java
public class OrderService {
    private EmailSender emailSender = new EmailSender(); // OrderService creates its own dependency

    public void placeOrder(Order order) {
        // ... order logic ...
        emailSender.send(order.getCustomerEmail(), "Order confirmed");
    }
}
```

This looks harmless, but it has real, specific costs:

- **Tight coupling.** `OrderService` is now permanently bound to the concrete `EmailSender` class. If you later want to send SMS instead, or swap providers, you have to open and edit `OrderService` itself.
- **Hard to test.** You can't test `OrderService`'s logic in isolation without a real `EmailSender` actually trying to send a real email — there's no clean way to substitute a fake one for a unit test.
- **No single source of truth for object creation.** If ten classes each need their own `EmailSender`, you either create ten separate instances scattered through the codebase, or you build your own manual wiring system to share one — and now you're writing the exact infrastructure Spring already built.

This is precisely the problem Week 6, Day 6's SOLID exercise walked you through by hand — you built a `NotificationService` that violated Dependency Inversion by instantiating `EmailSender` directly, then refactored it to accept a `MessageSender` interface via constructor injection. **Spring is that same fix, generalized and automated across an entire application**, instead of you manually wiring every constructor call yourself.

---

## Part 2: Inversion of Control (IoC) and Dependency Injection (DI)

**Inversion of Control (IoC)** is the core idea: instead of a class controlling the creation of the objects it depends on (`new EmailSender()` inside `OrderService`), that control is handed over to something external — a *container*. The class simply declares what it needs; it no longer decides how to get it.

**Dependency Injection (DI)** is the specific technique that achieves IoC: dependencies are *given to* (injected into) a class from outside, rather than the class constructing them itself.

```java
public class OrderService {
    private final EmailSender emailSender;

    // The dependency is injected via the constructor — OrderService
    // no longer knows or cares how EmailSender gets built.
    public OrderService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }

    public void placeOrder(Order order) {
        emailSender.send(order.getCustomerEmail(), "Order confirmed");
    }
}
```

Nothing here is Spring-specific yet — this is just good object-oriented design, the same idea you already practiced in Week 6. Spring's actual contribution is **automating the "who builds `OrderService` and hands it a working `EmailSender`" question** for your entire application, so you never have to manually wire it yourself.

**Three ways to inject a dependency**, in order of preference:

1. **Constructor injection (preferred, and what this plan uses throughout).** Dependencies are passed in through the constructor, as above. This lets fields be `final` (immutable once set), makes it impossible to have an `OrderService` floating around in a half-wired, broken state, and makes testing trivial — you just call the constructor with a fake dependency.
2. **Setter injection.** A public setter method assigns the dependency after construction. Useful for optional dependencies, rarer in modern Spring code.
3. **Field injection** (`@Autowired` directly on a field). The most common one you'll *see* in older tutorials, but generally discouraged — it hides dependencies (you can't tell what a class needs just by looking at its constructor), and makes the class harder to test without Spring itself running.

**A fourth, different kind of injection: configuration values with `@Value`.** Everything above injects an *object* (a bean). Sometimes what a class needs isn't another object — it's a simple value that lives in `application.yml` or `application.properties`, like a default page size or a feature flag. `@Value` injects that value directly:

```java
@Service
public class TaskService {
    @Value("${task.default-priority}")
    private String defaultPriority; // pulled from application.yml at startup, not from another bean
}
```

This matters because Day 51's `dev`/`prod` profiles and Day 96's Secret-based `DB_PASSWORD` injection both lean on this exact mechanism — a value living in external configuration, injected into a bean, rather than hardcoded. Day 51 covers the profile-switching mechanics properly; this is just the one annotation that makes "a value from config, not from Java code" possible in the first place.

---

## Part 3: The Spring Container and Beans

A **bean** is simply an object whose creation and lifecycle are managed by Spring, instead of by your own code calling `new`.

The **ApplicationContext** is Spring's actual IoC container — the thing doing the managing. When your application starts, the `ApplicationContext`:

1. Scans your codebase for classes marked as beans (Part 4 covers exactly how).
2. Creates instances of them.
3. Figures out each bean's dependencies and injects them automatically — this is where DI actually happens, mechanically.
4. Holds onto those instances and hands them out wherever they're needed, for the lifetime of the application.
5. On shutdown, tears them back down in reverse order — Part 9 covers the exact hooks for "run this code right as a bean is created" and "run this code right as it's destroyed."

By default, a bean is a **singleton** — the container creates exactly one instance and reuses it everywhere it's needed, rather than creating a fresh `OrderService` every time one is required. (A `prototype` scope exists for the rare case where you genuinely want a new instance each time, but singleton is what you'll use almost everywhere in this plan.)

**The mental model that matters:** the `ApplicationContext` is one giant, automated version of "figure out what depends on what, and wire it all together in the right order" — the exact manual task you'd otherwise be doing by hand for every class in a growing application.

---

## Part 4: Telling Spring What to Manage — Stereotype Annotations

Spring needs some way to know *which* classes should become beans. You tell it by annotating the class. These are called **stereotype annotations**, and while several of them behave identically under the hood, using the right one communicates the class's role — to Spring, and to anyone reading your code.

| Annotation | Marks a class as... | Used for |
|---|---|---|
| `@Component` | A generic, Spring-managed bean | The base annotation; the other three are specializations of it |
| `@Service` | A business-logic bean | Your `TaskService`, `OrderService` — the layer where actual logic lives |
| `@Repository` | A data-access bean | Persistence-layer classes; also enables automatic translation of database exceptions into Spring's own exception types |
| `@Controller` / `@RestController` | A web-layer bean handling HTTP requests | `@RestController` additionally auto-serializes return values to JSON — this is what Day 34's `TaskController` uses |
| `@Configuration` (+ `@Bean` on a method) | A class providing manually-defined beans | For wiring up third-party classes you can't put an annotation directly on |

```java
@Repository
public interface TaskRepository extends JpaRepository<Task, Long> {
    // Spring generates a real, working implementation of this interface
    // at runtime — you never write the class body yourself. More on
    // this in Part 10, since it can otherwise feel like actual magic.
}

@Service
public class TaskService {
    private final TaskRepository taskRepository; // a dependency, injected

    // Constructor injection — Spring sees TaskService needs a
    // TaskRepository, finds the bean it already created for that
    // interface, and hands it in automatically.
    public TaskService(TaskRepository taskRepository) {
        this.taskRepository = taskRepository;
    }
}

@RestController
public class TaskController {
    private final TaskService taskService; // also injected

    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }
}
```

Notice there is no `@Autowired` written anywhere above — **as of modern Spring, if a class has exactly one constructor, Spring infers the injection automatically.** `@Autowired` is still worth recognizing (you'll see it constantly in existing codebases and older tutorials), but for single-constructor classes, which is what you'll write throughout this plan, it's optional.

This is the whole chain: `TaskController` needs a `TaskService`, which needs a `TaskRepository`. You never write `new TaskService(new TaskRepositoryImpl())` anywhere — the `ApplicationContext` builds this entire chain automatically at startup, because each class simply declared what it needed via its constructor, and each one was marked as a bean.

---

## Part 5: Handling HTTP Request Data — `@RequestBody`, `@PathVariable`, `@RequestParam`

Part 4 shows `@RestController` methods with no parameters — but almost every real endpoint in this plan needs data *from* the incoming HTTP request: a JSON payload, a value embedded in the URL path, or a query string parameter. This is genuinely the biggest gap in explaining "what happens when a request arrives," and it's worth closing carefully, since it's used starting Day 34 itself and in nearly every endpoint after (Day 68's autocomplete, Day 78's pagination, every Task/Order/Payment creation call).

```java
@RestController
@RequestMapping("/tasks") // sets a shared base path for every method below — GET /tasks/5 instead of writing "/tasks" in every mapping
public class TaskController {

    private final TaskService taskService;

    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }

    // @RequestBody: Spring deserializes the incoming JSON request body directly
    // into a Task object, using Jackson — pulled in automatically as part of
    // spring-boot-starter-web (Part 7). You never parse JSON by hand.
    @PostMapping
    public Task createTask(@RequestBody Task task) {
        return taskService.create(task);
    }

    // @PathVariable: pulls a value out of the URL path itself.
    // A GET request to /tasks/5 binds id = 5.
    @GetMapping("/{id}")
    public Task getTask(@PathVariable Long id) {
        return taskService.findById(id);
    }

    // @RequestParam: pulls a value out of the query string (?key=value).
    // This is exactly the shape Day 68's autocomplete endpoint uses:
    // GET /tasks/autocomplete?prefix=X binds prefix = "X".
    @GetMapping("/autocomplete")
    public List<Task> autocomplete(@RequestParam String prefix) {
        return taskService.findByTitleStartingWith(prefix);
    }
}
```

The distinction that actually matters in an interview or a real codebase: `@PathVariable` is for identifying *which resource* (`/tasks/5` — this specific task), `@RequestParam` is for filtering, searching, or paginating *over* a resource (`/orders?page=0&size=5` — Day 78's exact example), and `@RequestBody` is for the payload of a `POST`/`PUT` that creates or replaces something. Mixing these up (e.g., putting a search filter in the path) is a common real-world REST design smell, not just a syntax choice.

---

## Part 6: How Spring Finds These Classes — Component Scanning

Marking a class `@Service` doesn't do anything by itself unless Spring actually looks for it. **Component scanning** is the process where, at startup, Spring walks through a specified package (and everything beneath it) looking for classes carrying a stereotype annotation, and registers each one it finds as a bean.

In a Spring Boot application, this happens automatically starting from the package containing your main application class — which is exactly why `todo-api`'s structure (packages nested under the main class's package) matters, not just as a style preference.

---

## Part 7: Spring vs. Spring Boot — What Boot Actually Adds

Everything above — the container, DI, beans, stereotypes — **is just Spring**, and has existed for two decades. Plain Spring is powerful but historically required substantial manual configuration (XML files, or verbose Java `@Configuration` classes) before an application would even start.

**Spring Boot is not a different framework — it's an opinionated layer on top of Spring**, and it adds exactly three things:

1. **Auto-configuration.** Spring Boot inspects what's actually on your classpath and configures sensible beans for you automatically. If it sees a PostgreSQL driver and Spring Data JPA on the classpath, it auto-configures a `DataSource` and `EntityManager` without you writing that configuration by hand — this is precisely why Day 36's `Task` entity and `TaskRepository` "just work" once you add the right dependencies, with no manual wiring.
2. **Starter dependencies.** A single line like `spring-boot-starter-web` pulls in a whole curated, version-compatible bundle (embedded Tomcat, Spring MVC, Jackson for JSON) instead of you hand-picking and version-matching a dozen individual libraries yourself.
3. **An embedded server.** Boot packages a servlet container (Tomcat, by default) directly inside your application. You run `java -jar todo-api.jar` and the whole thing starts — no separate server installation or deployment step, which is exactly what makes `curl localhost:8080/health` work the moment Day 34's app starts.

The single annotation that ties all of this together is `@SpringBootApplication`, which you'll see at the top of `todo-api`'s main class starting Day 34. It's actually three annotations combined into one:

```java
@SpringBootApplication
// is equivalent to writing all three of these together:
// @Configuration           -> this class can itself define beans
// @EnableAutoConfiguration -> turn on Boot's classpath-based auto-configuration
// @ComponentScan           -> scan this package and below for stereotype-annotated classes
public class TodoApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(TodoApiApplication.class, args);
    }
}
```

---

## Part 8: What Actually Happens When You Run `main()`

Walking through `SpringApplication.run(...)` step by step, since this is the exact line Day 34 has you write and run for the first time:

1. Spring Boot creates the `ApplicationContext` (the container from Part 3).
2. `@ComponentScan` triggers — every class under the base package carrying `@Component`, `@Service`, `@Repository`, or `@RestController` is discovered.
3. `@EnableAutoConfiguration` triggers — Boot inspects the classpath and registers additional beans automatically based on what dependencies are present (a `DataSource` if a DB driver is found, and so on).
4. Every discovered bean is instantiated, and the container works out the dependency order — a bean needing another bean is only created once that dependency already exists — and wires constructor injection automatically at each step, running any `@PostConstruct` hook (Part 9) right after each bean's dependencies are set.
5. If a web starter is present, the embedded Tomcat server starts, bound to port 8080 by default.
6. The application is now "up" — every bean is live in the container, and incoming HTTP requests get routed to the appropriate `@RestController` method, with `@RequestBody`/`@PathVariable`/`@RequestParam` (Part 5) binding the request's data into that method's parameters.

All of that happens in the few seconds between running the app and seeing "Started TodoApiApplication" in the console.

---

## Part 9: A Few More Annotations You'll Meet Constantly

These three don't each need a full section, but all three show up often enough — in this plan and in real Spring codebases — that leaving them out would mean re-deriving them from context later instead of recognizing them on sight.

**`@Transactional` — grouping multiple database operations into one atomic unit.** A method annotated `@Transactional` either fully succeeds or fully rolls back — if anything inside it throws, every database change made earlier in that same method is undone too, not left half-applied.

```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    // constructor injection omitted for brevity

    @Transactional
    public void placeOrder(Order order) {
        orderRepository.save(order);
        inventoryService.reserve(order.getItems());
        // if inventoryService.reserve(...) throws, the orderRepository.save(...)
        // above is rolled back too — both succeed, or neither does.
    }
}
```

**Worth being precise about, since it's a genuinely common point of confusion:** `@Transactional` guarantees atomicity *within a single database transaction, inside one service*. It has nothing to do with Day 70's Saga choreography, where Order and Payment are two entirely separate services coordinating over Kafka events — that's a different problem (distributed consistency across services) solved a different way (compensating actions), precisely because a single database transaction can't span two services' separate databases in the first place. Knowing why these are different tools for different problems, not two strengths of the same tool, is exactly the kind of distinction Day 79's 2PC-vs-Saga theory block later asks you to articulate.

**`@Qualifier` and `@Primary` — disambiguating when two beans implement the same interface.** Spring injects by type — but what happens when two classes implement the same interface, and both are beans? Spring can't guess which one you mean. This becomes directly relevant the moment a Strategy-pattern setup like Day 118's `PartnerMatchingStrategy` (with both `NearestPartnerStrategy` and `HighestRatedPartnerStrategy`) gets wired as actual Spring beans rather than plain Java objects:

```java
public interface PartnerMatchingStrategy { /* ... */ }

@Service("nearestPartner")
public class NearestPartnerStrategy implements PartnerMatchingStrategy { /* ... */ }

@Service("highestRatedPartner")
public class HighestRatedPartnerStrategy implements PartnerMatchingStrategy { /* ... */ }

@Service
public class OrderAssignmentService {
    private final PartnerMatchingStrategy strategy;

    // @Qualifier tells Spring exactly which of the two beans to inject here.
    public OrderAssignmentService(@Qualifier("nearestPartner") PartnerMatchingStrategy strategy) {
        this.strategy = strategy;
    }
}
```

`@Primary` is the alternative fix: mark one implementation as the default so plain injection works without a `@Qualifier` everywhere, and only use `@Qualifier` at the specific spots that need the *other* one.

**`@PostConstruct` and `@PreDestroy` — hooking into a bean's full lifecycle.** Part 3 mentions that the container manages a bean's "creation and lifecycle," but only actually explained creation — this is the rest of that claim. `@PostConstruct` marks a method that runs once, automatically, right after Spring finishes injecting a bean's dependencies but before it's handed out to anything else. `@PreDestroy` marks a method that runs once during a graceful shutdown.

```java
@Component
public class KafkaConnectionManager {

    @PostConstruct
    public void init() {
        // runs once, right after construction and DI — a natural place
        // to open a connection, warm a cache, or validate config
    }

    @PreDestroy
    public void cleanup() {
        // runs once during shutdown — release connections, flush buffers
    }
}
```

Flagged honestly: this pair isn't referenced by name anywhere in the day-by-day plan, so it isn't a "you'll need this on Day X" gap the way Part 5 and `@Transactional` are. It's included because it completes Part 3's own claim about lifecycle management, and because "what runs when a bean is created or destroyed" is a fair, common interview question about the container itself.

---

## Part 10: Connecting This Forward — Why This Primer Matters for the Rest of the Plan

- **Day 34 (`@RestController`, `@GetMapping`):** these are just stereotype annotations and routing metadata on top of everything explained above — `TaskController` is a bean, discovered by component scanning, wired by DI, invoked by the embedded server when a matching HTTP request arrives, with Part 5's binding annotations pulling the actual data out of that request.
- **Day 36 (`JpaRepository<Task, Long>`):** this is the one piece that looks like real magic without this primer. You write an *interface* with no method bodies, and it works. What's actually happening: Spring Data JPA generates a real implementing class for that interface **at runtime**, using a dynamic proxy, and registers *that* generated object as the bean. You're not missing a class file somewhere — it genuinely doesn't exist as source code you'll ever see.
- **Day 46 (Mockito, `@InjectMocks`):** this only works cleanly *because* `TaskService` uses constructor injection. Mockito can construct a `TaskService` by hand, passing in a mock `TaskRepository` through the exact same constructor Spring would normally use — proof that constructor injection isn't just a style preference, it's what makes a class testable in isolation at all.
- **Day 48 (Kafka producer on task creation) and Day 130 (Payment idempotency/ledger):** both are exactly the kind of multi-step write that raises the "what if this fails halfway through" question `@Transactional` answers for a single service's database — and exactly the case where it's worth pausing to ask whether the operation is actually single-service (reach for `@Transactional`) or cross-service (reach for Saga, Day 70's territory, not this one).
- **Day 51 (Spring Cloud Config, `dev`/`prod` profiles):** profiles are Spring's mechanism for telling the container "build *this* set of beans in this environment, a different set in that one" — the same bean-management system from Part 3, expressed through the `@Value`-based configuration injection from Part 2, just conditioned on which profile is active.
- **Day 62 (Spring AOP):** advice "wraps" a bean's method calls — this only makes sense once you know the object being wrapped is a container-managed bean in the first place, not a plain object you created yourself. (It's also, not coincidentally, the actual mechanism `@Transactional` itself is built on.)
- **Day 65 and Day 118 (Feign fallbacks, Strategy pattern with multiple implementations):** the moment more than one class implements the same interface and both need to be real Spring beans, `@Qualifier`/`@Primary` from Part 9 is what tells the container which one you mean at each injection point.

---

## Coding Exercise

This mirrors what Week 6, Day 6 already had you do by hand — now watch Spring do it automatically.

1. **Without Spring:** write a `NotificationService` class that manually constructs and uses an `EmailSender` (`new EmailSender()` inside the class, as in Part 1). Call it from a plain `main` method.
2. **With Spring:** create a tiny new Spring Boot project (Spring Initializr, "Web" dependency only — no need to connect it to `todo-api`). Rewrite both classes:
   - `EmailSender` as a `@Component`.
   - `NotificationService` as a `@Service`, receiving `EmailSender` via constructor injection — no `new` anywhere inside `NotificationService`.
   - Add a `@RestController` with two endpoints: a `POST /notifications` that takes a `@RequestBody` and calls `notificationService.send(...)`, and a `GET /notifications/{id}` that takes a `@PathVariable` and returns a hardcoded confirmation string — deliberately exercising both binding styles from Part 5, not just one.
3. Run it, hit both endpoints with `curl`, and confirm they work — then add a `System.out.println` inside `EmailSender`'s constructor and notice it only prints **once**, at startup, no matter how many times you hit either endpoint. That's the singleton-scoped bean from Part 3, made visible.

**Definition of done:** you can explain, out loud, without notes: what a bean is, what the `ApplicationContext` does at startup, why constructor injection is preferred, what the three things Spring Boot adds on top of plain Spring actually are, and the difference between `@RequestBody`, `@PathVariable`, and `@RequestParam`.

---

## Quick Reference

| Term | One-line definition |
|---|---|
| IoC (Inversion of Control) | Object creation is handed to a container instead of being done by the class itself |
| DI (Dependency Injection) | The technique that achieves IoC — dependencies are passed in from outside |
| Bean | An object whose lifecycle is managed by the Spring container |
| ApplicationContext | Spring's IoC container — creates, wires, and holds beans |
| Component Scanning | The startup process of finding stereotype-annotated classes to register as beans |
| `@Component` / `@Service` / `@Repository` / `@RestController` | Stereotype annotations marking a class as a bean, with role-specific meaning |
| `@Autowired` | Explicit injection marker — optional on single-constructor classes in modern Spring |
| `@Value` | Injects a configuration value (from `application.yml`/`.properties`) rather than a bean |
| `@RequestBody` | Deserializes the incoming HTTP request's JSON body into a method parameter |
| `@PathVariable` | Binds a value embedded in the URL path (e.g., `/tasks/{id}`) to a method parameter |
| `@RequestParam` | Binds a query-string value (e.g., `?prefix=X`) to a method parameter |
| `@Transactional` | Groups multiple DB operations into one atomic unit — all succeed, or all roll back |
| `@Qualifier` / `@Primary` | Disambiguates which bean to inject when multiple beans implement the same interface |
| `@PostConstruct` / `@PreDestroy` | Marks a method to run right after bean creation, or right before shutdown |
| Spring Boot | An opinionated layer adding auto-configuration, starter dependencies, and an embedded server on top of plain Spring |
| `@SpringBootApplication` | Combines `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` into one annotation |

### Daily Deliverable
- [ ] Can explain IoC and DI in your own words, without notes.
- [ ] Can explain what a bean and the `ApplicationContext` are, without notes.
- [ ] Can explain the difference between `@RequestBody`, `@PathVariable`, and `@RequestParam`, with a correct example of each.
- [ ] Can explain what `@Transactional` guarantees, and why it's a different tool from Day 70's Saga pattern, not a smaller version of it.
- [ ] Can name the three things Spring Boot adds on top of plain Spring.
- [ ] Coding exercise (manual wiring vs. Spring-managed wiring, both binding styles) complete, singleton behavior observed and understood.
- [ ] Ready for Day 34 with the mental model in place, not just the syntax ahead.
