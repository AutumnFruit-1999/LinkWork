# Workstation Model

LinkWork's core management unit is the **Workstation (Role)**. A workstation defines the complete capability profile of an AI worker — including responsibilities, skills, tool permissions, and security policies.

---

## Core Idea

Traditional AI Agents are processes running on the host machine with no boundaries and no constraints. LinkWork takes a fundamentally different approach:

**Every AI worker is a containerized service.**

Each AI worker runs in an independent Docker / K8s container with a fully isolated execution environment. Manage your AI team like a microservice cluster, fully leveraging the cloud-native ecosystem.

---

## Three-tier Model

```
Workstation (Role)
  └── Instance
        └── Task
```

### Workstation — Role Definition

A workstation is an **immutable capability template** that defines "who the AI worker is" and "what it can do":

| Attribute | Description |
|-----------|-------------|
| Name & Responsibilities | Role positioning and job description |
| Skills List | Skill modules equipped for this role |
| MCP Tools | External tools this role can use |
| Security Policy | Allow/deny/approval rules for command execution |
| Resource Quota | CPU, memory, and disk quotas |
| Model Config | LLM model and parameters to use |

> Create a "Frontend Developer" role, and any AI model instance can start working immediately. Swap the model without changing the role; swap the worker without losing capabilities.

### Instance — Role Instance

A role can run multiple instances simultaneously, each being an independent container:

- Instances are fully isolated from each other (filesystem, network, processes)
- Each instance has its own workspace and execution context
- Instance count can be scaled on demand

### Task

A task is a specific work instruction dispatched by a user to an AI worker:

- Each task executes within one instance
- Tasks have a complete lifecycle: Queued → Executing → Completed/Failed
- The entire execution process is observable, supporting real-time tracking via WebSocket

---

## Containerized Execution

Containerized execution of AI workers provides the following guarantees:

### Isolation

- **Filesystem Isolation** — Each worker has an independent filesystem, invisible to others
- **Network Isolation** — External network closed by default, only necessary services whitelisted on demand
- **Process Isolation** — AI reasoning process and command execution process run separately

### Resource Governance

- **Per-role Quotas** — CPU and memory allocated on demand, preventing any single worker from crashing the cluster
- **Elastic Scaling** — Auto scale up/down based on task volume, auto-release resources when idle
- **Self-healing** — Auto-restart on container crash, stale working directories auto-cleaned

### Persistence

- **Workspace Persistence** — Task outputs and intermediate state preserved across sessions
- **Long-term Memory** — Knowledge accumulation and vector memory persist across tasks

---

## Role Lifecycle

```
Create Role → Configure Skills/MCP/Policies → Build Image → Start Instances → Receive Tasks → Execute → Idle Reclamation
```

1. **Create**: Admin defines role name, responsibilities, and capability configuration
2. **Build**: Scheduling engine auto-builds the role image (Skills injection, MCP baking, policy embedding)
3. **Start**: Create container instances in the K8s cluster
4. **Run**: Instances consume tasks from the queue and execute them
5. **Reclaim**: Auto-release resources after idle timeout

> Configuration changes require rebuilding the image. This is a deliberate design choice — see [Harness Engineering](./harness-engineering.md).

---

## How It Differs from Traditional Agents

| Dimension | Traditional Agent | LinkWork Workstation |
|-----------|------------------|---------------------|
| Runtime | Host machine process | Independent container |
| Capability Management | Install at will during runtime | Pinned at build time, admin-approved |
| Isolation | None | Full filesystem/network/process isolation |
| Resource Control | None | K8s-native resource quotas |
| Reusability | Personal config, hard to replicate | Role templates, one-click duplication |
| Security Boundary | Relies on user discretion | Policy engine + approval workflow |

---

## Further Reading

- [Skills System](./skills.md) — Learn how to equip roles with skills
- [MCP Tools](./mcp-tools.md) — Learn how to authorize external tools for roles
- [Harness Engineering](./harness-engineering.md) — Understand the role image build mechanism
- [Architecture Overview](../architecture/overview.md) — Understand where roles fit in the system
