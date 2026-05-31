---
name: investigation-open
description: >
  打开新的内部调查事项——运行接收、生成来源检查清单、创建持久调查日志。当投诉或指控出现且律师需要建立特权调查工作空间时使用。
argument-hint: "[指控的简要描述]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /investigation-open

打开新的调查事项——运行接收、生成来源检查清单、创建持久调查日志。

## 说明

1. 加载 `~/.claude/plugins/config/claude-for-legal/employment-legal/CLAUDE.md`。
2. 加载 `internal-investigation` 参考 skill 并运行模式 1（打开）。
3. 如果具有相同 slug 的事项已存在，在覆盖前发出警告。

## 示例

```
/employment-legal:investigation-open
Harassment complaint filed against a manager in the Austin office.
```

```
/employment-legal:investigation-open
(skill 将询问详情)
```

> 详细的接收、特权形成要求、来源检查清单和日志模板位于 `internal-investigation` 参考 skill 中——在进行实质性工作之前加载它。
