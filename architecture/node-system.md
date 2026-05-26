---
title: 节点系统
description: Node / Agent / Router 的数据结构与行为——FileDescriptor、FilePattern、FileEvent、FileUpdate。
order: 5
---

# 节点系统

认知层的认知能力由节点承载。节点是文件驱动认知管道中的工作单元。代码实现见 `src/brain/kernel/base.py`。

**挼挼如是说**

> Agent 用 LLM 干活，Router 不用——就这么简单。Router 快、便宜、可预测；Agent 慢、贵、但有思考能力。大多数时候 Router 就够了，只有需要"理解"的时候才上 Agent。

## 类的层次

```
Node (ABC)             # 抽象基类
├── Agent(Node)        # LLM 驱动型
└── Router(Node)       # 纯逻辑型
```

## Node 基类

```python
class Node:
    id: str                          # 唯一标识
    guards: List[FilePattern]        # 监听的 glob 模式列表
    produces: List[FileDescriptor]   # 产出的文件描述符列表

    def on_event(event) -> bool      # 事件匹配判断
    async def execute() -> list[FileUpdate]  # 执行逻辑
    async def run()                  # 主循环
```

`Node.run()` 在被 `Circuit` 以 `asyncio.Task` 托管后循环等待 `_ready_event`，被总线置位后调用 `execute()`。

## Agent

```python
class Agent(Node):
    def __init__(self, node_id, host=None, *, system_prompt="", memory=None):
        ...
```

持有 `_host`（ApplicationHost 引用）、`_system_prompt`（注入 LLM 的消息）、`_memory`（UnifiedMemoryManager 引用）。

Agent 在执行时通常调用 `llm_chat()` 函数（`src/brain/ai/llm_gate.py`）——项目内所有 LLM 调用的唯一入口。

## Router

```python
class Router(Node):
    ...
```

只是标记了 `type = "router"`。Router 的实现类不调用 LLM，执行纯机械逻辑。

## 文件相关数据结构

### FileDescriptor

```python
@dataclass(slots=True)
class FileDescriptor:
    path: str                    # 文件路径
    schema: str = "json"         # 格式（json / text）
    lock: str = "write_overwrite"# 锁策略
```

### FilePattern

```python
@dataclass(slots=True)
class FilePattern:
    pattern: str    # glob 模式，如 "inbox/event_*.json"

    def match(self, file_path: str) -> bool
```

### FileEvent

```python
@dataclass(slots=True)
class FileEvent:
    path: str                # 变更文件路径
    change_type: str         # 变更类型
    timestamp: str           # 时间戳
    version: int = 0
    metadata: dict = {}      # 含 source_node 等
```

### FileUpdate

```python
@dataclass(slots=True)
class FileUpdate:
    descriptor: FileDescriptor
    content: Any
    mode: str = "overwrite"   # overwrite / append
```

## 节点状态机

```
IDLE → READY → RUNNING → IDLE
                 ↓
               ERROR
```

状态定义在 `NodeState` 枚举中：`IDLE`、`READY`、`RUNNING`、`WAITING`、`ERROR`、`TERMINATED`。`WAITING` 状态已在 `EventBridge` 和节点基类中使用（用于慢路径主动等待）。

## 已实现的节点列表

`src/brain/nodes/agents/`：

| 文件                      | 注册名           | 职责                     |
| ------------------------- | ---------------- | ------------------------ |
| `plan_agent.py`           | `planner`        | LLM 将事件组整合为计划   |
| `expand_agent.py`         | `expander`       | LLM 将计划展开为命令调用 |
| `execute_agent.py`        | `executor`       | 调用命令并 LLM 判定结果  |
| `goal_generator_agent.py` | `goal_generator` | 沉默时主动生成意图       |
| `reflex_learner_agent.py` | `reflex_learner` | 从成功动作中提取规则     |

`src/brain/nodes/routers/`：

| 文件                  | 注册名      | 职责                      |
| --------------------- | ----------- | ------------------------- |
| `fanout_router.py`    | `fanout`    | 扇出事件到多个下游        |
| `reflex_router.py`    | `reflex`    | 规则匹配，直接产出 action |
| `switch_router.py`    | `switch`    | 条件分支                  |
| `merge_router.py`     | `merge`     | 归并多个文件              |
| `heartbeat_router.py` | `heartbeat` | 定时自触发脉冲            |
| `terminal_router.py`  | `terminal`  | 关闭子图、移入归档        |
| `memory_router.py`    | `memory`    | 写入三级记忆              |

## 设计约束

- Agent 不直接调用 LLM API——走 `llm_chat()` 统一入口
- 节点无内部状态（LLM 上下文除外），`execute()` 后可回收
- 文件生命周期：`pending/` → `done/` → `archived/`

## 下一步阅读

- 想看完整的认知拓扑图：读 [认知架构](./brain-architecture.html)
- 想理解调度机制：读 [内核运行时](./kernel-runtime.html)
