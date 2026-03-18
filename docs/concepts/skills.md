# Skills System

Skills are **pluggable capability modules** for AI workers in LinkWork. Each Skill is an independent, declaratively defined capability unit that can be composed and installed across different roles as needed.

---

## Design Philosophy

Traditional Agent capabilities have blurry boundaries — what plugins are installed and what capabilities exist are only known at runtime, and the AI can install and modify them on its own.

LinkWork takes a different approach:

- Skills are **declaratively defined** — each Skill has a clear name, description, version, and dependencies
- Skills are **pinned at build time** — Skills are injected into the image during role building, read-only at runtime
- Skills are **admin-governed** — which roles can use which Skills is decided by administrators, not by the AI

> Install capabilities for AI workers like installing apps on a phone. Admins are the app store reviewers; AI workers are the users.

---

## Three-tier Capability Architecture

```
Role
  └── Skills (Capability Modules)
        └── MCP Tools (External Tool Access)
```

**Roles** define "who the AI worker is", **Skills** define "what the AI worker can do", **MCP Tools** define "what tools the AI worker can use". Three decoupled layers, freely composable, access-controlled.

---

## Skill Composition

A Skill consists of the following core elements:

| Element | Description |
|---------|-------------|
| Name & Description | Skill identifier and functionality description |
| Version Number | Semantic versioning (SemVer), supports rollback |
| Knowledge Content | Markdown knowledge files injected into the AI's context |
| Dependency Declaration | Other Skills or tools this Skill depends on |
| Scope | Tags and categories for marketplace discovery |

### Skill Knowledge Injection

The core value of Skills lies in **knowledge injection**. Each Skill is essentially a set of structured knowledge files (Markdown) that are automatically loaded into the AI's context during task execution:

```
skills/
├── code-review/
│   ├── SKILL.md           # Skill definition & metadata
│   ├── guidelines.md      # Code review guidelines
│   └── checklist.md       # Review checklist
└── data-analysis/
    ├── SKILL.md
    ├── methodology.md     # Analysis methodology
    └── templates.md       # Report templates
```

AI workers automatically receive the knowledge from all Skills configured for their role during task execution — no additional setup required.

---

## Skills Marketplace

LinkWork provides a built-in Skills marketplace supporting team sharing and reuse:

### Marketplace Features

| Feature | Description |
|---------|-------------|
| Publish | Publish personally validated Skills to the team marketplace |
| Discover | Search available Skills by category, tags, and popularity |
| Version Management | Independent version management per Skill, supports rollback |
| Usage Analytics | View call count and success rate for each Skill |
| Team Sharing | Set Skill visibility scope (personal / team / global) |

### Lifecycle

```
Write Skill → Personal validation → Publish to marketplace → Admin review → Team available → Continuous iteration
```

> Skills you've refined on your personal tools can go straight into LinkWork, becoming standardized capabilities your entire team can use.

---

## Build-time Pinning

Skills are injected into the container image during the role build phase:

1. Pull the corresponding version of Skills per role configuration
2. Pin the exact version number, written into the image
3. Read-only at runtime, cannot be modified

This means:
- All instances of the same role have identical Skills configurations
- Skills changes require rebuilding the image — no "some instances updated, some didn't" scenarios
- Build records are fully traceable, Skills configuration can be reproduced at any point in time

> See [Harness Engineering](./harness-engineering.md) for details.

---

## Common Skills Examples

| Skill | Description | Applicable Roles |
|-------|-------------|-----------------|
| Code Review | Code quality, security vulnerability, performance issue review | Development Engineer |
| Data Analysis | Data cleaning, statistical analysis, visualization reports | Data Analyst |
| Document Writing | Technical docs, API docs, user manuals | Technical Writer |
| Literature Tracking | Paper fetching, summary extraction, trend analysis | Research Assistant |
| Ops Inspection | Service health checks, log analysis, alert handling | Operations Engineer |

---

## Further Reading

- [MCP Tools](./mcp-tools.md) — Skills can be used together with MCP tools
- [Harness Engineering](./harness-engineering.md) — How Skills are baked into images at build time
- [Extension Guide](../guides/extension.md) — How to write custom Skills
