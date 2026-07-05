---
title: "Keyset Pagination"
category: concept
sources:
  - "raw/articles/2026-07-05-ultra-fast-pagination-in-spring-boot.md"
created: 2026-07-05
updated: 2026-07-05
tags: [keyset-pagination, cursor-pagination, database-performance, api-design]
aliases: [Cursor pagination, Cursor-based pagination]
confidence: medium
volatility: warm
verified: 2026-07-05
summary: "Keyset pagination retrieves the next records after a known ordered key, avoiding OFFSET scans and making large ordered lists more stable and efficient."
---

# Keyset Pagination

> Keyset pagination pages through data by asking for records after the last seen ordered key, rather than asking the database to skip a number of rows.

OFFSET/LIMIT pagination asks for a page number by skipping rows, such as `LIMIT 20 OFFSET 8000`. The source argues that relational databases still have to walk, sort, or count through skipped rows before returning the desired page. This cost grows with page depth and can also create duplicate or missing items when records are inserted or deleted while clients page through results.

Keyset pagination changes the query shape. Instead of page 401, the client asks for the next records after a cursor. In descending chronological order, the SQL condition may look like `WHERE created_at < :lastSeenCreatedAt ORDER BY created_at DESC LIMIT 20`. The database can use the ordered index to continue from the last seen position.

## Stable Ordering

A production keyset needs a deterministic ordering key. The source recommends using both `createdAt` and `id` because timestamps may collide. For descending order, the repository condition becomes:

```sql
WHERE (created_at < :createdAt)
   OR (created_at = :createdAt AND id < :id)
ORDER BY created_at DESC, id DESC
```

The cursor should be opaque to clients. Instead of exposing `lastId`, the server can encode a small JSON payload containing the ordered key, such as `createdAt` and `id`, and return it as `nextCursor`.

## Tradeoffs

Keyset pagination fits feeds, timelines, large lists, and high-traffic APIs where users move forward through results. It is less suitable when clients need random page jumps or exact total page counts.

## See Also

- [[api-pagination-design|API Pagination Design]] ([API Pagination Design](../topics/api-pagination-design.md)) - broader API contract choices for pagination.
- [[agent-facing-interface-design|Agent-Facing Interface Design]] ([Agent-Facing Interface Design](agent-facing-interface-design.md)) - related principle of designing interfaces around the actual consumer.

## Sources

- [Ultra fast Pagination in Spring Boot](../../raw/articles/2026-07-05-ultra-fast-pagination-in-spring-boot.md) - source for keyset pagination behavior, Spring Boot examples, and tradeoffs.
