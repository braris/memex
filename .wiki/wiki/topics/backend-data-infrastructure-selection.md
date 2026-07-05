---
title: "Backend Data Infrastructure Selection"
category: topic
sources:
  - "raw/articles/2026-07-05-why-i-tell-clients-not-to-use-postgresql.md"
  - "raw/articles/2026-07-05-stop-using-redis-just-for-caching-7-secret-features.md"
  - "raw/articles/2026-07-05-wrk-a-modern-and-scalable-http-benchmarking-tool.md"
created: 2026-07-05
updated: 2026-07-05
tags: [database-selection, backend-performance, redis, load-testing, postgresql, mysql, mongodb, cassandra]
aliases: [Database selection, Backend infrastructure choices]
confidence: medium
volatility: warm
verified: 2026-07-05
summary: "Backend data infrastructure selection should start from measured workload needs, consistency requirements, operational cost, and rollback plans rather than trend-driven migrations."
---

# Backend Data Infrastructure Selection

> Backend data infrastructure selection is the practice of choosing databases, caches, queues, streams, and benchmarks from the workload backward instead of from technology fashion forward.

The database-selection source argues against migrating because a database is fashionable or broadly considered "better." Its core lesson is workload fit: update-heavy OLTP, complex analytics, transaction integrity, flexible schemas, multi-datacenter availability, serverless cost profiles, and team expertise point to different choices.

## Selection Principles

Use a general-purpose relational database unless there is a measured reason not to. PostgreSQL and MySQL both cover a wide range of workloads, but they fail in different ways. The source claims PostgreSQL can be a poor fit for high-update workloads without careful vacuum and connection management, while MySQL can be better for simple high-update OLTP and mature replication. Treat those claims as source-reported guidance, not universal law.

Specialized systems should solve specific problems. MongoDB fits flexible document structures better than transactional ledgers. Cassandra fits simple, always-on, multi-region write patterns better than ad hoc relational queries. Redis can be a cache, but also a low-latency building block for Pub/Sub, list-backed queues, sorted-set leaderboards, HyperLogLog cardinality, Streams, and geospatial lookups.

## Measure Before Moving

The migration framework is straightforward: copy production-shaped data, run production-shaped queries, measure latency and write throughput, estimate operational and cloud cost, and define rollback thresholds before committing. Tools such as `wrk` support the application side of that discipline by generating high-concurrency HTTP load and reporting latency distributions.

## See Also

- [[spring-boot-production-patterns|Spring Boot Production Patterns]] ([Spring Boot Production Patterns](spring-boot-production-patterns.md)) - application architecture needs database, cache, and migration choices to match production load.
- [[keyset-pagination|Keyset Pagination]] ([Keyset Pagination](../concepts/keyset-pagination.md)) - an example of optimizing access patterns before changing infrastructure.
- [[java-api-operational-patterns|Java API Operational Patterns]] ([Java API Operational Patterns](java-api-operational-patterns.md)) - operational logs and traces provide evidence for infrastructure decisions.

## Sources

- [Why I Tell Clients NOT to Use PostgreSQL](../../raw/articles/2026-07-05-why-i-tell-clients-not-to-use-postgresql.md) - source for migration risk, database fit, and measurement-first selection.
- [Stop Using Redis Just for Caching](../../raw/articles/2026-07-05-stop-using-redis-just-for-caching-7-secret-features.md) - source for Redis data structures and backend use cases.
- [wrk: A Modern and Scalable HTTP Benchmarking Tool](../../raw/articles/2026-07-05-wrk-a-modern-and-scalable-http-benchmarking-tool.md) - source for HTTP load-testing capability.

