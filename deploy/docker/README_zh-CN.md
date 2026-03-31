# LinkWork Docker 单机部署

[English](./README.md) | 中文

## 前置条件

- Docker >= 24.0
- Docker Compose V2
- 最低 4 CPU / 8GB RAM / 40GB 磁盘（最小集）
- 推荐 8 CPU / 16GB RAM / 100GB（含 Milvus）

## 快速开始

```bash
cd LinkWork/deploy/docker

# 1. 复制环境变量模板并编辑
cp .env.example .env
# 编辑 .env，填写 AUTH_PASSWORD、AUTH_JWT_SECRET、LITELLM_API_KEY 等

# 2. 放入 DDL 文件
# 将实际 DDL 替换 initdb/001-schema.sql

# 3. 启动最小集（6 容器: mysql + redis + dind + backend + gateway + web）
docker compose -f docker-compose.yml up -d

# 或使用开发模式（挂载宿主机 Docker Socket，自动加载 override）
docker compose up -d
```

启动后访问:
- 前端 UI: http://localhost:3003
- 后端 API: http://localhost:8081
- MCP Gateway: http://localhost:9080

## 启动集合（Profile 控制）

```bash
# 最小集（默认 6 容器）
docker compose -f docker-compose.yml up -d

# 加 Milvus 向量记忆（+3 容器: etcd + minio + milvus）
docker compose -f docker-compose.yml --profile memory up -d

# 加 LLM 网关（+1 容器: litellm）
docker compose -f docker-compose.yml --profile llm up -d

# 全部启动（10 容器）
docker compose -f docker-compose.yml --profile memory --profile llm up -d
```

## Docker Daemon 模式

### DinD 模式（默认，生产推荐）

仅使用 `docker-compose.yml`（不加载 override）:

```bash
docker compose -f docker-compose.yml up -d
```

Backend 通过 `tcp://dind:2376` 连接独立的 Docker daemon。

### Socket 挂载模式（开发快捷）

默认行为（自动加载 `docker-compose.override.yml`）:

```bash
docker compose up -d
```

Backend 直接使用宿主机 Docker Socket，dind 容器不启动。

## 常用命令

```bash
# 查看容器状态
docker compose ps

# 查看日志
docker compose logs -f linkwork-backend

# 仅重建某个服务
docker compose build linkwork-backend
docker compose up -d linkwork-backend

# 停止并删除（保留数据卷）
docker compose down

# 停止并删除（含数据卷，慎用）
docker compose down -v
```

## 镜像构建说明

| 镜像 | 构建方式 | 说明 |
|------|---------|------|
| `linkwork-backend` | 三阶段 Maven | linkwork-server install → backend package → JRE runtime |
| `linkwork-mcp-gateway` | 两阶段 Go | 已有 Dockerfile，Go build → alpine runtime |
| `linkwork-web` | 两阶段 Node | Vite build → Nginx 静态服务 |

构建上下文均为 `LinkWork/` 根目录（gateway 除外，使用自身目录）。
