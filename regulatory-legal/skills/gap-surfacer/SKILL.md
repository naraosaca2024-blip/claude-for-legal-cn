---
name: gap-surfacer
description: >
  参考：共享的差距和评论跟踪器框架，支持 /regulatory-legal:gaps
  和 /regulatory-legal:comments。跟踪带有整改状态的开放策略差距，
  从 policy-diff 摄取差距，surfaced 开放和陈旧的项目，路由到所有者，
  并通过 Slack 通知差距所有者，每次发送都确认。由 gaps
  和 comments skills 在进行实质性工作之前加载。
user-invocable: false
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# Gap Surfacer

> 所有者通知：默认开启。要让所有者退出，将 `owner_slack` 留空。

## 每次发送确认——无例外

在发送任何 Slack 消息（分配通知、过期提醒、批量通知、状态报告）之前：

1. 向用户准确显示您即将发送的内容以及发送给谁："我即将将此发送给 [N] 个人：[预览]。"
2. 等待明确的 yes。
3. 如果消息包含任何引用、截止日期或合规结论，添加："⚠️ 此消息中的引用未经验证——我在发送之前不确认它们是当前的。您想让我添加'在行动之前验证'行吗？"
4. 没有确认绝不发送。不是按计划。不是批量。不是因为昨天发送过。

在没有确认的情况下自动发送是此插件中最不可逆的操作，发送此插件自己的页脚声称可能错误的内容，给无法检查的人。这种组合不能跳过审查。

## 事项上下文

**事项上下文。** 检查实践级 CLAUDE.md 中的 `## 事项工作区`。如果 `Enabled` 是 `✗`（内部用户的默认值），跳过此段的其余部分——skills 使用实践级上下文，事项机制不可见。如果启用且没有活动事项，询问："这是哪个事项？运行 `/regulatory-legal:matter-workspace switch <slug>` 或说 `practice-level`。"加载活动事项的 `matter.md` 以获取事项特定的上下文和覆盖。将输出写入事项文件夹 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/matters/<matter-slug>/`。除非 `跨事项上下文` 是 `on`，否则绝不读取另一个事项的文件。

---

## 目的

差距被发现然后被遗忘。此 skill 跟踪它们直到关闭，并通知负责关闭它们的人。

## 跟踪器

位于 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/gap-tracker.yaml`：

> **关于 comment-tracker.yaml 的说明：** `~/.claude/plugins/config/claude-for-legal/regulatory-legal/comment-tracker.yaml` 是由 comments skill 拥有的兄弟文件。它由 reg-feed-watcher（自动记录 NPRM）和 comments skill（跟踪用户发起的评论决定）写入。此 skill 不读取或交叉引用它。如果您修改 comment-tracker schema，请更新两个实际消费者。

```yaml
gaps:
  - id: GAP-001
    requirement: "[法规要求的内容]"
    regulation: "[名称 + 引用]"
    policy_affected: "[名称或'需要新策略']"
    gap_type: "partial"  # none | partial | full | new-policy | watch | comment-decision
    owner: "[来自策略索引的名称]"
    owner_slack: "[Slack 用户 ID 或句柄，如果已知]"
    opened: 2026-03-01
    due: 2026-06-01  # 法规生效日期、内部截止日期或评论截止日期
    status_verified: true  # 如果上游 policy-diff 无法确认规则生效则为 false；未验证的项目永远不会达到 🔴 过期
    status: "open"  # open | in-progress | closed | risk-accepted
    notified: false  # 在发送分配通知后设置为 true
    resolution: ""  # 关闭时填充
```

**永远不要将差距归类为未验证规则的过期。** 🔴 过期分类意味着"我们错过了有约束力的截止日期。"如果规则状态未验证（policy-diff 设置 `status_verified: false`，或规则 >12 个月 / 超过其适用日期且无货币确认），截止日期可能没有约束力。使用 🟡 "需要审查"并注意："如果此规则按发布生效，这将过期 [N] 天。在升级之前验证规则状态。"将未验证规则项目路由到 `watch`，而不是活动的过期/即将到期存储桶；`watch` 重新评估节奏在项目可以重新作为合规差距浮出水面之前强制进行规则状态检查。

**`gap_type` 语义：**

