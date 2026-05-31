---
name: diligence-issue-extraction
description: 阅读虚拟数据室（VDR）文档，根据内部类别和重要性阈值提取问题，以内部备忘录格式生成发现。当用户说"审查数据室"、"从 [文件夹] 提取问题"、"尽调审查"、"VDR 里有什么"或指向 VDR 文档时使用。
argument-hint: "[VDR 文件夹路径或类别名称]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /diligence-issue-extraction

1. 加载 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` + `~/.claude/plugins/config/claude-for-legal/corporate-legal/deals/[code]/deal-context.md`。
2. 使用以下工作流。
3. 检查 `ai-tool-handoff`——如果类别是批量且工具已配置，首先交接。
4. 阅读文档，应用重要性过滤器，按类别提取。
5. 以内部备忘录格式生成发现。将同意交接给关闭检查清单。

---

## 事项上下文

**事项上下文。** 检查执业级 CLAUDE.md 中的"## Matter workspaces"。如果 `Enabled` 是 `✗`（内部用户的默认值），跳过本段其余部分——skills 使用执业级上下文，事项机制不可见。如果启用且没有活跃事项，询问："这是哪个事项的？运行 `/corporate-legal:matter-workspace switch <slug>` 或说 `practice-level`。" 为事项特定上下文和覆盖加载活跃事项的 `matter.md`。将输出写入事项文件夹 `~/.claude/plugins/config/claude-for-legal/corporate-legal/matters/<matter-slug>/`。永远不要阅读另一个事项的文件，除非 `Cross-matter context` 是 `on`。

---

## 目的

VDR 有 2,000 份文档。其中有 30 份对交易很重要。此技能根据 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` 中的尽调类别和重要性阈值阅读文档，提取问题，并以内部备忘录格式编写。

## 加载背景

- `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` → 尽调结构（类别、重要性阈值）
- `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` → 问题备忘录格式（如何陈述发现）
- `~/.claude/plugins/config/claude-for-legal/corporate-legal/deals/[code]/deal-context.md` → 交易特定阈值、VDR 位置

如果 deal-context.md 不存在，询问这是哪个交易的。

## 工作流

### 步骤 1：清点 VDR

如果 VDR MCP（Box/Intralinks/Datasite）已连接，拉取索引。将 VDR 文件夹映射到尽调请求清单类别。注意差距——没有相应 VDR 内容的请求清单类别。

```markdown
## VDR 清单：[交易代码]

| 请求类别 | VDR 文件夹 | 文档数 | 状态 |
|---|---|---|---|
| Corporate & Organizational | /01-Corporate | 45 | 已审查 |
| Material Contracts | /02-Contracts | 312 | 进行中 |
| IP | /03-IP | 89 | 未开始 |
| [等等] | | | |

**差距：** [没有 VDR 内容的请求类别——需要后续请求]
```

### 步骤 2：应用重要性过滤器

