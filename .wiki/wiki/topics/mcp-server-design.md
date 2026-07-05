---
title: "MCP Server Design"
category: topic
sources:
  - "raw/articles/2026-07-05-mcp-is-not-the-problem-its-your-server.md"
  - "raw/articles/2026-07-05-ai-agent-tool-design-what-works-and-what-doesnt.md"
  - "raw/articles/2026-07-05-penpot.md"
created: 2026-07-05
updated: 2026-07-05
tags: [mcp, ai-agents, server-design, tool-design, schemas, design-tools]
aliases: [Model Context Protocol server design]
confidence: medium
volatility: warm
verified: 2026-07-05
summary: "MCP server design should expose curated, outcome-oriented, discoverable tools for agents instead of mirroring existing REST APIs one endpoint at a time."
---

# MCP Server Design

> MCP server design is the practice of exposing tools, resources, and prompts in a form that agents can reliably discover and use.

The source frames MCP as a user interface for agents. The protocol can standardize access to tools, resources, and prompts, but server quality depends on how well the server curates the agent's experience.

## Patterns

An MCP server should prefer outcomes over operations. Instead of exposing every underlying REST endpoint, server authors should compose low-level API calls behind a higher-level tool when that better matches the user's goal.

Tool arguments should be simple and explicit. Flat parameters, literals, defaults, and typed constraints reduce the chance that an agent invents nested keys or misses required structure.

Tool descriptions and errors are part of the interface. They should say when a tool should be used, what arguments are expected, and how to recover from common failures.

Tool sets should be curated. The source recommends a focused server with a small number of useful tools, tight response payloads, service-prefixed action names, and pagination metadata for large lists.

## Skills and MCP

The source treats Skills and MCP as complementary. MCP gives agents structured interfaces with schemas and typed responses. Skills package instructions and supporting resources that can teach an agent when and how to use tools for particular workflows.

## Tool Surface Discipline

The agent tool-design source adds a stricter operational rule: a server's tool catalog is itself part of the prompt. Loading too many overlapping tools into every context reduces selection accuracy and wastes token budget. A better MCP server exposes semantically distinct tools with bounded descriptions, tight argument schemas, structured errors, and explicit partial-success behavior.

Penpot's homepage is an example of MCP moving beyond generic developer operations into design workflows. Its product positioning includes a Penpot MCP Server and AI workflow support, suggesting that design systems, UI layouts, and code handoff can become agent-accessible domains when the interface is curated.

## See Also

- [[agent-facing-interface-design|Agent-Facing Interface Design]] ([Agent-Facing Interface Design](../concepts/agent-facing-interface-design.md)) - the underlying interface principle for MCP tools.
- [[api-pagination-design|API Pagination Design]] ([API Pagination Design](api-pagination-design.md)) - an adjacent case where API design improves reliability under scale.
- [[ai-agent-workflow-customization|AI Agent Workflow Customization]] ([AI Agent Workflow Customization](ai-agent-workflow-customization.md)) - MCP is one layer in the broader agent customization stack.
- [[collaborative-design-tools|Collaborative Design Tools]] ([Collaborative Design Tools](../references/collaborative-design-tools.md)) - design tools such as Penpot can expose agent-accessible workflow surfaces.

## Sources

- [MCP is Not the Problem, It's your Server](../../raw/articles/2026-07-05-mcp-is-not-the-problem-its-your-server.md) - source for MCP server best practices and Skills comparison.
- [AI Agent Tool Design: What Works and What Doesnt](../../raw/articles/2026-07-05-ai-agent-tool-design-what-works-and-what-doesnt.md) - source for tool catalog, schema, error, idempotency, and safety-gate patterns.
- [Penpot](../../raw/articles/2026-07-05-penpot.md) - source for Penpot MCP and AI workflow positioning.