| 值 | 含义 | 典型提醒节奏 |
|---|---|---|
| `none` | 策略已覆盖要求。仅记录用于审计跟踪。应该很少——如果大多数条目是 `none`，差异可能针对错误的策略运行。 | 无自动提醒。 |
| `partial` | 策略涉及主题但不完全覆盖新要求。需要修正。 | 截止前 30 天。 |
| `full` | 策略与新要求相矛盾或默默省略。需要重写或新章节。 | 截止前 30 天。 |
| `new-policy` | 没有现有策略涵盖此内容。需要起草策略。 | 截止前 30 天。 |
| `watch` | 前瞻性项目——ANPR、RFI、尚未最终的拟议规则。今天没有合规义务；策略工作等待最终规则。`due:` 是重新访问日期（通常是 NPRM 预期日期或一年期限），而不是合规截止日期。 | 无自动提醒；在 NPRM 发布时或重新访问日期重新评估。 |
| `comment-decision` | 规则前评论决策待定——团队正在决定是否提交评论的 ANPR 或 NPRM。`due:` 是评论截止日期。 | 截止前 21 天（比合规差距更紧，因为评论起草窗口更短）。 |

`watch` 或 `comment-decision` 条目不是合规差距——它是 watch skill 和 comments skill 产生的规则前项目的跟踪工件。在状态报告中在自己的存储桶中 surfaced 它们，以便在上午 7 点阅读的律师可以一目了然地区分哪些项目是"在监管机构注意到之前修复此内容"与"关注此内容"。

## 模式

### 模式 1：从 policy-diff 摄取

当 policy-diff 发现差距时，将它们附加到 gap-tracker.yaml。去重——相同要求 + 相同策略 = 相同差距，不要重复计算。

**摄取后，通知所有者：**

如果 Slack MCP 可用且设置了 `owner_slack`：

向差距所有者发送 Slack DM——但仅在此文件顶部的每次发送确认之后。向用户预览消息，等待明确的 yes，然后发送：

```
📋 分配给您的新合规差距

差距：[GAP-ID] — [要求，一句话]
法规：[名称 + 链接]
受影响的策略：[策略名称或"需要新策略"]
截止：[法规生效日期]

查看完整差距跟踪器：/regulatory-legal:gaps
```

发送后在跟踪器条目中设置 `notified: true`。

如果 Slack MCP 不可用：在状态报告中注明未发送所有者通知并标记手动跟进。

### 模式 2：状态报告

```markdown
[工作产品标题 — 根据插件配置 ## 输出 — 因角色而异；参见 `## 谁在使用此`]

## 开放差距 — [日期]

### 结论

[N 个差距需要在 [日期] 前采取行动 — 前 3 个：X、Y、Z]

### 🔴 过期

| ID | 要求 | 策略 | 所有者 | 截止 | 超过天数 |
|---|---|---|---|---|---|

### 🟠 <30 天内到期

[相同]

### 🟡 开放

[相同]

### 👀 观察项目（前瞻性 — 规则前）

[规则前跟踪 — `watch` 和 `comment-decision` 条目。这些不是
合规差距。单独 surfaced，以便过期 / 即将到期带仅包含
真正的合规截止日期。]

| ID | 项目 | 类型（ANPR/NPRM/RFI） | 评论截止日期 | 所有者 |
|---|---|---|---|---|

### 进行中

[相同]

### 最近关闭

[最后 5 个，带有解决方案]

---

**最古老的开放差距：** [ID]，[N] 天
**按所有者的差距：** [细目]
**已发送所有者通知：** [N] / [总差距 N]

---

**每个开放差距的下一步：** `/regulatory-legal:policy-redraft` 生成带有 `[verify]` 标记和变更摘要的标记策略重写草案。这是策略所有者审查的提案——不是对源文档的直接编辑。

---

**在依赖之前验证引用。** 此跟踪器中的法规引用是上游 AI 生成的（由 reg-feed-watcher 和 policy-diff），尚未与主要来源核对。在关闭或风险接受差距之前——或在证明、董事会报告或监管机构回应中引用差距——根据 Westlaw、您事务所的研究平台或发行机构的网站确认基础规则。AI 生成的法规引用有时会被伪造、误引用或过时。从上游携带的源标签（例如，`[Federal Register]`、`[web search — verify]`）显示每个引用的来源；带有 `verify` 的标签具有更高的伪造风险，应首先检查。在 surfacing 差距时绝不剥离标签。
```

## 配置依赖的后备方案

此 skill 从 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 读取差距响应所有者和升级路径。当它需要的值为空或仍然是 `[PLACEHOLDER]` 时：

- **缺少差距响应分流器：** 将分配保留开放并附加到输出："`## 差距响应流程` 中未设置分流器。使用 `/regulatory-legal:cold-start-interview --redo` 或通过编辑 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 分配一个，以便新差距得到路由。"
- **新摄取差距的所有者未知（策略库中没有所有者）：** 使用 `owner: [unassigned]` 记录差距并附加："[N] 个差距在没有所有者的情况下被摄取，因为策略库没有为受影响的策略命名一个。填写策略库中的所有者列以路由它们。"
- **过期材料差距缺少升级路径：** 仍将其报告为过期，并附加："未为材料过期差距设置升级路径。使用 `/regulatory-legal:cold-start-interview --redo` 或通过编辑 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 配置它。"

