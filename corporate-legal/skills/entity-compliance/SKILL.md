---
name: entity-compliance
description: 实体合规跟踪器——初始化、报告即将到期的截止期限、更新状态、运行健康审计、导出为 CSV。维护从实体表构建的 compliance-tracker.yaml，按实体和司法管辖区计算申报截止期限，并显示接下来 30/60/90 天内到期的事项。当用户说"实体合规"、"申报截止期限"、"年度报告到期"、"实体跟踪器"、"什么申报到期"、"实体健康状况"或"良好信誉"时使用。
argument-hint: "[--init | --report [--days N] | --update [--from-report] | --sweep | --audit | --export [--format csv|table]]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /entity-compliance

1. 加载 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` → `## Entity Management`（实体表、司法管辖区、注册代理人）。
2. 根据标志路由到以下正确模式：
   - 无标志或 `--init`：模式 1 ——从实体表初始化跟踪器
   - `--report`：模式 2 ——显示即将到期的截止期限和逾期项目
   - `--update`：模式 3a（手动）或 3b（--from-report 上传）——更新状态
   - `--sweep`：模式 3c ——逐个浏览未知/逾期项目
   - `--audit`：模式 4 ——完整健康审计
   - `--export`：模式 5 ——生成 CSV 或表格导出
3. 读/写 `~/.claude/plugins/config/claude-for-legal/corporate-legal/entities/compliance-tracker.yaml`。
4. 任何更新后：显示变更摘要和下一步行动。

---

## Purpose

年度报告、特许经营税、信息声明、双年度申报——每个州每个实体都有自己的时间表和错过截止期限的后果。此 skill 维护一个单一的 YAML 跟踪器，知道什么到期、何时到期、以及哪个实体到期。它设计轻量：跟踪器是你拥有的文件，Claude 按命令更新它，你在需要共享时导出它。

## Important: deadline reference caveat

> 此 skill 参考表中的申报截止期限反映 skill 构建日期的公开可用要求。州申报要求和到期日期可能会变化。**在依赖截止期限进行合规目的之前，始终与您的注册代理人或直接与相关州务卿确认截止期限。**如果您使用 CT Corp、National Registered Agents 或其他注册代理人服务，他们的合规日历对您的特定实体具有权威性——使用此跟踪器来组织和显示他们的数据，而不是替代它。

## Jurisdiction assumption

> 此跟踪器根据每个实体记录的成立/资格州或国家计算截止期限。申报规则、到期日期机制和费用结构因司法管辖区而实质不同。如果实体的实际足迹与 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` 中的内容不同（未披露的外国资格、已解散实体、司法管辖区重新国内化、由当地代理人管理的国际申报），输出可能不适用——请与该司法管辖区的注册代理人或当地律师确认。

## Entity-type disambiguation (especially Delaware)

> 申报日历取决于 **实体类型**，而不仅仅是司法管辖区。将"特拉华实体"视为单个桶是一个常见且有后果的错误——DE 公司、DE LLC 和 DE LP 有不同的申报、不同的截止期限和错过申报的后果。在计算或报告截止期限之前，从实体表确认实体类型，永远不要在同一州中将截止期限从一个实体类型复制到另一个实体类型。
>
> **特拉华——重要的区别：**
>
> - **DE Corporation（Inc., Corp.）：** 年度报告 **和** 特许经营税，均于 **3 月 1 日** 到期。特许经营税通过授权股份方法或假设面值资本方法计算（以较低者为准）；年度报告捕获董事/高级管理人员信息。法定依据：8 Del. C. §§ 501–502 [验证当前]。
> - **DE LLC：** 不需要年度报告。年度税是 **固定 $300**，于 **6 月 1 日** 到期。法定依据：6 Del. C. § 18-1107(d) [验证当前费用和日期]。
> - **DE LP：** 不需要年度报告。年度税是 **固定 $300**，于 **6 月 1 日** 到期（与 LLC 规则平行）。法定依据：6 Del. C. § 17-1109 [验证当前]。
>
> DE LLC **不需要**提交 3 月 1 日的年度报告——为 LLC 编写该截止日期带有真正风险（虚假的"逾期"标志掩盖实际的 6 月 1 日敞口，或更糟，反之：用户将 3 月 1 日公司规则视为普遍规则并错过 6 月 1 日 LLC 截止期限）。如果实体表记录了没有类型的特拉华实体，将其标记为 `type_unknown` 并在计算任一截止期限之前要求用户确认。
>
> 相同的实体类型纪律适用于每个其他司法管辖区，按实体类型有不同的申报制度（例如，CA 公司信息声明 vs. CA LLC SOI 节奏；TX 特许经营税适用于公司、LLC 和 LP，但有不同的无税门槛）。当填充司法管辖区的参考表时，确保它按实体类型索引，而不仅仅是按州索引。

---

## Tracker file

位于 `~/.claude/plugins/config/claude-for-legal/corporate-legal/entities/compliance-tracker.yaml`。结构：

```yaml
# Entity Compliance Tracker
# 生成时间：[date]
# 最后更新：[date]
# 免责声明：截止期限仅供参考——与注册代理人或州务卿确认

