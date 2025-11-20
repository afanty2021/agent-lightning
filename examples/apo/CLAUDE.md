[根目录](../../CLAUDE.md) > [examples](../) > **apo**

# APO (Automatic Prompt Optimization) 示例

## 相对路径面包屑

[根目录](../../../CLAUDE.md) > [examples](../../) > **apo**

## 示例职责

本示例展示了如何使用 Agent Lightning 的 APO（Automatic Prompt Optimization）算法来自动优化提示词。APO 通过梯度下降方法迭代改进提示模板，适用于需要精调提示词以提升模型性能的场景。

## 入口与启动

### 核心文件结构
```
examples/apo/
├── README.md                 # 示例说明文档
├── apo_custom_algorithm.py   # 自定义 APO 算法示例
├── apo_custom_algorithm_trainer.py  # 自定义算法训练器
├── apo_debug.py              # APO 调试工具
├── legacy_apo_client.py      # 遗留客户端实现
├── legacy_apo_server.py      # 遗留服务器实现
├── room_selector.py          # 房间选择智能体
└── room_selector_apo.py      # 房间选择 APO 训练
```

### 主要入口文件

#### 1. room_selector_apo.py
```bash
# 运行房间选择 APO 训练
python room_selector_apo.py
```

#### 2. apo_debug.py
```bash
# APO 调试和可视化
python apo_debug.py
```

## 对外接口

### 核心智能体实现

#### RoomSelector (room_selector.py)
```python
class RoomSelector(LitAgent):
    """房间选择智能体，根据用户需求推荐合适的房间。"""

    def rollout(self, task, resources, rollout):
        # 获取房间信息和用户需求
        user_request = task["user_request"]
        rooms = resources["rooms"]

        # 构建提示词
        prompt = resources["prompt_template"].format(
            user_request=user_request,
            rooms=rooms
        )

        # 调用 LLM 获取推荐
        response = self.llm_call(prompt)

        # 解析响应并计算奖励
        recommendation = parse_response(response)
        reward = calculate_recommendation_quality(
            recommendation, user_request, rooms
        )

        return reward
```

### APO 算法集成

#### 自定义 APO 实现 (apo_custom_algorithm.py)
```python
class MyCustomAPO(Algorithm):
    """自定义 APO 算法实现。"""

    def __init__(
        self,
        max_iterations: int = 10,
        learning_rate: float = 0.1,
        prompt_template: str = "default_prompt.poml"
    ):
        super().__init__()
        self.max_iterations = max_iterations
        self.learning_rate = learning_rate
        self.prompt_template = prompt_template

    def run(self, train_dataset=None, val_dataset=None):
        # APO 训练循环
        for iteration in range(self.max_iterations):
            # 1. 收集训练数据
            train_results = self.collect_training_data(train_dataset)

            # 2. 计算梯度
            gradients = self.compute_gradients(train_results)

            # 3. 更新提示模板
            updated_prompt = self.update_prompt(gradients)

            # 4. 验证性能
            val_reward = self.validate_performance(val_dataset, updated_prompt)

            # 5. 保存最佳模板
            if val_reward > best_reward:
                self.save_best_template(updated_prompt)
```

## 关键依赖与配置

### 依赖要求
```python
# pyproject.toml 中的依赖
apo = [
    "poml",  # Prompt Optimization Markup Language
]

# 额外依赖
from agentlightning import Trainer, APO
from agentlightning.litagent import LitAgent
import pandas as pd  # 数据处理
import matplotlib.pyplot as plt  # 可视化
```

### 配置示例

#### 基础配置
```python
# 房间选择任务配置
config = {
    "agent": {
        "type": "RoomSelector",
        "llm_model": "gpt-3.5-turbo",
        "temperature": 0.7
    },
    "algorithm": {
        "type": "APO",
        "max_iterations": 15,
        "temperature": 0.8,
        "batch_size": 8
    },
    "trainer": {
        "n_runners": 2,
        "max_rollouts": 50,
        "store": "InMemoryLightningStore"
    }
}
```

#### 提示模板配置
```poml
<!-- room_selector.poml -->
You are a helpful hotel room recommendation assistant.

User Request: {user_request}

Available Rooms:
{rooms}

Please recommend the best room and explain your reasoning.

Recommendation:
```

## 数据模型

### 任务输入格式
```python
task = {
    "user_request": "I need a quiet room for two nights",
    "preferences": {
        "price_range": "medium",
        "bed_type": "queen",
        "view": "garden"
    }
}
```

### 房间数据格式
```python
rooms = [
    {
        "id": "room_101",
        "type": "standard",
        "price": 120,
        "features": ["quiet", "garden_view", "queen_bed"]
    },
    {
        "id": "room_205",
        "type": "deluxe",
        "price": 200,
        "features": ["quiet", "city_view", "king_bed"]
    }
]
```

## 训练流程

### APO 训练循环
1. **初始化**: 加载初始提示模板
2. **数据收集**: 使用当前模板生成推荐
3. **奖励计算**: 评估推荐质量
4. **梯度估计**: 基于奖励计算提示词梯度
5. **模板更新**: 应用梯度更新提示模板
6. **验证**: 在验证集上测试性能
7. **迭代**: 重复直到收敛或达到最大迭代数

