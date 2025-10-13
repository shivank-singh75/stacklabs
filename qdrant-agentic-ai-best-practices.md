# 🤖 Qdrant Best Practices for Agentic AI (Node.js)

This document explains **how to design, structure, and optimize** intent detection for **Agentic AI systems** using **Qdrant Vector DB** in **Node.js**.

---

## 🧠 Overview

In an **Agentic AI** system, multiple specialized agents (like `ai-receptionist`, `ai-support-agent`, etc.) handle different types of user queries.  
To determine which agent a query belongs to, we use **semantic intent detection** powered by **vector similarity search** in **Qdrant**.

You can design your Qdrant storage in two ways:

- **Single Collection**: All agent intents are stored in one collection.
- **Multi-Collection**: Each agent type has its own isolated collection.

---

## ⚖️ Single vs Multi Collection Comparison

| Feature | Single Collection | Multi Collection (Recommended) |
|----------|-------------------|-------------------------------|
| **Setup Complexity** | ✅ Easy setup | ⚠️ Slightly more setup |
| **Search Speed** | ⚠️ Slower with large data | ✅ Faster (less noise per agent) |
| **Accuracy** | ⚠️ May mix intents across agents | ✅ Isolated, higher precision |
| **Scalability** | ⚠️ Hard to scale | ✅ Scales independently per agent |
| **Model/Threshold per Agent** | ❌ Shared across all | ✅ Tunable per agent |
| **Maintenance** | ✅ Simple | ⚠️ More config but cleaner long-term |
| **Use Case** | ≤2 agents or small demo | ≥3 agents or production |

**✅ Recommendation:**  
Use **Multi-Collection** when you have multiple distinct agent types.  
It improves **accuracy**, **isolation**, and **maintainability**.

---

## 🧩 Recommended Folder Structure

```
/agentic-ai
 ├── /src
 │   ├── /agents
 │   │   ├── receptionistAgent.js
 │   │   ├── supportAgent.js
 │   │   └── baseAgent.js
 │   ├── /qdrant
 │   │   ├── qdrantClient.js
 │   │   ├── collectionManager.js
 │   │   └── index.js
 │   ├── embeddings.js
 │   └── detectIntent.js
 ├── .env
 ├── package.json
 └── qdrant-agentic-ai-best-practices.md
```

---

## ⚙️ Step-by-Step Implementation

### 1. Qdrant Client Setup

```js
// src/qdrant/qdrantClient.js
import { QdrantClient } from "@qdrant/js-client-rest";

export const qdrant = new QdrantClient({
  url: process.env.QDRANT_URL || "http://localhost:6333",
  apiKey: process.env.QDRANT_API_KEY,
});
```

---

### 2. Create Collections per Agent

```js
// src/qdrant/collectionManager.js
import { qdrant } from "./qdrantClient.js";

export async function ensureCollection(agentType) {
  const name = `agent_${agentType}`;
  const { collections } = await qdrant.getCollections();
  const exists = collections.some(c => c.name === name);

  if (!exists) {
    await qdrant.createCollection(name, {
      vectors: {
        size: 1536, // based on embedding model
        distance: "Cosine",
      },
    });
    console.log(`✅ Created collection: ${name}`);
  }
}
```

---

### 3. Insert Intents per Agent Type

```js
// src/agents/receptionistAgent.js
import { qdrant } from "../qdrant/qdrantClient.js";

export async function addReceptionistIntent(id, text, vector) {
  await qdrant.upsert("agent_receptionist", {
    points: [
      {
        id,
        vector,
        payload: { text, agentType: "receptionist" },
      },
    ],
  });
}
```

---

### 4. Query Across All Agent Collections

```js
// src/detectIntent.js
import { qdrant } from "./qdrant/qdrantClient.js";
import { getEmbedding } from "./embeddings.js";

const AGENTS = ["receptionist", "support"];

export async function detectAgentIntent(query) {
  const vector = await getEmbedding(query);

  const results = await Promise.all(
    AGENTS.map(async (agent) => {
      const res = await qdrant.search(`agent_${agent}`, {
        vector,
        limit: 1,
        score_threshold: 0.78,
      });
      return { agent, match: res[0] || null };
    })
  );

  const best = results
    .filter(r => r.match)
    .sort((a, b) => b.match.score - a.match.score)[0];

  return best || { agent: null, match: null };
}
```

---

## 🚀 Hybrid Option (Single Collection + Filters)

If you prefer a single collection, use payload filters for logical separation:

```js
await qdrant.search("agent_intents", {
  vector: queryVector,
  filter: {
    must: [{ key: "agentType", match: { value: "receptionist" } }],
  },
});
```

⚠️ Works fine for small-scale projects but less accurate than isolated collections.

---

## 🎯 Accuracy Tips

1. Normalize user queries (lowercase, remove stopwords).
2. Use domain-specific embeddings per agent.
3. Tune `score_threshold` per agent.
4. Use Qdrant’s **Named Vectors** for multi-embedding (title, examples, description).
5. Add intent metadata: `category`, `language`, `context_tags`, etc.

---

## 🧰 Dependencies

- [`@qdrant/js-client-rest`](https://www.npmjs.com/package/@qdrant/js-client-rest)
- [`openai`](https://www.npmjs.com/package/openai)
- [`dotenv`](https://www.npmjs.com/package/dotenv)

---

## 🧭 Recommendation Summary

| Scenario | Recommended Setup |
|-----------|------------------|
| 🧪 Small POC / ≤2 agents | **Single Collection with Filters** |
| ⚙️ Production / >2 agents | **Multi Collection per Agent** |
| 🚀 Accuracy Priority | **Multi Collection + Multi-Vector Intents** |

---

## 📚 References

- [Qdrant Docs – Collections](https://qdrant.tech/documentation/concepts/collections/)
- [Qdrant Docs – Filters](https://qdrant.tech/documentation/concepts/filters/)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)
- [Sentence Transformers (HuggingFace)](https://www.sbert.net/)

---

## ✅ Conclusion

**Multi-Collection architecture** is the most scalable and accurate structure for **Agentic AI intent detection**.  
Each agent’s vector space stays isolated, giving you **faster queries**, **better accuracy**, and **independent retraining**.

---
