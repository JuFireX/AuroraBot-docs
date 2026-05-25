---
title: 快速开始
description: 从环境准备到启动运行，快速把 AuroraBot 跑起来。
order: 2
---

# 快速开始

从环境准备到启动运行，快速把 AuroraBot 跑起来。

::: info
此版本暂时只支持从源码运行. 后期会提供一键包安装.
:::

## 前期准备

- `Python >=3.12, <3.13`

## 克隆仓库

```bash
git clone https://github.com/AuroraBot-Dev/AuroraBot.git
cd AuroraBot
```

::: tip
或者你可以通过 [Releases](https://github.com/AuroraBot-Dev/AuroraBot/releases) 下载最新稳定版本的源码压缩包, 并解压到 `AuroraBot` 目录下.
:::

## 安装依赖

我们推荐使用 [uv](https://github.com/astral-sh/uv) 管理依赖:

```bash
pip install uv
uv sync
```

::: tip
如果你的网络环境不好导致 `pip` 下载缓慢, 你可以尝试使用以下命令来加速下载:

```bash
pip install uv -i https://pypi.tuna.tsinghua.edu.cn/simple
uv sync
```

:::

## 配置密钥

```bash
cp .env.example .env
```

::: tip
在 `.env` 中配置你的密钥:

```
# 适配器配置
ONEBOT_ACCESS_TOKEN=

# 模型配置
DEEPSEEK_URL_BASE=https://api.deepseek.com
DEEPSEEK_API_KEY=
LITELLM_MODEL=deepseek/deepseek-v4-flash

# 记忆配置
MEM0_VECTOR_STORE=chroma
MEM0_COLLECTION_NAME=aurora_memory_bgem3

MEM0_EMBEDDER_PROVIDER=openai
MEM0_EMBEDDER_API_KEY=
MEM0_EMBEDDER_BASE_URL=https://api.siliconflow.cn/v1/
MEM0_EMBEDDER_MODEL=BAAI/bge-m3

MEM0_LLM_PROVIDER=deepseek
MEM0_LLM_API_KEY=
MEM0_LLM_BASE_URL=https://api.deepseek.com
MEM0_LLM_MODEL=deepseek-v4-flash
```

更多配置说明见 [配置说明](./configuration)
:::

## 启动Bot

```bash
uv run bot.py
```

::: tip
此时你的Bot将会以默认人格**小光**启动，但是你还没有手段来与她互动. 你可以启动你的应用适配器，例如 [NapCat](https://github.com/NapNeko/NapCatQQ), 然后就可以试着向她发送消息了!
:::

::: tip
由于AuroraBot 是一个基于 NoneBot2 框架的再封装框架, 所以你可以参考 [NapCat 官方文档](https://napneko.github.io/use/integration#nonebot) 来对接你的 NapCat 适配器.
:::

::: info
框架第一适配 NapCat 适配器. 其他适配器将在后续测试后逐渐开放.
:::