### 奖励函数设计
```python
def calculate_reward(recommendation, user_request, rooms):
    """计算推荐质量的奖励函数。"""
    score = 0.0

    # 相关性匹配 (40%)
    if matches_preferences(recommendation, user_request):
        score += 0.4

    # 价格合理性 (30%)
    if is_reasonably_priced(recommendation, user_request):
        score += 0.3

    # 解释清晰度 (20%)
    if has_clear_explanation(recommendation):
        score += 0.2

    # 房间可用性 (10%)
    if is_room_available(recommendation):
        score += 0.1

    return score
```

## 测试与质量

### 测试数据
```python
# 测试用例示例
test_cases = [
    {
        "user_request": "Budget room for one night",
        "expected_room_type": "standard",
        "max_price": 150
    },
    {
        "user_request": "Luxury room with city view",
        "expected_room_type": "deluxe",
        "required_features": ["city_view"]
    }
]
```

### 性能指标
- **准确率**: 推荐房间符合用户需求的比例
- **平均奖励**: 训练过程中的平均奖励值
- **收敛速度**: 达到稳定性能所需的迭代数
- **提示词质量**: 最终提示词的清晰度和有效性

## 调试和可视化

### APO 调试工具 (apo_debug.py)
```python
def visualize_training_progress(training_history):
    """可视化训练进度。"""
    plt.figure(figsize=(12, 4))

    # 奖励趋势
    plt.subplot(1, 3, 1)
    plt.plot(training_history["rewards"])
    plt.title("Reward Progress")
    plt.xlabel("Iteration")
    plt.ylabel("Average Reward")

    # 提示词变化
    plt.subplot(1, 3, 2)
    plt.plot(training_history["prompt_similarity"])
    plt.title("Prompt Similarity")
    plt.xlabel("Iteration")

    # 性能分布
    plt.subplot(1, 3, 3)
    plt.hist(training_history["final_rewards"])
    plt.title("Final Performance Distribution")

    plt.tight_layout()
    plt.show()
```

### 日志记录
```python
import logging

# 配置详细日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger("APO_Training")

# 在训练循环中记录关键信息
logger.info(f"Iteration {iteration}: Avg Reward = {avg_reward:.3f}")
logger.debug(f"Prompt update: {prompt_diff}")
```

## 常见问题 (FAQ)

### Q: APO 适用于哪些场景？
A: APO 特别适用于：
- 需要精调提示词的任务
- 有明确奖励函数的场景
- 可以批量生成的任务
- 提示词质量直接影响结果的场景

### Q: 如何设计好的奖励函数？
A: 好的奖励函数应该：
- 反映任务的核心目标
- 数值范围合理（通常 0-1）
- 计算效率高
- 对输入变化敏感

### Q: APO 训练需要多少数据？
A: 数据需求取决于：
- 任务复杂度
- 提示词初始质量
- 期望的性能提升
- 通常建议每个迭代至少 8-16 个样本

### Q: 如何处理收敛问题？
A:
- 调整学习率
- 增加批量大小
- 使用梯度裁剪
- 尝试不同的提示模板初始化

## 扩展和定制

### 自定义梯度估计器
```python
class CustomGradientEstimator:
    def estimate_gradients(self, data, current_prompt):
        """自定义梯度估计逻辑。"""
        # 实现自定义的梯度估计算法
        pass
```

### 多任务 APO
```python
class MultiTaskAPO(APO):
    def __init__(self, tasks, task_weights):
        super().__init__()
        self.tasks = tasks
        self.task_weights = task_weights

    def compute_gradients(self, data):
        """计算多任务梯度。"""
        total_gradient = None
        for task, weight in zip(self.tasks, self.task_weights):
            task_data = data[task]
            task_gradient = super().compute_gradients(task_data)
            total_gradient = self.combine_gradients(
                total_gradient, task_gradient, weight
            )
        return total_gradient
```

## 相关文件清单

### 核心示例文件
- [`room_selector.py`](room_selector.py) - 房间选择智能体
- [`room_selector_apo.py`](room_selector_apo.py) - APO 训练脚本
- [`apo_debug.py`](apo_debug.py) - 调试和可视化工具
- [`apo_custom_algorithm.py`](apo_custom_algorithm.py) - 自定义 APO 实现

### 遗留兼容文件
- [`legacy_apo_client.py`](legacy_apo_client.py) - 遗留客户端
- [`legacy_apo_server.py`](legacy_apo_server.py) - 遗留服务器
- [`apo_custom_algorithm_trainer.py`](apo_custom_algorithm_trainer.py) - 遗留训练器

### 相关资源
- [`README.md`](README.md) - 示例详细说明
- 数据文件：房间信息、用户请求等

## 变更记录 (Changelog)

- **2025-11-20**: 初始化示例文档
- 添加了详细的 APO 算法说明
- 整理了调试和扩展指南
- 补充了最佳实践和常见问题

---

*最后更新：2025-11-20 | 示例版本：0.2.2*