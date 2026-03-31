# LinkWork Docker Single-Node Deployment

English | [中文](./README_zh-CN.md)

## Prerequisites

- Docker >= 24.0
- Docker Compose V2
- Minimum: 4 CPU / 8 GB RAM / 40 GB disk (minimal stack)
- Recommended: 8 CPU / 16 GB RAM / 100 GB disk (with Milvus)

## Quick Start

```bash
cd LinkWork/deploy/docker

# 1) Copy env template and edit values
cp .env.example .env
# Edit .env and set AUTH_PASSWORD, AUTH_JWT_SECRET, LITELLM_API_KEY, etc.

# 2) Put your DDL file in place
# Replace initdb/001-schema.sql with your actual schema file

# 3) Start minimal stack (6 containers: mysql + redis + dind + backend + gateway + web)
docker compose -f docker-compose.yml up -d

# Or use development mode (mount host Docker socket, auto-load override)
docker compose up -d
```

After startup:
- Web UI: http://localhost:3003
- Backend API: http://localhost:8081
- MCP Gateway: http://localhost:9080

## Startup Profiles

```bash
# Minimal stack (default, 6 containers)
docker compose -f docker-compose.yml up -d

# Enable vector memory (+3 containers: etcd + minio + milvus)
docker compose -f docker-compose.yml --profile memory up -d

# Enable LLM gateway (+1 container: litellm)
docker compose -f docker-compose.yml --profile llm up -d

# Start everything (10 containers)
docker compose -f docker-compose.yml --profile memory --profile llm up -d
```

## Docker Daemon Modes

### DinD Mode (Default, Recommended for Production)

Use only `docker-compose.yml` (do not load override):

```bash
docker compose -f docker-compose.yml up -d
```

Backend connects to an isolated Docker daemon via `tcp://dind:2376`.

### Socket-Mount Mode (Fast for Local Development)

Default behavior (auto-loads `docker-compose.override.yml`):

```bash
docker compose up -d
```

Backend uses the host Docker socket directly, and the dind container is not started.

## Common Commands

```bash
# Check container status
docker compose ps

# Tail logs
docker compose logs -f linkwork-backend

# Rebuild one service only
docker compose build linkwork-backend
docker compose up -d linkwork-backend

# Stop and remove containers (keep volumes)
docker compose down

# Stop and remove containers + volumes (destructive)
docker compose down -v
```

## Image Build Notes

| Image | Build Method | Notes |
|------|--------------|-------|
| `linkwork-backend` | 3-stage Maven build | linkwork-server install -> backend package -> JRE runtime |
| `linkwork-mcp-gateway` | 2-stage Go build | existing Dockerfile, Go build -> alpine runtime |
| `linkwork-web` | 2-stage Node build | Vite build -> Nginx static serving |

Build context is `LinkWork/` repo root for all services except gateway (gateway uses its own directory as context).