当值已填充时，不说明配置。

**截止日期提醒逻辑（在状态报告和计划代理期间运行）：**

提醒节奏是 `gap_type` 的函数——合规差距获得 30 天提前通知，comment-decision 项目获得 21 天（更紧，因为起草窗口更短），watch 项目没有自动提醒（在 NPRM 发布时重新评估）。

对于每个状态为"开放"或"进行中"的差距：
- `partial`、`full`、`new-policy`、`none`：如果截止日期在 30 天内且在过去 7 天内未发送提醒，预览 Slack DM（主题"⏰ 提醒：合规差距将在 [N] 天内到期"）并在发送前等待每次发送确认。
- `comment-decision`：如果评论截止日期在 21 天内且在过去 7 天内未发送提醒，预览 Slack DM（主题"💬 评论决策截止日期在 [N] 天内"）并在发送前等待每次发送确认。
- `watch`：无自动提醒。在审查跟踪器或为同一法规记录 NPRM 时重新访问。
- 如果合规差距的截止日期已过：在报告中标记为过期并预览 Slack DM——在发送前等待每次发送确认。
- 如果 `comment-decision` 项目的评论截止日期已过且未提交评论：标记为过期，预览 Slack DM（等待每次发送确认），并要求所有者更新为 `risk-accepted`（有意不评论）或 `closed`（已提交评论）并附带说明。
- 在跟踪器中记录提醒时间戳以避免重复唠叨。
- 批量提醒仍需要每次发送确认——预览"您即将发送 12 条 DM"并等待 yes 计数；静默触发批量不算。

### 后果行动关卡（证明合规）

**在将差距关闭为已解决或产生任何证明符合监管要求的输出（内部证明、董事会报告、审计回应、监管机构回应）之前：** 阅读 ~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md 中的 `## 谁在使用此`。如果角色是 **Non-lawyer**：

> 证明合规——或将差距关闭为已解决——具有法律后果。如果后来证明是错误的，证明可以被用来对抗公司，过早的关闭会使暴露未得到解决。您是否与律师审查过此内容？如果是，继续。如果否，以下是带给他们的简报：
>
> - 差距（要求、来源、策略差异发现的内容）
> - 提议的解决方案涵盖和不涵盖的内容
> - 任何残留差距或歧义
> - 未解决的问题和未解决的问题
> - 可能出什么问题（过于宽泛的证明、未解决的残留义务、不一致的先前立场）
> - 向律师问什么（这是否真正关闭；我们应该以理由风险接受而不是；我们需要外部律师同意吗）
>
> 如果您需要找到律师：您专业监管机构的转介服务是最快的起点（美国的州律师协会；英格兰和威尔士的 SRA/律师标准委员会；苏格兰/北爱尔兰/爱尔兰/加拿大/澳大利亚的律师协会；或您所在司法管辖区的同等机构）。

在没有明确的 yes 的情况下，不要将差距标记为关闭或在此关卡之后产生合规证明。状态报告和跟踪视图不需要关卡。

### 模式 3：关闭差距

```
/regulatory-legal:gaps --close GAP-001
解决方案："策略已更新 v2.3，批准于 [日期]"
```

将状态更新为关闭，记录解决方案和关闭日期。

### 模式 4：风险接受差距

有时答案是"我们不打算修复这个。"这是一个有效的决定——但应该记录。

```
/regulatory-legal:gaps --accept GAP-002
理由："要求仅适用于[我们不满足的条件]。如果[触发器]则重新访问。"
接受者：[有权威的姓名]
```

状态 → risk-accepted。停留在跟踪器中（未删除）但从开放差距报告中脱落。

## 集成：reg-change-monitor 代理

代理的摘要包括差距计数和最古老的开放差距年龄。如果有任何过期，那将在摘要的顶部。代理还运行截止日期提醒检查并发送任何未完成的 Slack 通知。

## 用下一步决策树结束

根据 CLAUDE.md `## 输出` 以下一步决策树结束。根据此 skill 刚刚生成的内容自定义选项——五个默认分支（起草 X、升级、获取更多事实、观察等待、其他事情）是起点，而不是锁定。树就是输出；律师选择。

如果跟踪器 surfaced 超过约 10 个开放差距，或者任何时候用户询问：提供仪表板（参见 CLAUDE.md `## 输出 → 数据重输出的仪表板提供`）。为此输出塑造提供——按严重程度的计数、按截止日期的差距时间线，以及带有所有者、状态和最后接触日期的可排序网格。

## 此 skill 不做什么

- 自己关闭差距。关闭需要解决方案笔记和人类行动笔记描述的行动。
- 如果未配置 Slack MCP 则发送 Slack 通知。回退到在状态报告中标记。
- 每 7 天每周期的差距发送超过一个提醒。唠叨一次，而不是不断唠叨。
