# Deployment Guide

From development environment to production deployment.

## Deployment Options

| Option | Use Case | Ops Cost |
|--------|----------|----------|
| Zilliz Cloud | Production recommended | Lowest |
| Docker | Self-hosted small scale | Medium |
| Kubernetes | Self-hosted large scale | Higher |

## Zilliz Cloud Deployment (Recommended)

### 1. Create Cluster

1. Visit [Zilliz Cloud](https://cloud.zilliz.com)
2. Create a Cluster
3. Choose configuration:
   - **Serverless**: Pay-per-use, suitable for development and small scale
   - **Dedicated**: Fixed resources, suitable for production

### 2. Get Connection Info

```
Endpoint: https://xxx.api.gcp-us-west1.zillizcloud.com
API Key: xxx
```

### 3. Update Connection Code

```python
from pymilvus import MilvusClient

# Local development
# client = MilvusClient(host="localhost", port="19530")

# Zilliz Cloud
client = MilvusClient(
    uri="https://xxx.api.gcp-us-west1.zillizcloud.com",
    token="your-api-key"
)
```

### 4. Data Migration

```python
# Export from local
local_client = MilvusClient(host="localhost", port="19530")
data = local_client.query(
    collection_name="my_collection",
    filter="",
    output_fields=["*"],
    limit=100000
)

# Import to Zilliz Cloud
cloud_client = MilvusClient(uri="...", token="...")
cloud_client.insert(collection_name="my_collection", data=data)
```

## Docker Deployment

### Standalone Deployment

```yaml
# docker-compose.yml
version: '3.5'

services:
  milvus:
    image: milvusdb/milvus:latest
    command: ["milvus", "run", "standalone"]
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    ports:
      - "19530:19530"
      - "9091:9091"
    volumes:
      - milvus_data:/var/lib/milvus
    depends_on:
      - etcd
      - minio

  etcd:
    image: quay.io/coreos/etcd:v3.5.5
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
    volumes:
      - etcd_data:/etcd
    command: etcd -advertise-client-urls=http://127.0.0.1:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd

  minio:
    image: minio/minio:RELEASE.2023-03-20T20-16-18Z
    environment:
      MINIO_ACCESS_KEY: minioadmin
      MINIO_SECRET_KEY: minioadmin
    volumes:
      - minio_data:/minio_data
    command: minio server /minio_data

volumes:
  milvus_data:
  etcd_data:
  minio_data:
```

```bash
docker-compose up -d
```

### Application Containerization

```dockerfile
# Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

```
# requirements.txt
pymilvus>=2.3.0
sentence-transformers>=2.2.0
fastapi>=0.100.0
uvicorn>=0.23.0
```

## Kubernetes Deployment

### Helm Install Milvus

```bash
# Add repo
helm repo add milvus https://milvus-io.github.io/milvus-helm/
helm repo update

# Install
helm install my-milvus milvus/milvus \
  --set cluster.enabled=true \
  --set service.type=LoadBalancer
```

### Application Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:latest
        ports:
        - containerPort: 8000
        env:
        - name: MILVUS_HOST
          value: "my-milvus.default.svc.cluster.local"
        - name: MILVUS_PORT
          value: "19530"
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8000
  selector:
    app: my-app
```

## Configuration Management

### Environment Variables

```python
import os

config = {
    "milvus_host": os.getenv("MILVUS_HOST", "localhost"),
    "milvus_port": os.getenv("MILVUS_PORT", "19530"),
    "milvus_token": os.getenv("MILVUS_TOKEN", ""),
    "collection_name": os.getenv("COLLECTION_NAME", "default"),
}
```

### Configuration File

```yaml
# config.yaml
milvus:
  host: ${MILVUS_HOST:localhost}
  port: ${MILVUS_PORT:19530}
  token: ${MILVUS_TOKEN:}

embedding:
  model: BAAI/bge-large-zh-v1.5
  device: cuda

search:
  default_limit: 10
  ef: 64
```

## Production Checklist

### Before Deployment
- [ ] Configuration read from environment variables
- [ ] No sensitive info in code
- [ ] Logging configured correctly
- [ ] Error handling complete

### After Deployment
- [ ] Health check working
- [ ] Monitoring and alerts configured
- [ ] Backup strategy in place
- [ ] Scaling tested

## Monitoring

### Health Check Endpoint

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/health")
def health():
    try:
        # Check Milvus connection
        from pymilvus import connections
        connections.connect(host=config["milvus_host"], port=config["milvus_port"])
        return {"status": "healthy"}
    except Exception as e:
        return {"status": "unhealthy", "error": str(e)}, 500
```

### Key Metrics

- Request latency (P50/P95/P99)
- QPS
- Error rate
- Milvus connection status

## Next Steps

After deployment:
- Demo showcase → `demo.md`
- Return to controller → `pilot`
