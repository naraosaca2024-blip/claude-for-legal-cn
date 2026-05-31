---
name: leave-tracker
description: >
  检查未结束假期的截止日期警报和所需决策。仅提示需要行动的假期并解释原因——不是状态板。每周使用，或当律师需要知道哪些假期有即将到来的指定、认证或耗尽截止日期时使用。
argument-hint: "[no arguments — works from HRIS or leave-register.yaml]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /leave-tracker

检查所有有硬性法律期限的未结束假期，仅提示需要决策或行动的项目。这不是状态板——而是告诉你需要做什么以及为什么。

## 说明

1. 加载 `leave-tracker` 代理并运行完整工作流。

2. 如果未连接 HRIS 且不存在 `~/.claude/plugins/config/claude-for-legal/employment-legal/leave-register.yaml`，提示律师上传假期电子表格或使用 `/employment-legal:log-leave` 添加条目。

3. 仅对需要行动的假期发出警报。无需处理的假期每行汇总一条。

## 示例

```
/employment-legal:leave-tracker
```

每周运行一次——设置周一早晨提醒来调用 `/employment-legal:leave-tracker`。自动化调度需要单独的集成（日历提醒、cron 作业等）；Claude Code 代理不会自动调度。
