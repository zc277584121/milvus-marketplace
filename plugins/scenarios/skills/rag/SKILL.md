---
name: rag
description: "Use when user wants to build RAG, Q&A system, or knowledge base. Triggers on: RAG, retrieval augmented generation, Q&A system, knowledge base, document Q&A, chat with docs, ChatGPT for docs, LLM + retrieval."
---

# RAG - Retrieval Augmented Generation

Build intelligent Q&A systems based on documents.

## Complete Implementation

```python
from pymilvus import connections, Collection, FieldSchema, CollectionSchema, DataType
from sentence_transformers import SentenceTransformer
from langchain.text_splitter import RecursiveCharacterTextSplitter
from openai import OpenAI

class RAGSystem:
    def __init__(self, collection_name: str = "rag_kb"):
        connections.connect(host="localhost", port="19530")
        self.embed_model = SentenceTransformer('BAAI/bge-large-zh-v1.5')
        self.llm = OpenAI()
        self.splitter = RecursiveCharacterTextSplitter(
            chunk_size=512,
            chunk_overlap=50
        )
        self.collection = self._init_collection(collection_name)

    def _init_collection(self, name: str):
        fields = [
            FieldSchema("id", DataType.INT64, is_primary=True, auto_id=True),
            FieldSchema("text", DataType.VARCHAR, max_length=65535),
            FieldSchema("source", DataType.VARCHAR, max_length=512),
            FieldSchema("embedding", DataType.FLOAT_VECTOR, dim=1024)
        ]
        schema = CollectionSchema(fields)
        collection = Collection(name, schema)
        collection.create_index("embedding", {
            "index_type": "HNSW",
            "metric_type": "COSINE",
            "params": {"M": 16, "efConstruction": 256}
        })
        collection.load()
        return collection

    def add_document(self, text: str, source: str = ""):
        """Add document"""
        chunks = self.splitter.split_text(text)
        embeddings = self.embed_model.encode(chunks).tolist()
        sources = [source] * len(chunks)
        self.collection.insert([chunks, sources, embeddings])
        self.collection.flush()
        return len(chunks)

    def retrieve(self, query: str, top_k: int = 5):
        """Retrieve relevant chunks"""
        query_embedding = self.embed_model.encode([query]).tolist()

        results = self.collection.search(
            data=query_embedding,
            anns_field="embedding",
            param={"metric_type": "COSINE", "params": {"ef": 64}},
            limit=top_k,
            output_fields=["text", "source"]
        )

        return [
            {"text": hit.entity.get("text"), "source": hit.entity.get("source")}
            for hits in results for hit in hits
        ]

    def generate(self, query: str, contexts: list):
        """Generate answer"""
        context_text = "\n\n".join([
            f"[Source: {c['source']}]\n{c['text']}" for c in contexts
        ])

        response = self.llm.chat.completions.create(
            model="gpt-4",
            messages=[{
                "role": "user",
                "content": f"""Answer the question based on the following reference materials. If there is no relevant information in the materials, please state so.

Reference materials:
{context_text}

Question: {query}

Answer:"""
            }],
            temperature=0.3
        )
        return response.choices[0].message.content

    def query(self, question: str):
        """Complete Q&A workflow"""
        contexts = self.retrieve(question)
        answer = self.generate(question, contexts)
        return {
            "answer": answer,
            "sources": list(set(c["source"] for c in contexts))
        }

# Usage
rag = RAGSystem()

# Add documents
rag.add_document("Milvus is an open-source vector database...", source="milvus_intro.md")
rag.add_document("RAG enhances LLM's answering capabilities through retrieval...", source="rag_guide.md")

# Q&A
result = rag.query("What is Milvus?")
print(result["answer"])
print("Sources:", result["sources"])
```

## Optimization Tips

### 1. Reranking

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder('BAAI/bge-reranker-large')

def rerank(query: str, contexts: list, top_k: int = 3):
    pairs = [[query, c["text"]] for c in contexts]
    scores = reranker.predict(pairs)
    sorted_results = sorted(zip(contexts, scores), key=lambda x: x[1], reverse=True)
    return [r[0] for r in sorted_results[:top_k]]

# Usage
contexts = rag.retrieve(question, top_k=10)
reranked = rerank(question, contexts, top_k=5)
answer = rag.generate(question, reranked)
```

### 2. Multi-turn Conversation

```python
def query_with_history(self, question: str, history: list = None):
    # Combine historical context
    if history:
        context_prompt = "\n".join([
            f"Q: {h['q']}\nA: {h['a']}" for h in history[-3:]
        ])
        question = f"Conversation history:\n{context_prompt}\n\nCurrent question: {question}"

    return self.query(question)
```

### 3. Streaming Output

```python
def stream_generate(self, query: str, contexts: list):
    context_text = "\n\n".join([c['text'] for c in contexts])

    stream = self.llm.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": f"...{context_text}...{query}"}],
        stream=True
    )

    for chunk in stream:
        if chunk.choices[0].delta.content:
            yield chunk.choices[0].delta.content
```

## Chunking Strategy Recommendations

| Document Type | chunk_size | overlap |
|---------------|------------|---------|
| General documents | 512 | 50 |
| Technical docs | 1024 | 100 |
| FAQ | 256 | 0 |
| Legal contracts | 1024 | 200 |

## Related Tools

- Data processing orchestration: `core:ray`
- Document chunking: `core:chunking`
- Vectorization: `core:embedding`
- With reranking: `scenarios:rag-with-rerank`
