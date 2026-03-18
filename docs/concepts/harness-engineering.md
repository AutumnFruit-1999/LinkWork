# Harness Engineering

AI Agent success isn't just about model capability — **execution environment determinism is equally decisive**.

LinkWork adopts a **"One Role, One Image"** paradigm: Skills, MCP tools, and security policies are all baked into the container image at build time. Runtime is read-only — no drift, no surprises.

---

## Why Harness Engineering

Traditional Agent frameworks typically install plugins, fetch configurations, and load tools dynamically at runtime. This creates a series of problems:

| Problem | Description |
|---------|-------------|
| Environment Drift | Instances started at different times may have inconsistent configurations |
| Non-reproducibility | What worked yesterday may fail today, with no way to pinpoint the cause |
| Version Chaos | Which instance uses which version of which plugin — impossible to trace |
| Security Risk | AI may install uncontrolled tools at runtime |

The core principle of Harness Engineering: **eliminate all uncertainty at build time**.

---

## Build-time Immutability

Each role build triggers a complete assembly pipeline executed by the scheduling engine:

### 1. Skills Injection

Pull the corresponding version of Skills per role configuration, pin exact version numbers into the image.

```
Role config: skills = [code-review@1.2.0, git-workflow@2.0.1]
    ↓
Build engine pulls Skills files
    ↓
Write to image: /opt/agent/skills/code-review/  (v1.2.0)
                 /opt/agent/skills/git-workflow/  (v2.0.1)
```

### 2. MCP Config Baking

Generate MCP configuration descriptor files based on the role's associated MCP tools, written read-only into the image.

### 3. Security Policy Embedding

Security policy files are packaged into the image and auto-loaded at startup. Policies define the command execution boundaries for the role's AI workers.

### 4. Version Snapshot Recording

Record the exact version of every Skill and MCP tool — builds are fully reproducible.

---

## Runtime Read-only

After image building is complete, the runtime capability configuration is **read-only**:

- AI workers cannot install new Skills on their own
- AI workers cannot modify MCP tool configurations
- AI workers cannot bypass security policies

**Configuration change → must rebuild the image.**

This is a deliberate design choice: ensuring every running AI worker's environment is fully predictable and reproducible.

---

## Context Priming

Image immutability addresses the determinism of "capabilities"; context priming addresses the determinism of "task startup".

At task startup, the runtime automatically completes execution context assembly:

| Step | Description |
|------|-------------|
| Skills Sync | Sync pre-installed Skills from the image to the working directory, loaded via standard paths |
| Git Repo Preparation | Auto-pull code to the task branch per configuration |
| Three-tier Prompt Strategy | Platform Prompt + Role Prompt + User Soul, building complete task background |

AI workers don't start from zero — they **arrive on the job with full environment and context**.

---

## Fail-fast Build Strategy

The build phase employs a strict fail-fast strategy:

- Skills configured but Git clone fails → **abort build**
- MCP configured but config generation fails → **abort build**
- Security policy file format error → **abort build**

**Never silently skipped.** Problems surface at build time, not when an AI worker runs with broken capabilities.

---

## Summary

> One Role, One Image: environment as code, versions pinned, builds reproducible, failures exposed early.

| Principle | Description |
|-----------|-------------|
| Environment as Code | Role capability configuration is fully defined by the build, version-managed like code |
| Version Pinning | Every Skill and MCP tool is pinned to an exact version |
| Reproducible Builds | Same configuration inputs always produce the same image |
| Early Failure Exposure | Build-phase fail-fast prevents problems from reaching runtime |

---

## Further Reading

- [Workstation Model](./workstation.md) — Complete role definition and lifecycle
- [Skills System](./skills.md) — Declarative Skill definitions and version management
- [Deployment Guide](../guides/deployment.md) — Practical guide for image building and deployment
