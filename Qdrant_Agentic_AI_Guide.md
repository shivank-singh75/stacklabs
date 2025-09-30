content = """# 🚀 Qdrant in Agentic AI Projects — Complete Guide

This guide explains how to use **Qdrant**, **AI models (LLMs & Embeddings)**, and **Postgres** together in an **Agentic AI system** for advanced intent detection, memory, and orchestration.

---

## 🔹 1. Roles of Key Components

| Component        | Role in System | Example |
|------------------|----------------|---------|
| **AI (Embeddings)** | Encodes user queries, intents, and documents into high-dimensional vectors that capture semantic meaning. | "schedule a meeting" → `[0.23, 0.81, ...]` |
| **AI (LLMs)** | Understands context, disambiguates meaning, and makes final reasoning decisions. | Classifies ambiguous queries like "I'm worried about my delivery." |
| **Qdrant Vector DB** | Stores embeddings (semantic memory), enables fast similarity search, supports multi-vector queries. | Finds that "set up a call" is closest to `appointment_scheduling`. |
| **Postgres (RDBMS)** | Stores structured transactional data: users, orders, history, metadata. | Looks up user profile, order status, appointment slots. |
| **LLM Orchestrator** | Acts as the “brain” — combines inputs from Qdrant + Postgres, applies logic, selects the right intent or agent. | Decides whether to trigger scheduling API or order-tracking API. |

---

## 🔹 2. Full Architecture Flow

```mermaid
flowchart TD
    A[🧑 User Input] --> B[🔢 Embedding Generator]
    B --> C[🧠 Qdrant Vector DB]
    A --> D[📜 Rule-Based Classifier]
    A --> E[🤖 LLM Classifier]
    C --> F[⚖️ Hybrid Scorer]
    D --> F
    E --> F
    F --> G[🧩 LLM Orchestrator]
    G --> H[📊 Postgres Lookup]
    H --> G
    G --> I[✅ Final Intent & Action]
    I --> J[💬 Response to User]
    J --> K[📝 Memory Update → Qdrant]
