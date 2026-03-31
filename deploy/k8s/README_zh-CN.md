# LinkWork K8s 集群部署

[English](./README.md) | 中文

基于 Kustomize 的 K8s 部署方案，支持 dev（Kind）和 prod（Harbor + Volcano）两种环境。

## 前置条件

- Kubernetes >= 1.28
- kubectl >= 1.28（内置 kustomize）
- 镜像已构建并可用（Kind load 或 Harbor push）

### 生产环境额外要求

- Harbor 镜像仓库
- Volcano scheduler（GPU/批调度场景）
- Ingress Controller（如 nginx-ingress）
- TLS 证书 Secret

## 部署命令

### 开发环境（Kind + 本地镜像）

```bash
# 构建镜像
docker build -t linkwork-backend:latest -f deploy/docker/backend/Dockerfile .
docker build -t linkwork-web:latest -f deploy/docker/web/Dockerfile .
docker build -t linkwork-mcp-gateway:latest linkwork-mcp-gateway/

# 导入 Kind
kind load docker-image linkwork-backend:latest
kind load docker-image linkwork-web:latest
kind load docker-image linkwork-mcp-gateway:latest

# 部署
kubectl apply -k deploy/k8s/overlays/dev

# 验证
kubectl -n linkwork-dev get pods -w
```

访问（NodePort）:
- 前端: http://localhost:30003
- 后端: http://localhost:30081
- 网关: http://localhost:30080

### 生产环境（Harbor + Volcano）

```bash
# 1. 创建 imagePullSecret
kubectl -n linkwork-prod create secret docker-registry linkwork-registry-secret \
  --docker-server=harbor.example.com \
  --docker-username=YOUR_USER \
  --docker-password=YOUR_PASS

# 2. 创建 TLS Secret
kubectl -n linkwork-prod create secret tls linkwork-tls \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key

# 3. 编辑 overlays/prod/patches/env-patch.yaml
#    修改 IMAGE_REGISTRY 为实际 Harbor 地址

# 4. 部署
kubectl apply -k deploy/k8s/overlays/prod

# 5. 验证
kubectl -n linkwork-prod get pods -w
```

访问（Ingress）: https://linkwork.example.com

## 目录结构

```
k8s/
├── base/                    # 公共资源（环境无关）
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
    ├── dev/                 # Kind 开发环境
    │   ├── kustomization.yaml
    │   └── patches/
    └── prod/                # 生产环境
        ├── kustomization.yaml
        └── patches/
```

## dev vs prod 差异

| 配置项 | dev | prod |
|--------|-----|------|
| namespace | linkwork-dev | linkwork-prod |
| replicas | 1 | 2-5 (HPA) |
| imagePullPolicy | Never | IfNotPresent |
| 镜像来源 | Kind load | Harbor |
| Service 类型 | NodePort | ClusterIP + Ingress |
| Volcano | 不依赖 | 启用 |
| OSS/Memory | 关闭 | 开启 |

## 常用排查命令

```bash
# 渲染清单（不部署）
kubectl kustomize deploy/k8s/overlays/dev

# 查看 Pod 日志
kubectl -n linkwork-dev logs -f deployment/linkwork-backend

# 查看事件
kubectl -n linkwork-dev get events --sort-by='.lastTimestamp'

# Dry-run 验证
kubectl apply -k deploy/k8s/overlays/dev --dry-run=client
```
