# Extension Guide

In LinkWork, each AI worker's capabilities are not determined solely by Prompts. From the container environment to runtime tools, the AI worker's capability stack is composed of multiple layers. This guide describes how to extend each layer.

---

## Capability Stack Overview

| Layer | Description | Extension Method |
|-------|-------------|-----------------|
| **Role** | Complete AI worker definition template | Create via management UI, configure responsibilities, Prompt, resource quotas |
| **Skills** | Pluggable knowledge & workflow modules | Write SKILL.md, upload to Skills marketplace |
| **MCP Tools** | Standardized external capability access | Register MCP service endpoints, authorize to roles |
| **File Management** | User file space & role workspace | Platform auto-mounts, isolated by user/role |
| **Git Projects** | Code repo cloning, operations, and delivery | Configure Git repos in role settings, credentials auto-injected at runtime |
| **Container Environment** | Toolchains and runtimes pre-installed in role images | Baked at build time (Python, Node.js, Java, Go, etc.) |
| **CLI Tools** | User-installed command-line tools | *Planned — future support for user-extensible role toolchains* |

---

## Creating Custom Roles

A Role is the complete definition of an AI worker, encompassing responsibilities, capabilities, and constraints.

### Role Configuration Elements

| Element | Description | Example |
|---------|-------------|---------|
| Name | Role identifier | "Frontend Developer" |
| Job Description | AI's role positioning and scope | "Responsible for frontend code review and bug fixes" |
| Prompt | Role-level system prompt | Includes tech stack preferences, output conventions, etc. |
| Skills | Equipped skill modules | code-review, git-workflow |
| MCP Tools | Authorized external tools | Tavily Search, GitLab, Database |
| Security Policy | Command execution boundaries | Allow git operations, deny rm -rf |
| Resource Quota | CPU, memory limits | 1 CPU, 2Gi memory |
| Model | LLM to use | Claude, Qwen, DeepSeek |

### Creation Process

1. **Create a role in the management UI**, fill in name and job description
2. **Write the role Prompt**, clarifying the AI's role positioning, work style, and output conventions
3. **Select Skills**, choose from the marketplace or create custom Skills
4. **Configure MCP tools**, authorize external tools the role can use
5. **Set security policies**, define allowed and prohibited operations
6. **Build the image**, Skills, MCP, and policies are automatically baked into the image
7. **Start and verify**, dispatch test tasks to validate role capabilities

### Role Design Best Practices

| Practice | Description |
|----------|-------------|
| Single Responsibility | Each role focuses on one type of work; avoid being a jack-of-all-trades |
| Concise Prompts | Clearly define role boundaries; avoid vague descriptions |
| Least Privilege | Only authorize necessary Skills and tools |
| Iterative Validation | Test on a small scale first, then roll out broadly |

---

## Writing Custom Skills

A Skill is a pluggable capability module for AI workers, essentially a set of structured knowledge files. During role image building, Skills are injected into the `/opt/agent/skills/` directory and synced to the working directory for loading during task execution.

### Skill Directory Structure

```
my-skill/
├── SKILL.md           # Skill definition file (required)
├── guidelines.md      # Knowledge file
├── checklist.md       # Knowledge file
└── templates/         # Template files (optional)
    └── report.md
```

### SKILL.md Format

`SKILL.md` is the Skill's entry definition file, containing metadata and core knowledge:

```markdown
---
name: code-review
version: 1.0.0
description: Code review skill providing code quality, security, and performance review capabilities
tags: [development, review]
---

# Code Review Skill

## Review Principles

(Your review knowledge and guidelines)

## Review Checklist

(Specific check items)
```

### Metadata Fields

| Field | Required | Description |
|-------|----------|-------------|
| name | Yes | Unique Skill identifier, kebab-case recommended |
| version | Yes | Semantic version number (SemVer) |
| description | Yes | Brief description of the Skill's functionality |
| tags | No | Tags for marketplace discovery |

### Knowledge Injection Mechanism

Markdown files within a Skill are automatically loaded into the AI's context during task execution. When writing Skill knowledge, keep in mind:

| Principle | Description |
|-----------|-------------|
| Structured | Organize knowledge using headings, lists, and tables |
| Actionable | Provide concrete steps and checklists |
| Bounded | Clearly define the Skill's scope and limitations |
| Right-sized | Control knowledge volume to avoid diluting attention with too much content |

Skills can also declare dependencies (Python/Node/Go packages), which are automatically checked and installed at build time. If a Skill depends on an MCP tool (e.g., Tavily Search), ensure the role has the corresponding MCP service bound.

### Publishing to the Skills Marketplace

