---
name: legal-hold
description: 发布、刷新、释放或报告法律保留——起草保留通知为 .docx、更新 _log.yaml 中的 legal_hold 字段、并将下次刷新加入日历。当用户说"发布保留"、"刷新保留"、"释放保留"或询问投资组合范围的保留状态报告时使用。
argument-hint: "[slug] [--issue | --refresh | --release | --status]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /legal-hold

1. 如果 `--status`（无 slug）：阅读 `_log.yaml`，生成投资组合范围保留报告。
2. 否则：加载 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md` + 日志行。
3. 加载 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` → 特权标记、保留模板指针、升级规范。
4. 遵循以下工作流和参考。
5. 按标志路由：
   - `--issue`：捕获范围、保管人、日期范围、系统。起草 `legal-hold-v1.docx`。更新 `legal_hold` 字段。附加历史条目。设置 `next_refresh`（默认 +6 个月）。
   - `--refresh`：捕获范围/保管人变更。起草下一版本。更新 `last_refresh` + `next_refresh`。标记已离任保管人。
   - `--release`：捕获释放日期、保留指令。起草释放通知。设置 `released:` 字段。
6. 在写入前确认。向用户显示起草的通知和日志差异。

---

# 法律保留

## Purpose

法律保留是内部法律顾问撰写的最机械的高风险文档。通知本身是模板化的。失败模式是操作性的：发布太晚、范围太窄、从未刷新、从未释放。此 skill 拥有所有四个阶段：**发布 → 刷新 → （释放）→ 跟踪**。

投资组合已经标记缺少的保留；此 skill 编写它们。

## Jurisdiction assumption

保留义务因论坛而实质不同。联邦普通法（通过 Zubulake / Residential Funding / Rule 37(e)）与州实践不同；州与州之间在触发时间、范围、制裁和销毁补救方面不同；监管保留义务在某些事项中叠加民事规则（SEC Rule 17a-4、HIPAA 等）。草稿中引用的触发、范围和制裁敞口是为事项中命名的论坛的起点阅读——在发布、刷新或释放前与律师确认。

## Load context

- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/_log.yaml` ——日志行（legal_hold 字段 + 状态）
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md` ——事项上下文（对手方、事实、来自 internal_owners 的关键保管人）
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` ——诉讼保留模板指针的内部风格、特权标记、升级规范

**冲突门槛——不可绕过。** 在发布、刷新或释放保留之前，检查事项 slug 的 `_log.yaml`。如果事项不在 `_log.yaml` 中，拒绝并路由：

> "我在事项日志中看不到 [matter slug]。首先运行 `/litigation-legal:matter-intake`，以便冲突检查运行并设置事项工作区。我不会对未接收的事项发布、刷新或释放法律保留——冲突检查是门槛，针对未管理事项发布的保留没有 `_log.yaml` 行来跟踪 `last_refresh` / `next_refresh` / `released`。"

不要在未接收的事项上继续。接收是运行冲突并写入 `--refresh` / `--release` / `--status` 标志操作的 `_log.yaml` 行的过程。

## Modes

命令接受一个标志：`--issue | --refresh | --release | --status`。默认（无标志）→ 提示。

### `--issue` — 首次发布

当 `legal_hold.issued == false` 且事项活跃或合理预期时需要。

**在向保管人发布保留之前（重大行为）：** 阅读 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 中的 `## Who's using this`。如果角色是 Non-lawyer：

> 发布法律保留有法律后果——范围、保管人列表和时间创建公司将在以后争论销毁时被评判的保留记录。你是否与律师审查了此内容？如果是，继续。如果否，这是带给他们的简报：
>
> [生成一页摘要：事项和触发、建议的范围和保管人、研究的论坛特定保留规则、已知的销毁敞口、可能出什么问题（太宽 / 太窄）、要问律师什么。]
>
> 如果你需要在你所在的司法管辖区找到持牌律师、事务律师、大律师或其他授权法律专业人士：你所在专业监管机构的转介服务是最快的起点（美国的州律师协会、英格兰和威尔士的 SRA/律师标准委员会、苏格兰/北爱尔兰/爱尔兰/加拿大/澳大利亚的律师协会，或你所在司法管辖区的同等机构）。

没有明确的是不要发送通知。起草和范围界定不需要门槛——发布需要。

**在发布前研究适用的保留规则。** 识别司法管辖区和保留义务的来源（普通法、民事诉讼规则、监管保留义务、合同）。确认当前适用的触发标准（义务何时附加）、范围标准（必须保留什么）和制裁敞口（论坛的销毁原则）。引用主要来源。注意联邦和州法律在触发时间、范围和补救方面可能实质不同——标记你依赖的论坛。如果不确定，说出来并在发布前获得外部律师签署。

