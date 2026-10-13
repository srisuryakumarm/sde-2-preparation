# Day 65 — Missing Primer: Reactor Basics for the Gateway, and Which Gateway You Are Building

**Do this first on Day 65 (≈ 75 min), before Day 66 sets up the Gateway, then continue with Day 65's regular plan.**
**Needed by:** Day 66 (routes), Day 67 (JWT `GlobalFilter` returning `Mono<Void>`), Day 68 (rate-limit filter).
**Why it is missing:** Day 67's book shows a filter that returns `Mono<Void>` and nothing explains `Mono`; the Concurrency primer even says reactive programming "never appears anywhere in this plan". The Gateway is the one place it does.
**Already in place:** lambdas (Day 30), threads and blocking (Day 29), the platform's multi-module build (Day 62), Spring Cloud BOM (Day 63's file).

## 1. Which Gateway? (verified for Spring Cloud 2025.1 / Gateway 5.x)
Two independent flavors exist:
| Flavor | Starter | Routes configured under |
|---|---|---|
| **WebFlux** (reactive, Netty) | `spring-cloud-starter-gateway-server-webflux` | `spring.cloud.gateway.server.webflux.routes` |
| WebMVC (servlet) | `spring-cloud-starter-gateway-server-webmvc` | `spring.cloud.gateway.server.webmvc.routes` |

The old `spring-cloud-starter-gateway` and the root `spring.cloud.gateway.routes` that Day 66's YAML uses are the pre-2025 names. **Use WebFlux** (Day 66–68's filters are written for it) and change Day 66's YAML root to `spring: cloud: gateway: server: webflux: routes:`.
Create `api-gateway` as its own fifth module containing **only** the gateway starter: a WebFlux gateway refuses to start if Spring MVC (`spring-boot-starter-webmvc`) is also on the classpath. The WebMVC flavor writes filters in servlet style with a different API than the book's; switch only deliberately.

## 2. Why reactive here
A gateway spends its life waiting on other services. Non-blocking I/O lets a few event-loop threads serve thousands of connections — **as long as nothing blocks one of those threads.** One `Thread.sleep` or JDBC call in a filter stalls every in-flight request.

## 3. `Mono` and `Flux` in one page
- `Mono<T>` = a value that will arrive later (0 or 1). `Flux<T>` = 0..N values. `Mono<Void>` = "finished, no value".
- **A `Mono` is a recipe.** Nothing happens until something subscribes; the framework subscribes to whatever your method returns. So: build the chain and **return it**. Ignoring a returned `Mono` means nothing runs.
- Operators you need: `map` (transform the value), `flatMap` (value → another `Mono`), `then(next)` (when this completes, continue with `next`), `Mono.fromRunnable(() -> ...)` (run a side effect as a `Mono`), `Mono.just(x)` / `Mono.empty()`.
- Never `.block()` inside a filter; never `Thread.sleep`.

## 4. A filter, annotated
```java
@Component
public class ApiKeyFilter implements GlobalFilter, Ordered {
    private static final Logger log = LoggerFactory.getLogger(ApiKeyFilter.class);

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String key = exchange.getRequest().getHeaders().getFirst("X-Demo");
        if (key == null) {                                               // "pre" logic: decide before forwarding
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();                 // stop here: answer 401 and finish
        }
        long start = System.nanoTime();
        return chain.filter(exchange)                                    // continue to the next filter / the backend
                .then(Mono.fromRunnable(() ->                            // "post" logic: runs after the response completes
                        log.info("{} took {} ms", exchange.getRequest().getPath(),
                                 (System.nanoTime() - start) / 1_000_000)));
    }

    @Override public int getOrder() { return -1; }                      // lower number runs earlier
}
```
`GlobalFilter` applies to every route; a `GatewayFilter` is attached to specific routes. Day 67's JWT check and Day 68's rate limiter have exactly this shape: look at a header → either `setComplete()` with `401`/`429` or `chain.filter(exchange)`.

## 5. Exercise (≈ 35 min)
1. Create `api-gateway` (WebFlux starter only) with one route to `product-service` (use the `server.webflux` YAML root).
2. Add `ApiKeyFilter`. `curl -i` the route without the header (expect `401`), then with `-H 'X-Demo: 1'` (expect the backend's answer).
3. Confirm the timing log line appears. Then deliberately add a `Thread.sleep(100)` in `filter`, hit it with a few parallel requests, and observe why it is forbidden here (latency stacks up on the event loop). Remove it.

## Checklist
- [ ] Can say what `Mono<Void>` means and why you must return the chain.
- [ ] Can name the WebFlux starter and the `server.webflux.routes` YAML root.
- [ ] Can explain why a blocking call in a filter is harmful.
- [ ] Filter working. Continue with Day 65's regular plan (Feign), then Day 66.
