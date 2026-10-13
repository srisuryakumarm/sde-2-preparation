# Day 63 — Missing Primer: Spring Cloud's Release Train and Resilience4j on Boot 4.1

**Do this first on Day 63 (Sunday; ≈ 30 min), then continue with Day 63's regular plan.**
**Needed by:** Day 64 (Resilience4j circuit breaker), Day 65 (Feign), Day 66 (Gateway), Day 95 (Eureka).
**Why it is missing:** Spring Cloud is versioned separately from Spring Boot (a "release train" with its own BOM), and a mismatch causes confusing startup errors. The plan names Resilience4j, Feign and Gateway but never says how to add them to a Boot 4.1 build.
**Already in place:** multi-module parent POM (Day 62's file), microservices orientation (Day 61).

## 1. The Spring Cloud BOM
Boot 4.x pairs with the **Spring Cloud `2025.1.x` ("Oakwood")** train (2025.1.2 added compatibility with Boot 4.1.0). Do not type the version from memory: generate a throwaway project at start.spring.io with your Boot 4.1.x and tick **Spring Cloud Gateway**; open its `pom.xml` and copy two things into the platform **root** POM — the `<spring-cloud.version>` property and this block:
```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>${spring-cloud.version}</version>
      <type>pom</type>
      <scope>import</scope>              <!-- imports a BOM; children then add Spring Cloud starters without versions -->
    </dependency>
  </dependencies>
</dependencyManagement>
```

## 2. Resilience4j in `order-service` (Day 64)
```xml
<dependency>
  <groupId>io.github.resilience4j</groupId>
  <artifactId>resilience4j-spring-boot4</artifactId>
  <version>2.4.0</version>       <!-- 2.4.0 or newer: the Boot 4 artifact exists from 2.4.0 and the Resilience4j BOM omitted it, so give it a version -->
</dependency>
<!-- also in order-service: spring-boot-starter-aspectj (annotations work through AOP) and spring-boot-starter-actuator (circuit-breaker health/metrics) -->
```
Boot 3 tutorials say `resilience4j-spring-boot3` — do not use it on Boot 4. The annotation and the configuration keys are unchanged:
```yaml
resilience4j:
  circuitbreaker:
    instances:
      productService:
        sliding-window-size: 5
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
```
```java
@CircuitBreaker(name = "productService", fallbackMethod = "fallback")
public Product fetch(String id) { ... }
private Product fallback(String id, Throwable t) { ... }     // same parameters + a Throwable
```
Because `@CircuitBreaker` works through a proxy, the same self-invocation rule from `@Transactional` (Day 41) applies: calling it from another method in the same class bypasses it.

## 3. Feign (Day 65)
`spring-cloud-starter-openfeign` ships in the same train (version from the BOM). Add `@EnableFeignClients` to the main class and declare `@FeignClient(name = "product-service", url = "http://localhost:8081")` until Eureka replaces the hard-coded URL on Day 95. (Spring's own `@HttpExchange` clients are an alternative; keep Feign — it is what the plan and most interviewers expect.)

## Checklist
- [ ] Spring Cloud BOM imported in the root POM, version copied from Initializr.
- [ ] `order-service` has `resilience4j-spring-boot4` (explicit version), the AspectJ starter and Actuator.
- [ ] Can say why Spring Cloud has its own version and BOM.
- [ ] Continue with Day 63's regular plan.
