# Solution Matching Guide

How to match the best solution for user requirements.

## Quick Matching Table

| User Says | Actual Need | Recommended Solution |
|-----------|-------------|---------------------|
| "Search", "find similar" | Semantic search | `scenarios:semantic-search` |
| "Q&A", "knowledge base", "RAG" | Knowledge Q&A | `scenarios:rag` |
| "Document Q&A", "PDF Q&A" | Document QA | `scenarios:doc-qa` |
| "Image-to-image", "image search" | Image search | `scenarios:image-search` |
| "Recommendations", "you might like" | Recommendation system | `scenarios:recommendation` |

## Requirement Recognition

### Semantic Search (`scenarios:semantic-search`)

**Typical expressions**:
- "Search for similar content"
- "Find related articles"
- "Semantic retrieval"
- "Full-text search but understand meaning"

**Core characteristics**:
- Input: Query text
- Output: Similar text list
- No answer generation needed

**Use cases**:
- Content retrieval
- Similar article recommendations
- Duplicate content detection

### RAG (`scenarios:rag`)

**Typical expressions**:
- "Knowledge base Q&A"
- "Let AI answer questions about my data"
- "Document-based conversation"
- "Private knowledge base"

**Core characteristics**:
- Input: User question
- Output: Answer based on knowledge base
- Requires LLM for answer generation

**Use cases**:
- Enterprise knowledge base
- Customer service bots
- Document assistant

### Document Q&A (`scenarios:doc-qa`)

**Typical expressions**:
- "Ask questions about this PDF"
- "Upload document then ask"
- "Analyze this report"

**Core characteristics**:
- Targeting specific documents
- Not a general knowledge base
- Temporary Q&A

**Difference from RAG**:
- RAG: Long-term knowledge base, multiple documents
- Doc-QA: Temporary documents, single or few documents

### Image Search (`scenarios:image-search`)

**Typical expressions**:
- "Image-to-image search"
- "Find similar images"
- "Image retrieval"
- "Search images with text"

**Core characteristics**:
- Images as input or output
- Visual similarity
- CLIP multimodal

**Use cases**:
- E-commerce find similar
- Image library management
- Visual search

### Recommendation System (`scenarios:recommendation`)

**Typical expressions**:
- "Recommend similar products"
- "You might like"
- "Personalized recommendations"
- "Collaborative filtering"

**Core characteristics**:
- Based on user behavior or content similarity
- Personalized
- Real-time requirements

**Use cases**:
- E-commerce recommendations
- Content recommendations
- Social recommendations

## No Match Situations

If no pre-built solution matches:

1. **Still in deep expertise domain**
   - Combine with core tools
   - Reference similar scenario patterns
   - Custom development

2. **In general domain**
   - Provide technical advice
   - Design architecture
   - Write code directly
   - No pre-built toolchain

## Mixed Requirements

User requirements may involve multiple scenarios:

**Example**: "I want to build an e-commerce platform with product search and recommendations"

**Breakdown**:
1. Product search → `scenarios:semantic-search` or `scenarios:image-search`
2. Recommendation system → `scenarios:recommendation`

**Strategy**:
- Implement each module separately
- Share Milvus infrastructure
- Unified data model design

## Next Steps

After solution is determined:
- Enter development workflow → See `development-workflow.md`
- Return to controller → `pilot`
