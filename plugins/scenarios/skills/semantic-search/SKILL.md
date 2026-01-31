---
name: semantic-search
description: "Use when user wants to build semantic/text search. Triggers on: semantic search, text search, full-text search, natural language search, find similar text, vector search."
---

# Semantic Search

Build vector-based semantic search systems.

## Complete Implementation

```python
from pymilvus import connections, Collection, FieldSchema, CollectionSchema, DataType
from sentence_transformers import SentenceTransformer

class SemanticSearch:
    def __init__(self, collection_name: str = "semantic_search"):
        connections.connect(host="localhost", port="19530")
        self.model = SentenceTransformer('BAAI/bge-large-zh-v1.5')
        self.dim = 1024
        self.collection = self._init_collection(collection_name)

    def _init_collection(self, name: str):
        fields = [
            FieldSchema("id", DataType.INT64, is_primary=True, auto_id=True),
            FieldSchema("text", DataType.VARCHAR, max_length=65535),
            FieldSchema("embedding", DataType.FLOAT_VECTOR, dim=self.dim)
        ]
        schema = CollectionSchema(fields)
        collection = Collection(name, schema)

        # Create index
        collection.create_index("embedding", {
            "index_type": "HNSW",
            "metric_type": "COSINE",
            "params": {"M": 16, "efConstruction": 256}
        })
        collection.load()
        return collection

    def add(self, texts: list):
        """Add documents"""
        embeddings = self.model.encode(texts).tolist()
        self.collection.insert([texts, embeddings])
        self.collection.flush()

    def search(self, query: str, limit: int = 10):
        """Search"""
        query_embedding = self.model.encode([query]).tolist()

        results = self.collection.search(
            data=query_embedding,
            anns_field="embedding",
            param={"metric_type": "COSINE", "params": {"ef": 64}},
            limit=limit,
            output_fields=["text"]
        )

        return [
            {"text": hit.entity.get("text"), "score": hit.score}
            for hits in results for hit in hits
        ]

# Usage
search = SemanticSearch()
search.add(["Python is a programming language", "Java is also a programming language", "Machine learning is popular"])
results = search.search("what is programming")
for r in results:
    print(f"{r['score']:.3f}: {r['text']}")
```

## Search with Filtering

```python
# Add category field to schema
FieldSchema("category", DataType.VARCHAR, max_length=256)

# Filter during search
results = collection.search(
    data=query_embedding,
    anns_field="embedding",
    param={"metric_type": "COSINE", "params": {"ef": 64}},
    limit=10,
    expr='category == "tech"',  # Filter condition
    output_fields=["text", "category"]
)
```

## Hybrid Search (Vector + Keyword)

```python
from pymilvus import AnnSearchRequest, RRFRanker

# Vector search
vector_req = AnnSearchRequest(
    data=query_embedding,
    anns_field="embedding",
    param={"metric_type": "COSINE", "params": {"ef": 64}},
    limit=20
)

# Keyword search (requires BM25 enabled)
text_req = AnnSearchRequest(
    data=[query],
    anns_field="text",
    param={"metric_type": "BM25"},
    limit=20
)

# Fusion
results = collection.hybrid_search(
    reqs=[vector_req, text_req],
    ranker=RRFRanker(k=60),
    limit=10,
    output_fields=["text"]
)
```

## Performance Optimization

```python
# 1. GPU accelerated embedding
model = SentenceTransformer('...', device='cuda')

# 2. Batch search
queries = ["query1", "query2", "query3"]
embeddings = model.encode(queries).tolist()
results = collection.search(data=embeddings, ...)

# 3. Adjust ef parameter
# Higher ef = higher precision, slower speed
param={"params": {"ef": 128}}  # High precision
param={"params": {"ef": 32}}   # High speed
```

## Related Tools

- Chunking: `core:chunking`
- Embedding: `core:embedding`
- Indexing: `core:indexing`
