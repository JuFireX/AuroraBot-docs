---
title: 关于适配器
description: NapCat、NoneBot2 适配器与协议端的部署与兼容性说明。
order: 3
---

# 关于适配器

AuroraBot 基于 NoneBot2，当前通过 NapCat + OneBot v11 协议连接 QQ。

## NapCat

[NapCatQQ](https://github.com/NapNeko/NapCatQQ) 是基于 NTQQ 的 Bot 协议端实现，运行环境支持：

| 部署方式       | 适用场景      | 说明                                                 |
| -------------- | ------------- | ---------------------------------------------------- |
| Windows 原生   | 桌面 / 笔记本 | 直接运行 exe，支持 LiteLoader 插件方式               |
| Linux AppImage | 物理机 / VPS  | 单文件运行，无需安装依赖                             |
| Linux 原生     | 物理机 / VPS  | 放入 QQ 安装目录运行                                 |
| Docker         | 服务器 (推荐) | 支持无头模式，内存占用极低，纯服务器环境无需桌面运行 |
| macOS          | Mac 用户      | 原生支持                                             |
| Android Termux | 移动端        | 在安卓终端中运行                                     |

::: tip 无头模式
NapCat 的**无头模式** (`headless`) 不需要启动 QQ 图形界面即可在服务器上运行，资源占用极低，适合 Linux 服务器部署。
:::

协议层面，NapCat 实现了 OneBot v11 标准协议。

## NoneBot2 适配器

NoneBot2 采用适配器模式：框架核心平台无关，通过各种适配器连接不同平台。

| 适配器                       | 对应平台                              | 成熟度   |
| ---------------------------- | ------------------------------------- | -------- |
| `nonebot-adapter-onebot` v11 | QQ (NapCat / go-cqhttp / Shamrock 等) | 非常成熟 |
| `nonebot-adapter-onebot` v12 | 多平台统一标准                        | 稳定     |
| `nonebot-adapter-telegram`   | Telegram                              | 成熟     |
| `nonebot-adapter-discord`    | Discord                               | 成熟     |
| `nonebot-adapter-qq`         | QQ 官方 Bot API                       | 稳定     |
| `nonebot-adapter-qqguild`    | QQ 频道                               | 稳定     |
| `nonebot-adapter-feishu`     | 飞书                                  | 稳定     |
| `nonebot-adapter-dingtalk`   | 钉钉                                  | 稳定     |
| `nonebot-adapter-kaiheila`   | KOOK (开黑啦)                         | 稳定     |
| `nonebot-adapter-red`        | Red 协议                              | 社区维护 |

目前 NoneBot2 生态已有 **20+ 平台适配器** 和 **500+ 社区插件**。

NoneBot2 还提供多种通信驱动：`FastAPI`、`HTTPX`、`WebSocket` 等，AuroraBot 当前使用 `~fastapi` 驱动。

## 当前状态

AuroraBot 目前仅加载 `nonebot-adapter-onebot`。虽然 NoneBot2 支持 20+ 平台，但 AuroraBot 尚未为其他平台编写对应的 App，也未加载其他适配器。

| 组件        | 能力                             | 当前使用          |
| ----------- | -------------------------------- | ----------------- |
| NapCat      | Windows / Linux / macOS / Docker | ✅ 推荐协议端     |
| NoneBot2    | 20+ 平台适配器                   | 仅加载 onebot v11 |
| OneBot 协议 | 多种实现可互换                   | NapCat            |
