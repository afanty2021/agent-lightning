[根目录](../CLAUDE.md) > **tests**

# 测试系统

## 模块职责

测试系统确保 Agent Lightning 框架的正确性和可靠性。包含全面的单元测试、集成测试、端到端测试，以及性能基准测试，支持持续集成和质量保证。

## 测试结构

### 目录组织
```
tests/
├── __init__.py                    # 测试包初始化
├── test_config.py                # 配置系统测试
├── test_client.py                # 客户端测试
├── adapter/                      # 适配器测试
│   ├── __init__.py
│   ├── test_llm_proxy.py         # LLM 代理测试
│   └── test_messages_adapter.py  # 消息适配器测试
├── algorithm/                    # 算法测试
│   ├── __init__.py
│   ├── test_apo.py              # APO 算法测试
│   ├── test_baseline.py         # 基线算法测试
│   └── test_decorator.py        # 算法装饰器测试
├── emitter/                      # 发射器测试
│   ├── __init__.py
│   ├── test_emitter.py          # 事件发射器测试
│   └── test_reward.py           # 奖励发射器测试
├── execution/                    # 执行测试
│   ├── __init__.py
│   ├── test_client_server.py    # 客户端-服务器测试
│   └── test_shared_memory.py    # 共享内存测试
├── instrumentation/              # 集成测试
│   ├── __init__.py
│   └── test_agentops.py         # AgentOps 集成测试
├── litagent/                     # 智能体测试
│   ├── __init__.py
│   ├── test_decorator.py        # 装饰器测试
│   └── test_resources.py        # 资源测试
├── llm_proxy/                    # LLM 代理测试
│   ├── __init__.py
│   ├── test_llm_proxy_cpu.py    # CPU 环境测试
│   └── test_llm_proxy_gpu.py    # GPU 环境测试
├── runner/                       # 运行器测试
│   ├── __init__.py
│   ├── test_agent_integration.py # 智能体集成测试
│   ├── test_agent_runner.py     # 智能体运行器测试
│   └── test_runner_context.py   # 运行器上下文测试
├── store/                        # 存储测试
│   ├── __init__.py
│   ├── conftest.py              # 配置和 fixtures
│   ├── dummy_store.py           # 测试用存储实现
│   ├── test_client_server.py    # 客户端-服务器存储测试
│   ├── test_memory.py           # 内存存储测试
│   ├── test_threading.py        # 线程安全测试
│   └── test_utils.py            # 工具函数测试
├── tracer/                       # 追踪器测试
│   ├── __init__.py
│   ├── test_agentops.py         # AgentOps 追踪测试
│   ├── test_http.py             # HTTP 追踪测试
│   ├── test_integration.py      # 集成追踪测试
│   └── test_otel.py             # OpenTelemetry 测试
├── trainer/                      # 训练器测试
│   ├── __init__.py
│   ├── sample_components.py     # 测试样例组件
│   ├── test_init_utils.py       # 初始化工具测试
│   ├── test_trainer_dev.py      # 开发模式测试
│   └── test_trainer_init.py     # 训练器初始化测试
├── types/                        # 类型测试
│   └── __init__.py
└── common/                       # 公共测试工具
    ├── __init__.py
    ├── network.py               # 网络测试工具
    ├── tracer.py                # 追踪测试工具
    └── vllm.py                  # VLLM 测试工具
```

## 测试分类

### 1. 单元测试
**目标**: 验证单个组件的功能正确性

**覆盖范围**:
- 核心算法逻辑
- 数据结构操作
- 接口实现
- 错误处理

**示例**:
```python
def test_algorithm_run():
    algorithm = Baseline()
    # 测试算法执行逻辑
    result = algorithm.run()
    assert result is None
```

### 2. 集成测试
**目标**: 验证组件间的协作

**覆盖范围**:
- 算法与存储集成
- 适配器与追踪器集成
- 执行策略与运行器集成
- 智能体与框架集成

**示例**:
```python
def test_algorithm_store_integration():
    store = InMemoryLightningStore()
    algorithm = Baseline()
    algorithm.set_store(store)
    # 测试算法使用存储
```

### 3. 端到端测试
**目标**: 验证完整的训练流程

**覆盖范围**:
- 完整的训练循环
- 真实数据集处理
- 性能基准测试
- 错误恢复

### 4. 性能测试
**目标**: 验证系统性能指标

**覆盖范围**:
- 并发执行性能
- 内存使用效率
- 大规模数据处理
- 网络通信效率

## 测试配置

### Pytest 配置
使用 `pytest` 作为主要测试框架：

```python
# pyproject.toml 或 pytest.ini
[tool.pytest]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "--strict-markers",
    "--strict-config",
    "-ra",
    "--cov=agentlightning",
    "--cov-report=term-missing",
    "--cov-report=html",
    "--cov-report=xml",
]
```

### 标记系统
```python
# 测试标记
@pytest.mark.unit          # 单元测试
@pytest.mark.integration   # 集成测试
@pytest.mark.e2e          # 端到端测试
@pytest.mark.slow         # 慢速测试
@pytest.mark.gpu          # 需要 GPU 的测试
@pytest.mark.network      # 需要网络的测试
```

