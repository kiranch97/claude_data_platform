# claude_data_platform: Data Lake on Kubernetes

> P4 Data Platform — MinIO Object Store + PostgreSQL + pgAdmin  
> Part of the kiranch97 Solutions Architecture

---

## Overview

A **production-grade data platform** on Kubernetes providing:

- **MinIO** — S3-compatible object store (datasets, model artifacts, raw data), 20Gi PVC
- **PostgreSQL** — Relational DB for structured data, feature tables, metadata, 10Gi PVC
- **pgAdmin** — Web UI to manage PostgreSQL databases, NodePort 30092
- **MinIO Console** — Web UI to manage buckets and objects, NodePort 30091

---

## Architecture

```
  Data Scientists / ML Pipelines
           |
  +--------+--------+
  |                 |
  v                 v
+----------+   +----------+
| MinIO    |   |PostgreSQL|
| S3 store |   | DB       |
| port:9000|   | port:5432|
| UI: 9001 |   |          |
| PVC: 20Gi|   | PVC: 10Gi|
| NP: 30091|   |          |
+----------+   +----------+
                    |
               +----v-----+
               | pgAdmin  |
               | Web UI   |
               | NP: 30092|
               +----------+
```

---

## Buckets (pre-created)
- `raw-data` — raw datasets uploaded by data engineers
- `processed-data` — cleaned/transformed datasets
- `model-artifacts` — trained model files
- `notebooks-output` — Jupyter notebook exports

---

## Directory Structure

```
claude_data_platform/
├── README.md
└── k8s/
    ├── namespace.yaml
    ├── minio.yaml         # MinIO + 20Gi PVC + NodePort 30091
    ├── minio-init.yaml    # Job: create default buckets
    ├── postgres.yaml      # PostgreSQL + 10Gi PVC
    └── pgadmin.yaml       # pgAdmin Web UI + NodePort 30092
```

---

## Deploy

```bash
git clone https://github.com/kiranch97/claude_data_platform.git
cd claude_data_platform
kubectl apply -f k8s/
```

---

## Access

| Service | URL | Credentials |
|---------|-----|-------------|
| MinIO Console | http://localhost:30091 | minioadmin / minioadmin |
| pgAdmin | http://localhost:30092 | admin@admin.com / admin |
| PostgreSQL | localhost:30093 (NodePort) | postgres / postgres |

---

## Connecting from Jupyter

```python
# PostgreSQL
import psycopg2
conn = psycopg2.connect(
    host="postgres.data-platform.svc.cluster.local",
    port=5432, dbname="dataplatform",
    user="postgres", password="postgres"
)

# MinIO / S3
import boto3
s3 = boto3.client("s3",
    endpoint_url="http://minio.data-platform.svc.cluster.local:9000",
    aws_access_key_id="minioadmin",
    aws_secret_access_key="minioadmin"
)
s3.upload_file("dataset.csv", "raw-data", "spam/dataset.csv")
```

---

## Cleanup

```bash
kubectl delete namespace data-platform
```

## Author
**kiranch97** — Built collaboratively with Claude AI | March 2026
