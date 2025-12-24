# Agent Lightning ⚡ AI Agent 开发框架

## 变更记录 (Changelog)

- **2025-11-20**: 初始化架构文档，基于版本 0.2.2
- 完成核心模块扫描和架构分析
- 生成模块结构图和导航链接

## 项目愿景

Agent Lightning 是微软开发的 AI Agent 训练框架，旨在通过强化学习、自动提示优化和监督微调等算法来优化 AI 智能体。其核心特点包括：

- **零代码变更优化**: 几乎无需修改现有代码即可将智能体转化为可优化的训练对象
- **多框架兼容**: 支持 LangChain、OpenAI Agent SDK、AutoGen、CrewAI 等主流 Agent 框架
- **选择性优化**: 可对多智能体系统中的一个或多个智能体进行针对性优化
- **算法丰富**: 集成强化学习、自动提示优化、监督微调等多种训练算法

## 架构总览

Agent Lightning 采用分层架构设计，主要由以下核心组件构成：

```mermaid
graph TD
    A["Agent Lightning Framework"] --> B["Algorithm Layer"];
    A --> C["Trainer Layer"];
    A --> D["Execution Layer"];
    A --> E["Storage Layer"];
    A --> F["Observability Layer"];

    B --> B1["APO Algorithm"];
    B --> B2["VERL Integration"];
    B --> B3["Custom Algorithms"];

    C --> C1["Trainer"];
    C --> C2["Runner"];
    C --> C3["Strategy"];

    D --> D1["Client-Server"];
    D --> D2["Shared Memory"];
    D --> D3["Inter-Process"];

    E --> E1["In-Memory Store"];
    E --> E2["SQLite Store"];
    E --> E3["Custom Store"];

    F --> F1["Tracer"];
    F --> F2["Adapter"];
    F --> F3["Instrumentation"];
```

## ✨ 模块结构图

```mermaid
graph TD
    A["(根) agent-lightning"] --> B["agentlightning"];
    A --> C["examples"];
    A --> D["docs"];
    A --> E["tests"];

    B --> F["adapter"];
    B --> G["algorithm"];
    B --> H["cli"];
    B --> I["emitter"];
    B --> J["execution"];
    B --> K["instrumentation"];
    B --> L["litagent"];
    B --> M["runner"];
    B --> N["store"];
    B --> O["tracer"];
    B --> P["trainer"];
    B --> Q["types"];
    B --> R["verl"];

    G --> G1["apo"];
    G --> G2["verl"];

    click F "./agentlightning/adapter/CLAUDE.md" "查看 adapter 模块文档"
    click G "./agentlightning/algorithm/CLAUDE.md" "查看 algorithm 模块文档"
    click H "./agentlightning/cli/CLAUDE.md" "查看 cli 模块文档"
    click I "./agentlightning/emitter/CLAUDE.md" "查看 emitter 模块文档"
    click J "./agentlightning/execution/CLAUDE.md" "查看 execution 模块文档"
    click K "./agentlightning/instrumentation/CLAUDE.md" "查看 instrumentation 模块文档"
    click L "./agentlightning/litagent/CLAUDE.md" "查看 litagent 模块文档"
    click M "./agentlightning/runner/CLAUDE.md" "查看 runner 模块文档"
    click N "./agentlightning/store/CLAUDE.md" "查看 store 模块文档"
    click O "./agentlightning/tracer/CLAUDE.md" "查看 tracer 模块文档"
    click P "./agentlightning/trainer/CLAUDE.md" "查看 trainer 模块文档"
    click Q "./agentlightning/types/CLAUDE.md" "查看 types 模块文档"
    click R "./agentlightning/verl/CLAUDE.md" "查看 verl 模块文档"

    C --> C1["apo"];
    C --> C2["calc_x"];
    C --> C3["rag"];
    C --> C4["search_r1"];
    C --> C5["spider"];
    C --> C6["unsloth"];
    C --> C7["tinker"];

    click C1 "./examples/apo/CLAUDE.md" "查看 apo 示例文档"
    click C2 "./examples/calc_x/CLAUDE.md" "查看 calc_x 示例文档"
    click C3 "./examples/rag/CLAUDE.md" "查看 rag 示例文档"
    click C4 "./examples/search_r1/CLAUDE.md" "查看 search_r1 示例文档"
    click C5 "./examples/spider/CLAUDE.md" "查看 spider 示例文档"
    click C6 "./examples/unsloth/CLAUDE.md" "查看 unsloth 示例文档"
    click C7 "./examples/tinker/CLAUDE.md" "查看 tinker 示例文档"
```

## 模块索引

