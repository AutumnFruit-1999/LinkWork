# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Each component maintains its own version independently.

---

## [Unreleased]

### linkwork-server
- Initial open-source release of the core scheduling engine

### linkwork-executor
- Initial open-source release of the secure command execution sandbox

### linkwork-agent-sdk
- Initial open-source release of the Agent runtime with LLM orchestration and MCP integration

### linkwork-mcp-gateway
- Initial open-source release of the MCP tool proxy with auth and usage metering

### linkwork-web
- Initial open-source release of the frontend reference implementation

---

## [0.1.0] - 2026-03-26

### linkwork-server
#### Added
- Role (AI position) management API
- Skill marketplace with version management and hot-reload
- MCP tool registration and routing
- Task scheduling and real-time WebSocket status streaming
- Approval workflow for high-risk operations
- Cron-based shift scheduling
- Multi-model support via OpenAI-compatible interface
- Docker Compose quick-start launcher

[Unreleased]: https://github.com/momotech/LinkWork/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/momotech/LinkWork/releases/tag/v0.1.0