根据 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` / deal-context 阈值。如果阈值说合同 >$X，则不要审查所有内容。

具体对于合同：按规定价值（如果在文件名/元数据中）或对手方重要性排序。自上而下审查，直到达到阈值或类别耗尽。

### 步骤 3：提取问题

对于阅读的每个文档，根据其类别的标准尽调关注点检查：

**重大合同——标准提取集：**
- 控制权变更条款（由此交易触发？需要同意？）
- 转让限制（合同能否转移给买方？）
- 专有性/竞业禁止（限制买方业务？）
- MFN（最惠国——定价约束）
- 终止权利（对手方能否因交易而退出？）
- 异常赔偿或责任敞口

**公司——标准提取集：**
- 股权表准确性、未行使期权/认股权证
- 交易的董事会同意要求
- 股东协议限制（拖售、随售、优先购买权）
- 子公司结构和公司间安排

**IP——标准提取集：**
- 所有权链（来自创始人/员工的转让是否到位？）
- 产品中的开源（著作权风险）
- 关键 IP 是许可还是拥有
- 未决或威胁的 IP 诉讼

**雇佣——标准提取集：**
- 控制权变更解雇触发（降落伞成本）
- 关键员工留存风险
- 未决雇佣诉讼
- 分类风险（看起来像员工的承包商）

**诉讼——标准提取集：**
- 未决事项和准备金
- 威胁索赔
- 监管询问
- 模式诉讼（消费者集体诉讼等）

### 步骤 4：陈述每个发现

> **来源归因。** 当发现引用法规、条例、案例或监管行动时——例如，在适用法律下分析的控制权变更条款、针对特定原则引用的 IP 所有权差距、带有案例引用的未决诉讼事项——用引用来源标记引用：`[Westlaw]`、`[CourtListener]` 或 MCP 工具名称（用于从法律研究连接器检索的引用）；`[web search — verify]`（用于网络搜索引用）；`[model knowledge — verify]`（用于从训练数据回忆的引用）；`[user provided]`（用于来自 VDR、交易团队备忘录或外部律师反馈的引用）。文档来源引用（VDR 路径、Bates、文件名）保留其原始参考。标记为 `verify` 的引用具有更高的编造风险，应首先检查。永远不要剥离或折叠标记。
>
> **当不同意用户引用的法规时，引用文本或拒绝描述它。** 如果用户（或交易团队备注，或卖方披露）引用法规来支持你认为不正确的主张，并且你没有从连接的研究工具或 VDR 获得法规文本可用，不要编造法规内容的描述。而是说："那个部分不符合我对 [批量销售通知 / 继任责任 / 无论什么] 要求的期望——我需要提取实际文本才能告诉你它实际涵盖的内容。`[statute unretrieved — verify]`"然后要么 (a) 通过配置的研究工具检索文本并引用它，(b) 要求用户粘贴文本，或 (c) 标记为外部律师。对真实法规的自信错误描述比"我不知道"更糟糕——引用编造子章节的交易团队备忘录比差距更难让人不再相信。适用于描述法规的每个 skill，而不仅仅是问题提取。
>
> **没有静默补充。** 如果对配置的法律研究工具的研究查询对于发现所需的法律依据返回很少或没有结果（例如，控制权变更同意要求的规则、IP 转让原则、雇佣分类测试），报告发现的内容并停止。不要在未询问的情况下从网络搜索或模型知识填充差距。说："搜索从 [tool] 返回了 [N] 个结果。对于 [规则 / 原则]，覆盖范围似乎很薄。选项：(1) 扩大搜索查询，(2) 尝试不同的研究工具，(3) 搜索网络——结果将标记为 `[web search — verify]`，在依赖前应根据主要来源检查，或 (4) 标记为未验证并停止。你想要哪个？"律师决定是否接受较低可信度的来源。

根据 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` 中的发现模板。如果种子备忘录使用了这个：

```
Issue #N: [标题]
Category: [请求清单类别]
Severity: [根据内部方案的级别]
Documents: [VDR 路径 + 文档名称]
Finding: [文档说什么以及为什么重要]
Recommendation: [价格调整 / 赔偿 / 需要同意 / 陈述与保证 / 放弃]
```

……然后完全使用那个。如果种子备忘录是项目符号，写项目符号。

**严重性校准**（如果内部方案是 R/Y/G）：
- 🔴 **红色：** 影响交易价值或结构。需要主要客户同意的控制权变更。未披露的重大诉讼。IP 所有权差距。
- 🟡 **黄色：** 需要关注，可解决。需要同意但可能获得。需要补救的开源。雇佣分类风险。
- 🟢 **绿色：** 为文件记录。与陈述一致。除陈述外无需行动。

### 步骤 5：按类别组装

按请求清单类别分组发现。在类别内，按严重性排序。

