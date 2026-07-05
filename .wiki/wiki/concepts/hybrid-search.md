---
title: "Hybrid Search"
category: concept
sources:
  - "raw/articles/2026-07-05-how-to-build-agentic-rag-with-hybrid-search.md"
  - "raw/articles/2026-07-05-build-semantic-search-with-llm-embeddings.md"
  - "raw/repos/2026-07-05-rohitg00-agentmemory.md"
created: 2026-07-05
updated: 2026-07-05
tags: [hybrid-search, rag, bm25, vector-search, information-retrieval, llm-agents, agent-memory]
aliases: [Hybrid retrieval, Agentic hybrid search]
confidence: medium
volatility: warm
verified: 2026-07-05
summary: "Hybrid search combines vector similarity with keyword retrieval so RAG systems can retrieve both semantically related content and exact identifiers or rare terms."
---

# Hybrid Search

> Hybrid search is a retrieval pattern that combines semantic vector search with keyword search so a system can match meaning, exact terms, identifiers, and rare phrases in the same query path.

Pure vector retrieval is useful when users express an idea with synonyms, paraphrases, or fuzzy wording. It is weaker when a query depends on an exact product code, ticket ID, function name, or rare keyword because embeddings may not give that individual token enough weight. Keyword retrieval, commonly BM25, has the opposite profile: it is strong for exact terms and rare tokens but weaker for semantic matches.

## Retrieval Shape

A simple hybrid retriever has three parts:

1. A vector retriever that embeds the query and ranks chunks by semantic similarity.
2. A keyword retriever that ranks chunks by lexical evidence such as BM25.
3. A merge or weighting step that combines the scores or candidate lists.

The weighting can be static, but the agentic RAG source argues that an agent can choose the query, retry retrieval, and adjust the balance between keyword and vector evidence based on the user request. That is useful when the agent can tell that a query contains an ID or specific phrase and should favor lexical matching.

## Agentic RAG Use

In an agentic RAG system, retrieval becomes a tool rather than a fixed pre-processing step. The agent can rewrite the search query, fetch more chunks if the first retrieval is insufficient, and change the hybrid weighting as it gathers evidence. This connects retrieval quality directly to [[agent-facing-interface-design|Agent-Facing Interface Design]] ([Agent-Facing Interface Design](agent-facing-interface-design.md)): the retrieval tool needs clear arguments, bounded results, and enough metadata for the agent to know whether to continue.

## Memory Retrieval Use

The agentmemory source applies hybrid retrieval to persistent agent memory. Memory search needs exact matching for identifiers, file names, issue numbers, and decisions, while also supporting semantic recall for related project context. The repository describes a combination of BM25, vector, and graph retrieval, with ranking fusion and session diversification. Performance claims in that source should be treated as source-reported, but the architecture reflects the same principle as RAG: exact and semantic evidence solve different retrieval failures.

## See Also

- [[semantic-search|Semantic Search]] ([Semantic Search](semantic-search.md)) - the vector half of most hybrid retrieval systems.
- [[llm-application-observability|LLM Application Observability]] ([LLM Application Observability](../topics/llm-application-observability.md)) - retrieval relevance and groundedness need measurement in production RAG systems.
- [[llm-ready-tooling|LLM-Ready Tooling]] ([LLM-Ready Tooling](../references/llm-ready-tooling.md)) - content preparation and knowledge-base tooling often feed hybrid retrieval.
- [[agent-memory-systems|Agent Memory Systems]] ([Agent Memory Systems](../topics/agent-memory-systems.md)) - persistent memory systems depend on hybrid retrieval for recall.

## Sources

- [How to Build Agentic RAG with Hybrid Search](../../raw/articles/2026-07-05-how-to-build-agentic-rag-with-hybrid-search.md) - source for hybrid search and agentic retrieval patterns.
- [Build Semantic Search with LLM Embeddings](../../raw/articles/2026-07-05-build-semantic-search-with-llm-embeddings.md) - source for the vector retrieval baseline.
- [rohitg00/agentmemory](../../raw/repos/2026-07-05-rohitg00-agentmemory.md) - source for memory retrieval using BM25, vector, and graph evidence.
