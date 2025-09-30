---
title: Using Qdrant in Agentic AI Projects
---

# 1. Full Flow of Qdrant in Agentic AI

Qdrant plays the role of semantic memory in an Agentic AI system. It
stores embeddings of knowledge, past interactions, and intents, enabling
the AI to recall relevant information based on semantic similarity
rather than just keyword matching.

## Flow Steps

1\. User Input → Query\
2. Query Embedding → Vector generation using an embedding model\
3. Semantic Search in Qdrant → Retrieves relevant docs, memories,
intents\
4. Structured Data Lookup (Postgres) → Fetches facts and transactional
data\
5. Agent Reasoning (LLM) → Combines context from Qdrant + facts from
Postgres\
6. Response to User → Natural language output\
7. Feedback & Memory Update → Stores conversation back into Qdrant

# 2. Advanced Intent Detection

The orchestrator can use multiple methods to detect user intent with
maximum accuracy:

## Rule-Based Detection (Fast Path)

\- Speed: \~5ms\
- Accuracy: 70-80%\
- Use Case: Common phrases and exact matches\
- Example: \"book appointment\" → appointment_scheduling

## Vector-Based Detection (Semantic Search)

\- Speed: \~200ms\
- Accuracy: 85-90%\
- Use Case: Natural language variations\
- Example: \"I need to schedule a meeting\" → appointment_scheduling

## LLM-Based Detection (Context Understanding)

\- Speed: \~200ms\
- Accuracy: 90-95%\
- Use Case: Complex, contextual messages\
- Example: \"My package hasn\'t arrived and I\'m worried\" →
order_management

## Hybrid Detection (Maximum Accuracy)

\- Speed: \~600ms\
- Accuracy: 92-97%\
- Use Case: Critical decisions requiring highest confidence\
- Method: Combines all approaches with weighted scoring

# 3. Approaches to Store Intents/Embeddings in Qdrant

\- Single Vector Representation: Each intent stored as one embedding\
- Multi-Vector Representation: Each intent stored with multiple
embeddings (title, examples, description)\
- Hybrid Representation: Combines vector search with metadata filtering\
- Episodic Memory: Store past conversations as embeddings for recall\
- Domain-Specific Buckets: Different collections per domain or project

# 4. Multi-Vector Representation

Multi-vector representation means each point in Qdrant can hold multiple
embeddings, representing different facets of the same intent, utterance,
or document.

## Why Useful for Intents?

\- Title meanings (short keywords like \"book meeting\")\
- Example utterances (\"Can you help me set up a call?\")\
- Description/context (\"This intent is for scheduling appointments\")

## How to Define Multi-Vectors

Example Qdrant collection definition:

vectors: {\
title_vector: { size: 768, distance: \"Cosine\" },\
example_vector: { size: 768, distance: \"Cosine\" },\
description_vector: { size: 768, distance: \"Cosine\" }\
}

## Querying Multi-Vectors

\- Search on a single vector field (e.g., example_vector)\
- Weighted Hybrid Search across multiple vectors

## Advantages

\- Richer representation of intents\
- More robust matching for short/long/noisy queries\
- Domain-specific weighting flexibility\
- Multi-modal support (text + audio + image)
