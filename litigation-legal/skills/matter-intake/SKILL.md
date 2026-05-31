---
name: matter-intake
description: 受理新事项——涵盖识别、冲突、来源、风险分流、重要性、外部律师、所有者、法律保留和关键日期的统一问题；编写 matter.md 和 history.md 并向 _log.yaml 附加结构化行。当用户说"new matter"、"intake this matter"或想将新事项带入组合时使用。
argument-hint: "[可选事项名称]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /matter-intake

1. 加载 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` → 风险校准（用于分流）、landscape（用于上下文、冲突方法）、stakeholders（用于通知谁）。
2. 遵循以下工作流和参考。
3. 运行统一 intake：识别、冲突检查、来源、风险分流、重要性、外部律师、内部所有者、法律保留、关键日期、初始姿态。
4. 从事项名称生成 slug（小写、连字符、年份）。
5. 创建 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md` — 完整叙述 intake。
6. 创建 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/history.md` — 以 intake 为第一个条目进行种子化。
7. 向 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/_log.yaml` 附加结构化行。
8. 与用户确认："这是我将要写的行——有任何编辑吗？"

---

# 事项 Intake

## 目的

每个新事项都经过相同的 intake，以便组合保持可比性。`_log.yaml` 中的统一行让 status skill 可以汇总。`matter.md` 中的叙述捕获了行无法捕获的内容。在此处种子化的历史文件成为事件记录。

## 加载上下文

- `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` — 风险校准（分流阈值、重要性、和解阶梯）、landscape（stakeholders、外部律师 bench）。
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/_log.yaml` — 确认 slug 唯一性。

## Intake

### 1. 识别

- 事项名称（如通常引用，例如"Acme v. Us 2026"）
- Counterparty
- 事项类型：`contract | employment | ip | regulatory | investigation | product | other`
- 我方角色：`plaintiff | defendant | claimant | respondent | investigated`
  - 如果执业档案的 `## Side` 是 `plaintiff`、`defense` 或"both — default X"变体，从该默认值预填充角色并确认。如果 `## Side` 是"varies by matter"，冷启动询问。永远不要默默假设执业档案未设定的姿态。
  - 角色驱动下游 skills：原告姿态事项将风险分流路由到案件价值/或有经济学；被告姿态事项路由到敞口/储备/保险 tender。
- 司法管辖区（法院、仲裁论坛或监管机构）

### 2. 冲突检查

