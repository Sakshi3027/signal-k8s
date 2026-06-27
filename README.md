# Signal K8s Edition

Production-grade Kubernetes deployment of the Signal Market Intelligence Platform on GKE.
Migrates the existing Signal agent (LangGraph, FastAPI, Qdrant, PostgreSQL) from Modal/Render
to Google Kubernetes Engine with full K8s orchestration.

## Architecture
Internet → Ingress (TLS) → signal-api-service → signal-api pods (HPA: 2-10)

↓

postgres-service    qdrant-service

↓                   ↓

postgres-pvc        qdrant-pvc

## Stack

- **Platform**: Google Kubernetes Engine (GKE)
- **API**: FastAPI on K8s Deployment with readiness probes
- **Autoscaling**: HorizontalPodAutoscaler (CPU 70%, Memory 80%, 2-10 pods)
- **Ingress**: NGINX Ingress with TLS via cert-manager
- **Storage**: PersistentVolumeClaims for PostgreSQL and Qdrant
- **Secrets**: K8s Secrets for database credentials and API keys
- **Packaging**: Helm chart for one-command deployment

## Deploy with Helm

```bash
# Install on GKE
helm install signal ./helm/signal-chart

# Scale up
helm upgrade signal ./helm/signal-chart --set api.replicas=5

# Override values
helm install signal ./helm/signal-chart \
  --set api.image=gcr.io/myproject/signal-api:v1.2 \
  --set ingress.host=signal.mydomain.com
```

## Manifests

| File | Description |
|---|---|
| `manifests/api/deployment.yaml` | FastAPI Deployment with resource limits |
| `manifests/api/service.yaml` | ClusterIP Service |
| `manifests/api/hpa.yaml` | HorizontalPodAutoscaler (2-10 pods) |
| `manifests/api/ingress.yaml` | NGINX Ingress with TLS |
| `manifests/postgres/` | PostgreSQL Deployment + PVC |
| `manifests/qdrant/` | Qdrant Deployment + PVC |
| `manifests/secrets.yaml` | K8s Secrets |
| `manifests/configmap.yaml` | Environment ConfigMap |
| `helm/signal-chart/` | Helm chart for full stack |

## Upgrade from Modal/Render

Signal was originally deployed on Modal (ingestion) and Render (API). This repo
migrates it to GKE with proper K8s primitives — HPA for autoscaling, PVCs for
persistent storage, and Ingress for TLS termination.
