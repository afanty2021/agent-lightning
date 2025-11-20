[根目录](../../CLAUDE.md) > [agentlightning](../) > **algorithm**

# Algorithm 模块

## 模块职责

Algorithm 模块是 Agent Lightning 的训练算法核心，提供了各种智能体优化算法的统一接口。包括强化学习、自动提示优化（APO）、监督微调等多种训练策略，以及与 VERL（Versatile Reinforcement Learning）框架的深度集成。

## 入口与启动

### 核心文件结构
```
agentlightning/algorithm/
├── __init__.py           # 模块导出
├── base.py              # 算法基类
├── decorator.py         # 算法装饰器
├── fast.py              # 快速算法基类
├── apo/                 # APO 算法模块
│   ├── __init__.py
│   ├── apo.py          # APO 主实现
│   └── prompts/        # APO 提示模板
└── verl/                # VERL 集成模块
    ├── __init__.py
    └── interface.py    # VERL 接口
```

### 主要导出
```python
from .algorithm import *
# 导出包含：Algorithm, FastAlgorithm, Baseline, APO, VERL 等
```

## 对外接口

### Algorithm (抽象基类)

所有训练算法的基类，定义了标准的算法生命周期和接口。

```python
class Algorithm:
    """算法策略，用于训练智能体。"""

    def run(
        self,
        train_dataset: Optional[Dataset[Any]] = None,
        val_dataset: Optional[Dataset[Any]] = None,
    ) -> Union[None, Awaitable[None]]:
        """执行训练算法。"""
        raise NotImplementedError

    def is_async(self) -> bool:
        """返回算法是否为异步执行。"""
```

### FastAlgorithm

快速算法基类，适用于开发和调试场景，提供同步执行保证。

```python
class FastAlgorithm(Algorithm):
    """快速执行算法，适用于开发和调试。"""

    def is_async(self) -> bool:
        return False  # 强制同步执行
```

### Baseline

基准算法，用于对比测试和快速验证。

```python
class Baseline(FastAlgorithm):
    """基准算法，不做任何优化，仅用于数据收集。"""
```

## 关键依赖与配置

### 依赖关系
- **agentlightning.store**: 存储接口，用于数据交换
- **agentlightning.adapter**: 适配器，处理数据转换
- **agentlightning.llm_proxy**: LLM 代理，用于算法调用
- **agentlightning.types**: 核心数据类型

### 配置示例
```python
# 在 Trainer 中配置算法
trainer = Trainer(
    algorithm={
        "type": "APO",
        "max_iterations": 10,
        "temperature": 0.7,
        "prompt_template": "my_template.poml"
    }
)
```

## 算法实现

### APO (Automatic Prompt Optimization)

自动提示优化算法，通过梯度下降优化提示词。

```python
class APO(Algorithm):
    """自动提示优化算法。"""

    def __init__(
        self,
        max_iterations: int = 10,
        temperature: float = 0.7,
        prompt_template: Optional[str] = None
    ):
        # 初始化参数和提示模板
```

#### APO 提示模板
APO 算法使用 `.poml` 格式的提示模板：
- `apply_edit_variant01.poml` - 应用编辑变体1
- `apply_edit_variant02.poml` - 应用编辑变体2
- `text_gradient_variant01.poml` - 文本梯度变体1

### VERL 集成

与 VERL 框架的深度集成，支持大规模强化学习训练。

```python
class VERLAlgorithm(Algorithm):
    """VERL 算法集成。"""

    def __init__(self, verl_config: Dict[str, Any]):
        # VERL 特定配置
```

## 算法装饰器

### @algorithm_decorator

为算法函数提供装饰器支持，简化算法实现。

```python
@algorithm_decorator
def my_algorithm(train_dataset, val_dataset):
    # 算法实现逻辑
    pass
```

## 数据流程

### 训练循环
1. **数据获取**: 从 Store 获取训练数据
2. **数据处理**: 通过 Adapter 转换数据格式
3. **算法执行**: 运行优化算法
4. **资源更新**: 更新模型参数或提示模板
5. **结果存储**: 将更新后的资源存回 Store

### 资源管理
- **初始资源**: 算法启动时的基础资源
- **资源更新**: 训练过程中产生的更新
- **版本控制**: 资源的版本管理和回滚

## 测试与质量

### 测试策略
- **单元测试**: 每个算法的独立测试
- **集成测试**: 算法与存储、适配器的集成
- **性能测试**: 大规模数据下的算法性能

### 质量保证
- **异步支持**: 所有算法支持异步执行
- **错误处理**: 完善的异常处理机制
- **资源管理**: 自动的资源清理和释放

## 常见问题 (FAQ)

### Q: 如何实现自定义算法？
A: 继承 `Algorithm` 基类，实现 `run` 方法。如果需要快速开发，可以继承 `FastAlgorithm`。

### Q: 算法如何访问训练数据？
A: 通过 `self.get_store()` 访问存储，使用 `get_latest_resources()` 获取最新资源。

### Q: 如何在算法中使用 LLM？
A: 通过 `self.get_llm_proxy()` 获取 LLM 代理，直接调用 LLM API。

### Q: APO 和 VERL 有什么区别？
A: APO 专注于提示优化，适合小规模快速迭代；VERL 是通用的强化学习框架，适合大规模训练。

## 相关文件清单

### 核心实现文件
- [`base.py`](base.py) - 算法基类和接口
- [`fast.py`](fast.py) - 快速算法基类
- [`decorator.py`](decorator.py) - 算法装饰器
- [`apo/apo.py`](apo/apo.py) - APO 算法实现
- [`verl/interface.py`](verl/interface.py) - VERL 集成接口

### APO 提示模板
- [`apo/prompts/apply_edit_variant01.poml`](apo/prompts/apply_edit_variant01.poml)
- [`apo/prompts/apply_edit_variant02.poml`](apo/prompts/apply_edit_variant02.poml)
- [`apo/prompts/text_gradient_variant01.poml`](apo/prompts/text_gradient_variant01.poml)

### 相关模块
- [`../store/base.py`](../store/base.py) - 存储接口
- [`../adapter/base.py`](../adapter/base.py) - 适配器接口
- [`../llm_proxy.py`](../llm_proxy.py) - LLM 代理
- [`../types/core.py`](../types/core.py) - 核心数据类型

## 扩展指南

### 创建新算法
```python
class MyAlgorithm(Algorithm):
    def __init__(self, my_param: str = "default"):
        super().__init__()
        self.my_param = my_param

    def run(self, train_dataset=None, val_dataset=None):
        # 获取存储和适配器
        store = self.get_store()
        adapter = self.get_adapter()

        # 实现算法逻辑
        # ...

        # 更新资源
        resources = {"my_resource": updated_value}
        store.add_resources(resources)
```

### 算法配置
```python
# 注册自定义算法
from agentlightning.trainer.registry import AlgorithmRegistry

AlgorithmRegistry.register("my_algorithm", MyAlgorithm)
```

## 变更记录 (Changelog)

- **2025-11-20**: 初始化模块文档
- 添加了 APO 和 VERL 算法的详细说明
- 整理了算法开发的最佳实践

---

*最后更新：2025-11-20 | 模块版本：0.2.2*