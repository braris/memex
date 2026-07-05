---
title: "API Pagination Design"
category: topic
sources:
  - "raw/articles/2026-07-05-ultra-fast-pagination-in-spring-boot.md"
  - "raw/articles/2026-07-05-mcp-is-not-the-problem-its-your-server.md"
created: 2026-07-05
updated: 2026-07-05
tags: [api-design, pagination, database-performance, tool-design]
aliases: [Pagination API design]
confidence: medium
volatility: warm
verified: 2026-07-05
summary: "API pagination design balances client ergonomics, database cost, result stability, and payload size; keyset pagination and paginated tool responses are both examples of designing for scalable consumers."
---

# API Pagination Design

> API pagination design defines how clients request bounded slices of a larger result set without overloading the backing system or creating unstable user-visible results.

Two raw sources touch pagination from different angles. The Spring Boot source focuses on database-backed HTTP APIs, arguing that OFFSET/LIMIT can degrade linearly with page depth and produce duplicate or skipped rows under concurrent writes. The MCP source focuses on agent tools, arguing that large results should be paginated with metadata rather than dumped into an agent's context.

## Consumer-Centered Pagination

Good pagination starts with how the consumer moves through data. If users scroll forward through feeds, timelines, orders, or messages, [[keyset-pagination|Keyset Pagination]] ([Keyset Pagination](../concepts/keyset-pagination.md)) often matches the interaction better than random page numbers. If consumers need arbitrary page jumps or exact page counts, OFFSET pagination may remain useful despite its scaling limits.

For agent tools, pagination also protects context. A tool that returns hundreds of records competes with instructions and conversation history. Bounded result sizes plus `has_more`, `next_offset`, `nextCursor`, or similar metadata give the caller a controlled way to continue.

## Contract Choices

Opaque cursors keep implementation details server-owned. A cursor can encode the last seen stable ordering key without exposing raw database IDs as the public contract. This lets the server change ordering internals while keeping the client contract stable.

Total counts should be promised only when they can be computed cheaply. In Spring Data-style APIs, returning a `Slice` rather than a `Page` communicates that the API knows whether another slice exists without claiming a full total.

## See Also

- [[keyset-pagination|Keyset Pagination]] ([Keyset Pagination](../concepts/keyset-pagination.md)) - database-oriented cursor pagination pattern.
- [[agent-facing-interface-design|Agent-Facing Interface Design]] ([Agent-Facing Interface Design](../concepts/agent-facing-interface-design.md)) - pagination as part of a broader tool interface.
- [[mcp-server-design|MCP Server Design]] ([MCP Server Design](mcp-server-design.md)) - pagination guidance for MCP tools.

## Sources

- [Ultra fast Pagination in Spring Boot](../../raw/articles/2026-07-05-ultra-fast-pagination-in-spring-boot.md) - source for keyset pagination and Spring Boot API examples.
- [MCP is Not the Problem, It's your Server](../../raw/articles/2026-07-05-mcp-is-not-the-problem-its-your-server.md) - source for paginated agent tool response guidance.
