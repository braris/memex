---
title: "Build Semantic Search with LLM Embeddings"
source: "https://machinelearningmastery.com/build-semantic-search-with-llm-embeddings/"
type: articles
ingested: 2026-07-05
tags: [semantic-search, embeddings, sentence-transformers, nearest-neighbors, rag]
summary: "A practical walkthrough for building a small semantic search engine with sentence embeddings and nearest-neighbor retrieval. The article contrasts keyword search with embedding-based meaning search, then demonstrates a Python pipeline using AG News, SentenceTransformers, and scikit-learn NearestNeighbors."
---

# Build Semantic Search with LLM Embeddings

Source: [Build Semantic Search with LLM Embeddings](https://machinelearningmastery.com/build-semantic-search-with-llm-embeddings/)

In this article, you will learn how to build a simple semantic search engine using sentence embeddings and nearest neighbors.

Topics we will cover include:

- Understanding the limitations of keyword-based search.
- Generating text embeddings with a sentence transformer model.
- Implementing a nearest-neighbor semantic search pipeline in Python.

Let's get started.

![Build Semantic Search with LLM Embeddings](https://machinelearningmastery.com/wp-content/uploads/2026/03/mlm-building-simple-semantic-search-engine-hero.png)

Build Semantic Search with LLM Embeddings
Image by Editor

## Introduction

Traditional **search engines** have historically relied on keyword search. In other words, given a query like "best temples and shrines to visit in Fukuoka, Japan", results are retrieved based on keyword matching, such that text documents containing co-occurrences of words like "temple", "shrine", and "Fukuoka" are deemed most relevant.

However, this classical approach is notoriously rigid, as it largely relies on exact word matches and misses other important semantic nuances such as synonyms or alternative phrasing, for example, "young dog" instead of "puppy". As a result, highly relevant documents may be inadvertently omitted.

**Semantic search** addresses this limitation by focusing on meaning rather than exact wording. Large language models (LLMs) play a key role here, as some of them are trained to translate text into numerical vector representations called **embeddings**, which encode the semantic information behind the text. When two texts like "small dogs are very curious by nature" and "puppies are inquisitive by nature" are converted into embedding vectors, those vectors will be highly similar due to their shared meaning. Meanwhile, the embedding vectors for "puppies are inquisitive by nature" and "Dazaifu is a signature shrine in Fukuoka" will be very different, as they represent unrelated concepts.

Following this principle, the remainder of this article guides you through the full process of building a compact yet efficient semantic search engine. While minimalistic, it performs effectively and serves as a starting point for understanding how modern search and retrieval systems, such as retrieval augmented generation (RAG) architectures, are built.

The code explained below can be run seamlessly in a Google Colab or Jupyter Notebook instance.

## Step-by-Step Guide

First, we make the necessary imports for this practical example:

```python
import pandas as pd
import json
from pydantic import BaseModel, Field
from openai import OpenAI
from google.colab import userdata
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report
from sklearn.preprocessing import StandardScaler
```

We will use a toy public dataset called `"ag_news"`, which contains texts from news articles. The following code loads the dataset and selects the first 1000 articles.

```python
from datasets import load_dataset
from sentence_transformers import SentenceTransformer
from sklearn.neighbors import NearestNeighbors
```

We now load the dataset and extract the `"text"` column, which contains the article content. Afterwards, we print a short sample from the first article to inspect the data:

```python
print("Loading dataset...")
dataset = load_dataset("ag_news", split="train[:1000]")
# Extract the text column into a Python list
documents = dataset["text"]
print(f"Loaded {len(documents)} documents.")
print(f"Sample: {documents[0][:100]}...")
```

The next step is to obtain embedding vectors, or numerical representations, for our 1000 texts. Some LLMs are trained specifically to translate text into numerical vectors that capture semantic characteristics. Hugging Face sentence transformer models, such as `"all-MiniLM-L6-v2"`, are a common choice. The following code initializes the model and encodes the batch of text documents into embeddings.

```python
print("Loading embedding model...")
model = SentenceTransformer("all-MiniLM-L6-v2")
# Convert text documents into numerical vector embeddings
print("Encoding documents (this may take a few seconds)...")
document_embeddings = model.encode(documents, show_progress_bar=True)
print(f"Created {document_embeddings.shape[0]} embeddings.")
```

Next, we initialize a `NearestNeighbors` object, which implements a nearest-neighbor strategy to find the *k* most similar documents to a given query. In terms of embeddings, this means identifying the closest vectors by smallest angular distance. We use the cosine metric, where more similar vectors have smaller cosine distances and higher cosine similarity values.

```python
search_engine = NearestNeighbors(n_neighbors=5, metric="cosine")
search_engine.fit(document_embeddings)
print("Search engine is ready!")
```

The core logic of our search engine is encapsulated in the following function. It takes a plain-text query, specifies how many top results to retrieve via `top_k`, computes the query embedding, and retrieves the nearest neighbors from the index.

The loop inside the function prints the top-*k* results ranked by similarity:

```python
def semantic_search(query, top_k=3):
    # Embed the incoming search query
    query_embedding = model.encode([query])
    # Retrieve the closest matches
    distances, indices = search_engine.kneighbors(query_embedding, n_neighbors=top_k)
    print(f"\n Query: '{query}'")
    print("-" * 50)
    for i in range(top_k):
        doc_idx = indices[0][i]
        # Convert cosine distance to similarity (1 - distance)
        similarity = 1 - distances[0][i]
        print(f"Result {i+1} (Similarity: {similarity:.4f})")
        print(f"Text: {documents[int(doc_idx)][:150]}...\n")
```

And that's it. To test the function, we can formulate a couple of example search queries:

```python
semantic_search("Wall street and stock market trends")
semantic_search("Space exploration and rocket launches")
```

The results are ranked by similarity. For "Wall street and stock market trends", the example returns stock market and oil price-related articles. For "Space exploration and rocket launches", the example returns articles about NASA propulsion, rocket launch contests, and private spaceflight.

## Summary

What we have built here can be seen as a gateway to retrieval augmented generation systems. While this example is intentionally simple, semantic search engines like this form the foundational retrieval layer in modern architectures that combine semantic search with large language models.

Now that you know how to build a basic semantic search engine, you may want to explore [retrieval augmented generation systems](https://machinelearningmastery.com/understanding-rag-part-i-why-its-needed/) in more depth.
