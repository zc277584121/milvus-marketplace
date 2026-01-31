---
name: indexing
description: "Use when user needs to create collections, indexes in Milvus. Triggers on: indexing, collection, create index, HNSW, IVF, schema, Milvus collection, vector storage."
---

# Indexing - Index Management

Create Collections and indexes in Milvus.

## Index Type Selection

| Data Scale | Recommended Index | Features |
|------------|-------------------|----------|
| <1M | HNSW | High precision, high memory |
| 1M-10M | IVF_FLAT | Balanced |
| >10M | IVF_PQ | Compressed storage |
| Need exact | FLAT | Brute force search |

## Code Examples

### Create Collection

```python
from pymilvus import connections, Collection, FieldSchema, CollectionSchema, DataType

# Connect
connections.connect(host="localhost", port="19530")

# Define fields
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name="text", dtype=DataType.VARCHAR, max_length=65535),
    FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=1024)
]

# Create
schema = CollectionSchema(fields, description="My collection")
collection = Collection("my_collection", schema)
```

### Create Index

**HNSW (Recommended, <1M data)**
```python
index_params = {
    "index_type": "HNSW",
    "metric_type": "COSINE",  # or L2, IP
    "params": {"M": 16, "efConstruction": 256}
}
collection.create_index("embedding", index_params)
```

**IVF_FLAT (1M-10M)**
```python
index_params = {
    "index_type": "IVF_FLAT",
    "metric_type": "L2",
    "params": {"nlist": 1024}
}
collection.create_index("embedding", index_params)
```

**IVF_PQ (>10M, memory efficient)**
```python
index_params = {
    "index_type": "IVF_PQ",
    "metric_type": "L2",
    "params": {"nlist": 1024, "m": 8, "nbits": 8}
}
collection.create_index("embedding", index_params)
```

### Load Collection

```python
collection.load()
```

### Collection with Partitions

```python
# Create partitions
collection.create_partition("2024_01")
collection.create_partition("2024_02")

# Insert with partition
collection.insert(data, partition_name="2024_01")

# Search with partition
collection.search(..., partition_names=["2024_01"])
```

### With Scalar Index

```python
# Create index for VARCHAR field (speeds up filtering)
collection.create_index(
    field_name="category",
    index_name="category_index"
)
```

## Distance Metric Selection

| Type | Use Case | Notes |
|------|----------|-------|
| COSINE | Text similarity | Normalized vectors |
| L2 | Euclidean distance | General purpose |
| IP | Inner product | Recommendation systems |

## Next Steps

After index creation:
- Import data: Use `core:data-ingestion`
- Start searching: Use corresponding scenario skill