metadata:
  company: "[Company Name]"
  generated: "[date]"
  last_updated: "[date]"
  last_audit: "[date or null]"

custom_jurisdictions:   # 手动添加——不在内置参考表中的美国州或国家
  []                    # 当遇到新司法管辖区时填充

entities:
  - name: "[Entity Name]"
    type: "[Corporation / LLC / LP / other]"
    state_of_formation: "[state]"
    formation_date: "[date or null]"
    status: "[active / dormant / dissolving]"
    registered_agent: "[CT Corp / National / in-house / other]"
    notes: ""

    jurisdictions:
      - state: "[state]"
        qualification: "[domestic / foreign]"
        qualified_date: "[date or null]"
        agent_managed: false   # 对于当地代理人处理合规的国际实体设置为 true
        local_agent: "[name or null]"
        filings:
          - type: "[Annual Report / Franchise Tax / Statement of Information / Biennial Statement / other]"
            due_date: "[YYYY-MM-DD]"
            due_basis: "[fixed date / anniversary month / other]"
            last_filed: "[date or null]"
            last_fee: "[amount or null]"
            status: "[current / due_soon / overdue / unknown]"
            confirmed_good_standing: "[date or null]"
            notes: ""
```

状态值：
- `current` ——已为当前期间申报，90 天内无到期
- `due_soon` ——90 天内到期
- `overdue` ——已过到期日期且未记录申报日期
- `unknown` ——无信息；需要手动确认

---

## Mode 1: Initialise

当跟踪器不存在时运行，或使用 `--rebuild` 从头重新生成。

### Step 1: Load entity table

阅读 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` → `## Entity Management` → 实体表。如果实体表已填充（从冷启动时的组织结构图上传），直接使用它。如果没有，要求用户运行冷启动模块或提供实体列表。

### Step 2: For each entity × jurisdiction, confirm the filing requirements

对于每个实体，与注册代理人或相关州务卿确认当前申报时间表。州申报时间表会变化（一些州从固定日期改为基于周年的日期，反之亦然，费用结构修订，申报类别重新分类）。不要依赖缓存的时间表。下面的跟踪器记录您确认的日期；当您的注册代理人发送提醒时更新它们。

对于实体注册的每个司法管辖区（国内或外国）：

1. 询问用户是否拥有来自注册代理人的当前合规报告——这是最权威的来源。
2. 如果没有，询问用户他们知道什么（申报类型、到期日期基础、上次申报日期、典型费用）。记录他们提供的内容。
3. 对于用户不知道的任何内容，将实体 × 司法管辖区条目标记为 `unknown` ——不要从缓存参考表填充日期。用户的下一步是与注册代理人或州务卿确认。

**在跟踪器中捕获细节而不是参考表：**

