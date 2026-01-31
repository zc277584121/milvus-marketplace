---
name: embedding
description: "Use when user needs to convert text/images to vectors. Triggers on: embedding, vectorize, encode, text-to-vector, model selection, sentence-transformers, OpenAI embeddings, BGE, CLIP."
---

# Embedding - Vectorization Model Selection

Convert text, images, and other data into vectors - the foundation of all vector retrieval applications.

## Quick Selection

### By User Type

| User Type | Recommended | Reason |
|-----------|-------------|--------|
| **Beginners** (don't want to manage infrastructure) | OpenAI / Cohere API | Sign up and use, no GPU needed |
| **Advanced users** (seeking best results) | BGE-M3 / Voyage AI | Open source control / best for specialized domains |
| **Chinese scenarios** | BGE-M3 or Qwen3-Embedding | Best Chinese performance |
| **Image scenarios** | CLIP / Chinese-CLIP | Cross-modal image-text |

### By Scenario

| Scenario | Recommended Model | Notes |
|----------|-------------------|-------|
| General text search | OpenAI text-embedding-3-small | Best value |
| High-precision RAG | Voyage-3-large / Cohere embed-v4 | MTEB top performers |
| Chinese knowledge base | BGE-M3 / BGE-large-zh | Chinese optimized |
| Multilingual | BGE-M3 / multilingual-e5 | 100+ languages |
| Code search | Voyage-code-3 | Code specialized |
| Legal/Finance | Voyage-finance/law | Domain specialized |
| Image search | CLIP-ViT-L-14 | General image-text |
| Chinese image-text | Chinese-CLIP | Chinese image-text |

---

## Model Overview

### API Models (Ready to Use)

Suitable for: Beginners, no GPU management, quick validation

| Model | Provider | Dimensions | Context | Price ($/1M tokens) | MTEB | Features |
|-------|----------|------------|---------|---------------------|------|----------|
| **text-embedding-3-large** | OpenAI | 3072 | 8K | $0.13 | 64.6 | Best general, mature ecosystem |
| **text-embedding-3-small** | OpenAI | 1536 | 8K | $0.02 | 62.3 | Best value |
| **embed-v4** | Cohere | 1024 | 512 | $0.10 | 65.2 | Multilingual, compressible |
| **voyage-3-large** | Voyage AI | 1024 | 32K | $0.06 | 65.8 | Long text, domain specialized |
| **voyage-3-lite** | Voyage AI | 512 | 32K | $0.02 | 61.2 | Low cost, low latency |

### Open Source Models (Local Deployment)

Suitable for: Advanced users, data privacy requirements, seeking best performance

| Model | Source | Dimensions | Context | Size | MTEB | Features |
|-------|--------|------------|---------|------|------|----------|
| **BGE-M3** | BAAI | 1024 | 8K | 2.2GB | 63.0 | Multi-lingual/functional/granular, best Chinese |
| **BGE-large-zh-v1.5** | BAAI | 1024 | 512 | 1.3GB | - | Pure Chinese, lightweight |
| **Qwen3-Embedding-8B** | Alibaba | 4096 | 32K | 16GB | 66.2 | Latest, best performance |
| **Qwen3-Embedding-0.6B** | Alibaba | 1024 | 32K | 1.2GB | 58.5 | Lightweight, surpasses BGE-M3 at same size |
| **jina-embeddings-v3** | Jina AI | 1024 | 8K | 1.2GB | 62.8 | Multi-task, adjustable dimensions |
| **nomic-embed-text** | Nomic | 768 | 8K | 548MB | 59.3 | Open source free, local first choice |
| **all-MiniLM-L6-v2** | SBERT | 384 | 256 | 80MB | 56.3 | Ultra lightweight, prototyping |

### Image Models

| Model | Source | Dimensions | Features |
|-------|--------|------------|----------|
| **CLIP-ViT-L-14** | OpenAI | 768 | Highest accuracy |
| **CLIP-ViT-B-32** | OpenAI | 512 | Fast |
| **Chinese-CLIP-ViT-H** | OFA-Sys | 1024 | Chinese optimized |
| **SigLIP** | Google | 1152 | Next generation, better performance |

---

## Cost Calculation

### API Model Monthly Cost Estimate

Assumption: 1M documents, average 500 tokens/document

| Model | Indexing Cost | Query Cost (100k/month) | Monthly Total |
|-------|---------------|-------------------------|---------------|
| OpenAI small | $10 | $2 | ~$12 |
| OpenAI large | $65 | $13 | ~$78 |
| Cohere v4 | $50 | $10 | ~$60 |
| Voyage-3 | $30 | $6 | ~$36 |
| Voyage-3-lite | $10 | $2 | ~$12 |

### Open Source Model Deployment Cost

| Model | Minimum GPU | Cloud GPU Monthly Cost |
|-------|-------------|------------------------|
| BGE-M3 | RTX 3090 (24GB) | ~$150-300 |
| Qwen3-0.6B | RTX 3060 (12GB) | ~$80-150 |
| all-MiniLM | CPU | $0 |

---

## Selection Decision Tree

```
Start
  │
  ├─ Have GPU?
  │    ├─ No → API models
  │    │         ├─ Tight budget → OpenAI small / Voyage-lite
  │    │         ├─ Best results → Voyage-3-large / Cohere v4
  │    │         └─ General use → OpenAI large
  │    │
  │    └─ Yes → Open source models
  │              ├─ Mainly Chinese → BGE-M3 / Qwen3-Embedding
  │              ├─ Multilingual → BGE-M3
  │              ├─ Ultra lightweight → all-MiniLM / nomic-embed
  │              └─ Images → CLIP / Chinese-CLIP
  │
  └─ Special domain?
       ├─ Code → Voyage-code-3
       ├─ Legal → Voyage-law-2
       └─ Finance → Voyage-finance-2
```

---

## Quick Code Examples

### Local Model (Recommended for Beginners)

```python
from sentence_transformers import SentenceTransformer

# Load model (auto-downloads on first use)
model = SentenceTransformer('BAAI/bge-large-zh-v1.5')

# Single encoding
text = "This is a sample text"
embedding = model.encode(text).tolist()

# Batch encoding (recommended)
texts = ["Text 1", "Text 2", "Text 3"]
embeddings = model.encode(texts, batch_size=32, normalize_embeddings=True)
```

### API Model

```python
from openai import OpenAI

client = OpenAI()  # Requires OPENAI_API_KEY

response = client.embeddings.create(
    model="text-embedding-3-small",
    input=["Text 1", "Text 2"]
)
embeddings = [item.embedding for item in response.data]
```

---

## Detailed Integration Guides

See `verticals/` directory for detailed integration for each model (registration, installation, code examples):

### API Models
- `openai.md` - OpenAI text-embedding-3
- `cohere.md` - Cohere Embed v4
- `voyage.md` - Voyage AI (general/code/legal/finance)
- `aliyun.md` - Alibaba Cloud DashScope

### Open Source Models
- `bge.md` - BGE series (BAAI)
- `qwen-embedding.md` - Qwen3-Embedding (Alibaba open source)
- `jina.md` - Jina Embeddings
- `nomic.md` - Nomic Embed
- `minilm.md` - all-MiniLM (lightweight)

### Image Models
- `clip.md` - CLIP / Chinese-CLIP
- `siglip.md` - SigLIP

---

## FAQ

### Q: Are more dimensions always better?
Not necessarily. More dimensions mean higher storage costs and slower search. 512-1024 dimensions are usually sufficient; 3072 dimensions are only necessary for extremely high precision scenarios.

### Q: Can I mix different models in the same Collection?
No. The same Collection must use the same model, otherwise the vector spaces are different and cannot be compared.

### Q: Must queries and documents use the same model?
Yes, queries and documents must be encoded with the same model.

### Q: Does OpenAI work well for Chinese?
It works, but BGE-M3 / Qwen3-Embedding perform better for Chinese scenarios.

---

## Related Tools

- Batch processing: `core:ray`
- Chunking strategy: `core:chunking`
- Index management: `core:indexing`
- Reranking: `core:rerank`
