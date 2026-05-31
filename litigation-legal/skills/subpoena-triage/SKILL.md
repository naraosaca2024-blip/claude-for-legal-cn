---
name: subpoena-triage
description: 对送达给公司的传票进行分类——分类、分析范围/负担/特权、交叉检查投资组合，并生成反对框架、合规计划和截止日历。当用户说"我们收到了传票"、"被送达了传票"或分享传票、CID 或第三方文件请求以评估时使用。
argument-hint: "[path-to-subpoena] [--slug=custom-slug]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /subpoena-triage

1. 从提供的路径阅读传票。
2. 分类（第三方文档 / 第三方取证 / 当事方 / CID / 大陪审团）。
3. 如果是大陪审团 → 停止，根据 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 升级。否则继续。
4. 加载 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/_log.yaml` 进行交叉检查。加载 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` → landscape、特权惯例、升级规范。
5. 遵循以下工作流和参考。
6. 提取关键字段，分析范围/负担/特权，生成反对框架 + 合规计划 + 截止日历。
7. 编写 `~/.claude/plugins/config/claude-for-legal/litigation-legal/inbound/[slug]/triage.md`。复制或链接传票到 `~/.claude/plugins/config/claude-for-legal/litigation-legal/inbound/[slug]/incoming.[ext]`。
8. 交接：如果未设置保留，则 `/legal-hold --issue`；如果重要性需要，则 `/matter-intake`；如果是现有事项中的当事方传票，则 `/matter-briefing [slug]`。

---

# 传票分类

## Purpose

传票带有期限到达。失败模式：错过截止期限、过度生产（特权放弃、本应反对的负担）、生产不足（藐视法庭风险）、或错过动议驳回窗口。此 skill 分类、分析并生成带有反对框架的合规计划。

## Jurisdiction assumption

步骤 0 中引用的规则是此论坛中此传票的适用规则。传票实践实质上各不相同：联邦（FRCP 45）vs. 州同等法规、州与州之间的变体、地方法规、法院特定的常设命令以及传票类型（审判、取证、文档生产）都会改变反对截止期限、遵守地点限制、特权日志要求和成本转移。此处输出的每条规则都是起点启发式——在书面主张之前确认有效性和当地变体。

## Side context

此 skill 本质上是防御性的——传票已送达给接收方，姿态是回应/反对/遵守。阅读执业档案中的 `## Side`。如果用户的默认方是 **原告**，请注意接收传票对原告来说也很常见（证人传票、针对原告自己记录的第三方请求），但这里的框架总是"传票送达给我们，我们如何回应"。如果用户是 **被告**（典型），框架与默认一致。如果事项的姿态与默认不同（例如，辩护从业者在其代表家庭成员 pro se 的事项中收到传票），在继续之前提示用户确认姿态。

## Load context

- 传票文档（用户提供路径或放入会话中）
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/_log.yaml` ——用于相关事项查找和法律保留状态
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` → landscape（我们打交道的监管机构）、内部特权惯例、升级规范

## Workflow

### Step 0: Research the applicable rule

**在分析此传票之前，研究论坛的适用民事诉讼规则（联邦为 FRCP 45，否则为州同等法规）和传票类型（审判、取证、文档生产）。识别：遵守地点限制、反对截止期限（这些通常从遵守日期或送达后固定天数中的较早者开始计算）、特权日志要求、以及谁承担成本。引用精确参考。验证有效性——规则和当地变体会更改。标记大陪审团传票以立即升级刑事法律顾问。**

**没有静默补充。** 如果对配置的法律研究工具（Westlaw、CourtListener、Trellis、Descrybe 或律所平台）的研究查询为论坛的规则、变体或精确引用返回很少或没有结果，报告发现的内容并停止。不要在未询问的情况下从网络搜索或模型知识填充差距。说："搜索从 [tool] 返回了 [N] 个结果。对于 [rule / forum / variant]，覆盖范围似乎很薄。选项：(1) 扩大搜索查询，(2) 尝试不同的研究工具，(3) 搜索网络——结果将标记为 `[web search — verify]`，在依赖前应根据主要来源检查，或 (4) 在此停止。你想要哪个？"律师决定是否接受较低可信度的来源；skill 不为他们决定。