> **外部可交付成果：**以下通知发送给保管人。不要在外发通知上包含 `PRIVILEGED & CONFIDENTIAL — ATTORNEY WORK PRODUCT — PREPARED AT THE DIRECTION OF COUNSEL` 标题；使用模板中的律师-客户标记。确认你的司法管辖区和事项的正确标记。

**输入：**
1. **范围**——文档、数据、通信的类别。具体开始：与对手方的合同、所有引用 [项目/主题] 的通信、相关财务记录、日历条目。`[SME VERIFY —范围太宽 = 操作负担；太窄 = 销毁风险]`
2. **保管人**——可能持有响应材料的指定个人。从 matter.md internal_owners 和常见角色（业务负责人、如果是雇佣则为 HR 合作伙伴、如果是数据则为 CISO）获取建议。`[SME VERIFY ——保管人列表是可辩护保留和差距论点之间的区别]`
3. **日期范围**——从何时开始保留（通常是：触发事件或更早），直到现在 + 持续。
4. **系统**——电子邮件、Slack/Teams、文件共享、设备（包括适用时的 BYOD）、Jira/Asana、CRM、旧系统。
5. **紧急性**——如果已送达诉讼或收到带有诉讼威胁的要求，这今天就要发出。
6. **生效日期**——保留的日期。

**使用以下模板起草每个保管人的通知**，如果 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 中配置了内部模板；否则使用以下默认模板。

**默认保留通知模板：**

```
[PRIVILEGED & CONFIDENTIAL — ATTORNEY-CLIENT COMMUNICATION]

DATE：[生效日期]
TO：[保管人姓名]
FROM：[签署人——根据 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 默认]
RE：诉讼保留通知——[事项简称]

您收到此通知是因为 [公司] 已确定 [争议/调查的一句话描述，避免不利细节]。法律要求保留可能与该事项相关的文档和通信。

立即生效，您必须保留：

1. 与 [范围要点 1] 相关的所有文档、电子邮件、短信、Slack/Teams 消息和其他通信。
2. [范围要点 2]
3. [范围要点 3]
...

此保留义务适用于：
- 电子邮件（包括已发送、已归档、已删除文件夹）
- Slack/Teams/消息平台
- 共享驱动器和云存储
- 用于公司业务的个人设备（BYOD）
- 纸质文档
- 语音邮件
- 日历条目和会议记录

不要：
- 删除、修改、销毁或处置任何潜在的响应材料
- 自动删除或"清空收件箱"任何电子邮件或消息

在与直属下属或 IT 共享此通知之前与 [法律联系人] 协调。

关于此通知或您的保留义务的直接问题请联系 [法律联系人]。您可以继续根据需要与同事讨论基础业务主题，但不要讨论此法律通知、诉讼或法律策略。

如果您不确定某些内容是否覆盖，请保留。

请在三个工作日内通过 [回复 / 链接 / 表格] 确认收到此通知。如果您有问题，请联系 [签署人电子邮件]。

此通知保持有效，直到您收到其释放的书面通知。您可能会被要求在定期间隔重新确认合规。

[签署人签名块]
```

**发送门槛（草稿的结束说明）：**附加到通知的聊天预览中——在通知发送给保管人之前删除：

> 这是律师审查的起草法律保留通知，不是准备发布的通知。发布保留触发公司将在以后任何销毁论点中被评判的保留义务，通知本身可能是可发现的。持牌律师审查、批准和发布。不要未经审查分发此草稿。

**写入：**
- 通过 `docx` skill 写入 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/legal-hold-v1.docx`
- 附加到 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/history.md`：
  ```
  ## [YYYY-MM-DD] — 法律保留已发布

  已向 [N] 个保管人发布保留：[列表]。
  范围：[一行摘要]。
  下次刷新：[YYYY-MM-DD（默认发布 + 6 个月）]。
  ```
- 更新 `_log.yaml` 行：
  ```yaml
  legal_hold:
    issued: true
    issued_date: [YYYY-MM-DD]
    scope: "[一行摘要]"
    custodians: [列表]
    last_refresh: [YYYY-MM-DD]   # 首次发布时与 issued_date 相同
    next_refresh: [YYYY-MM-DD]   # 默认：issued_date + 6 个月
    released: null
  ```

### `--refresh` — 定期重新确认

