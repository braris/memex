---
title: "Michaelliv/markit"
source: "https://github.com/Michaelliv/markit"
type: repos
ingested: 2026-07-05
tags: [github-repo, markdown-conversion, document-processing, llm-pipelines, typescript, cli, plugins]
summary: "markit is a TypeScript CLI and library published as markit-ai for converting many document, data, web, media, archive, code, and plain-text formats into Markdown. Its README documents CLI usage, supported formats, optional AI image/audio features, plugin extension points, SDK usage, configuration, agent-friendly JSON/raw output, development commands, and MIT licensing."
---

# Michaelliv/markit

Source: [Michaelliv/markit](https://github.com/Michaelliv/markit)

## Repository Snapshot

- Owner/repo: `Michaelliv/markit`
- Description: Convert anything to markdown. Mark it.
- Visibility: Public
- Primary language: TypeScript
- Package name: `markit-ai`
- Current package version observed at ingest: `0.5.3`
- License: MIT
- Default branch shown: `main`
- Repository scale at ingest: 66 commits, about 1.3k stars, about 53 forks
- Latest release shown: `v0.5.3`, dated 2026-04-11
- Top-level repository areas shown: `.github/workflows/`, `benchmark/`, `plugins/nutrient-pdf/`, `src/`, `test/`, `AGENTS.md`, `README.md`, `SKILL.md`, `package.json`, and TypeScript/Biome configuration files

## README Summary

`markit` converts many source formats to Markdown for command-line, SDK, and agent workflows. The README positions it as a broad Markdown conversion tool for PDFs, Office files, HTML, EPUB, Jupyter notebooks, feeds, images, audio, URLs, archives, source code, and plain text.

The project works both as a CLI and as a JavaScript/TypeScript library. The README install command uses npm:

```bash
npm install -g markit-ai
```

## CLI Usage

The README quick start shows direct conversion of documents, data files, URLs, images, and audio:

```bash
markit report.pdf
markit document.docx
markit slides.pptx
markit data.csv
markit config.json
markit https://example.com/article
markit photo.jpg
markit recording.mp3
```

It also supports writing to an output file and piping raw Markdown into downstream tools:

```bash
markit report.pdf -o report.md
markit report.pdf | pbcopy
markit data.xlsx -q | napkin create "Imported Data"
```

The CLI reference includes:

```bash
markit <source>
markit <source> -o output.md
markit <source> -p "instructions"
markit <source> --json
markit <source> -q
cat file.pdf | markit -
markit formats
markit init
markit config show
markit config get <key>
markit config set <key> <value>
markit plugin install <source>
markit plugin list
markit plugin remove <name>
markit onboard
```

## Supported Formats

The README documents built-in support for:

- PDF via `unpdf`
- Word `.docx` via Mammoth plus Turndown
- PowerPoint `.pptx` via XML parsing for slides, notes, and tables
- Excel `.xlsx` as one Markdown table per sheet
- HTML through Turndown with scripts and styles stripped
- EPUB with spine-ordered chapters and metadata
- Jupyter notebooks with Markdown cells, code, and outputs
- RSS and Atom feeds
- CSV and TSV as Markdown tables
- JSON, YAML, XML, and SVG as code blocks
- Images with EXIF metadata and optional AI description
- Audio with metadata and optional AI transcription
- ZIP archives through recursive conversion of contained files
- URLs fetched with `Accept: text/markdown`
- Wikipedia pages with main-content extraction
- Source code files as fenced code blocks
- Plain text and Markdown pass-through

## AI Features

Images and audio get metadata extraction without an LLM. For AI-powered image descriptions and audio transcription, users configure an OpenAI, Anthropic, or OpenAI-compatible provider.

The README examples show OpenAI as the default provider, Anthropic as an alternate provider, and OpenAI-compatible endpoints such as Ollama, Groq, or Together through `llm.apiBase`. Users can pass custom prompts with `-p`, for example to extract receipt line items, describe an architecture diagram, or extract whiteboard text verbatim.

## Plugins

The README describes a plugin system for adding formats, overriding built-in converters, and adding LLM providers.

Plugin installation supports npm packages, git sources, and local TypeScript files:

```bash
markit plugin install npm:markit-plugin-dwg
markit plugin install git:github.com/user/markit-plugin-ocr
markit plugin install ./my-plugin.ts
markit plugin list
markit plugin remove dwg
```

A plugin receives a `MarkitPluginAPI`, sets name/version metadata, and can register converters with `registerConverter`. Plugin converters run before built-ins, so a plugin can replace an existing converter such as PDF handling. Plugins can also register providers with `registerProvider`, including environment variable names, default API base URLs, default models, and provider functions such as image description.

## Agent-Oriented Features

The README explicitly includes a "For Agents" section. Every command supports `--json` for structured output and `-q` for raw Markdown output. The `markit onboard` command adds usage instructions to `CLAUDE.md`.

These features make the tool easier to call from agent shells, pipelines, and automation because callers can request parseable output or suppress non-content text.

## SDK Usage

The README shows `markit` as a library:

```typescript
import { Markit } from "markit-ai";

const markit = new Markit();
const { markdown } = await markit.convertFile("report.pdf");
const { markdown } = await markit.convertUrl("https://example.com");
const { markdown } = await markit.convert(buffer, { extension: ".docx" });
```

For AI-powered conversion, callers can pass plain `describe` and `transcribe` functions. The README includes examples using OpenAI for image description and audio transcription, Anthropic for vision, and a mix of providers for different media types.

The built-in provider helpers include `createLlmFunctions`, `loadConfig`, and `loadAllPlugins`, allowing SDK users to reuse CLI-style configuration and installed plugins.

## Configuration

The README documents `.markit/config.json`, created with:

```bash
markit init
```

Configuration commands include:

```bash
markit config show
markit config get llm.model
markit config set llm.provider anthropic
markit config set llm.apiKey sk-...
```

The config structure stores LLM provider settings such as provider, API base, API key, text/vision model, and transcription model. Environment variables override config values. The README lists `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, and `MARKIT_API_KEY` as supported provider-related environment variables.

## Package And Development Notes

The package metadata identifies the published npm package as `markit-ai`, exposes `dist/index.js` as the library entrypoint, and installs the `markit` binary from `dist/main.js`.

Development commands include:

```bash
bun install
bun run dev -- report.pdf
bun test
bun run check
```

The package scripts also include TypeScript build, Bun compile targets, Biome format/lint/check commands, and test execution through Bun.

## Dependencies

The package metadata lists runtime dependencies for terminal UX, file parsing, archive handling, XML/feed handling, media metadata, Office/document extraction, PDF work, and Markdown conversion. Notable dependencies include `commander`, `chalk`, `mammoth`, `mupdf`, `turndown`, `turndown-plugin-gfm`, `jszip`, `fast-xml-parser`, `rss-parser`, `exifr`, and `music-metadata`.

## License

The repository license is MIT.
