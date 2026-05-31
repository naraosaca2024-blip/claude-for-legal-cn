---
name: log-leave
description: >
  向请假登记册添加新假期条目，包含跟踪期限所需的最少信息。当员工开始休假且你希望跟踪器从第一天起监控指定、认证和耗尽时钟时使用。
argument-hint: "[描述假期——员工/角色、类型、司法管辖区、开始日期]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /log-leave

向 `~/.claude/plugins/config/claude-for-legal/employment-legal/leave-register.yaml` 添加新假期条目，包含开始跟踪期限所需的最少信息。当员工开始休假且你希望跟踪器从第一天起监控时钟时使用。

## 说明

1. 读取 `~/.claude/plugins/config/claude-for-legal/employment-legal/CLAUDE.md` → 司法管辖区表和系统部分。

2. 在单个提示中询问以下所有内容——不要逐个询问：

   > 几个快速问题以设置假期跟踪：
   >
   > - 员工姓名或角色（匿名化即可）
   > - 他们在哪里工作？（州——这决定适用哪些规则）
   > - 假期类型：FMLA / 州假期（哪个州）/ USERRA / ADA 住宿
   > - 假期开始日期
   > - 是否为间歇性假期？
   > - 预期返回日期（如果已知——不知道则留空）
   > - 指定通知是否已发送？如果是，何时？
   > - 医疗认证是否已请求？如果是，何时？

3. 使用 `~/.claude/plugins/config/claude-for-legal/employment-legal/CLAUDE.md` 中的司法管辖区表，查找此假期类型在此司法管辖区的适用假期权益（小时/周）。

4. 根据提供的信息计算第一个即将到来的截止日期：
   - 指定尚未发送 → 截止日期为假期开始后 5 个工作日
   - 医疗认证已请求但未收到 → 截止日期为请求日期后 15 天
   - 两者均已发送和收到 → 下一个截止日期为 75% 耗尽

5. 使用请假跟踪器代理的请假登记册格式向 `~/.claude/plugins/config/claude-for-legal/employment-legal/leave-register.yaml` 写入新条目。如果文件不存在，创建它。

6. 用一行确认：
   > "已记录。[员工/角色] — [假期类型] — [司法管辖区] — [日期] 开始。第一个截止日期：[是什么及何时]。请假跟踪器将自动提醒。"

## 示例

```
/employment-legal:log-leave
```

```
/employment-legal:log-leave
Sarah（高级工程师，在加州工作）今天因严重健康状况开始 FMLA。间歇性。尚未发送指定通知。
```
