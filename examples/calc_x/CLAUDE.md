[根目录](../../CLAUDE.md) > **calc_x**

# Calc-X 示例

## 项目职责

Calc-X 是一个数学推理智能体训练示例，展示了如何使用 Agent Lightning 框架训练基于 AutoGen 和 MCP 计算器工具的智能体。该示例针对数学问题求解任务，使用强化学习进行优化。

## 入口与启动

### 核心文件结构
```
examples/calc_x/
├── calc_agent.py              # 主要智能体定义
├── eval_utils.py             # 评估工具函数
├── train_calc_agent.py       # 训练脚本
├── legacy_calc_agent.py      # 旧版本智能体（兼容性）
├── legacy_calc_agent_debug.py # 调试版本
└── tests/                    # 测试目录
    ├── test_agentops.py      # AgentOps 集成测试
    └── test_mcp_calculator.py # MCP 计算器测试
```

### 运行方式
```bash
# 设置环境变量
export OPENAI_API_KEY=your_api_key
export OPENAI_BASE_URL=your_base_url

# 运行训练
python train_calc_agent.py
```

## 智能体定义

### MathProblem 数据结构
```python
class MathProblem(TypedDict):
    id: str              # 问题唯一标识
    question: str        # 数学问题描述
    chain: str          # 详细解题步骤（训练用）
    result: str         # 真实答案（评估用）
    source: str         # 数据来源
```

### AutoGen 集成智能体
使用 AutoGen 的 AssistantAgent 结合 MCP 计算器工具：

```python
def autogen_assistant_agent(
    model: str,
    openai_base_url: str,
    temperature: float,
    workbench: McpWorkbench
) -> AssistantAgent:
    # 配置 OpenAI 客户端
    model_client = OpenAIChatCompletionClient(
        model=model,
        base_url=openai_base_url,
        temperature=temperature,
    )

    # 创建智能体
    calc_agent = AssistantAgent(
        name="calc",
        model_client=model_client,
        workbench=workbench,
        reflect_on_tool_use=True,
    )
```

## Rollout 函数

### 核心执行逻辑
```python
@agl.rollout
async def calc_agent(task: MathProblem, llm: agl.LLM) -> None:
    # 创建 MCP 计算器服务器
    calculator_mcp_server = StdioServerParams(
        command="uvx",
        args=["mcp-server-calculator"]
    )

    async with McpWorkbench(calculator_mcp_server) as workbench:
        # 初始化 AutoGen 智能体
        calc_agent = autogen_assistant_agent(
            llm.model, llm.endpoint, llm.sampling_parameters, workbench
        )

        # 执行推理
        output_format = "Output the answer when you are ready..."
        prompt = task["question"] + " " + output_format
        result = await calc_agent.run(task=prompt)

        # 提取答案
        answer = extract_answer(result.messages[-1].content)

        # 评估并发射奖励
        reward = await evaluate(answer, str(task["result"]))
        agl.emit_reward(reward)
```

## 评估系统

### eval_utils.py 功能
- **答案提取**: 从 LLM 输出中提取数值答案
- **数值比较**: 支持整数、浮点数、分数的比较
- **容错机制**: 处理格式错误和特殊情况

```python
async def evaluate(predicted: str, ground_truth: str) -> float:
    """
    评估预测答案与真实答案的匹配度

    Returns:
        float: 1.0 表示完全匹配，0.0 表示不匹配
    """
    # 数值提取和标准化
    # 支持多种答案格式
    # 处理浮点精度问题
```

## 训练配置

### 数据格式
示例使用 JSONL 格式的数学问题数据集：
```json
{"id": "math_001", "question": "What is 15 + 27?", "result": "42", "source": "synthetic"}
{"id": "math_002", "question": "Calculate 8 * 6 - 10", "result": "38", "source": "synthetic"}
```

### 训练参数
- **模型**: 支持各种 OpenAI 兼容模型
- **温度**: 控制输出的随机性
- **批次大小**: 根据计算资源调整
- **验证集**: 用于评估提示优化效果

## 工具集成

### MCP (Model Context Protocol) 计算器
- **服务器**: `mcp-server-calculator`
- **启动方式**: 通过 `uvx` 动态安装和运行
- **功能**: 基础算术运算、复杂计算
- **集成**: AutoGen 的 McpWorkbench

### 工具使用特性
- **自动工具选择**: 智能体自主决定何时使用计算器
- **错误处理**: 处理工具调用失败的情况
- **反思机制**: `reflect_on_tool_use=True` 启用工具使用反思

## 测试策略

### 单元测试
- **MCP 计算器测试**: 验证工具的基本功能
- **AgentOps 集成测试**: 确保追踪系统正常工作
- **答案评估测试**: 验证评估逻辑的正确性

### 集成测试
- **端到端流程**: 从问题到答案的完整流程
- **工具使用场景**: 需要计算器的复杂问题
- **错误场景**: 异常情况的处理

## 性能优化

### 并行执行
- 使用 Agent Lightning 的异步执行策略
- 支持批量处理多个数学问题
- 可配置的并发级别

### 缓存机制
- 工具调用结果缓存
- 计算结果重用
- 减少重复计算开销

## 扩展功能

### 遗留版本支持
- `legacy_calc_agent.py`: 旧版本兼容
- `legacy_calc_agent_debug.py`: 调试增强版本
- 渐进式迁移支持

### 调试工具
- 详细的执行日志
- 中间步骤追踪
- 错误诊断信息

## 常见问题 (FAQ)

### Q: 如何添加新的数学运算类型？
A:
1. 扩展 `eval_utils.py` 中的评估逻辑
2. 更新答案提取的正则表达式
3. 添加相应的测试用例

### Q: MCP 计算器工具无法启动怎么办？
A:
1. 检查 `uvx` 是否正确安装
2. 确认网络连接正常
3. 尝试手动运行 `uvx mcp-server-calculator`

### Q: 如何处理更复杂的数学问题？
A:
1. 扩展问题描述格式
2. 添加更多工具（如符号计算库）
3. 调整智能体的提示模板

### Q: 训练过程中如何监控进度？
A:
1. 启用 AgentOps 集成查看详细轨迹
2. 使用 Agent Lightning 的内置日志
3. 监控验证集上的性能指标

## 相关文件清单

### 核心文件
- [`calc_agent.py`](calc_agent.py) - 主要智能体定义和 rollout 函数
- [`eval_utils.py`](eval_utils.py) - 评估和答案提取工具
- [`train_calc_agent.py`](train_calc_agent.py) - 训练脚本

### 兼容性文件
- [`legacy_calc_agent.py`](legacy_calc_agent.py) - 旧版本智能体
- [`legacy_calc_agent_debug.py`](legacy_calc_agent_debug.py) - 调试版本

### 测试文件
- [`tests/test_agentops.py`](tests/test_agentops.py) - AgentOps 集成测试
- [`tests/test_mcp_calculator.py`](tests/test_mcp_calculator.py) - 计算器工具测试

### 相关依赖
- [`../README.md`](../README.md) - 示例项目总体说明
- [`../../agentlightning/`](../../agentlightning/) - 框架核心代码

## 变更记录 (Changelog)

- **2025-11-20**: 创建示例文档
- 详细描述了 Calc-X 示例的实现细节
- 添加了配置和使用指南

---

*最后更新：2025-11-20 | 示例版本：0.2.2*