# Wiki Articles Index

> Synthesized articles compiled from raw sources and conversation.

Last updated: 2026-07-05

## Contents

| File | Summary | Tags | Updated |
|------|---------|------|---------|
| [concepts/agent-facing-interface-design.md](concepts/agent-facing-interface-design.md) | Agent-facing interface design treats tools, schemas, instructions, errors, and payloads as a user interface for AI agents rather than a direct translation of human-facing APIs. | ai-agents, interface-design, tool-design, mcp | 2026-07-05 |
| [concepts/keyset-pagination.md](concepts/keyset-pagination.md) | Keyset pagination retrieves the next records after a known ordered key, avoiding OFFSET scans and making large ordered lists more stable and efficient. | keyset-pagination, cursor-pagination, database-performance, api-design | 2026-07-05 |
| [concepts/semantic-search.md](concepts/semantic-search.md) | Semantic search retrieves documents by embedding queries and content into vectors, then ranking nearest neighbors by meaning rather than exact keyword overlap. | semantic-search, embeddings, retrieval, nearest-neighbors, rag | 2026-07-05 |
| [topics/api-pagination-design.md](topics/api-pagination-design.md) | API pagination design balances client ergonomics, database cost, result stability, and payload size; keyset pagination and paginated tool responses are both examples of designing for scalable consumers. | api-design, pagination, database-performance, tool-design | 2026-07-05 |
| [topics/mcp-server-design.md](topics/mcp-server-design.md) | MCP server design should expose curated, outcome-oriented, discoverable tools for agents instead of mirroring existing REST APIs one endpoint at a time. | mcp, ai-agents, server-design, tool-design | 2026-07-05 |
| [topics/ai-agent-workflow-customization.md](topics/ai-agent-workflow-customization.md) | AI agent workflow customization combines always-on instructions, reusable prompts, role-specific agents, skills, MCP servers, and hooks into a layered control system. | ai-agents, github-copilot, skills, mcp, hooks, workflow-design | 2026-07-05 |
| [topics/spring-boot-production-patterns.md](topics/spring-boot-production-patterns.md) | Spring Boot production patterns combine clean layering, tuned database access, scalable pagination, cache-first read paths, and explicit transaction and migration controls. | spring-boot, java, production-architecture, database-performance, layering | 2026-07-05 |
| [topics/java-api-operational-patterns.md](topics/java-api-operational-patterns.md) | Java API operational patterns make services debuggable and safe by using structured logs, correlation IDs, error codes, and RFC 7807-style error responses. | java, api-design, logging, exception-handling, observability, rfc-7807 | 2026-07-05 |
| [topics/go-production-reliability.md](topics/go-production-reliability.md) | Go production reliability combines inspectable error chains with memory and goroutine monitoring so failures can be classified before they become outages. | go, golang, error-handling, memory-leaks, pprof, observability | 2026-07-05 |
| [references/llm-ready-tooling.md](references/llm-ready-tooling.md) | LLM-ready tooling prepares, indexes, converts, retrieves, or renders content in formats that agents and language models can consume effectively. | llm-tools, markdown-conversion, knowledge-management, retrieval, text-to-speech | 2026-07-05 |
| [references/modern-command-line-tools.md](references/modern-command-line-tools.md) | Modern command-line tools improve developer workflows by replacing brittle legacy commands, adding searchable output, and exposing higher-level Git and Linux operations. | command-line-tools, linux, git, developer-productivity, unix-tools | 2026-07-05 |

## Categories

- **concepts**: compiled concept articles
- **topics**: compiled topic articles
- **references**: compiled reference articles
- **theses**: thesis investigations

## Recent Changes

- 2026-07-02: Initialized compiled article index.
- 2026-07-05: Compiled four articles from two raw sources.
- 2026-07-05: Compiled seven articles from newly ingested raw sources.