> 我在参考表中没有 [Jurisdiction] 的申报要求。
> 让我捕获它们，以便我们可以继续跟踪。
>
> 对于 [Jurisdiction] 中的 [Entity]：
> 1. 需要什么类型的申报？（年度报告、特许经营税、确认声明、年度申报或其他？）
> 2. 什么时候到期？（像 5 月 1 日这样的固定日期、周年月份或其他？）
> 3. 典型费用是多少？（大约即可——或"未知"。）
> 4. 您在那里有什么注册代理人或当地申报代理人？

在跟踪器的 `custom_jurisdictions` 块中存储答案：

```yaml
custom_jurisdictions:
  - jurisdiction: "[State / Country]"
    jurisdiction_type: "[US state / Canada province / EU member state / other]"
    filings:
      - type: "[filing type]"
        due_basis: "[fixed: MM-DD / anniversary month / other description]"
        typical_fee: "[amount or unknown]"
        notes: "[any other relevant information — e.g., local agent required, filing in local language]"
    added_by: "manual"
    added_date: "[date]"
```

然后此自定义定义应用于该司法管辖区中的所有实体。未来的 `--init` 运行和实体添加将自动使用它。

**特别是国际司法管辖区：**

国际申报因司法管辖区而差异巨大。始终通过上述自定义定义流程——在填充跟踪器之前与当地申报代理人或注册办公室代理人确认申报类型、节奏和费用。

对于国际实体，还要询问：
- 是否有当地申报代理人或注册办公室代理人处理合规？
  如果是，注意代理人姓名——跟踪器可以标记何时与他们跟进，而不是独立计算到期日期。
- 实体是否需要在此司法管辖区提交任何集团级报告（例如，国别报告、受益所有权登记、经济实质申报）？

在跟踪器中将具有当地代理人的国际实体标记为 `agent_managed: true`。报告模式将单独列出它们，并注意直接与当地代理人确认状态，而不是显示计算出的到期日期。

对于基于周年的申报：从跟踪器中的 formation_date 计算。如果 formation_date 为空：将状态设置为 `unknown` 并标记确认。

### Step 3: Write the tracker

生成 `~/.claude/plugins/config/claude-for-legal/corporate-legal/entities/compliance-tracker.yaml`，其中包含所有实体及其计算的申报要求。设置初始状态：
- 如果 last_filed 在当前申报期内，则为 `current`
- 如果在 90 天内到期且当前期间无 last_filed，则为 `due_soon`
- 如果到期日期已过且当前期间无 last_filed，则为 `overdue`
- 如果缺少 formation_date 或州不在参考表中，则为 `unknown`

生成后显示摘要：

```
实体合规跟踪器已初始化。

实体：[N]
总司法管辖区：[N]
跟踪的申报：[N]

状态摘要：
  ✅ Current：   [N]
  ⏰ Due soon：  [N]（接下来 90 天）
  🔴 Overdue：   [N]
  ❓ Unknown：   [N]（与注册代理人确认）

运行 /corporate-legal:entity-compliance --report 查看什么到期。
```

---

## Mode 2: Report

显示即将到期的截止期限并标记逾期项目。默认：接下来 90 天。

```
/corporate-legal:entity-compliance --report [--days 30|60|90|180]
```

输出格式：

```
ENTITY COMPLIANCE REPORT — [date]
[Company Name]

🔴 OVERDUE ([N])：
  [Entity] / [State] / [Filing type] ——于 [date] 到期

⏰ DUE WITHIN [N] DAYS ([N])：
  [Entity] / [State] / [Filing type] ——于 [date] 到期  [registered agent]
  [Entity] / [State] / [Filing type] ——于 [date] 到期

✅ RECENTLY FILED ([N] in last 90 days)：
  [Entity] / [State] / [Filing type] ——于 [date] 申报

❓ UNKNOWN STATUS ([N])：
  [Entity] / [State] / [Filing type] ——无信息；与注册代理人确认

🌐 AGENT-MANAGED ([N])：
  [Entity] / [Country] / [Filing type] ——由 [local agent] 管理；直接确认状态
  [Entity] / [Country] ——未记录当地代理人；使用 --update 添加一个

GOOD STANDING：
  最后确认：[date]
  已确认良好信誉的实体：[N] of [total]
  过去 12 个月内未确认的实体：[list]
```

