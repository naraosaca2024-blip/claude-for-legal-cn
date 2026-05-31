---
name: investigation-add
description: >
  向开放调查添加数据——文档、访谈笔记或观察。根据记录的提取标准处理批次，提取重要项目，并记录所有已审查内容用于覆盖验证。当新证据、访谈笔记或文档制作送达开放调查时使用。
argument-hint: "[事项名称或 slug，然后粘贴或附加数据]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /investigation-add

向开放调查日志添加数据。使用记录的提取标准处理文档批次，提取重要项目，记录所有已审查内容用于覆盖验证。

## 说明

1. 加载 `~/.claude/plugins/config/claude-for-legal/employment-legal/CLAUDE.md`。
2. 加载 `internal-investigation` 参考 skill 并运行模式 2（添加数据）。
3. 处理后，显示提取比率和提取项目列表。
4. 如果数据覆盖检查清单项目，提示更新来源检查清单。

## 示例

```
/employment-legal:investigation-add [事项名称]
[粘贴访谈笔记]
```

```
/employment-legal:investigation-add [事项名称]
[附加电子邮件导出]
```

> 详细的提取流程、日志条目格式、提取比率规则和来源检查清单跟踪位于 `internal-investigation` 参考 skill 中——在进行实质性工作之前加载它。