**来源归因。** 在分类输出中，用引用来源标记每条规则引用、案例、法规和条例：`[Westlaw]`、`[CourtListener]`、`[Trellis]`、`[Descrybe]` 或 MCP 工具名称，用于从法律研究连接器检索的引用；`[web search — verify]` 用于来自网络搜索的引用；`[model knowledge — verify]` 用于从训练数据回忆的引用；`[user provided]` 用于用户提供的引用（例如，来自传票或先前事项工作）。标记为 `verify` 的引用具有更高的编造风险，应首先检查。永远不要剥离或折叠标记——它们是律师在反对或备案中主张之前最快了解要验证哪些引用的信号。

### Step 1: Classify

传票有不同规则的类型；根据你刚刚研究的规则确认具体情况：

- **第三方文档传票（民事）**——我们不是诉讼的当事方；有人想要我们的文档。通常反对类别：相关性、负担、特权、遵守地点 / 地理范围。
- **第三方取证传票**——有人希望员工作证。范围、相关性、负担；可能动议驳回；需要证人准备。
- **当事方传票**——我们是当事方；这是我们正在跟踪的诉讼中的发现。视为发现，而不是传入的——它应该映射到现有事项。
- **监管民事调查要求（CID）**——FTC、SEC、DOJ、州总检察长。不同的规则、不同的姿态；通常更尊重，但也更有后果。
- **大陪审团传票**——刑事。立即升级给刑事律师；不同的技能路径（超出此 skill 范围——标记升级）。

### Step 2: Extract key fields

- **发布机构**——法院（哪个）、机构（哪个）、律师（如果是民事）
- **发布方**——谁请求的（如果是民事）
- **案件 / 事项标题**——我们被询问的诉讼
- **寻求的文档类别**——编号列表
- **证言主题**（如果是取证）——规则 30(b)(6) 指定
- **回应/反对截止期限**——送达日期 + 根据适用规则计算回应窗口
- **生产日期**——必须生产文档的日期
- **地理范围**——保管人、地点、涉及的系统
- **记录保管人指定**——公司谁是证人/签署人

### Step 3: Portfolio cross-check

- **当事方传票 → 与现有事项相关：**验证标题匹配 `_log.yaml` 中的事项。如果是，路由到该事项的工作流；此分类仅是信息性的。
- **第三方传票 → 我们不认识的标题：**捕获当事方；记录为独立传入。
- **来自同一案件的多个传票：**标记协调发布；可能适用单一回应策略。

### Step 4: Analyze scope, burden, privilege

**范围 / 相关性**
- 类别是否映射到我们实际拥有的文档？
- 任何类别是否是捕鱼式调查（过于宽泛、未与基础案件的主张/抗辩相关联）？
- 遵守地点 / 地理范围——应用研究的规则；限制因传票类型而异（审判 vs. 文档 vs. 取证）。

**负担**
- 涉及的保管人、搜索的系统、时间段
- 估计数量（粗略：小 / 中 / 大 / 极端）
- 成本——第三方响应者可能有成本转移可用；检查研究的规则。

**特权**
- 可能涉及律师-客户或工作产品？（几乎总是是的，对于任何与法律相关的内容；对于涉及内部或外部律师的通信通常是的）。
- 其他特权——商业秘密、HIPAA（如适用）、州特权、共同利益
- 将需要特权日志——根据 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 标记格式

**其他反对理由**
- 保密性——需要保护命令？
- 重复——他们是否已从另一方获得此内容？
- 未拥有——我们不具备他们要求的内容（具体文档）
- 送达不当——检查研究规则的送达要求

### Step 5: Objections framework

起草结构化的反对大纲——不是最终反对信，而是适用哪些反对意见以及原因的大纲。用户（通常与外部律师一起）最终确定。

