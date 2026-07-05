---
title: "Agent Memory Systems"
category: topic
sources:
  - "raw/repos/2026-07-05-rohitg00-agentmemory.md"
  - "raw/articles/2026-07-05-building-ai-agents-with-local-small-language-models.md"
created: 2026-07-05
updated: 2026-07-05
tags: [agent-memory, ai-agents, mcp, retrieval, hybrid-search, hooks, knowledge-graphs, iii-engine]
aliases: [Persistent agent memory, Coding agent memory, AI memory systems]
confidence: medium
volatility: hot
verified: 2026-07-05
summary: "Agent memory systems capture observations from agent sessions, consolidate them into searchable memories, and retrieve selected context for future work instead of relying only on static instruction files."
---

# Agent Memory Systems

> Agent memory systems are persistent context layers for AI agents: they record what happened, compress it into durable knowledge, and retrieve only the pieces relevant to a later task.

The local-agent tutorial shows the smallest memory form: a conversation buffer attached to one agent run. agentmemory describes a larger system for coding agents, with hooks, a local memory server, MCP and REST access, hybrid retrieval, a viewer, and integrations across multiple agent hosts.

## What Gets Captured

agentmemory's documented capture pipeline starts from lifecycle and tool hooks. It can record prompts, file access, tool calls, tool results, tool failures, compact events, and end-of-session summaries. Before storage, observations are deduplicated and privacy-filtered.

The goal is not to remember every token forever. The system stores raw observations, then consolidates them into facts, concepts, session summaries, relationships, and procedural patterns.

## Four Memory Tiers

The source describes four tiers:

- working memory: raw observations from current or recent tool use;
- episodic memory: compressed session summaries;
- semantic memory: extracted facts and concepts;
- procedural memory: workflows and decision patterns.

This model explains why memory systems need lifecycle management. A raw command output, a project fact, and a repeatable workflow do not have the same retention value.

## Retrieval

Persistent memory is useful only if the agent can find the right slice later. agentmemory combines BM25, vector similarity, and graph traversal, then fuses results with Reciprocal Rank Fusion and session diversification.

This is a concrete use of [[hybrid-search|Hybrid Search]] ([Hybrid Search](../concepts/hybrid-search.md)): keyword search finds exact files, identifiers, or commands; vector search finds paraphrased intent; graph traversal follows related entities and concepts.

## Agent Integration

The source treats memory as shared infrastructure rather than a feature of one agent. It documents integration through MCP, REST, host-specific hooks, and plugin packages for Claude Code, Codex CLI, Copilot CLI, Cursor, Gemini CLI, OpenCode, Hermes, OpenClaw, Warp, and other clients.

This makes memory a cross-agent coordination layer. The same project memory can be available to multiple shells or editors, while leases, signals, action graphs, and governance tools can support multi-agent workflows.

## Caveats

Benchmark and competitor claims should be treated as source-reported. The README itself notes that some comparisons use different datasets or vendor-reported figures, so they are useful for product positioning but not definitive head-to-head evidence.

Memory also changes the privacy boundary. A system that records prompts, tool use, files, and summaries needs secret filtering, local storage controls, explicit deletion, audit trails, and deployment decisions that match the user's risk profile.

## See Also

- [[local-ai-agent-architectures|Local AI Agent Architectures]] ([Local AI Agent Architectures](local-ai-agent-architectures.md)) - local model and tool layer that memory can support.
- [[hybrid-search|Hybrid Search]] ([Hybrid Search](../concepts/hybrid-search.md)) - retrieval mechanism used by memory search.
- [[ai-agent-workflow-customization|AI Agent Workflow Customization]] ([AI Agent Workflow Customization](ai-agent-workflow-customization.md)) - hooks, skills, MCP, and plugins connect memory to agent workflows.
- [[llm-application-observability|LLM Application Observability]] ([LLM Application Observability](llm-application-observability.md)) - traces and viewers make agent behavior inspectable.

## Sources

- [rohitg00/agentmemory](../../raw/repos/2026-07-05-rohitg00-agentmemory.md) - source for persistent memory server, capture hooks, retrieval, MCP tools, viewer, and agent integrations.
- [Building AI Agents with Local Small Language Models](../../raw/articles/2026-07-05-building-ai-agents-with-local-small-language-models.md) - source for basic conversation memory in a local agent.