| 模块路径 | 职责描述 | 主要组件 | 状态 |
|---------|----------|----------|------|
| `agentlightning/adapter` | 适配器层，处理不同框架和协议的数据转换 | TraceAdapter, TracerTraceToTriplet | ✅ 核心模块 |
| `agentlightning/algorithm` | 算法实现，包含训练和优化算法 | Algorithm, APO, VERL | ✅ 核心模块 |
| `agentlightning/cli` | 命令行工具，提供各种管理接口 | agl 命令, vllm, store | ✅ 用户接口 |
| `agentlightning/emitter` | 事件发射器，处理追踪和奖励事件 | MessageEmitter, RewardEmitter | ✅ 观测组件 |
| `agentlightning/execution` | 执行策略，管理进程和资源分配 | ExecutionStrategy, ClientServer | ✅ 运行时核心 |
| `agentlightning/instrumentation` | 集成工具，支持第三方监控工具 | AgentOps, LiteLLM, VLLM | ✅ 集成组件 |
| `agentlightning/litagent` | 轻量级智能体基类，定义智能体接口 | LitAgent | ✅ 开发接口 |
| `agentlightning/runner` | 运行器，负责执行智能体任务 | Runner, LitAgentRunner | ✅ 执行组件 |
| `agentlightning/store` | 存储层，管理任务、资源和追踪数据 | LightningStore, SQLite, Memory | ✅ 数据核心 |
| `agentlightning/tracer` | 追踪器，记录执行轨迹和性能指标 | Tracer, AgentOpsTracer, OpenTelemetry | ✅ 观测核心 |
| `agentlightning/trainer` | 训练器，协调算法、运行器和存储 | Trainer, TrainerLegacy | ✅ 编排核心 |
| `agentlightning/types` | 类型定义，核心数据模型和协议 | Span, Rollout, Attempt, Hook | ✅ 类型系统 |
| `agentlightning/verl` | VERL 集成，支持大规模强化学习 | VERL Interface, Dataset, Daemon | ✅ 算法扩展 |

## 运行与开发

### 安装依赖

项目使用 UV 作为包管理器，支持多种依赖组合：

```bash
# 基础安装
pip install agentlightning

# 开发环境
uv sync --group dev

# 完整训练环境（包含 PyTorch）
uv sync --group torch-stable

# 特定 Agent 框架支持
uv sync --group autogen  # AutoGen 集成
uv sync --group langchain  # LangChain 集成
```

### 快速开始

1. **创建智能体**：继承 `LitAgent` 类并实现 `rollout` 方法
2. **选择算法**：使用内置的 APO 或 VERL 算法
3. **配置训练器**：设置执行策略和存储后端
4. **启动训练**：调用 `trainer.fit()` 开始训练循环

### 开发工具

- **CLI 工具**：使用 `agl` 命令管理存储服务器和 vLLM 集成
- **调试支持**：内置 OpenTelemetry 追踪和 AgentOps 集成
- **并行执行**：支持多进程和多线程执行策略

## 测试策略

### 测试结构

```
tests/
├── unit/          # 单元测试
├── integration/   # 集成测试
├── examples/      # 示例测试
└── e2e/          # 端到端测试
```

### 质量保证

- **代码格式**：使用 Black 和 isort 进行代码格式化
- **类型检查**：Pyright 静态类型分析
- **预提交**：pre-commit hooks 自动化质量检查
- **CI/CD**：GitHub Actions 多平台自动化测试

### 示例验证

每个示例都有对应的 CI 工作流验证：
- `apo` 示例：自动提示优化教程
- `calc_x` 示例：数学推理智能体训练
- `spider` 示例：Text-to-SQL 强化学习训练
- `unsloth` 示例：监督微调与 4-bit 量化

## 编码规范

### 代码风格

- **Python 版本**：要求 Python 3.10+
- **行长度**：120 字符（Black 配置）
- **导入顺序**：isort 配置，遵循 Black 规范
- **类型注解**：使用现代类型注解语法

### 架构原则

1. **模块化设计**：每个模块职责单一，接口清晰
2. **异步优先**：核心 API 同时支持同步和异步调用
3. **可扩展性**：通过注册机制支持插件和自定义实现
4. **向后兼容**：保持 API 稳定性，提供迁移指南

### 文档要求

- **API 文档**：使用 mkdocstrings 自动生成
- **类型提示**：完整的类型注解
- **示例代码**：每个主要功能都有对应示例
- **变更日志**：维护详细的版本变更记录

## AI 使用指引

### 项目上下文

Agent Lightning 是一个 AI Agent 训练框架，主要特点：
- 支持零代码变更的智能体优化
- 多 Agent 框架兼容性
- 丰富的训练算法集成
- 完整的可观测性支持

### 关键模式

1. **适配器模式**：`agentlightning/adapter` 处理不同框架的数据转换
2. **策略模式**：`agentlightning/execution` 支持多种执行策略
3. **观察者模式**：`agentlightning/emitter` 和 `agentlightning/tracer` 实现事件追踪
4. **工厂模式**：`agentlightning/trainer` 中的组件构建机制

### 常见任务

- **添加新算法**：继承 `Algorithm` 基类，实现 `run` 方法
- **集成新框架**：创建新的 `Adapter` 实现
- **自定义存储**：实现 `LightningStore` 接口
- **扩展追踪**：添加新的 `Tracer` 或 `Instrumentation`

### 调试技巧

- 使用 `Trainer.dev()` 进行快速开发测试
- 启用 AgentOps 集成查看详细执行轨迹
- 使用 OpenTelemetry 进行分布式追踪
- 检查 `LightningStore` 中的任务状态和 Span 数据

## 相关资源

- **官方文档**：[https://microsoft.github.io/agent-lightning/](https://microsoft.github.io/agent-lightning/)
- **GitHub 仓库**：[https://github.com/microsoft/agent-lightning](https://github.com/microsoft/agent-lightning)
- **社区 Discord**：[加入讨论](https://discord.gg/RYk7CdvDR7)
- **学术论文**：[Agent Lightning: Train ANY AI Agents with Reinforcement Learning](https://arxiv.org/abs/2508.03680)

---

*最后更新：2025-11-20 | 版本：0.2.2*