---
name: gaps
description: 开放差距跟踪器——已标记但尚未关闭的内容。当用户问"有哪些开放差距"、"差距跟踪器"、"整改状态"或想要关闭 (--close GAP-ID) 或风险接受 (--accept GAP-ID) 跟踪的差距时使用。
argument-hint: "[可选：--close GAP-ID | --accept GAP-ID]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /gaps

1. 阅读 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/gap-tracker.yaml` 的差距跟踪器。
2. 如果 `--close`：标记差距已关闭并附解决方案注释。
3. 如果 `--accept`：记录风险接受理由和接受者，状态 → risk-accepted。
4. 否则：按年龄和重要性报告开放差距。

> 详细的跟踪器 schema、状态报告格式、所有者通知逻辑（每次发送确认，无例外）、提醒节奏、关闭/风险接受模式以及后果行动关卡位于 **gap-surfacer** 参考 skill 中——在进行实质性工作之前加载它。
