---
layout: home

hero:
  name: AuroraBot
  text: 新一代内驱式、自主决策的智能体框架
  tagline: 声明式认知拓扑 · 三级联合记忆 · 统一 LLM 网关
  image:
    src: /logo.svg
    alt: AuroraBot Logo
  actions:
    - theme: brand
      text: 快速开始
      link: /start/getting-started.html
    - theme: alt
      text: 架构总览
      link: /architecture/system-overview.html
    - theme: alt
      text: 应用开发
      link: /develop/app-development.html

features:
  - icon: 🧠
    title: 文件驱动认知管道
    details: Node / Agent / Router 节点通过事件总线协作，topology.yaml 声明式配置邻接关系，文件落盘自动触发下游节点。
  - icon: 📔
    title: 三级联合记忆
    details: L1 工作记忆 / L2 情景记忆 / L3 语义记忆，通过 UnifiedMemoryManager 统一写入与检索。
  - icon: 🤖
    title: 统一 LLM 网关
    details: 基于 litellm，所有 LLM 调用走 llm_chat() 统一入口，支持重试与错误分级。
  - icon: 🧩
    title: App 插件体系
    details: 每个 App 通过 manifest.yaml 声明能力，PlatformAPI 统一交互，按需启用。
  - icon: 📝
    title: 声明式配置
    details: 认知拓扑 / 应用配置 / 系统参数均通过 YAML 和环境变量管理，无需修改框架代码。
  - icon: 📖
    title: 文档驱动开发
    details: 随提交更新的项目文档站，保证文档与代码同步更新。
---
