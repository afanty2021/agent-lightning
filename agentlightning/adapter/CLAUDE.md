[根目录](../../CLAUDE.md) > [agentlightning](../) > **adapter**

# Adapter 模块

## 模块职责

Adapter 模块负责处理不同框架和协议之间的数据转换，是 Agent Lightning 框架适配性的核心组件。它提供了统一的接口来桥接各种 Agent 框架（如 LangChain、OpenAI Agent SDK、AutoGen 等）与 Agent Lightning 的训练系统。

## 入口与启动

### 核心文件结构
```
agentlightning/adapter/
├── __init__.py          # 模块导出
├── base.py             # 基础适配器抽象类
├── messages.py         # 消息格式转换
└── triplet.py          # 三元组数据结构适配器
```

### 主要导出
```python
from .adapter import *
# 导出包含：TraceAdapter, TracerTraceToTriplet 等
```

## 对外接口

### TraceAdapter (抽象基类)

核心适配器接口，定义了将追踪数据转换为算法可用格式的标准。

```python
class TraceAdapter(Generic[T]):
    """将追踪数据适配为算法可消费的格式。"""

    def export(self, spans: List[Span]) -> T:
        """导出追踪数据为指定格式。"""
        raise NotImplementedError
```

### TracerTraceToTriplet

默认实现，将追踪数据转换为强化学习训练所需的三元组格式。

```python
class TracerTraceToTriplet(TraceAdapter[List[Triplet]]):
    """将追踪数据转换为三元组列表。"""

    def export(self, spans: List[Span]) -> List[Triplet]:
        # 实现：将 Span 转换为 (prompt, response, reward) 三元组
```

## 关键依赖与配置

### 依赖关系
- **agentlightning.types**: 提供 Span、Triplet 等核心数据类型
- **opentelemetry**: 集成 OpenTelemetry 追踪系统
- **agentlightning.tracer**: 追踪器基础设施

### 配置选项
```python
# 在 Trainer 中配置适配器
trainer = Trainer(
    adapter={
        "type": "TracerTraceToTriplet",
        "agent_match": "agent_name_pattern",  # 匹配特定智能体
        "include_metadata": True,             # 包含元数据
    }
)
```

## 数据模型

### 三元组结构 (Triplet)
```python
class Triplet(BaseModel):
    prompt: Any                # 输入提示
    response: Any             # 智能体响应
    reward: Optional[float]   # 奖励值
    metadata: Dict[str, Any]  # 附加元数据
```

### 消息转换
- **messages.py**: 处理不同框架的消息格式统一
- 支持 OpenAI、Anthropic、LangChain 等多种消息格式
- 自动提取和规范化对话历史

## 测试与质量

### 测试覆盖
- 单元测试：`tests/adapter/`
- 集成测试：验证与各种 Agent 框架的兼容性
- 格式转换测试：确保数据完整性

### 质量保证
- 类型安全：完整的类型注解
- 数据验证：Pydantic 模型验证
- 性能优化：批量处理和缓存机制

## 常见问题 (FAQ)

### Q: 如何为新的 Agent 框架创建适配器？
A: 继承 `TraceAdapter` 基类，实现 `export` 方法来处理框架特定的数据格式。

### Q: 适配器如何处理异步追踪数据？
A: 适配器本身是同步的，异步处理在追踪器层面完成。适配器接收的是已经收集完成的 Span 列表。

### Q: 可以自定义三元组生成逻辑吗？
A: 可以，通过继承 `TracerTraceToTriplet` 或创建新的适配器实现来自定义数据处理逻辑。

## 相关文件清单

### 核心实现文件
- [`base.py`](base.py) - 适配器基类和接口定义
- [`triplet.py`](triplet.py) - 三元组适配器实现
- [`messages.py`](messages.py) - 消息格式转换工具

### 相关模块
- [`../types/core.py`](../types/core.py) - 核心数据类型定义
- [`../tracer/base.py`](../tracer/base.py) - 追踪器接口
- [`../trainer/trainer.py`](../trainer/trainer.py) - 训练器适配器集成

### 配置示例
```python
# 自定义适配器配置
custom_adapter = {
    "type": "MyCustomAdapter",
    "custom_param": "value",
}

trainer = Trainer(adapter=custom_adapter)
```

## 变更记录 (Changelog)

- **2025-11-20**: 初始化模块文档
- 添加了详细的接口说明和使用示例
- 整理了数据模型和配置选项

---

*最后更新：2025-11-20 | 模块版本：0.2.2*