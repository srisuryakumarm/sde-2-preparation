# Day 70 — Missing Primer: Spring Security Essentials

**Do this first on Day 70 (Sunday; ≈ 60 min), then continue with Day 70's regular plan.**
**Needed by:** SDE-2 backend interviews ("how does Spring Security work?"), and the design choice behind Day 67 — you wrote a *custom* JWT filter at the Gateway; this explains the standard framework you did not use and when you would.
**Why it is missing:** Spring Security appears nowhere in the repo. The plan secures the platform with a hand-written Gateway filter only.
**Already in place:** web security fundamentals (Day 63's file), JWT/Auth primer and JJWT (Day 67), servlet request path (Day 32), `ThreadLocal` (Day 42's file), beans and DI (Day 31).

---

## 1. The mental model
Spring Security is **a chain of servlet filters** in front of your controllers. A request passes `FilterChainProxy` → one `SecurityFilterChain` → its filters (authenticate, authorize, CSRF, CORS, headers…) → `DispatcherServlet`. If a filter rejects the request, your controller never runs.
- **`Authentication`** — who the caller is and their authorities (roles). Stored in the **`SecurityContext`**, reachable via `SecurityContextHolder` — a **`ThreadLocal`** (so it is not automatically visible in threads you start yourself).
- **`AuthenticationManager` / `AuthenticationProvider`** — verify credentials. **`UserDetailsService`** loads a user; **`PasswordEncoder`** (BCrypt) checks the password.
- **401** = not authenticated (who are you?). **403** = authenticated but not allowed.

## 2. Configuration: a `SecurityFilterChain` bean
```java
@Configuration
@EnableMethodSecurity
class SecurityConfig {
    @Bean
    SecurityFilterChain api(HttpSecurity http) throws Exception {
        http.csrf(csrf -> csrf.disable())                                   // stateless token API: no cookie session to forge
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(a -> a
                .requestMatchers("/health", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated())
            .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()));   // validate Bearer JWTs
        return http.build();
    }
}
```
- Adding the security starter alone locks **everything** (HTTP Basic with a generated password printed at startup).
- **Resource server:** tick *OAuth2 Resource Server* in Initializr and copy the starter name (on Boot 4 it may carry a `security-` prefix). Give it a key or issuer: `spring.security.oauth2.resourceserver.jwt.issuer-uri` (or `jwk-set-uri`).
- **Method security:** `@PreAuthorize("hasRole('ADMIN')")`, and ownership checks (`#id == authentication.name`) — the IDOR defense from Day 63.
- **CORS:** declare a `CorsConfigurationSource` bean, don't scatter `@CrossOrigin`.
- **Testing:** `spring-security-test`, `@WithMockUser`, `.with(jwt())` on `MockMvc`.

## 3. Gateway filter vs per-service security
Validating JWTs only at the Gateway (Day 67) means a service reachable on its own port is **unprotected**. Production systems usually do both: the Gateway rejects early; each service is also a resource server (**defense in depth**), or the mesh enforces mTLS (Day 138). Know this trade-off — it is the interview answer.

## 4. Common mistakes
Leaving `csrf` on for a stateless API (every POST → 403); permitting `anyRequest()` by accident; matching `/admin/*` instead of `/admin/**`; assuming `SecurityContext` exists inside `@Async` code; returning stack traces; storing the JWT secret in the repo.

## 5. Exercise (45 min, a scratch Spring project)
1. Add the security starter, start the app, and hit an endpoint: read the generated password and the `401`.
2. Add a `SecurityFilterChain` permitting `/health`, requiring auth elsewhere, with an in-memory user whose password is BCrypt-encoded; test `401`, `200`, and a `403` using `@PreAuthorize("hasRole('ADMIN')")`.
3. Write down (5 lines) how you would switch to JWT resource-server validation and what the Gateway would still do.

## Checklist
- [ ] Can describe the filter chain and where `Authentication` lives.
- [ ] Can say 401 vs 403 and write a minimal `SecurityFilterChain`.
- [ ] Can explain why CSRF is disabled for a stateless API.
- [ ] Can argue Gateway-only vs defense-in-depth. Continue with Day 70's regular plan.
