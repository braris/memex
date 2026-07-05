---
title: "markusmobius/go-trafilatura"
source: "https://github.com/markusmobius/go-trafilatura"
type: repos
ingested: 2026-07-05
tags: [github-repo, go, web-scraping, content-extraction, readability, trafilatura, cli, nlp-corpora]
summary: "go-trafilatura is a Go package and command-line tool that ports the Python Trafilatura web-content extraction library. Its README documents metadata, main-text and comment extraction, CLI batch/feed/sitemap workflows, fallback extractors, benchmark comparisons with Readability, DOM Distiller, and Python Trafilatura, Go module dependencies, and Apache-2.0 licensing."
---

# markusmobius/go-trafilatura

Source: [markusmobius/go-trafilatura](https://github.com/markusmobius/go-trafilatura)

## Repository Snapshot

- Owner/repo: `markusmobius/go-trafilatura`
- Description: Go port of the Trafilatura Python library.
- Visibility: Public
- Primary implementation language shown by GitHub: mostly HTML templates/test fixtures, with Go source files at the repository root and under `cmd/`
- License: Apache-2.0
- Default branch shown: `main`
- Repository scale at ingest: 381 commits, about 142 stars, about 17 forks
- Latest release shown: `2.0.0`, dated 2025-05-21
- Top-level areas shown: `cmd/go-trafilatura/`, `examples/`, `internal/`, `scripts/comparison/`, `test-files/`, `CHANGELOG.md`, `Makefile`, `README.md`, `go.mod`, core extractor files, metadata files, URL handling files, and tests

## README Summary

Go-Trafilatura is both a Go package and a command-line tool for downloading, parsing, and scraping web page data. It extracts metadata, main body text, and comments while preserving parts of text formatting and page structure.

The project is based on the Python `trafilatura` package by Adrien Barbaresi. The README says the Go port mirrors the Python code structure to simplify future improvements and to make pages parseable by original Trafilatura produce comparable results.

## Status And Differences From Python Trafilatura

The README states that the package is stable enough for use and up to date with original Trafilatura `v2.0.0` at commit `c6e8340`.

Documented differences from Python Trafilatura include:

- JSON-LD metadata extraction uses a JSON parser rather than regular expressions.
- Invalid JSON metadata may be skipped, but valid JSON-LD metadata extraction can be more accurate.
- Fallback extraction uses `go-readability` and `go-domdistiller` instead of Python `readability` and `jusText`.
- Custom fallback values can be specified rather than being limited to default extractors.
- The main output is HTML rather than XML, which changes treatment of formatting tags and paragraphs.

## Go Package Usage

The README installation command for library usage is:

```bash
go get -u -v github.com/markusmobius/go-trafilatura
```

Import path:

```go
import "github.com/markusmobius/go-trafilatura"
```

The README points readers to the repository examples for basic extraction usage.

## CLI Usage

The CLI is installed from the command package:

```bash
go get -u -v github.com/markusmobius/go-trafilatura/cmd/go-trafilatura
```

The CLI extracts readable content from either an HTML file or a URL. It also supports batch URL downloads from a file, RSS/Atom feeds, and sitemaps.

Top-level CLI commands listed in the README:

- `batch`: download and extract pages from URLs listed in a file
- `feed`: download and extract pages from a feed
- `sitemap`: download and extract pages from a sitemap
- `help`: command help

Selected CLI flags:

- `--deduplicate`: filter duplicate segments and sections
- `-f, --format`: output format, with `html` as default and `txt` or `json` as alternatives
- `--has-metadata`: only output documents with title, URL, and date
- `--images`: include images in the extraction result, marked experimental
- `-l, --language`: target language using ISO 639-1 codes
- `--links`: keep links in output, marked experimental
- `--no-comments`: exclude comments
- `--no-fallback`: disable fallback extraction using Readability and DOM Distiller
- `--no-tables`: include tables in the extraction result
- `--skip-tls`: skip TLS certificate verification
- `-t, --timeout`: download timeout, defaulting to 30 seconds
- `-u, --user-agent`: custom user agent
- `-v, --verbose`: enable logging

## Common CLI Workflows

The README shows direct URL extraction:

```bash
go-trafilatura http://www.domain.com/some/path
```

It also shows batch extraction from a text file of URLs:

```bash
go-trafilatura batch -o extract input.txt
```

For sitemaps, callers can provide a sitemap URL or a domain and let the tool discover the sitemap:

```bash
go-trafilatura sitemap -o extract http://www.domain.com/sitemap.xml
go-trafilatura sitemap -o extract http://www.domain.com
```

For feeds, callers can provide a feed URL or a domain and let the tool discover the feed:

```bash
go-trafilatura feed -o extract http://www.domain.com/feed-rss.php
go-trafilatura feed -o extract http://www.domain.com
```

## Performance Notes

The README emphasizes that this project and its dependencies rely heavily on regular expressions, while Go's standard regular expression engine is comparatively slow. To offset this, the project compiles several important regexes into Go code using `re2go`, improving speed without cgo or external regex packages.

The README compares Go-Trafilatura with Go-DomDistiller and Go-Readability. It characterizes DOM Distiller as very fast with good image and multipage extraction but a large archived codebase, Readability as fast and better for wiki/documentation pages with a smaller codebase, and Trafilatura as having the best accuracy and better metadata extraction, including language and publish date, at the cost of slower extraction.

## Benchmark Comparisons

The README reports a comparison over 960 documents dated 2025-05-01. In that table, `go-trafilatura` has higher accuracy and F-score than `go-readability` and `go-domdistiller`, while fallback mode improves recall and F-score but increases runtime.

The README also compares this Go port with original Python Trafilatura `v1.12.2`. It says accuracy is nearly identical because the code was ported nearly line by line, while the Go port is significantly faster because of precompiled regular expressions and can gain further speed through thread-safe concurrent use.

## Go Module And Dependencies

The `go.mod` file declares:

```go
module github.com/markusmobius/go-trafilatura

go 1.24.1
toolchain go1.24.2
```

Notable direct dependencies include:

- `github.com/andybalholm/cascadia`
- `github.com/beevik/etree`
- `github.com/go-shiori/dom`
- `github.com/go-shiori/go-readability`
- `github.com/markusmobius/go-domdistiller`
- `github.com/markusmobius/go-htmldate`
- `github.com/RadhiFadlillah/whatlanggo`
- `github.com/rs/zerolog`
- `github.com/spf13/cobra`
- `golang.org/x/net`
- `golang.org/x/sync`
- `golang.org/x/text`

The dependency set reflects the repository's scope: HTML parsing, CSS selection, metadata/date/language detection, fallback content extraction, CLI commands, logging, and test support.

## Research And Corpus Context

The README credits Adrien Barbaresi and the original Trafilatura project, linking it to academic work on web scraping, text discovery, metadata-enhanced web corpora, and corpus quality for natural language processing.

This makes the project relevant as both a production extraction library and an implementation reference for web-corpus construction pipelines.

## License

Like original Trafilatura, this repository is distributed under the Apache-2.0 license.
