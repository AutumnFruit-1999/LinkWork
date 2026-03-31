# LinkWork Kubernetes Cluster Deployment

English | [中文](./README_zh-CN.md)

Kustomize-based Kubernetes deployment with two environments: `dev` (Kind) and `prod` (Harbor + Volcano).

## Prerequisites

- Kubernetes >= 1.28
- kubectl >= 1.28 (with built-in Kustomize)
- Images are already built and available (via Kind load or Harbor push)

### Additional Requirements for Production

- Harbor image registry
- Volcano scheduler (GPU / batch scheduling scenarios)
- Ingress controller (e.g. nginx-ingress)
- TLS certificate Secret

## Deployment Commands

### Development (Kind + Local Images)

```bash
# Build images
docker build -t linkwork-backend:latest -f deploy/docker/backend/Dockerfile .
docker build -t linkwork-web:latest -f deploy/docker/web/Dockerfile .
docker build -t linkwork-mcp-gateway:latest linkwork-mcp-gateway/

# Load into Kind
kind load docker-image linkwork-backend:latest
kind load docker-image linkwork-web:latest
kind load docker-image linkwork-mcp-gateway:latest

# Deploy
kubectl apply -k deploy/k8s/overlays/dev

# Verify
kubectl -n linkwork-dev get pods -w
```

Access (NodePort):
- Web: http://localhost:30003
- Backend: http://localhost:30081
- Gateway: http://localhost:30080

### Production (Harbor + Volcano)

```bash
# 1) Create imagePullSecret
kubectl -n linkwork-prod create secret docker-registry linkwork-registry-secret \
  --docker-server=harbor.example.com \
  --docker-username=YOUR_USER \
  --docker-password=YOUR_PASS

# 2) Create TLS Secret
kubectl -n linkwork-prod create secret tls linkwork-tls \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key

# 3) Edit overlays/prod/patches/env-patch.yaml
#    Set IMAGE_REGISTRY to your Harbor registry

# 4) Deploy
kubectl apply -k deploy/k8s/overlays/prod

# 5) Verify
kubectl -n linkwork-prod get pods -w
```

Access (Ingress): https://linkwork.example.com

## Directory Structure

```text
k8s/
├── base/                    # Shared resources (environment-agnostic)
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── initdb-configmap.yaml
│   ├── mysql-statefulset.yaml
│   ├── redis-statefulset.yaml
│   ├── backend-deployment.yaml
│   ├── gateway-deployment.yaml
│   └── web-deployment.yaml
└── overlays/
    ├── dev/                 # Kind development environment
    │   ├── kustomization.yaml
    │   └── patches/
    └── prod/                # Production environment
        ├── kustomization.yaml
        └── patches/
```

## dev vs prod Differences

| Item | dev | prod |
|------|-----|------|
| namespace | linkwork-dev | linkwork-prod |
| replicas | 1 | 2-5 (HPA) |
| imagePullPolicy | Never | IfNotPresent |
| image source | Kind load | Harbor |
| Service type | NodePort | ClusterIP + Ingress |
| Volcano | not required | enabled |
| OSS/Memory | disabled | enabled |

## Troubleshooting Commands

```bash
# Render manifests without deploying
kubectl kustomize deploy/k8s/overlays/dev

# Tail backend logs
kubectl -n linkwork-dev logs -f deployment/linkwork-backend

# Check recent events
kubectl -n linkwork-dev get events --sort-by='.lastTimestamp'

# Client-side dry run
kubectl apply -k deploy/k8s/overlays/dev --dry-run=client
```
