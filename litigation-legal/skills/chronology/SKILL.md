---
name: chronology
description: 从声明的文档来源和上传构建或更新时间线——提取日期事件、去重，并根据事项理论标记重要性。当用户要求从生产或事项文件构建时间线或时间线、说"chron from the production"或"what happened when"，或需要工作、事实陈述或特定证人的时间线时使用。
argument-hint: "[slug] [--format=working|sof|witness-[name]]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /chronology

1. 加载 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md` → 理论、枢纽事实、关键事实。
2. 加载 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` → 文档存储来源、默认事项文件夹模式。
3. 遵循以下工作流和参考。
4. 按顺序识别来源：本次会话中用户提供的路径、默认事项文件夹、从 `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` 声明的来源。
5. 对于可读来源：提取日期事件。对于无法访问的来源：在差距中注明。
6. 去重，根据事件与来源列表合并。
7. 根据事项理论标记重要性 (🔴/🟡/⚪)。
8. 编写 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/chronology.md`（或根据标记的格式变体）。
9. 如果存在先前版本：版本号递增，向用户展示差异摘要。
10. 在完成之前确认："这是我构建的内容。扫描 🔴 条目——我误分类了什么吗？"

---

# Chronology

## 披露文档使用限制

在处理诉讼文档集之前，询问："这些文档中的任何一份是否通过法律程序中的披露或发现获得？"如果是：

- **英格兰和威尔士 (CPR 31.22)：** 通过披露获得的文档受默示承诺约束——您只能将它们用于披露它们的程序的目的，除非法院许可、披露方同意，或文档已在公开法庭上阅读。在未经许可的情况下将它们用于不同事项、不同索赔或商业目的是蔑视法庭。
- **美国：** 保护命令和规则 26(c) 可能施加类似的限制。请检查命令。
- **其他司法管辖区：** 类似的限制通常适用。请检查当地规则。

确认："此使用在披露这些文档的程序范围内，或我有许可/同意，或文档现在公开。"如果未确认，请标记："⚠️ 披露的文档可能有使用限制。在继续之前确认此使用是否允许。"

## 目的

事实按顺序发生。时间线是每个叙述所依赖的脊柱——简报中的事实陈述、保留备忘录、和解备忘录、证词准备、证人准备。手工构建时间线很慢；AI 擅长于结构化提取。问题：垃圾进，垃圾出。此 skill 从配置声明的来源和用户上传的任何内容中提取。

## 模式

此 skill 服务于两种执业设置。从插件配置 CLAUDE.md 中用户的 `## Role` 中选择默认值；用户可以使用标志每次运行时覆盖。

- **`--matter` 模式（内部诉讼律师的默认）。** 以事项历史为中心。从 `matter.md` 阅读事项的案件理论和关键事实，从声明的文档存储来源（Google Drive、SharePoint、Gmail、iManage、CLM——无论 CLAUDE.md 的 `## Landscape` 部分声明什么）提取，并将 `history.md` 视为运行内部日志（决定、保留、保留备忘录——有意不在时间线中）。输出以事项为中心：跨争议发生的事情，标记用于倡导使用。
- **`--documents` 模式（律所律师助理或律师的默认）。** 以生产文档为中心。从配置中读取案件理论，然后从 eDiscovery 导出、保管人文件集或 Bates 编号的生产中提取。输出以生产为中心：文档显示什么，带有 Bates 引用，根据案件理论标记。

两种模式汇聚于相同的输出结构（时间线、🔴/🟡/⚪ 重要性标签、差距、SoF 变体）。区别在于来源概况和重要性框架。

如果 `## Role` 是 `solo` 或 `other`，默认为 `--matter` 但在第一次运行时提及两种模式并让用户选择。

## 方框架（重要性标签）

同一事件根据从业者是在证明主张还是在反驳主张而具有不同的重要性。阅读执业档案中的 `## Side`（以及事项覆盖默认值时的逐事项姿态）：

