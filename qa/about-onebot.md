---
title: 关于OneBot协议
description: 本文简单阐述OneBot协议的基本概念
order: 4
---

# 关于OneBot协议

## OneBot 协议是什么?

OneBot 是一套**统一的聊天机器人接口标准**, 它的核心思想是:

```
你的框架  <-->  OneBot 协议  <-->  协议端实现 (NapCat 等)  <-->  QQ
```

同一个协议可以被多种协议端实现, 例如 OneBot v11 的实现就有 NapCat、go-cqhttp、Shamrock、LLOneBot 等。这意味着即使 NapCat 挂了, 只要换个协议端就能继续工作, 不会被单一实现绑架。

---

具体协议内容请参考[OneBot v11 协议](https://onebot.dev/)。
