---
name: rerank
description: "Use when user needs to improve search relevance with reranking. Triggers on: rerank, relevance, cross-encoder, search quality, top-k reranking, second-stage ranking."
---

# Rerank - Search Result Reranking

Use Cross-Encoder to re-rank initial search results for improved search relevance.

## Why Rerank

| Method | Speed | Precision | Use Case |
|--------|-------|-----------|----------|
| Vector Search (Bi-Encoder) | Fast | Medium | Large-scale recall |
| Rerank (Cross-Encoder) | Slow | High | Refine Top-K |

**Best Practice**: Vector search recalls 100 → Rerank takes Top 10

## Model Selection

| Model | Language | Features |
|-------|----------|----------|
| BAAI/bge-reranker-large | Chinese | Best for Chinese |
| BAAI/bge-reranker-v2-m3 | Multilingual | Good results, slower |
| cross-encoder/ms-marco-MiniLM-L-6-v2 | English | Fast |
| Cohere rerank-english-v2.0 | English | API, high quality |

## Code Examples

### Basic Rerank

```python
from sentence_transformers import CrossEncoder

# Load model
reranker = CrossEncoder('BAAI/bge-reranker-large')

# Original search results
query = "What is a vector database"
candidates = [
    "A vector database is a database specialized for storing and retrieving vectors",
    "Relational databases use SQL queries",
    "Milvus is an open source vector database",
    "Database indexes can speed up queries",
]

# Rerank
pairs = [[query, doc] for doc in candidates]
scores = reranker.predict(pairs)

# Sort by score
results = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)
for doc, score in results:
    print(f"{score:.3f}: {doc}")
```

### Combined with Vector Search

```python
from pymilvus import MilvusClient
from sentence_transformers import SentenceTransformer, CrossEncoder

# Initialize
embedding_model = SentenceTransformer('BAAI/bge-large-zh-v1.5')
reranker = CrossEncoder('BAAI/bge-reranker-large')
client = MilvusClient("./milvus.db")

def search_with_rerank(query, top_k=10, rerank_top=100):
    """Vector search + Rerank"""
    # 1. Vector search recalls more results
    query_embedding = embedding_model.encode(query).tolist()
    results = client.search(
        collection_name="docs",
        data=[query_embedding],
        limit=rerank_top,
        output_fields=["text"]
    )

    # 2. Extract candidate documents
    candidates = [hit["entity"]["text"] for hit in results[0]]

    # 3. Rerank
    pairs = [[query, doc] for doc in candidates]
    scores = reranker.predict(pairs)

    # 4. Return Top-K
    ranked = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)
    return ranked[:top_k]

# Usage
results = search_with_rerank("What is RAG")
for doc, score in results:
    print(f"{score:.3f}: {doc[:50]}...")
```

### Using Cohere Rerank API

```python
import cohere

co = cohere.Client("your-api-key")

def cohere_rerank(query, documents, top_k=10):
    response = co.rerank(
        model="rerank-english-v2.0",
        query=query,
        documents=documents,
        top_n=top_k
    )
    return [(doc.document["text"], doc.relevance_score)
            for doc in response.results]
```

### Batch Rerank

```python
def batch_rerank(queries, candidates_list, batch_size=32):
    """Batch reranking"""
    all_pairs = []
    pair_indices = []

    # Build all query-doc pairs
    for i, (query, candidates) in enumerate(zip(queries, candidates_list)):
        for doc in candidates:
            all_pairs.append([query, doc])
            pair_indices.append(i)

    # Batch predict
    all_scores = reranker.predict(all_pairs, batch_size=batch_size)

    # Reorganize results
    results = [[] for _ in queries]
    for (query, doc), score, idx in zip(all_pairs, all_scores, pair_indices):
        results[idx].append((doc, score))

    # Sort
    return [sorted(r, key=lambda x: x[1], reverse=True) for r in results]
```

## Performance Optimization

```python
# GPU acceleration
reranker = CrossEncoder('BAAI/bge-reranker-large', device='cuda')

# Control recall count (balance speed and quality)
# Reranking 50-200 items is usually sufficient

# Cache rerank results for common queries
from functools import lru_cache

@lru_cache(maxsize=1000)
def cached_rerank(query, candidates_tuple):
    candidates = list(candidates_tuple)
    pairs = [[query, doc] for doc in candidates]
    scores = reranker.predict(pairs)
    return tuple(sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True))
```

## When to Use Rerank

| Scenario | Needed? |
|----------|---------|
| High search quality requirements | Yes |
| Simple semantic search | Optional |
| Extremely low latency (<50ms) | Not recommended |
| Initial results already accurate | No |

## Next Steps

After rerank optimization:
- Poor results? Check if embedding model is appropriate
- Too slow? Reduce recall count or use smaller model
- Return to main flow: Use `core:pilot`