- **原告（进攻性框架）** — 🔴 标记*确立*主张要件（责任、因果关系、损害、通知）的事件，*封闭*被告方试图打开的差距的事件，或以有利于原告的方式*启动*诉讼时效时钟的事件。🟡 标记支持主张但可能被质疑的事件。⚪ 是背景上下文。
- **被告（防御性框架）** — 🔴 标记*打破*主张要件（因果关系失败、通知、信赖）的事件，*开启*诉讼时效或管辖权抗辩的事件，或*支持*积极抗辩（免除、弃权、风险承担、比较过错）的事件。🟡 标记削弱原告诉讼叙事的事件。⚪ 是背景。
- **两者兼有 / 因事项而异** — 按时间线询问用户应用哪一方的框架进行重要性标记。基础时间线是中立的；只有重要性解读会改变。

在输出顶部注明应用的框架：`重要性标签从 [原告 / 被告] 视角应用。` 当生成事实陈述变体时，使用方默认值，除非用户另有指定。

## 加载上下文

通用：
- 插件配置 CLAUDE.md → 案件理论上下文（内部法务：`## Landscape` 用于文档来源；律所律师：`## Case theory` 和 `## Document review` 用于平台 + 保管人）、`## Outputs` 用于工作产品标题、`## Decision posture` 用于特权标记规则。
- 此事项的先前 `chronology.md`（如存在）。
- 用户上传的任何文件或会话中提供的路径。

`--matter` 模式还读取：
- `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md` → 案件理论、关键事实、转折事实（用于重要性标记）、关键日期。
- CLAUDE.md 中的默认事项文件夹模式 → 此 slug 的文档存放位置。

`--documents` 模式还读取：
- 电子取证平台元数据（如果有可用的连接器：Everlaw、Relativity、DISCO、Aurora）——按保管人 + 日期范围。
- Bates 范围清单或制作索引（如果用户指向一个）。

**冲突门槛——不可绕过（`--matter` 模式）。** 在构建时间线之前，检查 `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/_log.yaml` 中的事项 slug。如果事项不在 `_log.yaml` 中，拒绝并路由：

> "我在事项日志中看不到 [matter slug]。首先运行 `/litigation-legal:matter-intake`，以便冲突检查运行并设置事项工作区。我不会对未接收的事项构建时间线——冲突检查是门槛。"

不要在未接收的事项上继续。接收是运行冲突并写入此 skill 读取的 `_log.yaml` 行的过程。`--documents` 模式（在没有事项 slug 的情况下对临时文档集运行）不受此门槛限制，但其输出应被视为诉前研究，不应作为事项工作产品归档。

## 工作流

### 步骤 0：特权门槛（每次首先运行）

时间线工作从文档中提取。文档通常享有特权（律师-客户、工作产品、共同利益、联合辩护）——内部法务事项文件通常默认享有；电子取证制作，特别是滚动制作或共同利益制作，通常包含享有特权或未审查的材料。从享有特权的文档中提取内容到后来被分享的时间线中可能*有风险*放弃特权，取决于谁接收它以及在什么原则下（共同利益、联合辩护、Kovel 和工作产品保护可能适用）。放弃分析是事实特定的——在分发前获得律师签署。

skill 在用户选择特权姿态之前不会提取：

> 在提取之前：来源是如何进行特权筛查的？
>
> - **A. 所有来源已清除** — 您已经筛查了这些。我在没有特权标记的情况下提取。输出是发现就绪姿态；仍然标记为工作产品。
>
> - **B. 混合或尚未筛查** — 我提取并用 `priv` 标志标记每个条目：`ok`（来自明确非特权材料）、`flag`（来自可能享有特权的材料——A/C、WP、共同利益）或 `review`（来源不明确）。标记的条目在输出中视觉标记，事实陈述变体默认过滤掉它们。
>
> - **C. 中止——先筛查** — 暂停 skill。筛查来源。返回并重新运行。

在时间线标题中记录选择为 `privilege_posture: A-cleared | B-mixed | C-aborted`。如果是 B 或 C，简要记录理由。

**为什么是门槛而不是警告：** 警告被阅读一次就会被遗忘。门槛将姿态决定强制纳入记录，这意味着每个时间线文件都带有自己的来源——任何后来阅读它的人都知道条目是否来自特权筛查过的材料。

### 步骤 1：识别文档来源

**`--matter` 模式：**

