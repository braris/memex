
# [Ultra fast Pagination in Spring Boot | Production Ready Keyset](https://medium.com/javarevisited/ultra-fast-pagination-in-spring-boot-production-ready-keyset-91e8e54d8922)

## OFFSET is Lying to You (and Your Database Knows It)

A few years ago, I shipped what I thought was a perfectly reasonable REST API.

`GET /orders?page=412&pageSize=20`

Clean. Familiar. Textbook Spring Data pagination.

It worked beautifully in staging. It worked in early production.

Then the business did what businesses do: **they grew**.

One morning, dashboards were slow. Latency graphs looked like a staircase. Database CPU was pinned.

The API itself wasn’t *down,* it was just… tired.

The root cause?

Pagination.

More specifically: **OFFSET/LIMIT pagination quietly melting our database**.

If you’re a Java or Spring Boot developer building APIs that *might* grow beyond a few hundred thousand rows, this article is your wake‑up call.

We’re going to talk about **Keyset (cursor based) pagination,** what it is, why it’s faster, and how to implement it cleanly in Spring Boot without turning your API into an unreadable mess.

And yes, this is opinionated. Because databases don’t care about your feelings.

> [Non members can read full article by clicking here!](https://medium.com/@harrymarkjava/91e8e54d8922?sk=1913c3b097ba4ba938744ebe56df5c2f)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*y-wFE9OHO6ILN2EgsJYvFw.png)

Image Edited on Canva

## Why OFFSET Pagination Becomes a Production Problem

OFFSET pagination feels harmless because it’s easy.

```c
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 20 OFFSET 8000;
```

Page 401. What could go wrong?

### The truth

Relational databases **do not jump** to row 8001.

They

1. Scan rows from the beginning
2. Sort them
3. Throw away the first 8000
4. Finally return 20

Every. Single. Time.

OFFSET is literally telling your database

> *“Do all the work… then discard most of it.”*

That’s not clever. That’s wasteful.

### The bigger your data, the worse it gets

- Page 1 → fast
- Page 10 → fine
- Page 100 → noticeable
- Page 500 → painful
- Page 1000 → **why is prod on fire?**

Indexes don’t save you here. They help with ordering, not with skipping work.

### OFFSET pagination is also unstable

If rows are inserted or deleted while a client is paging

- Items get skipped
- Items get duplicated
- Users lose trust

This is not theoretical. I’ve debugged this in real systems. Users notice.

OFFSET is fine for admin dashboards. OFFSET is **not** fine for high traffic, user facing APIs.

## How Keyset (Cursor based) Pagination Actually Works

Keyset pagination flips the question.

Instead of asking

> *“Give me page 401”*

You ask

> *“Give me the next 20 items* ***after this one****.”*

### The mental model

Think of it like scrolling your chat app.

You don’t say:

> *“Show me message number 10,240”*

You say

> *“Show me messages* ***older than this message****.”*

That’s a cursor.

### The SQL difference

OFFSET pagination

```c
LIMIT 20 OFFSET 8000
```

Keyset pagination

```c
WHERE created_at < :lastSeenCreatedAt
ORDER BY created_at DESC
LIMIT 20
```

Notice what’s missing?

No OFFSET. No skipping. No wasted scans.

The database uses the index, jumps straight to the right spot, and keeps going.

This is why keyset pagination scales.

## When Keyset Pagination Is the Right Choice

Keyset pagination is perfect when

- Data is large (millions of rows)
- Clients scroll forward (feeds, timelines, lists)
- Performance matters
- Consistency matters

It’s **not** ideal when

- You need random page jumps (page 37)
- You need exact total page counts

That’s a tradeoff. And it’s usually worth it.

## Designing a Clean Cursor Based API

Let’s talk API design, because most examples online stop at SQL.

### Bad API design

```c
GET /orders?lastId=928374
```

This leaks implementation details and breaks the moment ordering changes.

### Better API design

```c
GET /orders?cursor=eyJjcmVhdGVkQXQiOiIyMDI0LTAxLTEyVDEwOjMwOjAwWiIsImlkIjo5MjgzNzR9
```

The cursor is

- Opaque
- Encoded (Base64 / JSON)
- Owned by the server

Clients don’t care what’s inside. That’s the point.

## Implementing Keyset Pagination in Spring Boot

Let’s build this properly.

### Entity

```c
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false)
    private Instant createdAt;
    // other fields
}
```

### Why we need both createdAt and id

Because timestamps are not guaranteed to be unique.

We use `(createdAt, id)` as a **stable ordering key**.

### Repository

```c
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query("""
        SELECT o FROM Order o
        WHERE (o.createdAt < :createdAt)
           OR (o.createdAt = :createdAt AND o.id < :id)
        ORDER BY o.createdAt DESC, o.id DESC
    """)
    List<Order> findNextPage(
        @Param("createdAt") Instant createdAt,
        @Param("id") Long id,
        Pageable pageable
    );
}
```

This query

- Uses the index
- Avoids OFFSET
- Guarantees deterministic ordering

### Cursor Model

```c
public record OrderCursor(Instant createdAt, Long id) {
}
```

### Encoding / Decoding the Cursor

```c
public class CursorUtil {

    private static final ObjectMapper mapper = new ObjectMapper();
    public static String encode(OrderCursor cursor) {
        try {
            return Base64.getUrlEncoder()
                .encodeToString(mapper.writeValueAsBytes(cursor));
        } catch (Exception e) {
            throw new IllegalStateException("Failed to encode cursor", e);
        }
    }
    public static OrderCursor decode(String cursor) {
        try {
            byte[] decoded = Base64.getUrlDecoder().decode(cursor);
            return mapper.readValue(decoded, OrderCursor.class);
        } catch (Exception e) {
            throw new IllegalArgumentException("Invalid cursor", e);
        }
    }
}
```

### Service Layer

```c
@Service
public class OrderService {

    private final OrderRepository repository;
    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }
    public Slice<Order> getOrders(String cursor, int size) {
        Pageable pageable = PageRequest.of(0, size);
        if (cursor == null) {
            return repository.findAll(
                PageRequest.of(0, size, Sort.by("createdAt").descending())
            );
        }
        OrderCursor decoded = CursorUtil.decode(cursor);
        List<Order> orders = repository.findNextPage(
            decoded.createdAt(),
            decoded.id(),
            pageable
        );
        return new SliceImpl<>(orders, pageable, orders.size() == size);
    }
}
```

### Controller

```c
@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderService service;
    public OrderController(OrderService service) {
        this.service = service;
    }
    @GetMapping
    public ResponseEntity<?> getOrders(
        @RequestParam(required = false) String cursor,
        @RequestParam(defaultValue = "20") int size
    ) {
        Slice<Order> slice = service.getOrders(cursor, size);
        String nextCursor = slice.hasNext()
            ? CursorUtil.encode(
                new OrderCursor(
                    slice.getContent()
                        .get(slice.getNumberOfElements() - 1)
                        .getCreatedAt(),
                    slice.getContent()
                        .get(slice.getNumberOfElements() - 1)
                        .getId()
                )
            )
            : null;
        return ResponseEntity.ok(Map.of(
            "data", slice.getContent(),
            "nextCursor", nextCursor
        ));
    }
}
```

## Performance Reality

In real systems I’ve worked on

- OFFSET pagination degraded linearly with page number
- Keyset pagination stayed flat

Page 10 or page 10,000, same query plan, same cost.

Databases love this.

Your on call rotation will too.

## Best Practices You’ll Thank Yourself For Later

- Always order by a **unique, indexed key**
- Never expose raw IDs as cursors
- Use `Slice`, not `Page`
- Don’t promise total counts you can’t compute cheaply
- Document cursor behavior clearly

Pagination is part of your public contract. Treat it seriously.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*sncxLvOysPR7TslVtKlNNQ.png)

Image Edited on Canva

## Finally

OFFSET pagination isn’t evil.

It’s just lazy.

Keyset pagination is what you reach for when

If you’re building APIs in Spring Boot and expect growth, **do this early**.

Retrofitting pagination after production traffic hits is painful. I’ve been there.

If you disagree, I’d genuinely love to hear why.

> Drop a comment. Clap if this saved you time.

Share it with that teammate still shipping `PageRequest.of(page, size)` everywhere.

Your database will thank you.