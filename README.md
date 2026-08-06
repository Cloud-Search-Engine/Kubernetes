# CloudSearch Kubernetes

Kustomize manifests for **API**, **ingestion workers**, and **frontend** on Kubernetes (local Kind/minikube or Amazon EKS).

Infrastructure (VPC, EKS, RDS, Redis, S3, SQS) is owned by the **Terraform** repo.

## What’s in this repo

```
base/                 Shared manifests (namespace: cloudsearch)
  api/                Deployment, Service, HPA
  ingestion/          Deployment, health Service, HPA
  frontend/           Deployment, Service, HPA
overlays/
  local/              Low replicas, :local image tags
  production/         Higher resources/HPA, Ingress
```

Images (override in overlays as needed):

- `ghcr.io/kundanmergu/cloudsearch-api:latest`
- `ghcr.io/kundanmergu/cloudsearch-ingestion:latest`
- `ghcr.io/kundanmergu/cloudsearch-frontend:latest`

## Prerequisites

- `kubectl`
- A cluster (Kind, minikube, or EKS from Terraform)
- Container images built/pushed from Backend, Ingestion, Frontend
- Secret `cloudsearch-secrets` in namespace `cloudsearch`

## How to start (local overlay)

```bash
# Preview
kubectl kustomize overlays/local

# Create secret first (example)
kubectl create namespace cloudsearch --dry-run=client -o yaml | kubectl apply -f -
kubectl -n cloudsearch create secret generic cloudsearch-secrets \
  --from-literal=DATABASE_URL='postgres://...' \
  --from-literal=REDIS_URL='redis://...' \
  --from-literal=SQS_QUEUE_URL='...' \
  --from-literal=S3_BUCKET='cloudsearch-docs'

kubectl apply -k overlays/local
kubectl -n cloudsearch get pods
```

## How to start (production / EKS)

```bash
# After terraform apply + Database migrations + images pushed
kubectl apply -k overlays/production
```

## Secret keys expected

| Key | Used by |
| --- | --- |
| `DATABASE_URL` | API, ingestion |
| `REDIS_URL` | API, ingestion |
| `SQS_QUEUE_URL` | Ingestion |
| `S3_BUCKET` | Ingestion |

Non-secret config comes from ConfigMap `cloudsearch-config` via `envFrom`.

## Local docker-compose note

Day-to-day laptop development uses the parent folder’s `docker-compose.yml` (not this repo). Use Kubernetes when targeting a real cluster.