1. **用户提供的路径** — 本次会话中放入的任何内容（文件路径、驱动器链接、电子邮件导出）。
2. **默认事项文件夹** — 来自 CLAUDE.md 的文档存储模式，为此 slug 展开（例如，`G:/Legal/Matters/acme-v-us-2026`）。
3. **声明的来源** — CLAUDE.md 中的 `Document storage` 表，筛选为此事项可能涉及的（例如，发件方通信的 Gmail 归档、SharePoint 法务文件夹）。
4. **询问** — 如果来源看起来很薄，提示："我可以从我有的内容构建，但时间线将不完整。还有其他要指向我的吗？关键电子邮件、合同、内部备忘录、制作信函？"

**`--documents` 模式：**

1. **制作导出 / Bates 集** — 用户指向制作目录或清单；skill 按 Bates 范围 + 日期读取。
2. **电子取证连接器** — 如果 MCP 连接器可用（Everlaw、Relativity、DISCO、Aurora），按保管人 + 日期范围拉取。
3. **保管人文件** — 如果用户提供原始保管人邮箱或驱动器导出，也读取那些。
4. **询问** — 如果对关键保管人或日期范围的覆盖看起来很薄，提示。

### 步骤 2：拉取 + 阅读

对于每个有可读文件的来源：

- **PDF、电子邮件（.eml）、.docx、.txt** — 直接读取。
- **电子邮件归档（Gmail、Outlook）** — 如果 MCP 连接器已认证，按日期范围 + 对方 / 关键词查询；否则用户将相关线程导出到文件夹。
- **电子取证平台（Everlaw、Relativity、DISCO、Aurora）** — 如果连接器可用，按保管人 + 日期范围拉取；否则用户提供导出。

如果 skill 无法访问声明的来源，在输出的差距部分明确命名它，而不是静默继续。

**没有静默补充。** 如果事项某个时期的来源覆盖很薄——声明的時間窗口中文档少于预期、保管人邮箱不可访问、制作尚未到达——报告发现了什么并停止。不要在未询问的情况下从网络搜索、公共记录搜索或模型知识填补差距。说："来源为 [期间 / 保管人] 返回了 [N] 个事件。覆盖似乎很薄。选项：(1) 指向我的其他来源（Bates、文件夹、邮箱），(2) 尝试不同的 MCP 连接器（如已配置），(3) 在网络中搜索此窗口的公共记录事件——结果将标记为 `[web search — verify]`，在依赖前应根据主要来源检查，或 (4) 在此停止并注明差距。你想要哪个？"律师决定是否接受较低可信度的来源；skill 不为他们决定。

**来源归因。** 用事件来源标记每个时间线条目：文件路径、Bates 号码、MCP 连接器或声明的文档存储来源（已在来源列中捕获）。对于任何无法追溯到检索文档的事件或日期——例如，从模型训练数据回忆的事实、通过网络搜索找到的公共记录事件——内联标记：`[web search — verify]`、`[model knowledge — verify]` 或 `[user provided]`（用户在会话中陈述的事实）。标记为 `verify` 的条目比文档来源条目具有更高的编造风险，应首先检查。永远不要剥离或折叠标记——它们是律师在将条目拉入简报或事实陈述之前最快验证哪些条目的信号。

**标记覆盖陈述法律结论、截止期限或计算日期的每个部分——不仅仅是时间线条目。** 时间线来源于文档。差距部分、关键事件部分、理论联系线，以及任何关于诉讼时效、 tolling 事件、提交截止日期、发现截止或特权裁定的陈述，都是 skill 从模型知识编写的法律分析，除非有来源。每个此类陈述都带有来源标记：`[computed from: <rule cited with tag>]`、`[model knowledge — verify]`、`[user provided]` 或研究连接器标记（如果在此会话中检索到）。没有标记的诉讼时效窗口默认为 `[model knowledge — verify]`。将事实的法律重要性表征为"关键事件"的行是分析，需要标记。规则很简单：如果是关于法律的断言，而不是关于文档说了什么的断言，它必须携带与时间线条目相同的来源标记。当没有研究连接器可达且 skill 正在计算截止期限或引用规则时，在审阅者注释的 **Sources:** 行中记录（见插件 CLAUDE.md `## Outputs`）——不要发出独立的横幅。

