---
title: "microsoft/markitdown"
source: "https://github.com/microsoft/markitdown"
type: repos
ingested: 2026-07-05
tags: [github-repo, markdown-conversion, document-processing, llm-pipelines, python, microsoft]
summary: "MarkItDown is a Microsoft Python utility for converting files and office documents into Markdown for LLM and text-analysis workflows. Its README documents supported input formats, CLI and Python APIs, optional dependency groups, plugin support, Azure integrations, security considerations, contribution workflow, and MIT licensing."
---

# microsoft/markitdown

Source: [microsoft/markitdown](https://github.com/microsoft/markitdown)

## Repository Snapshot

- Owner/repo: `microsoft/markitdown`
- Description: Python tool for converting files and office documents to Markdown.
- Visibility: Public
- Primary language: Python
- License: MIT
- Default branch shown: `main`
- Repository scale at ingest: 309 commits, about 163k stars, about 11.5k forks
- Top-level repository areas shown: `.devcontainer/`, `.github/`, `packages/`, Dockerfile, README, license, security, support, and contribution files

## README Summary

MarkItDown is a lightweight Python utility that converts documents and other file types into Markdown for LLMs and text-analysis pipelines. The README compares it to `textract`, but emphasizes preserving document structure and content as Markdown, including headings, lists, tables, and links.

The output is intended primarily for machine consumption by text-analysis tools rather than high-fidelity document publishing.

## Supported Input Types

The README says MarkItDown supports conversion from:

- PDF
- PowerPoint
- Word
- Excel
- Images, including EXIF metadata and OCR-related processing
- Audio, including EXIF metadata and speech transcription
- HTML
- Text-based formats such as CSV, JSON, and XML
- ZIP files by iterating over contents
- YouTube URLs
- EPubs
- Additional formats beyond the listed examples

## Requirements And Installation

MarkItDown requires Python 3.10 or higher. The README recommends using a virtual environment to avoid dependency conflicts.

Standard installation uses pip:

```bash
pip install 'markitdown[all]'
```

Source installation clones the repository and installs the package from `packages/markitdown`:

```bash
git clone git@github.com:microsoft/markitdown.git
cd markitdown
pip install -e 'packages/markitdown[all]'
```

## Basic Usage

The primary CLI converts an input file to Markdown:

```bash
markitdown path-to-file.pdf > document.md
```

It can also write to an output path:

```bash
markitdown path-to-file.pdf -o document.md
```

The README also documents piping content into the CLI:

```bash
cat path-to-file.pdf | markitdown
```

Basic Python usage instantiates `MarkItDown` and converts a file:

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=False)
result = md.convert("test.xlsx")
print(result.text_content)
```

## Optional Dependencies

The README documents optional dependency groups for enabling specific converters:

- `[all]`
- `[pptx]`
- `[docx]`
- `[xlsx]`
- `[xls]`
- `[pdf]`
- `[outlook]`
- `[az-doc-intel]`
- `[az-content-understanding]`
- `[audio-transcription]`
- `[youtube-transcription]`

For a smaller install, the README gives this example:

```bash
pip install 'markitdown[pdf, docx, pptx]'
```

## Plugins

MarkItDown supports third-party plugins, but plugins are disabled by default. The README documents listing installed plugins with:

```bash
markitdown --list-plugins
```

Plugins can be enabled during conversion:

```bash
markitdown --use-plugins path-to-file.pdf
```

The README points developers to `packages/markitdown-sample-plugin` for plugin development and suggests searching GitHub for the `#markitdown-plugin` hashtag to find available plugins.

## markitdown-ocr Plugin

The README describes `markitdown-ocr` as a plugin that adds OCR support to PDF, DOCX, PPTX, and XLSX converters by using LLM vision through the same `llm_client` and `llm_model` pattern used for image descriptions.

Installation:

```bash
pip install markitdown-ocr
pip install openai
```

Example usage:

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)
result = md.convert("document_with_images.pdf")
print(result.text_content)
```

The README notes that if no `llm_client` is provided, the plugin still loads but OCR is skipped and the standard converter is used.

## Azure Integrations

The README documents two Azure-backed conversion paths.

Azure Content Understanding is positioned as the higher-capability option for multimodal conversion, structured field extraction, audio/video handling, configurable analyzers, and complex document extraction. It is installed with:

```bash
pip install 'markitdown[az-content-understanding]'
```

The CLI can use a Content Understanding endpoint:

```bash
markitdown path-to-file.pdf --use-cu --cu-endpoint "<content_understanding_endpoint>"
```

The Python API can auto-select analyzers by file type:

```python
from markitdown import MarkItDown

md = MarkItDown(cu_endpoint="<content_understanding_endpoint>")
result = md.convert("report.pdf")
print(result.markdown)
```

Azure Document Intelligence is documented as another conversion option:

```bash
markitdown path-to-file.pdf -o document.md -d -e "<document_intelligence_endpoint>"
```

Python usage passes `docintel_endpoint` to `MarkItDown`:

```python
from markitdown import MarkItDown

md = MarkItDown(docintel_endpoint="<document_intelligence_endpoint>")
result = md.convert("test.pdf")
print(result.text_content)
```

## LLM Image Descriptions

For image descriptions, currently documented for PowerPoint and image files, the README says callers can provide an LLM client, model, and optional prompt:

```python
from markitdown import MarkItDown
from openai import OpenAI

client = OpenAI()
md = MarkItDown(llm_client=client, llm_model="gpt-4o", llm_prompt="optional custom prompt")
result = md.convert("example.jpg")
print(result.text_content)
```

## Docker

The README documents building and running a containerized converter:

```bash
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < ~/your-file.pdf > output.md
```

## Security Considerations

The README warns that MarkItDown performs I/O with the privileges of the current process, similar to `open()` or `requests.get()`. For untrusted inputs, callers should validate and restrict file paths, URI schemes, network destinations, and access to private, loopback, link-local, or metadata-service addresses.

The README recommends calling the narrowest conversion method needed:

- `convert_local()` when only local files should be read
- `convert_response()` when the caller wants to fetch a remote response manually
- `convert_stream()` when the caller wants maximum control over the input stream

This matters for hosted or server-side deployments where user-controlled input could otherwise cause the converter to access resources available to the process.

## Contribution Notes

The repository uses Microsoft's Contributor License Agreement process and Microsoft Open Source Code of Conduct. The README points contributors to issues and pull requests, including labels for open contribution and review opportunities.

Test instructions navigate to `packages/markitdown`, install `hatch`, run `hatch shell`, and then run `hatch test`. The README also recommends running pre-commit checks:

```bash
pre-commit run --all-files
```

## License

The repository license file is MIT License with copyright attributed to Microsoft Corporation.
