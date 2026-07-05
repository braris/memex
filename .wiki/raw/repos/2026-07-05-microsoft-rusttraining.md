---
title: "microsoft/RustTraining"
source: "https://github.com/microsoft/RustTraining"
type: repos
ingested: 2026-07-05
tags: [github-repo, rust, training-materials, mdbook, programming-education, async-rust, engineering-practices, microsoft]
summary: "Microsoft RustTraining is a repository of Rust training books for multiple learner backgrounds and levels, including bridge courses for C/C++, C#, and Python programmers, deep dives on async Rust, advanced Rust patterns, type-driven correctness, and engineering practices. Its README documents the curriculum structure, rendered GitHub Pages site, mdBook local workflow, xtask build/deploy commands, dual MIT and CC-BY-4.0 licensing, and training-material caveats."
---

# microsoft/RustTraining

Source: [microsoft/RustTraining](https://github.com/microsoft/RustTraining)

## Repository Snapshot

- Owner/repo: `microsoft/RustTraining`
- Description: Beginner, advanced, expert level Rust training material.
- Visibility: Public
- Primary languages shown: Rust and JavaScript
- License: dual MIT and Creative Commons Attribution 4.0 International (CC-BY-4.0)
- Default branch shown: `main`
- Repository scale at ingest: 151 commits, about 14.7k stars, about 1.2k forks
- Releases shown at ingest: none published
- Top-level areas shown: `.cargo/`, `.github/workflows/`, `async-book/`, `c-cpp-book/`, `csharp-book/`, `engineering-book/`, `python-book/`, `rust-patterns-book/`, `type-driven-correctness-book/`, `xtask/`, `Cargo.toml`, `LICENSE`, `LICENSE-DOCS`, `README.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, and `SECURITY.md`

## README Summary

RustTraining is a set of seven Rust training books. The README describes the material as a cohesive curriculum for learning Rust from several programming backgrounds, then moving into async Rust, advanced patterns, type-driven correctness, and production engineering practices.

The README explicitly frames the books as training material rather than an authoritative reference. It recommends verifying critical details against the official Rust documentation and the Rust Reference.

## Curriculum Structure

The README groups the books by learning level:

- Bridge: learning Rust when coming from another language
- Deep Dive: focused exploration of a major Rust subsystem
- Advanced: patterns and techniques for experienced Rust developers
- Expert: type-level and correctness techniques
- Practices: engineering, tooling, and production readiness

## Books

The seven books listed in the README are:

- Rust for C/C++ Programmers: bridge material covering move semantics, RAII, FFI, embedded, and `no_std`
- Rust for C# Programmers: bridge material for Swift, C#, and Java-style backgrounds, focused on ownership and the type system
- Rust for Python Programmers: bridge material from dynamic languages to static typing and GIL-free concurrency
- Async Rust: a deep dive into Tokio, streams, and cancellation safety
- Rust Patterns: advanced material on `Pin`, allocators, lock-free structures, and unsafe Rust
- Type-Driven Correctness: expert material on type-state, phantom types, and capability tokens
- Rust Engineering Practices: practices material covering build scripts, cross-compilation, CI/CD, and Miri

The README says each book has 15-16 chapters with Mermaid diagrams, editable Rust playgrounds, exercises, and full-text search.

## Reading And Local Preview

The README points readers to a rendered GitHub Pages site at:

```text
https://microsoft.github.io/RustTraining/
```

For local reading and contribution workflows, it recommends installing Rust with `rustup`, then installing mdBook tooling:

```bash
cargo install mdbook mdbook-mermaid
```

The local preview workflow is:

```bash
git clone https://github.com/microsoft/RustTraining.git
cd RustTraining
cargo install mdbook mdbook-mermaid
cargo xtask serve
```

The served site is documented as running at `http://localhost:3000`.

## Maintainer Workflow

The README documents repository-wide book operations through `cargo xtask`:

```bash
cargo xtask build
cargo xtask serve
cargo xtask deploy
cargo xtask clean
```

The stated meanings are:

- `build`: build all books into `site/` for local preview
- `serve`: build and serve locally
- `deploy`: build all books into `docs/` for GitHub Pages
- `clean`: remove `site/` and `docs/`

For a single book, maintainers can enter the book directory and run mdBook directly:

```bash
cd c-cpp-book && mdbook serve --open
```

## Repository Organization

The repository is organized around separate mdBook projects for each course, plus an `xtask` helper crate for coordinated build, serve, deploy, and clean workflows.

The root `Cargo.toml` declares a Rust workspace with resolver version 2 and a single workspace member:

```toml
[workspace]
resolver = "2"
members = ["xtask"]
```

This means the Rust code at the repository root is primarily operational tooling around the books rather than a library crate.

## Inspirations And Source Positioning

The README acknowledges a range of Rust ecosystem sources, including The Rust Programming Language, Rust by Example, Rustonomicon, Jon Gjengset's advanced streams, withoutboats' async writing, Mara Bos on atomics and locks, matklad on Rust analyzer/API design/error handling, Niko Matsakis on language design and borrow checker internals, and broader community material.

The repository is therefore best treated as curated training material that synthesizes Rust learning resources, not as the canonical source for Rust semantics.

## Deployment

The README says the site auto-deploys to GitHub Pages through `.github/workflows/pages.yml` when changes are pushed to the default branch. It also documents a manual `cargo xtask deploy` command for building the `docs/` output used by GitHub Pages.

## Licensing And Trademark Notice

The README states that the project is dual-licensed under the MIT License and Creative Commons Attribution 4.0 International (CC-BY-4.0). The GitHub repository page lists both `LICENSE` and `LICENSE-DOCS`.

The README also includes a Microsoft trademark notice explaining that Microsoft trademarks and logos must follow Microsoft's Trademark and Brand Guidelines, and that third-party trademarks or logos are governed by their respective policies.