### 步骤 3：提取事件

对于每个文档，识别有日期的事件：

- **电子邮件：** `[日期] [发件人] 告知 [收件人] [主题/内容]`
- **会议：** `[日期] [参会者] 会议讨论 [主题]`（根据日历条目或笔记）
- **决定：** `[日期] [决策者] 决定 [什么]`（根据记录文档）
- **提交/诉状：** `[日期] [当事方] 提交 [动议/起诉状/答复]`
- **外部事件：** `[日期] [发生了什么]`（合同签署、产品发布、监管机构行动、事件越过阈值）

通常每个文档一个事件。偶尔零个（无日期或未确立事件）。有时多个（涵盖多个决定的会议摘要）。

**每个条目的特权标记（仅当 privilege_posture == B-mixed 时）。三态规则——永远不静默决定主观特权测试未通过：**

- `priv: ok` — 来源**确信**不享有特权（提交文件、监管通信、公开文档、无我方律师的对方通信）。仅在不存在合理特权理论时使用。
- `priv: flag` — 来源确信或可能享有特权（与律师的通信、工作产品备忘录、享有特权的草稿、联合辩护材料）。**任何不确定事项的默认值** —— 如果主导目的判断很接近，或诉讼考量处于边界，或内容混合，它应该在这里，而不是在 `ok`。
- `priv: review` — 来源表面上不明确，但 skill 无法做出判断（无发件人/收件人元数据、不可读等）。

当 `priv: flag` 或 `priv: review` 时，内联添加 `[SME VERIFY: privilege status]` 以便律师在审查时看到。标记不足会放弃特权（单向门）；标记过度由律师在审查中纠正（双向门）。优先选择可恢复的错误。

### 步骤 4：去重

同一事件在多个文档中出现：一个会议在三个日历上并产生一封摘要邮件——那是**一个有四个来源的事件**，不是四个事件。合并。合并的条目引用所有来源。

### 步骤 5：标记重要性——按案件理论

从 `matter.md`（`--matter` 模式）或配置的 `## Case theory` 部分（`--documents` 模式）阅读转折事实和关键事实。标记每个事件：

- 🔴 **关键** — 事件是转折事实或对我方有利/不利的关键事实的一部分
- 🟡 **相关** — 上下文、模式证据、支持次要论点
- ⚪ **背景** — 对完整性有用，不会写入简报

**纪律：** 300 个条目都标 🔴 的时间线等于没有标记。将 🔴 保留给真正会影响事实认定者的事件。如果不确定，用 🟡。

**边界标记：** 当条目处于 🔴 和 🟡 之间（或 🟡 和 ⚪ 之间）时，标记为较低重要性并内联添加 `[SME VERIFY — borderline significance call]`。律师的判断将覆盖 skill 的判断。自信过度标记的时间线不如浮现不确定性的时间线有用。

### 步骤 6：写入

默认输出是工作时间线。按请求提供变体。

## 输出格式

### 工作时间线（默认）

位置：`~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/chronology.md`。完整、标记、注释。律师工作的参考文档。

```markdown
[WORK-PRODUCT HEADER — per plugin config ## Outputs — differs by role; see `## Who's using this`]

> **Privilege inheritance.** This chronology is derived from matter documents that may be attorney-client-privileged, work-product-protected, common-interest / joint-defense material, or a mix. It inherits the sources' protection status. Distributing it beyond the privilege circle — to business stakeholders outside the engagement, to opposing counsel, to a regulator — can waive protection over both the chronology and the underlying sources. Store with privileged matter material, mark consistently with house privilege conventions, and make distribution decisions deliberately. The privilege-posture choice captured below is the provenance stamp for any later distribution call.

# Chronology — [Matter Name]

> Significance tags (🔴/🟡/⚪) and privilege flags (🔒) are first-pass reads requiring `[SME VERIFY]` before use in any external work product (briefs, SoF, board memo, outside counsel deliverable).

