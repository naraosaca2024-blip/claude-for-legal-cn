---
name: investigation-memo
description: >
  从调查日志起草或更新特权调查备忘录。当调查进展到足以撰写首次备忘录草稿，或新数据已添加且现有草稿需要更新时使用。
argument-hint: "[事项名称]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /investigation-memo

从日志起草特权调查备忘录的首次草稿，或在添加新数据时更新现有草稿。

## 说明

1. 加载 `internal-investigation` 参考 skill 并运行模式 4（起草或更新备忘录）。
2. 如果是首次起草，当高优先级来源在检查清单中仍然开放时发出警告。
3. 如果是更新，在重写之前展示变更内容。
4. 所有输出标记为 PRIVILEGED AND CONFIDENTIAL — ATTORNEY WORK PRODUCT。

## 示例

```
/employment-legal:investigation-memo [事项名称]
```

```
/employment-legal:investigation-memo [事项名称]
(如果备忘录已存在则更新)
```

> 详细的备忘录结构、可信度评估框架和更新规则位于 `internal-investigation` 参考 skill 中——在进行实质性工作之前加载它。
