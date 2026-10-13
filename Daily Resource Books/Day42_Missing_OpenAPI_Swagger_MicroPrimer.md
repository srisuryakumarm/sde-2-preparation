# Day 42 — Missing Primer: OpenAPI / Swagger UI for `todo-api`

**Do this first on Day 42 (Sunday; ≈ 45 min), then continue with Day 42's regular plan.**
**Needed by:** Day 139 — its README checklist expects "API docs: Swagger UI location and path" for the platform; nothing in the plan builds it.
**Already in place:** the full Task CRUD on Postgres (Days 35–41).

## 1. What it is
**OpenAPI** is a machine-readable description of your HTTP API (paths, parameters, bodies, responses). **Swagger UI** is a web page that renders that description and lets you call the endpoints. The `springdoc-openapi` library generates the description from your controllers and DTOs — you add one dependency and no code.

## 2. Add it (Boot 4.x)
```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>3.0.x</version>      <!-- springdoc 3.0.x is for Boot 4; Boot 3.5 would use 2.8.x. Not managed by the Boot BOM. -->
</dependency>
```
(Pick the latest 3.0.x release on Maven Central.) Start the app and open **`/swagger-ui.html`** (it redirects to the UI) and **`/v3/api-docs`** (the raw JSON).

## 3. Make it useful (20 min)
- Try every endpoint from the UI, including a validation failure and a `404`.
- Add `@Operation(summary = "...")` to three methods; confirm they show up. Do not annotate everything — noise hides signal.
- Check that your `ProblemDetail` errors appear (or add `@ApiResponse` for `404`/`400` on one endpoint).
- Screenshot the UI into `todo-api/docs/` and link it from the README ("API docs at `/swagger-ui.html`").

## 4. Later
On the platform (Day 62+), every module is its own app: each exposes its own `/swagger-ui.html` on its own port. Behind the Gateway you will not see them under one URL unless you route to them — list the per-module URLs in the README instead (that is what Day 139's checklist needs).

## Checklist
- [ ] `/swagger-ui.html` and `/v3/api-docs` load; you called an endpoint from the UI.
- [ ] README links the docs path; screenshot committed.
- [ ] Continue with Day 42's regular plan.
