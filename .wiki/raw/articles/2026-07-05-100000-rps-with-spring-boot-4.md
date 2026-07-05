---
title: "100,000 RPS with Spring Boot 4: The New Architecture Blueprint (2025 Edition)"
source: "https://blog.stackademic.com/100-000-rps-with-spring-boot-4-the-new-architecture-blueprint-2025-edition-2b4ee90470d9"
type: articles
ingested: 2026-07-05
tags: [spring-boot, java, virtual-threads, high-throughput, redis, kafka, architecture]
summary: "Stackademic article proposing a high-throughput Spring Boot architecture using virtual threads, Redis-backed hot paths, Kafka pipelines, local caching, and database writes kept off the request path."
---

# [ðŸš€ 100,000 RPS with Spring Boot 4: The New Architecture Blueprint (2025 Edition)](https://blog.stackademic.com/100-000-rps-with-spring-boot-4-the-new-architecture-blueprint-2025-edition-2b4ee90470d9)

![](https://miro.medium.com/v2/resize:fit:1386/format:webp/1*swcegXhPo88HQKIPuME4zw.png)

ðŸš€ 100,000 RPS with Spring Boot 4: The New Architecture Blueprint (2025 Edition)

## How We Achieved Ultra-High Throughput Using Virtual Threads, Netty, Redis, and Zero Async Code

==In 2025, hitting== ==**100,000 RPS**== ==with Java was a dream.==  
Today, with **Spring Boot 4**, **Virtual Threads**, and **modern architecture**, itâ€™s not just possible â€” itâ€™s becoming the *new standard*.

And the craziest part?

## âŒ No WebFlux

## âŒ No Reactor

## âŒ No @Async

## âŒ No CompletableFuture

## âŒ No Custom Thread Pools

## âŒ No Reactive Backpressure Hacks

We did it with:

## âœ” Virtual Threads

## âœ” Non-blocking networking

## âœ” Lightweight Redis caching

## âœ” Connection pooling done right

## âœ” Smart batching

## âœ” Zero-database hot path

Letâ€™s break down the full blueprint.

## ðŸ”¥ The 2025 Secret: Virtual Threads + Blocking I/O = Insane Throughput

Before 2025, you needed:

- WebFlux
- massive thread pools
- async everything
- callback hell
- reactive pipelines

Today:

```c
spring.threads.virtual.enabled=true
```

And Spring Boot gives you:

## âœ” 1,000,000+ cheap threads

## âœ” blocking I/O that scales

## âœ” synchronous code with async performance

## âœ” minimal context switching

## âœ” ultra-low overhead per request (â‰ˆ3Âµs)

Virtual threads transformed **Java from â€œheavyâ€ to â€œhyper-scaled.â€**

## ðŸ§© The Architecture That Hit 100,000 RPS

Hereâ€™s the exact production-grade architecture:

```c
Client â†’ CloudFront â†’ NGINX/ALB â†’
Spring Boot 4 (Virtual Threads) â†’
Redis Cluster (sub-millisecond gets) â†’
Kafka (event pipeline) â†’
PostgreSQL (async writers, no direct reads)
```

## ðŸ’¡ Key idea:

**The API layer should never hit the database.**  
It should read from:

- Redis
- In-memory LRU cache
- Pre-computed aggregates
- Kafka materialized views

DBs do writes.  
Caches do reads.  
Kafka does pipelines.  
Spring does compute.

## âš™ï¸ Spring Boot Config That Makes It Fly

## 1ï¸âƒ£ Enable virtual threads

```c
spring.threads.virtual.enabled=true
```

## 2ï¸âƒ£ Tune Tomcat (or Netty)

```c
server.tomcat.threads.max=100000
server.max-http-header-size=16KB
```

## 3ï¸âƒ£ Redis highly optimized

```c
spring.data.redis.client-type=lettuce
spring.data.redis.lettuce.pool.max-active=1024
spring.data.redis.lettuce.pool.max-wait=500ms
```

## 4ï¸âƒ£ Drop HikariCP (when possible)

DB reads are removed from the hot path.  
Writes handled asynchronously by consumers.

## ðŸ§ª The Code Path That Handles 1000 Requests in 7 ms

## ðŸš€ High-throughput controller with virtual threads

```c
@RestController
public class StatsController {
@Autowired RedisTemplate<String, String> redis;
    @GetMapping("/stats/{id}")
    public StatsResponse getStats(@PathVariable String id) {
        // 0.3ms Redis call
        var raw = redis.opsForValue().get("stats:" + id);
        if (raw == null) {
            return StatsResponse.empty();
        }
        return parse(raw); // Fast, in-memory
    }
}
```

This handler:

- takes **~0.7 ms** total
- runs on a virtual thread
- does zero JDBC work
- touches no blocking pools
- uses only Redis + CPU

Thatâ€™s how you scale.

## ðŸ§¨ How We Eliminated Database Load Completely

Instead of hitting PostgreSQL/MySQL for aggregates:

## 1\. Producers write raw events â†’ Kafka

## 2\. Kafka Streams generates aggregates

## 3\. Aggregates are pushed into Redis

## 4\. Spring Boot reads only from Redis

DB load reduced by:

## ðŸ”¥ 99.3%

Databases are now:

- write-optimized
- low contention
- simple schema
- used only for durability

## ðŸŽ›ï¸ Virtual Threads Load Test Results (JMeter)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*54rxkFWBuGF9qfb-vIB-QA.png)

No GC tuning.  
No async.  
No thread starvation.

Just **virtual threads + cheap blocking**.

## ðŸ§© Key Optimization Strategies

## âœ” 1. Zero DB Reads

All reads served from Redis or in-memory.

## âœ” 2. Redis Multi-Region Federation

Cross-region reads are < 1.5 ms.

## âœ” 3. Aggressive Local Caching

10k most-used keys cached in-memory with Caffeine.

## âœ” 4. Virtual Threads Everywhere

Controllers, services, RestClient, JDBC.

## âœ” 5. Kafka for Async Work

DB writes and heavy computations offloaded.

## âœ” 6. No WebFlux

Blocking code is now the fastest.

## ðŸ”¥ The 100k RPS Golden Path (The Only Code That Runs Per Request)

```c
public StatsResponse handle(String id) {
    return cache.get(id); // in-memory OR redis (0.3ms)
}
```

Thatâ€™s it.  
Everything else is offline / async.

## â™»ï¸ GC & Memory Optimization Tips

- Use **ZGC**
- Set heap to **1â€“2 GB max**
- Set `-XX:+UseCompressedOops`
- Avoid large object allocations
- Use record classes for DTOs
- Avoid Jackson reflection (use afterburner or JSON-B)

## ðŸ§  Why WebFlux Is No Longer Needed

Reactive is still powerfulâ€¦

â€¦but for 99% of workloads:

## Virtual Threads beat WebFlux in:

- throughput
- latency
- development speed
- debugging
- memory usage

And you get:

- imperative code
- blocking I/O that scales
- easier monitoring
- better error handling

## ðŸ Final Architecture Overview (Blueprint)

```c
KAFKA STREAMS
                        â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                        â”‚  Pre-Aggregation     â”‚
                        â”‚  Materialized Views  â”‚
                        â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                                  â”‚
                                  â–¼
CLIENT â†’ CDN â†’ LB â†’ SPRING BOOT 4 â†’ REDIS CLUSTER â†’ DB WRITER SERVICE
         (Virtual Threads)        (Global Cache)     (Async Writes)
```

This is how you hit **100,000+ RPS** with Spring Boot.

## ðŸŽ¯ Final Takeaways

## âœ” Virtual Threads are a revolution

## âœ” Databases must be kept out of hot paths

## âœ” Redis is now the single source of read truth

## âœ” Kafka moves heavy work off the API layer

## âœ” Blocking > Async in 2025

## âœ” Spring Boot 4 is built for 100k RPS+

You are not â€œoptimizingâ€ anymore â€”  
you are **engineering for hyperscale**.