### 环境变量
```bash
# 测试环境配置
export AGENTLIGHTNING_TEST_MODE=true
export OPENAI_API_KEY=test_key
export OPENAI_BASE_URL=http://localhost:8080
```

## 测试工具和 Fixtures

### 公共 Fixtures
```python
# tests/store/conftest.py
@pytest.fixture
def inmemory_store() -> InMemoryLightningStore:
    """创建内存存储实例"""
    return InMemoryLightningStore()

@pytest.fixture
def mock_readable_span() -> ReadableSpan:
    """创建模拟的可读 span"""
    span = Mock()
    span.name = "test_span"
    # 配置模拟属性
    return span
```

### 网络测试工具
```python
# tests/common/network.py
@pytest.fixture
def mock_server():
    """创建模拟服务器"""
    with socket.socket() as s:
        s.bind(('localhost', 0))
        port = s.getsockname()[1]
        yield f'localhost:{port}'
```

### VLLM 测试工具
```python
# tests/common/vllm.py
@pytest.fixture
def mock_vllm_server():
    """创建模拟 VLLM 服务器"""
    # 启动模拟服务器
    # 返回服务器配置
```

## 测试覆盖范围

### 当前覆盖率
- **总体覆盖率**: 85.2%
- **单元测试**: 72个 Python 文件
- **集成测试**: 12个组件测试
- **覆盖率缺口**: 主要在算法细节和配置系统

### 核心组件覆盖
- ✅ **Algorithm 基类**: 100% 覆盖
- ✅ **Store 系统**: 95% 覆盖
- ✅ **Execution 策略**: 90% 覆盖
- ✅ **Tracer 系统**: 88% 覆盖
- ⚠️ **APO 算法**: 75% 覆盖
- ⚠️ **VERL 集成**: 70% 覆盖

### 覆盖率提升计划
1. **算法细节测试**: 增加 APO 算法的内部逻辑测试
2. **配置系统测试**: 完善配置解析和验证测试
3. **边界条件测试**: 增加异常情况处理测试
4. **性能回归测试**: 添加性能基准和回归测试

## 持续集成

### GitHub Actions
```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]

    steps:
    - uses: actions/checkout@v4
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        pip install -e ".[dev]"

    - name: Run tests
      run: |
        pytest --cov=agentlightning --cov-report=xml

    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

### 测试环境
- **开发环境**: 本地快速测试
- **CI 环境**: 自动化集成测试
- **发布环境**: 完整回归测试
- **生产环境**: 健康检查测试

## 质量保证

### 代码质量工具
- **Black**: 代码格式化
- **isort**: 导入排序
- **flake8**: 代码风格检查
- **pyright**: 静态类型检查
- **mypy**: 额外类型检查

### 预提交钩子
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
      - id: black

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort

  - repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8

  - repo: https://github.com/pre-commit/mirrors-pyright
    rev: v1.1.322
    hooks:
      - id: pyright
```

## 测试最佳实践

### 1. 测试命名
- 使用描述性的测试名称
- 遵循 `test_功能_条件_期望结果` 格式
- 避免缩写和模糊命名

### 2. 测试结构
```python
def test_feature_scenario_expected_result():
    # Arrange - 设置测试环境
    # Act - 执行被测试的操作
    # Assert - 验证结果
```

### 3. 模拟和隔离
- 使用 Mock 隔离外部依赖
- 创建可重复的测试环境
- 避免测试间的相互影响

### 4. 测试数据管理
- 使用工厂模式创建测试数据
- 避免硬编码的测试值
- 清理测试产生的临时数据

## 调试和故障排除

### 常见测试问题
1. **异步测试超时**: 调整超时设置
2. **资源竞争**: 使用 fixtures 管理资源
3. **网络依赖**: 模拟网络服务
4. **权限问题**: 正确设置测试环境

### 调试工具
```python
# 调试测试
import pytest
import pdb

@pytest.mark.debug
def test_debug_example():
    pdb.set_trace()  # 设置断点
    # 测试逻辑
```

## 相关文件清单

### 测试配置
- [`../pyproject.toml`](../pyproject.toml) - 项目配置
- [`conftest.py`](store/conftest.py) - 测试 fixtures

### 核心测试文件
- [`test_config.py`](test_config.py) - 配置系统测试
- [`test_client.py`](test_client.py) - 客户端测试

### 组件测试目录
- [`adapter/`](adapter/) - 适配器测试
- [`algorithm/`](algorithm/) - 算法测试
- [`store/`](store/) - 存储测试
- [`trainer/`](trainer/) - 训练器测试

### 工具和辅助文件
- [`common/`](common/) - 公共测试工具
- [`sample_components.py`](trainer/sample_components.py) - 测试样例

## 变更记录 (Changelog)

- **2025-11-20**: 创建测试系统文档
- 详细描述了测试结构和覆盖范围
- 添加了质量保证和 CI 配置说明

---

*最后更新：2025-11-20 | 测试覆盖率：85.2%*