每个反对：
- 法律依据——引用步骤 0 中研究的规则的精确引用
- 对此传票的具体适用（哪些类别、哪些保管人）
- 强度（强 / 合理 / 弱）

### Step 6: Compliance plan

即使反对时，我们通常生产请求的部分内容。计划：

- **可能生产的范围**——反对后，我们要生产什么
- **要搜索的保管人**——姓名和系统
- **日期范围**
- **审查协议**——谁审查特权（我们、外部律师、合同审查者）
- **生产格式**——根据传票或根据谈判协议（TIFF+加载文件、原生、PDF）
- **特权日志要求**——格式、字段

### Step 7: Deadlines

使用步骤 0 研究中确定的截止期限。注意反对截止期限通常从遵守日期或送达后固定天数中的较早者开始计算——不要默认为单一数字，而未检查适用规则和当地变体。

- **回应截止期限**——根据研究的规则；注意用户是否需要更多时间（会议和协商以延展是标准的）
- **反对截止期限**——根据研究的规则（联邦 / 州规则 + 任何当地变体）
- **生产日期**——如果没有反对成功
- **动议驳回窗口**——如果追求该路径，时间至关重要

将所有这些都放入日历。立即行动项目。

### Step 8: Write triage

输出：`~/.claude/plugins/config/claude-for-legal/litigation-legal/inbound/[slug]/triage.md`。

```markdown
[工作产品标题——根据 plugin 配置 ## Outputs——因角色而异；参见 `## Who's using this`]

# 传票分类

> **不能替代外部律师。** 这是支持截止期限、保留和参与的快速决策的结构化分类和范围阅读。每条规则引用都是起点启发式；特定司法管辖区分析、反对最终确定、动议实践和特权裁决需要熟悉论坛的持牌律师。对于超出常规第三方文档范围的任何传票，聘请外部律师。

**Slug：** [slug]
**送达：** [YYYY-MM-DD]
**送达给：** [实体 / 注册代理人]
**传入文件：** [path]
**分类：** [third-party-docs / third-party-depo / party / CID / grand-jury]

---

## Key fields

- **发布机构：** [court/agency]
- **发布方：** [name]
- **案件标题：** [caption]
- **回应截止期限：** [date]
- **生产日期：** [date]
- **动议驳回窗口：** [date range]

## Categories sought (summary)

[编号列表，简洁]

## Custodians / systems likely implicated

[列表]

---

## Portfolio cross-check

**相关事项：** [slug 或 "none"]
**如果是当事方传票：** [路由到现有事项还是新事项？]
**如果是第三方：** [独立传入]

---

## Scope & burden analysis

**范围：** [按类别的相关性评估]
**负担估计：** [small / medium / large / extreme ——附推理]
**地理范围问题：** [任何]

## Privilege analysis

*特权范围是首次阅读；最终判断是律师的，而不是此 skill 的。*

