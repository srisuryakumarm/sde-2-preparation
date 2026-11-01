# Day 77 — Missing Primer: OAuth 2.0, OpenID Connect and Single Sign-On (SSO)

**Do this on Day 77 (Sunday, ≈ 60 min), then continue with Day 77's regular plan.**
**Needed by:** Day 105B (Rippling's products are identity-heavy — SSO, provisioning, permissions), Days 120–133 (HLD: "design the auth service", "log in with Google", service-to-service auth), **Day 144 (multi-tenant SaaS — enterprise customers will not buy without SSO)**, and SDE-2 backend loops, where *"explain the OAuth 2.0 authorization-code flow"* and *"what is the difference between OAuth and OIDC?"* are standard. It is also the missing explanation behind Day 70's `issuer-uri` / `jwk-set-uri` properties.
**Why it is missing:** Day 67 only states that JWT ≠ OAuth 2.0; the JWT primer scopes OAuth out ("deeper than this plan goes"); Day 70's Spring Security primer uses `oauth2ResourceServer` without teaching what an authorization server is. The terms *authorization code*, *PKCE*, *scope*, *OIDC*, *SSO*, *SAML* and *SCIM* are taught nowhere in the 148 books.
**Already in place:** authentication vs authorization and JWT structure / signatures (Day 67's file), web-security basics — cookies, CSRF, CORS, TLS (Day 63's file), Spring Security and resource-server validation (Day 70's file), HTTP redirects (Day 32's file), hashing (Day 67's file).

---

## 1. The problem OAuth 2.0 solves
**Delegated authorization:** let app A act on a user's data held by service B **without giving A the user's password** — a valet key that opens the car but not the glove box. (Not a login protocol — see §4.)
| Role | Example |
|---|---|
| **Resource Owner** | the user |
| **Client** | the app that wants access (your web app) |
| **Authorization Server (AS)** | issues tokens after the user consents (Google, Okta, Keycloak) |
| **Resource Server (RS)** | the API that holds the data and validates tokens (your Gateway / services) |
**Vocabulary:** **access token** — short-lived (minutes), sent as `Authorization: Bearer <token>`, may be opaque or a JWT; **refresh token** — long-lived, used only against the AS to get new access tokens (store it securely, rotate it); **scope** — a permission the client asks for (`orders:read`); **client id / secret**; **redirect URI** — where the AS sends the user back (must **exactly match** a registered value).

## 2. The grants (flows) — which one when
| Grant | Use for | Note |
|---|---|---|
| **Authorization Code + PKCE** | web apps, SPAs, mobile — **anything with a user** | the default today; PKCE is now expected for *all* clients |
| **Client Credentials** | **service-to-service**, no user (order-service → payment-service) | the client authenticates itself and gets a token with its own scopes |
| Device Code | TVs / CLIs with no browser | |
| Refresh Token | get a new access token silently | |
| ~~Implicit~~, ~~Resource-Owner Password~~ | — | discouraged by current security best-practice guidance; do not use or recommend |

## 3. Authorization Code + PKCE, step by step
1. The client sends the browser to the AS: `GET /authorize?response_type=code&client_id=…&redirect_uri=…&scope=openid profile orders:read&state=<random>&code_challenge=<…>&code_challenge_method=S256`
2. The user **logs in at the AS** and consents (your app never sees the password).
3. The AS redirects the browser back: `redirect_uri?code=<one-time code>&state=<same random>`.
4. The client's **backend** calls `POST /token` with `grant_type=authorization_code`, the `code`, the `redirect_uri` and the **`code_verifier`**.
5. The AS returns an **access token** (+ refresh token, + **ID token** if `openid` was requested).
6. The client calls the API with `Authorization: Bearer <access token>`.
**Why a *code* and not the token directly?** Step 1–3 travel through the **browser URL (the front channel)** — visible in history, logs and to other tabs. The code is worthless alone; it is exchanged for tokens over a **direct server-to-server call (the back channel)**.
**`state`** is a random value you check on return — it blocks **CSRF** on the callback. **PKCE:** the client invents a random `code_verifier`, sends only its hash, `code_challenge = BASE64URL(SHA256(verifier))`, in step 1, and the raw verifier in step 4. A stolen code is useless without the verifier — essential for apps that cannot keep a secret (SPAs, mobile).

## 4. OpenID Connect (OIDC) = OAuth 2.0 + *who the user is*
OAuth alone says "this token may call that API" — it does **not** define how to learn the user's identity. **OIDC** adds exactly that on top:
- request the scope **`openid`** → you also receive an **ID token**, a **JWT** about the *login event*: `iss` (issuer), `sub` (stable user id), `aud` (your client id), `exp`, `iat`, `nonce` (replay protection), plus optional `email` / `name`;
- a **`/userinfo`** endpoint, and a **discovery document** at `<issuer>/.well-known/openid-configuration` that lists every endpoint and the **`jwks_uri`** — the public keys used to verify signatures. **This is what Spring's `issuer-uri` reads, and why a resource server can validate tokens with no shared secret** (Day 67: asymmetric signatures).
"**Log in with Google**" is OIDC. **Say it this way in an interview: *OAuth is for authorization (access to an API); OIDC is the authentication layer built on top of it.***
**Access token vs ID token** (a favourite question): the **ID token is for the client** — to learn who logged in; the **access token is for the API**. Never send an ID token to an API as a credential, and never treat an access token as proof of identity for your client.
**Validating a JWT access token at the resource server:** signature (via the JWKS), `iss`, **`aud`** (is it meant for *this* API?), `exp` / `nbf`, then scopes → authorities.

## 5. Opaque vs JWT access tokens, revocation, logout
| | Validation | Revocation |
|---|---|---|
| **JWT** (self-contained) | local — check signature + claims, no network call (Day 67) | hard: valid until `exp` → keep access tokens **short (5–15 min)**, revoke the **refresh** token |
| **Opaque** (random string) | call the AS's **introspection** endpoint (RFC 7662) | easy and immediate; costs a network call (cache it briefly) |
Refresh-token **rotation** (each use returns a new refresh token; reuse of an old one signals theft) is the modern default. For browser apps, the safest pattern is a **Backend-for-Frontend**: tokens stay on the server and the browser only holds a session cookie (`HttpOnly`, `Secure`, `SameSite` — Day 63's file), because tokens in `localStorage` are readable by any XSS.

## 6. SSO — one login, many apps
**Single sign-on:** the user authenticates **once** at a central **Identity Provider (IdP)** (Okta, Microsoft Entra ID, Google Workspace) and then enters many applications without logging in again; each app (**Service Provider, SP**) *trusts the IdP*. Two protocols you will meet:
| | **SAML 2.0** | **OIDC** |
|---|---|---|
| Format | XML assertions, browser POSTs | JSON / JWT |
| Age / use | older; dominant in large enterprises | modern; consumer logins, newer SaaS |
| A SaaS vendor's reality | **must support both** — enterprise customers bring their own IdP | |
Enterprise buyers also expect **SCIM** — an API for the IdP to **provision and de-provision users automatically** (an employee leaves → their access disappears in every app). **Multi-tenant SaaS (Day 144):** store **per-tenant IdP configuration** (issuer, metadata, certificates), route the user to the right IdP from their email domain (`@acme.com` → Acme's IdP), and map IdP groups to your roles.

## 7. Where it plugs into your platform
- The **Gateway / services are resource servers.** Day 67's self-issued JWTs become tokens from an external AS (**Keycloak** is the usual open-source one; **Spring Authorization Server** is the Spring project): configure `spring.security.oauth2.resourceserver.jwt.issuer-uri` and Spring fetches the JWKS and validates `iss`/`exp`/signature; you add the `aud` check.
- **Scopes → authorities:** Spring maps scopes to `SCOPE_…` authorities, so `@PreAuthorize("hasAuthority('SCOPE_orders:write')")` works.
- **Service-to-service calls** use **Client Credentials** (a token per service, minimal scopes) instead of passing the end user's token everywhere.
*(You do not have to run an AS for this plan — Keycloak is heavy next to Minikube on an 8 GB Mac. Know the shape; diagrams are what interviews ask for.)*

## 8. Mistakes that fail security reviews
Loose `redirect_uri` matching (open redirect → stolen codes); no `state`; Implicit / password grants; long-lived access tokens; **not validating `aud`** (a token for service X accepted by service Y); tokens in `localStorage`; treating OAuth as authentication ("anyone with a token for *any* app is logged in"); accepting a JWT whose header says `alg: none` (always pin the expected algorithm).

## 9. Exercise (≈ 25 min, no AS required)
1. **Real discovery document:** `curl -s https://accounts.google.com/.well-known/openid-configuration | python3 -m json.tool | head -40` — find `issuer`, `authorization_endpoint`, `token_endpoint`, `jwks_uri`, `scopes_supported`. (No internet? read §4 instead.)
2. **Compute a PKCE pair (verified command):**
   ```bash
   VERIFIER=$(openssl rand -base64 48 | tr -d '=+/\n' | cut -c1-64); echo "$VERIFIER"
   printf %s "$VERIFIER" | openssl dgst -sha256 -binary | openssl base64 -A | tr '+/' '-_' | tr -d '='
   ```
   The second line is the `code_challenge` (43 characters). Say which of the two values goes in step 1 and which in step 4.
3. **Draw from memory** the §3 sequence diagram for "Log in with Google" on your platform with four lifelines — Browser, Your app, Authorization Server, API — and mark the **front channel** and **back channel** hops.
4. **Answer aloud (≈ 90 s):** (a) OAuth vs OIDC; (b) access token vs ID token; (c) why the code flow beats implicit; (d) JWT vs opaque tokens — trade-offs and revocation; (e) which grant do two microservices use?
5. **Decode a token payload** (any JWT, e.g. one from Day 67's work): take the middle segment, add `=` padding until its length is a multiple of 4, and `base64 -d` it. Name the claims.

## Checklist
- [ ] Can name the four roles, access vs refresh vs ID token, and what a scope is.
- [ ] Can walk through Authorization Code + PKCE and explain `state`, the front/back channel and why PKCE exists.
- [ ] Can say OAuth vs OIDC and pick the right grant for user logins vs service-to-service.
- [ ] Can explain how a resource server validates a JWT (signature via JWKS, `iss`, `aud`, `exp`) and the revocation trade-off vs opaque tokens.
- [ ] Can explain SSO, SAML vs OIDC, SCIM, and per-tenant IdPs for a multi-tenant SaaS. Continue with Day 77's regular plan.
