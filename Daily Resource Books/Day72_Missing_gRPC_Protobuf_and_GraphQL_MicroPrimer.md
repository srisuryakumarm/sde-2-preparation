# Day 72 — Missing Primer: gRPC, Protocol Buffers, and GraphQL (vs REST)

**Do this first on Day 72 (≈ 45 min, alongside the HTTP and load-balancer theory), then continue with Day 72's regular plan.**
**Needed by:** SDE-2 microservice and system-design interviews ("REST or gRPC between services?"), Day 69's Avro contrast, HLD days (service-to-service calls).
**Why it is missing:** gRPC, Protocol Buffers and GraphQL appear nowhere in the repo.
**Already in place:** REST (Days 32–35), Avro and Schema Registry (Day 69), HTTP/load balancers (Day 72's own theory), microservice orientation (Day 61's file).

---

## 1. gRPC in five lines
A **contract-first RPC framework**: you describe services and messages in a `.proto` file, generate client and server code, and call remote methods like local ones. Transport is **HTTP/2** (multiplexed streams, binary framing, header compression); payloads are **Protocol Buffers** (compact binary).
```proto
syntax = "proto3";
package product.v1;

service ProductService {
  rpc GetProduct (GetProductRequest) returns (Product);               // unary
  rpc WatchStock (GetProductRequest) returns (stream StockUpdate);    // server streaming
}
message GetProductRequest { string id = 1; }
message Product          { string id = 1; string name = 2; int64 price_minor = 3; }
message StockUpdate       { string id = 1; int32 quantity = 2; }
```
- Four call shapes: unary, server-streaming, client-streaming, bidirectional.
- **Field numbers are the wire identity.** Never reuse or renumber one. Adding a field is backward-compatible; when you remove one, mark it `reserved`.
- **Deadlines** propagate across hops (a first-class timeout); errors use status codes (`NOT_FOUND`, `DEADLINE_EXCEEDED`, `UNAVAILABLE`…).
- Long-lived HTTP/2 connections mean a plain L4 load balancer pins all calls to one backend — use client-side or L7 (HTTP/2-aware) balancing.
- Browsers cannot call gRPC directly (use gRPC-Web or a REST gateway).
- Spring: Boot 4.1 added gRPC support — check the reference docs when you try it.

## 2. REST vs gRPC vs GraphQL
| | REST/JSON | gRPC/Protobuf | GraphQL |
|---|---|---|---|
| Contract | informal (OpenAPI, Day 42) | strict `.proto` | typed schema |
| Payload | text JSON | compact binary | JSON |
| Best for | public APIs, browsers, simplicity | **internal service-to-service**, streaming, low latency | many client shapes over one backend (a BFF) |
| Weak at | chatty / over-fetching | browser reach, human debugging | caching, N+1 in resolvers, query-cost control |

## 3. GraphQL in one paragraph
One endpoint; the **client names the fields it wants** (query), changes data (mutation) or subscribes (subscription). A **resolver** fetches each field — which reintroduces the **N+1 problem** (Day 49's file) unless you batch with a **DataLoader**. HTTP caching is harder (usually `POST`), authorization is per-field, and you need depth/cost limits against abusive queries. Spring for GraphQL is the Spring integration.

## 4. Avro vs Protobuf vs JSON (ties back to Day 69)
All three can be compact and versioned. **Protobuf** identifies fields by number and ships generated classes; **Avro** identifies by field name and resolves reader/writer schemas through a **registry** (the Kafka-friendly choice); **JSON** is self-describing but larger and untyped.

## 5. Exercise (≈ 20 min, on paper)
1. Write the `.proto` for the platform's "get product with stock" call; map each field to the REST response it replaces.
2. For four interactions (browser → Gateway; Order → Product price check; Order → Payment event; admin dashboard → many services) pick REST, gRPC, GraphQL or Kafka and justify in one line each.
3. Add a field `currency = 4` to `Product`, then delete `name` — show what you write in the `.proto` and why.

## 6. Forward vs reverse proxies — where load balancers, gateways and CDNs fit
- A **forward proxy** sits in front of *clients* and forwards their requests out (corporate proxy, VPN-style, caching); the server sees the proxy.
- A **reverse proxy** sits in front of *servers*; clients think they are talking to the server. It does TLS termination, load balancing, caching, compression, routing and rate limiting, and hides the backends. Examples: Nginx, HAProxy, Envoy, an AWS ALB, a Kubernetes Ingress controller.
- **Load balancer** = a reverse proxy focused on spreading traffic (L4 or L7). **API gateway** = an L7 reverse proxy plus API concerns (auth, routing, rate limiting) — your Spring Cloud Gateway. **CDN** = a geographically distributed, caching reverse proxy.
- Apps behind proxies must trust the forwarded headers (`X-Forwarded-For`, `X-Forwarded-Proto`, `Host`); in Spring set `server.forward-headers-strategy` (`framework` or `native`) or redirects and client IPs come out wrong.
- Exercise (5 min): draw browser → CDN → load balancer → Gateway → service and label what each hop terminates and who it hides.

## Checklist
- [ ] Can say what gRPC adds over REST, and why field numbers must never change.
- [ ] Can state when you would not use gRPC (browser clients, simplicity).
- [ ] Can explain GraphQL's N+1 and how DataLoader fixes it.
- [ ] Continue with Day 72's regular plan.
