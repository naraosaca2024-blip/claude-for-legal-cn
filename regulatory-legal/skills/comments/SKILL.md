---
name: comments
description: 审查开放的 NPRM 评论期，记录决定，跟踪截止期限。当 NPRM 有开放的评论窗口且您需要 surfaced截止期限、决定是否提交，或记录提交/不提交/放弃决定 (--decide CMT-ID) 时使用。
argument-hint: "[可选：--decide CMT-ID]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /comments

## 目的

NPRM 有截止期限。提交或不提交评论的决定是律师的呼叫——但截止期限在没有记录决定的情况下消失是风险。此 skill surfaced 开放的评论期并记录决定。

## 加载上下文

`~/.claude/plugins/config/claude-for-legal/regulatory-legal/comment-tracker.yaml` → 所有跟踪的 NPRM 及其状态。
`~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` → 默认评论决定所有者。

## 默认视图——开放评论期

```markdown
## Comment Period Tracker — [日期]

### ⏰ <14 天内截止

| ID | 法规 | 截止期限 | 剩余天数 | 决定 | 所有者 |
|---|---|---|---|---|---|
| CMT-001 | [名称] | [日期] | [N] | 未决定 | [所有者] |

### 🟡 开放 (>14 天)

[相同表格]

### 最近决定

| ID | 法规 | 决定 | 理由 |
|---|---|---|---|
| CMT-002 | [名称] | 不提交 | [理由] |

---

**总计开放：** [N]  **<30 天内未决定：** [N]
```

## 记录决定

```
/regulatory-legal:comments --decide CMT-001
决定：[filing / not-filing / waived]
理由："[简短——例如，'Rule doesn't apply to our model' 或 'Filing comment on Section 3']"
```

更新跟踪器。如果决定是"filing"：提示提交截止提醒（评论截止减去 5 个工作日用于内部审查）。

## 通知

首次检测 NPRM 时（由 reg-feed-watcher 填充）：如果配置了 Slack MCP 且设置了 `owner_slack`，则向评论决定所有者发送 Slack DM。

如果决定仍为"未决定"，则在截止前 14 天提醒。
如果在截止前 3 天仍未决定——提升紧迫性。

## 后果行动关卡（提交监管评论 / 回应监管机构）

**在将决定记录为"filing"之前——并且始终在生产提交的评论信或监管机构回应草案之前：**阅读 ~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md 中的 `## Who's using this`。如果角色是 **Non-lawyer**：

> 向监管机构提交评论或回应有法律后果。这是公司立场的公开声明，它在规则制定或执法事项中记录在案，此处采取的立场约束公司，并可在后续程序中对其使用。您是否已与律师审查此内容？如果是，继续。如果否，这是带给他们的简报：
>
> - 规则制定或询问（监管机构、docket、截止期限）
> - 拟议的评论/回应说什么以及对哪些部分
> - 未解决的问题和未解决的问题
> - 可能出什么问题（不利承认、与先前立场不一致、与贸易协会的评论协调问题）
> - 向律师问什么（我们应该提交吗；我们应该通过贸易集团联合提交；有哪些我们不应该采取的立场）
>
> 如果您需要找到律师：您专业监管机构的转介服务是最快的起点（美国的州律师协会；英格兰和威尔士的 SRA/律师标准委员会；苏格兰/北爱尔兰/爱尔兰/加拿大/澳大利亚的律师协会；或您所在司法管辖区的同等机构）。

在没有明确是的情况下，不要在此关卡之后记录"filing"决定或产生提交就绪的草案。跟踪视图、截止提醒和"不提交/放弃"决定不需要关卡。

---

## 此 skill 不做什么

- 起草评论信。那是单独的律师任务。
- 做出提交决定。它跟踪决定；律师做出决定。
- 监控评论后活动。一旦提交决定，此跟踪器的工作就完成了——通过 `/regulatory-legal:reg-feed-watcher` 跟进规则制定。

> `comment-decision` `gap_type` 语义、每次发送的 Slack 确认规则和 comment-tracker.yaml schema 存在于 **gap-surfacer** 参考 skill 中——在进行实质性工作之前加载它。
