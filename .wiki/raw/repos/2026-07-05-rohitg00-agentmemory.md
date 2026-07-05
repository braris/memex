---
title: "rohitg00/agentmemory"
source: "https://github.com/rohitg00/agentmemory"
type: repos
ingested: 2026-07-05
tags: [github-repo, agent-memory, ai-agents, mcp, codex-plugin, claude-code, retrieval, iii-engine, typescript]
summary: "agentmemory is a TypeScript package and repository for persistent memory across AI coding agents. Its README documents a local memory server built on iii-engine, MCP and REST access, automatic hook capture, BM25/vector/graph retrieval, memory consolidation, real-time viewer, Codex/Claude/Copilot/Cursor/OpenCode and other agent integrations, native skills, benchmarks, deployment options, and Apache-2.0 licensing."
---

# rohitg00/agentmemory

Source: [rohitg00/agentmemory](https://github.com/rohitg00/agentmemory)

## Repository Snapshot

- Owner/repo: `rohitg00/agentmemory`
- Description: Persistent memory for AI coding agents based on real-world benchmarks.
- Visibility: Public
- Primary package: `@agentmemory/agentmemory`
- Package version observed at ingest: `0.9.27`
- License: Apache-2.0
- Default branch shown: `main`
- Repository scale at ingest: 463 commits, about 24.6k stars, about 2k forks
- Top-level areas shown: `.claude-plugin/`, `.codex-plugin/`, `READMEs/`, `assets/`, `benchmark/`, `deploy/`, `docs/`, `eval/`, `examples/python/`, `integrations/`, `packages/mcp/`, `plugin/`, `scripts/`, `src/`, `test/`, `website/`, `DESIGN.md`, `INSTALL_FOR_AGENTS.md`, `README.md`, `ROADMAP.md`, `package.json`, and iii config files

## README Summary

agentmemory is a persistent memory server for AI coding agents. The README describes it as a local memory system for Claude Code, GitHub Copilot CLI, Cursor, Gemini CLI, Codex CLI, Hermes, OpenClaw, pi, OpenCode, and any MCP-capable client.

The core use case is avoiding repeated project re-explanation between agent sessions. The system captures session activity, compresses it into searchable memory, and retrieves relevant context for later sessions.

## Installation And Quick Start

The README presents npm as the main install path:

```bash
npm install -g @agentmemory/agentmemory
agentmemory
agentmemory demo
agentmemory demo --serve
agentmemory connect claude-code
npx skills add rohitg00/agentmemory -y
```

It also supports no-install usage:

```bash
npx @agentmemory/agentmemory
```

The install runbook says agentmemory runs a REST API on port `3111`, a stream endpoint on `3112`, a viewer on `3113`, and an iii engine bridge on `49134`. It stores memories under `~/.agentmemory` and manages a pinned iii engine binary under `~/.agentmemory/bin`.

## Agent Integrations

The README says agentmemory works with agents that support hooks, MCP, REST APIs, or native plugin systems. The same memory server can be shared across agents.

Documented integration paths include:

- Claude Code via native plugin, hooks, skills, and MCP
- Codex CLI via plugin marketplace, MCP, lifecycle hooks, and skills
- GitHub Copilot CLI via MCP and plugin hooks/skills
- Cursor, Gemini CLI, Claude Desktop, Windsurf, Cline, Roo Code, Kilo Code, Goose, and similar MCP clients through a standard MCP server block
- OpenCode through MCP plus a plugin directory with hooks and commands
- Hermes, OpenClaw, OpenHuman, pi, Qwen Code, Antigravity, Kiro, Warp, Continue.dev, Zed, Droid, and Aider through host-specific adapters or REST/MCP patterns

For Codex CLI, the README documents:

```bash
codex plugin marketplace add rohitg00/agentmemory
codex plugin add agentmemory@agentmemory
```

The Codex plugin registers the bundled `@agentmemory/mcp` server, lifecycle hooks, and skills. The README also notes a Codex Desktop caveat where plugin-local hooks may be silent, with `agentmemory connect codex --with-hooks` as a workaround that mirrors hook commands into global Codex hooks.

## MCP Surface

The README describes a full MCP surface of 53 tools, 6 resources, 3 prompts, and native skills. The published `@agentmemory/mcp` package acts as a shim: it proxies the full tool set when it can reach a running agentmemory server through `AGENTMEMORY_URL`; otherwise, it falls back to a smaller local set.

Core tools include memory recall, file compression, saving insights, pattern detection, smart hybrid search, file history, session listing, timeline, profile, export, and relation queries.

Extended tools cover graph traversal, consolidation, Claude bridge sync, team sharing, audit/governance deletion, snapshots, action graphs, leases, routines, signals, checkpoints, mesh sync, sentinels, sketches, crystallization, diagnostics, healing, facet tags, and provenance verification.

## Memory Pipeline

The README's pipeline starts with hooks such as `PostToolUse`, deduplicates observations with a SHA-256 window, filters secrets, stores raw observations, compresses them into structured facts and narratives, embeds them, and indexes them for BM25 plus vector search.

At stop or session-end time, it summarizes sessions and can extract knowledge graphs or reflect into memory slots. At session start, it loads a project profile, runs hybrid search, respects a token budget, and injects relevant context.

## Memory Model

The README describes a four-tier consolidation model:

- Working: raw observations from tool use
- Episodic: compressed session summaries
- Semantic: extracted facts and patterns
- Procedural: workflows and decision patterns

It also describes decay, strengthening through access, stale-memory eviction, contradiction detection, relationship graphs, and provenance tracing.

## Retrieval

Search combines:

- BM25 keyword matching with stemming and synonym expansion
- Vector similarity when an embedding provider is configured
- Knowledge graph traversal when entities are detected

Results are fused with Reciprocal Rank Fusion and diversified by session. The README mentions broad tokenizer support and optional CJK segmenters.

Embedding providers include local `all-MiniLM-L6-v2`, Gemini, OpenAI, Voyage AI, Cohere, and OpenRouter. Local embeddings are presented as the recommended free/offline option.

## Benchmarks And Claims

The README reports benchmark claims for `coding-agent-life-v1` and LongMemEval-S. It lists LongMemEval-S figures of 95.2% R@5, 98.6% R@10, and 88.2% MRR for agentmemory, and contrasts BM25-only fallback results.

It also includes token-savings comparisons that position agentmemory as injecting selected memory rather than loading full context, and compares the project with mem0, Letta/MemGPT, Khoj, supermemory, MemPalace, oracleagentmemory, Hippo, and built-in agent memory.

The README notes that some competitor figures are from different datasets or vendor-reported claims, so later compilation should preserve those caveats.

## Viewer And iii Console

agentmemory ships a real-time viewer on port `3113` for observing memory creation, sessions, memory browsing, knowledge graph visualization, and health dashboards. The README also describes using the iii console alongside agentmemory to inspect OpenTelemetry traces, function calls, KV state, streams, queues, logs, and worker state.

## Programmatic Access

The README says agentmemory registers core operations as iii functions, including `mem::remember`, `mem::observe`, `mem::context`, `mem::smart-search`, and `mem::forget`. Python, Rust, and Node callers can use iii SDKs, while REST remains available for hosts without an iii runtime.

The repository includes `examples/python/` for a quickstart and observation/recall flow.

## Runtime And Package Metadata

The package metadata identifies the npm package as `@agentmemory/agentmemory`, an ESM TypeScript package with CLI binary `agentmemory` at `dist/cli.mjs`.

Runtime dependencies include `iii-sdk` pinned to `0.11.2`, Anthropic SDK packages, `dotenv`, `zod`, `@clack/prompts`, and `picocolors`. Optional dependencies include local embedding and tokenizer packages such as `@xenova/transformers`, ONNX runtime packages, `@node-rs/jieba`, and `tiny-segmenter`.

The package requires Node.js `>=20.0.0`. Scripts include build, start, test, skill generation/checking, benchmark loading, and evaluation runners for LongMemEval and coding-life corpora.

## Deployment

The README documents deploy templates for Fly.io, Railway, Render, and Coolify. Deployment templates run the npm package with a pinned iii engine, mount persistent storage at `/data`, configure external binding for REST, and keep the viewer loopback-bound with SSH-tunnel guidance.

## License

The repository is licensed under Apache-2.0.
