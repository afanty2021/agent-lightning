# API参考

<cite>
**本文档中引用的文件**  
- [client.py](file://agentlightning/client.py)
- [server.py](file://agentlightning/server.py)
- [config.py](file://agentlightning/config.py)
- [litagent/decorator.py](file://agentlightning/litagent/decorator.py)
- [algorithm/decorator.py](file://agentlightning/algorithm/decorator.py)
- [cli/__init__.py](file://agentlightning/cli/__init__.py)
- [cli/vllm.py](file://agentlightning/cli/vllm.py)
- [cli/store.py](file://agentlightning/cli/store.py)
- [cli/agentops_server.py](file://agentlightning/cli/agentops_server.py)
- [types/core.py](file://agentlightning/types/core.py)
- [types/resources.py](file://agentlightning/types/resources.py)
- [types/tracer.py](file://agentlightning/types/tracer.py)
- [store/base.py](file://agentlightning/store/base.py)
- [trainer/trainer.py](file://agentlightning/trainer/trainer.py)
- [litagent/litagent.py](file://agentlightning/litagent/litagent.py)
</cite>

## 目录
1. [Python API](#python-api)
2. [REST API](#rest-api)
3. [CLI命令](#cli命令)
4. [装饰器接口](#装饰器接口)
5. [跨API调用示例](#跨api调用示例)
6. [版本兼容性说明](#版本兼容性说明)

## Python API

### 公共类与方法

#### AgentLightningClient
该类为与旧版Agent Lightning服务器交互的客户端包装器，支持同步和异步操作。

**属性**:
- `endpoint`: Agent Lightning服务器的基URL。
- `poll_interval`: 当无任务可用时，轮询尝试之间的延迟（秒）。
- `timeout`: HTTP请求的超时时间（秒）。
- `task_count`: 此客户端生命周期内声明的任务数。

**方法**:
- `poll_next_task_async()`: 异步轮询服务器直到有任务可用。
- `get_resources_by_id_async(resource_id)`: 根据标识符异步获取特定资源包。
- `get_latest_resources_async()`: 异步获取服务器公布的最新资源包。
- `post_rollout_async(rollout)`: 异步提交已完成的rollout回服务器。
- `poll_next_task()`: 同步轮询服务器直到有任务可用。
- `get_resources_by_id(resource_id)`: 根据标识符同步获取特定资源包。
- `get_latest_resources()`: 同步获取服务器公布的最新资源包。
- `post_rollout(rollout)`: 同步提交已完成的rollout回服务器。

**Section sources**
- [client.py](file://agentlightning/client.py#L45-L374)

#### AgentLightningServer
该类为与原始Agent Lightning协议兼容的旧版HTTP服务器的高级控制器。

**属性**:
- `host`: 绑定HTTP服务器的主机名或IP地址。
- `port`: 服务器暴露的TCP端口。
- `endpoint`: 服务器的完整端点URL。
- `_task_timeout_seconds`: 在认为声明的任务已过期并重新排队之前的时间（秒）。

**方法**:
- `start()`: 在后台启动FastAPI服务器。
- `stop()`: 停止FastAPI服务器并等待优雅关闭。
- `run_forever()`: 无限运行服务器直到调用`stop()`。
- `queue_task(sample, mode, resources_id, metadata)`: 将任务添加到队列中供客户端处理。
- `update_resources(resources)`: 发布新的资源包并返回其生成的标识符。
- `get_completed_rollout(rollout_id)`: 根据标识符检索特定的已完成rollout。
- `poll_completed_rollout(rollout_id, timeout)`: 轮询已完成的rollout直到其可用或超时。
- `retrieve_completed_rollouts()`: 返回所有已完成的rollout并清除内部缓冲区。

**Section sources**
- [server.py](file://agentlightning/server.py#L220-L393)

#### Trainer
该类为连接Algorithm <-> Runner <-> Store的高级编排层。

**属性**:
- `algorithm`: 用于训练的`Algorithm`实例。
- `store`: 用于存储任务和跟踪的`LightningStore`实例。
- `runner`: 用于运行代理的`Runner`实例。
- `initial_resources`: 用于引导fit/dev过程的`NamedResources`实例。
- `n_runners`: 并行运行的代理运行器数量。
- `max_rollouts`: 每个运行器处理的最大rollout数量。
- `strategy`: 用于生成算法和运行器的`ExecutionStrategy`实例。
- `tracer`: 用于跟踪的`Tracer`实例。
- `hooks`: 在各种生命周期阶段调用的`Hook`实例序列。
- `adapter`: 用于将跟踪数据导出为算法可用格式的`TraceAdapter`实例。
- `llm_proxy`: 用于拦截LLM调用的`LLMProxy`实例。

**方法**:
- `__init__()`: 配置训练器并解析用户提供的组件规范。
- `fit(agent, train_dataset, val_dataset)`: 执行完整的算法/运行器训练循环。
- `dev(agent, train_dataset, val_dataset)`: 使用快速、同步的算法来调试基础设施。
- `_algorithm_bundle()`: 策略为算法角色执行的内部入口点。
- `_runner_bundle()`: 策略为每个运行器角色执行的内部入口点。

**Section sources**
- [trainer/trainer.py](file://agentlightning/trainer/trainer.py#L100-L556)

#### LightningStore
该类为协调训练rollout的持久控制平面的契约。

**方法**:
- `start_rollout(input, mode, resources_id, config, metadata)`: 注册一个rollout并立即创建其第一次尝试。
- `enqueue_rollout(input, mode, resources_id, config, metadata)`: 将rollout持久化在`queuing`状态，以便运行器稍后声明。
- `dequeue_rollout()`: 声明最旧的排队rollout并将其状态转换为`preparing`。
- `start_attempt(rollout_id)`: 为现有rollout创建手动重试尝试。
- `add_span(span)`: 持久化在rollout执行期间发出的预构建span。
- `add_otel_span(rollout_id, attempt_id, readable_span, sequence_id)`: 转换并持久化特定尝试的OpenTelemetry span。
- `query_rollouts(status, rollout_ids)`: 根据状态和/或显式标识符检索rollouts。
- `query_attempts(rollout_id)`: 返回为`rollout_id`创建的每次尝试，按升序排列。
- `get_rollout_by_id(rollout_id)`: 根据标识符获取rollout而不改变其状态。
- `get_latest_attempt(rollout_id)`: 获取`rollout_id`的最高`sequence_id`的尝试。
- `get_resources_by_id(resources_id)`: 根据标识符返回特定的命名资源快照。
- `get_latest_resources()`: 获取标记为全局默认的最新资源快照。
- `get_next_span_sequence_id(rollout_id, attempt_id)`: 分配用于对span排序的下一个严格递增的序列号。
- `wait_for_rollouts(rollout_ids, timeout)`: 阻塞直到目标rollouts达到终端状态或超时。
- `query_spans(rollout_id, attempt_id)`: 返回rollout的存储span，可选择限定于一个尝试。
- `add_resources(resources)`: 持久化新的不可变资源快照并将其标记为最新。
- `update_resources(resources_id, resources)`: 覆盖或扩展现有资源快照并将其标记为最新。
- `update_rollout(rollout_id, input, mode, resources_id, status, config, metadata)`: 更新rollout元数据并驱动状态转换。
- `update_attempt(rollout_id, attempt_id, status, worker_id, last_heartbeat_time, metadata)`: 更新尝试的簿记，如状态、工作器所有权和心跳。

**Section sources**
- [store/base.py](file://agentlightning/store/base.py#L100-L515)

#### LitAgent
该类为实现插入Agent Lightning的代理rollout的基类。

**方法**:
- `__init__(trained_agents)`: 初始化代理实例。
- `is_async()`: 当代理重写任何异步rollout方法时返回`True`。
- `set_trainer(trainer)`: 附加负责编排的训练器。
- `get_trainer()`: 返回与此代理关联的训练器。
- `get_tracer()`: 返回为此代理配置的跟踪器。
- `set_runner(runner)`: 附加负责执行rollout的运行器。
- `get_runner()`: 返回负责执行rollout的运行器。
- `on_rollout_start(task, runner, tracer)`: 在rollout开始前立即调用的钩子。
- `on_rollout_end(task, rollout, runner, tracer)`: 在rollout完成后立即调用的钩子。
- `rollout(task, resources, rollout)`: 同步执行rollout。
- `rollout_async(task, resources, rollout)`: 异步执行rollout。
- `training_rollout(task, resources, rollout)`: 同步处理单个训练任务。
- `validation_rollout(task, resources, rollout)`: 同步处理单个验证任务。
- `training_rollout_async(task, resources, rollout)`: 异步处理单个训练任务。
- `validation_rollout_async(task, resources, rollout)`: 异步处理单个验证任务。

**Section sources**
- [litagent/litagent.py](file://agentlightning/litagent/litagent.py#L40-L251)

### 参数与返回值

#### 类型注解说明
- `TaskInput`: 任意任务负载，接受任意有效载荷。
- `RolloutRawResult`: Rollout结果类型，可能的返回值包括`None`、`float`、`List[ReadableSpan]`或`List[Span]`。
- `RolloutStatus`: Rollout状态，包括`"queuing"`、`"preparing"`、`"running"`、`"failed"`、`"succeeded"`、`"cancelled"`或`"requeuing"`。
- `AttemptStatus`: 尝试状态，包括`"preparing"`、`"running"`、`"failed"`、`"succeeded"`、`"unresponsive"`或`"timeout"`。
- `RolloutMode`: Rollout模式，包括`"train"`、`"val"`或`"test"`。
- `NamedResources`: 命名资源的映射，将资源名称映射到其序列化有效载荷。
- `ResourcesUpdate`: 包含`resources_id`和`resources`的资源更新。
- `Dataset`: 通用数据集接口，具有类似`torch.utils.data.Dataset`的接口。

**Section sources**
- [types/core.py](file://agentlightning/types/core.py#L100-L395)
- [types/resources.py](file://agentlightning/types/resources.py)

## REST API

### HTTP方法与URL模式
该API提供以下端点：

- `GET /task`: 提供下一个可用任务给客户端。
- `GET /resources/latest`: 返回服务器公布的最新资源包。
- `GET /resources/{resource_id}`: 根据标识符返回特定版本的资源。
- `POST /rollout`: 持久化客户端报告的rollout。

### 请求/响应Schema
#### 任务获取
**请求**:
- 方法: `GET`
- URL: `/task`

**响应**:
```json
{
  "is_available": true,
  "task": {
    "rollout_id": "string",
    "input": "any",
    "mode": "train|val|test",
    "resources_id": "string",
    "create_time": "number",
    "last_claim_time": "number",
    "num_claims": "number",
    "metadata": {}
  }
}
```

#### 最新资源获取
**请求**:
- 方法: `GET`
- URL: `/resources/latest`

**响应**:
```json
{
  "resources_id": "string",
  "resources": {}
}
```

#### 特定资源获取
**请求**:
- 方法: `GET`
- URL: `/resources/{resource_id}`

**响应**:
```json
{
  "resources_id": "string",
  "resources": {}
}
```

#### Rollout提交
**请求**:
- 方法: `POST`
- URL: `/rollout`
- 内容类型: `application/json`

```json
{
  "rollout_id": "string",
  "task": {
    "rollout_id": "string",
    "input": "any",
    "mode": "train|val|test",
    "resources_id": "string",
    "create_time": "number",
    "last_claim_time": "number",
    "num_claims": "number",
    "metadata": {}
  },
  "final_reward": "number",
  "triplets": [
    {
      "prompt": "any",
      "response": "any",
      "reward": "number",
      "metadata": {}
    }
  ],
  "trace": [
    {}
  ],
  "logs": [
    "string"
  ],
  "metadata": {}
}
```

**响应**:
```json
{
  "status": "ok",
  "message": "string"
}
```

### 认证机制
该API未实现特定的认证机制，依赖于网络级别的安全措施。

### 错误码
- `404 Not Found`: 请求的资源不存在。
- `503 Service Unavailable`: 服务器未完全初始化。
- `400 Bad Request`: 请求格式不正确。

**Section sources**
- [server.py](file://agentlightning/server.py#L220-L393)

## CLI命令

### 可用命令
- `agl vllm`: 使用Agent Lightning检测运行vLLM CLI。
- `agl store`: 运行LightningStore服务器。
- `agl agentops`: 启动AgentOps服务器管理器。

### 选项参数
#### vllm命令
- 无特定选项，直接传递参数给vLLM。

#### store命令
- `--port`: 运行服务器的端口（默认: 4747）。

#### agentops命令
- `--daemon`: 作为守护进程运行服务器。
- `--port`: 运行服务器的端口（默认: 8002）。

### 使用示例
```bash
# 运行vLLM CLI
agl vllm --model meta-llama/Llama-2-7b-chat-hf

# 运行LightningStore服务器
agl store --port 8080

# 启动AgentOps服务器
agl agentops --daemon --port 8002
```

**Section sources**
- [cli/__init__.py](file://agentlightning/cli/__init__.py#L1-L55)
- [cli/vllm.py](file://agentlightning/cli/vllm.py#L1-L29)
- [cli/store.py](file://agentlightning/cli/store.py#L1-L37)
- [cli/agentops_server.py](file://agentlightning/cli/agentops_server.py#L1-L30)

## 装饰器接口

### llm_rollout装饰器
创建一个`FunctionalLitAgent`用于基于LLM的rollout。

**语法**:
```python
@llm_rollout
def my_agent(task, llm):
    return llm.endpoint

@llm_rollout(strip_proxy=False)
def my_agent_no_strip(task, llm):
    return llm.model
```

**参数配置**:
- `func`: 定义代理行为的可调用对象。支持的签名包括:
  - `(task, llm) -> result`
  - `(task, llm, rollout) -> result`
  - `async (task, llm) -> result`
  - `async (task, llm, rollout) -> result`
- `strip_proxy`: 当`True`时，在调用函数前将代理资源转换为具体的`LLM`实例。默认为`True`。

**执行上下文**: 该装饰器检查包装函数以确定注入哪些资源，允许同步和异步可调用对象参与训练循环而无需编写专用子类。

**Section sources**
- [litagent/decorator.py](file://agentlightning/litagent/decorator.py#L100-L300)

### prompt_rollout装饰器
创建一个`FunctionalLitAgent`用于基于提示模板的rollout。

**语法**:
```python
@prompt_rollout
def my_agent(task, prompt_template):
    messages = prompt_template.format(task=task.input)
    return messages
```

**参数配置**:
- `func`: 定义代理行为的可调用对象。支持的签名包括:
  - `(task, prompt_template) -> result`
  - `(task, prompt_template, rollout) -> result`
  - `async (task, prompt_template) -> result`
  - `async (task, prompt_template, rollout) -> result`

**执行上下文**: 该装饰器专为使用可调提示模板的代理设计，允许算法管理和优化提示模板，而代理消耗模板以执行rollout。

**Section sources**
- [litagent/decorator.py](file://agentlightning/litagent/decorator.py#L300-L400)

### rollout装饰器
从任意rollout函数创建一个`FunctionalLitAgent`。

**语法**:
```python
@rollout
def my_llm_agent(task, llm):
    client = OpenAI(base_url=llm.endpoint)
    response = client.chat.completions.create(
        model=llm.model,
        messages=[{"role": "user", "content": task.input}],
    )
    return response
```

**参数配置**:
- `func`: 实现rollout的可调用对象。支持的签名:
  - `[async ](task, llm[, rollout])` 用于基于LLM的代理
  - `[async ](task, prompt_template[, rollout])` 用于基于提示模板的代理

**执行上下文**: 该函数检查提供的可调用对象并根据其签名创建适当的代理类型，支持基于LLM和基于提示模板的代理。

**Section sources**
- [litagent/decorator.py](file://agentlightning/litagent/decorator.py#L400-L536)

### algo装饰器
将可调用对象转换为`FunctionalAlgorithm`。

**语法**:
```python
@algo
def batching_algorithm(*, store, train_dataset, val_dataset):
    for sample in train_dataset:
        store.enqueue_rollout(input=sample, mode="train")

@algo
async def async_algorithm(*, store, train_dataset=None, val_dataset=None):
    await store.enqueue_rollout(input={"prompt": "hello"}, mode="train")
```

**参数配置**:
- `func`: 实现算法逻辑的函数。可以是同步或异步的。函数可以期望以下参数的全部或子集:
  - `store`: `LightningStore`
  - `train_dataset`: `Dataset`
  - `val_dataset`: `Dataset`
  - `llm_proxy`: `LLMProxy`
  - `adapter`: `TraceAdapter`
  - `initial_resources`: `NamedResources`

**执行上下文**: 该装饰器检查可调用对象的签名以决定在运行时注入哪些依赖项，从而实现简洁的算法定义，同时仍利用完整的训练运行时。

**Section sources**
- [algorithm/decorator.py](file://agentlightning/algorithm/decorator.py#L100-L264)

## 跨API调用示例

### Python API与CLI集成
```python
from agentlightning.trainer import Trainer
from agentlightning.litagent.decorator import llm_rollout
from agentlightning.store.memory import InMemoryLightningStore

@llm_rollout
def simple_agent(task, llm):
    return f"Response to {task.input} using {llm.model}"

# 使用Python API创建训练器
trainer = Trainer(
    store=InMemoryLightningStore(),
    n_runners=2,
    max_rollouts=10
)

# 通过CLI启动存储服务器
# agl store --port 4747
```

### REST API与Python API交互
```python
import requests
from agentlightning.client import AgentLightningClient

# 使用Python API创建客户端
client = AgentLightningClient(endpoint="http://localhost:8000")

# 通过REST API轮询任务
task = client.poll_next_task()
if task:
    # 处理任务
    result = simple_agent.rollout(task, resources, rollout)
    # 通过REST API提交结果
    client.post_rollout(result)
```

**Section sources**
- [client.py](file://agentlightning/client.py#L45-L374)
- [server.py](file://agentlightning/server.py#L220-L393)
- [trainer/trainer.py](file://agentlightning/trainer/trainer.py#L100-L556)

## 版本兼容性说明

### 已弃用的API
以下API已被弃用，建议使用新的替代方案：

- `AgentLightningClient`: 已弃用，建议使用`LightningStoreClient`。
- `AgentLightningServer`: 已弃用，建议使用`LightningStoreServer`。
- `DevTaskLoader`: 已弃用，建议使用`Trainer.dev`。
- `Task`: 旧版HTTP客户端/服务器堆栈仍在使用此模型，建议为新工作流使用`LightningStore` API。
- `RolloutLegacy`: 已弃用，建议使用`Rollout`。
- `RolloutRawResultLegacy`: 已弃用，建议使用`RolloutRawResult`。
- `GenericResponse`: 不再用于新的`LightningStore` API。

### 向后兼容性
旧版HTTP服务器和客户端保持向后兼容性，以便与现有部署兼容。新应用程序应迁移到基于存储的架构。

### 迁移指南
- 从`AgentLightningClient`迁移到`LightningStore` API。
- 从`AgentLightningServer`迁移到`LightningStoreServer`。
- 从`DevTaskLoader`迁移到`Trainer.dev`方法。
- 从`Task`和`RolloutLegacy`模型迁移到`Rollout`和`Attempt`模型。

**Section sources**
- [client.py](file://agentlightning/client.py#L45-L374)
- [server.py](file://agentlightning/server.py#L220-L393)
- [types/core.py](file://agentlightning/types/core.py#L100-L395)
- [store/base.py](file://agentlightning/store/base.py#L100-L515)