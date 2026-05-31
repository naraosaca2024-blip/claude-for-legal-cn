---
name: expansion-kickoff
description: >
  为新国家启动国际扩张规划——收集接收信息、运行 EOR vs. 实体框架、起草跨职能问题、提取国家特定标志、创建持久追踪器。当有人说"我们要在[国家]招人"、"扩张到[国家]"或"在[国家]的第一位员工"时使用。
argument-hint: "[国家名称]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /expansion-kickoff

为新国家启动国际扩张项目——收集接收信息、运行 EOR vs. 实体框架、起草跨职能问题、提取国家特定标志、创建持久追踪器。

## 说明

1. 加载 `~/.claude/plugins/config/claude-for-legal/employment-legal/CLAUDE.md` → 司法管辖区足迹、升级表。
2. 加载 `international-expansion` 参考 skill 并运行完整工作流。
3. 如果此国家的追踪器文件已存在（`~/.claude/plugins/config/claude-for-legal/employment-legal/expansion-[slug].yaml`），
   标记它："[国家] 的扩张追踪器已存在。使用
   `/employment-legal:expansion-update [国家]` 来更新它，或确认你想重新开始。"
4. 完成后创建 `~/.claude/plugins/config/claude-for-legal/employment-legal/expansion-[slug].yaml`。

## 示例

```
/employment-legal:expansion-kickoff Germany
```

```
/employment-legal:expansion-kickoff
(skill 将询问哪个国家)
```

> 详细的 EOR vs. 实体框架、跨职能问题、简报模板和追踪器架构位于 `international-expansion` 参考 skill 中——在进行实质性工作之前加载它。
