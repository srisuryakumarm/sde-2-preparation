# Day 62 — Missing Primer: A Multi-Module Parent POM That Actually Builds on Boot 4.1

**Do this first on Day 62 (≈ 30 min), then continue with Day 62's regular plan.**
**Needed by:** Day 62's Project Block (multi-module Maven project: Product, Order, Payment, Notification), and every project day after it.
**Why it is missing:** Day 62's book explains multi-module Maven well (`<packaging>pom</packaging>`, `<modules>`, `<dependencyManagement>` vs `<dependencies>`), but its parent POM has no Spring Boot parent or BOM and pins `spring-boot-starter-aop` to `3.3.0`. A child that asks for `spring-boot-starter-webmvc` then has no version (`dependencies.dependency.version is missing`), and on Boot 4 the AOP starter is `spring-boot-starter-aspectj` and must not be pinned.
**Already in place:** the Day 62 book's multi-module explanation, Maven wrapper and pom anatomy (Day 31), the Boot version decision (Day 34), microservices orientation (Day 61).

## 1. Parent POM (repo root)
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.1.x</version>                 <!-- same patch you pinned on Day 34 -->
    <relativePath/>
  </parent>
  <groupId>com.yourname.ecommerce</groupId>
  <artifactId>scalable-ecommerce-platform</artifactId>
  <version>1.0.0</version>
  <packaging>pom</packaging>                 <!-- a parent has no code of its own -->
  <properties><java.version>21</java.version></properties>
  <modules>
    <module>product-service</module>
    <module>order-service</module>
    <module>payment-service</module>
    <module>notification-service</module>
  </modules>
  <!-- Day 63 adds the Spring Cloud BOM import under <dependencyManagement> -->
</project>
```
Because the root inherits `spring-boot-starter-parent`, **every child gets Boot's version management for free** — no version pins on Boot starters, ever.

## 2. A child module (e.g. `order-service/pom.xml`)
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.yourname.ecommerce</groupId>
    <artifactId>scalable-ecommerce-platform</artifactId>
    <version>1.0.0</version>
    <relativePath>../pom.xml</relativePath>
  </parent>
  <artifactId>order-service</artifactId>
  <dependencies>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc</artifactId></dependency>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-aspectj</artifactId></dependency> <!-- Day 62 AOP, Day 64 Resilience4j -->
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-webmvc-test</artifactId><scope>test</scope></dependency>
  </dependencies>
  <build><plugins>
    <plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin>
  </plugins></build>
</project>
```
Add the AspectJ starter only to modules that use AOP. Check every coordinate against a throwaway Initializr project (Day 31's trick) if one does not resolve.

## 3. Make each module a runnable service
Each module needs its own `@SpringBootApplication` main class and its own port: `server.port: 8081` … `8084` in each `application.yml`. Create them by generating each module at start.spring.io (to get the main class and a clean pom), moving the folder under the root, and replacing its `<parent>` as in §2. Keep **one** Maven wrapper (`mvnw`, `.mvn/`) at the root.

## 4. Commands
```bash
./mvnw -q test                                        # build and test every module
./mvnw -pl order-service -am test                     # only order-service (-pl) plus modules it depends on (-am)
./mvnw -pl order-service spring-boot:run              # run one service
```

## Checklist
- [ ] `./mvnw test` builds all four modules; each service starts on its own port.
- [ ] No Boot starter has a `<version>`; AspectJ starter only where AOP is used.
- [ ] Can explain `<dependencies>` vs `<dependencyManagement>` (Day 62's book) **and** why the root inherits the Boot parent.
- [ ] Continue with Day 62's regular plan (AOP theory and exercise).
