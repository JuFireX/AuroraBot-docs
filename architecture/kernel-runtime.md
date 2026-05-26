---
title: 内核运行时
description: Circuit + EventBridge 的实际运行机制——启动、调度、文件落盘与节点执行循环。
order: 3
---

# 内核运行时

本文描述 `src/main.py` 启动后认知引擎的运作方式。

## 启动

`main.py` 的 `startup_agent()` 按以下顺序初始化：

1. `Config.ensure_dirs()` — 创建 `data/`、`data/kernel/` 等目录
2. `load_apps_config()` — 读 `apps/config.yaml`
3. 遍历启用的 App，调用 `app_host.register()`（读 `manifest.yaml`、注册命令、调 `on_start()`）
4. `build_circuit(app_host)` — 读 `topology.yaml`，通过 `NODE_REGISTRY` 实例化节点
5. `circuit.start()` — 启动 `dispatch_forever` 协程 + 各个 `node.run()` 协程
6. `circuit._bootstrap_heartbeat()` — 写入首个 `heartbeat/tick.json`，注入初始 `FileEvent`
7. 根据 `RUN_MODE` 创建 `run_app_loop` 和 `run_event_bridge` 协程

## 运行时协程

`main.py` 通过 `asyncio.create_task` 创建两个顶层 `asyncio.Task`，`Circuit` 内部另有一组协程：

### ① App 循环 — `run_app_loop()`

`src/platform/loop.py`

```python
while not stop_event.is_set():
    await host.tick()     # 遍历所有 App，调用 on_tick()
    await asyncio.sleep(interval)
```

App 在 `on_tick()` 中感知外部变化（如 QQ 的 `on_message`），通过 `PlatformAPI.emit_event()` 将 `AppEvent` 推入 `ApplicationHost._events` 双端队列。

仅在 `RUN_MODE` 为 `app` / `application` / `prod` 时启动。

### ② 事件桥 — `run_event_bridge()`

`src/brain/nodes/event_bridge.py`

```python
while not stop_event.is_set():
    events = host.drain_events()
    for event in events:
        file_path = f"inbox/pending/event_{type}_{id}.json"
        circuit.apply_update(FileUpdate(...), node_id="event_bridge")
    await asyncio.sleep(interval)
```

`apply_update()` 由 `FileEventBus` 执行：写文件 → 生成 `FileEvent` → `publish` 入队列。

仅在 `RUN_MODE` 为 `agent` / `core` / `prod` 时启动。

### ③ 认知电路 — `Circuit` + `FileEventBus`

`src/brain/kernel/circuit.py` + `src/brain/kernel/event_bus.py`

启动流程：

1. `FileEventBus(nodes)` 创建事件总线和 `asyncio.Queue`
2. `dispatch_forever()` 协程启动，持续从队列取事件
3. 每个节点创建 `node.run()` 协程，等待 `_ready_event` 被置位
4. `_bootstrap_heartbeat()` 注入初始脉冲

运行时每个周期：

```
FileEvent → dispatch_forever() 从队列取出
  → 遍历 nodes → on_event() 匹配?
      → 是 → state = READY → _ready_event.set()
              → node.run() 被唤醒 → execute() → [FileUpdate, ...]
                  → apply_update(update) 落盘 → publish 新 FileEvent → 回到顶部
      → 否 → 继续等待下一事件
```

## 节点生命周期

```
IDLE ──on_event() 匹配──▶ READY ──被 run() 唤醒──▶ RUNNING ──execute() 完成──▶ IDLE
                            │                       │
                            └──TERMINATED◀──────────┘ (Circuit.stop())
```

Agent 在执行期间可能长时间等待 LLM 响应，期间状态保持 `RUNNING`。Router 执行时间可预测，通常毫秒级完成。

## 文件写入与锁

`FileEventBus.apply_update()` 为每个文件路径维护一个 `asyncio.Lock`：

```python
async def apply_update(self, update, node_id):
    lock = self._get_lock(descriptor.path)
    async with lock:
        self._write_file(...)
    self.publish(FileEvent(path=descriptor.path, change_type="write"))
```

同一文件被多个节点并发写入时，`asyncio.Lock` 保证串行化。文件格式支持 `"json"` 和普通文本，json 模式下可选 `"append"` 模式（向数组追加元素）。

## 关闭

`shutdown_agent()` 按顺序：

1. `_stop_event.set()` — 通知所有协程停止
2. 取消 `_bridge_task`（事件桥）
3. `circuit.stop()` — 将所有节点标记 `TERMINATED`，取消各 `node.run()` 协程和 `dispatch_forever`
4. 取消 `_app_task`（App 循环）
5. `app_host.stop_all()` — 调用所有 App 的 `on_stop()`