如果跟踪器涵盖超过约 10 个实体，或任何时候用户询问：提供仪表板（请参阅 CLAUDE.md `## Outputs → Dashboard offer for data-heavy outputs`）。为此输出塑造提议——按申报状态计数（逾期 / 即将到期 / 已申报 / 未知），按良好信誉状态计数，以及带有司法管辖区、申报类型和下次到期日期的可排序实体表。

---

## Mode 3: Update

更新跟踪器中的一个或多个实体。三个子模式：

### Consequential-action gate (file SOI / annual report)

**在指示或确认申报之前：** 阅读 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` 中的 `## Who's using this`。如果角色是 **非律师**：

> 与州务卿提交信息声明、年度报告或特许经营税申报表有法律后果——这是实体的正式陈述，它带有费用，错过或错误的申报可能导致失去良好信誉或特许经营税违约。您在申报前是否与律师（或合格的注册代理人）审查了此内容？如果是，继续记录申报。如果否，这是带给他们的简报：
>
> - 实体、司法管辖区、申报类型和到期日期
> - 跟踪器关于上次申报的说法（日期、费用、上次报告的高级管理人员/董事信息）
> - 未决问题（高级管理人员/董事信息是否仍然准确；注册代理人是否已变更；主要办公室是否已变更）
> - 可能出什么问题（过时的高级管理人员信息、错过触发特许经营税或解散的截止期限、费用计算错误）
> - 问律师什么（今年是否真的需要申报；是否有需要反映的章程修正或高级管理人员变更；谁应该签署）
>
> 如果您需要找到律师、事务律师、大律师或其他授权法律专业人士：联系您的专业监管机构（美国的州律师协会、英格兰和威尔士的 SRA/Bar Standards Board、苏格兰/NI/爱尔兰/加拿大/澳大利亚的 Law Society，或您所在司法管辖区的同等机构）以获取推荐服务。

没有明确的是，不要在此门槛之后记录新的 `last_filed` 日期。跟踪器读取、截止期限报告和"什么即将到期"输出不需要门槛。

### 3a: Manual update

```
/corporate-legal:entity-compliance --update
```

律师告诉 Claude 申报了什么：
> "我们在 3 月 1 日为 [Entity] 提交了特拉华年度报告。费用是 $450。"

Claude 更新：
- `last_filed` → 3 月 1 日日期
- `last_fee` → $450
- `status` → `current`
- 元数据中的 `last_updated`

### 3b: Registered agent report upload

```
/corporate-legal:entity-compliance --update --from-report
```

用户上传 CT Corp、National Registered Agents 或类似的合规报告（PDF、CSV 或 Excel）。Claude 阅读它并更新匹配的实体：

从报告中，为每个实体提取：
- 申报类型和到期日期
- 上次申报日期（如果存在）
- 良好信誉状态和确认日期
- 来自代理人的任何标志或警告

按名称将报告实体与跟踪器实体匹配（标记近似匹配以供确认——"Acme Holdings LLC" vs. "Acme Holdings, LLC" 可能是同一实体）。

处理后：
```
从报告更新了 [N] 个实体。

已匹配：[N]
未匹配（报告中，跟踪器中无）：[list ——可能需要添加到实体表]
报告中无（跟踪器中，无更新）：[list ——状态未变更]
```

### 3c: Bulk status sweep

```
/corporate-legal:entity-compliance --sweep
```

逐个浏览具有 `unknown` 或 `overdue` 状态的每个实体，并询问一次当前信息：

> [Entity] / [State] / [Filing type] ——目前显示为 [status]。
> 这是否已申报？如果是，何时以及费用是多少？

每次确认后更新跟踪器。生成完成摘要。

---

## Mode 4: Health audit

```
/corporate-legal:entity-compliance --audit
```

超出申报状态的更广泛审查。显示：

**申报合规：**
- 逾期项目（来自报告模式）
- 状态未知项目

**实体健康：**
- 标记为 `dormant` 的实体——标记审查：这些应该解散吗？
  维持休眠实体需要花钱（年度费用、注册代理人费用）并产生持续的合规义务。