```markdown
[工作产品标题——根据 plugin 配置 ## Outputs ——因角色而异；参见"## 谁在使用"]

> 此输出源自具有特权、机密或两者的 VDR 材料。它继承来源的特权和机密状态——超出特权圈的分发可能放弃特权。与事项的特权文件一起存储，并有意做出分发决定。

# 尽调问题：[交易代码] — [类别]

**已审查文档：** 类别中的 [M] 中的 [N]
**覆盖范围：** [全部 | >$X 阈值 | 前 N]
**发现：** [N]🔴 [N]🟡 [N]🟢

---

### 结论

[🔴 N 个阻塞 · 🟠 N 个高 · 🟡 N 个中等] — [交易团队需要知道的一件事]

---

[以内部格式呈现的每个发现]

---

## 差距

- [没有响应文档的请求清单项目]
- [已引用但不在 VDR 中的文档]
```

## Handoffs

- **To ai-tool-handoff:** If Luminance/Kira is in use per `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md`, hand bulk contract review there. This skill handles the nuanced documents (side letters, amendments, anything the AI tool struggles with).
- **To deal-team-summary:** Aggregated findings feed the deal team brief.
- **交给 material-contract-schedule：** 合同级别的提取输入披露清单。
- **交给 closing-checklist：** 任何暗示离散关闭前行动的发现都会成为检查清单项目。交接不仅限于第三方同意——它还涵盖：
  - **股东投票 / 其他关闭行动**——§280G 清洗投票、要求的股东同意、要求的董事会决议、评估权通知期、转换机制，或交易完成所需的任何其他公司批准。表征行动、批准阈值、法定或章程来源以及时间约束。
  - **监管申报和批准**——HSR、CFIUS、外国投资审查、提取期间标记的行业特定批准。
  - **来自对手方的同意**——控制权变更、反转让、MFN 触发同意。
  - **解除、终止或清偿**——与控制权变更相关的雇佣解除、清偿信、留置权解除。
  - **托管 / 扣留机制**——如果提取 surfaced 赔偿托管、R&W 保险交付物或与特定问题相关的扣留。
  每个带有关闭前行动标签的发现都应该到达 closing-checklist，而不仅仅是标记为"同意"的那些。如果发现处于灰色区域（可能需要关闭行动，可能是关闭后承诺），标记并交接——如果购买协议另有说明，closing-checklist 可以放弃它。交接不足是单向门；交接过度在审查中纠正。


**继任责任。** 标记：未决或威胁的侵权/产品责任索赔、环境问题和清理义务、批量销售/欺诈转让敞口（卖方是否保留足够资产支付其余债权人？）、卖方关闭后解散计划（如果卖方解散，原告会追逐买方），以及购买协议是否有实际覆盖已知敞口的假设/除外责任清单。即使在资产交易中，"事实合并"、"仅仅是延续"和"产品线"原则也可以转移责任——这是让认为他们正在干净购买资产的买方客户感到惊讶的分析。

## 批量处理

对于大型类别（300 份合同），分批处理。每批之后，更新运行中的问题列表并立即标记任何 🔴 项目——不要等待整个类别来 surfaced 影响交易的问题。

## 以下一步决策树结束

根据 CLAUDE.md `## Outputs` 的下一步决策树结束。自定义此技能刚生成的内容的选项——五个默认分支（起草 X、升级、获取更多事实、观察等待、其他）是起点，而不是锁定。树就是输出；律师来选择。

如果提取 surfaced 超过 ~10 个问题，或任何时候用户询问：提供仪表板（请参阅 CLAUDE.md `## Outputs → Dashboard offer for data-heavy outputs`）。为此输出塑造提议——按严重性计数（🔴 / 🟠 / 🟡 / 🟢）、按内部类别计数，以及带有重要性、类别和 VDR 来源的可排序问题网格。

## 此技能不做什么

- 它不对接近的案例做出重要性判断。它应用阈值；人类决定边界。
- 它不谈判陈述和保证。它生成告知它们的发现。
- 它不替代批量 AI 审查。对于大批量条款提取，根据 `~/.claude/plugins/config/claude-for-legal/corporate-legal/CLAUDE.md` 交接给 Luminance/Kira。此 skill 用于判断层。
