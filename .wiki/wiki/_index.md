# Wiki Articles Index

> Synthesized articles compiled from raw sources and conversation.

Last updated: 2026-07-05

## Contents

| File | Summary | Tags | Updated |
|------|---------|------|---------|
| [concepts/agent-facing-interface-design.md](concepts/agent-facing-interface-design.md) | Agent-facing interface design treats tools, schemas, instructions, errors, and payloads as a user interface for AI agents rather than a direct translation of human-facing APIs. | ai-agents, interface-design, tool-design, mcp | 2026-07-05 |
| [concepts/keyset-pagination.md](concepts/keyset-pagination.md) | Keyset pagination retrieves the next records after a known ordered key, avoiding OFFSET scans and making large ordered lists more stable and efficient. | keyset-pagination, cursor-pagination, database-performance, api-design | 2026-07-05 |
| [topics/api-pagination-design.md](topics/api-pagination-design.md) | API pagination design balances client ergonomics, database cost, result stability, and payload size; keyset pagination and paginated tool responses are both examples of designing for scalable consumers. | api-design, pagination, database-performance, tool-design | 2026-07-05 |
| [topics/mcp-server-design.md](topics/mcp-server-design.md) | MCP server design should expose curated, outcome-oriented, discoverable tools for agents instead of mirroring existing REST APIs one endpoint at a time. | mcp, ai-agents, server-design, tool-design | 2026-07-05 |

## Categories

- **concepts**: compiled concept articles
- **topics**: compiled topic articles
- **references**: compiled reference articles
- **theses**: thesis investigations

## Recent Changes

- 2026-07-02: Initialized compiled article index.
- 2026-07-05: Compiled four articles from two raw sources.