1. Develop and validate the Skill locally
2. Upload Skill files through the management interface
3. Fill in version notes and changelog
4. Set visibility scope (personal / team / global)
5. Submit for review (if publishing globally)

---

## Registering MCP Tools

MCP (Model Context Protocol) tools provide external capabilities for AI workers.

### Supported Tool Types

| Type | Description | Use Cases |
|------|-------------|-----------|
| HTTP | Standard HTTP API services | REST APIs, database queries, third-party services |
| SSE | Server-Sent Events services | Streaming data sources, real-time monitoring |

### Registration Process

1. **Navigate to the MCP management page**
2. **Fill in tool information**:
   - Name and description
   - Service type (HTTP / SSE)
   - Service address
   - Auth configuration (e.g., API Key)
   - Health check endpoint (optional)
3. **Save and wait for health check to pass**
4. **Associate the tool with a role in role configuration**

### Tool Authentication

MCP tool auth credentials are managed centrally by the platform:

- Credentials are configured during registration and stored on the platform side
- When AI workers invoke tools, the gateway automatically injects auth information
- AI workers themselves never hold any external credentials

### Health Checks

After registration, the platform automatically performs periodic health checks:

- Online: Service available, response normal
- Degraded: Response latency high or intermittent failures
- Offline: Multiple consecutive probe failures

The role configuration page displays each tool's real-time status, helping admins confirm tool availability before dispatching tasks.

---

## File Management

Each AI worker has a structured file space, automatically mounted by the platform at task startup (`fs_prepare`) and cleaned up after task completion (`fs_cleanup`).

| Path | Purpose | Description |
|------|---------|-------------|
| `/workspace/current/{taskId}` | Current task working directory | Primary directory for task execution |
| `/workspace/doc/user` | User file space | Mounted to NFS, isolated by user, persistent across tasks |
| `/workspace/doc/job` | Role workspace | Mounted to NFS, isolated by role, persistent across tasks |
| `/workspace/logs` | Task logs | Structured log output |

User file space and role workspace are persisted across tasks via NFS mounts, allowing AI workers to accumulate and reuse knowledge across different tasks.

---

## Git Projects

AI workers support working in real code repositories. Git integration is divided into two phases:

### Build Time

Specify Git repositories to pre-clone in the role configuration; code is automatically pulled to the working directory during image building.

### Runtime

After task startup, the platform automatically injects Git credentials (GitLab PAT / OAuth Token):

- `git fetch`, `git pull`, `git push`, `git clone` operations automatically carry credentials
- `git commit` automatically injects user identity information
- Credentials exist only in memory, never written to disk

After task completion, auto commit/push and Merge Request creation are supported, feeding outputs directly into the standard Code Review workflow.

---

## Container Environment

Each role image is built on Rocky Linux 9, pre-installed with the following toolchains:

| Category | Tools |
|----------|-------|
| Language Runtimes | Python 3.12, Node.js 24, Java 21, Go 1.22 |
| Package Managers | uv/uvx (Python), npm (Node.js) |
| Version Control | git |
| Network Tools | curl, wget, jq |
| AI Tools | Claude CLI, @anthropic-ai/claude-code |

Skills can also declare additional dependencies, which are automatically installed into the image at build time. All tools are baked at build time, read-only at runtime, ensuring environment consistency.

---

## CLI Tools (Planned)

Currently, role toolchains are fixed at build time, and users cannot install additional CLI tools at runtime. Future plans include:

- Users declaring additional CLI tools in role configuration
- Automatic installation and baking into images at build time
- Support for custom installation scripts

---

## Three-tier Prompt Strategy

LinkWork's Prompt consists of three tiers that together define AI worker behavior:

| Tier | Source | Description |
|------|--------|-------------|
| Platform Prompt | Platform built-in | General security rules, output format conventions |
| Role Prompt | Role configuration | Role positioning, tech stack preferences, work style |
| User Soul | User configuration | Personal preferences, communication style, special requirements |

**Platform Prompt + Role Prompt + User Soul → Complete task context.**

### Role Prompt Writing Tips

```markdown
# You are [Role Name]

## Role Positioning
(Brief description of role and responsibilities)

## Work Style
(Output style, detail level, language preferences, etc.)

## Tech Stack
(Relevant technologies and tool preferences)

## Output Conventions
(Deliverable formats, naming conventions, etc.)
```

---

## Further Reading

- [Skills System](../concepts/skills.md) — Design philosophy of Skills
- [MCP Tools](../concepts/mcp-tools.md) — Design philosophy of MCP tools
- [Workstation Model](../concepts/workstation.md) — Complete role concept
- [Literature Tracker Example](../examples/literature-tracker.md) — A complete role configuration walkthrough
