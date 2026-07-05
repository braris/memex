---
title: "Semantic Search"
category: concept
sources:
  - "raw/articles/2026-07-05-build-semantic-search-with-llm-embeddings.md"
  - "raw/articles/2026-07-05-how-to-build-agentic-rag-with-hybrid-search.md"
created: 2026-07-05
updated: 2026-07-05
tags: [semantic-search, embeddings, retrieval, nearest-neighbors, rag, hybrid-search]
aliases: [Embedding search, Vector search]
confidence: medium
volatility: warm
verified: 2026-07-05
summary: "Semantic search retrieves documents by embedding queries and content into vectors, then ranking nearest neighbors by meaning; hybrid search complements it with exact keyword retrieval."
---

# Semantic Search

> Semantic search is a retrieval pattern that represents queries and documents as embedding vectors so results can match meaning, synonyms, and related phrasing instead of only exact words.

Keyword search works well when users know the exact terms present in the target documents. It fails when relevant documents use synonyms, paraphrases, or related concepts. The raw semantic-search walkthrough illustrates this with a query such as "young dog" missing documents about "puppies" under literal keyword matching, while embeddings place those phrases near each other in vector space.

## Retrieval Pipeline

A minimal semantic-search system has four steps:

1. Collect documents and choose the text fields to search.
2. Encode each document into an embedding with a model such as a sentence transformer.
3. Fit or build a nearest-neighbor index over those embeddings.
4. Encode the user's query and retrieve the closest vectors by a metric such as cosine distance.

The source demonstrates this with the AG News dataset, `SentenceTransformer("all-MiniLM-L6-v2")`, and scikit-learn `NearestNeighbors(metric="cosine")`. Similarity is computed as `1 - cosine_distance`, and the top results are returned with snippets.

## Relationship To RAG

Semantic search is often the retrieval layer for retrieval augmented generation. A RAG system first finds semantically relevant chunks, then passes those chunks into an LLM as grounding context. The raw source is intentionally simple, but it shows the core mechanism: convert text to vectors, find nearest vectors, and return relevant context.

## Practical Limits

Embedding search does not remove the need for good data boundaries. Chunk size, metadata filters, freshness, ranking, deduplication, and evaluation still matter. A toy in-memory nearest-neighbor index works for learning; production systems usually need persistence, metadata-aware filtering, observability, and clear latency budgets.

Semantic search is also not the right tool for every query. The hybrid-search source points out that exact identifiers, rare keywords, product codes, and function names can be underweighted by vector similarity. [[hybrid-search|Hybrid Search]] ([Hybrid Search](hybrid-search.md)) combines vector retrieval with lexical ranking such as BM25 so RAG systems can retrieve both semantic matches and exact terms.

## See Also

- [[llm-ready-tooling|LLM-Ready Tooling]] ([LLM-Ready Tooling](../references/llm-ready-tooling.md)) - tools such as MarkItDown and llm-wiki prepare content for retrieval and synthesis.
- [[ai-agent-workflow-customization|AI Agent Workflow Customization]] ([AI Agent Workflow Customization](../topics/ai-agent-workflow-customization.md)) - semantic retrieval often feeds agent instructions, skills, and knowledge workflows.
- [[hybrid-search|Hybrid Search]] ([Hybrid Search](hybrid-search.md)) - combines semantic and keyword retrieval for RAG systems.

## Sources

- [Build Semantic Search with LLM Embeddings](../../raw/articles/2026-07-05-build-semantic-search-with-llm-embeddings.md) - source for the embedding and nearest-neighbor search pipeline.
- [How to Build Agentic RAG with Hybrid Search](../../raw/articles/2026-07-05-how-to-build-agentic-rag-with-hybrid-search.md) - source for hybrid retrieval and agentic RAG extensions.
