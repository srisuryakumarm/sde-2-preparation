# Day 45 — Missing Primer: Which Test Annotation Tests What (Boot 4.1 edition)

**Do this first on Day 45 (≈ 25 min), then continue with Day 45's regular plan.**
**Needed by:** Day 45 (JUnit 5 + AssertJ), Day 46 (Mockito), Day 47 (Testcontainers), Day 62 onward (controller tests in each module).
**Why it is missing:** Day 45's weekly plan says "JUnit 5 tests for `TaskController` using `@DataJpaTest`". `@DataJpaTest` loads only the JPA layer — it cannot test a controller. Day 45's book already corrects this for the repository half; this file adds the controller half and the Boot 4 changes.
**Already in place:** lambdas (`assertThatThrownBy(() -> ...)`, Day 30), Task CRUD on Postgres (Day 41), Flyway (Day 40).

## 1. Four test shapes
| Shape | Annotation | Loads | Use for |
|---|---|---|---|
| Plain unit test | none (JUnit 5 + AssertJ; Mockito `@Mock`/`@InjectMocks` on Day 46) | nothing from Spring | `TaskService` rules |
| Controller slice | `@WebMvcTest(TaskController.class)` + `MockMvc` | MVC layer only (controllers, `@ControllerAdvice`, Jackson) | status codes, validation, JSON shape; `TaskService` is mocked |
| Repository slice | `@DataJpaTest` | JPA + an embedded DB (H2 by default) | `TaskRepository` queries |
| Full | `@SpringBootTest` | the whole context | a few end-to-end checks |

## 2. Boot 4 changes
- **Mock beans:** `@MockBean` → **`@MockitoBean`** (and `@SpyBean` → `@MockitoSpyBean`). Plain Mockito tests still use `@Mock`/`@InjectMocks`.
- **Test starters are per technology:** Initializr adds a matching `…-test` starter for each starter you tick (e.g. `spring-boot-starter-webmvc-test`, `spring-boot-starter-data-jpa-test`). If you added a starter by hand (you did, on Day 36), add its test starter too — or copy the coordinates from a throwaway Initializr project. Import paths of `@WebMvcTest`/`@DataJpaTest` also moved; let IntelliJ import them.
- H2 for `@DataJpaTest`: `com.h2database:h2`, test scope.

## 3. Controller test shape
```java
@WebMvcTest(TaskController.class)
class TaskControllerTest {
    @Autowired MockMvc mockMvc;
    @MockitoBean TaskService service;                 // Boot 3.x: @MockBean

    @Test void missingTaskIs404() throws Exception {
        when(service.get(99L)).thenThrow(new NotFoundException("task 99"));
        mockMvc.perform(get("/tasks/99")).andExpect(status().isNotFound());
    }
}
```
Your `ApiExceptionHandler` is part of the slice, so the `404` mapping from Day 33 is exercised too.

## 4. The Flyway-on-H2 trap (new since Day 40)
`@DataJpaTest` swaps your datasource for H2, **and Flyway then runs your Postgres migration against H2**. If `V1` uses anything H2 rejects, the context fails to start. Quick fix for today only: in `src/test/resources/application.yml` set `spring.flyway.enabled: false` and `spring.jpa.hibernate.ddl-auto: create-drop`. Day 47 removes the problem properly by testing against real Postgres.

## Checklist
- [ ] Can say which annotation tests a controller, a repository and a service — and why `@DataJpaTest` cannot test a controller.
- [ ] One `@WebMvcTest` (404 path) and one `@DataJpaTest` (`findByStatus`) pass with `./mvnw test`.
- [ ] Continue with Day 45's regular plan.
