# Day 31 — Missing Primer: Spring Core — Inversion of Control, Dependency Injection, Beans

**Do this first on Day 31 (≈ 2 h, after the Maven primer), then continue with Day 31's regular plan.**
**Needed by:** Day 34 (every annotation in your first controller), Days 36–48 (repositories, services, Kafka beans), Day 62 (AOP), Day 64+ (circuit breakers, Feign).
**Supersedes:** Parts 1–4, 6, 8 of `Day34_Missing_Spring_and_Spring_Boot_Primer.md` (kept, tightened, corrected).
**Already in place:** interfaces and the SOLID/Dependency-Inversion refactor of `NotificationService` (**Week 1, Day 6** — v1 of this primer said "Week 6"), lambdas (Day 30), `Optional` (Day 30), the sandbox project (Maven primer).

---

## 1. The problem and the idea
On Day 6 you changed `NotificationService` from `new EmailSender()` inside the class to receiving a `MessageSender` through its constructor. Spring is that refactor, automated across the whole application.

- **IoC (Inversion of Control):** a class stops *creating* its collaborators and only *declares* what it needs.
- **DI (Dependency Injection):** the technique — collaborators are handed in from outside.
- **Bean:** an object whose creation and lifecycle Spring manages. **`ApplicationContext`:** the container — at startup it finds bean classes, creates them in dependency order, injects them, and holds them until shutdown.
- **Default scope = singleton:** one instance **per container**, shared by every caller. Consequences: (1) singletons are created **eagerly at startup**, so wiring mistakes fail at startup, not on the first request; (2) many request threads use the same instance (Day 29), so **a mutable field in a bean is a concurrency bug**.

## 2. Three injection styles
| Style | Verdict |
|---|---|
| **Constructor** | use this: fields can be `final`, no half-built objects, trivial to test (Day 46's `@InjectMocks` relies on it). A class with one constructor needs no `@Autowired`. |
| Setter | only for genuinely optional dependencies |
| Field (`@Autowired` on a field) | seen in old code; hides dependencies and blocks `final` — avoid |

## 3. Telling Spring what to manage
- **Stereotypes** — all register a bean; the name states the role: `@Component` (generic), `@Service` (business logic), `@Repository` (data access), `@RestController` (HTTP layer, returns JSON). **Spring Data repository interfaces need no `@Repository`** (v1 added it needlessly): `interface TaskRepository extends JpaRepository<Task, Long>` is detected and implemented automatically on Day 36.
- **`@Configuration` + `@Bean`:** a bean you build by hand, for classes you cannot annotate (a `Clock`, a `RestClient`, a Kafka template).
- **Component scanning:** Boot scans the package of your main class **and everything below it**. A controller in a sibling package is silently ignored → `404`. Keep the main class in the root package.
- **`@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`.**
- **Choosing among several beans of one type:** `@Primary` wins; else `@Qualifier("beanName")`; else Spring quietly falls back to matching the *parameter name* with a bean name. Be explicit; do not rely on the fallback. Default bean name = class name with a lowercase first letter (`SmsSender` → `smsSender`).
- **Config values:** `@Value("${app.greeting:Hello}")` (default after the colon). `@ConfigurationProperties` is the typed alternative once you have many (Day 51).
- **Lifecycle hooks:** `@PostConstruct` runs once after injection, `@PreDestroy` at shutdown. Recognize them; you rarely write them.
- **Circular constructor dependencies** (A needs B needs A) fail at startup. Redesign; do not "fix" with field injection.

## 4. Exercise A — "NotificationService, Spring edition" (60–75 min, in `spring-sandbox`)
1. `interface MessageSender { void send(String to, String msg); }` plus `@Component EmailSender` and `@Component SmsSender`; each prints a line in its **constructor**.
2. `@Service NotificationService` with **one constructor** taking `MessageSender sender` — name the parameter `sender`, not `emailSender`.
3. Add a runner so no web layer is needed:
   ```java
   @Bean CommandLineRunner run(NotificationService service) {      // @Bean method parameters are injected too
       return args -> service.notifyUser("sri", "hello");
   }
   ```
4. Start the app. It **fails** with `NoUniqueBeanDefinitionException`. Read the whole message. Fix it twice: `@Primary` on `EmailSender`; then remove that and use `@Qualifier("smsSender")` on the parameter.
5. **Singleton proof:** the constructor prints appear **once**. Inject `ApplicationContext` into the runner and print `ctx.getBean(EmailSender.class) == ctx.getBean(EmailSender.class)` → `true`.
6. `@Configuration` class exposing `@Bean Clock clock() { return Clock.systemUTC(); }`; inject it into a `TimeService` instead of calling `Instant.now()`. (This is *why* DI helps testing: a test can inject a fixed clock.)
7. Read the property `app.greeting` with `@Value("${app.greeting:Hello}")`. Put `app.greeting=Hi` in `application.properties` (Initializr's default file — you will convert to YAML on Day 36); delete the line and watch the default take over.

## 5. Stretch
- **MiniSpring in ~50 lines (40 min):** a `@MyComponent` annotation (`@Retention(RUNTIME)`), a `Container` that takes a list of classes, and for `getBean(X.class)` picks the single constructor, recursively resolves each parameter type, caches instances (the cache *is* the singleton scope) and throws a clear error on a cycle. After this Spring stops being magic — and Day 62's custom `@LogExecutionTime` annotation will make sense.
- **Singleton thread-safety made visible (15 min):** a controller with `private int hits;` incremented per request; fire `seq 200 | xargs -P 20 -I{} curl -s localhost:8080/hits >/dev/null`; the count comes out low; fix with `AtomicInteger`.

## Checklist
- [ ] Explain IoC vs DI, and why constructor injection, in 60–90 s, out loud.
- [ ] Explain bean, `ApplicationContext`, singleton — and the thread-safety consequence.
- [ ] Say what `@SpringBootApplication` expands to and where component scanning stops.
- [ ] Exercise A done, including the deliberate `NoUniqueBeanDefinitionException`; pushed.
- [ ] Continue with Day 31's regular plan.
