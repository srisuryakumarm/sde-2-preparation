# Day 49 — Missing Primer: JPA Relationships, Lazy vs Eager Fetching, and the N+1 Problem

**Do this first on Day 49 (Sunday; ≈ 75 min), then continue with Day 49's regular plan.**
**Needed by:** Day 69 (the `Order` entity), any e-commerce schema (`Order` → `OrderItem`), Day 78 (pagination) — and JPA interviews, where N+1 is the most common question.
**Why it is missing:** `@OneToMany`, `@ManyToOne`, `mappedBy`, `FetchType`, `JOIN FETCH`, `@EntityGraph` and `LazyInitializationException` appear nowhere in the repo. Day 36's `Task` is a single flat entity.
**Already in place:** entities, repositories and `@Transactional` (Days 36–41), DTOs (Day 32), SQL joins (Day 40).

---

## 1. Mapping a relationship
In the database the **foreign key lives on the "many" side**. In JPA the **owning side** is the entity with `@JoinColumn`; the other side says `mappedBy`.
```java
@Entity @Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Long id;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    public void addItem(OrderItem item) { items.add(item); item.setOrder(this); }   // keep BOTH sides in sync
}

@Entity
public class OrderItem {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false) @JoinColumn(name = "order_id")
    private Order order;

    private Long productId;      // another SERVICE's data: store the id, never a @ManyToOne to Product (Day 61)
    private int quantity;
}
```
- `cascade = ALL` + `orphanRemoval = true`: saving/deleting the `Order` saves/deletes its items; removing an item from the list deletes it. Use cascade only on a *parent → owned children* relationship — never on `@ManyToOne`.
- `@ManyToMany` hides a join table; prefer modelling the join as its own entity once it carries data.
- Never put the whole entity graph in `toString()`/JSON (infinite recursion) — another reason for DTOs.
- Inside a service, relationships stay within one **aggregate** (`Order` + its items); **across services** you hold ids.

## 2. Fetching: lazy vs eager
| Association | JPA default |
|---|---|
| `@ManyToOne`, `@OneToOne` | **EAGER** — set `fetch = LAZY` yourself |
| `@OneToMany`, `@ManyToMany` | LAZY |

Lazy associations are loaded on first access through a proxy — which needs an open session. Touching one after the transaction ends throws **`LazyInitializationException`**. Spring Boot hides this by default with *Open Session In View* (`spring.jpa.open-in-view=true`, and it logs a warning at startup). **Set `spring.jpa.open-in-view=false`** and load what you need inside the service transaction.

## 3. The N+1 problem
Load 10 orders, then read `order.getItems()` for each: **1 query for the orders + 10 queries for items = N+1**. It is invisible in small tests and ruinous at scale.
**See it:** `spring.jpa.show-sql=true`, or `spring.jpa.properties.hibernate.generate_statistics=true` with `logging.level.org.hibernate.stat=DEBUG` (prints the statement count).
**Fix it (pick per use case):**
```java
@EntityGraph(attributePaths = "items")                       // fetch items in the same query
List<Order> findByStatus(OrderStatus status);

@Query("select o from Order o join fetch o.items where o.status = :status")
List<Order> findWithItems(@Param("status") OrderStatus status);
```
- **Batch fetching:** `spring.jpa.properties.hibernate.default_batch_fetch_size=50` loads lazy collections in groups (N+1 → 1 + N/50).
- **DTO projection:** select only the columns you need (interface or constructor projection) — often the best answer for read endpoints.
- Traps: `join fetch` on a collection **with pagination** makes Hibernate page *in memory* (it warns); fetching **two `List` collections** at once throws `MultipleBagFetchException` — fetch one, batch the other.

## 4. Exercise (45 min, a scratch Spring project or `todo-api`)
1. Model `Order` (id, status, createdAt) and `OrderItem` as above; add Flyway migrations (`orders`, `order_items` with a foreign key and an index on `order_id`).
2. Insert 3 orders × 3 items. In a `@Transactional` service method, load `findAll()` and print each order's item count. Turn on statistics and **count the statements** (expect 4).
3. Fix it with `@EntityGraph`; confirm the count is 1. Then try batch fetching instead.
4. With `open-in-view=false`, call `order.getItems().size()` from the controller (outside the transaction) and read the `LazyInitializationException`. Fix it by mapping to a DTO inside the service.

## Checklist
- [ ] Can say which side owns the relationship and what `mappedBy` means.
- [ ] Can state the JPA default fetch type for `@ManyToOne` vs `@OneToMany`, and why you override the first.
- [ ] Can explain N+1, show how to detect it, and name three fixes.
- [ ] Can explain Open Session In View and why it is turned off.
- [ ] Can say why `Order` holds a `productId` instead of a `Product`. Continue with Day 49's regular plan.
