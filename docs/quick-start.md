# Quick Start

This guide helps you understand LinkWork's components and launch the platform services locally for exploration.

---

## Please Note: This Is Not a "Plug and Play" Project

LinkWork is an enterprise-grade AI workforce platform that relies on a K8s cluster for container orchestration and scheduling. **You cannot run the full system by simply starting a few Docker containers.**

The system has two layers:

| Layer | Components | Setup Difficulty | Description |
|-------|-----------|-----------------|-------------|
| **Platform Services** | linkwork-server, linkwork-web | Low | Can be started via Docker Compose or local build |
| **AI Worker Runtime** | linkwork-executor, linkwork-agent-sdk, AI worker containers | High | Requires full infrastructure cluster |

Running AI workers requires the following infrastructure to be fully operational:

| Infrastructure | Version Required | Purpose |
|---------------|-----------------|---------|
| Kubernetes Cluster | v1.33+ | Container orchestration, resource isolation, elastic scheduling |
| Volcano Scheduler | v1.13.0+ | Gang Scheduling, atomic multi-container scheduling |
| Harbor Image Registry | — | Role image storage and distribution |
| MySQL | 8.0+ | Business data storage |
| Redis | 7+ | Task queues, caching, data bus |
| NFS Shared Storage | — | AI worker workspace, user file persistence |
| Docker | 24.0+ | Role image building |

> If you just want to explore the system architecture and interactions, you can start the platform services (server + web) for browsing. To have AI workers actually execute tasks, complete the infrastructure setup in the [Deployment Guide](./guides/deployment.md) first.

---

## Step 1: Clone the Repository

```bash
git clone https://github.com/momotech/LinkWork.git
cd LinkWork

# Pull all submodule code
git submodule update --init --recursive
```

Submodule structure:

| Submodule | Description |
|-----------|-------------|
| linkwork-server | Core backend — task scheduling, role management, approvals |
| linkwork-executor | Secure executor — in-container command execution, policy engine |
| linkwork-agent-sdk | Agent runtime — LLM engine, Skills orchestration |
| linkwork-mcp-gateway | MCP tool gateway — tool discovery, auth, usage metering |
| linkwork-web | Frontend reference — task dashboard, role configuration |

---

## Step 2: Configure Environment Variables

> Configuration items are not yet fully externalized into a unified config. This will be improved over time. For now, refer to the default configuration files within each component.

---

## Step 3: Start Platform Services (Dev Mode)

Platform services (backend + frontend) can be quickly started locally via Docker Compose:

```bash
docker compose up -d
```

Service ports after startup:

| Service | Port | Description |
|---------|------|-------------|
| linkwork-server | 8081 | Backend API |
| linkwork-web | 3003 | Frontend UI |

Visit `http://localhost:3003` to open the LinkWork management interface.

> Note: Only platform management features are available in this mode. Creating, scheduling, and executing AI workers requires full K8s infrastructure support.

---

## Step 4: Create Your First Role

1. Navigate to the "Role Management" page, click "Create Role"
2. Fill in the role information:
   - **Role Name**: e.g. "Code Reviewer"
   - **Job Description**: Tell the AI its role and scope of work
   - **Skills Configuration**: Select required skills from the Skills marketplace
   - **MCP Tools**: Authorize external tools the role can use
   - **Security Policy**: Set allow/deny rules for command execution
3. Save and build the role image

> Role building automatically handles Skills injection, MCP config baking, and security policy embedding. See [Harness Engineering](./concepts/harness-engineering.md) for details.

---

## Step 5: Dispatch a Task

1. Go to the "Task Center", select the role you just created
2. Enter a task description, e.g.: "Review code changes on the feature/user-auth branch, focusing on security vulnerabilities and performance issues"
3. Click "Dispatch Task"

After dispatching, you can:

- Watch the AI worker's reasoning process and actions in real time
- Track the complete execution log in the WebSocket streaming panel
- High-risk operations will automatically trigger approval requests that require your confirmation before proceeding

---

## Next Steps

- Read [Core Concepts](./concepts/workstation.md) to understand the workstation model in depth
- Read [Extension Guide](./guides/extension.md) to learn how to write custom Skills
- Check the [Literature Tracker Example](./examples/literature-tracker.md) for a complete role configuration walkthrough
- Read the [Deployment Guide](./guides/deployment.md) for production environment deployment
