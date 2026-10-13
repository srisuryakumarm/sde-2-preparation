# Day 31 — Missing Primer: Maven, the Wrapper, and How to Read a `pom.xml`

**Do this first on Day 31 (≈ 60 min), then continue with Day 31's regular plan.**
**Needed by:** Day 34 (first Spring Boot project), Day 45 (`./mvnw test`), Day 62 (multi-module project), Day 139 (CI).
**Supersedes:** `Day34_Missing_Maven_Fundamentals_MicroPrimer.md` — still correct in spirit, but it says dependencies are listed "by name and version" (an Initializr pom has no starter versions) and uses `mvn`, which Week 1 never installed.
**Already in place:** JDK 21 and IntelliJ (Day 1).

---

## 1. What Maven does
Maven reads one file, `pom.xml`, and (1) downloads the libraries you depend on from Maven Central into `~/.m2/repository` (cached after the first download), (2) compiles `src/main/java`, (3) runs `src/test/java`, (4) packages the result into `target/*.jar`.

Layout is a convention: `src/main/java` (code) · `src/main/resources` (config) · `src/test/java` (tests) · `target/` (build output — never commit it).

## 2. Anatomy of a Spring Boot `pom.xml`
```xml
<parent>                                      <!-- 1. inherit Boot's version management -->
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>4.1.x</version>                    <!-- the ONE place the Boot version lives -->
</parent>
<groupId>com.yourname</groupId>               <!-- 2. your own coordinates: groupId : artifactId : version -->
<artifactId>spring-sandbox</artifactId>
<version>0.0.1-SNAPSHOT</version>
<properties><java.version>21</java.version></properties>
<dependencies>
  <dependency>                                <!-- 3. no <version>: the parent's BOM supplies it -->
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>   <!-- Boot 3.x called this spring-boot-starter-web -->
  </dependency>
  <dependency> ... <scope>test</scope> </dependency>     <!-- test scope: classpath for tests only -->
</dependencies>
<build><plugins>
  <plugin> spring-boot-maven-plugin </plugin> <!-- 4. repackages the jar so `java -jar` works ("fat jar") -->
</plugins></build>
```
- **GAV** = groupId + artifactId + version: the address of any library.
- **BOM** ("bill of materials") = a list that says "if you use library X, use exactly version Y". Inheriting `spring-boot-starter-parent` is how you get Boot's BOM, which is why starters have no `<version>`.
- **Starter** = a pom with no code, only a curated list of compatible libraries. The web starter ≈ Spring MVC + embedded Tomcat + Jackson (JSON).
- **Scope**: `compile` (default), `runtime` (needed at run time, not compile time — e.g. the Postgres driver), `test`.

## 3. The wrapper — why you type `./mvnw`, not `mvn`
Week 1 installed JDK, IntelliJ and Git — not Maven. Every Initializr project ships `mvnw` and `.mvn/`: a script that downloads the exact Maven version the project wants and runs it. **Commit `mvnw`, `mvnw.cmd` and `.mvn/`.**

| Command | Does |
|---|---|
| `./mvnw spring-boot:run` | compile and start the app |
| `./mvnw test` | compile, then run all tests (Day 45's exact command) |
| `./mvnw clean package` | delete `target/`, compile, test, build the jar |
| `./mvnw -DskipTests package` | same, skipping tests (use sparingly) |
| `java -jar target/<name>.jar` | run the packaged app — no Maven needed |
| `./mvnw dependency:tree` | every library on your classpath, and why |

(If `./mvnw` says "Permission denied": `chmod +x mvnw`.)

## 4. Adding a dependency (you will do this ~15 times in this plan)
Edit `pom.xml` → in IntelliJ click *Load Maven Changes* → confirm with `./mvnw dependency:tree | grep <name>`.
**Version-proof trick:** do not trust memory or old tutorials for a dependency's exact name — generate a throwaway project at start.spring.io with the same Boot version, tick the dependency, and copy the coordinates it writes into its `pom.xml`. (Boot 4 renamed several starters; see `Day34_Missing_Initializr_Setup_and_Boot_Version_Decision_MicroPrimer.md`.)

## 5. Exercise (≈ 30 min)
1. At **start.spring.io** choose: Maven · Java · Boot **4.1.x** (latest stable, not SNAPSHOT/RC) · Jar · Java 21 · dependency **Spring Web**. Name it `spring-sandbox`. Unzip; open the folder in IntelliJ (it is its own project).
2. From the pom itself answer: where is the Boot version? why does the web starter have no version? what does the Boot plugin do?
3. Run `./mvnw test`, then `./mvnw clean package`, then `java -jar target/spring-sandbox-0.0.1-SNAPSHOT.jar`.
4. Read the startup log once, top to bottom. Find: the Boot version, "No active profile set", `Tomcat started on port 8080`, `Started SpringSandboxApplication in … seconds`. Stop the app (Ctrl-C).
5. Run `./mvnw dependency:tree` and find Tomcat and Jackson in it.

Common errors: `release version 21 not supported` → IntelliJ/terminal is using an older JDK (check `java -version` and `JAVA_HOME`). `Port 8080 was already in use` → `lsof -i :8080`.

## Checklist
- [ ] Can say in one sentence what `pom.xml` is for, and where the Boot version lives.
- [ ] Can explain why starters have no `<version>` (parent BOM) and what the Boot plugin adds.
- [ ] Ran test, package and `java -jar` with `./mvnw`; `spring-sandbox` exists and is committed (without `target/`).
- [ ] Continue with Day 31's regular plan, then the Spring Core primer.