刷新节奏：默认 6 个月；可根据事项调整。当 `next_refresh < today`（或用户手动调用）时，skill 起草刷新通知。

**输入：**
1. 自上次刷新以来的任何**范围变更**（发现中出现的新主题、新保管人、新系统）。
2. 任何**要添加或移除的保管人**（离任需要特殊处理——见下文）。
3. 重新确认语言。

**刷新通知模板：**与发布类似；以"这是最初于 [日期] 发布的法律保留的重新确认"开头。列出当前范围（如需要则修改）。请求重新确认。

**已离任保管人：**如果保管人自上次刷新以来已离开公司，skill 将此标记为保留行动项目——离任员工的文件和电子邮件档案需要在 IT 级别保留，而不仅仅是通过通知个人。在 history.md 中记录为需要行动的单独条目。

**写入：**
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/legal-hold-v[N].docx`（下一版本号）
- `history.md` 条目
- `_log.yaml`：更新 `last_refresh` 和 `next_refresh` 字段；如变更则修改 `custodians` 列表

### `--release` — 关闭保留

通常在事项关闭时。确认事项真正结束（未上诉、不太可能重新开放、相关主张的诉讼时效已过）。

**在释放保留之前（重大行为——保留义务恢复正常保留）：** 阅读 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 中的 `## Who's using this`。如果角色是 Non-lawyer：

> 释放法律保留有法律后果——一旦释放，保管人可能开始删除材料。在错误时间释放会产生销毁敞口。你是否与律师审查了此内容？如果是，继续。如果否，这是带给他们的简报：
>
> [生成一页摘要：事项状态、为何现在提议释放、相关主张/上诉/诉讼时效敞口、保管人影响、可能出什么问题、要问律师什么。]
>
> 如果你需要在你所在的司法管辖区找到持牌律师、事务律师、大律师或其他授权法律专业人士：你所在专业监管机构的转介服务是最快的起点（美国的州律师协会、英格兰和威尔士的 SRA/律师标准委员会、苏格兰/北爱尔兰/爱尔兰/加拿大/澳大利亚的律师协会，或你所在司法管辖区的同等机构）。

没有明确的是不要发送释放通知。

**输入：**
1. 释放权限确认（通常是签署人或 GC）。
2. 释放日期。
3. 保留指令——保留下的材料会发生什么？（恢复正常保留？在定义期间内继续保留？转移到档案？）

**释放通知模板：**一段，正式。"关于 [事项] 于 [日期] 发布的诉讼保留于 [日期] 生效释放。正常保留恢复。"

**写入：**
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/legal-hold-release.docx`
- `history.md` 条目
- `_log.yaml`：设置 `released: [YYYY-MM-DD]`

### `--status` — 投资组合范围报告

阅读 `_log.yaml`。生成报告：

```markdown
# 法律保留状态——[今天]

## 活跃保留

| 事项 | 已发布 | 上次刷新 | 下次刷新 | 保管人 | 状态 |
|---|---|---|---|---|---|
| [slug] | [date] | [date] | [date] | [N] | [ok / ⚠️ 刷新到期 / ❌ 逾期] |

## ⚠️ 注意

- **刷新逾期：** [列出 next_refresh < today 的 slug]
- **30 天内刷新到期：** [列表]
- **活跃事项未发布保留：** [列表——高/关键风险优先]
- **已关闭事项保留仍活跃：** [列表——考虑释放]

## 最近释放

[最近 5 个释放保留及其日期]
```

这是一个单独的命令调用（`/legal-hold --status` 无 slug）或由 `/portfolio-status` 调用作为投资组合汇总的一部分。

## Integration with portfolio-status

`portfolio-status` skill 已经标记"活跃诉讼未发布保留"。此 skill 解决这些标记。在打开事项时值得在简报中交叉引用：如果 `legal_hold.issued == false`，`/matter-intake` 通过提议运行 `/legal-hold --issue` 关闭。

## What this skill does not do

- **执行保留。** 它发布通知；IT/保管人保留。skill 在保管人离开时标记（以便 IT 可以在系统级别保留）但不进入系统。
- **单独做出范围判断。** skill 从事项上下文提出范围；用户确认。范围太宽 = 操作负担。范围太窄 = 销毁风险。用户的判断。
- **未经审查自动刷新。** 即使当 `next_refresh` 到来时，用户在刷新通知发出前审查范围变更。
- **发送通知。** 起草 .docx；用户根据内部惯例通过电子邮件发送。（未来集成：Gmail/O365 MCP 可在用户审查后直接发送。）
