# Micro-Primer: Maven Fundamentals — Needed Before Day 34

**First needed:** Day 34, when Spring Initializr generates your first Maven-based project. Explicitly used again Day 45 (`mvn test`) and Day 62 (multi-module Maven project).

Maven is a build tool and dependency manager for Java projects. It downloads the libraries your code depends on, compiles your code, runs your tests, and packages the result — all driven from one command-line tool and one configuration file, instead of you managing JAR files and classpaths by hand.

**`pom.xml`** ("Project Object Model") is that configuration file. It lists your project's dependencies by name and version — for example, `spring-boot-starter-web` — and Maven downloads them automatically from a central public repository the first time you build, then caches them locally so it doesn't re-download on every build after.

Spring Initializr (Day 34) generates a working `pom.xml` for you, pre-filled with whichever dependencies you selected. You won't write one from scratch, but it's worth opening it and recognizing what you're looking at: a list of your actual dependencies, not magic.

**The build lifecycle commands you'll actually run:**
- `mvn compile` — just compile the code.
- `mvn test` — compile, then run your tests. This is Day 45's exact command.
- `mvn package` — compile, test, and bundle everything into a runnable `.jar`.

**Deferred:** nothing later in this plan needs deeper Maven knowledge than this. Day 62's "multi-module Maven project" is a natural extension once this lands — each module gets its own `pom.xml`, plus one parent `pom.xml` tying them together — and that structure will make sense on sight once you know what a single-module `pom.xml` already represents.

### Checklist
- [ ] Can explain what `pom.xml` is for, in one sentence.
- [ ] Can state what `mvn test` actually does (not just that it "runs tests").
- [ ] Ready for Day 34.
