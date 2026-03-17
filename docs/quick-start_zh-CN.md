# 快速开始

本指南帮助你了解 LinkWork 的组成结构，并在本地启动平台服务进行体验。

---

## 请先了解：这不是一个"开箱即用"的项目

LinkWork 是一个企业级 AI 劳动力平台，核心依赖 K8s 集群进行容器编排与调度。**你无法通过简单启动几个 Docker 容器来运行完整系统。**

系统分为两个层面：

| 层面 | 组件 | 启动难度 | 说明 |
|------|------|---------|------|
| **平台服务** | linkwork-server、linkwork-web | 较低 | 可通过 Docker Compose 或本地构建启动 |
| **AI 员工运行时** | linkwork-executor、linkwork-agent-sdk、AI 员工容器 | 较高 | 依赖完整的基础设施集群 |

AI 员工真正运行需要以下基础设施全部就位：

| 基础设施 | 版本要求 | 用途 |
|----------|---------|------|
| Kubernetes 集群 | v1.33+ | 容器编排、资源隔离、弹性调度 |
| Volcano 调度器 | v1.13.0+ | Gang Scheduling，多容器原子化调度 |
| Harbor 镜像仓库 | — | 岗位镜像存储与分发 |
| MySQL | 8.0+ | 业务数据存储 |
| Redis | 7+ | 任务队列、缓存、数据总线 |
| NFS 共享存储 | — | AI 员工工作空间、用户文件持久化 |
| Docker | 24.0+ | 岗位镜像构建 |

> 如果你只是想了解系统架构和交互方式，可以先启动平台服务（server + web）进行浏览。如果要让 AI 员工真正执行任务，请先完成 [部署指南](./guides/deployment_zh-CN.md) 中的基础设施准备。

---

## 第一步：获取代码

```bash
git clone https://github.com/momotech/LinkWork.git
cd LinkWork

# 拉取所有子项目代码
git submodule update --init --recursive
```

子项目结构：

| 子项目 | 说明 |
|--------|------|
| linkwork-server | 核心后端 — 任务调度、岗位管理、审批 |
| linkwork-executor | 安全执行器 — 容器内命令执行、策略引擎 |
| linkwork-agent-sdk | Agent 运行时 — LLM 引擎、Skills 编排 |
| linkwork-mcp-gateway | MCP 工具网关 — 工具发现、鉴权、用量统计 |
| linkwork-web | 前端参考实现 — 任务面板、岗位配置 |

---

## 第二步：配置环境变量

> 当前各组件的配置项尚未完全抽离为统一的外部配置，此部分后续会持续完善。目前请参考各组件内的默认配置文件。

---

## 第三步：启动平台服务（开发模式）

平台服务（后端 + 前端）可通过 Docker Compose 在本地快速启动：

```bash
docker compose up -d
```

启动后的服务端口：

| 服务 | 端口 | 说明 |
|------|------|------|
| linkwork-server | 8081 | 后端 API |
| linkwork-web | 3003 | 前端界面 |

访问 `http://localhost:3003` 可打开 LinkWork 管理界面。

> 注意：此模式下只有平台管理功能可用。AI 员工的创建、调度和执行需要完整的 K8s 基础设施支持。

---

## 第四步：创建你的第一个岗位

1. 进入「岗位管理」页面，点击「创建岗位」
2. 填写岗位信息：
   - **岗位名称**：例如「代码审查员」
   - **职责描述**：告诉 AI 它的角色定位和工作范围
   - **Skills 配置**：从 Skills 市场选择需要的技能
   - **MCP 工具**：授权岗位可使用的外部工具
   - **安全策略**：设定命令执行的允许/禁止规则
3. 保存并构建岗位镜像

> 岗位构建会自动完成 Skills 注入、MCP 配置固化和安全策略内嵌。详见 [Harness Engineering](./concepts/harness-engineering_zh-CN.md)。

---

## 第五步：下发任务

1. 进入「任务中心」，选择刚创建的岗位
2. 输入任务描述，例如："审查 feature/user-auth 分支的代码变更，关注安全漏洞和性能问题"
3. 点击「下发任务」

任务下发后，你可以：

- 实时查看 AI 员工的思考过程和执行动作
- 在 WebSocket 流式面板中追踪完整执行日志
- 高风险操作会自动弹出审批请求，你确认后才继续执行

---

## 下一步

- 阅读 [核心概念](./concepts/workstation_zh-CN.md) 深入理解岗位模型
- 阅读 [扩展开发](./guides/extension_zh-CN.md) 学习如何编写自定义 Skill
- 查看 [文献追踪员示例](./examples/literature-tracker_zh-CN.md) 了解完整的岗位配置案例
- 阅读 [部署指南](./guides/deployment_zh-CN.md) 了解生产环境部署