在继续之前，按照 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` → Conflicts clearance 运行冲突步骤。

- **状态：** `cleared | pending | not-run | waived`
- **方法：** 匹配 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 声明的内容（`corporate-legal | outside-counsel | system-check | informal | other`）。如果声明的方法是 `informal`，这样说——记录仍然捕获以律师判断检查为基础。
- **由谁清理：** 名称 / 团队 / 律所
- **清理日期：** YYYY-MM-DD
- **检查对象：** 运行的具体名称/实体的简要列表（对手方、已知关联方、对方律师（如已知）、关键证人）。薄是可以的；"否"不可以。
- **备注：** 标记但已清除的任何内容（例如，"Smith 在我们的董事会，2019–2021 年在对手方董事会任职——已清理为与此事项不重叠"）。

按状态的行为：

- `cleared` → 继续。
- `pending` → 继续 intake；在 `matter.md` 和日志行中突出标记冲突未解决；在每次 `/matter-update` 和 `/portfolio-status` 中再次浮现，直到解决。
- `waived` → 罕见；需要冲突放弃理由（编写放弃不在本 skill 范围内——捕获放弃存在、谁签署了它、它在哪里）。
- `not-run` → **停止。这是一个关卡。** 在冲突姿态解决之前，skill 不会创建 `matter.md`、`history.md` 或 `_log.yaml` 条目。三个可接受的路径：

  **路径 1 — 现在运行冲突。** 暂停此 intake。按照 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` Conflicts clearance 清理。带着 `status: cleared` 或 `status: waived` 附带理由返回。

  **路径 2 — 标记待定并附上所有者 + 到期日。** 仅当 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` Conflicts clearance 声明并行 intake 可接受时允许。捕获：谁在运行冲突、何时预期返回、他们在检查什么实体。Intake 继续；事项行携带 `conflicts.status: pending`；`/portfolio-status` 每次运行都标记它；`/matter-update` 重新提示直到解决。

  **路径 3 — 绕过并附有记录的理由。** 仅当用户明确确认绕过时。在 `conflicts.override` 中记录：

  ```yaml
  conflicts:
    status: not-run               # 保持原样
    override:
      by: [用户名称]
      date: [YYYY-MM-DD]
      rationale: [绕过冲突的原因——永久记录；不会自动过期]
  ```

  此字段在每个 `/portfolio-status`、每个 `/matter` 简报和每个 `/matter-update` 中可见，直到删除。skill 永远不会删除它——仅在冲突实际清理后通过用户对 `_log.yaml` 的显式编辑。

  **不要默默进行。** "我稍后会做"不是可接受的回答。必须选择路径 1/2/3 之一，并且选择被捕获在记录中。

此步骤不是关于 skill 决定冲突是否存在——那是用户/律所的判断。它是关于确保检查发生并且记录反映它。

### 3. 来源

这是如何到达的？
- `demand-letter | complaint-served | subpoena | regulator-inquiry | internal-report | pre-suit-threat`
- *种子文档机会：* "如果你有启动文档（complaint、demand、subpoena），附加或分享路径。它 sharpen 了 intake。"

### 4. 风险分流——对照内部校准

- 严重性：high | medium | low（参考 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 严重性带）
- 可能性：high | medium | low（参考 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 可能性带）
- 结果风险评级（按照矩阵）：high | medium | low | critical
- 损害敞口范围（最佳估计）
- 非金钱敞口（禁令？同意令？宣传？先例？）

如果 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 中的风险校准薄，不要伪造精确度。使用用户的直觉并记录薄度。

### 5. 重要性

对照 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 中的内部阈值：
- `reserved | disclosed | monitored | none`
- 如果 `reserved`：储备金额以及是否已通知财务
- 如果 `disclosed`：备案和脚注位置

### 6. 外部律师

- 律所
- 主要合伙人
- **主要合伙人电子邮件**（由 `/oc-status` 用于起草状态请求）
- 聘书状态：`signed | pending | none`
- 预算授权：金额和批准人
- *种子文档机会：* "聘书路径（如已签署）。"

如果风险为中等或更高且未分配外部律师——标记它。

### 7. 内部所有者

从 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` landscape——涉及哪些内部 stakeholder？
- 业务负责人
- HR 合作伙伴（如雇佣）
- 通讯联系人（如声誉风险）
- CISO（如数据或网络）
- 其他

### 8. 法律保留

- 已发布？如果是：日期、范围、保管人（姓名列表）。
- 下次刷新日期（默认：发布后六个月；按事项调整）。
- 如果否且这是活跃诉讼或合理预期：紧急标记；提议在 intake 完成后运行 `/litigation-legal:legal-hold [slug] --issue`。
- *种子文档机会：* "保留通知（如已发布）。"

### 9. 关键日期

- 响应截止日期（答辩、异议、反对）
- 下次听证/会议
- 诉讼时效截止（如适用）
- 任何监管截止日期

### 10. 初始姿态

一段式理论：
- 我们的故事是什么？
- 他们的故事是什么？
- 转折事实是什么？
- 初始姿态：`fight | settle | investigate | wait`

## 编写输出

### Slug

小写、连字符、年末。示例：`acme-v-us-2026`、`employment-smith-2026`、`ftc-inquiry-2026`。

在写入前确认 slug 在 `_log.yaml` 中唯一。

### `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md`

```markdown
[WORK-PRODUCT HEADER — 根据 plugin 配置 ## Outputs — 因角色而异；见 `## Who's using this`]

# [事项名称]

**Slug：** [slug]
**开启：** [YYYY-MM-DD]
**我方角色：** [plaintiff/defendant/etc.]
**状态：** [status]

---

## 识别

[counterparty、jurisdiction、事项类型、source]

## 冲突

