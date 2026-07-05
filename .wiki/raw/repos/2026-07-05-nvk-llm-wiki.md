---
title: "nvk/llm-wiki"
source: "https://github.com/nvk/llm-wiki"
type: repos
ingested: 2026-07-05
tags: [github-repo, llm-wiki, knowledge-management, ai-agents, codex-plugin, claude-plugin, obsidian]
summary: "llm-wiki is a repository for LLM-compiled knowledge bases that support source ingestion, wiki compilation, querying, research, audits, session capture, feedback curation, inventory, datasets, output generation, and multi-runtime packaging for Claude Code, Codex, OpenCode, Pi, and generic LLM agents."
---

# nvk/llm-wiki

Source: [nvk/llm-wiki](https://github.com/nvk/llm-wiki)

## Repository Snapshot

- Owner/repo: `nvk/llm-wiki`
- Description: LLM-compiled knowledge bases for any AI agent, with parallel research, thesis-driven investigation, source ingestion, compilation, querying, audits, and artifact generation.
- Visibility: Public
- License: MIT
- Default branch shown: `master`
- Repository scale at ingest: 198 commits, about 795 stars, about 87 forks
- Top-level areas shown: `.agents/`, `.claude-plugin/`, `.claude/`, `claude-plugin/`, `plugins/`, `scripts/`, `tests/`, `AGENTS.md`, `CLAUDE.md`, `README.md`, and `LICENSE`

## README Summary

llm-wiki is an LLM-operated knowledge-base manager. It ingests external sources into immutable raw files, compiles them into interlinked wiki articles, supports query and research workflows, and keeps outputs and operational state in a structured wiki directory layout.

The README describes the project as Obsidian-compatible and multi-runtime. It ships as:

- a Claude Code plugin
- an OpenAI Codex marketplace plugin
- an OpenCode instruction file
- a Pi instruction file for local models
- a portable `AGENTS.md` protocol for other LLM agents

## Main Capabilities

The repository describes these major workflow areas:

- source ingestion from URLs, inbox files, Git repos, MediaWiki dumps or APIs, message archives, and Wayback snapshots
- wiki compilation from raw sources into synthesized concept, topic, and reference articles
- query workflows over compiled wiki content
- parallel research and thesis-driven investigation
- collector catalogs for examples, tools, media, memes, entities, and source candidates
- truth-seeking audits for outputs, wiki content, and source provenance
- inventory records for durable items, candidates, corpora, entities, and next actions
- dataset manifests for large or external datasets that should stay queryable in their native location
- output generation for reports, slides, summaries, timelines, glossaries, comparisons, and project artifacts
- session capture and rehydration for operational memory across agent sessions
- feedback curation for high-signal user corrections, preferences, approvals, and plan-acceptance signals
- topic archive and restore workflows
- linting and deterministic helper scripts for structural repair

## Changelog Highlights In README

The README front-loads recent changelog entries:

- `v0.12.0`: Feedback curator for redacted high-signal corrections and preferences under `HUB/.sessions/feedback/`, with explicit promotion into topic raw notes.
- `v0.11.1`: Session helper compatibility for Python 3.9 and newer runtimes.
- `v0.11.0`: Automated session capture with status, enable, disable, capture, list, show, rehydrate, and promote workflows.
- `v0.10.2`: Collector production hardening, including kind-first collection topic slugs, scale boundary handling, media download timeouts, file-size caps, content-type checks, and IPv4 retry guidance.
- `v0.10.1`: Collector media downloads into `output/assets/collect-<slug>/`.
- `v0.10.0`: Collector catalogs for provenance-rich collections.
- `v0.9.0`: Topic archive lifecycle under `topics/.archive/`.
- `v0.8.7`: iCloud permission diagnostics for sandboxed agents.
- `v0.8.6`: Lint repair improvements for frontmatter, source references, stale indexes, and raw coverage gaps.
- `v0.8.5`: Safer hub-level lint defaults.
- `v0.8.4`: Portable iCloud hub resolution through `hub_path` and portable topic paths.

## Installation And Invocation

### Claude Code

The README lists native Claude Code plugin installation:

```bash
claude plugin install wiki@llm-wiki
```

### OpenAI Codex

The README lists Codex marketplace installation from GitHub:

```bash
codex plugin marketplace add nvk/llm-wiki
```

After marketplace installation, the README says to open `/plugins`, enable "LLM Wiki", and use `@wiki`.

It also documents local checkout installation with:

```bash
./scripts/bootstrap-codex-plugin.sh --scope user --verify
```

and manual local marketplace registration:

```bash
codex plugin marketplace add /absolute/path/to/llm-wiki
```

### OpenCode

The README describes using an instruction URL in `opencode.json`, with `external_directory` permissions for the wiki hub. It notes that OpenCode fetches the instruction URL fresh on each session start, and that local mode can use `.wiki/` in the project to avoid external wiki permissions.

### Pi

The README describes Pi usage through the OpenCode skill file, including a local `llama-server` backend option with `OPENAI_BASE_URL` pointed at a local server.

### Any LLM Agent

For other agents, the README says to copy `AGENTS.md` into the target agent context or project root. That file is presented as a portable single-file protocol for agents that can read and write files and search the web.

## Canonical Commands

The README presents `@wiki` as the canonical explicit invocation in Codex, with examples such as:

```text
@wiki research "hardware wallet threat models"
@wiki collect "bitcoin memes" --wiki memes-bitcoin
@wiki ingest https://example.com/article
@wiki audit --project coldcard-threat-model
@wiki session status
@wiki feedback list --unpromoted
@wiki session disable
@wiki ll "codex plugin install gotchas"
```

It also documents slash-style examples for wiki workflows:

```text
/wiki:research "nutrition" --new-topic
/wiki:thesis "fiber reduces neuroinflammation via SCFAs"
/wiki:collect "bitcoin memes" --wiki memes-bitcoin
/wiki:query "How does fiber affect mood?"
/wiki:ingest https://example.com/article
/wiki:ingest --inbox
/wiki:ingest-collection https://github.com/bitcoin/bips --wiki bitcoin
/wiki:inventory list --view actions --limit 10
/wiki:dataset list --view schema --limit 10
/wiki:archive topic old-interest --reason "No longer active"
/wiki:compile
/wiki:audit --project gut-brain-playbook
/wiki:output report --topic gut-brain
/wiki:lint --fix
```

## Multi-Runtime Architecture

The README says Claude Code is the principal user and that the project keeps one shared behavior layer with thin runtime-specific packaging layers.

The architecture section identifies:

- `claude-plugin/` as the primary distribution and UX target
- `claude-plugin/skills/wiki-manager/` as the behavioral source of truth
- `plugins/llm-wiki/skills/wiki/` as the generated Codex packaging target
- `plugins/llm-wiki-opencode/` as the OpenCode and Pi packaging target
- `.agents/plugins/marketplace.json` as the Codex marketplace entry
- `AGENTS.md` as the portable fallback protocol

Runtime mirrors are generated rather than hand-maintained. The README documents sync scripts:

```bash
./scripts/sync-codex-plugin.sh
./scripts/sync-opencode-plugin.sh
```

The README also says drift is caught by tests:

```bash
./tests/test-codex-sync.sh
./tests/test-opencode-sync.sh
```

## Sandbox And Hub Resolution Notes

The README includes `nono` sandbox guidance for Claude Code, OpenCode, and Codex. The important path classes are:

- `$HOME/.config/llm-wiki` for hub path configuration
- the actual wiki data directory for read/write access
- `$HOME/.codex` for Codex plugin install, cache, state, and marketplace temp files

It emphasizes that hub resolution checks `~/.config/llm-wiki/config.json` first and only falls back to `~/wiki` when no config exists. For iCloud or non-default hubs, it recommends setting a portable `hub_path` that begins with `~`.

The README also describes a diagnostic pattern where `stat` can succeed for an iCloud wiki path while reads or directory listings fail with `Operation not permitted`; in that case the issue is privacy or sandbox permission, not an invalid wiki path.

## Upgrade Notes

The README recommends HTTPS GitHub CLI authentication for agents and sandboxed sessions to avoid SSH host-key prompts and `known_hosts` writes:

```bash
gh auth login --web --git-protocol https
gh auth setup-git
```

For Codex, it lists marketplace upgrade:

```bash
codex plugin marketplace upgrade llm-wiki
```

For OpenCode and `AGENTS.md`, it documents refreshing from raw GitHub URLs.

## License

The repository license file is MIT License.