- 成立时间超过 5 年且状态为 `dormant` 的实体——标记为解散候选。
- 缺少 formation_date 的实体——标记为数据差距。

**良好信誉差距：**
- 没有 `confirmed_good_standing` 日期的实体——未知是否处于良好信誉；如果交易需要短期证书则有风险。
- `confirmed_good_standing` 超过 12 个月的实体——过时；值得刷新，特别是如果预期 M&A 或融资。

**外国资格差距：**
- 基于 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` 实体表：公司的运营足迹（办公室、员工）所在的州是否有实体未外国合格？这需要律师确认运营存在——Claude 可以标记问题但不能独立确定存在。

**公司间协议差距：**
- 从 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md`：如果公司间协议标记为部分或无，标记哪些实体关系可能需要协议（母子公司服务、IP 许可、贷款）。

输出格式：

```
ENTITY HEALTH AUDIT — [date]

FILING COMPLIANCE
  Overdue：[N]
  Unknown status：[N]
  Action：运行 --sweep 以确认未知项目

DORMANT ENTITIES ([N])
  [休眠实体列表及其年龄和已知年度携带成本]
  Dissolution candidates（休眠 >5 年）：[list]

GOOD STANDING
  No record：[N] entities
  Stale（>12 months）：[N] entities
  Consider refreshing before：[任何已知即将到来的交易或合同续签]

POTENTIAL GAPS
  Foreign qualification：[flag question ——确认运营存在于：]
    [来自 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` footprint 未在跟踪器中标记为合格的州列表]
  Intercompany agreements：[来自 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` 的状态]

RECOMMENDED ACTIONS
  1. [最高优先级行动]
  2. [等等]
```

---

## Mode 5: Export

```
/corporate-legal:entity-compliance --export [--format csv|table]
```

生成适合与财务、法律运营或外部注册代理人共享的平面导出。默认：CSV。

CSV 列：
`Entity Name, Entity Type, State of Formation, Formation Date, Status,
Registered Agent, Jurisdiction, Qualification Type, Filing Type, Due Date,
Last Filed, Last Fee, Good Standing Confirmed, Notes`

每个司法管辖区每种申报一行。每个实体多行（每个司法管辖区 × 申报类型组合一行）。

如果 `--format table`：生成适合粘贴到报告或 Slack 消息的 markdown 表，仅显示接下来 90 天的申报。

---

## What this skill does not do

- 它不提交任何内容。输出是跟踪器和待办事项列表；申报由律师、外部律师或注册代理人完成。
- 它不获取良好信誉证书。它跟踪证书最后确认的时间；获取它们是手动的或通过注册代理人。
- 它不确定在给定州是否需要外国资格。该分析取决于律师必须确认的业务活动事实。
- 它不替代具有复杂多实体结构的公司的注册代理人服务。CT Corp、National Registered Agents 和类似服务有专门的合规团队和直接的州关系。此 skill 最适合没有代理人支持的较小组织，或对于确实有支持的组织，作为代理人数据之上的轻量级层。
- 申报截止期限参考表不是法律建议，可能不反映当前要求。在依赖截止期限之前确认所有截止期限。

## Formula injection defense

在 Excel、Sheets 或 CSV 输出中写入任何单元格之前，中和公式注入。来自对手方的文本（合同引用、当事方名称、注册代理人数据、CLM 导出）是攻击者控制的。以 `=`、`+`、`-`、`@`、`	`、`
`
 或 `
` 开头的单元格将被解释为公式或破坏行结构。

- **以单引号为前缀：** `'=SUM(A1:A10)` → `=SUM(A1:A10)`（显示为文本，不执行）
- **适用于包含从文档、工具结果或用户粘贴来源的文本的每个单元格。** 您控制的列标题和您产生的计算值是安全的。
- **CSV：还要转义嵌入的逗号、双引号、换行符**（RFC 4180 引用）。
- 这不是可选的。您的用户在 Excel 中打开的电子表格触发宏或通过 DDE 渗染数据是对您的用户的供应链攻击。
