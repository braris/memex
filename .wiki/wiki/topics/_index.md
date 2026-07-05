# Topics Index

> Compiled topic articles.

Last updated: 2026-07-05

## Contents

| File | Summary | Tags | Updated |
|------|---------|------|---------|
| [api-pagination-design.md](api-pagination-design.md) | API pagination design balances client ergonomics, database cost, result stability, and payload size; keyset pagination and paginated tool responses are both examples of designing for scalable consumers. | api-design, pagination, database-performance, tool-design | 2026-07-05 |
| [mcp-server-design.md](mcp-server-design.md) | MCP server design should expose curated, outcome-oriented, discoverable tools for agents instead of mirroring existing REST APIs one endpoint at a time. | mcp, ai-agents, server-design, tool-design, schemas, design-tools | 2026-07-05 |
| [ai-agent-workflow-customization.md](ai-agent-workflow-customization.md) | AI agent workflow customization combines always-on instructions, reusable prompts, role-specific agents, skills, MCP servers, hooks, project-local skill folders, and persistent memory into a layered control system. | ai-agents, github-copilot, claude-code, skills, mcp, hooks, workflow-design, spring-boot, agent-memory | 2026-07-05 |
| [backend-data-infrastructure-selection.md](backend-data-infrastructure-selection.md) | Backend data infrastructure selection should start from measured workload needs, consistency requirements, operational cost, and rollback plans rather than trend-driven migrations. | database-selection, backend-performance, redis, load-testing, postgresql, mysql, mongodb, cassandra | 2026-07-05 |
| [spring-boot-production-patterns.md](spring-boot-production-patterns.md) | Spring Boot production patterns combine clean layering, tuned database access, scalable pagination, cache-first read paths, observability, multi-tenant boundaries, built-in framework features, native/AOT deployment choices, and declarative HTTP clients. | spring-boot, java, production-architecture, database-performance, layering, opentelemetry, native-image, rest-clients, jpa, multi-tenancy | 2026-07-05 |
| [java-api-operational-patterns.md](java-api-operational-patterns.md) | Java API operational patterns make services debuggable and safe by using structured logs, correlation IDs, trace IDs, OpenTelemetry export, distributed tracing, error codes, and RFC 7807-style error responses. | java, api-design, logging, exception-handling, observability, rfc-7807, opentelemetry, distributed-tracing, trace-context | 2026-07-05 |
| [llm-application-observability.md](llm-application-observability.md) | LLM application observability traces prompts, completions, tool calls, retrieval steps, memory operations, service spans, costs, and evaluations so teams can debug production AI systems. | llm-observability, tracing, evaluation, cost-tracking, prompt-management, rag-evaluation, opentelemetry, distributed-tracing, agent-memory | 2026-07-05 |
| [go-production-reliability.md](go-production-reliability.md) | Go production reliability combines inspectable error chains with memory and goroutine monitoring so failures can be classified before they become outages. | go, golang, error-handling, memory-leaks, pprof, observability | 2026-07-05 |
| [local-ai-agent-architectures.md](local-ai-agent-architectures.md) | Local AI agent architectures run model inference, tools, memory, and retrieval on user-controlled infrastructure to reduce cloud dependency, improve privacy, and support offline or low-cost workflows. | ai-agents, local-ai, small-language-models, ollama, langchain, langgraph, privacy, offline-ai | 2026-07-05 |
| [agent-memory-systems.md](agent-memory-systems.md) | Agent memory systems capture observations from agent sessions, consolidate them into searchable memories, and retrieve selected context for future work instead of relying only on static instruction files. | agent-memory, ai-agents, mcp, retrieval, hybrid-search, hooks, knowledge-graphs, iii-engine | 2026-07-05 |
| [spring-boot-data-access-patterns.md](spring-boot-data-access-patterns.md) | Spring Boot data access patterns focus on explicit JPA behavior, bounded persistence contexts, tenant isolation, cache keys, and query shapes that avoid hidden database and memory costs. | spring-boot, jpa, hibernate, database-performance, multi-tenancy, batching, lazy-loading, saas | 2026-07-05 |

## Categories

- **topics**: api-pagination-design.md, mcp-server-design.md, ai-agent-workflow-customization.md, backend-data-infrastructure-selection.md, spring-boot-production-patterns.md, java-api-operational-patterns.md, llm-application-observability.md, go-production-reliability.md, local-ai-agent-architectures.md, agent-memory-systems.md, spring-boot-data-access-patterns.md

## Recent Changes

- 2026-07-02: Initialized topics index.
- 2026-07-05: Added MCP server design and API pagination design.
- 2026-07-05: Added AI agent workflow customization, Spring Boot production patterns, Java API operational patterns, and Go production reliability.
- 2026-07-05: Added backend data infrastructure selection and LLM application observability.
- 2026-07-05: Added local AI agent architectures, agent memory systems, and Spring Boot data access patterns; updated related operational topics.