**Matter:** [slug]
**Mode:** matter | documents
**Built:** [YYYY-MM-DD]
**Sources:** [N] documents across [source types]
**Entries:** [N] ([N] 🔴 / [N] 🟡 / [N] ⚪)
**Pivot fact:** [one sentence]
**Privilege posture:** A-cleared | B-mixed | C-aborted
**Flagged entries:** [N] 🔒 *(only present when posture == B-mixed)*

---

## Timeline

| Date | Event | Tag | 🔒 | Sources |
|---|---|---|---|---|
| [YYYY-MM-DD] | [what happened, one sentence] | 🔴/🟡/⚪ | [blank / 🔒-flag / 🔒-review] | [file paths or Bates] |

---

## Key events (🔴 only)

[Pulled out, each with a line on why it matters to the theory.]

### [date] — [event title]
- What: [one line]
- Theory tie: [why this matters]
- Sources: [list]

---

## Gaps

**Date ranges with no events:**
[ranges — where are documents for this period?]

**Expected but missing:**
[events we'd expect to see documented but don't — e.g., "contract amendments between 2024-06 and 2025-03 — not produced"]

**Unreadable sources:**
[sources declared in CLAUDE.md but not accessible this run — e.g., "Everlaw production — no MCP connector; export needed"]

---

## Marker discipline

- `[VERIFY: factual assertion — date, attendees, content]` — not yet confirmed against the underlying doc
- `[UNCERTAIN: legal characterization — e.g., whether an event establishes a regulatory trigger]`
- `[CITE NEEDED: Bates / exhibit / depo page:line]`
- `[SME VERIFY: privilege status | borderline significance call]` — counsel judgment needed

---

## Version
- v[N] built on [date] from [source summary]
- v[N-1] built on [date] (prior, superseded)
```

### 事实陈述时间线（按请求）

仅筛选 🔴 和相关 🟡。按时间顺序叙述以散文形式呈现——简报事实部分的骨架。每段是一个事件或紧密关联的集群，附有记录引用。

**特权过滤器默认值：** 当 `privilege_posture == B-mixed` 时，🔒-标记和 🔒-审查条目默认**排除**。SoF 变体旨在最终用于外部用途（简报、披露、与对方谈判）——🔒 条目在律师确认特权状态之前不属于那里。如果用户仍然想要包含 🔒 条目，需要明确的 `--include-flagged` 确认；在输出标题中捕获确认作为永久记录。

### 特定证人时间线（按请求）

筛选以命名证人为发件人、收件人、参会者或主题的事件。供证人准备使用，帮助重建证人何时知道什么。

## 增量构建

如果 `chronology.md` 存在：

- 阅读先前版本
- 从当前来源构建新时间线
- 差异：新事件（自上次构建以来）、修改的条目（向现有事件添加了新来源）、删除的条目（罕见；注明原因）
- 保留先前版本号；用 `v[N+1]` 写入新版本
- 输出变更摘要

## 与 matter.md / history.md 的集成

**有意分离**（内部法务 `--matter` 模式）。`history.md` 是律师的运行日志——决定、更新、程序里程碑、内部策略笔记。`chronology.md` 是面向倡导的事实时间线。它们重叠但不合并：

- 发布了保留通知 → 进入 history.md（内部行动）。通常不在时间线中（不是争议的事实）。
- 对方在 3 月 14 日发送了违约通知 → 进入 chronology.md（🟡——确立其知情）。如果 intake 引用了它，也在 history.md 中。
- 我们的储备建议备忘录已起草 → 仅在 history.md。

当律师想要将历史事件放入时间线时，可以粘贴它们。默认是保持分离。

## 此 skill 不做什么

- **解决矛盾。** 当两个文档对事件发生时间说法不同时，两个条目都放入并标记。解决是律师的判断；可能需要证人访谈或进一步发现。
- **发明来源中不存在的事件。** 如果不在文档中（也不在 matter.md 或配置中作为捕获的事实），就不在时间线中——但"差距"可能会将其标记为缺失。
- **保证完整性。** 时间线只与来源一样好。如果电子取证制作正在进行且只有 20% 到达，时间线反映这一点。注明限制。
- **为用户决定特权状态。** 步骤 0 门槛强制姿态选择；每个条目的 `priv` 标记捕获首次分类。实际的特权裁定是律师根据 `[SME VERIFY]` 标记的判断。
