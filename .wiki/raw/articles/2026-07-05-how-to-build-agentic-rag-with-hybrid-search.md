---
title: "How to Build Agentic RAG with Hybrid Search"
source: "https://towardsdatascience.com/how-to-build-agentic-rag-with-hybrid-search/"
type: articles
ingested: 2026-07-05
tags: [rag, agentic-rag, hybrid-search, vector-search, bm25, information-retrieval, llm-agents]
summary: "Article explaining why hybrid search combines vector similarity with keyword retrieval such as BM25, and how an agentic RAG system can use retrieval as a tool for query rewriting, iterative fetching, and dynamic weighting between semantic and keyword search."
fetched: 2026-07-05
---


# [How to Build Agentic RAG with Hybrid Search](https://towardsdatascience.com/how-to-build-agentic-rag-with-hybrid-search/)

, also known as RAG, is a powerful method to find relevant documents in a corpus of information, which you then provide to an LLM to give answers to user questions.

Traditionally, RAG first uses vector similarity to find relevant chunks of documents in the corpus and then feeds the most relevant chunks into the LLM to provide a response.

This works really well in a lot of scenarios since semantic similarity is a powerful way to find the most relevant chunks. However, semantic similarity struggles in some scenarios, for example, when a user inputs specific keywords or IDs that need to be explicitly located to be used as a relevant chunk. In these instances, vector similarity is not that effective, and you need a better approach to find the most relevant chunks.

This is where keyword search comes in, where you find relevant chunks while using keyword search and vector similarity, also known as hybrid search, which is the topic Iâ€™ll be discussing today.

![Learn how to build an agentic hybrid search RAG.](https://contributor.insightmediagroup.io/wp-content/uploads/2026/03/image-135.png)

This infographic highlights the main contents of this article. Iâ€™ll be discussing how you can implement an agentic RAG system using hybrid search. Image by Gemini

## Why use hybrid search

Vector similarity is very powerful. It is able to effectively find relevant chunks from a corpus of documents, even if the input prompt has typos or uses synonyms such as the word lift instead of the word elevator.

However, vector similarity falls short in other scenarios, specifically when searching for specific keywords or identification numbers. The reason for this is that vector similarity doesnâ€™t weigh individual words or IDs specifically highly compared to other words. Thus, keywords or key identifiers are typically drowned in other relevant words, which makes it hard for semantic similarity to find the most relevant chunks.

Keyword search, however, is incredibly good at keywords and specific identifiers, as the name suggests. With BM25, for example, if you have a word that only exists in one document and no other documents, and that word is in the user query, that document will be weighed very highly and most likely included in the search results.

This is the main reason you want to use a hybrid search. Youâ€™re simply able to find more relevant documents if the user is inputting keywords into their query.

## How to implement hybrid search

There are numerous ways to implement hybrid search. If you want to implement it yourself, you can do the following.

- Implement vector retrieval via semantic similarity as you would have normally done. I wonâ€™t cover the exact details in this article because itâ€™s out of scope, and the main point of this article is to cover the keyword search part of hybrid search.
- Implement BM25 or another keyword search algorithm that you prefer. BM25 is a standard as it builds upon TF-IDF and has a better formula, making it the better choice. However, the exact keyword search algorithm you use doesnâ€™t really matter, though I recommend using BM25 as the standard.
- Apply a weighting between the similarity found via semantic similarity and keyword search similarity. You can decide this weighting yourself depending on what you regard as most important. If you have an agent performing a hybrid search, you can also have the agent decide this weighting, as agents will typically have a good intuition for when to use or when to wait, left or similarity more, and when to weigh keyword search similarity more

There are also packages you can use to achieve this, such as TurboPuffer vector storage, which has a Keyboard Search package implemented. To learn how the system really works, however, itâ€™s also recommended that you implement this yourself to try out the system and see if it works.

Overall, however, hybrid search isnâ€™t really that difficult to implement and can give a lot of benefits. If youâ€™re looking into a hybrid search, you typically know how vector search itself works and you simply need to add the keyword search element to it. Keyword search itself is not really that complicated either, which makes hybrid search a relatively simple thing to implement, which can yield a lot of benefits.

## Agentic hybrid search

Implementing hybrid search is great, and it will probably improve how well your RAG system works right off the bat. However, I believe that if you really want to get the most out of a hybrid search RAG system, you need to make it agentic.

By making it agentic, I mean the following. A typical RAG system first fetches relevant chunks, document chunks, feeds those chunks into an LLM, and has it answer a user question

However, an agentic RAG system does it a bit differently. Instead of doing the trunk retrieval before using an LLM to answer, you make the trunk retrieval function a tool that the LLM can access. This, of course, makes the LLM agentic, so it has access to a tool and has several major advantages:

- The agent can itself decide the prompt to use for the vector search. So instead of using only the exact user prompt, it can rewrite the prompt to get even better vector search results. Query rewriting is a well-known technique you can use to improve RAG performance.
- The agent can iteratively fetch the information, so it can first do one vector search call, check if it has enough information to answer a question, and if not, it can fetch even more information. This makes it so the agent can review the information it fetched and, if needed, fetch even more information, which will make it better able to answer questions.
- The agent can decide the weighting between keyword search and vector similarity itself. This is incredibly powerful because the agent typically knows if itâ€™s searching for a keyword or if itâ€™s searching for semantically similar content. For example, if the user included a keyword in their search query, the agent will likely weigh the keyword search element of hybrid search higher, and letâ€™s get even better results. This works a lot better than having a static number for the weighting between keyword search and vector similarity.

Todayâ€™s Frontier LLMs are incredibly powerful and will be able to make all of these judgments themselves. Just a few months ago, I would doubt if you should give the agent as much freedom as I described in the bullet points above, having it select prompt use, iteratively fetching information, and the weighting between keyword search and semantic similarity. However, today I know that the latest Frontier LLMs have become so powerful that this is very doable and even something I recommend implementing.

Thus, by both implementing HybridSearch and by making it agentic, you can really supercharge your RAG system and achieve far better results than you would have achieved with a static vector similarity-only RAG system.

## Conclusion

In this article, Iâ€™ve discussed how to implement hybrid search into your RAG system. Furthermore, I described how to make your RAG system authentic to achieve far better results. Combining these two techniques will lead to an incredible performance increase in your information retrieval system, and it can, in fact, be implemented quite easily using coding agents such as Claude Code. I believe Agentex Systems is the future of information retrieval, and I urge you to provide effective information retrieval tools, such as a hybrid search, to your agents and make them perform the rest of the work.