**可能涉及律师-客户 / 工作产品：** [yes/no + 哪些类别] `[SME VERIFY]`
**其他特权：** [商业秘密、HIPAA、州、共同利益] `[SME VERIFY]`
**需要的特权日志格式：** [根据 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md`]

---

## Objections framework

*以下每一行在书面主张之前需要 `[SME VERIFY]`——司法管辖区、规则有效性、放弃风险。*

| Objection | Legal basis | Applies to | Strength | SME verified? |
|---|---|---|---|---|
| Relevance | [rule] | [categories] | [strong/reasonable/weak] | [ ] |
| Burden | [rule] | [categories] | | [ ] |
| Privilege | A/C, WP | [all producing docs] | strong (always) | [ ] |
| Duplicative | [rule/doctrine] | [if applicable] | | [ ] |
| [other] | | | | [ ] |

---

## Compliance plan (if responding)

- **可能生产的范围：** [反对后]
- **保管人 / 系统：** [列表]
- **日期范围：** [range]
- **审查协议：** [谁、如何]
- **生产格式：** [format]
- **特权日志：** [format、估计条目]

---

## Deadlines (calendar these)

*以下所有截止期限来自步骤 0 规则研究。`[SME VERIFY]` 确认此论坛和此传票类型的规则、变体和计算——州变体和地方法规各不相同。*

- **回应截止期限：** [date] `[SME VERIFY]`
- **反对截止期限：** [date] ——引用：[rule + pinpoint] `[SME VERIFY]`
- **会议和协商截止：** [date]（通常在反对截止期限之前）`[SME VERIFY]`
- **生产日期：** [date]

---

## Immediate actions

- [ ] 法律保留已发布——[yes/no]——如果否，运行 `/legal-hold [slug] --issue` 传票范围
- [ ] 外部律师已聘请——[yes/who/TBD]
- [ ] 会议和协商已安排——[date]
- [ ] 在日志中创建事项——[yes/no/TBD ——通常对于超过最小第三方文档传票的任何事项都是肯定的]
- [ ] 保险 / 成本转移分析——[如果负担很大]
- [ ] 内部升级——[who]

---

## Recommendation

[两段话：做什么。反对姿态。生产姿态。外部律师处理反对还是我们处理。是否动议驳回。]

---

## Citation verification

此分类中的每条规则引用、案例、法规和条例——包括步骤 0 研究引用、反对依据和特权日志格式指针——都是 AI 生成和未验证的。在依赖任何引用之前（特别是在反对、动议驳回或与发布方或法院的通信中），根据法律研究工具（Westlaw、CourtListener、Trellis、Descrybe 或你的律所平台）运行验证通过，以确保准确性、良好法律地位和当地变体。备案文件中的编造或错误引用引用已导致制裁。每个引用上的来源标签（例如，`[Westlaw]`、`[web search — verify]`）显示它来自哪里；`verify` 标记具有更高的编造风险，应首先检查。
```

### Step 9: Hand off

**在回应传票之前（送达反对、生产文档、出庭取证或提交动议驳回——对发布方或法院的任何实质性回应）：** 阅读 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 中的 `## Who's using this`。如果角色是 Non-lawyer：

> 回应传票有法律后果——错过截止期限有藐视法庭风险、过度生产放弃特权、生产不足有制裁风险。你是否与律师审查了此内容？如果是，继续。如果否，这是带给他们的简报：
>
> [生成一页摘要：传票类型、发布机构、截止期限、寻求内容的范围、反对框架和强度、特权和负担问题、建议回应姿态、可能出什么问题、要问律师什么。]
>
> 如果你需要在你所在的司法管辖区找到持牌律师、事务律师、大律师或其他授权法律专业人士：你所在专业监管机构的转介服务是最快的起点（美国的州律师协会、英格兰和威尔士的 SRA/律师标准委员会、苏格兰/北爱尔兰/爱尔兰/加拿大/澳大利亚的律师协会，或你所在司法管辖区的同等机构）。

没有明确的是，不要越过此门槛继续。分类、范围界定和内部日历不需要门槛——对发布机构的回应需要。

- 如果归类为 **大陪审团传票** → 停止，根据 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 标记升级，不要继续标准分类。
- 如果归类为 **CID**：标记适用监管机构特定规范；建议外部监管律师。
- 否则：提议创建事项（通常是肯定的——传票几乎总是足够重要以跟踪）。
- 如果未发布具有传票范围的法律保留，立即交接给 `/legal-hold --issue`。

## Close with the next-steps decision tree

根据 CLAUDE.md `## Outputs` 结束下一步决策树。自定义选项以适应此 skill 刚生成的内容——五个默认分支（起草 X、升级、获取更多事实、观察等待、其他）是起点，而不是锁定。树就是输出；律师来选择。

## What this skill does not do

- **起草最终反对信。** 生成框架；信由用户 + 外部律师起草（未来：专门的反对起草 skill）。
- **动议驳回。** 提出选项；动议是需要特定司法管辖区分析的法律工作。
- **跨司法管辖区验证规则。** 步骤 0 研究产生此传票的适用规则；skill 不会独立确认有效性或当地变体。在行动前标记律师验证。
- **处理大陪审团传票。** 升级。这超出了分类范围。