**状态：** [cleared / pending / not-run / waived]
**方法：** [corporate-legal / outside-counsel / system-check / informal / other]
**由谁清理：** [name]
**清理日期：** [YYYY-MM-DD]
**检查对象：** [运行的实体]
**备注：** [任何标记的清理项、放弃参考（如适用）]

## 风险分流

**严重性：** [band] — [为什么，参考内部严重性定义]
**可能性：** [band] — [为什么]
**风险评级：** [high/medium/low/critical]
**敞口：** [美元范围 + 非金钱]

## 重要性

[reserved/disclosed/monitored/none — 附储备金额、披露位置，或如为"none"的理由]

## 外部律师

[firm、lead、engagement 状态、budget]

## 内部所有者

[stakeholder 及每个为何涉及]

## 法律保留

[状态、日期、范围]

## 关键日期

[列表]

## 初始理论

[一段：我们的故事、他们的故事、转折事实、初始姿态] `[SME VERIFY — intake 处的理论是工作假设；在任何依赖此框架的备案或实质性沟通之前与外部律师确认]`

## 未决问题

[任何尚未知道但重要的事项——例如，"保险 tender 待定"、"不清楚我们是否有 X 的覆盖"]

---

## 种子文档

| 文档 | 路径/指针 |
|---|---|
| [例如，complaint] | [路径或"not yet shared"] |
```

### `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/history.md`

将历史文件种子化为以 intake 为条目零：

```markdown
# 历史：[事项名称]

仅追加事件日志。最近的最前。

---

## [YYYY-MM-DD] — 事项开启

[来源、谁带入、初始分流摘要、分配的外部律师、法律保留发布是/否。]
```

### 附加到 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/_log.yaml`

按照 schema 添加一行。示例：

```yaml
- id: acme-v-us-2026
  name: "Acme Corp v. Company"
  type: contract
  role: defendant
  counterparty: "Acme Corp"
  jurisdiction: "N.D. Cal."
  # status 从来源派生：
  #   source: pre-suit-threat | demand-letter           → status: threatened
  #   source: complaint-served | subpoena | regulator-inquiry → status: active
  #   source: internal-report                           → status: threatened（默认）或如正式流程已开始则为 active
  status: active
  stage: pleadings
  source: complaint-served
  outside_counsel:
    firm: "Wilson Sonsini"
    lead: "J. Reyes"
    email: "jreyes@wsgr.example.com"
    engagement: signed
  conflicts:
    status: cleared
    method: corporate-legal
    cleared_by: "K. Patel"
    cleared_date: 2026-04-20
    override:                   # 仅在路径 3 绕过时填充
      by: null
      date: null
      rationale: null
  risk: high
  materiality: reserved
  exposure_range: "$2M–$5M"
  internal_owners:
    business_lead: "Jane Smith"
    hr_partner: null
    comms_contact: null
  legal_hold:
    issued: true
    issued_date: 2026-02-15
    scope: "Sales org 2023–2026"
    custodians: ["Jane Smith", "R. Chen", "T. Patel"]
    last_refresh: 2026-02-15
    next_refresh: 2026-08-15
    released: null
  related_matters: []
  opened: 2026-04-20
  next_deadline: 2026-05-15
  last_updated: 2026-04-20
  path: matters/acme-v-us-2026/
```

## 写入前确认

向用户展示行和 matter.md 内容：

> 这是我将要写入的内容。在我提交之前标记任何错误或薄的地方。

## 关闭时使用下一步决策树

根据 CLAUDE.md `## Outputs` 以下一步决策树结束。根据此 skill 刚刚生成的自定义选项——五个默认分支（起草 X、升级、获取更多事实、观察等待、其他）是起点，而非锁定。树就是输出；律师选择。

## 此 skill 不做什么

- **自己运行冲突检查。** 它记录结果、状态、方法和检查的实体。实际清理发生在内部执业档案声明的任何系统（或判断）中。如果用户说"已清理"，skill 按面值接受并捕获元数据。
- 决定初始理论。它捕获用户所说的；它不发明一个。
- 发布法律保留。如果缺少则标记。用户发布它。
