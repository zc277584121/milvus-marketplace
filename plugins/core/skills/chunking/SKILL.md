---
name: chunking
description: "Use when user needs to split documents into chunks for RAG or search. Triggers on: chunking, split, chunk size, text splitter, token limit, overlap."
---

# Chunking - Document Chunking

Split long documents into smaller chunks suitable for vectorization and retrieval.

## Chunking Strategy Selection

| Scenario | chunk_size | overlap | Notes |
|----------|------------|---------|-------|
| Precise Q&A | 256-512 | 50 | More precise matching |
| Summarization | 1024-2048 | 100 | More complete context |
| Code documentation | By function/class | 0 | Keep code complete |

## Code Examples

### Basic Chunking

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,
    chunk_overlap=50,
    separators=["\n\n", "\n", ".", " ", ""]
)

text = "Your long document content..."
chunks = splitter.split_text(text)
```

### Token-based Chunking

```python
from langchain.text_splitter import TokenTextSplitter

splitter = TokenTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)

chunks = splitter.split_text(text)
```

### Preserving Metadata

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.schema import Document

splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,
    chunk_overlap=50
)

# Document with metadata
doc = Document(
    page_content="Document content...",
    metadata={"source": "doc.pdf", "page": 1}
)

chunks = splitter.split_documents([doc])
# Each chunk retains original metadata
```

### Markdown Chunking

```python
from langchain.text_splitter import MarkdownHeaderTextSplitter

headers = [
    ("#", "h1"),
    ("##", "h2"),
    ("###", "h3"),
]

splitter = MarkdownHeaderTextSplitter(headers_to_split_on=headers)
chunks = splitter.split_text(markdown_text)
```

### Code Chunking

```python
from langchain.text_splitter import (
    Language,
    RecursiveCharacterTextSplitter
)

splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON,
    chunk_size=1000,
    chunk_overlap=100
)

chunks = splitter.split_text(code)
```

## Chunk Quality Check

```python
def analyze_chunks(chunks):
    sizes = [len(c) for c in chunks]
    print(f"Total chunks: {len(chunks)}")
    print(f"Average size: {sum(sizes)/len(sizes):.0f}")
    print(f"Min: {min(sizes)}, Max: {max(sizes)}")

analyze_chunks(chunks)
```

## Next Steps

After chunking:
- Vectorization: Use `core:embedding`
- Storage: Use `core:indexing`
