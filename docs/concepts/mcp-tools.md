# MCP Tools

LinkWork provides AI workers with external capabilities through the **MCP Tool Bus**. Compatible with the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) standard, all external tools are accessed through a unified proxy layer that automatically handles auth, routing, and usage metering.

---

## Design Philosophy

To accomplish real work, AI workers need more than just LLM reasoning — they need to query databases, access APIs, operate filesystems, control browsers, and more.

LinkWork chose MCP as the tool integration standard:

- **Protocol Standardization** — Follows the MCP open standard with good ecosystem compatibility
- **Unified Proxy** — All tool calls go through the gateway proxy, no direct connections to external services
- **Auth Governance** — Tool-level permission control, admins decide who can use which tools
- **Observable Usage** — Every tool call is recorded, supporting metering and auditing

---

## Three-tier Tool Architecture

```
linkwork-mcp-gateway (Tool Gateway)
  ├── Auth & Routing
  ├── Health Checks
  └── Usage Metering
        │
        ▼
  MCP Tool Services
  ├── Database Query Tool
  ├── API Call Tool
  ├── File Operation Tool
  ├── Browser Control Tool
  └── ...more tools
```

### linkwork-mcp-gateway

The MCP tool gateway is the sole entry point for all tool calls, responsible for:

| Function | Description |
|----------|-------------|
| Tool Registration & Discovery | Manage all registered MCP tool services |
| Auth Proxy | Control tool access by role permissions |
| Health Checks | Periodic probing, auto-offline unhealthy tool services |
| Usage Metering | Record call duration, results, and costs for each tool invocation |
| Rate Limiting | Per-service QPS limits to prevent abuse |

---

## Tool Types

LinkWork currently supports the following types of MCP tool services:

| Type | Description | Examples |
|------|-------------|---------|
| HTTP | Remote HTTP API services | Jira, GitLab, database queries |
| SSE | Remote Server-Sent Events services | Real-time monitoring, streaming data sources |

> More types (e.g., stdio local processes) will be supported in future releases.

---

## Tool Lifecycle

### Registration

Admins register MCP tool services through the management interface, providing:

- Service name and description
- Service type (HTTP / SSE)
- Service address
- Auth configuration (API Key, Token, etc.)
- Health check endpoint (optional)

### Health Checks

After registration, the platform automatically performs periodic health checks:

| Status | Meaning |
|--------|---------|
| Online | Recent probe successful, response normal |
| Degraded | Response latency high or intermittent failures |
| Offline | Multiple consecutive probe failures |

### Role Association

Assign tools to specific roles:

- A role can be associated with multiple MCP tools
- Tool configuration is baked into the image during role building
- AI workers can only use authorized tools at runtime

---

## MCP Factory

The MCP Factory is part of the AI supply chain, providing unified tool management and governance:

- **Tool Registration & Discovery** — Unified tool catalog, searchable by category and tags
- **Version Management** — Tool service version tracking
- **Health Monitoring** — Real-time awareness of all tool service availability
- **Usage Analytics** — Call volume, latency, error rate, and other statistics

---

## Security Considerations

MCP tool integration follows LinkWork's security principles:

- **Least Privilege** — Each role can only access authorized tools
- **Credential Isolation** — Tool auth credentials are managed by the platform, invisible to AI workers
- **Network Governance** — Tool calls go through the gateway proxy, AI workers cannot directly connect to external services
- **Audit Trail** — Every tool call has a complete audit record

---

## Further Reading

- [Skills System](./skills.md) — Collaboration between Skills and MCP tools
- [Security Architecture](../architecture/security.md) — Security guarantees for tool calls
- [Extension Guide](../guides/extension.md) — How to register custom MCP tools
